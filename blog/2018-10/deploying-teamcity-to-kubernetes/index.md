---
title: Deploying TeamCity to Kubernetes using Octopus Deploy
description: Learn how to spin up a build server on demand using Octopus Deploy and Kubernetes
author: bob.walker@octopus.com
visibility: public
published: 2018-09-39
tags:
 - Kubernetes
---

Hi!  My name is Bob Walker and I am a solution architect here at Octopus Deploy.  My primary focus is to make sure our customers are successful in using Octopus Deploy.  That simple sentence covers a wide range of duties, from reviewing configuration to creating custom implementation using the API to providing demos showing off all the new functionality we've been adding.  At Octopus Deploy we don't have a preference for what our build server our customers use.  That flexibility is great for our customers because they can continue using their build tool of choice.  A side effect of that is I get to learn how to use every build server by creating a build for a demo.  

A the time of this writing, we have two additional solution architects focused on customer's success, Ryan Rousseau and Derek Campbell.  Whenever possible we like to share the demos we have set up.  Our background is either in developerment or infrastructure.  That background has resulted in us really disliking it anytime we have to reinvent the wheel.  The easiest way we found to share demo resources is to leverage SaaS whenever possible.  We use Octopus Cloud for our Demo Octopus instance.  In fact we were one of the alpha users of Octopus Cloud and are one of the bleeding edge users.  We help dogfood Octopus Cloud everyday.  

Our first two build servers, VSTS (now Azure DevOps) and AppVeyor, were rather easy to setup.  They are already SaaS.  But a large chunk of our users are using TeamCity.  All three of us have a local TeamCity instance.  It is time to move that to the cloud.

Side Note: at times in this article I am going to swap Kubernetes and K8s.  They mean the same thing.  

## Requirements

We want our build server to support a number of different technologies.  This list includes, but not limited to:

1. .NET Framework 4.7.x applications (ASP.NET, Windows Services, etc)
2. .NET Core applications
3. Windows Container applications
4. Linux Container applications (for my .NET core apps!)
5. Java
6. Web Frameworks (Angular, React, etc)

## Platform

I can go the easy way out and run TeamCity on a Linux VM with Windows build agent on another VM.  JetBrains provides AWS and Azure templates to make setup as easy as possible.  But I don't want to have to maintain VMs.  And VMs can get expensive.

