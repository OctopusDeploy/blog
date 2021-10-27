---
title: Build docker image in Workflow and publish to ECR
description: Build docker image in Workflow and publish to ECR
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

Fork Repo [https://github.com/terence-octo/simple-octo]

AWS IAM

AWS Registry

GitHub Settings

**Settings, Secrets, New repository secret**

- **REPO_NAME**
- **AWS_ACCESS_KEY_ID**
- **AWS_SECRET_ACCESS_KEY**

workflow file

```
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

name: AWS ECR push

jobs:
  deploy:
    name: Deploy
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

    - name: Build, tag, and push the image to Amazon ECR
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: ${{ secrets.REPO_NAME }}
        IMAGE_TAG: simple-octo
      run: |
        # Build a docker container and push it to ECR 
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        echo "Pushing image to ECR..."
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"
```

GitHub Actions Success

![GitHub Actions Success](githubactions-success.png)

ECR success
![ECR Success](ecr-success.png)


