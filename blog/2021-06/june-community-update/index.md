---
title: Octopus Deploy June Community update
description: See all the latest updates from the Octopus community in June 2021
author: derek.campbell@octopus.com
visibility: public
published: 2021-06-15-1400
metaImage: blogimage-community-youtube-2021.png
bannerImage: blogimage-community-youtube-2021.png
tags:
 - Community

---

![Community Image](blogimage-community-youtube-2021.png)

I've recently moved into a Community Director role here at Octopus Deploy HQ and as part of my role, I am hoping to engage the community better as our Community is essential, and in this blog, we cover Community updates over the last month and share them with you.

## Webinars

In this section, I'll list out the most recent webinars that were delivered.

### Delivering Database DevOps with Liquibase and Octopus Deploy

Most organizations have some form of automation to deliver their applications. We often see the Database lags behind and requires manual deployments to Production. In this webinar, Shawn Sesna, a Solutions Architect at Octopus Deploy, and Senior Solutions Architect Mike Olivas from Liquibase, talk and demo:

- What is Liquibase and Octopus Deploy?
- How to create a MongoDB and deploy it
- How to take a code change from commit to Pull Request and then to Dev, Test, and Production.

<iframe width="560" height="315" src="https://www.youtube.com/embed/1nrxnF4LxGw" frameborder="0" allowfullscreen></iframe>

### Accelerating your Azure DevOps Pipeline and Azure Platform as a Service with Octopus Deploy

In this webinar, [Gregor Suttie](https://twitter.com/gregor_suttie), an Azure MVP and Azure Architect, and I go through how to build a multi-tier application in Azure DevOps, deploy the infrastructure using Runbooks in Octopus, and then use Octopus to deploy the website, Database, product service API and shopping cart service to Azure Platform-as-a-service. You'll see and learn about:

- Azure DevOps Classic Editor builds, and YAML builds
- How to connect Azure DevOps pipelines to Octopus
- Creating Azure PaaS using IaC Runbooks
- How to take your code and deploy it to Production from a single release

<iframe width="560" height="315" src="https://www.youtube.com/embed/NRwFdpvNYyA" frameborder="0" allowfullscreen></iframe>

### What's new in Octopus 2020.6 and 2021.1

In this webinar, we go through all the new features of our first releases of 2021. 2020.6 is a transition release, as it's the last release of 2020 and the first release of 2021. You can read more about this change in this [blog](https://octopus.com/blog/octopus-release-2021-q1).

In 2020.6 and 2021.1, we added:

- Octopus Server Linux Docker Image
- Tentacle for ARM/ARM64
- Global Search
- API Keys improvements
- Project Migrations
- New and Improved Azure App Service step
- Github Actions for Octopus Deploy

<iframe width="560" height="315" src="https://www.youtube.com/embed/NRwFdpvNYyA" frameborder="0" allowfullscreen></iframe>

## Blogs

In this section, we'll cover our favorite blogs of the last month.

### Octopus 2021 Q2

In this [blog](https://octopus.com/blog/octopus-release-2021-q2), we announce the latest release of Octopus Deploy. We provide updates on the LTS version of Octopus and contains the Octopus Release Tour. You can also check out the webinar, which shows the latest features off.

### Using Hashicorp Vault with Octopus Deploy

If you are using Vault by Hashicorp, then this comprehensive [blog](https://octopus.com/blog/using-hashicorp-vault-with-octopus-deploy) by Mark Harrison is worth a read. In this blog, Mark goes through how to use the steps to retrieve secrets.

## Step Templates of the month

Mark Harrison, a Solutions Architect in the Octopus Solutions team, has been busy creating new secrets retrieval step templates. If you're using [Azure Key Vault](https://azure.microsoft.com/en-au/services/key-vault/) or [Vault for Hashicorp](https://www.vaultproject.io/), then you can see our new Community Step Templates below.

- [Azure Key Vault - Retrieve Secrets](https://library.octopus.com/step-templates/6f59f8aa-b2db-4f7a-b02d-a72c13d386f0/actiontemplate-azure-key-vault-retrieve-secrets)
- [HashiCorp Vault - AppRole Get Wrapped Secret ID](https://library.octopus.com/step-templates/76827264-af27-46d0-913a-e093a4f0db48/actiontemplate-hashicorp-vault-approle-get-wrapped-secret-id)
- [HashiCorp Vault - AppRole Login](https://library.octopus.com/step-templates/e04a9cec-f04a-4da2-849b-1aed0fd408f0/actiontemplate-hashicorp-vault-approle-login)
- [HashiCorp Vault - AppRole Unwrap Secret ID](https://library.octopus.com/step-templates/c1f56030-0bcd-458d-bc70-b4f43ec0d30f/actiontemplate-hashicorp-vault-approle-unwrap-secret-id)
- [HashiCorp Vault - AppRole Unwrap Secret ID and Login](https://library.octopus.com/step-templates/aa113393-e615-40ed-9c5a-f95f471d728f/actiontemplate-hashicorp-vault-approle-unwrap-secret-id-and-login)
- [HashiCorp Vault - Key Value (v1) retrieve secrets](https://library.octopus.com/step-templates/9aab9522-25e0-4539-841c-8b726e6b1520/actiontemplate-hashicorp-vault-key-value-(v1)-retrieve-secrets)
- [HashiCorp Vault - Key Value (v2) retrieve secrets](https://library.octopus.com/step-templates/337f1b67-cdb0-4f33-9e08-6bf804f672d2/actiontemplate-hashicorp-vault-key-value-(v2)-retrieve-secrets)
- [HashiCorp Vault - LDAP Login](https://library.octopus.com/step-templates/de807003-3b05-4649-9af3-11a2c7722b3f/actiontemplate-hashicorp-vault-ldap-login)

## Conclusion

Happy deployments!
