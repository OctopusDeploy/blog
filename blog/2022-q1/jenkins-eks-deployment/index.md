---
title: Build docker image in Jenkinsfile, publish to ECR and deploy to EKS
description: Build docker image in Jenkinsfile, publish to ECR, and deploy to EKS
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

This blog will build a docker image in a Jenkinsfile workflow and publish the image to Amazon Elastic Container Registry (ECR). A deployment will be triggered to deploy to Amazon Elastic Kubernetes Service (EKS). To follow along, you will need:

- An Amazon Web Services Account (AWS)
- A GitHub account
- [A Jenkins instance set up with a pipeline](https://github.com/OctopusDeploy/blog/blob/2022-q1/blog/2022-q1/jenkins-docker-ecr/index.md)

For this blog we will extend the repository to include a deployment YAML file. Jenkins will use this deployment file to deploy to EKS. Add this file to the root level of your repository.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecr-app-jenk-new
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
        - name: simple-octo
          image: 720766170633.dkr.ecr.us-east-2.amazonaws.com/jenkins-ecr-2:latest
          ports:
            - containerPort: 80
              protocol: TCP

```

Create a file named `Jenkinsfile` in the root level of your repository

```

node {

    stage('Clone repository') {

        checkout scm
    }

    stage('Build image') {
        /* Referencing the image name in AWS */

        app = docker.build("jenkins-ecr-2")
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
    
    stage("kubernetes deployment"){
        sh 'kubectl apply -f deployment.yml'
    }
} 

```
Jenkins will clone, build, test, push and deploy the image to an EKS cluster. The deployment is specified in the deployment.yaml file. 

## Jenkins as a CD tool

Jenkins is a continuous integration tool. It is primarily focussed on building and pushing images to a remote repository. Using it as a continuous deployment tool is possible, however it does not have the ability to track a release through various deployment stages. A dedicated continous deployment tool like Octopus Deploy can help in the release management that will arise when managing complex deployments. The next part of the blog series will focus on how Octopus Deploy can be used in the deployment flow to enable the benefits of a dedicated continous deployment tool.

![Jenkins Success](jenkins-success.png)

To view the deployment, we port forward a local port

 kubectl port-forward deployment/ecr-app-underwater  28015:80
 
Navigate to `127.0.0.1:28008` to see the web app

In this blog, you have deployed a web application to EKS with Jenkins

Happy Deployments!
