---
title: Good AWS Lambda deployments
description: Discover Octopus Deploy’s view on what makes a good deployment for an AWS Lambda and how you can get started quickly using GenAI.
author: mark.harrison@octopus.com
visibility: public
published: 2026-01-01-1400
metaImage: img-opinions-on-lambda-deployments.png
bannerImage: img-opinions-on-lambda-deployments.png
bannerImageAlt: Two people are discussing in front of a lambda symbol with speech bubbles and a green check mark.
isFeatured: false
tags:
  - DevOps
  - Product
  - AI 
---

Deploying AWS Lambda functions can be simple, ranging from quick CLI commands to complete infrastructure as code pipelines. That is until you need to do it reliably across multiple environments, with visibility, auditability, and a team of developers involved. 

In this post, I share some strong opinions on what a good Lambda deployment looks like. These are shaped by patterns we've seen work and how, through the Octopus AI Assistant, GenAI can get you started with these opinions in your own projects quicker.

## Use S3 for your Lambda Code

You may have heard the term "build once, deploy often" as one of Octopus' core opinions on software deployment. It'll likely come as little surprise that we think the same is true for a Lambda deployment. Instead of relying on external tools (like the `aws lambda` CLI), we recommend uploading your function application to an S3 storage bucket and referencing it in your deployment templates. This offers some key advantages:

1. Uploading a versioned deployment artifact (zip) keeps your deployment reproducible — you're consistently deploying the same tested artifact, not whatever is built manually.
2. Using S3 avoids the AWS CLI’s `50MB` direct upload limit for Lambda code, which can block larger functions from deploying cleanly.
3. Promoting code across environments is simpler when the artifact doesn’t change between stages.
4. S3-based deployments allow greater flexibility in integrating with CI/CD pipelines. You can have your build process push artifacts to S3 once, or you can have the deployment pipeline upload them to S3.

## Deploy with SAM and CloudFormation

Defining your function and supporting resources in an AWS [SAM](https://aws.amazon.com/serverless/sam/) template and using it as the foundation for your Lambda deployment provides a more scalable and repeatable approach compared to other methods:

1. AWS SAM templates provide a simplified syntax that makes defining Lambda functions, supporting resources, and permissions much easier than writing a large CloudFormation template. 
2. SAM templates are just an extension of AWS CloudFormation, so you still get the reliability and auditability of the CloudFormation engine: Stack events, change sets, and rollback protection are all included.
3. SAM supports AWS best practices, like the logical separation of resources, environment variables, and IAM controls scoped to the Lambda function, which encourages a secure application architecture.

These advantages make SAM more than just a template format. It's a toolkit for building and deploying serverless applications. For example, here's an example SAM file you can use to deploy a Lambda function using the Java runtime:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: AWS Serverless Quarkus - quarkus-amazon-lambda-common-deployment
Globals:
  Api:
    EndpointConfiguration: REGIONAL
    BinaryMediaTypes:
      - "*/*"

Resources:
  Test:
    Type: AWS::Serverless::Function
    Properties:
      Handler: io.quarkus.amazon.lambda.runtime.QuarkusStreamHandler::handleRequest
      Runtime: java17
      CodeUri: function.zip
      MemorySize: 256
      Timeout: 15
      Policies: AWSLambdaBasicExecutionRole
```

After you upload your function to S3, simply change the path to the `CodeUri` property above to the S3 bucket location, and deploy the template.

:::hint
Octopus supports replacing YAML properties at deployment time using the [structured configuration variables](https://octopus.com/docs/projects/steps/configuration-features/structured-configuration-variables-feature) feature.
:::

## Lock down your Lambda

When deploying with AWS SAM, you gain built-in support for security best practices, but you still have to apply them. Here are some practical ways to secure your Lambda functions using SAM:

1. **Use IAM roles of least privilege**: Define fine-grained IAM policies using SAM's `Policies` or `Role` properties. Avoid the use of overly broad policies. Instead, use AWS-managed policies like `AWSLambdaBasicExecutionRole` or, where you need more control, create custom policies that grant the minimum necessary permissions your function needs.

2. **Manage secrets securely**: Instead of hardcoding, reference secrets from core AWS services, like [Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) or [Secrets Manager](https://docs.aws.amazon.com/secretsmanager/?icmpid=docs_homepage_security), directly in your SAM template.

3. **Integrate security scanning in CI/CD**: Before deployment, tools like **cfn-nag** or **Checkov** can be used to (statically) scan your SAM templates for misconfigurations and highlight any insecure approaches. Once your application has been deployed, use tools like **Trivy** to scan any SBOM files for vulnerabilities.

## Accelerate your Lambda projects with GenAI

With the Octopus AI Assistant, you can speed up your AWS Lambda function deployments. Using the **Prompt-based project creation** functionality, you can write a prompt like:

> Create an AWS Lambda Project called "Fast-track SAM" in the project group "AWS"

Octopus will generate a fully structured sample project for AWS Lambda deployments. This ensures your project contains the best practices discussed here, right from the start.

## Conclusion

Building good AWS Lambda deployments is more than zipping code and pushing it to the cloud. By using S3 for deployment artifacts, SAM for deployment management, enforcing strong security practices, and using Octopus' GenAI assistant, your team can move faster without sacrificing quality or control.

Happy deployments!