---
title: "Lessons learned porting Octopus Server to .NET Core 3.1"
description: "We ported Octopus Server to .NET Core 3.1 to unlock the ability to run on Linux, Docker Containers, and Kubernetes. This post shares lessons learned along the way."
author: rob.pearson@octopus.com
visibility: public
bannerImage: eskimo-octopus-linux-land.png
bannerImageAlt: Lessons learned porting Octopus Server to .NET Core 3.1
metaImage: eskimo-octopus-linux-land.png
published: 2020-03-19
tags:
- Engineering
---

![Lessons learned porting Octopus Server to .NET Core 3.1](eskimo-octopus-linux-land.png)

With the release of Octopus 2020.1, Octopus Server now runs on .NET Core 3.1, which means it can be installed on Linux, Docker Containers, and Kubernetes. This was a significant effort, we already shared our [reflections on the launch of Octopus Cloud 1.0](/blog/2019-10/octopus-cloud-1.0-reflections/index.md) and [why we chose Kubernetes, Linux, and .NET Core for Octopus Cloud 2.0](/blog/2019-11/octopus-cloud-v2-why-kubernetes/index.md), in this post, we share the benefits of the change and our top three lessons learned.

## Benefits

Before we share the lessons learned, here are the benefits of porting Octopus Server to .NET Core:

* Modern development environment, framework, and tooling. 
* Cross-platform support for Windows and Linux.
* Access to the mature ecosystem for running Linux containers in Kubernetes.
* Choice and flexibility: Our customers can choose to run Octopus on Windows or Linux, or they can use Octopus Cloud.
* Octopus Cloud has gained improved performance and reduced operating costs. See the links above for concrete numbers.

This also makes our development environment more productive as our engineering teams now have the choice to develop on Windows or Linux. 

## Top three lessons learned

We learned a lot going through this process; however, there are three major lessons we learned.

### 1. Know and plan for differences between Windows and Linux

There are platform-specific differences in the implementations of .NET Core on Windows and Linux. Most of the issues were small and had easy workarounds, but we did find a few significant problems that are worth sharing.

**Configuration settings and the Windows registry**

To support both Windows and Linux platforms, we had to remove any Windows-specific code. Octopus Server started as a Windows product, and it followed the platform conventions and stored some configuration settings in the Windows registry, which is a problem for Linux.

**Solution**: 

We shifted everything stored in the registry to the file system or the Octopus database. This was a simple task, but it took time and testing to get it right.

**Database performance problems**

