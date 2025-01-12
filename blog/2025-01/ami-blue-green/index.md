---
title: Blue/Green deployments with Amazon EC2 Auto Scaling Groups
author: matthew.casperson@octopus.com
visibility: private
published: 2025-01-20-1400
metaImage: todo.png
bannerImage: todo.png
bannerImageAlt: 
isFeatured: false
tags:
- DevOps
- AWS
---

AWS EC2 Auto Scaling groups (ASG) have been the workhorse of cloud deployments for almost as long as we have had the concept of the cloud. ASGs provide teams deploying Virtual Machines (VMs) with the ability to scale up and down based on demand, and to replace instances that have failed. When combined with load balancers, ASGs can also provide advanced deployment processes such as Blue/Green deployments.

In this post, we'll look at how to set up Blue/Green deployments with ASGs, and how Octopus Deploy can help automate the process.

## What are Blue/Green deployments?

Blue/Green deployments involve maintaining two stacks: the blue stack and the green stack. One of these stacks is live, while the other is inactive. When a new version of the application is ready to be deployed, the inactive stack is updated with the new version. Once the update is complete and all health checks pass, the inactive stack becomes the live stack, and the old live stack is deactivated.

Blue/Green deployments provide a way to mimimize downtime and reduce the risk of a failed deployment. If the new version of the application fails to start, the inactive stack remains offline and the live stack continues to serve traffic. Or, if an issue is detected after deployment, the two stacks can be quickly switched back, reverting to the previous version of the application.

## Blue/Green deployments with ASGs

Implementing Blue/Green deployments with ASGs requires a few key components:

* Two ASGs: one for the blue stack and one for the green stack.
* A [NLB](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html) or [ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) with a [Listener](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) and [Listener Rule](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-update-rules.html) to handle network traffic.
* Two [target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html): one for the blue stack and one for the green stack.

![Blue/Green deployment with ASGs](blue-green-diagram.png)

Listener rules are very flexible and able to distribute traffic to target groups in a variety of ways. However, for Blue/Green deployments, we assume that either the blue or green target group is receiving 100% of traffic, and that a deployment will switch all traffic from one target group to the other.

## Creating the AWS infrastructure

The Blue/Green deployment presented in this post requires a number of AWS resources to be created. 

We first need a VPC with public subnets. These subnets will host our EC2 instances, as well as our load balancer. It also defines a security group allowing HTTP and SSH traffic:

```yaml
AWSTemplateFormatVersion: 2010-09-09
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      InstanceTenancy: default
      Tags:
        - Key: Name
          Value: Test VPC
        - Key: OwnerContact
          Value: "@matthewcasperson"
        - Key: Purpose
          Value: Test VPC
  InternetGateway:
    Type: AWS::EC2::InternetGateway
  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  SubnetA:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: ap-southeast-2a
      VpcId: !Ref VPC
      CidrBlock: 10.0.0.0/24
      MapPublicIpOnLaunch: true
  SubnetB:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: ap-southeast-2b
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
  RouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
  InternetRoute:
    Type: AWS::EC2::Route
    DependsOn: InternetGateway
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
      RouteTableId: !Ref RouteTable
  SubnetARouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref RouteTable
      SubnetId: !Ref SubnetA
  SubnetBRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref RouteTable
      SubnetId: !Ref SubnetB
  PackerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      Tags:
        - Key: Name
          Value: "packer-build-sg"
      GroupName: "packer Security Group"
      GroupDescription: "Allow HTTP Traffic"
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp:  0.0.0.0/0
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp:  0.0.0.0/0
      SecurityGroupEgress:
        - IpProtocol: -1
          CidrIp: 0.0.0.0/0
Outputs:
  VpcId:
    Value: !Ref VPC
    Description: VPC ID
  SubnetAId:
    Value: !Ref SubnetA
    Description: SubnetA ID
  SubnetBId:
    Value: !Ref SubnetB
    Description: SubnetB ID
  PackerSecurityGroupId:
    Value: !Ref PackerSecurityGroup
    Description: Security Group ID
```

Next we create an ALB, a listener, two target groups, and a security group that allows HTTP traffic:

