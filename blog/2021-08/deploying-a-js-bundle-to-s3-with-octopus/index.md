---
title: Deploying a JavaScript library project with octopus
description: Learn how to handle cache busting and config of a shared JavaScript library bundle and make it easy to reference in other Octopus projects.
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

A frontend dev pattern I've seen at many companies starts with the best of intentions, but it can lead to pain if not handled with care. You see a need for reuse of frontend code across multiple projects, possibly maintained by different teams using different tech on the server. You create a shared JavaScript library project with its own repo and release process. It's a sensible idea, but it opens questions that need good answers, to stop our little bundle of joy from growing into a monster. In this post, I'll explain a simple example of how to manage the deployment process for a shared JavaScript project that is simple to reference from other Octopus projects. My example uses a Vue JS bundle deployed to Amazon S3, but I hope you'll see how the same principles can be applied to any combination of frontend framework and hosting provider.

## The process

Our finished deployment process will look like this in Octopus.

![deployment process](aws%20bundle%20process.png)

I'll explain the reason for each step and how they work.

## Create an S3 bucket if it does not exist

In the spirit of treating servers like cattle and not pets, I don't assume much about our deployment target, beyond having an AWS account with appropriate permissions. In a specific case, I had dedicated buckets for combinations of different regions and the environments of test, staging, and production, so I appreciated a build process that only needs me to name the bucket and region in scoped variables and will set it up correctly if required. This is achieved with an [AWS CLI Step](https://octopus.com/docs/deployments/custom-scripts/aws-cli-scripts) that runs the following PowerShell script, which uses the AWS CLI to see if we get a non-error result trying to list the contents of the bucket. Otherwise, it creates the bucket, then polls to confirm the bucket exists before the step finishes.

```ps
$bucket = $OctopusParameters["s3-bucket-name"]
$region = $OctopusParameters["s3-region"]
$found = aws s3 ls s3://$bucket/ --recursive --summarize | Select-String -Pattern 'Total Objects:'
if ([string]::IsNullOrWhiteSpace($found)) {
    aws s3api create-bucket --bucket $bucket --region $region
    aws s3api wait bucket-exists --bucket $bucket
}
```

## Set S3 CORS policy

This is another AWS CLI script with the following PowerShell inline.

```ps
echo '{"CORSRules": [ { "AllowedOrigins": ["*"], "AllowedHeaders": [],"AllowedMethods": ["GET"] } ] }' | out-file -encoding ASCII cors.json
aws s3api put-bucket-cors --bucket bundle-s3 --cors-configuration file://cors.json
```

