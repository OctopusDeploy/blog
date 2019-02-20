One of the most exciting aspects of database deployments is the number of tools released in the last 10 years.  Looking at my [previous posts](https://octopus.com/blog/tag/Database%20Deployments) on this topic and I have shown a clear bias towards Redgate's tooling.  I do have a bit of a bias towards their tools.  I'm a [friend of Redgate](https://www.red-gate.com/hub/events/friends-of-rg/friend/BobWalker) for a reason.  

I'm going to switch gears a bit and focus on a different tool for this post, [DbUp](https://dbup.readthedocs.io/en/latest/).  DbUp is a free, open-source tool, which we use here at Octopus Deploy for our database deployments.  Anytime you install or upgrade Octopus Deploy, DbUp is the one who runs the scripts to update your database.  Our founder, Paul Stovell, wrote a [blog post back](https://github.com/DbUp/DbUp/graphs/contributors) in 2012 on how to use DBUp to deploy to a SQL Server.  For the most part, that blog post still holds up today.  Except for the outdated images.  Want to see what Octopus Deploy looked like years ago?  Check out that post.  

This post is an update to that old post.  I am going to walk through some of the new features recently added to DbUp and create a process to use DbUp for production deployments.  It even includes a review step for a DBA to approve!  Read on!

!toc

## Changes to DbUp

At it's core, DbUp is a script runner.  Each change made to the database is done by a script.

- Script001_AddTableA.sql
- Script002_AddColumnTestToTableA.sql
- Script003_AddColumnTestAgainToTableA.sql

DbUp is then run through a console application you write yourself.  You control which options to use.  The amount of code needed is pretty small.

```C
static int Main(string[] args)
{
    var connectionString =
        args.FirstOrDefault()
        ?? "Server=(local)\\SqlExpress; Database=MyApp; Trusted_connection=true";

    var upgrader =
        DeployChanges.To
            .SqlDatabase(connectionString)
            .WithScriptsEmbeddedInAssembly(Assembly.GetExecutingAssembly())
            .LogToConsole()
            .Build();

    var result = upgrader.PerformUpgrade();

    if (!result.Successful)
    {
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine(result.Error);
        Console.ResetColor();

        return -1;
        }
    }

    Console.ForegroundColor = ConsoleColor.Green;
    Console.WriteLine("Success!");
    Console.ResetColor();
    return 0;
}
```

You bundle up those scripts and tell DbUp to run them.  It compares that list against a list stored in the destination database.  Any scripts not in that destination's database list will be ran.  The scripts are run in alphabetical order.  The results of each script is displayed.  Very simple to implement.  Very simple to understand.  

![](dbup-output.png)

And that works great...when deploying to a development or test environment.  A lot of companies I talk to prefer a DBA approve scripts prior to going to production.  And, maybe a staging or pre-production environment as well.  Especially starting out.  When the trust in the process is low.  

### HTML Report

Migration scripts are a double edged sword.  Just like memory management in C++.  You have total control, which gives you a lot of power.  But it is also easy to mess up.  It all depends on the type of change being done and the SQL Skills of the writer.  The trust of the DBAs will be very low when have inexperienced C# developers writing these migration scripts.  Especially if you are starting down the automated database deployment path.  

Recently I added in the the functionality to DbUp to generate an HTML report.  It is an extension method where you give it the path of the report you wish to generate.  That means this section goes from:

```C
var result = upgrader.PerformUpgrade();

if (!result.Successful)
{
    Console.ForegroundColor = ConsoleColor.Red;
    Console.WriteLine(result.Error);
    Console.ResetColor();

    return -1;
    }
}
```

To:

```C
// --generateReport is the name of the example argument.  You can call it anything
if (args.Any(a => "--generateReport".Equals(a, StringComparison.InvariantCultureIgnoreCase))) 
{
    upgrader.GenerateUpgradeHtmlReport("C:\\DeploymentLocation\\UpgradeReport.html");
}
else
{
    var result = upgrader.PerformUpgrade();

    if (!result.Successful)
    {
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine(result.Error);
        Console.ResetColor();
        return -1;
    }
}
```

This will generate a report containing all the scripts which are going to be run.

![](db-htmlreport.png)

### Always Run Script and Script Grouping

By default DbUp will run a script once.  Majority of the time that is fine.  There are times where it would be nice to always run a script.  An example would be a post deployment script to refresh all the views.  Or a script to rebuild all the indexes and regenerate stats.  You don't want to write a new script for each deployment.

Another feature I added to DbUp is the ability to mark a group of scripts as `always run` and provide a run group.  

```C
var upgradeEngineBuilder = DeployChanges.To
    .SqlDatabase(connectionString, null) //null or "" for default schema for user
    .WithScriptsEmbeddedInAssembly(Assembly.GetExecutingAssembly(), script => script.StartsWith("SampleApplication.PreDeployment."), new SqlScriptOptions { ScriptType = ScriptType.RunAlways, RunGroupOrder = 1})
    .WithScriptsEmbeddedInAssembly(Assembly.GetExecutingAssembly(), script => script.StartsWith("SampleApplication.Scripts."), new SqlScriptOptions { ScriptType = ScriptType.RunOnce, RunGroupOrder = 2})
    .WithScriptsEmbeddedInAssembly(Assembly.GetExecutingAssembly(), script => script.StartsWith("SampleApplication.PostDeployment."), new SqlScriptOptions { ScriptType = ScriptType.RunAlways, RunGroupOrder = 3})
    .LogToConsole();

var upgrader = upgradeEngineBuilder.Build();

var result = upgrader.PerformUpgrade();

// Display the result
if (result.Successful)
{
    Console.ForegroundColor = ConsoleColor.Green;
    Console.WriteLine("Success!");
}
else
{
    Console.ForegroundColor = ConsoleColor.Red;
    Console.WriteLine(result.Error);
    Console.WriteLine("Failed!");
}
```

## Create the DbUp Console Application

With these new features we are going to put together a .NET Core DbUp console application to deploy to SQL Server.  Then we will put together a process in Octopus Deploy for it to run that console application.  

**Please Note:** I chose .NET Core over .NET Framework because it could be built and run anywhere.  DbUp is a .NET Standard library.  It will work just as great in a .NET Framework application.

Alright, let's fire up our IDE of choice and create an .NET Core Console Application.

**Please Note:** I am using JetBrain's Rider to build this console application.  I prefer it over Visual Studio.

Alright, we have the console created.  Now we need to bring in the DbUp NuGet packages.  Let's go to our NuGet Package Manager.

![](rider-managenugetpackagesselection.png)

Next we want to select the DbUp-SqlServer package.  This includes the core package as well as the necessary code to deploy to SQL Server.  If you want to deploy to PostgreSQL, MySQL, Oracle, or Sqlite you would pick those.  

![](rider-selectingdbuppackage.png)

The console application needs some scripts to deploy.  I'm going to add three folders and populate them with some script files.  

![](rider-createfolderswithsamplescripts.png)

By default .NET will not include those scripts files when the console application is built.  We want to include those script files as embedded resources.  Thankfully we can easily add a reference those files by including this code in the `.csproj` file.

```XML
    <ItemGroup>
        <EmbeddedResource Include="BeforeDeploymentScripts\*.sql" />
        <EmbeddedResource Include="DeploymentScripts\*.sql" />
        <EmbeddedResource Include="PostDeploymentScripts\*.sql" />
    </ItemGroup>

```

The entire file would then look like.

![](rider-sampleprojectcsprojfile.png)

