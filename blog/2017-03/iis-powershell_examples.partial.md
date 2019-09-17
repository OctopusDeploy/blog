### Creating sites (simple)


```powershell WebAdministration
Import-Module WebAdministration

New-Website -Name "Website1" -Port 80 -IPAddress "*" -HostHeader "" -PhysicalPath "C:\Sites\Website1"
```

```powershell IISAdministration
Import-Module IISAdministration

New-IISSite -Name "Website1" -BindingInformation "*:80:" -PhysicalPath "C:\Sites\Website1"

# Examples of -BindingInformation: 
#    "*:80:"                - Listens on port 80, any IP address, any hostname
#    "10.0.0.1:80:"         - Listens on port 80, specific IP address, any host
#    "*:80:myhost.com"      - Listens on port 80, specific hostname
```

### Creating sites (advanced)

Most likely, you’ll want to specify a few extra settings when you create a real site. To do that, you can get the site after creating it, and add the extra settings. Since we are making multiple changes, we can use delayed commits to write them all to applicationHost.config at once.

Here we’ll add an additional binding, and set the site ID:

```powershell WebAdministration
Import-Module WebAdministration

New-Item -Path "IIS:\Sites" -Name "Website1" -Type Site -Bindings @{protocol="http";bindingInformation="*:8021:"}
Set-ItemProperty -Path "IIS:\Sites\Website1" -name "physicalPath" -value "C:\Sites\Website1"
Set-ItemProperty -Path "IIS:\Sites\Website1" -Name "id" -Value 4
New-ItemProperty -Path "IIS:\Sites\Website1" -Name "bindings" -Value (@{protocol="http";bindingInformation="*:8022:"}, @{protocol="http";bindingInformation="*:8023:"})

Start-Website -Name "Website1"
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager
$site = $manager.Sites.Add("Website1", "http", "*:8022:", "C:\Sites\Website1")
$site.Id = 4
$site.Bindings.Add("*:8023:", "http")
$site.Bindings.Add("*:8024:", "http")
$manager.CommitChanges()
```

### Creating applications in virtual directories

Most of the time, when people think of deploying a .NET app to a *virtual directory*, they mean creating an *application* underneath a web site. The example below creates an application that would be viewable at `http://site/MyApp`. We also assign an application pool:

```powershell WebAdministration
Import-Module WebAdministration

New-Item -Type Application -Path "IIS:\Sites\Website1\MyApp" -physicalPath "C:\Sites\MyApp"
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager
$app = $manager.Sites["Website1"].Applications.Add("/MyApp", "C:\Sites\MyApp")
$manager.CommitChanges()
```

### Creating application pools

You need to assign each application (website or application in a virtual directory) to an *application pool*. The application pool defines the executable process in which requests to the application are handled. 

IIS comes with a handful of application pools already defined for common options, but I always recommend creating your own application pool for each website or application that you deploy. This provides process-level isolation between applications and lets you set different permissions around what each application can do. The examples below show many of the common application pool settings. For the IIS Administration module, there are no built-in CmdLets to create application pools, so you have to do it with the `ServerManager` object directly:

