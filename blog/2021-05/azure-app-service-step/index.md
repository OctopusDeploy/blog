---
title: A new Azure App Service step 
description: Octopus 2021.1 included an improved Azure App Service deployment step 
author: michael.richardson@octopus.com
visibility: private
published: 2021-05-30-1400
metaImage: 
bannerImage: 
tags:
 - Product 
---

Octopus 2021.1 includes a new _Deploy an Azure App Service_ step, bringing some major improvements for deploying Azure web applications, including:

- Deploying to Linux app service plans (without any configuration hacks) 
- Deploying container images 
- Executing on Linux Octopus workers
- Configuring application settings and connection strings

TODO: step tile picture

The _Deploy an Azure App Service_ step is designed to supercede the _Deploy an Azure Web App_ step, though the original step is still available. The _Deploy an Azure Web App_ step relied on Web Deploy as the deployment mechanism; this restricted the step to executing on Windows workers, required specifial configuration to work with Linux app service plans, and didn't support container images.  The new step relies on the zip deploy API for file-based packages (zip, nupkg, war) and also supports container images. 

