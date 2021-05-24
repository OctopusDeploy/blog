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

![Add Azure App Service step tile](azure-app-service-step-tile.png "width=500")

The _Deploy an Azure App Service_ step is designed to supercede the _Deploy an Azure Web App_ step, though the original step is still available. The _Deploy an Azure Web App_ step relied on [Microsoft Web Deploy](https://www.iis.net/downloads/microsoft/web-deploy) as the deployment mechanism. This restricted the step to executing on Windows workers, required specifial configuration to work with Linux app service plans, and didn't support container images.  The new step relies on the [zip deploy API](https://docs.microsoft.com/en-us/azure/app-service/deploy-zip#rest) for file-based packages (zip, nupkg, war) and also supports container images. 

The new step also brings the ability to configure the Azure application settings and connection strings. 

![Configuring app settings and connection strings](app-settings-and-connection-strings.png "width=500")

This is done by supplying the settings and/or connection strings as JSON. Using the JSON format brings a couple of benefits: 
It is the same JSON format as used by the [bulk-edit feature in the Azure portal](https://docs.microsoft.com/en-us/azure/app-service/configure-common#edit-in-bulk), making it easy to move existing app settings into Octopus.  These fields also allow including [Octopus variable expressions](https://octopus.com/docs/projects/variables/variable-substitutions), allowing advanced scenarios such as conditionally including values, and iterating over collections of Octopus variables to build the app settings or connection strings.  

In the future we'll likely also add a friendlier UI over these fields, maintaining the JSON configuring as an 'advanced' mode. 

When deploying applications packaged as zip, nupkg, or war files, then multiple options are available for transforming configuration files.  Octopus will extract the package and perform these transforms before pushing to Azure.  