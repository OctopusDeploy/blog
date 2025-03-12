---
title: "The state of config file formats: XML vs. YAML vs. JSON vs. HCL"
description: Learn about the different configuration file formats that are available and when you should use them.
author: adbertram@gmail.com
visibility: public
published: 2021-07-14-1400
metaImage: blogimage-config-as-code-explanation_2020.png
bannerImage: blogimage-config-as-code-explanation_2020.png
tags:
 - DevOps
 - Configuration as Code
---

![The state of config file formats: XML vs. YAML vs. JSON vs. HCL](blogimage-config-as-code-explanation_2020.png)

Every application, environment, or system requires some level of configuration. Configuration formats, such as the `ini` format to plain text-based files, have fluctuated in popularity over the years. As application and system needs have evolved, so has the configuration complexity and structure that’s required.

Today, many configuration formats exist, but some are more popular in cloud environments. XML, JSON, YAML, and HCL support complex configurations, each with their advantages and disadvantages.

## [Extensible Markup Language (XML)](https://www.w3.org/XML/)

Among the first structured configuration languages, XML was introduced in 1998 with W3C’s version 1.0 specification. Designed for general use and simplicity, XML underpinned many transport languages, services, and configuration formats.

Despite XML’s ubiquity, newer configuration languages are overtaking XML’s dominance. There are still many situations though, where the structured nature of XML and its flexibility works best for complex configurations. 

Simple configuration languages, such as JSON, work for many applications, but when you need proper validation, schema, and namespace support, XML is often best.

### Advantages of XML

- XML is a generalized language that easily allows different formats to be realized from a common syntax.
- Schemas exist for validation and creation of custom types, while namespaces avoid collisions between elements.
- XPath and XQuery facilitate complex queries into XML documents.

### Disadvantages of XML

- The XML language is verbose and often contains redundant syntax.
- Higher verbosity increases storage capacity and bandwidth needs.
- Not considered easily human-readable due to the descriptive nature of elements.

## [JavaScript Object Notation (JSON)](https://www.json.org/)

Recognized as a formal specification in 2013, JSON has been around since the early 2000s. It's used in a multitude of different applications and emphasizes readability and simple syntax. 

Derived from JavaScript, JSON is language independent. JSON parsers exist for many different programming languages and tooling.

JSON is a simpler alternative to XML. With a clean and easy to use syntax, JSON has taken over many configuration setups. Developers appreciate the simplicity and speed of processing that JSON offers. 

JSON is one of the most common languages in use today, and is popular despite the drawbacks inherent in its design.

### Advantages of JSON

- In contrast to XML, far more human-readable due to a more compact syntax.
- Simpler syntax with limited markup.
- Quick for systems and languages to parse.
- JSONPath, like XPath, is available for complex queries.

### Disadvantages of JSON

- Limited data type support containing only *strings, numbers, JSON object, array, boolean, and null*.
- No namespace, comment, or attribute support.
- Simple structures may not support complex configurations.

## [YAML Ain’t Markup Language (YAML)](https://yaml.org/)

First known as Yet Another Markup Language, the official definition changed to express a data focus over a document focus. Created in 2001, YAML doesn't have a formal W3C specification. The latest YAML 1.2 specifications have been stable since 2009. 

YAML is very human-readable, because it’s intended for defining configurations instead of as a data transport language.

Although YAML looks different to JSON, YAML is a superset of JSON. As a superset of JSON, a valid YAML file can contain JSON. Additionally, JSON can transform into YAML as well. YAML itself can also contain JSON in its configuration files.

### Advantages of YAML

- Exceptionally human-readable syntax.
- Compact syntax, which uses Python-style indentation to denote structure.
- Supports language-independent object types. Scalers such as `int`, `binary`, `str`, and `bool` or collections such as `map`, `set`, `pairs`, and `seq`.

### Disadvantages of YAML

- Indentation format is prone to syntax and validation errors.
- Portability with certain types may not exist due to a lack of features across all languages.
- Debugging is difficult, due to the declarative nature of YAML. 
- Breakpoints and similar functionality do not exist.

## [HashiCorp Configuration Language (HCL)](https://github.com/hashicorp/hcl2/blob/master/hcl/hclsyntax/spec.md)

The only specific configuration language listed, HCL is not intended to replace languages such as YAML or JSON. HCL is a tool-specific language intended for use with the Terraform toolset. HCL looks visually similar to JSON, with unique data structures added.

HCL is a combination of three sub-languages: 

- Structural
- Expression
- Template 

Together these sub-languages create an HCL configuration. This schema efficiently describes environmental configurations, the primary purpose of Terraform.

Are other toolsets and languages able to use HCL? Parsers exist for other languages such as Go, Java, and Python. In that context, it's possible to use HCL as a configuration language. These parsers are typically not native packages.

### Advantages of HCL

- Using Terraform is far easier when using HCL.
- Visually and functionality similar to JSON in addition to being compatible.
- With features such as attributes and templates, HCL is more full-featured than competing languages, such as JSON.

### Disadvantages of HCL

- As evidenced by HashiCorp being in the name, HCL was developed primarily for its products, and development is geared towards those ends.
- Despite stated compatibility with JSON, a strict conversion between the two languages is complicated by ambiguity in mapping certain language constructs.
- As a new language, many features are still being developed and toolkit maturity is still growing.

## Conclusion

There are many configuration languages, each with their advantages and disadvantages. XML, JSON, YAML, and HCL are somewhat interoperable, but trade-offs are necessary depending on the language you choose.

Simple applications and environments benefit from a structured configuration. As configurations increase in complexity, so do the disadvantages of one language over another. 

To choose the right language, it helps to decide on configuration goals early in the application lifecycle.

As configuration itself becomes a first-class citizen in the DevOps environment and cloud computing, choosing the right language for your application becomes critical to ensure success. Although any of these configuration languages will work well, you need to weigh the advantages and disadvantages for a successful configuration framework.

Happy deployments!

---

Adam Bertram is a 20+ year veteran of IT and an experienced online business professional. He’s a consultant, Microsoft MVP, blogger, trainer, published author and content marketer for multiple technology companies. Catch up on Adam’s articles at [adamtheautomator.com](http://adamtheautomator.com/), connect on [LinkedIn](https://www.linkedin.com/in/adbertram), or follow him on Twitter at [@adbertram](https://twitter.com/adbertram).

!include <related-content>
