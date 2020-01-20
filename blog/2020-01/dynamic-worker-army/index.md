---
title: "Building a dynamic worker army with Terraform and AWS autoscaling groups"
description: How to create dynamic worker infrastructure using Terrafrom and AWS autoscaling groups.
author: shawn.sesna@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2021-01-15
tags:
 - DevOps
---

The advent of Infrastructure as Code (IaC) has been a tremendous leap forward, especially within the cloud space.  The ability to programatically define how the Infrastructure should look has led to environmental consistency and more predictable application behavior.  IaC has also given us the freedom to stop treating our servers as pets, since they can be spun up again the exact same way each time.  Cloud providers have taken IaC a step further by implementing what is commonly referred to as Auto Scaling.  Auto Scaling is the ability to dynamically spin up or tear down resources based on specific criteria.  A common use of Auto Scaling is eCommerce web applications running on cloud infrastructure.  When an increase in load is detected, Auto Scaling can provision additional resources, such as web servers.  Using IaC, the new web server is configured exactly like existing web servers.  Automated deployment softare, such as Octopus Deploy, then deploy the software to the newly provisioned server.  Once the deployment is complete, the server is added to the farm to distribute load.  When load subsides, the additional resources are no longer needed and are automatically torn down.  All of this can occur without a single human ever being involved. 

In this post, I will show you how to create a dynamic Octopus Deploy worker army using Terraform and Amazon Web Services (AWS) Auto Scaling groups.

## Definitions

### Terraform
Terraform, by HashiCorp, is an IaC technology that was developed to ease the configuration of cloud resources through use of Terraform Providers.  These Providers contain the necessary logic to make calls to the cloud provider API to configure the desired resources using the HashiCorp Configuration Language (HCL).

### AWS Auto Scaling Groups
AWS Auto Scaling Groups dynamically spin up or tear down resources based on specific criteria.

### Octopus Deploy workers
Octopus Deploy workers are Tentacles that perform work on behalf of the Octopus Deploy Server.  Tasks within Octopus Deploy that don't deploy to targets, such has scripts that execute against APIs, originally executed on the Octopus Server.  Workers were created to offload this work.

## Creating the worker army
For this demonstration, we're going to use AWS EC2 instances to host our Octopus Deploy workers using Terraform.

## Scenario
Let's pretend we're working for a small startup.  Our deployment process utilizes Octopus Deploy to deploy our software to a cloud provider.  Many of the deployment tasks utilize workers to execute against various APIs.  As a modern company, we have all of our server resources implemented in AWS versus maintaining our own datacenter, or renting space in one.  We've just entered a phase of exponential growth of the engineering department, deployment frequency has increased tenfold and our provisioned worker count is sometimes insufficient for demand.  Spinning up additional workers would solve the problem, but they're not always needed.  In addition, our existing workers are idle at night when nobody is working.

## The solution: Terraform + AWS + Octopus Deploy
AWS has a Terraform provider that will create Auto Scaling groups for our workers.  This will allow us to dynamically add additional workers when demand is high or deprovision workers when they're no longer needed.  In addition, we can use Octopus Deploy to schedule the destruction of all workers during the night when no one is working!

### Terraform files
It is entirely possible to define all of the resources we need in a single file, however, I've opted to logically separate the different components into indvidual files for ease of maintenance.  When Terraform executes, it evaluates all of the files within the folder and assembles the required resources

- autoscaling.tf
- autoscalingpolicy.tf
- backend.tf
- provider.tf
- securitygroup.tf
- vars.tf
- vpc.tf

#### autoscaling.tf
The autoscaling.tf file contains the resource definitions to create our AWS Autoscaling Groups.  There are two groups defined within this file, one for Linux workers and one for Windows workers.  Both aws_launch_configuration definitions contain a user_data section which contains a script that will automatically download the Tentacle for the given OS, install it, and connect it to our Octopus Deploy server.

