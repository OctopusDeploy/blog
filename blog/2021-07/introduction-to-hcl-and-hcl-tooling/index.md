---
title: Introduction to HCL and HCL tooling
description: Learn about the different tooling you can use with HCL.
author: adbertram@gmail.com
visibility: public
published: 2021-07-21-1400
metaImage: blogimage-config-as-code-explanation_2020.png
bannerImage: blogimage-config-as-code-explanation_2020.png
bannerImageAlt: A laptop screen displaying analytics data that is peeled up on one corner showing the code behind it
tags:
 - DevOps
 - Configuration as Code
---

![A laptop screen displaying analytics data that is peeled up on one corner showing the code behind it](blogimage-config-as-code-explanation_2020.png)

[HashiCorp Configuration Language (HCL)](https://www.terraform.io/docs/configuration/index.html) is a unique configuration language. It was designed to be used with HashiCorp tools, notably Terraform, but HCL has expanded as a more general configuration language. It’s visually similar to JSON with additional data structures and capabilities built-in.

HCL consists of three sub-languages: 

- Structural
- Expression
- Templates 

When combined, the sub-languages form a well structured HCL configuration file. This structure helps accurately and easily describe environmental configurations necessary for the Terraform tool.

Recently, HCL has migrated away from using Version 1 of the language in favor of Version 2. This post assumes we're talking about HCL2 instead of HCL1. 

HCL2 is a combination of HCL and the HashiCorp Interpolation Language (HIL). HIL adds string interpolation and a greater ability to use functions in variable declarations.

HCL can also be used with tools other than just Terraform. Over time different parsers have become available, such as Go, Java, and Python. In this post, I discuss getting started using HCL and which tools take advantage of its unique features.

## The HCL language and features

HCL is a JSON compatible language that adds features to help you use the Terraform tool to its highest potential. These features make HCL a [powerful configuration language](https://github.com/hashicorp/hcl/blob/hcl2/hclsyntax/spec.md) and address some of JSON's shortcomings.

- Comments are available as single line or multi-line:
    - Single Line: `#` or `//`.
    - Multi-Line: `/* */` (no nesting of block comments).
- Variable assignments use the `key = value` construction where whitespace does not matter and the value can be a primitive such as a string, number, boolean, object, or a list.
- Strings are quoted and can contain any UTF-8 characters.
- Numbers can be written and parsed in a number of different ways:
    - Base 10 numbers are the default.
    - Hexadecimal: Prefix a number with `0x`.
    - Octal: Prefix a number with a `0`.
    - Scientific numbers: Use the notation such as `1e10`.
- Arrays and lists of objects are easy to create using `[]` for arrays and `{ key = value }` for lists.

This overview only scratches the surface of what HCL can do. It’s easier to see how HCL works by exploring an example configuration file and analyzing how it works.

## Creating a simple HCL configuration file

To understand what an HCL configuration actually looks like, let's create a simple configuration, demonstrating a few of the features available:

```bash
/*
Define the default configuration values here
*/
default_address = "127.0.0.1"
default_message = upper("Incident: ${incident}")
default_options = {
  priority: "High",
  color: "Red"
}

incident_rules {
	# Rule number 1
	rule "down_server" "infrastructure" {
		incident = 100
		options  = var.override_options ? var.override_options : var.default_options
		server   = default_address
		message  = default_message
	}
}
```

You may notice we use a function call and string interpolation for the `default_message` variable. The `upper()` function will make a string uppercase, while the `${}` construct replaces the variables inside it with their values in a given string.

Another feature that stands out, compared to other configuration languages, is the `rule "down_server" "infrastructure"` format. This is a `type label label` format. In this example, we define the rule type and the rule category for use in our application.

You can also see we use a ternary conditional for the `options` variable. If the `override_options` variable exists we use that value, otherwise we fall back to using the `default_options`. This configuration demonstrates the powerful ability for HCL to take advantage of logic, string interpolation, and operations from within the language itself.

## Editing HCL configurations easily

[Visual Studio Code](https://code.visualstudio.com/) (VS Code) is currently one of the most popular editors and it’s offered for free by Microsoft. VS Code has extensions that add additional functionality to the base editor. These include:

- An [HCL extension](https://marketplace.visualstudio.com/items?itemName=wholroyd.HCL) to offer proper language colorization.
- The [Terraform extension](https://marketplace.visualstudio.com/items?itemName=4ops.terraform) that also adds HCL support, despite being named for one of the HashiCorp tools. This extension offers syntax highlighting and basic validation.

The [Atom editor](https://atom.io/) is an alternative editor to VS Code and is also popular. It offers an [HCL syntax highlighting](https://atom.io/packages/language-hcl) package as well. 

## HCL processing and tooling in other languages

So far, I've talked about using HCL in the context of the HashiCorp tools, but there is other tooling that consumes a HCL file for use in different applications. 

One example is the [hclq command-line processor](https://github.com/mattolenik/hclq). This command-line processor offers the following features:

- Inspection and validation of configurations.
- An alternative to parsing files with `grep` or `sed`.
- Preprocessing for value interpolation.

:::hint
Note that this tool doesn't currently have HCL2 support but it is planned.
:::

Many organizations now use the Go programming language, so it may be an attractive option to use HCL in a Go program. There is a [Go package](https://godoc.org/github.com/hashicorp/hcl) that can decode HCL into usable Go structures.

This module consumes the HCL configuration and decodes the input into an abstract syntax tree (AST) that enables easy manipulation of the configuration from within Go. Integrating HCL2 into server-side Go programs allows you to use complex configurations that are easy to understand.

For Python, there is an [HCL2 parser](https://pypi.org/project/python-hcl2/) that does not support HCL1 but for most projects this isn't necessary. Built using Lark, a parsing toolkit, Python HCL2 makes it easy to integrate an HCL2 configuration into any Python project.

## Conclusion

The HashiCorp Configuration Language started out specific to HashiCorp but has evolved to become more attractive in a variety of projects. 

The recent HCL2 rewrite further incorporates string interpolation and additional functions. This increases the usability of an already flexible language. The power of a configuration language that's easy to understand with built-in templating, is quickly making HCL2 the language of choice for complex configurations.

Happy deployments!

---

Adam Bertram is a 20+ year veteran of IT and an experienced online business professional. He’s a consultant, Microsoft MVP, blogger, trainer, published author and content marketer for multiple technology companies. Catch up on Adam’s articles at [adamtheautomator.com](http://adamtheautomator.com/), connect on [LinkedIn](https://www.linkedin.com/in/adbertram), or follow him on Twitter at [@adbertram](https://twitter.com/adbertram).
