# PowerShell and SQL Server: Common Scenarios
Our goal at Octopus Deploy has always been to make automated application deployments easy, and application deployments often require database management during the process. My goal here is to provide some common examples of SQL Server management through PowerShell to make integration into deployments that much more straightforward.

## The SqlServer PowerShell Module
Microsoft recommends using the **SqlServer** module for interacting with SQL Server from PowerShell. While this isn't used in all of the following examples, it contains many useful cmdlets for SQL Server administration. 

## Test Connectivity to SQL Server
So you want to connect to a SQL Server database. Presumably, you have the connection details handy within the deployment. One of the simplest ways to test a connection is to use a connection string and the basic **SqlClient** class:

```
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

With the connection tested at the beginning of your deployment, you can move forward, knowing that you will be able to make the calls you need against your SQL Server.

## Create Login
For proper segregation of permissions within your server or instance, you may find the need to create a new login for your application or associated service. A cmdlet from the SqlServer module will do the trick:

```
# To run in a non-interactive mode, such as through an Octopus deployment, you will most likely need to pass the new login credentials as a PSCredential object.
$pass = ConvertTo-SecureString "Th!sI$Y0urP4ss" -AsPlainText -Force

# Create the PSCredential object
$loginCred = New-Object System.Management.Automation.PSCredential("YourLogin",$loginCred)

# Create login using the Add-SqlLogin cmdlet
Add-SqlLogin -ServerInstance YourInstance -LoginPSCredential $loginCred -LoginType SqlLogin
```

This cmdlet greatly simplifies the creation of a SQL Server login along with the flexibility of using built-in flags such as `-MustChangePasswordAtNextLogin` or `-ConnectionTimeout.` If you are using this command within an Octopus deployment process, you may find it handy to include flags such as `-Enable` to ensure the login is available later in your deployment or `-GrantConnectSql` to allow the login to connect to the database engine. For more information, see the [Add-SqlLogin reference](https://docs.microsoft.com/en-us/powershell/module/sqlserver/Add-SqlLogin).

## Create Database and Assign Owner
Surprisingly, Microsoft does not provide a cmdlet out of the box to create a database, so there are two main routes one can follow to get a database up and running without a prior backup. Both are fairly simple. (If you *do* have a backup, the **Restore-SqlDatabase** cmdlet is your friend.)

```
# Run straight SQL against your instance.
# This could also come from a file.
Invoke-Sqlcmd -Query "CREATE DATABASE YourDB" -ServerInstance YourInstance

# Use SMO objects to do the heavy lifting.
$dbname = "YourDB"

# Create a SQL Server database object
$srv = New-Object Microsoft.SqlServer.Management.Smo.Server("YourInstance")
if($srv.Databases[$dbname] -ne $null)
{
    $db = New-Object Microsoft.SqlServer.Management.Smo.Database($srv, $dbname)

    # Create the database
    $db.Create()
}
```

Now that your database is in place, let's change the owner to "NewOwner":

```
# Here we'll need a user with administrative privileges to set the owner.
# Let's say that $SqlAdmin contains the username and $SqlAdminPass contains the password as a secure string.
$dbname = "YourDB"

# Create the server connection object
$conn = New-Object Microsoft.SqlServer.Management.Common.ServerConnection("YourInstance", $SqlAdmin, $SqlAdminPass)

# Create the server object
$srv = New-Object Microsoft.SqlServer.Management.Smo.Server($conn)

# Check to see if a database with that name already exists
if($srv.Databases[$dbname] -ne $null)
{
    # If it does not exist, create it
    $db = New-Object Microsoft.SqlServer.Management.Smo.Database($srv, $dbname)
    $db.SetOwner("NewOwner")
}
else
{
    # There was an error creating the database
}
```

## Run a Script From a File
With the new database in place, perhaps there are changes to settings that need to be made, or some tables are missing their default data. PowerShell can run SQL scripts directly from a file to resolve these issues. Let's assume that `alter_script.sql` is in a known location (potentially inside a package in your Octopus Deployment) That script can be run directly from PowerShell with the following command:

```
# Assumes you are working from the package directory
Invoke-Sqlcmd -InputFile .\alter_script.sql -ServerInstance YourInstance -Database YourDB
```

## Summary
Whether you are automating the configuration of a new database each time you deploy a release from Octopus, or are running one-time commands against an existing database, PowerShell and the SqlServer module are the right tools for the job.

Do you have any additional scenarios or specific questions? Leave a comment here or join us on our [Community Slack](https://octopus.com/slack).

## References
- For more information on the **SqlServer** module, please see [this article](https://docs.microsoft.com/en-us/sql/powershell/download-sql-server-ps-module) from Microsoft
- [SqlServer module installation instructions](https://docs.microsoft.com/en-us/sql/powershell/download-sql-server-ps-module)
- [SqlServer Module cmdlets documentation](https://docs.microsoft.com/en-us/powershell/module/sqlserver/?view=sqlserver-ps)