```terraform
resource "aws_launch_configuration" "dynamic-linux-worker-launchconfig" {
    name_prefix = "dynamic-linux-worker-launchconfig"
    image_id = var.LINUX_AMIS
    instance_type = "t2.micro"
    
    security_groups = [aws_security_group.allow-octopusserver.id]
  
    # script to run when created
    user_data = <<-EOT
        #!/bin/bash
        serverUrl="#{Project.Octopus.Server.Url}"
        serverCommsPort="#{Project.Octopus.Server.PollingPort}"
        apiKey="#{Project.Octopus.Server.ApiKey}"
        name=$HOSTNAME
        configFilePath="/etc/octopus/default/tentacle-default.config"
        applicationPath="/home/Octopus/Applications/"
        workerPool="#{Project.Octopus.Server.WorkerPool}"
        machinePolicy="#{Project.Octopus.Server.MachinePolicy}"

        sudo apt-key adv --fetch-keys "https://apt.octopus.com/public.key"
        sudo add-apt-repository "deb https://apt.octopus.com/ stretch main"
        sudo apt-get update
        sudo apt-get install tentacle

        sudo /opt/octopus/tentacle/Tentacle create-instance --config "$configFilePath" --instance "$name"
        sudo /opt/octopus/tentacle/Tentacle new-certificate --if-blank
        sudo /opt/octopus/tentacle/Tentacle configure --noListen True --reset-trust --app "$applicationPath"
        echo "Registering the worker $name with server $serverUrl"
        sudo /opt/octopus/tentacle/Tentacle register-worker --server "$serverUrl" --apiKey "$apiKey" --name "$name"  --comms-style "TentacleActive" --server-comms-port $serverCommsPort --workerPool "$workerPool" --policy "$machinePolicy"

        cat >> /opt/octopus/tentacle/tentacle.service <<EOL
        [Unit]
        Description=Octopus Tentacle
        After=network.target

        [Service]
        Type=simple
        User=root
        WorkingDirectory=/etc/octopus/Tentacle/
        ExecStart=/opt/octopus/tentacle/Tentacle run --instance $name --noninteractive
        Restart=always

        [Install]
        WantedBy=multi-user.target

        EOL

        sudo cp /opt/octopus/tentacle/tentacle.service /etc/systemd/system/tentacle.service
        sudo chmod 644 /etc/systemd/system/tentacle.service
        sudo systemctl start tentacle
        sudo systemctl enable tentacle

    EOT    
}

resource "aws_launch_configuration" "dynamic-windows-worker-launchconfig" {
    name_prefix = "dynamic-windows-worker-launchconfig"
    image_id = var.WINDOWS_AMIS
    instance_type = "t2.micro"
    
    security_groups = [aws_security_group.allow-octopusserver.id]

    user_data     = <<-EOF
<script>
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"            

choco install octopusdeploy.tentacle -y

@"C:\Program Files\Octopus Deploy\Tentacle\Tentacle.exe" create-instance --config "c:\octopus\home"

@"C:\Program Files\Octopus Deploy\Tentacle\Tentacle.exe" new-certificate --if-blank

@"C:\Program Files\Octopus Deploy\Tentacle\Tentacle.exe" configure --noListen True --reset-trust --app "c:\octopus\applications"

@"C:\Program Files\Octopus Deploy\Tentacle\Tentacle.exe" register-worker --server "#{Project.Octopus.Server.Url}" --apiKey "#{Project.Octopus.Server.ApiKey}"  --comms-style "TentacleActive" --server-comms-port "#{Project.Octopus.Server.PollingPort}" --workerPool "#{Project.Octopus.Server.WorkerPool}" --policy "#{Project.Octopus.Server.MachinePolicy}"

@"C:\Program Files\Octopus Deploy\Tentacle\Tentacle.exe" service --install

@"C:\Program Files\Octopus Deploy\Tentacle\Tentacle.exe" service --start


</script>
EOF    
}

resource "aws_autoscaling_group" "dynamic-linux-worker-autoscaling" {
    name = "dynamic-linux-worker-autoscaling"
    vpc_zone_identifier = [aws_subnet.worker-public-1.id, aws_subnet.worker-public-2.id, aws_subnet.worker-public-3.id]
    launch_configuration = aws_launch_configuration.dynamic-linux-worker-launchconfig.name
    min_size = 2
    max_size = 3
    health_check_grace_period = 300
    health_check_type = "EC2"
    force_delete = true

    tag {
        key = "Name"
        value = "Octopus Deploy Linux Worker"
        propagate_at_launch = true
    }
}

resource "aws_autoscaling_group" "dynamic-winodws-worker-autoscaling" {
    name = "dynamic-windows-worker-autoscaling"
    vpc_zone_identifier = [aws_subnet.worker-public-1.id, aws_subnet.worker-public-2.id, aws_subnet.worker-public-3.id]
    launch_configuration = aws_launch_configuration.dynamic-windows-worker-launchconfig.name
    min_size = 2
    max_size = 3
    health_check_grace_period = 300
    health_check_type = "EC2"
    force_delete = true

    tag {
        key = "Name"
        value = "Octopus Deploy Windows Worker"
        propagate_at_launch = true
    }
}
```

