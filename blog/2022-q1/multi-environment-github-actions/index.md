---
title: Multi-environment deployments with GitHub Actions, ECR and EKS
description: Build a Docker image in GitHub Actions, Push to ECO and deploy to EKS with Octopus Deploy
author: terence.wong@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - GitHub actions
 - AWS
---

Fork Repository


This blog will build a docker image in a GitHub Actions workflow and publish the image to Amazon Elastic Container Registry (ECR). To follow along, you will need:

- An Amazon Web Services Account (AWS)
- A GitHub account


## Amazon Web Services setup

To set up AWS for GitHub Actions, we need to create an access key and an ECR repository to store the image.

To create an access key, go to **Amazon Console &rarr; IAM &rarr; Users &rarr; [your user] &rarr; Security credentials &rarr; Create Access Key**

Your browser will download a file containing the Access Key ID and the Secret Access Key. These values will be used in Jenkins to authenticate to Amazon.

To create a repository, go to the **Amazon Console &rarr; ECR &rarr; Create Repository**

The ECR requires an image repository set up for each image you want to publish. Name the repository the name you want the image to have. 

Under **Amazon ECR &rarr; Repositories**, you will see your repository. Make a note of the zone it is in, which is in the URI field.

![ECR Repository](ecr-repository.png)

### AWS Cluster setup

[Set up the a cluster in AWS using this guide](https://github.com/OctopusDeploy/blog/blob/2022-q1/blog/2022-q1/eks-cluster-aws/index.md)

## GitHub setup

For this example, we will use a sample web application that displays an animated underwater Octopus named simple-octo.

Fork the repository at https://github.com/terence-octo/simple-octo

Go to **Settings &rarr; Secrets &rarr; New repository secret**

- **REPO_NAME**- the name of the AWS ECR repository you created
- **AWS_ACCESS_KEY_ID**- the Access Key ID from earlier
- **AWS_SECRET_ACCESS_KEY**- the Secret Access Key from earlier

We need to create a workflow file in the repository. A Github Actions workflow contains instructions on how to perform operations on the code repository. There are several pre-built step templates that will allow you to do many different tasks on a code repository. In this example we use a step template that will build and push the code to an AWS ECR repository and deploy it from Octopus Deploy.


Create a file named main.yml in the .github/workflow directory of the root folder. Paste the following code in the main.yml file:

```
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

name: AWS ECR push

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest

    steps:
    - name: Install Octopus CLI
      uses: OctopusDeploy/install-octopus-cli-action@v1.1.1
      with:
          version: latest
    - name: Checkout
      uses: actions/checkout@v2
      
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-2

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push the image to Amazon ECR
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: ${{ secrets.REPO_NAME }}
        IMAGE_TAG: "latest"
        
      run: |
        # Build a docker container and push it to ECR 
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        echo "Pushing image to ECR..."
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
        
    - name: create Octopus release
      run: octo create-release --project aws --version 0.0.i --server=${{ secrets.OCTOPUS_SERVER }} --apiKey=${{ secrets.OCTOPUS_APIKEY }}
        
    - name: deploy Octopus release
      run: octo deploy-release --project aws --version=latest --deployto Production --server=${{ secrets.OCTOPUS_SERVER }} --apiKey=${{ secrets.OCTOPUS_APIKEY }}    
      
```

The yml file is triggered by a push or pull request on the main branch. The steps checks out the code, authenticates and logs into AWS, then builds, tags and pushes the image to Amazon ECR. A similar step template could be used to push to other cloud repositories like Google or Microsoft. 

Commit your changes and go to the **Actions** tab and click the title of your commit message. You will see the various stages of the workflow as it reaches completion.

![GitHub Actions Success](githubactions-success.png)

Go to your Amazon ECR repository to confirm that the image has been pushed successfully. 

![ECR Success](ecr-success.png)

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

This command will point the CLI to your cluster:

    kubectl get deployments

Running this command will get the list of deployments on the cluster. You should see the deployment `octopus-deployment`. Use this name to expose the web application:

    kubectl expose deployment octopus-deployment --type=LoadBalancer --name=my-service
    
This command creates a service named 'my-service' that generates a public IP to view the web application:

    kubectl get services

Run this command, and you will see "pending" under the External-IP. Wait one minute, run again, and you should see a public IP in that field. Go to the IP address in the browser to view your web application.

![RandomQuotes](random-quotes.png "RandomQuotes")