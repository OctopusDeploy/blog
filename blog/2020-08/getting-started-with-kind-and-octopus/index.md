---
title: Getting started with Kind and Octopus
description: Learn how to perform your first Kubernetes deployment with Kind and Octopus
author: matthew.casperson@octopus.com
visibility: public
published: 2020-08-10
metaImage: octopus-kind.png
bannerImage: octopus-kind.png
bannerImageAlt: Getting started with Kind and Octopus
tags:
 - DevOps
 - Kubernetes
---

![Getting started with Kind and Octopus](octopus-kind.png)

When you first get started with Kubernetes, the sheer number of tools and options available can present a significant hurdle before deploying even the simplest example application. Unlike most other platforms, Kubernetes does not provide a standard package that you can download and install onto your local development PC. The community has filled this void with many options like [Minikube](https://github.com/kubernetes/minikube), [MicroK8s](https://microk8s.io/), [k3s](https://k3s.io/), and [Docker Desktop with Kubernetes](https://docs.docker.com/desktop/kubernetes/).

For this blog post, we’ll look at [Kind](https://kind.sigs.k8s.io/). Although any of the previously mentioned solutions are excellent choices, I prefer Kind because it works seamlessly across all major operating systems and plays nicely in [WSL2](https://docs.microsoft.com/en-us/windows/wsl/wsl2-about), which makes it easy for Windows developers jumping between Windows and Linux.

## Install Kind

Kind creates a Kubernetes cluster as Docker containers. It can be a little mind-bending to think of a Docker container implementing the Kubernetes platform, which in turn orchestrates more Docker containers, but in practice, the process of setting up a Kind Kubernetes cluster is quick and easy.

After you have [installed Docker](https://docs.docker.com/get-docker/), install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) and the [Kind executable](https://kind.sigs.k8s.io/docs/user/quick-start/). Both kubectl and Kind are self-contained executables, meaning they only need to be downloaded and saved in a directory on your PATH.

Then create a cluster with the command `kind create cluster`. This command creates or updates the Kubernetes configuration file at `~/.kube/config` with a cluster and user called `kind-kind`. An example of the `config` file is shown below:

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSU...
    server: https://127.0.0.1:55827
  name: kind-kind
contexts:
- context:
    cluster: kind-kind
    user: kind-kind
  name: kind-kind
current-context: kind-kind
kind: Config
preferences: {}
users:
- name: kind-kind
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSU...
    client-key-data: LS0tLS1CRUdJTiBSU0EgUF...
```

To verify the cluster is running, execute `kubectl get nodes`. You should see output that looks something like this:

```
$ kubectl get nodes
NAME                 STATUS   ROLES    AGE    VERSION
kind-control-plane   Ready    master   101s   v1.18.2
```

We now have a local Kubernetes cluster ready for testing.

## Extracting the certificates

The `config` file created by Kind embeds a cluster certificate used to secure API traffic, and a client key and certificate used to identify the Kubernetes user. We need to extract these values into files that can be imported into Octopus.

The Bash and PowerShell scripts below extract the data, decode it, and combine the client key and certificate into a single PFX file. These scripts give us two files: `cluster.crt` and `client.pfx`:

Here is the Bash script:

```bash
kubectl config view --raw -o json | jq -r ".users[] | select(.name==\"$1\") | .user[\"client-certificate-data\"]" | base64 -d > client.crt
kubectl config view --raw -o json | jq -r ".users[] | select(.name==\"$1\") | .user[\"client-key-data\"]" | base64 -d > client.key
kubectl config view --raw -o json | jq -r ".clusters[] | select(.name==\"$1\") | .cluster[\"certificate-authority-data\"]" | base64 -d > cluster.crt
openssl pkcs12 -export -in client.crt -inkey client.key -out client.pfx -passout pass:
rm client.crt
rm client.key
```

Here is the PowerShell script, where the `openssl` executable was downloaded from [here](https://slproweb.com/products/Win32OpenSSL.html):

```powershell
param($username)

kubectl config view --raw -o json |
  ConvertFrom-JSON |
  Select-Object -ExpandProperty users |
  ? {$_.name -eq $username} |
  % {
  	[System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String($_.user.'client-certificate-data')) | Out-File -Encoding "ASCII" client.crt
  	[System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String($_.user.'client-key-data')) | Out-File -Encoding "ASCII" client.key
    & "C:\Program Files\OpenSSL-Win64\bin\openssl" pkcs12 -export -in client.crt -inkey client.key -out client.pfx -passout pass:
    rm client.crt
    rm client.key
  }

  kubectl config view --raw -o json |
  ConvertFrom-JSON |
  Select-Object -ExpandProperty clusters |
  ? {$_.name -eq $username} |
  % {
  	[System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String($_.cluster.'certificate-authority-data')) | Out-File -Encoding "ASCII" cluster.crt
  }
```

## Creating the Octopus Kubernetes target

The files `cluster.crt` and `client.pfx` are uploaded to the Octopus certificate store. Here I have called these certificates **Kind User** and **Kind Cluster Certificate**:

![](certificates.png "width=500")

We also need a local environment:

![](environments.png "width=500")

The final step is to create the Kubernetes target. This target uses the certificate **Kind User** for authentication, **Kind Cluster Certificate** for the server certificate authority, and https://127.0.0.1:55827 for the cluster URL. This URL comes from the `clusters[].clusters.server` field in the Kubernetes `config` file:

![](k8starget.png "width=500")

## A word on Workers

Because the Kubernetes URL references `127.0.0.1` (or `localhost`), we either need to run Octopus on our local development PC or install a Worker on our local PC, which allows a remote Octopus instance to tunnel into our local PC.

In the screenshots below you can see some of the steps from the Tentacle Manager which configure a Worker:

Select a Polling Tentacle:

![](worker1.png "width=500")

Configure the Tentacle as a Worker:

![](worker2.png "width=500")

Register the Worker with the Octopus Server:

![](worker3.png "width=500")

Here we can see the new Worker assigned to the **Default Worker Pool**:

![](worker-instance.png "width=500")

With the Worker in place, the Kubernetes target on the remote Octopus Server can now access our local Kubernetes cluster:

![](health-check.png "width=500")

## Conclusion

We have now successfully created a local Kubernetes cluster with Kind, extracted the certificates from the Kubernetes configuration file, imported the certificates into Octopus, and created a Kubernetes target in Octopus that connects to our local cluster via a Worker.

From here, we can learn how to use Octopus to deploy Kubernetes resources. The blog posts below show you how to:

* [Deploy your first container to Kubernetes via Octopus](/blog/2020-08/deploy-your-first-container-to-kubernetes/index.md).
* [Import an existing Kubernetes YAML file into Octopus](/blog/2020-08/importing-kubernetes-yaml-in-octopus/index.md).
* [Deploy a Helm chart via Octopus](/blog/2020-08/deploy-helm-chart-with-octopus/index.md).
* [Perform custom Kubernetes scripting in Octopus](/blog/2020-08/custom-kubectl-scripting-in-octopus/index.md).
