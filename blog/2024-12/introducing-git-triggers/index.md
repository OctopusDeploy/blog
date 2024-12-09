---
title: Introducing Git triggers
description: Trigger a deployment straight fromm your Git repository
author: harriet.alexander@octopus.com
visibility: public
published: 2024-12-11-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
- Product
---

We're excited to introduce Git triggers – a new feature that streamlines the release creation process.

Until now, to deploy new versions of applications or infrastructure, you needed to use an explicit **Create a release** step in a Continuous Integration (CI) pipeline. However, we're seeing more customers transition to a GitOps model, where dependencies and deployments are increasingly controlled through version control. With this in mind, we want to provide more flexibility in how you can trigger your deployments. With Git Triggers, we're moving toward a pull-based deployment model directly from your Git repositories.

In this post, I explain our new Git triggers feature and the benefits of using it.

## What are Git triggers?

Git triggers are an automated way to trigger deployments based on changes in your Git repository (repo). Rather than relying on the typical CI pipeline to create releases, Git triggers let you automatically create releases when changes occur in the repo. This is especially useful for teams adopting GitOps practices, who manage infrastructure, application configurations, and deployments directly through Git repos.

With Git triggers, deployments are no longer tied to explicit CI actions. Instead, you can start a deployment when you make a commit to your repository. This makes the entire release cycle faster and aligns better with modern development workflows.

## When to use Git triggers

There are several common use cases for Git triggers in Octopus that show how this feature can improve your workflow:

1. **Deploying a new version of an application with Helm** – If you're using Helm charts to manage your Kubernetes applications, a change to the version number in your Helm chart repository can automatically trigger a new release and deployment. This eliminates manual interventions and complex CI steps when updating versions.
2. **Deploying with Kustomize** – If you're deploying off-the-shelf applications with Kustomize, changes to Kustomize YAML files in your Git repository can automatically trigger a new release. Whether you're managing multiple environments or simply updating your configurations, Git triggers make deployments seamless.
3. **Managing infrastructure with Terraform** – For infrastructure as code (IaC) users, Git triggers are incredibly useful. A commit that changes your Terraform resource files can automatically create a release and update your infrastructure in the cloud, ensuring everything is up to date and aligned with your latest Git commits.

## How Git triggers work

Git triggers are simple to set up and work just like other triggers in Octopus (like External Feed or Scheduled triggers). Here's a quick rundown of how they operate:

1. To get started, you configure Git triggers in your project, just like you would with any other trigger type. You select the repositories you want Octopus to monitor for changes.

![Git Triggers along with other Trigger options](https://github.com/user-attachments/assets/44507ebd-6e5e-44c3-b34a-b09b9e6ebb53)

![image of how to configure a git trigger](https://github.com/user-attachments/assets/70244c9d-e6eb-43a1-b4a7-79a0a03f1dcf)

2. Octopus checks these repositories every 3 minutes for new commits. When it detects a new commit, a new release gets created, and can be set up to automatically deploy.

3. To trigger a release only for commits in a specific folder of your repository, you can configure **include** or **exclude** paths using glob patterns such as `*` . This ensures that only commits to the specified folder start a new release, helping avoid unnecessary deployments.

![How to include and exclude file paths](https://github.com/user-attachments/assets/3494fb32-5415-45f2-bacd-c710e2a003ac)

### Why choose Git triggers?

The benefits of Git triggers go beyond just convenience.

- Simplicity: Automates release creation without needing an explicit CI step.
- Flexibility: Works seamlessly with tools like Helm, Kustomize, and Terraform for both application and infrastructure deployments.
- Efficiency: Allows for faster feedback loops by immediately triggering a release when you commit changes to your Git repo.
- GitOps alignment: Perfect for teams adopting GitOps practices, as it keeps everything in Git without needing extra tools or manual interventions.

## Conclusion

Git triggers aim to make your deployment workflows simpler, faster, and more reliable. 

This feature is now available in all Cloud instances. Our self-hosted customers can expect it in the 2024.4 server release. When you start using Git triggers, we'd love to hear your feedback in the comments below.

Happy deployments!