---
title: Variable use in Octopus Deploy
description: Find our why it's so hard to see where variables are used.
author: stephen.heise@octopus.com
visibility: public
published: 2022-10-12-1400
metaImage:
bannerImage:
bannerImageAlt: 
isFeatured: false
tags:
 - DevOps
 - Variables
---

Being able to see where variables are used has been a common feature request for Octopus Deploy. 

In this post, I explore some of the issues involved with delivering such a feature.

## Why people want to see variable use

There are many reasons people want to see variable use:

- Determining the purpose of a variable. It may not be obvious from the variable name.
- Changing a variable’s value. Identifying where a variable is used helps determine the impact of a value change and whether it's safe to change.
- Deleting unused variables. Identifying all potential variable uses would help determine if the variable is unused and can safely be deleted.

These use cases are concerned with *if* and *where* the values of variables are actually used. For the last 2 use cases, it's very important that *all* potential uses are identified.

Related reasons to see variable use:

- Identifying duplicate variables (by name or by value). This would allow duplicate variables to be extracted into a library variable set and be shared between the projects that use them.
- Identifying clashing scope issues, for example, indeterministic results or missing values for an environment in the lifecycle.
- Identifying plain text variables that should be sensitive, for example, variable names containing `password`,  `pwd`, `token`, or `apikey`, or containing values that look like API keys and suggesting they be changed to sensitive.

These use cases are concerned with what variables *exist* and what their *values* are. They're not concerned with exactly where they're used. While there certainly may be value in supporting these use cases, we won’t consider them here as they're not directly related to variable *use*.

Let’s have a look at how variable uses might be found and some of the problems encountered along the way.

## Static search

This would use the current API to gather information about places where variables are used. Project variables and library variable sets can be searched. The following uses can be determined using the current API:

- Variable values
- Step parameters in the project’s deployment process or any runbooks
- Uses in step scripts

Just looking at variable values, we run into problems trying to find all the uses. At first glance, it should be easy to look for variable uses in variable values - just search for `#{variablename}`. However, variable values can be defined using [Octostache](https://github.com/OctopusDeploy/Octostache) expressions, which is the variable substitution syntax for Octopus Deploy. **Octostache** expressions are very flexible and allow for some weird and wonderful edge cases, particularly when the [extended syntax](https://octopus.com/docs/projects/variables/variable-substitutions) is used. As an example, consider these variables and their values:

|  Variable | Value |
|---|---|
| `Greeting.English` | `Hello` |
| `Greeting.Māori` | `Kia ora` |
| `Language` | `English` |
| `Greeting` | `#{Greeting.#{Language}}` |

In this example, there's no easy way to determine the `Greeting.English` variable is used. Step properties also support **Octostache** expressions and so have the same problem.

There are many other **Octostache** expressions that would trip up any attempt to find variable uses. **Octostache** expressions allow for flexible and creative ways to use variables, but they make it impossible to find all variable uses.

Variable uses in step scripts (for example, the ‘Inline Source Code’ property of a **Run A Script** step) can also cause problems. Variable names can be almost anything and can be accessed dynamically. For example, we have the following variable names:

- `*(_🤦_*(& KL" P{}$^[p]'!`  (yes, this really is a valid variable name)
- `FrankieSay"Relax"`
- `PowerMax`

And this PowerShell script:

``` PowerShell
# *(_🤦_*(& KL" P{}$^[p]'! is a silly variable name.
Write-Host $OctopusParameters["FrankieSay""Relax"""]
$powerLevel = 'Max'
Write-Host $OctopusParameters["Power" + $powerLevel]
```

Doing a straight text search for variable names generates a false positive for `*(_🤦_*(& KL" P{}$^[p]'!` because it's in a comment. It also misses `FrankieSay"Relax"` and `PowerMax` because they're referenced by a name generated at runtime. It may be possible to use a smarter search to deal with some of the issues above, but like **Octostache** expressions, there are many (perhaps endless) variations that would also cause problems. Trying to catch them all would be impossible.

## Other static searching options

Currently, if a project has been version-controlled using Config as Code, the deployment process will be defined in text files. The intention is to add variables themselves and runbooks to Config as Code in the near future, which will mean that they will also be stored in text files. When that happens, finding variable uses (subject to the above limitations) will be as simple as doing a text search in any text editor.

Another option that will work for non-version controlled projects is to use a PowerShell script to query the current API and search for usages. An example script is below. It will only search for usages of project variables, although could be extended to include library variable sets. It also only does a straight text search, so is subject to all the limitations discussed above and certainly is not guaranteed to find all variable usages.

``` PowerShell
$octopusURL = "http://yourinstance.octopus.app"
$octopusAPIKey = "API-Key"
$spaceName = "Default"

# Set up
$usages = @()
$header = @{ "X-Octopus-ApiKey" = $octopusAPIKey }
$space = (Invoke-RestMethod -Method Get -Uri "$octopusURL/api/spaces/all" -Headers $header) | Where-Object { $_.Name -eq $spaceName }
$projects = Invoke-RestMethod -Method Get -Uri "$octopusURL/api/$($space.Id)/projects/all" -Headers $header

foreach ($project in $projects) {
    # Get project variables
    $projectVariableSet = Invoke-RestMethod -Method Get -Uri "$octopusURL/api/$($space.Id)/variables/$($project.VariableSetId)" -Headers $header
    $variableNames = ($projectVariableSet.Variables).Name | Sort-Object -Unique

    foreach ($variableName in $variableNames) {
        $escapedVariableName = $variableName -replace '[\[]', '`$&'
        foreach ($sourceVariable in $projectVariableSet.Variables) {
            if ($sourceVariable.Value -like "*#{$escapedVariableName}*") {
                $usages += [pscustomobject]@{
                    Project  = $project.Name
                    Variable = $variableName
                    Source   = "Project variable value, Project variable: $($sourceVariable.Name)"
                    Usage    = $sourceVariable.Value
                    Link     = "$octopusURL$($project.Links.Web)/variables"
                }
            }
        }
    }

    # Get project deployment process
    $url = "$octopusURL/api/$($space.Id)/deploymentprocesses/$($project.DeploymentProcessId)"
    $steps = (Invoke-RestMethod -Method Get -Uri $url -Headers $header).Steps

    foreach ($step in $steps) {
        foreach ($variableName in $variableNames) {
            $escapedVariableName = $variableName -replace '[\[]', '`$&'
            $properties = $step.Actions.Properties | Get-Member | Where-Object { $_.MemberType -eq "NoteProperty" -and $_.Definition -like "*$escapedVariableName*" }
            foreach ($property in $properties) {
                $usages += [pscustomobject]@{
                    Project  = $project.Name
                    Variable = $variableName
                    Source   = "Deployment process, Step: $($step.Name), Property: $($property.Name)"
                    Usage    = $property.Definition
                    Link     = "$octopusURL$($project.Links.Web)/deployments/process/steps?actionId=$($step.Actions[0].Id)"
                }
            }
        }
    }

    $url = "$octopusURL/api/$($space.Id)/projects/$($project.Id)/runbooks"
    $runbooks = (Invoke-RestMethod -Method Get -Uri $url -Headers $header).Items

    foreach ($runbook in $runbooks) {
        Write-Host $runbook.Name

        $url = "$octopusURL/api/$($space.Id)/runbookProcesses/$($runbook.RunbookProcessId)"
        $steps = (Invoke-RestMethod -Method Get -Uri $url -Headers $header).Steps

        foreach ($step in $steps) {
            foreach ($variableName in $variableNames) {
                $escapedVariableName = $variableName -replace '[\[]', '`$&'
                $properties = $step.Actions.Properties | Get-Member | Where-Object { $_.MemberType -eq "NoteProperty" -and $_.Definition -like "*$escapedVariableName*" }
                foreach ($property in $properties) {
                    $usages += [pscustomobject]@{
                        Project  = $project.Name
                        Variable = $variableName
                        Source   = "Runbook: $($runbook.Name), Step: $($step.Name), Property: $($property.Name)"
                        Usage    = $property.Definition
                        Link     = "$octopusURL$($project.Links.Web)/operations/runbooks/$($runbook.Id)/process/$($runbook.RunbookProcessId)/steps?actionId=$($step.Id)"
                    }
                }
            }
        }
    }
}

