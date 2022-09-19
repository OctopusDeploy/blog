---
title: The different types of software tests
description: Learn why testing is important, read about manual and automated testing, and functional and non-functional testing with examples.
author: terence.wong@octopus.com
visibility: public
published: 2022-09-26-1400
metaImage: blogimage-runningunittestsingithubactions-2022.png
bannerImage: blogimage-runningunittestsingithubactions-2022.png
bannerImageAlt: Open laptop sits behind a screen in dark mode showing a table of rows with green ticks, red crosses, and orange exclamation marks to indicate test results.
isFeatured: false
tags:
  - DevOps
  - Testing
---

For software teams, testing makes sense, applications should be screened for bugs. But why is testing important for your business and how does it fit into DevOps?

Testing is part of Continuous Delivery that assures quality at each stage of the delivery pipeline before moving on to the next stage. DevOps is an iterative cycle of building, testing, and releasing software in short iterations. A comprehensive testing environment helps each iteration of the DevOps loop strengthen the quality of the product. A weak testing phase can mean defects progress to release, and developers need to fix bugs while the product is live. Development teams fall on both sides of the testing spectrum. 

A survey by [Mabel on the state of testing in DevOps](https://www.dropbox.com/s/nnagymzdcnoswc6/Benchmark-Report-State-of-Testing-in-DevOps.pdf?dl=0) indicates that automated testing (at least 4–5 different types of tests) is key to customer happiness. The [2021 State of DevOps DORA Report](https://www.dropbox.com/s/xycst8qsxnpsieu/state-of-devops-2021.pdf?dl=0) reveals that continuous testing is an indicator of success, with elite performers who meet their reliability targets being 3.7 times more likely to use continuous testing.

In this post, I discuss automated and manual tests, and 2 common types of tests: functional and non-functional. 

At Octopus Deploy, we help make complex deployments easier by providing a best-in-class deployment management tool that works with your DevOps process to create the deployment loop shown below:

![Octopus DevOps](devops-cycle.png "width=500")

## Manual and automated tests

Software tests can be manual or automatic. If you've used an app on your device and reported a bug, you've carried out a manual test. Automated tests are scripted in advance and executed by a machine, they compare an expected result with the actual result. Both methods of testing have their place in a software application, however manuals tests are slower and require an environment for the testers. 

As developers write automated tests in advance, the errors found in manual testing can inform automated tests to strengthen the test suite. Manual tests are suitable when opinion and nuance play a role, like user experience. There is no pre-determined result for an automated test to check in these cases.

Automated tests are near-instant and execute in the hundreds or thousands at runtime. Automated tests check for functionality and make sure every line of code and feature works as intended. In a DevOps process, automated tests enable Continuous Delivery by giving a test coverage of the application. If you want to set up your application with a test coverage, you install automated tests for every component of the application. 

When you add new features to a release, you can run the tests to identify whether test coverage has decreased. You can use the results to identify bugs for the new release. Automated tests complement a Continuous Delivery DevOps strategy. The more tests you automate, the faster your application can iterate and cycle through the DevOps loop of building, testing, and releasing.

## Functional and non-functional tests

There are many tests you can perform on your application. One way to categorize tests is functional and non-functional. 

Functional tests ask questions like:

- Does this button work?
- Does one module work with another module?
- Does the user journey work from the start of the experience to the end?

Maintenance testing checks if the application has retained all functionality from version to version. It asks whether any functionality in the application has regressed between versions. I include maintenance testing under functional tests because it relates to functionality, though some sources list it as a third type. 

Non-functional tests check the way a system operates rather than the functions of the system. Non-functional tests asks questions like:

- How secure is the application?
- How much load can the application handle?
- Can the application scale if needed?

### Examples of functional tests

#### Unit tests

Unit tests test an individual unit of code for functionality. In the example below, we test a function, which is a block of code that does one task.

```

1. Test if a function works. This function averages the weather for the previous 14 days.
2. Give the function the weather inputs for the previous 14 days.
3. Check whether the expected output matches the actual output.

```

#### Integration tests

Integration tests verify the functionality between 2 or more modules. This example tests the integration between the e-commerce storefront and the shopping cart module.

```

1. Load the e-commerce store and add some items to the cart, and go to checkout.
2. Check that the correct number of items are in the cart and the listed price is correct.

```

#### Smoke tests

Smoke testing is preliminary testing to reveal failures that can result in rejection of a release.

```

1. Does the web server return a 200 OK response?
2. Can I ping the database?
```

#### Acceptance tests

Acceptance testing confirms your application is working according to a requirements specification. In this example, there's a requirement for a rewards system to work with an application. The tests check for the expected behavior of a rewards system.

```

1. If a user tries to purchase a product with rewards points and they have enough points, the purchase price should be $0.
2. If a user tries to purchase a product with rewards points and they do not have enough points, the purchase price should be $20.

```

### Examples of non-functional tests

#### Load and performance tests

Load and performance tests check metrics like speed, response time, and resource usage of the application.

```

1. Measure the loading time of the home page and flag if it exceeds a threshold value.
2. Measure database response time when handling 100 or more concurrent requests. Flag if response time exceeds threshold value.
3. Under increasing load, test how many nodes an application needs to recover.

```

#### Security tests

Security tests check for security related weaknesses in the system.

```

1. Scan log files for sensitive information. eg. credit card numbers or email addresses.
2. Scanning for long running sessions as a sign that session handling isn't working as expected.

```

#### Scalability tests

Scalability tests test for issues related to scaling the application.

```

1. Under increasing load, test how many nodes an application needs to recover.
2. Test the length of time required for more nodes to be added and the application to recover. Chart how quickly you can expect 90% of scale up events to complete in.

```

## Conclusion

Testing is essential to DevOps processes, making sure your software meets quality requirements before moving to the next stage. Research has shown that automated testing is a strong indicator of customer happiness and successful teams. 

Tests can be run manually or be automated, and there are 2 main types of tests: functional and non-functional. 

A robust testing environment fits well with Octopus Deploy. Octopus takes care of the release and deployment, and automates sections of the DevOps lifecycle, making complex deployments easier.

To learn more about the importance of testing, read our post about [why you should track vulnerabilities after deployment](https://octopus.com/blog/track-vulnerabilities-after-deployment).

Happy deployments!