```powershell WebAdministration
Import-Module WebAdministration

New-Item -Path "IIS:\AppPools" -Name "My Pool" -Type AppPool

# What version of the .NET runtime to use. Valid options are "v2.0" and 
# "v4.0". IIS Manager often presents them as ".NET 4.5", but these still 
# use the .NET 4.0 runtime so should use "v4.0". For a "No Managed Code" 
# equivalent, pass an empty string.
Set-ItemProperty -Path "IIS:\AppPools\My Pool" -name "managedRuntimeVersion" -value "v4.0"

# If your ASP.NET app must run as a 32-bit process even on 64-bit machines
# set this to $true. This is usually only important if your app depends 
# on some unmanaged (non-.NET) DLL's. 
Set-ItemProperty -Path "IIS:\AppPools\My Pool" -name "enable32BitAppOnWin64" -value $false

# Starts the application pool automatically when a request is made. If you 
# set this to false, you have to manually start the application pool or 
# you will get 503 errors. 
Set-ItemProperty -Path "IIS:\AppPools\My Pool" -name "autoStart" -value $true

# What account does the application pool run as? 
# "ApplicationPoolIdentity" = best
# "LocalSysten" = bad idea!
# "NetworkService" = not so bad
# "SpecificUser" = useful if the user needs special rights. See other examples
# below for how to do this.
Set-ItemProperty -Path "IIS:\AppPools\My Pool" -name "processModel" -value @{identitytype="ApplicationPoolIdentity"}

# Older applications may require "Classic" mode, but most modern ASP.NET
# apps use the integrated pipeline. 
# 
# On newer versions of PowerShell, setting the managedPipelineMode is easy -
# just use a string:
# 
#   Set-ItemProperty -Path "IIS:\AppPools\My Pool 3" `
#      -name "managedPipelineMode" ` 
#      -value "Integrated"
# 
# However, the combination of PowerShell and the IIS module in Windows 
# Server 2008 and 2008 R2 requires you to specify the value as an integer.
#
#  0 = Integrated
#  1 = Classic
# 
# If you hate hard-coding magic numbers you can do this (or use the string
# if 2008 support isn't an issue for you):
#  
#   Add-Type -Path "${env:SystemRoot}\System32\inetsrv\Microsoft.Web.Administration.dll"
#   $pipelineMode = [Microsoft.Web.Administration.ManagedPipelineMode]::Integrated
#   Set-ItemProperty -Path "..." -name "managedPipelineMode" -value ([int]$pipelineMode)
# 
# If this DLL doesn't exist, you'll need to install the IIS Management 
# Console role service.
Set-ItemProperty -Path "IIS:\AppPools\My Pool" -name "managedPipelineMode" -value 0

# This setting was added in IIS 8. It's different to autoStart (which means 
# "start the app pool when a request is made") in that it lets you keep 
# an app pool running at all times even when there are no requests. 
# Since it was added in IIS 8 you may need to check the OS version before
# trying to set it. 
# 
# "AlwaysRunning" = application pool loads when Windows starts, stays running
# even when the application/site is idle. 
# "OnDemand" = IIS starts it when needed. If there are no requests, it may 
# never be started. 
if ([Environment]::OSVersion.Version -ge (new-object 'Version' 6,2)) {
    Set-ItemProperty -Path "IIS:\AppPools\My Pool" -name "startMode" -value "OnDemand"
}
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager
$pool = $manager.ApplicationPools.Add("My Pool")

# Older applications may require "Classic" mode, but most modern ASP.NET
# apps use the integrated pipeline
$pool.ManagedPipelineMode = "Integrated"

# What version of the .NET runtime to use. Valid options are "v2.0" and 
# "v4.0". IIS Manager often presents them as ".NET 4.5", but these still 
# use the .NET 4.0 runtime so should use "v4.0". For a "No Managed Code" 
# equivalent, pass an empty string.
$pool.ManagedRuntimeVersion = "v4.0"

# If your ASP.NET app must run as a 32-bit process even on 64-bit machines
# set this to $true. This is usually only important if your app depends 
# on some unmanaged (non-.NET) DLL's. 
$pool.Enable32BitAppOnWin64 = $false

# Starts the application pool automatically when a request is made. If you 
# set this to false, you have to manually start the application pool or 
# you will get 503 errors. 
$pool.AutoStart = $true

# "AlwaysRunning" = application pool loads when Windows starts, stays running
# even when the application/site is idle. 
# "OnDemand" = IIS starts it when needed. If there are no requests, it may 
# never be started. 
$pool.StartMode = "OnDemand"

# What account does the application pool run as? 
# "ApplicationPoolIdentity" = best
# "LocalSysten" = bad idea!
# "NetworkService" = not so bad
# "SpecificUser" = useful if the user needs special rights
$pool.ProcessModel.IdentityType = "ApplicationPoolIdentity"

$manager.CommitChanges()
```

### Assigning application pools

Once you’ve defined your application pool, you must assign applications to it. The examples below show you how to assign websites or applications in a virtual directory to your new application pool:
```powershell WebAdministration
Import-Module WebAdministration

# Assign the application pool to a website
Set-ItemProperty -Path "IIS:\Sites\Website1" -name "applicationPool" -value "My Pool"

# Assign the application pool to an application in a virtual directory
Set-ItemProperty -Path "IIS:\Sites\Website1\MyApp" -name "applicationPool" -value "My Pool"
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager

