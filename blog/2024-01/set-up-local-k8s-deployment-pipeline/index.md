---
title: Set up a local Kubernetes deployment pipeline
description: We teach you how to set up a local Kubernetes deployment pipeline, so you can experiment without risk.
author: andrew.corrigan@octopus.com
visibility: public
published: 2024-01-15-1400
metaImage: blogimage-kubernetes-generic-4-2023.png
bannerImage: blogimage-kubernetes-generic-4-2023.png
bannerImageAlt: Kubernetes icon peeking out of package with someone sitting in front of it with a laptop.
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
  - GitHub Actions
  - Docker
---

Kubernetes (also known as K8s) is a container management tool that solves complexity for software teams who might:

- Need to scale their application suddenly or automatically due to an influx of customers
- Manage different versions of their software for many customers, as with multi-tenancy
- Want fluidity and flexibility in their infrastructure to react to the ebb and flow of traffic
- Use a microservices software architecture

Though it solves complexity, Kubernetes is complex itself. It's known for causing a few headaches for new adopters.

A great way to start with Kubernetes, however, is to set up a local instance. A local instance lets you:

- Figure out how things work without pressure
- Make those early mistakes without real environment consequences
- Build out the local instance as you get more comfortable

In this post, we guide you through setting up a local Kubernetes deployment pipeline that:

- Builds a simple containerized application using GitHub Actions
- Pushes the application's image to Docker Hub
- Deploys the app to your local Kubernetes instance using Octopus Server

## Before you start

Aside from a couple of external services, this guide creates the pipeline on Microsoft Windows.

We use [Canonical's MicroK8s](https://microk8s.io/) for the local Kubernetes instance. We consider it the easiest entry point as it includes everything you need to get started quickly (with a few minor quirks, which we'll cover).

