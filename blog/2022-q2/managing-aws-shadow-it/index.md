---
title: Identifying AWS shadow IT resources
description: As part of our series on Octopus Runbooks, learn how to find unmanaged resources in AWS using runbooks.
author: matthew.casperson@octopus.com
visibility: private
published: 2022-05-02-1400
metaImage: blogimage-identifyingawsshadowitresources-2022.png
bannerImage: blogimage-identifyingawsshadowitresources-2022.png 
bannerImageAlt: Detective-like person with a hat and trench coat holds a giant magnifying glass over a cloud revealing a web server.
isFeatured: false
tags: 
  - DevOps
  - Runbooks Series
  - Runbooks
  - AWS
---

The term shadow IT often refers to adhoc IT resources deployed by members of a DevOps team to solve an immediate need, but which lack the kind of ongoing maintenance and management to support long lived infrastructure.

In AWS, shadow IT takes the form of virtual machines, S3 buckets, virtual private clouds, and any other kind of resource created on the fly, usually through the web console. 

In this post, I explain some of the problems associated with shadow IT and how to identify unmanaged resources in your AWS account.

## The issues with shadow IT

Imagine you're a new member of a DevOps team tasked with resolving a failed AWS VM simply called "Web Server". Who created this VM? What applications does it host? Where are the backups located? How can the VM be recreated? 

None of this information is readily available from the VM resource itself, especially if it's failed to the point of no longer being able to log into it. Not having access to this information makes it almost impossible to provide support.

Any growing team that fails to implement standard practices around the creation of new cloud resources will likely find themselves years later with no idea what any given resource does. This presents security challenges, because it's unclear who's responsible for tasks like OS patching, or what networking rules are appropriate when a permissive "allow all" rule set was originally applied. It also makes it difficult to know if the current state of the resource is correct, or if someone accidentally made undesirable changes.

In AWS, the first step towards describing resource attributes like who owns it and its purpose are to apply a set of standardized tags. The next step is to ensure all resources are created with declarative templates like CloudFormation, which allows resources to be recreated if needed, and also allows the detection of unwanted manual changes, known as drift.

To identify noncompliant resources, you'll create two simple runbooks to scan your AWS account for resources without the expected tags.

## Identifying untagged resources

The following Bash script scans the entire AWS account for resources that lack any of the required tags. In this example, the tags `Team`, `Deployment Project`, and `Environment` refer to the team that owns the resource, the CI or Octopus project that deployed it, and the environment (such as development or production) that the resource belongs to:

```bash
REQUIREDTAGS=("Team"  "Deployment Project"  "Environment")
OUTPUT=$(aws resourcegroupstaggingapi get-resources --tags-per-page 100)

for ((i = 0; i < ${#REQUIREDTAGS[@]}; i++)); do
    COUNT=$(echo ${OUTPUT} | jq -r "[.ResourceTagMappingList[] | select(contains({Tags: [{Key: \"${REQUIREDTAGS[$i]}\"} ]}) | not)] | length")
    echo "==========================================================="
    echo "The following ${COUNT} resources lack the ${REQUIREDTAGS[$i]} tag."
    echo "==========================================================="
    echo ${OUTPUT} | jq -r ".ResourceTagMappingList[] | select(contains({Tags: [{Key: \"${REQUIREDTAGS[$i]}\"} ]}) | not) | .ResourceARN"
done
```

## Identifying unmanaged resources

Finding resources that were not created by a CloudFormation template (i.e., unmanaged resources) is very similar to the script above.

Almost all AWS resources support tags, and when those resources are created by a CloudFormation template, the `aws:cloudformation:stack-id` tag is automatically applied. This means identifying unmanaged resources is as simple as finding any resources that lack the `aws:cloudformation:stack-id` tag.

The Bash script below scans all resources, except for CloudFormation stacks, that lack the `aws:cloudformation:stack-id` tag:

```bash
OUTPUT=$(aws resourcegroupstaggingapi get-resources --tags-per-page 100)
COUNT=$(echo $OUTPUT | jq -r '[.ResourceTagMappingList[] | select(contains({Tags: [{Key: "aws:cloudformation:stack-id"} ]}) | not) | select(.ResourceARN | test("arn:aws:cloudformation:[a-z]+-[a-z]+-[0-9]+:[0-9]+:stack/.*") | not)] | length')

echo "==========================================================="
echo "The following ${COUNT} resources were not created by CloudFormation"
echo "==========================================================="
echo $OUTPUT | jq -r '.ResourceTagMappingList[] | select(contains({Tags: [{Key: "aws:cloudformation:stack-id"} ]}) | not) | select(.ResourceARN | test("arn:aws:cloudformation:[a-z]+-[a-z]+-[0-9]+:[0-9]+:stack/.*") | not) | .ResourceARN'
```

:::hint
Note that at the time of writing there are open issues where tags like `aws:cloudformation:stack-id` are not applied to some resources. For example, [SQS topics do not have the tags applied](https://github.com/aws-cloudformation/cloudformation-coverage-roadmap/issues/652).
:::

## Resolving noncompliant resources

Defining tags on resources that lack them is usually a case of manually adding them through the web console or CLI. The script below shows an example of adding common tags to resources in bulk:

```bash
aws resourcegroupstaggingapi tag-resources --resource-arn-list \
    arn:aws:lambda:us-west-1:133577413914:function:Production-audits-0-SQS \
    arn:aws:lambda:us-west-1:133577413914:function:Production-product-0-InitDB \
    arn:aws:lambda:us-west-1:133577413914:function:Production-GithubActionWorkflowBuilderGithubOAuthCodeProxy \
    arn:aws:lambda:us-west-1:133577413914:function:Production-audits-0-Web \
    arn:aws:lambda:us-west-1:133577413914:function:Production-audits-0-InitDB \
    --tags Environment=Development \
    --region us-west-1
```

The process of importing unmanaged resources into a CloudFormation stack is described in the [AWS documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resource-import-existing-stack.html).

## Conclusion

Identifying shadow IT resources in your AWS account is the first step to establishing infrastructure that can be effectively managed by your DevOps teams. Then, by establishing a consistent tagging scheme, you're able to document who is responsible for what, the purpose of resources, and which external processes created them.

In this post, you saw a number of scripts for finding noncompliant resources, and tips on how to add missing tags or import resources into CloudFormation stacks. These simple steps can make a huge difference as your infrastructure grows in size and complexity.

!include <q2-2022-newsletter-cta>

Happy deployments!