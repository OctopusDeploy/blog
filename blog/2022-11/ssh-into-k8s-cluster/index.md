---
title: SSH into a Kubernetes cluster
description: Learn how to set up a SSH bastion host in your Kubernetes cluster.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-11-09-1400
metaImage: blogimage-testingkubernetes-2022.png
bannerImage: blogimage-testingkubernetes-2022.png
bannerImageAlt: Kubernetes logo on an open laptop screen
tags:
 - DevOps
 - Kubernetes
 - Docker
---

Jump boxes or bastion hosts are a common networking strategy to expose a single secure entry point to the public internet, to access a private network. This single point of entry lets security teams closely monitor and control network access to the private network. Often the bastion host exposes a well known remote access service, like RDP or SSH, which teams can assume have been widely vetted and are trustworthy.

In this post, you learn how to host an OpenSSH server in a Kubernetes cluster to perform administrative tasks.

## Deploying an SSH Server

SSH servers have long been used to provide remote access to Linux servers, and it's relatively easy to host an SSH server as a Kubernetes pod.

The YAML file shown below creates a service account with a role and role-binding granting access to common resources in the current namespace. It then deploys an instance of the `linuxserver/openssh-server` image, inheriting the permissions of the service account, and exposes it via a load balancer service:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: k8s-admin
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: k8s-admin-role
rules:
- apiGroups: ["", "extensions", "apps", "networking.k8s.io"]
  resources: ["deployments", "replicasets", "pods", "services", "ingresses", "secrets", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get"]  
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: k8s-admin-role-binding
subjects:
- kind: ServiceAccount
  name: k8s-admin
  apiGroup: ""
roleRef:
  kind: Role
  name: k8s-admin-role
  apiGroup: ""
---
apiVersion: v1
kind: Service
metadata:
  name: my-ssh-svc
  labels:
    app: ssh
spec:
  type: LoadBalancer
  ports:
  - port: 2222
  selector:
    app: ssh
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-ssh
  labels:
    app: ssh
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ssh
  template:
    metadata:
      labels:
        app: ssh
    spec:
      serviceAccountName: k8s-admin
      containers:
      - name: ssh
        image: lscr.io/linuxserver/openssh-server:latest
        ports:
        - containerPort: 2222
        env:
        - name: PUID
          value: "1000"
        - name: PGID
          value: "1000"
        - name: TZ
          value: "Australia/Brisbane"
        - name: USER_NAME
          value: "admin"
        - name: USER_PASSWORD
          value: "Password01!"
        - name: PASSWORD_ACCESS
          value: "true"
        - name: SUDO_ACCESS
          value: "true"          
```

Note that, for convenience, this SSH server allows password access, the example YAML file embeds an insecure example password, and allows sudo access. A more robust solution is to use key files for authentication. Refer to the [Docker Hub documentation](https://hub.docker.com/r/linuxserver/openssh-server) for examples showing how to use key files for authentication.

Save the YAML above to a file called `ssh.yaml` and apply it with the command:

```bash
kubectl apply -f ssh.yaml
```

You can then find the IP address or hostname of the load balancer service with the command:

```bash
kubectl get service my-ssh-svc
```

On my local Kubernetes cluster, this command returned:

```bash
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)          AGE
my-ssh-svc   LoadBalancer   10.96.164.169   172.21.255.202   2222:31628/TCP   29m
```

You can then SSH into the external IP address with the command:

```bash
ssh admin@172.21.255.202 -p 2222
```

You then have an interactive session inside the pod on the Kubernetes cluster.

## Installing and configuring kubectl

To do anything useful with the cluster, you need to download `kubectl` and configure it to access the cluster from within the pod. Download and install `kubectl` with the commands:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

By default, pods have a number of files mounted under `/var/run/secrets/kubernetes.io/serviceaccount` that let the pod interact with the host cluster. To configure `kubectl` to use these files, save the following file to `~/.kube.config`:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    server: https://kubernetes.default
  name: localk8s
contexts:
- context:
    cluster: localk8s
    user: user
  name: localk8s
current-context: localk8s
kind: Config
preferences: {}
users:
- name: user
  user:
    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
```

You can now run `kubectl` from your SSH session and interact with the parent cluster, providing a convenient and secure environment for cluster administration.

## Building a custom OpenSSH Docker image

Downloading `kubectl` and copying the configuration file is easy enough, but the ephemeral nature of Kubernetes pods means eventually the container will be deleted and recreated, forcing you to download and configure `kubectl` again.

A better solution is baking `kubectl` and its configuration file into a custom Docker image. This ensures the files are available in the container when it's first started.

Save the following to a file called `Dockerfile`:

```Dockerfile
FROM lscr.io/linuxserver/openssh-server:latest
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
	install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
RUN printf 'apiVersion: v1\n\
clusters:\n\
- cluster:\n\
    certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt\n\
    server: https://kubernetes.default\n\
  name: localk8s\n\
contexts:\n\
- context:\n\
    cluster: localk8s\n\
    user: user\n\
  name: localk8s\n\
current-context: localk8s\n\
kind: Config\n\
preferences: {}\n\
users:\n\
- name: user\n\
  user:\n\
    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token' >> /opt/kubeconfig
RUN printf 'mkdir /config/.kube \n\
cp /opt/kubeconfig /config/.kube/config' >> /etc/cont-init.d/100-kubeconfig
```

You then build the custom Docker image with the following command, where `yourdockerregistry` is replaced with the name of a Docker registry you have the ability to push images to:

```bash
docker build . -t yourdockerregistry/openssh-server:latest
```

Replace the `image` property in the Kubernetes YAML file with:

```
image: yourdockerregistry/openssh-server:latest
```

After the new SSH server pods are created using your custom image, `kubectl` and its configuration file are ready to use without first downloading them.

## Conclusion

A bastion host running OpenSSH on your Kubernetes cluster provides you with a single, secure entry point for administration and debugging tasks. By customizing the Docker image to include common tools like `kubectl`, DevOps teams can rely on the bastion host having any required tools for common administration tasks.

Happy deployments!