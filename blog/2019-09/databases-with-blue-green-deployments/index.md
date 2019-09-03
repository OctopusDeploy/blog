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
 - Blue/Green Deployments
---

The worst time to deploy software is 2:00 AM on Saturday.  It ruins Friday night, can't do anything fun because I have to go to bed early.  And it ruins Saturday day because I am so tired from waking up in the middle of the night.  I learned that fact by working on an application where that deployment time was mandated.  Fast deployments for that application took two hours; average deployments took long enough to see the sunrise.  What frustrated me was a load balancer was in front of the VMs hosting the application.  I wanted to deploy to VMs not in the load balancer during the middle of the day.  After the deployment was complete, verify the code.  Do this work when people are not exhausted. Once everything looks good, then later in the night run a quick script to change the load balancer. In other words, why weren't we doing Blue/Green deployments?  Simple, it was the database which tripped everything up.  I was doing 2:00 AM Saturday deployments back in 2010/2011, and it was for an ASP.NET Webforms application.  The .NET stack and my experience have come a long way since then.  It is possible to do Blue/Green deployments with a database such as SQL Server, Oracle, MySql, or PostgreSQL.  In this post, I will walk through some techniques on how to do that.  

!toc

## A Brief Intro to Blue/Green Deployments

Before diving into the weeds, a brief intro on Blue/Green deployments.  Blue/Green deployments are when there are two identical production environments labeled `Blue` and `Green`.  At any given time, only one of those environments, for example, `Blue`, is live.  Deployment is done to the non-live environment, for example, `Green`, which is then verified.  After verification is complete, a switchover occurs, and the live environment becomes `Green`.  

