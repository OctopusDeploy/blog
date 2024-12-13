---
title: Why observability is so important for Kubernetes deployments
description: This blog post is an excerpt from our free guide, Kubernetes delivery unlocked which examines why observability is so important to Kubernetes deployments.
author: liam.mackie@octopus.com
visibility: public
published: 2025-01-08-1400
metaImage: blog-kubernetes-unlocked-observability-750x400.png
bannerImage: blog-kubernetes-unlocked-observability-750x400.png
bannerImageAlt: TBC
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
---

Nothing over the last decade has changed deployments more than Kubernetes. It brings many benefits and a new level of flexibility, but also adds a layer of complexity for developers to overcome.

This blog post is an excerpt from our free guide, [Kubernetes delivery unlocked](https://octopus.com/whitepapers/kubernetes-delivery-unlocked), which examines why observability is so important to Kubernetes deployments, and how to implement the principles in practice. 

## Observability in Kubernetes deployments 

Observability is an integral part of the deployment process. It lets you be confident that your service is running as it should be. Plus, the details collected by automatic observability pipelines are often used to drive your pipeline's decision to move forward or roll back a deployment.

Observability is often seen as a group of pillars, but the core of observability is seeing your application from the outside. You (usually) can't pause and debug your application in production, so what do you need to expose to the outside to know what's going on?

The information your application emits usually fits into 3 loose groups:

- Exceptional circumstances
	- Errors
	- Exceptions
	- Crash dumps
	- Stack traces
 - Routine operations
	- Access logs
	- DB migration logs
- Runtime metrics
	- Memory and CPU utilization
	- Request averages
	- Disk IO and utilization

You can mainly expose this information to your Pods through logs, metrics, and traces. Often, you're limited by the application you're running. Not all applications support tracing, for example. Observability, like most engineering disciplines, is a balancing act. You need enough information to know if something is going wrong, and how to fix it. At the same time, you don't want to spend too much money and time collecting and shipping metrics. There are also performance implications to excessive emission of telemetry data, including accidentally filling your disk with logs!

Many platforms let you collect and analyze your telemetry:

- Azure Log Analytics
- AWS CloudWatch
- Google Cloud Logging
- Datadog
- Better Stack
- Elastic Stack
- Sumo Logic

Platforms often have software that automates data collection and shipping. If you're not sure which vendor to choose or want to start with open-source tools, OpenTelemetry is a vendor-neutral library for collecting, aggregating, and shipping telemetry from your cluster and workloads.

There are additional projects that let you aggregate the data that OpenTelemetry collects, and visualize it, like:

- Jaeger: Trace collection and visualization
- Prometheus: Metric collection and visualization
- Loki: Log collection and visualization


### Logs

Kubernetes, by default, captures container logs to stdout and stderr. These logs are available for the current container, and the previous instance of the container (if, for example, it restarted). You can access these logs using most Kubernetes UI tools, the kubectl command-line tool, and often, your CD pipelines. 

While this can be useful, aggregating and annotating these logs at scale is beneficial. The tools mentioned earlier can help you aggregate and query your logs efficiently, helping your troubleshooting efforts. Logs are one of the largest types of telemetry, so it's worth making sure that you're only shipping and keeping the logs that you care about. This will help reduce the cost of storing and ingesting your logs.

### Metrics

Kubernetes doesn't come with any metrics providers pre-loaded, but does include APIs that allow metric collectors to provide data for Kubernetes. The most common way to get metrics working in Kubernetes is with the Prometheus operator.  

Prometheus is an open-source time-series metric server. You can install Prometheus using the kube-prometheus stack . This installs the Prometheus operator, as well as tools and dashboards to monitor Kubernetes, and the workloads on it. It provides visibility over the core metrics of Kubernetes. 

After you install the Kube-Prometheus stack, you can use the metrics provided to scale using horizontal and vertical Pod autoscale. It also alerts you if there are problems with your workloads.

### Traces

Traces help you track a request's path, even between separate microservices.  They can be incredibly useful in distributed systems, where it can be hard to understand where a failure may have originated. 

Tracing is usually enabled as a feature in your workload, and you can then configure it to ship to an internal collector, like the OpenTelemetry Collector, or to an external vendor. 

[Honeycomb](https://www.honeycomb.io/) is an industry leader in observability and a good resource for learning more about tracing.

## Conclusion

Observability is critical to all deployment processes and is even more important when it comes to Kubernetes. It gives you confidence that your service is running as it should be. By making sure you have the right tools and measures in place to access logs, metrics, and traces, you’ll set your deployments up for success. 

This blog post was an excerpt from our new guide, Kubernetes delivery unlocked, which helps you understand:

- How to use Kubernetes' architecture to achieve seamless, zero-downtime deployments
- How to define and build your deployment process using CD principles
- Common challenges faced by developers using Kubernetes and how to overcome them

[Download the free guide, Kubernetes delivery unlocked](https://octopus.com/whitepapers/kubernetes-delivery-unlocked).

Happy deployments! 