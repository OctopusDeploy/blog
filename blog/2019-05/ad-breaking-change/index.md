---
title: Active Directory Breaking change in 2019.5
description: What does the Active Directory breaking change mean for my organisation?
author: derek.campbell@octopus.com
visibility: public
published: 2019-05-10
metaImage:
bannerImage:
tags:
  - Product
---

We announced in version 2019.5 that there was a breaking change for Active Directory and I wanted to write a blog to help people understand what it means for your organisation, infosec team and most importantly Administrators of Octopus and you can see this issue on [Github](https://github.com/OctopusDeploy/Issues/issues/5549).

## The background & issue

In most organisations, they use [Principle of Least Privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege). What this means is that technical staff have a standard user as part of the Active Directory domain for their regular duties such as writing documentation, reading emails and browsing the web. They also have access to a privileged account which they use in their daily technical duties which may include deployments, development, resetting users passwords and accessing sensitive systems.  

Before version 2019.5.0, Octopus treated user emails like a key and expected them to be unique. This caused an issue with Active Directory, where there are no such constraints and when multiple users have the same email address Octopus thinks they are the same user. They are the same person, but not the same user in an Active Directory sense. No other authentication provider works this way, but to prevent issues with Active Directory when it's being used this way we need to drop the uniqueness constraint and assume the same person can be using different user accounts tied to one email address.

Active Directory login checking also needs to be changed to support detecting the duplicates and also still detecting if a user has been modified in AD versus a new user logging in for the first time. This is the cover the scenarios where Active Directory admins pick users up and move them to another OU or assign them all new UPNs or SamAccountNames. We've had several customers do this in the past and lose all access to their Octopus instances because the users all suddenly looked different and we treated them as new users.

## The fix and who is affected?

Our fix to this issue which was to ensure that these accounts were not matched and merged based on the email address. This was to ensure if a user named Robert.Jones who has a named Active Directory user Work\Robert.Jones and an admin account named Work\admin-RJ, who both had the email address of Robert.Jones@Work.com were not matched to the same Octopus account. 

If only the administrator account has access to Octopus, then you are not affected. 

If the user accounts you use do not share email addresses, then you are not affected. 

If you do share email accounts between non-administrative and administrator accounts, then you are affected, and we recommend doing a proof of concept upgrade and testing that administrator accounts have the required access after the upgrade. 

If you previously had used the same email address in the past on both users, then you will now be able to have separated accounts that won't be matched and merged based on email address. 

If you're reading this after the fact and you've accidentally locked yourself out of Octopus after a proof of oncept upgrade, we have a way to gain access to an admin account in our [docs](https://octopus.com/docs/api-and-integration/octopus.server.exe-command-line/admin). This will grant you access to your Octopus instance but if you get stuck then get in touch with [Support](mailto:Support@Octopus.com). 

## Conclusion

We believe this is the correct approach to this particular issue for our customers, but if you have any issues, please get in touch with [Support](mailto:Support@Octopus.com). 