---
title: Command line life - tab completion for the Octopus CLI
description: Enable tab completion for the Octopus CLI today! We'll also dive into how shell completion works in some popular shell environments.
author: jim.burger@octopus.com
visibility: private
published: 2999-01-01
tags:
 - Octopus
 - DevOps
---

As a developer, I like my IDE and text editors to provide me with useful hints while I'm doing a lot of typing. This not only speeds me up when I know what I'm trying to do, it also speeds me up _when I'm learning_ how to use a tool or framework feature. Thinking back, I probably owe the speed at which I picked up C# programming to features like Intellisense.

I was recently a bit frustrated with our own [Octopus CLI](https://octopus.com/downloads/octopuscli) because I was having to flip between my shell and a browser to check what flags I needed to pass.

Why should my command line experience be any different to visual studio? I can't remember the exact CLI invocations at the best of times, let alone when I'm under pressure to fix something!

## Shell completion to the rescue

The good news is that for a small up front investment, you can tune your command line experience to give these kinds of hints and automatically complete phrases and options for you, and CLI creators can make changes to their products to make things even easier.

The result of my frustration was some additional features in our CLI to support & configure tab completion in popular shells.

If you grab the latest version, you can install the required scripts into popular shells too.

```powershell
# install into your ~/.zshrc
octo install-autocomplete --shell zsh

# install into your ~/.bashrc
octo install-autocomplete --shell bash

# install into your pwsh $PROFILE
octo install-autocomplete --shell pwsh

# using legacy powershell on windows?
octo install-autocomplete --shell powershell

# unsure? do a dry run first and show the result without saving
octo install-autocomplete --shell bash --dryRun
```

Once installed, just dot source or restart your shell and you can complete all the things! Here it is in action...

![animation of octo CLI using zsh tab completion](octo-complete.gif)

## How does tab completion work?
`powershell`, `bash` and `zsh` alike, as well as many other shell environments provide 'built in' commands to accept suggestions from an external source - like a file, or another application. This means that you can [write them for any command line tool](https://www.cyberciti.biz/faq/add-bash-auto-completion-in-ubuntu-linux). These builtins all work in roughly the same way:

1. Register a command to invoke when the tab key is encountered
2. Process the text input prior to the tab key
3. Read in suggestions from a source (list, other command)
4. If there are multiple suggestions, display the choices to the user. Some shells even allow the user to choose one!
5. If there is a single suggestion, use that

Some example of these builtin commands are [compctl](https://linux.die.net/man/1/zshcompctl) (zsh), [complete and compgen](https://www.gnu.org/software/bash/manual/html_node/Programmable-Completion-Builtins.html) (bash), [Register-ArgumentCompleter](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/register-argumentcompleter?view=powershell-7) (powershell & pwsh).

For example, the `dotnet` CLI provides a little known subcommand called `complete` which allows you to type something like `dotnet command lis`. What is returned is a list of possible matches against the different options and subcommands that are available in dotnet: 

```bash
$ dotnet complete lis

--list-runtimes
--list-sdks
list
publish
```

They implemented this so that it can be used to provide the shell completion builtins the data they need to do their work. [As per their documentation](https://github.com/dotnet/cli/blob/master/Documentation/general/tab-completion.md?WT.mc_id=-blog-scottha#how-to-enable-it), in order for this to work you then need to go and edit your profile and sprinkle in some functions to hook it all up.

### Using shell built-in completion commands

Imagine we have a CLI called `acme` which has a subcommand called `suggest` that simply takes a search argument and spits out the suggestions. To hook this into `zsh` we would insert the following script into our profile `~/.zshrc`:

```bash
# ~/.zshrc

_acme_zsh_complete() 
{
  local completions=("$(acme suggest "$words")")

  reply=( "${(ps:\n:)completions}" )
}

compctl -K _acme_zsh_complete acme
```

The last line sets up a registration: when the command `acme` is used prior to a `tab` key hit, then execute the function called `_acme_zsh_complete` which is declared on the lines above.

```bash
compctl -K _acme_zsh_complete acme
```

The `_acme_zsh_complete` function handles passing the words used prior to the `tab` key to the `acme suggest` subcommand and capturing its newline delimited output into a local variable. This is then rendered as a space delimited string to a variable called `reply`. 

```bash
  local completions=("$(acme suggest "$words")")

  reply=( "${(ps:\n:)completions}" )
```

> That last line there is really tricky. In zsh we can modify the formatting of parameters using `Parameter Expansion Flags` (see: http://zsh.sourceforge.net/Doc/Release/Expansion.html#Parameter-Expansion). In short, its using a couple of flags to turn a newline delimited string into an array which is then rendered out as a space delimited list.

`compctl` knows about the variables `$words` and `$reply` and it uses the latter to render the list of suggestions to the user while they are still typing.

The Powershell equivalent would work in a similar way. It might look something like this:

```powershell

# when `acme foo[tab] happens: run this script block
Register-ArgumentCompleter -Native -CommandName acme -ScriptBlock {
    param($wordToComplete, $commandAst, $cursorPosition)
    acme suggest $commandAst.ToString() | % {
        [System.Management.Automation.CompletionResult]::new($_, $_, 'ParameterName', $_)
    }
}
```

Again, the registration knows to look for `acme` invocations and when the `tab` key is hit, the script block is executed. Its job is to take the output of our `acme suggest` command and transform it into something that `pwsh` can understand. In this case, that is a `CompletionResult` object.

## Wrapping up

Shell completion is a great time saver, and with some up front effort you can either use other people's autocompletion scripts via shell plugins, or by writing them yourself. Octopus CLI is now able to help you hook this up really quickly in your favorite shell. Enjoy!


