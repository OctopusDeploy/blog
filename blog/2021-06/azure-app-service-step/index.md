---
title: Improved Azure App Service deployments 
description: Learn about the improved Azure App Service deployment step included in Octopus 2021.1.
author: michael.richardson@octopus.com
visibility: public
published: 2021-06-07-1400
metaImage: img-blog-azure-app-service-zip-deploy-2021.png
bannerImage: img-blog-azure-app-service-zip-deploy-2021.png
bannerImageAlt: Octopus, a zip file and Azure logo connected in the clouds
tags:
 - Product 
 - Azure
---

![Octopus, a zip file and Azure logo connected in the clouds](img-blog-azure-app-service-zip-deploy-2021.png)

The [Octopus Deploy Q2 release](https://octopus.com/blog/octopus-release-2021-q2) includes a new _Deploy an Azure App Service_ step. This step delivers major improvements for deploying Azure web applications, including:

- Deploying to Linux app service plans (without obscure configuration hacks). 
- Deploying container images. 
- Executing deployments on Linux Octopus workers.
- Configuring application settings and connection strings.

![Add Azure App Service step tile](azure-app-service-step-tile.png "width=500")

## What's changed?

The _Deploy an Azure App Service_ step is designed to supersede the existing _Deploy an Azure Web App_ step, however, the original step is still available. 

The _Deploy an Azure Web App_ step relied on [Microsoft Web Deploy](https://www.iis.net/downloads/microsoft/web-deploy) as the deployment mechanism. This restricted the step to executing on Windows workers, required special configuration to work with Linux app service plans, and didn't support container images.  The new step relies on the [zip deploy API](https://docs.microsoft.com/en-us/azure/app-service/deploy-zip#rest) for file-based packages (zip, nupkg, war), and also supports deploying container images.

## App settings and connection strings 

The new step also brings the ability to configure the Azure application settings and connection strings.

![Configuring app settings and connection strings](app-settings-and-connection-strings.png "width=500")

This is done by supplying the settings and/or connection strings as JSON. 

The benefits of using the JSON format are: 

- It is the same JSON format as the [bulk-edit feature in the Azure portal](https://docs.microsoft.com/en-us/azure/app-service/configure-common#edit-in-bulk), making it easy to move existing app settings into Octopus.  
- These fields accept [Octopus variable expressions](https://octopus.com/docs/projects/variables/variable-substitutions), allowing advanced scenarios such as conditionally including values, and iterating over collections of Octopus variables to build the app settings or connection strings.  

In the future we hope to add a friendlier UI over these fields, and maintain the JSON configuration as an 'advanced' mode. 

## Transforming configuration files

When deploying applications packaged as zip, nupkg, or war files, multiple options are available for transforming configuration files within the package.  Octopus will extract the package and perform these transforms before pushing to Azure.

![Azure app service configuration file options](azure-app-service-config-files.png "width=500")

The following options for transforming configuration files are provided: 

- Performing [variable substitution](https://octopus.com/docs/projects/steps/configuration-features/substitute-variables-in-templates#:~:text=The%20Substitute%20Variables%20in%20Files,Octopus%20Variables%20into%20any%20file.) on templated files, containing Octopus variable expressions (e.g. `#{MyVariable}`). 
- Using [structured configuration variables](https://octopus.com/docs/projects/steps/configuration-features/structured-configuration-variables-feature) to transform YAML, JSON, XML, and other structured configuration files.  
- Performing [.NET XML configuration transforms](https://octopus.com/docs/projects/steps/configuration-features/configuration-transforms).

## Conclusion

We tip our hats to our friends at [Clear Measure](https://clearmeasure.com/) for their help in developing this new Azure App Service integration.     

The _Deploy an Azure App Service_ step is available now in both Octopus Cloud instances and in Octopus 2021.1 on the self-hosted [downloads page](https://octopus.com/downloads). We hope this makes it easier than ever to deploy your Azure web applications.

Happy deployments!