# Assign to a website
$website = $manager.Sites["Website1"]
$website.Applications["/"].ApplicationPoolName = "My Pool"

# Assign to an application in a virtual directory
$website = $manager.Sites["Website1"]
$website.Applications["/MyApp"].ApplicationPoolName = "My Pool"

$manager.CommitChanges()
```

### Check whether sites, virtual directories, or application pools already exist

You’ll re-deploy your application to IIS many times over the course of a project, so you can’t assume the script is being run for the first time. The examples below show a pattern for checking whether a site, application, or application pool already exists before making a change:

```powershell WebAdministration
Import-Module WebAdministration

# The pattern here is to use Test-Path with the IIS:\ drive provider

if ((Test-Path "IIS:\AppPools\My Pool") -eq $False) {
    # Application pool does not exist, create it...
    # ...
}

if ((Test-Path "IIS:\Sites\Website1") -eq $False) {
    # Site does not exist, create it...
    # ...
}

if ((Test-Path "IIS:\Sites\Website1\MyApp") -eq $False) {
    # App/virtual directory does not exist, create it...
    # ...
}
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager

# The pattern here is to get the things you want, then check if they are null

if ($manager.ApplicationPools["My Pool"] -eq $null) {
    # Application pool does not exist, create it...
    # ...
}

if ($manager.Sites["Website1"] -eq $null) {
    # Site does not exist, create it...
    # ...
}

if ($manager.Sites["Website1"].Applications["/MyApp"] -eq $null) {
    # App/virtual directory does not exist, create it...
    # ...
}

$manager.CommitChanges()
```

### Change physical path of a site or application

When deploying a new version of an application, my preference (and the way Octopus Deploy works) is to deploy to a fresh new folder on disk, then update IIS to point to it. So you begin with:

    C:\Sites\Website1\1.0   <--- IIS points here

You deploy the new version:

    C:\Sites\Website1\1.0   <--- IIS points here
    C:\Sites\Website1\1.1

You can then make any necessary changes to configuration files, etc. and then update IIS to point to it:

    C:\Sites\Website1\1.0
    C:\Sites\Website1\1.1   <--- Now IIS points here

Should you ever need to roll back in a hurry, you can leave the old folder on disk and point back to it:

    C:\Sites\Website1\1.0   <--- IIS points here (we rolled back manually)
    C:\Sites\Website1\1.1   

```powershell WebAdministration
Import-Module WebAdministration

# The pattern here is to use Test-Path with the IIS:\ drive provider

Set-ItemProperty -Path "IIS:\Sites\Website1" -name "physicalPath" -value "C:\Sites\Website1\1.1"
Set-ItemProperty -Path "IIS:\Sites\Website1\MyApp" -name "physicalPath" -value "C:\Sites\Website1\1.1"
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager

# Remember, in the IIS Administration view of the world, sites contain 
# applications, and applications contain virtual directories, and it is 
# virtual directories that point at a physical path on disk. 

# Change for a top-level website
$manager.Sites["Website1"].Applications["/"].VirtualDirectories["/"].PhysicalPath = "C:\Sites\Website1\1.1"

# Change for an app within a website
$manager.Sites["Website1"].Applications["/MyApp"].VirtualDirectories["/"].PhysicalPath = "C:\Sites\Website1\1.1"

$manager.CommitChanges()
```

### Changing authentication methods

IIS supports a number of authentication methods. As described above, these are *locked* to `applicationHost.config` by default, if you want to automatically enable them, the examples below show how to enable/disable: 

 - Anonymous authentication
 - Basic authentication
 - Digest authentication
 - Windows authentication

IIS Manager also shows `ASP.NET impersonation` and `Forms authentication` as settings at the same level, but these are actually set in your app’s `web.config` file so I’ve left them out here:
```powershell WebAdministration
Import-Module WebAdministration

# The pattern here is to use Test-Path with the IIS:\ drive provider. 

