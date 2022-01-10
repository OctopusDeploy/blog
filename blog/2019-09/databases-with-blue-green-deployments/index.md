---
title: Automated blue/green database deployments
description: Learn some techniques for automating database deployments when using a blue/green deployment strategy.
author: bob.walker@octopus.com
visibility: public
metaImage: img-blog-database-deployments-blue-green.png
bannerImage: img-blog-database-deployments-blue-green.png
bannerImageAlt: Illustration showing two database (one green and one blue) on a seesaw
published: 2019-09-16
tags:
 - DevOps
 - Database Deployments
 - Deployment Patterns
---

![Illustration showing two database (one green and one blue) on a seesaw](img-blog-database-deployments-blue-green.png)

Nobody wants to do deployments at 2 a.m. on Saturday morning, but several years ago, I worked on an application, and that was the only time an extended outage could be scheduled. A quick deployment and verification took two hours. A typical deployment took four hours.  I wanted to implement blue/green deployments, which allow for zero-downtime deployments, but the database for the application made everything complicated enough to stop us from moving to blue/green deployments.  

In this post, I’ll walk through some techniques I’ve learned since that time to make blue/green deployments with a database achievable and straightforward.      

I’ll cover high-level concepts and recommendations, but I won’t go into detail about how to do blue/green deployments with Octopus Deploy. We’ll cover that in a later post.

!toc

## A brief introduction to blue/green deployments

Blue-green deployments have two identical production environments, one is labeled `Blue` and the other is labeled `Green`.  Only one of the environments is ever live, and deployments are always done to the inactive environment. For example, if `Green` is the live environment, deployment is done to the `Blue` (inactive) environment, and after verification has occurred, a switchover happens, which makes the `Blue` environment the live environment, and the `Green` environment inactive.

![Blue/Green Deployments](blue-green-deployments.png)

There are several advantages to this approach.  Rollbacks are just a matter of switching from `Blue` to `Green` or `Green` to `Blue`.  When switchovers occur, they are seamless because the code has already been running, and there is no need to wait for it to compile or warm-up.  Changes are verified in production without any customers hitting the code, which reduces the risk in the deployments.  If something doesn’t work, you don’t make the switch, and you can try again.

It’s important to note that not all applications can take advantage of blue/green deployments.  It’s a combination of architecture, how stateful the app is, and the technology being used.  The more stateless and decoupled the application, the better the chance the blue/green deployment strategy is suitable. For instance, a .NET Core Web API with an Angular front-end is better suited for blue/green deployments than an ASP.NET WebForms application that requires sticky sessions from the load balancer and doesn't have a business logic or data layer.

## The scenario

