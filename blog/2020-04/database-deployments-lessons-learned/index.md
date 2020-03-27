---
title: Lessons learned while implementing database deployments
description: Exploring the pitfalls you may encounter when deploying database changes
visibility: private
published: 2999-01-01
metaImage:
bannerImage:
tags:
 - DevOps
---


The following post is an account of the different issues I (we?) encountered while implementing automated database deployments.

## Tightly coupled databases
It happens and most of us have seen it, two different systems accessing each others database.  They need information from each other and the quickest way to get it is to reach in and grab it.  While it may start off with a single table or view, as each system grows, the coupling gets tighter and tighter.  With this coupling, comes a multitude of problems.

### Breaking changes
At my last job, I can recall an emergency meeting following the deployment of an application (we'll refer to as A) which caused a different application (this one we'll call B) to start failing.  Tempers were high and fingers from both teams were firmly pointed at the other.  At the time, my role was Configuration Manager and in charge of deployments, so I was facilitating the meeting.  What had happened was the deployment introduced changes to the database schema of A which was causing B to fail.  The most troubling part about the situation was that A had no idea that B was pulling data, so naturally A didn't understand why B was so angry.  My first suggestion was for A to create an API or service for B to use.  Everyone agreed that was the correct approach but since B was down, the implemented solution was a view from A that supplied the data to B in the expected format.  The API or service was never developed.

### Circular dependencies
Some time later I was evaluating different automated database deployment methods.  I created a SQL Server Database Project for Database A.  As it turned out, database A was grabbing data from another database, we'll call it C.  This external dependency required me to create another SQL Server Database Project for database C so that database A project could use it as a reference for compiling a DACPAC.  After creating and compiling the Databaes Project for database C, I once again attempted to compile Database Project A and encountered another dependency, database A was now reaching into database B and would not compile without a reference to a compiled database B.  With a heavy sigh, I created a Database Project for database B.  After resolving a dependency on database C, I encountered a compilation error that database B needed a reference to a compiled database A.

## Redgate SQL Source Control and three part naming convention
My organization settled on using [Redgate SQL Source Control](https://www.red-gate.com/products/sql-development/sql-source-control/) as the chosen method for maintaining database schema.  It had been practice for quite some time that when writing views or stored procedures, references to database objects would always use the three part naming convention, database.schema.object.  For Redgate SQL Source Control, this caused an issue.  When specifying the database in the object reference, it thought it was an external database call and didn't take it into account when determining the build order of objects.  This sometimes caused views to attempt to be built before the underlying table existed.

## Constraints without names
Microsoft SQL Server can be fairly forgiving, sometimes to it's own detrament.  One issue that we ran into was not giving a default constraint a name.  The following is valid SQL syntax for creating a table with a default constraint:

```
CREATE TABLE Persons
(
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255) DEFAULT 'Sandnes'
)
```

In this instance, Microsoft SQL Server will "help" you and give the constraint a GUID based name.

[INSERT SCREENSHOT HERE]

The issue that this presents is when using state-based (aka model-based) deployment methods, the "scratch" database created from the scripts folder uses the same script.  The resulting table will be created successfully, however, the table's constraint name will be a completely different GUID.  When the post deployment check to ensure the target database is in the desired state executes, the deployment fails because the constraints don't have the same name.

## Mixing deployment technologies
At my previous organization, we used Redgate SQL Source Control for schema changes and DBUp for data changes.  Mixing technologies in such a way came with a unique issue.

All of the migrations-based deployment technologies ([DBup](https://dbup.readthedocs.io/en/latest/), [Flyway](https://flywaydb.org), and [RoundhousE](https://github.com/chucknorris/roundhouse)) create a table in the target database to keep track of which scripts have already been executed so they're not run again.  This table is the issue we ran into.  The state-based approach to database deployments makes the target database look like the desired state.  During the deployment process, if the state-based approach finds objects that do not exist in the desired state, those objects are assumed as not needed and are deleted.  This is what happened to the `schemaversions` table that DBUp created.  Each time we ran a deployment, the Redgate step ran first and deleted the schemaversions table.  Then, DBUp ran. It detected the schemaversions was missing, assumed it was the first time it's ever been run, recreated schemaversions and ran all scripts.  Once we made the Redate SQL Source control project aware of schemaversions, things ran as desired.