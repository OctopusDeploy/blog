---
title: Create an EKS cluster in AWS
description: Create an EKS cluster in AWS
author: terence.wong@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - DevOps
 - AWS
---

You will need:

- AWS account
- kubectl
- eksctl
- IAM Permissions


## CLI interface

**AWS Console &rarr; IAM &rarr; Users &rarr; Add Users**

**Users &rarr; Security Credentials &rarr; Create Access Key**

Command line aws login - put in access keys

**Users &rarr; [Your User] &rarr; Add Policy**

https://github.com/weaveworks/eksctl/blob/main/userdocs/src/usage/minimum-iam-policies.md

createservicelinkedrole


```
eksctl create cluster \
--name my-cluster \
--region us-west-2 \
--fargate
```

    kubectl get svc
    
![Command One](command-1.png)
    
fsdf

    kubectl get nodes -o wide
    
![Command Two](command-2.png)

fsda

    kubectl get pods --all-namespaces -o wide

![Command Three](command-3.png)
blah

    eksctl delete cluster --name my-cluster --region us-west-2
    
## Console interface

![EKS Console](eks-console.png)


## Check Clusters

![EKS Two Clusters](eks-two-clusters.png)


