---
title: Deploying a JavaScript bundle project with octopus
description: Learn how to handle cache busting and config of a shared JavaScript bundle and make it easy to reference in other Octopus projects.
author: lee.meyer@octopus.com
visibility: private
published: 2025-08-23-1400
isFeatured: false
tags:
 - DevOps
 - Frontend
 - Variable sets
 - Amazon S3
---

A frontend dev pattern I've seen at many companies starts with the best of intentions, but it can lead to pain if not handled with care. You see a need for reuse of frontend code across multiple projects, possibly maintained by different teams using different tech on the server. You create a shared JavaScript bundle project with its own repo and release process. It's a sensible idea, but it opens questions we need good answers for, to stop our little bundle of joy growing into a monster. In this post, I'll explain a simple example of how to manage the deployment process for a shared JavaScript project that is simple to reference from other Octopus projects. My example is using a Vue JS bundle deployed to an Amazon S3 bucket, but I hope you'll see how the same principles could be applied to any combination of frontend framework and hosting provider. 

## The process

Our finished deployment process will look like this in Octopus.

![deployment process](aws%20bundle%20process.png)

I'll explain the reason for each step and how they work.

## Create S3 bucket if it does not exist

In the spirit of treating servers as cattle and not pets, I don't want to assume much about our deployment target, beyond having an AWS account with appropriate permissions. In a specific case I had dedicated buckets for combinations of different regions and the environments of test, staging, and production, so I appreciated a build process that only needs me to name the bucket in a scoped variable and will set it up correctly if required. This is achieved with an [AWS CLI Step](https://octopus.com/docs/deployments/custom-scripts/aws-cli-scripts) that runs the following PowerShell script, which uses the AWS CLI to see if we get a non-error result trying to list the contents of the bucket. Otherwise it creates the bucket, then polls to confirm the buckets exist before the step finishes.

​```ps
$bucket = $OctopusParameters["s3-bucket-name"] 
$region = $OctopusParameters["s3-region"]
$found = aws s3 ls s3://$bucket/ --recursive --summarize | Select-String -Pattern 'Total Objects:'
if ([string]::IsNullOrWhiteSpace($found)) {
    aws s3api create-bucket --bucket $bucket --region $region
    aws s3api wait bucket-exists --bucket $bucket
}
​```

## Set S3 CORS policy

This is another AWS CLI script with the following PowerShell inline.

​```ps
echo '{"CORSRules": [ { "AllowedOrigins": ["*"], "AllowedHeaders": [],"AllowedMethods": ["GET"] } ] }' | out-file -encoding ASCII cors.json
aws s3api put-bucket-cors --bucket bundle-s3 --cors-configuration file://cors.json
​```

You could get more sophisticated with CORS as needed, but since in my example I'm assuming our bundles live in their own dedicated bucket, it makes sense to have a simplistic "allow all GET requests." The encoding step was important rather than just echoing straight to a file. I don't really know why the [CLI command](https://docs.aws.amazon.com/cli/latest/reference/s3api/put-bucket-cors.html) for setting CORS insists on reading from a file and won't just let me pass JSON through the command line, but if you desire a more complicated CORS policy, it might be cleaner to choose the "Script file inside a package option" and have the .ps1 and cors.json files source controlled in your bundle repo, rather than the inline option I've used here. 

![script options](cors%20script.png)

## Upload bundle to S3

There a few prequisites for this next AWS CLI step to work as desired, that I'll explain before showing how it's setup in Octopus. I'll be showing how it was achieved in Vue, and the steps will be different for other frameworks, but the expanation should least point you in the right direction.

### One JavaScript file to rule them all

By default Vue will create a separate CSS file, a production sourcemap file, and a vendor libraries file that's an optimisation webpack performs by default for better caching of common dependencies that don't change often. These are sensible defaults, but for a shared JS bundle, assuming it isn't massive, we can start with the simplest thing that could possibly work, allowing cosumers to reference the one JS file that will work as expected. We can always introduce support for optimistations and sourcemaps and external CSS as needed later on. 

To tell Vue to build just one JavaScript file, you can add the following vue.config.js at the root of your Vue project next to pacakage.json.

```js
 module.exports = {
  configureWebpack: {
    optimization: {
      splitChunks: false
    }
  },
  css: {
    extract: false,
  },
  productionSourceMap: false
}
```

### A separate config.json file 

[Octopus variable subsitutions](https://octopus.com/docs/projects/variables/variable-substitutions) are powerful stuff, but in order to take advantage of them for our frontend project, we'd like to be able to tell Octopus about a config.json file that sits next to our bundle for Octopus to have its way with. To make Vue CLI include such a file in its dist folder that will be zipped to create the package sent to octopus, we can create "js\config.json" in the "public" folder Vue creates for us on initialization of a new project. Now if we run 

```console
npm run build       
```

we see that Vue has dutifully copied the config.json as a separate file in the output folder. To tell Vue to use it, we can create the following helper module.

```js
const configUrl = document.currentScript.src.substring(0, document.currentScript.src.lastIndexOf('/')) + '/config.json'

module.exports = async function() { 
    const response = await fetch(configUrl);
    return await response.json();
};
```

Now when we call this function, the bundle will fetch its neighbouring config.json in the S3 folder it has been deployed to.

Here's how we could use that in a Vue component.

```html
<template>
  <div id="app">
    <img alt="Vue logo" :src="`${config.bucketUrl}/assets/logo.png`">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import getConfig from './config.js'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data() {
    return { config: { } }
  },
  async created () {
    this.config = await getConfig();
  },
}
</script>
​```

Sidenote: you will indeed need to tell any images our other references to external files where to find them as shown above with the "bucketUrl" setting, as the relative paths Vue produces by default won't work if you want to reference shared assets that have been uploaded to S3 together with your bundle. 








## How do we do cache busting?
The CLIs that come with the major frontend frameworks will generate an index file that references our hash-named production bundle. Unfortunately, we probably cannot use that index file in legacy systems that want to consume our bundle. It can be unclear [which of the many cache busting strategies](https://css-tricks.com/strategies-for-cache-busting-css/) will suit best. You want to avoid anything that requires the consumer to know too much about the bundle, or the bundler to know too much about the consumer. The whole point is reuse - the last thing you need is each consumer implementing its own way of referencing the latest bundle. We also don't want the build process for our JavaScript containing special casing for the sake of different consumers. You'll score bonus pain points if you try to introduce runtime logic in the consumer to figure out which bundle to use. In my opinion, the perfect world is one where consumers know only that they have a config setting with a bundle URL.

Octopus Deploy can provide us with just that with using variable sets.

### What about configuration?
The boring issue of config might not be front of mind when you are in the honeymoon phase of your relationship with a shiny new JavaScript framework, but it's going to be less fun to work with if  

## Conclusion
I hope this post clarifies how we can apply the concepts of scoped variables, servers as cattle, and variable sets to achieve sane management of a shared JavaScript project. 

I've had good results following this strategy in production compared to other solutions I have tried for managing JavaScript projects. I did find myself explaining a few times to frontend specialists that they have to re-release the consumer project on test server to make it upgrade itself to the latest version of the bundle, but it's logical once they get the hang of it, but I've had very good feedback from the frontend specialists on my team on the way this deployment process pattern works.

Happy bundling!
