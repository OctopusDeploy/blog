---
title: Introducing the Octopus Datadog integration
description: Learn how Octopus and Datadog give you visibility across your entire CI/CD pipeline, helping you correlate data, to find and solve issues faster.
author: tegan.ali@octopus.com
visibility: public
published: 2025-06-02-1400
metaImage: img-blog-datadog-octopus-integration-2025.png
bannerImage: img-blog-datadog-octopus-integration-2025.png
bannerImageAlt: Octopus and Datadog logos connected by plugs with little stars around the connection.
isFeatured: false
tags: 
  - Product
  - Continuous Delivery
  - Integrations
  - Peformance
---

Octopus is a best-of-breed platform that provides Continuous Delivery at scale and integrates with all your tools across the software delivery pipeline. Datadog is the best-of-breed monitoring platform that monitors, secures, and provides alerts for all the tools in your DevOps stack. This includes infrastructure monitoring, application performance, and software delivery pipelines. 

Now, with the Octopus Datadog integration, it's even easier to use Octopus and Datadog together. 

The Octopus Datadog integration monitors Octopus deployments through the Datadog Agent. This gives customers using Datadog and Octopus visibility across their entire CI/CD pipeline, helping you correlate data, to find and solve issues faster.

In this post, we introduce the Octopus Datadog integration. We explain how it helps orchestrate the insights from all your tools and connect the dots when something goes wrong. 

## The need for better observability across your entire pipeline

Without robust monitoring in place, it’s challenging to get visibility across your entire CI/CD pipeline and connect problems with the issues that caused them. Sometimes you don’t even have access to all the systems in your stack.  

For teams using Kubernetes, you might get an alert when an issue arises, but often you won’t have access to the Kubernetes cluster. You're left making guesses and assumptions based on what changed.

If you’re using legacy systems, it’s a struggle to debug your systems when few, if any, people on the team have in-depth knowledge about them anymore. Plugins fall out of date, and performance and reliability begin to wane, so issues arise more often. It can take days to find and fix problems. This has flow-on effects for other work, like urgent security patches, and slows down releases, affecting due dates. 

Modern tools and practices are better designed to prevent issues from occurring and make troubleshooting easier, but it’s still difficult to get visibility across your entire CI/CD pipeline without an observability tool. We also know that many organizations are modernizing over time, so they rely on a mix of modern and heritage technologies. Luckily, both Datadog and Octopus provide best-of-breed solutions for monitoring and delivering software at scale, regardless of technology. 

## Monitoring and observability in Octopus

Octopus already provides great visibility into all your deployments on our dashboards. You can see the stage, environment, and state of each deployment across multiple repositories in one place. You can also dive into the details of your projects and releases from the bird's-eye view of your dashboard. 

This year, we strengthened our observability and monitoring with [Kubernetes Live Object Status](https://octopus.com/blog/kubernetes-live-object-status), which gives you real-time updates about the state of your applications running on Kubernetes. You can:

- Get the live status of your applications on the main and project dashboards.
- Get the list of all deployed objects and their live status.
- Review object events, logs, and manifests running in a cluster.
- Compare the applied (desired) and live (currently running) manifests for any Kubernetes object.

Octopus's [DevOps Insights](https://octopus.com/docs/insights) uses DORA metrics to tell you exactly how you're performing, so you can find areas for improvement in your deployment processes.

## Why use Octopus and Datadog together?

While Octopus provides visibility and monitoring for your deployments at scale, it can be hard to correlate data across the entirety of your CI/CD pipeline when things happen in systems outside your deployment pipeline. 

That’s where Datadog helps. It monitors everything in your stack, covering all infrastructure, systems, apps, and services in one place. 

Used together, Octopus and Datadog become part of your world-class pipeline, with both tools supporting software at scale and integrating with any technology. You get a single pane of glass for monitoring, making it easier to track, visualize, and improve key metrics across all projects.

## Better observability and faster recovery

The new Octopus Datadog integration makes it easy for Datadog to collect metrics from Octopus so you can centralize your monitoring.

For DevOps and platform engineers, you can track the performance of your teams’ pipelines and visualize key metrics across all projects in one place.

For site reliability engineers (SREs) and operations folks, the Octopus Datadog integration lets you monitor your applications running in production. You can see logs, detect issues fast, and debug issues with critical context to resolve them quickly.

If you’re deploying to Kubernetes but don’t have access to the Kubernetes cluster, you can check the status in Octopus using Kubernetes Live Object Status. Using both Octopus and Datadog, you can see what’s broken, then easily redeploy in Octopus using [runbooks](https://octopus.com/docs/runbooks) to roll back or forward. 

If you’re using heritage systems, Datadog monitors those, too, and provides alerts so you can find issues fast. You can easily redeploy in Octopus to fix configuration values that teams sometimes change directly. Octopus also lets you restart web servers quickly and reliably with runbooks. 

## How the integration works

The Octopus Datadog integration monitors Octopus deployments through the Datadog Agent. 

The Datadog Agent collects:

- Metrics used for performance monitoring, like average deployment time per environment, and deployment failure rate for a project. 
- Deployment logs, which are useful for debugging failed deployments. 
- Server logs, which provide diagnostic information about the Octopus server. 

After the integration is set up to send data to Datadog, you can use out-of-the-box dashboards to get an overview of each deployment by project. Build monitors alert you of notable events, like a failed deployment, so you can take action promptly. Octopus's [audit log stream](https://octopus.com/docs/security/users-and-teams/auditing/audit-stream#configure-audit-stream) can further enrich this data, which you can combine within Datadog for more insights.

## Who can use the integration

The integration is available to any Octopus and Datadog customers with [Datadog’s Infrastructure Monitoring](https://www.datadoghq.com/dg/enterprise/it-infrastructure-monitoring/).

The metrics sent from the integration, the dashboard, and the monitors, are all standard as part of the integration. (Logs collected from the integration are subject to [logs pricing with Datadog](https://docs.datadoghq.com/account_management/billing/pricing/#log-management).)

To set up the Octopus Datadog integration, [follow the simple instructions provided by Datadog](https://docs.datadoghq.com/integrations/octopus_deploy/).

## Conclusion

Datadog is a best-of-breed monitoring and observability platform, letting you see inside any stack, any app, at any scale, anywhere. When integrated with Octopus as a best-of-breed Continuous Delivery platform, you get world-class observability and CD that let you find problems fast, recover rapidly, and deliver software quickly and reliably at scale.

If you're already using both Octopus and Datadog, you can [set up the integration by following the instructions in Datadog's docs](https://docs.datadoghq.com/integrations/octopus_deploy/).

If you're a Datadog customer interested in using Octopus for your Continuous Delivery, you can [sign up for a free Octopus trial](https://octopus.com/start) or [request a demo](https://octopus.com/lp/schedule-a-demo). 

You can also see the Octopus Datadog integration in action by visiting us at DASH, Datadog’s annual observability conference, in New York City, June 10–11, 2025. We’ll be at booth S-7.

Happy deployments!
