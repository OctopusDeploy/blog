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

When a linux server is set up, a developer often has to configure the system for their needs. This could be for web development, system administration, data science, or other types of tasks. Each of these use cases would have a slightly different configuration needs. When new linux servers are created, they would  also have to be configured to suit the requirements. Runbooks provides a repeatable, automatic way of configuring linux servers, and can be adapted to suit different configuration needs. In this blog, I configure a linux server by using a runbook to specify all the dependencies required to do a specific task. This runbook can be saved and exported to be reused for future servers with the same need.

To follow along, you will need:

- A linux server (if you do not have one, I will show you how to create one in Azure)
- An Octopus Instance (create a [free trial to get started](https://octopus.com/docs/octopus-cloud))

## Creating a linux server in Azure

Microsoft Azure is a cloud computing platform that allows you to create virtual machines, web applications, kubernetes clusters and more. We will use this service to create a linux server for our runbook example.

To do this, you will need an Azure account. [You can sign up for a free trial on their website](https://portal.azure.com/).

In the home page navigate to **Create a resource, Ubuntu Server 20.04 LTS, Create**.

The resource should be linked to a subscription created with the trial. Choose to create a new resource group. Give the server a name. Under Administrator account it will ask you to create a SSH public key or Password. Either is fine, just remember to save the passwork as you will need it later. I went with all of the default settings so you can select **Review + Create** to accept them.

When your server is created, select **Go to resource**. Connect to the server by clicking **Connect, Bastion**. You will need to enter either your SSH key or Password from earlier.

## Installing an octopus tentacle on the linux server

When connected you wil be preseted with a bash shell. This is where we want to set up an Octopus Tentacle to communicate with the Octopus instance running our runbook. Run the following commands to install the tentacle:

```

sudo apt-key adv --fetch-keys https://apt.octopus.com/public.key

sudo add-apt-repository "deb https://apt.octopus.com/ stretch main"
# for Raspbian use
# sh -c "echo 'deb https://apt.octopus.com/ buster main' >> /etc/apt/sources.list"

sudo apt-get update
sudo apt-get install tentacle


```

As the tentacle needs to communicate with your Octopus instance, we need a key to provide to the tentacle to authenticate. To do this, go to you Octopus instance **Your name in the top right, Profile, My API Keys, New API Key**. Give the key a name and make sure to save it, it will only be visible once.

Configure the tentacle on the linux server by running the following command:

```
/opt/octopus/tentacle/configure-tentacle.sh
```

The setup script will ask you a series of questions. Make sure to specify the following parameters:

- Polling Tentacle
- The Server URL is the URL of your Octopus instance
- Enter the API key configured earlier
- Deployment Target
- Specify the environment of the Octopus instance
- Specify the role of the tentacle (this will be used later in the runbook)
- all other parameters can be left as default

### Confirming the tentacle connection

Once the tentacl is setup, confirm that it is connect by going to **Infrastructure, Deployment Targets**. Here you should see your connected deployment target.

![Linux Deployment Target](linux-deployment-target.png "width=500")

## Setting up the runbook in octopus

Now that we have created the linux server, installed and connected the tentacle, we can create the runbook. This run book creates a developement environment for a web developement use case. Other types of use cases would require a slightly different configuration setup. The power of runbooks that each time a web developer needs a server set up, the runbook can be run each time to create the same result. A new runbook can be created to accomodate for different types of configuration requirememts.

Create a project to host the runbok by going to **Projects, Add Project**. In your project, go to **Runbooks, Add Runbook, Process, Add Step, Script, Run a Script, Add**. Under `on Targets in Roles` add the role that you specified in the tentacle set up script. Specify a Bash script and add the following code:

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

You should get a success result when the runbook has completed.

![Runbook succes web developer](runbook-web-success.png "width=500")

Go to the linux server and run the npm command to confirm the installation.

![npm command](npm.png "width=500")

## Creating a different configuration

To demonstrate the flexibility of runbooks, lets create a different kind of configuration. We will use a simple .NET configuration. Create another linux server and follow the same steps to create another runbook with a Octopus tentacle. Add the following code:

```
wget https://packages.microsoft.com/config/ubuntu/21.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0

```

![Runbook success](runbook-web-success.png "width=500")

Go to the linux server and run the dotnet command to confirm the installation.

![dotnet command](dotnet.png "width=500")

## Comments on the usefulness of this approach

This workflow demonstrated that runbooks can be used to configure linux servers. In this example a web development configuration was applied to the server. This can easily be extended to other types of configuration by adding another runbook with seperate configuration parameters. In this case we used a .NET configuration. Anytime a new linux server must be set up with the web developer configuration requirements, the runbook can be automatically triggered upon server creation to automatically install the required tools.


## Conclusion

Configuring servers can be a manual process. Often there are many different configuration requirements that a server could have. To help with this, runbooks in Octopus Deploy provide a repeatable, automated way to configure servers. They can be set up to cater to specific configuration needs and automatically triggered when a server is created. The repeatable nature of runbooks creates consistency within the organizational infrastructure. Runbooks reduces the load on system administrators to focus on other tasks. If you would like to know more about how Octopus Deploy and runbooks can help with you deployment needs, [contact us today!](sales@octopus.com)

!include <q2-2022-newsletter-cta>

Happy deployments!
