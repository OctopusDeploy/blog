---
title: "Lessons learned porting Octopus Server to .NET Core 3.1"
description: "TODO"
author: rob.pearson@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2020-02-05
tags:
- Engineering
---

With the release of Octopus 2020.1, Octopus Server is now running on .NET Core v3.1 unlocking the ability to run on Linux, Docker Containers and Kuberentes. This was a significant effort and in this post, we'll share why we ported Octopus Server to .NET Core, the benefits of the change and our top three lessons learned.

!toc

## Why

`tl;dr` We ported Octopus Server to .NET Core so it could run on the Linux platform, Docker containers and Kubernetes to reduce costs and increase performance of our Octopus Cloud SaaS product.

---

In July 2018, we launched Octopus Cloud, our hosted version of Octopus, for customers who prefer to use an online version of Octopus without having to worry about the infrastructure to run it. This first release of Octopus Cloud runs on Amazon Web Services, and each customer has an instance that executes in a dedicated virtual machine. Rather than rebuild our entire product as a multi-tenant SaaS product, we decided the simplest minimum viable product (MVP) was to run each customer instance in their own VM. This structure allowed us to bring the product to market quickly, and it provided an isolated, secure, and stable solution for our customers. We've [written about the journey](/blog/2019-10/octopus-cloud-1.0-reflections/index.md), if you're interested in learning more. That said, this approach was quite expensive to run, it had numerous components that we had to support, and we regularly hit AWS services limits. We are proud of bringing this to market, but it was clear we needed to iterate.

In 2019, we kicked off the effort to to maintain the positive aspects of our first iteration, but also tackle the big problems.

- **Reduce costs:** Especially for low-use or dormant instances.
- **Increase performance:** Improve provisioning times, and provide more options for highly utilized instances.

We [evaluated numerous options](/blog/2019-11/octopus-cloud-v2-why-kubernetes/index.md) including the following. 

1. Single multi-tenant server
2. Windows process per customer
3. Azure App Services
4. Kubernetes

In the end, we decided **Octopus would be built against .NET Core, run on Linux, be containerized, and orchestrated by Kubernetes.**

> We decided the effort to port Octopus to .NET Core was effort we _wanted to spend_. 

> We could roll our own orchestration solution, but Kubernetes was built for the problem we were trying to solve. We’ve always preached using Octopus rather than trying to roll your own deployment automation, so you can spend the time saved on making your core software better.  This was a chance to heed our own advice.

> It was also an exciting chance to drink our own champagne again. We could take advantage of the Kubernetes support we’d built into Octopus at a large scale.

This launched in late 2019 and our customers have been deploying and executing runbooks on this platform ever since. This entire journey has been a learning experience and we're continuing to iterate and improve. 

In the future, we will be publishing both Windows and Linux contains to run Octopus on-prem fully supported.

### Benefits

Porting Octopus Server to .NET Core has brough a number of benefits. 

* Modern framework and tooling. 
* Cross platform support for Windows and Linux.
* Access to the mature ecosystem for running Linux containers in Kubernetes.
* Choice and flexibility: our customers can chose to run Octopus on Windows, Linux or via Octopus Cloud.
* Octopus Cloud has gained improved performance and reduced operating costs

This also brings a richer development environment as our engineering teams have the choice to develop on Windows or Linux. 

## Top 3 Lessons Learned

We learned a lot going through this process however we have three major lessons learned.

### 1. Platform specific differences

The first and foremost lesson we learned is there are platform specific differences in the implementations of .NET Core on Windows and Linux. Most of the issues were small differences or problems and the vast majority had easy work arounds. That said, we did find a few larger problems that are worth sharing.

The takeaway from this process is that each time will encounter their own unique set of problems and some may be easier to solve than others.

**Configuration settings and the windows registry**

The most immediate change we had to make to support both Windows and Linux platforms was to remove any Windows specific code. Octopus Server started out as a Windows product and it followed the platform conventions and storing some configuration settings in the Windows registry. This is fine on Windows but Linux is very different.

Solution: 

We had to shift everything stored in the registry to the file system or Octopus database. This is a simple task but it took time and testing to get right.

**Multiple Active Result Sets**

