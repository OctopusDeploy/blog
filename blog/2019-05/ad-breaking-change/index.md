---
title: Active Directory Breaking change in 2019.5
description: What does the Active Directory breaking change mean for my organisation?
author: derek.campbell@octopus.com
visibility: public
published: 2019-05-10
metaImage:
bannerImage:
tags:
  - Security
---

We recently announced in version 2019.5 that there was a breaking change for Active Directory and I wanted to write a blog to help people understand what it means for your organisation, infosec team and most importantly Administrators of Octopus. 

## The background & issue

In most organisations, they sensibly use [Principle of Least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege). What this means is that technical staff will have a normal user as part of the domain for their normal duties such as writing documentation, reading emails and browsing the web. They will then have a privileged account which they use in their day to technical duties which may include Deployments, Development, resetting users passwords and accessing sensitive systems.  

Before version 2019.5.0 Octopus treated user Emails like a key, and expected them to be unique. This caused an issue with Active Directory, where there is no such constraints and when multiple users have the same email address Octopus thinks they are the same user. They are the same person, but not the same user in an Active Directory sense. No other provider works this way, but to prevent issues with Active Directory when it's being used this way we need to drop the uniqueness constraint and assume the same person can be using different user accounts tied to one email address.

Active Directory login checking also needs to be changed to support detecting the duplicates and also still detecting if a user has actually been modified in AD versus a new user logging in for the first time. This is the cover the scenarios where AD admins pick users up and move them to another OU or assign them all new UPNs or SamAccountNames. We've had a number of customers do this in the past and lose all access to their Octopus instances because the users all suddenly looked different and we treated them as new users.

## The fix and who is affected?

Our fix to this issue which was raised by one of our largest companies was to ensure that these accounts were not matched and merged based on email address so that in future if a user named Robert.Jones who has a named Active Directory user Work\Bob.Jones and an admin account named Work\admin-db who both had email of Robert.Jones@Work.com were not matched to the same Octopus account. 

If only the administrator account is able to login to Octopus, then you will not be affected. 

If the user accounts you use do not share email addresses, then you will not be affected. 

If you do share email accounts between non-administrative and administrator accounts, then you will be affected and we recommend doing a Proof of Concept upgrade and testing that administrator accounts have the required access after upgrade. If you're reading this and you've accidentally locked yourself out of Octopus, we have a handy way to manage admin accounts in our [docs](https://octopus.com/docs/api-and-integration/octopus.server.exe-command-line/admin). 

## Conclusion

This may not be for everyone but we think this is the right solution for most people but if you have any issues please get in touch with [Support](mailto:Support@Octopus.com). 
