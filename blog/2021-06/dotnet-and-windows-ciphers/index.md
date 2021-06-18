---
title: Unsupported versions of Windows and .NET
description: Understand why old versions of Windows can fail to make network requests.
author: matthew.casperson@octopus.com
visibility: public
published: 2021-06-22-1400
metaImage: blogimage-unsupported-versions-of-windows-and-.net-2021.png
bannerImage: blogimage-unsupported-versions-of-windows-and-.net-2021.png
tags:
 - Engineering
---

![](blogimage-unsupported-versions-of-windows-and-.net-2021.png)

If you're running .NET applications on unsupported versions of Windows, you may be surprised to see errors like `Authentication failed` when nothing appeared to change in the software you were running or the way it was configured. To understand these errors, we need to dig into the cipher suites supported by Windows, and therefore supported by .NET applications.

In this blog post, I diagnose an example of a failed HTTPS connection via a .NET application on an unsupported version of Windows.

## The sample application

Our sample application uses the `HttpClient` class to perform a HTTP GET request against the URL passed in as the first argument:

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

namespace DotNetHttpClientExample
{
    class Program
    {
        static async Task<int> Main(string[] args)
        {
            using var client = new HttpClient();
            var result = await client.GetAsync(args[0]);
            Console.WriteLine("Result was " + result.StatusCode);
            return (int) result.StatusCode >= 200 && (int) result.StatusCode <= 299 ? 0 : 1;
        }
    }
}

```

To demonstrate an example of a website that fails on older versions of Windows (at the time of writing), we'll pass the URL https://getambassador.io to it.

On my Windows 10 machine, the application produces the expected output of `Result was OK`. However, on an older copy of Windows such as Windows 7, the result is the following exception:

```
Unhandled exception. System.Net.Http.HttpRequestException: The SSL connection could not be established, see inner exception.
 ---> System.Security.Authentication.AuthenticationException: Authentication failed because the remote party sent a TLS alert: 'HandshakeFailure'.
 ---> System.ComponentModel.Win32Exception (0x80090326): The message received was unexpected or badly formatted.
   --- End of inner exception stack trace ---
   at System.Net.Security.SslStream.ForceAuthenticationAsync[TIOAdapter](TIOAdapter adapter, Boolean receiveFirst, Byte[]reAuthenticationData, Boolean isApm)
   at System.Net.Http.ConnectHelper.EstablishSslConnectionAsyncCore(Boolean async, Stream stream, SslClientAuthenticationOptions sslOptions, CancellationToken cancellationToken)
   --- End of inner exception stack trace ---
   at System.Net.Http.ConnectHelper.EstablishSslConnectionAsyncCore(Boolean async, Stream stream, SslClientAuthenticationOptions sslOptions, CancellationToken cancellationToken)
   at System.Net.Http.HttpConnectionPool.ConnectAsync(HttpRequestMessage request, Boolean async, CancellationToken cancellationToken)
   at System.Net.Http.HttpConnectionPool.CreateHttp11ConnectionAsync(HttpRequestMessage request, Boolean async, CancellationToken cancellationToken)
   at System.Net.Http.HttpConnectionPool.GetHttpConnectionAsync(HttpRequestMessage request, Boolean async, CancellationToken cancellationToken)
   at System.Net.Http.HttpConnectionPool.SendWithRetryAsync(HttpRequestMessage request, Boolean async, Boolean doRequestAuth, CancellationToken cancellationToken)
   at System.Net.Http.RedirectHandler.SendAsync(HttpRequestMessage request, Boolean async, CancellationToken cancellationToken)
   at System.Net.Http.HttpClient.SendAsyncCore(HttpRequestMessage request, HttpCompletionOption completionOption, Boolean async, Boolean emitTelemetryStartStop, CancellationToken cancellationToken)
   at DotNetHttpClientExample.Program.Main(String[] args)
   at DotNetHttpClientExample.Program.<Main>(String[] args)
```

Messages like `Authentication failed` don't make much sense on the surface, as we're not passing any credentials as part of this network request. So why does this sample application work on one copy of Windows and not another?

## Matching ciphers between the website and the OS

Using a tool like [ScanSSL](https://github.com/rbsec/sslscan) we can interrogate the website to see which ciphers it will accept. The result shows a very targeted list of ciphers:

```
  Supported Server Cipher(s):
Preferred TLSv1.3  256 bits  TLS_AES_256_GCM_SHA384        Curve 25519 DHE 253
Accepted  TLSv1.3  256 bits  TLS_CHACHA20_POLY1305_SHA256  Curve 25519 DHE 253
Accepted  TLSv1.3  128 bits  TLS_AES_128_GCM_SHA256        Curve 25519 DHE 253
Preferred TLSv1.2  256 bits  ECDHE-RSA-AES256-GCM-SHA384   Curve 25519 DHE 253
Accepted  TLSv1.2  256 bits  ECDHE-RSA-CHACHA20-POLY1305   Curve 25519 DHE 253
Accepted  TLSv1.2  128 bits  ECDHE-RSA-AES128-GCM-SHA256   Curve 25519 DHE 253
```

The cipher names reported here are based on OpenSSL. The ciphers referenced by Windows use the IANA naming convention. To convert between the two, use the table at [https://testssl.sh/openssl-iana.mapping.html](https://testssl.sh/openssl-iana.mapping.html). This gives us the following IANA cipher names:

* TLS_AES_256_GCM_SHA384
* TLS_CHACHA20_POLY1305_SHA256
* TLS_AES_128_GCM_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

Microsoft maintains [documentation listing all the supported ciphers across current and previous versions of Windows](https://docs.microsoft.com/en-au/windows/win32/secauthn/cipher-suites-in-schannel). Looking through the lists, unsupported versions of Windows like Server 2012 and 8.1 do not list any of the ciphers accepted by the website. Because .NET applications rely on the ciphers exposed by the underlying OS, our sample application can't establish a secure HTTPS connection.

## Why your browser still works

It's tempting to assume that because a web browser will successfully open the website, all applications should work. This isn't the case though. Browsers like Chrome and Firefox maintain and ship their own ciphers. This means that if your browser is up to date, it will likely include the modern ciphers required to establish most HTTPS connections.

Platforms like Go and Java also maintain their own ciphers, so applications written in those languages may support newer ciphers while running on older versions of Windows.

.NET applications however rely on the ciphers provided by the OS, and the only way to get new ciphers into the OS is through a patch from Microsoft. Unsupported versions of Windows typically do not receive these patches, so over time you can expect an increasing number of websites to stop working with .NET applications.

## Conclusion

It's not recommended to run unsupported versions of Windows, and this is usually explained with vague statements like "it is not secure". While that statement is true, this blog post demonstrates a specific example of how unsupported versions of Windows can no longer interact with external services that implement strict requirements for HTTPS connections.

Happy deployments!
