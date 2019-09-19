---
title: Packaging Node.js applications
description: Just because you are using Node.js, don't violate the principle of build-once, deploy-many
author: robert.erez@octopus.com
visibility: public
metaImage: metaimage-nodejs.png
bannerImage: blogimage-nodejs.png
published: 2018-06-12
tags:
 - Engineering
---

Node.js has become one of the hottest programming languages to work with in recent years. JavaScript has been ranked as the most popular programming language in the [Stack Overflow developer surveys](https://insights.stackoverflow.com/survey/2018) for several years running and it’s not hard to see why.

The ubiquity of JavaScript as the front-end language for the web means that is used and understood by developers whether they come from a background of Java, C#, or Ruby.  The number of developers out there who already know JavaScript means that the community is packed with libraries and frameworks for every possible requirement, so it stands to reason that using it for server-side projects often feels like the right approach.

## Things go wrong

For some reason, when moving from a compiled language to a scripted language, many programmers seem to leave their good habits at the door. It’s too common to see Node.js applications (or pure JavaScript applications that use Node.js as the development environment) where the developers commit their code to GitHub, then pull the source code directly from GitHub onto their production servers to be followed by an `npm install` and `<grunt|gulp|webpack> build`. Often they are diligent enough to write tests but decide it’s easier to rebuild their application each time they deploy it. There are several problems this may cause.

### Dependency versioning

By using a [semver](https://docs.npmjs.com/misc/semver) range in your `packages.json` file, you may end up in a situation where your config says some package dependency needs to match version `~4.17.2`, so your development machine pulls down `4.17.3`, tests are run against `4.17.4`, and by the time production is deployed, the library author has made another update and your production software is running `4.17.9`. I say again, in production! There are plenty of cases of library authors accidentally (or without realizing the effects of their change) pushing breaking changes with a corresponding version bump that result in consuming code crashing due to broad version matching. The problem is exacerbated by the complex dependency chain that can occur in `node_modules`. For instance, when an explicitly dependent module A, depends on some version range of module B, which itself depends on some version range of a bunch of other modules. Remember when you run `npm install` it's not just _your_ modules which are dynamically determined and resolved.

This problem is somewhat mitigated by the addition of the [`package-lock`](https://docs.npmjs.com/files/package-lock.json) feature that was introduced into npm 5. **Make sure you commit this file to source control**.

### Deployment success outside your control

Every time you deploy the application you are downloading new, potentially untested dependencies! And that’s assuming everything goes well. Even though npm no longer supports pushing new files to existing versions, I have heard of more than one occasion where production releases failed because the packages couldn’t be downloaded due to npm being temporarily down or network issues. This failure to install is often accompanied by overwriting the old version, resulting in a server that is effectively worthless (or worse) until access to npm is restored. In one case, a user was performing this *fresh install* process each time their AWS Elastic Beanstalk tried to spin up a new web server when the existing farm was under load. With the npm servers themselves experiencing an outage, the new servers failed to come online, and the inability to handle some of the traffic ended up bringing down the existing, previously fine website. This is _after_ the original deployment had already *succeeded* many days earlier, and the same product version was being deployed to new servers.

## Packaging your Node.js app

Hopefully, we can all agree that JavaScript applications should be treated the same as any other compiled language and packaged into a deployable artifact at build time.

Assuming we are developing the [RandomQuote-JS](https://github.com/OctopusSamples/RandomQuotes-JS) dummy application, a simple build step could be:
```
> npm install
> gulp build    # insert build tool of choice
> zip -r ../RandomQuotes.1.0.zip .
# ../RandomQuotes.1.0.zip created (5.2 MB)
```
The generated zip can then be extracted on my production servers making the application ready to run as-is. In this case, however, we are zipping up the whole application directory, most of which contains the various libraries I used for debugging and compiling. I don't need all the typescript compliers, webpack libraries, and testing frameworks once everything has been built and is ready to be run in production, but how do we separate the dev-time dependencies from the production dependencies?

If you run [`npm prune --production`](https://docs.npmjs.com/cli/prune) npm will remove all the packages from `node_modules` that are specified in the `devDependencies` section of your `package.json`. Even if you don’t use our tooling, I recommend running this command prior to zipping up your entire project folder into an archive for your CD system, whether your deployment tool of choice is Octopus Deploy, VSTS, Chef, or some manual process.

To make this process simpler, Octopus Deploy has published a Node.js cli tool, [octojs](https://github.com/OctopusDeploy/octojs), to create zip (or tar.gz) archives without having to perform the prune step. The package name is generated by default using the name and version in your project’s `package.json`; however, these values can all be overridden. `Octojs` can be used directly through the console or imported and used in code as part of the build process.

### Package via command-line

The easiest way to use the tool is to install it globally:

```
> npm install -g @octopusdeploy/octojs
```

Now, from within any project `octojs` can be invoked to package the application. Assuming our project has just been built and tested, packaging the application is as simple as:

```
> octojs pack -O C:\Bin --gitignore --dependencies prod

Created package C:\Bin\RandomQuotes.1.0.0.zip (1.66 MB)
```

The size of the generated package is now much smaller, and the package ID and version are generated from the project itself. With the package now generated, we can push it directly to the Octopus Deploy built-in feed for deployment:

```
> octojs push --package C:\Bin\ramdomquotes.1.0.0.zip --server http://octopusserver.acme.com --apiKey API-F2K29BA08AA123
Push response 201 - Created
```

### Package via code

Alternatively, you can use this same library to pack and push your project one step through code:

```JavaScript
var octo = require("@octopusdeploy/octojs");
octo.pack({dependencies: 'prod', bypassDisk: true, root: "."},
    (err, data) => {
            console.log("Uploading package: "+ data.name);
            octo.push(data.stream, {
                apiKey: 'API-F2K29BA08AA123 ',
                server: 'http://octopusserver.acme.com',
                name: data.name
            }, () => console.log("Uploaded"));
        }
    })
    .append('hello.txt', new Buffer('This Is An Additional File'))
    .finalize(true);
```

The `octojs` library is open sourced on [GitHub](https://github.com/OctopusDeploy/octojs) along with [gulp](https://github.com/OctopusDeploy/gulp-octo) and [grunt](https://github.com/OctopusDeploy/grunt-octo) versions of these libraries that will help package your project during your build process (with a webpack version on the way).

## Let’s start treating Node.js like a serious language

Node.js is a serious language, so we need to start treating it seriously in our CD pipelines. This means downloading dependencies and building (or transpiling) once on the build server and packaging the result along with the dependencies into a self-contained deployment package. Octopus JS libraries can help with this, but ultimately it doesn’t matter what tool you use to package and deploy your application, what matters is that it’s built once, and deployed across your environments.

*Update:* Check out a [recent post](https://octopus.com/blog/javascript-configuration) where I provide some examples of how to get your environment specific configuration supplied at deploy-time.

## Learn more

* DevOps best practice: [How Octopus handles rollbacks](https://hubs.ly/H0gCHKh0).
* [Octopus vs. Build Servers - Why should I use Octopus when I already have a CI Server?](https://hubs.ly/H0gCFzS0).
* Documentation: [Node on Linux](https://hubs.ly/H0gCHKk0).
