---
title: Automated Database Deployments Iteration Zero
description: Automated Database Deployments Iteration Zero
author: bob.walker@octopus.com
visibility: public
published: 2018-06-18
tags:
 - Database Deployments
---

The [previous post](/blog/2018-06/automated-database-deployments-series-kickoff.md) of this series walked-through a real-world case of what happens after automating database deployments.  Now you are ready to get started and dive right in.  Hold on for a second.  Friction is the enemy of change.  The higher the friction, the slower the adoption.  This is especially true for automating database deployments.  As you get started you will run into friction from areas you didn't expect.  Before diving into it is important to have some initial discussions and decisions.  A sort of iteration zero phase to this whole process.  

## Deployment Approaches

Deploying databases can be very complex.  This is why Octopus Deploy doesn't have a built-in step for deploying databases.  Instead, Octopus Deploy integrates with third-party tools to handle this functionality.  When evaluating the available third-party tools out there you will soon discover they handle their deployments in one of two ways.

As a team or organization, you will need to decide which approach you want to take.  Each approach has its pros and cons.

### #1 Model-Driven Approach
With the model-driven approach, the desired state of the database is defined.  The state is saved into source control.  During the deployment, the tool will compare the desired state with the deployment target and generate a delta script.

The database desired state is stored as individual create scripts in source control.  When making changes those create scripts will be updated.

#### Model-Driven Pros
The model-driven approach is very easy to pick up and understand.  Often times the tooling integrates right with the IDE of your choice.  For example, Redgate's tooling integrates with SQL Server Management Studio.  While Microsoft's SSDT tooling integrates with Visual Studio.  The changes to the schema are made using the IDE.  After the changes are made the plug-in for the IDE will take over.  It will run a comparison to determine the difference between the change and what is currently in source control.  Then it will make the change to the necessary script on the file system.

All the file system interaction happens behind the scenes.  The tool keeps track of the changes being made.  This allows you to focus on making the database changes and testing them.  After you are done testing them then you can save the change script to source control.

Having individual create scripts makes it very easy to see the desired state of a specific object.  All you have to do is go to the file in source control and open it up.  The same goes for the history of a specific object.  All you have to do is go to the file in source control and view the history.

Finally, some of the tools allow you to mark a table as "static data."  The data itself is checked into source control.  During deployments, the tool will check the data in the destination table.  If the destination table is missing data or the data is incorrect the delta script will include data change T-SQL statements. 

#### Model-Driven Cons
A unique delta script will be generated during deployment per environment.  This is because a change could have been applied to one environment (dev) but not a higher environment (pre-production or production).  This also means the tooling tends to be much more complex because it has to know how to do these comparisons.  Every once in a while the tool will generate a delta script where an unexpected change is included.  Especially if permissions are not set correctly.

Typically the tooling will want to control everything about the database.  From the tables to the schemas to the users.  Often times you want to create a unique user per environment.  So you have to configure the tool to exclude certain pieces of the database, such as users, from the deployment process.

As smart as the tooling is, it has a difficult time handling more complex changes.  For example, moving a column from one table to another.  The tool doesn't know that is your intention.  What it will do is drop the column from the old table and create a new empty column in the new table.  Typically the tooling will include some sort of migration script ability where you can write your own migration scripts.  But often times that has its own set of rules.  

The lack of control can be a bit of a burden sometimes.  What I ended up doing is adding two folders to my process.  Pre-deployment and post-deployment.  I added a pre-deployment and post-deployment step in Octopus Deploy to run the scripts I wanted to run.  This allowed me to get a little more control, but it was a bit odd to follow one process for one scenario and another process for another scenario.

### #2 Change-Driven Approach
A change-driven approach is where all the necessary delta scripts are handwritten.  Those scripts are checked into source control.  During the deployment, the tool will look to see which change scripts have not been run on the destination database and run them in a specific order.  

#### Change-Driven Pros
With the change-driven approach, you have complete control over all the scripts.  When deploying a change you know exactly what script is going to run.  Complex changes are much easier to deal with, you just need to write the script and save it to source control.  In addition, it is much easier to exclude items from deployments.  Just don't include the script for items you want to exclude.

#### Change-Driven Cons
One of the advantages of the model-driven approach ensures the entire destination database matches the desired state.  Not so with the change-driven approach.  A new table could be added to the destination database outside of the process.  Everyone who has permissions to change the database has to be on board and using the process.  Otherwise, the delta between the environments could be large.

It is much, much harder to see the history of a specific object like a table or a stored procedure.  Instead of going to a single file and viewing the history you are required to do a search to find all the files where the object was changed.  Depending on the number of table changes going on, it could be easy to miss a key change and not even know it.  

Finally, a lot of developers are not expert SQL Developers.  They are used to using SQL Server Management Studio UI to create tables and indexes.  They don't know how to write a lot of the changes being made by hand.  It takes a lot of practice to get the T-SQL syntax memorized.  Having to handwrite all the of the possible change scripts can be very time-consuming.  

### Picking an approach

The right approach is very subjective.  I can't say with 100% certainty which approach will work right for you.

The model-driven approach works best when any of the following apply:

- Early stages of a project with lots of churn on the database
- Multiple people/teams are changing the database
- First getting into automating database deployments
- Mature databases with very little changes
- Want to force developers to follow the process automatically (if a change is made to a destination database and it is not checked in then it will get deleted, that only needs to happen once)
- Inexperience at making complex T-SQL Statements

The change-driven approach works best when

- A disciplined team which always follows the process
- Have the experience to make complex database changes
- Keep bumping into restrictions imposed by the model-driven approach
- Want the most possible control over the process

