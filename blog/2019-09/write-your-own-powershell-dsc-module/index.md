---
title: Writing your own Desired State Configuration (DSC) module
description: How to write your own PowerShell Desired State Configuration (DSC) module
author: shawn.sesna@octopus.com
visibility: public
bannerImage: 
metaImage: 
published: 
tags:
 - configuration
 - powershell
---

## Introduction
So, you've discovered PowerShell Desired State Configuration (DSC), got some experience under your belt, but you have encountered a situation where the available modules don't quite fit what you want to do.  Doing some research, you've found that you can write [Script Resources](https://docs.microsoft.com/en-us/powershell/dsc/reference/resources/windows/scriptresource) to do what you want.  You'll eventually come to the realization that they don't scale well, are difficult to pass parameters to, and do not to provide a method for encryption leaving passwords in clear text.  You say to yourself, "I know!  I'll write my own module!  Can't be that hard, can it?"

## Tool to help you write your module
Writing your own module isn't really that hard.  The most difficult part is getting the files and folders in the correct locations, DSC is quite specific in what goes where.  Microsoft recognizes that this can be quite frustrating and has developed a PowerShell module to help you get started, [xDscResourceDesigner](https://docs.microsoft.com/en-us/powershell/dsc/resources/authoringresourcemofdesigner).  Using this module, you can easily define what properties your resource needs to have and it will generate the entire module structure for you, including the MOF schema file.  If you're a first timer, I highly suggest using this module, it could save you quite a bit of frustration (take it from this author).

### Installing xDscResourceDesigner
Installing the module is no different than installing any other module onto your system

```PS
Install-Module -Name xDscResourceDesigner
```

As the [Microsoft article](https://docs.microsoft.com/en-us/powershell/dsc/resources/authoringresourcemofdesigner) points out, if you have a version of PowerShell prior to version 5, you may need to install the PowerShellGet module for installation to work.

### Using xDscResourceDesigner
Using the xDscResourceDesigner is actually pretty easy, there are only two functions; New-DscResourceProperty and New-xDscResource.  New-DscResourceProperty is what you use to define the properties of your DSC resource.  Once you've done that, you pump that information to New-xDscResource function and it generates everything you need to implement your resource!

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
And there you have it, your very own DSC module with all the stubs generated!

#### Understanding resource Attribute property
For the Attribute component of a Resource Property within DSC, there are four possible values

- Key
- Read
- Required
- Write

##### Key
Every node that uses your resource must have a key that makes the node unique.  Similar to database tables, this key need not be a single property, but can be made up of several properties, each bearing the Key attribute.  In our example above, Property1 is our Key for the resource.  However, it could also be done this way

```PS
# Define properties
$property1 = New-xDscResourceProperty -Name Property1 -Type String -Attribute Key
$property2 = New-xDscResourceProperty -Name Property2 -Type PSCredential -Attribute Write
$property3 = New-xDscResourceProperty -Name Property3 -Type String -Attribute Required -ValidateSet "Present", "Absent"
$property4 = New-xDscResourceProperty -Name Property4 -Type String -Attribute Key
$property5 = New-xDscResourceProperty -Name Property5 -Type String -Attribute Key
```
In this example, property1, property4, and property5 are what make up the unique value for the node.  Key attributes are always writable and are required.

##### Read
Read attributes are read-only and cannot have values assigned to them.

##### Required
Required attributes are assignable properties that must be specified when declaring the configuration.  Using our example from above when we created our resource, the Property3 property is set to be required.  

##### Write
Write attributes are optional attributes that you specify a value to when defining the node.  In our example, Property2 is defined as a Write attribute.

#### ValidateSet switch
The ValidateSet switch is someting that can be used with Key or Write attributes that specify the allowable values for a given property.  In our example, we've specified that Property3 can only be either "Absent" or "Present".  Any other value will result in an error.

## DSC module file and folder structure
Whether you decided to [do it yourself](https://docs.microsoft.com/en-us/powershell/dsc/resources/authoringResourceMOF) or use the tool, the folder and file structure will look like the following

```
$env:ProgramFiles\WindowsPowerShell\Modules (folder)
    |- DemoModule (folder)
        |- DSCResources (folder)
            |- DemoResource1 (folder)
                |- DemoResource1.psd1 (file, optional)
                |- DemoResource1.psm1 (file, required)
                |- DemoResource1.schema.mof (file, required)
```

### The MOF file
MOF stands for Managed Object Format and is the language used to describe Common Information Model (CIM) classes.  Using the example from the 'Tool to help you write your module' section, the resulting MOF file would be

```
[ClassVersion("1.0.0.0"), FriendlyName("DemoResource1")]
class DemoResource1 : OMI_BaseResource
{
    [Key] String Property1;
    [Write, EmbeddedInstance("MSFT_Credential")] String Property2;
    [Required, ValueMap{"Present","Absent"}, Values{"Present","Absent"}] String Property3;
};
```

The MOF file will only contain the properties that we are going to be using in our module, along with their attributes and data types.  Unless we're adding or removing properties, this is pretty much all we do with the MOF file.

### the psm1 file
The psm1 file is where the bulk of our code is going to be.  This file will contain three required functions

- Get-TargetResource
- Test-TargetResource
- Set-TargetResource

#### Get-TargetResource
Get-TargetResource returns the current value(s) of what the resource is responsible for.  Our stubbed function from using xDscResourceDesigner would look like the following

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
Note that the optional parameter (Write attribute) Property2 is not required for this function.

#### Test-TargetResource
The Test-TargetResource function returns a boolean value of whether or not the resource is in desired state.  From our generated example, the function would look like this,

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

#### Set-TargetResource
The Set-TargetResource function is used to configure the resource to the specified desired state.  Our generated example would look like this,

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
Whether simplistic or complex, the steps for creating your own DSC module would be the same.  This post is aimed at getting you started in the right direction.  From here, you can create your module to fit whatever resource you need to configure and keep in desired state.  For the full example of a working module check out [xCertificatePermission](https://github.com/twerthi/xCertificatePermission) on my GitHub repo.