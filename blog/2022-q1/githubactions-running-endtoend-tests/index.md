---
title: Running end to end tests in GitHub Actions
description: Learn how to run end to end tests in GitHub Actions and capture the results
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

GitHub Actions has native support for executing build steps inside Docker containers. This functionality, combined with the fact that all popular end-to-end testing tools provide supported Docker images, means running end-to-end tests in GitHub Actions is quite easy to configure.

In this post you'll learn how to run browser tests with Cypress and API tests with Postman as part of a GitHub Actions workflow.

## Prerequisites

GitHub Actions are a hosted service, so the only prerequisite is a GitHub account. All other dependencies like Software Development Kits (SDKs) are installed during the execution of the GitHub Actions workflow, or provided by the Docker images published by testing platforms.

## Running browser tests with Cypress

Cypress is a browser automation tool that allows you to interact with web pages in much the same way an end user would by clicking on buttons and links, filling in forms, scrolling the page etc. You can also verify the content of a page to ensure the correct results have been displayed.

The [Cypress documentation provides an example first test](https://docs.cypress.io/guides/getting-started/writing-your-first-test) which has been saved to the [junit-cypress-test GitHub repo](https://github.com/OctopusSamples/junit-cypress-test). The test is shown below:

```javascript
describe('My First Test', () => {
  it('Does not do much!', () => {
    expect(true).to.equal(true)
  })
})
```

This test is configured to generate a JUnit report file in the `cypress.json` file:

```json
{
  "reporter": "junit",
   "reporterOptions": {
      "mochaFile": "cypress/results/results.xml",
      "toConsole": true
   }
}
```

The workflow file below executes this test with the [Cypress GitHub Action](https://docs.cypress.io/guides/continuous-integration/github-actions#Cypress-GitHub-Action), saves the generated video file as an artifact, and processes the test results:

```yaml
name: Cypress

on:
  push:
  workflow_dispatch:

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v1

      - name: Cypress run
        uses: cypress-io/github-action@v2

      - name: Save video
        uses: actions/upload-artifact@v2
        with:
          name: sample_spec.js.mp4
          path: cypress/videos/sample_spec.js.mp4

      - name: Report
        uses: dorny/test-reporter@v1
        if: always()
        with:
          name: Cypress Tests
          path: cypress/results/results.xml
          reporter: java-junit
          fail-on-error: true
```

Note that test-reporter does have the ability to process Mocha JSON files, and Cypress uses Mocha for reporting, so an arguably more idiomatic solution would be to have Cypress generate Mocha JSON reports. Unfortunately there is a [bug in Cypress](https://github.com/cypress-io/cypress/issues/18014) that presents the JSON reporter from saving results as a file. Generating JUnit report files is a useful workaround until this issue is resolved:

![Cypress results](cypress-results.png "width=500")

The video file is captured as an artifact, which is listed in the **Summary** page:

![Artifacts](github-actions-artifacts.png "width=500")

## Running API tests with Newman