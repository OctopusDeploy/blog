---
title: Bootstrap a .NET Core development environment for Linux
description: This post covers how to bootstrap a Linux-based development environment for day to day .NET Core development.
author: jim.burger@octopus.com
visibility: public
published: 2019-11-25
metaImage: bootstrapping_netcore_dev_on_linux.png
bannerImage: bootstrapping_netcore_dev_on_linux.png
bannerImageAlt: Bootstrap a .NET Core development environment for Linux
tags:
 - Engineering
 - Linux
---

![Bootstrap a .NET Core development environment for Linux](bootstrapping_netcore_dev_on_linux.png)

Microsoft .NET Core has great cross-platform support, giving .NET developers the ability to do much of their development on Linux distributions or macOS. In this post, I’ll go through how you can get started and what tools are available to support developers.

<h2>In this post</h2>

!toc

## Use the tools that make you happy

One of the things I truly appreciate about working at [Octopus Deploy](https://octopus.com) is that I’m encouraged to work in the way that I choose, that maximizes my happiness and my productivity.

Earlier this year, I decided to make the switch to Linux for my day to day working environment, and I haven’t regretted it. Unix-like operating systems have always been something I prefer, but I have never been able to settle on it at work for development as a .NET developer.

I’ll discuss some of the options for .NET developers on Linux and show you how I got my stack together with some scripts I use regularly.

## Using containers to manage development time database & logging servers

Much like my colleague [Bob Walker](https://octopus.com/blog/automate-sql-server-install-using-linux-docker#docker-compose), I personally like to use `docker-compose` to speed up the process of dealing with dependencies like my SQL & Logging servers. What do I like the most about it? The setup for a database and log server is less than 20 lines of YAML!

For example, this lets me quickly spin up an [SQL server](https://www.microsoft.com/en-us/sql-server/) and [Seq](https://datalust.co/seq), our preferred logging tool that we use in dev, test, and prod:

```yaml
---
version: "3"
services:
  # this is my database, there are many others like it, but this one is mine
  db:
    image: "mcr.microsoft.com/mssql/server:2017-latest-ubuntu"
    volumes: # Mount volumes like this: host/dir:container/dir
      - /home/user/.devenv/sql:/var/opt/mssql
    ports:   # Expose ports like this: host_port:container_port
      - "1433:1433"
    env_file: .env  # more on this later ;)

  # seq is an easy to use log server - how simple is this?
  seq:
    image: datalust/seq:latest
    environment: # declare environment variabels inline
      - "ACCEPT_EULA=Y"
    volumes:
      - /home/user/.devenv/seq:/data
    ports:
      - "5341:80"
```

You might also notice that there are no database passwords or API keys in this file. That’s because I can keep those sensitive things out of sight using an `.env` file. Environment files are something `docker-compose` supports so you can declare your environment variables all in the one place.

`.env` files are key value pairs delimited by newlines:

```
SA_PASSWORD=ForYourEyesOnly007#
ACCEPT_EULA=true
MSSQL_PID=Developer
```

To bring up the environment, use `docker-compose up`. You can keep your shell available afterward by supplying `-d`:

```bash
docker-compose up -d
docker stats        # show how the containers are operating (CTRL+C to exit)
docker-compose down # stop the stack you've created
```
If you prefer, Microsoft has a rather nice [VS Code extension](https://github.com/microsoft/vscode-docker) to give you that right-click menu feeling!

![docker-compose in vs code terminal](docker-compose.png)

Based on this, I created a collection of [convenience scripts](https://github.com/jburger/devenv) to demonstrate some possibilities, and I recorded a [screencast](https://asciinema.org/a/AUXSaRj6hfqQS1QQqflrITrX0) to show you how to use them.

### Database management

Of course, Microsoft SQL Server is not the only game in town on Linux, far from it. However, if you’re like me and your *hometown* is MS SQL, then you’ll be happy to know there are some industrial-strength options for working with it.

I use [Datagrip](https://www.jetbrains.com/datagrip/), which is an awesome tool for working with a wide variety of databases, and has great intellisense and refactoring features. My favorite feature is being able to assign a color to each database connection (e.g., green in test, and red in production).

A free alternative from Microsoft for working with SQL Server is the [`Azure Data Studio`](https://docs.microsoft.com/en-us/sql/azure-data-studio/download?view=sql-server-ver15#get-azure-data-studio-for-linux). It provides a nice and simple experience for those not wanting to outlay money on heavier tools.

![Azure data studio](azure-data-std.png)

## Linux friendly development tools

There are a tonne of options for development tooling, but here are the ones I like to use for .NET engineering at Octopus Deploy.

### Integrated Development Environment (IDE)

There are a few great IDE options out there for .NET developers on Linux now!

I personally love the `Jetbrains` [toolbox](https://www.jetbrains.com/toolbox-app/) for IDE and database tools. `Rider`, `Datagrip`, and `Webstorm` are *go to* tools for me on a daily basis. I also use `Clion` for learning about rust development in my spare time. One of the benefits I find is that each is tailored to the style of development they represent while maintaining a consistent keyboard shortcut scheme.

If I do need to use Windows, I can install them there too without having to re-learn any keystrokes. Rider has a great visual debugger and many comparable features to Visual Studio.

![rider debugging some tests on Linux](rider-running-on-linux.png)

I also love `VS Code` for tasks that don’t need a debugger, like parsing logs and writing blogs or documentation.

#### No way! Vim or bust.

Ok, so, if you’re in this camp, you probably already know what you’re doing! That said, I can confirm .NET productivity is possible with a lofi toolbox. If you’re interested in exploring this as an option, here are some tools to get you started.

- [ranger: a neat little file browser that works like vim](https://wiki.archlinux.org/index.php/Ranger).
- [vim and whatever plugins make you a happy person](https://wiki.archlinux.org/index.php/vim).
- [omnisharp for vim](https://github.com/OmniSharp/omnisharp-vim).
- [tmux for managing multiple consoles](https://wiki.archlinux.org/index.php/tmux).
- [dotnet watch](https://docs.microsoft.com/en-us/aspnet/core/tutorials/dotnet-watch?view=aspnetcore-3.0).

![ranger, vim & omnisharp](tmux-vim-dotnet.gif)

### Git source control
For simple things, I still stick to Git at the command line, `zsh` has a sweet plugin for [just about everything](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins), including Git.

For dealing with complex trees, I think [GitKraken](https://www.gitkraken.com/) is a pretty slick option, it looks great, performs well, and integrates with GitHub.

### Installing apps from the command line
It's common for Windows developers to swear by [boxstarter](https://boxstarter.org/) and [chocolatey](https://chocolatey.org/) to install their favorite tools. This allows them to keep a script of favorite tools and run them on new machines. You can do the same on Linux.

Most distributions come with their own [package manager](https://www.linode.com/docs/tools-reference/linux-package-management/) for installing tools, each with their own command interface. There is also [snapd](https://snapcraft.io) which is a cross-distribution option.

On Ubuntu and other distributions that support it, you can use `snapd` to install some of the popular tools. If you don’t like them, they’ll uninstall cleanly:

```bash
sudo snap install --classic code
sudo snap install gitkraken

# optional non-free IDE options
sudo snap install --classic rider
sudo snap install --classic datagrip
sudo snap install --classic webstorm
```

To remove things:

```bash
sudo snap remove dont_want_this
```

Getting started with [snapd is easy](https://snapcraft.io/docs/getting-started), it comes with some [interesting security features](https://snapcraft.io/docs/snap-confinement) and is unique to the Linux ecosystem!

## Vagrant boxes for isolated environments

Containers are great, but they [aren’t quite as isolated](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/containers-vs-vm) as a virtual machine (VM). For example, when I’m dealing with some un-trusted binaries during triage, I’ll use a machine separate from my main development environment. I like to use [Vagrant](https://www.vagrantup.com/) by HashiCorp to manage temporary VM lifetimes; it’s almost as easy as using docker!

Here is an example of a lightweight Arch Linux environment. Its got the basics, ready for tweaking, using, and then destroying later:

* i3: a lightweight tiling window manager
* sakura: a lightweight terminal emulator
* firefox
* git
* dotnet sdk
* a text editor (if you prefer, swap `vim` out for your favorite editor)

Just adjust the script in `config.vm.provision` to your liking and run `vagrant up` to build it:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "archlinux/archlinux" # from their official repository

  config.vm.provider "virtualbox" do |vb|
    # show console
    vb.gui = true
    # RAM
    vb.memory = 4096
    # CPU
    vb.cpus = 2
  end

  # Hook the provision event and run an inline shell script to install your favorite tools here
  config.vm.provision "shell", inline: <<-SHELL
    echo "installing tools"
      pacman -Sy \
      xorg-server \
      xorg-xinit \
      xorg-apps \
      lxdm \
      i3 \
      dmenu \
      firefox \
      sakura \
      git \
      vim \
      dotnet-sdk \
      --noconfirm
      sed -i 's|# session=/usr/bin/startlxde|session=/usr/bin/i3|g' /etc/lxdm/lxdm.conf

    systemctl enable lxdm
    systemctl start lxdm
  SHELL
end
```

The Vagrant CLI has all the tools you need to quickly automate various operations for your VM from scripts or the command line:

```bash
# change into a directory with a Vagrant file
cd /my/box/
# bring it up
vagrant up
# push a new snapshot onto the stack
vagrant snapshot push
# Tweak a Vagrant file and re-provision while a VM is running
vagrant provision
# easy login via ssh
vagrant ssh
# rollback to the previous snapshot
vagrant snapshot pop
# bring it down
vagrant halt
# save a named snapshot
vagrant snapshot save [vm_name] [snapshot-name]
# rollback to a named snapshot
vagrant snapshot restore [vm_name] [snapshot-name]
```

## A word on using non-official Vagrant boxes

It is preferable to use official boxes if you can. While it is super convenient to use somebody else’s box from the library, it also comes with an element of risk. What if they installed a crypto miner on it?

If you’re stuck because of this risk, with a bit of extra work, you can author your own images. I highly recommend taking a look at [this great post by Matt Hodgkins](https://hodgkins.io/best-practices-with-packer-and-windows#best-practices) on using Hashicorp `packer` best practices to build your own images that you can then use in Vagrant.

## Wrapping up

Thanks for reading. If you’re interested in making Linux your home OS, or interested in ways to leverage `docker` and `vagrant` in your favored operating system, I hope this post helped in some way to get you started. Let us know in the comments about your favorite tools!
