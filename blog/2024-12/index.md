---
title: Introducing Git Triggers
description: Trigger a deployment straight fromm your Git repository
author: harriet.alexander@octopus.com
visibility: public
published: 2024-12-06
metaImage: 
bannerImage: 
bannerImageAlt: 
tags:
- Product
---
At Octopus, we’re always looking for ways to simplify and automate deployment processes, and we’re excited to introduce Git Triggers—a new feature that streamlines the release creation process.
Up until now, deploying new versions of applications or infrastructure at Octopus required an explicit “Create A Release” step in a Continuous Integration (CI) pipeline. However as we see of our customers transition to a GitOps model, where dependencies and deployments are increasingly controlled through version control, we at Octopus wanted to provide more flexibility in how our customers trigger their deployments. With Git Triggers, we’re taking steps toward a pull-based deployment model directly from your Git repositories.


### What Are Git Triggers?

Git Triggers are an automated way to trigger deployments based on changes in your Git repository. Rather than relying on the typical CI pipeline to create releases, Git Triggers let you automatically create releases when changes occur in the repository. This approach is especially useful as teams move towards GitOps practices, where infrastructure, application configurations, and deployments are managed directly through Git repositories.

With Git Triggers, deployments are no longer tied to explicit CI actions. Instead, they can be initiated by a simple commit to your repository, making the entire release cycle more efficient and aligned with modern development workflows.

### When Should You Use Git Triggers?
Git Triggers are an automated way to trigger deployments based on changes in your Git repository. Rather than relying on the typical CI pipeline to create releases, Git Triggers let you automatically create releases when changes occur in the repository. This approach is especially useful as teams move towards GitOps practices, where infrastructure, application configurations, and deployments are managed directly through Git repositories.

With Git Triggers, deployments are no longer tied to explicit CI actions. Instead, they can be initiated by a simple commit to your repository, making the entire release cycle more efficient and aligned with modern development workflows.

### When Should You Use Git Triggers?
There are several common use cases for Git Triggers within Octopus that highlight how this feature can improve your workflow:
1. **Deploying a New Version of an Application with Helm**
   If you’re using Helm charts to manage your Kubernetes applications, a change to the version number within your Helm chart repository can automatically trigger a new release and deployment. This eliminates the need for manual intervention or complex CI steps when updating versions.
2. **Deploying with Kustomize**
   If you’re deploying off-the-shelf applications with Kustomize, changes to Kustomize YAML files in your Git repository can automatically trigger a new release. Whether you’re managing multiple environments or simply updating your configurations, Git Triggers make deployments seamless.
3. **Managing Infrastructure with Terraform**
   For infrastructure as code (IaC) users, Git Triggers are incredibly useful. A commit that changes your Terraform resource files can automatically create a release and update your infrastructure in the cloud, ensuring everything is up to date and aligned with your latest Git commits.
### How Do Git Triggers Work?
Git Triggers are simple to set up and work just like other triggers in Octopus (such as External Feed or Scheduled triggers). Here’s a quick rundown of how they operate:
1. **Configuring Git Triggers**
   To get started, you configure Git Triggers in your project, just like you would with any other trigger type. You’ll select the repositories you want Octopus to monitor for changes.
![image of Git Triggers along with other Trigger options](https://github.com/user-attachments/assets/44507ebd-6e5e-44c3-b34a-b09b9e6ebb53)
![image of how to configure a git trigger](https://github.com/user-attachments/assets/70244c9d-e6eb-43a1-b4a7-79a0a03f1dcf)

2. **Polling Repositories**
   Octopus will then check these repositories every 3 minutes for new commits. When new commits are detected, a new release will automatically be created,and can be set up to automatically deploy.
3. **Customizing What Triggers a Release**
    To trigger a release only for commits in a specific folder of your repository, you can configure **include** or **exclude** paths using glob patterns such as `*` . This ensures that only commits affecting the specified folder initiate a new release helping avoid unnecessary deployments.
![image of how to include and exclude file paths](https://github.com/user-attachments/assets/3494fb32-5415-45f2-bacd-c710e2a003ac)

### Why Choose Git Triggers?
The benefits of Git Triggers go beyond just convenience. Here’s why you should consider using them for your deployments:
- **Simplicity:** Automates release creation without needing an explicit CI step.
- **Flexibility:** Works seamlessly with tools like Helm, Kustomize, and Terraform for both application and infrastructure deployments.
- **Efficiency:** Allows for faster feedback loops by immediately triggering a release upon committing changes to your Git repo.
- **GitOps Alignment:** Perfect for teams adopting GitOps practices, as it keeps everything within Git without needing extra tools or manual interventions.


## Get Started with Git Triggers Today

We believe Git Triggers will greatly enhance your deployment workflows, making them simpler, faster, and more reliable. This feature is now available in all Cloud instances and will be released in the upcoming 2024.4 server release setting you up to enhance your deployment process in the New Year. Start using Git Triggers and let us know your thoughts.


Happy deployments!
