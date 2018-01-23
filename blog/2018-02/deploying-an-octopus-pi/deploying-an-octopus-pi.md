---
title: "Using Octopus Deploy to deploy .Net Core applications to a Raspberry Pi"
description: ""
author: ben.pearce@octopus.com
visibility: private
published: 2018-02-??
tags:
 - Walkthrough
---

API Key API-6FGRLBN3XYXMMXWM70B9D9BLI6Y


Requirements before starting:
Editor - Visual Studio, Visual Studio Code, Rider
Octopus Command Line - http://octopus.com/downloads
Octopus Server and API key
Dotnet Core - https://www.microsoft.com/net/download/windows, https://www.microsoft.com/net/download/macos
A Raspberry pi 3 with dotnet core installed
node and npm on your development machine - if your chosen application requires it (angular or react)
nodejs on your Pi 
    - ASP.NET includes NodeServices in its bundle which requires Node to be installed before it can serve any requests. When you install Node.js on the Raspberry Pi, it installs version 4.x and the executable is called `nodejs`, but NodeServices is looking for `node` in your path. I was able to fix this by creating a symlink: `sudo ln -s /usr/bin/nodejs /usr/bin/node`

Create a basic .Net Core Application
```
dotnet new angular
```

Modify the application to listen for external requests.
Add the following after the `.UseStartup<Startup>()` in `Program.cs`
```
.UseKestrel(options => {
                    options.Listen(System.Net.IPAddress.Any, 5000);
                })
```

Now build the application:
```
npm install
dotnet build
mkdir publish
dotnet publish -o publish --self-contained -r linux-arm
```

Package it up
```
octo.exe pack --id core4pi --version 1.0.0 --format nupkg
```

Send it on its way
```
octo.exe push --server http://localhost:8085 --apikey API-6FGRLBN3XYXMMXWM70B9D9BLI6Y --package core4pi.1.0.0.nupkg
```

Create Infrastructure:

If you don't already have an environment configured for your Raspberry Pi, create one:
```
octo.exe create-environment --server http://localhost:8085 --apikey API-6FGRLBN3XYXMMXWM70B9D9BLI6Y --name "Pi Dev"
```
Or use the web interface via {Infrastructure,Environments,Add Environments}
![create-new-environment.png]

Next, create an account to access the Pi, this can either be a Username / Password or and SSH Key
![pi-account.png]

Then finally, create a deployment target under {Infrastructure,Deployment Targets,Add Deployment Target} as an SSH target.
After filling in the details (IP Address or DNS name, SSH port and account), under the .Net section, ensure that you select _Mono not installed_, don't worry about the platform, we will be changing that later.
![dotnet-not-mono.png]

Custom Calamari
Currently, Calamari does not support running on ARM architecture out of the box. You can easily fix this yourself with a few steps.
- Fork the [Calamari](https://github.com/OctopusDeploy/Calamari) repo.
- Pull down your forked version of Calamari
    - git clone http://github.com/_username_/Calamari
- Edit ./source/Calamari.csproj file, replacing the `<RuntimeIdentifiers>` line with `<RuntimeIdentifiers>linux-arm</RuntimeIdentifiers>`
- Run build
- Follow the instructions in the Calamari [README.md](https://github.com/OctopusDeploy/Calamari/blob/master/README.md) to configure Octopus Deploy to use a custom build of Calamari.

Hack your target config to specify the Calamari version as `linux-arm`

```
c# code using Octopus.Clients to load target and modify the version string
```

Create deployment process

- Deploy web application
    - deploy package step using package produced earlier
- install and start service 
    - deploy new package with service definition file
    - bash script with package


To get the application to run as a service, Microsoft have a documentation page for [hosting .Net Core on Linux](https://docs.microsoft.com/en-au/aspnet/core/host-and-deploy/linux-nginx?tabs=aspnetcore2x)

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

```info
The `[deploy web site]` strings in the `core4pi.service` text above, represents the name of the step that deploys the package.
This output variable will contain the path to the newly installed service. This will ensure when the service is installed it is looking at the latest version.
```

Create a package for the service definition and push it to the Octopus Server:
```text
```
octo.exe pack --id core4pi.service --version 1.0.0 --format nupkg
octo.exe push --server http://localhost:8085 --apikey API-6FGRLBN3XYXMMXWM70B9D9BLI6Y --package core4pi.service.1.0.0.nupkg
```

Edit your deployment process to add a second `Deploy Package` step.

![service-installation-step.png]

You will need to `Configure Features` for this step:
![feature-configuration.png]

Then in the `Substitute Variables in Files` feature add the name of the service definition file `core4pi.service`:
![substitute-variables-in-service.png]

Under the `Configuration Scripts` feature paste the below script in to the `Deployment Script` section:
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
This `bash` script will actually perform the service installation.


