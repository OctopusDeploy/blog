---
title: Why you should take a look at Kotlin's standard library
description: See how run, let, also and apply can improve your Kotlin code.
author: matthew.casperson@octopus.com
visibility: public
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - Java
---

As we add support for Java deployments at [Octopus](https://octopus.com/download), more integration code is being written in Kotlin. As a long time Java developer, I took the opportunity to learn some of the improvements that the Kotlin language designers added to their language over Java.

About the same time I completed Dave Fancher's [Functional Programming with C#](https://app.pluralsight.com/library/courses/functional-programming-csharp/table-of-contents) course on Pluralsite. I enjoyed the course, because it provides some clear and practical advice on how to approach function programming in C#. In it Dave provides two extension methods, `Map` and `Tee`, which allow you to transform objects and pass them onto other mutating methods, along with examples on how and why you would use them.

I'd recommend the course, even for Kotlin developers, because the ideas presented by Dave map nicely (no pun intended) to similar methods Kotlin provides as part of its [standard library](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt).

The Kotlin `run` and `let` methods are roughly equivalent to the C# `Map` method, while the Kotlin `also` and `apply` methods are roughly equivalent to the C# `Tee` method.

So what are the differences between these standard functions? To demonstrate the differences I have created a simple Kotlin project, which you can find on [GitHub](https://github.com/mcasperson/kotlin-demo).

## run vs let

`run` and `let` are transformation functions. They take the value of the object they are called against, and return a new value.

The most visible difference between these function are the variables they expose to their block functions.

The `run` function exposes the value of the object that it was called from as `this` inside the block.

```java
@Test
fun runExample () {
    val result = "Local String".run {
        System.out.println(this) // prints "Local String"
        "New String"
    }
    System.out.println(result) // prints "New String"
}
```

The `let` function exposes the value of the object that it was called from as `it` inside the block, while `this` is retained from the outer scope.

```java
@Test
fun letExample () {
    val result = "Local String".let {
        System.out.println(this.name) // prints "Demo Class"
        System.out.println(it) // prints "Local String"
        "New String"
    }
    System.out.println(result) // prints "New String"
}
```

You can rename the default `it` parameter. You may do this to avoid clashing the default `it` parameter with an existing variable in the scope (which will happen if you nest two `let` functions).

```java
@Test
fun letExample2 () {
    val result = "Local String".let {me ->
        System.out.println(this.name) // prints "Demo Class"
        System.out.println(me) // prints "Local String"
        "New String"
    }
    System.out.println(result) // prints "New String"
}
```

## also vs apply

`also` and `apply` are typically used when the value of the object they are called against needs to be used for some mutating operation. Any return value from the `also` and `apply` blocks is ignored, and the value of the original object is returned.

In this way we can make use of the original value to perform some mutating logic (whose return value is not consumed by our own code), while retaining for the original value.

Like the `run` function, `apply` exposes the value of the object it is called against as `this`.

```java
@Test
fun applyExample() {
    val result = "Local String".apply {
        System.out.println(this) // prints "Local String"
        "New String" // return value is ignored
    }
    System.out.println(result) // prints "Local String"
}
```

Like the `let` function, `also` exposes the object that it was called from as `it` inside the block, while `this` is retained from the outer scope.

```java
@Test
fun alsoExample() {
    val result = "Local String".also {
        System.out.println(this.name) // prints "Demo Class"
        System.out.println(it) // prints "Local String"
        "New String" // return value is ignored
    }
    System.out.println(result) // prints "Local String"
}
```

And `it` can be renamed.

```java
@Test
fun alsoExample2() {
    val result = "Local String".also { me ->
        System.out.println(this.name) // prints "Demo Class"
        System.out.println(me) // prints "Local String"
        "New String" // return value is ignored
    }
    System.out.println(result) // prints "Local String"
}
```

## Transformation vs mutation

I've described `run` and `let` as transformation functions, and `also` and `apply` as mutation functions.

As you can see by the previous examples, `run` and `let` will also happily let you mutate state in their function blocks (as we have done by writing to the console), so the distinction between transformation and mutation is conceptual rather than enforced by the language.

However, this conceptual distinction is useful, as it allows you to describe the intention of your code, allowing you to gain an understanding of what the code does simply from its "shape".

## What is the shape of code?

So what do I mean by the "shape" of the code? Let's take a look at a simple function designed to deploy an AWS CloudFormation template.

```java
fun createCloudFormationStack(newStackName: String) {
  val credentialsProvider = ProfileCredentialsProvider()
  val client = AmazonCloudFormationClientBuilder
          .standard()
          .withCredentials(credentialsProvider)
          .withRegion(Regions.US_EAST_1)
          .build()

  val templatePath = javaClass.classLoader.getResource("WordPress.template.json").file
  val templateFile = File(templatePath)
  val template = FileUtils.readFileToString(templateFile, Charset.defaultCharset())

  val request = CreateStackRequest()
  request.stackName = newStackName
  request.templateBody = template

  val response = client.createStack(request)
  System.out.println("StackID: " + response.stackId)
}
```

Now let's take out all the function calls, and see what the shape of the code is.

```java
val credentialsProvider = ...
val client = ...

val templateFile = ...
val template = ...

val request = ...

val response = ...
```

What can we determine from the "shape" of this code?

The variable names give us some indication as to the kinds of objects we are creating, and the order of the variables does limit the kind of dependencies the objects have on each other. For example, `client` may or may not depend on `credentialsProvider`, but `credentialsProvider` can not rely on `client` because `credentialsProvider` is declared first.

Otherwise though there is not a lot of indication as to what this code is doing.

Let's take a look at a version using the standard library functions.

```java
fun createCloudFormationStack2(newStackName: String) {
    val client = ProfileCredentialsProvider().let { credentials ->
        AmazonCloudFormationClientBuilder
                .standard()
                .withCredentials(credentials)
                .withRegion(Regions.US_EAST_1)
                .build()
    }

    val template = javaClass.classLoader.getResource("WordPress.template.json").file.let { path ->
        File(path)
    }.let { templateFile ->
        FileUtils.readFileToString(templateFile, Charset.defaultCharset())
    }

    CreateStackRequest().also { request ->
        request.stackName = newStackName
        request.templateBody = template
    }.let { request ->
        client.createStack(request)
    }.also { response ->
        System.out.println("StackID: " + response.stackId)
    }
}
```

Stripping out all the function calls (leaving in the standard functions), we get this shape of the code.

```java
val client = ... .let { credentials ->
    ...
}

val template = ... .let { path ->
    ...
}.let { templateFile ->
    ...
}

... .also { request ->
    ...
}.let { request ->
    ...
}.also { response ->
    ...
}
```

What does this shape tell us?

* We can tell that something called `credentials` was transformed (via the `let` function) into the value assigned to `client`.
* We can tell something called `path` was transformed into something called `templateFile`, which in turn was transformed into the value assigned to `template`.
* We can tell that something called `request` was mutated (via the `also` function), and then transformed into something called `response`.
* We can tell that the `response` is used in a mutating function call.

## What are the benefits of the standard functions?

We can extract a lot more context from the code.

The transformations of objects are clearly described as:
* `credentials` -> `client`
* `path` -> `templateFile` -> `template`
* `request` -> `response`

Because the standard functions allow us to reduce the scope of some variables down to a single block, there are significantly fewer combinations of objects that could be used. The construction of the `template` variable could make use of the `client` (even though we don't, the shape of the code doesn't prevent it), and `client` and `template` could be used in the final block creating the `request` (which they are).

Compare that to the original code, where each of the 6 variables could make use of any combination of those that proceed it. Just looking at the shape of the first code example gives us no idea how all the variables are related.

Reducing variables also makes the code much easier to reason about. The [magical number](https://en.wikipedia.org/wiki/The_Magical_Number_Seven,_Plus_or_Minus_Two) describes the limit of people's memory capacity as somewhere between 5 and 9. Although this number is not a hard and fast rule, it rings true in my own experience.

The 6 variables from the first version of the code mean that function is pushing the limits of what an average person can hold in their memory. The 3 variables from the second version, plus one or two additional ones as we move in and out of the `let` and `also` function, places this code nicely within the average person's memory capacity.

We have also been able to quickly identity two mutating functions thanks to the calls to `also`. This gives us an idea of how much additional work it would be to test this code, as mutating functions often indicate the presence of external state that will need to be mocked and validated in a testing environment.

In our case the first mutation function is setting parameters on an object that doesn't have a builder interface, and doesn't accept the properties in the constructor. This doesn't require external state to be tracked.

However the second mutating function writes to the console, which may or may not be important to test depending on the context of the application.

## Conclusion

While they are quite simple, the standard functions in Kotlin provide a powerful way to describe the intention of your code. They will reduce variable counts, make code much easier to reason about, and highlight mutation.
