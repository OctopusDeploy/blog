---
title: Deploying AWS Lambdas across environments
description: Learn how to progress Lambda deployments across environments using CloudFormation
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Serverless is the latest iteration in a steady shift away from managing physical or virtual machines. The term "serverless" is a little misleading, because there are still servers running code. But the promise of serverless is that you don't have to think about servers any more. Serverless platforms like AWS Lambda handle creating, destroying, updating and exposing these servers for you, allowing you to focus on running your code.

The serverless model is very compelling for certain types of workloads. Infrequently run code, like a function triggered by a file upload or the addition of a database row, is very convenient to host as a Lambda. You may even find more traditional workloads like website hosting can be accommodated by Lambdas in a cost effective and scalable manner.

Deploying serverless application is trivial these days. CLI tools and IDE plugins allow you to go from code to production with just a single command or click. Eventually though such deployments will need a more robust process to allow changes to be batched together and verified by teams who don't write the code. The traditional solution to these requirements is to have multiple environments, and progress deployments through internal testing environment before they reach production.

In this blog post we'll dive into how multi-environment serverless deployments can be expressed in CloudFormation and progressed in a reliable manner.

## Self-contained and decoupled deployments

For this post we'll consider two styles of serverless deployments.

The self-contained style wraps up all the Lambda functions and the services that trigger them (API Gateway in our case) as a single CloudFormation template creating independent and isolated infrastructure stacks for each environment. 

Self-contained deployments have the following benefits:

* Everything is created, and destroyed, as a group.
* A deployment is progressed to the next environment as a group.
* It is easy to reason about the state of a deployed application, even when the application has multiple Lambdas.

While a self-contained deployment is easy to create, it does have the downside that you can not deploy separate Lambdas independently. Serverless platforms are a natural fit for microservices, and to get the most from microservices you must be able to develop and deploy each microservice independently.

The decoupled style accommodates the flexibility required by microservice deployments. In a decoupled deployment, each Lambda is deployed independently, while still being exposed by a single shared API Gateway.

Decoupled deployments have the following benefits:

* Each Lambda manages its own deployment lifecycle.
* A single, shared API gateway allows Lambdas to interact via relative URLs.
* A shared hostname makes it easier to manage HTTPS certificates.

## Creating a self-contained deployment

A self-contained deployments involves creating a single CloudFormation template with the following resources:

* [AWS::ApiGateway::RestApi](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-restapi.html) - The API Gateway REST API
* [AWS::Logs::LogGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-logs-loggroup.html) - The CloudWatch log group for the Lambda logs
* [AWS::IAM::Role](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html) - The permissions for the Lambda to access the log group
* [AWS::Lambda::Function](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html) - The Lambda function
* [AWS::Lambda::Permission](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-permission.html) - A permission that grants API Gateway the ability to execute a Lambda
* [AWS::ApiGateway::Resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-resource.html) - A resource is a component of the URL path that exposed the Lambda
* [AWS::ApiGateway::Method](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-method.html) - Methods expose HTTP methods on resources
* [AWS::ApiGateway::Stage](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-stage.html) - A stage exposes the URLs defined in the REST API
* [AWS::ApiGateway::Deployment](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-deployment.html) - A deployment captures the state of the REST API configuration as an immutable resource. A deployment is deployed to a stage to expose the API.

### The AWS::ApiGateway::RestApi resource

The `AWS::ApiGateway::RestApi` resource creates a REST API. API Gateway offers multiple kinds of APIs. [REST APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-rest-api.html) were the first, and are the most configurable. [HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api.html) are another option, but we won't use HTTP APIs here.

The snippet below creates the REST API resource:

```json
    "RestApi": {
      "Type": "AWS::ApiGateway::RestApi",
      "Properties": {
        "Description": "My API Gateway",
        "Name": "Self-contained deployment",
        "EndpointConfiguration": {
          "Types": [
            "REGIONAL"
          ]
        }
      }
    }
```

## The AWS::Logs::LogGroup resource

To help debug and monitor or Lambda function, we create a CloudWatch log group.

The name of the log group is based on the name of the Lambda. [This name is not configurable](https://stackoverflow.com/a/39233203/157605), and so we build the log group name from the name of the environment and the name of the service, which combine to create the name of the Lambda:

```JSON
    "AppLogGroup": {
      "Type": "AWS::Logs::LogGroup",
      "Properties": {
        "LogGroupName": { "Fn::Sub": "/aws/lambda/${EnvironmentName}-${ServiceName}" }
      }
    }
```

## The AWS::IAM::Role resource

In order for our Lambda to have permission to interact with the log group, we need an IAM role to grant access.

```JSON
    "IamRoleLambdaExecution": {
      "Type": "AWS::IAM::Role",
      "Properties": {
        "AssumeRolePolicyDocument": {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Effect": "Allow",
              "Principal": {
                "Service": [
                  "lambda.amazonaws.com"
                ]
              },
              "Action": [
                "sts:AssumeRole"
              ]
            }
          ]
        },
        "Policies": [
          {
            "PolicyName": { "Fn::Sub": "${EnvironmentName}-${ServiceName}-policy" },
            "PolicyDocument": {
              "Version": "2012-10-17",
              "Statement": [
                {
                  "Effect": "Allow",
                  "Action": [
                    "logs:CreateLogStream",
                    "logs:CreateLogGroup",
                    "logs:PutLogEvents"
                  ],
                  "Resource": [
                    {
                      "Fn::Sub": "arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/${EnvironmentName}-${ServiceName}*:*"
                    }
                  ]
                }
              ]
            }
          }
        ],
        "Path": "/",
        "RoleName": { "Fn::Sub": "${EnvironmentName}-${ServiceName}-role" },
      }
    }
```

## The AWS::Lambda::Function resource

This is where we create the Lambda itself

```JSON
    "Lambda": {
      "Type": "AWS::Lambda::Function",
      "Properties": {
        "Code": {
          "S3Bucket": "bamboo-support",
          "S3Key": "app.zip"
        },
        "Environment": {
          "Variables": {}
        },
        "FunctionName": { "Fn::Sub": "${EnvironmentName}-${ServiceName}" },
        "Handler": "index.handler",
        "MemorySize": 128,
        "PackageType": "Zip",
        "Role": {
          "Fn::GetAtt": [
            "IamRoleLambdaExecution",
            "Arn"
          ]
        },
        "Runtime": "nodejs12.x",
        "Timeout": 20
      }
    }
```