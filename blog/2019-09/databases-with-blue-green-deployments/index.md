---
title: Automated database blue-green deployments 
description: Learn some techniques for automating database deployments when using a blue-green deployment strategy.
author: bob.walker@octopus.com
visibility: public
metaImage: 
bannerImage: 
published: 2019-09-30
tags:
 - Database Deployments
 - Blue-Green Deployments
---

I hate when deployments are scheduled for 2:00 AM Saturday.  It ruins Friday night and Saturday because it throws off my sleep schedule.  Several years ago I was working on an application and 2:00 AM Saturday is the only time an extended outage can be taken.  A quick deployment and verification took two hours.  A typical deployment took four hours.  I wanted to do blue-green deployments, which is a pattern for zero downtime deployments, to stop that madness.  So why weren't we doing blue-green deployments?  It was the database, which made everything much more complicated; enough to stop us from implementing blue-green deployments.  In this post I will walk through some techniques I've learned since that time to make blue-green deployments with a database achievable and straight-forward.      

**Please Note:** This post will cover high level concepts and recommendations.  It won't cover how to do blue-green deployments with Octopus Deploy.  That will be covered in a later post.

!toc

## A brief intro to blue-green deployments

Before diving into the weeds, a brief intro on blue-green deployments.  blue-green deployments are when there are two identical production environments labeled `Blue` and `Green`.  At any given time, only one of those environments, for example, `Blue`, is live.  Deployment is done to the non-live environment, for example, `Green`, which is then verified.  After verification is complete, a switchover occurs, and the live environment becomes `Green`.  

