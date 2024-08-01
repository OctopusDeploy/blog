---
title: Git Protections 
description: Git Protections will add an extra layer of protection when deploying your code. Making sure the right resources are being deployed to the right environments.
author: harriet.alexander@octopus.com
visibility: public
published: 
metaImage: https://github.com/user-attachments/assets/c8944f71-a357-48af-bfd3-969870bbf948.png
bannerImage: https://github.com/user-attachments/assets/c8944f71-a357-48af-bfd3-969870bbf948.png
bannerImageAlt: Git connected to locked channels for deployment.
isFeatured: false
tags: 
  - Git
  - Product
---

## Introducting Git Protections
At Octopus Deploy, we are constantly enhancing our tools to better support your Git-based projects and workflows. We are thrilled to announce the introduction of Git Protection Rules, a pivotal addition aimed at safeguarding your deployments and ensuring that only the intended Git resources are being deployed into the right environments.  

In recent years, Octopus Deploy has introduced numerous features to streamline interaction with Git repositories. This includes  Config-as-Code as well as the ability to source files from both external Git repositories and the project repository used in version controlled projects. While these features have empowered users to integrate Git resources seamlessly into their deployment processes, they have also highlighted the need for better protections. 

You may be wondering how this is going to help when you already have version rules in place for your packages. Git Protection Rules allows you to take a step further by defining your own restrictions on the Git Resources used in deployments, which will ensure that only intended resources are deployed to their intended environment. This allows you to continue to leverage the full potential of Git whilst maintaining control over your deployment process.  

### Key features of Git Protections include:
**Customisable Rules:** Define rules tailored to your project's needs, specifying which branches or tags are permissible for deployment.

**Enhanced Security:** Mitigate risks associated with unauthorized or outdated Git resources being used in production or other sensitive environments.


#### External Repository rules

Octopus supports sourcing files from an external Git repository configured on the step, enabling scenarios such as storing scripts or Kubernetes manifests in a repository for use during a deployment.
External repository rules allow you to configure which branches and tags can be used for these steps when creating a release, ensuring that only approved Git resources are used during a deployment to protected environments such as Production.
<img width="968" alt="external-repository-rules" src="https://github.com/user-attachments/assets/eabf2b8b-6620-4bd5-862c-70b9089d9839">




#### Project Repository rules 

Octopus supports storing the deployment process and variables for a project in a Git repository, enabling you to use the power of Git branches and tags to manage and iterate on the steps within a deployment. 
Rules for the project repository for version controlled projects allow you to configure which branches and tags can be used as the source of the deployment process and variables when creating a release. This helps ensure that only approved processes and variables are used during deployments to protected environments such as Production.
<img width="968" alt="project-repository" src="https://github.com/user-attachments/assets/d179bf11-b015-488e-8f26-6dc8a79309f0">

## Conclusion

The additon of Git Protections is being rolled out to Cloud instances from late July. If you are a self-hosted customer, you can expect this feature later in the year.

We are always happy to hear feedback. Feel free to jump into the blog comments to share your thoughts.

## Learn More
- [Git Protections documentation](https://octopus.com/docs/releases/channels)

Happy deployments!


 
