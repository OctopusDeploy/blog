---
title: Deploy a Lambda with CloudFormation
description: Learn how to deploy a Lambda with this sample CloudFormation template
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Lambda is a serverless Function as a Service (FaaS) offering from AWS. Lambda's provide native scaling, high availability, and the ability to scale to 0 to keep costs down for infrequently used deployments.

In this post you'll learn how to deploy a Lambda using CloudFormation.

## An RDS CloudFormation template

The template below deploys a Lambda into a new VPC with a private and public subnet:

```yaml
Parameters:
  Tag:
    Type: String
  LambdaS3Bucket:
    Type: String
  LambdaS3Key:
    Type: String
  LambdaName:
    Type: String
    
Resources: 
  VPC:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: "10.0.0.0/16"
      InstanceTenancy: "default"
      Tags:
      - Key: "Name"
        Value: !Ref "Tag"
        
  InternetGateway:
    Type: "AWS::EC2::InternetGateway"
    
  VPCGatewayAttachment:
    Type: "AWS::EC2::VPCGatewayAttachment"
    Properties:
      VpcId: !Ref "VPC"
      InternetGatewayId: !Ref "InternetGateway"
        
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

  SubnetC:
    Type: "AWS::EC2::Subnet"
    Properties:
      AvailabilityZone: !Select 
        - 1
        - !GetAZs 
          Ref: 'AWS::Region'
      VpcId: !Ref "VPC"
      CidrBlock: "10.0.2.0/24"
      
  RouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref "VPC"
      
  InternetRoute:
    Type: "AWS::EC2::Route"
    Properties:
      DestinationCidrBlock: "0.0.0.0/0"
      GatewayId: !Ref InternetGateway
      RouteTableId: !Ref RouteTable
      
  SubnetARouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref RouteTable
      SubnetId: !Ref SubnetA
      
  EIP:
    Type: "AWS::EC2::EIP"
    Properties:
      Domain: "vpc"
      
  Nat:
    Type: "AWS::EC2::NatGateway"
    Properties:
      AllocationId: !GetAtt "EIP.AllocationId"
      SubnetId: !Ref "SubnetA"
      
  NatRouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref "VPC"
      
  NatRoute:
    Type: "AWS::EC2::Route"
    Properties:
      DestinationCidrBlock: "0.0.0.0/0"
      NatGatewayId: !Ref "Nat"
      RouteTableId: !Ref "NatRouteTable"
      
  SubnetBRouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref NatRouteTable
      SubnetId: !Ref SubnetB

  SubnetCRouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref NatRouteTable
      SubnetId: !Ref SubnetC

  InstanceSecurityGroup:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupName: "Example Security Group"
      GroupDescription: "Lambda Traffic"
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

  AppLogGroup:
    Type: "AWS::Logs::LogGroup"
    Properties:
      LogGroupName: !Sub "/aws/lambda/${LambdaName}"
      
  IamRoleLambdaExecution:
    Type: "AWS::IAM::Role"
    Properties:
      Path: "/"
      RoleName: !Sub "${LambdaName}-role"  
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
        - Effect: "Allow"
          Principal:
            Service:
            - "lambda.amazonaws.com"
          Action: "sts:AssumeRole"
      ManagedPolicyArns: 
      - "arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole"
      Policies:
      - PolicyName: !Sub "${LambdaName}-policy"
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
          - Effect: "Allow"
            Action:
            - "logs:CreateLogStream"
            - "logs:CreateLogGroup"
            - "logs:PutLogEvents"
            Resource:
            - !Sub "arn:${AWS::Partition}:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/${LambdaName}*:*"
        
  MyLambda:
    Type: "AWS::Lambda::Function"
    Properties:
        Code:
          S3Bucket: !Ref "LambdaS3Bucket"
          S3Key: !Ref "LambdaS3Key"
        Description: "My Lambda"
        FunctionName: !Ref "LambdaName"
        Handler: "not.used.in.provided.runtime"
        MemorySize: 256
        PackageType: "Zip"
        Role: !GetAtt "IamRoleLambdaExecution.Arn"
        Runtime: "provided"
        Timeout: 30
        VpcConfig:
            SecurityGroupIds: !Ref "InstanceSecurityGroup"
            SubnetIds: 
            - !Ref "SubnetB"
            - !Ref "SubnetC"
      
Outputs:
  VpcId:
    Description: The VPC ID
    Value: !Ref VPC
```