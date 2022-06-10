---
title: Creating an RDS instance with CloudFormation
description: As part of our Runbooks series, learn how to create an RDS instance with this sample CloudFormation template.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-06-06-1400
metaImage: blogimage-createrdsinstancecf-2022.png
bannerImage: blogimage-createrdsinstancecf-2022.png
bannerImageAlt: A crane lowering one level of a database server onto three levels already stacked up below it.
isFeatured: false
tags:
 - DevOps
 - Runbooks Series
 - AWS
 - CloudFormation
---
 
[Amazon Relational Database Service](https://aws.amazon.com/rds/) (RDS) implements managed databases supporting a number of platforms such as MySQL, MariaDB, Oracle, Postgres, and SQL Server. Almost every custom application requires persistent data storage, and RDS provides a convenient, scalable, and highly available solution.

In this post, you learn how to deploy an RDS instance with a CloudFormation template.

## An RDS CloudFormation template

The template below deploys an RDS instance into a new VPC with 2 private subnets:

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

The VPC, subnets, and route tables were described in a [previous post](https://octopus.com/blog/aws-vpc-private). This template then places a number of additional resources into the VPC to support or create the RDS instance.

RDS instances need at least 2 subnets to achieve high availability. These subnets are grouped together in an [AWS::RDS::DBSubnetGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbsubnet-group.html) resource:

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

Network access to the RDS instance is defined in a security group, represented by an [AWS::EC2::SecurityGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html) resource. This security group allows all outbound traffic, but doesn't specify any rules for inbound traffic. Inbound traffic rules are taken care of with another resource:

```yaml
  InstanceSecurityGroup:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupName: "Example Security Group"
      GroupDescription: "RDS traffic"
      VpcId: !Ref "VPC"
      SecurityGroupEgress:
      - IpProtocol: "-1"
        CidrIp: "0.0.0.0/0"
```

It's rare that a production database is accessible to public traffic. In fact, RDS solutions like Aurora Serverless (which you create next) [are only accessible in a VPC](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless.html#aurora-serverless.requirements):

> You can't give an Aurora Serverless v1 DB cluster a public IP address. You can access an Aurora Serverless v1 DB cluster only from within a VPC.

To grant resources in the VPC access to the RDS instance, you create an ingress rule that grants network traffic to any resource that has been assigned the security group defined above. Allowing resources that share a security group to communicate with each other is a convenient way to group related resources without having to partition them in special CIDR blocks.

Ingress rules are represented by the [AWS::EC2::SecurityGroupIngress](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group-ingress.html) resource:

```yaml
  InstanceSecurityGroupIngress:
    Type: "AWS::EC2::SecurityGroupIngress"
    DependsOn: "InstanceSecurityGroup"
    Properties:
      GroupId: !Ref "InstanceSecurityGroup"
      IpProtocol: "tcp"
      FromPort: "0"
      ToPort: "65535"
      SourceSecurityGroupId: !Ref "InstanceSecurityGroup"
```

You now have everything in place to deploy the RDS instance, represented by the [AWS::RDS::DBCluster](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html) resource. The example below creates a [serverless Aurora instance](https://aws.amazon.com/rds/aurora/serverless/):

```yaml
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
```

## Conclusion

RDS provides a managed, scalable, and highly available database platform supporting a number of popular database providers. 

This post built on our [post describing VPCs with private subnets](https://octopus.com/blog/aws-vpc-private), and demonstrated the resources required to deploy a serverless Aurora RDS instance with security groups ready to be attached to any additional resources that required database access.

We have [other posts about CloudFormation templates](https://octopus.com/blog/tag/CloudFormation) you might find helpful too.

!include <q2-2022-newsletter-cta>

Happy deployments!
