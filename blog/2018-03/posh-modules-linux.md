---
title: How to use PowerShell script modules... on Linux
description: Learn how to use your library of PowerShell script modules on Linux
author: shane.gill@octopus.com
visibility: public
published: 2018-03-08
metaImage: metaimage-powershell-linux.png
bannerImage: blogimage-powershell-linux.png
bannerImageAlt: Tux the Penguin reading a Powershell book
tags:
 - Product
---

![Tux the Penguin reading a Powershell book](blogimage-powershell-linux.png)

Octopus Deploy supports re-usable [PowerShell script modules](https://octopus.com/docs/deploying-applications/custom-scripts/script-modules) that can be included in deployments across all of your projects.  When a script module is included in a project, it will automatically become available to PowerShell scripts that run on Windows. You can also run your favorite PowerShell script modules on Linux, but they are not automatically made available to your PowerShell on Linux scripts. While we plan to provide first class PowerShell on Linux support, here is the trick to get them working.

Review the [PowerShell on Linux installation instructions](https://github.com/PowerShell/PowerShell/blob/master/docs/installation/linux.md) for more information.

For this example we will use a script module called `Say Hello` with the following content:

```
function Say-Hello($name)
{
    Write-Output "Hello $name. Welcome to Octopus!"
}
```

For help creating a script module in Octopus follow the [instructions in the documentation](https://octopus.com/docs/deploying-applications/custom-scripts/script-modules).

We will use a project called `PowerShell modules on Linux` that includes the `Say Hello` script module. Our deployment process consists of a `Run a Script` step that will invoke the `Say-Hello` method from PowerShell. Our naive bash script would look like this:

```
pwsh -Command "Say-Hello"
```

If we attempt to run this script we are met with an error message indicating that the script module has not been loaded:

> The term 'Say-Hello' is not recognized as the name of a cmdlet, function, script file, or operable program.

The contents of the script module are available to the step in an Octopus variable called `Octopus.Script.Module[Say Hello]`. In order to use the script module in our bash script we will retrieve the contents, save them to a file and then import them in our PowerShell script. We'll save the module to the `/tmp` folder for simplicity - PowerShell's `Import-Module` expects an absolute path. Here is what the bash script looks like with the module imported:

```
say_hello_module=$(get_octopusvariable "Octopus.Script.Module[Say Hello]")
echo "$say_hello_module" > /tmp/SayHello.psm1
pwsh -noprofile -Command "&Import-Module /tmp/SayHello.psm1; Say-Hello World"
```

When we run the script, our module is imported and we see our expected output:

> Hello World. Welcome to Octopus!

Now you can re-use your library of PowerShell script modules on Windows and Linux.