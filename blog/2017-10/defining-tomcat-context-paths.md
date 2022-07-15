---
title: Defining Tomcat context paths
description: Learn how Tomcat defines the context path of your web application.
author: matthew.casperson@octopus.com
visibility: public
published: 2017-10-29
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - DevOps
---

The context path of a web application defines the URL that end users will access the application from. A simple context path like `myapp` means the web app can be accessed from a URL like http://localhost:8080/myapp.  A nested context path like `myapp/v1` means the web app can be accessed from a URL like http://localhost:8080/myapp/v1.

Tomcat provides a number of ways to define the context path of a web app, although the configuration is not quite as straight forward as you might expect.

In this blog post, we’ll explore the options Tomcat provides for deploying web applications and defining their context paths.

## The `<Host>` configuration element

Many of the options available in Tomcat for deploying applications are defined in the `<Host>` element in the `config/server.xml` file.

The default `<Host>` element in Tomcat 9.01 looks like this:

```xml
<Host name="localhost"  appBase="webapps"
      unpackWARs="true" autoDeploy="true">
```

We’ll explore how these attributes affect deployments in Tomcat below.

## Exploded deployments vs WAR packages

There are two ways to deploy Java web applications.

The first way is to deploy a WAR file. A WAR file is just a ZIP archive with a directory structure that is recognised by Java application servers like Tomcat. WAR files are convenient because they are a single package that is easy to copy, and the contents of the WAR file are compressed making it quite a compact package.

The second way is to deploy all the individual files that make up a web application. This is called an exploded deployment, or an exploded WAR. This kind of deployment can be very useful during development, as files like HTML pages and CSS files can be edited while the application is deployed and reloaded on the fly.

By default, when you deploy a WAR file to Tomcat, it will be extracted into an exploded deployment for you. In the screenshot below, you can see that the end result of deploying a file called `demo.war` is a directory called `demo` with the context of the `demo.war` archive extracted into it:

![Tomcat Exploded Deployment](tomcat-exploded-deployment.png)

This behaviour can be disabled by setting the `unpackWARs` attribute on the `<Host>` element to `false`, which stops the WAR file from being unpacked as part of the deployment process.

## The `webapps` Directory

The `webapps` directory is where deployed applications reside in Tomcat.

The `webapps` directory is the default deployment location, but this can be configured with the `appBase` attribute on the `<Host>` element.

If Tomcat is set to autodeploy applications (and it is set to do this by default) then any WAR file or exploded deployment copied into the `webapps` folder will be deployed automatically while Tomcat is running.

The autodeployment of applications can be disabled by setting the `autoDeploy` attribute on the `<Host>` element to `false`. In this case, applications will de deployed on startup.

In turn, the deployment of applications on startup can be disabled by setting the `deployOnStartup` attribute on the `<Host>` element to `false`.

:::hint
If both `autoDeploy` and `deployOnStartup` are false, you can deploy applications by manually adding a `<Context>` element inside the `<Host>` element in the `conf/server.xml` file. See the section "Defining the context in the `server.xml` file" for an example.
:::

## Embedding the path in the (exploded) WAR filename

When an application is deployed from the `webapps` directory, it will be made available under a context path that matches the name of the WAR file or the name of the directory under `webapps` that the exploded deployment was copied to.

For example, if you deploy an WAR file called `demo.war`, it will be made available under the `demo` context. Likewise, if you deploy an exploded war to `webapps/demo`, it will also be made available under the context of `demo`.

Tomcat supports nested context paths. These are embedded in the WAR filename after a single hash character. For example, if you deploy a WAR file called `demo#v1.war`, it will be made available under the `demo/v1` context. Contexts can be multiple levels deep, so if you deploy a WAR file called `demo#v1#myfeature.war` it will be made available under the `demo/v1/myfeature` context.

The same pattern applies to the directories holding exploded deployments. For example, if you deploy an exploded war to `webapps/demo#v1`, it will be made available under the `demo/v1` context.

## Defining the context path from the `server.xml` file

It is possible to configure WAR files or exploded deployment directories by adding a `<Context>` element to the `<Host>` element in the `conf/server.xml` file. Here is an example:

```xml
<Host name="localhost"  appBase="webapps"
      unpackWARs="false" autoDeploy="false" deployOnStartup="false">
      <Context path="/mydemo/version1" docBase="demo#v1.war"/>
      ...
</Host>
```

The `docBase` attribute is a path to the WAR file or exploded deployment directory. It is relative to the `webapps` directory, although an absolute path can be used.

The `path` attribute is the one we are most interested in, as it defines the context path of the application. In this case we have exposed the web app under the `/mydemo/version1` context.

:::warning
The `path` attribute can only be defined if the WAR or exploded deployment directory is not under the `webapps` directory, or if the `autoDeploy` and `deployOnStartup` attributes on the `<Host>` element are `false`.

In this example we are referencing the file `webapps\demo#v1.war`, which means that the `autoDeploy` and `deployOnStartup` attributes on the `<Host>` element must be `false`.

