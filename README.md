# Octopus blog

This repository contains the [Octopus.com/blog](https://octopus.com/blog/) posts.

Authors must sign the [Contribution License Agreement (CLA)](https://cla-assistant.io/OctopusDeploy/docs) before we can accept your contribution.

See the [Octopus style guide](https://octopusdeploy.github.io/OctoStyle/) for the following information:

- [Markdown quick reference](https://octopusdeploy.github.io/OctoStyle/markdown)
- [Writing tips for the blog](https://octopusdeploy.github.io/OctoStyle/writing-tips-for-the-blog) 
- [Blog template](templates/readme.md)

## Snippets

The blog includes snippets from the [snippets repo](https://github.com/OctopusDeploy/snippets). For more information see [Octopus snippets](https://octopusdeploy.github.io/OctoStyle/octopus-snippets).

If the latest snippets do not appear in the output, use the following to update the snippets:

```
cd blog/blog/snippets/
git fetch
git merge
```

And merge the blog repo.

# How to submit a blog post 

Internal authors can create a branch for their work, external authors need to fork the repo.

Posts are organized in year-month directories (i.e., 2020-01/), find or create the directory that is roughly when your post is going out and add your files there. 

When you're happy with your post and think it's ready to be reviewed, create a PR and assign @robpearson and @wordlee as reviewers.

If you're drafting a post that you'd like to keep private until it's published, use the [internal blog drafts repo](https://github.com/OctopusDeploy/internal-blog-drafts).
