---
title: Escaping the pipe character in PowerShell commands
description: Using the pipe commands in PowerShell commands is not as simple as it seems.
author: matthew.casperson@octopus.com
visibility: public
published: 2019-12-16
metaImage: powershell-escape-pipe-character.png
bannerImage: powershell-escape-pipe-character.png
bannerImageAlt: Escaping the pipe character in PowerShell commands
tags:
 - Octopus
 - PowerShell
---

![Escaping the pipe character in PowerShell commands](powershell-escape-pipe-character.png)

Recently I was tasked with spinning up some Azure web applications, and to save some time, I use the Azure CLI to run the command `az webapp create -g testgroup -p testplan -n testapp --runtime "node|10.6"`. This resulted in the very obtuse error `'10.6' is not recognized as an internal or external command, operable program or batch file.`, and it took me some Googling to understand the problem. PowerShell escape characters didn't help the way I'd expected them to.

In this blog post, we'll look at some of the ways to resolve this error.

## Wrap the string with quotes

The first way we can solve this error is to wrap up the string containing the pipe character with single quotes like:

```PowerShell
az webapp create -g testgroup -p testplan -n testapp --runtime '"node|10.6"'
```

or:

```PowerShell
az webapp create -g testgroup -p testplan -n testapp --runtime 'node"|"10.6'
```

You can also wrap up the string in escaped double quotes:
```PowerShell
az webapp create -g testgroup -p testplan -n testapp --runtime "`"node|10.6`""
```

This also applies to the `Start-Process` CmdLet:
```PowerShell
start-process az -argumentlist @("webapp", "create", "-g", "testgroup", "-p", "testplan", "-n", "testapp", "--runtime", '"node|10.6"') -nonewwindow -wait
```

And you can define the parameter in an external variable with the pipe character wrapped in quotes:

```PowerShell
$runtime='node"|"10.6'
az webapp create -g testgroup -p testplan -n testapp --runtime $runtime
```

## Use the PowerShell stop-parsing symbol

The stop parse symbol (--%) can be added to the command to instruct PowerShell to stop trying to interpret the string, resulting in a command like:
```PowerShell
az webapp create -g testgroup -p testplan -n testapp --runtime --% "node|10.6"
```

You can find more information on this symbol in the [PowerShell documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_parsing?view=powershell-6).

## What doesn't work

Wrapping the pipe character with single quotes doesn't work:
```PowerShell
PS C:\Users\Matthew> az webapp create -g testgroup -p testplan -n testapp --runtime "node'|'10.6"
''10.6' is not recognized as an internal or external command,
operable program or batch file.
```

Using single quotes doesn't work:

```PowerShell
PS C:\Users\Matthew> az webapp create -g testgroup -p testplan -n testapp --runtime 'node|10.6'
'10.6' is not recognized as an internal or external command,
operable program or batch file.
```

Placing the runtime string into a variable doesn't work:

```PowerShell
PS C:\Users\Matthew> $runtime = "node|10.6"
PS C:\Users\Matthew> az webapp create -g testgroup -p testplan -n testapp --runtime $runtime
'10.6' is not recognized as an internal or external command,
operable program or batch file.
```

Using `Start-Process` also doesn't work:

```PowerShell
PS C:\Users\Matthew> start-process az -argumentlist @("webapp", "create", "-g", "testgroup", "-p", "testplan", "-n", "testapp", "--runtime", "node|10.6") -nonewwindow -wait
'10.6' is not recognized as an internal or external command,
operable program or batch file.
```

Escaping the pipe character doesn't work:

```PowerShell
PS C:\Users\Matthew> az webapp create -g testgroup -p testplan -n testapp --runtime "node`|10.6"
'10.6' is not recognized as an internal or external command,
operable program or batch file.
```

Using the PowerShell escape character `^` doesn't work. This is how you escape the pipe character in the Windows Command Prompt,
and I have seen it incorrectly suggested as a solution for PowerShell as well:

```PowerShell
PS C:\Users\Matthew> az webapp create -g testgroup -p testplan -n testapp --runtime "node^|10.6"
'10.6' is not recognized as an internal or external command,
operable program or batch file.
```

## Conclusion

Passing arguments with the pipe character in PowerShell requires some special processing that is not immediately obvious, but the stop processing symbol or some special escaping with quotes will get you out of trouble.