This post covers a complex scenario using a [Single Page Application](https://en.wikipedia.org/wiki/Single-page_application)(SPA) with an ASP.NET Web API back-end connected to a dedicated database. The following changes will be made to the app:

- In the UI, the fields `First Name` and `Last Name` are being combined into a single field called `Full Name`. Not all cultures use first and last names.
- A new column called `CustomerFullName` will be added to the customer table.
- The pre-existing `CustomerFirstName` and `CustomerLastName` columns are also on the customer table.  
- `CustomerFullName` will be populated by combining `CustomerFirstName` and `CustomerLastName` when `CustomerFullName` is `Null`.
- If data exists in `CustomerFirstName` and `CustomerLastName` and an API request is sent with only `CustomerFullName` then do not set those columns to `Null`.  In addition, do not try to split up `CustomerFullName` as that is very difficult and error-prone.

Admittedly, this scenario is relatively complex.  Most database changes don’t combine two columns into one and try to backfill a new column, but if this scenario can be solved, the majority of other scenarios can too.  We’ll walk through each change, the questions to consider, recommendations for how to solve them, and a lot of different changes that must be completed for successful blue/green deployments.

The blue and green environments for our deployments in this post, look like this:

 - `Green` is live.
 - `Blue` is inactive.  

That means, we’ll deploy to `Blue`, and after `Blue` is verified, it will become the live environment, and `Green` will become inactive.

:::warning
The specific steps in this post are recommendations only and do not cover every possible change you can make to a database. My goal is to provide you with something you can modify to meet your needs.  
:::

## How database changes add complexity

After-hours deployments are done to avoid users accessing the application while the code and database changes are being deployed and verified.  Because all the changes happen at once, a backfill script will be written to set `CustomerFullName` to `CustomerFirstName` and `CustomerLastName`.  If a user accesses the application when these changes are happening, it will cause errors and confusion.

A very simplified deployment process for that change could look like this:

1. Disable user access or turn off the website.
2. Run the script to add the column `CustomerFullName`.
3. Deploy the code to `Production`.
4. Run the backfill script to populate `CustomerFullName`.
5. Run the script to remove `CustomerFirstName` and `CustomerLastName`.
6. Verify the code in `Production`.

To accomplish the same changes with blue/green deployments requires a lot more planning.  Without blue/green deployments, the only worry is that someone might use the application before everything is deployed and verified.  If the user gets an error indicating a column is missing, no worries, they shouldn’t be in the system at that point anyway.  That is not the case with blue/green deployments.  Users will be in the application; they will be running on the `Green` servers, which means data is going to be manipulated and queried.  

Here are some of the scenarios to consider when doing the same change with blue/green deployments:

- The application and database changes will be deployed to `Blue`, including the new column `CustomerFullName`.  Does the application use any stored procedures?  Specifically, around inserting/updating data into the customer table?  The code running on `Green` won’t know about any new parameters added to those stored procedures.
- When the changes are deployed to `Blue`, there will be no data in the `CustomerFullName` column.  When should that column be backfilled?  Using the same backfill script created for a downtime deployment?  
- Verification needs to happen after the changes to the application and database are deployed to `Blue`. During that time, users will be using the application on `Green` which is still running code referencing `CustomerFirstName` and `CustomerLastName` columns.  Those columns cannot be deleted until after verification is complete and `Blue` becomes active.  When should those columns be deleted?  As soon as `Blue` becomes active?  
- While the updated code and database is being verified on `Blue` users will be adding and updating records in the customer table using the code on `Green`.  If a backfill script is run prior to verification, those changes would not be picked up.  Should the backfill script be rerun?
- Because this is an SPA App, the JavaScript won’t know when `Blue` becomes live, and `Green` becomes inactive.  The JavaScript is stored in the user’s browser.  Users using the application when `Blue` becomes live will be sending API requests with only the fields for `CustomerFirstName` and `CustomerLastName`.   The `CustomerFullName` field will not be sent in during this time.  It could take anywhere from one minute to several days before users start requesting updated JavaScript files from the server.

The first try at the blue/green deployment process for this change could look like this:

1. Run the script to add the column `CustomerFullName`.
2. Run the backfill script to populate `CustomerFullName`.
3. Deploy code to the `Blue` environment.
4. Verify the code and database changes in the `Blue` environment.
5. Swap the live environment from `Green` to `Blue`.
6. Run the backfill script to populate `CustomerFullName`.

That’s not the ideal blue/green deployment process. It runs the backfill script multiple times, and it doesn’t answer when `CustomerFirstName` and `CustomerLastName` should be deleted, but it is a start.

## Database changes

I firmly believe databases should only store data.  The database should not contain business rules or business logic.  Only the code should store business rules and business logic.  When the database stores business rules or business logic, it makes blue/green deployments much harder.

Business logic in the database includes, but isn’t limited to, formatting, calculation, the inclusion of IsNull checks, if/then statements, while statements, default values, and filtering more than just by an ID.  Even if it is an empty string or zero, those are values.  If a column doesn’t have a value, it needs to be set to `Null`, which is the absence of a value.  Having business rules and business logic in the database is asking for trouble.  Compared to code, such as C#, JavaScript, or Java, they are much harder to write unit tests for, even when using a tool such as tSQLt.  They are also much harder for developers to find, as typically most developers search using their IDE of choice which excludes databases.

This section walks through recommendations for table changes, stored procedure, and view changes to support blue/green deployments.  It will also cover techniques on how to avoid having business rules and business logic in the database.

### Table Changes

You should make non-destructive database changes when doing blue/green deployments.  In our scenario, that means making the `CustomerFullName` nullable when it is added.  Making a column non-nullable without a default value would be a destructive change.  Insert statements on `Green` would stop working because it doesn’t know about that new column.  That doesn’t mean the column should be made non-nullable with a default value; even an empty string.  Remember, a default value is a business rule.

The other problem is that it takes quite a bit of time for most database servers (i.e., SQL Server or Oracle) to add a non-nullable column.  When a non-nullable column is added, the table definition is updated along with every record.  When a nullable column is added, only the table definition is updated.  If you must have a default value, then the script should add a column as nullable, update all the records, and then set the column to non-nullable. That script can be tuned to run surprisingly fast.

Some other examples of destructive database changes include reusing an existing column or renaming an existing column.  With blue/green deployments that will fail because the code in `Green` will throw errors or show inaccurate data.   

So far this section has covered adding the new column, `CustomerFullName` to the customer table.  What about the older `CustomerFirstName` and `CustomerLastName` columns?  In the scenario section, it essentially says to leave `CustomerFirstName` and `CustomerLastName` alone.  Don’t overwrite it with `Null` and don’t try to guess what those values will be.  In addition, once `Blue` goes live with these changes, any new records will never have a value for `CustomerFirstName` and `CustomerLastName`, which means `CustomerFirstName` and `CustomerLastName` should be made nullable.

At some point, the `CustomerFirstName` and `CustomerLastName` columns will be removed.  Blue-green deployments make that more complex as well.  Assume `CustomerFullName` has been deployed to `Blue`, and it is active.  Running a script to delete `CustomerFirstName` and `CustomerLastName` from all the tables, views, functions, and stored procedures will cause errors. The code running on `Blue` is using the `CustomerFirstName` and `CustomerLastName` fields to populate `CustomerFullName` when `CustomerFullName` is null.  

It will take multiple deployments to delete those columns from the database.  

1. The deployment adds `CustomerFullName` to the customer table.  `CustomerFirstName` and `CustomerLastName` are needed because the older code still references them.
2. Another deployment occurs and the code removes all references to `CustomerFirstName` and `CustomerLastName`.
3. Final deployment deletes `CustomerFirstName` and `CustomerLastName` from the database.

How can the `CustomerFirstName` and `CustomerLastName` columns get removed with that restriction in place?  To answer that, look at a typical standard deployment with an outage.

Those deployments don’t have to be deployed within days of each other.  I’ve seen several months go between each of those deployments.  

### Stored procedures and views

Imagine the application had two stored procedures:

- `usp_GetCustomerById`
- `usp_GetAllCustomers`

Fundamentally, they both get customers from the database.  The `CustomerFullName` is a new column, but it is null.  As seen earlier, the backfill script raised a lot of questions.  One option is to skip the backfill script completely, and have the stored procedures do a quick IsNull check:

```SQL
Select CustomerFirstName,
       CustomerLastName,
       IsNull(CustomerFullName, CustomerFirstName + ' ' + CustomerLastName) as CustomerFullName
from dbo.Customer
```

That only hides bad or missing data; it doesn’t solve missing data.  It is a one-liner and easy to copy-paste to other stored procedures.  In this example, there is only one other stored procedure, but if anybody ever adds a stored procedure, they’d need to remember to include that one-liner.  It’s also possible to forget to update the stored procedures when it comes time to remove `CustomerFirstName` and `CustomerLastName` from the database.  Imagine if a stored procedure had 50 columns and `CustomerFullName` had several dozen lines between it and `CustomerFirstName` and `CustomerLastName`.

Retrieved stored procedures and views should return the data as is, without any null checks or formatting:

```SQL
Select CustomerFirstName,
       CustomerLastName,
       CustomerFullName
from dbo.Customer
```

Create or update stored procedures is slightly different.  `Green` will be active while `Blue` is being verified.  `Green` has no concept of `CustomerFullName` which means it will call stored procedures without including that new column.  When `Blue` goes active, it will no longer be sending in `CustomerFirstName` and `CustomerLastName` fields to the stored procedures.  The stored procedures need to handle calls from either `Blue` or `Green`:

```SQL
ALTER procedure [dbo].[usp_UpdateCustomer] (
    @CustomerId int,
    @CustomerFirstName varchar(128) = null,
    @CustomerLastName varchar(128) = null,
    @CustomerFullName varchar(256) = null
)
Begin
    Update dbo.Customer
        set CustomerFirstName = @CustomerFirstName,
            CustomerLastName = @CustomerLastName,
            CustomerFullName = @CustomerFullName
    where CustomerId = @CustomerId
End
Go
```

It is possible to run an insert command without specifying the columns, but you shouldn’t do that.  Specify all the columns.  It only causes headaches down the line as the order of the columns in the insert statement have to match the order of the columns in the table.  If a column is added in the middle of the table (which happens!), the insert statements will start randomly failing or inserting data into the wrong column.  The insert stored procedure will be similar to the update stored procedure, all the parameters, in this case, have the default value set to `Null` to handle both `Blue` and `Green` calling it:

```SQL
ALTER procedure [dbo].[usp_InsertCustomer] (    
    @CustomerFirstName varchar(128) = null,
    @CustomerLastName varchar(128) = null,
    @CustomerFullName varchar(256) = null
)
Begin
    Insert into dbo.Customer (CustomerFirstName, CustomerLastName, CustomerFullName)
        value (@CustomerFirstName, @CustomerLastName, @CustomerFullName)


    select SCOPE_IDENTITY()  
End
Go  
```

Views are great at providing a layer of abstraction, and when written correctly, can give a nice performance boost.  However, I’ve seen too many that are written poorly and end up causing performance problems. To help resolve those performance problems, the *no business logic in the database* rule gets broken.  I recommend avoiding that as much as possible.  Just like stored procedures, return all three columns, `CustomerFirstName`, `CustomerLastName`, and `CustomerFullName`.

An everyday use case for views is to help data warehousing.  The abstraction layer views provide, make it easy for a process to come through and copy all the data over to a data warehouse that business intelligence teams can use to create their reports.  There isn’t a right automated way to make this change known to the data warehouse.  My recommendation is to notify them of the change as soon as possible.

:::warning
I realize moving business logic out of the database is quite a change.  If you do decide to make that switch, be pragmatic about it.  Don’t try to change everything at once.  The code and the database are working fine now.  This section is about changes to the database going forward.  I always try to leave the database and code in a better state than when I found it.  Meaning, if I make a change to a column and I come across a business rule in the database for that column, I do some analysis.  If the change is relatively easy to make, I will make it then.  If it’s complex, I will create a card and put it on our technical debt backlog to be fixed later (hopefully).
:::

## Code Changes

As stated in the database changes section, business rules and business logic should exist in the code.  The scenario defined a couple of business rules which need to be placed into the code.  

- `CustomerFullName` will be populated by combining `CustomerFirstName` and `CustomerLastName` when `CustomerFullName` is `Null`.
- If data exists in `CustomerFirstName` and `CustomerLastName` and an API request is sent with only `CustomerFullName` then do not set those columns to `Null`.  In addition, do not try to split up `CustomerFullName` as that is very difficult and error-prone.

The next section walks through the techniques for how to put those rules into code using C# as the example language.  

### Handling Null in CustomerFullName

It is very easy to put a check like this in a stored procedure, but as stated earlier, it’s not a good idea.

```SQL
IsNull(CustomerFullName, CustomerFirstName + ' ' + CustomerLastName) as CustomerFullName
```

One alternative option is to put that formatting rule in the model representing the customer table:

```C#
public class CustomerModel
{
    private string _fullName;

    public int CustomerId {get; set;}
    public string FirstName {get; set;}
    public string LastName {get; set;}
    public string FullName
    {
        get
        {
            if (string.IsNullOrDefault(_fullName))
            {
                return this.GetFormattedFullName();
            }

            return _fullName;
        }
        set
        {
            if (string.IsNullOrDefault(value))
            {
                _fullName = this.GetFormattedFullName();
            }
            else
            {
                _fullName = value;
            }
        }
    }

    public string GetFormattedFullName()
    {
        return $"{this.FirstName} {this.LastName}";
    }
}
```

That works fine if the same model is used by all layers, the UI, business, and database.  That takes a great deal of discipline as the models and the database have to be a one-to-one match, and that often isn’t the case.

Here’s an alternative to putting all the logic in the model:

```C#
public Interface ICustomer
{
    public int CustomerId {get;set;}
    public string FirstName {get;set;}
    public string LastName {get;set;}
    public string FullName {get;set;}
}

public Interface ICustomerDataAdapter
{
    void InsertCustomer(ICustomer customer);
    void UpdateCustomer(ICustomer customer);
    ICustomer GetCustomerById(int customerId);
}

public static class CustomerNameFormatter
{
    public string GetFullName(this ICustomer customer)
    {
        if (string.IsNullOrWhiteSpace(customer.FullName) == false)
        {
            return customer.FullName;
        }

        return $"{customer.FirstName} {customer.LastName}";        
    }
}

public class CustomerFacade
{
    private ICustomerDataAdapter _customerDataAdapter;

    public CustomerFacade(ICustomerDataAdapter customerDataAdapter)
    {
        _customerDataAdapter = customerDataAdapter
    }

    public void GetCustomerById(int customerId)
    {
        var customer = _customerDataAdapter.GetCustomerById(customerId);

        customer.FullName = customer.GetFullName();        

        _customerDataAdapter.UpdateCustomer(customer);
    }
}
```

Using that same formatter extension method, the logic to insert a customer record would look something like this:

```C#
public Interface ICustomer
{
    public int CustomerId {get;set;}
    public string FirstName {get;set;}
    public string LastName {get;set;}
    public string FullName {get;set;}
}

public Interface ICustomerDataAdapter
{
    void InsertCustomer(ICustomer customer);
    void UpdateCustomer(ICustomer customer);
    ICustomer GetCustomerById(int customerId);
}

public static class CustomerNameFormatter
{
    public string GetFullName(this ICustomer customer)
    {
        if (string.IsNullOrWhiteSpace(customer.FullName) == false)
        {
            return customer.FullName;
        }

        return $"{customer.FirstName} {customer.LastName}";        
    }
}

public class CustomerFacade
{
    private ICustomerDataAdapter _customerDataAdapter;

    public CustomerFacade(ICustomerDataAdapter customerDataAdapter)
    {
        _customerDataAdapter = customerDataAdapter
    }

    public void InsertCustomer(ICustomer customer)
    {
        var existingCustomer = _customerDataAdapter.GetCustomerById(customer.CustomerId);

        customer.FullName = customer.GetFullName();        

        _customerDataAdapter.InsertCustomer(customer);
    }
}
```

### Do not overwrite CustomerFirstName or CustomerLastName

After the changes on `Blue` go live, there will still be data in the `CustomerFirstName` and `CustomerLastName` columns, and it would be bad for the code to set that data to null.  With new records, that data won’t be present.  The code shouldn’t try to guess how to split up the first name and last name.  For existing records, the code should leave the data as-is.  For new records, it needs to set those columns to `Null`.  

Just like before, there’s a temptation to make the update statement check for null in the parameter (It is only one line, what is the harm?):

```SQL
    set CustomerFirstName = IsNull(@CustomerFirstName, CustomerFirstName)
```

The code for `Blue` should be where that null check lives.  Putting the `IsNull` check in the database hides a business rule, which is what we want to avoid:

```C#
public Interface ICustomer
{
    string FirstName {get;}
    string LastName {get;}
    string FullName {get;}
}

public Interface ICustomerDataAdapter
{
    void InsertCustomer(ICustomer customer);
    void UpdateCustomer(ICustomer customer);
    ICustomer GetCustomerById(int customerId);
}

public class CustomerFacade
{
    private ICustomerDataAdapter _customerDataAdapter;

    public CustomerFacade(ICustomerDataAdapter customerDataAdapter)
    {
        _customerDataAdapter = customerDataAdapter
    }

    public void UpdateCustomer(ICustomer customer)
    {
        var existingCustomer = _customerDataAdapter.GetCustomerById(customer.CustomerId);

        customer.FirstName = string.IsNullOrEmpty(customer.FirstName) ? existingCustomer?.FirstName : null;
        customer.LastName = string.IsNullOrEmpty(customer.LastName) ? existingCustomer?.LastName : null;

        _customerDataAdapter.UpdateCustomer(customer);
    }
}
```

## Backfill the new column with data

Most backfill scripts I’ve seen are nothing more than a SQL script to update the underlying data.  This means the formatting logic will exist in both the code and the backfill script.  It is very easy to update the code to update to the formatting rule but forget to update the backfill script.  I've had that happen to me.  Trust me, it makes for a bad day.

In addition, with blue/green deployments, you have to decide when to run the script:

- After the code has been deployed to `Blue` and the database changes have been pushed out but before verification starts?  
- Or after `Blue` goes live, and `Green` has become inactive?

You also have to consider:

- Traditional backfill SQL scripts update low-level data and could lock the table.  
- Depending on the number of records, the script could take a long time.  Maybe the decision is made to update 1000 records at a time in the SQL script.  
- What if users are updating the same record as the script and a deadlock occurs; is it okay if the script wins the deadlock, instead of the user update?
- Depending on when the script runs, it may need to have guard clauses in place to prevent an accidental update.

Also, most database deployment tools, Redgate, DBUp, RoundhousE, and SSDT don’t have a mechanism for running specific SQL scripts multiple times.  Without that built-in functionality, a hack needs to be put in place to support it.

The code has the formatting rules in place.  It has the necessary logic to handle a variety of scenarios.  It should hopefully have unit tests covering it.  This is what is verified by QA.  The backfill script should be written in PowerShell or Bash and invoke the API instead of updating the database directly.  It can query the database to find a list of customers and then use that list of customers to hit the API.  Logic can be added to the script to minimize the risk of both the script and user updating a record at the same time.  Maybe the user tied to a customer has a timezone preference, the script is run each hour via an automated process and only updates records when the user’s timezone is between 2 a.m. and 3 a.m.  

Below is an example backfill script written in PowerShell:

```PowerShell
$url = "https://myapp/api"
$apiKey = "Some Key"
$header = @{ "apiKey" = $apiKey }
$connectionString = "Server=MyServer;Database=ApplicationDatabase;integrated security=true;"

$sqlConnection = New-Object System.Data.SqlClient.SqlConnection
$sqlConnection.ConnectionString = $connectionString

$command = $sqlConnection.CreateCommand()
$command.CommandType = [System.Data.CommandType]'Text'
$command.CommandText = "Select CustomerId
        from Customer
        where IsNull(CustomerFullName, '') = ''
            and (IsNull(CustomerFirstName, '') <> '' or IsNull(CustomerLastName, '') <>'')"            

$sqlConnection.Open()

$dataAdapter = new-object System.Data.SqlClient.SqlDataAdapter $SqlCommand
$dataSet = new-object System.Data.DataSet
$dataAdapter.Fill($dataSet)

$sqlConnection.Close()

$customerTable = $dataSet.Tables[0]

foreach($customerRow in $customerTable)
{
    $customerId = $customerRow["CustomerId"]

    Write-Host "Getting customer $customerId"
    $customer = (Invoke-RestMethod "$url/customers/$customerId" -Headers $header)

    Write-Host "Updating customer $customerId"
    $updateCustomerResult = (Invoke-RestMethod "$url/customer/$customerId" -Headers $header -Method Post -Body $customer -ContentType "application/json")
}
```

The script is non-destructive, and it can be run at any time, so it doesn’t need to occur during deployment.  Having it run outside of deployment means it isn’t holding anything up.  The script can be kicked off, and it can plug away for as long as it needs to.  All the timing concerns are taken care of.  There is no need to worry about running the backfill script multiple times to make sure a record isn’t somehow missed. There’s also no need to worry about how long it will take to finish running.  It will help make the transition from `Green` to `Blue` as seamless as possible.

## Versioning stored procedures, views, and APIs

The typical rule of thumb is:

> If you make a breaking change, then version the API/Stored Procedure/View.

On paper, that's a good rule to follow.  In practice, that rule can fall apart quickly.  Versioning puts a significant burden on the people maintaining the code.  The maintainers know there are multiple code paths or multiple instances they need to worry about.  The longer an older version sticks around, the harder it will be to move to the new version.  I’ve seen companies spend hundreds of hours on projects to get people off old versions.  

My recommendation is that the default position should be to make changes as backward compatible as possible.  Have a single code base, single stored procedure, single view, which everyone uses.  A single code base will make maintenance easier, and it will make it easier for changes to be made (over time).  Look at multiple deployments to make small changes over time compared to a big bang.  Explore all the options before jumping into the versioning pool.  Versioning should be considered after all other options are exhausted.  

## When database changes should be deployed

Taking everything from this post into account for the test scenario; I’d argue the database changes are not required to be deployed at the same time as the code.  The only requirement is the database changes have to be deployed before the code.  Deploying a database change days or even weeks before deploying the code might have an added benefit.  If another application is using the same database (even just for a view), this will give them time to change and test their code.  Once the database change is up there, the other teams can deploy when they’re ready.  

The real question is, does it make sense to deploy the database changes days or even a week before the code?  That is a bit trickier to answer.  My recommendation is to do what makes sense dependent upon the scenario, but the fewer changes made during a deployment, the better.  Generally, fewer changes mean less risk.  Pushing database changes to production before the code includes risks as well.  Once development, testing, or UAT starts, additional database changes might be required.  If the first change makes it to production, other changes will require another push to production.  

My preference is to store the code and database in the same repository.  Get everything working in a feature branch.  Merge all the changes in the feature branch at the same time and test and verify at the same time.  

What I do like about blue/green deployments is the flexibility it gives for database deployments.  Before blue/green deployments, everything had to go out at the same time during the deployment.  Now there is a choice.  

## Testing and verification

Automated testing and verification of `Blue` before swapping with `Green` (and vice versa) makes everything go a lot faster.  Automated testing and verification isn’t a requirement for blue/green deployments, and deployment tools like Octopus Deploy have the manual intervention step which can pause a deployment and wait until someone gives the go-ahead to proceed.  

Most people starting blue/green deployments will not have a full test suite they can run in Production on day one.  The key is to start somewhere.  It will take time to build out that test suite and overcome technical hurdles.  Depending on the tooling, even a simple scenario, changing from the URL for `Blue` with `Green` could take a bit to figure out.  As tests come online include them in the deployment pipeline to help automate the verification.  Hopefully, the time will come when the vast majority of use cases can be verified, and the manual intervention step can be removed.

## Common database change scenarios

This post covered a very complex scenario, the combination of two columns into one.  There are several other database change scenarios detailed in the list below.  For each scenario, I’ve included which sections could be applied to the scenario.

- **Adding a new column**: Follow all the steps except deleting old columns and handling legacy columns.
- **Renaming a column**: Don’t rename. Follow the steps above to add a new column and remove the old column.
- **Adding a new table**: Similar to adding a new column.
- **Renaming a table**: Don’t rename.  Follow the steps above to add a new column and delete the old columns.
- **Moving a column to another table**: Very similar to adding a new column and removing the old column.  The only difference is where the column was added.
- **Deleting a table**: Very similar to deleting a column.  Hopefully, that table isn’t used anymore, and all the data has been migrated to other tables.
- **Adding a new view/stored procedure**: Very similar to adding a new column.  The updated code will use the new view/stored procedure; the old code will not use the view/stored procedure.
- **Updating an existing view/stored procedure**: As long as columns aren’t removed from a view, this should be fine.  If columns are removed, then the process to delete columns from above should be followed.
- **Removing a view/stored procedure**: Very similar process to removing columns.  Except there won’t be data, just references in code and potentially other stored procedures.

## Wrapping-up

As you can see, blue/green deployments with a database require a few minor shifts in how you write code and make database changes.  It also requires more planning for how a change is implemented than doing a standard deployment with an outage.

The first stab at a blue/green deployment process looked like this:

1. Run the script to add the column `CustomerFullName`.
2. Run the backfill script to populate `CustomerFullName`.
3. Deploy code to the `Blue` environment.
4. Verify the code and database changes in the `Blue` environment.
5. Swap the live environment from `Green` to `Blue`.
6. Run the backfill script to populate `CustomerFullName`.

That process has now evolved into this:

1. Run the script to add the column `CustomerFullName`.
2. Deploy code to `Blue` environment.
3. Verify the code and database changes in the `Blue` environment.
4. Swap the live environment from `Green` to `Blue`.

The backfill script is not included at all; it is run after the deployment to help populate the new `CustomerFullName` column.  In fact, that script doesn’t need to run until the code needs to be changed to remove the `CustomerFirstName` and `CustomerLastName` columns.  

Before ending this post, I do want to point out blue/green deployments are not for every application.  It requires the right alignment of architecture, infrastructure, and tooling.  If you are considering blue/green deployments, take a feature with a database change and walk through what it would take to implement that using a blue/green deployment strategy.  Do you have to change anything in your application’s architecture?  Is the necessary infrastructure, such as a load balancer and additional VMs, in place?  Can your CI/CD tool support blue/green deployments?

In terms of time, the initial cost to do blue/green deployments can be high.  That cost is worth it when it’s possible to deploy a change in the middle of the day.  Being able to do that opens up so many different possibilities.  Soon the conversation will move from “how can I deploy in the middle of the day” to “with blue/green deployments in place, now what can I do?”  Blue-green Deployments, or seamless daytime deployments, feels like it is the end of the CI/CD journey.  I’d argue that it is just the beginning.

Until next time, Happy Deployments!
