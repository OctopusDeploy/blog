---
title: Sourcing Kubernetes manifests from Git
description: You can now reference YAML configurations from your Git repository in the "Deploy raw Kubernetes YAML" step. Say goodbye to building packages and copying and pasting code.
author: nikita.dergilev@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Product
  - Containers
  - Kubernetes
---

See https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags,

In this blog post we'll show how to use YAML manifests from Git in Octopus for Kubernetes deployments. It's an simple but powerful tool enabling complex scenarioes like configuration templating.

## Body

Octopus suggests two strategies to configure your Kubernetes deployments. First, you can create the configuration in Octopus using our built-in steps like `Deploy Kubernetes containers`. This approach allows you to start fast and evolve your configuration in Octopus, leveraging our UI.

An alternative approach is to create and evolve your configuration as YAML code. This blog post will focus on this method.

Until recently, there were also two ways to deploy YAML in Octopus. You could paste code in Octopus (in a script or the `Deploy raw Kubernetes YAML` step) or provide code with a package.

We've added a new method — referencing files from a Git repo. This is the latest addition and now the default option for the `Deploy raw Kubernetes YAML` step.

![New option on the step configuration to fetch files from Git repos](/blog/2023-05/manifests-from-git/git-manifest-enable.png "width=500")

### What it does

Despite the clear benefit of sourcing files without the need to package them, the new functionality introduces a few more improvements.

* You can reference many files in one step. No need to run multiple steps or combine everything in one YAML file.
* You can use glob patterns to define multiple files (in this case, they will be applied all at once in alphabetical order).
* You can define multiple paths if you need to define a specific order.

The points above unlock scenarios like deploying many apps within one step. Let's say you might want to

* deploy all Secrets and ConfigMaps first (i.e. `/configuration/*-secret*.yaml` and `/configuration/*-configmap.yaml` or `/configuration/secrets-and-configs/*`),

* deploy your first app after that (e.g. `/configuration/db-*.yaml`),

* and the second app at the end (e.g. `/configuration/web-*.yaml`).

You might notice that a file like `/configuration-db-configmap.yaml` would be referenced twice. It's not neat, but the deployment will work anyway (unless it includes jobs, the second deployment typically wont have any effect).

There is also nothing wrong with using the same files multiple times in different steps. In this case, you can consider the files templates and change them with [Octopus variables](https://octopus.com/docs/projects/variables) embedded in YAML.

You can also use [structured configuration variables](https://octopus.com/blog/structured-variables-raw-kubernetes-yaml) if you don't want to change your YAML files, so you can still use them for deployments outside of Octopus.

Finally, you can use Octopus variables in the paths or repository links, making powerful step templates.

### Example

Imagine we have a complex app consisting of 100 microservices. We combined these microservices in three similar groups so that we could define YAML files for each group. At the same time, we still want to change some parameters for each app (like the image name).

We also want our software teams to quickly configure new deployments without diving deep into YAML configurations.

In this case, we can create three new repos with configuration files. One for each group of microservices. Storing all the configuration files in one repo will also work, but for the sake of the example, let's say we have three.

We can create a new step template from the `Deploy raw Kubernetes YAML` and specify a git repo path with the `Octopus.ProjectGroup.Name`. This allows us to use the project group name as a reference.

![Adding variable to the Git path row](/blog/2023-05/manifests-from-git/git-manifest-paths.png "width=500")

After that, we can specify multiple paths to the YAML templates (if we keep the structure in all three template repos the same).

We can use variables like `Octopus.Project.Name` in combination with `Octopus.Release.Number` to modify YAML files. For example, we can change container images, namespaces and other configuration parameters. In this case, software engineers won't have to specify these values for their deployment.

Finally, we can expose (i.e. allow to modify when using the step template) variables like the number of replica sets or container ports.

![Variables exposed via step templates](/blog/2023-05/manifests-from-git/git-manifest-step-template.png "width=500")

In this scenario, a new app deployment configuration would be as simple as creating a new project, adding a step template and specifying a few variables.

### How to configure

1. Open the process editor and find a `Deploy raw Kubernetes YAML` step you want to modify (or add a new one).
2. Choose the option to source YAML manifests from Git (the default option for newly added steps).
3. Choose Git credentials (or add new ones to the library following the link).
4. Specify a Git repo URI. You can use a variable in this row; it might be handy to create a step template. You cannot scope this variable per environment; it should be one value per project.
5. Specify the files you want to use for the deployment (provide path).
6. Save the process configuration.

Now you can create a release and deploy it. Octopus will clone your Git repo, find the files you specified and deploy them.

Variables replacement, including structured configuration variables, will work as usual. Therefore, you can use Octopus variables in your YAML files if needed.

#### How to configure file paths

You can configure multiple paths. Paths should be separated by `;`.

Each path can point to one or many files.

* If you want to point to one file, type the path to it, including folder structure. E.g. `configs/yaml/deployment.yaml`
* If you want to fetch a several files with one path, you can use glob patterns. E.g. `configs/yaml/appname-*.yaml`. In this case, if there are multiple files like `appname-service.yaml` and `appname-deployment.yaml` in this folder, Octopus will apply them but will ignore files like `someoneselseapp-deployment.yaml`.

![Multiple paths example](/blog/2023-05/manifests-from-git/git-manifest-paths.png "width=500")

You can use glob to specify a folder, e.g. `configs/yaml/*`

#### How Octopus will apply files from the paths

Octopus applies all files from one path at once using `kubectl apply`. The files will be applied by kubectl itself in alphabetical order. Therefore, if you want to enforce a specific application order within one path, you need to name the files accordingly.

Octopus applies files from different paths in sequence using multiple `kubectl apply` commands. Octopus doesn't wait for the cluster to implement an applied configuration before executing the following `kubectl apply` command. However, it waits for the `kubectl` command to complete.

⚠️ You can use paths to enforce a particular order of files application.

You can also use variables in the path row and specify multiple files in a variable.

#### When Octopus will fetch files from Git

⚠️ Octopus will record the commit hash specified during release creation (this may be the tip of a branch or label), and during deployment will fetch files from the provided repo. This means that if you add new files to Git before creating a release but after configuring the step, Octopus will still use those files for deployments (if they meet the path configuration). But if you add or update files to the same branch after a release is created, Octopus won't deploy these files with that release.

### What you see on the release page

Octopus shows you the list of saved release files.


```
![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*
```
If including images, please include alt text. Alt text is primarily used to describe images to people unable to see them, and can be 125 characters max including spaces. You can also include an image caption if the reader would benefit from additional information or context.

## Conclusion

Sourcing files from Git enable Kubernetes configuration templating at a new level. Give it a try and share your thoughts with us.

## Learn more

- [link](https://www.example.com/resource)
