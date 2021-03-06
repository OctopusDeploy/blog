---
title: Deploying PowerShell Desired State Configuration (DSC) like an App with Octopus Deploy
description: Use Octopus Deploy custom step templates and configuration data files (DSC configuration) to deploy your Infrastructure as Code using PowerShell Desired State Configuration (DSC).
author: shawn.sesna@octopus.com
visibility: public
published: 2019-06-19
bannerImage: img_powershell_dsc.png
metaImage: img_powershell_dsc.png
tags:
 - Engineering
 - PowerShell
---

PowerShell DSC is a fantastic technology to have in your tool-belt for managing Windows-based servers. This post is part of a series:

- [Getting started with PowerShell Desired State Configuration (DSC)](https://octopus.com/blog/getting-started-with-powershell-dsc)
- [Write your own PowerShell Desired State Configuration (DSC) module](https://octopus.com/blog/write-your-own-powershell-dsc-module)

We also have articles on using PowerShell DSC with Octopus Deploy:
- [Configuration Management with Octopus and PowerShell DSC](https://octopus.com/blog/octopus-and-powershell-dsc)
- **Deploying PowerShell DSC like an App with Octopus Deploy**

---

In a previous [post](https://octopus.com/blog/octopus-and-powershell-dsc), Paul Stovell showed us how to use Octopus Deploy to deploy PowerShell Desired State Configuration (DSC) to servers to:

 - Manage infrastructure.
 - Use Machine Policies to monitor for drift.
 - Automatically correct drift with a Project Trigger.  

This blog post will expand on that idea by:

 - Separating the configuration data into a configuration data file (.psd1).
 - Turning a PowerShell DSC script into a step template.
 - Capturing the items that have drifted in a Machine Policy.
 - Taking advantage of the Substitute Variables in Files feature to set properties that need to change for an Environment or Project.  

By the end of this post, we'll have created a re-usable PowerShell DSC script as a step template that can be used in a deployment.  We'll also have created a Machine Policy that will report any items that have drifted and mark the machine as unhealthy.

!toc

## PowerShell DSC

### Paul's Original Script
The code sample that Paul provided for DSC PowerShell was a good example of a basic script:

```PS
Configuration WebServerConfiguration
{  
  Node "localhost"
  {        
    WindowsFeature InstallWebServer
    {
      Name = "Web-Server"
      Ensure = "Present"
    }

    WindowsFeature InstallAspNet45
    {
      Name = "Web-Asp-Net45"
      Ensure = "Present"
    }
  }
}

WebServerConfiguration -OutputPath "C:\DscConfiguration"

Start-DscConfiguration -Wait -Verbose -Path "C:\DscConfiguration"
```

### Separating Node Data

In his example, the Node data is contained within the script itself and the features to be configured are static.  If we separate the Node data from the script into a Configuration Data File (DSC configuration), we can make our DSC PowerShell script more dynamic:

```PS
@{
    AllNodes =  @(
        @{

            # node name
            NodeName = $env:COMPUTERNAME

            # required windows features
            WindowsFeatures = @(
				@{
					Name = "Web-Server"
					Ensure = "Present"
					Source = "d:\sources\sxs"
				},
				@{
					Name = "Web-Asp-Net45"
					Ensure = "Present"
					Source = "d:\sources\sxs"
				}
			)
		}
    )
}
```

### Making the DSC PowerShell Script More Dynamic

With the Node data now separated, the DSC PowerShell script can be changed to be dynamic, installing features that have been specified in the configuration data file (DSC configuration):

```PS
Configuration WebServerConfiguration
{
    Import-DscResource -ModuleName 'PSDesiredStateConfiguration' # We get a warning if this isn't included

    Node $AllNodes.NodeName
    {
        # loop through features list and install
        ForEach($Feature in $Node.WindowsFeatures)
        {
            WindowsFeature "$($Feature.Name)"
            {
                Ensure = $Feature.Ensure
                Name = $Feature.Name
                Source = $Feature.Source # Needed if for some reason the resource isn't on the OS already and needs to be retrieved from something like a mounted ISO
            }
        }
    }
}

WebServerConfiguration -ConfigurationData "C:\DscConfiguration\WebServer.psd1" -OutputPath "C:\DscConfiguration"

Start-DscConfiguration -Wait -Verbose -Path "C:\DscConfiguration"
```

### Filling in More Details of the DSC configuration

Great!  We've got a good start for our web server implementation, but there's more to configuring a web server than installing Windows Features.  Let's add some more data to our configuration data file by adding some additional Windows Features, Sites, Applications, setting a default log path for IIS Sites, and harden our security with Ciphers, Hashes, Protocols, Key Exchanges, and specifying our Cipher Suite order.  

NOTE: This is just for demonstration purposes, consult your security team to determine which settings are best for your organization.

The following is a snippet, the complete file can be found on our Examples GitHub repository at https://github.com/OctopusSamples/DSC-as-a-template/blob/master/src/CompleteScript/WebServer.psd1:

```PS
@{
    AllNodes =  @(
        @{

            # node name
            NodeName = $env:COMPUTERNAME

            # required windows features
            WindowsFeatures = @(
				...
				@{
					Name = "Web-Server"
					Ensure = "Present"
					Source = "d:\sources\sxs"
				},
				...
			)

            # default IIS Site log path
            LogPath = "c:\logs"

            # Define root path for IIS Sites
            RootPath = "C:\inetpub\wwwroot"

            # define IIS Sites
            Sites = @(
				...
                 @{
					Name = "OctopusDeploy.com"
					Ensure = "Present"
					State = "Started"
					BindingInformation = @(
						@{
							Port = "80"
							IPAddress = "" # leave blank or comment out to set to All Unassigned
							Protocol = "HTTP"
						},
						@{
							Port = "443"
							IPAddress = ""
							Protocol = "HTTPS"
							CertificateStoreName  = "WebHosting" # WebHosting | My
							Exportable = $true
						}

					)
					Pool = @{
							PipeLine = "Integrated"
							RuntimeVersion = "v4.0"
							State = "Started"
						}
					Applications = @(
						...
						@{
							Name = "OctoFX"
							FolderName = "OctoFX"
							Ensure = "Present"
							Pool = @{
								Pipeline = "Integrated"
								RuntimeVersion = "v4.0"
								State = "Started"
							}
							Authentication = @{
								Windows = $false
								Anonymous = $true
							}

						},
						...
					)
				}
            )

            # fill in this section to enable or disable encryption protocols, hashes, ciphers, and specify cipher suite ordering
            Encryption = @{
				Ciphers = @(
					...
					@{
						Name = "DES 56/56"
						Enabled = "0" # Disabled = 0, Enabled = -1
					},
					...
				)
				Hashes = @(
					...
					@{
						Name = "MD5"
						Enabled = "0"
					},
					...
				)
				Protocols = @(
					...
					@{
						Name = "Multi-Protocol Unified Hello"
						Enabled = "0"
					},
					...
				)
				KeyExchanges = @(
					...
					@{
						Name = "Diffie-Hellman"
						Enabled = "-1"
					},
					...
				)
				CipherSuiteOrder = @("TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384", ...)
			}
		}
    )
}
```

### A More Complete DSC Script

As you might have guessed, we'll need to update our DSC script to configure the options we've specified in our configuration data file.

The following is a snippet, the complete file can be found on our Examples GitHub repository at https://github.com/OctopusSamples/DSC-as-a-template/blob/master/src/CompleteScript/IISDSC-Complete.ps1:

```PS
Configuration WebServerConfiguration
{
    Import-DscResource -ModuleName 'PSDesiredStateConfiguration'
    Import-DscResource -Module xWebAdministration

    Node $AllNodes.NodeName
    {
        # loop through features list and install
        ForEach($Feature in $Node.WindowsFeatures)
        {
            WindowsFeature "$($Feature.Name)"
            {
                ...
            }
        }

        # stop default web site
        xWebSite DefaultSite
        {
            ...
        }

        # loop through list of default app pools and stop them
        ForEach($Pool in @(".NET v2.0", `
        ".NET v2.0 Classic", `
        ".NET v4.5", `
        ".NET v4.5 Classic", `
        "Classic .NET AppPool", `
        "DefaultAppPool"))
        {
            xWebAppPool $Pool
            {
                ...
            }
        }

        # make sure log path exists
        File LoggingPath
        {
            ...
        }

        # loop through the sites
        ForEach($Site in $Node.Sites)
        {
            # create the folder
            File $Site.Name
            {
                ...
            }

            # create the site app pool
            xWebAppPool $Site.Name
            {
                ...
            }

            # create the site
            xWebSite $Site.Name
            {
                ...
            }

            # Loop through site application collection and create folders
            ForEach($Application in $Site.Applications)
            {
                File "$($Site.Name)-$($Application.Name)"
                {
                    ...
                }


                # create application pool
                xWebAppPool $Application.Name
                {
                    ...
                }

                # create application
                xWebApplication $Application.Name
                {
                    ...
                }
            }
        }
    }
}

