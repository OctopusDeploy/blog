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

## The sample applications

We'll deploy two very simple Lambda applications in this example.

The first is written in Go and can be found at https://github.com/OctopusSamples/GoLambdaExample. 

The second is written in Node.js and can be found at https://github.com/OctopusSamples/NodeLambdaExample.

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
    "AppLogGroupOne": {
      "Type": "AWS::Logs::LogGroup",
      "Properties": {
        "LogGroupName": { "Fn::Sub": "/aws/lambda/${EnvironmentName}-NodeLambda" }
      }
    }
```

## The AWS::IAM::Role resource

In order for our Lambda to have permission to interact with the log group, we need an IAM role to grant access:

```JSON
    "IamRoleLambdaOneExecution": {
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
            "PolicyName": { "Fn::Sub": "${EnvironmentName}-NodeLambda-policy" },
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
                      "Fn::Sub": "arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/${EnvironmentName}-NodeLambda*:*"
                    }
                  ]
                }
              ]
            }
          }
        ],
        "Path": "/",
        "RoleName": { "Fn::Sub": "${EnvironmentName}-NodeLambda-role" },
      }
    }
```

## The AWS::Lambda::Function resource

This is where we create the Lambda itself. It will execute using the IAM role created above:

```JSON
    "LambdaOne": {
      "Type": "AWS::Lambda::Function",
      "Properties": {
        "Code": {
          "S3Bucket": "deploy-lambda-blog",
          "S3Key": "nodelambdaexample.zip"
        },
        "Environment": {
          "Variables": {}
        },
        "FunctionName": { "Fn::Sub": "${EnvironmentName}-NodeLambda" },
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

## The AWS::Lambda::Permission resource

In order for the REST API to be able to execute the Lambda, it needs to be granted access.

There are two ways to grant API Gateway access to a Lambda: [IAM roles or resource-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_compare-resource-policies.html). We have chosen to use resource-based policies here, as this is how the API Gateway console grants itself access to a Lambda if you integrate the two systems manually:

```JSON
    "LambdaOnePermissions": {
      "Type": "AWS::Lambda::Permission",
      "Properties": {
        "FunctionName": {
          "Fn::GetAtt": [
            "LambdaOne",
            "Arn"
          ]
        },
        "Action": "lambda:InvokeFunction",
        "Principal": "apigateway.amazonaws.com",
        "SourceArn": {
          "Fn::Join": [
            "",
            [
              "arn:",
              {
                "Ref": "AWS::Partition"
              },
              ":execute-api:",
              {
                "Ref": "AWS::Region"
              },
              ":",
              {
                "Ref": "AWS::AccountId"
              },
              ":",
              {"Ref": "RestApi"},
              "/*/*"
            ]
          ]
        }
      }
    }
```

# The AWS::ApiGateway::Resource resources

The elements in a path exposed by an API gateway are called resources. For example, the URL path of `/vehicles/cars/car1` is made up of three resources: `vehicles`, `cars`, and `car1`.

Resources can match the entire remaining path with the `{proxy+}` syntax.

The template below creates two resources that combine to match the path `/nodefunc/{proxy+}`:

```JSON
    "ResourceOne": {
      "Type": "AWS::ApiGateway::Resource",
      "Properties": {
        "RestApiId": {"Ref": "RestApi"},
        "ParentId": { "Fn::GetAtt": ["RestApi", "RootResourceId"] },
        "PathPart": "nodefunc"
      }
    },
    "ResourceTwo": {
      "Type": "AWS::ApiGateway::Resource",
      "Properties": {
        "RestApiId": {"Ref": "RestApi"},
        "ParentId": {
          "Ref": "Resource0"
        },
        "PathPart": "{proxy+}"
      }
    }
```

# The AWS::ApiGateway::Method resources

We need to expose a method in order to respond to a HTTP request on a resource.

When calling a Lambda, API gateway has the option of using [proxy integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html).

Prior to the proxy integration option, calling a Lambda from API Gateway involved a significant amount of boilerplate configuration to bridge the world of HTTP requests and Lambda executions. HTTP requests expose a range of information in the requested URL, query strings, headers, and HTTP body. A HTTP response can then include a status code, headers, and a body. On the other side we have a Lambda, which accepts a single object as input and returns a single object as output. This means that API Gateway had to be configured to marshall the various inputs in a HTTP call into a single object when calling a Lambda, and unmarshall the Lambda's response into the HTTP response. In practice, this same configuration was done for every method, resulting in a lot of duplicated effort.

Proxy integrations were created to provide a tick-box solution for this common problem. With proxy integration enabled, API Gateway marshalls the incoming HTTP request into a [standard object to be consumed by the Lambda](https://docs.amazonaws.cn/en_us/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format), and expects an [object of a certain shape to be returned](https://docs.amazonaws.cn/en_us/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format), from which the HTTP response is generated.

Another common issue with API Gateway is the distinction between calling a URL with the trailing slash and without. In our example, calling `/nodefunc` will call methods attached to the first resources, and `/nodefunc/` calls the methods attached to the second resource. 

Finally note that we reference the build the Lambda URL from a stage variable. This allows us to call a distinct Lambda from each API Gateway stage.

The more "traditional" approach is to match an API Gateway stage to a Lambda alias, with both stages and aliases representing a progression through environments. However, Lambda aliases have significant limitation which I believe make them fundamentally unsuitable to solve the common use cases for environmental progression. You can read more about this in the blog post [Why you should not use Lambda aliases to define environments](https://octopus.com/blog/multi-environment-lambda-deployments).

So this methods here are configured to allow different Lambdas per environment. This will set us up later as we split this self-contained deployment into a decoupled deployment.

Below are the two methods, with proxy integration, referencing Lambdas via the stage variable `StageName`:

```JSON
    "LambdaOneMethodOne": {
      "Type": "AWS::ApiGateway::Method",
      "Properties": {        
        "HttpMethod": "ANY",
        "Integration": {          
          "IntegrationHttpMethod": "POST",          
          "TimeoutInMillis": 20000,
          "Type": "AWS_PROXY",
          "Uri": {
            "Fn::Join": [
              "",
              [
                "arn:",
                {
                  "Ref": "AWS::Partition"
                },
                ":apigateway:",
                {
                  "Ref": "AWS::Region"
                },
                ":lambda:path/2015-03-31/functions/",
                "arn:aws:lambda:",
                {
                  "Ref": "AWS::Region"
                },
                ":",
                {
                  "Ref": "AWS::AccountId"
                },
                ":function:${stageVariables.StageName}-NodeLambda",
                "/invocations"
              ]
            ]
          }
        },        
        "ResourceId": {
          "Ref": "Resource0"
        },
        "RestApiId": {"Ref": "RestApi"}
      }
    },
    "LambdaOneMethodTwo": {
      "Type": "AWS::ApiGateway::Method",
      "Properties": {        
        "HttpMethod": "ANY",
        "Integration": {          
          "IntegrationHttpMethod": "POST",          
          "TimeoutInMillis": 20000,
          "Type": "AWS_PROXY",
          "Uri": {
            "Fn::Join": [
              "",
              [
                "arn:",
                {
                  "Ref": "AWS::Partition"
                },
                ":apigateway:",
                {
                  "Ref": "AWS::Region"
                },
                ":lambda:path/2015-03-31/functions/",
                "arn:aws:lambda:",
                {
                  "Ref": "AWS::Region"
                },
                ":",
                {
                  "Ref": "AWS::AccountId"
                },
                ":function:${stageVariables.StageName}-NodeLambda",
                "/invocations"
              ]
            ]
          }
        },        
        "ResourceId": {
          "Ref": "ResourceTwo"
        },
        "RestApiId": {"Ref": "RestApi"}
      }
    }
```

## The AWS::ApiGateway::Deployment resource

The resources and methods created above have been created in a kind of working stage. This configuration is not exposed to traffic until it is captured in a deployment, and promoted to a stage.

The deployment is created below. The description of the deployment matches the Octopus release version number. We'll take advantage of this once we move to a decoupled deployment model.

Note that we attach a random string to the resource name. Deployments are immutable, and so each time this CloudFormation template is published to a stack, we create a new deployment resource:

```JSON
    "Deployment93b7b8be299846a5b609121f6fca4952": {
      "Type": "AWS::ApiGateway::Deployment",
      "Properties": {
        "RestApiId": {"Ref": "RestApi"},
        "Description": "Octopus Release #{Octopus.Release.Number}"
      },
      "DependsOn": [
        "LambdaOneMethodOne",
        "LambdaOneMethodTwo"
      ]
    }
```

## The AWS::ApiGateway::Stage resource

The final step in this journey is to create a stage. It is here that we create the stage variable `StageName` that the methods referenced, and "promote" the working stage by referencing the deployment resource:


```json
    "Stage": {
      "Type": "AWS::ApiGateway::Stage",
      "Properties": {
        "CanarySetting": {
          "DeploymentId": {"Ref": "Deployment93b7b8be299846a5b609121f6fca4952"},
          "PercentTraffic": 0
        },
        "DeploymentId": {"Ref": "Deployment93b7b8be299846a5b609121f6fca4952"},
        "RestApiId": {"Ref": "RestApi"},
        "StageName": {"Ref": "${EnvironmentName}"},
        "Variables": {
          "StageName": {"Ref": "${EnvironmentName}"}
        }
      }
    }
```