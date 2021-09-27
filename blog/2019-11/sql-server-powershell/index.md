---
title: "SQL Server and PowerShell: Practical Examples"
description: SQL Server database administration using PowerShell
author: james.chatmas@octopus.com
visibility: public
bannerImage: sql-server-powershell-examples.png
bannerImageAlt: SQL Server and PowerShell Practical Examples
metaImage: sql-server-powershell-examples.png
published: 2019-11-26
tags:
 - DevOps
 - PowerShell
---

![SQL Server and PowerShell: Practical Examples](sql-server-powershell-examples.png)

One of our goals at Octopus Deploy has always been to help make automated application deployments easy, and application deployments often require database management during the process. My goal in this post, is to provide some common examples of SQL Server database management with PowerShell to make integration into deployments much more straightforward.

The source for all of these examples is available on [GitHub](https://github.com/OctopusSamples/sql-server-powershell-examples). If you run into any problems or have suggestions for changes, feel free to post to the GitHub repository’s issue list or send a pull request!

!toc

## Installing the SQL Server PowerShell module
Microsoft recommends using the **SqlServer** module for interacting with SQL Server from PowerShell. This isn’t used in all of the following examples, but it does contain many useful cmdlets for SQL Server administration.

The **SqlServer** module can be installed from the PowerShell Gallery using the following command:

```ps
Install-Module -Name SqlServer
```

Additionally, if this module is already installed, you can update it using the following command if you are using PowerShell 5.0 or later:

```ps
Update-Module -Name SqlServer
```

If you are using a PowerShell version earlier than 5.0, use these commands:

```ps
Uninstall-Module -Name SqlServer
Install-Module -Name SqlServer
```

For more information on installing the **SqlServer** module, please see this Microsoft doc: [Install SQL Server PowerShell module](https://docs.microsoft.com/en-us/sql/powershell/download-sql-server-ps-module).

## Test connectivity to SQL Server

One of the simplest ways to test a connection to a SQL Server database is to use a connection string and the basic **SqlClient** class:

```ps
try
{
    # This is a simple user/pass connection string.
    # Feel free to substitute "Integrated Security=True" for system logins.
    $connString = "Data Source=YourInstance;Database=YourDB;User ID=YourUser;Password=YourPassword"

    #Create a SQL connection object
    $conn = New-Object System.Data.SqlClient.SqlConnection $connString

    #Attempt to open the connection
    $conn.Open()
    if($conn.State -eq "Open")
    {
        # We have a successful connection here
        # Notify of successful connection
        Write-Host "Test connection successful"
        $conn.Close()
    }
    # We could not connect here
    # Notify connection was not in the "open" state
}
catch
{
    # We could not connect here
    # Notify there was an error connecting to the database
}
```

With the connection tested at the beginning of your deployment, you can move forward knowing you can make the calls you need against your SQL Server.

## Create SQL Server login

For proper segregation of permissions within your server or instance, you may want to create a new login for your application or associated service. A cmdlet from the SQL Server module will do the trick:

```ps
# To run in a non-interactive mode, such as through an Octopus deployment, you will most likely need to pass the new login credentials as a PSCredential object.
$pass = ConvertTo-SecureString "Th!sI5Y0urP4ss" -AsPlainText -Force

# Create the PSCredential object
$loginCred = New-Object System.Management.Automation.PSCredential("NewUser",$pass)

# Create login using the Add-SqlLogin cmdlet
Add-SqlLogin -ServerInstance YourInstance -LoginPSCredential $loginCred -LoginType SqlLogin
```

This cmdlet greatly simplifies the creation of a SQL Server login along with the flexibility of using built-in flags such as `-MustChangePasswordAtNextLogin` or `-ConnectionTimeout.` If you are using this command within an Octopus deployment process, you may find it useful to include the following flags:

- `-Enable` to ensure the login is available later in your deployment.
- `-GrantConnectSql` to allow the login to connect to the database engine.

For more information, see the [Add-SqlLogin reference](https://docs.microsoft.com/en-us/powershell/module/sqlserver/Add-SqlLogin).

## Create SQL Server database and assign an owner

Surprisingly, Microsoft does not provide a cmdlet out of the box to create a database, so there are two main routes you can follow to get a database up and running without a prior backup. Both are fairly simple. Creating a new blank database is accomplished with either of the following commands:

Run straight SQL against your instance to create the database:
```ps
# This query could also come from a file
Invoke-Sqlcmd -Query "CREATE DATABASE YourDB" -ServerInstance YourInstance
```

Alternatively, using SQL Server Management Objects (SMO) objects to do the heavy lifting:

```ps
#Name your database
$dbname = "YourDB"
# Create a SQL Server database object
$srv = New-Object Microsoft.SqlServer.Management.Smo.Server("YourInstance")
if($null -ne $srv.Databases[$dbname])
{
    $db = New-Object Microsoft.SqlServer.Management.Smo.Database($srv, $dbname)

    # Create the database
    $db.Create()
}
```

If you *do* have a database backup you would like to restore rather than creating a blank database, the **Restore-SqlDatabase** cmdlet is your friend:

```ps
$backupFile = "\\SQLBackups\YourDBBackup.bak"
Restore-SqlDatabase -ServerInstance YourInstance -Database YourDB -BackupFile $backupFile
```
More information on the **Restore-SqlDatabase** cmdlet can be found in this Microsoft doc: [Restore-SqlDatabase](https://docs.microsoft.com/en-us/powershell/module/sqlserver/restore-sqldatabase?view=sqlserver-ps).

Now that your database is in place, let’s change the database owner to `NewOwner`:

```ps
# Here we'll need a user with administrative privileges to set the owner.
# Let's say that $SqlAdmin contains the username and $SqlAdminPass contains the password as a secure string.
$dbname = "YourDB"

# Create the server connection object
$conn = New-Object Microsoft.SqlServer.Management.Common.ServerConnection("YourInstance", $SqlAdmin, $SqlAdminPass)

# Create the server object
$srv = New-Object Microsoft.SqlServer.Management.Smo.Server($conn)

# Check to see if a database with that name already exists
if($null -ne $srv.Databases[$dbname])
{
    # If it does not exist, create it
    $db = New-Object Microsoft.SqlServer.Management.Smo.Database($srv, $dbname)
    $db.SetOwner("NewOwner")
}
else
{
    # There was an error creating the database object
}
```

## Run a SQL script from a file

With the new database in place, perhaps there are changes to settings that need to be made, or some tables are missing their default data. PowerShell can run SQL scripts directly from a file to resolve these issues. Let’s assume that `alter_script.sql` is in a known location (potentially inside a package in your Octopus Deployment). That script can be run directly from PowerShell with the following command:

```ps
# Assumes you are working from the package directory
Invoke-Sqlcmd -InputFile .\alter_script.sql -ServerInstance YourInstance -Database YourDB
```

Additional `Invoke-Sqlcmd` information can be found at this Microsoft doc: [Invoke-Sqlcmd cmdlet](https://docs.microsoft.com/en-us/powershell/module/sqlserver/invoke-sqlcmd).

## Run inline SQL commands
Finally, you might need to run some SQL queries to verify the database contains the correct data (a sanity check that everything is well). PowerShell has the ability to run inline SQL queries directly from the console window. Let’s run a quick query to check for the number of rows in a table:

```ps
$rowcount = "SELECT COUNT(*) FROM YourTable"
Invoke-Sqlcmd -ServerInstance YourInstance -Database YourDatabase -Query $rowcount
```

## Conclusion

Whether you are automating the configuration of a new database each time you deploy a release from Octopus or running one-time commands against an existing database, PowerShell and the SQL Server module are the right tools for the job.

Do you have any additional scenarios or specific questions? Leave a comment here or join us on our [Community Slack](https://octopus.com/slack).

## References
- [SqlServer PowerShell module](https://docs.microsoft.com/en-us/sql/powershell/download-sql-server-ps-module)
- [SqlServer Module installation instructions](https://docs.microsoft.com/en-us/sql/powershell/download-sql-server-ps-module)
- [SqlServer Module cmdlets documentation](https://docs.microsoft.com/en-us/powershell/module/sqlserver/?view=sqlserver-ps)
