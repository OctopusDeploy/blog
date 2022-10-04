---
title: Variable use in Octopus Deploy
description: Find out why it's so hard to see where variables are used.
author: stephen.heise@octopus.com
visibility: public
published: 2022-10-12-1400
metaImage:
bannerImage:
bannerImageAlt:
isFeatured: false
tags:
 - Engineering
 - DevOps
 - Variables
---

Many customers have requested the ability to see where variables are used in Octopus Deploy.

In this post, I explore some of the challenges of finding all the places a variable might be used.

## Why people want to see variable use

There are many reasons people want to see variable use:

- Determining the purpose of a variable. It may not be obvious from the variable name.
- Changing a variable’s value. Identifying where a variable is used helps determine the impact of a value change and whether it's safe to change.
- Deleting unused variables. Identifying all potential variable uses would help determine if the variable is unused and can safely be deleted.

These use cases are concerned with *if* and *where* the values of variables are actually used. When you're changing a variable's value or deleting unused variables, it's very important that *all* potential uses are identified.

Related reasons people want to see variable use:

- Identifying duplicate variables (by name or by value). This allows duplicate variables to be extracted into a library variable set and be shared between the projects that use them.
- Identifying clashing scope issues, for example, indeterministic results or missing values for an environment in the lifecycle.
- Identifying plain text variables that should be sensitive, for example, variable names containing `password`,  `pwd`, `token`, or `apikey`, or containing values that look like API keys and suggesting they be changed to sensitive.

These use cases are concerned with which variables *exist* and what their *values* are. They're not concerned with exactly where they're used. While there may be value in supporting these use cases, we won’t consider them in this post, as they're not directly related to variable *use*.

Let’s have a look at how variable uses might be found and some of the challenges encountered along the way.

## Static search

A static search would use the current API to gather information about places where variables are used. Project variables and library variable sets can be searched. The following uses can be determined using the current API:

- Variable values
- Step parameters in the project’s deployment process or any runbooks
- Uses in step scripts

With variable values, we run into problems trying to find all the uses. At first glance, it should be easy to look for variable uses in variable values - just search for `#{variablename}`. However, variable values can be defined using [Octostache](https://github.com/OctopusDeploy/Octostache) expressions, which is the variable substitution syntax for Octopus Deploy. **Octostache** expressions are very flexible and allow for some weird and wonderful edge cases, particularly when the [extended syntax](https://octopus.com/docs/projects/variables/variable-substitutions) is used. As an example, consider these variables and their values:

|  Variable | Value |
|---|---|
| `Greeting.English` | `Hello` |
| `Greeting.Māori` | `Kia ora` |
| `Language` | `English` |
| `Greeting` | `#{Greeting.#{Language}}` |

In this example, there's no easy way to determine where the `Greeting.English` variable is used. Step properties also support **Octostache** expressions and so have the same problem.

There are many other **Octostache** expressions that would trip up any attempt to find variable uses. **Octostache** expressions allow for flexible and creative ways to use variables, but they make it difficult to find all variable uses.

Variable uses in step scripts (for example, the inline source code property of a **Run A Script** step) can also cause problems. Variable names can be almost anything and can be accessed dynamically. For example, we have the following variable names:

- `*(_🤦_*(& KL" P{}$^[p]'!`  (this really is a valid variable name)
- `FrankieSay"Relax"`
- `PowerMax`

And this PowerShell script:

``` PowerShell
# *(_🤦_*(& KL" P{}$^[p]'! is a silly variable name.
Write-Host $OctopusParameters["FrankieSay""Relax"""]
$powerLevel = 'Max'
Write-Host $OctopusParameters["Power" + $powerLevel]
```

Doing a straight text search for variable names generates a false positive for `*(_🤦_*(& KL" P{}$^[p]'!` because it's in a comment. It also misses `FrankieSay"Relax"` and `PowerMax` because they're referenced by a name generated at runtime. It may be possible to use a smarter search to deal with some of the issues above, but like **Octostache** expressions, there are many (perhaps endless) variations that would also cause problems. Trying to catch them all feels impossible.

## Other static searching options

Currently, if a project has been version-controlled using Config as Code, the deployment process and variables are defined in text files. Finding variable use (subject to the above limitations) can be done by a simple text search in any text editor. We're planning to add runbooks to Config as Code in the future, which will mean a text search will also find similar variable uses in runbooks.

Another option for non-version controlled projects is to use a PowerShell script to query the current API and search for variable uses. Two example scripts are below. They only perform a straight text search, so they're subject to all the limitations mentioned above and they're not guaranteed to find all variable uses.

- [Find variable usage](https://octopus.com/docs/octopus-rest-api/examples/variables/find-variable-usage)
- [Find library variable set variables usage](https://octopus.com/docs/octopus-rest-api/examples/variables/find-variableset-variables-usage)

## Deployment-time search

This type of search captures variable uses as they happen during a deployment. There are 3 features available in all deployment steps that can use variables to perform various types of replacements:

- Structured configuration variables
- .NET configuration variables
- Substitute variables in templates

Every time a variable match and replacement is done, the details could be sent back to the Octopus Server and stored. One advantage of this approach is that it can capture replacements in files sourced from outside of Octopus Deploy. Building the infrastructure into Octopus Deploy to support this type of search wouldn't be straightforward.

It's interesting to consider how this type of search would work. Steps can be conditional and may only execute during a production deployment, for example. It therefore might need multiple deployments to capture all variable uses. Should Octopus Deploy try to capture variable uses from multiple deployments and somehow collate them? At what point should they be reset? How could a user be confident that uses from all possible deployment types have been captured?

Ultimately, no matter how this feature is implemented, it requires careful set up and input from the user to capture variable uses. It wouldn't be as simple as a single click to get all the uses.

## Conclusion

A static search and deployment-time search would both capture different types of variable uses. Neither can be guaranteed to capture all uses, but they would capture most uses in most circumstances. Missed uses would mostly be where variables are being used in unusual or creative ways.

The question now becomes, should we build this? Is there enough value in a feature that will *probably* work *most* of the time? To be confident that a variable really is unused and can safely be removed, *all* variable uses need to be identified - something that can't be guaranteed. Even if this limitation is made very clear in the UI, it's inevitable some users will get caught out.

As building a deployment-time search would require a significant effort and wouldn't be straightforward to use, should we just build the static search? That feels like a half solution. How much value is there adding a static search to the UI when the same functionality can already be achieved with a PowerShell script?

What do you think? We'd love to hear your thoughts in the comments below.

Happy deployments!