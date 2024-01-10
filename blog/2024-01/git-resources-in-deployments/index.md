---
title: Using Git resources directly in deployments
description: Additional support for sourcing dependencies directly from Git without intermediate packaging required
author: robert.erez@octopus.com
visibility: public
published: 2025-01-15-1400
metaImage: na
bannerImage: na
bannerImageAlt: na
isFeatured: false
tags: 
  - ???
---

Deployment processes often rely on artifacts that don’t require any sort of processing during a continuos integration build. Scripts that can be sourced directly from a repository without some intermediate compilation are the norm. In some systems this can also include entire declarative configuration files, such as Kubernetes manifests or Terraform templates. 

Octopus previously required users to bundle these resources up into an artifact such as a zip archive. These build outputs would exist just so that it could be referenced by Octopus and later extracted and utilized during a deployment. This artificial requirement felt unnatural and added additional steps to what should otherwise be a simple process. The file is right there in your repository, why can’t you use it in your deployment directly?!

With recent additions to Octopus Deploy, we have transformed the way in which users can source these dependencies through a direct reference to the Git repository itself in a deployment process.

There are two new scenarios that we now support; files that exist in the same Git repository that the version controlled Octopus project is stored, and files that exist in some other external Git repository. Let’s explore these two options and how we might use them.

## External Git references
The recently available support for [YAML manifests from Git](https://octopus.com/blog/manifests-from-git) on the `Raw YAML` step, provided a sneak peek at a new way of sourcing dependencies for your deployments. In this update we streamlined the way in which entire Kubernetes manifests could be brought directly into a deployment without the need for any intermediate packaging or build steps outside Octopus.

This functionality has now been enabled across the board (as of [v2023.4.7982](https://github.com/OctopusDeploy/Issues/issues/8442)) to all steps that support scripts and configuration manifests. This makes sourcing scripts from your Git repositories easier than ever and provides a mechanism for the centralization of these shared dependencies across multiple Octopus projects. 

### Externally sourced script example

In the following example we have a simple bash script committed to our repository.

![Simple Script in Git](external-git-bash.png)

We can now reference this script directly, without needing it to be packaged by selecting the new Git Repository option on our Run a Script step in Octopus. This works for [Config as Code projects](https://octopus.com/docs/projects/version-control) and standard database backed projects.

![Run-a-Script Step with external Git resource](external-git-run-a-script.png)

Providing the default branch on the step itself means that we can easily provide backwards compatibility with tooling that might not yet know that this feature exists. At the time of release creation, the tip of this branch will be used to source the script contents.

![Release creation with external Git resource](external-git-release-creation.png)

The commit selected during release creation will be recorded, and at deployment time that specific commit will be checked out and its contents used during the execution. This does mean that just like with standard packages, the commit needs to be available during the deployment. If the repository is deleted for example, then future releases referencing it may fail to run.

### Limitations
* We have some work items in our backlog to support selecting the branch at release creation however at the time of writing this, neither the branch nor commit be changed during release creation. The tip of the provided default pranch will always be what is recorded at release creation.

* Since we need to resolve a specific commit at release creation time, the `Repository Url` and `Branch` properties only support the use of unscoped variables. Conversely, the `Path` and `Parameters` options can use any scope since these are resolved and utilized during a deployment itself. Attempting to use a variable utilizing an unsupported scope will result in an error at release creation.

* A full clone of the relevant project repository on the Octopus Server machine takes place during deployment so that the relevant files can be extracted. In addition, since Octopus Server cannot know which files will ultimately be required from deep within a customer’s potentially dynamic scripts, the entire repository contents are transferred across to the target or worker during a deployment. In future we may support cloning on the target or performing shallow clones, however in this initial release you should expect some time spent on the initial clone for each repository used.

## Config As Code Project References

If you are using a Config as Code (CaC) Octopus Project, you can now also make use of files stored in the very same repository as Octopus's deployment process files. Similar to the externally sourced scripts above, this allows you to write and commit your files directly into a Git repository and make use of them in your project.

By sourcing the scripts from the same repository as your project configuration, there is much less information that needs to be provided to Octopus to source the relevant files. The same repository and commit that is checked-out for the deployment process will be used to source the script itself. 

This option also makes it much simpler and convenient to iterate on a script alongside the deployment process that uses it. You can be confident that regardless of which branch or commit you are deploying, the right dependencies will automatically be included in the deployment being executed. You can work on your scripts and process simultaneously in a feature branch, testing them together before merging the changes in a single commit into your release branch.

This approach also bypasses the current limits with external Git references mentioned above that constrains which branch that the files can be sourced from. Since it will automatically use whatever commit that the process itself is using, this could be any branch, tag or PR.

### Config As Code Project sourced script example

When configuring the sample Terraform step in a CaC enabled project, the `Git Repository` option will expose a second option for `Repository Source`.  Selecting `Project` will use the repository associated with the relevant CaC project itself while selecting `External` will expose the options noted above for Externally Git references.

![Terraform Step with project Git resource](cac-git-terraform.png)

This `Repository Source` option will only show up in CaC projects and it is otherwise implied for non-CaC projects that `Git Repository` sourced files are using an external repository.

Having configured our deployment process, when we go to create the release, you can see that we only need to select the branch that defines the CaC project process. The same commit selected for the release itself will be used for sourcing the relevant files.

![Alt text](image.png)

## Why would I want to use this?

**Improved Development Experience**

When creating or updating scripts or configurations for use in a deployment, you probably want to run them locally to validate the changes that you make. Rather than cutting and pasting files between the Octopus portal and your local filesystem, its much safer and reliable to be able to edit the scripts directly from files sourced from a repository. This provides all the additional standard benifits of using source control of versioning, auditability and change control.

**Accessing Dependencies**

Often the dependencies required are made up of more than just one file. A Kustomze deployment will typically be made up of multiple configuration files in various folder locations. Complex scripts might be composed of several sub scripts or rely on small tools. Since we clone the entire repository, all of these pieces will be available at deploy time for use.

**Managing Centralized Scripts and Resources**

By storing our scripts in a Git repository rather than directly in the process itself, this provides another mechanism with which we can share processes across projects, and even Octopus instances. Perhaps the platform team has provided a managed Kubernetes manifest that is used for multiple teams to deploy their application. Each team only need to configure their different application images and variables that are injected into the manifest at runtime. You may have also have maintenance scripts that you want to use in runbooks across different spaces and when that process is updated you want all new releases to use that latest version without consumers even having to think about it. All sorts of new simpler templating and sharing options become available.


## Summary
Although we have had [GitHub Feeds](https://octopus.com/docs/packaging-applications/package-repositories/github-feeds) available to users for some time, this introduces CI complexities and requires additional overhead which in todays world of GitOps should no longer be required.

Sourcing your deployment dependencies from either the same Git repository used for your CaC project, or from a seperate external repository opens up a range of new options and ways in which you can structure your deployments.

We are excitied to see how our users make use of this new feature, so please let us know what new capabilities this does (or doesn't) open up for your release pipelines.

Happy (Git) Deployments!