---
title: Using local images with minikube
description: Learn how to deploy locally built Docker images to minikube.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-11-21-1400
metaImage: blogimage-microservicesframeworks-2022.jpg
bannerImage: blogimage-microservicesframeworks-2022.jpg
bannerImageAlt: 3 people building an unstable looking tower with blue blocks, beside 2 people building a stable, lower tower with blue blocks.
tags:
 - DevOps
 - Kubernetes
 - Docker
---

[minikube](https://minikube.sigs.k8s.io/docs/start/) provides DevOps teams with a local development Kubernetes cluster. Developing Kubernetes applications locally often entails building and deploying local Docker images. While minikube will download any Docker images hosted on an external Docker registry, exposing locally built images requires loading the images into the minikube cluster and being aware of some edge cases that throw unhelpful error messages. 

In this post, I show you how to deploy locally built Docker images to minikube.

## Building the Docker images

The [Octopus underwater sample app](https://github.com/OctopusSamples/octopus-underwater-app) provides a simple Docker image for testing. Run the following command to clone the git repo:

```bash
git clone https://github.com/OctopusSamples/octopus-underwater-app.git
```

Enter the project directory:

```bash
cd octopus-underwater-app
```

Then build the image with the command:

```bash
docker build . -t underwater
```

Finally, run the Docker image with the command:

```bash
docker run -p 5000:80 underwater
```

The sample web app is then available at `http://localhost:5000`.

## Pushing images to minikube

Pushing local images to minikube is a straightforward process with the command:

```bash
minikube image load underwater
```

## Deploying the image

To deploy the image, save the following YAML to a file called `underwater.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: underwater
  labels:
    app: web
spec:
  selector:
    matchLabels:
      app: web
  replicas: 1
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: underwater
          image: underwater
          imagePullPolicy: Never
          ports:
            - containerPort: 80
```

Then deploy the app with the command:

```bash
kubectl apply -f underwater.yaml
```

The application is then successfully deployed to the minikube cluster.

It's important to note that the Docker image has no tag, which means it has the default tag of `latest`. So the `image` property in the YAML above could be replaced with the following text, as the 2 image references are equivalent:

```
image: underwater:latest
```

Using images with the `latest` tag has special implications which require setting the `imagePullPolicy` to `Never` (or `IfNotPresent`). To understand why, you need to understand the default image pull policy.

## Using latest images

The Kubernetes documentation provides this advice on the [default image pull policy](https://kubernetes.io/docs/concepts/containers/images/#imagepullpolicy-defaulting):

> * if you omit the imagePullPolicy field, and the tag for the container image is :latest, imagePullPolicy is automatically set to Always;
> * if you omit the imagePullPolicy field, and you don't specify the tag for the container image, imagePullPolicy is automatically set to Always;
> * if you omit the imagePullPolicy field, and you specify the tag for the container image that isn't :latest, the imagePullPolicy is automatically set to IfNotPresent.

To understand why this is important, deploy the following YAML with no `imagePullPolicy` value set:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: underwater
  labels:
    app: web
spec:
  selector:
    matchLabels:
      app: web
  replicas: 1
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: underwater
          image: underwater
          ports:
            - containerPort: 80
```

View the status of the pods created by this deployment with the command:

```bash
kubectl get pods
```

You see that the status is `ImagePullBackOff`:

```bash
NAME                          READY   STATUS             RESTARTS   AGE
underwater-847d6f9646-pvzxb   0/1     ImagePullBackOff   0          15m
```

This is because you deployed an image with the `latest` tag and didn't specify the `imagePullPolicy`, meaning the default value of `Always` is used. This in turn means minikube attempts to download the image `docker.io/underwater:latest`, because images with no registry in their name default to `docker.io`. The image `docker.io/underwater:latest` does not exist, hence the `ImagePullBackOff` error.

There are 2 ways around this:

- Set `imagePullPolicy` to `Never` or `IfNotPresent`
- Add a tag to the image, for example, `docker build . -t underwaterapp:0.0.1` and `minikube image load underwater:0.0.1`

## Conclusion

Using locally built Docker images in minikube is an easy process, but you need to be aware of the rules surrounding the image pull policy to ensure Kubernetes doesn't attempt to download a non-existent image from the default Docker registry.

Happy deployments!