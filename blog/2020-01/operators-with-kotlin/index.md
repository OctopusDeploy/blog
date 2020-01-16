---
title: Creating a Kubernetes Operator with Kotlin
description: Learn what Kubernetes Operators are, and see an example Kotlin Operator
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage:
bannerImage:
tags:
 - DevOps
---

Most environments will initially treat their Kubernetes cluster as tool to orchestrate containers and configure traffic between them. Kubernetes supports this use case very well by providing declarative descriptions of the desired container state and their connections.

When used in this way, developers and operations staff sit outside of the cluster and look in. The cluster is managed with calls to `kubectl` made in an ad-hoc fashion or from a CI/CD pipeline. This means Kubernetes itself is quite naïve; it understand how to reconfigure itself to match the desired state, but has no understanding of what that state represents.

For example, a common Kubernetes deployment might see three pods created: a front end web application, a backend web service and a database. The relationship between these pods is well understood by the developers deploying them, but Kubernetes only sees three pods to be deployed, monitored and exposed to network traffic.

The operator pattern has evolved as a way of encapsulating business knowledge and operational workflows in the Kubernetes cluster itself, allowing a cluster to implement high level, domain specific concepts with the common, low level resources like pods, services, deployments etc.

The term was originally coined by Brandon Philips in the blog post [Introducing Operators: Putting Operational Knowledge into Software](https://coreos.com/blog/introducing-operators.html), and offers this definition:

> It builds upon the basic Kubernetes resource and controller concepts but includes domain or application-specific knowledge to automate common tasks.

The three key components identified in this definition are:

* resource
* controller
* domain or application-specific knowledge

In practice, a *resource* means a Custom Resource Definition (CRD), a *controller* means an application integrated into the Kubernetes API, and the *application-specific knowledge* is the logic implemented in the *controller* to implement high level concepts (like a three-tier application) from standard Kubernetes resources.

To understand the operator pattern, let's look at a simple example written in Kotlin. This operator will enrich the Kubernetes cluster with the concept of a web server with a `WebServer` CRD and a controller that builds pods with a image known to provide an sample web server.

The CRD meets the *resource* requirement, the code we'll write interacting with the Kubernetes API meets the *controller* requirement, and the knowledge that a particular Docker image can be used to implement a sample web server is the *application-specific knowledge*.

## The pom.xml file

We start with the Maven `pom.xml` file. This file defines the dependencies required for Kotlin itself and the [fabric8 Kubernetes client library](https://github.com/fabric8io/kubernetes-client). The complete `pom.xml` file is shown below:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.octopus</groupId>
    <artifactId>kotlink8soperator</artifactId>
    <version>1.0</version>

    <properties>
        <kotlin.version>1.3.61</kotlin.version>
        <version.fabric8.client>4.7.0</version.fabric8.client>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-stdlib</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
        <dependency>
            <groupId>io.fabric8</groupId>
            <artifactId>kubernetes-client</artifactId>
            <version>${version.fabric8.client}</version>
        </dependency>
    </dependencies>

    <build>
        <sourceDirectory>${project.basedir}/src/main/kotlin</sourceDirectory>
        <testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>

        <plugins>
            <plugin>
                <groupId>org.jetbrains.kotlin</groupId>
                <artifactId>kotlin-maven-plugin</artifactId>
                <version>${kotlin.version}</version>

                <executions>
                    <execution>
                        <id>compile</id>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>

                    <execution>
                        <id>test-compile</id>
                        <goals>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## Anatomy of a Kubernetes resource

Before we dive into the Kotlin code, we first need to understand the common structure of all Kubernetes resources. Here is the YAML definition of a deployment resource that we'll use as an example:

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
status:
  availableReplicas: 2
  observedGeneration: 1
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2
```

This resource can be broken down into four components.

The first component is the Group, Version, Kind (GVK). The deployment resource has a group of `apps`, a version of `v1` and a kind of `Deployment`:

```YAML
apiVersion: apps/v1
kind: Deployment
```

The second component is the metadata. This is where labels, annotations, names and namespaces are defined:

```YAML
metadata:
  name: nginx-deployment
  labels:
    app: nginx
```

The third component is the spec, which defines the properties of the specific resource.

```YAML
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

The fourth component is the status. The details in this component are generated by Kubernetes to reflect the current state of the resource:

```YAML
status:
  availableReplicas: 2
  observedGeneration: 1
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2
```

## The CRD classes

Now that we know the components that make up a Kubernetes resource, we can look at the code that reflects the CRD implemented by this operator.

We are creating a new CRD called `WebServer`, which is represented by a class also called `WebServer`. This class has two properties defining the spec and the status:

```Kotlin
package com.octopus.webserver.operator.crd

import io.fabric8.kubernetes.client.CustomResource

data class WebServer(var spec: WebServerSpec = WebServerSpec(),
                     var status: WebServerStatus = WebServerStatus()) : CustomResource()
```

The spec for our CRD is represented in the `WebServerSpec` class. This class is extremely simple, with a single field called `replicas` indicating how many web server pods this CRD is responsible for creating:

```Kotlin
package com.octopus.webserver.operator.crd

import com.fasterxml.jackson.databind.annotation.JsonDeserialize
import io.fabric8.kubernetes.api.model.KubernetesResource

@JsonDeserialize
data class WebServerSpec(val replicas: Int = 0) : KubernetesResource
```

The status of our CRD is represented in the `WebServerStatus` class. It contains a single field called `count` that reports how many pods have been created:

```
package com.octopus.webserver.operator.crd

import com.fasterxml.jackson.databind.annotation.JsonDeserialize

@JsonDeserialize
data class WebServerStatus(var count: Int = 0)
```

A final class called `WebServerList` represents a collection of the CRD resources. There are no custom properties of logic in the class, as it is boilerplate code required by the fabric8 library:

```
package com.octopus.webserver.operator.crd

import io.fabric8.kubernetes.client.CustomResourceList

class WebServerList : CustomResourceList<WebServer>()
```
