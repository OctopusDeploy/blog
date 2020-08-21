---
title: Interacting With the Octopus Deploy API using PowerShell
description: Learn how to leverage your existing skills with PowerShell to make API related calls to Octopus Deploy
author: michael.levan@octopus.com
visibility: private
published: 2030-08-04
metaImage:
bannerImage:
tags:
 - PowerShell
 - Programming
---

Any platform or solution that you want to use without having to click a million buttons through a UI needs an Application Program Interface (API) to interact with. An API is the engine under the hood. It's the way you can interact with an application, platform, and even Internet-of-Things (IoT) devices at the programmatic level. 

In this blog post, you're going to learn how to interact with the Octopus Deploy API using one of the most popular programming languages in the Microsoft and Azure realm, PowerShell.

## Prerequisites

To follow along in this blog post, you will need the following:

- An intermediate level knowledge of PowerShell
- An Octopus Deploy cloud or on-prem server
- A text editor or IDE. For the purposes of this blog post, VS Code will be used. If you don't have VS Code installed, you can find it [here](https://code.visualstudio.com/download).

## The Code

When you're thinking about using something like PowerShell to interact with an API, you're typically going to go the wrapper route. A wrapper is simply, as it sounds, code wrapped around the API call.

In this blog post, you're going to take a look at a wrapper around the Lifecycle API.

Below is the code that you'll use,

```powershell
# The function is created and is called Get-Lifecycle

function Get-Lifecycle {

# The cmdletbinding() attribute to ensure the function is an advanced function, which gives us the ability to use PowerShell features like the $PSCmdlet class, error action preferences, etc.
    [cmdletbinding()]

# The one parameter called is the Octopus Deploy server URL. It's a mandatory parameter.
    param(
        [parameter(Position = 0, Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [alias('URL')]
        [string]$OctopusBaseURL
    )
# You're prompted to securely pass in the API key
$octopusAPIKey = Read-Host 'Input API Key' -AsSecureString

# The API key is passed in with the X-Octopus-ApiKey authentication method
$headers = @{ "X-Octopus-ApiKey" = $octopusAPIKey }

# The header is converted to plain test so Octopus can read the API key
$header = @{ "X-Octopus-ApiKey" = $octopusAPIKey | ConvertFrom-SecureString -AsPlainText }

# Invoke-WebRequest occurs, AKA, makes the API call to Octopus Deploy and returns the Lifecycles
$listLifecycles = $(Invoke-WebRequest -Method GET -Uri $OctopusBaseURL/lifecycles -Headers $header).content
$convert = $listLifecycles | ConvertFrom-Json
$convert

}
```

## The API

The Octopus Deploy API can be looked at using a swagger definition, which allows you to describe your APIs so that machines (or programming languages) can read them. Because the Octopus Deploy API uses Swagger, it's easily readable right from an Octopus Deploy Server.

1. Open up a web browser.
2. Type in your Octopus Deploy server URL
3. At the end of the server URL, add the following:

```bash
/api/swaggerjson
```

  You will see a JSON output similar to the screenshot below.

![](images/1.png)

The great thing about the swagger output is, we don't have to hunt for or guess anything on the API. For example, in the screenshot above we see under `paths` there is an API call to accounts:

```bash
/api/accounts/all
```

Right from the start, you know what the API landscape looks like and you get a good idea of how to interact with the API.

Once you know how to interact with the API, it's time to think about authentication.

## Authentication

Understanding the interactions with an API is of course crucial, but if you can't authenticate to it, you'll be eating a `403 unauthorized` sand-which all day. In this section you'll take a look at the authentication process, which is quite straight-forward in Octopus.

When using PowerShell to interact with APIs, you may have seen the following needed:

- UTF8 encodings
- Base64 converted authentication
- Multiple key/value pairs in the headers hashtable
- Bearer tokens
- Certain .NET namespaces like `System.Net.WebClient` or `System.Net.NetworkCredentials`

With Octopus, it's quite basic to authenticate. In fact, all you need is an API key that you can generate from an Octopus Deploy server. It doesn't need to be converted or manipulated any way.

### Retrieving an API Key

1. Log into Octopus Deploy
2. Under the log in name, go to **Profile**
3. Under **Profile**, click on **My API Keys**
4. Click on the NEW API KEY button. Store the API key in a safe location as you will be using it in the next section.

## Making the API Call

Now that we have all of our ducks in a row, AKA, the API key and the code, it's time to see the API call in action by running the function.

1. Open up VS Code and save the code from the previous section **The Code** in a location of your choosing, like the Desktop for example.
2. Highlight the code, right click, and choose **Run Selection** as shown in the screenshot below. Using the **Run Selection** option will store the function in memory.

![](images/2.png)

  3. Within the terminal, run the following cmdlet:

```powershell
Get-Lifecycle -OctopusBaseURL server_url/api
```

  4. You will be prompted to input the API key, like in the screenshot below.

![](images/3.png)

  5. Once you type in an API key for Octopus Deploy, you will see an output similar to the screenshot below which contains Lifecycle information.

![](images/4.png)

Congrats! You have successfully used PowerShell to interact with the Octopus Deploy API.

## Conclusion

There are several ways to interact with an API. In short, you can interact with pretty much any API with any programming language. Even if there isn't a wrapper or an SDK available, you can make your own by using a couple `curls`. If you don't want to spend your time pointing and clicking in a UI, chances are you want to programmatically interact with a platform, like Octopus Deploy.