# create credential object
$Credentials = @()

WebServerConfiguration -ConfigurationData "C:\DscConfiguration\WebServer.psd1" -OutputPath "C:\DscConfiguration"

Start-DscConfiguration -Wait -Verbose -Path "C:\DscConfiguration"
```

This isn't a post on how to do PowerShell DSC, so I won't go over everything that was added.  However, I did want to highlight one line, `Import-DscResource -Module xWebAdministration`.  This xWebAdministration isn't one that is installed by default, like PSDesiredStateConfiguration, and will need to be included when we do a deployment.  We'll discuss more on that later in this post.

## Making the DSC Script a Step Template

Okay!  We have our configuration data separated into its own file, and we've got a DSC script that will configure an IIS Web server.  Now let's start hooking it all together in Octopus Deploy and make our DSC script a Step Template!

We'll start by logging into our Octopus Deploy instance and clicking on the Library tab, then Step Templates:

![](step-templates.png)

Click on the Add button:

![](step-templates-add.png)

Choose the Deploy a Package template:

![](deploy-a-package-template.png)

### Settings

Fill in Settings:

![](settings-page.png)

### Parameters

We'll define three Parameters, DSC Path, Configuration Data File step, and Configuration Data file name.  These will be used by our DSC script.

#### DSC Path
This is the path where the .MOF file will be written to when DSC executes:

![](dsc-path.png)

#### Configuration Data File Name
This is the name of the configuration data file we created, WebServer.psd1:

![](data-file-name.png)

#### Package ID
This is the ID of the package from the Library that will be used for deployment:

![](package-id.png)

### Step Tab

#### Configure Features
On the Step tab, click the Configure Features button:

![](configure-features-template.png)

Enable Custom Deployment Scripts and Substitute Variables in Files:

![](template-enabled-features.png)

#### Setting Package Variable
On the Step tab, under Package Details, click the chain link icon to enable binding to a variable:

![](chain-link-icon.png)

Now, click on the #{} to bring up the list of variables, and choose the DSCPackageId Parameter we created:

![](package-id-variable.png)

#### Implementing the DSC Script

Expand Custom Deployment Scripts and paste our PowerShell DSC script into the Deployment script box:

![](step-code.png)

Enter Full Screen mode by clicking on the opposing arrows so we can more easily tweak our script to use the Parameters we've defined:

![](full-screen.png)

Scroll to the bottom and change:

```PS
WebServerConfiguration -ConfigurationData "C:\DscConfiguration\WebServer.psd1" -OutputPath "C:\DscConfiguration"

