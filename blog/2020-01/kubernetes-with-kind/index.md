---
title: Kubernetes testing with KIND
description: KIND offers a new solution for spinning up Kubernetes clusters for testing and development
author: matthew.casperson@octopus.com
visibility: public
published: 2020-01-20
metaImage: kubernetes-in-docker.png
bannerImage: kubernetes-in-docker.png
bannerImageAlt: Kubernetes in Docker (KIND)
tags:
 - DevOps
 - Kubernetes
---

![Kubernetes in Docker (KIND)](kubernetes-in-docker.png)

There are many solutions for creating a local test Kubernetes environment, such as [minikube](https://github.com/kubernetes/minikube) or [MicroK8s](https://microk8s.io/), but a new project called [KIND](https://github.com/kubernetes-sigs/kind) offers a fresh approach that may interest Kubernetes developers and administrators.

KIND stands for Kubernetes IN Docker, and as the name suggests, it creates a Kubernetes cluster using Docker to host the nodes. This is a novel approach, that takes advantage of Docker’s easy, self-contained deployments and cleanup to create the test Kubernetes infrastructure.

## KIND Installation

Make sure you have [Docker](https://docs.docker.com/install/) and [Go](https://golang.org/doc/install) installed, and then install KIND with the command:

```
GO111MODULE="on" go get sigs.k8s.io/kind@v0.6.1
```

This will place the `kind` executable in the directory `$GOPATH/bin/kind`, which will be `~/go/bin/kind` by default.

Assuming the Go `bin` directory is in the `PATH`, build a test cluster with the command:

```
kind create cluster --name mycluster
```

The first cluster takes a little while to download KIND Docker images, although subsequent clusters take less than a minute to create.

KIND will add the new cluster details as a context to your `~/.kube/config` file, so you can test the cluster is up and running with the command:

```
kubectl cluster-info --context kind-mycluster
```

You can set this context as the default with the command:

```
kubectl config use-context kind-mycluster
```

## Using the KIND Kubernetes cluster

The most immediate issue I ran into when using KIND was accessing the services deployed to the cluster.

By default KIND only exposes the Kubernetes API server. This means `kubectl` will work as expected. So you can deploy to the cluster and query resources, but accessing your services requires some extra work.

A solution is to use `kubectl` as a proxy:

```
kubectl proxy --port=8080
```

Kubernetes services are then available through a specifically formed URL like http://localhost:8080/api/v1/namespaces/default/services/webserver:http-web/proxy/.

The [Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-services/#manually-constructing-apiserver-proxy-urls) has more details on these proxy URLs:

> To create proxy URLs that include service endpoints, suffixes, and parameters, you simply append to the service’s proxy URL: http://kubernetes_master_address/api/v1/namespaces/namespace_name/services/[https:]service_name[:port_name]/proxy

Port forwarding removes the need to construct special URLs. Start port forwarding with a command like:

```
kubectl port-forward svc/webserver 8081:80
```

The service is now available on the local port of 8081, directing traffic to the service port 80. This means the URL http://localhost:8081 can be used to access the service.

You can also use port forwarding on an ingress controller:

```
kubectl port-forward svc/nginx-release-nginx-ingress 8081:80
```

This command allows you to access services via any ingress rules you have configured.

The KIND documentation also provides some additional details on how to [expose ingress controllers](https://kind.sigs.k8s.io/docs/user/ingress/) for a more permanent solution.

## First impressions

Once I got around the issue of accessing my services, KIND performed remarkably well for a beta release. External tools like Helm worked fine, and I could deploy custom dashboards to the cluster.

I appreciated the fact the KIND was so self-contained. Because everything is a Docker container, creating the cluster was quick, and when it was cleaned up afterward there was nothing left running on the system.

## Conclusion

Getting a Kubernetes cluster running locally isn’t that difficult these days, but KIND makes it especially easy to create a cluster. Admittedly, Kubernetes running on Docker to orchestrate Docker is a little mind bending, but it can’t be beaten for convenience.

The real value of KIND is the ability to run it as part of automated tests. I didn’t have a use case for this personally, but I’m sure it lives up to the promise.

I’ll seriously consider using KIND over minikube for local testing from now on.
