---
title: "Loading External Variables During a Deployment"
description: It turns out that combining a few variable features provides a simple mechanism for dynamically loading and using external variables
author: robert.erez@octopus.com
visibility: private
tags:
 - New Releases
---
At Octopus Deploy, we pride ourselves on the level of support that we provide to our customers and always attempt to respond to tickets within a business day. Sometimes they are bug reports (not that _I_ ever write buggy code right?) but often it is just users looking for some guidance on how to help them achieve some particular deployment pattern in Octopus. An interesting question was brought to my attention last week that resulted in discovering a useful ability that could be accomplished by combining a couple of our [variable](https://octopus.com/docs/deployment-process/variables) features. It's a question I had heard before so thought it might be a nice trick to share.

One of the questions asked in the email was:
> ...is there a way in Octopus to have variables managed from outside the tool...

As our readers are probably aware, the variables available during a deployment can be contributed through various places throughout Octopus, through Project variables, tenant variables, Library variables or even the targets themselves. But what about if you want to make use of variables in your deployment from _outside_ Octopus itself?

The first reaction might be to just encourage the use of the [API](https://octopus.com/docs/api-and-integration/api) to run a process externally that keeps Octopus variables in sync with the external source. But that feels like something that would lead to the perfect example of yak shaving. Can't we do better?

First lets start a step that actually obtains these variables during the deployment.

In this case we will retrieve the variables from a package but it could come from anywhere, possibly an external HTTP configuration store or somewhere on the local filesystem.

The important thing is that the variables are formatted as JSON. With that JSON blob What we then do is set these variables into an output variable so that we can access it in the next set of steps.

 But having to remember the step name and write the long verbose name to access an output variable can be tiresome, so lets add a project variable that itself resolves directly to that value.