To quote from the [documentation](https://tomcat.apache.org/tomcat-9.0-doc/config/context.html):
> If this rule is not followed, double deployment is likely to result.

:::

:::warning
Defining `<Context>` elements in the `server.xml` file is not considered best practice. This information should be defined in files saved under `conf/Catalina/localhost/`. See "The Confusing Case of the `context.xml` File" for more information.
:::

## The Confusing Case of the `context.xml` File

So far we have seen two ways of defining the context path:
1. From the name of the WAR file or the exploded deployment directory.
2. From the `path` attribute on the `<Context>` element in the `server.xml` file (with the caveat that the application being deployed is not located under the `webapps` directory, or if it is under the `webapps` directory, that the `autoDeploy` and `deployOnStartup` attributes on the `<Host>` element are `false`).

Tomcat also allows us to include a file in our web app called `META-INF/context.xml`, or to create the file `conf/Catalina/localhost/<context>.xml` under the Tomcat directory. These files contains the same `<Context>` element as the `<Host>` element in the `server.xml` file.

This naturally leads you to assume you can define the `path` attribute on the `<Context>` element in these XML files, and Tomcat will deploy the application to the defined context path.

However, this is not the case.

For example, let’s assume the following XML is saved as the `META-INF/context.xml` file inside a WAR file called `demo#v1.war`:

```xml
<Context path="/mydemo/version1"/>
```

When the `demo#v1.war` file is placed in the `webapps` folder and deployed by Tomcat, it will be made available under the `demo/v1` context. The `path` attribute is ignored.

Likewise, if that same XML context was saved to the `conf/Catalina/localhost/demo#v1.xml` file, the application would still be made available under the `demo/v1` context.

This is a little counter-intuitive, but it is clearly spelled out in the [documentation](https://tomcat.apache.org/tomcat-9.0-doc/config/context.html):

>[The path] attribute must only be used when statically defining a Context in server.xml. In all other circumstances, the path will be inferred from the filenames used for either the .xml context file or the docBase.

This means it is the name of the WAR file or exploded deployment directory, or the name of the XML file under `conf/Catalina/localhost`, that defines the context path.

Indeed when creating an XML file under the `conf/Catalina/localhost` directory for the purposes of defining the context of an application deployed from the `webapps` directory, the XML file needs to have the same name as the WAR file or exploded deployment directory.

For example, if you have a file called `webapps\demo#v1.war`, then the corresponding XML file must be called `conf/Catalina/localhost/demo#v1.xml`. These files need to have matching filenames, and the filename defines the context.

When configuring the context for a deployment outside of the `webapps` directory, the `docBase` attribute has to be defined. This attribute points to the WAR file or exploded deployment.

In this case, it is still the name of the XML file that defines the context. For example, if the XML below is saved to `conf/Catalina/localhost/application#version1.xml`, the application from `/apps/myapp#v1.war` will be made available under the context `application/version1`. The WAR filename is not used to generate the context in this case.

```xml
<Context docBase="/apps/myapp#v1.war"/>
```

## Uploading via the Manager Application

Finally, the context path of an application can be defined while uploading it through the manager REST API. This can be done with a `PUT` request to `http://localhost:8080/manager/text/deploy?path=/foo`, where the request data is the WAR file to be deployed, and the `path` query parameter is the desired context path.

:::hint
Requests to `/manager/html` require the credentials of a user from the `manager-gui` group. This URL is the one you access to view the Manager app via a web browser.

Requests to `/manager/text` require the credentials of a user from the `manager-script` group. This URL is considered to be the Manager API.

Just because you have a user that can access the Manager application via a browser does not necessarily mean that the user can interact with the API. In fact, it is considered bad practice for a single user to be part of the `manager-gui` and `manager-script` groups. To quote from the [documentation](https://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html):

>It is recommended to never grant the manager-script or manager-jmx roles to users that have the manager-gui role.

:::

This file upload will result in a deployment with context path embedded in the file name inside the `webapps` folder. So actually uploading a file via the Manager app is not a new way to define the context of an application, it is just a convenient way to ensure a correctly named web application is copied into the `webapps` directory.

## Conclusion

This table summaries the various context paths that will be assigned to web applications deployed from `webapps`, referenced in the `server.xml` file, or referenced from a file under `conf/Catalina/localhost/`.

| Configuration | Context |
|-|-|
| WAR file deployed under `webapps/app.war` | `app` |
| Exploded deployment under `webapps/app` | `app` |
| WAR file deployed under `webapps/app#v1.war` | `app/v1` |
| Exploded deployment under `webapps/app#v1` | `app/v1` |
| WAR file deployed under `webapps/app#v1#feature.war` | `app/v1/feature` |
| Exploded deployment under `webapps/app#v1#feature` | `app/v1/feature` |
| `<Context path="/mydemo/version1" docBase="/apps/demo#v1.war"/>` in `conf/server.xml` | `/mydemo/version1` |
| `<Context path="path/is/ignored" docBase="/apps/myapp#v1.war"/>` in `conf/Catalina/localhost/mydemo#version1.xml` (i.e. config for `/apps/myapp#v1.war`) | `/mydemo/version1` |
| `<Context path="/path/is/ignored"/>` in `conf/Catalina/localhost/mydemo#version1.xml` (i.e. config for `webapps/mydemo#version1.war`) | `/mydemo/version1` |

If you are interested in automating the deployment of your Java applications to Tomcat, [download a trial copy of Octopus Deploy](https://octopus.com/downloads), and take a look at [our documentation](https://octopus.com/docs/deployments/java/deploying-java-applications).

## Learn more

* Tutorial: [Deploying Spring Boot Applications as Windows Services](https://hubs.ly/H0gBK-l0)
* Getting Started 101: [Installing Tomcat for your next Java project](https://hubs.ly/H0gBK-H0)
* Documentation: [Java Applications](https://hubs.ly/H0gBK-R0)
* Video: [Deploying a Spring Boot web application with Octopus Deploy](https://hubs.ly/H0gBKKW0)