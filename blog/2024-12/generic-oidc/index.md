---
title: Generic OpenId Connect accounts
description: Generic OpenId Connect Accounts now support Octopus as a Client to autheticate with services like hashicorp vault, GCP and many more.
author: isaac.calligeros@octopus.com
visibility: public
published: 2024-12-09-1400
metaImage: generic-oidc-banner-img.png
bannerImage: generic-oidc-banner-img.png
bannerImageAlt: Generic OpenId Connect accounts
isFeatured: false
tags: 
  - Product
  - Trust and Security
  - Google Cloud Platform
  - HashiCorp Vault
---

In 2025.1 Octopus is introducing Generic OpenID Connect (OIDC) accounts. This account supports Octopus acting as a client by contributing a JWT as a variable to a deployment. This offers a flexible way to authenticate with third-party systems that support OAuth 2.0 and JSON Web Tokens (JWTs). In this post, we’ll explore how Generic OIDC accounts work and their practical applications for systems like HashiCorp Vault, and Google Cloud using Workload Identity Federation.

## Creating a Generic OIDC Account
These accounts allow two fields to be configured, the audience which is specific to the service you are integrating with and the subject generation. For an in depth explanation of how subjects are generated and what is required for OAuth flows please see our [OpenId Connect docs](https://octopus.com/docs/infrastructure/accounts/openid-connect).
![Generic OpenId Connect account creation](generic-oidc-account-creation.png).

## Authenticating with HashiCorp Vault
HashiCorp Vault is a powerful tool for managing secrets and identities, and it supports authentication via OIDC.
A vault instance can be configured to enable OAuth Jwt authentication with the following commands. In this example we include the basic jwt configuration required, for full details on configuration see the [HashiCorp Vault Jwt docs](https://developer.hashicorp.com/vault/docs/auth/jwt).

``` bash
vault auth enable jwt
vault write auth/jwt/config oidc_discovery_url="https://your-instance-url" bound_issuer="https://your-instance-url"
vault write auth/jwt/role/jwttest role_type="jwt" bound_audiences="<audience>" bound_subject="account:<account-slug>" user_claim="<audience>" policies="default" ttl="1h"
```

This token can be used on our [HashiCorp Vault login step](https://octopus.com/integrations/hashicorp-vault/hashicorp-vault-jwt-login) by setting the Jwt Token value to `#{<your account variable>.OpenIdConnect.Jwt}`. 

Token exchange can also be handled directly in the vault CLI:
```bash
vault write auth/jwt/login role="jwttest" jwt="#{[your account variable].OpenIdConnect.Jwt}"
```

This approach extends beyond Vault to any third-party service that supports OAuth 2.0 JWT flows. Whether you’re integrating with APIs, databases, or custom applications, the Generic OIDC account offers a flexible and secure solution for authentication flows.

## Using Generic OIDC Accounts with Google Cloud
Google Cloud’s Workload Identity Federation allows secure access to GCP resources using OIDC tokens without the need for static service account keys. Generic OIDC accounts in Octopus integrate with the existing run a gcloud script step, allowing the account to be selected on the step.

To use a Generic OIDC account with Google Cloud the subject generator and audience have to match your Workload Identity Federation configuration. The default audience can be found on the provider and follows the format: `https://iam.googleapis.com/projects/{project-id}/locations/global/workloadIdentityPools/{pool-id}/providers/{provider-id}`. Further details on the configuration of Workload Identity Federations can be found [here](https://cloud.google.com/iam/docs/workload-identity-federation-with-other-providers). This authentication flow will sign in as the provided Workload Identity principal. To impersonate a service account the Workload Identity Principal requires the Workload Identity User permissions. These sessions are configured to expire after one hour.


## Conclusion 

Generic OIDC accounts bring flexibility to your deployment pipelines, enabling seamless authentication with systems that support OAuth 2.0 Jwt flows like HashiCorp Vault, Google Cloud and many more. Whether you’re managing secrets, accessing cloud resources, or integrating with custom systems, these accounts provide a simple, secure, and scalable solution, removing the need for static credentials.

We hope this post helps you unlock new possibilities with Octopus Deploy and OIDC-based authentication. If you have any questions or feedback, we’d love to hear from you.

Happy deployments!