#### autoscalingpolicy.tf
The autoscalingpolicy.tf file contains the resource definitions for auto scaling triggers.  This file has the policy for both:

- Scale up
- Scale down

```terraform
# scale up alarm

resource "aws_autoscaling_policy" "linux-worker-cpu-policy" {
  name                   = "linux-worker-cpu-policy"
  autoscaling_group_name = aws_autoscaling_group.dynamic-linux-worker-autoscaling.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = "1"
  cooldown               = "300"
  policy_type            = "SimpleScaling"
}

resource "aws_cloudwatch_metric_alarm" "linux-worker-cpu-alarm" {
  alarm_name          = "linux-worker-cpu-alarm"
  alarm_description   = "linux-worker-cpu-alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "30"

  dimensions = {
    "AutoScalingGroupName" = aws_autoscaling_group.dynamic-linux-worker-autoscaling.name
  }

  actions_enabled = true
  alarm_actions   = [aws_autoscaling_policy.linux-worker-cpu-policy.arn]
}

# scale down alarm
resource "aws_autoscaling_policy" "linux-worker-cpu-policy-scaledown" {
  name                   = "linux-worker-cpu-policy-scaledown"
  autoscaling_group_name = aws_autoscaling_group.dynamic-linux-worker-autoscaling.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = "-1"
  cooldown               = "300"
  policy_type            = "SimpleScaling"
}

resource "aws_cloudwatch_metric_alarm" "linux-worker-cpu-alarm-scaledown" {
  alarm_name          = "linux-worker-cpu-alarm-scaledown"
  alarm_description   = "linux-worker-cpu-alarm-scaledown"
  comparison_operator = "LessThanOrEqualToThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "5"

  dimensions = {
    "AutoScalingGroupName" = aws_autoscaling_group.dynamic-linux-worker-autoscaling.name
  }

  actions_enabled = true
  alarm_actions   = [aws_autoscaling_policy.linux-worker-cpu-policy-scaledown.arn]
}

resource "aws_autoscaling_policy" "windows-worker-cpu-policy" {
  name                   = "windows-worker-cpu-policy"
  autoscaling_group_name = aws_autoscaling_group.dynamic-windows-worker-autoscaling.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = "1"
  cooldown               = "300"
  policy_type            = "SimpleScaling"
}

resource "aws_cloudwatch_metric_alarm" "windows-worker-cpu-alarm" {
  alarm_name          = "windows-worker-cpu-alarm"
  alarm_description   = "windows-worker-cpu-alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "30"

  dimensions = {
    "AutoScalingGroupName" = aws_autoscaling_group.dynamic-windows-worker-autoscaling.name
  }

  actions_enabled = true
  alarm_actions   = [aws_autoscaling_policy.windows-worker-cpu-policy.arn]
}

# scale down alarm
resource "aws_autoscaling_policy" "windows-worker-cpu-policy-scaledown" {
  name                   = "windows-worker-cpu-policy-scaledown"
  autoscaling_group_name = aws_autoscaling_group.dynamic-windows-worker-autoscaling.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = "-1"
  cooldown               = "300"
  policy_type            = "SimpleScaling"
}

resource "aws_cloudwatch_metric_alarm" "windows-worker-cpu-alarm-scaledown" {
  alarm_name          = "windows-worker-cpu-alarm-scaledown"
  alarm_description   = "windows-worker-cpu-alarm-scaledown"
  comparison_operator = "LessThanOrEqualToThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "5"

  dimensions = {
    "AutoScalingGroupName" = aws_autoscaling_group.dynamic-windows-worker-autoscaling.name
  }

  actions_enabled = true
  alarm_actions   = [aws_autoscaling_policy.windows-worker-cpu-policy-scaledown.arn]
}
```

#### backend.tf
When using Terraform locally, the state files are stored on your local machine.  When executing this on Octopus Deploy, it does not keep the state files so we need to configure an external storage location.  The backend.tf creates a folder within an AWS S3 bucket to maintain state.

```terraform
terraform {
    backend "s3" {
        bucket = "#{Project.AWS.S3.Bucket}"
        key = "#{Project.AWS.S3.Key}"
        region = "#{Project.AWS.Region}"
    }
}
```