You could get more sophisticated with CORS as needed, but since in my example I'm assuming our bundles live in their own dedicated bucket, it makes sense to have a simplistic "allow all GET requests." The encoding step was important rather than just echoing straight to a file. I don't really know why the [CLI command](https://docs.aws.amazon.com/cli/latest/reference/s3api/put-bucket-cors.html) for setting CORS insists on reading from a file and won't just let me pass JSON through the command line, but if you desire a more complicated CORS policy, it might be cleaner to choose the "Script file inside a package" option and have the .ps1 and cors.json files source controlled in your bundle repo, rather than the inline option I've used here.

![script options](cors%20script.png)

## Upload bundle to S3

There are a few prerequisites for this next AWS CLI step to work as desired, which I'll explain before showing how it's set up in Octopus. I'll be showing how it was achieved in Vue, and the steps will be different for other frameworks, but the explanation should point you in the right direction.

### One JavaScript file to rule them all

By default, Vue will create a separate CSS file, a production source map file, and a vendor libraries file that's an optimization webpack performs by default for better caching of common dependencies that don't change often. These are sensible defaults, but for a shared JS bundle, assuming it isn't massive, we can start with the simplest thing that could work, allowing consumers to reference the one JS file to get all styling and behavior. We can always introduce support for optimizations and source maps and external CSS as needed later on.

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

[Octopus variable subsitutions](https://octopus.com/docs/projects/variables/variable-substitutions) are powerful stuff, but to take advantage of them for our frontend project, we'd like to be able to tell Octopus about a config.json file that sits next to our bundle for Octopus to have its way with. To make Vue CLI include such a file in its dist folder that will be zipped to create the package sent to octopus, we can create "js\config.json" in the "public" folder Vue creates for us on initialization of a new project. Now if we run

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

Now when we call this function, the bundle will fetch its neighboring config.json in the S3 folder it has been deployed to.

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
```

Sidenote: you will need to tell any images or other references to external assets where to find them as shown above with the use of the "bucketUrl" setting, because the relative paths Vue produces by default won't work on the consumer for assets in S3.

Now to tell Octopus to substitute variables in our config.json file, we can click the "Configure Features" button and enable "Structured configuration variables."

![config variables](config%20variables.png)

And tell Octopus to replace variables in "MyBundle\js\config.json" where "MyBundle" is the ID of your package.

![config variables 2](config%20variables%202.png)

### Upload your bundle!

Finally, we can give the step the CLI command to upload our bundle, config.json, and assets to a folder named after the current release.

```ps
aws s3 cp MyBundle s3://#{s3-bucket-name}/release_#{Octopus.Release.Number} --recursive --exclude index.html --acl public-read
```

I skip uploading the index.html file that Vue CLI produces because legacy consumers of our bundle won't be able to use that index.html and will need the URL to the uniquely named bundle file instead. Providing the URL of the newest bundle for the environment to any project that wants it is the focus of the next, final step.

## Update the bundle URL in a variable set

Being able to tell other projects where to get the cache-busted shared bundle through an automatically populated config setting is one of the big advantages of building an Octopus process for this type of JavaScript project. There's an [https://css-tricks.com/strategies-for-cache-busting-css/](astonishing variety) of strategies for cache busting and in my experience, many will lead to pain. The pain I have experienced on past projects that attempted this stemmed from either the consumer knowing too much about the bundling process or the bundler knowing too much about the consumer. In my opinion, the ideal world is where any project that consumes the bundle only needs to know to read a config setting with the URL of the bundle. It is too cool that Octopus allows us to achieve this at deployment time without  too much custom code.

After deployment, this custom script step will update the bundle URL for the scoped "BundleUrl" variable in a [library variable set](https://octopus.com/docs/projects/variables/library-variable-sets) that can be referenced by other projects. To do that, we reference the Octopus.Client NuGet package within the step as explained [here](https://octopus.com/docs/octopus-rest-api/octopus.client/using-client-in-octopus). The step also needs a reference to the package we deployed, as that's where it will find the name of the JavaScript file we uploaded. Then it can run the following PowerShell:

```ps
Add-Type -Path 'Octopus.Client/lib/net452/Octopus.Client.dll'

$bundle = Get-ChildItem -Path MyBundle/js/*.js | Select-Object -First 1

$endpoint = new-object Octopus.Client.OctopusServerEndpoint $octopusURI,$octopusApiKey
$repository = new-object Octopus.Client.OctopusRepository $endpoint

$scope = New-Object Octopus.Client.Model.ScopeSpecification
$enviornmentName = $OctopusParameters["Octopus.Environment.Name"]
$envID = $repository.Environments.FindByName($enviornmentName).Id
$scope.Add([Octopus.Client.Model.Scopefield]::Environment,(New-Object Octopus.Client.Model.ScopeValue($envID)))

$libraryVariableSetId = $repository.LibraryVariableSets.FindByName('BundleVariables').Id
$libraryVariableSet = $repository.LibraryVariableSets.Get($libraryVariableSetId);
$variables = $repository.VariableSets.Get($libraryVariableSet.VariableSetId);
$releaseId = $OctopusParameters["Octopus.Release.Number"]
$variables.AddOrUpdateVariableValue("BundleUrl", $bucketUrl + 'release_' + $releaseId + '/js/' + $bundle.Name,$scope)
$repository.VariableSets.Modify($variables)
```

That's it! Now any number of other projects can reference our shared JavaScript bundle by including the "BundleVariables" library variable set.

## Conclusion
I hope this post clarifies how we can apply the concepts of scoped variables, servers as cattle, and variable sets to achieve sane management of a shared JavaScript project.

I've had good results following this strategy in production compared to other solutions I have tried for managing JavaScript projects. I did find myself explaining a few times to frontend specialists that they have to re-release the consumer project to make it upgrade itself to the latest version of the bundle, but it's logical once people get the hang of it, and I've had good feedback from the frontend specialists on my team about the way this deployment process pattern works.

Happy bundling!
