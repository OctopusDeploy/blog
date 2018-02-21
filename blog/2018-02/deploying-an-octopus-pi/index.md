---
title: "Using Octopus Deploy to deploy .NET Core applications to a Raspberry Pi"
description: "You can use Octopus to deploy your .NET Core applications to a Raspberry Pi."
author: ben.pearce@octopus.com
visibility: public
metaImage: metaimage-raspberrypi.png
bannerImage: blogimage-raspberrypi.png
published: 2018-02-21
tags:
 - Walkthrough
---
![Octopus enjoying a Raspberry Pi](blogimage-raspberrypi.png)

.NET Core has come a long way in the last few years, and Octopus Deploy has too.  A while back, we added support for running [Calamari without Mono](https://octopus.com/blog/octopus-release-3-16#ssh-targets-sans-mono), and in this post I will walk you through how to deploy .NET Core applications on to a Raspberry Pi 3, no Mono required.

In this post, I will show you that it is possible to deploy and run .NET Core applications on the Raspberry Pi 3, and along the way describe some of the different ways that you can interact with your Octopus Deploy server.

## Requirements Before Starting

* Editor - Visual Studio, Visual Studio Code, Rider
* [Octopus Command Line](http://octopus.com/downloads).
* [Octopus Server](http://octopus.com/downloads) and an [API key](https://octopus.com/docs/api-and-integration/api/how-to-create-an-api-key).
* Dotnet Core - https://www.microsoft.com/net/download/windows, https://www.microsoft.com/net/download/macos.
* A Raspberry Pi 3 running [Raspian](https://www.raspberrypi.org/downloads/raspbian/), with dotnet core 2.0 Runtime [installed](https://github.com/dotnet/core/blob/master/samples/RaspberryPiInstructions.md).
    * Download link: [Linux ARM (armhf)](https://github.com/dotnet/core-setup).
* For Angular or React applications:
    * node and npm on your development machine - if your chosen application requires it (angular or react).
    * nodejs on your Pi. 
* [Curl](https://curl.haxx.se/download.html)

:::hint
ASP.NET includes NodeServices in its bundle which requires Node to be installed before it can serve any requests. When you install Node.js on the Raspberry Pi, it installs version 4.x and the executable is called `nodejs`, but NodeServices is looking for `node` in your path. You can fix this by creating a symlink: `sudo ln -s /usr/bin/nodejs /usr/bin/node`
:::

## Build the Application

### Create a Basic .NET Core Application
```powershell
dotnet new angular
```

### Modify the Application to Listen For External Requests
By default, an ASP.NET Core application will only serve requests to `http://localhost:5000`, to allow the web host to serve requests to your local network, add the following after the `.UseStartup<Startup>()` in `Program.cs`:
```c#
.UseKestrel(options => {
    options.Listen(System.Net.IPAddress.Any, 5000);
})
```

For more information on configuring the Kestrel Web Host, check the [docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore2x).

### Build the Application
```powershell
npm install
dotnet build
mkdir publish
dotnet publish -o publish --self-contained -r linux-arm
```

### Package It Up
The simplest way to create a package of a .NET Core application is using the `Octo.exe` command line tool.

Create an `artifacts` directory and then use the `Octo Pack` command to create the package:

```powershell
mkdir artifacts
octo.exe pack --id core4pi --version 1.0.0 --format zip --outFolder artifacts --basePath publish
```

Using Octo.exe again, push the package to the server:

```powershell
octo.exe push --server http://octopus/ --apikey API-ABCDEF123456 --package artifacts\core4pi.1.0.0.zip
```

## Building a Service Definition
To get the application to run as a service, see Microsoft’s documentation for [hosting .NET Core on Linux](https://docs.microsoft.com/en-au/aspnet/core/host-and-deploy/linux-nginx?tabs=aspnetcore2x).

Create a file called `core4pi.service` containing the following text:
```text
[Unit]
Description=core4pi

[Service]
WorkingDirectory=#{Octopus.Action[deploy web site].Output.Package.InstallationDirectoryPath}
ExecStart=/usr/local/bin/dotnet "#{Octopus.Action[deploy web site].Output.Package.InstallationDirectoryPath}/core4pi.dll"
Restart=always
RestartSec=10
User=pi
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

:::info
The `[deploy web site]` strings in the `core4pi.service` text above, represents the name of the step that deploys the package.
This output variable will contain the path to the newly installed service. This will ensure when the service is installed it is looking at the latest version.
:::

Create a package for the service definition and push it to the Octopus Server:
```powershell
octo.exe pack --id core4pi.service --version 1.0.0 --format zip --outFolder artifacts
octo.exe push --server http://octopus/ --apikey API-ABCDEF123456 --package artifacts\core4pi.service.1.0.0.zip
```

## Create Infrastructure
If you don't already have an Octopus environment configured for your Raspberry Pi, create one either at the command line:
```powershell
octo.exe create-environment --server http://octopus/ --apikey API-ABCDEF123456 --name "Pi Dev"
```

Or using the web interface via *Infrastructure* > *Environments* > *Add Environments*.

![](create-new-environment.png "width=500") 

Next, create an account to access the Pi, this can either be a Username / Password or an SSH Key in the *Infrastructure* > *Accounts* section in the web interface.

![](pi-account.png "width=500")

Then finally, create a deployment target under *Infrastructure* > *Deployment Targets* > *Add Deployment Target* as an **SSH target**.
Set the targets role to something that represents the responsibility of the target, e.g `PiWeb`.
After filling in the details (IP Address or DNS name, SSH port and account), under the .NET section, ensure that you select _Mono not installed_, don't worry about the platform, we will change that later.

![](dotnet-not-mono.png "width=500")

### Modify the Target Config to Specify the Calamari Version as `linux-arm`
This code can easily be run from [LinqPad](http://www.linqpad.net/)

```c#
string machineId = "Machines-1";
HttpClient client = new HttpClient();
client.BaseAddress = new Uri(@"http:\\octopus");
client.DefaultRequestHeaders.Add("X-Octopus-Apikey", "API-ABCDEF123456");
var machineJson = client.GetAsync($"api/machines/{machineId}").Result.Content.ReadAsStringAsync().Result;
machineJson = machineJson.Replace("linux-x64","linux-arm");
client.PutAsync($"api/machines/{machineId}", new StringContent(machineJson));
```

The machine id, defined on the first line, can be obtained from the Web Portal URL when viewing the deployment target `app#/infrastructure/machines/_machineId_/settings` or using the command line:
```powershell
octo list-machines --server http://octopus/ --apikey API-ABCDEF123456
```

You can also filter the list at the command line by using the JSON output format and filtering in Powershell:
```powershell
octo list-machines --server http://octopus/ --apikey API-ABCDEF123456 --outputformat=json | 
    ConvertFrom-Json | 
    % { $_ } |
    Where { $_.Name -eq 'target name' }
```

:::info
The `% { $_ }` line unwraps the top-level array that is being returned, which seems to be a quirk of the `ConvertFrom-Json` command in Powershell.
:::

### Download Calamari for linux-arm

```powershell
curl https://octopus.myget.org/F/octopus-dependencies/api/v2/package/Calamari.linux-arm/4.3.6 -L -o "c:\Program Files\Octopus Deploy\Octopus\Calamari.linux-arm.nupkg"
```

Replace the output path with the path to your Octopus Installation, if required.

:::info
The `linux-arm` Calamari package, will be provided in future releases
:::

## Creating the Deployment Project
Create a new Project via the {Projects} section in the Octopus web interface, or using the command line:

```powershell
octo create-project --server http://octopus/ --apikey API-ABCDEF123456 --name "PiWeb" --projectgroup "All projects" --lifecycle "Default Lifecycle"
```

### Create a Deployment Step For the Application
In the new PiWeb project, define your deployment process. 

Add a `Deploy a Package` step, called `deploy web site`. 

:::hint
The step name here will allow the values in the service definition file to be updated correctly.
:::

![](deploy-package-step-library.png)

Set the **Environment** to the `Pi Dev` environment.

Set the **Role** to the `PiWeb` role (or whatever you set the SSH target role to).

Under the **Package** section, select the package that you pushed to the server, `core4pi`.

The rest of the options in here don't need to be configured. *Save it*.

### Create a Deployment Step For the Service Definition
Add another `Deploy a Package` step. This one will install a service on the target to run the application.
For the package selection, select the `core4pi.service` package from the **Octopus Server (built in)** package feed.

![](service-installation-step.png "width=500")

You will need to `Configure Features` for this step:

![](feature-configuration.png "width=500")

In the `Substitute Variables in Files` feature add the name of the service definition file `core4pi.service`:

![](substitute-variables-in-service.png "width=500")

Under the `Configuration Scripts` feature, select **Bash**, paste the below script in to the `Deployment Script` section:
```bash
#!/bin/bash
if [ -e /lib/systemd/system/core4pi.service ]
then
    echo stopping service
    sudo systemctl stop core4pi.service
fi

echo installing service
sudo cp core4pi.service /lib/systemd/system/
sudo chmod 644 /lib/systemd/system/core4pi.service
sudo systemctl daemon-reload
sudo systemctl enable core4pi.service
echo starting service
sudo systemctl start core4pi.service
```

This script will be executed during the step execution and performs the service installation.

## Deploy It

On the Project navigation menu, press **Create Release**. 

The **Create Release** page will allow you to set a version number for the release, you can just leave the default. It will also allow to pick which versions of the packages you want to deploy, by default it will pick the latest version.

Press **Save** and then press **Deploy to PI Dev** and then **Deploy** to start the deployment process.

**Create Release** can also be performed from the command line

```powershell
octo create-release --server http://octopus/ --apikey API-ABCDEF123456 --project "PiWeb"
octo deploy-release --server http://octopus/ --apikey API-ABCDEF123456 --project "PiWeb" --deployto="Pi Dev" --version "0.0.1"
```

:::info
The first time you deploy, Octopus Server will update Calamari on the target machine, this may take a couple of minutes.
:::

## Test It
After the deployment has finished, navigate to the IP address or DNS name of your Raspberry Pi on port 5000, you should see the application

![](its-alive.png "width=500")


## Conclusion

With the alignment of a number of different technologies, deploying .NET to a Raspberry Pi is possible, and Octopus Deploy makes it painless. Throughout this post, you have also seen a number of different ways that you can integrate with your Octopus server, including command line, API and the web portal.
 
If you are interested in automating the deployment of your .NET Core applications, [download a trial copy of Octopus Deploy](https://octopus.com/downloads), and take a look at [our documentation](https://octopus.com/docs/deploying-applications).
