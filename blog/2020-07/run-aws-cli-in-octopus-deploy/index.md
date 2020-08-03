---
title: Run the AWS CLI in Octopus Deploy
description: This post provides a step by step guide on how to run AWS CLI commands inside of Octopus Deploy.
author: michael.levan@octopus.com
visibility: private
published: 3020-03-09
metaImage: 
bannerImage: 
tags:
 - Product
 - DevOps
---

Have you ever found yourself in a situation where you knew you wanted to automate the creation of an object or perhaps even list out objects and get a report, but you didn't want to jump around between programming languages? CLI's give you a way to have the full usability of an SDK and they typically always run the same on every system, which means you don't have to create a wrapper around some API.

In this blog post, we take a look at how to use the AWS CLI in Octopus Deploy. The demonstration is around creating an S3 bucket with the **Run an AWS CLI Script** step template.

## Prerequisites

To follow along with this blog post, you should have the following:

- An Octopus Deploy server, on-premises or Octopus Cloud.
- An AWS account set up with an IAM user in Octopus Deploy.
- Experience with the AWS CLI.
- At least one environment set up in Octopus Deploy.
- Octopus CLI installed. If you don't have it installed, you can download and install it from the [Octopus CLI download page](https://octopus.com/downloads/octopuscli).

## Create a new Project

Before running any AWS CLI commands or creating steps, 

you need to configure a project so that you have somewhere to create the AWS CLI process and steps. To do this, we'll use the power of another CLI, the Octopus CLI.

Open up a terminal and run the following command to create a new project with the appropriate switch values added:

```
octo create-project --name AWSCLIDeployments --server=octopus_server_url --apiKey=octopus_server_api_key --projectGroup project_group --lifecycle=lifecycle_name
```

Open a web browser and log into the Octopus Web Portal. You should now see the new project available.

![AWS CLI Project](images/2.png)

## Configure the variables

Now that the project is created, you can start the configuration process of the project itself. The first configuration that you'll take a look at is variables. For the AWS CLI step template to work, it needs the AWS account to be a variable.

1. In the Octopus Web Portal, go to **{{Projects,AWSCLIDeployments}}** to start configuring the variables.

![](images/3.png)

2. Under the project pane, click on **Variables**.
3. Within the project variables under **value**, choose the drop down and select **CHANGE TYPE**.
4. Under the type options, choose **AWS Account**.
5. Choose an existing AWS Account setup and give it a name. When complete, click the green **DONE** button**.
6. Click the green **SAVE** button to save the variable into the project.

The variable for the AWS account has now been configured.

## Add the AWS CLI Step

In the previous section, you set an AWS account as a variable. Now you're ready to start configuring the actual AWS CLI step itself to run AWS CLI commands. To do that, you're going to create a new process.

1. Under the project pane, choose **Process**.

![](images/9.png)

2. On the Process page you can start adding new steps, specifically, the AWS CLI step. Click the **ADD STEP** button.
3. Under **Choose Step Template**, choose AWS.

Under AWS you'll see a few options under two categories:

- Installed Step Templates
- Community Contributed Step Templates

4. The AWS CLI step is under the Installed Step Templates and is called **Run an AWS CLI Script**. After you find it, click the step.

Under the **Run an AWS CLI Script** step, there are several options that are based on your needs. For example, which environment to run in and worker pools. Because of that, this blog post will outline the specific tasks in the step that are specified to the AWS CLI.

5. The first task is the AWS Tools task. That has everything we need so we select it.

![](images/13.png)

6. The second task for AWS specific tasks is under Amazon Web Services, which is where you can select the account variable and specify a region. For example, in the screenshot below, the `AWSAccount` account variable and **us-east-1** region are specified.

![](images/14.png)

The final task-specific for AWS is the Script section, where you can add in the AWS CLI command you want to use. You have two primary options:

- Inline source code
- Script in a package

For the purposes of this blog post, the inline source code was used. Under the Inline **Source Code option**, type in the following code which will be used to create an S3 bucket. You can also change the name of the bucket for an environment you're in instead. Remember, the S3 bucket names must be unique.

```
aws s3api create-bucket --bucket octopusdeploys392 --region us-east-1
```

7. After you type in the code, click the green **SAVE** button.

You are now ready to run the pipeline.

## Run the Pipeline

The step is now created to use the AWS CLI, the inline code has been added, and you're ready to start the deployment process of the pipeline. 

1. Under the project, click the blue **CREATE RELEASE** button.
2. To save the release, click the green **SAVE** button.
3. Choose which environment you want to deploy to. For example, Dev.
4. Click the green **DEPLOY** button and the deployment will begin.
5. When complete, you will see the task summary in which creating the S3 bucket was complete.

![](images/19.png)

Congrats! You have successfully used the AWS CLI to create an S3 bucket in AWS.

## Conclusion

Many CLI's give you the ability to perform simple actions around tasks that may be complex or cumbersome in the UI. With a CLI, you can still interact directly with a platform from a programmatic perspective to ensure you have the ability to automate tasks.

In this blog post you learned not only how to get a project up and running in Octopus Deploy, but also how to configure the AWS CLI task to create an S3 bucket in AWS.
