---
title: Create an RDS instance with CloudFormation
description: Learn how to create an RDS instance with this sample CloudFormation template
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

[Amazon Relational Database Service](https://aws.amazon.com/rds/) (RDS) provides managed databases supporting a number of platforms such as MySQL, MariaDB, Oracle, Postgres, and SQL Server. Almost every custom application requires persistent data storage, and RDS provides a convenient, scalable, and highly available solution.

In this post you'll learn how to deploy an RDS instance with a CloudFormation template.

## An RDS CloudFormation template

The template below deploys an RDS instance into a new VPC with two private subnets:

```yaml
Parameters:
  Tag:
    Type: "String"
  DBUsername:
    Type: "String" 
  DBPassword:
    Type: "String" 
    
Resources: 
  VPC:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: "10.0.0.0/16"
      Tags:
      - Key: "Name"
        Value: !Ref "Tag"
        
  SubnetA:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select 
        - 0
        - !GetAZs 
          Ref: 'AWS::Region'
      VpcId: !Ref "VPC"
      CidrBlock: "10.0.0.0/24"
      
  SubnetB:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select 
        - 1
        - !GetAZs 
          Ref: 'AWS::Region'
      VpcId: !Ref "VPC"
      CidrBlock: "10.0.1.0/24"
      
  RouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref "VPC"
      
  SubnetGroup:
    Type: "AWS::RDS::DBSubnetGroup"
    Properties:
      DBSubnetGroupName: "subnetgroup"
      DBSubnetGroupDescription: "Subnet Group"
      SubnetIds:
      - !Ref "SubnetA"
      - !Ref "SubnetB"

  InstanceSecurityGroup:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupName: "Example Security Group"
      GroupDescription: "RDS traffic"
      VpcId: !Ref "VPC"
      SecurityGroupEgress:
      - IpProtocol: "-1"
        CidrIp: "0.0.0.0/0"
      
  InstanceSecurityGroupIngress:
    Type: "AWS::EC2::SecurityGroupIngress"
    DependsOn: "InstanceSecurityGroup"
    Properties:
      GroupId: !Ref "InstanceSecurityGroup"
      IpProtocol: "tcp"
      FromPort: "0"
      ToPort: "65535"
      SourceSecurityGroupId: !Ref "InstanceSecurityGroup"
      
  RDSCluster:
    Type: "AWS::RDS::DBCluster"
    Properties:
      DBSubnetGroupName: !Ref "SubnetGroup"
      MasterUsername: !Ref "DBUsername"
      MasterUserPassword: !Ref "DBPassword"
      DatabaseName: "products"
      Engine: "aurora"
      EngineMode: "serverless"
      VpcSecurityGroupIds:
      - !Ref "InstanceSecurityGroup"
      ScalingConfiguration:
        AutoPause: true
        MaxCapacity: 16
        MinCapacity: 2
        SecondsUntilAutoPause: 300

Outputs:
  VpcId:
    Description: The VPC ID
    Value: !Ref VPC
```

The VPC, subnets, and route tables were described in a [previous post](https://octopus.com/blog/aws-vpc-private). 

This template then places a number of additional resources into the VPC to support or create the RDS instance.

```yaml
  SubnetGroup:
    Type: "AWS::RDS::DBSubnetGroup"
    Properties:
      DBSubnetGroupName: "subnetgroup"
      DBSubnetGroupDescription: "Subnet Group"
      SubnetIds:
      - !Ref "SubnetA"
      - !Ref "SubnetB"
```