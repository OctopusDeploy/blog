---
title: Using the GitLab feeds with Octopus Deploy
description: Learn how to connect GitLab feed types to Octopus Deploy.
author: shawn.sesna@octopus.com
visibility: private 
published: 2022-12-06-1400
metaImage: blogimage-usinggitlabfeedswithoctopus-2022.png
bannerImage: blogimage-usinggitlabfeedswithoctopus-2022.png
bannerImageAlt: Packages on conveyer belt starting in an orange machine, traveling through a blue machine, coming out the other side with version numbers.
isFeatured: false
tags:
 - DevOps
---

Octopus Deploy contains a [built-in repository](https://octopus.com/docs/packaging-applications/package-repositories/built-in-repository) that supports a variety of [file types](https://octopus.com/docs/packaging-applications#supported-formats).  While this is a beneficial feature of Octopus Deploy, some customers prefer to use a third-party repository or even utilize the repositories built-in to their Continuious Integration (CI) tools.  In this post, I will demonstrate how to use the Registries built into GitLab as external feeds for Octopus Deploy.


## GitLab Registries
There are multiple registries types supported in GitLab
- Package Registry
- Container Registry
- Infrastructure Registry

For this post, we'll cover just the first two; Package and Container.

### Package Registry
The [GitLab Package Registery](https://gitlab.octopusdemos.app/help/user/packages/package_registry/index.md#use-gitlab-cicd-to-build-packages) supports a wide variety of package types, however, Octopus only supports the following GitLab package registry types as external feeds:
- Maven
- NuGet

#### Maven
Maven repositories are often associated with Java applications, though they can be used in a more generic fasion (more on that later).  The [PetClinic](https://bitbucket.org/octopussamples/petclinic/src/master/) example application contains most of the components necessary to demonstrate the Maven registry capabilities of GitLab.  The only thing missing is the mechanism to authenticate to the Maven registry.  [GitLab documentation](https://docs.gitlab.com/ee/user/packages/maven_repository/#create-maven-packages-with-gitlab-cicd-by-using-maven) shows this can easily be accomplished by adding an `ci_settings.xml` file with the following contents, 

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd">
  <servers>
    <server>
      <id>gitlab-maven</id>
      <configuration>
        <httpHeaders>
          <property>
            <name>Job-Token</name>
            <value>${env.CI_JOB_TOKEN}</value>
          </property>
        </httpHeaders>
      </configuration>
    </server>
  </servers>
</settings>
```

Once this has been added, you are ready to proceed with defining your build definition.  

GitLab uses a YAML based approach for configuring build definitions.  Below is an example build defintion for the PetClinic application

```yaml
stages:
    - set-version
    - java-build
    - maven-push

set-version:
  stage: set-version
  tags:
    - shell
  script:
    - dayOfYear=$(date +%j)
    - hour=$(date +%H)
    - minutes=$(date +%M)
    - seconds=$(date +%S)
    - year=$(date +%y)
    - echo "1.0.$year$dayOfYear.$hour$minutes$seconds" >> version.txt
  artifacts:
    paths:
      [ version.txt ]

java-build:
  stage: java-build
  tags:
    - shell
  script:
    - VERSION_NUMBER=$(cat version.txt)
    - mvn clean package -DskipTests=true -Dproject.versionNumber=$VERSION_NUMBER
  artifacts:
    paths:
      - "$CI_PROJECT_DIR/target/*.war"

maven-push:
  stage: maven-push
  tags:
    - shell
  variables:
    GROUP: "com.octopus"
    VERSION: $VERSION_NUMBER
    REPO_PROJECT_ID: $CI_PROJECT_ID
  script:
    - VERSION_NUMBER=$(cat version.txt)
    - mvn deploy:deploy-file -s ci_settings.xml -DgroupId=$GROUP 
      -Dversion=$VERSION_NUMBER
      -Dfile=$CI_PROJECT_DIR/target/petclinic.web.$VERSION_NUMBER.war
      -Durl=${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/maven
      -DrepositoryId=gitlab-maven
      -DpomFile=pom.xml
    - zip -r $CI_PROJECT_DIR/petclinic.mysql.flyway.$VERSION_NUMBER.zip $CI_PROJECT_DIR/flyway
    - mvn deploy:deploy-file -s ci_settings.xml
      -DgroupId=$GROUP
      -DartifactId=petclinic.mysql.flyway
      -Dpackaging=zip
      -Dfile=$CI_PROJECT_DIR/petclinic.mysql.flyway.$VERSION_NUMBER.zip
      -DrepositoryId=gitlab-maven
      -Durl=${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/maven
      -DartifactId=petclinic.mysql.flyway
      -Dversion=$VERSION_NUMBER
```
The PetClinic application consists of two major components; the Java web application and [Flyway](https://flywaydb.org/), which is used to perfrom the database updates.  To deploy both of these using Octopus Deploy, they must first be added to the Maven registry so Octopus can find and deploy them.  The `maven-push` section of the build definition is where the artifacts are uploaded to the Maven registry.  You'll note that there are two `mvn deploy:deploy-file` commands defined in the `script` section.  The first `deploy` command uploads the `.war` file that was compiled in the `java-build` stage to the registry.  Unlike the web component, the Flyway piece is already compiled, it just needs to be zipped then uploaded to the registry.  The `zip` command zips the contents of the `flyway` folder, then, using the directions from Matt Casperson's [blog post](https://octopus.com/blog/deploy-and-consume-zip-files-from-maven), uploads the `.zip` file to a Maven registry.

![](gitlab-maven-packages.png)

#### NuGet
For dotnet applications, the most commonly used registry type is NuGet.  The dotnet core example application, [OctoPetShop](https://github.com/OctopusSamples/OctoPetShop), works well for this demonstration.  Use the following for the build definition (this definition takes advantage of the Docker-in-Docker (DIND) feature of GitLab)
```yaml
image: ubuntu:latest

stages:
    - set-version
    - build-dotnet
    - package-dotnet
    - push-packages

set-version:
  stage: set-version
  script:
    - dayOfYear=$(date +%j)
    - hour=$(date +%H)
    - minutes=$(date +%M)
    - seconds=$(date +%S)
    - year=$(date +%y)
    - echo "1.0.$year$dayOfYear.$hour$minutes$seconds" >> version.txt
  artifacts:
    paths:
      [ version.txt ]
    
build-dotnet:
  stage: build-dotnet
  image: mcr.microsoft.com/dotnet/core/sdk:3.1
  script:
    - VERSION_NUMBER=$(cat version.txt)
    - dotnet build OctopusSamples.OctoPetShop.Database/OctopusSamples.OctoPetShop.Database.csproj --output "$CI_PROJECT_DIR/output/OctopusSamples.OctoPetShop.Database"
    - dotnet publish OctopusSamples.OctoPetShop.ProductService/OctopusSamples.OctoPetShop.ProductService.csproj --output "$CI_PROJECT_DIR/output/OctopusSamples.OctoPetShop.ProductService"
    - dotnet publish OctopusSamples.OctoPetShop.ShoppingCartService/OctopusSamples.OctoPetShop.ShoppingCartService.csproj --output "$CI_PROJECT_DIR/output/OctopusSamples.OctoPetShop.ShoppingCartService"
    - dotnet publish OctopusSamples.OctoPetShop.Web/OctopusSamples.OctoPetShop.Web.csproj --output "$CI_PROJECT_DIR/output/OctopusSamples.OctoPetShop.Web"

  artifacts:
    paths:
      - "$CI_PROJECT_DIR/output/"

package-dotnet:
  stage: package-dotnet
  image: octopuslabs/gitlab-octocli
  script: 
    - VERSION_NUMBER=$(cat version.txt)
    - octo pack --id=OctopusSamples.OctoPetShop.ProductService --version=$VERSION_NUMBER --basePath="$CI_PROJECT_DIR/output/OctopusSamples.OctoPetShop.ProductService" --outFolder="$CI_PROJECT_DIR/packages" --format="NuPkg"
    - octo pack --id=OctopusSamples.OctoPetShop.Database --version=$VERSION_NUMBER --basePath="$CI_PROJECT_DIR/output/OctopusSamples.OctoPetShop.Database" --outFolder="$CI_PROJECT_DIR/packages" --format="NuPkg"
    - octo pack --id=OctopusSamples.OctoPetShop.ShoppingCartService --version=$VERSION_NUMBER --basePath="$CI_PROJECT_DIR/output/OctopusSamples.OctoPetShop.ShoppingCartService" --outFolder="$CI_PROJECT_DIR/packages" --format="NuPkg"
    - octo pack --id=OctopusSamples.OctoPetShop.Web --version=$VERSION_NUMBER --basePath="$CI_PROJECT_DIR/output/OctopusSamples.OctoPetShop.Web" --outFolder="$CI_PROJECT_DIR/packages" --format="NuPkg"
  artifacts:
    paths:
      - "$CI_PROJECT_DIR/packages/"
    

push-packages:
  stage: push-packages
  image: mcr.microsoft.com/dotnet/core/sdk:3.1
  script:
    - dotnet nuget add source "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/nuget/index.json" --name gitlab --username gitlab-ci-token --password $CI_JOB_TOKEN --store-password-in-clear-text
    - dotnet nuget push "$CI_PROJECT_DIR/packages/*.nupkg" --source gitlab
```
The `IsPackable` property of a Web Application set to false by default.  Rather than mess with project properties, the Octopus Deploy GitLab CLI image can be used to pack the compiled applications into NuGet packages.  Once the `.nupkg` files have been created, `dotnet` commands can be used to push the packages into the GitLab NuGet registry.

![](gitlab-nuget-packages.png)

### Container registry
Along with Package registries, GitLab also contains a container registry.  To demonstrate container registry usage, we'll again use the OctoPetShop application,

```yaml
image: ubuntu:latest

stages:
    - set-version
    - build-docker

set-version:
  stage: set-version
  script:
    - dayOfYear=$(date +%j)
    - hour=$(date +%H)
    - minutes=$(date +%M)
    - seconds=$(date +%S)
    - year=$(date +%y)
    - echo "1.0.$year$dayOfYear.$hour$minutes$seconds" >> version.txt
  artifacts:
    paths:
      [ version.txt ]

build-docker:
    stage: build-docker
    image: docker:1.11
    services:
      - docker:dind
    before_script:
      - docker info
    script:
      - VERSION_NUMBER=$(cat version.txt)
      - docker build -t $CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME/octopetshop.database:$VERSION_NUMBER ./OctopusSamples.OctoPetShop.Database/ 
      - docker build -t $CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME/octopetshop.productservice:$VERSION_NUMBER ./OctopusSamples.OctoPetShop.ProductService/ 
      - docker build -t $CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME/octopetshop.shoppingcartservice:$VERSION_NUMBER ./OctopusSamples.OctoPetShop.ShoppingCartService/ 
      - docker build -t $CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME/octopetshop.web:$VERSION_NUMBER ./OctopusSamples.OctoPetShop.Web/ 
      - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
      - docker push $CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME/octopetshop.database:$VERSION_NUMBER
      - docker push $CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME/octopetshop.productservice:$VERSION_NUMBER
      - docker push $CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME/octopetshop.shoppingcartservice:$VERSION_NUMBER
      - docker push $CI_REGISTRY/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME/octopetshop.web:$VERSION_NUMBER
```
![](gitlab-container-packages.png)

## Connecting GitLab registries as external feeds
Aside from the built-in repository, Octopus Deploy supports the use of [external feeds](https://octopus.com/docs/packaging-applications/package-repositories) to pull packages for deployment.  To add an external feed, click on Library -> External Feeds -> **ADD FEED**

![](octopus-add-external-feed.png)

### Adding a GitLab Maven feed
To add a GitLab Maven registry, you will need either the Project or Group Id the feed is associated with, this post is demonstrates the project level.  To get the Project Id, navigate to the project, the Id is displayed on the initial screen

![](gitlab-project-id.png)

Fill in the form fields:
- Feed Type: Maven Feed
- Name: Give the feed a name
- URL: https://[gitlabserver]/api/v4/projects/[project or group id]/packages/maven
- (Optional) Credentials: Username and password to use to access the feed

![](octopus-add-maven-feed.png)

Click on **SAVE AND TEST** to ensure the feed is functional.  Enter the name of the package using the GROUP:ARTIFACT format

![](octopus-test-maven-feed.png)

### Adding a GitLab NuGet feed
Similar to the Maven feed, you will need the Project or Group Id that the feed is associated with.

Fill in the form fields:
- Feed Type: NuGet Feed
- Name: Give the feed a name
- URL: https://[gitlabserver]/api/v4/projects/[project or group id]/packages/nuget/index.json
- (Optional) Credentials: Username and password to access the feed

![](octopus-add-nuget-feed.png)

Click on **SAVE AND TEST** to ensure the feed is functional.  Enter the name of the package (can be partial)

![](octopus-test-nuget-feed.png)

### Adding a GitLab Container feed
Unlike the other two feed types, the container feed doesn't need the Project or Group Id.  It does, however, need to know which port on the server is configured to host the container registry.

Fill in the form fields:
- Feed Type: Docker Container Registry
- Name: Give the feed a name
- URL: https://[gitlabserver]:[port]
- Credentials: Username and password to access the feed

![](octopus-add-container-feed.png)

Click on **SAVE AND TEST** to ensure the feed is functional.  Enter the name of the package (can be partial)

![](octopus-test-container-feed.png)

## Conclusion
Use of the built-in repository is useful for many scenarios, however, don't work well if you need something cross-space and don't want to duplicate.  This post demonstrated how to utilize the built-in registries of GitLab as External Feeds to Octopus Deploy.  Happy Deployments!