---
title: Using the ‘Deploy a Bicep Template’ Step
description: Configuring the ‘Deploy a Bicep Template’ Step
author: isaac.calligeros@octopus.com
visibility: public
published: 2023-05-29-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: Bicep logo, a mechanical arm with bolts for shoulders and elbow.
isFeatured: false
tags:
  - Product
  - Azure
---


Bicep is a human-readable language for deploying resources using ARM templates. Our latest step makes deploying Azure Resources using Bicep more intuitive and user-friendly. This blog walks through configuring the "Deploy a Bicep Template" step.

## Configuring the ‘Deploy a Bicep Template’ Step

To start, add the Deploy a Bicep Template step to your deployment process. This step depends on the [bicep module](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/install) of the Azure CLI in the execution environment. Assuming the Azure CLI is already installed and on path, the bicep module can is installed using:
```
az bicep install
```

![Deploy a Bicep Template process editor](bicep-process-editor.png "width=500")

## Configuration

The templates can be stored using the source code editor or packages when configuring the Deploy a Bicep Template Step. The first and easier option is the Source Code editor, here you can directly edit your bicep template. Alternatively, you can store your bicep files in a package and provide a path to the template.  When packages are used, many bicep templates can be provided, and modules in [local files](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules#local-file) can be referenced using relative paths.  The relative path to the Bicep template file contained in the package will need to be provided when using packages.

![Code Editor or Package Selector](codeeditor-or-package-selector.png "width=500")


To provide parameters to these bicep templates, a list of key-value pairs is provided.

For example given a bicep template with the following param definitions
```
param appicationName string
param location string
```
The following parameters definitions
![Bicep Parameters](bicep-parameters.png "width=500")

Regardless of your template source, you’ll need to decide on a [deployment mode](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-modes), we default to incremental mode. In incremental mode, resources that exist in the resource group but are not specified in the template are left unchanged by Resource Manager. Resources in the template are added to the resource group. 
In complete mode, Resource Manager deletes resources that exist in the resource group but aren't specified in the template.


## Account

An Azure account along with a target resource group are both required. For help configuring an Azure Account see [connect an Azure Account to Octopus Deploy](https://octopus.com/docs/infrastructure/accounts/azure#azure-service-principal). When specifying a resource group if it does not exist, it will be created as part of the deployment process. If the resource group does not exist and is being created as a part of the deployment, the resource group location must be specified.

![Bicep Account Configuration](bicep-account.png "width=500")


Happy deployments!

