---
title: Introducing Buildkite Integration
description: Integration between Buildkite and Octopus Deploy is available through plugins that are available for build agents.
author: john.bristowe@octopus.com
visibility: private
published: 2022-02-14-1400
metaImage: to-be-added-by-marketing
bannerImage: to-be-added-by-marketing
bannerImageAlt: Buildkite is now integrated with Octopus Deploy
isFeatured: false
tags: 
  - Continuous Integration
  - Engineering
---

Octopus Deploy now integrates with [Buildkite](https://buildkite.com/), enabling build agents to create releases, push build information, and run runbooks as part of a pipeline. This blog post demonstrates how you can use these plugins to perform various operations with Octopus Deploy as part of a pipeline in Buildkite.

## What is Buildkite?

Buildkite is a platform for running continuous integration pipelines on your infrastructure. This enables it to be fast, secure, and scalable.

Builds are conducted through agents. These are small, reliable and cross-platform build runners controlled through workflows defined in YAML. Agents are also extensible through plugins. These provide additional functionality to the workflows. They can do things like execute steps in Docker containers, read values from a credential store, or add test summary annotations to builds.

## Buildkite Integration with Octopus Deploy

Buildkite integration with Octopus Deploy is supported through the following plugins:

* [create-release-buildkite-plugin](https://github.com/OctopusDeploy/create-release-buildkite-plugin)
* [push-build-information-buildkite-plugin](https://github.com/OctopusDeploy/push-build-information-buildkite-plugin)
* [run-runbook-buildkite-plugin](https://github.com/OctopusDeploy/run-runbook-buildkite-plugin)

These plugins require the [Octopus CLI](https://octopus.com/downloads/octopuscli) to be installed on the Buildkite agent.

## Create Release

In Octopus Deploy, a release is a snapshot of the deployment process and the associated assets (packages, scripts, variables) as they existed when the release was created. The release is given a version number, and you can deploy that release as many times as you need to, even if parts of the deployment process have changed since the release was created (those changes will be included in future releases but not in this version).

When you deploy the release, you are executing the deployment process with all the associated details, as they existed when the release was created.

Creating a release in Octopus Deploy through Buildkite is accomplished by incorporating [create-release-buildkite-plugin](https://github.com/OctopusDeploy/create-release-buildkite-plugin) into your pipeline:

```yaml
steps:
  - label: Create a release in Octopus Deploy 🐙
  - plugins: 
    - OctopusDeploy/create-release#v0.0.1:
        api_key: "${MY_OCTOPUS_API_KEY}"
        project: "HelloWorld"
        server: "${MY_OCTOPUS_SERVER}"
```

It is strongly recommended that you use environment variables for sensitive values such as the API key or server address.

## Push Build Information

When deploying a release, it is useful to know which build produced the artifact, what commits it contained, and which work items it is associated with. The Build information feature allows you to upload information from your build server, manually or with the use of a plugin, to Octopus Deploy.

Build information is associated with a package and includes:

* Build URL: A link to the build which produced the package
* Commits: Details of the source commits related to the build
* Issues: Issue references parsed from the commit messages

Pushing build information to Octopus Deploy from Buildkite can be done through [push-build-information-buildkite-plugin](https://github.com/OctopusDeploy/push-build-information-buildkite-plugin):

```yaml
steps:
  - label: Push build info to Octopus Deploy 🐙
    plugins: 
      - OctopusDeploy/push-build-information#v0.0.1:
          api_key: "${MY_OCTOPUS_API_KEY}"
          packages: "HelloWorld"
          package_version: "1.0.0"
          server: "${MY_OCTOPUS_SERVER}"
```

## Run Runbook

Runbooks automate routine maintenance and emergency operations tasks, such as infrastructure provisioning, database management, and website failover and restoration. Runbooks include all the necessary permissions for the infrastructure they run on, so anybody on the team can execute the runbook, and because they're managed in Octopus there's a complete audit trail. Runbooks can be parameterized with prompted variables so that human interaction is required.

A runbook can be run in Octopus Deploy through Buildkite using the run-runbook-buildkite-plugin:

```yaml
steps:
  - label: Run runbook in Octopus Deploy 🐙
    plugins: 
      - OctopusDeploy/run-runbook#v0.0.1:
          api_key: "${MY_OCTOPUS_API_KEY}"
          environments: "Test"
          project: "Hello World"
          runbook: "Greeting"
          server: "${MY_OCTOPUS_SERVER}"
```

## Conclusion

The integration provided through these plugins represent our initial design and release. Our plan is to build additional plugins and eliminate the dependency on the Octopus CLI by providing integration through bash scripts.

If you're an existing Octopus Deploy customer, check out Buildkite as part of your build pipeline. If you're an existing Buildkite customer, check out Octopus Deploy for deployments. And if you haven't tried either product, consider them both as part of a CI/CD pipeline.

Happy deployments!