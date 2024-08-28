---
title: Inside DevOps with Thiago Marcolino from YLD
description: A series where we share lessons learned from those on the frontlines of DevOps. This post features Thiago Marcolino, Platform Engineer from YLD.
author: joanna.wyganowska@octopus.com
visibility: public
published: 2024-09-02-1400
metaImage: img-meta-inside-devops-with-thiago-marcolino.png
bannerImage: img-blog-inside-devops-with-thiago-marcolino-2024.png
bannerImageAlt: Photo of Thiago Marcolino
isFeatured: false
tags: 
  - DevOps
  - Inside DevOps
---

This post is the next in our [Inside DevOps series](https://octopus.com/blog/tag/Inside%20DevOps), where we share lessons learned from those on the frontlines of DevOps.  

Hear from [Thiago Marcolino](https://www.linkedin.com/in/thiagomarcolinodasilva/), Platform Engineer with [YLD](https://www.linkedin.com/company/yld/), a software engineering consultancy company. Thiago currently supports [NewDay](https://www.linkedin.com/company/newday/), a leading provider of consumer credit in the UK.

**What is DevOps to you? How do you define it?**

*Thiago:* It’s challenging to deviate from the standard definition of DevOps as a discipline that bridges the gap and fosters collaboration between developers and operations teams. However, over time, I’ve realized that it’s more than just helping developers deploy their code using pipelines. It encompasses multiple competencies such as business rules, company culture, security, cost management, performance, understanding the target audience, and more.

Beyond tools, principles, and code deployment, the culture of a company or team is crucial when discussing DevOps. It fundamentally shapes how these principles are applied and how effectively teams can collaborate. A culture that encourages open communication, continuous learning, and shared responsibility is key to successfully implementing DevOps practices. It not only facilitates smoother workflows but also promotes a sense of shared ownership and accountability, which are essential for the rapid and reliable delivery of high-quality software.

**How did your DevOps journey start?**

*Thiago:* My journey into DevOps began when I was working as a backend engineer, frequently faced with the challenge of deploying our team’s code across various environments. It was not uncommon for the pipelines to fail. The resolution process was often time-consuming due to the limited number of dedicated DevOps engineers who were overwhelmed by the volume of requests. Gradually, I gained the necessary credentials and autonomy to resolve these issues myself. Eventually, I became proficient enough in coding, testing, and delivery to assist other teams with their pipeline issues.

During this time, I noticed a significant shift in the IT industry. There was a growing demand for backend engineers with a more comprehensive skill set. This included traditional development tasks and also a thorough understanding of DevOps practices.

In response to this shift, I broadened my knowledge and expertise in DevOps. I dedicated myself to learning about various tools and methodologies, earning certifications, participating in online training, and attending conferences such as KubeCon. I also became proficient in CI/CD, Infrastructure as Code (IaC), containerization, and observability.

My journey into DevOps has been transformative. It’s equipped me with the skills and confidence to tackle complex challenges, provide more effective support to my team, and stay at the forefront of a rapidly evolving industry. I’m eager to continue my growth in this field and contribute to the success of future projects.

**What’s the most challenging part of DevOps?**

*Thiago:* There are numerous challenges DevOps teams must tackle, but I believe that for even the toughest ones, you can usually find a reasonable solution. For example, if the fundamental principles of DevOps are crumbling and the development is not in sync with operations, the solution could be as simple as encouraging frequent cross-team communication. Or you could hold regular meetings, or introduce automation to enhance visibility and reduce workload. If teams are struggling to understand the pipeline and security procedures, training could be the answer. 

I could cite more scenarios, but all of them will somehow touch on the same topic: even the simplest solution is unachievable without a cultural shift. In fact, any DevOps recommendations require the commitment of the entire organization, particularly from its leaders.

**What’s the most rewarding part of DevOps?**

*Thiago:* The “freedom” to create technical solutions within the constraints of business, security, cost, and maintenance is truly rewarding. At many times, DevOps solutions are highly complex as they require maintaining a balance among various components. Using creativity, research, and overcoming frustrations to achieve an objective that effectively solves a problem is extremely gratifying. 

It’s even more rewarding when usability and maintenance are simplified, and using the solution becomes as easy as fitting Lego pieces together. This sense of accomplishment and the impact it has on the efficiency of operations is what makes DevOps so rewarding.

**What are some DevOps best practices you and your organization have implemented?**

*Thiago:* My top 3 are:

- Infrastructure as Code (IaC): From the early stages of our projects, we’ve prioritized avoiding manual cloud management. We manage almost all cloud changes through IaC tools. We primarily use Terraform, but have also achieved significant results with ARM templates and Microsoft Bicep for applying IaC to Azure. Also, for a specific scenario, Crossplane proved to be an excellent choice, with a smooth implementation process. More recently, one of my clients adopted Pulumi with TypeScript as the primary tool for abstracting the engineering platform. This has been a challenging yet rewarding experience, yielding fruitful results.

- GitOps: I achieved excellent results using GitOps practices with Argo CD to automate processes and ensure that environments match the state defined in the repository. I chose Argo CD primarily for its user-friendly interface, and it has proven to be a great choice. It works seamlessly with the EKS clusters I manage.

- Observability: We’ve implemented comprehensive observability practices to monitor and analyze the performance of our applications and infrastructure. Tools like Prometheus, Grafana, AWS Cloudwatch, and Azure Application Insights have been instrumental in various projects. These tools facilitate real-time monitoring and alerting, and help with quick issue detection and resolution, improving system reliability and performance.

**What’s the biggest challenge Octopus has helped you with?**

*Thiago:* Octopus has been instrumental for our organization in centralizing and managing the provisioning of Azure resources using Infrastructure as Code (IaC) modules such as Terraform, ARM templates, and Bicep. This approach aligns with our organizational principles of simplicity, clonability, scalability, cost efficiency, 24/7 availability, and PCI compliance. By using these IaC modules, we’ve significantly reduced deployment times and addressed several key challenges, including:

- Enforcing naming conventions
- Implementing tagging for cost control and informational purposes
- Achieving 100% IaC coverage
- Ensuring VNet integration
- Maintaining security

**What advice would you give to those just starting their DevOps journey?**

*Thiago:* 

- Learn the basics: Understand the fundamental concepts of DevOps, including Continuous Integration (CI), Continuous Deployment (CD), Infrastructure as Code (IaC), and monitoring.

- Embrace automation: Start automating repetitive tasks, like code deployments, testing, and infrastructure provisioning. This will save time, ensure consistency, and reduce errors.

- Continuous learning: The DevOps landscape is constantly evolving. Stay up-to-date with the latest trends, tools, and best practices. Follow blogs, attend webinars, participate in online communities, and go to conferences on-site when possible.

- Security first: Integrate security practices into your DevOps pipeline (DevSecOps). Ensure you consider security at every stage of the development process to avoid vulnerabilities.

- Get hands-on experience: Practical experience is invaluable. Set up your own projects, contribute to open-source projects, or use your personal GitHub account to practice and showcase your skills.

**What DevOps book would you recommend reading?**

*Thiago:* For those embarking on their DevOps journey, there are 2 books that I believe are particularly beneficial: [*The Phoenix Project*](https://octopus.com/devops/reading-list/#the-phoenix-project) and [*The DevOps Handbook*](https://octopus.com/devops/reading-list/#the-devops-handbook).

*The Phoenix Project* is a novel that provides a unique perspective on the challenges that DevOps aims to solve. It presents a fictional narrative that encapsulates the essence of DevOps, making it an engaging and insightful read.

On the other hand, *The DevOps Handbook* is more technical and straightforward. It’s a comprehensive guide with practical advice and strategies for implementing DevOps in your organization. The book covers a wide range of topics, from Continuous Integration and Delivery to automating infrastructure and architecture. It’s a valuable resource that I always keep close.

Both books offer valuable insights into the world of DevOps from a different angle. Whether you prefer narrative-driven books or a direct, hands-on guide, they’re a good starting point for understanding and implementing DevOps practices.

**What’s one thing about you that might surprise us?**

*Thiago:* This may not surprise you, but it could be unique. I hold a degree in physics, which served as my gateway into the world of programming, as I learned coding in the physics labs. However, if I wasn’t immersed in the IT profession, I would gladly set physics aside once more to embrace a career as a gardener. There is something profoundly satisfying about nurturing plants, and working with soil is something I find irresistibly appealing.

**Thank you for sharing your DevOps insights with us, Thiago; it’s much appreciated.**

:::hint
If you’d like to be featured in our series, [Inside DevOps](https://octopus.com/blog/tag/Inside%20DevOps), please reach out to [Joanna on LinkedIn](https://www.linkedin.com/in/joannawyganowska/) to set up a time for a quick chat.
:::