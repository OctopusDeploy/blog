---
title: A look at using AWS IAM roles in Octopus
description: IAM roles allow users to temporarily assume new permissions or perform work from an EC2 instance without any additional credentials.
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Managing credentials for cloud providers is a challenge, especially when you consider that you won't have the luxury of physical security, meaning one leaked admin key could grant access to your entire account from anywhere in the world. Nor are cloud accounts immune from the preverbal "rf -rf" scenario where an admin account accidentally deletes resources they shouldn't.

IAM roles can be used to provide task specific authorization, and when a role is assigned to an EC2 instance, users with access to that VM can inherit the role.

In this blog post we'll take a look at IAM roles in AWS and learn how they can be used in Octopus.

## Creating a role

Roles are created in the AWS  IAM console. When you create a new role, you will be presented with a list of services that the role will apply to. With the default selection of **AWS service** selected, click the **EC2** link:

![](createrole.png "width=500")

We won't attach any permissions or tags to this role, so skip to the end, give the role a name and create it:

![](finishcreaterole.png "width=500")

Open up the newly created role, click the **Trust relationships** tab, and click the **Edit trust relationship** button:

![](trustrelationships.png "width=500")

The trust relationship is a JSON structure that configures which service or user can inherit the role. Because we selected the EC2 service when creating the role, the EC2 service has been granted the ability to assume it:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

## Assigning a role to an EC2 instance

The role can be assigned to a new EC2 instance when it is created:

![](ec2role.png "width=500")

When you log into this instance, the name of the role assigned to the VM is available from the instance metadata HTTP interface with the command:

```
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

We can also verify the role that commands issued through tools like the AWS CLI assume with the command:

```
aws sts get-caller-identity
```

In both cases we see the role **mytestrole**:

![](rolename.png "width=500")

What this means is that tools that are aware of the instance metadata HTTP interface can operate with the role assigned to the EC2 instance. So long as you have access to the VM, you can interact with AWS with the associated IAM role.

The AWS CLI is one example of a tool that is aware of the instance metadata, and Octopus tentacles are another. We can take advantage of the EC2 IAM roles and Octopus workers to run commands against AWS services without any AWS credentials.

## EC2 as Octopus worker

To connect to the Linux VM we need to create a **SSH Key Pair** account with the certificate used when the VM was created. For VMs created with Amazon AMIs, the username is **ec2-user**:

![](keypairaccount.png "width=500")

We can then create a SSH worker and connect to the VM:

![](worker.png "width=500")

Now we can add a **Run an AWS CLI Script** step, set the script to run on the worker pool containing the worker we just created, and select the option to **Execute using the AWS service role for an EC2 instance**:

![](awsscriptstepauth.png "width=500")

Now if we run the command `aws sts get-caller-identity` in this script, we see the same results as before:

![](awscriptresult.png "width=500")

We now have the ability to perform deployments and execute scripts without needing to share AWS credentials via the worker and the IAM role it assumes from the underlying VM.

## Assuming roles for Kubernetes targets

With Octopus 2020.4.0 it is also possible to interact with an EKS cluster using an IAM role associated with a VM.

However, for an IAM role to have any permissions inside the Kubernetes cluster, we need to map the IAM role to a Kubernetes role. This mapping is done in the `aws-auth` config map in the `kube-system` namespace.

The default contents of this file will look something like this eample. Note however that the existing role mapping is specific to each EKS cluster, as it allows the roles assigned to the nodes to access the cluster. You will need to modify the config map from your cluster rather than copy and paste the one shown below, as using this example directly will cause errors in your cluster:

```YAML
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::968802670493:role/eksctl-k8s-cluster-nodegroup-ng-1-NodeInstanceRole-1FI6JXPS9MDWK
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    []
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
```

In my example, I add a new role mapping between the `mytestrole` IAM role and the Kubernetes `system:master` role:

```YAML
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::968802670493:role/eksctl-k8s-cluster-nodegroup-ng-1-NodeInstanceRole-1FI6JXPS9MDWK
      username: system:node:{{EC2PrivateDNSName}}
    - rolearn: arn:aws:iam::968802670493:role/mytestrole
      username: admin
      groups:
        - system:masters
  mapUsers: |
    []
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
```

Once this config map is applied, we can configure our Kubernetes target to **Execute using the AWS service role for an EC2 instance**:

![](eksiamrole.png "width=500")

We also need to ensure that the target uses the worker connecting to the EC2 instance with the IAM role applied:

![](eksworker.png "width=500")

Our Kubernetes target will now complete a health check without any AWS credentials, and instead using the IAM role assigned to the VM the worker connects to:

![](ekshealth.png "width=500")

## Conclusion

In this post we created an IAM role that can be assigned to an EC2 instance. We logged into the EC2 instance and verified that the AWS CLI showed the local user as having the EC2 role. We then connected a worker to the EC2 instance and run both AWS CLI scripts and a Kubernetes health check from Octopus that also assumed the role assigned to the EC2 instance.

Assigning roles to EC2 instances is a convenient way to remove the need to share AWS credentials, and with Octopus 2020.4, full support for assuming those roles is available in steps and Kubernetes targets.