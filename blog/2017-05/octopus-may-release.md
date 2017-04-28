---
title: "Octopus May Release 3.13"
description: TODO
author: mark.siedle@octopus.com
visibility: private
tags:
 - New Release
 - Azure Service Fabric
---

This month's release brings some exciting new features including support for Azure Service Fabric, HSTS, optional lifecycles and performance improvements, among other things!

<h2 id="in-this-post">In this post</h2>
<div class='toc'>
  <ul>
    <li class='toc-h2'><a href='#in-this-post'>In this post</a></li>
    <li class='toc-h2'><a href='#release-tour'>Release Tour</a></li>
    <li class='toc-h2'><a href='#introducing-azure-service-fabric-support'>Introducing Azure Service Fabric support</a></li>
    <li class='toc-h2'><a href='#hsts-support'>HSTS support</a></li>
    <li class='toc-h2'><a href='#optional-lifecycles'>Optional lifecycles</a></li>
    <li class='toc-h2'><a href='#browser-caching'>Browser caching</a></li>
    <li class='toc-h2'><a href='#failing-a-script-with-a-message'>Failing a script with a message</a></li>
    <li class='toc-h2'><a href='#modify-task-state'>Modify task state</a></li>    
    <li class='toc-h2'><a href='#upgrading'>Upgrading</a></li>
    <li class='toc-h2'><a href='#wrap-up'>Wrap Up</a></li>
  </ul>
</div>

<h2 id="release-tour">Release Tour</h2>
<iframe width="560" height="315" src="#TODO" frameborder="0" allowfullscreen></iframe>

<h2 id="introducing-azure-service-fabric-support">Introducing Azure Service Fabric support</h2>
We are excited to announce that Octopus now includes first-class support for [Deploying Azure Service Fabric applications](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric).

Since the [RFC](https://octopus.com/blog/rfc-azure-service-fabric) earlier this year, we've been busy creating new step templates to help you connect to and deploy your Service Fabric cluster applications.

These new Service Fabric steps can now assist you with:

- Deploying a Service Fabric App ([learn more](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric/deploying-a-package-to-a-service-fabric-cluster))
- Running a Service Fabric SDK PowerShell Script ([learn more](https://octopus.com/docs/deploying-applications/custom-scripts/service-fabric-powershell-scripts))

Both steps require connection to a cluster. As such, we've included support for security modes including unsecure, Client Certificates and Azure Active Directory.

:::hint
**Service Fabric SDK**
Due to Service Fabric dependencies, you will need to manually install the [Service Fabric SDK](https://g.octopushq.com/ServiceFabricSdkDownload) onto your Octopus Server. Then you can then start using Octopus to help orchestrate your Service Fabric application deployments.
:::

<h3>Want to learn more?</h3>

You can learn more about these new features from our main [Deploying to Service Fabric](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric) documentation. 

We also have a new guide explaining [Continuous Integration for Service Fabric](https://octopus.com/docs/guides/service-fabric) where you can learn how Octopus Deploy fits into a Continuous Deployment pipeline for you Service Fabric applications.

<h2 id="hsts-support">HTTP Strict-Transport-Security (HSTS)</h2>

HTTP Strict Transport Security is an HTTP header that can be used to tell the web browser that it should only ever communicate with the website using HTTPS, even if the user tries to use HTTP. This can substantially lessen your attack surface, and is frequently recommended by security professionals. 

We can now send this header on demand, but as there are some potential complexations, it is not enabled by default. If you have your Octopus Server exposed on the internet, we recommend [reading up on and enabling HSTS](https://octopus.com/docs/how-to/expose-the-octopus-web-portal-over-https#HSTS) if you can.

<h2 id="optional-lifecycles">Optional lifecycles</h2>

TODO

<h2 id="browser-caching">Browser caching</h2>

TODO

<h2 id="failing-a-script-with-a-message">Failing a script with a message</h2>

The message on the deployment overview can now be customised, refer to [failing a script with a message](https://octopus.com/docs/deploying-applications/custom-scripts#failing-a-script-with-a-message)

<h2 id="modify-task-state">Modify Task State</h2>
Have you ever deployed to Production only to have your last step "Email release party invites!" fail?  Or maybe you deployed sucessfully but after some QA decided to roll back. Now you can modify the state of a task.  When a task has completed with the state `Success`, `Failed` or `Canceled` you can edit the state from the task screen by providing the new task state and the reason for the change.  Once submitted, the task state will be updated and an entry in the task history will contain an audit entry with the change.  A new permission called `TaskEdit` is required to perform this action.  By default the `TaskEdit` permission has only been granted to the built-in Administrators team.

<h2 id="upgrading">Upgrading</h2>

<p>All of the usual <a href="https://octopus.com/docs/administration/upgrading">steps for upgrading Octopus Deploy</a> apply.

<h2 id="wrap-up">Wrap Up</h2>

<p>That’s it for this month. We hope you enjoy the latest features and our new release. Feel free to leave us a comment and let us know what you think!  Happy deployments!</p>
