---
title: Creating a Kubernetes Operator with Kotlin
description: Learn what Kubernetes Operators are, and see an example Kotlin Operator
author: matthew.casperson@octopus.com
visibility: public
published: 2020-02-24
metaImage: java-octopus.png
bannerImage: java-octopus.png
bannerImageAlt: Creating a Kubernetes Operator with Kotlin
tags:
 - DevOps
 - Kubernetes
---

![Creating a Kubernetes Operator with Kotlin](java-octopus.png)

Most environments will initially treat their Kubernetes cluster as a tool to orchestrate containers and configure traffic between them. Kubernetes supports this use case very well by providing declarative descriptions of the desired container state and their connections.

When used in this way, developers and operations staff sit outside of the cluster, looking in. The cluster is managed with calls to `kubectl` that are made in an ad-hoc fashion or from a CI/CD pipeline. This means Kubernetes itself is quite naïve; it understands how to reconfigure itself to match the desired state, but it has no understanding of what that state represents.

For example, a common Kubernetes deployment might see three pods created: a front end web application, a backend web service, and a database. The relationship between these pods is well understood by the developers deploying them as a classic three-tier architecture, but Kubernetes literally sees nothing more than three pods to be deployed, monitored, and exposed to network traffic.

The operator pattern has evolved as a way of encapsulating business knowledge and operational workflows in the Kubernetes cluster itself, allowing a cluster to implement high level, domain-specific concepts with the common, low-level resources like pods, services, and deployments, etc.

