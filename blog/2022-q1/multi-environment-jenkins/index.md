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


This blog will build a docker image in a Jenkinsfile workflow and publish the image to Amazon Elastic Container Registry (ECR). A deployment will be triggered in Octopus Deploy to deploy to Amazon Elastic Kubernetes Service. To follow along, you will need:

- An Amazon Web Services Account (AWS)
- A GitHub account
- [A Jenkins instance set up with a pipeline](https://github.com/OctopusDeploy/blog/blob/2022-q1/blog/2022-q1/jenkins-docker-ecr/index.md)

Extend the pipeline with Octopus Release and Deploy commands

```

node {
    def app

    stage('Clone repository') {

        checkout scm
    }

    stage('Build image') {
        /* Referencing the image name in AWS */

        app = docker.build("jenkins-ecr")
    }
    
    stage('Test image') {
    /* Empty for test purposes */

    }

    stage('Push image') {
        /* Referencing the AWS registry. Tagging with the Jenkins build number and the latest tag */
        docker.withRegistry('https://720766170633.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2:aws-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
    
        stage('Create and Deploy Release') {
        octopusCreateRelease additionalArgs: '', cancelOnTimeout: false, channel: '', defaultPackageVersion: '', deployThisRelease: false, deploymentTimeout: '', environment: "Production", jenkinsUrlLinkback: false, project: "jenkins-octo", releaseNotes: false, releaseNotesFile: '', releaseVersion: "1.0.${BUILD_NUMBER}", tenant: '', tenantTag: '', toolId: 'Default', verboseLogging: false, waitForDeployment: false
        octopusDeployRelease cancelOnTimeout: false, deploymentTimeout: '', environment: "Production", project: "jenkins-octo", releaseVersion: "1.0.${BUILD_NUMBER}", tenant: '', tenantTag: '', toolId: 'Default', variables: '', verboseLogging: false, waitForDeployment: true
            
    }
}

```



![Octopus Success](octopus-success.png)


## Set Up Octopus Deploy

In your Octopus Deploy instance, create a project called `aws` by going to **Project, Add Project** Add the `aws` title and click **Save**.

Set up a Production Environment by going to **Infrastructure, Environements, Add Environment**. Give it a name and click **Save**

We need to set up the Amazon account to deploy to EKS. Go to **Infrastructure, Accounts, Add Account, AWS Account**. Give it a name and fill out the **Access Key ID and Secret Access Key** from earlier.

Set up your AWS Kubernetes cluster as a deployment target in Octopus Deploy by going to **Infrastructure, Deployment Targets, Add Deployment Target, Kubernetes Cluster, Add**


## Deploy to EKS step

In your `aws` project, go to **Process, add deployment step, Kubernetes, Deploy Kubernetes Containers**

Add the following YAML into the YAML section

```
# This YAML exposes the fields defined in the UI. It can be edited directly or have new YAML pasted in.
# Not all available Kubernetes properties are recognized by the form exposed in the UI, and unrecognized properties are ignored during import.
# If the required properties are not supported by this step, the 'Deploy raw Kubernetes YAML' step can be used to deploy YAML directly to Kubernetes, and supports all properties.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecr-app
  labels:
    app: random-quotes
spec:
  selector:
    matchLabels:
      octopusexport: OctopusExport
  replicas: 3
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: random-quotes
        octopusexport: OctopusExport
    spec:
      containers:
        - name: random-quotes
          image: terence.wong
          ports:
            - containerPort: 80
              protocol: TCP

```

Click **SAVE

## Deploy in Jenkins


Make a change to the code on GitHub and the build will trigger in Jenkins and Octopus

![Jenkins Octopus Icon](jenkins-octo-icon.png "Jenkins Octopus Icon")

You can navigate directly to the Octopus Deploy instance and deployment through the Octopus Icon.


This command will point the CLI to your cluster:

    kubectl get deployments

Running this command will get the list of deployments on the cluster. You should see the deployment `octopus-deployment`. Use this name to expose the web application:

    kubectl expose deployment octopus-deployment --type=LoadBalancer --name=my-service
    
This command creates a service named 'my-service' that generates a public IP to view the web application:

    kubectl get services

Run this command, and you will see "pending" under the External-IP. Wait one minute, run again, and you should see a public IP in that field. Go to the IP address in the browser to view your web application.

![RandomQuotes](random-quotes.png "RandomQuotes")