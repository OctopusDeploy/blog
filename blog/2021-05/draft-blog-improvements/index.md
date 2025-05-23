---
title: Draft blog post for blog improvement testing
description: This is where the meta description goes, keep it under 170 characters including spaces.
author: jed.lehmann@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage_deploying-a_nodejs_application_with_octopusdeploy_2021.png
bannerImage: blogimage_deploying-a_nodejs_application_with_octopusdeploy_2021.png
tags:
  - tag
---

![blog image](blogimage_deploying-a_nodejs_application_with_octopusdeploy_2021.png)

Introductory paragraph that tells the reader why they should read on.

Testing syntax without capitalization: **{{manage jenkins > configure system}}** 

## Body

The body of the post is where you share your hypothesis, how-to, or story.

### Subheadings

Use three ### to include H3 headings.

Use **Bold** text for UI labels, use single back-tics for `parameters` and `filepaths`, and three back-tics for code blocks:

```
Write-Host "Hello, World!"
```

Use the following (minus the backtics) to include images:

```
![Description of the image](/path/to/image.png "width=500")
```

## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)

The [Octopus blog](https://github.com/OctopusDeploy/blog) and the [Octopus documentation](https://github.com/OctopusDeploy/docs) are written in Markdown and rendered using [markdig](https://github.com/lunet-io/markdig). Markdig supports [GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown) as well as some extra syntax. 

- [Filenames](#filenames)
- [Files and directories](#files-and-directories)
- [YAML headers](#yaml-headers)
- [Table of contents](#table-of-contents)
- [Headings](#headings)
- [Formatting text](#formatting-text)
- [Images](#images)
- [Lists](#lists)
- [Tables](#tables)
- [Links](#links)
- [Navigation paths](#navigation-paths)
- [Code samples](#code-samples)
- [Call-outs](#call-outs)
- [Reuse text](#reuse-text)
- [Referencing Docker images](#referencing-docker-images)
- [Link to the Octopus Guides](#link-to-the-octopus-guides
)
- [Redirects](#redirects)

## Filenames

Markdown filenames are lowercase and end with `.md`. Use hyphens to separate words:

- `installation.md`
- `backup-and-restore.md`

## Files and directories

Directories must have an index file: `index.md`.

- directory/index.md
- directory/another-file.md

## YAML headers

The Markdown files in the blog and docs have a YAML header:

### YAML header (blog)

```md
---
title: Runbooks best practices <!-- post title -->
description: This post provides a step by step template you can use to generate high quality runbooks in Octopus
author: email@octopus.com <!-- use your email address -->
visibility: public <!-- options are public or private -->
published: 2020-03-09 <!-- The date the post will be published  -->
metaImage: runbooks-best-practices.png
bannerImage: runbooks-best-practices.png
tags: <!-- see blog/tags.txt for a comprehensive list of tags -->
 - Product
 - Runbooks
---
```

### YAML header (docs)

```md
---
title: Installation <!-- page title -->
description: How to install the Octopus Server.
position: 20 <!-- position of the document relative to the other documents in the same section -->
hideInThisSection: true  <!-- Optional. Hides the automatic "In this section" section that lists child documents in the same section. Leave out if not needed. -->
hideInThisSectionHeader: true <!-- Optional. Only hides the header for the "In this section" section -->
---
```

## Table of contents

Use `!toc` within the body of a page to include a table of contents that lists the sections on the current page.

## Headings

Use `##` to create h2 headers and `###` to create h3 headers.

The first header you include on a page must be a h2 header. The title of the page comes from the title in the YAML block.

## Formatting text

Bold text with `**` on both sides of text to create `**bold text**`: **bold text**.

Italicize text with `*` on both sides of text to create `*emphasized text*`: *emphasized text*.

## Images

Image filenames must be all lowercase. 

Add images to an `images/` directory in the same directory as the file that references the image.

Images are added to documents with the following syntax:

	![](images/image-name.png)

Images should include alt text for accessibility:

	![A brief description of the image](images/image-name.png)

Control the size of the image in pixels by adding: `width=500`:

	![A brief description of the image](images/image-name.png "width=500")

## Lists

Bullet lists are written with a hyphen at the beginning of the line:

```md
- Item 1
- Item 2
```

Which is rendered as:

- Item 1
- Item 2

Numbered lists are written with a number at the beginning of the line. The numbers do not need to increment as this will happen automatically:

```md
1. Item 1
1. Item 2
```

Which is rendered as:

1. Item 1
2. Item 2

You can nest lists by adding three spaces before the nested list items.

```md
1. Item 1
   1. Item 1.1
   1. Item 1.2
1. Item 2
```

Which is rendered as:

1. Item 1
   1. Item 1.1
   1. Item 1.2
1. Item 2

If you include an interruption between list items, you need to resume the list with the number the list should restart from:

```md
1. Item 1
1. Item 2

A break in the list.

3. Item 3
3. Item 4

```

Which is rendered as:

<ol>
<li>Item 1</li>
<li>Item 2</li>

A break in the list.

<li>Item 3</li>
<li>Item 4</li>
</ol>

## Tables

Tables are written in the following following way:

```
|  Header 1 | Header 2 | Header 3 | Header 4 | Header 4 |
|---|---|---|---|---|
| Row 1 Cell 1 | Cell 2 | Cell 3 | Cell 4 | Cell 4 |
| Row 2 Cell 1 | Cell 2 | Cell 3 | Cell 4 | Cell 4 |
```

There are generators online you can use to make this process a bit easier, for instance, [Table generator](https://www.tablesgenerator.com/markdown_tables).

## Links

To link to other pages within the **documentation**, use the following syntax (include the full filename and extension):

For more information, see the `[installation page](/docs/installation/index.md)` and review the `[installation requirements](/docs/installation/requirements.md)`.

This will link to https://www.octopus.com/docs/installation and https://www.octopus.com/docs/installation/requirements.

To link to other posts within the **blog**, use the following syntax (include the full filename and extension):

This is part one in a series of posts, read `[part two](blog/2020-03/blog-title-part-two.md)`.

This links to https://www.octopus.com/blog/blog-title-part-two.

Note, blog posts are organized in the repo into year-month folders, and you need to include this in your link.

### Anchor links

To link to a specific section within a document, add the section heading as an anchor and replace the spaces with hyphens:

Octopus can be installed on these versions of `[Windows Server](docs/installation/requirements.md#windows-server)`.

This will link to https://octopus.com/docs/installation/requirements#windows-server

If you'd like to control the anchor text (to ensure it doesn't change even if the title does), use the following syntax:

`## Windows Server {#windows-server}`

Special characters will break the anchor text, so don't include special characters in the anchor.

## Navigation paths

When instructing users to navigate through multiple options in the UI, use the following syntax:

`{{ infrastructure,Deployment Targets }}`

Which will be rendered:

**Infrastructure ➜ Deployment Targets**

## Code samples

Use GitHub-style fenced code blocks. For example:

    ​```ps
    Write-Host "Hello"
    ​```

If your example uses multiple languages or files, you can combine them together to add tab headings:

    ​```ps PowerShell
    Write-Host "Hello"
    ​```
    ​```cs C#
    Console.WriteLine("Hello");
    ​```

Snippets are highlighted by Highlight.js

* [Documentation](https://highlightjs.readthedocs.io/)
* [Language List](https://github.com/highlightjs/highlight.js/blob/master/SUPPORTED_LANGUAGES.md)

| language     | key            |
| ------------ | -------------- |
| c#           | cs          |
| xml          | xml          |
| no format    | no-highlight |
| command line | bash         |
| powershell   | ps           |
| json         | json         |
| sql          | sql         |
| f#           | fsharp       |
| python       | python       |
| text         | text           |

If no language is defined, highlightjs will guess the language, and it regularly gets it wrong.

## Call-outs

To create a call-out that draws the reader's attention, use the following syntax:

```md
:::warning
This release includes the following breaking changes...
:::
```

This will be rendered as:

```html
<div class="alert alert-warning">
<p>This release includes the following breaking changes...</p>
</div>
```

There are several keys, each of which map to a different colored alert:

| Key       | Color  |
| --------- | ------ |
| `success` | green  |
| `hint`    | blue   |
| `warning` | yellow |
| `problem` | red    |

Call-outs are added through bootstrap alerts [https://getbootstrap.com/components/#alerts](https://getbootstrap.com/components/#alerts).

## Reuse text

To create reusable text that is automatically added to any document that references it, add the text to a new file and save the file with a key followed by `.include.md`. For instance, `latest-version.include.md`, and save the file to the `docs/shared-content/` or `blog/shared-content/` directory respectively:

```md
The latest version of Octopus Deploy is 2020.1
```

## Link to the Octopus Guides

The Octopus Guides combine content to allow users to specify their entire CI/CD pipeline and access a guide for their specific pipeline. It is sometimes helpful to link to the guides with specific options pre-defined rather than the default options.

You can create the links to use by adding query parameters to the URL for the guides `https://www.octopus.com/docs/guides`:

- Application: add `?application=PHP`:

    [https://www.octopus.com/docs/guides?application=PHP](https://www.octopus.com/docs/guides?application=PHP)
- Build server: add `?build-server=jenkins`:

    [https://www.octopus.com/docs/guides?buildServer=Jenkins](https://www.octopus.com/docs/guides?buildServer=Jenkins)
- Source control: `sourceControl=TFVC`:

    [https://octopus.com/docs/guides?sourceControl=TFVC](https://octopus.com/docs/guides?sourceControl=TFVC)
- Package repository: `?packageRepository=Artifactory`:

    [https://octopus.com/docs/guides?packageRepository=Artifactory](https://octopus.com/docs/guides?packageRepository=Artifactory)
- Destination: `?destination=NGINX`:

    [https://octopus.com/docs/guides?destination=NGINX](https://octopus.com/docs/guides?destination=NGINX)

If you'd like to pre-fill more than one option, add multiple queries parameters to the URL:

https://octopus.com/docs/guides?application=PHP&buildServer=TeamCity&destination=NGINX

## Redirects

If you delete or rename a file in either the docs or blog repos, you must add a redirect for that file otherwise publishing will fail.

Redirects are added to `docs/redirects.txt` and `blog/redirects.txt` files respectively.

The redirects.txt file looks like this:
```
from-file-path -> to-file-path                 #DO NOT DELETE THIS LINE
docs/page1.md -> docs/page2.md
```
In the above example, `/docs/page1` is redirected to `/docs/page2`.

Add your redirect to the end of the file, after the redirect is added, the original file (`page1`) needs to be deleted from the repo.