![](https://i.octopus.com/docs/deployment-patterns/blue-green-deployments/images/3278250.png)

There are several advantages to doing this.  Rollbacks should be easy, only a matter of switching from blue to green or green to blue.  When a switchover does occur, it should be seamless as the code has already been running, there is no need to wait for it to compile or "warm-up."  And changes can be verified in production without having any customers hitting the code, making the deployment much less risky.  If something doesn't work, don't make the switch, try again at a later time. 

Before diving into too much farther into the article, I should point out not all applications can take advantage of blue-green deployments.  It is a combination of architecture, how stateful the app is, and the technology used.  The more stateless and decoupled the application is, the better the chance it can be deployed using the blue-green deployment strategy.  A .NET Core Web API with an Angular front-end is suited much better for blue-green deployments than an ASP.NET WebForms application without a business logic or data layer and requires the sticky sessions from the load balancer.

## The scenario

This post will be covering a complex scenario.  The following changes will be made to an [Single Page Application](https://en.wikipedia.org/wiki/Single-page_application)(SPA) with an ASP.NET Web API back-end connecting to a dedicated database.  

- In the UI, there used to be two fields, `First Name` and `Last Name`, but the decision was made to combine the two fields into one called `Full Name`.  Some cultures have multiple names which do not fit into the `First Name` and `Last Name` commonly seen in the US.  
- To support this, a new column `CustomerFullName` will be added to the customer table.
- The pre-existing `CustomerFirstName` and `CustomerLastName` columns are also on the customer table.  
- `CustomerFullName` will be populated by combining `CustomerFirstName` and `CustomerLastName` when `CustomerFullName` is `Null`. 
- If data exists in `CustomerFirstName` and `CustomerLastName` and an API request is sent with only `CustomerFullName` then do not set those columns to `Null`.  In addition, do not try to split up `CustomerFullName` as that is very difficult and prone to error.

Admittedly, that scenario is relatively complex.  Most database changes don't combine two columns into one and try to backfill the new column.  This article will walk through how to solve that scenario.   That scenario touches on a lot of different changes which must be done for blue-green deployments.  If that scenario can be solved, then the majority of other scenarios can be solved.  It will walk through each change, the questions to consider, and recommendations on how to solve them.

While it is possible to do blue-green deployments with a shared database, it requires a lot of communication and coordination between teams.  This scenario is a complex change to a database, I didn't want to muddy the waters with the inclusion of shared databases.  

For this entire article, when talking about deployments, `Green` is currently live, while `Blue` is inactive.  When we deploy, it will be to `Blue`.  Once `Blue` is verified it will become the live environment and `Green` will be inactive.

**Please Note:** These are recommendations only.  There is no way I can cover every possible change you can make to a database.  My goal is to provide you with something which you can then modify to meet your own needs.  

## How database changes add complexity

After-hours deployments are done to avoid users accessing the application while the code and database changes are being deployed and verified.  Because all the changes are happening at once, a backfill script will be written to set `CustomerFullName` to `CustomerFirstName` and `CustomerLastName`.  Having a user access the application while all this is happening will cause errors and confusion.

A very simplified deployment process for that change would look like this:

1. Disable access to users or turn off the website.
2. Run the script to add the column `CustomerFullName`.
3. Deploy the code to `Production`.
4. Run the backfill script to populate `CustomerFullName`.
5. Run the script to remove `CustomerFirstName` and `CustomerLastName`.
6. Verify the code in `Production`.
7. Profit.

To accomplish the same changes with blue-green deployments requires a lot more planning.  Without blue-green deployments, the only worry was someone is using the application before everything is deployed and verified.  If the user gets an error indicating the column `CustomerFirstName` is missing during the deployment, no worries, they shouldn't be in the system at that point anyhow.  That is not the case with blue-green deployments.  Users will be in the application; they will be running on the `Green` servers.  That means data is going to get manipulated and queried.  

Here are some of the scenarios to consider when doing the same change with blue-green deployments: 

- The application and database changes will be deployed to `Blue` which include the new column `CustomerFirstName`.  Does the application use any stored procedures?  Specifically around inserting/updating data into the customer table?  The code running on `Green` won't know about any new parameters added to those stored procedures.
- When the changes are deployed to `Blue` there will be no data in the `CustomerFirstName` column.  The temptation will be there to use the same backfill script as before.  The thought being the backfill script will help with verification and make the transition from `Green` to `Blue` seamless.
- Verification will need to happen after the changes to the application and database are deployed to `Blue`. During that time, users will be using the application on `Green`.  `Green` is still running code referencing `CustomerFirstName` and `CustomerLastName` columns.  Those columns cannot be deleted until after verification is complete and `Blue` becomes active.  When should those columns be deleted?  As soon as `Blue` becomes active?  
- While the updated code and database is being verified on `Blue` users will be adding and updating records in the customer table using the code on `Green`.  Those changes could be made while the backfill script is running or after the backfill script finishes.  To pick up those new changes the backfill script will need to be rerun.
- With this being a SPA App, the JavaScript won't know when `Blue` becomes live and `Green` becomes inactive.  The JavaScript is stored in the user's browser.  Users using the application when `Blue` becomes live will be sending API requests with only the fields for `CustomerFirstName` and `CustomerLastName`.   The `CustomerFullName` field will not be sent in during this time.  It could take anywhere from one minute to several days before users start requesting updated JavaScript files from the server.   

The first stab at the blue-green deployment process for this change would look like:

1. Run the script to add the column `CustomerFullName`.
2. Run the backfill script to populate `CustomerFullName`.
3. Deploy code to `Blue` environment.
4. Verify the code and database changes in the `Blue` environment.
5. Swap the live environment from `Green` to `Blue`.
6. Run the backfill script to populate `CustomerFullName`.

That is not the ideal blue-green deployment process. It runs the backfill script multiple times.  It doesn't answer when `CustomerFirstName` and `CustomerLastName` should be deleted.  But it is a start.  The rest of this post will walk through techniques on how to answer those questions and how to make an ideal blue-green deployment process.

## Database changes

I am firmly in the database should only store data camp.  The database should not contain business rules or business logic.  Only the code should store business rules and business logic.  A business rule would be a default value on a column.  When the database stores business rules or business logic it makes blue-green deployments much harder.

Even if it is an empty string or zero.  Those are values.  If a column doesn't have a value it needs to be set to `Null`, which is the absence of a value.  Business logic in the database includes, but not limited to, formatting, calculation, the inclusion of IsNull checks, if/then statements, while statements, default values, and filtering more than just by an Id.  Having business rules and business logic in the database is asking for trouble.  Compared to code, such as C#, JavaScript or Java, they are much harder to write unit tests for, even when using a tool such as tSQLt.  They are much harder for developers to find, as typically most developers will search using their IDE of choice which excludes databases.

This section will walk through recommendations for table changes, stored procedure and view changes to support blue-green deployments.  It will also cover techniques on how to avoid having business rules and business logic in the database which can trip up blue-green deployments.

### Table Changes

Make non-destructive database changes when doing blue-green deployments.  In our scenario, that means making the `CustomerFullName` nullable when it is added.  Making a column non-nullable without a default value would be a destructive change.  Insert statements on `Green` would stop working because it doesn't know about that new column.  That doesn't mean the column should be made non-nullable with a default value, even an empty string.  As said before, a default value is a business rule.

The other problem is it takes quite a bit of time for most database servers (IE SQL Server or Oracle) to add a non-nullable column.  When a non-nullable column is added the table definition is updated along with every record.  When a nullable column is added only the table definition is updated.  If you do have to have a default value then the script should add a column as nullable, update all the records, then set the column to non-nullable.  That script can be tuned to run surprisingly fast.  
Some other examples of destructive database changes would be reusing an existing column or renaming an existing column.  With blue-green deployments that will fail as the code in `Green` will start throwing errors or showing inaccurate data.   

So far this section has covered adding the new column, `CustomerFullName` to the customers table.  What about the older `CustomerFirstName` and `CustomerLastName` columns?  In the scenario section it essentially says to leave `CustomerFirstName` and `CustomerLastName` alone.  Don't overwrite it with `Null` and don't try to guess what those values will be.  In addition, once `Blue` goes live with these changes any new records will never have a value for `CustomerFirstName` and `CustomerLastName`.  Because of that `CustomerFirstName` and `CustomerLastName` should be made nullable.

At some point, the `CustomerFirstName` and `CustomerLastName` columns will be removed.  Blue-green deployments make that more complex as well.  Assume `CustomerFullName` has been deployed to `Blue`, and it is active.  Running a script to delete `CustomerFirstName` and `CustomerLastName` from all the tables, views, functions, and stored procedures will cause errors.   The code running on `Blue` is using the `CustomerFirstName` and `CustomerLastName` fields to populate `CustomerFullName` when `CustomerFullName` is null.  

It will take multiple deployments to delete those columns from the database.  

1. Deployment adds `CustomerFullName` to the customer table.  `CustomerFistName` and `CustomerLastName` are needed because the older code still references them.
2. Deployment occurs where the code removes all references to `CustomerFirstName` and `CustomerLastName`.
3. Final deployment to delete `CustomerFirstName` and `CustomerLastName` from the database.
How can the `CustomerFirstName` and `CustomerLastName` columns get removed with that restriction in place?  To answer that, look at a typical standard deployment with an outage.

Those deployments don't have to be deployed within days of each other.  I've seen several months go between those each deployments.  

I've worked on an application which was one of dozen or so which connected to the same database.  Having a shared database makes removing columns very, very tricky.  In this case, communication, planning, and effort are essential.  If it is essential to remove those columns, then the effort should be made to do so.  I'd argue the effort is worth it.  Be pragmatic about it; if it is going to take 1000 hours to make the change, I'd say the effort isn't worth it.  

### Stored procedures and views

Imagine the application had two stored procedures:

- usp_GetCustomerById
- usp_GetAllCustomers

Fundamentally they do the same thing, get customers from the database.  The `CustomerFullName` is a new column, but it is null.  As seen earlier, the backfill script raised a lot of questions.  One option would be to skip the backfill script completely, and have the stored procedures do a quick IsNull check.

```SQL
Select CustomerFirstName,
       CustomerLastName,
       IsNull(CustomerFullName, CustomerFirstName + ' ' + CustomerLastName) as CustomerFullName
from dbo.Customer
```

That only hides bad or missing data; it doesn't solve missing data.  It is a one-liner, and oh so easy to copy-paste to other stored procedures.  In this example, it is only one other stored procedure.  Then one day someone adds a new stored procedure, and they forget to include that.  It is also possible to forget to update the stored procedures when it comes time to remove `CustomerFirstName` and `CustomerLastName` from the database.  Imagine if a stored procedure had 50 columns and `CustomerFullName` had several dozen lines between it and `CustomerFirstName` and `CustomerLastName`.

Retrieve stored procedures and views should return the data as is, without any null checks or formatting.  

```SQL
Select CustomerFirstName,
       CustomerLastName,
       CustomerFullName
from dbo.Customer
```

Create or update stored procedures are slightly different.  `Green` will be active while `Blue` is being verified.  `Green` has no concept of `CustomerFullName` which means it will call stored procedures without including that new column.  Once `Blue` goes active, it will no longer be sending in `CustomerFirstName` and `CustomerLastName` fields to the stored procedures.  The stored procedures need to be able to handle either `Blue` or `Green` calling it. 

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

While it is possible to run an insert command without specifying the columns, don't do that.  Specify all the columns.  It only causes headaches down the line as the order of the columns in the insert statement have to match order of the columns in the table.  If a column is added in the middle of the table (which happens!) the insert statements will start randomly failing or inserting data into the wrong column.  The insert stored procedure will be similar to the update stored procedure, all the parameters, in this case, have the default value set to `Null` to handle both `Blue` and `Green` calling it.  

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

Views are great at providing a layer of abstraction, and when written correctly, can give a nice performance boost.  However, I've seen too many of them written poorly and end up causing more performance problems.  A lot of people love to abuse Common Table Expressions (CTE) in views, which is where a lot of performance problems stem.  To help resolve those performance problems, the "no business logic in the database" rule get broken.  I recommend avoiding that as much as possible.  Just like stored procedures, return all three columns, `CustomerFirstName`, `CustomerLastName`, and `CustomerFullName`.

An everyday use case for views is to help data warehousing.  The abstraction layer views provide makes it easy for a process to come through and copy all the data over to a data warehouse for business intelligence teams to create reports from.  There isn't a right automated way to make this change known to the data warehouse.  My recommendation is to notify them of the change as soon as possible.  

**Please Note:** I realize moving business logic out of the database is quite the change.  If you do decide to make that switch, be pragmatic about it.  Don't try to change everything at once.  The code and the database are working fine now.  This section is about changes to the database going forward.  My rule of thumb is to leave the database and code in a better place than when I found it.  Meaning, if I am making a change to a column and I come across a business rule in the database for that column, then I do some analysis.  If the change is relatively easy to make, then I will make it then.  If it is complex, then I will create a card and put it on our technical debt backlog to be fixed later (hopefully).  Sometimes backlogs are where work goes to die.  I don't tear through all the code and stored procedures and start making mass changes.  That is a surefire way to end up doing an emergency deployment on the weekend.

## Code Changes

As stated in the database changes section, business rules and business logic should exist in the code.  The scenario defined a couple of business rules which need to be placed into the code.  

- `CustomerFullName` will be populated by combining `CustomerFirstName` and `CustomerLastName` when `CustomerFullName` is `Null`. 
- If data exists in `CustomerFirstName` and `CustomerLastName` and an API request is sent with only `CustomerFullName` then do not set those columns to `Null`.  In addition, do not try to split up `CustomerFullName` as that is very difficult and prone to error.

This section will walk through the techniques on how to put those rules into code using C# as the example language.  

### Handling Null in CustomerFullName
I'm not going to lie, it is very easy to put a check like this in a stored procedure.  But as stated before, that is not a good idea for a variety of reasons.

```SQL
IsNull(CustomerFullName, CustomerFirstName + ' ' + CustomerLastName) as CustomerFullName
```

One option would be to put that formatting rule in the model representing the customer table.

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

That works fine if the same model is used by all layers, the UI, business, and database.  In my experience, it takes a great deal of discipline as the models, and the database has to be a one to one match.  However, that is rarely the case.  I've seen ApiModels, ViewModels, or some other phrase which indicates there is a difference in the model and the database.  

An alternative to putting all the logic in the model would be:

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
Once all the changes on `Blue` go live, there will still be data in the `CustomerFirstName` and `CustomerLastName` columns.  It would be bad for the code to set that data to null.  With new records, that data won't be present.  The code shouldn't try to guess how to split up the first name and last name.  For existing records, the code should leave the data as-is.  For new records, it needs to set those columns to `Null`.  

Just like before, the temptation is there to make the update statement check for null in the parameter.  It is only one line, what is the harm?

```SQL
    set CustomerFirstName = IsNull(@CustomerFirstName, CustomerFirstName)
```

The code for `Blue` should be where that null check lives.  Putting the `IsNull` check in the database hides a business rule, which is what we want to avoid.

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

Most backfill scripts I've seen are nothing more than a SQL Script to update the underlying data.  This means the formatting logic will exist in both the code and the backfill script.  It is very easy to update the code to handle an update to the formatting rule and forget to update the backfill script.  

In addition, with blue-green deployments, there is that question of when should that script run?  After the code has been deployed to `Blue` and the database changes have been pushed out but before verification starts?  After `Blue` goes live and `Green` is inactive?  A traditional backfill SQL script is updating low level data and it could be locking the table.  Depending on the number of records it could take a long time.  Maybe the decision is made to update 1000 records at a time in the SQL script.  What if users are updating the same record as the script and a deadlock occurs and the script wins the deadlock, not the user update?  Is that okay?  Depending on when the script runs, it may need to have guard clauses in place to prevent an accidental update.

As an added bit of fun, most database deployment tools, be it Redgate, DBUp, RoundhousE, or SSDT don't have a mechanism for running specific SQL scripts multiple times.  Without that built-in functionality, a hack will have to put into place to support it. 

The code has the formatting rules in place.  It has the necessary logic to handle a variety of scenarios.  It should hopefully have unit tests covering it.  This is what is verified by QA.  The backfill script should be written in PowerShell or Bash and invoke the API instead of updating the database directly.  It can query the database to find a list of customers and then use that list of customers to hit the API.  Logic can be added to the script to minimize the risk of both the script and user updating a record at the same time.  Maybe the user tied to a customer has a timezone preference, the script is run each hour via an automated process and only updates records when the user's timezone is between 2 AM and 3 AM.  

Below is an example backfill script written in PowerShell.

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

The script is non-destructive, and it can be run at any time.  Because it can be run at any time, it doesn't need to occur during deployment.  Having it run outside of deployment means it isn't holding up anything.  The script can be kicked off, and it can plug away for as long as it needs to.  All the timing concerns are taken care of.  There is no need to worry about running the backfill script multiple times to make sure a record isn't missed somehow.  No need to worry about how long it will take to finish running.  It will help make the transition from `Green` to `Blue` as seamless as possible.

## Versioning stored procedures, views, and APIs

The typical rule of thumb is "if you make a breaking change, then version the API/Stored Procedure/View."  On paper, that is a good rule to follow.  In practice, that rule can fall apart quickly.  Versioning puts a significant burden on the people maintaining the code.  The maintainers now there are multiple code paths or multiple instances they need to worry about.  The longer an older version sticks around, the harder and harder it will be to get off the old versions.  I've seen companies spends hundreds of hours on projects to get people off of past versions.  

I recommend the default position should be to make changes as backward compatible as possible.  Have a single code base, single stored procedure, single view, which everyone uses.  A single code base will make maintenance easier, and it will make it easier for changes to be made (over time).  Look at multiple deployments to make small changes over time compared to a big bang.  Explore all the options before jumping into the versioning pool.  Versioning should be considered after all other options are exhausted.  

## When database changes should be deployed

Taking everything from this article into account for the test scenario; I'd argue the database changes are not required to be deployed at the same time as the code.  The only thing that is required is the database changes have to be deployed before the code.  Deploying a database change days or even weeks before deploying the code might have an added benefit.  If another application is using the same database (even if it's just for a view), this will give them time to change and test their code.  Once the database change is up there, the other teams can deploy when they're ready.  

The real question is, does it make sense to deploy the database changes days or even a week before the code?  That is a bit trickier to answer.  My recommendation is to do what makes sense dependent upon the scenario — the less change made during a deployment, the better.  For the most part, fewer changes mean less risk.  Pushing database changes to production before the code has a bit of a risk as well.  Once development, testing, or UAT starts, additional database changes might be required.  If the first change made it to production, any other changes would require another push to production.  

My preference is to store the code and database in the same repository.  Get everything working in a feature branch.  Merge all the changes in the feature branch at the same time and test and verify at the same time.  

What I do like about blue-green deployments is the flexibility it gives for database deployments.  Before blue-green deployments, everything had to go out at the same time during the deployment.  Now there is a choice.  

## Testing and verification

Automated testing and verification of `Blue` before swapping with `Green` (and vice versa) makes everything go a lot faster.  Computers can test a heck of a lot faster than people.  Computers can perform more tests in less time, which makes everyone more confident in the deployment.  Automated testing and verification isn't a requirement for blue-green deployments.  They help a whole lot.  Deployment tools such as Octopus Deploy have the manual intervention step which can pause a deployment and wait until someone gives to go-ahead to proceed.  

Most people starting blue-green deployments will not have a full test suite they can run in Production on day one.  The key is to start somewhere.  It will take time to build out that test suite and overcome technical hurdles.  Depending on the tooling, even a simple scenario, changing from the URL for `Blue` with `Green` could take a bit to figure out.  As tests come online include them in the deployment pipeline to help automate the verification.  Hopefully, the time will come where the vast majority of use cases can be verified, and the manual intervention step can be removed.

## Common database change scenarios

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

## Wrapping-up

As you can see, blue-green deployments with a database requires a few minor shifts in how you write code and making database changes.  It also requires more planning on how a change is implemented than doing a standard deployment with an outage.  It is a lot like playing chess, where you will have to think several steps ahead. 

The first stab at a blue-green deployment process looked like this:

1. Run the script to add the column `CustomerFullName`.
2. Run the backfill script to populate `CustomerFullName`.
3. Deploy code to `Blue` environment.
4. Verify the code and database changes in the `Blue` environment.
5. Swap the live environment from `Green` to `Blue`.
6. Run the backfill script to populate `CustomerFullName`.

That process has now evolved into this:

1. Run the script to add the column `CustomerFullName`.
2. Deploy code to `Blue` environment.
3. Verify the code and database changes in the `Blue` environment.
4. Swap the live environment from `Green` to `Blue`.

The backfill script is not included at all, it is run after the deployment to help populate the new `CustomerFullName` column.  In fact, that script doesn't need to run until the code needs to be changed to remove the `CustomerFirstName` and `CustomerLastName` columns.  

Before ending this post I do want to point out blue-green deployments are not for every application.  It requires the right alignment of architecture, infrastructure, and tooling.  If you are considering blue-green deployments, take a feature with a database change and walk through what it would take to implement that using a blue-green deployment strategy.  Do you have to change anything in your application's architecture?  Is the necessary infrastructure, such as a load balancer and additional VMs, in place?  Can your CI/CD tool support blue-green deployments?

In terms of time, the initial cost to do blue-green deployments can be high.  That cost is worth it when it is possible to deploy a change in the middle of the day.  Being able to do that opens up so many different possibilities.  Soon the conversation will move from "how can I deploy in the middle of the day" to "with blue-green deployments in place, now what can I do?"  Blue-green Deployments, or seamless daytime deployments, feels like it is the end of the CI/CD journey.  I'd argue that it is just the beginning.

Until next time, Happy Deployments!