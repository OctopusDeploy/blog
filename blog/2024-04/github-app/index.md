---
title: Octopus Deploy in the GitHub Marketplace 
description: 
author: michael.richardson@octopus.com 
visibility: private
published: 2024-04-22-1400
metaImage: img-moreseamlessintegrationgithubusingoctopusapp-2024.png
bannerImage: img-moreseamlessintegrationgithubusingoctopusapp-2024.png
bannerImageAlt: Octopus Deploy and GitHub logos connected by arrows.
isFeatured: false
tags: 
  - Product
  - GitHub Actions
---

We are excited to announce we are launching an [Octopus Deploy App for GitHub](https://github.com/marketplace/octopus-deploy), available now for Octopus Cloud customers.

Many Octopus customers use GitHub and Octopus together, and we are focused on ensuring the integration is seamless.  The Octopus Deploy GitHub App is a major step on this journey.

Last year we shipped support for [OpenID Connect between GitHub Actions and Octopus](https://roadmap.octopus.com/c/70-openid-connect-oidc-for-github-actions), removing the need for Octopus credentials to be managed in GitHub. The Octopus Deploy GitHub App is now providing this benefit in the opposite direction, removing the need for GitHub credentials to be managed in Octopus. With the combination of OpenID Connect and the Octopus Deploy GitHub App, there is no longer a need for shared credentials when integrating GitHub and Octopus Deploy. This made our SecOps team very happy, and we hope it does the same for yours.

An immediate benefit is for Octopus projects using Config as Code with GitHub as the repository. Creating Octopus projects version-controlled in GitHub is now easier and more secure. 

### Before - Without the Octopus Deploy GitHub App 
Previously, connecting Octopus to your Config as Code GitHub repository required creating an account in GitHub to represent Octopus Deploy, adding a Personal Access Token, and configuring the access token as credentials in Octopus.


![GitHub Personal Access Token](github-pat.png "width=500")

![GitHub Credentials in Octopus](version-control-password.png "width=500")



### After - With the Octopus Deploy GitHub App 

Using the Octopus GitHub App, the app is installed into your GitHub organization, and is granted access to selected repositories.

![Granting the Octopus App access to Repositories](octopus-github-app-install.png "width=500")

Octopus is then configured to use the app for GitHub integration.  No credentials required! The Octopus GitHub App can be shared across all Octopus projects, making setup easier, removing the need for management of access tokens, and removing the risk of inadvertently leaking tokens.

![Configuring the app in your Octopus Project](octopus-app-project.png "width=500")

We are intent on making GitHub and Octopus Deploy the world's most powerful combination for Continuous Delivery. The Octopus Deploy GitHub App lays the foundation for deeper integration between GitHub and Octopus Deploy.  

The Octopus Deploy GitHub App is available now, for Octopus Cloud customers. 

_A note for our self-hosted Octopus Server customers: we developed this as a cloud-first feature as supporting self-hosted instances introduces additional complexity.  If you are an Octopus Server customer, and you would benefit from the Octopus Deploy GitHub App, please talk to your Account Manager or add your voice to the [roadmap feature card](https://roadmap.octopus.com/c/107-github-app-for-server-customers).  We're listening._  





