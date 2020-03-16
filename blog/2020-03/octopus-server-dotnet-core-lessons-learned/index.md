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

* Cross platform support for Windows and Linux.
* Access to the mature ecosystem for running Linux containers in Kubernetes.
* Choice and flexibility: our customers can chose to run Octopus on Windows, Linux or via Octopus Cloud.
* Octopus Cloud has gained improved performance and reduced operating costs

This also brings a richer development environment as our engineering teams have the choice to develop on Windows or Linux. 

## Top 3 Lessons Learned

We have learned a ton while going through this process however we have three major lessons learned.

### 1. Platform specific differences

We encountered a number of large and small platform specific differences during the porting process. Most of the issues were small differences or problems and the vast majority had easy work arounds. That said, we did find a few problems that are worth sharing.

**Configuration settings and the windows registry**

Octopus Server started out as a Windows service running on Windows Servers and it followed the Windows convention by storing configuration settings in the Windows registry. This is fine on Windows but Linux has very different conventions and so we had to move all settings to be stored on the filesystem or in the Octopus database. 

**Multiple Active Result Sets**

The most significant problem we faced was poor database performance due to different handling of database queries on Windows and Linux. This GitHub issue linked above shares the details and a simple code sample to reproduce the problem 

We have hit one mojor problem 

* MARS <- SQL server - performance problems on *nix

* Ask shannon about perf w/ something on Auth/Azure Linux - Active Directory 
Windows auth on Linux. We needed granular control over auth providers not just all or nothing. 

Solution.

* MARS - Turn off support. Rely 
We open two connections and pick the one that suits. 

5 areas that still have MARS. 

Still workin with microsoft to resolve but we don't have a concrete timeline. 

* Windows auth on Linux.
Routing - Run two websites listening on the same URL and port and handles things nicely. - Found on the Internet. 

Examples. 

### 2. Learning how to debug .NET Core on Linux and Docker

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

I found Visual Studio 2019 and Rider quite not there yet, even though they do support attaching to processes via SSH. The tool I settle on (at least for now) is [Visual Studio Code](https://code.visualstudio.com/) with the [Remote Development extension](https://aka.ms/vscode-remote/download/extension). This extension is still in Preview but it works really well.

So with this Remote Development extension I can start debugging, running and code edit in VSCode running in WSL (or a Docker container), all I have to do is point it to my Windows folder that contains Octopus Server code, and pretty much just F5 from there. It is that simple!

### 3. Shipping self-contained packages

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

We ported Octopus Server to .NET Core 3.1 to unlock the ability to run on Linux, Docker Containers and Kuberentes. This change was made to reduce costs and increase performance of our Octopus Cloud SaaS product and it's been a great success. It wasn't a simple journey and we shared our top three lessons learned. 
1. Platform specific differences
2. Learning how to debug .NET Core on Linux and Docker
3. Shipping self-contained packages
