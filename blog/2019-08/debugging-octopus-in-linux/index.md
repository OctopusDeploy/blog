---
title: Debugging OctopusServer  in Linux
description: Introducing Octopus Server to Linux
author: john.simons@octopus.com
visibility: public
bannerImage: blogimage-octopusid.png
metaImage: metaimage-octopusid.png
published: 2019-08-01
tags:
 - Octopus Server
 - Linux
---

## Code changes for multi targeting

We added compile directives to our code, we standardised on using the non version compile directive, so we use `NETFRAMEWORK` and `NETCOREAPP` instead of the ones that include the version example `NETCOREAPP2.2`, this way allows us to upgrade the target versions without having to modify these directives.

We also added `Conditions` in our `csproj` files to specify different references per target framework, example:

```xml
<ItemGroup Condition="'$(TargetFramework)' == 'net462'">
    <PackageReference Include="Nancy.Hosting.Self" Version="2.0.0" />
    ...
    <Reference Include="System.Runtime.Caching" />
    <Reference Include="WindowsBase" />
</ItemGroup>
<ItemGroup Condition="'$(TargetFramework)' == 'netcoreapp2.2'">
    <PackageReference Include="Microsoft.AspNetCore.Hosting" Version="2.1.1" />
    <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel" Version="2.1.3" />
    <PackageReference Include="Microsoft.AspNetCore.Server.HttpSys" Version="2.1.1" />
    ...
</ItemGroup>
```

Finally, we added the [API Analyzer](https://devblogs.microsoft.com/dotnet/introducing-api-analyzer/) to all our code by adding it into the `Directory.Build.props`:

```xml
<ItemGroup Condition="'$(TargetFramework)' == 'netstandard2.0' Or '$(TargetFramework)' == 'netcoreapp2.2'">
    <PackageReference Include="Microsoft.DotNet.Analyzers.Compatibility" Version="0.2.12-alpha">
        <PrivateAssets>all</PrivateAssets>
        <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
</ItemGroup>
```

This last addition is very useful, because it validates that the code we are using is available in all platforms supported at compile time, which means hopefully we won't ship code that works in Windows but not in Linux.



## How to debug Octopus Server on WSL (and Docker containers)

I found it easier to run all Octopus Server setup as non `sudo`, unfortunately there is an area that Octopus Server writes and reads that you need to explicitly add permissions to. We write to `/etc/octopus` this is where we add the instances config files and masterkey, this is the equivalent of `c:\Octopus` in Windows but in Linux OS `/etc` is a root privileged folder, so we need to manually `sudo chown -R $USER /etc/octopus`. We will make this part of the installation script one day!

Like any good software we have quite a few tests and as part of our integration testing we test the usage of  [Polling Tentacles over WebSockets](https://octopus.com/docs/infrastructure/deployment-targets/windows-targets/polling-tentacles-web-sockets) and that requires us to listen on https. So as part of the setup for this test we have to add some self signed certificates to the trusted store, so we use the following script:

```bash
#!/bin/bash

echo "Setup certificates"
CERTS_PATH_DEST="/usr/local/share/ca-certificates"

if [ ! -d "$CERTS_PATH_DEST" ]
then
    echo "Creating $CERTS_PATH_DEST"
    mkdir ${CERTS_PATH_DEST}
fi

MY_PATH="`dirname \"$0\"`"
CERT_PFX_FILES=(${PWD}/${MY_PATH}/../Octopus.E2ETests/Universe/*.pfx)

for CERT_PFX in "${CERT_PFX_FILES[@]}"
do
    FILE_NAME=$(basename "$CERT_PFX")
    CERT_CRT="${CERTS_PATH_DEST}/${FILE_NAME%.*}.crt"
	if [ ! -e "${CERT_CRT}" ]
    then
        echo "Converting '${CERT_PFX}' to '${CERT_CRT}'"
        openssl pkcs12 -in "${CERT_PFX}" -clcerts -nokeys -out "${CERT_CRT}" -passin pass:password
    fi
done

update-ca-certificates
```



In Linux we also found that we obviously cannot use integrated security when connecting to Sql Server but the one that has given us more grief is connection pooling, for some reason we found our testing a lot more reliable when we explicitly turn **pooling off**.



## So now to debug Octopus on Linux

I found Visual Studio 2019 and Rider quite not there yet, even though they do support attaching to processes via SSH. The tool I settle on (at least for now) is [Visual Studio Code](https://code.visualstudio.com/) with the [Remote Development extension](https://aka.ms/vscode-remote/download/extension). This extension is still in Preview but it works really well.

So with this Remote Development extension I can start debugging, running and code edit in VSCode running in WSL (or a Docker container), all I have to do is point it to my Windows folder that contains Octopus Server code, and pretty much just F5 from there. It is that simple!


## Wrapping up

Some kind of conclusion....



