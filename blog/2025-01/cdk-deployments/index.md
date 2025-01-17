---
title: "Repeatable CDK deployments with Octopus"
description: "Learn how to orchestrate CDK deployments with Octopus Deploy."
author: matthew.casperson@octopus.com
visibility: public
published: 2025-01-13-1400
metaImage: blogimage-asg-bluegreen.png
bannerImage: blogimage-asg-bluegreen.png
bannerImageAlt: Hexagons falling into an AWS-styled cube
isFeatured: false
tags:
- DevOps
- AWS
---

Cloud providers have invested a lot of effort into streaming the development and deployment of cloud native applications. Developers used to be responsible for provisioning infrastructure and developing their applications separately, but this made local testing difficult, and often forced developers to have a deep understanding of network configuration like VPCs and subnets.

The AWS Serverless Application Model (SAM) introduced tooling and streamlined CloudFormation syntax to automate the testing and deployment of Lambdas. While the CLI tooling improved developers local testing experience, it was not mandatory, as the application code and SAM CloudFormation templates were still separate entities could be deployed like any other CloudFormation stack. The post [Deploying AWS SAM templates with Octopus](https://octopus.com/blog/aws-sam-and-octopus) describes how to deploy a SAM application using standard Octopus Deploy features such as uploading files to S3 and deploying CloudFormation stacks.

The AWS Cloud Development Kit (CDK) takes this concept further by allowing developers to define their infrastructure and applications using a programming language like TypeScript or Python. While conceptually similar to AWS SAM, CDK approaches the process of developing and deploying applications differently enough that we must adopt a new way to deploy CDK applications with Octopus Deploy.

## Differences between SAM and CDK

With SAM, you are responsible for writing CloudFormation templates directly. The SAM CloudFormation syntax is simplified to remove much of the boilerplate configuration required to deploy a Lambda, but you're still writing JSON or YAML files.

CDK also uses CloudFormation to deploy infrastructure and applications, but [the generated CloudFormation templates are treated as an implementation detail](https://docs.aws.amazon.com/cdk/v2/guide/best-practices.html#best-practices-code):

> Treat AWS CloudFormation as an implementation detail that the AWS CDK uses for robust cloud deployments, not as a language target. You're not writing AWS CloudFormation templates in TypeScript or Python, you're writing CDK code that happens to use CloudFormation for deployment.

SAM keeps application code and CloudFormation infrastructure templates separate concepts (with [inline code](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-function.html#sam-function-inlinecode) being the exception to the rule). While it is common to have a SAM template stored alongside the application code, each can be deployed independently.

[CDK combines the application code and infrastructure into a single codebase](https://docs.aws.amazon.com/cdk/v2/guide/best-practices.html#best-practices-code):

> In addition to generating AWS CloudFormation templates for deploying infrastructure, the AWS CDK also bundles runtime assets like Lambda functions and Docker images and deploys them alongside your infrastructure. This makes it possible to combine the code that defines your infrastructure and the code that implements your runtime logic into a single construct. It's a best practice to do this. These two kinds of code don't need to live in separate repositories or even in separate packages.

The different approaches take by SAM and CDK mean we need to adopt a new way to deploy CDK applications with Octopus Deploy.

## Deploying CDK applications

Because CDK provides a self-contained package for deploying application code and infrastructure, the most reliable way to deploy CDK packages is to use the cdk CLI. If we look at the [sample GitHub Actions workflows provided by AWS for deploying CDK packages](https://github.com/aws-samples/aws-cdk-golang-serverless-cicd-github-actions/blob/f3be84fde0e8b378ab1f9ce37b3c596f8071b7e1/.github/workflows/reusable-cd.yaml#L67), we can see that they call `cdk deploy` directly:

```yaml
- name: Deploy CDK Stack
  run: |
    cd ${{ inputs.FILE_PATH }}
    ENV=${{ inputs.Environment }} IMAGETAG=${{ inputs.DOCKER_TAG}} ECR_ARN="arn:aws:ecr:${{ inputs.AWS_REGION }}:${{ inputs.AWS_ACCOUNT_ID }}:repository/${{ inputs.ECR_REPOSITORY }}" cdk deploy --require-approval never
```