Set-WebConfigurationProperty `
    -Filter "/system.webServer/security/authentication/windowsAuthentication" `
    -Name "enabled" `
    -Value $true `
    -Location "Website1/MyApp" `
    -PSPath IIS:\    # We are using the root (applicationHost.config) file

# The section paths are:
# 
#  Anonymous: system.webServer/security/authentication/anonymousAuthentication
#  Basic:     system.webServer/security/authentication/basicAuthentication
#  Windows:   system.webServer/security/authentication/windowsAuthentication
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager

# ServerManager makes it easy to get the various config files that belong to 
# an app, or at the applicationHost level. Since this setting is locked
# to applicationHost, we need to get the applicationHost configuration. 
$config = $manager.GetApplicationHostConfiguration()

# Note that we have to specify the name of the site or application we are 
# editing, since we are working with individual <location> sections within
# the global applicationHost.config file.
$section = $config.GetSection(`
    "system.webServer/security/authentication/windowsAuthentication", `
    "Website1")
$section.Attributes["enabled"].Value = $true

# The section paths are:
# 
#  Anonymous: system.webServer/security/authentication/anonymousAuthentication
#  Basic:     system.webServer/security/authentication/basicAuthentication
#  Windows:   system.webServer/security/authentication/windowsAuthentication

# Changing options for an application in a virtual directory is similar, 
# just specify the site name and app name together:
$section = $config.GetSection(`
    "system.webServer/security/authentication/windowsAuthentication", `
    "Website1/MyApp")
$section.Attributes["enabled"].Value = $true

$manager.CommitChanges()
```

### Changing logging settings

HTTP request logging is provided by IIS and can be specified server-wide or at the individual site level. Applications and virtual directories can disable logging, but they can’t do any logging of their own. The examples below show how to set logging at the site level. 

Logging settings are stored in `applicationHost.config` underneath the site:

```xml
<system.applicationHost>
    <!-- ... -->
    <sites>
        <site name="Default Web Site" id="1">
            <bindings>
                <binding protocol="http" bindingInformation="*:80:" />
            </bindings>
            <logFile logFormat="IIS" directory="%SystemDrive%\inetpub\logs\LogFiles1" period="Hourly" />
        </site>
        <siteDefaults>
            <logFile logFormat="W3C" directory="%SystemDrive%\inetpub\logs\LogFiles" />
            <traceFailedRequestsLogging directory="%SystemDrive%\inetpub\logs\FailedReqLogFiles" />
        </siteDefaults>
        <!-- ... -->
```
```powershell WebAdministration
Import-Module WebAdministration

$settings = @{ `
    logFormat="W3c";                `   # Formats:   W3c, Iis, Ncsa, Custom
    enabled=$true;                  `
    directory="C:\Sites\Logs";      `
    period="Daily";                 `
}

Set-ItemProperty "IIS:\Sites\Website1" -name "logFile" -value $settings
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager

$site = $manager.Sites["Website1"]
$logFile = $site.LogFile
$logFile.LogFormat = "W3c"               # Formats:   W3c, Iis, Ncsa, Custom
$logFile.Directory = "C:\Sites\Logs"     
$logFile.Enabled = $true
$logFile.Period = "Daily"

$manager.CommitChanges()
```

### Running application pools as a specific user

You can usually get by running your application pools as the `ApplicationPoolIdentity` accounts. This creates a virtual account for each different application pool automatically, isolating them from each other. On the local machine, you can grant access to resources like the file system to each separate application pool. For remote resources (like an SQL Server on a different machine), the application pool identities act as Network Service, so you can grant access at the machine level. Learn more about [application pool identities](https://www.iis.net/learn/manage/configuring-security/application-pool-identities). 

For more control over what the application pool can do, you should run it under a specific, custom user account. You’ll want to use [`aspnet_regiis`](https://msdn.microsoft.com/en-us/library/k6h9cz8h.aspx) to give your custom account all the permissions it needs to run as an application pool and execute ASP.NET requests. You can then set your application pool to run as that user:

```powershell WebAdministration
Import-Module WebAdministration

New-Item -Path "IIS:\AppPools" -Name "My Pool" -Type AppPool

$identity = @{  `
    identitytype="SpecificUser"; `
    username="My Username"; `
    password="My Password" `
}
Set-ItemProperty -Path "IIS:\AppPools\My Pool" -name "processModel" -value $identity
```

```powershell IISAdministration
Import-Module IISAdministration

$manager = Get-IISServerManager
$pool = $manager.ApplicationPools.Add("My Pool")
$pool.ProcessModel.IdentityType = "SpecificUser"
$pool.ProcessModel.Username = "My User"
$pool.ProcessModel.Password = "Password"

$manager.CommitChanges()
```

