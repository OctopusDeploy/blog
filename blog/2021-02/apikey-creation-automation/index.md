---
title: "Scripting the creation of Octopus API keys"
description: Using PowerShell to simulate Octopus login and API key creation.
author: phil.stephenson@octopus.com
visibility: public
published: 2021-02-18
tags:
 - DevOps
 - PowerShell
 - API
---

Advanced users of Octopus will already be familiar with the robust, feature-rich [Octopus REST api](https://octopus.com/docs/octopus-rest-api). Maybe you've used it to automate some unique process that isn't available in the Octopus Web UI. Before you can interact with the API though, you first need an Octopus [API key](https://octopus.com/docs/octopus-rest-api/how-to-create-an-api-key). You can create an API key following the instructions in the previous link, but these steps require a human! Alas, if you're in the habit of automating as much as you can this is not ideal. In this blog post, we're going to walk through scripting the creation of an API key for use with the Octopus REST api.

TL;DR - Just want to see the final script? Check out [my GitHub gist here](https://gist.github.com/pstephenson02/3cf2dc3b9d68db28722ad568c9eb49eb).

## Swagger

The first place I go when I want to automate something using the Octopus API is the [Swagger](https://swagger.io/) docs. Each Octopus Server comes with a built-in route where all API documentation is published. Just add `/swaggerui` to the base URL path of your Octopus Server. For example: https://samples.octopus.app/swaggerui

The generated page is organized by API [resources](https://cloud.google.com/apis/design/resources), and there are many of them. Let's do a `CTRL-F` find for `apikey`:

![Swagger ApiKeys resource screenshot](find-apikey.png "width=500")

Sure enough, the first row under ApiKeys: `POST /users/{userId}/apikeys` allows us to create a new API key for the specified user. API keys in Octopus are associated with an Octopus User, and the keys inherit the permissions assigned the user to which they belong.

:::hint
It is best practice when provisioning new API keys to set up [Octopus Service Accounts](https://octopus.com/docs/security/users-and-teams/service-accounts) dedicated for specific functions or integrations.
:::

Let's assume we know our `{userId}` and use a quick one liner PowerShell cmdlet to create an API key:

```pwsh
> Invoke-RestMethod -Method Post -Uri https://samples.octopus.app/api/users/Users-561/apikeys -Headers @{'X-Octopus-ApiKey'='API-XXXX...'} -Body (@{'Purpose'='Just blogging'} | ConvertTo-Json)

Id      : apikeys-I6D74k9rh7eyoqXDlqJCvlsVgU
Purpose : Just blogging
UserId  : Users-561
ApiKey  : API-XXXX...
Created : 2/18/2021 2:25:49 PM
Expires :
Links   : @{Self=/api/users/Users-561/apikeys/apikeys-I6D74k9rh7eyoqXDlqJCvlsVgU}
```

Easy right? Well, attentive readers might now be saying "But hold on Phil! I need an API key to create an API key?". You're absolutely right in this example. Let's see if we can get away without using one.

## Causality dilemma: The chicken or the egg?

Imagine you are writing automation to provision your Octopus Server itself. Perhaps you've written some scripts using the [Octopus Deploy Chocolatey Package](https://chocolatey.org/packages/OctopusDeploy/), the [`Octopus.Server.exe` command line tool](https://octopus.com/docs/octopus-rest-api/octopus.server.exe-command-line), or even the new [Octopus Deploy Terraform Provider](https://octopus.com/blog/octopusdeploy-terraform-provider). You don't want a break in the middle of your provisioning automation when you're forced to login to your Octopus Server, create an API key, then manually plug that into the rest of your automation.

But is it possible to create an API key without having one first? When we create new users in Octopus, those users have no API keys yet but the Octopus Web Portal lets us create one when we're logged in. If the browser can do it, then so can we! Here's what we need to do:

1. Simulate the browser login with a username and password.
2. Retrieve any necessary cookies sent back to us from Octopus Server.
3. Use our cookies to make the same request the browser does when creating a user's first API key.

When we first [install Octopus Server](https://octopus.com/docs/installation#install-octopus), it asks us to create a `Local System Account` or `Custom Domain Account`. For sake of simplicity, lets assume you have a Local System Account. You can also create one using the `Octopus.Server.exe` CLI's [`admin` command](https://octopus.com/docs/octopus-rest-api/octopus.server.exe-command-line/admin).

## Chrome DevTools

We need to inspect what's happening in the browser when we log into our Octopus Web Portal. Let's use [Chrome's excellent built-in DevTools](https://developers.google.com/web/tools/chrome-devtools). Navigate to your Octopus Web Portal login page, and either: Right click on the page and select `Inspect`, or use the hot keys `Ctrl-Shift-i` to open DevTools. Select the `Network` tab shown in the screenshot:

![Chrome DevTools screenshot](chrome-devtools.png "width=500")

After entering your username and password, click `Sign In`. In the DevTools window you should see Chrome recording the many requests occurring during login. These requests include necessary asset files (js, css, etc.), resource requests for initializing the Octopus Dashboard, and more. Scroll back up to the top of the sequence and look for the request labeled `login`:

![Login Recorded screenshot](login-recorded.png "width=500")

Click the `login` request. A window to the right of the requests list will appear showing you additional info. This window has its own set of tabs - select `Headers`. Scrolling to the bottom of the Headers page shows you the exact request body used. This will be important when we build our request in PowerShell. Inspecting the `Response headers` also gives us some important info:

![Response headers screenshot](set-cookie.png "width=500")

The Octopus Server sends back two `Set-Cookie` headers to the browser after logging in which the browser then dutifully stores in its Cookie storage. Upon subsequent requests to the same domain the browser is programmed to send along those cookies in the `Cookie` header. This is how the Octopus Server recognizes my unique session. Let's write some PowerShell to simulate part of that process.

```pwsh
$octopusUrl = 'https://samples.octopus.app'
$username = 'admin'
$password = 'your-password'

$loginPath = "$octopusUrl/api/users/login"

$response = Invoke-WebRequest -Method Post `
    -Uri $loginPath `
    -Body (@{'Username' = $username;'Password' = $password} | ConvertTo-Json) `
    -SessionVariable 'session'
```

A couple things to note here: We use the `Invoke-WebRequest` cmdlet here rather than `Invoke-RestRequest` because we need access to the response headers directly to get the cookies. Notice as well the `-SessionVariable` parameter. This option creates a variable `$session` of type [WebRequestSession](https://docs.microsoft.com/en-us/dotnet/api/microsoft.powershell.commands.webrequestsession?view=powershellsdk-7.0.0) which makes it very easy to get the cookies as we'll see next. The response body will also contain the `User ID`. We'll need for the next request as well. Let's store the variables we need:

```pwsh
$userId = ($response.Content | ConvertFrom-Json).Id
$csrfToken = $session.Cookies.GetCookies($loginPath) | Where-Object { $_.Name.StartsWith('Octopus-Csrf-Token') }
$sessionToken = $session.Cookies.GetCookies($loginPath) | Where-Object { $_.Name.StartsWith('OctopusIdentificationToken') }
```

Now, you could just throw these two tokens into the `Cookie` header and hope it works. But since we're using Chrome DevTools already, why not actually record a request made to generate an API key and make sure we understand how the browser uses these values? We'll navigate to our profile page and then the `My API Keys` page. With our DevTools still open and recording, let's create an API key and look for the `apikeys` POST request shown in the screenshot below. Take a close look at the Request headers:

![API key generation headers screenshot](generate-apikey-recorded.png "width=500")

If we compare the two `Set-Cookie` response headers we received after our login request we can see exactly how the browser uses the values. Now we can complete our request:

```pwsh
$headers = @{
    'Cookie' = $sessionToken.Name + '=' + $sessionToken.Value
    'X-Octopus-Csrf-Token' = $csrfToken.Value
}

Invoke-RestMethod -Method Post `
    -Uri "$octopusUrl/api/users/$userId/apikeys" ` # $octopusUrl is defined at the top of the script
    -Headers $headers `
    -Body (@{'Purpose'='Just Blogging'} | ConvertTo-Json)
```

### Finished product

```pwsh
$octopusUrl = 'https://samples.octopus.app'
$username = 'admin'
$password = 'your-password'

$loginPath = "$octopusUrl/api/users/login"

$response = Invoke-WebRequest -Method Post `
    -Uri $loginPath `
    -Body (@{'Username' = $username;'Password' = $password} | ConvertTo-Json) `
    -SessionVariable 'session'

$userId = ($response.Content | ConvertFrom-Json).Id
$csrfToken = $session.Cookies.GetCookies($loginPath) `
    | Where-Object { $_.Name.StartsWith('Octopus-Csrf-Token') }
$sessionToken = $session.Cookies.GetCookies($loginPath) `
    | Where-Object { $_.Name.StartsWith('OctopusIdentificationToken') }

$headers = @{
    'Cookie' = $sessionToken.Name + '=' + $sessionToken.Value
    'X-Octopus-Csrf-Token' = $csrfToken.Value
}

Invoke-RestMethod -Method Post `
    -Uri "$octopusUrl/api/users/$userId/apikeys" `
    -Headers $headers `
    -Body (@{'Purpose'='Automation'} | ConvertTo-Json)
```

## Conclusion

Scripting the creation of API keys can also be useful for new service users, provisioning your Octopus instance, or use with the Octopus Terraform Provider.

Using Chrome DevTools (or Firefox, Edge, Safari equivalents) is a powerful way to see what's happening in the interactions between the Octopus front end Web Portal (a single page React app) and the Octopus REST api. Occasionally you may find that the Swagger documentation is just slightly off or doesn't have all the info you need and using the browser's tools can give you the definitive answer.

Interested in learning more about the Octopus REST api? Check out our recent webinar:

[Using the Octopus API to save time by automating repetitive tasks](https://octopus.zoom.us/webinar/register/6016118341944/WN_ykrzzdSvRZOMWFojvxNguw)

Thanks for reading, and happy deployments!