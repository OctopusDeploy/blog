---
title: Running SQL Server Developer in a Docker Container
description: SQL Server can run in a docker container, how can developers leverage that on their development machines?
author: bob.walker@octopus.com
visibility: public
published: 2019-09-30
metaImage: 
bannerImage: 
tags:
 - Developer Machine Setup
 - Database Deployments
---

For quite some time, SQL Server has been a docker image.  Running SQL Server on Docker could solve two of the major headaches I have with running SQL Server Developer edition as a Windows Service, which I covered how to automate the installation in my [previous article](https://octopus.com/blog/automate-sql-server-install). Running SQL Server locally is a double-edge sword.  It's nice to have a SQL Server I have total control over; I can make mistakes, try out new things, and no one else is blocked.  However, that SQL Server is running all the time which consumes resources, and I am responsible for patching it.  When I was a full-time developer, I needed to run SQL Server all the time.  With my new role, I only want SQL Server running some of the time, and I don't want to worry about having to patch it.  Can running SQL Server on Docker solve that?  In this article I answer that question, along with how I accomplished configuring SQL Server to run on Docker.

!toc

## The Scenario

In a couple of months, I will be speaking at [TechBash 2019](https://techbash.com/sessions). The presentation will cover how my automated database deployment process progressed from being purely on TeamCity to being TeamCity + Octopus Deploy.  I have all that running on VMs hosted on a hypervisors in my office, but I need to migrate that setup to my laptop.  However, I don't need or want all of that running all the time.  Only when I am prepping or practicing my demo.  In other words, I want to spin up and down an entire CI/CD pipeline using Docker.

## A Brief Introduction to Docker

To ensure we are talking the same lingo, I want to go over the basics of Docker.  Think of Docker as an virtualization host, similar to Hyper-V or VMWare.  But instead of VMs, in Docker hosts containers, which run applications.  Containers store all the application's binaries and dependencies in a completely isolated environment.  Unlike VMs, which require the full OS, only the bare essentials needed to run the application are included in the container.  

Below is a list of [Docker nomenclature](https://docs.docker.com/engine/docker-overview/) which will be used in this article:

- **Images:** An image is a read-only template for creating the Docker Container.  I like to think of them like an ISO file.  You can use images provided by the community via [Docker Hub](https://hub.docker.com/), or you can create images yourself.  Images are typically built upon other images.  Think of it like inheritance in source code.  Object C is a child object of Object B, which is a child object of Object A.  Typically, you'd create custom images to host your applications.  This article will use images found in the Docker Hub, it will not go into how to create your own image (there are tons of articles which do that).  
- **Container:** A container is a runnable instance of an image with configuration options specified.  For this article, we are going to be taking the SQL Server, TeamCity, and Octopus Deploy images located on Docker Hub and run them in containers.  
- **Volumes:** By default, containers are considered stateless, any changes made to the container while it is running will not persist when it shuts down.  If you were to create a database in a container hosting SQL Server it would not be there the next time it starts up.  Volumes allow for data to be persisted by accessing the host's file system.  This article will be making use of volumes to store all the configuration information for the CI/CD pipeline.
- **Docker Compose:** Docker Compose makes it easy to define multi-container Docker applications.  It uses YAML to define all the containers needed for the application.  This article will be using Docker Compose to spin up and down the CI/CD pipeline on my local machine.

## Prep Work

Some prep work needs to be done before jumping in and writing scripts.  My work laptop is running Windows 10 Professional.  Here are the steps I took to get Docker installed and ready for use on my laptop.

### Enable CPU Virtualization
When it comes down to it, Docker is a virtualization host.  Just like any other virtualization host, the CPU has to support virtualization and that has to be enabled.  Typically virtualization is enabled in the BIOS.  Which means you'll have to do a Google search on how to to enable it in your computer manufacturer's BIOS.  Intel calls their virtualization technology [Intel VT](https://www.intel.com/content/www/us/en/virtualization/virtualization-technology/intel-virtualization-technology.html), along with Intel VTx.  AMD calls their virtualization technology [AMD V](https://www.amd.com/en/technologies/virtualization), sometimes you'll see it called VDI or SVM.  

### Installing Docker for Windows
After the quick BIOS update, it is time to install Docker for Windows.  I will be using Docker for Windows instead of Docker on Windows.  Docker on Windows only includes the base Docker engine.  I want to leverage Docker Compose and at some point in the future (aka not for this post), the Docker CLI.  The whole process to install Docker for Windows is [nicely documentation](https://docs.docker.com/docker-for-windows/install/).  No need to repeat that.

One thing to note, if you don't have Hyper-V enabled, the installer will enable it for you.  That will require a restart of the computer.  

### Configuring Shared Drives
Now I need to share my C:\ drive with Docker.  I'm doing this so I can mount a volume in the container to keep the data persisted.  All of that data will be stored on my computer in C:\Docker\Volumes.

![](docker-shared-folders.png)

//TODO: This looks like it isn't support for Windows Containers, need to test

## Running SQL Server Developer Container for the first time
Right now, Octopus Deploy has to run in a Windows based container.  That should hopefully change with our port to .NET core and the fact Octopus Cloud v2 is running on Linux containers.  But until that time, everything in my CI/CD pipeline will run as Windows containers.  

One downside of running a Windows container (aside from the space overhead), is anti-virus products can and will block them from downloading.  This is because of how Docker stores images on the Windows file system.  Essentially, another folder called Windows will appear in a random location.  When anti-virus scanners see that they freak out.  Make sure you are using the latest version of your anti-virus of choice.

![](extracted-docker-files-windows.png)

I want to start simple and then go complex.  The easiest thing to do is to get the docker image downloaded.  

```PowerShell
docker pull microsoft/mssql-server-windows-developer:latest
```

I like to specify latest to ensure I am always on the latest version of SQL Server.  I do this by specifying `:latest` in the command.  If that isn't included it will use the default, which is typically latest, but not always.  

Brew up some tea or coffee and sit back, because this might take a while to complete.

![](docker-download-sql-server.png)

Now that the image is downloaded, let's get it fired up and run some SQL scripts.  Please make note of the `--name` parameter being sent in.  This will make it easier later when we need to figure out how to connect to it.  In addition, we will be setting the port to the default SQL Server port, 1433.  

```PowerShell
docker run --name SQLServer -d -p 1433:1433 -e sa_password=Password_01 -e ACCEPT_EULA=Y microsoft/mssql-server-windows-developer
```

![](docker-run-sql-server-developer.png)

The container is now started, now let's connect to it via SSMS.  To do that, we will need to get the IP Address of the container, which we can retrieve using this command.

```PowerShell
$docker = docker inspect SQLServer | convertfrom-json
$docker[0].NetworkSettings.Networks.nat.IpAddress
```

![](docker-get-ip-address-of-sql-server.png)

With that IP Address, we can connect to SQL Server.

![](ssms-connect-to-sql-server.png)

Typing in the `sa` as the username and the password defined above and we are in.

![](ssms-connected-to-docker-image.png)

Just like regular SQL Server, everything works as expected.  I can create a database and tables without any issue.

![](create-table-inside-instance.png)

Let's see what happens when we shut down this instance, remove it, and start up a fresh one.

```PowerShell
docker stop SQLServer
docker rm SQLServer
docker run --name SQLServer -d -p 1433:1433 -e sa_password=Password_01 -e ACCEPT_EULA=Y microsoft/mssql-server-windows-developer
$docker = docker inspect SQLServer | convertfrom-json
$docker[0].NetworkSettings.Networks.nat.IpAddress
```

Note that the container has a new IP Address associated with it.

![](restart-docker-container.png)

And as expected, the databases were all deleted.  

![](restart-container-no-new-databases.png)

But you'll notice that I am not only running the `stop` command, I am also running the `rm` command, which is short for remove.  Let's see what happens when I change the command to this:

```PowerShell
docker stop SQLServer
docker start SQLServer
$docker = docker inspect SQLServer | convertfrom-json
$docker[0].NetworkSettings.Networks.nat.IpAddress
```

The IP Address changes when I do that.

![](restart-docker-container-no-remove.png)

The database still exists after I just do a simple restart.

![](test-database-after-restart-only.png)

The answer is simple then, just stop and start the same image over and over again.  Well not so fast.  Containers are immutable.  That means when an update for SQL Server (or Octopus Deploy or TeamCity) is released then I have to destroy the container and create a new one.  As you saw, once I destroyed a container the database created in the container were also destroyed.  That is not ideal.  Especially, if we want to connect TeamCity and OctopusDeploy to it.

## Persisting Data with Volumes

I need to store all the database files used in a container in a volume.  That volume will be pointed at `C:\Docker \Volumes\SQLServer`. There are [many](https://blog.sixeyed.com/docker-volumes-on-windows-the-case-of-the-g-drive/), [many articles](https://github.com/docker/labs/blob/master/windows/sql-server/part-3.md) about [Docker Volumes](https://docs.docker.com/storage/volumes/).  THe TL;DR; is add the `--volume` switch to `docker run` to add a volume where we can persist data.  But to do that we first need to destroy the container one more time.

```PowerShell
docker stop SQLServer
docker rm SQLServer
docker run --name SQLServer -d -p 1433:1433 --volume c:\Docker\Volumes\SQLServer:c:\SQLData -e sa_password=Password_01 -e ACCEPT_EULA=Y microsoft/mssql-server-windows-developer
$docker = docker inspect SQLServer | convertfrom-json
$docker[0].NetworkSettings.Networks.nat.IpAddress
```

When I connect to that SQL Server instance, all my database create commands need to specify `C:\SQLData\` as the directory for the data.  I know I will need a database for Octopus Deploy and another for TeamCity, so let's get those created.

```SQL
CREATE DATABASE [OctopusDeploy]
 CONTAINMENT = NONE
 ON  PRIMARY 
( NAME = N'OctopusDeploy', FILENAME = N'C:\SQLData\OctopusDeploy.mdf' , SIZE = 8192KB , FILEGROWTH = 65536KB )
 LOG ON 
( NAME = N'OctopusDeploy_log', FILENAME = N'C:\SQLData\OctopusDeploy_log.ldf' , SIZE = 8192KB , FILEGROWTH = 65536KB )
GO
CREATE DATABASE [TeamCity]
 CONTAINMENT = NONE
 ON  PRIMARY 
( NAME = N'TeamCity', FILENAME = N'C:\SQLData\TeamCity.mdf' , SIZE = 8192KB , FILEGROWTH = 65536KB )
 LOG ON 
( NAME = N'TeamCity_log', FILENAME = N'C:\SQLData\TeamCity_log.ldf' , SIZE = 8192KB , FILEGROWTH = 65536KB )
GO
```

The databases were created successfully.

![](create-teamcity-octopus-databases.png)

And they now show up in the directory on the host system.

![](sql-server-volume.png)

Now that those databases exist, we can pass in the names and paths of those databases next time we create that container using the `attach_dbs` environment variable.

```PowerShell
$attachDbs = "[{'dbName':'OctopusDeploy','dbFiles':['C:\\SQLData\\OctopusDeploy.mdf','C:\\SQLData\\OctopusDeploy_log.ldf']},{'dbName':'TeamCity','dbFiles':['C:\\SQLData\\TeamCity.mdf','C:\\SQLData\\TeamCity_log.ldf']}]"
docker stop SQLServer
docker rm SQLServer
docker run --name SQLServer -d -p 1433:1433 --volume c:\Docker\Volumes\SQLServer:c:\SQLData -e sa_password=Password_01 -e ACCEPT_EULA=Y -e attach_dbs=$attachDbs microsoft/mssql-server-windows-developer
$docker = docker inspect SQLServer | convertfrom-json
$docker[0].NetworkSettings.Networks.nat.IpAddress
```

Success, those databases are now mounted when the container is recreated.  

![](attach-dbs-with-sql-server.png)