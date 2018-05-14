---
title: Service Fabric Deployment Targets
description: TODO
author: mark.siedle@octopus.com
visibility: private
published: 2018-05-21
metaImage: metaimage-will-it-deploy.png
bannerImage: blogimage-will-it-deploy.png
tags:
- Azure
---

With the [release of 2018.5](https://octopus.com/blog/octopus-release-2018.5) and the introduction of [Azure Service Fabric Cluster targets](https://octopus.com/blog/paas-targets), we thought it'd be the perfect opportunity to provide a quick overview of the new Service Fabric deployment targets with Octopus Deploy.

Service Fabric can be more than a little overwhelming if you're just starting out :) Especially if you're simply trying to get a high-level overview of the steps required to create a flexible/repeatable deployment scenario. This post hopes to address that by showing off the new Azure Service Fabric Cluster targets in action!

#Service Fabric: Why Bother?
Azure recently [announced](https://blogs.msdn.microsoft.com/appserviceteam/2018/03/12/deprecating-service-management-apis-support-for-azure-app-services/) that from June 30th 2018 they are retiring support for Service Management API. After this announcement (and much confusion to the community regarding the future of Cloud Services - source: me =P), it was confirmed that Cloud Services will still be supported, but there's a clear push to move people towards the newer Resource Management world. There's also statements like this to consider: "Cloud Services is similar to Service Fabric in degree of control versus ease of use, but it's now a legacy service and Service Fabric is recommended for new development" ([source](https://docs.microsoft.com/en-us/azure/app-service/choose-web-site-cloud-service-vm)).

With Service Fabric being the recommended path for new cloud service development in Azure moving forward, let's see how easy it is to get setup with your own Service Fabric cluster and deploy to it.

#Creating a Service Fabric Cluster on Azure
Azure have made it really easy to get up and running securely with Service Fabric clusters. The [documentation on Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/) and resources have come a long way in the last year or so, and is a great resource for anyone wanting to see some quick-start guides and step-by-step tutorials.

To jump to the fun bits, you can get a walkthrough of [creating an Azure Service Fabric cluster via the Azure portal](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-cluster-creation-via-portal) or, if scripting's more your thing, [creating an Azure Service Fabric cluster via ARM](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-cluster-creation-via-arm).

For this example, we simply created our cluster manually via the Azure portal method.

## Gotcha #1: The Initial Wait Time
When spinning up Service Fabric clusters on Azure, you need to carefully watch the status of your cluster and make sure it says "Ready" before trying to connect or deploy anything. This is worth mentioning because there is typically a long wait time after creating your cluster, until all nodes have been provisioned and "baseline upgrades" of your nodes take place, then finally it all becomes ready. This process can take anywhere from 1-6 hours from what we've observed, depending on the alignment of the moon and the stars on any given day :)

Once you've successfully created your cluster, it should appear like this in your Azure portal:

![Service Fabric Cluster Status](sf-cluster-status.png "width=500")

In this case, we've setup both [Certificate](https://octopus.com/docs/deploying-applications/azure-deployments/deploying-to-service-fabric/connecting-securely-with-client-certificates) and [Azure Active Directory](https://octopus.com/docs/deploying-applications/azure-deployments/deploying-to-service-fabric/connecting-securely-with-azure-active-directory) security modes on our cluster, so we can test connections with either security method.

#Packaging for Service Fabric
Microsoft made it super ~developer~ demo-friendly to deploy to Service Fabric via Visual Studio.

The problem is, demo-friendly != real-world (_"Friends don't let friends right-click publish"_ amirite).

When deploying directly from Visual Studio, Microsoft only _partially_ package all of your files. During the deployment, they actually _call back_ into your source code for the `PublishProfiles` and `ApplicationParameters` (so your package folder from Visual Studio ootb is useless to anything except Visual Studio _sigh_).

![When one facepalm isn't enough](azure-double-facepalm.png "width=300")

To get around this, we've written specific [packaging documentation](https://octopus.com/docs/deploying-applications/azure-deployments/service-fabric/packaging) that explains how to work around this in your real-world deployment software (and not from an over-developed IDE that's sitting on your intern's laptop).

For this example, we've used the [Custom build targets](https://octopus.com/docs/deploying-applications/azure-deployments/service-fabric/packaging#custom-build-targets) section of the packaging documentation to help us copy the `PublishProfiles` and `ApplicationParameters` that we need for our Service Fabric package, to ensure we have everything needed for our deployment.

![Custom Build Targets file](sf-solution-targets.png "width=500")

With our custom build targets, we can actually get a preview of what Octopus will see by packaging in Visual Studio. If you Package your main Service Fabric project:

![Packaging Service Fabric 1](sf-package1.png "width=500")

You should then see a "pkg" folder show up near your build output with both your `PublishProfiles` and `ApplicationParameters` (thanks to the custom build targets file). This is an example of the final structure that Octopus needs from your package.

![Packaging Service Fabric 2](sf-package2.png "width=500")

You can now zip that up using SemVer versioning into a package that can be consumed by Octopus. In this case we've made a file called `MarksServiceFabricAppOfAwesomenessNetCore.1.0.0.zip` which contains that package output mentioned above and uploaded this manually to our Octopus Server's [build-in package repository](https://octopus.com/docs/packaging-applications/package-repositories/pushing-packages-to-the-built-in-repository).

#Installing the Service Fabric SDK
Because Microsoft loves their GAC, before anything deployment-related can work against Service Fabric, your deployment server needs to have the [Service Fabric SDK](https://g.octopushq.com/ServiceFabricSdkDownload) installed and PowerShell script execution needs to be enabled (full instructions can be [found here](https://octopus.com/docs/deploying-applications/azure-deployments/deploying-to-service-fabric/deploying-a-package-to-a-service-fabric-cluster)).

#Time to Deploy
Congratulations. If you've made it this far:

- your cluster is saying it's "Ready"
- you've successfully understood the packaging instructions for creating a standalone Service Fabric package
- you've installed the Service Fabric SDK on your Octopus deployment server and set the PowerShell script execution as described above

Now you're in the deployment world! (the fun bit)

Before we continue, if you're using certificates for authentication, make sure you've [added the certificate to the Octopus Certificate Library](https://octopus.com/docs/deploying-applications/certificates/add-certificate) in preparation for its use in your new Service Fabric Cluster Target.

We now need to create a Service Fabric cluster deployment target in Octopus that will let us communicate with our cluster on Azure.

##Create an Octopus Service Fabric Cluster Target
We can now reference our Service Fabric Cluster as a fully-fledged deployment target within Octopus.

Firstly, head over to `Infrastructure > Deployment Targets`, click `Add Deployment Target` and select `Azure Service Fabric Cluster` from the list:

![Creating Azure Service Fabric Targets - step 1](sf-create-target1.png "width=500")

Next, fill out the details of your Service Fabric Cluster on Azure, remembering to select the right security mode for you. In this case, we're referencing the certificate we uploaded to our Certificate Library earlier:

![Creating Azure Service Fabric Targets - step 2](sf-create-target2.png "width=500")

Hit `Save`, then wait for a health check to complete. If all has gone well, your Octopus Server will have used the Service Fabric SDK that's installed on your server to run a health check against your SF cluster (using the security mode parameters you defined) and will have found your target healthy:

![Creating Azure Service Fabric Targets - step 3](sf-create-target3.png "width=500")

You can now reference this deployment target as part of your deployment process, just like any other target!

##Deploying to our new Service Fabric Cluster Target

For this example, we've created a test project and want to **Deploy a Service Fabric Application** package, so we type in "fabric" from our list of available steps and find the step we want:

![Deploying Service Fabric App package - step 1](sf-step1.png "width=500")

We then need to select the role that this step will be deploying to (the role we assigned to our target) and select our Service Fabric package that we packaged earlier, including the path to our publish profile:

![Deploying Service Fabric App package - step 2](sf-step2.png "width=500")

Hit Save and we're ready to deploy!

After running a deployment, we can see the Azure SF PowerShell cmdlets at work successfully deploying our package to our Azure Service Fabric Cluster:

![Deploying Service Fabric App package](sf-deploy.png "width=500")

Great success!
