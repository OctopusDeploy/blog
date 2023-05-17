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

In this blog post we'll show how to use YAML manifests from Git in Octopus for Kubernetes deployments. It's an easy bit powerful tool enabling complex scenarious like configuration templating.

## Body

Octopus suggests two strategies to configure your Kubernetes deployments. You can create the configuration in Octopus using our buil-in steps like `Deploy Kubernetes containers`. This approach allows you to start fast and simple and evolve your configuration in Octopus leveraging our UI.

An alternative approach is to create end evolve your configuration as YAML code from the beginning. This blog post will focus on this method.

Till recently there were two ways to deploy YAML in Octopus. You could paste code in Octopus (in a scipt or the `Deploy raw Kubernetes YAML` step), or you could provide code with a package.

We've just added a new method — referencing files from a Git repo. This is the newest addition and now the default option for the `Deploy raw Kubernetes YAML` step.

### What it does

Despite the obvious benefit of sourcing files without the need to package them, the new functionality introduces a few more imprivements

* You can reference multiple files in one step. No need to run multiple steps or conbine everything in one YAML file.
* You can reference folders or use glob to define multiple files (in this case, they will be applied all at once in the alphabetical order).
* If you need to define a specific order, you can define multiple paths.

The points above unlock scenarious like deploying multiple services within one step. Let's say you might want tp deploy all Secrets and ConfigMaps first (i.e. `/configuration/*-secret*.yaml` and `/configuration/*-configmap.yaml` or `/configuration/secrets-and-configs/*`). Deploy your first service after that (e.g. `/configuration/db-*.yaml`) and the second service after that (e.g. `/configuration/web-*.yaml`). You might notice that a file like `/configuration-db-configmap.yaml` will be reference twice, it's not neat but the deployment will totally work anyway (just the second deployment won't cause any changes).

There is also nothing wrong in using the same files multiple times in different steps. In this case, you can consider the files as templates and modify them with [Octopus variables](https://octopus.com/docs/projects/variables) embeded in YAML.

You can also use [structured configuration variables](https://octopus.com/blog/structured-variables-raw-kubernetes-yaml) if you don't want to change your YAML files, so you can still use them for deployments outside of Octopus.

Finally, you can also use Octopus variables in the paths or repository links, making really powerful step templates.

### Example

Imagine we have a complex app consists of 100 microservices. We combined these microservices in three groups similar enuogh, so we can define standard YAML files for each group. At the same time, we still want to modify some parameters for each app (like the image name).

We also want our software teams to easily configure new deployment without diving deep in YAML configurations.

In this case, we can create three new repos with configuratio files, one for each group of microservices. We could store all the ocnfiguration files in one repo, and it will work as well, but just for the sake of the example, let's say we have three.

We can create a new step template from the `Deploy raw Kubernetes YAML` and specify git repo path with the `Octopus.ProjectGroup.Name`. This allows us to use project group name as a reference. 

After that we can specify multiple paths to the YAML templates (if we keep the structure in all three template repos same).

We can use variables like `Octopus.Project.Name` in combination with `Octopus.Release.Number` to modify YAML fils defining container images, namespaces and other configuration parameters which don't require fine tuning.

Finally, we can expose (i.e. allow to modify when using the step template) variables like number of replica sets or container ports.

In this scenario, a new app deployment configuration would be as simple as creating a new project, adding a step template and specifying a few variables.

### How it works










The body of the post is where you share your hypothesis, how-to, or story.

If there are any previous posts on the same topic, please link to them to help with our SEO efforts. For example:
Our post about [DORA metrics](https://octopus.com/blog/dora-metrics-devops-business-outcomes) discusses how agility-based metrics can help improve profitability, market share, and productivity. 

### Subheadings

Use three ### to include H3 headings.

Use **Bold** text for UI labels, use single back-tics for `parameters` and `filepaths`, and three back-tics for code blocks:

```
Write-Host "Hello, World!"
```

Use the following (minus the backtics) to include images:

```
![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*
```
If including images, please include alt text. Alt text is primarily used to describe images to people unable to see them, and can be 125 characters max including spaces. You can also include an image caption if the reader would benefit from additional information or context.

## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)

## Register for the webinar: {webinar title here}

Short webinar description here, for example: A robust rollback strategy is key to any deployment strategy. In this webinar, we’ll cover best practices for IIS deployments, Tomcat, and full stack applications with a database. We’ll also discuss how to get the rollback strategy right for your situation. 

We're running 3 sessions of the webinar, from {webinar dates here, for example: 4 November to 5 November, 2021.}

<span><a class="btn btn-success" href="/events/rollback-strategies-with-octopus-deploy">Register now</a></span>

## Watch the webinar: {webinar title here}

<iframe width="560" height="315" src="https://www.youtube.com/embed/F_V7r80aDbo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

We host webinars regularly. See the [webinars page](https://octopus.com/events) for details about upcoming events, and live stream recordings.

Happy deployments!
