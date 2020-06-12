---
title: Introducing the Octopus Server Linux Docker image
description: 
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

While Octopus started life as a Windows application, it became clear that migrating to Linux would be a cornerstone of our plans to provides a hosted solution. Today, all our hosted instances are running Octopus in Linux containers inside Kubernetes.

Today I would like to announce early access the Octopus Server Linux Docker image. Based on the same code that powers our hosted solution, these images allow Linux users to host Octopus natively on their operating system of choice.

## Getting started

The easiest way to get started is with the following Docker Compose template, which configures Microsoft SQL server and Octopus with a single command.

Here is the `docker-compose.yml` file:
```yaml
version: '3'
services:
   db:
    image: ${SQL_IMAGE}
    environment:
      SA_PASSWORD: ${SA_PASSWORD}
      ACCEPT_EULA: ${ACCEPT_EULA}
    ports:
      - 1401:1433
    healthcheck:
      test: [ "CMD", "/opt/mssql-tools/bin/sqlcmd", "-U", "sa", "-P", "${SA_PASSWORD}", "-Q", "select 1"]
      interval: 10s
      retries: 10
   octopus-server:
    image: octopusdeploy/octopusdeploy:${OCTOPUS_SERVER_TAG}
    privileged: true
    environment:
      ACCEPT_EULA: ${ACCEPT_OCTOPUS_EULA}
      OCTOPUS_SERVER_NODE_NAME: ${OCTOPUS_SERVER_NODE_NAME}
      DB_CONNECTION_STRING: ${DB_CONNECTION_STRING}
      ADMIN_USERNAME: ${ADMIN_USERNAME}
      ADMIN_PASSWORD: ${ADMIN_PASSWORD}
      ADMIN_EMAIL: ${ADMIN_EMAIL}
      OCTOPUS_SERVER_BASE64_LICENSE: ${OCTOPUS_SERVER_BASE64_LICENSE}
      MASTER_KEY: ${MASTER_KEY}
      ADMIN_API_KEY: ${ADMIN_API_KEY}
    ports:
      - 8080:8080
    depends_on:
      - db
```

Here is the `.env` file, which defines the environment variables used by Docker Compose. Be sure to change the `ACCEPT_EULA` and `ACCEPT_OCTOPUS_EULA` values!

```
# It is highly recommended this value is changed as it's the password used for the database user.
SA_PASSWORD=N0tS3cr3t!

# Tag for the Octopus Server image. See https://hub.docker.com/repository/docker/octopusdeploy/octopusdeploy for the tags.
OCTOPUS_SERVER_TAG=2020.3.0

# Sql Server image. Set this variable to the version you wish to use. Default is to use the latest.
SQL_IMAGE=mcr.microsoft.com/mssql/server

# The default created user username for login to the Octopus Server
ADMIN_USERNAME=admin

# It is highly recommended this value is changed as it's the default user password for login to the Octopus Server
ADMIN_PASSWORD=Passw0rd123

# Email associated with the default created user. If empty will default to octopus@example.local
ADMIN_EMAIL=

# Accept the Microsoft Sql Server Eula found here: https://hub.docker.com/r/microsoft/mssql-server-windows-express/
ACCEPT_EULA=N

# Use of this Image means you must accept the Octopus Deploy Eula found here: https://octopus.com/company/legal
ACCEPT_OCTOPUS_EULA=N

# Unique Server Node Name - If left empty will default to the machine Name
OCTOPUS_SERVER_NODE_NAME=

# Database Connection String. If using database in sql server container, it is highly recommended to change the password.
DB_CONNECTION_STRING=Server=db,1433;Database=OctopusDeploy;User=sa;Password=N0tS3cr3t!

# Your License key for Octopus Deploy. If left empty, it will try and create a free license key for you
OCTOPUS_SERVER_BASE64_LICENSE=

# Octopus Deploy uses a Master Key for encryption of your database. If you're using an external database that's already been setup for Octopus Deploy, you can supply the Master Key to use it. If left blank, a new Master Key will be generated with the database creation.
MASTER_KEY=

# The API Key to set for the administrator. If this is set and no password is provided then a service account user will be created. If this is set and a password is also set then a standard user will be created.
ADMIN_API_KEY=
```

Create the containers with the command:

```
docker-compose up
```

Octopus is then accessible on http://localhost:8080.