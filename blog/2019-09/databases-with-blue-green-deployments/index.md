---
title: Automated Database Blue/Green Deployments 
description: Learn some techniques on how to automate database deployments when using a Blue/Green deployment strategy.
author: bob.walker@octopus.com
visibility: public
metaImage: 
bannerImage: 
published: 2019-09-30
tags:
 - Database Deployments
---

The worst time to deploy software is 2:00 AM on Saturday.  It ruins Friday night, can't do anything fun because I have to go to bed early.  And it ruins Saturday day because I am so tired from waking up in the middle of the night.  I learned that fact by working on an application where that deployment time was mandated.  Fast deployments for that application took two hours; average deployments took long enough to see the sunrise.  What frustrated me was a load balancer was in front of the VMs hosting the application.  I wanted deploy to a VMs not in the load balancer during the middle of the day, verify the code when we are not exhausted, and at night run a quick script to change the load balancer. In other words, why weren't we doing Blue/Green deployments?  Simple, it was the database which tripped everything up.  This was back in 2010/2011 and it was for a ASP.NET Webforms application.  The .NET stack, and my experience has come a long way since then.  It is possible to do Blue/Green deployments with a database such as SQL Server, Oracle, MySql, or PostgreSQL.  In this post, I will walk through some techniques on how to do that.  

!toc

## A Brief Intro to Blue/Green Deployments

Before diving into the weeds, a brief intro on Blue/Green deployments.  Blue/Green deployments are when there are two identical production environments labeled `Blue` and `Green`.  At any given time, only one of those environments, for example, `Blue`, is live.  Deployment is done to the non-live environment, for example, `Green`, which is then verified.  After verification is complete, a switchover occurs, and the live environment becomes `Green`.  

