---
title: Create a mixed AWS VPC with CloudFormation
description: Learn how to create a mixed AWS VPC with this sample CloudFormation template.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-05-10-1400
metaImage: blogimage-createamixedawsvpcwithcloudformation-2022.png
bannerImage: blogimage-createamixedawsvpcwithcloudformation-2022.png
bannerImageAlt: Two blue padlocks sitting amongst clouds, one open and branded with an open eye,  the other closed and branded with a closed eye. 
isFeatured: false
tags:
 - DevOps
 - Runbooks Series
 - AWS
 - CloudFormation
---

In our first post, [Create a private AWS VPC with CloudFormation](https://octopus.com/blog/aws-vpc-private), you looked at how to create a VPC with private subnets, and then, in our second post, [add an internet gateway to grant internet access inside public subnets](https://octopus.com/blog/aws-vpc-public).

By mixing both private and public subnets, it's possible to create a VPC that exposes some instances publicly, while restricting access to private instances. This is a common configuration for VPCs that host a public website, and the website accesses a private database.

In this post, you create a VPC with a mix of public and private subnets.

## Types of subnets

AWS has two types of subnets: public and private.

A public subnet has a connection to the internet via an [internet gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html), and can host resources with public IP addresses. An internet gateway is defined by AWS as:

> a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet. 

A private subnet does not route traffic to an internet gateway. Resources in a private subnet do not have public IP addresses, and can only communicate with resources in other subnets in the same VPC.

One or more subnets can be placed in a VPC. It is possible to mix and match public and private subnets in a VPC, allowing some resources in the VPC to access the internet, and some to only access other resources in the VPC.

In a VPC with public and private subnets, it's possible to route outgoing internet traffic from the private subnets through a [NAT gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html). Much like your home router, a NAT Gateway allows outbound internet traffic to be established, and for responses to those outbound requests to be routed back to the device in the private subnet. But a connection can not be initiated from an external connection through a NAT Gateway.

A VPC with public and private subnets is the most complicated to build, but offers the most flexibility when deploying instances that can either be accessed from the public internet, or can only be accessed from in the VPC.

## Creating a VPC with public and private subnets

The following CloudFormation template creates a VPC with one public subnet and one private subnet:

```yaml
Parameters:
  Tag:
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
      
Outputs:
  VpcId:
    Description: The VPC ID
    Value: !Ref VPC

```

This template builds on the [previous post](https://octopus.com/blog/aws-vpc-public), so refer to that post for details on internet gateways, routes, and route associations.

The template above treats `SubnetA` as the public subnet, and `SubnetB` as the private subnet. 

To make `SubnetB` private, the route association that directed public traffic to the internet gateway has been removed.

However, instances in `SubnetB` will still have internet access via a NAT gateway.

The NAT gateway requires a public IP, represented by the [AWS::EC2::EIP](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-eip.html) resource:

```yaml
  EIP:
    Type: "AWS::EC2::EIP"
    Properties:
      Domain: "vpc"
```

You then create a Nat gateway, represented by the [AWS::EC2::NatGateway](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-natgateway.html) resource. The NAT Gateway is created in the public subnet to give it internet access:

```yaml
  Nat:
    Type: "AWS::EC2::NatGateway"
    Properties:
      AllocationId: !Ref "EIP"
      SubnetId: !Ref "SubnetA"
```

A second route table, represented by the [AWS::EC2::RouteTable](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-routetable.html) resource, is created to hold the network rules directing traffic to the NAT Gateway:

```yaml
  NatRouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties:
      VpcId: !Ref "VPC"
```

A new route, defined by the [AWS::EC2::Route](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-route.html) resource, directs all internet traffic to the NAT gateway:

```yaml
  NatRoute:
    Type: "AWS::EC2::Route"
    Properties:
      DestinationCidrBlock: "0.0.0.0/0"
      NatGatewayId: !Ref "Nat"
      RouteTableId: !Ref "NatRouteTable"
```

The new route table is associated with `SubnetB` via a [AWS::EC2::SubnetRouteTableAssociation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet-route-table-assoc.html) resource:

```yaml
  SubnetBRouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties:
      RouteTableId: !Ref NatRouteTable
      SubnetId: !Ref SubnetB
```

After it's created, the VPC contains a mix of public and private subnets. Any instances created in `SubnetA` have internet access via the internet gateway, and can be accessed via a public IP address. Instances created in `SubnetB` have internet access via the NAT gateway, but traffic can not be initiated from the internet.

## Conclusion

Including both public and private subnets in a VPC provides the most flexibility when placing instances that must be accessed from the internet or benefit from the extra security provided by not being exposed to public traffic. Even though private subnets don't allow public traffic to initiate a connection, instances in private subnets can still make outbound network requests via a NAT gateway.

In this post, you looked at a sample CloudFormation template that created a VPC with a public and a private subnet. This, along with the templates to create [VPCs with public subnets](https://octopus.com/blog/aws-vpc-public) and [VPCs with private subnets](https://octopus.com/blog/aws-vpc-private), provides you with a quick starting point to create resources in AWS.

!include <q2-2022-newsletter-cta>

Happy deployments!