The most significant problem we encountered was extremely poor database performance due to different handling of database queries on Windows and Linux. Octopus uses Microsoft SQL Server as its datastore and we discovered a [significant problem](https://github.com/dotnet/SqlClient/issues/422) in the .NET Core SQL client library on Linux. If we have the `MultipleActiveResultSets` setting set to True, the result is exceptions and database timeouts which is problematic. The GitHub issue linked above shares the full details and a simple code sample to reproduce the problem. 

Solution: 

Our solution in the short term is to disable the `MultipleActiveResultSets` setting and use it very sparingly. In general, we open two connections to our database, one with this setting enabled and one with it disabled. We primarily use the disabled connect except for specific instances where we need it to be true. 

We have been working with Microsoft to help provide information to resolve the issue and we hope to see a proper fix in the future. 

**Authentication providers**

Another thing we enountered was a need to host the Octopus Server web host differently on each platform. We use `HttpSys` on Windows and _Kestrel_ on Linux and this produced challenges for our authentication. Octopus needs to support multiple authentication schemes including cookies based authentication, and the ability for users to log in/out and have multiple authentication providers enabled at once.

The core issue we hit was that HttpSys supports integrated authentication (i.e. Windows authentication) but it's a binary on/off for every endpoint in the host. This is inflexible and it's a change from our non .NET Core code-base. The end result of this is that a user could logged in automatically and then they could never log out. 

Solution: 

We considered a number of options but after going through this [ASP.NET Core issue](https://github.com/dotnet/aspnetcore/issues/5888), we decided to follow the advice there and use two hosts. One standard web host and and a second one to look/behave like a virtual directory off the main API site's root, i.e. `/integrate-challenge`, and is therefore consistent with the location in earlier versions of Octopus Server. The host only has that one route and it initiates the challenge, using a 401 response, when the user isn't already authenticated.

Solution: 

* Windows auth on Linux.
Routing - Run two websites listening on the same URL and port and handles things nicely. - Found on the Internet. 

### 2. Learning how to debug .NET Core on Linux and Docker

Another thing we needed to learn as we progressed through the port is that we needed to debug problems from time to time. We have unit tests and an extensive suite of end-to-end (E2E) tests but we still needed to debug problems that we couldn't figure out. This proved to be an interesting topic that we thought we'd share.

## How to debug Octopus Server on WSL (and Docker containers)

I found it easier to run all Octopus Server setup as non `sudo`, unfortunately there is an area that Octopus Server writes and reads that you need to explicitly add permissions to. We write to `/etc/octopus` this is where we add the instances config files and masterkey, this is the equivalent of `c:\Octopus` in Windows but in Linux OS `/etc` is a root privileged folder, so we need to manually `sudo chown -R $USER /etc/octopus`. We will make this part of the installation script one day!

Like any good software we have quite a few tests and as part of our integration testing we test the usage of  [Polling Tentacles over WebSockets](https://octopus.com/docs/infrastructure/deployment-targets/windows-targets/polling-tentacles-web-sockets) and that requires us to listen on https. So as part of the setup for this test we have to add some self signed certificates to the trusted store, so we use the following script:

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

In Linux we also found that we obviously cannot use integrated security when connecting to Sql Server but the one that has given us more grief is connection pooling, for some reason we found our testing a lot more reliable when we explicitly turn **pooling off**.

## So now to debug Octopus on Linux

I found Visual Studio 2020 and Rider quite not there yet, even though they do support attaching to processes via SSH. The tool I settle on (at least for now) is [Visual Studio Code](https://code.visualstudio.com/) with the [Remote Development extension](https://aka.ms/vscode-remote/download/extension). This extension is still in Preview but it works really well.

So with this Remote Development extension I can start debugging, running and code edit in VSCode running in WSL (or a Docker container), all I have to do is point it to my Windows folder that contains Octopus Server code, and pretty much just F5 from there. It is that simple!

### 3. Shipping self-contained packages

Octopus 

Talk about supporting deploying self-contained packages

Context

Why? 

Benefits.

Caveats - tradeoff - drop support for older systems. Better for staying modern and innovating etc. Link to other blog post. 
Competitor started today, they wouldn't support old operation systems. 

(stay relevant)

Business decision to move forward. 

Docker is self-contained thus it's a a very supportable platform.

## Conclusion

We ported Octopus Server to .NET Core 3.1 to unlock the ability to run on Linux, Docker Containers and Kuberentes. This change was made to reduce costs and increase performance of our Octopus Cloud SaaS product and it's been a great success. 

It wasn't a simple journey but we learned on the journey.
1. Platform specific differences
2. Learning how to debug .NET Core on Linux and Docker
3. Shipping self-contained packages