```yaml
Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
  SubnetIdAz1:
    Type: AWS::EC2::Subnet::Id
  SubnetIdAz2:
    Type: AWS::EC2::Subnet::Id
Resources:
  LBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      Tags:
        - Key: Name
          Value: "lb-sg"
      GroupName: "Load Balancer Security Group"
      GroupDescription: "Allow HTTP Traffic"
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: 0.0.0.0/0
      SecurityGroupEgress:
        - IpProtocol: -1
          CidrIp: 0.0.0.0/0
  MyNLB:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: packer-alb
      Type: application
      Subnets:
        - !Ref SubnetIdAz1
        - !Ref SubnetIdAz2
      Scheme: internet-facing
      SecurityGroups:
        - !Ref LBSecurityGroup
      LoadBalancerAttributes:
        - Key: idle_timeout.timeout_seconds
          Value: '60'
  GreenTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckEnabled: true
      HealthCheckIntervalSeconds: 5
      HealthCheckPath: /
      HealthCheckPort: 80
      HealthCheckProtocol: HTTP
      HealthCheckTimeoutSeconds: 2
      HealthyThresholdCount: 2
      Matcher:
        HttpCode: 200
      Name: OctopusGreenTargetGroup
      Port: 80
      Protocol: HTTP
      TargetType: instance
      UnhealthyThresholdCount: 5
      VpcId: !Ref VpcId
  BlueTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckEnabled: true
      HealthCheckIntervalSeconds: 5
      HealthCheckPath: /
      HealthCheckPort: 80
      HealthCheckProtocol: HTTP
      HealthCheckTimeoutSeconds: 2
      HealthyThresholdCount: 2
      Matcher:
        HttpCode: 200
      Name: OctopusBlueTargetGroup
      Port: 80
      Protocol: HTTP
      TargetType: instance
      UnhealthyThresholdCount: 5
      VpcId: !Ref VpcId
  MyListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - FixedResponseConfig:
            StatusCode: 404
          Order: 1
          Type: fixed-response
      LoadBalancerArn: !Ref MyNLB
      Port: 80
      Protocol: HTTP
  MyListenerRule:
    Type: AWS::ElasticLoadBalancingV2::ListenerRule
    Properties:
      Actions:
        - ForwardConfig:
            TargetGroups:
              - TargetGroupArn: !Ref GreenTargetGroup
                Weight: 0
              - TargetGroupArn: !Ref BlueTargetGroup
                Weight: 100
          Order: 1
          Type: forward
      Conditions:
        - Field: path-pattern
          PathPatternConfig:
            Values:
              - /*
      ListenerArn: !Ref MyListener
      Priority: 10
Outputs:
  MyNLB:
    Description: The ALB
    Value: !Ref MyNLB
  MyListener:
    Description: The ALB listener
    Value: !Ref MyListener
  MyListenerRule:
    Description: The ALB listener rule
    Value: !Ref MyListenerRule
  BlueTargetGroup:
    Description: The blue target group
    Value: !Ref BlueTargetGroup
  GreenTargetGroup:
    Description: The green target group
    Value: !Ref GreenTargetGroup
```

Finally, we create the ASG:

```yaml
AWSTemplateFormatVersion: 2010-09-09
Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
  SubnetIdAz1:
    Type: AWS::EC2::Subnet::Id
  SubnetIdAz2:
    Type: AWS::EC2::Subnet::Id
  SecurityGroupId:
    Type: AWS::EC2::SecurityGroup::Id
  AmiId:
    Type: AWS::EC2::Image::Id
  GreenTargetGroup:
    Type: String
  BlueTargetGroup:
    Type: String
Resources:
  BlueLaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: !Sub ${AWS::StackName}-launch-template-blue
      LaunchTemplateData:
        ImageId: !Ref AmiId
        InstanceType: t3.small
        SecurityGroupIds:
          - !Ref SecurityGroupId
        TagSpecifications:
          - ResourceType: instance
            Tags:
              - Key: Name
                Value: "Blue instance"
          - ResourceType: volume
            Tags:
              - Key: Name
                Value: "Blue volume"
  GreenLaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: !Sub ${AWS::StackName}-launch-template-green
      LaunchTemplateData:
        ImageId: !Ref AmiId
        InstanceType: t3.small
        SecurityGroupIds:
          - !Ref SecurityGroupId
        TagSpecifications:
          - ResourceType: instance
            Tags:
              - Key: Name
                Value: "Green instance"
          - ResourceType: volume
            Tags:
              - Key: Name
                Value: "Green volume"
  AutoScalingGroupGreen:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      # These values priorities quickly replacing instances at the expense of availability
      InstanceMaintenancePolicy:
        MinHealthyPercentage: 0
        MaxHealthyPercentage: 100
      HealthCheckType: ELB
      HealthCheckGracePeriod: 30
      DefaultInstanceWarmup: 10
      VPCZoneIdentifier:
        - !Ref SubnetIdAz1
        - !Ref SubnetIdAz2
      LaunchTemplate:
        LaunchTemplateId: !Ref GreenLaunchTemplate
        Version: !GetAtt GreenLaunchTemplate.LatestVersionNumber
      MaxSize: '1'
      MinSize: '1'
      TargetGroupARNs:
        - !Ref GreenTargetGroup
  AutoScalingGroupBlue:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      # These values priorities quickly replacing instances at the expense of availability
      InstanceMaintenancePolicy:
        MinHealthyPercentage: 0
        MaxHealthyPercentage: 100
      HealthCheckGracePeriod: 10
      HealthCheckType: ELB
      # Setting this value means new EC2 instances will be marked healthy faster
      # https://docs.aws.amazon.com/autoscaling/ec2/userguide/understand-instance-refresh-default-values.html
      DefaultInstanceWarmup: 30
      VPCZoneIdentifier:
        - !Ref SubnetIdAz1
        - !Ref SubnetIdAz2
      LaunchTemplate:
        LaunchTemplateId: !Ref BlueLaunchTemplate
        Version: !GetAtt BlueLaunchTemplate.LatestVersionNumber
      MaxSize: '1'
      MinSize: '1'
      TargetGroupARNs:
        - !Ref BlueTargetGroup
Outputs:
  AutoScalingGroupGreenId:
    Value: !Ref AutoScalingGroupGreen
    Description: Auto Scaling Group Green ID
  AutoScalingGroupBlueId:
    Value: !Ref AutoScalingGroupBlue
    Description: Auto Scaling Group Blue ID
```