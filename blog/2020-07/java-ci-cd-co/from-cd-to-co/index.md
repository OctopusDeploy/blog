---
title: From Continuous Delivery to Continuous Operations
description: In this post we create example runbooks to implement Continuous Operations
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

In the last blog post we integrated Jenkins and Octopus to trigger a deployment to Kubernetes once the Docker image was pushed to Docker Hub. We also added some additional environments in Octopus to represent the canonical Dev -> Test -> Prod progression. This left us with a complete CI/CD pipeline with automated (if not necessarily automatic) deployments to our environments.

While a traditional CI/CD pipeline ends with a deployment to production, Octopus treats deployments as the beginning of a new phase called Continuous Operations (CO). By automating common tasks like database backups, log collection, and service restarts via runbooks, Octopus provides a complete CI/CD/CO pipeline covering the entire lifecycle of an application.

## Adding a database

Before we can create runbooks for database backups, we first need a database.

In a production setting you would typically use a hosted service like RDS. You get out of the box high availability, backups, maintenance windows, security and more, all of which would require a significant effort to replicate with a local database. However, for the purposes of this blog we'll deploy MySQL to EKS and point our pet clinic application to it. We can then script some common management tasks against the database to demonstrate the kind of continuous operations that keep a production deployment running.

We'll use the official [MySQL](https://hub.docker.com/_/mysql) Docker image, but we also need some additional tools on the image to allow us to transfer a backup to a second location. Since we are using AWS to host our Kubernetes cluster, we'll backup our database to S3. This means we need the AWS CLI included in the MySQL Docker image to transfer a database backup.

Adding new tools to an image is easy. We take the base **mysql** image and run the commands required to install the AWS CLI. So our [Dockerfile](https://github.com/mcasperson/mysqlwithawscli/blob/master/Dockerfile) looks like this:

```
FROM mysql
RUN apt-get update; apt-get install python python-pip -y
RUN pip install awscli
```

We then build this image, push it to Docker Hub, and create and deploy a release in Octopus with the following [Jenkinsfile](https://github.com/mcasperson/mysqlwithawscli/blob/master/Jenkinsfile). You will note that this Jenkinsfile is almost an exact copy of the previous one, with some changes to the name of the Docker image and the Octopus project that is deployed:

```groovy
pipeline {
    agent {
        label 'docker'
    }
    //  parameters here provide the shared values used with each of the Octopus pipeline steps.
    parameters {
        // The space ID that we will be working with. The default space is typically Spaces-1.
        string(defaultValue: 'Spaces-1', description: '', name: 'SpaceId', trim: true)
        // The Octopus project we will be deploying.
        string(defaultValue: 'MySQL', description: '', name: 'ProjectName', trim: true)
        // The environment we will be deploying to.
        string(defaultValue: 'Dev', description: '', name: 'EnvironmentName', trim: true)
        // The name of the Octopus instance in Jenkins that we will be working with. This is set in:
        // Manage Jenkins -> Configure System -> Octopus Deploy Plugin
        string(defaultValue: 'Octopus', description: '', name: 'ServerId', trim: true)
    }
    stages {
        stage ('Add tools') {
            steps {
                tool('OctoCLI')
            }
        }
        stage('Building our image') {
            steps {
                script {
                    dockerImage = docker.build "mcasperson/mysqlwithawscli:$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy our image') {
            steps {
                script {
                    // Assume the Docker Hub registry by passing an empty string as the first parameter
                    docker.withRegistry('' , 'dockerhub') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('deploy') {
            steps {                                
                octopusCreateRelease deployThisRelease: true, environment: "${EnvironmentName}", project: "${ProjectName}", releaseVersion: "1.0.${BUILD_NUMBER}", serverId: "${ServerId}", spaceId: "${SpaceId}", toolId: 'Default', waitForDeployment: true                
            }
        }
    }
}
```

The Kubernetes deployment YAML is also very similar to our previous example, with a new image name and the addition of two environment variables to configure the database credentials and create an initial database:

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    spec:
      containers:
        - name: mysql
          image: mcasperson/mysqlwithawscli
          ports:
            - name: sql
              containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: Password01!
            - name: MYSQL_DATABASE
              value: petclinic
```

Because we don't need to access the database publicly, we expose the MySQL instance with a cluster IP service, which allows other pods to access the database, but will not create a public load balancer:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: ClusterIP
  ports:
    - name: sql
      port: 3306
      protocol: TCP
```

Deploying the resources created by the YAML above results in a MySQL instance that can be accessed from other pods in the cluster.

To configure pet clinic to use the MySQL database we need to define four environment variables:

* `MYSQL_URL`, which is the JDBC URL to the MySQL database.
* `MYSQL_USER`, which is the MySQL user to connect as, set to `root`.
* `MYSQL_PASS`, which is the MySQL password, set to the password we set in the same value as the `MYSQL_ROOT_PASSWORD` environment variable on the MySQL pod.
* `SPRING_PROFILES_ACTIVE`, which defines the profile file that Spring will use to configure the application, set to [mysql](https://github.com/mcasperson/spring-petclinic/blob/main/src/main/resources/application-mysql.properties).

The YAML for our new pet clinic deployment is shown below:

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: petclinic
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: petclinic
          image: mcasperson/petclinic
          ports:
            - name: web
              containerPort: 8080
          env:
            - name: MYSQL_URL
              value: 'jdbc:mysql://mysql/petclinic'
            - name: SPRING_PROFILES_ACTIVE
              value: mysql
            - name: MYSQL_USER
              value: root
            - name: MYSQL_PASS
              value: Password01!
```