---
title: Write your own PowerShell Desired State Configuration (DSC) module
description: How to write your own PowerShell Desired State Configuration (DSC) module
author: shawn.sesna@octopus.com
visibility: public
bannerImage: write-powershell-dsc-module.png
bannerImageAlt: Octopus learning how to write a custom PowerShell DSC Module
metaImage: write-powershell-dsc-module.png
published: 2019-10-23
tags:
 - DevOps
 - PowerShell
---

![Octopus learning how to write a custom PowerShell DSC Module](write-powershell-dsc-module.png)

PowerShell DSC is a fantastic technology to have in your tool-belt for managing Windows-based servers. This post is part of a series:

- [Getting started with PowerShell Desired State Configuration (DSC)](https://octopus.com/blog/getting-started-with-powershell-dsc)
- **Write your own PowerShell Desired State Configuration (DSC) module**

We also have articles on using PowerShell DSC with Octopus Deploy:
- [Configuration Management with Octopus and PowerShell DSC](https://octopus.com/blog/octopus-and-powershell-dsc)
- [Deploying PowerShell DSC like an App with Octopus Deploy](https://octopus.com/blog/powershelldsc-as-template)

---

As you gain experience with PowerShell Desired State Configuration (DSC) you might encounter situations where the available modules don’t quite fit what you want to do.  You could write your own [Script Resources](https://docs.microsoft.com/en-us/powershell/dsc/reference/resources/windows/scriptresource), but they don’t scale well, passing parameters is difficult, and they don’t provide a method for encryption, leaving passwords in clear text, however, you can write your own DSC modules.

In this PowerShell DSC tutorial, I'll cover how to write your first PowerShell DSC module.

## A tool to help you write your PowerShell DSC module

Writing your own PowerShell DSC module isn’t really that hard.  The most difficult part is getting the files and folders in the correct locations because DSC is quite specific about what goes where. However, Microsoft recognizes this can be quite frustrating and has developed [xDscResourceDesigner](https://docs.microsoft.com/en-us/powershell/dsc/resources/authoringresourcemofdesigner), a PowerShell module to help you get started.  Using this module, you can easily define what properties your resource needs and it will generate the entire module structure for you, including the MOF schema file.  If you’re a first-timer, I highly recommend using this module, it could save you quite a bit of frustration (take it from me).

## Installing xDscResourceDesigner

Installing the module is no different from installing any other module onto your system:

```PS
Install-Module -Name xDscResourceDesigner
```

As this [Microsoft article](https://docs.microsoft.com/en-us/powershell/dsc/resources/authoringresourcemofdesigner) points out, if you have a version of PowerShell prior to version 5, you may need to install the PowerShellGet module for installation to work.

## Using xDscResourceDesigner

Using the xDscResourceDesigner is actually pretty easy, there are only two functions: `New-DscResourceProperty` and `New-xDscResource`.  `New-DscResourceProperty` is what you use to define the properties of your DSC resource.  After you’ve done that, you send that information to the `New-xDscResource` function, and it generates everything you need to implement your resource:

```PS
# Import the module for use
Import-Module -Name xDscResourceDesigner

# Define properties
$property1 = New-xDscResourceProperty -Name Property1 -Type String -Attribute Key
$property2 = New-xDscResourceProperty -Name Property2 -Type PSCredential -Attribute Write
$property3 = New-xDscResourceProperty -Name Property3 -Type String -Attribute Required -ValidateSet "Present", "Absent"

# Create my DSC Resource
New-xDscResource -Name DemoResource1 -Property $property1, $property2, $property3 -Path 'c:\Program Files\WindowsPowerShell\Modules' -ModuleName DemoModule
```

And there you have it, your very own DSC module with all the stubs generated.

## Understanding the resource Attribute property

For the Attribute component of a Resource Property within DSC, there are four possible values:

- Key
- Read
- Required
- Write

### Key

Every node that uses your resource must have a key that makes the node unique.  Similar to database tables, this key doesn’t need to be a single property, but it can be made up of several properties, each bearing the Key attribute.  In the example above, `Property1` is our Key for the resource.  However, it could also be done this way:

```PS
# Define properties
$property1 = New-xDscResourceProperty -Name Property1 -Type String -Attribute Key
$property2 = New-xDscResourceProperty -Name Property2 -Type PSCredential -Attribute Write
$property3 = New-xDscResourceProperty -Name Property3 -Type String -Attribute Required -ValidateSet "Present", "Absent"
$property4 = New-xDscResourceProperty -Name Property4 -Type String -Attribute Key
$property5 = New-xDscResourceProperty -Name Property5 -Type String -Attribute Key
```

In this example, `property1`, `property4`, and `property5` are what make up the unique value for the node.  Key attributes are always writable and are required.

### Read

Read attributes are read-only and cannot have values assigned to them.

### Required

Required attributes are assignable properties that must be specified when declaring the configuration.  Using our example from above when we created our resource, the `Property3` property is set to be required.  

### Write

Write attributes are optional attributes that you specify a value to when defining the node.  In the example, `Property2` is defined as a Write attribute.

### ValidateSet switch

The `ValidateSet` switch is something that can be used with Key or Write attributes that specify the allowable values for a given property.  In our example, we’ve specified that `Property3` can only be either `Absent` or `Present`.  Any other value will result in an error.

## DSC module file and folder structure

Whether you decided to [do it yourself](https://docs.microsoft.com/en-us/powershell/dsc/resources/authoringResourceMOF) or use the tool, the folder and file structure will look like the following:

```
$env:ProgramFiles\WindowsPowerShell\Modules (folder)
    |- DemoModule (folder)
        |- DSCResources (folder)
            |- DemoResource1 (folder)
                |- DemoResource1.psd1 (file, optional)
                |- DemoResource1.psm1 (file, required)
                |- DemoResource1.schema.mof (file, required)
```

## The MOF file

MOF stands for Managed Object Format and is the language used to describe Common Information Model (CIM) classes.  Using the example from the *Tool to help you write your module* section, the resulting MOF file will look like this:

```
[ClassVersion("1.0.0.0"), FriendlyName("DemoResource1")]
class DemoResource1 : OMI_BaseResource
{
    [Key] String Property1;
    [Write, EmbeddedInstance("MSFT_Credential")] String Property2;
    [Required, ValueMap{"Present","Absent"}, Values{"Present","Absent"}] String Property3;
};
```

The MOF file will only contain the properties we will in our module, along with their attributes and data types.  Unless we’re adding or removing properties, this is pretty much all we do with the MOF file.

## the psm1 file

The psm1 file is where the bulk of our code is going to be.  This file will contain three required functions:

- `Get-TargetResource`
- `Test-TargetResource`
- `Set-TargetResource`

### Get-TargetResource

The `Get-TargetResource` function returns the current value(s) of what the resource is responsible for.  Our stubbed function from using `xDscResourceDesigner` looks like the following:

```PS
function Get-TargetResource
{
    [CmdletBinding()]
    [OutputType([System.Collections.Hashtable])]
    param
    (
        [parameter(Mandatory = $true)]
        [System.String]
        $Property1,

        [parameter(Mandatory = $true)]
        [ValidateSet("Present","Absent")]
        [System.String]
        $Property3
    )

    #Write-Verbose "Use this cmdlet to deliver information about command processing."

    #Write-Debug "Use this cmdlet to write debug information while troubleshooting."


    <#
    $returnValue = @{
    Property1 = [System.String]
    Property2 = [System.Management.Automation.PSCredential]
    Property3 = [System.String]
    }

    $returnValue
    #>
}
```
Note that the optional parameter (Write attribute) `Property2` is not required for this function.

### Test-TargetResource

The `Test-TargetResource` function returns a boolean value indicating whether or not the resource is in the desired state.  From our generated example, the function looks like this:

```PS
function Test-TargetResource
{
    [CmdletBinding()]
    [OutputType([System.Boolean])]
    param
    (
        [parameter(Mandatory = $true)]
        [System.String]
        $Property1,

        [System.Management.Automation.PSCredential]
        $Property2,

        [parameter(Mandatory = $true)]
        [ValidateSet("Present","Absent")]
        [System.String]
        $Property3
    )

    #Write-Verbose "Use this cmdlet to deliver information about command processing."

    #Write-Debug "Use this cmdlet to write debug information while troubleshooting."


    <#
    $result = [System.Boolean]

    $result
    #>
}
```

### Set-TargetResource

The `Set-TargetResource` function is used to configure the resource to the specified desired state.  Our generated example looks like this:

```PS
function Set-TargetResource
{
    [CmdletBinding()]
    param
    (
        [parameter(Mandatory = $true)]
        [System.String]
        $Property1,

        [System.Management.Automation.PSCredential]
        $Property2,

        [parameter(Mandatory = $true)]
        [ValidateSet("Present","Absent")]
        [System.String]
        $Property3
    )

    #Write-Verbose "Use this cmdlet to deliver information about command processing."

    #Write-Debug "Use this cmdlet to write debug information while troubleshooting."

    #Include this line if the resource requires a system reboot.
    #$global:DSCMachineStatus = 1
}
```

## Summary

Whether simplistic or complex, the steps for creating your own PowerShell DSC module will be the same.  This post is aimed at getting you started in the right direction.  From here, you can create your module to fit whatever resource you need to configure and keep in a desired state.  For the full example of a working module check out [xCertificatePermission](https://github.com/twerthi/xCertificatePermission) on my GitHub repo.
