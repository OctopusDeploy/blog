---
title: How to assign an IAM role as an EKS Pod Identity for the Octopus Kubernetes worker agent.
description: Learn how to configure an IAM role to be used for the Octopus Kubernetes worker agent.
author: shawn.sesna@octopus.com
visibility: private
published: 3020-01-01
metaImage: to-be-added-by-marketing
bannerImage: to-be-added-by-marketing
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: <!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->
 - DevOps
 - Company
 - Product
 - Engineering
---


See https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags.


The release of the Kubernetes Agent Worker allows customers to scale workers on demand.  The Agent Worker automatically scales pods to carry out tasks using a single worker registration.  This means customers no longer need to provision worker machines or containers, which may be idle most of the time.  Assigning workers an IAM role is common practice when using AWS to interact with other AWS resources securely.  In this post, I will demonstrate how to attach an IAM role to the agent worker pods.


## Methods for assigning the IAM roles to the Kubernetes Agent Worker
There are two methods that can be used to assign an IAM role to Elastic Kubernetes Service (EKS) pod:
- IAM Roles for Service Accounts
- EKS Pod Identity


The steps necessary to configure these differ depending on which method you choose.


### IAM Roles for Service Accounts
The first method is to assign the IAM role to the Kubernetes Service Account that the pods will run as.  Configuring a Kubernetes Service Account to use an IAM role consists of the following steps:
- Configure the OIDC provider for the EKS cluster
- Creating or updating the IAM role to be used with a Service Account
- Specifying the Service Account (sa) annotation in the Octopus Deploy Helm Chart values


#### Configure the OIDC provider for the EKS cluster
To use the IAM Roles for Service Account method, you will need to configure the OIDC provider for the EKS cluster.  Configuration of the OIDC provider can be done via:
- AWS Management Console
- [eksctl](https://eksctl.io/) command line utility


##### AWS Management Console
This post will not cover the steps for the AWS Management Console, see [AWS official documentation](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html) for more details.


##### Using EKSCTL
The `eksctl` utility is included in the `octopusdeploy\worker-tools` Docker image and can be used to automate the configuration of the OIDC provider.  Below is an example PowerShell script to configure the OIDC provider using the eksctl command-line utlity:


```powershell
# Assign variables
$clusterName = $OctopusParameters["Project.AWS.EKS.Cluster.Name"]
$region = $OctopusParameters["Project.AWS.Region.Code"]


# Enable OIDC on the cluster
eksctl utils associate-iam-oidc-provider --cluster $clusterName --region $region --approve
```


#### Create/update the IAM Role
Aside from granting permissions to the IAM role you wish to use, you will also need to configure the **Trust relationships** for the role.  Below is the minimum JSON needed for the Service Account to be able to assume the role:


```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account-id>:oidc-provider/oidc.eks.<region>.amazonaws.com/id/<eks-cluster-id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.<region>.amazonaws.com/id/<eks-cluster-id>:sub": "system:serviceaccount:<namespace>:<service-account-name>"
        }
      }
    }
  ]
}
```


:::info
The **cluster id** is required for the Trust relationship as indicated above.  The id can be found on the **Overview** tab of an EKS cluster on the AWS console.  The id is in the first part of the `API server endpoint` or the last part of the `OpenID Connect provider URL`


![EKS Cluster Id](aws-eks-cluster-id.png)
:::


#### Create/update the Kubernetes Service Account
The last step is to add the annotation to the Service Account.  This is done by assigning the `scriptPods.serviceAccount.annotations` value for the Octopus Deploy Helm Chart


```bash
helm upgrade --install --atomic \
--set agent.acceptEula="Y" \
--set agent.serverUrl="https://YourUrl/" \
--set agent.serverCommsAddresses="{http://octopusserver1:10943,http://octopusserver2:10943}" \
--set agent.space="Default" \
--set agent.name="MyCluster" \
--set agent.deploymentTarget.initial.environments="{development}" \
--set agent.deploymentTarget.initial.tags="{Octopub-Audit,Octopub-Frontend,Octopub-Product}" \
--set agent.deploymentTarget.enabled="true" \
--set agent.bearerToken="BearerTokenValue" \
--set scriptPods.serviceAccount.annotations."eks\.amazonaws\.com/role-arn"="arn:aws:iam::<account-id>:role/<iam-role-name>" \
--version "2.*.*" \
--create-namespace --namespace octopus-agent-mycluster \
mycluster \
oci://registry-1.docker.io/octopusdeploy/kubernetes-agent
```


With all of the above configured, all pods used for executing worker activities will be assigned the specified IAM role.


#### Adding annotation after agent is already installed
The annotation can also be added manually using the following command:


```
kubectl annotate sa <service-account-name> eks.amazonaws.com/role-arn=arn:aws:iam::<account-id>:role/<iam-role-name> -n <namespace>
```


### EKS Pod Identity
An alternative method is to use the Amazon EKS Pod Identity Agent addon.  This addon facilitates the IAM role assignment to pods created by the Agent.  There are a couple of pre-requisites to complete before installing the addon:
- Create/update the IAM Role
- Create a Pod Identity Association
- Add the `Amazon EKS Pod Identity Agent` addon to your EKS cluster


#### Create/update the IAM Role
You'll first need to configure an IAM Role for the pods to use.  Below is the minimum JSON for the **Trust relationships** needed for the addon to work:


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowEksAuthToAssumeRoleForPodIdentity",
            "Effect": "Allow",
            "Principal": {
                "Service": "pods.eks.amazonaws.com"
            },
            "Action": [
                "sts:AssumeRole",
                "sts:TagSession"
            ]
        }
    ]
}
```


#### Create a Pod Identity Association
With the role configured, run the `aws eks create-pod-identity-association` CLI command to associate the role to the service account `octopus-agent-scripts`


```powershell
aws eks create-pod-identity-association --cluster-name <ClusterName> --role-arn <RoleArn> --namespace "<Namespace>" --service-account octopus-agent-scripts
```


#### Add the Amazon EKS Pod Identity Agent addon to your EKS cluster
The final step is to install the [Amazon EKS Pod Identity Agent](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html) addon to your EKS cluster.  This agent grants the identity to pods when they're created.  Adding the add-on can be done manually using the AWS Management Console or programmatically using the AWS CLI.  This post will not cover installing the addon via the AWS Management Console.


```powershell
# Add pod identity addon
aws eks create-addon --cluster-name <ClusterName> --addon-name "eks-pod-identity-agent"
```


#### Example script to automate
All of the above commands can conveniently be placed within a [Runbook](https://octopus.com/docs/runbooks) to automate installation when spinning up a new cluster


```powershell
# Assign variables
$clusterName = $OctopusParameters["Project.AWS.EKS.Cluster.Name"]
$region = $OctopusParameters["Project.AWS.Region.Code"]
$roleName = $OctopusParameters["Project.AWS.EC2.Role.Name"]


# Get role
$iamRole = ((aws iam get-role --role-name $roleName) | ConvertFrom-Json)


# Attach service role
aws eks create-pod-identity-association --cluster-name $clusterName --role-arn $iamRole.Role.Arn --namespace "octopus-worker-awsagentworker" --service-account octopus-agent-scripts


# Add pod identity addon
aws eks create-addon --cluster-name $clusterName --addon-name "eks-pod-identity-agent"
```


## Conclusion
In this post, I demonstrated the two methods that can be used to apply IAM roles to the Octopus Deploy Kubernetes Agent Worker pods to facilitate secure communication and or access to other AWS resources.


Happy deployments!