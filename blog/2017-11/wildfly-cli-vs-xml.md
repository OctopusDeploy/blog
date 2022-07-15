---
title: Configuring WildFly via XML Templates or CLI Scripts
description: Configuring a WildFly server can be done either by editing the XML files directly, or by running CLI script. But which is the best choice?
author: matthew.casperson@octopus.com
visibility: public
published: 2017-11-07
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - DevOps
---

When building a WildFly server with automation tools like Puppet, Chef, Ansible etc you will be faced with the question of how to go from stock download to customized server. A big part of this customization is how you will edit the configuration files (`standalone.xml`, `domain.xml`, `host.xml` and `host-slave.xml`) during the build process.

You have two main choices when editing these XML files: edit the text directly, or make changes using the WildFly CLI. Both have pros and cons, which we will look at in this blog post.

## Editing XML Directly

Editing the XML configuration files directly is perhaps the most natural choice. Since XML files are just plain text, and every deployment tool has some form of templating support, it is quite easy to make the required changes in a base template and copy the customized version during deployment.

This approach has a number of benefits.

First, you don’t need a running instance of WildFly to make the configuration changes. This removes some of the headaches you’ll encounter with the CLI tool, which by default requires an instance to be running, and will force you to restart the server for some changes to take effect.

Second, all changes are essentially done in "batch" mode, which is to say that all changes are done at once. This might sound obvious when you are copying a template XML file, but it does remove something that has to be considered when making changes via the CLI.

There are some significant downsides to editing the XML configuration files directly though.

Some of the XML files used by WildFly are not static. For example, WildFly will update the `domain.xml` and `standalone.xml` files with details of the currently deployed applications. A goal of any deployment script is to be idempotent, but if you blindly copy a fresh XML configuration file onto the server with each deployment, you can find yourself losing the runtime changes WildFly made. Among other things, you may find that a server with a fresh XML configuration file has no deployments, which might come as a rude shock in production.

Without some forethought you may also find yourself in a situation where is it quite difficult to see what changes have been made to the XML files. These configuration files are quite long, and without a diff tool it would be nearly impossible to spot the changes made in a customized template.

For the same reason, applying your customizations to the configuration files in a new version of WildFly can be challenging. Given that WildFly does a major release every year or so (and this release schedule is set to be accelerated starting with WildFly 12), you really want to be able to easily apply your customizations to the next version, if only to take advantage of security patches in later WildFly releases.

My recommendation for those applying changes to the XML configuration files directly is to add clear comments for each change. For example, you may change the configuration to bind to any network adaptor with code like this:

```xml
<interfaces>
        <interface name="management">
            <!-- CHANGE: Bind to any IP address -->
            <any-address/>
        </interface>
        <interface name="public">
            <!-- CHANGE: Bind to any IP address -->
            <any-address/>
        </interface>
</interfaces>
```

The comments will be discarded by WildFly when it boots up (WildFly overwriting XML files at runtime is the reason why you can’t edit these files while WildFly is running), but with a simple search string you can find all the changes you have made in your template. This makes it easy to port the changes to a new version, or to simply understand how your customized version differs from the stock download.

## Updating with the CLI

Using the WildFly CLI tool to apply customizations is more advanced than editing an XML file, but it does have a number of advantages.

You may find that your CLI scripts will apply over a new version of WildFly without any changes. For example, the following CLI commands bind the public interface to any address:

```
/interface=public/:undefine-attribute(name=inet-address)
/interface=public/:write-attribute(name=any-address,value=true)
```

I would expect these commands to work in all recent versions of WildFly, and I would expect them to work in upcoming releases too. This makes upgrading your WildFly version much easier.

These CLI commands also offer a concise way to understand what changes are being made to the stock download. Unlike a template XML file where the changes are scattered throughout a large file, the CLI commands are easy to see and understand.

