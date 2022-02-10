---
title: Configuring Linux Servers with Runbooks
description: A guide to configuring linux servers with runbooks
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Runbooks Series
  - Runbooks
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->



When setting up a Linux server, a developer often has to configure the system for their needs. Developers could use the server for web development, system administration, data science, or other types of tasks. Each of these use cases would have different configuration needs. Runbooks provide a repeatable, automatic way of configuring Linux servers. They can be adapted to suit different configuration needs. In this blog, I configure a Linux server using a runbook to specify all the dependencies required to do a specific task. This runbook can be saved and exported for future servers with the same need.

To follow along, you will need:

- A Linux server (if you do not have one, I will show you how to create one in Azure)

- An Octopus Instance (create a [free trial to get started](https://octopus.com/docs/octopus-cloud))

## Creating a Linux server in Azure

Microsoft Azure is a cloud computing platform that allows you to create virtual machines, web applications, and other cloud-based resources. We will use this service to create a Linux server for our runbook example.

To do this, you will need an Azure account. [You can sign up for a free trial](https://portal.azure.com/).

On the home page, navigate to **Create a resource, Ubuntu Server 20.04 LTS, Create**.

Choose the linked subscription and create a new resource group. Give the server a name. Under **Administrator account** generate an SSH public key or password, which you will use later. Select **Review + Create** to accept the default settings.

When finished, select **Go to resource**. Connect to the server by clicking **Connect, Bastion**. You will need to enter either your SSH key or password from earlier.

## Installing an octopus tentacle on the Linux server

When connected, you will see a bash shell. We set up an Octopus Tentacle to communicate with the Octopus instance running our runbook. Run the following commands to install the tentacle:

```

sudo apt-key adv --fetch-keys https://apt.octopus.com/public.key

sudo add-apt-repository "deb https://apt.octopus.com/ stretch main"

# for Raspbian use

# sh -c "echo 'deb https://apt.octopus.com/ buster main' >> /etc/apt/sources.list"

sudo apt-get update

sudo apt-get install tentacle

```

As the tentacle needs to communicate with your Octopus instance, we need a key to provide to the tentacle to authenticate. To do this, go to your Octopus instance **Your name in the top right, Profile, My API Keys, New API Key**. Give the key a name and make sure to save it. It will only be visible once.

Configure the tentacle on the Linux server by running the following command:

```

/opt/octopus/tentacle/configure-tentacle.sh

```

The setup script will ask you a series of questions. Make sure to specify the following parameters:

- Polling Tentacle

- The Server URL is the URL of your Octopus instance

- Enter the API key configured earlier

- Deployment Target

- Specify the environment of the Octopus instance

- Specify the role of the tentacle (You will use this later in the runbook)

- all other parameters can be left as default

### Confirming the tentacle connection

Confirm that it is connected by going to **Infrastructure, Deployment Targets**. Here you should see your connected deployment target.

![Linux Deployment Target](linux-deployment-target.png "width=500")

## Setting up the runbook in octopus

Now that we have created the Linux server, installed and connected the tentacle, we can create the runbook. This runbook establishes a development environment for a web development use case. Other types of use cases would require a different configuration setup. Operations can run the runbook when a web developer needs a server set up. Operations can create a new runbook to accommodate other configuration requirements.

Create a project to host the runbook by going to **Projects, Add Project**. In your project, go to **Runbooks, Add Runbook, Process, Add Step, Script, Run a Script, Add**. Under `on Targets in Roles`, add the role that you specified in the tentacle setup script. Specify a Bash script and add the following code:

```

apt update

apt upgrade

apt install -y build-essential

apt install -y curl

apt-get install -y git-core

apt-get install -y nodejs

apt-get install -y npm

```

Click **SAVE** and **Run**.

You should get a successful result.

![Runbook succes web developer](runbook-web-success.png "width=500")

Go to the Linux server and run the npm command to confirm the installation.

![npm command](npm.png "width=500")

## Creating a different configuration

Let's create a different kind of configuration. We will use a simple .NET configuration. Create another Linux server and follow the same steps to create another runbook with an Octopus tentacle. Add the following code:

```

wget https://packages.microsoft.com/config/ubuntu/21.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb

sudo dpkg -i packages-microsoft-prod.deb

rm packages-microsoft-prod.deb

sudo apt-get update; \

sudo apt-get install -y apt-transport-https && \

sudo apt-get update && \

sudo apt-get install -y dotnet-sdk-6.0

```

Run the runbook to configure the server

![Runbook success](runbook-web-success.png "width=500")

Go to the Linux server and run the dotnet command to confirm the installation.

![dotnet command](dotnet.png "width=500")

## Comments on the usefulness of this approach

This workflow demonstrated that runbooks could configure Linux servers. In this example, you applied a web development configuration to the server. Runbooks extend to other servers by adding another runbook with separate configuration parameters. In this case, we used a .NET configuration. Different runbooks allow the automatic setup of varying server configurations.

## Conclusion

Configuring servers can be a manual process. Often there are many different configuration requirements that a server could have. To help with this, runbooks in Octopus Deploy provide a repeatable, automated way to configure servers. They can be set up to cater to specific configuration needs and triggered when needed. The repeatable nature of runbooks introduces consistency within the organizational infrastructure. Runbooks minimize the load on system administrators to focus on other tasks. If you would like to know more about how Octopus Deploy and runbooks can help with your deployment needs, [contact us today!](sales@octopus.com)


!include <q2-2022-newsletter-cta>

Happy deployments!
