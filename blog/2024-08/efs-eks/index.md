---
title: How to do connect EFS to an EKS cluster
description: Learn how to connect EFS storage to an EKS cluster.
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

I recently worked with a customer to configure the [Octopus Kubernetes Agent](https://octopus.com/docs/kubernetes/targets/kubernetes-agent) on an AWS Elastic Kubernetes Service (EKS) cluster.  The Octopus Kubernetes Agent requires a [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/) to provide volumes so it has a file system to work with. The customer wanted to use the Elastic File System (EFS) for the storage class.  I wasn't able to find a walkthrough which describes the steps necessary to configure EFS to work with EKS and instead had to rely on a combination of AWS documentation and blog posts to get it configured.  This led to a frustrating amount of incremental successes as I went from brick wall to brick wall trying to get the two technologies to work together.  This blog post is meant to cover what is required to connect the two technologies in a single location so you don't have to struggle like I did.

## Pre-requisites for this post
This post assumes that you have an EKS cluster and it's associated roles, node groups, security groups, etc... already configured.  If you're starting from scratch, the easiest method is to use the [eksctl](https://eksctl.io/) command-line tool.  The eksctl tool provisions all of the necessary items for a functional EKS cluster with a single, easy to use [command](https://eksctl.io/getting-started/#basic-cluster-creation).  The cluster created for this post used eksctl and also utlizes several other eksctl commands to complete the configuration.

## Resources required for this post
Along with the EKS cluster, there are a few items that will be created to demonstrate how to connect EFS to EKS:
- An EFS File System
- An IAM OIDC provider for your cluster
- An IAM Service Account and Role for the CSI driver

### Create an EFS file system
Creating an EFS file system is relatively straight forward process.  For simplicity, this post uses the UI to create the EFS file system.  I will walk through the steps to create the EFS file system and configure it.

#### Creating the EFS file system using the UI
An EFS file system can be created in just a few clicks:
1. Log into the AWS console.
2. Click on the search bar and enter `efs`.  This will filter the services list, EFS should be the first result.  

![Search for the EFS service](aws-services-efs.png)

3. Click on EFS to be taken to the EFS service.
4. Click on the **Create file system** button

![Create new EFS file system](aws-efs-create.png)

5. Give the EFS file system a name and select the VPC to use.  **Note:** Make sure you select the correct VPC so the two technologies can communicate.

![Enter a name and choose the appropriate VPC](aws-efs-filesystem.png)

The above image selects the VPC that was created by the eksctl tool.

6. Once created, click on the entry to configure settings

![Click on the newly created entry](aws-efs-myefs.png)

7. Copy the EFS ID number and save it in something like Notepad, we'll need it for later

![Copy the EFS ID number](aws-efs-myefs-id.png)

8. Click on the `Network` tab to view the assigned `Security groups`.  EFS selects the default security groups for the VPC when created.

![Review the assigned security groups](aws-efs-network.png)

In my case, the eskctl tool created its own security groups, so I needed to update these to the appropriate groups.  To update the groups, click **Manage**, then update the groups assigned to the EKS Nodes.

![Assign the correct security groups](aws-efs-security-groups.png)

This was a stumbling point for me, one of the various blog posts I read on the subject briefly pointed out this as a potential issue.

### Create an IAM OIDC provider for your cluster
EKS clusters come with an OpenID Connect provider URL as part of the EKS offering.  This allows you to create an OIDC provider for your cluster to provide secure authentication between EFS and your cluster.  You can find the documentation on this process [here](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html).

The documentation describes two different ways to configure the provider:
- Using eksctl
- Using the UI

For this post, we'll use the eksctl method.  The documenation provides a  `bash` example to programatically create the OIDC provider.  I was using a Windows machine so I've converted it to `PowerShell` (the step comments are the step numbers from the documentation).

```powershell
# Step 1
$clusterName = "MyEKSCluster"
$regionCode = "us-west-2"
$cluster = (aws eks describe-cluster --name $clusterName --region $regionCode)

$cluster = ($cluster | ConvertFrom-JSON)

# Step 2
$oidc_id = ($cluster.cluster.identity.oidc.issuer.split("/"))[4]

# Step 3
./eksctl utils associate-iam-oidc-provider --cluster $clusterName --region $regionCode --approve
```

### Create an IAM Service Account and Role for the CSI driver
To connect EFS to your EKS cluster, you will need to create an IAM Service Account and Role for the CSI driver.  This was yet another AWS documentation page that I needed to reference to configure the solution ([see AWS documentation page](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html)).  Similar to the other AWS documenation page, the provided example was only available in `bash`, here is the `PowerShell` equivalent

```powershell
$clusterName="MyEKSCluster"
$roleName="AmazonEKS_EFS_CSI_DriverRole_Blog"
$regionCode="us-west-2"
./eksctl create iamserviceaccount `
    --name efs-csi-controller-sa `
    --namespace kube-system `
    --cluster $clusterName `
    --role-name $roleName `
    --role-only `
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEFSCSIDriverPolicy `
    --approve `
    --region $regionCode

# Update trust policy
$trust_policy = (aws iam get-role --role-name $roleName)
$trust_policy = $trust_policy.Replace(":efs-csi-controller-sa", ":efs-csi-*")
$trust_policy = $trust_policy.Replace("StringEquals", "StringLike")
$trust_policy = ($trust_policy | ConvertFrom-JSON)

# Write policy to file
Set-Content -Path "policy.json" -Value "$($trust_policy.Role.AssumeRolePolicyDocument | ConvertTo-JSON -Depth 10)"

# Update the Trust Policy on the role
aws iam update-assume-role-policy --role-name $roleName --policy-document file://policy.json
```

## Configuring the EKS cluster to use EFS
Now that we have the IAM Service Account, Role, and the EFS file system created, we can configure the EKS cluster to utilize them.  To connect EFS to EKS, you'll need to do the following:
- Add the Amazon EFS CSI Driver to the cluster
- Set the IAM Service Account role for the EFS CSI Driver
- Create the Kubernetes `Storage Class` resource for the EFS CSI Driver

### Add the Amazon EFS CSI Driver
The Amazon EFS CSI Driver is an `Add-on` to an EKS cluster.  To add the add-on, first navigate to your cluster and perform the following steps:

1. Click on **Add-ons**

![Click on Add-ons](aws-eks-mycluster.png)

2. Click on the **Get more add-ons** button

![Click the Get more add-ons button](aws-eks-get-more-addons.png)

3. Select the `Amazon EFS CSI Driver` from the list and click **Next**

![Select Amazon EFS CSI Driver and click Next](aws-eks-efs-driver.png)

4. Select the IAM Role you created earlier and click **Next**

![Select the IAM Role](aws-eks-efs-driver-role.png)

5. Review the configuration and click **Create**

![Review add-on configuration and create](aws-eks-efs-review.png)

## Create a StorageClass resource on your cluster
The final step is to create a `StorageClass` resource in your EKS cluster to use the EFS file system you created.  Create a YAML file with the following (you will need the file system ID value we copied earlier)

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-XXXXXXXXXXXXX # The file system ID we copied from earlier
  directoryPerms: "700"
  gidRangeStart: "1000" # optional
  gidRangeEnd: "2000" # optional
  basePath: "/dynamic_provisioning" # optional
  subPathPattern: "${.PVC.namespace}/${.PVC.name}" # optional
  ensureUniqueDirectory: "true" # optional
  reuseAccessPoint: "false" # optional
```

Apply this to your cluster and you're done!  Whew!

## Conclusion

The information to connect EFS to EKS _is_ readily available, however, it's spread across multiple AWS documentation pages that don't seem to reference each other.  My hope is that by consolodating the steps in one spot, it will save others from the frustration of having to piece together the necessary steps.

Happy deployments!
