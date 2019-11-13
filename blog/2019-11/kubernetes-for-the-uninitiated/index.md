---
title: Beyond Hello World: Kubernetes for the uninitiated
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

By now, you've most undoubtedly have heard the term Kubernetes, but just what is it other than the latest buzzword?  In this post, I will cover what Kubernetes (or K8s for short) is and demonstrate how to run a real-world web application, [OctoPetShop](https://github.com/OctopusSamples/OctoPetShop), on it!

Note: There are 8 letters between the K and the s in Kubernetes, hence K8s.

This post is a continuation of my [last article](link to it) where I covered how to create Docker images using OctoPetShop as an example.

## What is Kubernetes?
Kubernetes is a container orchistration technology.  Machines that are running Kubernetes are referred to as `nodes`.  Nodes are what make up a Kubernetes `cluster`, though it is possible to have a cluster with a single node.  Nodes run containers in what is called a `pod`.

![](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

## Docker Desktop and Kubernetes
My previous post made mention of using Docker Desktop for local development of Docker containers.  Docker Desktop also contains an implementation of Kubernetes which makes local development and testing a breeze!

## Creating the OctoPetShop Kubernetes components
In order to run our OctoPetShop container images on Kubernetes, we need to create the YAML files to define the different resources required for our application.  Resource types are called `kind`, the kind for containers are called a `deployment`.  As you recall from my last article, the OctoPetShop application has three main components:
- Web front-end
- Product Service
- Shopping Cart Service

In addition to those components, OctoPetShop also needs a database server and database to function.  For OctoPetShop, we will need to create the following deployments:
- Web front-end
- Product Service
- Shopping Cart Service
- Microsoft SQL Server

### Deployments

Below is the deployment YAML file for the front-end.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: octopetshop-web-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: web
  template:
    metadata:
      labels:
        component: web
    spec:
      containers:
        - name: web
          image: octopussamples/octopetshop-web
          ports:
            - containerPort: 5000
              name: http-port
            - containerPort: 5001
              name: https-port
          env:
            - name: ProductServiceBaseUrl
              value: http://octopetshop-productservice-cluster-ip-service:5011/
            - name: ShoppingCartServiceBaseUrl
              value: http://octopetshop-shoppingcart-cluster-ip-service:5012
```

### Anatomy of the YAML
#### apiVersion
Kubernetes provides an [API reference](https://kubernetes.io/docs/reference/#api-reference) to help determine which version to choose.  For kind deployment, apps/v1 is what you use.
#### kind
This tells Kubernetes what type of resource we are deploying
#### metadata - name
This is the unique name of our deployment
#### spec - replicas
The number of container instances to run
#### spec - selector - matchlabels
Selector - The selector field defines how Kubernetes objects find which pods to manage
Labels - Labels are key/value pairs that are attached to objects used to specify identifying attributes
#### template
The pods template specification
#### template - spec - containers
This section is an array of containers that this deployment will run.
Each container gets:

- Name: the name we give the container
- Image: the Docker Hub image to use for the container
- Ports: an array of the ports that the container will expose to the pod network
- Env: an array of environment variables for the container

(See [OctoPetShop](https://github.com/OctopusSamples/OctoPetShop/tree/master/k8s) for the rest of the Deployment YAML files.)

Deployments will create the pods for our application, but we still need a way for them to communicate.

![](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

Each pod on a node has an internal IP address assigned by the node.  Within our containers we've specified port to expose to the pod, but those are still only within the pod and not exposed to the node.  To allow connectivity between pods on a node, we need to create a `service` for each pod.

### Services
There are a number of different services availabe via the API, we're going to focus on the specific services to allow our OctoPetShop applicaiton to function.  Our web front-end pod needs to the ability to talk to the Product Service and Shopping Cart Service pods.  The Product Service and Shopping Cart Service pods need to be able to communicate to the SQL Server Pod.  To make this possible, we need to create a `ClusterIP` service for the Product Service, Shopping Cart Service and the SQL Server pods.  Below is the YAML to create the ClustIP service for the Product Service

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

In the above YAML, we've created a service that maps a pod port to a container port to allow pod-to-container communication within the node.  It is important to note that this service does not expose the port at the node level, so external access is not possible.  One important thing to note is the metadata name.  The name will create a DNS entry so the service can be referenced by DNS name.  In our previous web front-end YAML, we declared an Environment Variable for the Product Service URL which contained octopetshop-productservice-cluster-ip-service as the DNS entry.  The ClusterIP service for the Product Service is where that came from.

### Allowing external access
To allow external access to the node, we'll need to define either an `Ingress` or a `LoadBalancer` service.  In our case, we've chosen a LoadBalancer service to allow access to the web front-end.

```
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
spec:
  selector:
    component: web
  ports:
    - port: 80
      targetPort: 5000
      name: http-port
    - port: 5001
      targetPort: 5001
      name: https-port
  type: LoadBalancer
  externalIP: <IPAddress>
```
NOTE: when using a hosting provider such as Azure, the externalIP line can be ommitted as the hosting provider will automatically provision a public IP resource and connect it.

### Job
Included in our solution for OctoPetShop is a Dbup project which contains the scripts that will both create and seed our database.  DbUp is a console application that executes and then stops, so we don't want to use a Deployment kind as Kubernetes will attempt to keep it running.  For this, we want to use a kind of `Job` which is specifically meant to terminate once they've completed.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: octopetshop-dbup
spec:
  template:
    spec:
      containers:
        - name: dbup
          image: octopussamples/octopetshop-database
          env:
            - name: DbUpConnectionString
              value: Data Source=octopetshop-sqlserver-cluster-ip-service;Initial Catalog=OctoPetShop; User ID=sa; Password=SomePassword
      restartPolicy: Never
```



## Kubectl
Kubectl is the command-line program used for Kubernetes.

### Starting OctoPetShop in Kubernetes
To run a Kubernetes YAML file, you run the command `kubectl apply -f <YAMLFile>`.  However, it is possible to specify a folder instead of an individual file.  This is the main reason that the OctoPetShop repo has all of the Kubernetes YAML files stored in the k8s folder.  If you've cloned the repo, you can run `kubectl apply -f k8s` to get the entire cluster running with one command!

```
PS C:\GitHub\OctoPetShop> kubectl apply -f k8s
job.batch/octopetshop-dbup created
service/web-loadbalancer created
service/octopetshop-productservice-cluster-ip-service created
deployment.apps/octopetshop-productservice-deployment created
service/octopetshop-shoppingcart-cluster-ip-service created
deployment.apps/octopetshop-shoppingcartservice-deployment created
deployment.apps/sqlserver-deployment created
service/octopetshop-sqlserver-cluster-ip-service created
service/octopetshop-web-cluster-ip-service created
deployment.apps/octopetshop-web-deployment created
```
### Checking status of pods
The apply command will show us that it ran the YAML files, but not much else.  To check the status of our pods, we run the command `kubectl get pods`

```
PS C:\GitHub\blog> kubectl get pods
NAME                                                         READY   STATUS      RESTARTS   AGE
octopetshop-dbup-9kt54                                       0/1     Completed   0          2m55s
octopetshop-productservice-deployment-6f955ff576-4nwtw       1/1     Running     0          2m55s
octopetshop-shoppingcartservice-deployment-9f94574f9-t96ck   1/1     Running     0          2m55s
octopetshop-web-deployment-7b6d499d69-f9jsp                  1/1     Running     0          2m55s
sqlserver-deployment-784d755db-8vbwk                         1/1     Running     0          2m55s
```

This command shows us how many pods are running and how many pods should be running.  In our case, we specified our replicas as 1, so there should only be 1 instance in our pods.  You'll note that the octopetshop-dbup pod has 0 of 1 pods ready.  Since we defined the octopershop-dbup with a kind of job, this is normal since it is supposed to terminate once it has run.

### Displaying logs
Unlike docker compose, running the kubectl apply command didn't show any outpout from the pods or containers.  When a pod fails, it's useful to know why.  Let's change the password for our octopetshop-dbup job so that it fails

```
apiVersion: batch/v1
kind: Job
metadata:
  name: octopetshop-dbup
spec:
  template:
    spec:
      containers:
        - name: dbup
          image: octopussamples/octopetshop-database
          command: [ "dotnet", "run", "--no-launch-profile" ]
          env:
            - name: DbUpConnectionString
              value: Data Source=octopetshop-sqlserver-cluster-ip-service;Initial Catalog=OctoPetShop; User ID=sa; Password=ThisPasswordIsWrong
      restartPolicy: Never
```
If we run our `kubectl apply -f k8s` command, we'll see something like the following

```
PS C:\GitHub\OctoPetShop> kubectl get pods
NAME                                                         READY   STATUS    RESTARTS   AGE
octopetshop-dbup-76cxk                                       1/1     Running   0          5s
octopetshop-dbup-bjsj8                                       0/1     Error     0          35s
octopetshop-dbup-mt9lk                                       0/1     Error     0          25s
octopetshop-dbup-rm97t                                       0/1     Error     0          49s
octopetshop-productservice-deployment-6f955ff576-bvc88       1/1     Running   0          49s
octopetshop-shoppingcartservice-deployment-9f94574f9-mkr7h   1/1     Running   0          49s
octopetshop-web-deployment-7b6d499d69-975kg                  1/1     Running   0          49s
sqlserver-deployment-784d755db-7dh95                         1/1     Running   0          49s
```

Kubernetes attempted to run the job octopetshop-dbup three times, but they all failed.  After three retry attemps, Kubernetes will stop attempting to run teh pod.  To troubleshoot the issue, we can run `kubectl logs jobs/octopetshop-dbup` to display the output from the container

```
PS C:\GitHub\OctoPetShop> kubectl logs job/octopetshop-dbup
Found 3 pods, using pod/octopetshop-dbup-rm97t
Master ConnectionString => Data Source=octopetshop-sqlserver-cluster-ip-service;Initial Catalog=master;User ID=sa;Password=*********************

Unhandled Exception: System.Data.SqlClient.SqlException: A connection was successfully established with the server, but then an error occurred during the pre-login handshake. (provider: TCP Provider, error: 0 - Success)
   at System.Data.SqlClient.SqlInternalConnectionTds..ctor(DbConnectionPoolIdentity identity, SqlConnectionString connectionOptions, Object providerInfo, Boolean redirectedUserInstance, SqlConnectionString userConnectionOptions, SessionData reconnectSessionData, Boolean applyTransientFaultHandling)
   at System.Data.SqlClient.SqlConnectionFactory.CreateConnection(DbConnectionOptions options, DbConnectionPoolKey poolKey, Object poolGroupProviderInfo, DbConnectionPool pool, DbConnection owningConnection, DbConnectionOptions userOptions)
   at System.Data.ProviderBase.DbConnectionFactory.CreatePooledConnection(DbConnectionPool pool, DbConnection owningObject, DbConnectionOptions options, DbConnectionPoolKey poolKey, DbConnectionOptio   at System.Data.ProviderBase.DbConnectionPool.CreateObject(DbConnection owningObject, DbConnectionOptions userOptions, DbConnectionInternal oldConnection)
   at System.Data.ProviderBase.DbConnectionPool.UserCreateRequest(DbConnection owningObject, DbConnectionOptions userOptions, DbConnectionInternal oldConnection)
   at System.Data.ProviderBase.DbConnectionPool.TryGetConnection(DbConnection owningObject, UInt32 waitForMultipleObjectsTimeout, Boolean allowCreate, Boolean onlyOneCheckConnection, DbConnectionOptions userOptions, DbConnectionInternal& connection)
   at System.Data.ProviderBase.DbConnectionPool.TryGetConnection(DbConnection owningObject, TaskCompletionSource`1 retry, DbConnectionOptions userOptions, DbConnectionInternal& connection)
   at System.Data.ProviderBase.DbConnectionFactory.TryGetConnection(DbConnection owningConnection, TaskCompletionSource`1 retry, DbConnectionOptions userOptions, DbConnectionInternal oldConnection, DbConnectionInternal& connection)
   at System.Data.ProviderBase.DbConnectionInternal.TryOpenConnectionInternal(DbConnection outerConnection, DbConnectionFactory connectionFactory, TaskCompletionSource`1 retry, DbConnectionOptions userOptions)
   at System.Data.SqlClient.SqlConnection.TryOpen(TaskCompletionSource`1 retry)
   at System.Data.SqlClient.SqlConnection.Open()
   at SqlServerExtensions.SqlDatabase(SupportedDatabasesForEnsureDatabase supported, String connectionString, IUpgradeLog logger, Int32 timeout, AzureDatabaseEdition azureDatabaseEdition, String collation)
   at OctopusSamples.OctoPetShopDatabase.Program.Main(String[] args) in /src/Program.cs:line 16
```

## OctoPetShop running in Kubernetes
Once all of the YAML has been run successfully, the OctoPetShop application should be running!  Navigate to http://localhost:5000

:::warning
OctoPetShop is written to redirect to https, you will most likley receive a warning about the site not being secure.  This is normal and it is safe to proceed since we did not configure a certificate as part of our process
:::

![](octopetshop.png)

## Including Kubernetes in a CI/CD pipeline
Kubernetes doesn't have anything that needs to be built, other than the Docker images it uses.  For this, we'd focus on the Continuous Delivery (CD) portion of a CI/CD pipeline.  Release management software such as Azure DevOps Pipelines and Octopus Deploy contain steps that can be used to automate the deployment of Kubernetes clusters.

## Conclusion
Before embarking on the journey to containerize OctoPetShop with the eventual goal of getting it running on Kubernetes, I viewed Kubernetes as a complex monster.  Now that I've gone through the motion of containerizing an appliciation and getting it to run, I've realized that Kubernetes is indeed a complex monster, but getting an application to run on it isn't really that hard.

The [OctoPetShop](https://github.com/OctopusSamples/OctoPetShop) GitHub repo contains all of the docker, docker-compose, and Kubernetes YAML files for the OctoPetshop application.