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

It's 2:00 AM on Saturday sometime in the fall of 2011.  I'm up for one reason, to help deploy some software to production.  At 10 minutes to 2:00 AM, everyone jumps on a conference call.  

- Me: Alright, let's get this party started.
- Ops: Alright, I'll add [app_offline.htm](https://docs.microsoft.com/en-us/aspnet/web-forms/overview/deployment/advanced-enterprise-web-deployment/taking-web-applications-offline-with-web-deploy#task-overview) to both servers
- QA: Verified, getting a maintenance message for the application
- Me: DBA, go ahead and start running the database scripts
- DBA: Starting
- Me: Ops, go ahead and start deploying the code
- Ops: Okay, taking Server A out of the load balancer
- QA: Confirmed, Server A is out of the load balancer
- Ops: Deploying Code, and I'm done for Server A, removing app_offline.htm
- QA: Getting an error on the server, something about a column missing
- Me: I don't think the database scripts have finished
- DBA: Nope, still running them
- QA: Let me know when they're finished
- DBA: Okay, now they are done
- QA: Code has been verified on Server A
- Ops: Adding Server A into the load balancer, removing Server B and removing app_offline.htm
- QA: Verified Server A is in the load balancer, but I am getting an error about missing columns on Server B
- Ops: I haven't deployed to Server B yet
- QA: Ah, okay, let me know when you're done

Sadly, that kind of deployment was very, very common for the first 11 years of my career as a developer.  Fast deployments for that application took two hours; average deployments took long enough to see the sunrise.  When deployments take at least two hours to deploy the code, run the database scripts, and verify the changes there is no way we could do them in the middle of the day.  We had a robust load balancer and plenty of hypervisors to run VMs in production.  Why weren't we doing Blue/Green deployments?  With Blue/Green deployments, we could deploy the code to production in the middle of the day to inactive servers in production and verify them.  At night (even 2:00 AM Saturday), run a script to tell the load balancer to point all traffic to the new servers.  Simple, it was the database.

I have a feeling that the database is what is preventing a lot of you from doing Blue/Green deployments.  More often than not, there is a single database which 1 to N VMs points to.  It is possible to do Blue/Green deployments with a database such as SQL Server, Oracle, MySql, or PostgreSQL.  In this post, I will walk through some techniques on how to do that.  

!toc

## A Brief Intro to Blue/Green Deployments

If you're not familiar with Blue/Green deployments, then time for a brief intro.  Blue/Green deployments are when you have two identical production environments labeled `Blue` and `Green`.  At any given time, only one of those environments, for example, `Blue`, is live.  Deployment is done to the non-live environment, for example, `Green`, which is then verified.  After verification is complete, a switchover occurs, and the live environment becomes `Green`.  

