---
title: "Octopus 2020.5 - K8s improvements and Config as Code update"
description: "Octopus 2020.5 introduces TODO"
author: rob.pearson@octopus.com
visibility: public
bannerImage: release-2020.5.png
metaImage: release-2020.5.png
published: 2020-11-30
tags:
- Product
---

![Octopus 2020.5](release-2020.5.png)

I'm pleased to share that Octopus 2020.5 is now available. This is a quieter release as we're doing a lot of behind-the-scenes work but it's still has some solid improvements.

This release is the [fifth of six in 2020](/blog/2020-03/releases-and-lts/index.md), and it includes six months of long-term support. The following table shows our current versions with long-term support:

| Release               | Long-term support  | LTS end date |
| --------------------- | ------------------ | ------------ |
| Octopus 2020.5        | Yes                | 2021-05-30   |
| Octopus 2020.4        | Yes                | 2021-03-21   |
| Octopus 2020.3        | Yes                | 2021-01-20   |
| Octopus 2020.2        | Expired            | 2020-09-30   |
| Octopus 2020.1        | Expired            | 2020-08-24   |


## Create self-signed certificates in the certificate library 

![Create self-signed certificates in the certificate library ](self-signed-certificates.png)

Creating a self-signed certificate for development and testing purposes isn't difficult but Octopus now makes this faster and simpler. You can create a self-signed certificate in the Certificate library (**{{ infrastructure,Deployment Targets }}**) and take advantage of it in your automation processes. This is very handy testing the deployment of new or updated services. You can also download it if you need to use it with command line interfaces (CLI), desktop applications or other tasks.

[Learn more](https://octopus.com/docs/deployment-examples/certificates)

## GitHub container registry support 

Technically Octopus already supported GHCR, though you would never guess it from the experience of trying to configure it.

GHCR has not (yet) implemented the Docker catalog API which allows search for repositories.

This is resulting in the UI not behaving nicely when:

[Learn more](https://github.com/octopusdeploy/issues/issues/6567)

## Kubernetes Updates

This release includes two small updates to improve our Kubernetes support driven by customer feedback.

**Expose `envFrom` fields in Deploy Kubernetes containers step**

Kubernetes 1.16 exposed the envFrom property to allow the contents of a secret or configmap to be created as environment variables. Octopus now supports `envFrom` fields to provides a way for multiple values to be included in a deployment.

[Learn more](/blog/2020-12/k8s-envfrom/index.md)  
  
**Allow `Daemonsets` and `Statefulsets` to be created and deployed**

The Deploy Kubernetes containers step allows K8s deployments to be created and in a cluster. `DaemonSets` and `StatefulSets` are very similar in the fields the accept (the container definitions are the same for example). We have updated the Octopus Web Portal to provider an interface for these additional resources.

[Learn more](https://github.com/octopusdeploy/issues/issues/6551)

## Terraform update

Octopus now supports support HCL2 and Terraform 0.12+ for inline scripts. This means you get rich syntax highlighting for modern Terraform scripts in the Octopus Web Portal.

[Learn more](https://github.com/octopusdeploy/issues/issues/6562)

## Add markdown notes to automation steps

![Add markdown notes to automation steps](automation-step-notes.png)

You can now annotate your DevOps automation processes with markdown notes. Add text-based notes, with markdown formatting support, to any deployment or runbook step and it will be displayed in the process summary.

This is a useful to help future-self or other team meber to understand complex automated processes at a glance.

![Add markdown notes to automation steps](https://github.com/octopusdeploy/issues/issues/6608)

## Config as Code Update

![Config as Code Update](branch-switcher.png)

We originally wanted to launch the early access preview of our Config as Code feature this month (November 2020), but unfortuantely we're not ready for this. We underestimated how long some key components would take to build, but also because this feature is an especially tricky one to ship incrementally.

That said, we're making very good progress and we've completed some major parts of the overall feature set.
* Projects can be configured with a git repository.
* You can switch branches in the Octopus Web Portal, viewing and editing the deployment process on different branches.
* Changes can be committed, including adding a commit message.
* The result is stored in git in a format based on HCL.
* The branch can be specified when creating releases, selecting the version of the deployment process to use. This will eventually support selecting tags or commits also.

So what’s left, you may ask? The short answer is “all the small things”. It turns out that relocating a chunk of Octopus from the database (where there’s a single version, indexes, foreign-keys, etc) and dropping it into a git repository (with text files and limitless branches), leaves a bunch of loose ends. Who would have guessed? 

Click the learn more link below to read more about the factors that have gone into designing and building our config as code support. We explicility call out several anti-patterns that we have intentionally avoided which contribute to the complexity of our implementation. 

[Learn more](/blog/2020-11/shaping-config-as-code/index.md)

## Breaking changes

This release doesn’t include any breaking changes.

## Upgrading

Octopus Cloud users are already running this release, and self-hosted Octopus customers can [download](https://octopus.com/downloads/2020.5.0) the latest version now.  

As usual, the [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2020.5.0) for further information.

## What’s coming in Octopus 2020.6?

Check out our [public roadmap](https://octopus.com/roadmap) to see what’s coming next and register for updates. TODO

## Conclusion

Octopus 2020.5 is now generally available, and it includes a collection of improvements to support self-signed certficates, Terraform updates, Kubernetes updates, automation step notes and a ton of behind-the-scenes changes to support our upcoming config as code feature.


Feel free to leave a comment, and let us know what you think! Happy deployments!

## Related posts

* [Deconstructing blue/green deployments in Kubernetes](/blog/2020-10/deconstructing-blue-green-deployments/index.md)
* [Creating multi-environment Kubernetes deployments](/blog/2020-12/multi-environment-k8s-deployments/index.md)
* [Exposing Octopus variables to a Kubernetes container](/blog/2020-10/k8s-envfrom/index.md)
* [Shaping Configuration-as-Code](/blog/2020-11/shaping-config-as-code/index.md)
