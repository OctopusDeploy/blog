---
title: Kubernetes for the uninitiated
description: Describing a high level overview of what Kubernetes is
author: shawn.sesna@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2020-10-23
tags:
 - Kubernetes
---

![](dilbert-kubernetes.jpg)

By now, you've most undoubtedly have heard the term Kubernetes, but just what is it other than the latest buzzword?  In this post, I will describe what Kubernetes is and why it is appealing.

## Containers
To understand what Kubernetes is, you first need to know what containers are.  Similar to VMs, containers have thier own RAM, CPU, and filesystem.  However, containers rely on the host operating system (OS) for a lot of their base functionality, which makes them lightweight and portable.  Where a VM would require its own OS and all of the specialized components for an application to be installed, a container bundles the required components for the application to function into what is referred to as an `image`.  These images are completely self-contained and immutable, meaning they cannot be modified during their life.  If an update needs to be made to a container any running instances must first be destroyed before being replaced by the new version.  As you can imagine, this is somewhat problematic if the container needs to retain any data (such as a database).  There are ways to persist data when a container is destroyed and replaced by using persistent volumes and persistent volume claims, however, this post is meant to be higher level.

### Docker
The most popular container technology is called Docker.  Docker is an engine installed on either Windows or Linux that uses OS-level virtualization to run containers (at the time of this writing, containers are built for either Windows or Linux and are not cross-platform).  A traditional hypervisor/virtual machine (VM) architecture looks like this,

![](https://www.docker.com/sites/default/files/d8/2018-11/container-vm-whatcontainer_2.png)

In the above diagram, the hypervisor and each VM have their own OS, working somewhat independently (other than the VM requires the hypervisor to function.)  The applications are then deployed to the VM and served up using a series of virtual hardware (networking, RAM, CPU, etc...).

With Docker, the hypervisor is eliminated and the containers run directly off the host OS through the Docker engine.

![](https://www.docker.com/sites/default/files/d8/2018-11/docker-containerized-appliction-blue-border_2.png)

### How do you create an image for a container?
A Docker file contains the instructions necessary for Docker to build a container image.  In almost all cases, images are built from a base image that contains the bare minimum components to run your container such as the .NET Core SDK.  Let's use [OctoPetShop](https://github.com/OctopusSamples/OctoPetShop) as an example.  OctoPetShop is a sample application written in .NET Core which contains three main components; a web front-end, a product service, and a shopping cart service.  It also has a need for a database, we'll cover that later in this post.

To build the OctoPetShop front-end as a container, we'd define a dockerfile like this

```
FROM mcr.microsoft.com/dotnet/core/sdk:2.1

RUN mkdir /src
WORKDIR /src
ADD . /src
RUN dotnet restore
RUN ["dotnet", "build", "--configuration", "release"]

EXPOSE 5000
EXPOSE 5001

ENV ASPNETCORE_URLS="http://+:5000;https://+:5001"
ENV ASPNETCORE_ENVIRONMENT="Production"

ENTRYPOINT [ "dotnet", "run", "--no-launch-profile" ]
```

Our OctoPetShop front-end image starts from the base image of `mcr.microsoft.com/dotnet/core/sdk:2.1` which contains the .NET Core SDK.  We then create a new folder within the container image, copy the source code, run a build, expose some networking ports, then define the starting command of the container so our .NET Core app will start when the container is run.

## Kubernetes
Now that we have a basic understanding of what containers are, let's talk about how they fit with Kubernetes.  One of the most difficult parts about using something like Docker is coordination of multiple containers.  Docker Compose was written to help with this, but isn't very scalable.  Kubernetes is a container orchistration technology that solves the issues of:

- scalability
- resiliancy
- container networking 

### Scalability
Configuraging resources in Kubernetes is done using YAML files, which stands for YALM Ain't Markup Language (YAML).  These files define what type of resources you are creating called `kind`.  The resource kind for a container is called a `deployment`.  The `replicas` property of a deployment is what you use to specify how many instances of a container to run.  If you need to scale up, simply change the replicas number in the YAML file and re-apply.  Kubernetes will see that there now needs to be more instances of the container running and create new instances.  Scaling down is the same process, Kubernetes will see the change and destroy instances until it reaches the desired number.  The instances of a container are grouped together in what Kubernetes calls `pods`.  
![](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

Machines that are running Kubernetes are called `nodes`.  With Kubernetes, it is possible to connect nodes together to create a cluster.  When running in a cluster, Kubernetes can spread the pods amongst the nodes to create even more scalability.

Node
![](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

Cluster

![](https://d33wubrfki0l68.cloudfront.net/152c845f25df8e69dd24dd7b0836a289747e258a/4a1d2/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)

### Resiliancy 
Kubernetes keeps a close watch on the number of replicas defined.  Should a container crash, Kubernetes will simply spin up another container until it reaches the desired number.  Since the pods are distributed amongst the cluster, if a node crashes, the application will still run since the the other nodes still have pods running.

### Container networking
The EXPOSE commands within our Docker file opens networking ports to the container.  When a container is running within a pod, the designated port is open on all running instances.  To be able to communicate with the containers within a pod from other pods, a resource kind of `service` with a type of `ClusterIP` needs to be created.  The ClusterIP service maps the pod port to the exposed container port.  Using OctoPetShop as an example the front-end web application would use the ClusterIP service of the product service to retrieve a list of products available.

```
apiVersion: v1
kind: Service
metadata:
  name: octopetshop-productservice-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: productservice
  ports:
    - port: 5011
      targetPort: 5011
      name: http-port
    - port: 5014
      targetPort: 5014
      name: https-port
```
For ClusterIP resources, the name under the metadata section becomes the DNS name.  Other pods are able to reference this pod by the DNS name.  The Product service of OctoPetShop requires a database to pull its information from.  Included in the example of OctoPetShop, is a deployment of a Microsoft SQL Server container,

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sqlserver-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: sqlserver
  template:
    metadata:
      labels:
        component: sqlserver
    spec:
      containers:
        - name: sqlserver
          image: microsoft/mssql-server-linux:2017-latest
          ports:
            - containerPort: 1433
          env:
            - name: SA_PASSWORD
              value: "Password"
            - name: ACCEPT_EULA
              value: "Y"
```
:::warning
In this instance, the sa password is in clear text.  In a production scenario, this would typically be handled by creating a `secret` in Kubernetes that pods would be able to reference:::

Here, we've opened port 1433 on the container, but need this port to be available to other pods which requires a ClusterIP service to be defined

```
apiVersion: v1
kind: Service
metadata:
  name: octopetshop-sqlserver-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: sqlserver
  ports:
    - port: 1433
      targetPort: 1433
```

With the ClusterIP defined, the Product service can use the DNS name of octopetshop-sqlserver-cluster-ip-service in the connection string.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: octopetshop-productservice-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: productservice
  template:
    metadata:
      labels:
        component: productservice
    spec:
      containers:
        - name: productservice
          image: octopussamples/octopetshop-productservice
          env:
            - name: OPSConnectionString
              value: Data Source=octopetshop-sqlserver-cluster-ip-service;Initial Catalog=OctoPetShop; User ID=sa; Password=Password
````
In the above deployment, we've defined an `environment` variable called `OPSConnectionString` which is passed to the service when it starts to be used as the connection string.  As you can see, we've used the DNS of the SQL Server ClusterIP service as the server to connect to.