$usages | Sort-Object "Project", "Variable", "Source"  | Format-Table -AutoSize
```

## Deployment-time search

This type of search captures variable uses as they happen during a deployment. There are 3 features available in all deployment steps that can use variables to perform various types of replacements:

- Structured Configuration Variables
- .NET Configuration Variables
- Substitute Variables in Templates

When these replacements are done, any matches could be sent back to the Octopus Server and stored. One advantage of this approach is that it can capture replacements in files that are sourced from outside of Octopus Deploy. Building the infrastructure into Octopus Deploy to support this type of search wouldn't be straight forward.

It's interesting to consider how thius type of search would work. Steps can be conditional and may only execute during a production deployment, for example. It therefore may require multiple deployments to capture all variable uses. Should Octopus Deploy try to capture variable uses from multiple deployments and somehow collate them? At what point should they be reset? How could a user be confident that uses from all possible deployment types have been captured?

Ultimately, no matter how this feature is implemented, it requires careful setup and input from the user to capture uses. It certainly wouldn't be a straightforward, single click to get all the uses.

## Conclusion

A static search and deployment-time search would both capture different types of variable uses. Neither can be guaranteed to capture all uses, but they would capture most uses in most circumstances. Missed uses would mostly be where variables are being used in unusual or creative ways.

The question now becomes should we build this? Is there enough value in a feature that will *probably* work *most* of the time? To be confident that a variable really is unused and can safely be removed, *all* variable uses need to be identified - something that can't be guaranteed. Even if this limitation is made very clear in the UI, it's inevitable some users will get caught out.

As the deployment-time search would require a significant effort to build and would not be straightforward to use, should we just build the static search? It seems like it would be a ‘half solution’. How much value would there be in adding a static search to the UI when the same functionality can already be achieved with a PowerShell script?

What do you think? We'd love to hear your thoughts in the comments below.

Happy deployments!