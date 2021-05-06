---
title: Using classes in custom Step Templates
description: Learn how to implement a class in a custom Step Template
author: shawn.sesna@octopus.com
visibility: private
published: 2022-05-03-1400
metaImage: 
bannerImage: 
tags:
 - 
 - 
---

At Octopus Depoloy, staff are afforded dedicated time to learning in what we refer to as Sharpening time.  During my last Sharpening period, I wanted to see if it would be possible to modify the [DACPAC](https://library.octopus.com/step-templates/e4a60d6f-036f-425d-a3f7-793034fc0f49/actiontemplate-sql-deploy-dacpac-from-package-parameter) step template to use [Azure Active Directory Managed Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) for authentication.  In this post, I will show you how I was able to accomplish this with PowerShell and a custom class.

## Preparation
The first thing we'll need to do is provision an Azure SQL Server and an Azure VM.  This post assumes you're already framliar with how to provision those types of resources and will not cover those topics.

### Configure the VM for a Managed Identity
Once the Azure VM has been created, we will need to configure it to use the Managed Identity feature.  Navigate to the VM resource and click on **Identity** in the left hand pane.

![](azure-vm-identity.png)

Once on the Identity pane, switch the `Status` to `On`

![](azure-vm-managed-identity.png)

And you're done!  The VM can now authenticate to Azure resources such as Azure SQL Server.

### Configure Azure SQL Server for managed identity authentication
There are two methods that you could use to grant authentication of the managed identity to SQL server:
- Configure the VM managed identity as the `Active Directory admin` for the Azure SQL Server
- Add the managed identity as an external login to a database

#### Configure Active Directory admin
Navigate to the Azure SQL Server resource and click on `Active Directory admin`

:::warning
As you can only have one account confugured as an `Active Directory admin`, I would not recommend this approach for Production use.
:::

![](azure-sql-aad-admin.png)

Once on the `Acive Directory admin` screen, click on **Set admin**.  Select the VM you want and click **Select**.  The account will now show as the selected Administrator account, click **Save** to save your changes.

![](azure-sql-select-admin.png)

#### Add managed identity as an external login to the database
Using something like SQL Server Management Studio (SSMS), you can execute a script that will grant permissions to the VM.  Select the database you wish to add permissions to and run the following, the `USER` name is the name of the VM

``` sql
CREATE USER [Octo-AAD-Worker] FROM EXTERNAL PROVIDER
ALTER ROLE db_owner ADD MEMBER [Octo-AAD-Worker]
```

:::warning
Ensure that you are executing the script against the desired database.
:::

![](azure-sql-user-script.png)

Your VM now has the `db_owner` role for the selected database.

### Testing connectivity
To verify that the managed identity is working properly, log on to the server and bring up PowerShell or PowerShell ISE.  Once loaded, run the following script to verify that it is connecting:

``` PowerShell
$response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatabase.windows.net%2F' -Method GET -Headers @{Metadata="true"}
$content = $response.Content | ConvertFrom-Json
$AccessToken = $content.access_token

$connectionString = "Data Source=aad-sql-test.database.windows.net; Initial Catalog=TestDB2;"

$sqlConnection = New-Object System.Data.SqlClient.SqlConnection
$sqlConnection.ConnectionString = $connectionString
$sqlConnection.AccessToken = $AccessToken

$command = $sqlConnection.CreateCommand()
$command.CommandType = [System.Data.CommandType]'Text'
$command.CommandText = "SELECT @@VERSION"

$sqlConnection.Open()

$command.ExecuteNonQuery();

$sqlConnection.Close()
```
Execution of this script should succeed without any error messages and will return a -1 as a result.  -1 does not mean failure in this case and it in fact succeeded.

This script calls the internal to Azure identity service to return an Access Token used to authenticate to the database server.

## DACPAC step template
The DACPAC step template uses .NET objects to interact with SQL Server instead of calling the sqlpackage.exe command line program.  To establish a connection with the database server, the code creates a [DacServices](https://docs.microsoft.com/en-us/dotnet/api/microsoft.sqlserver.dac.dacservices?view=sql-dacfx-150) object passing a Connection String to the constructor.  As you can see in the above script, authenticating using a managed identity requires an Access Token which cannot be added to the connection string itself.  The above script shows that the [SqlConnection](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlconnection?view=dotnet-plat-ext-5.0) object contains the property `AccessToken` which is what is assigned for authentication purposes.  `DacServices`, however, contains no such property.  In order to use managed identity authentication method, you must instantiate a `DacServices` object using an [overloaded constructor](https://docs.microsoft.com/en-us/dotnet/api/microsoft.sqlserver.dac.dacservices.-ctor?view=sql-dacfx-150#Microsoft_SqlServer_Dac_DacServices__ctor_System_String_Microsoft_SqlServer_Dac_IUniversalAuthProvider_) that takes an object that implements the `Microsoft.SqlServer.Dac.IUniversalAuthProvider` interface.

### Creating classes in PowerShell
As of PowerShell version 5, you have the ability to create custom classes within PowerShell itself.

```PowerShell
class AzureADAuth : Microsoft.SqlServer.Dac.IUniversalAuthProvider
{
	[string] GetValidAccessToken()
    {
      # Call Azure API to retrieve the token
      $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatabase.windows.net%2F' -Method GET -Headers @{Metadata="true"} -UseBasicParsing
      $content = $response.Content | ConvertFrom-Json
      $azureAccessToken = $content.access_token
      # Return access token
      return $azureAccessToken
    }
}
```

The problem I ran into was that `class` definitions are evaluated before code execution.  Despite loading the appropriate assemblies prior to the class definition, PowerShell would fail because it could not find the type.  If the DLL was in the Global Assembly Cache (GAC), it might have worked, however, I couldn't assume that would be the case for a step template.

```
+ class AzureADAuth : Microsoft.SqlServer.Dac.IUniversalAuthProvider
+                     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Unable to find type [Microsoft.SqlServer.Dac.IUniversalAuthProvider].
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : TypeNotFound
```

#### Alternate class creation method
My researched found that I could create classes by defining them first within a string variable using C# syntax!  In addition, I could pass in dependent assembly references when adding the type

``` Powershell
# Define C# class
$authClass = @"
public class AzureADAuth : Microsoft.SqlServer.Dac.IUniversalAuthProvider
{
	public string GetValidAccessToken()
    {
    	System.Net.HttpWebRequest request = (System.Net.HttpWebRequest)System.Net.WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://database.windows.net/");
		request.Headers["Metadata"] = "true";
		request.Method = "GET";
		string accessToken = null;
        
        System.Net.HttpWebResponse response = (System.Net.HttpWebResponse)request.GetResponse();
        
        System.IO.StreamReader streamResponse = new System.IO.StreamReader(response.GetResponseStream());
        string stringResponse = streamResponse.ReadToEnd();
        System.Web.Script.Serialization.JavaScriptSerializer j = new System.Web.Script.Serialization.JavaScriptSerializer();
        System.Collections.Generic.Dictionary<string, string> list = (System.Collections.Generic.Dictionary<string, string>) j.Deserialize(stringResponse, typeof(System.Collections.Generic.Dictionary<string, string>));
        accessToken = list["access_token"];

		return accessToken;
    }
}
"@

# Create new object
Add-Type -TypeDefinition $authClass -ReferencedAssemblies @("$($DacDLL.FullName)", "System.Net", "System.Web.Extensions", "System.Collections")

$dacServices = New-Object Microsoft.SqlServer.Dac.DacServices $connectionString, $azureAuth
```

:::hint
The DACPAC step template is quite long, only the relevant portions of code for class creation are included in this post.  The searching for the location of the value for $DacDLL is not included here.
:::

### Deployment
Testing our code shows a successful deployment using the managed identity method!

![](octopus-deployment.png)

## Conclusion
In this post I demonstrated methods on creating custom classes in PowerShell including implementation of an interface.  I am hoping that what I learned will help you work with PowerShell in the future.

Happy Deployments!

:::hint
The modifications for the Step Template are incomplete at this point, I'm still hoping to add other authentication methods such as Azure Active Directory Integrated and Azure Active Directory Username/Password.
:::

