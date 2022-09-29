---
title: Bulk deletion of Kubernetes resources
description: Learn how to delete Kubernetes resources like pods in bulk.
author: matthew.casperson@octopus.com
visibility: public
published: 3020-01-01-1400
metaImage: blogimage-testingkubernetes-2022.png
bannerImage: blogimage-testingkubernetes-2022.png
bannerImageAlt: Kubernetes logo on an open laptop screen
isFeatured: false
tags: 
  - Kubernetes
---

Kubernetes makes it easy to create many resources at once, with the `kubectl apply -f filename.yaml` command creating all the resources in a compound YAML file. But how do you then delete multiple resources without specifying them individually? In this post you'll learn how to perform bulk deletions of Kubernetes resources.

## Example deployment

Let's take a look at a typical YAML file describing a Kubernetes deployment and service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

When this YAML is saved to a file called `nginx.yaml`, the resources are created with the following command:

```bash
kubectl apply -f nginx.yaml
```

You can then view the new resources created with the following commands:

```bash
kubectl get pods
kubectl get deployments
kubectl get services
```

You will see three pods, one deployment, and one service are created. The pods are not directly defined in the YAML file and are created by the deployment, with three pods created due to the `replicas` property being set to `3`.

## Deleting resources from file

The easiest way to delete these resources is to use the `delete` command and pass the same file that was used to initially create the resources:

```bash
kubectl delete -f nginx.yaml
```

If you rerun the `kubectl get` commands above you'll see the pods, deployment, and service are deleted. Because the pods are managed by the deployment, deleting the deployment also deletes the pods.

## Manually deleting resources

To manually delete specific types of resources, the `kubectl delete` command accepts an `--all` argument that defines the type of resource to delete. For example, the following command deletes all the services:

```bash
kubectl delete --all services
```

You can confirm the services are deleted with the command:

```bash
kubectl get services
```

This command deletes all the pods:

```bash
kubectl delete --all pods
```

The output of the command will look something like this:

```bash
pod "my-nginx-6595874d85-88jlr" deleted
pod "my-nginx-6595874d85-9w52c" deleted
pod "my-nginx-6595874d85-dpzds" deleted
```

However, something interesting happens when you confirm the pods are deleted. Run the following command to list any pods:

```bash
kubectl get pods
```

You will notice is that there are still 3 pods, with the output looking something like this:

```bash
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-6595874d85-2j4g8   1/1     Running   0          76s
my-nginx-6595874d85-4vrfb   1/1     Running   0          76s
my-nginx-6595874d85-4wj9p   1/1     Running   0          76s
```

If you look closely, the pod names shown by the `kubectl get pods` command are different to those returned by the `kubectl delete --all pods` command. This is because the pods are managed by the deployment, and when the deployment sees that the pods it was managing have been deleted, it recreates new pods to fulfil its `replica` count.

Deleting pods managed by a deployment essentially recreates them, which is useful if you want to force the pods to restart. But the only way to permanently delete the pods is to delete their parent deployment. This is done with the command:

```bash
kubectl delete --all deployments
```

Once the deployment is deleted, there will be no deployments or pods.

## Deleting namespaces

Namespaces are a convenient way to group related resources. Create a new namespace called `foo` with the command:

```bash
kubectl create namespace foo
```

Then create the NGINX resources in the new namespace with the command:

```
kubectl apply -f nginx.yaml -n foo
```

List the resources with the commands:

```bash
kubectl get pods -n foo
kubectl get deployments -n foo
kubectl get services -n foo
```

Then delete the namespace with the command:

```bash
kubectl delete namespace foo
```

This results in the namespace, and all the resources contained in it, being deleted.

## Shorthand "all" resource

You can pass `all` for the resource type when calling `kubectl` to reference a common subset of Kubernetes resource types. So the following command will delete the service, deployment, and pods:

```bash
kubectl delete all --all
```

The `all` type includes:

* pod
* service
* daemonset
* deployment
* replicaset
* statefulset
* job
* cronjobs

## Deleting resources matching a label

Labels are used to enrich resources with metadata often describing things like the resource's purpose, environment, version etc. You can select resources based on these labels in order to delete them. This allows you to selectively delete groups of resources. The following command deletes deployments with the label called `app` set to `nginx`:

```bash
kubectl delete deployments -l app=nginx
```

Likewise you can delete the services with the same label:

```bash
kubectl delete service -l app=nginx
```

## Dry runs

Bulk deletion of resources is convenient, but dangerous. Fortunately, `kubectl` has the `--dry-run` argument that allows you to see the resources that would be deleted, but without actually deleting them. The following command previews the resources that would be matched by the `all` resource type:

```bash
kubectl delete all --all --dry-run
```

## Conclusion

Bulk deleting resources is easy with `kubectl`, and in this post you learned how to delete resources:

* Defined in a YAML file
* Matching a single resource type
* Grouped in the `all` resource type
* Contained in a namespace
* With matching labels

You also learned how to use the `--dry-run` argument to preview any resources that would be deleted.

Happy deployments!