And because you are applying targeted changes to the current configuration (as opposed to overwriting the entire XML file), CLI commands have the potential to be idempotent. With some use of the [CLI flow control statements](https://developer.jboss.org/wiki/If-elseControlFlow), it is possible to only apply a change if the current setting is not the desired state.

CLI commands provide a level of error checking. To be fair the XML validation is quite strict in WildFly, so it unlikely that you could have invalid XML and a bootable server, but the CLI will offer more immediate feedback when you try to do something wrong.

For complex environments, you can take advantage of the CLI Java libraries to script up your changes in any JVM language. Groovy is a nice choice for this. Using [Grape](http://docs.groovy-lang.org/latest/html/documentation/grape.html) you can pull down any required dependencies, and your script can be run like any other executable in a Linux or MacOS environment.

This example groovy script uses the WildFly CLI libraries to execute the required CLI commands. Although this is quite a simple example, it can be modified to accommodate far more complex requirements.

```java
#!/usr/bin/env groovy

@Grab(group='org.wildfly.core', module='wildfly-embedded', version='2.2.1.Final')
@Grab(group='org.wildfly.security', module='wildfly-security-manager', version='1.1.2.Final')
@Grab(group='org.wildfly.core', module='wildfly-cli', version='3.0.0.Beta23')
import org.jboss.as.cli.scriptsupport.*

final DEFAULT_HOST = "localhost"
final DEFAULT_PORT = "9990"
final DEFAULT_PROTOCOL = "remote+http"

def cli = new CliBuilder()

cli.with {
    h longOpt: 'help', 'Show usage information'
    c longOpt: 'controller', args: 1, argName: 'controller', 'WildFly controller'
    d longOpt: 'port', args: 1, argName: 'port', type: Number.class, 'Wildfly management port'
    e longOpt: 'protocol', args: 1, argName: 'protocol', 'Wildfly management protocol i.e. remote+https'
    u longOpt: 'user', args: 1, argName: 'username', required: true, 'WildFly management username'
    p longOpt: 'password', args: 1, argName: 'password', required: true, 'WildFly management password'
}

def options = cli.parse(args)

if (!options) {
    return
}

if (options.h) {
    cli.usage()
    return
}

def jbossCli = CLI.newInstance()

jbossCli.connect(
                options.protocol ?: DEFAULT_PROTOCOL,
                options.controller ?: DEFAULT_HOST,
                Integer.parseInt(options.port ?: DEFAULT_PORT),
                options.user,
                options.password.toCharArray())

jbossCli.cmd("/:take-snapshot")
jbossCli.cmd("/interface=public/:undefine-attribute(name=inet-address)")
jbossCli.cmd("/interface=public/:write-attribute(name=any-address,value=true)")

System.exit(0)
```

The script can be run with:

```
./script.groovy -u admin -p password01
```

Running CLI scripts does have drawbacks though.

Typically you will need to have a running instance of WildFly in order to run your CLI scripts. This can be tricky when the server is not configured correctly in order to boot, and in turn be available to run CLI scripts against. For example, the interface bindings might not be correct, ports may not be correct, or a slave instance may not be able to connect to the domain master.

This situation can be mitigated by running the [embed-server](http://www.mastertheboss.com/jbossas/wildfly9/configuring-wildfly-9-from-the-cli-in-offline-mode), which is a kind of "offline" mode that gives you access to the CLI without necessarily having a running server.

You also need to be aware of what changes require a server restart and what changes need to be made as part of a batch. These are two things you don’t need to worry about when copying an XML file and starting the server.

## Final Thoughts

Where idempotency is required, CLI scripts are the best choice. They allow you to only modify the settings that are not in the desired state, and are non-destructive. CLI scripts tend to be easier to port to new WildFly versions, and provide a concise definition of how your instance differs from a stock download.

For immutable environments, you can’t go past the simplicity of editing XML files. Every sysadmin will know how to edit an XML file (whereas the CLI can be tricky for newcomers) and every deployment tool supports some kind of templating that will work with XML files. And as long as you are careful to clearly identify the changes that have been made to a stock config file, it is easy enough to port the changes to new versions of WildFly.

If you are interested in automating the deployment of your Java applications, [download a trial copy of Octopus Deploy](https://octopus.com/downloads), and take a look at [our documentation](https://octopus.com/docs/deployments/java/deploying-java-applications).
