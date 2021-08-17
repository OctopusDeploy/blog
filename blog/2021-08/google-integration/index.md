---
title: Introducing Google Cloud support in Octopus
description: Learn how Octopus supports your deployments to Google Cloud
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Octopus 2021.2 brings a number of features to support teams deploying to the Google Cloud Platform (GCP). With 2021.2, Octopus has first class support for the AWS, Azure, and Google cloud platforms.

This post introduces the new features in Octopus supporting GCP deployments and provides some tips on how they can be used in your own deployment processes.

## Service account support

Octopus includes a new account type called **Google Cloud Account**. This account securely stores a JSON key generated for a service account:

![](serviceaccount.png "width=500")

## Inheriting VM service accounts

For teams that prefer to manage credentials outside of Octopus, each integration with GCP allows a service account to be inherited from a worker. 

Here is a Google Compute Engine (GCE) VM with an associated service account:

![](vm-service-account.png "width=500")

This VM has a worker Tentacle installed on it and linked to the **GCP** worker pool:

![](worker.png "width=500")

We can then use the service account associated with the VM. Here is an example of a Kubernetes target configured to inherit the credentials of the worker it is run on:

![](assume-service-account.png "width=500")

Note that the target must be configured with the worker pool containing the GCE worker:

![](workerpool.png "width=500")

Operations like health checks an deployments are performed with the credentials assigned to the worker VM, removing the need to store those details in Octopus:

![](healthcheck.png "width=500")

## Google Container Registry support

Google Container Registry (GCR) support has been included in the existing Docker feed type. Define the feed URL as one of the [regional GCR URLs](https://cloud.google.com/container-registry/docs/pushing-and-pulling#add-registry) and supply a service account JSON file for authentication:

:::hint
To query GCR feeds, the **Cloud Resource Manager** API must be enabled. This can be done [here](https://console.developers.google.com/apis/api/cloudresourcemanager.googleapis.com/overview). Without this API, image searches return no results in Octopus.
:::

![](gcr.png "width=500")

Images are then available from GCR repositories:

![](gcr-test.png "width=500")

## Gcloud script support

A new script step called **Run gcloud in a Script** is available to run scripts within the context of a GCP account. Any scripts run as part of this step can take advantage of the fact that Octopus has already performed a login, allowing the script to focus on the operations it needs to perform rather than the boilerplate process of logging in:

![](gcloud-script.png "width=500")

## Terraform support

The Terraform steps include the ability to establish a context with the selected Google credentials, lifting this context from the Terraform template an into the step:

:::hint
Deploying Terraform requires the ability to persist state. A convenient solution for Google users is to [save Terraform state in a Google Cloud Storage (GCS) bucket](https://www.terraform.io/docs/language/settings/backends/gcs.html):

```hcl
terraform {
  backend "gcs" {
    bucket  = "octopus-tf-state" # change this to match the name of your GCS bucket
    prefix  = "terraform/state"
  }
}
```
:::

![](terraform.png "width=500")

## Conclusion

With support for GCP service accounts, GCR feeds, GKE authentication options, a dedicated GCP script step, and Google authentication support in Terraform, Octopus 2021.2 makes it easy to deploy to and manage your GCP infrastructure.

But we would love to hear your feedback! We have a [GitHub issue where you can post a comment](https://github.com/OctopusDeploy/StepsFeedback/issues/7) about how these new features worked, or didn't work, for you, as well an any suggestions for future GCP features. All feedback is welcome, and we would love to know:

* What deployments or operations tasks do you perform to GCP today?
* What are the pain points when deploying to or managing GCP?
* Do you use platforms like Google Kubernetes Engine, Google App Engine, Google Cloud Functions, Google Cloud Run, or others?
* Do the new features work for you? If not, what suggestions do you have that could improve them?