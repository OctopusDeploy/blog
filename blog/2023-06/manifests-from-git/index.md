---
title: Sourcing Kubernetes manifests from Git
description: You can now reference YAML configurations from your Git repository in the "Deploy raw Kubernetes YAML" step. Say goodbye to building packages and copying and pasting code.
author: nikita.dergilev@octopus.com
visibility: private
published: 2023-06-26-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Product
  - Containers
  - Kubernetes
---

There are a few ways to configure Kubernetes, but the de-facto default option we think about mentioning Kubernetes is the declarative configuration with YAML manifests. Another de-facto standard is storing configurations in Git. It's a natural choice for engineers who used to evolve and audit all other code this way.

Till now, you cannot source configuration files directly from Git; an interim step was required – packaging files and sending them to Octopus. This approach has a few advantages, like no dependency on Git at the moment of deployment or easy release management. And yet it didn't feel like the most native way to deploy to Kubernetes.

That's why we introduce an improvement allowing you to source files directly from Git. We managed to preserve the benefits of the package approach, so you won't have to give up on release immutability.

Having said that, we don't see the new way to source YAML manifests as just an alternative to packages. It enables more advanced scenarios. In this blog post, we'll illustrate one of them — configuration templating. In this scenario, you can reference the same set of YAML files in multiple projects and modify them with Octopus variables on the project level. It works well in case you manage multiple similar services and aim to save time and introduce standards in how they should be configured.

## Templating Kubernetes configuration when sourcing YAML files from GIt

Referencing files from a Git repo is the latest addition and the new default option for the `Deploy raw Kubernetes YAML` step.

![New option on the step configuration to fetch files from Git repos](/blog/2023-06/manifests-from-git/git-manifest-enable.png "width=500")

### What the new feature does

Despite the clear benefit of sourcing files without the need to package them, the new functionality introduces a few more improvements.

* You can reference many files in one step. No need to run multiple steps or combine everything in one YAML file.
* You can use glob patterns to define multiple files (in this case, they will be applied all at once in alphabetical order).
* You can define multiple paths if you need to define a specific order.

The points above unlock scenarios like deploying many apps within one step. Let's say you might want to

* deploy all Secrets and ConfigMaps first (i.e. `/configuration/*-secret*.yaml` and `/configuration/*-configmap.yaml` or `/configuration/secrets-and-configs/*`),

* deploy your first app after that (e.g. `/configuration/db-*.yaml`),

* and the second app at the end (e.g. `/configuration/web-*.yaml`).

You might notice that a file like `/configuration-db-configmap.yaml` would be referenced twice. It's not neat, but the deployment will work anyway (unless it includes jobs, the second deployment typically wont have any effect).

