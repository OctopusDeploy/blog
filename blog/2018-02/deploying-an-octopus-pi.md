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




Create a basic .Net Core Application

```
dotnet new angular
npm install
...coffee...
dotnet build
mkdir publish
dotnet publish -o publish
```


Package it up
```
octo.exe pack --id core4pi --version 1.0.0 --format nupkg
```

Send it on its way
```
octo.exe push --server http://localhost:8085 --apikey API-6FGRLBN3XYXMMXWM70B9D9BLI6Y --package core4pi.1.0.0.nupkg
```

Create Infrastructure
    - SSH key or username/password
    - Environment target machine


Build Calamari

Setup custom calamari version

Create deployment process




