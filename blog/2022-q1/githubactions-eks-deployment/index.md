---
title: Build a workflow in GitHub Actions, push to ECR and deploy to EKS
description: Build a workflow in GitHub Actions, push to ECR and deploy to EKS
author: terence.wong@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - DevOps
 - GitHub Actions
---

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

First, we need to create a deployment YAML file for GitHub actions to deploy to EKS. Create a file named `git-deployment.yml` in the root level of your repository with the following code:

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecr-app-git
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
          image: 720766170633.dkr.ecr.us-east-2.amazonaws.com/github-ecr:latest
          ports:
            - containerPort: 80
              protocol: TCP

```



We need to create a workflow file in the repository. A Github Actions workflow contains instructions on how to perform operations on the code repository. There are several pre-built step templates that will allow you to do many different tasks on a code repository. In this example we use a step template that will build and push the code to an AWS ECR repository and deploy it to EKS.


Create a file named main.yml in the .github/workflow directory of the root folder. Paste the following code in the main.yml file:

```

on: push
name: deploy
jobs:
  deploy:
    name: deploy to cluster
    runs-on: ubuntu-latest
    steps:
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

    - name: Push image and deploy
      uses: observian/littleci-littlecd-eks@master
      with:
        access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        account_id: ${{ secrets.AWS_ACCOUNT_ID }}
        repo: github-ecr
        region: us-east-2
        tags: 0.1.1.${{ github.run_number }},${{ github.sha }}
        eks_cluster_name: my-cluster-cli
        k8s_manifest: git-deployment.yml
        k8s_image_tag: 0.1.1.${{ github.run_number }}

```

![GitHub Success](github-success.png)

## GitHub Actions as a CD tool

GitHub Actions is able to build, push and deploy a GitHub repository to a Kubernetes cloud platform like EKS. To integrate with cloud platforms and other tools, it relies on community built step templates. In my experience with the tools, these step templates are not standardised. I tried several different templates. Some worked slightly differently to another depending on the variables it was calling.

A tool like Octopus also uses step templates, but they are standardised across the Octopus Deploy application. Gaining familiarity with the Octopus Deploy application translates when using newer step templates as the expereince is designed to be consistent.

I found that using a new step template in GitHub required a layer of learning each time.






