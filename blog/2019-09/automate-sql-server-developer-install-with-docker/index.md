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

For quite some time, SQL Server has been a docker image.  Running SQL Server on Docker could solve two of the major headaches I have with running SQL Server Developer edition as a Windows Service; namely it is running all the time consuming resources and I have to patch it.  It may be hard to believe, but installing SQL Server Developer edition is not one of those concerns, I've [automated that](https://octopus.com/blog/automate-sql-server-install).  When I was a full-time developer, I needed to run SQL Server all the time.  With my new role, I only want SQL Server running some of the time, and I don't want to worry about having to patch it.  Can running SQL Server on Docker solve that?  In this article I answer that question, along with how I accomplished configuring SQL Server to run on Docker.

!toc

## The Scenario

In a couple of months, I will be speaking at [TechBash 2019](https://techbash.com/sessions). The presentation will cover how my automated database deployment process progressed as I learned more about automation.  The presentation will start with TeamCity and then move onto Octopus Deploy.  I don't need or want all of that running all the time.  Only when I am prepping or practicing my demo.  In other words, I want to spin up and down an entire CI/CD pipeline on my laptop.  So why not use Docker?

Please note, this is Part 1 of a 2 part series on setting up a CI/CD pipeline in Docker.  This article will cover getting SQL Server up and running in Docker.

## Prep Work

This article will assume you have a passing familiarity with Docker.  If you are not familiar with the core concepts of Docker, I encourage you to read the [Docker overview page](https://docs.docker.com/engine/docker-overview/).  The key thing to understand is the difference between images and containers.  

My laptop is running Windows 10 professional.  I will be using [Docker Desktop](https://docs.docker.com/docker-for-windows/), which is sometimes known as Docker for Windows.  Some prep-work will be needed before I can jump in and start writing scripts.

### Enable CPU Virtualization
When it comes down to it, Docker is a virtualization host.  Just like any other virtualization host, the CPU has to support virtualization and that has to be enabled.  Typically virtualization is enabled in the BIOS.  Which means you'll have to do a Google search on how to to enable it in your computer manufacturer's BIOS.  Intel calls their virtualization technology [Intel VT](https://www.intel.com/content/www/us/en/virtualization/virtualization-technology/intel-virtualization-technology.html), along with Intel VTx.  AMD calls their virtualization technology [AMD V](https://www.amd.com/en/technologies/virtualization), sometimes you'll see it called VDI or SVM.  

### Installing Docker for Windows
After the quick BIOS update, it is time to install Docker Desktop, which includes Docker Compose and the Docker CLI.  The whole process to install Docker for Windows is [nicely documentation](https://docs.docker.com/docker-for-windows/install/).  No need to repeat that.

One thing to note, if you don't have Hyper-V enabled, the installer will enable it for you.  That will require a restart of the computer.

As I stated earlier, I want to eventually run my entire CI/CD pipeline on Docker.  At the time of this writing, the Octopus Deploy image is for Windows-based containers only.  After Docker Desktop has been installed, I need to switch over to Windows containers.  That is done by right clicking on the Docker Desktop icon in the taskbar and selecting `Switch to Windows containers...`.  

![](docker-desktop-switch-to-windows-containers.png)

### Setting up Folders to Share with Docker Containers
By default, Docker treats all containers as stateless.  Any changes made to the container, such as a creating a database, will be deleted when the container is removed.  This problem can be solved by making use of volumes in docker.  I setup a folder on my hard drive, C:\Docker\Volumes, to store those volumes.

![](docker-shared-folders.png)

It is important to note that if I were running these as Linux based containers I would need to follow the steps listed in the [Docker documentation on sharing drives](https://docs.docker.com/docker-for-windows/#shared-drives).

### Anti-Virus Configuration
One downside of running a Windows container (aside from the space overhead), is anti-virus products can and will block them from downloading.  This is because of how Docker stores images on the Windows file system.  Essentially, another folder called Windows will appear in a random location.  When anti-virus scanners see that they freak out.  Make sure you are using the latest version of your anti-virus of choice.

![](extracted-docker-files-windows.png)

## Bootstrapping
When setting up SQL Server, TeamCity or Octopus Deploy in Docker you will go through the same process.

1. Get the container running.
2. Configure something, such as create a database, add a build agent, create a service account.
3. Stopping the container.
4. Updating the Docker Compose file or the script kicking off Docker to use that item.
5. Restarting the container.

I wish I could provide you with a single PowerShell script which will bootstrap everything.  But that would require configuring quite a number of components in Docker by hand.  Creating a database in SQL Server or adding a service account in Octopus Deploy, only needs to happen once.  It doesn't make sense to hand write a complex bootstrap script just to avoid a few minutes of work.

## Configuring the SQL Server Developer Container
As stated before, by default all containers are stateless.  This makes things a bit tricky as SQL Server, TeamCity, and Octopus Deploy are stateful applications.  For each of the containers I will be running in my CI/CD pipeline I need to solve for the following:

1. Get the container up and running with no extra configuration.
2. Persist data created in a container, such as a SQL Database, to a volume.
3. Have a static IP or hostname to keep configuration easy.
4. Communicate between containers, such as Octopus Deploy -> SQL Server or TeamCity -> Octopus Deploy.
5. Start everything at the same time.

This article will solve the first 3 of those problems.  

### Running SQL Server Developer Container for the first time
That is a pretty ambitious list to solve Docker newbies such as myself.  I want to take it a step of a time.  I am first going to pull down the SQL Server Windows Developer image from Docker Hub.

```PowerShell
docker pull microsoft/mssql-server-windows-developer
```

If you are following along step by step, brew up some tea or coffee and sit back, because this might take a while to complete.  Not only is it downloading that image, it is also downloading all the dependencies as well.    

![](docker-download-sql-server.png)

Now that the image is downloaded, it is time to get it fired up and run some SQL scripts.  Thankfully, the documentation [Microsoft added to Docker Hub](https://hub.docker.com/r/microsoft/mssql-server-windows-developer) makes this easy.  Please make note of the `--name` parameter being sent in.  This will make it easier later when we need to figure out how to connect to it.  Along with naming the instance, I will be setting the port to the default SQL Server port, `1433`.  

```PowerShell
docker run --name SQLServer -d -p 1433:1433 -e sa_password=Password_01 -e ACCEPT_EULA=Y microsoft/mssql-server-windows-developer
```

![](docker-run-sql-server-developer.png)

The SQL Server container is running.  But what is the IP Address to connect to it via SSMS?  The container is a assigned a dynamic IP Address, such as `172.19.1.0`.  This is different than running SQL Server as a Windows Service where it is possible to connect to it via `.` or `127.0.0.1`.  The below command will get the IP address of that instance.

```PowerShell
$docker = docker inspect SQLServer | convertfrom-json
$docker[0].NetworkSettings.Networks.nat.IpAddress
```

In this example the IP Address is `172.19.98.212`.

![](docker-get-ip-address-of-sql-server.png)

Now it is just a matter entering that IP Address, along with `sa` as the username / password defined above, to connect SQL Server Management Studio(SSMS) to that database.  

![](ssms-connected-to-docker-image.png)

Just like regular SQL Server, everything works as expected.  I can create a database and tables without any issue.

![](create-table-inside-instance.png)

What happens if the container needs to be restarted?

```PowerShell
docker stop SQLServer
docker start SQLServer
```

The IP Address changes when I do that.

![](restart-docker-container-no-remove.png)

The database still exists after the restart.  

![](test-database-after-restart-only.png)

But what about if the container needs to be recreated?  Typically that would be done when the container configuration changes or new version is released.  In addition to the `stop` command, I'll need to run the `rm` command which removes the container.

```PowerShell
docker stop SQLServer
docker rm SQLServer
docker run --name SQLServer -d -p 1433:1433 -e sa_password=Password_01 -e ACCEPT_EULA=Y microsoft/mssql-server-windows-developer
```

![](restart-docker-container.png)

And as expected, the databases were all deleted.  

![](restart-container-no-new-databases.png)

What is needed is a way to persist data to handle both restarting of the container and the recreation of the container.

### Persisting Data
The database files need to be persisted.  That will be accomplished using a volume will be pointed at `C:\Docker \Volumes\SQLServer`. There are [many](https://blog.sixeyed.com/docker-volumes-on-windows-the-case-of-the-g-drive/), [many articles](https://github.com/docker/labs/blob/master/windows/sql-server/part-3.md) about [Docker Volumes](https://docs.docker.com/storage/volumes/).  The TL;DR; is add the `--volume` switch to `docker run` to add a volume.  If the container is already running it needs to be destroyed before adding a volume.

```PowerShell
docker stop SQLServer
docker rm SQLServer
docker run --name SQLServer -d -p 1433:1433 --volume c:\Docker\Volumes\SQLServer:c:\SQLData -e sa_password=Password_01 -e ACCEPT_EULA=Y microsoft/mssql-server-windows-developer
```

All the database create commands need to specify `C:\SQLData\` as the directory for the data.  I know I will need a database for Octopus Deploy and another for TeamCity.

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

Those databases names and paths can be passed to the container using the `attach_dbs` environment variable.  It is possible to bootstrap all of this using a PowerShell script.  I didn't do that with this article because I only needed to create the databases once and I didn't see the point in spending the effort in writing a script to solve a problem I only need to do once.  

```PowerShell
docker stop SQLServer
docker rm SQLServer
$attachDbs = "[{'dbName':'OctopusDeploy','dbFiles':['C:\\SQLData\\OctopusDeploy.mdf','C:\\SQLData\\OctopusDeploy_log.ldf']},{'dbName':'TeamCity','dbFiles':['C:\\SQLData\\TeamCity.mdf','C:\\SQLData\\TeamCity_log.ldf']}]"
docker run --name SQLServer -d -p 1433:1433 --volume c:\Docker\Volumes\SQLServer:c:\SQLData -e sa_password=Password_01 -e ACCEPT_EULA=Y -e attach_dbs=$attachDbs microsoft/mssql-server-windows-developer
```

Those databases are now mounted when the container is recreated.  

![](attach-dbs-with-sql-server.png)

### Setting the IP Address and Host Name
Up until this point I have been using this command to get the IP Address for the SQL Server container.  

```PowerShell
$docker = docker inspect SQLServer | convertfrom-json
$docker[0].NetworkSettings.Networks.nat.IpAddress
```

As I build out my CI/CD pipeline, I don't want to have to calculate the IP address each time the container is spun up.  That is going to make it tricky to connect via SSMS or through a web browser for Octopus Deploy and TeamCity. What would be even better is to not have to worry about IP Addresses at all.  This is where [Docker Compose](https://docs.docker.com/compose/) enters the picture.  Docker Compose handles a lot of the behind the scenes work for us.  All it takes is converting the existing commands we have been using to a YAML file.

```YAML
version: '3.7'
services:
  SQLServer:
   image: microsoft/mssql-server-windows-developer
   environment:
     - ACCEPT_EULA=Y
     - SA_PASSWORD=Password_01   
     - attach_dbs=[{'dbName':'OctopusDeploy','dbFiles':['C:\\SQLData\\OctopusDeploy.mdf','C:\\SQLData\\OctopusDeploy_log.ldf']},{'dbName':'TeamCity','dbFiles':['C:\\SQLData\\TeamCity.mdf','C:\\SQLData\\TeamCity_log.ldf']}]
   ports:
     - '1433:1433'
   volumes:
     - c:\Docker\Volumes\SQLServer:c:\SQLData
```

I saved that docker-compose file in the C:\Docker folder on my hard drive.  

![](docker-compose-file.png)

Then I ran this command in PowerShell.

```PowerShell
Set-Location C:\Docker
docker-compose Up
```

Now I can access my SQL Server, which is running in Docker, by connecting to localhost instead of an IP address.

![](docker-compose-ssms-sucess.png)

## Wrapping Up

Getting SQL Server running in Docker turned out to be a lot less work than I thought it would be.  I was expecting hours upon hours of work, but in the end I had something up and running after a couple of hours.  And that included the necessary time to research Docker.  My hope is this article gave you enough direction for you to take the dive yourself into Docker and realize it is not so big and scary.  And maybe, just maybe, you'll use Docker to host SQL Server on your development machine instead of installing SQL Server Developer.

Until next time, Happy Deployments!