---
title: Jenkins security tips
description: As Jenkins connects to many industry tools, it's a good idea to keep your instance as secure as possible. Here are our tips to keep your team safe.
author: andrew.corrigan@octopus.com
visibility: public
published: 2022-01-25-1400
metaImage: blogimage-jenkinsconfigurationsecurity-2022.png
bannerImage: blogimage-jenkinsconfigurationsecurity-2022.png
bannerImageAlt: Padlock password icon and security badge icon beside a computer screen with security shield icon on screen.
isFeatured: false
tags:
  - DevOps
  - CI Series
  - Continuous Integration
  - Jenkins
---

For such an open, customizable platform, Jenkins provides decent security even in its default state. Given it connects to countless industry tools (Octopus included), though, there are a few other ways to help protect your projects.

In this post, we look at some of the methods and tools to keep your Jenkins instance safe, secure, and protect those using it.

## Keep everything updated

[As December 2021 reminded us](https://octopus.com/blog/octopus-deploy-log4j-response), software vulnerabilities come to light at any time. Software providers not only update their applications to fix bugs or add new features, but also to remove security exploits.

[Jenkins have a security advisories page](https://www.jenkins.io/security/advisories/) to keep you informed about vulnerabilities for their platform. It's still a good idea, however, to keep your instance updated, including its plugins.

To check for updates in Jenkins:

1. Click **Manage Jenkins** from the menu.
1. The **Manage Jenkins** screen will tell you at the top if there's a new version available. Click the **Or Upgrade Automatically** button to upgrade straight away. Otherwise, you can download the latest version and upgrade at a scheduled time.

You can also roll back an upgrade from the same screen - just click the **Downgrade** button.

To update Jenkins plugins:

1. Click **Manage Jenkins** from the menu.
1. Click **Manage Plugins**.
1. Make sure you're on the **Updates** tab, tick the updates you want to install and click **Download now and install after restart**.
1. Restart Jenkins to complete updates.

You can install Jenkins on most major operating systems and containers, so keep those updated too. Seek out your operating system's documentation for more information on how.

## Only change Jenkins' security defaults if you're sure

Jenkins enables most of its security features on install to make things as secure as possible. Given the many ways you can use Jenkins, though, there's no 'one size fits all' approach for how best to configure or lock down your instance.

So while we can't offer advice on what's best for your team (with an exception we'll explore next), usefully, Jenkins provides detailed documentation on the important features you should look at. See the [Securing Jenkins page](https://www.jenkins.io/doc/book/security/) for help with security related to:

- Basic setup
- Build behavior
- User interface

You should only make changes with careful consideration and, if possible, a chat with your cyber security specialist. You can make these changes in the **Configure Global Security** page – find it by selecting **Manage Jenkins** from the left menu.

## Avoid building on your controller

Jenkins offers a built-in node so you can run tests as soon as possible to see if it's the solution for you. Builds that run on a single instance, however, have access to your operating system's file system. For this reason, Jenkins recommends you have jobs run on ‘agents' instead (this happens in a scalable setup, which we talked about in our last post, [Using dynamic build agents to automate scaling in Jenkins](https://octopus.com/blog/jenkins-dynamic-build-agents)).

Agents are virtual Jenkins instances that run jobs instead of your controller. When using agents, you can prevent your controller from running builds to limit access to files that can do harm.

To stop your controller from running builds:

1. Click **Manage Jenkins** from the menu.
1. Click **Manage Nodes and Clouds**.
1. Click the cog to the right of the **Built-In Node**.
1. You have 2 options to prevent builds on the controller. Choose one and click **Save**:
   - Change the **Number of executors** to **0** if you never want to build on the controller.
   - Select **Only build jobs with label expressions matching this node** from the **Usage** dropdown if you want to build on the controller when needed.

## Only give your team access to what they need

Security is more than just protecting yourself from incoming threats. It's also about protecting your environment from within because accidents can happen. And they're more likely to happen if:

- You're running a Jenkins instance with a single admin account
- Everyone has access to everything
- People can change things they shouldn't change

Here are a few suggestions for managing your user access.

### Give each Jenkins user an account

To help track what your users are doing, create individual user accounts for anyone using your Jenkins instance. This way you can see all activity and who's done what.

To create extra users:

1. Click **Manage Jenkins** from the menu.
1. Scroll down and select **Manage Users**.
1. Click **Create User** from the left.
1. Complete all fields and click **Create User**.

### Use the Matrix Authorization Strategy plugin

We recommend using the [Matrix Authorization Strategy plugin](https://plugins.jenkins.io/matrix-auth/) to manage user access to Jenkins on a more granular level. For example, with this plugin you could:

- Restrict users' access so they can only see and manage builds for the projects they're part of
- Give read-only access to project managers so they can see how builds are progressing

To install the plugin:

1. Click **Manage Jenkins** from the left menu.
1. Click **Manage Plugins**.
1. Click the **Available** tab and start typing `Matrix Authorization`. The plugin will appear in the predicted results.
1. Tick the box to the left of the plugin and click **Install without restart**.

To set permissions with the plugin:

1. Click **Manage Jenkins** from the menu.
1. Click **Configure Global Security**.
1. Click the radio button for either:
   -	**Matrix-based security** – allows you to manage global user and group permissions.
   -	**Project-based Matrix Authorization Strategy** – allows you to manage user and group permissions at a project level.
1. Regardless of your choice, use the buttons to add users or groups, and select their level of access using the checkboxes in the table. Click **Save** when you're done.

### Other user access plugins you should consider
If you already use other systems for access management, you might be able to authenticate your Jenkins users with those. For example, there are plugins for both [Microsoft's Active Directory](https://plugins.jenkins.io/ui/search?sort=relevance&categories=&labels=&view=Tiles&page=1&query=Active%20Directory) and [OpenID](https://plugins.jenkins.io/ui/search?sort=relevance&categories=&labels=&view=Tiles&page=1&query=OpenID), which can save you from managing access in more than one spot.

We also recommend looking at both the [Folders](https://plugins.jenkins.io/cloudbees-folder/) and [Folder-based Authorization Strategy](https://plugins.jenkins.io/folder-auth/) plugins. 

The Folders plugin allows you to group jobs as you want in nestable folders. This plugin lets you group jobs that share security needs, which helps you keep a closer eye on them. 

The Folder-based Authorization Strategy plugin extends security for folders, by letting you set folder access using roles.

## Securely store your credentials

The [Credentials Binding plugin](https://plugins.jenkins.io/credentials-binding/) is the best option for encrypting and securely storing credentials that connect Jenkins with other services. Jenkins recommends it too – as one of their suggested plugins when installing Jenkins for the first time. Plus, plenty of other plugins use it as a dependency.

This plugin lets you store and reuse all types of authentication methods, such as:

- Username and passwords
- SSH usernames and private keys
- Secret files
- Secret text
- Certificates

We'll cover the Credentials Binding plugin in detail in a future post.

## Conclusion
As you can see, there are plenty of ways to ensure safe use of Jenkins to protect projects from risks outside and within. Check [Jenkins' documentation](https://www.jenkins.io/doc/book/security/) for even more information on keeping your instances secure.

Check out our other posts about configuring Jenkins:

- [Using dynamic build agents to automate scaling in Jenkins](https://octopus.com/blog/jenkins-dynamic-build-agents)
- [Managing credentials in Jenkins](https://octopus.com/blog/managing-jenkins-credentials)

!include <jenkins-webinar-jan-2022>

!include <q1-2022-newsletter-cta>

Happy deployments!
