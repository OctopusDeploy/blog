---
title: Building a Docker image in Jenkinsfile, publishing to ECR, and deploying to EKS
description: As part of our series about Continuous Integration and build servers, learn how to build a Docker image in Jenkinsfile, publish to ECR, and deploy to EKS.
author: terence.wong@octopus.com
visibility: private
published: 2999-01-01-1400
metaImage: blogimage-jenkinsbuilddockerimageinjenkinsfilepublishtoecrdeploytoeks-2022.png
bannerImage: blogimage-jenkinsbuilddockerimageinjenkinsfilepublishtoecrdeploytoeks-2022.png
bannerImageAlt: Illustration of Docker logo being pushed to ECR logo being deployed to EKS logo
isFeatured: false
tags:
 - DevOps
 - CI Series
 - Continuous Integration
 - Jenkins
---

This blog will build a docker image in a Jenkinsfile workflow and publish the image to Amazon Elastic Container Registry (ECR). Jenkins will trigger a deployment to Amazon Elastic Kubernetes Service (EKS). To follow along, you will need:

- An Amazon Web Services Account (AWS)
- A GitHub account
- [A Jenkins instance set up with a pipeline](https://github.com/OctopusDeploy/blog/blob/2022-q1/blog/2022-q1/jenkins-docker-ecr/index.md). This was set up as part of a previous blog.

We will extend the repository to include a deployment YAML file for this blog. Jenkins will use this deployment file to deploy to EKS. Add this file to the root level of your repository.

This blog will use the [Octopus Underwater app repository](https://github.com/OctopusSamples/octopus-underwater-app). You can fork the repository and use the main branch to follow along. The jenkins-deploy branch contains the template files needed to complete the steps in this blog. You will have to replace some values with your own. I have included my values in this blog as a reference.

As we are working with Kubernetes, the agent needs to be configured with a config file. [This documentation shows you how to configure your agent](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/eks/update-kubeconfig.html). AWS also requires the [aws-iam-authenticator binary](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html).

## Amazon Web Services setup

To set up AWS for Jenkins, we need to create an access key and an ECR repository to store the image.

To create an access key, go to **Amazon Console &rarr; IAM &rarr; Users &rarr; [your user] &rarr; Security credentials &rarr; Create Access Key**

Your browser will download a file containing the Access Key ID and the Secret Access Key. These values will be used in Jenkins to authenticate to Amazon.

To create a repository, go to the **Amazon Console &rarr; ECR &rarr; Create Repository**

The ECR requires an image repository set up for each image you publish. Name the repository the name you want the image to have. 

You will see your repository under **Amazon ECR &rarr; Repositories**. Make a note of the zone it is in, in the URI field.

![ECR Repository](ecr-repository.png)

### AWS Cluster setup

[Set up the cluster in AWS using this guide](https://github.com/OctopusDeploy/blog/blob/2022-q1/blog/2022-q1/eks-cluster-aws/index.md)

Create a file named `deployment.yml` in the root level of the repository.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: underwater-app-jenkins 
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
          image: 720766170633.dkr.ecr.us-east-2.amazonaws.com/octopus-underwater-app:latest
          ports:
            - containerPort: 80
              protocol: TCP
          imagePullPolicy: Always

```

Create a file named `Jenkinsfile` in the root level of your repository.

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
                 app = docker.build("octopus-underwater-app")
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
                 sh 'kubectl apply -f deployment.yml'
            }
        }
        
    }
}

```
Jenkins will clone, build, test, push and deploy the image to an EKS cluster. Jenkins does this through the deployment file created earlier.

## Jenkins as a CD tool

Jenkins is a continuous integration tool. Jenkins focuses on building and pushing images to a remote repository. Using it as a continuous deployment tool is possible. However, it cannot track a release through various deployment stages. A  continuous deployment tool like Octopus Deploy can help in the release management of complex deployments. Octopus Deploy enables the benefits of a dedicated continuous deployment tool. (link)

![Jenkins Success](jenkins-success.png)

We will port forward locally to inspect the service. Use this command to inspect the web application. The port 28015 is chosen based on the example in the Kubernetes documentation:

    kubectl port-forward deployment/underwater-app-jenkins  28015:80
    
Go to the IP address http://127.0.0.1:28015/ in the browser to view your web application.

![Octopus Underwater App](octopus-underwater-app.png)

In this blog, you have deployed a web application to EKS with Jenkins.

Happy Deployments!
