---
title: Multi environment deployments with Jenkins and Octopus Deploy
description: Set up a multi-environment deployment with Jenkins and Octopus Deploy
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


This blog will build and push the Octopus Underwater App to Amazon Elastic Container Registry (ECR). Jenkins will trigger a deployment in Octopus Deploy. Octopus Deploy will deploy the App to Amazon Elastic Kubernetes Service. To follow along, you will need:

- An Amazon Web Services Account (AWS)
- A GitHub account
- [A Jenkins instance set up with a pipeline](https://github.com/OctopusDeploy/blog/blob/2022-q1/blog/2022-q1/jenkins-docker-ecr/index.md)

This blog will use the [Octopus Underwater app repository](https://github.com/terence-octo/octopus-underwater-app). You can fork the repository and use the main branch to follow along. The jenkins-octopus branch contains the template files needed to complete the steps in this blog. You will have to replace some values with your own. I have included my values in this blog as a reference.


Extend the pipeline with Octopus Release and Deploy commands. Create a Jenkinsfile and paste the following code.

```



pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
         stage('Clone repository') { 
            steps { 
                script{
                checkout scm
                }
            }
        }
        
        stage('Build') { 
            steps { 
                script{
                 app = docker.build("underwater")
                }
            }
        }
        stage('Test'){
            steps {
                 echo 'Empty'
            }
        }
        stage('Push') {
            steps {
                script{
                        docker.withRegistry('https://720766170633.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2:aws-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                    }
                }
            }
        }
        stage('Deploy'){
            steps {
                script{
                    octopusCreateRelease additionalArgs: '', cancelOnTimeout: false, channel: '', defaultPackageVersion: '', deployThisRelease: false, deploymentTimeout: '', environment: "Development", jenkinsUrlLinkback: false, project: "underwater-octo", releaseNotes: false, releaseNotesFile: '', releaseVersion: "1.0.${BUILD_NUMBER}", tenant: '', tenantTag: '', toolId: 'Default', verboseLogging: false, waitForDeployment: false
                    octopusDeployRelease cancelOnTimeout: false, deploymentTimeout: '', environment: "Development", project: "underwater-octo", releaseVersion: "1.0.${BUILD_NUMBER}", tenant: '', tenantTag: '', toolId: 'Default', variables: '', verboseLogging: false, waitForDeployment: true
                }
            }
        }
        
    }
}

```

## Set Up Octopus Deploy

In your Octopus Deploy instance, create a project called `aws` by going to **Project, Add Project** Add the `aws` title and click **Save**.

Set up a Production Environment by going to **Infrastructure, Environments, Add Environment**. Give it a name and click **Save**

We need to set up the Amazon account to deploy to EKS. Go to **Infrastructure, Accounts, Add Account, AWS Account**. Give it a name and fill out the **Access Key ID and Secret Access Key** from earlier.

Set up your AWS Kubernetes cluster as a deployment target in Octopus Deploy by going to **Infrastructure, Deployment Targets, Add Deployment Target, Kubernetes Cluster, Add**

We need to add the Amazon feed to the Octopus Instance. Go to **Library &rarr; External Feeds &rarr; Add Feed** and select the **AWS Elastic Container Registry**. Enter your **Access Key ID, Secret Access Key and Zone** of your registry. 


## Deploy to EKS step

In your `aws` project, go to **Process, add deployment step, Kubernetes, Deploy Kubernetes Containers**

Add the following YAML into the YAML section.

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecr-app-underwater
  labels:
    app: octopus-underwater-app
spec:
  selector:
    matchLabels:
        app: octopus-underwater-app
  replicas: 3
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: octopus-underwater-app
    spec:
      containers:
        - name: octopus-underwater-app
          image: 720766170633.dkr.ecr.us-east-2.amazonaws.com/underwater
          ports:
            - containerPort: 80
              protocol: TCP
          imagePullPolicy: Always
```

Click **SAVE**.

## Deploy in Jenkins

To connect Jenkins to Octopus you will need to install the [Octopus Plugin](https://plugins.jenkins.io/octopusdeploy/). Go to **Dashboard &rarr; Manage Plugins**. If the plug in is not already installed, search for the Octopus Plugin in the available tab and install the plugin. [Follow this guide on how to set up Jenkins and the Octopus Plugin](https://octopus.com/docs/packaging-applications/build-servers/jenkins)

Once the Octopus plugin is set up, make a change to the code on GitHub, and the build will trigger in Jenkins and Octopus.

![Jenkins Octopus Icon](jenkins-octo-icon.png "Jenkins Octopus Icon")

We will port forward locally to inspect the service. Use this command to inspect the web application:

    kubectl port-forward deployment/octopus-underwater-app  28019:80
    
Go to the IP address in the browser to view your web application.

![Octopus Underwater App](octopus-underwater-app.png)

## The benefits of a dedicated CD tool

Octopus is a dedicated continuous delivery tool. It natively supports release management. Jenkins defines environments through the pipeline file. They are dividers to the pipeline code. In Octopus, environments are dedicated spaces. Octopus Deploy makes it easy to stop a deployment at a staging environment before it gets pushed to production. The following dashboard shows the capability. Different releases are present in different environments, and it is easy to see the stage where releases are in the lifecycle.

Jenkins is a continuous integration tool. It can do some parts of CD, but not all. Jenkins is commonly used to build and push images to a central repository. Octopus Deploy can interface with several different repositories and manage the deployment process. This separation of concerns allows Jenkins and Octopus Deploy to focus on what they are good at, enabling happier deployments.



![Release Management](release-management.png "Release Management")
