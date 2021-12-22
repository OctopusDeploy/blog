---
title: Use dynamic build agents to automate scaling in Jenkins
description: With some setup, Jenkins can automatically react to your processing needs, creating extra nodes to manage processes. This post explains 2 setup methods.
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Continuous Integration
  - Jenkins
  - Testing
---

One instance of Jenkins is fine if you’re running a small project with few developers. But you’ll find, as your team and product grow, that single instance may not remain stable. When the number of commits increase, so do the processes Jenkins needs to run, and a lone instance’s performance can soon falter and slow your team down.

Thankfully, Jenkins is a scalable platform. Scalability means as your processing needs grow, Jenkins can grow with them.

Scalability with Jenkins sees one instance act as a controller, directing jobs to other instances known as ‘agents’. The controller knows the capacity of each agent and will send your builds and tests to the most suitable at the time. By using dynamic build agents, this process can happen automatically, allowing Jenkins to react to your needs.

Thanks to virtual environments like Kubernetes and Amazon Web Services, you don’t need a finite number of agents or physical hardware. Jenkins in a dynamic setup is smart enough to spin up new agents if none are suitable, prune unused agents and even replace corrupted installs. And all without manual intervention.

In this post, we look at 2 popular ways to set up dynamic scaling from start to finish, with [Kubernetes](#method1) and [Amazon Web Services (AWS)](#method2).

## Method 1: Scale with Kubernetes {#method1}

Kubernetes is a tool that automatically scales the number of containers needed to keep an application running smoothly. This makes it a terrific choice for help with scaling your Jenkins instance.

By deploying your Jenkins controller to Kubernetes, your CI setup becomes easier to manage, copy or recreate if you have problems.

:::hint
Containers are lite virtual machines that are easily deployable and to most operating systems or cloud services.
:::

### Before you start

This guide is an example only, and you should experiment with scaling before changing an existing Jenkins setup.

In this example, we set up scalability on a local minikube cluster and use the tools below to configure. If following along, you should install the tools in the order listed:

- Docker Desktop – Only needed if you’re on Windows. Make sure Docker Desktop is set to manage Linux containers rather than Windows containers.
- minicube – allows you to install Kubernetes clusters on your computer
- Chocolatey – only needed if you’re on Windows. It’s a command line software management package you’ll use to install…
- Kubectl – a command line tool to control Kubernetes clusters

You can set up scalability with whatever tools you’re used to, but you may need to adjust our instructions slightly.

### Step 1: Create a Jenkins controller image

First, we must create a dockerfile and use that to build a Jenkins controller image.

:::hint
A dockefile is a text file used to create images in Docker, which you can then push to the Docker Hub.
:::

Our example dockerfile will create an image that includes Jenkins, plus the Blue Ocean and Kubernetes plugins.

To create the dockerfile and build an image:

1. Create a text file called ‘Dockerfile’ and add the Jenkins-suggested script. You can add more plugins to the list if you need – just add their names separated by spaces:
   ```FROM jenkins/jenkins:lts-slim 
   # Pipelines with Blue Ocean UI and Kubernetes
   RUN jenkins-plugin-cli --plugins blueocean kubernetes
   ```
1. Save and close the file. If using Notepad on Windows, you must remove the ‘.txt’ file extension.
1. Open a terminal and use the CD command to move to the folder with the file. Use the following command to create the image:
   ```
   docker build . -t [username]/jenkinsdockerfile
   ```
1. The build will take a little while to process, but once complete you’ll see the image in Docker Desktop or with the command `docker images`.

When we create a minikube cluster in the next step, it won't see the image stored locally on your computer as the cluster runs on a virtual environment. Do get around this, we can push the image to Docker Hub.

If on Windows, you can do this in Docker Desktop:

1. Click **Images** on Docker Desktop’s left.
1. Hover your cursor over your image, click the menu icon (3 vertical dots) and select **Push to Hub**.

Alternatively, you can use the following command line in a terminal:

```
Docker push -t [username]/[image name]
```

### Step 2: Create a Kubernetes cluster

Open terminal window and use the following command to create a Kubernetes cluster.

```
minikube start
```

The cluster may take a while to set up. Once done, you should give it a new namespace to make the cluster easier to use and reference. Use the following command in the terminal to create the namespace 'jenkins':

```
kubectl create namespace jenkins
```

### Step 3: Install Jenkins using a YAML deployment file

Copy the following code into a text file and save it as `jenkins.yaml`. Make sure to change the **Image** line to point to your image, for example [Docker username]/[image name].

```
# Service account docs:
# https://support.cloudbees.com/hc/en-us/articles/360038636511-Kubernetes-Plugin-Authenticate-with-a-ServiceAccount-to-a-remote-cluster
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jenkins
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get","list","watch"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins
subjects:
- kind: ServiceAccount
  name: jenkins
---
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: jenkins 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: jenkins 
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins 
        image: [docker username]/jenkins 
        ports:
        - containerPort: 8080
        - containerPort: 50000
        volumeMounts: 
        - name: jenkins-home
          mountPath: /var/jenkins_home 
      volumes:
      - name: jenkins-home 
        emptyDir: { } 
      serviceAccountName: jenkins
---
apiVersion: v1 
kind: Service 
metadata: 
  name: jenkins 
spec: 
  type: NodePort 
  ports: 
  - port: 8080 
    targetPort: 8080 
  selector: 
    app: jenkins
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-agent
spec:
  selector: 
    app: jenkins
  type: NodePort  
  ports:
    - port: 50000
      targetPort: 50000
```

To deploy the image to namespace, run the following command from the file’s directory:

```
kubectl apply -f jenkins.yaml -n jenkins jenkins.yaml
```

The YAML script will now create a Jenkins instance in a Kubernetes pod. The process might take a while, but you can check progress with the following command. It's ready when the **Status** column reads as **Running**:

```
kubectl get pods -n jenkins
```

### Step 4: Find your Jenkins instance URL and connect to it

Run the following command to find out the instance’s IP address and keep a note of it:

```
minikube ip
```

Now we need to discover the port. Run the following command:

```
kubectl get services -n jenkins
```

The **Ports** column will show your instance’s ports as in an 8080:54321 format. Take a note of the 5 numbers after the colon.

We’ll now combine the minikube IP and the port to make up the URL. For example, if the IP is 123.123.123.123 and the port number is 54321, your Jenkins URL would be http://123.123.123.123:54321.

:::hint
If your URL doesn’t work, it’s possible your firewall could be blocking the instance. Speak to your network admin for help. If you’re following along on your own computer and hit this problem, you can temporarily forward your ports with `kubectl port-forward svc/jenkins -n jenkins 8080:8080`. This would make your URL http://localhost:8080.
:::

Go to the URL in your web browser and you should see the **Getting Started** screen. This will ask for a one-time Administrator password. With a Kubernetes cluster you need to find this via command line.

First, we need the name of your Kubernetes pod. Use the following command:

```
kubectl get pods -n Jenkins
```

Your pod name will look something like this: **jenkins-27bc5dcd98-xk9mp**.

Now run the following command, replacing [podname] with the name you just took note of.

```
kubectl logs [podname] -n jenkins
```

Scroll through the result and find the admin password separate by lines of asterisks. Paste this into your Jenkins instance and you can complete setup. See [Jenkins documentation](https://www.jenkins.io/doc/) for how to set up Jenkins the first time.

### Step 5: Get final information and set up the Jenkins plugin

Now we can set up the plugin in Jenkins. Return to Jenkins in your web browser:

1. Click **Manage Jenkins** from the menu.
1. Click **Manage Nodes and Clouds**.
1. Click **Configure Clouds**.
1. Select **Kubernetes** from the dropdown, then click **Kubernetes Cloud details**.
1. Complete the following fields and click **Save**:
   - **Kubernetes URL** – enter `https://kubernetes.default`
   - **Kubernetes Namespace** – enter `jenkins`
   - **Jenkins URL** – enter `http://jenkins.jenkins.svc.cluster.local:8080`
   - **Jenkins Tunnel** – enter `jenkins-agent.jenkins.svc.cluster.local:50000`
1. Scroll to the bottom and click **Pod Templates**, then **Add Pod Template**, and **Pod Template Details**.
1. Complete the following fields and click **Save**:
   - **Name** – enter `jenkins-agent`
   - **Namespace** – enter `jenkins`
   - **Labels** – enter `jenkins-agent`
   - **Usage** - select **Use this node as much as possible** from the dropdown

### Step 6: Test everything’s working

To test that Jenkins will now scale suitably, we can create some simple build jobs to check how they’re distributed.

First, we’ll want to set Jenkins so it won’t run jobs on the controller (unless we tell it otherwise):

1. Click **Manage Jenkins** from the menu.
1. Click **Configure System**.
1. Select **Only build jobs with label expressions matching this node** from the **Usage** dropdown, then click **Save**.

Then we’ll create 2 “Hello World” build jobs:

1. Click **New Item** from the Jenkins dashboard.
1. Enter a suitable name, such as `Testing 1`, select **Freestyle project** and click **OK**.
1. Under the **Build** heading, select **Execute shell** from the dropdown box.
1. Enter `echo "Hello World"` into the **Command box** and click **Save**:
1. Repeat the steps but call your second job `Testing 2`.

Run both build jobs at the same time. If working correctly, they’ll appear in the **Build Queue** on Jenkins’ left. During a build job, they'll appear under the **Build Executor Status** heading with the 'jenkins-agent' prefix we set earlier. You can also check the build history to double-check exactly where the job has run and if it was successful.

## Method 2: Scale with Amazon Web Services (AWS) and the EC2 Fleet plugin {#method2}

An alternative way to manage Jenkins scalability is with EC2 (Amazon Elastic Compute Cloud) containers and the [EC2 Fleet plugin](https://plugins.jenkins.io/ec2-fleet/).

This is a great option for teams who:

- already have access to AWS
- want to easily add scaling to an existing Jenkins instance
- prefer to perform setup using a UI rather than command lines

Though AWS comes at a cost, your financial limits are set at an account level.

### Configure AWS

Sign up to AWS if you don’t have an account or log into your account if you have already access.

#### Step 1: Create a policy

We’ll start by creating a policy to allow access to use AWS EC2 Spot Fleet and the Auto Scaling Group:

1. Click the **Services** menu at the top, select **Security, Identity, & Compliance** then **IAM**.
1. Click **Policies** from the left menu under the **Access management** heading.
1. Click **Create policy** from the right.
1. Create a new policy with the visual editor to use both EC2 Spot Fleet and Auto Scaling Group. Alternatively, use the JSON editor and paste in the [code from the plugin’s setup guide](https://plugins.jenkins.io/ec2-fleet/#plugin-content-3-configure-user-permissions). Click **Next: tags**.
1. Add tags if needed then click **Next: Review**.
1. Give the policy a name and description and click **Create policy**.

#### Step 2: Create an IAM user with programmatic access

Now we’ll create a user with the programmatic access and assign the new policy to it:

1. Click the **Services** menu at the top, select **Security, Identity, & Compliance** and then **IAM**.
1. Click **Users** from the left menu under the **Access management** heading.
1. Click **Add users** from the right.
1. Give your account a suitable name, select **Access key – Programmatic access** and click **Next: Permissions**.
1. Click the **Attach existing policies directly** button. Search for the policy you created in the previous step, tick the checkbox to the left and click **Next: Tags**.
1. Add tags if needed then click **Next: Review**.
1. Review your new user and click **Create user**.

#### Step 3: Set credentials to connect AWS to Jenkins

Next, we’ll create both authentication methods needed to connect AWS to Jenkins.

First, we’ll set an access key ID for your newly created IAM user. This will allow Jenkins to see information related to your AWS setup. To set the access key ID:

1. Click the **Services** menu at the top, **select Security, Identity, & Compliance** and then **IAM**.
1. Click **Users** from the left menu under the **Access management** heading.
1. Search for and click the IAM user you created in step 2.
1. Go to the **Security credentials** tab and click **Create access key**.
1. Make a note of both the access key ID and the secret access key (click **show** to see it) and keep it somewhere safe, such as a password manager. You can only see the secret access key at this stage, you’ll need to create new one if you lose it.

Now we’ll create a key pair. This allows Jenkins to connect to the instances that AWS will create when scaling. To set a key pair:

1. Click the **Services** menu at the top, select **Compute** and click **EC2**.
1. Click **Key Pairs** in the left menu under the **Network & Security** heading.
1. Click **Create key pair**.
1. Complete the following options and click **Create key pair**:
   - **Name** – give the key pair a descriptive name
   - **Key pair type** - leave as **RSA**
   - **Private key file format** – select **.pem**
   - **Tags** – add if needed.
1. The private key will automatically download in a text file. Keep this file safe, you’ll need it later.

#### Step 4: Create an AWS EC2 Spot Fleet or an Auto Scaling Group

You can either an EC2 Spot Fleet or an Auto Scaling Group. In this example, we created an EC2 Spot Fleet, but check the AWS documentation if you want more info about [creating Auto Scaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/GettingStartedTutorial.html).

Before starting, you should also have a clear idea on what Amazon Machine Image (AMI) you want to use. An AMI is a pre-built image that includes the operating system and software your EC2 Fleet will create extra machines from.

Your AMI image should include Java 11 as Jenkins won’t scale without it. We used the [OpenJDK 11 Java 11 Ubuntu 18.04 AMI from the AWS Marketplace](https://aws.amazon.com/marketplace/server/configuration?productId=dd67a7a9-d67f-4c91-a5ee-7e32da4da5c8&ref_=psb_cfg_continue), however, you could build your own if you need something specific. See [AWS’s documentation for more information on AMIs](https://docs.aws.amazon.com/marketplace/latest/userguide/ami-products.html).

To create an EC2 Spot Fleet:

1. Click the **Services** menu at the top, select **Compute** and click **EC2**.
1. Click **Spot Requests** from the left menu under the **Instances** heading.
1. Click **Request Spot Instances**.
1. Set the following options at a minimum – the others you can change as you need:
   - **AMI** – remember to select an AMI that includes Java 11.
   - **Key pair name** – select the key pair name we created in step 3.
   - **Maintain target capacity** – check this box.
1. Scroll to the bottom and click **Launch** when finished.

It’ll take a few moments for AWS to create your fleet. Then we can configure Jenkins.

### Configure Jenkins
#### Step 1: install the EC2 Fleet plugin in Jenkins

To install the EC2 Fleet plugin:

1. Click **Manage Jenkins** from the menu.
1. Click **Manage Plugins**.
1. Click the **Available** tab and start typing ‘EC2 Fleet’ into the **Filter field**. The plugin should appear in the predicted search results.
1. Check the tick box to the left of the plugin then click **Install without restart**.

Jenkins will install the plugin and all dependencies, including other plugins, extensions, and Amazon Software Development Kits (SDKs).

If you haven’t already, you should also install the [Credentials Binding plugin](https://plugins.jenkins.io/credentials-binding/).

#### Step 2: Add the AWS IAM account and key pair to Jenkins

To add your AWS IAM account:

1. Click **Manage Jenkins** from the menu.
1. Scroll down to the **Security** heading and click **Manage Credentials**.
1. Click **Jenkins** under the **Stores scoped to Jenkins** heading.
1. Click **Global credentials (unrestricted)** under the **System** heading.
1. If no credentials exist, you can click the **How about adding some credentials?** link, otherwise click **Add Credentials** from the left.
1. Complete the following fields and click **OK**:
   - **Kind** – select **AWS Credentials** from the dropdown.
   - **ID** – give the credentials a name you’ll use to identify the credentials in Jenkins.
   - **Description** – Enter a meaningful description.
   - **Access Key ID** – Enter the Access Key ID from when you created the IAM account in AWS.
   - **Secret Access Key** – Enter the access string from when you created the IAM account in AWS.

To add your key pair:

1. Click **Manage Jenkins** from the menu.
1. Scroll down to the **Security heading** and click **Manage Credentials**.
1. Click **Jenkins under the Stores scoped to Jenkins** heading.
1. Click **Global credentials (unrestricted)** under the **System** heading.
1. Click **Add Credentials** from the left.
1. Complete the following fields and click **OK**:
   - **Kind** – select **SSH Username with private key** from the dropdown.
   - **ID** – give the credentials a name you’ll use to identify the credentials in Jenkins.
   - **Description** – enter a meaningful description.
   - **Username** – this username is for the AMI you selected.
   - **Private Key** – check the **Enter directly** radio button, click the **Add** button, and paste in the contents of the key pair file you downloaded earlier.

#### Step 3: Connect Jenkins to your AWS EC2 Spot Fleet

Now we will connect Jenkins to the AWS EC2 Spot Fleet:

1. Click **Manage Jenkins** from the menu.
1. Click **Manage Nodes and Clouds**.
1. Click **Configure Clouds**.
1. Select **Amazon EC2 Fleet** from the **Add a new cloud** dropdown. Complete the following fields and click **Save**:
   - **Name** – enter a descriptive name for use in Jenkins.
   - **AWS Credentials** – select your Access Key ID from the **AWS Credentials** dropdown.
   - **Region** – select your fleet’s EC2 fleet’s region from the dropdown.
   - **EC2 Fleet** – any EC2 Spot Fleets connected to your Access Key ID will appear here. You may need to click **Test Connection** if the credentials aren’t showing.
   - **Launcher** – select **Launch agents via SSH**, select the key pair credentials you added in Step 2 and **Non verifying Verification Strategy** from both new dropdown boxes.

#### Step 4: Test everything’s working

After a few minutes, go to your Jenkins dashboard. Under the **Build Executor Status** on the left, you should see both your controller instance and your new EC2 Fleet. If the EC2 fleet is showing any errors, click its name to see logs to identify the problem.

To test that Jenkins will now scale suitably, we can create some simple build jobs to check how they’re distributed.

First, we’ll want to set Jenkins so it won’t run jobs on the controller (unless we tell it otherwise):

1. Click **Manage Jenkins** from the menu.
1. Click **Configure System**.
1. Select **Only build jobs with label expressions matching this node** from the **Usage** dropdown, then click **Save**.

Now we’ll create a “Hello World” build job:

1. Click **New Item** from the Jenkins dashboard.
1. Enter a suitable name, such as `Testing`, select **Freestyle project** and click **OK**.
1. Under the **Build** heading, select **Execute shell** from the dropdown box.
1. Enter `echo "Hello World"` into the **Command** box and click **Save**:

Run the build job and, if working correctly, you’ll see it appear under your fleet in the Build Executor Status on the left. Once complete, you can also check your build history to check exactly where the job ran.

### Change your scaling options in AWS

Once set up and working, you can change how you want Jenkins to scale in AWS.

1. Log into AWS.
1. Click the **Services** menu at the top, select **Compute** and click **EC2**.
1. Click **Spot Requests** from the left menu under the **Instances** heading.
1. Click the **Request ID** of your EC2 Spot Fleet.
1. Scroll to the bottom of the page to see the scaling options. Click the **Configure** button under the **Auto Scaling** heading.
1. Change settings as you need and save.

## What's next

Make sure to watch this space for more Jenkins and build server posts in the coming weeks. For more information on scaling Jenkins, read through their [official scaling documentation](https://www.jenkins.io/doc/book/scaling/).