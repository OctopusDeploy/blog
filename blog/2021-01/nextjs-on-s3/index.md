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

Static content websites deserve proper automation just as much as the rest of your software. Popular frameworks like (Next.js)[https://nextjs.org/] and (Create React App)[https://github.com/facebook/create-react-app] support features to bundle your site's assets into files, but deploying those assets somewhere with a web server is up to you. In this post, we'll use (GitHub Actions)[https://github.com/features/actions] to bundle a Next.js blog and deploy it to (AWS S3)[https://aws.amazon.com/s3/] using Octopus Deploy.

Our project's source code can be found (here)[https://github.com/OctopusSamples/nextjs-blog] on GitHub.

## Build and Package

Before we can deploy our blog using Octopus, we'll need to package the site and push it to a package repository. Packaging our site is useful for many reasons and you can read more about why packaging Node.js apps is important in (this previous blog post)[https://octopus.com/blog/deploying-nodejs] by my colleague Matt Casperson.

For simplicity, we'll use Octopus'es (built in package repository)[https://octopus.com/docs/packaging-applications/package-repositories/built-in-repository] to store our packages. Each package in the built in feed requires an **ID** and a **version**, such that the name of the file is `ID.version.ext` or for example: `nextjs-blog.0.1.0.zip`.

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

`npm run export` runs the following (npm script)[https://docs.npmjs.com/cli/v6/using-npm/scripts] in our package.json file:
```json
"scripts": {
  ...
  "build": "next build",
  "export": "next build && next export"
},
```

See [next.js documentation](https://nextjs.org/docs/advanced-features/static-html-export) for more info on `next export`.

This is a good start, but now we need some way to create new version numbers to give to our package. How do we automate that?

## Tagging with semantic-release

The version numbers assigned to packages in Octopus'es built-in repository must be valid (semantic versions)[https://octopus.com/docs/packaging-applications/create-packages/versioning#semver]. Automatically generating new, valid semantic versions would be difficult to do manually. Luckily, there is an excellent open source project called (semantic-release)[https://semantic-release.gitbook.io/] that can help us do just that.

semantic-release works by evaluating my commit messages based on some pre-defined convention. Depending on the format of my recent commit messages, the library will generate the next appropriate semantic version after finding the most recent version and updating either the major, minor, or patch version accordingly. The details of this library are out of scope for this blog post, but definitely check this project out if you've never used it before.

There is even a community contributed (GitHub Action for Semantic Release)[https://github.com/marketplace/actions/action-for-semantic-release]. Let's use this project to generate our new version automatically and tag our commit:

```yaml
      ...
      - name: Tag and Create Release
        id: semantic
        uses: cycjimmy/semantic-release-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Now that we've built and tagged our new release, it's time to create our package and publish it to Octopus.

### Pack and Publish with octopackjs

If you've ever tried creating packages with npm, it can be quite frustrating.