## Moving To Dedicated Databases

Regardless of the tool you pick, I recommend moving the developers to dedicated databases.  The typical approach for dedicated databases is to install SQL Server on the developer's laptop.  This provides many advantages.

1. You can try out risky changes without having to worry about affecting anybody else.  If you break something it is only one person affected.
2. Support for branching.  With a shared model there is only one database.  If a breaking database change was made without a corresponding code change then it would stop everybody from working.  Now, all changes can be made and checked in on a branch and deployed at the same time.

## Communication Between Team Members for Changes

Automating database deployments introduces an interesting challenge.  Prior to the automation, a developer could make a change on a shared database and everyone would see it right away.  That made it a little trickier to step on each other's toes.  But with automated database deployments and dedicated databases, the risk is much higher of that.  A change to a table could be made by developer A in their branch.  The same table could be changed by developer B in their branch.  Both of those changes could conflict with one another.  What this means is there will need to be a mechanism to let others know if what table changes are being made.  It could something as simple as a slack message or discuss it during a daily stand-up.  The important thing is to make sure everyone is on the same page.  

## Permissions
In the past, DBAs were the ones who ran the scripts in production because they were the only ones with the necessary permissions.  In my experience, DBAs would also take the opportunity to glance over the script to make sure it wasn't about to do something really wrong.  Say, delete the database it is trying to deploy to.

Now an automated process will be doing that work.  That can be really scary.  If something goes wrong and the data is lost it could be very difficult to get it back.  In addition, a script could be added to grant a random user sysadmin privileges in production.

Trust is key when building out this process.  The best way to build trust is to initially limit what the automated database deployment process can do.  In addition, have the process use Octopus Deploy's artifact functionality to create a file containing all the database changes about to be applied.  Finally, create a unique account per environment.  This will ensure the tentacle deploying the database changes to dev doesn't have rights to do the same in production.

When adopting automated database deployments for the enterprise I recommend the following permissions for the account making the changes:

- Database Permissions
    - db_ddladmin -> can run any Data Definition Language (DDL) command in a database.
    - db_datareader -> can read all the data from all user tables
    - db_datawriter -> can add, delete, or change data from all user tables
    - db_backupoperator -> can backup the database
    - Can View Any Definition

That is the most restrictive permission possible while still automating deployments.  With that, no new database users can be created.  No new schemas can be created.  Users cannot be added to roles.

As time moves on and trust has been built the restrictions can be lifted.  The next level of permissions would be:

- Database Permissions
    - db_ddladmin -> can run any Data Definition Language (DDL) command in a database.
    - db_datareader -> can read all the data from all user tables
    - db_datawriter -> can add, delete, or change data from all user tables
    - db_backupoperator -> can backup the database
    - db_securityadmin -> modify role membership and manage permissions
    - db_accessadmin -> can add or remove access to the database for logins
    - Can View Any Definition

That gives the ability to manage users at the database level as well as create schemas.  The user still has to exist, but now it is possible to add them to the database using a SQL Command through Octopus Deploy.

Microsoft has provided a handy chart to see what permissions are available to what role.  

![](permissions-of-database-roles.png)

If the time comes and you are ready to fully automate everything here are the permissions to do so:

- Server Permissions
    - dbcreator -> ability to create new databases
    - securityadmin -> ability to create new users and grant them permissions (you will need a check in place to ensure it doesn't grant random people sysadmin roles)
- Database Permissions
    - db_ddladmin -> can run any Data Definition Language (DDL) command in a database.
    - db_datareader -> can read all the data from all user tables
    - db_datawriter -> can add, delete, or change data from all user tables
    - db_backupoperator -> can backup the database
    - db_securityadmin -> modify role membership and manage permissions
    - db_accessadmin -> can add or remove access to the database for logins
    - Can View Any Definition

This will give automated database process the ability to create databases and users.  Perfect for instances where you need to stand up a new database on the fly.  A lot of trust will need to be in the process before getting to this point.  Don't be surprised if you get a lot of pushback when you propose it.

## Where to install the tentacle

You do not want to install the tentacle directly on the SQL Server.  Often times a SQL Server will be a cluster or a high-availability group.  And you do not want to have the tentacles being used to deploy windows services or IIS web applications handle database deployments.  Those tentacles could be in a DMZ.  The tentacle should be running under a specific service account with the necessary permissions to perform the deployment.  Finally, the majority of the tools make use of port 1433 and simply run a series of T-SQL Scripts.  The tentacle can be installed on any machine as long as it has a connection to the database.  For those reasons, I recommend you make use of a jump box which sits between Octopus Deploy and SQL Server.  

[Our documentation](https://octopus.com/docs/deployment-examples/sql-server-databases#SQLServerdatabases-Tentacles) does a good job of covering this topic (with included diagrams!), please refer to that for more information.

## Conclusion

At first glance, this feels like a lot of prep-work.  The important thing to remember is these are guidelines.  Don't spend weeks discussing and debating.  Come up with an initial plan and iterate through it.  At the start of this process, we spent about two days discussing and working through an initial plan.  My recommendation for rolling this out is:

1. Discuss the items above and come up with an initial plan
2. Establish a pilot team to iterate through any issues
3. Wait for the pilot team to have several successful deployments to production
4. Roll this out to other projects, one at a time.  Don't be surprised with each time you roll it out to a new project you come across something new which will require a change of some sort.  
5. Don't be afraid to reach out and ask experts.  There are several companies out there which provide consulting services which are happy to help.