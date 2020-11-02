---
title: Deploying a microservice to Kubernetes with Octopus
description: Learn the features available in Octopus to streamline and manage Kubernetes deployments with an example microservices application stack.
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Microservices can be a powerful design pattern to allow large teams of developers to deliver code to production without requiring code to be coordinated in a single code base and released on a common schedule. Deploying these microservices can be a challenge though as the cost of orchestrating Kubernetes resources and promoting between environments is paid by each individual microservice.

Octopus has a number of useful features to help streamline and manage microservice deployments. In this post and screencast we'll run through the process of deploying the sample microservice application created by Google called [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo).

## Screencast

<iframe width="1280" height="720" src="https://www.youtube.com/embed/pJjriVVLWQ0" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Creating the deployment template

If you look at the [YAML containing all the Kubernetes resource definitions](https://github.com/GoogleCloudPlatform/microservices-demo/blob/master/release/kubernetes-manifests.yaml) you will notice a pattern where each deployment is exposed by a matching service. Pairing deployments and services like this is a common pattern in Kubernetes deployments, and this pattern is captured by the **Deploy Kubernetes containers** step in Octopus.

You will also notice that the deployment definitions are largely similar for the majority of the microservices. They all define:

* A container,
* A port the container is exposed on,
* Resource requests and limits,
* GRPC liveness and readiness probes and,
* Environment variables.

The similarities between deployment resources is easy to see using a diff tool:

![](deployment-diff.png "width=500")

To remove the boilerplate code required to define a deployment and its associated service, we take advantage of a feature in Octopus called step templates. The YAML below captures the fields used by most of the microservice applications in their deployments, with application specific values replaced with variables:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: "#{ServiceName}"
spec:
  template:
    metadata:
      labels:
        app: "#{ServiceName}"
    spec:
      terminationGracePeriodSeconds: "#{if ServiceTerminationGracePeriodSeconds}#{ServiceTerminationGracePeriodSeconds}#{/if}"
      containers:
      - name: server
        image: "#{ServiceImage}"
        ports:
        - containerPort: "#{ServicePort}"
        envFrom:
        - secretRef:
            name: "mysecret-#{Octopus.Deployment.Id | ToLower}"
        resources:
          requests:
            cpu: "#{ServiceCPURequest}"
            memory: "#{ServiceMemoryRequest}"
          limits:
            cpu: "#{ServceCPULimit}"
            memory: "#{ServiceMemoryLimit}"
        readinessProbe:
          initialDelaySeconds: "#{if ServiceReadinessProbeInitialDelaySeconds}#{ServiceReadinessProbeInitialDelaySeconds}#{/if}"
          periodSeconds: "#{if ServiceReadinessProbePeriodSeconds}#{ServiceReadinessProbePeriodSeconds}#{/if}"
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:#{ServicePort}", "#{if ServiceReadinessProbeTimeout}-rpc-timeout=#{ServiceReadinessProbeTimeout}s#{/if}"]
        livenessProbe:
          initialDelaySeconds: "#{if ServiceLivenessProbeInitialDelaySeconds}#{ServiceLivenessProbeInitialDelaySeconds}#{/if}"
          periodSeconds: "#{if ServiceLivenessProbePeriodSeconds}#{ServiceLivenessProbePeriodSeconds}#{/if}"
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:#{ServicePort}", "#{if ServiceLivenessProbeTimeout}-rpc-timeout=#{ServiceLivenessProbeTimeout}s#{/if}"]
```

There are a few interesting aspects of this YAML to call out.

The `if` syntax (for example `#{if ServiceTerminationGracePeriodSeconds}#{ServiceTerminationGracePeriodSeconds}#{/if}`) provides a way to return the variable value if it has been defined, or an empty string if the variable has not been defined. Octopus will ignore empty YAML properties where appropriate to avoid validation errors when deploying the final resource.

The list of individual environment variables has been replaced by `envFrom.secretRef`. This allows Kubernetes to inject environment variables based on the values saved in an external secret. The secret we reference here is called `mysecret-#{Octopus.Deployment.Id | ToLower}`, and will be created as a custom resource later in the step.

Next we have the service template. Unlike the deployment template, the service template is the same for all microservices:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: "#{ServiceName}"
spec:
  type: ClusterIP
  selector:
    app: "#{ServiceName}"
  ports:
  - name: grpc
    port: "#{ServicePort}"
    targetPort: "#{ServicePort}"
```

Finally we create the secret that holds the environment variables exposed to the pod.

This resource uses the [Octopus variable repetition syntax](https://octopus.com/docs/projects/variables/variable-substitutions#VariableSubstitutionSyntax-Repetition) to add all matching Octopus variables as fields in the secret resource:

```yaml
apiVersion: v1
data:
#{each var in EnvironmentVariables}
  #{var}: "#{var.Value | ToBase64}"
#{/each}
kind: Secret
metadata:  
  name: mysecret
type: Opaque
```

The variables take the form `GroupName[VariableName].VariableProperty`, for example `EnvironmentVariables[REDIS_ADDR].Value`, `EnvironmentVariables[PORT].Value`, or `EnvironmentVariables[LISTEN_ADDR].Value`.

The variables that make up the deployment are then exposed as parameters:

![](parameters.png "width=500")

The container image name is defined as a package, allowing the image version to be selected during release creation:

![](server-image-parameter.png "width=500")

This parameter is referenced in the container definition with the **Let the project select the package** option for the Docker image field:

![](container-image-parameter.png "width=500")

## Deploying the template

With the template deployed we now create those microservices that originally shared that common deployment pattern. Since all the common boilerplate code has been abstracted away, all that is left is to populate the parameters defined by the step template.

![](deploy-template.png "width=500")

The environment variables are then defined using the group syntax noted above:

![](environment-variables.png "width=500")

For those microservices that don't follow the standard template (for example the frontend app and the redis database) we can simply copy the deployment and service YAML into the **Edit YAML** section, which will populate the UI for us:

![](edit-yaml.png "width=500")

## Promoting an entire environment

