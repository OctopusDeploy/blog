---
title: Getting Started with LDAP Auth Provider
description: Learn how to configure your Self-Hosted Octopus Deploy instance to work with the new LDAP auth provider
author: bob.walker@octopus.com
visibility: private 
published: 2999-01-01
metaImage: blogimage-security.png
bannerImage: blogimage-security.png
tags:
 - Octopus
---

In Octopus Deploy 2021.2, we are adding the [LDAP](https://ldap.com/) authentication provider.  At first look, this might not seem like a big deal, but it is.  For a lot of our customers, they wish to migrate over to the Octopus Linux Container, but they had to authenticate via Active Directory.  Active Directory is also an LDAP server, meaning with the new LDAP provider you can now use the Octopus Linux Container AND authenticate to Active Directory.  In this post, I will walk you through the steps I did to configure the LDAP authentication provider.  By the end of this post, my Octopus Deploy instance will authenticate over LDAP to my local domain, `devopswalker.local`, running on Windows Server 2019.

:::hint
This article assumes you are familiar with directory services core concepts.  If you are unsure of any of these concepts, please talk with your local system administrator.
:::

## LDAP Background

LDAP, or Lightweight Directory Access Protocol, is an open, vendor-neutral, industry-standard protocol for interacting with directory servers.  It is easy to confuse LDAP with a directory server such as Active Directory.  LDAP itself is not a directory server.  It is the protocol used to communicate with a directory server.  Like `http` is the protocol for web servers, or `wss` is the protocol to communicate with web servers via sockets.  The default configuration for Active Directory enables LDAP support.  Chances are if you have Active Directory running on-premise, you already have an LDAP server!

## Why LDAP

There are three primary use cases why we added LDAP support.

1. Not everyone is running Active Directory.  LDAP is vendor-neutral, meaning more non-Microsoft users can take advantage of external authentication.  Most, if not all, directory servers support LDAP.  
2. Active Directory / integrated authentication requires servers to be added to the domain.  That does not work with the Octopus Linux Container.
3. Users with non-Windows clients (specifically MacOS) will have the exact same experience as Windows clients.  With Active Directory, if you clicked the `Sign in with a domain account` button on Chrome running on MacOS you could end up an enless login prompt loop.  At least, I did (and I didn't have the patience to fix it).

There are several other advantages to using LDAP, such as cross-domain queries.  I'd recommend spending a few hours on [ldap.com](https://ldap.com) to learn more about what LDAP can and cannot offer.  It is a very flexible protocol, and each directory server, be it Active Directory or OpenLDAP offer a lot of functionality.

## Secure your LDAP Server First

By default, LDAP traffic is not encrypted.  By monitoring network traffic, an eavesdropper could learn your LDAP password.  Before configuring the LDAP provider in Octopus Deploy, please consult the vendor documentation for your directory server for communicating over SSL or TLS.  Securing an LDAP server is outside the scope of this blog post, and is unique to each vendor.  The rest of this post assumes you have worked with your system administrators on securing your LDAP server.

## Understanding DNs

In LDAP, a DN, or a distinguished name, uniquely identifies an entry and the position in a directory information tree.  Think of it as a path to a file on a file system.

As I stated earlier, my domain is `devopswalker.local`.  Translating that to a DN LDAP can understand is `dc=devopswalker,dc=local`.  All users and groups on my directory server are stored have a common DN of `cn=users,dc=devopswalker,dc=local`.  My user account `Bob Walker` DN is `cn=Bob Walker,cn=users,dc=devopswalker,dc=local`. 

## What you will need

Before configuring LDAP, you will need the following.

- The fully qualified domain name, or FQDN, of the server to query.  In my example, it will be `DC01.devopswalker.local`.
- The port number and security protocol to use.  I'm using the standard secure LDAP port 636 for my domain controller and SSL.
- The username and password of a service account that can perform user and group lookups.  In my example, it will be `cn=Octopus Service,cn=users,dc=devopswalker,dc=local`.
- The root DN you wish to use for user and group look up.  In my example, both will be `cn=users,dc=devopswalker,dc=local`.

:::hint
Leverage a tool such as `ldp.exe` for Windows or [LDAP Administrator](https://www.ldapadministrator.com/download.htm#browser) to find where the highest part on the tree / forest you want to start at.  Starting at the root, for example `dc=devopswalker,dc=local` could lead to performance problems as the LDAP query is forced to to traverse hundreds or thousands of records.  The less data to go through, the faster the query will be.  Because of my active directory configuration, all users and groups is stored in `cn=users,dc=devopswalker,dc=local`.  Your server might be different.

![the results of an ldp browser](ldp-browser.png)
:::

## Configuring LDAP Authentication Provider in Octopus Deploy

Navigate to {{Configuration, Settings, LDAP}}.  Enter values in the following fields:

- Server: Enter the FQDN of your server.
- Port: Change the port (if your secure port is different from the default).
- Security Protocol: Change to SSL or StartTLS.
- Username: Enter the username that will be used to perform the user lookups.  It can either be `[username]@[domain name]` or the user's DN.
- User base DN: enter the base DN for your users, which in my example is `cn=users,dc=devopswalker,dc=local`.
- Group base DN: enter the base DN for your groups, which in my example is `cn=users,dc=devopswalker,dc=local`.
- Is Enabled: Check the check box to enable the feature.

:::hint
As stated earlier, I am using Active Directory running on Windows Server 2019.  Your root DN might be very different than mine.  Please consult a system administrator or use an LDAP explorer to find the right user and group DNs for your directory service.
:::

![basic configuration for LDAP authentication provider](ldap-auth-provider-configuration.png)

## Testing the LDAP Authentication Provider

After I configured the LDAP authentication provider, I made the mistake of logging out and logging back in.  That was rather annoying during the troubleshooting sessions.  I discovered two easy tests I can do without performing the authentication dance.

- External User Lookup
- External Group Lookup

For the external user lookup, go to {{Configuration,Users}} and select a user account.  Once that screen is loaded, expand the LDAP section under logins and click the `ADD LOGIN` button.  If everything is working correctly, then you will see a modal window similar to this.

![successful user lookup](successful-user-lookup.png)

However, I did not see that screen at first. Instead, I saw this:

![failed user lookup](failed-user-lookup.png)

The error `Unable to connect to the LDAP server.  Please see your administrator if this re-occurs.  Error Code 49 Invalid Credentials` is an LDAP lookup error.  In my case, this was caused by me mistyping the password for the lookup user.  

LDAP also returns a data code for each error code.  In the event you need to look up that information, you'd need to open up your Octopus Server logs.   

![data error code](ldap-error-data.png)

:::hint
If you get errors you can't explain, try performing a similar action using an LDAP explorer on the same server hosting Octopus Deploy.  In other words, take Octopus out of the equation.  Get everything working via the explorer tool, then try configuring Octopus Deploy.  If everything works via the explorer and something still isn't working with Octopus Deploy, reach out to [support@octopus.com](mailto:support@octopus.com) for more help.
:::

The external group lookup is the same as the external user lookup.  Except, go to {{Configuration,Teams}} and select a team.  Then click the button `ADD LDAP GROUP` and perform a search.  If everything is configured correctly, then you will see this message:

![external group lookup successful](external-group-success.png)

If the lookup fails, then perform the same troubleshooting you did for the user lookup.

## Signing In

After the above tests are successful, it is time to try the next test, logging into Octopus using the LDAP authentication provider.  I created a test account, `Professor Octopus`, and added it to the `Developers` group.  When I first tried to sign in as `professor.octopus@devopswalker.local`, I got this error:

![UPN Error](failed-sign-in.png)

Changing the username to just `professor.octopus` worked as expected.  This is because the default configuration is to use the `sAMAccountName` to match on.  The new user was created and assigned to the appropriate team.

![Successful sign in](new-user-created.png)

My personal preference is to use `professor.octopus@devopswalker.local` to log in.  If you have a similar preference (or company policy), change the User Filter to be `(&(objectClass=person)(userPrincipalName=*))`.  In our testing we did notice less reliable results using `user@domain`; however your configuration might be different than our testing environment.

![Updated User Filter](updated-ldap-user-filter.png)

I chose `userPrincipalName` because fully qualified name that is where `professor.octopus@devopswalker.local` is stored in my domain controller.  Your directory server might be different.

![user principal vs user id](user-id-vs-principal.png)

## Conclusion

As you can see, unlike the Active Directory authentication provider, the LDAP authentication provider is much more flexible because LDAP itself is much more flexible.  That flexibility does make the LDAP authentication provider more complex.  It might take a some trial and error to dial in your settings.  Because of that, I recommend setting up a test instance to test all your LDAP authentication settings.  

That being said, I'm very excited about adding the LDAP authentication provider to Octopus Deploy.  For quite some time, I've wanted to have active directory authentication but dreaded turning it on.  I have multiple user accounts on my local active directory, each with different permissions in Octopus.  The thought of logging out/logging in on my actual computer each time I wanted to test something was not appealing.  LDAP solves that problem, along with a host of other problems.

Until next time, Happy Deployments!