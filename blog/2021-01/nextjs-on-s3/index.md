---
title: "Automating Next.js static content site releases with GitHub Actions and Octopus Deploy"
description: In this post we'll use GitHub Actions to bundle our Next.js assets and deploy them to AWS S3 using Octopus Deploy.
author: phil.stephenson@octopus.com
visibility: public
published: 2021-01-25
tags:
 - DevOps
 - Next.js
 - GitHub Actions
 - AWS S3
---

Static content websites deserve proper automation just as much as the rest of your software. Popular frameworks like [Next.js](https://nextjs.org/) and [Create React App](https://github.com/facebook/create-react-app) support features to bundle your site's assets into files, but deploying those assets somewhere with a web server is up to you. In this post, we'll use [GitHub Actions](https://github.com/features/actions) to bundle a Next.js blog and deploy it to [AWS S3](https://aws.amazon.com/s3/) using Octopus Deploy.

Our project's source code can be found [here](https://github.com/OctopusSamples/nextjs-blog) on GitHub.

### Build and Package

Before we can deploy our blog using Octopus, we'll need to package the site and push it to a package repository. Packaging our site is useful for many reasons and you can read more about why packaging Node.js apps is important in [this previous blog post](https://octopus.com/blog/deploying-nodejs) by my colleague Matt Casperson.

For simplicity, we'll use Octopus'es [built in package repository](https://octopus.com/docs/packaging-applications/package-repositories/built-in-repository) to store our packages. Each package in the built in feed requires an **ID** and a **version**, such that the name of the file is `ID.version.ext` or for example: `nextjs-blog.0.1.0.zip`.

Since our project is already hosted on GitHub, let's setup a GitHub Action that helps us create our package. Our workflow will something like this:
For each push to our `main` branch, we'll
- Checkout the source code
- Run `npm ci` to get our `node_modules` dependencies (implicit in this step is having node.js setup in our Actions environment)
- Tag our commit with a new version number
- Use `next export` to generate our static asset files
- Bundle those assets into our package
- Finally, push our package to the Octopus built-in repository

Let's start with a GitHub Action template that at least checks out our source code, runs `npm ci`, and generates our assets:

```yaml
// main.yml - https://github.com/OctopusSamples/nextjs-blog/blob/main/.github/workflows/main.yml
on:
  push:
    branches:
      - main
jobs:
  release:
    name: Create Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v2
        run: |
          npm ci
          npm run export
```

`npm run export` runs the following [npm script](https://docs.npmjs.com/cli/v6/using-npm/scripts) in our package.json file:
```json
"scripts": {
  ...
  "build": "next build",
  "export": "next build && next export"
},
```

See [next.js documentation](https://nextjs.org/docs/advanced-features/static-html-export) for more info on `next export`.

This is a good start, but now we need some way to create new version numbers to give to our package. How do we automate that?

#### Tagging with semantic-release

The version numbers assigned to packages in Octopus'es built-in repository must be valid [semantic versions](https://octopus.com/docs/packaging-applications/create-packages/versioning#semver). Automatically generating new, valid semantic versions would be difficult to do manually. Luckily, there is an excellent open source project called [semantic-release](https://semantic-release.gitbook.io/) that can help us do just that.

semantic-release works by evaluating my commit messages based on some pre-defined convention. Depending on the format of my recent commit messages, the library will generate the next appropriate semantic version after finding the most recent version and updating either the major, minor, or patch version accordingly. The details of this library are out of scope for this blog post, but definitely check this project out if you've never used it before.

There is even a community contributed [GitHub Action for Semantic Release](https://github.com/marketplace/actions/action-for-semantic-release). Let's use this project to generate our new version automatically and tag our commit:

```yaml
      ...
      - name: Tag and Create Release
        id: semantic
        uses: cycjimmy/semantic-release-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Note that since our primary git branch is named `main`, we needed a small piece of configuration in our `package.json` to tell semantic-release to evaluate commits only on pushes to the `main` branch:
```json
  "release": {
    "branches": [
      "main"
    ]
  }
```

Now that we've built and tagged our new release, it's time to create our package and publish it to Octopus.

#### Pack and Publish with octopackjs

If you've ever tried creating packages with plain ol' npm, it can be quite frustrating.
- The `npm pack` [command](https://docs.npmjs.com/cli/v6/commands/npm-pack) exposes very few cli parameters.
- You cannot explicitly define the name of the generated package.
- The version of the package must come from the `version` key in `package.json` and cannot be overridden with a cli parameter.
- Your package root *must* contain the `package.json` file
- You cannot easily define the directory structure that ultimately ends up in your package.

It just seems as if npm was designed exclusively for the bundling of your packages for consumption by... other npm projects - not for deployment.

[octopackjs](https://github.com/OctopusDeploy/octopackjs) is an open source project maintained by Octopus that is uniquely designed for bundling and pushing your packages to an Octopus server. After running `npm run export`, Next.js places our static asset files in a directory `out`. Let's write a small node script using octopackjs to package that directory and push it to our Octopus server:

```js
// publish.js - https://github.com/OctopusSamples/nextjs-blog/blob/main/publish.js
var octo = require('@octopusdeploy/octopackjs');
octo.pack()
    .appendSubDir('out', true)
    .toFile('.', function (err, data) {
        console.log('Package Saved: ' + data.name);
        octo.push(data.name, {
            host: 'https://samples.octopus.app',
            apikey: 'MY-API-KEY',
            replace: true
        }, function(err, result) {
            if (!err) {
                console.log(result);
            }
        });
    });
```

See [our documentation](https://octopus.com/docs/octopus-rest-api/how-to-create-an-api-key) for creating API keys for use with Octopus Deploy. Security conscious readers might notice right away that it appears my API key is just hard coded directly into our script - a big no no! Let's use an environment variable here instead:

```js
octo.push(data.name, {
    ...
    apikey: process.env.OCTOPUS_APIKEY,
    ...
```

We can inject this environment variable using an [encrypted secret in GitHub Actions](https://docs.github.com/en/actions/reference/encrypted-secrets). First we'll add our `OCTOPUS_APIKEY` secret to our repository (follow the instructions in the Actions docs):

![Encrypted Secret in GitHub repository screenshot](actions-secret.png "width=500")

Now, we'll reference our secret in our `main.yml` GitHub Action template:
```yaml
...
- if: steps.semantic.outputs.new_release_published == 'true'
name: Publish Package
env:
    OCTOPUS_APIKEY: ${{ secrets.OCTOPUS_APIKEY }}
run: |
    npm ci
    npm run export
    OCTOPUS_APIKEY="$OCTOPUS_APIKEY" node publish.js
```

:::hint
In this example, we're *pushing* our package from GitHub Actions to our Octopus Cloud instances at https://samples.octopus.app. If you're running an Octopus Server that is not publicly accessible from github.com, you might instead consider pushing to a third-party package repository and have your Octopus Server pull your packages from an external feed.
:::

### Deploy

To do...