---
title: Getting remote access to a Kubernetes cluster
description: Learn the different ways to gain remote access to a Kubernetes cluster.
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Jump boxes or bastion hosts are a common networking strategy that exposes a single secure entrypoint to the public internet in order to access a private network. This single point of entry allows security teams to closely monitor and control network access to the private network. Often the bastion host exposes a well known remote access service, like RDP or SSH, which teams can assume have been widely vetted and can be trusted.

In this post you'll learn some of the ways to gain remote access to a Kubernetes cluster in order to perform administrative tasks.

## SSH Server

SSH servers have long been used to provide remote access to Linux servers, and it is relatively easy to host an SSH server as a Kubernetes pod.

The YAML file shown below deploys an instance of the `linuxserver/openssh-server` image and exposes it via a loadbalancer service:

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

Note that, for convenience, this SSH server allows password access, and the example YAML file embeds an insecure example password, and allows sudo access. A more robust solution is to use key files for authentication. The [documentation](https://hub.docker.com/r/linuxserver/openssh-server) contains examples showing how to use key files for authentication.

Save the YAML above to a file called `ssh.yaml` and apply it with the command:

```bash
kubectl apply -f ssh.yaml
```

You can then find the IP address or hostname of the loadbalancer service with the command:

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

To do anything useful with the cluster, you will need to download `kubectl` and configure it to access the cluster from within the pod. Download and install `kubectl` with the commands:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

By default, pods have a number of files mounted under `/var/run/secrets/kubernetes.io/serviceaccount` that allow the pod to interact with the host cluster. Then save the following file to `~/.kube.config`:

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

At this point you can run `kubectl` from your SSH session and interact with the parent cluster, providing a convenient and secure environment for cluster administration.

