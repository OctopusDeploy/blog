---
title: What you should consider before building CD for Kubernetes
description: There is more than one approach to deploy to Kubernetes; some decisions made in the beginning will be hard to change later. This post concerns a few things one should consider before configuring CD.
author: nikita.dergilev@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
---

Kubernetes is a powerful but complicated platform. One has to make multiple decisions before moving to Kubernetes. For this blog post, I will keep architectural  choices like self-hosted/managed or mono/microservices aside and focus on what to consider before automating deployments to Kubernetes.

Being simple in the beginning, a deployment process can become a real headache in the future as the complexity of your application and infrastructure grows. This headache can come from unexpected places like the cost of onboarding new team members or the development pace in the future. That is why we should think about a deployment process in the business context.

Therefore, the blog post starts with ownership of Kubernetes clusters and the deployment process. The team structure will define if you need to standardise deployments and the level of expertise you could expect from a person running a deployment. The first point below addresses the ownership question; other points cover resulting technological choices.

## Questions to ask before configuring deployments to Kubernetes

### Team structure and ownership — Centralisation vs Autonomy

Will you have a centralised team (e.g. a platform team) to manage Kubernetes clusters and CD pipelines? Or will each software team own their deployment configurations and, maybe, manage separate clusters? The choice depends on factors such as organizational structure, scale, resource availability, desired level of standardisation and autonomy.

You will benefit from standardisation and governance if you choose the centralised approach. A platform team can enforce best practices to minimise configuration e errors. The centralised team will likely have more experience in Kubernetes, and you won't need much expertise on the software team's level. It's possible that centralisation will also give you more efficient resource management.

On the other hand, there will be a strong dependency on the platform team, and some software teams might struggle with the lack of flexibility. 

The decentralised approach will give more autonomy to the software teams, potentially enabling faster iterations and flexible choice of technologies. However, as you grow, you might face issues with inefficiencies and inconsistent practices. Decentralisation might be an issue if multiple teams work on a set of tightly coupled services.

The choice you make will impact other decisions down the line. The centralised approach will require templating capabilities and tools to hide Kubernetes complexity from software teams. 

### How to write Kubernetes configurations

Now that you know how centralised you want to be, you can decide how you want to write Kubernetes configurations.

The recommended way to manage Kubernetes is to provide declarative configurations for resources in JSON/YAML format. In most cases, it makes sense to follow this recommendation. However, there is more than one way to create and manage such configurations.

You need to provide a version of a resource manifest to each cluster you're going to run your application on. Therefore, you might need three variations of the same resource if you have three environments. Same with tenants (e.g. if your app runs in multiple regions). You might want to standardise your configurations (see the centralisation vs silos above); thus, you might reuse the same resource manifest for different apps.

As you can see, writing and maintaining configurations per app, environment, and tenant isn't always a viable option. Multiple alternatives are available to solve this problem.

1. __Helm.__ A powerful package manager to create templates for Kubernetes. It's very powerful and has a large community. At the same time, it might be hard to learn, and it doesn't store configurations in the valid YAML format, which makes it harder to inspect.
2. __Kustomize.__ Another powerful way to create and manage templates. It's also complex and requires some initial investment. Kustomize also cannot manage dependencies. However, it allows engineers to start with valid YAML files.
3. __Jsonnet.__ Arguably, the most powerful tool on the list allows engineers to describe resources with programme code. It could be a great tool for complex scenarios, but debugging isn't easy.
4. __Terraform.__ You can configure Kubernetes deployments with Terraform. It's not the most popular way, but engineers can use one tool for all infrastructure as code. On the other hand, Terraform is limited compared to the previous options.
5. __Variable replacement.__ Octopus and some other CD tools can use variables to change values in YAML files. Using Octopus, you have a choice of explicitly adding variables in YAML or using structured variables replacement. The second option allows engineers to use valid YAML. Another advantage of variables — a platform team can create a step template and expose a limited number of variables to software teams hiding all the Kubernetes complexity.

Some of the options above are mutually exclusive. For example, it would be hard to move from Helm to Jsonnet. However, combining one of the first four options with variable replacement might provide excellent results. For example, you can use Helm to create per-app charts and alternate the charts per environment or tenants with Octopus variables.

### Where to store variables

Despite the configuration method to choose, you'll have to deal with variables in some form. The question is where to keep them. 

You can store them all in a CD tool or put all variables in your customisation/variables files or choose a hybrid approach. The decision here depends on who will manage the variables and the CD process overall. 

Another aspect of this problem is if you want to deploy the variables with the app or if you want to have a separate process for this. Some Octopus customers combine all the ConfiguMaps and Secrets in one project and redeploy them every time they update the values. Such a solution can work well if you reuse some values for multiple apps.

### Adding new apps

Finally, you need to consider how you will configure deployments for new apps, especially if you want to follow a standard. The approach you choose should be based on all the choices above.

A big question is who should create a new project (a software or platform team) and how standardised the projects should be.

If you lean on a CD tool, you can use the functionality available in the tool. Octopus allows you to create step templates, so adding a new app might be as easy as creating a new project, adding a step template and configuring a few variables. You can go even further and create new projects in Octopus with Terraform.

An alternative approach would be cloning a Helm chart from a template or forking a Kustomize repo. It depends on the tool you use.

Of course, you can combine a few strategies, e.g., use Helm to template Kubernetes configuration and use Terraform to create an Octopus project with one step — update Helm chart.

![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*

## Conclusion

Before configuring a CD pipeline for apps running on Kubernetes, there are a few things to consider. 

- Start with choosing a centralised vs siloed approach
- Consider what you want to use to create and template configurations
- Think about where you store variables and how you deploy them
- Find a way to create new projects

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
