---
title: Canary deployments with ECS
description: Learn how to use an external deployment controller to perform Canary deployments in ECS
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Canary deployments are a popular pattern that allow you to progressively roll out a new version of your application to an increasing number of end users. By watching for errors or undesirable effects from the new version during the rollout, it is possible to catch and revert production errors before they impact the majority of your users.

ECS has native support for [rolling updates](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-ecs.html), where tasks in a service are progressively, but automatically, updated with a new version of the application. By integrating with CodeDeploy, it is possible to perform what ECS refers to as a [Blue/Green deployment](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-bluegreen.html), although this deployment option can be configured to perform a Canary deployments that shifts the traffic to the new version. You can even [create your own deployment strategy](https://docs.aws.amazon.com/cli/latest/reference/deploy/create-deployment-config.html), but you are limited to a time based canary rule, which is:

> A configuration that shifts traffic from one version of a Lambda function or ECS task set to another in two increments.

Or a time based linear rule, which is:

> A configuration that shifts traffic from one version of a Lambda function or ECS task set to another in equal increments, with an equal number of minutes between each increment.

There are times though when the decision to shift more traffic to the canary deployment is not something you can easily determine over a fixed period of time. For example, you may need to have a person make the decision to move forward with a canary deployment based on a range of inputs like support requests, errors in logs, resource usage. This kind of manual intervention in the deployment requires more flexibility than the Blue/Green strategy exposed by ECS.

Fortunately, [ECS can defer the decision to progress a deployment to an external system](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-external.html). It requires some work to set up, but is incredibly flexible.

In this blog post we'll look at how to manage an ECS canary deployment with Octopus.

## ECS CloudFormation resources

Our ECS deployment will be created and managed via CloudFormation, and will make use of the following resources:

* [AWS::ECS::TaskDefinition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-taskdefinition.html) - Task definitions configure the containers to be executed by ECS.
* [AWS::ECS::Service](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-service.html) - A service keeps one or more instances of a task definition (or many task definitions, when using task sets) running in the ECS cluster.
* [AWS::ECS::TaskSet](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-taskset.html) - A service can contain many task sets, with each task set configured with its own task definition. Multiple task sets allow a single service to manage tasks created from multiple task definitions.
* [AWS::ElasticLoadBalancingV2::Listener](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-listener.html) - A listener defines the port and protocol that it will receive load balancer traffic on.
* [AWS::ElasticLoadBalancingV2::ListenerRule](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-listenerrule.html) - A listener rule defines the high level rules, such as path, query string, header matching etc, that must be satisfied to deliver traffic to a target group.
* [AWS::ElasticLoadBalancingV2::TargetGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-targetgroup.html) - A target group binds downstream services, like ECS clusters, to a load balancer listener rule.

The resulting architecture looks like this:

![](architecture.png "width=500")

## The sample application

To demonstrate a canary deployment, we'll use the Docker image created from the code at https://github.com/OctopusSamples/DockerHelloWorldWithVersion. This is a simple "hello world" Node.js application that also prints the value of the `APPVERSION` environment variable in response to a HTTP request.

## The AWS::ECS::TaskDefinition resource

The task definition configures the sample application to listen to traffic on port 4000. It also defines an environment variable called `APPVERSION`, which will be displayed in the response.

We'll update the `APPVERSION` environment variable as a way of simulating new application versions being deployed.

Here is the first task definition:

```json
    "MyTask1": {
      "Type": "AWS::ECS::TaskDefinition",
      "Properties": {
        "ContainerDefinitions": [
          {
            "Cpu": 256,
            "Image": "octopussamples/helloworldwithversion",
            "Memory": 512,
            "MemoryReservation": 128,
            "Name": "mycontainer",
            "Environment": [
              {
                "Name": "APPVERSION",
                "Value": "1.0.0"
              }
            ],
            "PortMappings": [
              {
                "ContainerPort": 4000,
                "HostPort": 4000,
                "Protocol": "tcp"
              }
            ]
          }
        ],
        "Cpu": "256",
        "Family": "mytask",
        "Memory": "512",
        "RequiresCompatibilities": [
          "FARGATE"
        ],
        "NetworkMode": "awsvpc"
      }
    }
```

Here is the second task definition. It is the same as above, but with the `APPVERSION` value incremented:

```json
    "MyTask2": {
      "Type": "AWS::ECS::TaskDefinition",
      "Properties": {
        "ContainerDefinitions": [
          {
            "Cpu": 256,
            "Image": "octopussamples/helloworldwithversion",
            "Memory": 512,
            "MemoryReservation": 128,
            "Name": "mycontainer",
            "Environment": [
              {
                "Name": "APPVERSION",
                "Value": "1.0.1"
              }
            ],
            "PortMappings": [
              {
                "ContainerPort": 4000,
                "HostPort": 4000,
                "Protocol": "tcp"
              }
            ]
          }
        ],
        "Cpu": "256",
        "Family": "mytask",
        "Memory": "512",
        "RequiresCompatibilities": [
          "FARGATE"
        ],
        "NetworkMode": "awsvpc"
      },
      "DependsOn": "MyTask1"
    }
```

## The AWS::ECS::Service resource

The service will ensure the desired number of tasks are run, and continue to run. Typically we would configure the service to reference a task definition directly, but in our case we will use task sets to link a service to a task definition. This means our service is quite sparse:

```json
"MyService": {
      "Type": "AWS::ECS::Service",
      "Properties": {
        "Cluster": "arn:aws:ecs:us-east-1:968802670493:cluster/mattctest",
        "ServiceName": "myservice",
        "DeploymentController": {
          "Type": "EXTERNAL"
        }
      }
    }
```

## The AWS::ElasticLoadBalancingV2::TargetGroup resources

```json
    "GreenTargetGroup": {
      "Type": "AWS::ElasticLoadBalancingV2::TargetGroup",
      "Properties": {
        "HealthCheckEnabled": true,
        "HealthCheckIntervalSeconds": 30,
        "HealthCheckPath": "/",
        "HealthCheckPort": "4000",
        "HealthCheckProtocol": "HTTP",
        "HealthCheckTimeoutSeconds": 10,
        "HealthyThresholdCount": 5,
        "Matcher": {
          "HttpCode": "200"
        },
        "Name": "OctopusGreenTargetGroup",
        "Port": 4000,
        "Protocol": "HTTP",
        "TargetType": "ip",
        "UnhealthyThresholdCount": 5,
        "VpcId": "vpc-04fb5b2e72c17ca68"
      }
    }
```

```json
    "BlueTargetGroup": {
      "Type": "AWS::ElasticLoadBalancingV2::TargetGroup",
      "Properties": {
        "HealthCheckEnabled": true,
        "HealthCheckIntervalSeconds": 30,
        "HealthCheckPath": "/",
        "HealthCheckPort": "4000",
        "HealthCheckProtocol": "HTTP",
        "HealthCheckTimeoutSeconds": 10,
        "HealthyThresholdCount": 5,
        "Matcher": {
          "HttpCode": "200"
        },
        "Name": "OctopusBlueTargetGroup",
        "Port": 4000,
        "Protocol": "HTTP",
        "TargetType": "ip",
        "UnhealthyThresholdCount": 5,
        "VpcId": "vpc-04fb5b2e72c17ca68"
      }
    }
```

## The AWS::ECS::TaskSet resources

```json
    "GreenTaskSet": {
      "Type": "AWS::ECS::TaskSet",
      "Properties": {
        "Cluster": "arn:aws:ecs:us-east-1:968802670493:cluster/mattctest",
        "ExternalId": "OctopusGreenStack",
        "LaunchType": "FARGATE",
        "NetworkConfiguration": {
          "AwsvpcConfiguration": {
            "AssignPublicIp": "ENABLED",
            "SecurityGroups": [
              "sg-043789abf52c12d9a"
            ],
            "Subnets": [
              "subnet-0af41f8e0404d7b23",
              "subnet-0c2515119bdf77d4c",
              "subnet-09d1a3362fac596a9"
            ]
          }
        },
        "LoadBalancers": [
          {
            "ContainerName": "mycontainer",
            "ContainerPort": 4000,
            "TargetGroupArn": {
              "Ref": "GreenTargetGroup"
            }
          }
        ],
        "Scale": {
          "Unit": "PERCENT",
          "Value": 100
        },
        "Service": "myservice",
        "TaskDefinition": {"Ref": "MyTask2"}
      },
      "DependsOn": [
        "MyService",
        "GreenTargetGroup"
      ]
    }
```

```json
    "BlueTaskSet": {
      "Type": "AWS::ECS::TaskSet",
      "Properties": {
        "Cluster": "arn:aws:ecs:us-east-1:968802670493:cluster/mattctest",
        "ExternalId": "OctopusBlueStack",
        "LaunchType": "FARGATE",
        "NetworkConfiguration": {
          "AwsvpcConfiguration": {
            "AssignPublicIp": "ENABLED",
            "SecurityGroups": [
              "sg-043789abf52c12d9a"
            ],
            "Subnets": [
              "subnet-0af41f8e0404d7b23",
              "subnet-0c2515119bdf77d4c",
              "subnet-09d1a3362fac596a9"
            ]
          }
        },
        "LoadBalancers": [
          {
            "ContainerName": "mycontainer",
            "ContainerPort": 4000,
            "TargetGroupArn": {
              "Ref": "BlueTargetGroup"
            }
          }
        ],
        "Scale": {
          "Unit": "PERCENT",
          "Value": 100
        },
        "Service": "myservice",
        "TaskDefinition": {"Ref": "MyTask1"}
      },
      "DependsOn": [
        "MyService",
        "BlueTargetGroup"
      ]
    }
```

## The AWS::ElasticLoadBalancingV2::Listener resource

```json
    "MyListener": {
      "Type": "AWS::ElasticLoadBalancingV2::Listener",
      "Properties": {
        "DefaultActions": [
          {
            "FixedResponseConfig": {
              "StatusCode": "404"
            },
            "Order": 1,
            "Type": "fixed-response"
          }
        ],
        "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-1:968802670493:loadbalancer/app/mattctest/3a1496378bd20439",
        "Port": 80,
        "Protocol": "HTTP"
      }
    }
```

## The AWS::ElasticLoadBalancingV2::ListenerRule resource

```json
    "MyListenerRule": {
      "Type": "AWS::ElasticLoadBalancingV2::ListenerRule",
      "Properties": {
        "Actions": [
          {
            "ForwardConfig": {
              "TargetGroups": [
                {
                  "TargetGroupArn": {
                    "Ref": "GreenTargetGroup"
                  },
                  "Weight": 100
                },
                {
                  "TargetGroupArn": {
                    "Ref": "BlueTargetGroup"
                  },
                  "Weight": 0
                }
              ]
            },
            "Order": 1,
            "Type": "forward"
          }
        ],
        "Conditions": [
          {
            "Field": "path-pattern",
            "PathPatternConfig": {
              "Values": [
                "/*"
              ]
            }
          }
        ],
        "ListenerArn": {
          "Ref": "MyListener"
        },
        "Priority": 10
      },
      "DependsOn": [
        "MyListener",
        "GreenTargetGroup",
        "BlueTargetGroup"
      ]
    }
```