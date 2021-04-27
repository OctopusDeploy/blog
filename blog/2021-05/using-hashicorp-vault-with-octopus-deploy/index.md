---
title: Using HashiCorp Vault with Octopus Deploy
description: Introducing new step templates to allow Secrets stored in HashiCorp Vault to be used in deployments or runbooks.
author: mark.harrison@octopus.com
visibility: private
published: 2022-05-01
metaImage: octopus-step-template.png
bannerImage: octopus-step-template.png
tags:
 - Product
 - Security
---

![Using HashiCorp Vault with Octopus Deploy](octopus-step-template.png)

Octopus has supported the concept of [sensitive variables](https://octopus.com/docs/projects/variables/sensitive-variables) since [Octopus 2.0](https://octopus.com/blog/new-in-2.0/sensitive-variables). However, we're often asked by customers about support for Secret managers. You only have to do a [quick search](https://www.google.com/search?q=secret+managers) to see that all of the major cloud providers have secret managers. In addition, there are a number of third-party products in the marketplace such as [HashiCorp Vault](https://www.vaultproject.io/).

Storing sensitive values in Octopus Deploy solves many problems; the trade-off is, if your organisation has standardised on a secrets manager, that might mean storing sensitive values twice, making secrets management more complicated.

In this post, I walk through a number of new [HashiCorp Vault step templates](https://library.octopus.com/listing/hashicorp%20vault) to enable you to retrieve secrets from Vault for use in your deployments or runbooks.


<h2>In this post</h2>

!toc

## Introduction

This post assumes some familiarity with [Custom step templates](https://octopus.com/docs/projects/custom-step-templates) and the Octopus [Community Library](https://octopus.com/docs/projects/community-step-templates). If you want to learn more about these, my colleague Ryan Rousseau wrote up an excellent [two-part series](https://octopus.com/blog/creating-an-octopus-deploy-step-template) on how to create your own step template and publish it to the Library.

In addition, this post doesn't cover how to configure a Vault server, or its main concepts in great detail.

The step templates cover both [Vault authentication](https://www.vaultproject.io/docs/concepts/auth) and Secret retrieval for both versions `1` and `2` of the [Key-Value (kv)](https://www.vaultproject.io/docs/secrets/kv) Secrets Engine. 

All of the step templates make use of the Vault [HTTP API](https://www.vaultproject.io/api-docs) so no additional dependencies are required to use them, except being able to connect to your Vault server of course!

## Authentication

Authentication with Vault can be achieved with a number of different auth methods. As Octopus executes both runbooks and deployments on deployment targets (machines), auth methods that are suited to this non-interactive mode are preferable. The following methods are included:

- [LDAP login](#ldap-login)
- [AppRole login](#approle-login)

Once you have authenticated with Vault, a [token](https://www.vaultproject.io/docs/concepts/tokens) is generated that can be used in further interactions with Vault. The primary use in our case is to retrieve secrets from a Secrets Engine.

### LDAP login #{ldap-login}

The [HashiCorp Vault - Login with LDAP](https://library.octopus.com/step-templates/de807003-3b05-4649-9af3-11a2c7722b3f/actiontemplate-hashicorp-vault-ldap-login) step template authenticates with a Vault Server using the [LDAP](https://www.vaultproject.io/docs/auth/ldap) auth method. This is useful as it allows Vault integration without having to duplicate username or password configuration.

Once authenticated, the `client_token` from the Vault response will be made available as a [sensitive output variable](https://octopus.com/docs/projects/variables/output-variables#sensitive-output-variables) named `LDAPAuthToken` for use in other step templates.

#### LDAP login parameters

The step template has the following parameters:

- `Vault Server URL`: The URL of the Vault instance you are connecting to, including the port (The default is `8200`).
- `API version`: All Vault [API routes](https://www.vaultproject.io/api-docs) are prefixed with a version e.g. `/v1/`.
- `LDAP Auth Login path`: The path that the [LDAP method is mounted at](https://www.vaultproject.io/api-docs/auth/ldap). The default is `/auth/ldap`.
- `Username`: The LDAP username.
- `Password`: The LDAP password.

![Parameters for the Vault LDAP login step](vault-login-step-parameters.png)

### AppRole login #{approle-login}

### Response wrapping #{response-wrapping}

:::hint
Link to Vault AppRole best practices
:::

### Get wrapped Secret ID #{get-wrapped-secretid}

### Unwrap Secret ID #{unwrap-secretid}

### Unwrap Secret ID and Login #{unwrap-secretid-login}

## Retrieving secrets #{retrieving-secrets}

### Retrieve KV (version 1) secrets #{retrieve-kv-v1-secrets}

### Retrieve KV (version 1) secrets #{retrieve-kv-v2-secrets}

## Conclusion


