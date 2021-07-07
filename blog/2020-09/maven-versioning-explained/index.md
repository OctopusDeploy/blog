---
title: Maven versions explained
description: There is more than meets the eye when it comes to Maven versions. Learn how Maven treats different version strings.
author: matthew.casperson@octopus.com
visibility: public
published: 2020-09-29
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
bannerImageAlt: Maven versions explained
tags:
 - Engineering
---

![Maven versions explained](java-octopus.png)

Version strings are usually easy to understand, but Maven has a number of rules and edge cases that are not immediately obvious. In this post, I take a look at how Maven version strings work.

## The source of truth

The Maven distribution includes a class called [ComparableVersion](https://github.com/apache/maven/blob/master/maven-artifact/src/main/java/org/apache/maven/artifact/versioning/ComparableVersion.java) which is the source of truth when it comes to how different version strings compare to each other.

By writing some tests against this class, we can explore how Maven versions work.

## A sorted list of versions

Let’s start with a test that takes an array of `ComparableVersion` objects, clones the array, sorts it, and compares it back to the original list. The fact that the test passes proves the original list is in order from the earliest to the latest version:

```java
private static final ComparableVersion[] VERSIONS = new ComparableVersion[]{
        new ComparableVersion("NotAVersionSting"),
        new ComparableVersion("1.0a1-SNAPSHOT"),
        new ComparableVersion("1.0-alpha1"),
        new ComparableVersion("1.0beta1-SNAPSHOT"),
        new ComparableVersion("1.0-b2"),
        new ComparableVersion("1.0-beta3.SNAPSHOT"),
        new ComparableVersion("1.0-beta3"),
        new ComparableVersion("1.0-milestone1-SNAPSHOT"),
        new ComparableVersion("1.0-m2"),
        new ComparableVersion("1.0-rc1-SNAPSHOT"),
        new ComparableVersion("1.0-cr1"),
        new ComparableVersion("1.0-SNAPSHOT"),
        new ComparableVersion("1.0"),
        new ComparableVersion("1.0-sp"),
        new ComparableVersion("1.0-a"),
        new ComparableVersion("1.0-RELEASE"),
        new ComparableVersion("1.0-whatever"),
        new ComparableVersion("1.0.z"),
        new ComparableVersion("1.0.1"),
        new ComparableVersion("1.0.1.0.0.0.0.0.0.0.0.0.0.0.1")
};

@Test
public void ensureArrayInOrder() {
    ComparableVersion[] sortedArray = VERSIONS.clone();
    Arrays.sort(sortedArray);
    assertArrayEquals(VERSIONS, sortedArray);
}
```

This list reveals some curious facts about how Maven versions compare to each other.

Qualifiers like `alpha`, `beta`, `milestone` (or their shorthand equivalents of `a`, `b`, and `mc`), `rc`, `sp`, `ga`, and `final` have special meanings. Separators like periods and dashes can be used interchangeably, or not used at all in some cases. And versions strings that don’t follow any particular format at all are still valid and comparable.

## Maven version components

While the `ComparableVersion` class is the source of truth for how versions compare to each other, it does not parse versions in a particularly useful data structure. For that, we have a second class from the [build helper](http://www.mojohaus.org/build-helper-maven-plugin/parse-version-mojo.html) plugin called [VersionInformation](https://github.com/mojohaus/build-helper-maven-plugin/blob/master/src/main/java/org/codehaus/mojo/buildhelper/versioning/VersionInformation.java).

`VersionInformation` breaks down Maven version strings into 5 parts:

* Major
* Minor
* Patch
* Build number
* Qualifier

The Major, Minor, Patch, and Build number are all integer values.

The Qualifier can hold any value, although some qualifiers do have special meaning.

## Qualifiers and Aliases

Maven recognizes a number of special qualifiers, shown here in order of precedence:

* alpha or a
* beta or b
* milestone or m
* rc or cr
* snapshot
* (the empty string) or ga or final
* sp

We saw in the list of sorted versions that these qualifiers do indeed result in Maven versions being sorted into the same order as the bullet point list.

Versions with unrecognized qualifiers are treated as later releases than an unqualified version, and unrecognized qualifiers are compared as case insensitive strings.

Some of the qualifiers have shorthand aliases. This test shows how various qualifiers result in equal Maven version:

```java
@Test
public void testAliases() {
    assertEquals(new ComparableVersion("1.0-alpha1"), new ComparableVersion("1.0-a1"));
    assertEquals(new ComparableVersion("1.0-beta1"), new ComparableVersion("1.0-b1"));
    assertEquals(new ComparableVersion("1.0-milestone1"), new ComparableVersion("1.0-m1"));
    assertEquals(new ComparableVersion("1.0-rc1"), new ComparableVersion("1.0-cr1"));
}

@Test
public void testDifferentFinalReleases() {
    assertEquals(new ComparableVersion("1.0-ga"), new ComparableVersion("1.0"));
    assertEquals(new ComparableVersion("1.0-final"), new ComparableVersion("1.0"));
}
```

Note that the shorthand aliases must have a number after them, while their complete equivalents do not. If you look closely at the list of sorted versions introduced at the start of this post, you will see that versions `1.0-alpha` and `1.0a1-SNAPSHOT` are two of the earliest versions, while `1.0-a` is toward the end of the list.

All qualifiers are case insensitive, as this test demonstrates:

```java
@Test
public void testCase() {
    assertEquals(new ComparableVersion("1.0ALPHA1"), new ComparableVersion("1.0-a1"));
    assertEquals(new ComparableVersion("1.0Alpha1"), new ComparableVersion("1.0-a1"));
    assertEquals(new ComparableVersion("1.0AlphA1"), new ComparableVersion("1.0-a1"));
    assertEquals(new ComparableVersion("1.0BETA1"), new ComparableVersion("1.0-b1"));
    assertEquals(new ComparableVersion("1.0MILESTONE1"), new ComparableVersion("1.0-m1"));
    assertEquals(new ComparableVersion("1.0RC1"), new ComparableVersion("1.0-cr1"));
    assertEquals(new ComparableVersion("1.0GA"), new ComparableVersion("1.0"));
    assertEquals(new ComparableVersion("1.0FINAL"), new ComparableVersion("1.0"));
    assertEquals(new ComparableVersion("1.0-SNAPSHOT"), new ComparableVersion("1-snapshot"));
}
```

Where version stings can not be parsed as major.minor.patch.build and the qualifier is not recognized, the entire string is considered a qualifier. These qualifiers are then compared as case insensitive strings:

```java
@Test
public void testQualifierOnly() {
    assertTrue(new ComparableVersion("SomeRandomVersionOne").compareTo(
            new ComparableVersion("SOMERANDOMVERSIONTWO")) < 0);
    assertTrue(new ComparableVersion("SomeRandomVersionThree").compareTo(
            new ComparableVersion("SOMERANDOMVERSIONTWO")) < 0);
}
```

## Separators

When transitioning from a digit to a qualifier, the use of a separator like a dash or period is optional:

```java
@Test
public void testSeparators() {
    assertEquals(new ComparableVersion("1.0alpha1"), new ComparableVersion("1.0-a1"));
    assertEquals(new ComparableVersion("1.0alpha-1"), new ComparableVersion("1.0-a1"));
    assertEquals(new ComparableVersion("1.0beta1"), new ComparableVersion("1.0-b1"));
    assertEquals(new ComparableVersion("1.0beta-1"), new ComparableVersion("1.0-b1"));
    assertEquals(new ComparableVersion("1.0milestone1"), new ComparableVersion("1.0-m1"));
    assertEquals(new ComparableVersion("1.0milestone-1"), new ComparableVersion("1.0-m1"));
    assertEquals(new ComparableVersion("1.0rc1"), new ComparableVersion("1.0-cr1"));
    assertEquals(new ComparableVersion("1.0rc-1"), new ComparableVersion("1.0-cr1"));
    assertEquals(new ComparableVersion("1.0ga"), new ComparableVersion("1.0"));
}
```

The same is not true when transitioning from a qualifier to a digit though:

```java
@Test
public void testUnequalSeparators() {
    assertNotEquals(new ComparableVersion("1.0alpha.1"), new ComparableVersion("1.0-a1"));
}
```

A dash or a period can be used to separate digits:

```java
@Test
public void testDashAndPeriod() {
    assertEquals(new ComparableVersion("1-0.ga"), new ComparableVersion("1.0"));
    assertEquals(new ComparableVersion("1.0-final"), new ComparableVersion("1.0"));
    assertEquals(new ComparableVersion("1-0-ga"), new ComparableVersion("1.0"));
    assertEquals(new ComparableVersion("1-0-final"), new ComparableVersion("1-0"));
    assertEquals(new ComparableVersion("1-0"), new ComparableVersion("1.0"));
}
```

## Long versions

While the `VersionInformation` class only recognizes the major.minor.patch.build format, the `ComparableVersion` class recognizes any number of digits:

```java
@Test
public void testLongVersions() {
    assertEquals(new ComparableVersion("1.0.0.0.0.0.0"), new ComparableVersion("1"));
    assertEquals(new ComparableVersion("1.0.0.0.0.0.0x"), new ComparableVersion("1x"));
}
```

## The complete test

This is the complete test class that was used to generate the examples above:

```java
package org.apache.maven.artifact.versioning;

import org.junit.Test;

import java.util.Arrays;

import static org.junit.Assert.assertTrue;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotEquals;
import static org.junit.Assert.assertArrayEquals;

public class VersionTest {

    private static final ComparableVersion[] VERSIONS = new ComparableVersion[]{
            new ComparableVersion("NotAVersionSting"),
            new ComparableVersion("1.0-alpha"),
            new ComparableVersion("1.0a1-SNAPSHOT"),
            new ComparableVersion("1.0-alpha1"),
            new ComparableVersion("1.0beta1-SNAPSHOT"),
            new ComparableVersion("1.0-b2"),
            new ComparableVersion("1.0-beta3.SNAPSHOT"),
            new ComparableVersion("1.0-beta3"),
            new ComparableVersion("1.0-milestone1-SNAPSHOT"),
            new ComparableVersion("1.0-m2"),
            new ComparableVersion("1.0-rc1-SNAPSHOT"),
            new ComparableVersion("1.0-cr1"),
            new ComparableVersion("1.0-SNAPSHOT"),
            new ComparableVersion("1.0"),
            new ComparableVersion("1.0-sp"),
            new ComparableVersion("1.0-a"),
            new ComparableVersion("1.0-RELEASE"),
            new ComparableVersion("1.0-whatever"),
            new ComparableVersion("1.0.z"),
            new ComparableVersion("1.0.1"),
            new ComparableVersion("1.0.1.0.0.0.0.0.0.0.0.0.0.0.1")

    };

    @Test
    public void ensureArrayInOrder() {
        ComparableVersion[] sortedArray = VERSIONS.clone();
        Arrays.sort(sortedArray);
        assertArrayEquals(VERSIONS, sortedArray);
    }

    @Test
    public void testAliases() {
        assertEquals(new ComparableVersion("1.0-alpha1"), new ComparableVersion("1.0-a1"));
        assertEquals(new ComparableVersion("1.0-beta1"), new ComparableVersion("1.0-b1"));
        assertEquals(new ComparableVersion("1.0-milestone1"), new ComparableVersion("1.0-m1"));
        assertEquals(new ComparableVersion("1.0-rc1"), new ComparableVersion("1.0-cr1"));
    }

    @Test
    public void testDifferentFinalReleases() {
        assertEquals(new ComparableVersion("1.0-ga"), new ComparableVersion("1.0"));
        assertEquals(new ComparableVersion("1.0-final"), new ComparableVersion("1.0"));
    }

    @Test
    public void testQualifierOnly() {
        assertTrue(new ComparableVersion("SomeRandomVersionOne").compareTo(
                new ComparableVersion("SOMERANDOMVERSIONTWO")) < 0);
        assertTrue(new ComparableVersion("SomeRandomVersionThree").compareTo(
                new ComparableVersion("SOMERANDOMVERSIONTWO")) < 0);
    }

    @Test
    public void testSeparators() {
        assertEquals(new ComparableVersion("1.0alpha1"), new ComparableVersion("1.0-a1"));
        assertEquals(new ComparableVersion("1.0alpha-1"), new ComparableVersion("1.0-a1"));
        assertEquals(new ComparableVersion("1.0beta1"), new ComparableVersion("1.0-b1"));
        assertEquals(new ComparableVersion("1.0beta-1"), new ComparableVersion("1.0-b1"));
        assertEquals(new ComparableVersion("1.0milestone1"), new ComparableVersion("1.0-m1"));
        assertEquals(new ComparableVersion("1.0milestone-1"), new ComparableVersion("1.0-m1"));
        assertEquals(new ComparableVersion("1.0rc1"), new ComparableVersion("1.0-cr1"));
        assertEquals(new ComparableVersion("1.0rc-1"), new ComparableVersion("1.0-cr1"));
        assertEquals(new ComparableVersion("1.0ga"), new ComparableVersion("1.0"));
    }

    @Test
    public void testUnequalSeparators() {
        assertNotEquals(new ComparableVersion("1.0alpha.1"), new ComparableVersion("1.0-a1"));
    }

    @Test
    public void testCase() {
        assertEquals(new ComparableVersion("1.0ALPHA1"), new ComparableVersion("1.0-a1"));
        assertEquals(new ComparableVersion("1.0Alpha1"), new ComparableVersion("1.0-a1"));
        assertEquals(new ComparableVersion("1.0AlphA1"), new ComparableVersion("1.0-a1"));
        assertEquals(new ComparableVersion("1.0BETA1"), new ComparableVersion("1.0-b1"));
        assertEquals(new ComparableVersion("1.0MILESTONE1"), new ComparableVersion("1.0-m1"));
        assertEquals(new ComparableVersion("1.0RC1"), new ComparableVersion("1.0-cr1"));
        assertEquals(new ComparableVersion("1.0GA"), new ComparableVersion("1.0"));
        assertEquals(new ComparableVersion("1.0FINAL"), new ComparableVersion("1.0"));
        assertEquals(new ComparableVersion("1.0-SNAPSHOT"), new ComparableVersion("1-snapshot"));
    }

    @Test
    public void testLongVersions() {
        assertEquals(new ComparableVersion("1.0.0.0.0.0.0"), new ComparableVersion("1"));
        assertEquals(new ComparableVersion("1.0.0.0.0.0.0x"), new ComparableVersion("1x"));
    }

    @Test
    public void testDashAndPeriod() {
        assertEquals(new ComparableVersion("1-0.ga"), new ComparableVersion("1.0"));
        assertEquals(new ComparableVersion("1.0-final"), new ComparableVersion("1.0"));
        assertEquals(new ComparableVersion("1-0-ga"), new ComparableVersion("1.0"));
        assertEquals(new ComparableVersion("1-0-final"), new ComparableVersion("1-0"));
        assertEquals(new ComparableVersion("1-0"), new ComparableVersion("1.0"));
    }
}
```

If you are interested in automating the deployment of your Java applications, sign up for a starter license for [Octopus Deploy](https://octopus.com/free), and take a look at [our documentation](https://octopus.com/docs/deploying-applications/deploy-java-applications).