Start-DscConfiguration -Wait -Verbose -Path "C:\DscConfiguration"
```
To:

```PS
# set location for mof files
Set-Location -Path $DSCTempPath

# get the configuration data file
$ConfigurationDataFile = (Get-ChildItem -Path $OctopusParameters["Octopus.Action.Package.InstallationDirectoryPath"] | Where-Object {$_.Name -eq $DataFileName}).FullName

# Display which file it's using
Write-Host "The configuration data file is: $ConfigurationDataFile"

# Execute and generate .MOF file
WebServerConfiguration -ConfigurationData $ConfigurationDataFile -OutputPath $DSCTempPath

# Configure the server using the MOF file
Start-DscConfiguration -Wait -Verbose -Path $DSCTempPath
```

#### Set up Variable Substitution
Expand the Substitute Variables in Files section. For Target files, click on #{} and choose the DataFileName variable:

![](template-var-sub.png)

Now, save the template.

Cool!  We've just created a dynamic, re-usable Step Template that will configure IIS Web servers using PowerShell DSC!  Now we need to create a new project to use it.

## Caveat for PowerShell DSC Resource Modules
Earlier in this post, I talked about the `Import-DscResource -Module xWebAdministration` line in our DSC script.  This is a reference to the xWebAdministration PowerShell DSC resource module, which will need to be downloaded from either the [PowerShell Gallery](https://www.powershellgallery.com/packages?q=xwebadministration) or from [GitHub](https://github.com/PowerShell/xWebAdministration).  The caveat to this is that any DSC Modules referenced in your script must be installed **before** the DSC script executes.  

## Make Your Configuration Data File (DSC Configuration) and Your Referenced PowerShell DSC Modules Deployable Packages
Presumably, you have placed your configuration data file (DSC configuration) and referenced PowerShell DSC modules into source control.  Once in source control, your build server can easily package them and ship them to Octopus Deploy for deployment.

## Configure your Project
Now that we have our configuration data file package and our PowerShell DSC Modules package, we can configure our project!

### Create Project Variables

Before we define our process, let's create some variables that will be used in our deployment; Project.PowerShellModulePath, Project.DSCPath, Project.PackageId, and Project.ConfigurationDataFile.  Click on the Variables tab and fill in the variables like this:

![](variables-1.png)

### Define Deployment Process

#### Step 1: Deploy the PowerShell DSC Modules
PowerShell DSC will use the paths defined in $env:PSModulePath to find modules.  For the purposes of this demonstration, we're going to place our modules in `c:\Program Files\WindowsPowerShell\Modules` that we defined in our variable Project.PowerShellModulePath.  

Add a new step to our Project by clicking on Add Step:

![](add-step.png)

Choose the Deploy a Package template:

![](deploy-a-package.png)

To specify a specific location, click on the Configure Features button and enable Custom Installation Directory:

![](custom-install-dir.png)

To reference a variable, click on the #{} to bring up the list and choose Project.PowershellModulePath.  Then click Save.

Warning!  Do **not** choose Purge this directory before installation, there are other modules that PowerShell needs in there.

When done, your step should look something like this:

![](step-1.png)

#### Step 2: Our Custom Step Template
The second step will be our custom step template that we created previously.  Add this step by choosing the Library Step Templates category and then choosing the step, in this case, it is Web Server PowerShell DSC:

![](custom-step-template.png)

Fill in the parameters that we created with variables from our project:

![](step-2.png)

And that's it!  Once we've saved our Project, we can create a release and configure a server!

## PowerShell DSC and Octopus Deploy Combine, Form of Awesome!

Once the deployment has completed, we should see something like the following:

![](deployment-complete.png)

Logging into our Web server, we should find that IIS has been installed with Sites, Application Pools, and Applications defined:

![](web-server.png)

## More Awesomeness with Variable substitution!

But wait!  In our configuration data file we've statically set where the IIS Sites log to, but what if I want something different per project?  This is where we can use the Substitute Variables in Files feature of Octopus Deploy!  

Let's create a variable called Project.LogPath for the log location:

![](log-path.png)

Let's change the `LogPath = "c:\logs"` line in our configuration data file to `LogPath = "#{Project.LogPath}"`.  The #{LogPath} is Octopus Deploy syntax for where the variable LogPath will go.  Don't forget to check the change in so it can be delivered to Octopus Deploy!

Since we enabled the Substitute Variables in Files feature in our custom step template, we're already set up to handle this!

With our variables defined and our new configuration data file package delivered to Octopus Deploy, we can create a new release and deploy!  Once the deployment is complete, we'll pop over to our IIS server and we should see that the log file path has been updated:

![](updated-log-path.png)

## Monitoring for Naughtiness with Machine Policies

Wow!  That's awesome!  I can deploy server configuration changes just like I would an application!  What if someone was naughty and made a change manually?  Didn't you say something about monitoring for drift?  Yup, sure did!  We can tweak Paul's Machine Policy script to show us which items are no longer in desired state, and mark the machine as unhealthy.

Paul's Machine Policy for monitoring drift looked like this:

```PS
$result = Test-DscConfiguration -Verbose -ErrorAction SilentlyContinue