![](https://i.octopus.com/docs/deployment-patterns/blue-green-deployments/images/3278250.png)

There are several advantages to doing this.  Rollbacks should be easy, only a matter of switching from blue to green or green to blue.  When a switchover does occur, it should be seamless as the code has already been running, there is no need to wait for it to compile or "warm-up."  And changes can be verified in production without having any customers hitting the code, making the deployment much less risky.  If something doesn't work, don't make the switch, try again at a later time.

## How Database Changes Add Complexity

Let's start with what seems like a simple example to see how database changes add complexity.  A new column `CustomerFullName` is added.  `CustomerFirstName` is a combination of `CustomerFirstName` and `CustomerLastName` columns.  In the UI, there used to be two columns, `First Name` and `Last Name`, but the decision was made to combine the two columns.  Some cultures have multiple names which do not fit into the `First Name` and `Last Name` we commonly see in the US.  A script was written to backfill `CustomerFullName` with `CustomerFirstName` and `CustomerLastName` during the deployment.  The `CustomerFirstName` and `CustomerLastName` columns will no longer be needed in the database; an additional script will be written to delete them.

In the example from the introduction, you saw what it looked like if we didn't have Blue/Green deployments, which is an after-hours deployment.  After hours deployments are done to avoid users accessing the application during the deployment.  Specifically, preventing people from accessing the application while the backfill script is running.

A very simplified deployment process for that change would look like this:

1. Disable access to users or turn off the website.
2. Run the script to add the column `CustomerFullName`.
3. Run the backfill script to populate `CustomerFullName`.
4. Run the script to remove `CustomerFirstName` and `CustomerLastName`.
5. Deploy the code to `Production`.
6. Verify the code in `Production`.
7. Profit.

Now let's see what that same change looks like with Blue/Green Deployments.  In this example, `Green` is live, and `Blue` is inactive.  Without Blue/Green deployments, the only worry was someone is using the application before everything is deployed and verified.  If the user gets an error indicating the column `CustomerFirstName` is missing during the deployment, no worries, they shouldn't be in the system at that point anyhow.  That gets flipped on its head; users are going to be running on the `Green` servers during deployment where data is getting queried and manipulated.  Users using the application during the deployment adds a whole slew of items to consider.

- To deploy to `Blue` and verify the changes `CustomerFullName` needs to be added to the database.  Verification isn't complete unless the backfill script populating the column correctly is tested.  So the schema change script and backfill script need to be run before deploying to `Blue.`  
- Customers will be using the application on `Green` so we cannot delete `CustomerFirstName` and `CustomerLastName` during this deployment.  Doing so will cause errors.  That will need to happen during the next deployment.  
- How long will the deployment and verification take?  During that time, how many new records will be added, or existing ones are updated?  The backfill script will have to be run again after the environments are swapped to pickup any new changes.  
- Can the backfill script run a second time and will it take the same amount of time?  Is it okay if the Customer Name is empty or blank during that time?  
- Is the application a SPA App ([Single Page Applications](https://en.wikipedia.org/wiki/Single-page_application))?  Will `Blue` start getting API requests from `Green` front-end code?  Some users could be sending API requests without the `CustomerFullName` column, just the `CustomerFirstName` and `CustomerLastName` column.  The code being deployed to `Blue` will need to take that into consideration and be able to handle that.
- Are there any stored procedures, specifically around inserting/updating data which need to be updated?  Will `Green` fail because the `CustomerFirstName` parameter will not be sent in?    

The deployment process for this change would look something like this:

1. Run the script to add the column `CustomerFullName`
2. Run the backfill script to populate `CustomerFullName`
3. Deploy code to `Blue` environment
4. Verify the code and database changes in the `Blue` environment
5. Swap the live environment from `Green` to `Blue`
6. Run the backfill script to populate `CustomerFullName`

As you can see, there are a lot of questions to answer when you allow users into an application during a deployment.  

Admittedly, that scenario is fairly complex.  Most database changes don't combine two columns into one and try to backfill the new column.  If that can be solved, then I would argue the majority of the scenarios are solved.  Which is why this section will focus on how to solve the scenario of combining two columns into one.  It will walk through each change, the questions to consider, and recommendations on how to solve them.

**Please Note:** These are recommendations only.  There is no way I can cover every possible change you can make to a database.  My goal is to provide you with something which you can then modify to meet your own needs.  

## Adding the New Columns

Keep it simple, add the new column `CustomerFullName` as a nullable column.  It takes most databases (SQL Server, Oracle) quite a bit of time to add a new non-nullable column with a default value.  This is because it has to update each record on the table.  Adding a nullable column only requires updating the table definition (metadata) which takes a few milliseconds.  

Avoid the following if at all possible:

- Rename an existing column, for example `CustomerLastName` to `CustomerFullName` for one reason or another.  Don't know why this was ever proposed, but I've seen it happen and I've seen it fail because running the rename sproc for a column doesn't rename the column in the stored procedures or code.  This will cause `Green` to start throwing errors when doing Blue/Green deployments.
- Reuse an existing column, for example `CustomerFirstName` now stores the full customer name.  That makes writing queries very confusing.  And while `Green` is active, `CustomerFirstName` will only contain the first name.  Once `Blue` goes active `CustomerFirstName` will start containing the full name.  It seems like it will save time, the code is already there to use that data, it is just a UI update.  That shortcut only causes confusion in the end.
- Add the column, backfill with data, then change the column to nonnullable.  It should work, except, `Green` isn't sending in data for that column.  Which leads to an error.  `Null` is the absence of a value.  It is not a bad thing.  

## Backfill the new column with data

As you saw earlier, the script to backfill the new column had to be run twice in our scenario.  Even then, there is the risk of the second run taking a while or getting missing data from API requests.  There are too many what ifs.  It is easy go a little overboard thinking of all the possible scenarios.  This is why I recommend taking the backfill script out of the equation.  In other words, but don't write scripts to backfill `CustomerFullName`.  Have the code backfill the data over time and handle it when no data is found.

There is not a set in stone solution on how to have the code accomplish backfilling of the `CustomerFullName` column.  Every application has its own rules about patterns and practices to follow.  I am going to suggest a couple of approaches, but use them as guidelines.  Follow your application's patterns and practices.  It is better to have consistency in the code than it is go to off and do something which you think is "correct."  

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

That works fine if all the properties are on the same model.  And that is the same model being returned by the data layer and sent as a response in an API call along with accepting the same model from the API.  In my experience, it takes a great deal of discipline along with some simple code for that to actually be the case.  The models and the database have to be a one to one match.  In my experience, that is rarely the case.  I've seen ApiModels, ViewModels, or some other phrase which indicates there is a difference in the model and the database.  

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

    public void InsertCustomer(Customer customer)
    {
        using (var conn = new SQLConnection(_connectionString))
        {            
            using (var command = new SQLCommand("dbo.usp_InsertCustomer", conn))
            {
                command.Parameters.AddWithValue("@CustomerFullName", customer.GetFullName());
            }                
        }
    }

    public void UpdateCustomer(Customer customer)
    {
        var existingCustomer = GetCustomerById(customer);
        
        using (var conn = new SQLConnection(_connectionString))
        {            
            using (var command = new SQLCommand("dbo.usp_UpdateCustomer", conn))
            {
                command.Parameters.AddWithValue("@CustomerFullName", customer.GetFullName());
            }                
        }
    }

    private DataTable GetCustomerById(Customer customer)
    {
        using (var conn = new SQLConnection(_connectionString))
        {            
            using (var command = new SQLCommand("dbo.usp_GetCustomerById", conn))
            {
                command.Parameters.AddWithValue("@CustomerId", customer.CustomerId);

                using (var adapter = new SQLAdapter(command))
                {   
                    var customerTable = new DataTable();           
                    adapter.Fill(customerTable);
                }
            }                
        } 
    }
}

```

I recommend this approach for a number of reasons.

1. All the timing concerns are taken care of.  There is no need to worry about running the backfill script multiple times to make sure a record isn't missed somehow.  No need to worry about how long it will take to finish running.  It will help make the transition from `Green` to `Blue` be as seamless as possible.
2. This change needs to happen anyway to prevent errors from happening.  While `Blue` is being deployed and verified users will be making database updates on `Green` after the first run of the backfill script.  If the person or automated process doing the verification on `Blue` happens to hit one of those records and the code wasn't updated then an error would occur. 
3. The code also needs to change in the event the application somehow gets a request without the `CustomerFullName` property populated.  Could come from a SPA application or for some other reason.  With or without a backfill script, setting that column in the database to `Null` after it has been populated would not be great.  But if it did happen the code would be able to handle it.  
4. Backfill scripts are SQL Scripts.  SQL is a great language for querying databases.  It is not a great language to write functional programming, integration tests, or unit tests.  The code should be the only place where the logic of backfilling the `CustomerFullName` column is located.  Having a backfill script AND the code handle the backfilling opens the door to errors.  Maybe you'll remember to change the backfill script but forget to change the code.  Or, an error is found by QA in the code and that is fixed but the backfill script is forgotten about.
5. Most database deployment tools, be it Redgate, DBUp, RoundhousE or SSDT don't have a great mechanism for running specific scripts multiple times.  If the tooling does have that functionality, or you implement your own mechanism for doing this, then guard clauses will have to be put in place around the script to make sure data isn't overwritten by accident.

## Stored Procedures and Views

I am a big believer in the database should be used as a data store only.  Keep as much logic as possible out of the database.  This includes stored procedures and views.  Business logic in the database includes, but not limited to, formatting, calculation, inclusion of IsNull checks, if/then statements, while statements, and filtering more than just by an Id.  

It is far too easy to end up copying/pasting that logic between stored procedures.  Imagine the application had two stored procedures:

- usp_GetCustomerById
- usp_GetAllCustomers

Fundamentally they do the same thing, get customers from the database.  The CustomerFullName is a new column but it is null.  Rather than having a function in the code check to see if `CustomerFullName` is null, that logic is placed in the database.

```SQL
Select CustomerFirstName,
       CustomerLastName,
       IsNull(CustomerFullName, CustomerFirstName + ' ' + CustomerLastName) as CustomerFullName
from dbo.Customer
```

That only hides bad or missing data; it doesn't solve bad or missing data.  It is a one-liner, and oh so easy to copy paste to other stored procedures.  In this example, it is only one other stored procedure.  Then one day someone adds a new stored procedure and they forget to include that.  But, it has been long enough the `CustomerFullName` column is populated for 90% of the records.  It is only a matter of time before an error occurs.  And when it comes time to remove `CustomerFirstName` and `CustomerLastName` from the database they could miss that IsNullCheck.  Imagine if a stored procedure had 50 columns.  `CustomerFullName` was added towards the bottom of the list while `CustomerFirstName` and `CustomerLastName` were towards the top of the list. 

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

Update dbo.Customer
    set CustomerFirstName = @CustomerFirstName,
        CustomerLastName = @CustomerLastName,
        CustomerFullName = @CustomerFullName
where CustomerId = @CustomerId
```

While it is possible to run an insert command without specifying the columns, don't do that.  Specify all the columns.  It only causes headaches down the line.

```SQL
Insert into dbo.Customer (CustomerFirstName, CustomerLastName, CustomerFullName)
    value (@CustomerFirstName, @CustomerLastName, @CustomerFullName)


select SCOPE_IDENTITY()  
```

I've worked on applications where views were created for other teams, such as Business Intelligence (BI) teams, to suck in data from the application's database into their data warehouse databases.  The views were created because the other teams are not experts on the application; they don't know the schema, and their import process shouldn't be tied directly to the schema.  Views provide a layer of abstraction for them.  This is typically when I see the "no business logic in databases" rule get broken.  The data needs to be presented in such a way to make it easy to import into a data warehouse.  For example, when I was working on a loan origination system the BI team said they wanted the view to show "loans with their customers."  A loan could have 1 to N number of customers, so I did an inner join on the loan and customers table and called it good.  The BI team then asked why they kept getting duplicate loans.  What they really wanted was "loans with the primary customer," which required me to add a `IsPrimary = 1` filter into the view.  

My advice is still to provide a view which gives the data in its rawest form as possible.  In my example for the loan origination system, I should've added the column `IsPrimary` to the view and let the BI team filter out what they didn't want.  For the `CustomerFullName` example, it would be all three columns, `CustomerFirstName`, `CustomerLastName`, and `CustomerFullName`.  Communicate to them and let them know `CustomerFirstName` and `CustomerLastName` is going to be dropped, now is the time to start updating their reports and cubes and all that other fun stuff they create.  

I realize moving business logic out of the database is quite the change.  If you do decide to make that switch, be pragmatic about it.  Don't try to change everything at once.  The code and the database is working fine now.  This section is about changes to the database going forward.  My rule of thumb is to leave the database and code in a better place than when I found it.  Meaning, if I am making a change to a column and I come across a business rule in the database for that column, then I fix it at that time.  I don't tear through all the code and start making mass changes.  That is a surefire way to end up doing an emergency deployment on a weekend.

## Removing Columns

At some point, the `CustomerFirstName` and `CustomerLastName` columns will be removed.  Blue/Green deployments make that more complex as well.  Assume `CustomerFullName` has been deployed to `Blue` and it is active.  Running a script to delete `CustomerFirstName` and `CustomerLastName` from all the tables, views, functions and stored procedures will cause errors in the code `Blue` is running.   The code running on `Blue` is using the `CustomerFirstName` and `CustomerLastName` fields to populate `CustomerFullName` when `CustomerFullName` is null.  

How can the `CustomerFirstName` and `CustomerLastName` columns get removed then?  To answer that, consider what a deployment would look like if an outage was taken.

1. Run a backfill script to take care of the remaining records.
2. Remove references in the frontend code to `CustomerFirstName` and `CustomerLastName`.
3. Remove references to any other applications which might use `CustomerFirstName` and `CustomerLastName`.
4. Remove references in the backend code to `CustomerFirstName` and `CustomerLastName`.
5. Delete `CustomerFirstName` and `CustomerLastName` 