JetBrains provides a [server docker container](https://hub.docker.com/r/jetbrains/teamcity-server/) and a [agent docker container](https://hub.docker.com/r/jetbrains/teamcity-server/).  If you look closely at the agent docker container you can see it can either be run as a Linux container or a Windows container.  They also include a number of build tools, such as .NET and Git.  And, Octopus Deploy recently added Kubernetes support.  The challenge today is to get TeamCity running in a Kubernetes cluster.  

For fun, I am going to be using Octopus Deploy to deploy TeamCity to Kubernetes which will in turn push packages back to the same Octopus Deploy instance.  I have some real snake eating the tail, but why not?  YOLO, right?

## Step 1: Create K8s cluster and connect Octopus Deploy to it

First we need to setup Octopus Deploy to be able to deploy to the K8s cluster.  In order for that to happen I need to create a K8s cluster first.  For the purposes of this article I will be using Google Cloud Platform, or GCP, to host my K8s cluster.  

Lets go to [Google Console Login](https://console.cloud.google.com).

Once you create your account and login you will be sent to the dashboard.  On the left menu select Kubernetes.

![](gcp_dashboard_select_k8s.png)

Click on the create cluster button.

![](gcp_create_k8s_cluster.png)

You will be presented with a wizard.  You can leave the defaults as is.  All I did was enter in a name and click on create.  I left the size (1 vCPU and 3.75 GB of memory) and the location (us-central1-a) alone.  Fun fact, that data center is only 15ish miles from my house.  If there is any latency I have a few questions and follow up questions.

![](gcp_create_k8s_cluster_wizard.png)

It takes anywhere from 5 to 20 minutes for the cluster to be created.  While we are waiting, let's get Octopus Deploy ready to connect to it.  I will be using my team's hosted Octopus instance for this.  Because I am using the hosted instance, I know I am on the latest version, which at the time of this writing is 2018.8.6.  You need to be using at least 2018.8.0 for this to work.

Currently, K8s is disabled by a feature flag.  First things first, go to configuration -> features and enable it.  

![](enable_k8s_octopus_deploy.png)

Next up, it is time to create a worker.  The worker will have Kubectl installed on it.  I want these machines partitioned off from the rest of my targets because they will have admin access to my cluster.  To do this I first created a worker pool called "Kubernetes Worker Pool."

![](octopus_k8s_worker_pool.png)

The worker is nothing fancy, just a listening tentacle assigned to a specific worker pool.  

![](octopus_deploy_k8s_worker.png)

Don't forget to add KubeCtl on the machine.  I'm using Windows, so I'll let chocolatey handle the heavy lifting.

```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

choco install kubernetes-cli -y
```

Alright, we've killed enough time.  Let's check on GCP and see where my cluster is at.  Nice, it is finished.  Click on the pencil icon.

![](gcp_cluster_dashboard.png)

This next page shows us the overview of the cluster.  Make note of the IP Address.  That is how Octopus will communicate with this cluster.  And before you wonder, "hey isn't showing your IP address and other sensitive information dangerous?"  Yes, it is.  I deleted everthing in GCP before this went live.  

![](gcp_sample_k8s_overview.png)

If you click the "show credentials" link, that will show the user name and password.  

![](gcp_username_pw.png)

We are going to save those credentials in Octopus.  Go to Infrastructure -> Accounts.  On the top right click on add account and select username/password.

![](octopus_add_usernamepassword.png)

All you need to do here is enter in the username and password from GCP.

![](octopus_deploy_create_account.png)

Now let's add the certificate from GCP so we can use TLS verifications.  Save the certificate to your desktop as a .pem file.  Then go to Library -> Certificates and click the add certificate.

![](octopus_add_certificate.png)

Fill out the form and select the certificate file you just created on your desktop.

![](octopus_deploy_create_cert.png)

If successful you will see a screen similar to this.  You will notice the certificate is self-signed and set to expire in 5 years.  If I wanted to I could setup a subscription to notify me when this expires.  But not now, that is future Bob's problem.

![](octopus_deploy_cert_success.png)

Now it is time to add the Kubernetes deployment target.  Go to add deployment target

![](octopus_deploy_add_k8s_target.png)

First, create a name, assign it to environments and a role.

![](octopus_deploy_add_k8s_start.png)

Next select the username/password account, enter in the IP Address, and select the certificate.  

**Please note:** it is important to use https://[Ip Address] for the URL.  If you don't use that, Octopus will be unable to connect to your K8s cluster and you will be spending a lot of time wondering why.  I know this because I forgot and I kept scratching my head wondering why it wasn't working.

![](octopus_deploy_add_k8s_options.png)

I like to perform a health check right away to make sure I didn't screw anything up.

![](octopus_deploy_success_connect_k8s.png)

Finally we need add Docker Hub as a feed.  This is where the TeamCity container will be pulled from.  Go to library -> external feeds and click the "Add Feed" button on the top right.  

![](octopus_deploy_add_docker_feed.png)

## Step 2: Deploy Team City Server

I want to start simple and go complex.  It is possible to configure TeamCity to use external storage and an external database.  But it is also possible to configure it to use local storage and a local database.  I know if I configure it to use the local resources that they will be destroyed each time I do a deployment.  For right now, that is fine.  I just want to get it running.  I won't set up any users or projects.  

In Octopus Deploy, create a new project.  I created a new environment called "SpinUp" and lifecycle called "SpinUp Only."  Please feel free to configure your environment and lifecycle however you want.  It is just what I did.

![](octopus_deploy_create_project.png)

The process will consist of a single step, Deploy Kubernetes Containers.  Please note, this step has a lot of options.  For the sake of brevity I will only include screenshots of items I changed.  If you don't see something in the screenshot, just assume I left them alone.

![](octopus_deploy_select_k8s_step.png)

First, enter in the basic information for this step.  

![](octopus_deploy_teamcity_server_basic.png)

Next, click on the "configure resources" and disable "secret" and "configure map."  This is to keep the step a little easier to walk through.

![](octopus_deploy_k8s_features.png)

Enter in a deployment name.  The name can only contain lowercase alphanumeric characters plus "-".

![](octopus_deploy_k8s_deploymentname.png)

For the deployment strategy I am leaving it at recreate deployment.

![](octopus_deploy_deployment_strategy.png)

Now it is time to specify the container.  Click on Add Container.

![](octopus_deploy_add_container.png)

The full container name is jetbrains/teamcity-server.  Jetbrains is the username who created the container and teamcity-server is the name of the container.  Enter in all that information into the start of the add container screen.

![](octopus_deploy_container_details.png)

Now it is time to specify the port number.  The default port number exposed by TeamCity is 8111.

![](teamcity_container_port.png)

That is all we are going to specify for now.  Go ahead and click on the ok button.  

After the modal window closes, in the features selection enter in the name of the service and select the option of Load Balancer.

![](octopus_deploy_server_features.png)

We need a way to access the server.  Click the add port button to get the service port modal window to open.

![](octopus_deploy_adding_external_port.png)

For this we will be using port 80.  Give it a name, use port 80 and have it point back to the named port defined when we selected the container.

![](octopus_deploy_service_port.png)

When it is all said and done your summary screen should look similar to this:

![](octopus_deploy_team_city_server_summary_k8s.png)

That is it!  Now it is time to save and we can create our first deployment!

![](octopus_deploy_create_release.png)

The deployment itself does not take very long.  

![](octopus_deploy_team_city_server_results.png)

But go back to GCP's interface and click on the services link on the left.  You can see GCP will take a minute to create the load balancer.

![](gcp_creating_loadbalancer.png)

After the load balancer is complete let's click on the URL.  If all goes well we should be presented with this screen!

![](team_city_k8s_port_started.png)

## Step 3: Persistent Volume

Alright, things are coming along.  However, we have a slight problem.  Each time Octopus Deploy does a deployment it will destroy the node and recreate it.  That means the data directory in the above screenshot will be cleared out.  The end result is we have to recreate the TeamCity configuration each time.  That is...just awful.  What kind of build server is that?  

**Please Note:** This is not directly caused by Octopus Deploy, it is a feature of Kubernetes.  By default it assumes pods can be destroyed and recreated whenever needed.

What we need is to persist that data between deployments.  Thankfully, Kubernetes already has that functionality.  We just need to tap into it.  

To keep things simple I will be using a simple persistant volume.  This uses the storage which is backing the Kubernetes cluster.  It is possible to make use of other storage options.  There are [many options](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).  Pick the best one which works for your company / needs.  

To do this we will need to learn a little bit about the K8s CLI which is called kubectl.  The command we are interested in is the [apply command](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply).  This command will be create a resource if it doesn't already exist.  Which is perfect, that means we can use it in our deployment process.

Now a quirk with the apply command is you have to supply it a YAML or JSON file.  The file can be a URL or right on the hard drive.  What I did was create a GitHub repo to store these types of files.  Then I could reference the file from Github.

The file.  It is creating a 30 GB hard drive for me to use as my data drive.  
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dt-pm-claim
  labels:
    app: data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi
```

Octopus Deploy can run kubectl commands.  Let's go ahead and find that step to add to our process.

![](ocotpus_deploy_kube_ctl.png)

The script I need to run is:

```
kubectl apply -f [URL of file]
```

Let's go ahead and add that to the step.  Don't forget to set the worker and the role!

![](octopus_deploy_kube_ctl_command.png)

After saving my process looks like this.  

![](octopus_deploy_post_script_process.png)

That doesn't look right, we want to create the volume for the server.  Let's reorder that.

![](octopus_deploy_persistant_process_ordered.png)

