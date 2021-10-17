---
title: Traditional Jenkins Installation
description: Learn how to install Jenkins via the traditional installers
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

The traditional method of installing Jenkins is via the installers made available on the [Jenkins website](https://www.jenkins.io/download/) or using your local operating system's package manager.

The installation process is generally simple, but there are a few tricks to be aware of. In this post we'll run through the installation of Jenkins on Windows and Linux, and provide some insights into customizing the installation.

## Installing Jenkins on Windows

Jenkins provides an MSI download that allows it to be installed as a Windows service through the traditional Windows wizard style installation process. But before we start the installation there are a number of prerequisites we must address.

### Installing OpenJDK

Jenkins requires Java to to run. In recent years Oracle changed the licensing terms of their Java Runtime Environment (JRE) and Java Development Kit (JDK) to restrict commercial usage to paying customers. Fortunately, the OpenJDK project provides a free and open source alternative that you can use to run Jenkins.

There are many OpenJDK distributions to choose from including [OpenJDK](https://openjdk.java.net), [AdoptOpenJDK](https://adoptopenjdk.net), [Azul Zulu](https://www.azul.com/downloads/), [Red Hat OpenJDK](https://developers.redhat.com/products/openjdk/download), and more. I typically use the Azul Zulu distribution, although any distribution would do.

Download and install JDK 11 from your chosen OpenJDK distribution, and make a note of the directory it was installed to as you'll need that during the Jenkins installation.

### Adding a Jenkins Windows service account

Jenkins runs as a Windows service, and to do so requires a Windows account to run the service under. The installer provides the option to use the existing [LocalService](https://docs.microsoft.com/en-us/windows/win32/services/localservice-account) account, but notes that this option is not recommended. So it is recommended that you create a new account specifically for running Jenkins.

To do this task from the command line you must first install the Carbon PowerShell module. Carbon provides many useful CMDLets for managing Windows, and you'll use one of these to grant the new Jenkins user the rights to log on as a service.

Run the following PowerShell command to install Carbon from the PowerShell Gallery:

```powershell
Install-Module -Name 'Carbon' -AllowClobber
```

By default PowerShell will prevent you from running code from an external source. To remove this warning, run the following command:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

You then import Carbon:

```powershell
Import-Module 'Carbon'
```

The next step is to create a user called `jenkins` to run the Jenkins Windows service:

```powershell
net user jenkins Password01! /ADD
```

Finally, you must grant the `jenkins` user the permission to log on as a service:

```powershell
Grant-CPrivilege -Identity "jenkins" -Privilege SeServiceLogonRight
```

Start by downloading the MSI from the [Jenkins download page](https://www.jenkins.io/download/)

### Installing Jenkins

Double-click the MSI file to begin the Jenkins installation. Click the **Next** button:

![Jenkins Windows Installer](win-install-1.png "width=500")

The default installation directory is fine, so click the **Next** button:

![Jenkins Windows Installer](win-install-2.png "width=500")

You are now prompted to select the user that runs the Windows service. Enter the credentials for the user you created earlier and click the **Test Credentials** button. Once the test passes, click the **Next** button:

![Jenkins Windows Installer](win-install-3.png "width=500")

The default port of **8080** is fine. Click the **Test Port** button to ensure the port is available, and then click the **Next** button:

![Jenkins Windows Installer](win-install-4.png "width=500")

You are now prompted to enter the path to the Java distribution you installer earlier. The default path for the Zulu 11 distribution is `C:\Program Files\Zulu\zulu-11`. Enter the appropriate path for you chosen distribution, and click the **Next** button:

![Jenkins Windows Installer](win-install-6.png "width=500")

You will likely want to expose Jenkins through the Windows firewall to allow external clients to access it. Change the **Firewall Exception** feature to be installed, and click the **Next** button:

![Jenkins Windows Installer](win-install-7.png "width=500")

All the installation values are now configured, so click the **Install** button:

![Jenkins Windows Installer](win-install-8.png "width=500")

Once the installation is complete, click the **Finish** button:

![Jenkins Windows Installer](win-install-9.png "width=500")