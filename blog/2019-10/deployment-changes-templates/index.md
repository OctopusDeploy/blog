In `2019.4` we introduced a number of new features around getting build information into Octopus and then utilizing that information in release notes templates. In this post we're going to look at Deployment Changes Templates, a new feature that builds on this previous work.

While this work compliments the build information and release notes templates, it does not rely on it. So if you're interested in controlling the formatting of the output of the changes in your deployments, this feature might be for you.

## Why Deployment Changes Templates?

The key problem that the deployment changes templates are aimed at addressing is controlling the output of the deployment changes information. Why is this important? 

Well, when we introduced the functionality to provide build information to Octopus we included the ability to template the release notes that were created for releases. We also aggregated the release changes information, across releases, onto deployments and made the information available to the step, e.g. so it could be used on email steps and so on.

We then also used the deployment's release changes information to render information in the portal UI, but we hard coded the layout for it. This had some limitations for some customers though, for example where information got duplicated because of the nuances of their development pipeline.

As another example, it meant that if you had an email and a slack notification you had to format the information twice, and maintain it in separate steps.

## Deployment Changes Templates

Our solution to these issues was to include a new template option in the project settings, alongside the other deployment level settings defined there.

The template is applied during the creation of the deployment and has access to almost all of the same variables that the deployment does. The key exception is the machine related variables, they don't exist in the creation scope and are therefore not available.

Something we have added though is a new variable, `Octopus.Deployment.Targets`, that is only available to the template. It represents the targets, across all steps in the process, that are included in the deployment. Each target object has an `Id` and `Name` property and an example usage is

```
#{each target in Octopus.Deployment.Targets}
- #{target.Name}
#{/each}
```

The output of the template is stored in a variable called `Octopus.Deployment.ChangesMarkdown`, which is available during the deployment and could be used in an email templates like so

```
#{Octopus.Deployment.ChangesMarkdown | MarkdownToHtml}
```

## Do I have to use Deployment Changes Templates with Release Notes Templates?

In short, no. As mentioned earlier the features certainly compliment either other, but the Deployment Changes Template in now way depends on build information or release notes templates. The changes data for the deployment contains all of the release notes for the releases, which in itself could still be valuable in a number of scenarios.

We think the inclusion of the Targets information in the template will also make this feature valuable even if the targets is all that it is being used for.

## Wrap up

This feature closes out the core pieces of the broader story we have been working towards with build information and release notes templates. Our aim has been that each of these pieces form a coherent part of the whole, while still being capable of standing alone to provide value.

