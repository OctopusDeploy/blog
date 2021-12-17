---
title: Build docker image in Jenkinsfile and publish to ECR
description: Build docker image in Jenkinsfile and publish to ECR
author: terence.wong@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - DevOps
 - Jenkins
---

This blog will build and push the Octopus Deploy Underwater App to Amazon Elastic Container Registry (ECR) using Jenkins.  To follow along, you will need:

- An Amazon Web Services Account (AWS)
- A Jenkins instance
- A GitHub account

The [Traditional Jenkins Installation](/blog/2022-q1/jenkins-install-guide/index.md), [Docker Jenkins Installation](/blog/2022-q1/jenkins-docker-install-guide/index.md), or [Helm Jenkins Installation](/blog/2022-q1/jenkins-helm-install-guide/index.md) guides provide instructions to install Jenkins in your chosen environment.

## Amazon Web Services setup

To set up AWS for Jenkins, we need to create an access key and an ECR repository to store the image.

To create an access key, go to **Amazon Console &rarr; IAM &rarr; Users &rarr; [your user] &rarr; Security credentials &rarr; Create Access Key**

Your browser will download a file containing the Access Key ID and the Secret Access Key. These values will be used in Jenkins to authenticate to Amazon.

To create a repository, go to the **Amazon Console &rarr; ECR &rarr; Create Repository**

The ECR requires an image repository set up for each image you publish. Name the repository the name you want the image to have. 

You will see your repository under **Amazon ECR &rarr; Repositories**. Make a note of the zone it is in, in the URI field.

![ECR Repository](ecr-repository.png)

## Octopus Deploy Underwater Scene App

In this blog, we will build the Octopus Deploy Underwater App and push it to Amazon ECR. You will use these images in later blog posts.

## Jenkins setup

We first install some necessary plugins to interact with docker and Amazon. Go to the **Dashboard &rarr; Manage Jenkins &rarr; Manage Plugins. You will need the following plugins:

- [CloudBees AWS Credentials](https://plugins.jenkins.io/aws-credentials/)
- [Amazon ECR](https://plugins.jenkins.io/amazon-ecr/)
- [Docker Pipeline](https://plugins.jenkins.io/docker-workflow/)

You can search for these plugin in the available tab. Once they are installed they will appear in the installed tab.

We will use a Jenkinsfile to compile, build, test, and push the image to Amazon ECR. A Jenkins file is a configuration file that defines a Jenkins Pipeline. A Jenkins Pipeline is a series of steps that Jenkins will perform on an artifact to achieve the desired result. In this case, it is the clone, build, test, and push of an image to Amazon ECR. The power of using a Jenkinsfile is to check it into source control to manage different versions of the file.

In your Jenkins instance, go to **Manage Jenkins &rarr; Manage Credentials &rarr; Jenkins Store &rarr; Global Credentials (unrestricted) &rarr; Add Credentials**

![Manage Credentials](manage-credentials.png)

Fill in the following fields, leaving everything else as default:

- **Kind**-AWS credentials
- **ID** - amazon-credentials
- **Access Key ID**- Access Key ID from earlier
- **Secret Access Key**- Secret Access Key from earlier 

Click **OK** to save.

![Input Key](input-key.png)

Go to the **Jenkins Dashboard &rarr; New Item**

Give your pipeline a name and select the pipeline item then **OK**

![Add Pipeline](add-pipeline.png)

Fill out the following fields for the pipeline, leaving everything else as default:

**GitHub hook trigger for GITScm polling** - Check the box

**Definition** - Pipeline script from SCM
- **SCM** - Git
- **Repository URL** - fork this URL: https://github.com/terence-octo/octopus-underwater-app
- **Credentials** - zone of the repository
- **Branch Specifier** -*/main

Click **SAVE**

## GitHub setup

We will use a sample web application that displays an animated underwater scene with helpful links for this example.

Fork the repository at https://github.com/terence-octo/octopus-underwater-app

We want to set up a webhook so that Jenkins can know when the repository is updated. To do this, go to **Settings &rarr; Webhooks**

![web-hook](webhook.png)

Fill out the following fields, leaving everything else as default.

**Payload URL** - http://[jenkins-url]/github-webhook/

**Content Type** - application/json

**Which events would you like to trigger this webhook?**- Just the push event

Click **Add webhook** to save.

Add a Jenkins file to the root level of the repository. You will need to reference your Amazon ECR repository. Note the following changes required below:

```
node {
    def app

    stage('Clone repository') {

        checkout scm
    }

    stage('Build image') {
        /* Referencing the image name in AWS */

        app = docker.build("[name of your AWS ECR repository]")
    }
    
    stage('Test image') {
    /* Empty */

    }

    stage('Push image') {
        /* Referencing the AWS registry. Tagging with the Jenkins build number and the latest tag */
        
        docker.withRegistry('[the URI of your repository]', 'ecr:[the zone of your AWS ECR repository]:[the ID of your jenkins AWS credentials]') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
}
```

The Jenkinsfile consists of different stages. Jenkins will run each of these stages in order in Jenkins, and if the build fails, you will be able to see which stage failed. Commit your code to GitHub. The commit will trigger a build job in Jenkins. Go to your Jenkins instance URL to see the build.

I had to trigger a Jenkins job via the build now button manually. After this, the webhook triggers would start working on every push.

![Jenkins Success](jenkins-success.png)

After the build finishes, you can go to the Amazon ECR to see a new image built and pushed to the repository. It has tagged the latest push with the Jenkins build number and `latest.`

![ECR Success](ecr-success.png)

In this blog, you have set up a Jenkins pipeline to build a GitHub repository and push it to Amazon ECR. The Jenkinsfile can push to other repositories such as Google or Microsoft. It can also include additional stages depending on the build requirements. Once the image is pushed, you can use a tool like Octopus Deploy to deploy the image to a target environment

Happy Deployments!






