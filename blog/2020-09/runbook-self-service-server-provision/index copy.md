---
title: Using Runbooks To Provision Infrastructure
description: How to use Runbooks to provision/destory infrastructure and give self service access to teams in the organisation.
author: Adam Close
visibility: private
published: 3020-01-01
metaImage: 
bannerImage: 
tags:
  - runbooks
---

In this blog post, I'll show you how to use Runbooks to create & destroy application servers. Control cloud expenditure and give your organizations teams the ability to safely spin up infrastructure on-demand in a controlled and gated process. 

We will be configuring Octopus Runbooks to spin up infrastructure in AWS using CloudFormation steps. We will also be creating another Runbook to tear down the infrascture and adding a scheduled trigger to run the Tear Down Runbook on a schedule.

## Create Octopus Teams

I want to control who can and can't spin up infrastucutre in AWS or at least provide a way of notifying a controlling team. To do this I need a new team in Octopus with the users that the abilisty to approve my Runbook runs.
 
 When you create the team assign the role Runbook Consumer 


## Create AWS resources

* Credentials 
* Key Pair

### Create AWS Key Pair

A key pair is a private and public key used to prove your identity when connecting to EC2 instances.

In the AWS Management Console, navigate to **{{EC2, Key Pairs, Click *Create key pair*, Name your key pair, click Create Key Pair}}**

[AWS Keypair](aws-keypair.png "width=500")

:::hint
AWS will automaticly download a .ppk file to your machine. Save this and it can be used later to SSH onto your running instances.
:::

### Create the AWS credentials

The first thing you need is an AWS Access key and Secret key. To get this log into the [AWS Management Console](https://aws.amazon.com/console/) and create an account using Identity and Access Management (IAM).  After the account has been created, click on the user, then **Security Credentials** to create an `Access Key`.

[AWS user access key](aws-credentials.png "width=500")

After the access key has been created, save the `Secret Key` as you will only be shown the value *once*.  The `Access Key` and `Secret Key` combination is used to authenticate with AWS.

### Create AWS Account

To deploy CloudFormation templates in Octopus you need to setup an AWS account.

Using your `Access Key` and `Secret Key` [create an AWS account in Octopus](https://octopus.com/docs/infrastructure/deployment-targets/aws#create-an-aws-account).

:::hint
It is possible to AWS IAM roles rather than the IAM credentials. If the Octopus server built in worker or external worker deploying the steps have an IAM role attached then it's possible to use the IAM role rather than the `Access Key` and `Secret Key`.
:::

## Setup Octopus Runbooks

With all the resouces created we can now start configuring Octopus Runbooks to create our infrascture.

### Create a project

We need to create a project so that we can create some Runbooks. To create a project and and in my case I'm calling it Random Qutoes 

### Create Variables

To create the Runbook we need some variables for paramters in our steps.

| Parameter  | Description | Example |
| ------------- | ------------- | ------------- |
| Project.Octopus.Thumbprint | Octopus Instance Thumbprint | 7F226XXXXXXXX |
| Project.Octopus.key | Octopus API Key | API-XXXXXXXX |
| Project.AWS.Account | Octopus AWS Account | MyAccount |
| Project.AWS.KeyPair | AWS Key Pair Name | MyKeyPairName |
| Project.AWS.Region | Name of AWS region| eu-west-2 |
| Project.AWS.StackName | Name cloudformaton stack| octopus-blog-random-quotes-#{Octopus.Environment.Name} |



### Creating the Runbooks

You may already have a project created to deploy your application and you can create your Runbooks there but if not create a new project and two new Runbooks. One to spin up our infrascture and another to tear it down.

For helping creating projects you can find more information here.

To create a Runbook, inside your project click Runbooks > Add Runbooks > Give the Runbooks a name and an optioal description.

[Octopus Runbooks](octopus-runbooks.png "width=500")
[AWS user access key](aws-credentials.png "width=500")

We can now start building out our Spin up infrastrcuture Runbook. Click on the Runbook and click **DEFINE YOUR RUNBOOK PROCESS** , Click **ADD STEP**, Search for **manual intervention**, click **Manual Intervention Required** from the list of installed step templates and click **ADD**.

There a number of options to configure in this step and I've added my config below


Next, click ** ADD STEP** 

Firstly, Click **ADD STEP**, Search for **CloudFormation**, click **Deploy an AWS CloudFormation template** from the list of installed step templates and click **ADD**.

Next, Add a ** De

Steps

Approval 

Deploy cloud formation

Register Tentacle


### Destorying infrastucture 



### Destorying infrastucture on a schedule 

Using Runbook to destory infrastcuture is great for saving costs on 

To do this we are going to setup a Runbook Trigger that will Run the destroy infreatcture Runbook 



## Conclusion

In this post I showed you how to use Runbooks to provsion servers in the cloud and control cloud costs with triggers. 

## Learn more

- [link](https://www.example.com/resource)