![](https://i.octopus.com/docs/deployment-patterns/blue-green-deployments/images/3278250.png)

There are several advantages to doing this.  Rollbacks should be easy, only a matter of switching from blue to green or green to blue.  When a switchover does occur, it should be seamless as the code has already been running, there is no need to wait for it to compile or "warm-up."  And changes can be verified in production without having any customers hitting the code, making the deployment much less risky.  If something doesn't work, don't make the switch, try again at a later time. 

Before diving into too much farther into the article, I should point out not all applications can take advantage of Blue/Green deployments.  It is a combination of architecture, how stateful the app is, and the technology used.  The more stateless and decoupled the application is, the better the chance it can be deployed using the Blue/Green deployment strategy.  A .NET Core Web API with an Angular front-end is suited much better for Blue/Green deployments than an ASP.NET WebForms application without a business logic or data layer and requires the sticky sessions from the load balancer.

## The Scenario

This post will be covering a complex scenario.  A new column `CustomerFullName` is added, which is a combination of `CustomerFirstName` and `CustomerLastName` columns.  In the UI, there used to be two fields, `First Name` and `Last Name`, but the decision was made to combine the two columns.  Some cultures have multiple names which do not fit into the `First Name` and `Last Name` commonly seen in the US.  A script was written to backfill `CustomerFullName` with `CustomerFirstName` and `CustomerLastName` during the deployment.  The `CustomerFirstName` and `CustomerLastName` columns will no longer be needed in the database; an additional script will be written to delete them.

For this entire article, when talking about deployments, `Green` is currently live, while `Blue` is inactive.  All deployments will go to `Blue` and once verified, will be with `Green` to become live.    

## How Database Changes Add Complexity

After-hours deployments are done to avoid users accessing the application during the deployment.  Specifically, preventing people from accessing the application while the backfill script is running.

A very simplified deployment process for that change would look like this:

1. Disable access to users or turn off the website.
2. Run the script to add the column `CustomerFullName`.
3. Run the backfill script to populate `CustomerFullName`.
4. Run the script to remove `CustomerFirstName` and `CustomerLastName`.
5. Deploy the code to `Production`.
6. Verify the code in `Production`.
7. Profit.

As stated before, `Green` is live, and `Blue` is inactive.  Without Blue/Green deployments, the only worry was someone is using the application before everything is deployed and verified.  If the user gets an error indicating the column `CustomerFirstName` is missing during the deployment, no worries, they shouldn't be in the system at that point anyhow.  That is not the case with Blue/Green deployments.  Users will be in the application; they will be running on the `Green` servers.  That means data is going to get manipulated and queried.  Users using the app during the deployment adds a whole slew of items to consider.

- To deploy to `Blue` and verify the changes `CustomerFullName` needs to be added to the database.  Verification isn't complete unless the backfill script populating the column is tested.  So the schema change script and backfill script need to be run before deploying to `Blue.`  
- Customers will be using the application on `Green` the `CustomerFirstName`, and `CustomerLastName` columns cannot be deleted during this deployment.  Doing so will cause errors.  Later deployments will handle the deletion of those columns.  
- How long will the deployment and verification take?  During that time, how many new records will be added, or existing ones are updated?  The backfill script will have to be rerun after the environments are swapped to pick up any further changes.  
- Can the backfill script run a second time and will it take the same amount of time?  Is it okay if the Customer Name is empty or blank during that time?  
- Is the application a SPA App ([Single Page Applications](https://en.wikipedia.org/wiki/Single-page_application))?  Will `Blue` start getting API requests from `Green` front-end code?  Some users could be sending API requests without the `CustomerFullName` column, just the `CustomerFirstName` and `CustomerLastName` column.  The change being deployed to `Blue` will need to consider that and be able to handle that.
- Are there any stored procedures, specifically around inserting/updating data which need to be updated?  Will `Green` fail because the `CustomerFirstName` parameter will not be sent in?    

The deployment process for this change would look something like this:

1. Run the script to add the column `CustomerFullName`.
2. Run the backfill script to populate `CustomerFullName`.
3. Deploy code to `Blue` environment.
4. Verify the code and database changes in the `Blue` environment.
5. Swap the live environment from `Green` to `Blue`.
6. Run the backfill script to populate `CustomerFullName`.

There are a lot of questions when users are allowed to use the application during the deployment.

Admittedly, that scenario is relatively complex.  Most database changes don't combine two columns into one and try to backfill the new column.  This article will walk through how to solve that scenario.   That scenario touches on a lot of different changes which must be done for Blue/Green deployments.  If that scenario can be solved, then the majority of other scenarios can be solved.  It will walk through each change, the questions to consider, and recommendations on how to solve them.

**Please Note:** These are recommendations only.  There is no way I can cover every possible change you can make to a database.  My goal is to provide you with something which you can then modify to meet your own needs.  

## Adding the New Column

Keep it simple; add the new column `CustomerFullName` as a nullable column.  It takes most databases (SQL Server, Oracle) quite a bit of time to add a new non-nullable column with a default value.  That is because the database has to update each record on the table.  Adding a nullable column requires updating the table definition (metadata) which takes a few milliseconds.  Typically, adding a column as nullable, setting a value, and then setting as non-nullable is faster than adding a new column with a default value.  

Avoid default values and keeping new columns as nullable.  Default values are hidden business rules in the database.  Hard to test, harder to find.

Avoid the following if at all possible:

- Rename an existing column, for example, `CustomerLastName` to `CustomerFullName` for one reason or another.  Renaming a column will cause a lot of problems.  The rename stored procedure doesn't rename the column in the stored procedures or code.  Also, `Green` expects the column to be there, and when it is not, it will start throwing errors when doing Blue/Green deployments.
- Reuse an existing column, for example, `CustomerFirstName` now stores the full customer name.  That makes writing queries very confusing.  And while `Green` is active, `CustomerFirstName` will only contain the first name.  Once `Blue` goes active, `CustomerFirstName` will have the full name.  It seems like it will save time; the code is already there to use that data; it is only a UI update.  That shortcut only confuses the end.
- Add the column, backfill with data, then change the column to non-nullable.  It should work, except, `Green` isn't sending in data for that column.  Which leads to an error.  `Null` is the absence of a value.  It is not a bad thing.  

## Backfill the new column with data

Most backfill scripts I've seen are nothing more than a SQL Script to update the underlying data.  With Blue/Green deployments, that script has to be run at least twice, before verification and after the swap from `Green` to `Blue`.  That second run is probably riskier and will require quite a bit of testing.  Users could be adding records with only the `CustomerFullName` column being populated.  And what if users are updating the same record as the script?  Who wins?

There are too many what-if scenarios and pitfalls.  That is why I recommend waiting to run the backfill or migration script until after the swap to `Green` to `Blue` is complete.  Also, don't make it a SQL script.  Update the code to backfill the data and handle all the possible missing data scenarios.  The backfill script should instead be a PowerShell script or a BashScript which invokes the API.  

**Please Note:** There is not a set in stone solution on how to have the code accomplish backfilling of the `CustomerFullName` column.  Every application has its own rules about patterns and practices to follow.  I am going to suggest a couple of approaches, but use them as guidelines.  Follow your application's patterns and practices.  It is better to have consistency in the code than it is to do something which you feel is correct "correct."  

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

That works fine if all the properties are on the same model.  And the same model is used by all layers, the UI, business, and database.  In my experience, it takes a great deal of discipline as the models, and the database has to be a one to one match.  In my experience, that is rarely the case.  I've seen ApiModels, ViewModels, or some other phrase which indicates there is a difference in the model and the database.  

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

As stated earlier, the backfill script can be PowerShell or Bash.  It can query the database to find a list of customers and then use that list of customers to hit the API.  Unlike the typical SQL Script, the only time this example script touches the database is to find a list of customers who have the `CustomerFullName` set to `Null`.

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

That script will be invoking code which is currently running in production.  It is most likely getting hit thousands of times per day.  And there should be automated tests around it as well.  It will take much longer to run the script than a simple SQL update script.  But it is nondestructive, and it can be run at any time.  Because it can be run at any time, it doesn't need to occur during deployment.  Having it run outside of deployment means it isn't holding up anything.  The script can be kicked off, and it can plug away for as long as it needs to.  

I recommend this approach for several reasons.

1. All the timing concerns are taken care of.  There is no need to worry about running the backfill script multiple times to make sure a record isn't missed somehow.  No need to worry about how long it will take to finish running.  It will help make the transition from `Green` to `Blue` as seamless as possible.
2. This change needs to happen anyway to prevent errors from occurring.  While `Blue` is being deployed and verified, users will be making database updates on `Green` after the first run of the backfill script.  If the person or automated process doing the verification on `Blue` happens to hit one of those records and the code wasn't updated, then an error would occur. 
3. The code also needs to change in the event the application somehow gets a request without the `CustomerFullName` property populated.  Could come from a SPA application or for some other reason.  Setting that column in the database to `Null` after it has been populated is a problem.  But if it did happen the code would be able to handle it.  
4. Backfill scripts are typically SQL Scripts.  SQL is perfect for querying databases.  It is not a great language to write functional programming, integration tests, or unit tests.  The code should be the only place where the logic of backfilling the `CustomerFullName` column is located.  Having a backfill script AND the code contains that logic opens the door to errors.  Any change in rules will require an update to both the code and the script.
5. Most database deployment tools, be it Redgate, DBUp, RoundhousE, or SSDT don't have a mechanism for running specific SQL scripts multiple times.  Without that built-in functionality, a hack will have to put into place to support it.  
6. Each time the script runs, it will come across "correct" data.  Guard clauses - which are hard to test in SQL Scripts - will have to be put into place to prevent overwriting of data.

## Stored Procedures, Views and Legacy Columns

I am a big believer in the database should be used as a data store only.  Keep as much logic as possible out of the database, including stored procedures and views.  Business logic in the database includes, but not limited to, formatting, calculation, the inclusion of IsNull checks, if/then statements, while statements, default values, and filtering more than just by an Id.  

It is far too easy to end up copying/pasting that logic between stored procedures.  Imagine the application had two stored procedures:

- usp_GetCustomerById
- usp_GetAllCustomers

Fundamentally they do the same thing, get customers from the database.  The CustomerFullName is a new column, but it is null.  Rather than having a function in the code check to see if `CustomerFullName` is `Null`, that logic is placed in the database.

```SQL
Select CustomerFirstName,
       CustomerLastName,
       IsNull(CustomerFullName, CustomerFirstName + ' ' + CustomerLastName) as CustomerFullName
from dbo.Customer
```

That only hides bad or missing data; it doesn't solve missing data.  It is a one-liner, and oh so easy to copy-paste to other stored procedures.  In this example, it is only one other stored procedure.  Then one day someone adds a new stored procedure, and they forget to include that.  But, it has been long enough the `CustomerFullName` column is populated for 90% of the records.  It is only a matter of time before an error occurs.  And when it comes time to remove `CustomerFirstName` and `CustomerLastName` from the database they could miss that IsNullCheck.  Imagine if a stored procedure had 50 columns and `CustomerFullName` was several dozen lines apart.

Retrieve stored procedures and views should return the data as is, without any null checks or formatting.  

```SQL
Select CustomerFirstName,
       CustomerLastName,
       CustomerFullName
from dbo.Customer
```

Create or update stored procedures are slightly different.  `Green` will be active while `Blue` is being verified.  `Green` has no concept of `CustomerFullName` which means it will call stored procedures without including that new column.  Once `Blue` goes active, it will no longer be sending in `CustomerFirstName` and `CustomerLastName` fields to the stored procedures.  The stored procedures need to be able to accept both scenarios.  

That brings up an interesting scenario.  Once `Blue` goes active, there will still be data in the `CustomerFirstName` and `CustomerLastName` columns.  It would be bad for the code to set that data to null.  With new records, that data won't be present.  The code shouldn't try to guess how to split up the first name and last name.  For existing records, the code should leave the data as-is.  For new records, it needs to set those columns to `Null`.  Because of that, if the column doesn't allow `Null`, then it needs to be modified to accept that.  Don't set the value to an empty string.  An empty string is a value.  `Null` is the absence of a value.

Just like adding the column to the table as nullable, the parameter for `CustomerFullName` should be nullable to support when `Green` is active. The parameters for `CustomerFirstName` and `CustomerLastName` should be nullable to support when `Blue` is active.

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

While it is possible to run an insert command without specifying the columns, don't do that.  Specify all the columns.  It only causes headaches down the line.  The insert stored procedure will be similar to the update stored procedure, all the parameters, in this case, have the default value set to `Null` to handle both `Blue` and `Green` calling it.  

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

Just like before, the temptation is there to make the update statement check for null in the parameter.  It is only one line, what is the harm?

```SQL
    set CustomerFirstName = IsNull(@CustomerFirstName, CustomerFirstName)
```

The code for `Blue` should be where that null check lives.  Putting the `IsNull` check in the database hides a business rule.  That makes it harder to test, as writing unit tests for SQL Server, even with a tool like tSQLt, is much harder than writing unit tests for code.  It also makes it harder to find that rule, as typically people will search through C# code.  And as stated before, that same SQL statement could be duplicated across multiple stored procedures.  

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

Views are great at providing a layer of abstraction, and when written correctly, can give a nice performance boost.  However, I've seen too many of them written poorly and end up causing more performance problems.  A lot of people love to abuse Common Table Expressions (CTE) in views, which is where a lot of performance problems stem.  To help resolve those performance problems, the "no business logic in the database" rule get broken.  I recommend avoiding that as much as possible.  Just like stored procedures, return all three columns, `CustomerFirstName`, `CustomerLastName`, and `CustomerFullName`.

An everyday use case for views is to help data warehousing.  The abstraction layer views provide makes it easy for a process to come through and copy all the data over to a data warehouse for business intelligence teams to create reports from.  There isn't a right automated way to make this change known to the data warehouse.  My recommendation is to notify them of the change as soon as possible.  

**Please Note:** I realize moving business logic out of the database is quite the change.  If you do decide to make that switch, be pragmatic about it.  Don't try to change everything at once.  The code and the database are working fine now.  This section is about changes to the database going forward.  My rule of thumb is to leave the database and code in a better place than when I found it.  Meaning, if I am making a change to a column and I come across a business rule in the database for that column, then I do some analysis.  If the change is relatively easy to make, then I will make it then.  If it is complex, then I will create a card and put it on our technical debt backlog to be fixed later (hopefully).  Sometimes backlogs are where work goes to die.  I don't tear through all the code and stored procedures and start making mass changes.  That is a surefire way to end up doing an emergency deployment on the weekend.

## Removing Columns

At some point, the `CustomerFirstName` and `CustomerLastName` columns will be removed.  Blue/Green deployments make that more complex as well.  Assume `CustomerFullName` has been deployed to `Blue`, and it is active.  Running a script to delete `CustomerFirstName` and `CustomerLastName` from all the tables, views, functions, and stored procedures will cause errors.   The code running on `Blue` is using the `CustomerFirstName` and `CustomerLastName` fields to populate `CustomerFullName` when `CustomerFullName` is null.  

How can the `CustomerFirstName` and `CustomerLastName` columns get removed with that restriction in place?  To answer that, look at a typical standard deployment with an outage.

1. Remove references in the code to `CustomerFirstName` and `CustomerLastName`.
2. Delete `CustomerFirstName` and `CustomerLastName` from the database.

Each of those steps will be a separate deployment.  They don't have to be deployed within days of each other.  I've seen several months go between those two deployments.  That timespan is to allow other applications to make the necessary adjustments to stop referencing those columns.

I've worked on an application which was one of dozen or so which connected to the same database.  The application list was split amongst three or four teams.  To keep things somewhat organized all standard tables and stored procedures were in the `dbo` schema.  While tables and stored procedures only used by a single application wherein an application schema.  That reduces possible database changes to add only to the `dbo` schema while having a bit more flexibility in the application schema.

In some cases, it might not be possible to remove columns or tables.  I hope that isn't the case, but be prepared for that scenario.  In this case, communication, planning, and effort are essential.  If it is essential to remove those columns, then the effort should be made to do so.  I'd argue the effort is worth it.  Be pragmatic about it; if it is going to take 1000 hours to make the change, I'd say the effort isn't worth it.

## Versioning Stored Procedures, Views, and APIs

The typical rule of thumb is "if you make a breaking change, then version the API/Stored Procedure/View."  On paper, that is a good rule to follow.  In practice, that rule can fall apart quickly.  Versioning puts a significant burden on the people maintaining the code.  The maintainers now there are multiple code paths or multiple instances they need to worry about.  The longer an older version sticks around, the harder and harder it will be to get off the old versions.  I've seen companies spends hundreds of hours on projects to get people off of past versions.  

I recommend the default position should be to make changes as backward compatible as possible.  Have a single code base, single stored procedure, single view, which everyone uses.  A single code base will make maintenance easier, and it will make it easier for changes to be made (over time).  Look at multiple deployments to make small changes over time compared to a big bang.  Explore all the options before jumping into the versioning pool.  Versioning should be considered after all other options are exhausted.  

## When database changes should be deployed

Taking everything from this article into account for the test scenario; I'd argue the database changes are not required to be deployed at the same time as the code.  The only thing that is required is the database changes have to be deployed before the code.  Deploying a database change days or even weeks before deploying the code might have an added benefit.  If another application is using the same database (even if it's just for a view), this will give them time to change and test their code.  Once the database change is up there, the other teams can deploy when they're ready.  

The real question is, does it make sense to deploy the database changes days or even a week before the code?  That is a bit trickier to answer.  My recommendation is to do what makes sense dependent upon the scenario — the less change made during a deployment, the better.  For the most part, fewer changes mean less risk.  Pushing database changes to production before the code has a bit of a risk as well.  Once development, testing, or UAT starts, additional database changes might be required.  If the first change made it to production, any other changes would require another push to production.  

My preference is to store the code and database in the same repository.  Get everything working in a feature branch.  Merge all the changes in the feature branch at the same time and test and verify at the same time.  

What I do like about Blue/Green deployments is the flexibility it gives for database deployments.  Before Blue/Green deployments, everything had to go out at the same time during the deployment.  Now there is a choice.  

## Testing and Verification

Automated testing and verification of `Blue` before swapping with `Green` (and vice versa) makes everything go a lot faster.  Computers can test a heck of a lot faster than people.  Computers can perform more tests in less time, which makes everyone more confident in the deployment.  Automated testing and verification isn't a requirement for Blue/Green deployments.  They help a whole lot.  Deployment tools such as Octopus Deploy have the manual intervention step which can pause a deployment and wait until someone gives to go-ahead to proceed.  

Most people starting Blue/Green deployments will not have a full test suite they can run in Production on day one.  The key is to start somewhere.  It will take time to build out that test suite and overcome technical hurdles.  Depending on the tooling, even a simple scenario, changing from the URL for `Blue` with `Green` could take a bit to figure out.  As tests come online include them in the deployment pipeline to help automate the verification.  Hopefully, the time will come where the vast majority of use cases can be verified, and the manual intervention step can be removed.

## Common Database Change Scenarios

This post covered a very complex scenario, the combination of two columns into one.  There are several other database change scenarios detailed in the list below.  I've included which of the above sections could be applied to the scenario.

- Add a new column -> follow all the steps except deleting old columns and handling legacy columns
- Renaming a column -> Don't rename.  Follow the steps above to add a new column and remove the old column.
- Adding a new table -> Similar to adding a new column
- Renaming a table -> Don't rename.  Follow the steps above to add a new column and delete the old columns.
- Moving a column to another table -> Very similar to adding a new column and removing the old column.  The only difference is where the column was added.
- Deleting a table -> Very similar to deleting a column.  Hopefully, that table isn't used anymore, and all the data has been migrated to other tables.
- Adding a new view/stored procedure -> Very similar to adding a new column.  The updated code will use the new view/stored procedure; the old code will not use the view/stored procedure. 
- Updating an existing view/stored procedure -> As long as columns aren't removed from a view, this should be fine.  If columns are removed, then the process to delete columns from above should be followed.
- Removing a view/stored procedure -> Very similar process to removing columns.  Except there won't be data, just references in code and potentially other stored procedures.

## Wrapping Up

Blue/Green deployments with a database will require more planning on how a change is implemented than doing a standard deployment with an outage.  It is a lot like playing chess, where you will have to think several steps ahead.  Blue/Green deployments are not for every application.  It requires the right alignment of architecture, infrastructure, and tooling.  If you are considering Blue/Green deployments, take a feature with a database change and walk through what it would take to implement that using a Blue/Green deployment strategy.  Do you have to change anything in your application's architecture?  Is the necessary infrastructure, such as a load balancer and additional VMs, in place?  Can your CI/CD tool support blue/green deployments?

In terms of time, the initial cost to do Blue/Green deployments can be high.  That cost is worth it when it is possible to deploy a change in the middle of the day.  Being able to do that opens up so many different possibilities.  Soon the conversation will move from "how can I deploy in the middle of the day" to "with Blue/Green deployments in place, now what can I do?"  Blue/Green Deployments, or seamless daytime deployments, feels like it is the end of the CI/CD journey.  I'd argue that it is just the beginning.

