---
title: "Octopus Tentacle on ARM/ARM64"
description: "Tentacle now supports ARM/ARM64 TODO"
author: ben.pearce@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2021-12-31
tags:
- Product
---

TODO: Find a graphic.

We're pleased to share that our Tentacle agent now supports ARM and ARM64 hardware. This update makes it possible to deploy your apps and services to Raspberry Pi 3 & 4, AWS A1 EC2 instances, or any ARM hardware that can run [.NET Core 3.0+](https://devblogs.microsoft.com/dotnet/announcing-net-core-3-0/#platform-support). 

In this post, I'll explain why it's valuable to run Tentacle on ARM servers, I'll show you how to get started plus I'll share additional resources so you can do bigger and better things. 

## Why Tentacle on ARM/ARM64? 

Back in late 2019, we introduced support for ARM-based deployment targets and workers as an SSH connection option.* The [Linux Tentacle](https://octopus.com/downloads/tentacle#linux) has been around for a while now, but only supported `x86_64/amd64` based machines, leaving ARM adopters stuck with SSH.

Connecting over SSH is fine, but doesn't work for everyone, such as highly secure environments where port 22 is a no-no. By installing a Polling Tentacle on your ARM device you can avoid having to open a firewall port for an SSH connection, which is especially important if you are using the Octopus Hosted service. 
Please see our [previous blog post](https://octopus.com/blog/tentacle-on-linux) which goes into further detail about the benefits of using a Linux Tentacle for deployments rather than SSH.

Running your workloads on ARM hardware has some benefits:
- Lower running cost
- Low-cost replaceable units in the case of the small form-factor manufacturers such as Raspberry Pi.
- Faster compute compared to x86 servers.

*** insert link here to back up some of these claims

\* _Technically, it was available earlier than this but there were some awkward hoops you had to jump through._

## Getting Started

All the instructions for getting Tentacle running on Linux are available in our [docs](https://octopus.com/docs/infrastructure/deployment-targets/linux/tentacle) but I will provide a walkthrough of the steps. At the end, we will have a Raspberry Pi inside a private network, connected to an Octopus Hosted instance.

For this example, I am running [Fedora 33 Server](https://getfedora.org/en/server/download/) on a Raspberry Pi 3B+ and have signed up for a [hosted Octopus instance](https://octopus.com/start/cloud).

Before we install the Tentacle application, you will need the Octopus Server URL and an API key for authentication.
* If you are using an on-prem Octopus Server instance, you can use **Username / Password** for authentication.

Installing the Tentacle package is straightforward:

```bash
sudo wget https://rpm.octopus.com/tentacle.repo -O /etc/yum.repos.d/tentacle.repo
sudo install tentacle
```

At the end of the installation, you will see the following message
```
To set up a Tentacle instance, run the following script:
    /opt/octopus/tentacle/configure-tentacle.sh
```

There are sample scripts available on the [Linux Tentacle documentation](https://octopus.com/docs/infrastructure/deployment-targets/linux/tentacle) if you need to make an automated, repeatable install but for now, we will run the configuration script and mostly just take the defaults.

For this example, it is important to select **Polling** (2) for the kind of Tentacle. The [Polling Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/windows-targets/tentacle-communication#polling-tentacles) will make a connection out to the Octopus Server so we don't need to open up any additional ports in the firewall.

```bash
[root@fedora ~]# /opt/octopus/tentacle/configure-tentacle.sh

Name of Tentacle instance (default Tentacle):
Invalid characters will be ignored, the instance name will be: 'Tentacle'

What kind of Tentacle would you like to configure: 1) Listening or 2) Polling (default 1): 2
Where would you like Tentacle to store log files? (/etc/octopus):
Where would you like Tentacle to install applications to? (/home/Octopus/Applications):
Octopus Server URL (eg. https://octopus-server): https://***.octopus.app
Select auth method: 1) API-Key or 2) Username and Password (default 1): 1
API-Key: ...
Select type of Tentacle do you want to setup: 1) Deployment Target or 2) Worker (default 1): 1
What Space would you like to register this Tentacle in? (Default):
What name would you like to register this Tentacle with? (fedora): fedorapi
Enter the environments for this Tentacle (comma seperated): test
Enter the roles for this Tentacle (comma seperated): pi

The following configuration commands will be run to configure Tentacle:
sudo /opt/octopus/tentacle/Tentacle create-instance --instance "Tentacle" --config "/etc/octopus/Tentacle/tentacle-Tentacle.config"
sudo /opt/octopus/tentacle/Tentacle new-certificate --instance "Tentacle" --if-blank
sudo /opt/octopus/tentacle/Tentacle configure --instance "Tentacle" --app "/home/Octopus/Applications" --noListen "True" --reset-trust
sudo /opt/octopus/tentacle/Tentacle register-with --instance "Tentacle" --server "https://***.octopus.app" --name "fedorapi" --comms-style "TentacleActive" --server-comms-port "10943" --apiKey "API-XXXXXXXXXXXXXXXXXXXXXXXXXX" --space "Default" --environment "test"  --role "pi"
sudo /opt/octopus/tentacle/Tentacle service --install --start --instance "Tentacle"
Press enter to continue...

Creating empty configuration file: /etc/octopus/Tentacle/tentacle-Tentacle.config
Saving instance: Tentacle
Setting home directory to: /etc/octopus/Tentacle
A new certificate has been generated and installed. Thumbprint:
9B691824225B6A77AB68...
These changes require a restart of the Tentacle.
Removing all trusted Octopus Servers...
Application directory set to: /home/Octopus/Applications
Tentacle will not listen on a port
These changes require a restart of the Tentacle.
Checking connectivity on the server communications port 10943...
Connected successfully
Registering the tentacle with the server at https://***.octopus.app/
Detected automation environment: NoneOrUnknown
Machine registered successfully
These changes require a restart of the Tentacle.
Service installed: Tentacle
Service started: Tentacle

Tentacle instance 'Tentacle' is now installed
```

After the script has finished configuring the tentacle, you will be able to see the Linux Tentacle in the Deployment Targets page on your instance.

![Deployment target](deployment-target.png "width=200")

The next thing will be to run something against our new Tentacle.
For this step, I am going to setup a new project and configure a runbook to install the latest package updates.

In your instance create a new _Project_, I called mine **Pi 🥧**

![New project](project.png "width=200")

I then created a new _Runbook_ called **Upgrade it!** and added a single script step containing:
```bash
sudo yum upgrade -y
```

The step is configured to run a _bash_ script against the role **pi**, which is the role I specified in the configuration script earlier.

![Script step](script-step.png)

The beauty of running against a role, is you could have many targets with the same role and the execution will run against each of these. Hook up a schedule trigger and you have yourself a real DevOps process.

## Next steps

*** need to fill this in a bit more.

## Conclusion

There are many reasons that you might be using ARM hardware from cost savings to performance or remote IoT devices. Being able to connect them to an Octopus instance via Tentacle allows you to deploy application updates to them or manage them using our [Runbooks](https://octopus.com/docs/runbooks) feature to centralise management of the operating system and applications.
