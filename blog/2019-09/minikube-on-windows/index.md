---
title: Installing Minikube on Windows
description: Learn how to get a test Kubernetes environment on Windows with Minikube
author: matthew.casperson@octopus.com
visibility: public
published: 2019-09-25
metaImage: installing_minikube_on_windows.png
bannerImage: installing_minikube_on_windows.png
bannerImageAlt: Octopus driving a Kubernetes speedboat
tags:
 - Engineering
 - Kubernetes
---

![Octopus driving a Kubernetes speedboat](installing_minikube_on_windows.png)

Getting started with a test Kubernetes cluster has been made easier thanks to the [Minikube project](https://kubernetes.io/docs/tasks/tools/install-minikube/). By using the HyperV functionality in Windows 10, a test Kubernetes cluster can be created in just a few minutes.

In this post, we’ll run through the process of configuring HyperV, installing `kubectl` and Minikube, and interacting with the test Kubernetes cluster.

## Create an external switch

Minikube requires an external HyperV switch to operate, and you may find that you don’t have one by default.

To view the list of switches, open the `HyperV Manager` and select `Virtual Switch Manager...` from the list of actions:

![](hyperv-actions-01.png "width=300")

*The HyperV actions menu.*

In the screenshot below, you can see there are no existing external switches, so we need to create one by selecting the `External` option and clicking the `Create Virtual Switch` button:

![](create-virtual-switch.png "width=500")

*The list of existing switches.*

Connect the virtual switch to the PCs local network adapter (in this example, the wireless network adapter). Click the `OK` button to create the switch:

![](vswitch.png "width=500")

*Creating a new HyperV external switch.*

You will receive a warning that the network connection will be disrupted. Click `Yes` to continue.

![](warning-01.png "width=300")

*A warning about network interruption as a result of creating a new switch.*

## Install kubectl

The command-line tool we use to interact with Kubernetes is called `kubectl`, which we will install before we configure Minikube. `kubectl` can be installed in two ways:

* With [Chocolatey](https://chocolatey.org/packages/kubernetes-cli) using the command `choco install kubernetes-cli`.
* Manually downloaded from [https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe](https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe).

:::hint
Note that the version of the latest download can be found at [https://storage.googleapis.com/kubernetes-release/release/stable.txt](https://storage.googleapis.com/kubernetes-release/release/stable.txt). Simply replace the `v1.15.0` in the first URL with the version returned by the second URL.
:::

## Install Minikube

We are now ready to install Minikube. You can install Minikube in two ways:

* With [Chocolatey](https://chocolatey.org/packages/Minikube) using the command `choco install minikube`.
* Downloading and installing the executable from [GitHub](https://github.com/kubernetes/minikube/releases/latest/download/minikube-installer.exe).

With Minikube installed, open up a PowerShell terminal as an Administrator:

![](run-as-administrator.png "width=500")

*Launching the terminal as administrator.*

Minikube can then be started with the command:

```PowerShell
minikube start `
--vm-driver hyperv `
--hyperv-virtual-switch "External Switch"
```

A new virtual machine called `minikube` will be created:

![](minikube-vm.png "width=500")

*The Minikube VM created as a result of running Minikube start.*

After the installation has completed, we can interact with the Minikube cluster via `kubectl`:

![](minikube-start.png "width=500")

*The Minikube cluster has been started successfully.*

## Running some commands

The Minikube installation will update the file at `~/.kube/config`, the configuration file used by `kubectl`, with the details of the test cluster:

```PowerShell
PS C:\Users\Matthew> cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: C:\Users\Matthew\.minikube\ca.crt
    server: https://10.1.1.122:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: C:\Users\Matthew\.minikube\client.crt
    client-key: C:\Users\Matthew\.minikube\client.key
```

We can test the installation is working by listing the nodes:

```PowerShell
PS C:\Users\Matthew> kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   6m47s   v1.15.2
```

## Connecting Octopus to Minikube

To use Minikube from Octopus, we need to upload the client certificate. Minikube splits the certificate and key into two files (referenced by the `client-certificate` and `client-key` properties in the `~/.kube/config` file), so we use [OpenSSL](https://slproweb.com/products/Win32OpenSSL.html) to merge them into a single PFX file:

```PowerShell
& "C:\Program Files\OpenSSL-Win64\bin\openssl.exe" pkcs12 `
  -passout pass: `
  -export `
  -out certificateandkey.pfx `
  -in C:\Users\Matthew\.minikube\client.crt `
  -inkey C:\Users\Matthew\.minikube\client.key
```

The resulting `certificateandkey.pfx` file can then be uploaded to Octopus:

![](certificate.png "width=500")

*The certificate and key that was created by Minikube and combined by OpenSSL.*

To allow Octopus to communicate with the local virtual machine, we need to create a [Worker Tentacle](https://octopus.com/docs/infrastructure/workers) on our local PC:

![](worker.png "width=500")

*A polling Worker Tentacle running on the same machine as the Minikube cluster.*

We can now create a Kubernetes target pointing to the local URL of the Minikube cluster, which can be found from this line in the `~/.kube/config` file:

```YAML
server: https://10.1.1.122:8443
```

The Kubernetes target will use the certificate we uploaded earlier for authentication, skip TLS verification for convenience, and use the local Tentacle Worker by default.

:::hint
You could upload the certificate reference by the `certificate-authority` property in the `~/.kube/config` file and set that as the server certificate if you wanted to.
:::

![](k8s-target.png "width=500")

*A Kubernetes target, configured to use the local Worker Tentacle, talking to the internal Minikube IP address.*

With the target defined, we can use the [Octopus Script Console](https://octopus.com/docs/administration/managing-infrastructure/script-console) to interact with it:

![](script-console.png "width=500")

*The script console is a handy way to run ad-hoc scripts.*

The command will be executed via the Worker Tentacle to interact with the local Minikube cluster:

![](script-result.png "width=500")

*The script result, which mirrors the result when running kubectl locally.*

## Conclusion

Minikube is an easy way to get a test Kubernetes cluster up and running. In Windows, Minikube utilizes HyperV, and requires an external switch to operate. Once started, Minikube configures `kubectl`, and we can start running commands against the test cluster.

It is also possible to interact with the Minikube cluster from Octopus. By using a Worker Tentacle on the same PC as the Minikube VM, a Kubernetes target can issue commands to the private IP of the cluster.

For more information on deploying to Kubernetes, please see [our documentation](https://octopus.com/docs/deployment-examples/kubernetes-deployments).
