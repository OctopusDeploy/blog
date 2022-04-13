---
title: Monoliths vs Microservices
description: A brief summary of the post, 170 characters max including spaces.
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Runbooks Series
  - Runbooks
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

Monoliths and microservices are two different approaches to building software. The choice between these two approaches can significantly impact development and performance. This blog discusses:

 - The difference between monoliths and microservices
-  Which one is better for enterprise software applications 
-  Some challenges of both approaches

## What are monoliths and microservices?

A monolith is a singular executable software application. A monolith contains one codebase that defines the user interface, application logic, and database required to run an application. A monolithic application is built and maintained as a single, indivisible unit and is served from a central location.

By contrast, a microservice architecture breaks down an application into smaller independent units. Microservices are connected to make the whole application through Application Programming Interfaces (APIs). An API allows microservices to communicate with each other to execute a task. For example, a microservice sends input data to a second microservice. The second microservice sends input data to several other microservices, and so on. 

Microservices are modularized. The components that make up a microservice architecture are built independently and only become associated with one another when required to create an application. A database component built as part of a microservice could be part of two different software applications through software reuse.

The following diagram visualizes the differences between monoliths and microservices.

## Diagram
<!-- Placeholder Image, get design to create a Octopus Image -->

<!--![Monolith vs Microservices](monolith-vs-microservices.jpg "width=500") -->

## Which one is better for enterprise software applications?

Monolithic software is the traditional way of building software applications and has predated microservices for many years. The advancement of cloud technologies, particularly containerization, has allowed microservices to become very popular in delivering software. A Google Trends search shows that microservices started to become popular after 2013. 2013 was also when Docker (the leading containerization technology) was released.

![Google Trends Microservices](google-trends-microservices.png "width=500")

Microservices are more agile than monoliths. Building, testing, and deploying a monolithic application involves the entire application. Each microservice can be built and tested independently without building a larger application. The agility of microservices allows them to be more reusable than monoliths. To reuse code from a monolith, developers create new interfaces for the new application. Through APIs, microservices have immediate interfaces with other applications, making them easier to reconfigure than monoliths.

A container is a lightweight, portable computing environment with all the necessary files to run independently. Containerization is the process of making an application runnable as a container. Containerization has helped microservices gain popularity and widespread adoption through scaling. Microservice applications can leverage cloud orchestration tools like Kubernetes to schedule and scale workloads. Suppose an application contains a database and a web front end, but the database is also serving a second application. A Kubernetes node dynamically manages load for the front end and database by increasing or decreasing load on each service, depending on demand. The Kubernetes node services two applications, and only uses the resources required for the task. A monolith can only scale by adding more nodes that scale the entire application, leading to unnecessary costs.

Enterprise software applications are applications that meet an organization's needs. Enterprise software applications operate at scale and compete with other products for market share. Time to market, scale, iterative feedback, and cost are important for enterprise software applications. Microservices are more agile and flexible than monoliths. These benefits allow microservice applications to be delivered faster to market and achieve cost savings at scale. Microservices can be delivered iteratively with feedback instead of delivering a monolith as a single executable. The benefits of microservices make them well suited for enterprise software applications over monoliths.

## The challenges of microservices over monoliths

Despite the many benefits, there are some challenges when using microservices over monoliths. An application using microservices may have to make hundreds of API calls for one function. A monolith application would only need one call to do the same function. Monoliths can have higher performance at scale for applications that require several functions, as there is no need for multiple API calls per function.

Both the monolithic and microservice architecture suffer from complexity issues. Developing a monolith means the code base of the application becomes very large. No single developer or team likely understands the application in its entirety. Microservices contain multiple interacting services. Each service may use a different framework, and combining numerous services increases the complexity of the application. The number of services in microservices also increases the demand for documentation as the number of services increases.

Microservice downtime can cause cascading effects in the application. Even though microservices are independent, they are connected. If one microservice sends incorrect outputs, this may cause other microservices to behave unpredictably. Microservices are decentralized, which introduces a networking problem. Firewalls, latency, and security are more significant problems in microservices than in monoliths, where an application is self-contained and runs in a central location.

## Conclusion

Monoliths and microservices are two different approaches to software development. Monoliths have predated microservices for several years. Around 2013, microservices started to become popular. The growth in popularity may be due to improved cloud and container technologies. Monoliths are a singular executable that provides high performance but lowers agility. Microservices are independent services that make up a larger application. Microservices provide high agility, flexibility, and scale, making them better suited for enterprise software applications. 


!include <q2-2022-newsletter-cta>

Happy deployments!
