---
title: The different types of software tests
description: This post explains why testing is important. The post discusses the two methods of testing, manual and automated and the two broad types of testing, functional and non-functional. The post gives some examples of different types of tests.
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Testing
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

## Why is testing important?

Testing is part of continuous delivery. Testing assures each stage of the delivery pipeline for quality before moving on to the next stage. DevOps is an iterative cycle of building, testing, and releasing. A comprehensive testing environment will ensure that each iteration of the DevOps loop strengthens the quality of the product. A weak testing phase will mean defects progress to release, and developers must fix bugs while the product is live. Development teams fall on both sides of the testing spectrum. A survey by [Mabel on the state of testing in DevOps](https://www.dropbox.com/s/nnagymzdcnoswc6/Benchmark-Report-State-of-Testing-in-DevOps.pdf?dl=0) indicates that automated testing (at least 4-5 different types of tests) is key to customer happiness. The [2021 State of DevOps DORA Report](https://www.dropbox.com/s/xycst8qsxnpsieu/state-of-devops-2021.pdf?dl=0), continuous testing is an indicator of success: `continuous testing is a strong predictor of
successful continuous delivery. Elite performers who meet their reliability targets are 3.7 times more likely to leverage continuous testing`


This blog discusses automated and manual tests and common types of tests: functional and non-functional. At Octopus Deploy, we help make complex deployments easier by providing a best-in-class deployment management tool. This tool works with your DevOps process to create the deployment loop shown below:

![Octopus DevOps](devops-cycle.png "width=500")

## Manual and automated tests

Software tests can be manual or automatic. A person carries out a manual test. A person will click through an application and use it to find any bugs. Automated tests are scripted in advance and executed by a machine. Automated tests compare an expected result with the actual result. Both methods of testing have their place in a software application. A manual test is much slower and requires an environment for the tester. As developers must write automated tests in advance, the errors found in manual testing can inform and turn into automated tests to strengthen the test suite. Manual tests are suitable for cases where opinion and nuance play a role, such as UX or user experience. There is no pre-determined result for an automated test to check in these cases.

Automated tests are near-instant and execute in the hundreds or thousands at runtime. Automated tests check for functionality and ensure that every line of code and feature works as intended. In a DevOps process, automated tests enable continuous delivery. Automated tests will have a rating of test coverage. When developers add new features to a release, developers can run the tests to identify whether test coverage has decreased. Developers can pinpoint where tests fail to identify bugs for the new release. Automated tests complement a continuous delivery DevOps strategy. The more tests are automated, the faster an application can iterate and cycle through the DevOps loop of building, testing, and releasing.

## Functional and non-functional tests

There are many different kinds of tests you could perform on your application. One way to categorize types of tests is functional and non-functional testing.

Functional tests ask questions like:

- Does this button work?
- Does one module work with another module?
- Does the user journey work from the start of the experience to the end?

Functional tests test for functionality. Maintenance testing tests if the application has retained all functionality from version to version. It asks whether any functionality in the application has regressed between versions. I have included maintenance testing under functional tests as it has to do with functionality. Some sources list it as a third type.

Non-functional tests test the way a system operates rather than the functions of the system. Non-functional tests asks questions such as:

- How secure is the application?
- How much load can the application handle?
- Can the application scale if needed?

## Test types

The following are some types of functional and non-functional tests. I have provided some general points of the kind of test and an example.

### Functional

#### Unit

```

Testing an individual unit of code for functionality. In this case we test a function, which is a block of code that does one task.

1. Test if a function works. This function averages the weather for the previous 14 days.
2. Give the function the weather inputs for the previous 14 days
3. Check whether the expected output matches the actual output

```

#### Integration

```

Testing the functionality between two or more modules. This example tests the integration between the e-commerce store front, and the shopping cart module

1. Load the e-commerce store and add some items to the cart, and go to checkout
2. Check that the correct number of items are in the cart and the listed price is correct

```

#### Smoke

```

Smoke testing is preliminary testing to reveal failures that could result in a rejection of a release.

1. Does the web server return a 200 OK response?
2. Can I ping the database?
```

#### Acceptance

```

Confirm that the application is working according to a requirements specification. In this example there is a requirement for a rewards system to work with an application. The tests test for the expected behavior of a rewards system.

1. If a user tries to purchase a product with rewards points and they have enough points, the purchase price should be $0
2. If a user tries to purchase a product with rewards points and they do not have enough points, the purchase price should be $20

```

### Non-Functional

#### Load/Performance

```

Tests performance metrics like speed, response time, and resource usage of the application.

1. Measure the loading time of the home page and flag if it exceeds a threshold value
2. Measure database response time when handling 100 or more concurrent requests. Flag if response time exceeds threshold value
3. Under increasing load, test how many nodes an application needs to recover

```

#### Security

```

Tests for security related weaknesses in the system.

1. Scan log files for sensitive information. eg. credit card numbers or email addresses
2. Scanning for long running sessions as a sign that session handling isn't working as expected

```

#### Scalability

```

Tests for issues related to scaling the application.

1. Under increasing load, test how many nodes an application needs to recover
2. Test the length of time required for more nodes to be added and the application to recover. Chart how quickly you can expect 90% of scale up events to complete in.

```

## Conclusion

Testing is an essential factor in DevOps processes. Testing assures each stage in a DevOps process for quality before moving to the next stage. Tests can be run manually or automated. This blog has given examples of the two main types of tests: functional and non-functional. A robust testing environment fits well with Octopus Deploy. Octopus Deploy takes care of the release, deploy and automate sections of the DevOps lifecycle, and makes complex deployments easier!

Happy Deployments!