![](https://i.octopus.com/docs/deployment-patterns/blue-green-deployments/images/3278250.png)

There are several advantages to doing this.  Rollbacks should be easy, only a matter of switching from blue to green or green to blue.  When a switchover does occur, it should be seamless as the code has already been running, there is no need to wait for it to compile or "warm-up."  And changes can be verified in production without having any customers hitting the code, making the deployment much less risky.  If something doesn't work, don't make the switch, try again at a later time. 

Before diving into too much farther into the article, I should point out not all applications can take advantage of Blue/Green deployments.  It is a combination of architecture, how stateful the application is, and the technology used.  The more stateless and decoupled the application is, the better chance it can be deployed using the Blue/Green deployment strategy.  A .NET Core Web API with a Angular front-end is suited much better for Blue/Green deployments than a ASP.NET WebForms application without a business logic or data layer and requires the sticky sessions from the load balancer.

## The Scenario

This post will be covering a complex scenario.  A new column `CustomerFullName` is added, which is a combination of `CustomerFirstName` and `CustomerLastName` columns.  In the UI, there used to be two fields, `First Name` and `Last Name`, but the decision was made to combine the two columns.  Some cultures have multiple names which do not fit into the `First Name` and `Last Name` commonly seen in the US.  A script was written to backfill `CustomerFullName` with `CustomerFirstName` and `CustomerLastName` during the deployment.  The `CustomerFirstName` and `CustomerLastName` columns will no longer be needed in the database; an additional script will be written to delete them.

## How Database Changes Add Complexity

The introduction walked through an after-hours deployment.  They are done to avoid users accessing the application during the deployment.  Specifically, preventing people from accessing the application while the backfill script is running.

A very simplified deployment process for that change would look like this:

1. Disable access to users or turn off the website.
2. Run the script to add the column `CustomerFullName`.
3. Run the backfill script to populate `CustomerFullName`.
4. Run the script to remove `CustomerFirstName` and `CustomerLastName`.
5. Deploy the code to `Production`.
6. Verify the code in `Production`.
7. Profit.

In this example, `Green` is live, and `Blue` is inactive.  Without Blue/Green deployments, the only worry was someone is using the application before everything is deployed and verified.  If the user gets an error indicating the column `CustomerFirstName` is missing during the deployment, no worries, they shouldn't be in the system at that point anyhow.  With Blue/Green deployments that restriction is flipped on its head; users are going to be running on the `Green` servers during deployment where data is getting queried and manipulated.  Users using the application during the deployment adds a whole slew of items to consider.

- To deploy to `Blue` and verify the changes `CustomerFullName` needs to be added to the database.  Verification isn't complete unless the backfill script populating the column is tested.  So the schema change script and backfill script need to be run before deploying to `Blue.`  
- Customers will be using the application on `Green` the `CustomerFirstName`, and `CustomerLastName` columns cannot be deleted during this deployment.  Doing so will cause errors.  Later deployments will handle the deletion of those columns.  
- How long will the deployment and verification take?  During that time, how many new records will be added, or existing ones are updated?  The backfill script will have to be run again after the environments are swapped to pick up any new changes.  
- Can the backfill script run a second time and will it take the same amount of time?  Is it okay if the Customer Name is empty or blank during that time?  
- Is the application a SPA App ([Single Page Applications](https://en.wikipedia.org/wiki/Single-page_application))?  Will `Blue` start getting API requests from `Green` front-end code?  Some users could be sending API requests without the `CustomerFullName` column, just the `CustomerFirstName` and `CustomerLastName` column.  The code being deployed to `Blue` will need to take that into consideration and be able to handle that.
- Are there any stored procedures, specifically around inserting/updating data which need to be updated?  Will `Green` fail because the `CustomerFirstName` parameter will not be sent in?    

The deployment process for this change would look something like this:

1. Run the script to add the column `CustomerFullName`.
2. Run the backfill script to populate `CustomerFullName`.
3. Deploy code to `Blue` environment.
4. Verify the code and database changes in the `Blue` environment.
5. Swap the live environment from `Green` to `Blue`.
6. Run the backfill script to populate `CustomerFullName`.

There are a lot of questions when users are allowed to use the application during the deployment.

Admittedly, that scenario is fairly complex.  Most database changes don't combine two columns into one and try to backfill the new column.  If that can be solved, then I would argue the majority of the scenarios are solved, which is why this section will focus on how to solve the scenario of combining two columns into one.  It will walk through each change, the questions to consider, and recommendations on how to solve them.

**Please Note:** These are recommendations only.  There is no way I can cover every possible change you can make to a database.  My goal is to provide you with something which you can then modify to meet your own needs.  

## Adding the New Columns

Keep it simple; add the new column `CustomerFullName` as a nullable column.  It takes most databases (SQL Server, Oracle) quite a bit of time to add a new non-nullable column with a default value.  This is because it has to update each record on the table.  Adding a nullable column requires updating the table definition (metadata) which takes a few milliseconds.  

Avoid the following if at all possible:

- Rename an existing column, for example, `CustomerLastName` to `CustomerFullName` for one reason or another.  Don't know why this was ever proposed, but I've seen it happen and I've seen it fail because running the rename sproc for a column doesn't rename the column in the stored procedures or code.  This will cause `Green` to start throwing errors when doing Blue/Green deployments.
- Reuse an existing column, for example, `CustomerFirstName` now stores the full customer name.  That makes writing queries very confusing.  And while `Green` is active, `CustomerFirstName` will only contain the first name.  Once `Blue` goes active, `CustomerFirstName` will start containing the full name.  It seems like it will save time; the code is already there to use that data; it is only a UI update.  That shortcut only causes confusion in the end.
- Add the column, backfill with data, then change the column to non-nullable.  It should work, except, `Green` isn't sending in data for that column.  Which leads to an error.  `Null` is the absence of a value.  It is not a bad thing.  

## Backfill the new column with data

With Blue/Green deployments, the script to backfill the new column had to be run twice in our scenario.  Even then, there is the risk of the second run taking a while or getting missing data from API requests.  There are too many what-ifs.  It is easy to go a little overboard thinking of all the possible scenarios.  This is why I recommend taking the backfill script out of the equation.  In other words, but don't write scripts to backfill `CustomerFullName`.  Have the code backfill the data over time as well as contain the logic to handle when no data is found.

//TODO: Reword this a bit, because we can do a backfill script, but it is done through the API not SQL

There is not a set in stone solution on how to have the code accomplish backfilling of the `CustomerFullName` column.  Every application has its own rules about patterns and practices to follow.  I am going to suggest a couple of approaches, but use them as guidelines.  Follow your application's patterns and practices.  It is better to have consistency in the code than it is to do something which you feel is correct "correct."  

Some example code for this would be:

```C#

public class CustomerModel 
{
    private string _fullName;

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

That works fine if all the properties are on the same model.  And that is the same model being returned by the data layer and sent as a response in an API call along with accepting the same model from the API.  In my experience, it takes a great deal of discipline as the models, and the database has to be a one to one match.  In my experience, that is rarely the case.  I've seen ApiModels, ViewModels, or some other phrase which indicates there is a difference in the model and the database.  

An alternative to putting all the logic in the model would be:

```C#
public Interface ICustomer
{
    public string FirstName {get;}
    public string LastName {get;}
    public string FullName {get;}
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

public class Customer : ICustomer 
{
    public int CustomerId {get; set;}
    public string FirstName {get; set;}
    public string LastName {get; set;}
    public string FullName {get; set;}    
}

public class CustomerDataAdapter
{
    private _connectionString;

    public CustomerDataAdapter (string connectionString)
    {
        _connectionString = connectionString;
    }

    public void InsertCustomer(ICustomer customer)
    {
        using (var conn = new SQLConnection(_connectionString))
        {            
            using (var command = new SQLCommand("dbo.usp_InsertCustomer", conn))
            {
                command.Parameters.AddWithValue("@CustomerFirstName", customer.FirstName);
                command.Parameters.AddWithValue("@CustomerLastName", customer.LastName);
                command.Parameters.AddWithValue("@CustomerFullName", customer.GetFullName());
            }                
        }
    }
}

```

I recommend this approach for a number of reasons.

1. All the timing concerns are taken care of.  There is no need to worry about running the backfill script multiple times to make sure a record isn't missed somehow.  No need to worry about how long it will take to finish running.  It will help make the transition from `Green` to `Blue` be as seamless as possible.
2. This change needs to happen anyway to prevent errors from happening.  While `Blue` is being deployed and verified, users will be making database updates on `Green` after the first run of the backfill script.  If the person or automated process doing the verification on `Blue` happens to hit one of those records and the code wasn't updated, then an error would occur. 
3. The code also needs to change in the event the application somehow gets a request without the `CustomerFullName` property populated.  Could come from a SPA application or for some other reason.  With or without a backfill script, setting that column in the database to `Null` after it has been populated would not be great.  But if it did happen the code would be able to handle it.  
4. Backfill scripts are SQL Scripts.  SQL is a great language for querying databases.  It is not a great language to write functional programming, integration tests, or unit tests.  The code should be the only place where the logic of backfilling the `CustomerFullName` column is located.  Having a backfill script AND the code contains that logic opens the door to errors.  Any change in rules will require an update to both the code and the script.
5. Most database deployment tools, be it Redgate, DBUp, RoundhousE, or SSDT don't have a great mechanism for running specific scripts multiple times.  If the tooling does have that functionality, or a hack is put into place to implement that functionality, then guard clauses will have to be put in place around the script to make sure data isn't overwritten by accident.

## Stored Procedures and Views

I am a big believer in the database should be used as a data store only.  Keep as much logic as possible out of the database.  This includes stored procedures and views.  Business logic in the database includes, but not limited to, formatting, calculation, the inclusion of IsNull checks, if/then statements, while statements, and filtering more than just by an Id.  

It is far too easy to end up copying/pasting that logic between stored procedures.  Imagine the application had two stored procedures:

- usp_GetCustomerById
- usp_GetAllCustomers

Fundamentally they do the same thing, get customers from the database.  The CustomerFullName is a new column, but it is null.  Rather than having a function in the code check to see if `CustomerFullName` is null, that logic is placed in the database.

```SQL
Select CustomerFirstName,
       CustomerLastName,
       IsNull(CustomerFullName, CustomerFirstName + ' ' + CustomerLastName) as CustomerFullName
from dbo.Customer
```

That only hides bad or missing data; it doesn't solve bad or missing data.  It is a one-liner, and oh so easy to copy-paste to other stored procedures.  In this example, it is only one other stored procedure.  Then one day someone adds a new stored procedure, and they forget to include that.  But, it has been long enough the `CustomerFullName` column is populated for 90% of the records.  It is only a matter of time before an error occurs.  And when it comes time to remove `CustomerFirstName` and `CustomerLastName` from the database they could miss that IsNullCheck.  Imagine if a stored procedure had 50 columns.  `CustomerFullName` was added towards the bottom of the list while `CustomerFirstName` and `CustomerLastName` were towards the top of the list. 

Retrieve stored procedures and views should return the data as is, without any null checks or formatting.  

```SQL
Select CustomerFirstName,
       CustomerLastName,
       CustomerFullName
from dbo.Customer
```

Create or update stored procedures are slightly different.  `Green` will be active while `Blue` is being verified.  `Green` has no concept of `CustomerFullName` which means it will call stored procedures without including that new column.  Just like adding the column to the table as nullable, the parameter for `CustomerFullName` should be nullable.  In the code section above, 

```SQL
ALTER procedure [dbo].[usp_UpdateCustomer] (
    @CustomerId int,
    @CustomerFirstName varchar(128),
    @CustomerLastName varchar(128),
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

While it is possible to run an insert command without specifying the columns, don't do that.  Specify all the columns.  It only causes headaches down the line.

```SQL
Insert into dbo.Customer (CustomerFirstName, CustomerLastName, CustomerFullName)
    value (@CustomerFirstName, @CustomerLastName, @CustomerFullName)


select SCOPE_IDENTITY()  
```

I've worked on applications where views were created for other teams, such as Business Intelligence (BI) teams, to suck in data from the application's database into their data warehouse databases.  The views were created because the other teams are not experts on the application; they don't know the schema, and their import process shouldn't be tied directly to the schema.  Views provide a layer of abstraction for them.  This is when I see the "no business logic in databases" rule gets broken.  The data needs to be presented in such a way to make it easy to import into a data warehouse.  For example, when I was working on a loan origination system, the BI team said they wanted the view to show "loans with their customers."  A loan could have 1 to N number of customers, so I did an inner join on the loan and customers table and called it good.  The BI team then asked why they kept getting duplicate loans.  What they really wanted was "loans with the primary customer," which required me to add `IsPrimary = 1` as a where clause into the view.  

My advice is still to provide a view which gives the data in its rawest form as possible.  In my example for the loan origination system, I should've added the column `IsPrimary` to the view and let the BI team filter out what they didn't want.  For the `CustomerFullName` example, it would be all three columns, `CustomerFirstName`, `CustomerLastName`, and `CustomerFullName`.  Communicate to them and let them know `CustomerFirstName` and `CustomerLastName` is going to be dropped, now is the time to start updating their reports and cubes and all that other fun stuff they create.  

**Please Note:** I realize moving business logic out of the database is quite the change.  If you do decide to make that switch, be pragmatic about it.  Don't try to change everything at once.  The code and the database is working fine now.  This section is about changes to the database going forward.  My rule of thumb is to leave the database and code in a better place than when I found it.  Meaning, if I am making a change to a column and I come across a business rule in the database for that column, then I fix it at that time.  I don't tear through all the code and start making mass changes.  That is a surefire way to end up doing an emergency deployment on the weekend.

## Handling Legacy Columns

`CustomerFullName` is a combination of `CustomerFirstName` and `CustomerLastName`, which means at some point `CustomerFirstName` and `CustomerLastName` will be deleted.  Those two columns won't be deleted right away.  While `Blue` is being verified, `Green` will be saving data to them.  Once `Blue` goes live, it will be getting the `CustomerFullName` field from the UI, it will no longer get `CustomerFirstName` or `CustomerLastName`.  The application shouldn't set those columns to `Null` or an empty string when it previously had a value.  On the flip side of that, the code shouldn't attempt to populate those columns on an insert of a new record.  It never had the data, and it never will.

First, set the `CustomerFirstName` and `CustomerLastName` columns to nullable in the database if they currently are set to non-nullable. `Null` is the absence of a value.  The temptation is there to set those columns to an empty string in the code.  An empty string is a value.  The code will no longer get a value from the UI.  The database should reflect that.

Next, if there are insert stored procedures and update stored procedures, then change the parameters for `CustomerFirstName` and `CustomerLastName` to have a default value of null.  This will handle the cases where the code no longer sends in a value.

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

Finally, the code which updates the customer record will need to be changed to first pull in the existing customer record.  When the UI sends in `Null` for  `FirstName` and `LastName`, and the database has data in those columns then use the value from the database.

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

## Removing Columns

At some point, the `CustomerFirstName` and `CustomerLastName` columns will be removed.  Blue/Green deployments make that more complex as well.  Assume `CustomerFullName` has been deployed to `Blue`, and it is active.  Running a script to delete `CustomerFirstName` and `CustomerLastName` from all the tables, views, functions, and stored procedures will cause errors.   The code running on `Blue` is using the `CustomerFirstName` and `CustomerLastName` fields to populate `CustomerFullName` when `CustomerFullName` is null.  

How can the `CustomerFirstName` and `CustomerLastName` columns get removed with that restriction in place?  To answer that, consider what a deployment would look like if an outage was taken.

1. Run a backfill script to take care of the remaining records.
2. Remove references in the code to `CustomerFirstName` and `CustomerLastName`.
3. Delete `CustomerFirstName` and `CustomerLastName` 

Each of those steps will be a separate deployment.  The first step, run a backfill script, seems to counter my previous thought, which is don't write a backfill script.  My main complaint about typical backfill scripts remain true; they are typically written in SQL, which means the script has to duplicate logic from C#.  The trend towards making all APIs RESTful eliminates that concern because the script can invoke the API.  The only time a script has to touch the database is to find a list of customers who have the `CustomerFullName` set to `Null`.

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

That script is invoking code which is currently running in production.  It is most likely getting hit thousands of times per day.  And there are hopefully automated tests around it as well.  It will take much longer to run the script than a simple SQL update script.  But it is nondestructive, and it can be run at any time.  Because it can be run at any time, it doesn't need to occur during deployment.  Having it run outside of deployment means it isn't holding anything, it can be kicked off, and it can plug away for as long as it needs to.  

While the script is running a branch can be created to remove references to `CustomerFirstName` and `CustomerLastName` in the code.  Once the script finishes running and the code is tested by the appropriate people it can be deployed to production.  

After the code has been deployed and has been running for a while, then it is finally time to write the script to delete `CustomerFistName` and `CustomerLastName` from the database.  After that has been tested by the appropriate people, that script can go out on the next deployment to production. 

## Common Database Change Scenarios

This post covered a very complex scenario, the combination of two columns into one.  There are several other database change scenarios detailed in the list below.  I've included which of the above sections could be applied to the scenario.

- Add a new column -> follow all the steps except deleting old columns and handling legacy columns
- Renaming a column -> Don't rename.  Follow the steps above to add a new column and delete the old column.
- Adding a new table -> Similar to adding a new column
- Renaming a table -> Don't rename.  Follow the steps above to add a new column and delete the old columns.
- Moving a column to another table -> Very similar to adding a new column and deleting the old column.  The only difference is where the column was added
- Deleting a table -> Very similar to deleting a column.  Hopefully, that table isn't used anymore, and all the data has been migrated to other tables.
- Adding a new view/stored procedure -> Very similar to adding a new column.  The updated code will use the new view/stored procedure; the old code will not use the view/stored procedure. 
- Updating an existing view/stored procedure -> As long as columns aren't removed from a view, this should be fine.  If columns are removed, then the process to delete columns from above should be followed.
- Removing a view/stored procedure -> Very similar process to removing columns.  Except there won't be data, just references in code and potentially other stored procedures.

## Multiple Applications connecting to a single database

I've worked on an application which was one of dozen or so which connected to the same database.  The application list was split amongst three or four teams.  To keep things somewhat organized all common tables and stored procedures were in the `dbo` schema, while tables and stored procedures only used by a single application where in an application schema.  That reduces possible database changes to add only to the `dbo` schema, while having a bit more flexibility in the application schema.  

## Wrapping Up

If you takeaway anything from this post it is this: Blue/Green deployments with a database will require more planning on how a change is implemented than doing a standard deployment with an outage.  It is a lot like playing chess, where you will have to think several steps ahead.  Blue/Green deployments are not for every application.  It requires the right alignment of architecture, infrastructure and tooling.  If you are considering Blue/Green deployments, take a feature with a database change and walk through what it would take to implement that using a Blue/Green deployment strategy.  Do you have to change anything in your application's architecture?  Is the necessary infrastructure, such as a load balancer and additional VMs, in place?  Can your CI/CD tool support blue/green deployments?

