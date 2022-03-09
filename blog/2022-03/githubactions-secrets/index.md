---
title: Secrets in GitHub Actions
description: As part of our series about Continuous Integration and build servers, learn how to add secrets in GitHub to use with GitHub Actions, plus how to call them in workflows.
author: andrew.corrigan@octopus.com
visibility: public
published: 2022-03-01-1400
metaImage: blogimage-githubconfigurationaddingsecrets-2022.png
bannerImage: blogimage-githubconfigurationaddingsecrets-2022.png
bannerImageAlt: Locked padlock beside an online form input field showing 6 asterisks to represent a password.
isFeatured: false
tags:
  - DevOps
  - CI Series
  - Continuous Integration
  - GitHub Actions
---

In simple terms, a ‘secret’ is data that’s encrypted and hidden from view, but still useable in your projects. Secrets could be sensitive data about your business or a verification method, such as an API key to connect with other services in your pipeline.

GitHub allows you to store secrets at 3 different levels:

- Repository
- Environment
- Organization

In this post, we look at those 3 levels and how to add secrets to them. We also look at how to call secrets in an example workflow.

## Repository secrets

GitHub ties repository secrets to only one repository. They’re available to anyone with the collaborator role to use in actions.

You can store 100 secrets per repository.

### Add a repository secret

1. Open your project’s repository in GitHub and click **Settings** in the top menu.
1. Click **Secrets** in the left menu.
1. Click **New repository secret**.
1. Complete the following fields and click **Add secret**:
   - **Name** - give your secret a suitable name. You can’t use spaces or special characters other than underscores.
   - **Value** - enter the secret, such as your API key.

You can update the value of your secret at any time. Return to **Secrets** in the repository menu and click **Update** on the secret you need to change.

## Environment secrets

Like with Octopus, you can set GitHub to deploy to your pipeline’s deployment targets with environments.

If your repo is public or you have an enterprise license, you can set environment-specific secrets that only work for that one environment.

You can store 100 secrets per environment.

### Add a secret when creating a new environment

1. Open your project’s repository in GitHub and click **Settings** in the top menu.
1. Click **Environments** in the left menu.
1. Click **New environment**.
1. Enter a name for your environment and click **Configure environment**.
1. Set the environment’s protection rules and deployment branches as you need, then click **Add secret**.
1. Complete the following fields and click **Add secret**:
   - **Name** - give your secret a suitable name. You can’t use spaces or special characters other than underscores.
   - **Value** - enter the secret, such as your API key.

### Add a secret to an existing environment

1. Open your project’s repository in GitHub and click **Settings** in the top menu.
1. Click **Environments** in the left menu.
1. Select the environment you need to set a secret for.
1. Click **Add secret** at the bottom of the page.
1. Complete the following fields and click **Add secret**:
   - **Name** - give your secret a suitable name. You can’t use spaces or special characters other than underscores.
   - **Value** - enter the secret, such as your API key.

## Organization secrets

1. Open your organization’s page in GitHub and click **Settings** in the top menu.
1. Click **Secrets** in the left menu.
1. Click **New organization secret**.
1. Complete the following fields and click **Add secret**:
   - **Name** - give your secret a suitable name. You can’t use spaces or special characters other than underscores.
   - **Value** - enter the secret, such as your API key.
   - **Repository access** - select a policy from the dropdown list. You can restrict use to either public or private repos, or manually select them.

## Using secrets in GitHub Actions workflows

:::warning
You can’t use secrets with workflows from a forked repository.

Don’t capture secrets in log files. GitHub will hide secrets if included, but it’s safer not to include them at all.
:::

You use the `secrets` context to call your secret data in workflows. Contexts are what GitHub uses to pull information from its various sources, including workflows, jobs, and steps. See [GitHub’s documentation for the full list of contexts](https://docs.github.com/en/actions/learn-github-actions/contexts).

As an example, if you want to send a package over to an Octopus server for deployment as part of your workflow, you can use the secrets context, along with the secrets’ names, to connect your workflow to Octopus.

```
- name: Push Package to Octopus
   uses: OctopusDeploy/push-package-action@v1.0.0
   with:
       api_key: ${{ secrets.OCTOPUS_TEST_API }}
       packages: ${{ steps.package.outputs.artifacts }}
       server: ${{ secrets.OCTOPUS_PACKAGE_STORE }}
```

To explain what’s happening, this part of the workflow:

1. Triggers an action to push the package to Octopus.
1. Gets the Octopus API key from your secrets store to allow GitHub to pass data to Octopus.
1. Packages your artifacts as per your action’s steps.
1. Gets the hidden package destination from your secrets store.

Here the workflow uses both your API and destination but keeps them hidden from view.

## Conclusion

Secrets are a great security measure that allow you to protect data and connect to services without revealing sensitive information.

## What's next?

We recommend you see GitHub’s documentation for:

- [More information about secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets#reviewing-access-to-organization-level-secrets)
- [GitHub Actions’ contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [GitHub Actions’ syntax](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions#jobsjob_idstepsenv)

If you haven’t already, check out some of our [other blogs in this Continuous Integration series](https://octopus.com/blog/tag/CI%20Series).

Octopus has also built a useful [GitHub Actions workflow generator](https://githubactionworkflows.com/) to help you build a CI pipeline for GitHub.

!include <githubactions-webinar-feb-2022>
  
!include <q1-2022-newsletter-cta>

Happy deployments!
