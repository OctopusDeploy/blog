---
title: Lessons learned using Docker for development and testing
description: Learn some of the lessons I learned using Docker for development and testing
author: bob.walker@octopus.com
visibility: public
published: 2020-09-30
metaImage: 
bannerImage: 
tags:
 - Engineering 
---

I do a lot of CI/CD pipeline demos, typically using TeamCity and Octopus Deploy.  Demos don't take up 100% of my time, it doesn't make much sense to run those services all day every day on my laptop.  I also didn't want to worry about updating them.  I chose Docker because I could update the tooling and spin up and down a CI/CD pipeline using a simple command.  In setting that up I learned some interesting lessons about using Docker for development and testing.

!toc

## Iterate on the Docker Compose file

Ideally, all Docker containers would be stateless.  However, several companies, including Octopus Deploy, have created stateful containers.  And those stateful applications require typically configuration.  This makes using a tool such as [Docker Compose](https://docs.docker.com/compose/) a bit trickier.  If you are not familiar with Docker Compose, it makes it easy to define and start multiple containers using a single command.  

My CI/CD pipeline required multiple components, which makes Docker Compose an ideal tool.  However, it doesn't handle that initial configuration to well.  For the CI/CD pipeline, I needed to create the databases for TeamCity and Octopus Deploy.  Once Octopus Deploy server was up and running, I needed to add the Octopus Deploy tentacles.  To add the tentacles a username/password or an API key needs to be used for authentication, which requires the server to be configured and running.  

The order of operations was:

1. Start up SQL Server in Docker Container.
2. Create databases.
3. Start up TeamCity and Octopus Deploy Docker containers.
4. Configure both containers.
5. Start TeamCity build agent and Octopus Deploy Tentacle Docker containers.

Docker Compose doesn't support that kind of dependency.  Docker Compose provides the ability for the TeamCity Container to wait until SQL Server Container starts up.  But it is not possible to configure a Docker Compose file to wait until the initial configuration is finished.  How could it know it needs to wait?  What would it wait on?  

This required iterating on the Docker Compose file.  That looked like:

1. Add SQL Server to Docker Compose and start it up using `docker-compose up`.
2. Create databases and update the Docker Compose file to ensure they are attached to SQL Server on restart.
3. Run `docker-compose down` to tear everything down.
4. Add TeamCity and Octopus Deploy to Docker Compose and start it up.
5. Configure TeamCity and Octopus Deploy.
6. Tear everything down using the same command as before.
7. Add TeamCity build agent and Octopus Deploy Tentacle(s) to Docker Compose file and start it up.

I learned the Docker Compose file is really the desired configuration of all the Docker Containers.  Trying to make it handle all that initial configuration made everything much more complicated than it needed to be.  Ideally, all I would need to do is share a single Docker Compose file with someone any they could have a CI/CD pipeline up and running in a few minutes.  However, that initial configuration, or bootstrap, should be handled by a separate script.  That 

If I gave someone that final Docker Compose file, I'd either have to write a script to bootstrap everything.  Or, zip up the directories storing the database files and configuration files.  

## Windows Based Containers tend to consume more resources

Linux based containers run on the Hyper-V VM `Docker Desktop VM`.  That VM is assigned vCPUs and is allocated memory.  

![](docker-linux-hyper-v-vm.png)

Each Windows based container run in their own process which appears as `Vnmem` in the Task Manager.  They can consume as many resources as they need.  Specifically the CPU.  

![](docker-windows-based-containers-running.png)

My first attempt at a CI/CD pipeline spun up 11 Docker containers.  Needless to say, it consume quite a bit of resources.  For roughly 10ish minutes after start the laptop CPU jumped up to 100% as those containers run their bootstrap scripts.  Once I learned each container is a separate process I scaled back the number of containers to 5.

In addition, Windows based images consume a lot more disk space.  Here are the Linux based Docker Images I have on my laptop.  Make note of the SQL Server image, `mcr.microsoft.com/mssql/server`.  That includes all the dependencies for the image.

![](docker-linux-images.png)

Compare that do Windows based images.  The SQL Server image, `microsoft/mssql-server-windows-developer` when including all the dependencies, is 10x the size.

![](windows-docker-images.png) 

## Containers are not read-only

When I read the phrase `Docker Image` I was picturing an ISO file, which cannot be changed.  That is not the case with Docker Containers.  The image is the base of the container.  In fact, a number of Docker Images include a configure script to kick off any necessary configuration.  The Octopus Deploy image does this.  When you start up a container with the Octopus Deploy image for the first time it will run several of the `Octopus.Server.exe` commands, such as `configure`, `admin`, `license`, and `metrics`.  You can see the script which Octopus runs [here](https://github.com/OctopusDeploy/Octopus-Docker/blob/master/Server/Scripts/configure.ps1).  

Once it is a Docker Container is running it can be changed.  New software can be installed on it.  Those changes are lost when the container is destroyed, typically for an update or a change in configuration.  Knowing that opens up a whole world of possibilities.  Docker containers are not read-only ISO images, they are really short-lived headless VMs.

## .NET Framework Connection Strings require SQL Server to be referenced by IP Address when using Docker Compose

I wanted both SQL Server and Octopus Deploy running in Docker containers.  The next question was how can I get Octopus Deploy to see SQL Server.  Docker Compose provides the ability to name a Docker Container and it does a lot of behind the scenes work so other containers in the can reference each other by name.  I thought that would work for .NET connection strings.  For example:

```
Server=SQLServer,1433;Initial Catalog=OctopusDeploy;Persist Security Info=False;User ID=sa;Password=Password_01;MultipleActiveResultSets=False;Connection Timeout=30;
```

That did not work for .NET connection strings.  Octopus Deploy is a .NET Framework application.  It could not find SQL Server by the container name on the same Docker network.  It only worked when the connection string used an IP address for the server.  However, the IP address changes each time `docker-compose up` is run.  What I did to solve that problem was creating a new network in Docker Compose file.

```YAML
networks: 
  cicd_net:
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16 
```

Using that network I could then hardcode the IP Address of each container.

```YAML
  SQLServer:
   image: microsoft/mssql-server-windows-developer
   environment:
     - ACCEPT_EULA=Y
     - SA_PASSWORD=Password_01
     - attach_dbs=[{'dbName':'OctopusDeploy','dbFiles':['C:\\SQLData\\OctopusDeploy.mdf','C:\\SQLData\\OctopusDeploy_log.ldf']}]
   ports:
     - '1433:1433'   
   volumes:
     - c:\Docker\Volumes\SQLServer\Databases:c:\SQLData
     - c:\Docker\Volumes\SQLServer\Backups:c:\Backups
   networks: 
    cicd_net:
      ipv4_address: 172.28.1.1   
```

The resulting connection string is:

```
Server=172.28.1.1,1433;Initial Catalog=OctopusDeploy;Persist Security Info=False;User ID=sa;Password=Password_01;MultipleActiveResultSets=False;Connection Timeout=30;
```

The entire Docker Compose file looks like this:

```YAML
version: '3.7'
services:
  SQLServer:
   image: microsoft/mssql-server-windows-developer
   environment:
     - ACCEPT_EULA=Y
     - SA_PASSWORD=Password_01
     - attach_dbs=[{'dbName':'OctopusDeploy','dbFiles':['C:\\SQLData\\OctopusDeploy.mdf','C:\\SQLData\\OctopusDeploy_log.ldf']}]
   ports:
     - '1433:1433'   
   volumes:
     - c:\Docker\Volumes\SQLServer\Databases:c:\SQLData
     - c:\Docker\Volumes\SQLServer\Backups:c:\Backups
   networks: 
    cicd_net:
      ipv4_address: 172.28.1.1   
  OctopusDeploy:
   image: octopusdeploy/octopusdeploy    
   ports:
     - '81:81'
     - '10943:10943'
   depends_on:
     - SQLServer       
   environment:
     - sqlDbConnectionString=Server=172.28.1.1,1433;Initial Catalog=OctopusDeploy;Persist Security Info=False;User ID=sa;Password=Password_01;MultipleActiveResultSets=False;Connection Timeout=30;
     - masterKey=YtnHskuInxiyH5MUIFEdVA==
   volumes:
     - c:\Docker\Volumes\Octopus\Server:c:\Octopus
     - c:\Docker\Volumes\Octopus\Server\Artifacts:c:\Artifacts
     - c:\Docker\Volumes\Octopus\Server\Repository:c:\Repository
     - c:\Docker\Volumes\Octopus\Server\TaskLogs:c:\TaskLogs
   links:
     - SQLServer
   networks: 
    cicd_net:
      ipv4_address: 172.28.1.2  
  OctopusDeploy_Worker01:
   image: octopusdeploy/tentacle    
   ports:     
     - '85:80'
   depends_on:
     - OctopusDeploy       
   environment:
     - serverApiKey=API-JSZTATMYECVBOY9CPWARAANHM0
     - serverUrl="http://172.28.1.2:81"
     - targetWorkerPool=DatabaseWorker     
     - serverPort=10943
     - targetName=DockerTentacle-DatabaseWorker01
   volumes:
     - C:\Docker\Volumes\Octopus\Worker01:c:\Applications
     - c:\Docker\Volumes\SQLServer\Backups:c:\Backups     
   links:
     - OctopusDeploy
   networks: 
    cicd_net:
      ipv4_address: 172.28.1.3
networks: 
  cicd_net:
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16           
```

## Local Development

Knowing how Docker works opens up a world of possibilities.  For this example, I will use running SQL Server locally.  I typically ran SQL Server as a Windows Service on my laptop for development.  I'm in the middle of working on a feature with lots of database changes.  QA manages to cause an error in the test environment by entering something odd through the UI they need me to look at.  When that happened I would:

- Commit all the pending changes to a branch.
- Checkout the commit which is in test.
- Point the code at the test database and start debugging.

QA are naturally curious people, while I am still debugging they poke around the UI a bit and in doing so cause the right piece of code to run and the error gets fixed.  

If I could, I would tell QA to not poke around.  But it isn't just them.  Anyone on the team could open up the record and cause the data to fix itself. In the past I solved this by copying the data down to my local database using SQL Data Compare, or using a hand-written data cloner.  I would have to revert any database changes for the cloner to work properly.  Which was annoying because depending on the what is being reverted I could end up losing data.  I could backup and restore the test database to my local machine, but I wouldn't want to overwrite my existing database, which means going in and changing the connection strings.

If I ran SQL Server as a Docker Container I could write a script to:

1. Stop the current SQL Server Docker Container
2. Backup the database to a shared location
3. Start up a new SQL Server Docker Container
4. Restore that backup into that container
5. The database name (MyAppDatabase) and the server name (localhost) will be the same as before, no need to change connection strings.

When I am done with debugging the issue I would need to remove that new container, start up my development SQL Server Docker Container, and switch the code back to the branch I was working on.  

That is just one example.  Imagine how you could leverage Docker for QA testing or beta testing.

## Wrapping Up

Docker is not a big and scary tool.  It has a bit of a learning curve, especially if you want to use it in Production.  But using it locally has been surprisingly smooth.  When I started building my CI/CD pipeline I understood the core concepts of Docker.  I had gotten a container running, but never multiple containers all talking to one another.  All in all, it took me less than a week to get the CI/CD pipeline up and running in Docker Containers.  As you can see above, I did run into a few brick walls along the way.  My hope is this article will soften the blow when you run into the same brick wall.
