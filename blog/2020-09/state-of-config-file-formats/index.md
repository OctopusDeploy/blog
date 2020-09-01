
---
title: "The state of config file formats: XML vs. YAML vs. JSON vs. HCL"
description: 
author: adbertram@gmail.com
visibility: private
published: 2029-09-01
metaImage: 
bannerImage: 
tags:
 - DevOps
 - Config as Code
---

Every application, environment, or system requires some level of configuration. Configuration formats, such as the `ini` format to plain text-based files, have waxed and waned in popularity over the years. As application and system needs have evolved, so did the configuration complexity and structure that's needed.

Today a multitude of configuration formats exist, but some have taken center stage in modern cloud environments. XML, JSON, YAML, and HCL support complex configurations, each with their advantages and disadvantages.

## [Extensible Markup Language (XML)](https://www.w3.org/XML/)

Among the first structured configuration languages, XML has been around for decades. XML first entered the scene in 1998 with W3C's version 1.0 specification. Designed for general use and simplicity, XML became the underpinning of many transport languages, services, and configuration formats.

Despite XML's ubiquity in the world of computing, newer configuration languages are overtaking XML's dominance. There are still many situations where the structured nature of XML and its flexibility works best for complex configurations. Simple configuration languages, such as JSON, work for many applications, but when you need proper validation, schema, and namespace support, XML is often best.

### When should you use XML?

- XML is a generalized language that easily allows different formats to be realized from a common syntax.
- Schemas exist for validation and creation of custom types, while namespaces avoid collisions between elements.
- XPath and XQuery facilitate complex queries into XML documents.

### When should you not use XML?

- The XML language is verbose and often contains redundant syntax.
- With the higher verbosity, comes increased storage capacity and bandwidth needs.
- Not considered easily human-readable due to the descriptive nature of elements.

## [JavaScript Object Notation (JSON)](https://www.json.org/)

Recognized as a formal specification in 2013, JSON has been around since the early 2000s. Used in a multitude of different applications, JSON emphasizes readability and simple syntax. Derived from JavaScript, JSON is language independent. JSON parsers exist for many different programming languages and tooling.

In contrast to XML, JSON offers a simpler alternative. With a clean and easy to use syntax, JSON has taken over many configuration setups. Developers appreciate the simplicity and speed of processing that JSON offers. One of the most common languages in use today, JSON is popular despite the drawbacks inherent in its design.

### When should you use JSON?

- In contrast to XML, far more human-readable due to a more compact syntax.
- Simpler syntax with limited markup.
- Quick for systems and languages to parse.
- JSONPath, similar to XPath, is available for complex queries.

### When should you not use JSON?

- Limited data type support containing only *strings, numbers, JSON object, array, boolean, and null*.
- No namespace, comment, or attribute support.
- Simple structures may not support complex configurations.

## [YAML Ain't Markup Language (YAML)](https://yaml.org/)

First known as Yet Another Markup Language, the official definition changed to express a data focus over a document focus. Created in 2001, YAML does not have a formal W3C specification. The latest YAML 1.2 specifications have been stable since 2009. Intended for defining configurations instead of as a data transport language, YAML is very human-readable.

Despite the differences in how YAML may look to JSON, YAML is a superset of JSON. As a superset of JSON, a valid YAML file can contain JSON. Additionally, JSON can transform into YAML as well. YAML itself can also contain JSON within its configuration files. 

### When should you use YAML?

- Exceptionally human-readable syntax.
- Compact syntax, which uses Python-style indentation to denote structure.
- Supports language-independent object types. Scalers such as `int`, `binary`, `str`, and `bool` or collections such as `map`, `set`, `pairs`, and `seq`.

### When should you not use YAML?

- Indentation format is prone to syntax and validation errors.
- Portability with certain types may not exist due to lack of features across all languages.
- Due to the declarative nature of YAML debugging is difficult. Breakpoints and similar functionality does not exist.

## [Hashicorp Configuration Language (HCL)](https://github.com/hashicorp/hcl2/blob/master/hcl/hclsyntax/spec.md)

The only specific configuration language listed, HCL is not intended to replace languages such as YAML or JSON. HCL is a tool specific language intended for use with the Terraform toolset. HCL looks visually similar to JSON, with unique data structures added.
HCL is a combination of three sub-languages, structural, expression, and template languages. Taken together, these sub-languages combine to create an HCL configuration. This schema efficiently describes environmental configurations, the primary purpose of Terraform.
Are other toolsets and languages able to use HCL? Parsers exist for other languages such as Go, Java, and Python. In that context, it is possible to use HCL as a configuration language. These parsers are typically not native packages.

### When should you use HCL?

- Using Terraform is far easier when using HCL.
- Visually and functionality similar to JSON in addition to being compatible.
- With features such as attributes and templates, HCL is more full-featured than competing languages, such as JSON.

### When should you not use HCL?

- As evidenced by Hashicorp being in the name, HCL was developed primarily for its products, and development is geared towards those ends.
- Despite stated compatibility with JSON, a strict conversion between the two languages is not straightforward due to ambiguity in mapping certain language constructs.
- As a relatively new language, many features are still being developed and toolkit maturity is still growing.

## Conclusion

There are many configuration languages, each with its advantages and disadvantages. XML, JSON, YAML, and HCL are somewhat interoperable, but depending on the language chosen, trade-offs are necessary.

Simple applications and environments will benefit from a structured configuration. As configuration complexity increases, so do the disadvantages of one language over another. Deciding configuration goals early in the application lifecycle will help in choosing the right language.

As configuration itself becomes a first-class citizen within the DevOps environment and cloud computing, the importance of choosing the right language for your application becomes critical to ensuring success. Although any of these configuration languages will work well, weighing the advantages and disadvantages of each will ultimately lead to a successful configuration framework.
