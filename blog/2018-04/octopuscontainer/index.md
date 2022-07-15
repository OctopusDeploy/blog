---
title: Running Octopus Deploy inside a container
description: The Octopus Deploy Docker image makes it a simple process to provision a new Octopus Server instance.
author: robert.erez@octopus.com
visibility: public
published: 2018-04-27
metaImage: metaimage-dockercontainer.png
bannerImage: blogimage-dockercontainer.png
bannerImageAlt: Octopus Docker Container Banner
tags:
 - Engineering
---

![Octopus Docker Container Banner](blogimage-dockercontainer.png)

In today's fast-paced world, what some of us really need is a quicker way to download and run Octopus Deploy. Thankfully Octopus Deploy Containers are now available and allow users to run an Octopus Server or Tentacle directly from inside a Docker container.
The [octopusdeploy/octopusdeploy](https://hub.docker.com/r/octopusdeploy/octopusdeploy/) and [octopusdeploy/tentacle](https://hub.docker.com/r/octopusdeploy/tentacle/) images are built alongside our standard Octopus Sever and Tentacle build process, so you can always be sure to have the latest version available (and the upgrade process is _crazy_ easy as you'll shortly see).

## Running Octopus Server in a container

Assuming you have a SQL database handy (which we'll shortly demonstrate itself isn't even necessary) starting the Octopus Server container at it's simplest looks like:

```shell
docker run -i --env sqlDbConnectionString=<MyConnectionString> octopusdeploy/octopusdeploy
```

Rather than go into details about the anatomy of the Docker commands in this post, I would encourage you to check out the [Docker docs](https://docs.docker.com/engine/reference/run/) for more detailed explanations.

As good container adherents, we all know we should treat containers as immutable and could be torn down at any point. To deal with this, we should ensure that the server files used by the instance are written to something more permanent. The Octopus Server container provides several mount points to make maintaining these files a little easier. In addition, although the container exposes the Octopus Web Portal on port `81` we may want to map it to a specific port on the host. Overall the above command is nice and simple, but it's probably ideal to provide our own admin credentials to access the newly created instance than rely on the defaults.

Let's add a little more configuration to this command:

```PowerShell
docker run --interactive --detach `
    --name OctopusServer `
    --publish "8081:80" `
    --volume "E:/Octopus/Repository:C:/Repository" `
    --volume "E:/Octopus/Artifacts:C:/Artifacts" `
    --volume "E:/Octopus/TaskLogs:C:/TaskLogs" `
    --env sqlDbConnectionString="Server=172.23.192.1,1433;Initial Catalog=Octopus;Persist Security Info=False;User ID=sa;Password=P@ssw0rdz;MultipleActiveResultSets=False;Connection Timeout=30;" `
    --env OctopusAdminUsername=Mario `
    --env OctopusAdminPassword=ItsAMe! `
    octopusdeploy/octopusdeploy:2018.4.0
```

Now that everything has been externalized from the container, we can easily upgrade the Octopus Server instance when that hot new Octopus feature we have been waiting for becomes available. First, we need to get the master key from our original Octopus instance. As part of the startup process of this container, the master key is written to the logs so it can be found by running `docker logs OctopusServer`. With the key in hand, we can just stop the original container and rerun the above run command with the new master key environment variable and new version tag:

```PowerShell
docker run --interactive --detach `
    ...
    ...
    --env masterKey=7dnak8asdn23hjasd== `
     octopusdeploy/octopusdeploy:2018.4.1
```

And boom, the new version will download, perform any necessary database migrations and be up and running. _Told you it was crazy simple._ Check out [our docs](https://octopus.com/docs/installation/octopus-in-container/octopus-server-container) on the Octopus Server Images for further details about the available configuration.

## Running Tentacle in a container
An Octopus Tentacle container is also available however since any deployment tasks that run against the Tentacle in Octopus will run _within_ the container itself, you are less likely to use it for things like updating an IIS website or deploying up a NodeJS application. Where this will start providing more value, is when the Tentacle executable will be able for [running tasks](https://github.com/OctopusDeploy/Specs/blob/master/Workers/index.md) that are currently confined to "run-on-server".

When a Tentacle container starts up, it registers itself with the server details provided. With future changes, the Tentacle will also re-register itself when it shuts down however that functionality has only become available in [recent](https://github.com/moby/moby/issues/25982) Windows Container builds so that side is not yet accounted for in the current build.

```PowerShell
docker run --interactive --detach  `
    --name MyTentacle `
    --env ServerApiKey=API-48AC758FF8912B `
    --env ServerUrl=http://myoctopus.acme.com `
    --env TargetEnvironment=Development `
    --env TargetRole=InnerContainer `
    octopusdeploy/octopusdeploy:2018.4.1
```

The Tentacle can be configured in polling or listening mode and again, check out [our docs](https://octopus.com/docs/installation/octopus-in-container/octopus-tentacle-container) for further details on the available settings.

## No SQL? No Worries
"But Rob!" I hear you saying, "I don't have a SQL Server hanging about to run Octopus against." Well, that's fine because we can make use of [Docker Compose](https://docs.docker.com/compose/overview/) to spin up a SQL database container alongside our Octopus Server. (_Caution: There are [many opinions](http://patrobinson.github.io/2016/11/07/thou-shalt-not-run-a-database-inside-a-container/) around running a database inside a container for production purposes. We tend to agree. These next examples are probably best left for testing and experimentation._)

Using the following `docker-compose.yml` file:

```YAML
version: '2.1'
services:
  db:
    image: microsoft/mssql-server-windows-express
    environment:
      sa_password: "${SA_PASSWORD}"
      ACCEPT_EULA: "Y"
    healthcheck:
      test: [ "CMD", "sqlcmd", "-U", "sa", "-P", "${SA_PASSWORD}", "-Q", "select 1" ]
      interval: 10s
      retries: 10
  octopus:
    image: octopusdeploy/octopusdeploy:${OCTOPUS_VERSION}
    environment:
      OctopusAdminUsername: "${OCTOPUS_ADMIN_USERNAME}"
      OctopusAdminPassword: "${OCTOPUS_ADMIN_PASSWORD}"
      sqlDbConnectionString: "Server=db,1433;Initial Catalog=Octopus;Persist Security Info=False;User ID=sa;Password=${SA_PASSWORD};MultipleActiveResultSets=False;Connection Timeout=30;"
    ports:
     - "8081:81"
    depends_on:
      db:
        condition: service_healthy
    stdin_open: true
    volumes:
      - "E:/Octopus/Repository:C:/Repository"
      - "E:/Octopus/TaskLogs:C:/TaskLogs"
networks:
  default:
    external:
      name: nat
```

along with an accompanying `.env` file:

```
SA_PASSWORD=P@ssw0rd!
OCTOPUS_VERSION=2018.3.13
OCTOPUS_ADMIN_USERNAME=admin
OCTOPUS_ADMIN_PASSWORD=SecreTP@ass
```

and simply run:

```shell
docker-compose --project-name Octopus up -d
```

After a short wait, you will have a self-contained SQL Server and Octopus Server instance ready to perform deployments for your applications.

In [our docs](https://octopus.com/docs/installation/octopus-in-container/docker-compose#octopus-server-and-tentacle) we also point out how you can leverage the `C:\Import` volume mount on the Octopus Server image to provide initialization data to pre-populate the database and then include multiple Tentacles in the same `docker-compose` script.

## Octopus and Containers
This latest ability to run Octopus from within a container further cements the commitment at Octopus Deploy to provide improved solutions for users looking to make better use of containers in their environments. The [Kubernetes feature](https://octopus.com/blog/kubernetes-rfc) that is currently in the pipeline will allow even deeper integration with containers in your deployments regardless of platform. Let us know your thoughts on our future direction, take a look at our [existing](https://octopus.com/docs/deployments/docker) Docker offering and happy (containerized) deployments!