if ($result -eq $false) {
    Write-Host "Machine has drifted"
    Fail-HealthCheck "Machine has drifted"
} elseif ($result -eq $true) {
    Write-Host "Machine has not drifted"
} else {
    Write-Host "No configuration has been applied to the machine yet"
}
```
If we change a few things, we can easily list which resources have drifted:

```PS
# Capture the detailed results
$result = Test-DscConfiguration -Detailed

# Check to see if anything is in the NotInDesiredState collection
if ($result.ResourcesNotInDesiredState.Count -gt 0)
{
	# Loop through the resources
	foreach ($resource in $result.ResourcesNotInDesiredState)
	{
		# Display warning
		Write-Warning "Resource $($resource.ResourceId) is not in desired state!"
	}

	# Fail the health check
	Fail-HealthCheck "Machine has drifted."
}
else
{
	# All good!
	Write-Host "Machine has not drifted."
}
```

Let's test it by stopping the OctopusDeploy.com web site on our IIS server.  After stopping the site, we should see something like this when running a Health Check on the machine:

![](failed-health-check.png)

## Summary

In this post, we created a PowerShell Desired State Configuration (DSC) script, converted it into an Octopus Deploy Step Template, separated Node data into a configuration data file (DSC configuration), and created a Machine Policy for monitoring for drift.

Source code for this post is available at https://github.com/OctopusSamples/DSC-as-a-template
