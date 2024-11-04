---
title: Announcing Codefresh steps for Octopus Deploy
description:  You can now integrate your Codefresh builds with deployments in Octopus Deploy thanks to our Codefresh steps. Learn how the new steps improve your deployment experience.
author: susan.pan@octopus.com
visibility: private
published: 2024-11-06-1400
metaImage: codefresh_steps_blog_image_octopus.png
bannerImage: codefresh_steps_blog_image_octopus.png
bannerImageAlt: Stylized version of Octopus steps in Codefresh UI
isFeatured: false
tags: 
  - DevOps
  - Product
---

We're excited to announce the first set of official [Codefresh Steps](https://codefresh.io/steps/) for Octopus Deploy. Codefresh is a modern CI/CD platform built on top of docker. Their CI product, Codefresh Pipelines, are customisable workflows made from a series of steps, each performing their own action. 

In this blog post I'll walk you through the process of creating a release and deploying it to an Octopus environment through Codefresh Pipelines.

## What are Codefresh Steps?

Codefresh steps are the building blocks for creating Codefresh pipelines. They can be chained together in any sequence. Each step is executed within the context of its own docker container, with a default workspace that is [shared between all steps within the pipeline](https://codefresh.io/docs/docs/pipelines/introduction-to-codefresh-pipelines/#sharing-the-workspace-between-build-steps). The shared volume contains data that is accessible to all steps in the pipeline, so files used and accessed in one step can be used in all subsequent steps. 

All of our Octopus Deploy steps use and execute inside the [Octopus-cli docker](https://hub.docker.com/r/octopuslabs/octopus-cli) container. Within each step, we use the Octopus CLI to interact with your Octopus instance to create, push and deploy. We’re thrilled to introduce the addition of the following steps to the [Codefresh Steps marketplace](https://codefresh.io/steps/):

- Login to Octopus Deploy
- [Push packages to Octopus Deploy](https://codefresh.io/steps/step/octopusdeploy-push-package)
- [Create releases in Octopus Deploy](https://codefresh.io/steps/step/octopusdeploy-create-release)
- [Deploy a release in Octopus Deploy](https://codefresh.io/steps/step/octopusdeploy-deploy-release)
- [Deploy a tenanted release in Octopus Deploy](https://codefresh.io/steps/step/octopusdeploy-deploy-release-tenanted)
- [Run a runbook in Octopus Deploy](https://codefresh.io/steps/step/octopusdeploy-run-runbook)
- [Push build information to Octopus Deploy](https://codefresh.io/steps/step/octopusdeploy-push-build-information)


## Step Configuration

Each step will need the following details to run: an octopus instance url and a space name. We recommend using [Codefresh Variables](https://codefresh.io/docs/docs/pipelines/variables/) to set these values, so that each step can use the same variable value. For sensitive values like api keys, remember to [encrypt your variable](https://codefresh.io/docs/docs/pipelines/variables/) to avoid security vulnerabilities. 

## Example Workflow

The following example Codefresh workflow is a common scenario that customers of Octopus Deploy may be familiar with. For this example, we will be configuring the following steps:

- Clone a git repository
- Create a zip of the repository
- Push the package to the [Octopus Deploy built-in feed](https://octopus.com/docs/packaging-applications/package-repositories/built-in-repository)
- Create a new release in project `Example Project` with the pushed package
- Deploy the created release to environment `Development`

Since each Codefresh step calls the Octopus CLI, each step’s required and optional arguments should follow the CLI’s required and optional parameters quite closely. Documentation on each individual step can be found by searching for the step in [Codefresh’s step marketplace](https://codefresh.io/steps/). 

In the following example, we’ve split the steps into 2 [stages](https://codefresh.io/docs/docs/pipelines/stages/): build and push, and deploy. These stages help with the pipeline’s visualization, but have no effect on the order of execution of the steps. By default, the steps should flow sequentially from the first defined in the yaml file to the last. 

```yaml
version: "1.0"

stages:
  - "build and push"
  - "deploy"

steps:
  clone:
	title: "Cloning repository"
	type: "git-clone"
	stage: "build and push"
	repo: "https://github.com/repoName/exampleRepository"
	revision: "main"
	working_directory: "/codefresh/volume"
	credentials:
  	username: ${{GITHUB_USERNAME}}
  	password: ${{GITHUB_PASSWORD}}
  
create-package:
	title: "Create package"
	type: octopusdeploy-create-package
	stage: "build and push"
	arguments:
  	ID: "Hello"
  	VERSION: "1.0.0-${{CF_BUILD_ID}}"
  	BASE_PATH: "/codefresh/volume"
  	OUT_FOLDER: "/codefresh/volume"
  
push-package:
	title: "Push package"
	type: octopusdeploy-push-package
	stage: "build and push"
	arguments:
  	OCTOPUS_API_KEY: ${{OCTOPUS_API_KEY}}
  	OCTOPUS_URL: ${{OCTOPUS_URL}}
  	OCTOPUS_SPACE: "Spaces-42"
  	PACKAGES:
    	- "/codefresh/volume/Hello.1.0.0-${{CF_BUILD_ID}}.zip"
  	OVERWRITE_MODE: 'overwrite'

  create-release:
	type: octopusdeploy-create-release
	title: "Create release"
	stage: "deploy"
	arguments:
  	OCTOPUS_API_KEY: ${{OCTOPUS_API_KEY}}
  	OCTOPUS_URL: ${{OCTOPUS_URL}}
  	OCTOPUS_SPACE: "Spaces-42"
  	PROJECT: "Demo Project"
  	RELEASE_NUMBER: "1.0.0-${{CF_BUILD_ID}}"
  	PACKAGES:
   	- "Hello:1.0.0-${{CF_BUILD_ID}}"
  	RELEASE_NOTES: This is a release note

  deploy:
	type: octopusdeploy-deploy-release
	title: "Deploy release"
	stage: "deploy"
	arguments:
  	OCTOPUS_API_KEY: ${{OCTOPUS_API_KEY}}
  	OCTOPUS_URL: ${{OCTOPUS_URL}}
  	OCTOPUS_SPACE: "Spaces-42"
  	PROJECT: "Demo Project"
  	RELEASE_NUMBER: "1.0.0-${{CF_BUILD_ID}}"
  	ENVIRONMENTS:
    	- "Development"
```
For the last step in the workflow, we’ve defined a step with the name `deploy`. This step uses one of our Codefresh Octopus Deploy steps - the `octopusdeploy-deploy-release` step. We’ve defined an Octopus instance URL and api key using Codefresh variables, and are passing them in as arguments using the Codefresh variable substitution syntax. 

The output of the pipeline build running in Codefresh will look something like this:

![Example pipeline workflow with Octopus Deploy steps](codefresh-pipeline-example.png "width=500")*Example pipeline workflow with Octopus Deploy steps*

## Step output values
Along with sharing files between steps using the shared volume, each step also produces outputs. These output values can then be accessed by any subsequent step in the pipeline. 

| Step | Output | Description |
| -------- | ------- | -------- |
| octopusdeploy-login |  |  | 
| octopusdeploy-create-package | `FILE` | JSON object with property ‘path’ for the path to the created zip file |
| octopusdeploy-push-package | - | This step has no output. | 
| octopusdeploy-create-release | `RELEASE` | JSON object with `channel` and `version` for the created release | 
| octopusdeploy-deploy-release | DEPLOYMENTS | JSON array of objects with `deploymentId` and `serverTaskId` for created deployments | 
| octopusdeploy-deploy-release-tenanted | `DEPLOYMENTS`  | JSON array of objects with `deploymentId` and `serverTaskId` for created deployments | 
| octopusdeploy-run-runbook | `DEPLOYMENTS` | JSON array of objects with `runbookId` and `serverTaskId` for created runbooks | 
| octopusdeploy-push-build-information | - | This step has no output. | 

# Conclusion

Codefresh Steps for Octopus Deploy are yet another way to integrate and streamline your processes. Customize your build pipeline through Codefresh and watch your Octopus deployments fly through the door. 

Happy deployments!