The most significant problem we encountered was abysmal database performance due to the different way of handling database queries on Windows and Linux. Octopus uses Microsoft SQL Server as its data store, and we discovered a [significant problem](https://github.com/dotnet/SqlClient/issues/422) in the .NET Core SQL client library on Linux. If we have the `MultipleActiveResultSets` setting set to `True` we get exceptions and database timeouts. The GitHub issue linked above shares the full details and a simple code sample to reproduce the problem. 

**Solution**:

Our short-term solution was to disable the `MultipleActiveResultSets` setting and use it very sparingly. In general, we open two connections to our database, one with this setting enabled and one with it disabled. We primarily use the disabled connection and only use the enabled one when it is required.

We have been working with Microsoft to help provide information to resolve the issue, and we hope to see a proper fix in the future. 

**Authentication providers**

We also encountered the need to host the Octopus Server web host differently on each platform. We use `HttpSys` on Windows and _Kestrel_ on Linux, and this made our authentication challenging. Octopus needs to support multiple authentication schemes, including cookies-based authentication, and the ability for users to log in and out and have multiple authentication providers enabled at once.

The core issue we hit was that `HttpSys` supports integrated authentication (i.e., Windows authentication), but its a binary on/off setting for every endpoint in the host. This is inflexible, and it's a change from our non-.NET Core code-base. Users could log in automatically, but they could never log out. 

Note: We don't use Kestrel on Windows because it doesn't support virtual directories and we have customers that share the same port with other services. So to ensure we maintain backwards compatibility, we decided to use `HttpSys` for Windows only.

**Solution**: 

We considered several options, but after going through this [ASP.NET Core issue](https://github.com/dotnet/aspnetcore/issues/5888), we decided to follow the advice there and use two hosts. One standard web host and a second one to look/behave like a virtual directory off the main API site's root, i.e., `/integrate-challenge`, and is therefore consistent with the location in earlier versions of Octopus Server. The host only has that one route, and it initiates the login challenge using a 401 response when the user isn't already authenticated.

### 2. Sharpen your debugging skills for Linux and Docker

As we progressed through the .NET Core port, we also learned how to code, test, and debug problems with Windows, Windows Subsystem for Linux (WSL), Linux, and Docker. Historically, our team all developed on Windows, but this has evolved into individuals coding on Windows, Linux, and macOS, and as a result, we've learned several lessons:

**Running as non-root or non-admin**

We found it much easier to run all builds and test Octopus Server on Linux as non-root or non-sudo to limit permission-based errors and problems. That said, we sometimes need to run commands as root using `sudo` and then afterwards fix up the ownership of files that were created during those commands with `chown`, like `sudo chown -R $user:$user ~/Octopus/MyInstance`. This is a bit quick and dirty, but it does the job. 

We are planning to change this in future, such that octopus runs as a user that is a member of a group, and during installation we configure the group owner of `/etc/octopus` to be that group. 

**Certificate management**

Our end-to-end (E2E) test suite runs a range of tests that run against the Octopus Server listening over HTTPS (i.e., port 443). This requires us to convert and import some self-signed certificates into the local certificate store in `/etc/` on a Linux box. To solve this problem, we wrote the following script:

```bash
#!/bin/bash
echo "Setup certificates"
CERTS_PATH_DEST="/usr/local/share/ca-certificates"
if [ ! -d "$CERTS_PATH_DEST" ]
then
    echo "Creating $CERTS_PATH_DEST"
    mkdir ${CERTS_PATH_DEST}
fi
MY_PATH="`dirname \"$0\"`"
CERT_PFX_FILES=(${PWD}/${MY_PATH}/../Octopus.E2ETests/Universe/*.pfx)
for CERT_PFX in "${CERT_PFX_FILES[@]}"
do
    FILE_NAME=$(basename "$CERT_PFX")
    CERT_CRT="${CERTS_PATH_DEST}/${FILE_NAME%.*}.crt"
	if [ ! -e "${CERT_CRT}" ]
    then
        echo "Converting '${CERT_PFX}' to '${CERT_CRT}'"
        openssl pkcs12 -in "${CERT_PFX}" -clcerts -nokeys -out "${CERT_CRT}" -passin pass:password
    fi
done
update-ca-certificates
```

**Connecting to the SQL Server database**

Octopus uses Microsoft SQL Server as its database, and teams generally connect to it via integrated Windows authentication on Windows Servers. This no longer works. Our solution here was to switch to user name and password-based authentication. 

Further, we found our suite of end-to-end (E2E) tests runs much faster and more reliable with database connection pooling turned off. We haven't gotten to the bottom of this yet, but it's likely a platform-specific problem related to the database performance issue mentioned above.

**Debug Octopus Server on Linux with Visual Studio Code**

Our team writes code with a variety of tools, including the following:

* [Visual Studio](https://visualstudio.microsoft.com/vs/)
* [JetBrains Rider](https://www.jetbrains.com/rider/)
* [Visual Studio Code](https://code.visualstudio.com)

Currently, the most popular is [Visual Studio Code](https://code.visualstudio.com/) with the [Remote Development extension](https://aka.ms/vscode-remote/download/extension). This extension is still in preview, but we find it works very well.

With Visual Studio Code and the Remote Development extension, we can run applications, test, and debug them as well as edit code in Linux (or a Docker container).  Simply point VS Code at the folder that contains the Octopus Server code, and then it's pretty much just F5 from there. It's that simple!

### 3. Simplify with self-contained packages

Porting Octopus to .NET Core has allowed us to ship [self-contained packages](https://www.hanselman.com/blog/MakingATinyNETCore30EntirelySelfcontainedSingleExecutable.aspx) which brings multiple benefits.

- **Fewer dependencies**: Shipping a single self-contained executable means we no longer require .NET Core to be installed on the Octopus server. The result is reduced installation requirements that make Octopus easier to install. This is a big win.
- **Improved supportability**: In a nutshell, fewer dependencies make Octopus easier to install and support. There are fewer components and fewer things that can be accidentally changed. Shipping Docker container images for [Windows](https://hub.docker.com/r/octopusdeploy/octopusdeploy) and Linux (coming soon) eliminates further dependencies as even more of the dependencies are built into the containers. 
- **Modern software and tooling**: Using modern tools and frameworks enables our team to continue to innovate and ship software quickly with useful features for our customers. 

Unfortunately, this also has some tradeoffs as moving to .NET Core 3.1 required us to [drop support](/blog/2019-11/raising-minimum-requirements-for-octopus-server/index.md) for older operating systems, including Windows Server 2008-2012 and some Linux distros. Supporting older servers and browsers drains our time and attention, making it harder for us to innovate and move the Octopus ecosystem forward. 

## Conclusion

We ported Octopus Server to .NET Core 3.1 so that the server can run on Linux, Docker Containers, and Kubernetes. We made this change to reduce costs and increase the performance of our Octopus Cloud SaaS product, and it has been a great success. 

It wasn't a simple journey, but we learned a lot on the way.

1. Know and plan for differences between Windows and Linux
2. Sharpen your debugging skills for Linux and Docker
3. Simplify with self-contained packages

## Related posts

- [Octopus 2020.1: Dark mode and better Linux support](/blog/2020-03/octopus-release-2020-1/index.md)
- [Introducing dark mode](/blog/2020-03/octopus-dark-mode/index.md)