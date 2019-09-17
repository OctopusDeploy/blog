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
When you become experienced at using PowerShell Desired State Configuration (DSC), you may enounter situations where existing modules don't solve your specific need.  In this post, I'll show you how to create your own DSC modules to solve those situations.

## Scenario
While implenting DSC at my previous job, there was a need to give the identity assigned to an IIS App Pool permissions to read an installed cerificate.  Certificates installed to certificate stores are just files in a specific location, once you find the correct file, simply access the ACL and you can assign or remove permissions.  Using PowerShell, I was able to connect to the certificate store and find the certificate I needed.  According to my research, the filename was located in the `$certificate.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName` property.  The issue was `PrivateKey` was null despite the `$certificate.HasPrivateKey` being `true`.  After scouring the Internet for solutions, I came across [this post](https://www.sysadmins.lv/blog-en/retrieve-cng-key-container-name-and-unique-name.aspx) which provided the solution I was looking for.  Rather than try to write a Script Resource, I decided to write my own module.

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
$Thumbprint = New-xDscResourceProperty -Name Thumbprint -Type String -Attribute Key
$Location = New-xDscResourceProperty -Name Location -Type String -Attribute Key -ValidateSet "LocalMachine", "CurrentUser"
$Store = New-xDscResourceProperty -Name Store -Type String -Attribute Key
$Ensure = New-xDscResourceProperty -Name Ensure -Type String -Attribute Required -ValidateSet "Present", "Absent"
$UserAccount = New-xDscResourceProperty -Name UserAccount -Type String -Attribute Write
$Permission = New-xDscResourceProperty -Name Permission -Type String -ValidateSet "Read", "FullControl" -Attribute Write

# Create my DSC Resource
New-xDscResource -Name xCertificatePermission -Property $Thumbprint, $Location, $Store, $Ensure, $UserAccount, $Permission -Path 'c:\Program Files\WindowsPowerShell\Modules' -ModuleName xCertificatePermission
```
And there you have it, your very own DSC module with all the stubs generated!

#### Understanding resource Attribute property
For the Attribute component of a Resource Property within DSC, there are four possible values

- Key
- Read
- Required
- Write

##### Key
Every node that uses your resource must have a key that makes the node unique.  Similar to database tables, this key need not be a single property, but can be made up of several properties, each bearing the Key attribute.  

##### Read
Read attributes are read-only and cannot have values assigned to them.

##### Required
Required attributes are assignable properties that must be specified when declaring the configuration.  

##### Write
Write attributes are optional attributes that you specify a value to when defining the node.  

#### ValidateSet switch
The ValidateSet switch is someting that can be used with Key or Write attributes that specify the allowable values for a given property.  In our example, we've specified that Ensure can only be either "Absent" or "Present".  Any other value will result in an error.

## DSC module file and folder structure
Whether you decided to [do it yourself](https://docs.microsoft.com/en-us/powershell/dsc/resources/authoringResourceMOF) or use the tool, the folder and file structure will look like the following

```
$env:ProgramFiles\WindowsPowerShell\Modules (folder)
    |- xCertificatePermission (folder)
        |- DSCResources (folder)
            |- xCertificatePermission (folder)
                |- xCertificatePermission.psd1 (file, optional)
                |- xCertificatePermission.psm1 (file, required)
                |- xCertificatePermission.schema.mof (file, required)
```

### The MOF file
MOF stands for Managed Object Format and is the language used to describe Common Information Model (CIM) classes.  Using the example from the 'Tool to help you write your module' section, the resulting MOF file would be

```
[ClassVersion("1.0.0.0"), FriendlyName("xCertificatePermission")]
class xCertificatePermission : OMI_BaseResource
{
    [Key] String Thumbprint;
    [Key, ValueMap{"LocalMachine","CurrentUser"}, Values{"LocalMachine","CurrentUser"}] String Location;
    [Key] String Store;
    [Required, ValueMap{"Present","Absent"}, Values{"Present","Absent"}] String Ensure;
    [Write] String UserAccount;
    [Write, ValueMap{"Read","FullControl"}, Values{"Read","FullControl"}] String Permission;
};
```

The MOF file will only contain the properties that we are going to be using in our module, along with their attributes and data types.  Unless we're adding or removing properties, this is pretty much all we do with the MOF file.

### The psm1 file
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
        $Thumbprint,

        [parameter(Mandatory = $true)]
        [ValidateSet("LocalMachine","CurrentUser")]
        [System.String]
        $Location,

        [parameter(Mandatory = $true)]
        [System.String]
        $Store,

        [parameter(Mandatory = $true)]
        [ValidateSet("Present","Absent")]
        [System.String]
        $Ensure
    )

    #Write-Verbose "Use this cmdlet to deliver information about command processing."

    #Write-Debug "Use this cmdlet to write debug information while troubleshooting."


    <#
    $returnValue = @{
    Thumbprint = [System.String]
    Location = [System.String]
    Store = [System.String]
    Ensure = [System.String]
    UserAccount = [System.String]
    Permission = [System.String]
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
        $Thumbprint,

        [parameter(Mandatory = $true)]
        [ValidateSet("LocalMachine","CurrentUser")]
        [System.String]
        $Location,

        [parameter(Mandatory = $true)]
        [System.String]
        $Store,

        [parameter(Mandatory = $true)]
        [ValidateSet("Present","Absent")]
        [System.String]
        $Ensure,

        [System.String]
        $UserAccount,

        [ValidateSet("Read","FullControl")]
        [System.String]
        $Permission
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
        $Thumbprint,

        [parameter(Mandatory = $true)]
        [ValidateSet("LocalMachine","CurrentUser")]
        [System.String]
        $Location,

        [parameter(Mandatory = $true)]
        [System.String]
        $Store,

        [parameter(Mandatory = $true)]
        [ValidateSet("Present","Absent")]
        [System.String]
        $Ensure,

        [System.String]
        $UserAccount,

        [ValidateSet("Read","FullControl")]
        [System.String]
        $Permission
    )

    #Write-Verbose "Use this cmdlet to deliver information about command processing."

    #Write-Debug "Use this cmdlet to write debug information while troubleshooting."

    #Include this line if the resource requires a system reboot.
    #$global:DSCMachineStatus = 1


}
```

## Summary
Whether simplistic or complex, the steps for creating your own DSC module would be the same.  This post is aimed at getting you started in the right direction.  From here, you can create your module to fit whatever resource you need to configure and keep in desired state.  For the full version of xCertificatePermission, check out my [GitHub repo](https://github.com/twerthi/xCertificatePermission).