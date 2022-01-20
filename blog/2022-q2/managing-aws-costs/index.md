---
title: Managing AWS costs with Instance Scheduler
description: Learn how to deploy and configure the Instance Scheduler to shutdown unused AWS resources
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

The promise of cloud computing has been to allow teams to efficiently scale up and down on demand. While on-premises infrastructure can save on electricity and cooling costs by shutting down, cloud based resources can avoid almost all charges (storage fees usually apply to stopped resources) by stopping any that are unused.

AWS provides the [Instance Scheduler](https://aws.amazon.com/solutions/implementations/instance-scheduler/) to shutdown and restart EC2 and RDS resources on demand. This is a great solution for teams that have Octopus resources like workers running in AWS, and where they are unused for most of day.

In this post you'll learn how to install the Instance Scheduler, configure it with custom periods, and tag resources to be automatically shutdown and restarted.

## Prerequisites

You'll need Python 3, `jq`, `curl`, and `unzip` installed to complete the steps in this post. To install these tools in Ubuntu, run the command:

```bash
apt-get install jq curl unzip python3
```

To install jq in Fedora, RHEL, Centos, and Amazon Linux, run the command:

```bash
yum install jq curl unzip python3
```

The Instance Scheduler is managed with a custom Python application called [Scheduler CLI](https://docs.aws.amazon.com/solutions/latest/instance-scheduler/scheduler-cli.html). 

The CLI requires Python 3. If you have Python 2 and Python 3 installed, you can force the use of Python 3 with the command:

```bash
alias python=python3
```

Run the following commands to install the CLI:

```bash
curl -O https://s3.amazonaws.com/solutions-reference/aws-instance-scheduler/latest/scheduler-cli.zip
unzip scheduler-cli.zip
python setup.py install
```

