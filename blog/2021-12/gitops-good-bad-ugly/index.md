---
title: GitOps with the hype
description: 
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

GitOps is a relatively new addition to the growing list of "Ops" paradigms taking shape in our industry. It all started with DevOps, and while the term DevOps has been around for some years now, it seems we still can't agree whether it is a process, mindset, job title, set of tools, or some combination of them all. The term GitOps suffers from the same ambiguity, and so in this post you'll look at the history of GitOps, its goals and ideals, the tools that support it, and the practical implications of adopting GitOps in your own organization.

## The origins of GitOps

The term GitOps was originally coined in a blog post by WeaveWorks called [GitOps - Operations by Pull Request](https://www.weave.works/blog/gitops-operations-by-pull-request), which described how WeaveWorks utilized git as a source of truth, leading to the following benefits:

>    Our provisioning of AWS resources and deployment of k8s is declarative
>    Our entire system state is under version control and described in a single Git repository   
>    Operational changes are made by pull request (plus build & release pipelines)
>    Diff tools detect any divergence and notify us via Slack alerts; and sync tools enable convergence
>    Rollback and audit logs are also provided via Git  

Since that original blog post, initiatives like the [GitOps Working Group](https://github.com/gitops-working-group/gitops-working-group) have been organized to:

> clearly define a vendor-neutral, principle-led meaning of GitOps, which will establish a foundation for interoperability between tools, conformance, and certification.

This working group recently released version 1 of their [principals](https://github.com/open-gitops/documents/blob/main/PRINCIPLES.md), which states that:

> The desired state of a GitOps managed system must be:
>
>    Declarative
>
>    A system managed by GitOps must have its desired state expressed declaratively.
>
>    Versioned and Immutable
>
>    Desired state is stored in a way that enforces immutability, versioning and retains a complete version history.
>
>    Pulled Automatically
>
>    Software agents automatically pull the desired state declarations from the source.
>
>    Continuously Reconciled
>
>    Software agents continuously observe actual system state and attempt to apply the desired state.

The contrast between low level descriptions of GitOps found in most blog posts and the high level ideals of a GitOps system described by the working group is worth some discussion, as the differences between them is a source of much confusion.

## GitOps doesn't imply the use of git

Most discussions around GitOps center on how building processes on git give rise to many of the benefits ascribed to the GitOps paradigm. Git naturally provides an (almost) immutable history of changes, with changes annotated and approved via pull requests, and where the current state of the git repository naturally represents the desired state of a system, thus acting as a source of truth. The overlap between git and GitOps is undeniable.

However, you may have noticed that git was never mentioned as a requirement of GitOps by the working group. So while git is a natural component of a GitOps solution, GitOps itself is concerned with the functional requirements of a system rather than checking your declarative templates into git.

This distinction is important, because many teams are fixated on the "Git" part of GitOps. The term GitOps is an unfortunate name for the concept it is trying to convey, leading many to believe git is the central aspect of GitOps. But GitOps has won the marketing battle and gained mind share within I.T. departments, so while it may be a proscriptive term to describe functional requirements unrelated to git, GitOps is now the shorthand for describing processes that implement a set of high level concerns.

## GipOps doesn't imply the use of Kubernetes

Kubernetes was the first widely used platform to combine the ideas of declarative state and continuous reconciliation along with the ability to host running applications. It really is magic to watch a Kubernetes cluster reconfigure itself to match the latest templates applied to the system. So it is no surprise that Kubernetes is the foundation of GitOps tools like Flux and Argo CD, while posts like [30+ Tools List for GitOps](https://dzone.com/articles/30-tools-list-for-gitops) mention Kubernetes 20 times.

While continuous reconciliation is impressive, it is not really magic. Behind the scenes Kubernetes runs a number of [operators](https://octopus.com/blog/operators-with-kotlin) that are notified of configuration changes and execute custom logic to bring the cluster back to the desired state.

The key requirements of continuous reconciliation is:

* Access to the configuration or templates declaratively expressing the desired state.
* The ability to execute a process capable of reconciling a system when configuration is changed.
* An environment in which the process can run.

Kubernetes bakes these requires into the platform, making it very easy to achieve continuous reconciliation. But these requirements can also be met with some simple orchestration, Infrastructure as Code (IaC) tools like Terraform, Ansible, Puppet, Chef, CloudFormation, Arm Templates etc, and an execution environment like a CI server or Octopus:

* IaC templates can be stored in git, file hosting platforms like S3 or Azure Blob Storage, complete with immutable audit histories.
* CI/CD systems can poll the storage, are notified of changes via webhooks, or have builds or deployments triggered via platforms like GitHub Actions.
* The IaC tooling is then executed, bringing the system in line with the desired state.

Indeed, real world end-to-end GitOps system will inevitably have to incorporate some orchestration outside of Kubernetes. For example, Kubernetes is unlikely to manage your DNS records, centralized authentication platforms, or messaging systems like Slack. You will also likely find at least one managed service for things like databases, message queues, scheduling, reporting etc more compelling than attempting to replicate them in a Kubernetes cluster. Also, any established I.T. department is guaranteed to have non-Kubernetes systems that would benefit from GitOps.

So while the initial selection of specialized GitOps tools tend to be tightly integrated into Kubernetes, achieving the functional requirements of GitOps across established infrastructure will inevitably require orchestrating one or more IaC tools.

## GitOps is not a complete solution

While GitOps describes many desirable traits of well managed infrastructure and deployment processes, it is not a complete solution. In addition to the four functional requirements described by GitOps, a robust system must be:

* Verifiable - infrastructure and applications must be testable once they are deployed.
* Recoverable - teams must be able to recover from an undesirable state.
* Visible - the state of the infrastructure and the applications deploy to it must be surfaced in an easily consumed summary.
* Secure - rules must exist around who can make what changes to which systems.
* Measurable - meaningful metrics must be collected and exposed in an easily consumed format.
* Standardized - applications and infrastructure must be described in a consistent manner.
* Maintainable - support teams must be able to query and interact with the system, often in non-declarative ways.
* Coordinated - changes to applications and infrastructure must be able to be coordinated between teams.

GitOps offers little advice or insight into what happens before configuration is committed to a git repo or other versioned and immutable storage, but it is "left of the repo" where the bulk of your engineering process will be defined. 

If you git repo is the authoritative representation of your system, then anyone who can edit a repo essentially has administrative rights. However, git repos don't provide a natural security boundary for the kind of nuanced segregation of responsibility you'll find in established infrastructure. This means you'll end up creating one repo per app per environment per role. Gaining visibility over each of these repos and ensuring they have the correct permissions is no trivial undertaking.

You will also quickly find that just because you can save anything in git doesn't mean you should. It is not hard to imagine a rule that says development teams must create Kubernetes deployment resources instead of individual pods, use ingress rules that respond to very specific hostnames, and always include a standard security policy. This kind of standarization is tedious to enforce through pull requests, so a much better solution is to give teams standard resource templates that they populate with their specific configuration. But this is not a feature inherent to git or GitOps.

We then have those processes "right of the git repo" where management and support tasks are defined.

Reporting on the intent of a git commit is almost impossible. If you were to look at a diff between two commits and saw that a deployment image tag was increased, new secret values were added, and a config map was deleted, how would you describe the intent of that change? The easy answer is to read the commit message, but this is not a viable option for reporting tools that must map high level events like "deployed a new app version" or "bugfix release" (which are critical if you want to measure yourself against standard metrics like those presented in the [DORA report](https://www.devops-research.com/research.html)) to the diff between two commits. Even if you could divine an algorithm that understood the intent of a git commit, a git repo was never meant to be used as a timeseries database.

GitOps also provides no guidance on how to perform support tasks once the system is in its desired state. What would you commit to a git repo to delete misbehaving pods so they can be recreated by their parent deployment? Maybe a job could do this, but you will have to be careful that Kubernetes doesn't try to apply that job resource twice. But then what would you commit to the repo to view the pod logs? My mind boggles at the thought of all the asynchronous message handling you would need to implement to recreate `kubectl logs mypod` in a GitOps model.

Adhoc reporting and management tasks like this don't have a natural solution in the GitOps model.

This is not to say that GitOps is flawed or incomplete, but rather that it solves specific problems, and must be complemented with other processes and tools to satisfy basic operational requirements.

## Git is the least interesting part of GitOps

I'd like to present you with a theory and a thought experiment to apply it to:

**In any sufficiently complex GitOps process, a git repo is indistinguishable from a structured database.**

You start your GitOps journey using the common combination of git and Kubernetes. All changes are reviewed by pull request, committed to a git repo, consumed by a tool like Argo CD or Flux, and deployed to your cluster. You have satisfied all the functional requirements of GitOps, and enjoy the benefits of a single source of truth, immutable change history, and continuous reconciliation.

But it starts to become tedious to have a person open a pull request to bump the image property in a deployment resource every time a new image is built and published. So you instruct your build server to pull the git repo, edit the deployment resource YAML file, and commit the changes. You now have GitOps and CI/CD.

You now need to measure the performance of your engineering teams. How often are new releases deployed to production? You quickly realize that extracting this information from git commits is inefficient at best, and that the Kubernetes API was not designed to frequent and complex queries, so you choose to populate a more appropriate datastore like a database with deployment events.

As the complexity of your cluster grows, you find you need to implement standards around what kind of resources can be deployed. Engineering teams can only create deployment, secrets, and configmaps. The deployment resources must include resource limits, a set of standard labels, and the pods must not be privileged. Meanwhile the security team can create roles, role bindings, and security policies. In fact, it turns out that of the hundreds of lines of YAML that make up the resources deployed to the cluster, only about 10 should be customized. Combined with the fact that you already have automated processes that expect to find a deployment YAML file with a given name so the image tag can be updated, you implement a rule that 