The term was originally coined by Brandon Philips in the blog post [Introducing Operators: Putting Operational Knowledge into Software](https://coreos.com/blog/introducing-operators.html) and offers this definition:

> It builds upon the basic Kubernetes resource and controller concepts but includes domain or application-specific knowledge to automate common tasks.

The three key components identified in this definition are:

* Resource
* Controller
* Domain or application-specific knowledge

In practice, a *resource* means a Custom Resource Definition (CRD), a *controller* means an application integrated into and responding to the Kubernetes API, and the *application-specific knowledge* is the logic implemented in the *controller* to reify high-level concepts from standard Kubernetes resources.

To understand the operator pattern, let’s look at a simple example written in Kotlin. The code for this operator is available from [GitHub](https://github.com/OctopusSamples/KotlinK8SOperator), and it is based on the code from this [RedHat blog](https://developers.redhat.com/blog/2019/10/07/write-a-simple-kubernetes-operator-in-java-using-the-fabric8-kubernetes-client/). The operator will extend the Kubernetes cluster with the concept of a web server with a `WebServer` CRD and a controller that builds pods with an image known to expose a sample web server.

The CRD meets the *resource* requirement, the code we’ll write to interact with the Kubernetes API meets the *controller* requirement, and the knowledge that a particular Docker image is used to expose a sample web server is the *application-specific knowledge*.

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

Before we dive into the Kotlin code, we need to understand the common structure of all Kubernetes resources. Here is the YAML definition of a deployment resource that we’ll use as an example:

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

The first component is the Group, Version, and Kind (GVK). The deployment resource has a group of `apps`, a version of `v1` and a kind of `Deployment`:

```YAML
apiVersion: apps/v1
kind: Deployment
```

The second component is the metadata. This is where labels, annotations, names, and namespaces are defined:

```YAML
metadata:
  name: nginx-deployment
  labels:
    app: nginx
```

The third component is the spec, which defines the properties of the specific resource:

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

Now that we know the components that make up a Kubernetes resource, we can look at the code that reflects the CRD implemented by the operator.

We are creating a new CRD called `WebServer`, which is represented by a class also called `WebServer`. This class has two properties defining the spec and the status:

```Kotlin
package com.octopus.webserver.operator.crd

import io.fabric8.kubernetes.client.CustomResource

data class WebServer(var spec: WebServerSpec = WebServerSpec(),
                     var status: WebServerStatus = WebServerStatus()) : CustomResource()
```

The spec for our CRD is represented in the `WebServerSpec` class. This has a single field called `replicas` indicating how many web server pods this CRD is responsible for creating:

```
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
import io.fabric8.kubernetes.api.model.KubernetesResource

@JsonDeserialize
data class WebServerStatus(var count: Int = 0) : KubernetesResource
```

The final two classes, called `WebServerList` and `DoneableWebServer`, contain no custom properties or logic, and are boilerplate code required by the fabric8 library:

```
package com.octopus.webserver.operator.crd

import io.fabric8.kubernetes.client.CustomResourceList

class WebServerList : CustomResourceList<WebServer>()
```

```
package com.octopus.webserver.operator.crd

import io.fabric8.kubernetes.client.CustomResourceDoneable
import io.fabric8.kubernetes.api.builder.Function

class DoneableWebServer(resource: WebServer, function: Function<WebServer,WebServer>) :
        CustomResourceDoneable<WebServer>(resource, function)
```

## The main function

The `main()` function is the entry point to our controller. Here is the complete code:

```
package com.octopus.webserver.operator

import com.octopus.webserver.operator.controller.WebServerController
import com.octopus.webserver.operator.crd.WebServer
import com.octopus.webserver.operator.crd.WebServerList
import io.fabric8.kubernetes.api.model.Pod
import io.fabric8.kubernetes.api.model.PodList
import io.fabric8.kubernetes.api.model.apiextensions.CustomResourceDefinitionBuilder
import io.fabric8.kubernetes.client.DefaultKubernetesClient
import io.fabric8.kubernetes.client.dsl.base.CustomResourceDefinitionContext


fun main(args: Array<String>) {
    val client = DefaultKubernetesClient()
    client.use {
        val namespace = client.namespace ?: "default"
        val podSetCustomResourceDefinition = CustomResourceDefinitionBuilder()
                .withNewMetadata().withName("webservers.demo.k8s.io").endMetadata()
                .withNewSpec()
                .withGroup("demo.k8s.io")
                .withVersion("v1alpha1")
                .withNewNames().withKind("WebServer").withPlural("webservers").endNames()
                .withScope("Namespaced")
                .endSpec()
                .build()
        val webServerCustomResourceDefinitionContext = CustomResourceDefinitionContext.Builder()
                .withVersion("v1alpha1")
                .withScope("Namespaced")
                .withGroup("demo.k8s.io")
                .withPlural("webservers")
                .build()
        val informerFactory = client.informers()
        val podSharedIndexInformer = informerFactory.sharedIndexInformerFor(
                Pod::class.java,
                PodList::class.java,
                10 * 60 * 1000.toLong())
        val webServerSharedIndexInformer = informerFactory.sharedIndexInformerForCustomResource(
                webServerCustomResourceDefinitionContext,
                WebServer::class.java,
                WebServerList::class.java,
                10 * 60 * 1000.toLong())
        val webServerController = WebServerController(
                client,
                podSharedIndexInformer,
                webServerSharedIndexInformer,
                podSetCustomResourceDefinition,
                namespace)

        webServerController.create()
        informerFactory.startAllRegisteredInformers()

        webServerController.run()
    }
}
```

We create a `DefaultKubernetesClient`, which gives us access to the Kubernetes API:

```
val client = DefaultKubernetesClient()
```

The client knows how to configure itself based on the environment it is executed in. We’ll run this code locally when testing, meaning the client will access the details of the Kubernetes cluster from the `~/.kube/config` file. The namespace is then extracted from the client’s configuration, or set to `default` if no namespace setting was found:

```
val namespace = client.namespace ?: "default"
```

The `CustomResourceDefinitionBuilder` defines the `WebServer` CRD that this controller manages. This is used when working with the client to update resources in the cluster.

```
val podSetCustomResourceDefinition = CustomResourceDefinitionBuilder()
        .withNewMetadata().withName("webservers.demo.k8s.io").endMetadata()
        .withNewSpec()
        .withGroup("demo.k8s.io")
        .withVersion("v1alpha1")
        .withNewNames().withKind("WebServer").withPlural("webservers").endNames()
        .withScope("Namespaced")
        .endSpec()
        .build()
```

The controller works by listening to events that indicate the resources it should be managing have changed. To listen to events relating to the `WebServer` CRD, we create a `CustomResourceDefinitionContext`:

```
val webServerCustomResourceDefinitionContext = CustomResourceDefinitionContext.Builder()
        .withVersion("v1alpha1")
        .withScope("Namespaced")
        .withGroup("demo.k8s.io")
        .withPlural("webservers")
        .build()
```

We are notified of events through informers, and the informers are created from a factory provided by the client:

```
val informerFactory = client.informers()
```

Here we create an informer that will notify us of events relating to pods. Because pods are a standard resource in Kubernetes, creating this informer did not require a `CustomResourceDefinitionContext`:

```
val podSharedIndexInformer = informerFactory.sharedIndexInformerFor(
        Pod::class.java,
        PodList::class.java,
        10 * 60 * 1000.toLong())
```

Here we create an informer that will notify us of events relating to our CRD. This required the `CustomResourceDefinitionContext` created previously:

```
val webServerSharedIndexInformer = informerFactory.sharedIndexInformerForCustomResource(
        webServerCustomResourceDefinitionContext,
        WebServer::class.java,
        WebServerList::class.java,
        10 * 60 * 1000.toLong())
```

The logic of the operator is contained in the controller. In this project, the `WebServerController` class fulfills the role of the controller:

```
val webServerController = WebServerController(
        client,
        podSharedIndexInformer,
        webServerSharedIndexInformer,
        podSetCustomResourceDefinition,
        namespace)
```

The controller links up event handlers in the `create()` method, we start listening for events, and then enter the reconcile loop by calling the `run()` method:

```
webServerController.create()
informerFactory.startAllRegisteredInformers()

webServerController.run()
```

## The controller

The `WebServerController` class implements the controller in our operator. Its job is to listen for changes to Kubernetes resources and reconcile the current state with the desired state. The complete code for the class is shown below:

```
package com.octopus.webserver.operator.controller

import com.octopus.webserver.operator.crd.DoneableWebServer
import com.octopus.webserver.operator.crd.WebServer
import com.octopus.webserver.operator.crd.WebServerList
import io.fabric8.kubernetes.api.model.OwnerReference
import io.fabric8.kubernetes.api.model.Pod
import io.fabric8.kubernetes.api.model.PodBuilder
import io.fabric8.kubernetes.api.model.apiextensions.CustomResourceDefinition
import io.fabric8.kubernetes.client.KubernetesClient
import io.fabric8.kubernetes.client.informers.ResourceEventHandler
import io.fabric8.kubernetes.client.informers.SharedIndexInformer
import io.fabric8.kubernetes.client.informers.cache.Cache
import io.fabric8.kubernetes.client.informers.cache.Lister
import java.util.*
import java.util.AbstractMap.SimpleEntry
import java.util.concurrent.ArrayBlockingQueue


class WebServerController(private val kubernetesClient: KubernetesClient,
                          private val podInformer: SharedIndexInformer<Pod>,
                          private val webServerInformer: SharedIndexInformer<WebServer>,
                          private val webServerResourceDefinition: CustomResourceDefinition,
                          private val namespace: String) {
    private val APP_LABEL = "app"
    private val webServerLister = Lister<WebServer>(webServerInformer.indexer, namespace)
    private val podLister = Lister<Pod>(podInformer.indexer, namespace)
    private val workQueue = ArrayBlockingQueue<String>(1024)

    fun create() {
        webServerInformer.addEventHandler(object : ResourceEventHandler<WebServer> {
            override fun onAdd(webServer: WebServer) {
                enqueueWebServer(webServer)
            }

            override fun onUpdate(webServer: WebServer, newWebServer: WebServer) {
                enqueueWebServer(newWebServer)
            }

            override fun onDelete(webServer: WebServer, b: Boolean) {}
        })

        podInformer.addEventHandler(object : ResourceEventHandler<Pod> {
            override fun onAdd(pod: Pod) {
                handlePodObject(pod)
            }

            override fun onUpdate(oldPod: Pod, newPod: Pod) {
                if (oldPod.metadata.resourceVersion == newPod.metadata.resourceVersion) {
                    return
                }
                handlePodObject(newPod)
            }

            override fun onDelete(pod: Pod, b: Boolean) {}
        })
    }

    private fun enqueueWebServer(webServer: WebServer) {
        val key: String = Cache.metaNamespaceKeyFunc(webServer)
        if (key.isNotEmpty()) {
            workQueue.add(key)
        }
    }

    private fun handlePodObject(pod: Pod) {
        val ownerReference = getControllerOf(pod)

        if (ownerReference?.kind?.equals("WebServer", ignoreCase = true) != true) {
            return
        }

        webServerLister
                .get(ownerReference.name)
                ?.also { enqueueWebServer(it) }
    }

    private fun getControllerOf(pod: Pod): OwnerReference? =
            pod.metadata.ownerReferences.firstOrNull { it.controller }

    private fun reconcile(webServer: WebServer) {
        val pods = podCountByLabel(APP_LABEL, webServer.metadata.name)
        val existingPods = pods.size

        webServer.status.count = existingPods
        updateStatus(webServer)

        if (existingPods < webServer.spec.replicas) {
            createPod(webServer)
        } else if (existingPods > webServer.spec.replicas) {
            kubernetesClient
                    .pods()
                    .inNamespace(webServer.metadata.namespace)
                    .withName(pods[0])
                    .delete()
        }
    }

    private fun updateStatus(webServer: WebServer) =
            kubernetesClient.customResources(webServerResourceDefinition, WebServer::class.java, WebServerList::class.java, DoneableWebServer::class.java)
                    .inNamespace(webServer.metadata.namespace)
                    .withName(webServer.metadata.name)
                    .updateStatus(webServer)

    private fun podCountByLabel(label: String, webServerName: String): List<String> =
            podLister.list()
                    .filter { it.metadata.labels.entries.contains(SimpleEntry(label, webServerName)) }
                    .filter { it.status.phase == "Running" || it.status.phase == "Pending" }
                    .map { it.metadata.name }

    private fun createPod(webServer: WebServer) =
            createNewPod(webServer).let { pod ->
                kubernetesClient.pods().inNamespace(webServer.metadata.namespace).create(pod)
            }

    private fun createNewPod(webServer: WebServer): Pod =
            PodBuilder()
                    .withNewMetadata()
                    .withGenerateName(webServer.metadata.name.toString() + "-pod")
                    .withNamespace(webServer.metadata.namespace)
                    .withLabels(Collections.singletonMap(APP_LABEL, webServer.metadata.name))
                    .addNewOwnerReference()
                    .withController(true)
                    .withKind("WebServer")
                    .withApiVersion("demo.k8s.io/v1alpha1")
                    .withName(webServer.metadata.name)
                    .withNewUid(webServer.metadata.uid)
                    .endOwnerReference()
                    .endMetadata()
                    .withNewSpec()
                    .addNewContainer().withName("nginx").withImage("nginxdemos/hello").endContainer()
                    .endSpec()
                    .build()

    fun run() {
        blockUntilSynced()
        while (true) {
            try {
                workQueue
                        .take()
                        .split("/")
                        .toTypedArray()[1]
                        .let { webServerLister.get(it) }
                        ?.also { reconcile(it) }
            } catch (interruptedException: InterruptedException) {
                // ignored
            }
        }
    }

    private fun blockUntilSynced() {
        while (!podInformer.hasSynced() || !webServerInformer.hasSynced()) {}
    }
}
```

The `create()` method assigns anonymous classes as informer event handlers. The event handlers identify instances of the `WebServer` CRDs that need to be processed by calling either `enqueueWebServer()` or `handlePodObject()`:

```
fun create() {
        webServerInformer.addEventHandler(object : ResourceEventHandler<WebServer> {
            override fun onAdd(webServer: WebServer) {
                enqueueWebServer(webServer)
            }

            override fun onUpdate(webServer: WebServer, newWebServer: WebServer) {
                enqueueWebServer(newWebServer)
            }

            override fun onDelete(webServer: WebServer, b: Boolean) {}
        })

        podInformer.addEventHandler(object : ResourceEventHandler<Pod> {
            override fun onAdd(pod: Pod) {
                handlePodObject(pod)
            }

            override fun onUpdate(oldPod: Pod, newPod: Pod) {
                if (oldPod.metadata.resourceVersion == newPod.metadata.resourceVersion) {
                    return
                }
                handlePodObject(newPod)
            }

            override fun onDelete(pod: Pod, b: Boolean) {}
        })
    }
```

`enqueueWebServer()` creates a key identifying the `WebServer` CRD and adds it to the `workQueue`:

```
private fun enqueueWebServer(webServer: WebServer) {
    val key: String = Cache.metaNamespaceKeyFunc(webServer)
    if (key.isNotEmpty()) {
        workQueue.add(key)
    }
}
```

`handlePodObject()` first determines if the pod is managed by a `WebServer` through the ownerReference. If it is, the owning `WebServer` is added to the `workQueue` by calling `enqueueWebServer()`:

```
private fun handlePodObject(pod: Pod) {
    val ownerReference = getControllerOf(pod)

    if (ownerReference?.kind?.equals("WebServer", ignoreCase = true) != true) {
        return
    }

    webServerLister
            .get(ownerReference.name)
            ?.also { enqueueWebServer(it) }
}

private fun getControllerOf(pod: Pod): OwnerReference? =
        pod.metadata.ownerReferences.firstOrNull { it.controller }
```

`reconcile()` provides the logic that ensures the cluster has as many pods as required by the `WebServer` CRD. It calls `podCountByLabel()` to find out how many pods exist, and updates the status of the CRD with a call to `updateStatus()`. If there are too few pods to meet the requirements, `createPod()` is called. If there are too many pods, one is deleted.

By continually creating or deleting pods to push the cluster towards the desired state, we will eventually satisfy the requirements of the `WebServer` CRD:

```
private fun reconcile(webServer: WebServer) {
    val pods = podCountByLabel(APP_LABEL, webServer.metadata.name)
    val existingPods = pods.size

    webServer.status.count = existingPods
    updateStatus(webServer)

    if (existingPods < webServer.spec.replicas) {
        createPod(webServer)
    } else if (existingPods > webServer.spec.replicas) {
        kubernetesClient
                .pods()
                .inNamespace(webServer.metadata.namespace)
                .withName(pods[0])
                .delete()
    }
}
```

`updateStatus()` uses the client to update the status component of our custom resource. The status component is unique because updating it does not trigger an update event in our code. Only a controller can update the status component of a resource, and Kubernetes has been designed to prevent status updates from triggering an infinite event loop:

```
private fun updateStatus(webServer: WebServer) =
        kubernetesClient.customResources(webServerResourceDefinition, WebServer::class.java, WebServerList::class.java, DoneableWebServer::class.java)
                .inNamespace(webServer.metadata.namespace)
                .withName(webServer.metadata.name)
                .updateStatus(webServer)
```

`podCountByLabel()` returns the names of pods that are managed by the CRD that are either running or in the process of being created:

```
private fun podCountByLabel(label: String, webServerName: String): List<String> =
        podLister.list()
                .filter { it.metadata.labels.entries.contains(SimpleEntry(label, webServerName)) }
                .filter { it.status.phase == "Running" || it.status.phase == "Pending" }
                .map { it.metadata.name }
```

`createPod()` and `createNewPod()` create a new pod. It is here that our business logic has been codified with the use of the `nginxdemos/hello` Docker image as our test web server:

```
private fun createPod(webServer: WebServer) =
        createNewPod(webServer).let { pod ->
            kubernetesClient.pods().inNamespace(webServer.metadata.namespace).create(pod)
        }

private fun createNewPod(webServer: WebServer): Pod =
        PodBuilder()
                .withNewMetadata()
                .withGenerateName(webServer.metadata.name.toString() + "-pod")
                .withNamespace(webServer.metadata.namespace)
                .withLabels(Collections.singletonMap(APP_LABEL, webServer.metadata.name))
                .addNewOwnerReference()
                .withController(true)
                .withKind("WebServer")
                .withApiVersion("demo.k8s.io/v1alpha1")
                .withName(webServer.metadata.name)
                .withNewUid(webServer.metadata.uid)
                .endOwnerReference()
                .endMetadata()
                .withNewSpec()
                .addNewContainer().withName("nginx").withImage("nginxdemos/hello").endContainer()
                .endSpec()
                .build()
```

The `run()` method is an infinite loop continually consuming a web server resource ID added to the `workQueue` by the event listeners and passing it to the `reconcile()` method:

```
fun run() {
    blockUntilSynced()
    while (true) {
        try {
            workQueue
                    .take()
                    .split("/")
                    .toTypedArray()[1]
                    .let { webServerLister.get(it) }
                    ?.also { reconcile(it) }
        } catch (interruptedException: InterruptedException) {
            // ignored
        }
    }
}

private fun blockUntilSynced() {
    while (!podInformer.hasSynced() || !webServerInformer.hasSynced()) {}
}
```

## The CRD YAML

The final piece of the operator is the CRD itself. A CRD is simply another Kubernetes resource, and we define it in the following YAML:

```YAML
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: webservers.demo.k8s.io
spec:
  group: demo.k8s.io
  version: v1alpha1
  names:
    kind: WebServer
    plural: webservers
  scope: Namespaced
  subresources:
    status: {}
```

## Putting it all together

To run the operator, we first need to apply the CRD YAML:

```
kubectl apply -f crd.yml
```

Then, we create an instance of our CRD with the YAML:

```YAML
apiVersion: demo.k8s.io/v1alpha1
kind: WebServer
metadata:
  name: example-webserver
spec:
  replicas: 5
```

The controller can then run locally. Because the client we used in our code knows how to configure itself based on where it is run, executing our code locally, means that the client configures itself from the `~/.kube/config` file. In the screenshot below you can see the controller run directly from my IDE:

![](intellij.png "width=500")

The controller responds to the new web server CRD and creates the required pods:

```
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
example-webserver-pod92ht9   1/1     Running   0          54s
example-webserver-podgbz86   1/1     Running   0          54s
example-webserver-podk58gz   1/1     Running   0          54s
example-webserver-podkftmp   1/1     Running   0          54s
example-webserver-podpwzrt   1/1     Running   0          54s
```

The status of the web server resource is updated with the `count` of the pods it has successfully created:

```
$ kubectl get webservers -n default -o yaml
apiVersion: v1
items:
- apiVersion: demo.k8s.io/v1alpha1
  kind: WebServer
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"demo.k8s.io/v1alpha1","kind":"WebServer","metadata":{"annotations":{},"name":"example-webserver","namespace":"default"},"spec":{"replicas":5}}
    creationTimestamp: "2020-01-16T20:19:23Z"
    generation: 1
    name: example-webserver
    namespace: default
    resourceVersion: "112308"
    selfLink: /apis/demo.k8s.io/v1alpha1/namespaces/default/webservers/example-webserver
    uid: 9eb08575-8fa1-4bc9-bb2b-6f11b7285b68
  spec:
    replicas: 5
  status:
    count: 5
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

## The power of operators

Without an operator, the concept of a test web server lived outside of the cluster. Developers may have emailed around the YAML they use to create test pods with, but more likely, everyone had their own opinion of what a test web server was.

The operator we created extends our Kubernetes cluster with a specific implementation of a test web server. Encapsulating this business knowledge allows the cluster to create and manage high-level concepts specific to our environment.

Creating and managing new resources is just one example of what an operator can do. Automating tasks like security scans, reporting, and load testing are all valid use cases for operators. A list of popular operators is available [here](https://github.com/operator-framework/awesome-operators).

## Conclusion

Operators are a much hyped but often poorly understood pattern. With the definition from the original blog post describing operators, we saw the three simple parts to an operator: a resource to define them, a controller to act on the Kubernetes resources, and logic to implement application-specific knowledge. We then implemented a simple operator in Kotlin to create test web servers.
