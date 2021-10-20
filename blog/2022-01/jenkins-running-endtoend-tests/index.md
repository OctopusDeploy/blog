---
title: Running end to end tests in Jenkins
description: Learn how to run end to end tests in Jenkins and capture the results
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

End-to-end (E2E) tests represent the of the final stages of automated testing. E2E are long running (certainly with respect to unit tests that can complete thousands of tests in seconds), and are typically executed by external tools which interact with the application under test through public interfaces like web pages or HTTP APIs.

In this post you'll learn how to run E2E tests with Cypress, to validate interactions with web pages, and Postman, to validate HTTP APIs.

## Prerequisites

To follow along with this post you'll need a Jenkins instance. The [Traditional Jenkins Installation](/blog/2022-01/jenkins-install-guide/index.md), [Docker Jenkins Installation](/blog/2022-01/jenkins-docker-install-guide/index.md), or [Helm Jenkins Installation](/blog/2022-01/jenkins-helm-install-guide/index.md) guides provide instructions to install Jenkins in your chosen environment.

Both Cypress and Newman (the Postman command line test runner) require Node.js to be installed. The [Node.js website](https://nodejs.org/en/download/) provides downloads, or offers [installation instructions for package managers](https://nodejs.org/en/download/package-manager/).

## Running browser tests with Cypress

```groovy
pipeline {
  // This pipeline requires the following plugins:
  // * Git: https://plugins.jenkins.io/git/
  // * Workflow Aggregator: https://plugins.jenkins.io/workflow-aggregator/
  // * JUnit: https://plugins.jenkins.io/junit/
  agent 'any'
  stages {
    stage('Checkout') {
      steps {
        script {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/OctopusSamples/junit-cypress-test.git']]])
        }
      }
    }
    stage('Dependencies') {
      steps {
        sh(script: 'npm install')        
      }
    }
    stage('Test') {
      steps {
        sh(script: 'NO_COLOR=1 node_modules/.bin/cypress run || true')          
      }
    }
  }
  post {
    always {
      junit(testResults: 'cypress/results/results.xml', allowEmptyResults : true)
      archiveArtifacts(artifacts: 'cypress/videos/sample_spec.js.mp4', fingerprint: true) 
    }
  }
}
```

## Running API tests with Newman

```groovy
pipeline {
  // This pipeline requires the following plugins:
  // * Git: https://plugins.jenkins.io/git/
  // * Workflow Aggregator: https://plugins.jenkins.io/workflow-aggregator/
  // * JUnit: https://plugins.jenkins.io/junit/
  agent 'any'
  stages {
    stage('Checkout') {
      steps {
        script {
            checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/OctopusSamples/junit-newman-test.git']]])
        }
      }
    }
    stage('Dependencies') {
      steps {
        sh(script: 'npm install')        
      }
    }
    stage('Test') {
      steps {
        sh(script: 'node_modules/.bin/newman run GitHubTree.json --reporter-junit-export results.xml --reporters cli,junit || true')          
      }
    }
  }
  post {
    always {
      junit(testResults: 'results.xml', allowEmptyResults : true)
    }
  }
}
```

## Conclusion