There is also nothing wrong with using the same files multiple times in different steps. In this case, you can consider the file templates and change them with [Octopus variables](https://octopus.com/docs/projects/variables) embedded in YAML.

You can also use [structured configuration variables](https://octopus.com/blog/structured-variables-raw-kubernetes-yaml) if you don't want to change your YAML files, so you can still use them for deployments outside of Octopus.

Finally, you can use Octopus variables in the paths or repository links, making powerful step templates.

### How to create a template

Imagine you're a member of a platform team owning Kubernetes clusters and deployment pipelines. Your company develops a complex app consisting of 100 microservices. You combined these microservices in three similar groups so that you could define YAML templates for each group. You still need to change some parameters for each app (like the image name).

In this case, you can create three new repos with configuration files. One for each group of microservices. Storing all the configuration files in one repo will also work, but for the sake of the example, let's say you have three.

There are two ways of modifying configuration files with Octopus. The most straightforward one is to use Octopus variables. In this case, you need to replace values you want to change in your configuration files with variables like `#{containerImage}` or `#{appLabel}`. You can combine variables; for example, the value for `spec.template.spec.containers[0].image` can look like `#{containerImage}:#{containerTag}`. You can even use [extended syntax](https://octopus.com/docs/projects/variables/variable-substitutions#VariableSubstitutionSyntax-ExtendedSyntax) to cater for advanced scenarios.

Files with Octopus variables are easy to inspect, and you can achieve tremendous flexibility with variables. However, sometimes you might need to have valid YAML in your repository. For example, if you need to test the configuration locally. In this case, you can take another step and use [structured variable replacement](https://octopus.com/blog/structured-variables-raw-kubernetes-yaml).

Despite the option you choose, you can configure some variables for all projects using [library sets](https://octopus.com/docs/projects/variables/library-variable-sets). It's handy if all your apps, for example, should have a certain number of replicas in a given environment. Project-specific variables can be configured at the project level.

### Hiding complexity

You might want to hide Kubernetes complexity from your software teams by exposing a limited number of properties a team should change to make the template work. We can achieve this by creating a step template. With step templates, there is no need for software teams to learn YAML, know where templates are stored  or even fill in all the variables when creating a project.

You can create a new step template from the `Deploy raw Kubernetes YAML`, and use variables like `Octopus.Project.Name`, `Octopus.Release.Number`, and many other [system variables](https://octopus.com/docs/projects/variables/system-variables) so software teams won't have to fill in these values. For example, your `spec.template.spec.containers[0].image` can actually look like `#{Octopus.Project.Name}:#{Octopus.Release.Number}`. 

Finally, you can expose (i.e. allow to modify when using the step template) variables like the number of replica sets or container ports.

![Variables exposed via step templates](/blog/2023-06/manifests-from-git/git-manifest-step-template.png "width=500")

In this scenario, a new app deployment configuration would be as simple as creating a new project, adding a step template and specifying a few variables.

### How to configure

Now you just need to configure the deployment step.

1. Open the process editor and find a `Deploy raw Kubernetes YAML` step you want to modify (or add a new one).
2. Choose the option to source YAML manifests from Git (the default option for newly added steps).
3. Choose Git credentials (or add new ones to the library following the link).
4. Specify a Git repo URI. You can use a variable in this row (e.g. `Octopus.ProjectGroup.Name`). You cannot scope this variable per environment; it should be one value per project.
6. Specify the files you want to use for the deployment (provide path).
7. Save the process configuration.

![Adding variable to the Git path row](/blog/2023-06/manifests-from-git/git-manifest-paths.png "width=500")

Now you can create a release and deploy it. Octopus will clone your Git repo, find the files you specified and deploy them.

Variables replacement, including structured configuration variables, will work as usual. Therefore, you can use Octopus variables in your YAML files if needed.

#### How to configure file paths

You can configure multiple paths. Paths should be separated by `;`.

Each path can point to one or many files.

* If you want to point to one file, type the path to it, including folder structure. E.g. `configs/yaml/deployment.yaml`
* If you want to fetch a several files with one path, you can use glob patterns. E.g. `configs/yaml/appname-*.yaml`. In this case, if there are multiple files like `appname-service.yaml` and `appname-deployment.yaml` in this folder, Octopus will apply them but will ignore files like `someoneselseapp-deployment.yaml`.

![Multiple paths example](/blog/2023-06/manifests-from-git/git-manifest-paths.png "width=500")

You can use glob to specify a folder, e.g. `configs/yaml/*`

#### How Octopus will apply files from the paths

Octopus applies all files from one path at once using `kubectl apply`. The files will be applied by kubectl itself in alphabetical order. Therefore, if you want to enforce a specific application order within one path, you need to name the files accordingly.

Octopus applies files from different paths in sequence using multiple `kubectl apply` commands. Octopus doesn't wait for the cluster to implement an applied configuration before executing the following `kubectl apply` command. However, it waits for the `kubectl` command to complete.

⚠️ You can use paths to enforce a particular order of files application.

You can also use variables in the path row and specify multiple files in a variable.

#### When Octopus will fetch files from Git

⚠️ Octopus will record the commit hash specified during release creation (this may be the tip of a branch or label), and during deployment will fetch files from the provided repo. This means that if you add new files to Git before creating a release but after configuring the step, Octopus will still use those files for deployments (if they meet the path configuration). But if you add or update files to the same branch after a release is created, Octopus won't deploy these files with that release.

### What you see on the release page

When you create a release, Octopus shows you the list of saved release files and Git references.

![Release page with files sourced from Git](/blog/2023-06/manifests-from-git/git-manifest-release-page.png "width=500")

## Conclusion

Sourcing files from Git enable Kubernetes configuration templating. You can make Kubernetes deployments easy for people in your company and maintain standardisation at the same time.

There are many other ways to use this feature. We're looking forward to hearing your thoughts so we can improve it further. Feel free to leave your feedback in the feedback form you find when you configure the `Deploy raw Kubernetes YAML` step.

## Learn more

- [A LINK TO DOCS SHOULD BE HERE](https://www.example.com/resource)
