---
title: Sharing workers across spaces
description: Learn methods to easily share workers between spaces
author: shawn.sesna@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-bestpracticesforoctopusspaces-2022.png
bannerImage: blogimage-bestpracticesforoctopusspaces-2022.png
bannerImageAlt: Overhead shot of a round table split into three equal parts, with people working on laptops and notebooks at each section.
isFeatured: false
tags: 
  - 
---

[Spaces](https://octopus.com/docs/administration/spaces) in Octopus Deploy are hard-walled and don't allow you to share anything between them (except users).  While this is great for isolation, it can be problematic when there are things that you do want to share amongst all spaces such as workers.  That specific scenario is something we needed to solve to support our [Samples](https://samples.octopus.app) instance.  In this post, I will go over our solution that automates sharing workers across multiple spaces.

## The problem
The infrastructure for Samples is created and destroyed daily.  As the Samples instance grew, more and more demands were being made on the [Dynamic Workers](https://octopus.com/docs/infrastructure/workers/dynamic-worker-pools) cloud instances are allocated (one Windows and one Linux).  We soon ran into resource contention and a chicken and egg scenario as the [Runbooks](https://octopus.com/docs/runbooks) used to create infrastructure would often use the [Run Octopus Deploy Runbook](https://library.octopus.com/step-templates/0444b0b3-088e-4689-b755-112d1360ffe3/actiontemplate-run-octopus-deploy-runbook) template.  This template can be configured to wait for the invoked runbook is complete, however, this sometimes caused a condition where the parent process would wait for the child process to complete, but the child process needed  exclusive access (see [this](https://octopus.com/blog/workers-explained) blog post for conditions where this can occur) and would wait for the first task to be complete.  

## The solutions
The overall solution was to have more workers available to perform the tasks.  We came up with two different solutions:
- Dedicated workers
- Shared workers

### Solution 1: Dedicated workers
Our first iteration to solve this problem was to have each space create their own, dedicated worker.  While this worked well, there were two glaring issues; the workers were idle 99% of the day, and as more spaces were added, our costs would increase.  We reached the point where we needed to reevaluate our cloud resource usage and look for some cost savings.  Workers, along with other things such as database server instances, were identified as items that didn't need to be dedicated and could be shared.

### Solution 2: Shared workers
Each space in Samples were creating workers using the cloud provider that suited their needs.  To continue to support this, it was decided to create workers in all three cloud providers to share with all spaces.  Using Terraform, we created workers using the cloud provider scaling features so if we needed more or less workers, we simply adjust the scale accordingly.  Provisioning workers uses a runbook with the following steps:

- Get Space List
- Create workers
- Wait for workers to register themselves
- Add workers to remaining spaces

#### Get Space List
In order to add the workers to all of the spaces, we first need to gather a list of all spaces on the instance.  The `Get Space List` step retrieves the list and sets the following output variables:
- InitialSpaceName - The name of the first space in our Spaces list, this value is used in the script that's run on the VMs so they can register themselves to the instance.
- InitialSpaceId - The Id of the initial space the workers will be added to.
- RemainingSpaceIds - A comma delimited list of the remaining space Ids to add the workers to
- WorkerPoolName - Name of the worker pool to register and add to in the spaces.  Samples has created a consistent list of worker pools in all spaces using Terraform.

```powershell
function Get-OctopusItems
{
	# Define parameters
    param(
    	$OctopusUri,
        $ApiKey,
        $SkipCount = 0
    )
    
    # Define working variables
    $items = @()
    $skipQueryString = ""
    $headers = @{"X-Octopus-ApiKey"="$ApiKey"}

    # Check to see if there there is already a querystring
    if ($octopusUri.Contains("?"))
    {
        $skipQueryString = "&skip="
    }
    else
    {
        $skipQueryString = "?skip="
    }

    $skipQueryString += $SkipCount
    
    # Get intial set
    $resultSet = Invoke-RestMethod -Uri "$($OctopusUri)$skipQueryString" -Method GET -Headers $headers

    # Check to see if it returned an item collection
    if ($resultSet.Items)
    {
        # Store call results
        $items += $resultSet.Items
    
        # Check to see if resultset is bigger than page amount
        if (($resultSet.Items.Count -gt 0) -and ($resultSet.Items.Count -eq $resultSet.ItemsPerPage))
        {
            # Increment skip count
            $SkipCount += $resultSet.ItemsPerPage

            # Recurse
            $items += Get-OctopusItems -OctopusUri $OctopusUri -ApiKey $ApiKey -SkipCount $SkipCount
        }
    }
    else
    {
        return $resultSet
    }
    

    # Return results
    return $items
}


# Define variables
$baseUrl = $OctopusParameters['Samples.Octopus.Url'] 
$apiKey = $OctopusParameters['Samples.Octopus.Api.Key']
$header = @{ "X-Octopus-ApiKey" = $apiKey }
$initialSpaceName = ""
$initialSpaceId = ""
$remainingSpaceIds = @()
$workerPoolName = "$($OctopusParameters['Project.CloudProvider.Folder.Name']) Worker Pool TF"

# Get all spaces
Write-Host "Getting list of all spaces on $baseUrl ..."
$spaces = Get-OctopusItems -OctopusUri "$baseUrl/api/spaces" -ApiKey $apiKey

# Loop through the spaces
foreach ($space in $spaces)
{
    # Get worker pools of space
    Write-Host "Getting all worker pools for space $($space.Name) ..."
    $workerPools = Get-OctopusItems -OctopusUri "$baseUrl/api/$($space.Id)/workerPools" -ApiKey $apiKey
    
    # Check to see if it has the pool we're looking for
    if ($workerPools.Name -contains $workerPoolName)
    {
    	# Check to see if we have an initial space
        if ([string]::IsNullOrWhitespace($initialSpaceName))
        {
        	# Assign the values
            Write-Host "Initial Space: $($space.name)"
            $initialSpaceName = $space.Name
            $initialSpaceId = $space.Id
        }
        else
        {
        	# Add to list
            Write-Host "Adding space: $($space.Name)($($space.Id))"
            $remainingSpaceIds += $space.Id
        }
    }
}

# Set output variables
Write-Host "Setting output variable InitialSpaceName to $initialSpaceName"
Set-OctopusVariable -name "InitialSpaceName" -value $initialSpaceName
Write-Host "Setting output variable InitialSpaceId to $initialSpaceId"
Set-OctopusVariable -name "InitialSpaceId" -value $initialSpaceId
Set-OctopusVariable -name "RemainingSpaceIds" -value ($remainingSpaceIds -join ",")
Write-Host "Setting output variable WorkerPoolName to $workerPoolName"
Set-OctopusVariable -name "WorkerPoolName" -value $workerPoolName

```

#### Create workers
For our shared workers, we've separated the Terraform files into folders designated by the cloud provider.  Below are the snippets of Terraform we created for each provider (see full implementation [here](https://github.com/OctopusSamples/IaC/tree/master/octopus-samples-instances/shared-workers-terraform))

<details>
    <summary>AWS worker Terraform</summary>

```terraform
resource "aws_iam_instance_profile" "linux-worker-profile" {
    name = var.octopus_aws_instance_profile_name
    role = var.octopus_aws_role_name
}


resource "aws_launch_configuration" "linux-worker-launchconfig" {
    name_prefix = var.octopus_aws_launch_configuration_name
    image_id = "${var.octopus_aws_linux_ami_id}"
    instance_type = var.octopus_aws_ec2_instance_type

    iam_instance_profile = "${aws_iam_instance_profile.linux-worker-profile.name}"
    
    security_groups = ["${var.octopus_aws_security_group_id}"]
  
    # script to run when created
    user_data = "${file("../configure-tentacle.sh")}"

    # root disk
    root_block_device {
        volume_size           = "30"
        delete_on_termination = true
    }
}

resource "aws_autoscaling_group" "linux-worker-autoscaling" {
    name = var.auto_scaling_group_name
    vpc_zone_identifier = var.octopus_aws_subnets
    launch_configuration = "${aws_launch_configuration.linux-worker-launchconfig.name}"
    min_size = var.octopus_aws_autoscalinggroup_size
    max_size = var.octopus_aws_autoscalinggroup_size
    health_check_grace_period = 300
    health_check_type = "EC2"
    force_delete = true
    
    tag {
        key = "Name"
        value = "Samples Linux Worker"
        propagate_at_launch = true
    }
}
```
</details>

<details>
    <summary>Azure worker Terraform</summary>

```terraform
// Define resource group
resource "azurerm_resource_group" "octopus-samples-azure-workers" {
  name      = var.octopus_azure_resourcegroup_name
  location  = var.octopus_azure_location
  tags = var.tags
}

// Define virtual network
resource "azurerm_virtual_network" "octopus-samples-workers-virtual-network" {
  name                = "octopus-samples-workers"
  address_space       = ["10.0.0.0/16"]
  location            = var.octopus_azure_location
  resource_group_name = var.octopus_azure_resourcegroup_name
  depends_on = [
     azurerm_resource_group.octopus-samples-azure-workers
  ]
  tags = var.tags
}

// Define subnet
resource "azurerm_subnet" "octopus-samples-workers-subnet" {
  name                 = "octopus-samples-workers-subnet"
  resource_group_name  = var.octopus_azure_resourcegroup_name
  virtual_network_name = azurerm_virtual_network.octopus-samples-workers-virtual-network.name
  address_prefixes     = ["10.0.2.0/24"]
  depends_on = [
     azurerm_resource_group.octopus-samples-azure-workers,
     azurerm_virtual_network.octopus-samples-workers-virtual-network
  ]  
}

// Define azure scale set
resource "azurerm_linux_virtual_machine_scale_set" "samples-azure-workers" {
  name                = var.octopus_azure_scaleset_name
  resource_group_name = var.octopus_azure_resourcegroup_name
  location            = var.octopus_azure_location
  sku                 = var.octopus_azure_vm_size
  instances           = var.octopus_azure_vm_instance_count
  admin_username      = var.octopus_azure_vm_admin_username
  admin_password =  var.octopus_azure_vm_admin_password
  disable_password_authentication = false
  user_data = "${base64encode(file("../configure-tentacle.sh"))}"
  
  identity {
    type = "SystemAssigned"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = var.octopus_azure_vm_sku
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }

  network_interface {
    name    = "example"
    primary = true

    ip_configuration {
      name      = "internal"
      primary   = true
      subnet_id = azurerm_subnet.octopus-samples-workers-subnet.id
    }
  }
  tags = var.tags
}
```
</details>

<details>
    <summary>GCP worker Terraform</summary>

```terraform
resource "google_compute_instance" "vm_instance" {
  count        = var.instance_count
  name         = "${var.instance_name}-${count.index + 1}"
  machine_type = var.instance_size

  boot_disk {
    initialize_params {
      image = var.instance_osimage
      size = 30
    }
  }

  network_interface {
    network = "default" #google_compute_network.vpc_network.name

    access_config {
      // Ephemeral public IP - needed to send and receive traffic directly to and from outside network
    }
  }

  metadata_startup_script = file("../configure-tentacle.sh")

  service_account {
    email = google_service_account.database_service_account.email
    scopes = ["cloud-platform"]
  }

  tags = ["octopus-samples-worker"]
}

output "ip" {
  value = google_compute_instance.vm_instance[*].network_interface[0].access_config[0].nat_ip
}
```
</details>


Included in the Terraform package is a bash script that is run on the VMs created by the cloud scaling technology.  When executed, the VM will register itself to the Space and Pool from the output variables of the `Get Space List` step

```bash
#!/bin/bash
serverUrl="#{Samples.Octopus.Url}"
serverCommsPort="10943"
apiKey="#{Samples.Octopus.Api.Key}"
name=$HOSTNAME
configFilePath="/etc/octopus/default/tentacle-default.config"
applicationPath="/home/Octopus/Applications/"
workerPool="#{Octopus.Action[Get Samples Spaces].Output.WorkerPoolName}"
machinePolicy="Default Machine Policy"
space="#{Octopus.Action[Get Samples Spaces].Output.InitialSpaceName}"

# Install Tentacle
sudo apt-key adv --fetch-keys "https://apt.octopus.com/public.key"
sudo add-apt-repository "deb https://apt.octopus.com/ focal main"
sudo apt-get update
sudo apt-get install tentacle -y

# Install Docker
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt-get install docker-ce docker-ce-cli containerd.io -y

# Install wget
sudo apt-get install wget -y

# Download the Microsoft repository GPG keys
wget https://packages.microsoft.com/config/debian/10/packages-microsoft-prod.deb

# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb

# Update the list of products
sudo apt-get update

# Install PowerShell
sudo apt-get install -y powershell

# Pull worker tools image
sudo docker pull #{Project.Docker.WorkerToolImage}:#{Project.Docker.WorkerToolImageTag}

# Configure and register worker
sudo /opt/octopus/tentacle/Tentacle create-instance --config "$configFilePath" --instance "$name"
sudo /opt/octopus/tentacle/Tentacle new-certificate --if-blank
sudo /opt/octopus/tentacle/Tentacle configure --noListen True --reset-trust --app "$applicationPath"
echo "Registering the worker $name with server $serverUrl"
sudo /opt/octopus/tentacle/Tentacle service --install --start
sudo /opt/octopus/tentacle/Tentacle register-worker --server "$serverUrl" --apiKey "$apiKey" --name "$name"  --comms-style "TentacleActive" --server-comms-port $serverCommsPort --workerPool "$workerPool" --policy "$machinePolicy" --space "$space"
sudo /opt/octopus/tentacle/Tentacle service --restart
```

#### Wait for workers to register themselves
Before the workers can be added to the other spaces, they must first register themselves.  This step monitors the worker pool until the desired number of worker machines has been registered.

```powershell
# Define parameters 
$baseUrl = $OctopusParameters['Samples.Octopus.Url'] 
$apiKey = $OctopusParameters['Samples.Octopus.Api.Key']
$header = @{ "X-Octopus-ApiKey" = $apiKey }
$spaceId = $OctopusParameters['Octopus.Action[Get Samples Spaces].Output.InitialSpaceId']
$spaceName = $OctopusParameters['Octopus.Action[Get Samples Spaces].Output.InitialSpaceName']
$workerPoolName = $OctopusParameters['Octopus.Action[Get Samples Spaces].Output.WorkerPoolName']

if ($baseUrl.EndsWith("/"))
{
	$baseUrl = $baseUrl.SubString(0, $baseUrl.LastIndexOf("/"))
}

# Get worker pool
Write-Host "Getting reference to $workerPoolName in space $spaceName ..."
$workerPool = ((Invoke-RestMethod -Method Get -Uri "$baseUrl/api/$($spaceId)/workerpools/all" -Headers @{"X-Octopus-ApiKey"="$apiKey"}) | Where-Object {$_.Name -eq $workerPoolName})

# Check worker pool
if ($null -ne $workerPool)
{
  # Get all workers
  Write-Host "Checking to see if workers have registered themselves ..."
  $workers = (Invoke-RestMethod -Method Get -Uri "$baseUrl/api/$($spaceId)/workerpools/$($workerPool.Id)/workers" -Headers @{"X-Octopus-ApiKey"="$apiKey"})

  $retries = 20

  $cloudProvider = $OctopusParameters['Project.CloudProvider.Folder.Name']
  $numberOfWorkersKey = $OctopusParameters.Keys | Where-Object {$_ -like "*$cloudProvider*" -and $_ -like "*Instance.Count*"}
  $numberOfWorkers = $OctopusParameters[$numberOfWorkersKey]
   
  while ($workers.Items.Count -ne $numberOfWorkers)
  {
    if ($retries -gt 0)
    {
    	Write-Host "Waiting 60 seconds for $numberOfWorkers workers to register themselves, $retries tries remaining ..."
        Start-Sleep -Seconds 60
        $retries--
        $workers = (Invoke-RestMethod -Method Get -Uri "$baseUrl/api/$($spaceId)/workerpools/$($workerPool.Id)/workers" -Headers @{"X-Octopus-ApiKey"="$apiKey"})
    }
    else
    {
    	Write-Error "Workers didn't show up in time!"
    }
  }
}
```

#### Add workers to remaining spaces
With the workers added to the first space, we can use the API to add them to the remaining spaces

```powershell
function Get-OctopusItems
{
	# Define parameters
    param(
    	$OctopusUri,
        $ApiKey,
        $SkipCount = 0
    )
    
    # Define working variables
    $items = @()
    $skipQueryString = ""
    $headers = @{"X-Octopus-ApiKey"="$ApiKey"}

    # Check to see if there there is already a querystring
    if ($octopusUri.Contains("?"))
    {
        $skipQueryString = "&skip="
    }
    else
    {
        $skipQueryString = "?skip="
    }

    $skipQueryString += $SkipCount
    
    # Get intial set
    $resultSet = Invoke-RestMethod -Uri "$($OctopusUri)$skipQueryString" -Method GET -Headers $headers

    # Check to see if it returned an item collection
    if ($resultSet.Items)
    {
        # Store call results
        $items += $resultSet.Items
    
        # Check to see if resultset is bigger than page amount
        if (($resultSet.Items.Count -gt 0) -and ($resultSet.Items.Count -eq $resultSet.ItemsPerPage))
        {
            # Increment skip count
            $SkipCount += $resultSet.ItemsPerPage

            # Recurse
            $items += Get-OctopusItems -OctopusUri $OctopusUri -ApiKey $ApiKey -SkipCount $SkipCount
        }
    }
    else
    {
        return $resultSet
    }
    

    # Return results
    return $items
}


# Define variables
$baseUrl = $OctopusParameters['Samples.Octopus.Url'] 
$apiKey = $OctopusParameters['Samples.Octopus.Api.Key']
$header = @{ "X-Octopus-ApiKey" = $apiKey }
$remainingSpaceIds = $OctopusParameters['Octopus.Action[Get Samples Spaces].Output.RemainingSpaceIds'].Split(",")
$initialSpaceId = $OctopusParameters['Octopus.Action[Get Samples Spaces].Output.InitialSpaceId']
$initialWorkerPoolName = "$($OctopusParameters['Project.CloudProvider.Folder.Name']) Worker Pool TF"

# Get registered workers
$initialWorkerPool = (Get-OctopusItems -OctopusUri "$baseUrl/api/$initialSpaceId/workerPools" -ApiKey $apiKey | Where-Object {$_.Name -eq $initialWorkerPoolName})
$workers = Get-OctopusItems -OctopusUri "$baseUrl/api/$initialSpaceId/workerPools/$($initialWorkerPool.Id)/workers" -ApiKey $apiKey

# Loop through the spaces
foreach ($spaceId in $remainingSpaceIds)
{
	$workerPool = (Get-OctopusItems -OctopusUri "$baseUrl/api/$spaceId/workerPools" -ApiKey $apiKey | Where-Object {$_.Name -eq $initialWorkerPoolName})    
    
    # Check worker pool
    Write-Host "Verifying that space Id $spaceId has a worker pool called $initialWorkerPoolName ..."
    if ($null -ne $workerPool)
    {
    	# Get default machine policy
        $machinePolicy = (Get-OctopusItems -OctopusUri "$baseUrl/api/$spaceId/machinepolicies" -ApiKey $apiKey | Where-Object {$_.Name -eq "Default Machine Policy"})    
        
        # Loop through workers
        foreach ($worker in $workers)
        {
        
            # Build JSON payload
            $jsonPayload = @{
                Name = $worker.Name
                MachinePolicyId = $machinePolicy.Id
                IsDisabled = $worker.IsDisabled
                HealthStatus = $worker.HealthStatus
                HasLatestCalamari = $worker.HasLatestCalamari
                IsInProcess = $true
                EndPoint = $worker.Endpoint
                WorkerPoolIds = @($workerPool.Id)
            }

            try
            {
                # Add worker
                Write-Host "Adding $($worker.Name) to space Id $spaceId..."
                
                Invoke-RestMethod -Method Post -Uri "$baseUrl/api/$($spaceId)/workers" -Body ($jsonPayload | ConvertTo-Json -Depth 10) -Headers $header
            }
            catch
            {
    			Write-Highlight "An error occured adding $($worker.Name) to space Id $spaceId"
                Write-Warning "An error occured adding $($worker.Name) to space Id $spaceId"
                Write-Host "StatusCode:" $_.Exception.Response.StatusCode.value__ 
    			Write-Host "StatusDescription:" $_.Exception.Response.StatusDescription            		
            }
        }
    }
}
```

## Conclusion
Sharing workers across all spaces on Samples allowed us to consolidate and make more effecient use of resources.  I hope this post gives you some ideas on how to do the same.  

Happy Deployments!