#### provider.tf
The provider.tf file defines which provider Terraform is going to use.  In our case, we're using the AWS Terraform provider

```terraform
provider "aws" {
  region     = var.AWS_REGION
}
```

#### securitygroup.tf
AWS, among others, call firewall rules security groups.  In this file, we define both the ingress and egress rules for our instances.  For egress, we're allowing all traffic out from the server.  For ingress, we've defined both SSH and RDP ports.  These are used for debugging and can be omitted for Production use.

```terraform
resource "aws_security_group" "allow-octopusserver" {
  vpc_id      = aws_vpc.worker_vpc.id
  name        = "allow-octopusserver"
  description = "Security group that allows traffic to the worker from the Octpus Server"
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 10933
    to_port     = 10933
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port = 22
    to_port = 22
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port = 3389
    to_port = 3389
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "allow-octopusserver"
  }
}
```

#### vars.tf
The vars.tf file is what is commonly used to define all of the variables we use within our Terraform templates.  Our vars.tf contains:

- AWS_REGION - the region that we're using for our AWS resources
- LINUX_AMIS - the name of the ami for our Linux workers
- WINDOWS_AMIS - the name of the ami for our Windows workers

```terraform
variable "AWS_REGION" {
  default = "#{Project.AWS.Region}"
}

variable "LINUX_AMIS" {
  default =  "ami-084a6c14d8630bb68"
}

variable "WINDOWS_AMIS"{
  default = "ami-087ee25b86edaf4b1"
}
```

#### vpc.tf
To keep our AWS resources logically separated, I've created our resources within their own Virtual Private Cloud (VPC).  This file contains the following resource definitions:

- Vpc
- Subnets
- Internet gateway
- Route table
- Route table associations

```terraform
# Internet VPC
resource "aws_vpc" "worker_vpc" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_support   = "true"
  enable_dns_hostnames = "true"
  enable_classiclink   = "false"
  tags = {
    Name = "worker_vpc"
  }
}

# Subnets
resource "aws_subnet" "worker-public-1" {
  vpc_id                  = aws_vpc.worker_vpc.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "${var.AWS_REGION}a"

  tags = {
    Name = "worker-public-1"
  }
}

resource "aws_subnet" "worker-public-2" {
  vpc_id                  = aws_vpc.worker_vpc.id
  cidr_block              = "10.0.2.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "${var.AWS_REGION}b"

  tags = {
    Name = "worker-public-2"
  }
}

resource "aws_subnet" "worker-public-3" {
  vpc_id                  = aws_vpc.worker_vpc.id
  cidr_block              = "10.0.3.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "${var.AWS_REGION}c"

  tags = {
    Name = "worker-public-3"
  }
}

resource "aws_subnet" "worker-private-1" {
  vpc_id                  = aws_vpc.worker_vpc.id
  cidr_block              = "10.0.4.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "${var.AWS_REGION}a"

  tags = {
    Name = "worker-private-1"
  }
}

resource "aws_subnet" "worker-private-2" {
  vpc_id                  = aws_vpc.worker_vpc.id
  cidr_block              = "10.0.5.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "${var.AWS_REGION}b"

  tags = {
    Name = "worker-private-2"
  }
}

resource "aws_subnet" "worker-private-3" {
  vpc_id                  = aws_vpc.worker_vpc.id
  cidr_block              = "10.0.6.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "${var.AWS_REGION}c"

  tags = {
    Name = "worker-private-3"
  }
}

# Internet GW
resource "aws_internet_gateway" "worker-gw" {
  vpc_id = aws_vpc.worker_vpc.id

  tags = {
    Name = "worker"
  }
}

# route tables
resource "aws_route_table" "worker-public" {
  vpc_id = aws_vpc.worker_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.worker-gw.id
  }

  tags = {
    Name = "worker-public-1"
  }
}

# route associations public
resource "aws_route_table_association" "worker-public-1-a" {
  subnet_id      = aws_subnet.worker-public-1.id
  route_table_id = aws_route_table.worker-public.id
}

resource "aws_route_table_association" "worker-public-2-a" {
  subnet_id      = aws_subnet.worker-public-2.id
  route_table_id = aws_route_table.worker-public.id
}

resource "aws_route_table_association" "worker-public-3-a" {
  subnet_id      = aws_subnet.worker-public-3.id
  route_table_id = aws_route_table.worker-public.id
}
```