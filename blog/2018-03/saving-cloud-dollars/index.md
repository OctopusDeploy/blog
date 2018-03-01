---
title: "Saving Cloud Dollars: Consumption usage details in Azure"
description: "Using Octopus to notify if an Azure Resource group exceeds cost limits"
author: lawrence.wilson@octopus.com
visibility: private
tags:
 - Get-AzureRmConsumptionUsageDetail
---

## Summary
Have you ever deployed an expensive Virtual Machine into Azure for a quick 10 minute test only to realise it's been running constantly for the last 2 months? This blog post shows how you can use Octopus to notify you via Slack if a resource has reached a spending limit which you can specify using Resource Group tags.

Today, I'm going to show you how I like to use Octopus to save dollars in the cloud. Since Octopus has the ability to authenticate to and add resources to your cloud environments, it naturally has the ability to see everything. This makes Octopus a great place to run the scripts we need to report how much our resources are costing so that we can spot ones which are getting too expensive, too quickly.

### Scenario
I love a good scenario, so without further adue, please meet Fictional Company 1; A firm of highly skilled developers who need to run tests against resources in many different cloud platforms. They have chosen to use Octopus to run these reports because it allows them to apply a similar aproach to scripting a cost report, irrespective of which platform they are reporting against. Also, when developers at Fictional Company 1 want to recieve a notification, they prefer to see slack notifications over emails. So, let's get it done!

Fictional Company 1 have decided that in Azure, every resource group must be tagged with an OwnerContact where the value is either a slack channel, or the handle of an individul person. They have also defined a second tag called NotifyCostLimit which, when this limit is hit, Octopus will send out slack notifications. If there is no NotifyCostLimit applied, the default value of $100 will be assumed. If there is no OwnerContact tag applied (a big taboo among staff), the notification is set to notify #general (@here is used to discourage people from leaving a blank OwnerContact tag).

## Planning the new PowerShell Scripts
A classic example of a "package" that doesn't need a build process is a script that you want to be able to version control outside of Octopus, but is run as part of a deployment. The following sections look at we might configure a deployment to execute a script from the `OctopusDeploy/AcmeScripts` GitHub repository.

### Objective 1: Get all cost items for all subscriptions in Azure.
One key thing to note here is that the cmdlets used to retrieve the cost items can only get Resource Manager details, this means that we won't be seeing any Service Manager Resources here.

```PowerShell

```

### Objective 2: Find all of the resource group names which existed during this billing period.

```PowerShell

```

### Objective 3: Calculate the cost of each Resource Group, then flag if there is a problem.

```PowerShell

```

### Objective 4: Notify the correct people using a Slack notification.
Finally, we will be reminding people that they still have a test resource which has reached the cost limit for that resource.

```PowerShell

```

## Configure it in Octopus
Create your new Project in Octopus, mine is called `Cloud Cost` then, define these 2 project variables:
- SubscriptionId (The Subscription Id which can be retrieved from Azure PowerShell by running `Get-AzureRmSubscription`)
- DateRangeInDays (The script takes the current day and counts backwards by this many days to form a range to check usage cost)

### Create your first Step (Azure PowerShell Script)
Below is the full script you will need to paste (WIP):

```PowerShell
$SubCostItem = Get-AzureRmConsumptionUsageDetail -StartDate $($now.Date.AddDays(-30)) -EndDate $($now.Date)


#Objective 1: Get all cost items for all subscriptions in Azure
$playgroundItem = $SubCostItem | ? { $_.Id.StartsWith($SubIdPrefix) } 
$resourceGroups = @()
$RgTotal = @()

#Objective 2: Find all of the resource group names which existed during this billing period.
$RgIdPrefixNoName = $SubIdPrefix + "/resourceGroups/"
foreach ($line in $playgroundItem) {
    if ($line.InstanceId -ne $null ) {
        #$thisRgName = $($line.InstanceId).Trim($RgIdPrefixNoName)
        $thisRgName = $($line.InstanceId.ToLower()).Replace($RgIdPrefixNoName.ToLower(),"")
        $toAdd = $thisRgName.Split("/")[0]
        $toAdd = $toAdd.ToString()
        $toAdd = $toAdd.ToLower()
        $toAdd = $toAdd.Trim()

        if ($RgTotal.Name -notcontains $toAdd) {
            $resourceGroups = [PSCustomObject]@{
            Name                = $toAdd
            }
            $RgTotal += $resourceGroups
        }
    }
}

$x = 0

# Objective 3: Calculate the cost of each Resource Group, then flag if there is a problem.
$currentResourceGroups = Get-AzureRmResourceGroup

foreach ($rg in $RgTotal) {
    
    $thisRg = $null

    $RgIdPrefix = $SubIdPrefix + "/resourceGroups/" + $rg.Name
    $ThisRgCost = $null
    $playgroundItem | ? { if ( $_.InstanceId -ne $null) { $($_.InstanceId.ToLower()).StartsWith($RgIdPrefix.ToLower()) } } |  ForEach-Object { $ThisRgCost += $_.PretaxCost   }

    $toaddCost = [math]::Round($ThisRgCost,2)

    $RgTotal[$x] | Add-Member -MemberType NoteProperty -Name "Cost" -Value $toaddCost

    
    
    if ($currentResourceGroups.ResourceGroupName -contains $rg.Name) {
        
        $thisMyRgNameStuff = Get-AzureRmResourceGroup -Name $($rg.Name)

        $RgTotal[$x] | Add-Member -MemberType NoteProperty -Name "OwnerContact" -Value $($thisMyRgNameStuff.tags.OwnerContact)
        

    }

    $x ++

}

$totalCost = $null

foreach ($rgl in $RgTotal) {

  $totalCost += $rgl.cost

}


$RgTotal
```


### Create your second Step (Azure PowerShell Script)
Below is the full script you will need to paste (WIP):
```PowerShell
#slack-notify 
```