We reference the MicroK8s commands and steps to complete this guide as you need them. If you want to know more about MicroK8s and Kubernetes commands beyond this post, [MicroK8s' documentation](https://microk8s.io/docs) is fantastic. It uses easy-to-understand language and has simpler explanations than most other Kubernetes guides.

MicroK8s needs Microsoft's HyperV and Windows Hypervisor Platform enabled to run clusters as it hosts MicroK8s on an Ubuntu virtual machine. Use Windows search to find the **Turn Windows features on or off** option and tick boxes for both features in the list. If you use VMWare on the same computer, we recommend fully uninstalling it (or at least stopping its services) before enabling HyperV, as it might cause you problems later on.

You also need [GitHub](https://github.com/) and [Docker Hub](https://hub.docker.com/) accounts.

Lastly, we use Octopus Server to deploy our Underwater App to your cluster. If you don't already have a license for Octopus Server, don't worry - we explain how to sign up for a free trial in the guide.

## Step 1: Install MicroK8s on Windows

Installing MicroK8s is easy. For Windows, download and run the [MicroK8s installer](https://microk8s.io/).

Make sure you select `MicroK8s` and `Kubectl` when asked what commands to add to your system PATH. Leave all other steps as default.

The installer will set up your MicroK8s instance. You can check it's running by using the `microk8s status --wait-ready` command in Windows Terminal.

You can stop and start MicroK8s with the following commands. Stopping MicroK8s when not in use will save system resources and battery performance:

- `microk8s start`
- `microk8s stop`

:::hint
MicroK8s seems to regularly fail starting the first time, advising an 'Exit Code 2' error. Just run the command again and it should start.
:::

## Step 2: Configure MicroK8s networking

You can experience problems running MicroK8s on Windows due to the way HyperV allocates IP addresses when it starts. We can solve those problems by adding an address string to MicroK8s' DNS settings that removes reliance on IP addresses. That way, the other tools in our pipeline always see our clusters.

1. Open a Windows Terminal and connect directly to Ubuntu by running `multipass shell microk8s-vm`.
1. Run `sudo nano /var/snap/microk8s/current/certs/csr.conf.template`.
1. Use the arrow keys to move the cursor down to the DNS section and add the following entry to the DNS list: `DNS.6 = microk8s-vm.mshome.net`
1. Press Ctrl and X to exit. Type `Y` to confirm the changes. Press **Enter** to confirm the file you're overwriting.

Now run `microk8s config` in a fresh Windows Terminal for cluster information. Make a note of the 'Server' field as you need it later. It looks something like `https://111.111.111.111:12345`. Specifically, note the numbers after the colon. This is your cluster's networking port number.

1. Run `microk8s stop` in your Windows Terminal. Wait for it to stop.
1. Open a Windows Explorer and browse to `C:\Users\*your profile*\AppData\Local\MicroK8s` and open the 'config' file. Select **Notepad** if you don't already have a default app. The 'AppData' folder may be hidden by default on Windows. You can make it visible by changing the [view settings for your profile folder](https://support.microsoft.com/en-us/windows/view-hidden-files-and-folders-in-windows-97fbc472-c603-9d90-91d0-1166d1d9f4b5).
1. Find the 'Server' field and replace its address with `https://microk8s-vm.mshome.net:12345`. The last 5 digits should be the port number we noted earlier when running `microk8s config`.
1. Save the file and close it.
1. Run `Start microk8s` in Windows Terminal again (and yet again if the start command fails).

We also need to run the following command in Windows Terminal to enable Ingress on your local Kubernetes instance: `microk8s enable ingress`.

Ingress is a network routing tool for MicroK8s. It lets you see your app on your local instance without complex or problematic workarounds, like port forwarding.

## Step 3: Install kubectl-cli on your computer

Before we install Octopus, we install and use a software manager called [Chocolatey](https://chocolatey.org/install) to install the kubectl-cli (command-line interface).

The kubectl-cli is what Octopus uses to interact with your local cluster and trigger processes for the deployment. It's different to the kubectl installed during the MicroK8s setup, which is only for your local K8s cluster.

To install Chocolatey and the kubectl-cli:

1. Open a PowerShell tab in Windows Terminal as an administrator.
1. Run the Chocolately install command: `Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))`.
1. The install takes a few minutes. Close and reopen Windows Terminal when finished.
1. Now we can install kubectl. Run `choco install kubernetes-cli` in Windows Terminal, and wait for the process to finish.

If you have any problems, see [Chocolatey's documentation](https://chocolatey.org/install) for more information. Not running PowerShell or Windows Terminal as an administrator tends to be the cause of most issues.

## Step 4: Install SQL Server Express

Octopus needs a SQL Server to store information about your projects and deployments.

Technically, you can install SQL Server Express during the Octopus Setup Wizard covered in the next step. However, we'll install it now for clarity as it saves switching between apps during steps.

As some point during the install, you may see a Windows Firewall warning. You should allow access otherwise you'll experience problems with connections.

1. [Download SQL Server Express](https://www.hanselman.com/blog/download-sql-server-express). I used the Basic SQL Server 2019 Express Edition. It's a free download and the basic version is fine.
1. When downloaded, run the installer from your Downloads folder (or wherever you chose to save it.)
1. Choose where you'd like to extract the files and click **OK**. It should default to your Downloads folder.
1. The 'SQL Server Installation Center' will open. Click **New SQL Server stand-alone installation or add features to an existing installation**.
1. Accept the 'MICROSOFT SOFTWARE LICENSE TERMS' and click **Next**.
1. You can use all the defaults for the rest of the setup wizard. Just click **Next** to progress, then **Close** when the install completes.

## Step 5: Install and set up Octopus on your computer

Now we can install Octopus Server.

1. Go to the [Octopus downloads page](https://octopus.com/downloads) and click the **Download** button for the latest version.
1. When downloaded, run the installer MSI from your Downloads folder (or wherever you chose to save it.)
1. Progress through each stage of the install process by clicking **Next** and click **Finish** at the end. Make sure you:
   - Accept the 'End-User License Agreement'
   - Install Octopus where you're comfortable. The default install location is `C:\Program Files` and is fine in most cases.

Octopus auto-starts after the installation and takes you to the Octopus Setup Wizard. Complete the screens as follows:

1. **License key**: Click the **free trial license key** link and enter your details. Copy the license text from the website code-box into the setup wizard's license-box. Click **Next**.
1. **Home**: Choose a home directory for Octopus to store settings and related files. The default is fine in most cases. Click **Next**.
1. **Service Account**: We're setting up a local pipeline, so select **Use Local System account**.
1. **Database**: Select **(local)\SQLEXPRESS** from the 'Server name' dropdown (unless you called the SQL Server instance something else) and click **Next**.
1. Click **OK** on the pop-ups that ask:
   - If you want to create the 'OctopusDeploy-OctopusServer' database
   - If you want to grant access to the 'NT AUTHORITY\SYSTEM' account
1. **Web Portal**: The default is fine. Click **Next**.
1. **Authentication**: Complete the following fields and click **Next**:
   - **Authentication mode**: Select **Username/passwords stored in Octopus**
   - **Username**: Enter a username to log into Octopus
   - **Email**: Enter your email address
   - **Password and Retype password**: Enter the password you'll use to login into Octopus Server
1. Click **Finish**.

Octopus Manager will now open. Click **Open in browser** to launch the Octopus dashboard and log in with the credentials you just created.

## Step 6: Create a project in Octopus

1. Go to your Octopus dashboard and click **Projects** from the top menu.
1. Click **Add New Project**.
1. Give your project a suitable name and click **SAVE**.
1. Remove all environments but 'Development' and click **SAVE**.

We'll be back to create the deployment process later.

## Step 7: Connect Octopus to your local Kubernetes instance

Run `microk8s kubectl create token default --duration 525600m` in your Windows Terminal to create a security token for your Kubernetes instance. This command sets the token to be valid for 10 years. By not stating a duration, the token will only last a few hours.

To add the token to Octopus:

1. Click **Infrastructure** from Octopus's top menu.
1. Click **Accounts** from the left menu.
1. Click **ADD ACCOUNT** and select **Token** from the dropdown.
1. Complete the following fields (leave everything else as default) and click **SAVE**:
   - **Name**: Give the account a short name. I opted for `Kubernetes Account`
   - **Token**: Paste in the token you just generated in Windows Terminal
   - **Environment**: Select **Development** from the dropdown 

Now we'll create the deployment target:

1. Go to your Octopus dashboard and click **Infrastructure** from the top menu.
1. Click **Deployment Targets**.
1. Click **ADD DEPLOYMENT TARGET**.
1. Click **KUBERNETES CLUSTER** and then **Kubernetes Cluster** from the filtered results below.
1. Complete the following fields (leave everything else as default) and click **SAVE**:
   - **Display Name**: Enter what you'll call the cluster in Octopus. I called it `Test Cluster`.
   - **Environments**: Select **Development** from the dropdown menu.
   - **Target Roles**: Type in a name for the Target Role and click **Add new role**. We use target roles to help direct where deployments go. I called mine `Local Cluster`.
   - **Authentication**: Select **Token** from the dropdown, then select the credentials we created earlier from the **Select account** dropdown.
   - **Kubernetes cluster URL**: Use the server address we set in the config file earlier: `https://microk8s-vm.mshome.net:12345` (remember to replace the last 5 digits with the port number we noted earlier)
   - **Skip TLS verification**: Tick this box.

When saved, click **Connectivity** from the left menu and click **CHECK HEALTH**. Octopus will check it can see your cluster. If you see a green tick, you're good to go. If you see a red cross, go back and check your settings.

## Step 8: Create your GitHub project's repository

Log in to GitHub and create a fork of the [Octopus Underwater App repository](https://github.com/OctopusSamples/octopus-underwater-app) by clicking the **Fork** button and completing the short form.

You should now see a copy of the repository in your list of repositories.

## Step 9: Create a Docker repository

1. Log into your [Docker Hub](https://hub.docker.com/) account.
1. Click **Create repository**.
1. Complete the following fields and click **Create**:
   - **Namespace**: Set as your account name.
   - **Repository Name**: Enter `octopus-underwater-app`.
   - **Visibility**: Leave as **Public**.

You should now see the Docker repository in your list.

## Step 10: Set up a package feed in Octopus

1. Go back to Octopus Server and click on **Library** from the top menu.
1. Click **External Feeds** from the left menu.
1. Click **ADD FEED**.
1. Complete the following settings and click **SAVE**:
   - **Feed Type**: Select **Docker Container Registry** from the dropdown.
   - **Name**: Give the feed a name to use in Octopus. I called mine `Docker`.
   - **URL**: Leave as `https://index.docker.io`.
   - **Credentials**: Select **Username and Password** and enter your Docker username and password into the 'Feed Username' and 'Feed Password' fields.
1. Click the **TEST** button and search for your Docker repository name to see if it appears in the results. If it's there, it's working.

## Step 11: Set up a GitHub Action to push the project to Docker Hub

First, we'll add your DockerHub credentials as secrets to your GitHub repository.

1. Click **Settings** from your repository's top menu.
1. Expand **Secrets and variables** in the left menu and click **Actions**.
1. Under the **Repository secrets** heading click **New repository secret**.
1. Complete the following fields and click **Add secret**:
   - **Name**: Enter `DOCKERHUB_USERNAME`.
   - **Secret**: Enter your Docker username.
1. Click **Add secret** again.
1. Complete the following fields and click **Add secret**:
   - **Name**: Enter `DOCKERHUB_TOKEN`.
   - **Secret**: Enter your Docker Hub password.

Now we can create the action. For this, we use the Build and push [Docker images GitHub Action](https://github.com/marketplace/actions/build-and-push-docker-images). Don't worry, we made it simpler by providing the action's workflow in the steps below.

1. Click **Actions** from your repository's top menu.
1. Click **New workflow**.
1. Click **Set up a workflow yourself**.
1. Call the file `build-push-action.yml` and make sure it lives in a subdirectory called `workflows`. If you don't already have a 'workflows' folder in your repository, you can create it by adding `workflows/` when creating the workflow file.
1. Copy and paste the following code into the code editing box.
1. In the final line labelled 'tags', replace 'dockerusername' and 'dockerrepositoryname' with your own Docker username and repository name. For example, mine reads `andyoctopus/octopus-underwater-app:latest`.
   
```
   name: build-push-action

on:
  push:
    branches:
      - 'main'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      -
        name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      -
        name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: dockerusername/dockerrepositoryname:latest
```
Next, click **Commit changes...**.

The Action should automatically run. You can track its progress by clicking the **Actions** button again.

If the action completes with a green tick, you can check your Docker repository to see if it has an image.

## Step 12: Create, run, and test your first deployment

Now we can go back and create the deployment process in Octopus. Remember, if we don't mention something, leave it at the default.

1. In Octopus, click **Projects** from the top menu, then click on your recently created project.
1. Click **CREATE PROCESS**.
1. Click **Kubernetes** and then **Deploy Kubernetes Containers** from the filtered options below.
1. Complete the following sections and fields:
   - **On Behalf Of**: Select the target role we created earlier from the dropdown. Mine was `Local Cluster`.
   - **Deployment**:
      - **Deployment Name**: Enter `octopus-underwater-app`.
   - **Container**:
      - Click **ADD CONTAINER**, complete the following fields in the pop-up and click **OK**:
         - **Name**: Enter `octopus-underwater-app`.
         - **Package Feed**: Select name of the feed we set earlier. I called mine `Docker`.
         - **Package ID**: Start typing your Docker username and select **octopus-underwater-app** from the list.
      - Click the **ADD PORT** button and complete the fields as follows:
         - **Name**: Enter `http`.
         - **Port**: Enter `80`.
         - **Protocol**: Select **TCP** from the dropdown.
   - **Service**:
      - **Service Name**: Enter `octopus-underwater-service`.
      - **Service Ports**: Click **ADD PORT**, complete the following fields and click **OK**:
         - **Name**: Enter `http`.
         - **Port**: Enter `80`.
         - **Protocol**: TCP.
   - **Ingress**:
	   - **Ingress Name**: Enter `octopus-underwater-ingress`.
	   - **Ingress Host Roles**: Click **ADD HOST RULE**, then **ADD PATH**. Complete the following fields and click **OK**:
	      - **Path**: Enter `/`.
	      - **Service port**: Select **HTTP** from the dropdown.
	      - **Path Type**: Select **Prefix** from the dropdown.
1. Leave everything else as default and click **SAVE**.

Now we create a release and try to deploy it:

1. Click **CREATE RELEASE** from the left menu. Octopus will check it can see everything related to your package.
1. Click **SAVE**.
1. Click **DEPLOY TO DEVELOPMENT...** and wait for the deployment to finish.

When the deployment completes successfully, you can test that the application's working by going to the URL we set in the DNS earlier: `http://microk8s-vm.mshome.net`. An animated scene should greet you, with some reading suggestions.

## Conclusion

Congratulations, you just deployed an application to your own newly-created local Kubernetes instance!

Now you have a mostly-local Kubernetes deployment pipeline, you can experiment with it consequence free. You could try:

- Adding or swapping tooling in the pipeline
- Deploying your own project or changing the one we provided
- Creating new clusters and adding QA and production environments to see how lifecycles work in Octopus

Remember, this guide represents Kubernetes as its simplest layer and there's much more to learn from here. Kubernetes is an excellent solution, but it's a complicated one. Its complexity only snowballs the more you scale - something large organizations, enterprises, and those with modern software architectures often discover.

Octopus is a deployment automation tool that helps solve the complexity of Kubernetes deployments at scale. [Read how Octopus helps with Kubernetes deployments](https://octopus.com/use-case/kubernetes).

Happy deployments!