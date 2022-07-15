---
title: Arquillian for Infrastructure Testing
description: Arquillian's ability to spin up real application servers and integrate them with unit tests makes it a powerful solution for infrastructure testing.
author: matthew.casperson@octopus.com
visibility: public
published: 2017-11-28
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - DevOps
---

In a [previous blog post](/blog/2017-11/arquillian-testing.md) we had a look at how Arquillian solves the problem of testing real objects as they exist in a real application server. While this is perhaps the more traditional way Arquillian is used, it is not the only way.

One of the challenges writing a tool like [Octopus](https://octopus.com) is that it has to support a huge range of Java application servers. Right now we support:

* WildFly 10
* WildFly 11
* Red Hat JBoss EAP 6
* Red Hat JBoss EAP 7
* Tomcat 7
* Tomcat 8
* Tomcat 9

And not only that, but we support WildFly and JBoss EAP in standalone or domain mode. This gives us over 10 different application server configurations to test our code against.

:::hint
Octopus also supports [deploying Java artifacts](https://octopus.com/docs/deployments/java/deploying-java-applications) like JAR and WAR files directly to the filesystem, which means application servers like WebSphere, WebLogic, GlassFish, Payara etc can also be integrated into an Octopus deployment process.
:::

We could have created virtual machines with the various application servers and run tests against those, but Arquillian's ability to spin up real application servers and integrate those servers into unit tests means that we can test our code with standard unit test libraries like JUnit and run those tests directly from Maven.

In this blog post we'll take a look at how to use Arquillian to do infrastructure testing.

## The Maven POM File

A lot of the work required to configure Arquillian goes into the Maven POM file. Arquillian already has comprehensive [getting started documentation](http://arquillian.org/guides/getting_started/), so I'll touch on the highlights here.

We'll be using Java 8 for the tests, and so we set the `maven.compiler.source` and `maven.compiler.target` properties accordingly.

```xml
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

We'll use the Arquillian Bill of Materials (BOM) to provide the versions for the Arquillian dependencies we'll define later.

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.jboss.arquillian</groupId>
            <artifactId>arquillian-bom</artifactId>
            <version>1.1.14.Final</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>
```

We then need the Arquillian, JUnit and Apache HTTP Client dependencies.

 Note that we are using [Arquillian Chameleon](https://github.com/arquillian/arquillian-container-chameleon), which provides an easy way to manage the various containers (i.e application servers) that we'll be testing with.

 The Apache HTTP Client will be used to perform a simple test to ensure that the application servers are running.

```xml
<dependencies>
   <dependency>
       <groupId>org.apache.httpcomponents</groupId>
       <artifactId>httpclient</artifactId>
       <version>4.5.3</version>
   </dependency>
   <dependency>
       <groupId>org.jboss.arquillian.junit</groupId>
       <artifactId>arquillian-junit-container</artifactId>
       <scope>test</scope>
   </dependency>
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.12</version>
       <scope>test</scope>
   </dependency>
   <dependency>
       <groupId>org.arquillian.container</groupId>
       <artifactId>arquillian-container-chameleon</artifactId>
       <version>1.0.0.Final-SNAPSHOT</version>
       <scope>test</scope>
   </dependency>
</dependencies>
```

We'll define two Maven profiles: `Tomcat8` and `WildFly9`. These profiles define the `arquillian.launch` system property, which is used to select the application server that Chameleon will download for us, and set the path to the tests that will be run. In the `WildFly9` profile, we will be running the `wildfly9` container, and running tests in the `src/test/java/wildfly9` directory.

```xml
<profile>
    <id>WildFly9</id>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <systemPropertyVariables>
                        <arquillian.launch>wildfly9</arquillian.launch>
                    </systemPropertyVariables>
                    <includes>
                        <include>**/wildfly9/**</include>
                    </includes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

The `Tomcat8` profile will run the `tomcat8` container, and run tests from the `src/test/java/tomcat8` directory.

```xml
<profile>
  <id>Tomcat8</id>
  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-surefire-plugin</artifactId>
              <configuration>
                  <systemPropertyVariables>
                      <arquillian.launch>tomcat8</arquillian.launch>
                  </systemPropertyVariables>
                  <includes>
                      <include>**/tomcat8/**</include>
                  </includes>
              </configuration>
          </plugin>
      </plugins>
  </build>
</profile>
```

We use profiles because only one application server can be run at a time. Profiles allow us to select the kind of server to test with Maven command line parameters. We also split up the test class files to ensure that only tests for a given server are run with the selected profile.

Here is the complete POM file.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.octopus</groupId>
    <artifactId>arquillian-infrastructure-testing</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.jboss.arquillian</groupId>
                <artifactId>arquillian-bom</artifactId>
                <version>1.1.14.Final</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.3</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian.junit</groupId>
            <artifactId>arquillian-junit-container</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.arquillian.container</groupId>
            <artifactId>arquillian-container-chameleon</artifactId>
            <version>1.0.0.Final-SNAPSHOT</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <profiles>
        <profile>
            <id>WildFly9</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <configuration>
                            <systemPropertyVariables>
                                <arquillian.launch>wildfly9</arquillian.launch>
                            </systemPropertyVariables>
                            <includes>
                                <include>**/wildfly9/**</include>
                            </includes>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <profile>
            <id>Tomcat8</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <configuration>
                            <systemPropertyVariables>
                                <arquillian.launch>tomcat8</arquillian.launch>
                            </systemPropertyVariables>
                            <includes>
                                <include>**/tomcat8/**</include>
                            </includes>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
</project>
```

## Configuring Arquillian

Arquillian needs a file called `arquillian.xml` which defines the containers (or application servers) that it can run. In our case we need to define two containers to match the values assigned to the `arquillian.launch` system properties by the Maven profiles.

### Tomcat 8 Container

The `tomcat8` container downloads Tomcat 8.0.47 from the [binary distribution published to Maven](http://central.maven.org/maven2/org/apache/tomcat/tomcat/8.0.47/).

We also need to define the username and password that Arquillian will use to access the manager application provided with Tomcat. In this case we use `arquillian` for both.

To actually define the user `arquillian`, we configure Tomcat with a custom configuration file called `tomcat8-server.xml`.

```xml
<container qualifier="tomcat8">
    <configuration>
        <property name="target">tomcat:8.0.47:managed</property>
        <!-- relative to CATALINA_BASE/conf; catalinaBase is set by chameleon itself -->
        <property name="serverConfig">../../../../../src/test/resources/tomcat8-server.xml</property>
        <property name="user">arquillian</property>
        <property name="pass">arquillian</property>
    </configuration>
</container>
```

Inside the `tomcat8-server.xml` file we reference the `tomcat-users.xml` file.

```xml
<!-- Global JNDI resources
     Documentation at /docs/jndi-resources-howto.html
-->
<GlobalNamingResources>
    <!-- Editable user database that can also be used by
         UserDatabaseRealm to authenticate users
    -->
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="${catalina.home}/../../../../src/test/resources/tomcat-users.xml"/>
</GlobalNamingResources>
```

And finally in the `tomcat-users.xml` file, we define the user `arquillian`.

```xml
<?xml version='1.0' encoding='utf-8'?>
<tomcat-users>
    <user username="arquillian" password="arquillian" roles="manager-script"/>
</tomcat-users>
```

:::hint
CATALINE_BASE and CATALINA_HOME (referenced by `${catalina.home}` in the `tomcat-server.xml` file and the relative location of the `serverConfig` setting in the `arquillian.xml` file) is `target/server/tomcat_8.0.47/apache-tomcat-8.0.47`, which is the location where Arquillian Chameleon downloads Tomcat to. The long string of parent directory references go from this directory back to the test `resources` directory.
:::

### WildFly 9 Container

The WildFly container is less complicated. It too downloads WildFly from the [binary distribution published to Maven](http://central.maven.org/maven2/org/wildfly/wildfly-dist/9.0.0.Final/). The only other setting is the name of the server configuration file, which we set to `standalone.xml`.

```xml
<container qualifier="wildfly9">
    <configuration>
        <property name="target">wildfly:9.0.0.Final:managed</property>
        <property name="serverConfig">standalone.xml</property>
    </configuration>
</container>
```

### Complete Arquillian Configuration File

Here is the compete `arquillian.xml` file.

```xml
<arquillian xmlns="http://jboss.org/schema/arquillian" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="
        http://jboss.org/schema/arquillian
        http://jboss.org/schema/arquillian/arquillian_1_0.xsd">

    <container qualifier="tomcat8">
        <configuration>
            <property name="target">tomcat:8.0.47:managed</property>
            <!-- relative to CATALINA_BASE/conf; catalinaBase is set by chameleon itself -->
            <property name="serverConfig">../../../../../src/test/resources/tomcat8-server.xml</property>
            <property name="user">arquillian</property>
            <property name="pass">arquillian</property>
        </configuration>
    </container>

    <container qualifier="wildfly9">
        <configuration>
            <property name="target">wildfly:9.0.0.Final:managed</property>
            <property name="serverConfig">standalone.xml</property>
        </configuration>
    </container>

</arquillian>
```

## Configuring Arquillian Chameleon

Chameleon doesn't normally require much configuration, although with the version we are using here (1.0.0.Final-SNAPSHOT) we do have to provide a custom `containers.yaml` file that configures Tomcat 8. This version was based on the version from the [Chameleon GitHub repo](https://github.com/arquillian/arquillian-container-chameleon/blob/58f36a281721d8ce8776fa60275d322e7c3b9b24/arquillian-chameleon-container-model/src/main/resources/chameleon/default/containers.yaml).

Chameleon provides [documentation](https://github.com/arquillian/arquillian-container-chameleon#development) on the `containers.yaml` file, so I won't go into much detail here.

```yaml
- name: JBoss EAP
  versionExpression: 7.*
  adapters:
    - type: remote
      gav: org.wildfly.arquillian:wildfly-arquillian-container-remote:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.remote.RemoteDeployableContainer
    - type: managed
      gav: org.wildfly.arquillian:wildfly-arquillian-container-managed:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.managed.ManagedDeployableContainer
      configuration: &EAP7_CONFIG
        jbossHome: ${dist}
    - type: embedded
      gav: org.wildfly.arquillian:wildfly-arquillian-container-embedded:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.embedded.EmbeddedDeployableContainer
      configuration: *EAP7_CONFIG
  dist: &EAP7_DIST
    gav: org.jboss.as:jboss-as-dist:zip:${version}
  exclude: &EAP7_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
- name: JBoss EAP Domain
  versionExpression: 7.*
  adapters:
    - type: managed
      gav: org.wildfly.arquillian:wildfly-arquillian-container-domain-managed:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.domain.managed.ManagedDomainDeployableContainer
      configuration: *EAP7_CONFIG
    - type: remote
      gav: org.wildfly.arquillian:wildfly-arquillian-container-domain-remote:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.domain.remote.RemoteDomainDeployableContainer
  dist: &EAP7_DIST
    gav: org.jboss.as:jboss-as-dist:zip:${version}
  exclude: &EAP7_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
- name: JBoss EAP
  versionExpression: 6.0.*
  adapters:
    - type: remote
      gav: org.jboss.as:jboss-as-arquillian-container-remote:7.1.2.Final
      adapterClass: org.jboss.as.arquillian.container.remote.RemoteDeployableContainer
    - type: managed
      gav: org.jboss.as:jboss-as-arquillian-container-managed:7.1.2.Final
      adapterClass: org.jboss.as.arquillian.container.managed.ManagedDeployableContainer
      configuration: &EAP_CONFIG
        jbossHome: ${dist}
    - type: embedded
      gav: org.jboss.as:jboss-as-arquillian-container-embedded:7.1.2.Final
      adapterClass: org.jboss.as.arquillian.container.embedded.EmbeddedDeployableContainer
      configuration: *EAP_CONFIG
  dist: &EAP_DIST
    gav: org.jboss.as:jboss-as-dist:zip:${version}
  exclude: &EAP_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
    - "*:wildfly-arquillian-protocol-jmx"
    - "*:jboss-as-arquillian-testenricher-msc"
    - "*:jboss-as-arquillian-protocol-jmx"
- name: JBoss EAP Domain
  versionExpression: 6.0.*
  adapters:
    - type: managed
      gav: org.jboss.as:jboss-as-arquillian-container-domain-managed:7.1.2.Final
      adapterClass: org.jboss.as.arquillian.container.domain.managed.ManagedDomainDeployableContainer
      configuration: *EAP_CONFIG
    - type: remote
      gav: org.jboss.as:jboss-as-arquillian-container-domain-remote:7.1.2.Final
      adapterClass: org.jboss.as.arquillian.container.domain.remote.RemoteDomainDeployableContainer
  dist: &EAP_DIST
    gav: org.jboss.as:jboss-as-dist:zip:${version}
  exclude: &EAP_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
    - "*:wildfly-arquillian-protocol-jmx"
    - "*:jboss-as-arquillian-testenricher-msc"
    - "*:jboss-as-arquillian-protocol-jmx"
- name: JBoss EAP
  versionExpression: 6.*
  adapters:
    - type: remote
      gav: org.jboss.as:jboss-as-arquillian-container-remote:7.1.3.Final
      adapterClass: org.jboss.as.arquillian.container.remote.RemoteDeployableContainer
    - type: managed
      gav: org.jboss.as:jboss-as-arquillian-container-managed:7.1.3.Final
      adapterClass: org.jboss.as.arquillian.container.managed.ManagedDeployableContainer
      configuration: *EAP_CONFIG
    - type: embedded
      gav: org.jboss.as:jboss-as-arquillian-container-embedded:7.1.3.Final
      adapterClass: org.jboss.as.arquillian.container.embedded.EmbeddedDeployableContainer
      configuration: *EAP_CONFIG
  dist: *EAP_DIST
  exclude: *EAP_EXCLUDE
- name: JBoss EAP Domain
  versionExpression: 6.*
  adapters:
    - type: managed
      gav: org.jboss.as:jboss-as-arquillian-container-domain-managed:7.1.3.Final
      adapterClass: org.jboss.as.arquillian.container.domain.managed.ManagedDomainDeployableContainer
      configuration: *EAP_CONFIG
    - type: remote
      gav: org.jboss.as:jboss-as-arquillian-container-domain-remote:7.1.3.Final
      adapterClass: org.jboss.as.arquillian.container.domain.remote.RemoteDomainDeployableContainer
  dist: *EAP_DIST
  exclude: *EAP_EXCLUDE
- name: JBoss AS
  versionExpression: 7\.0\.[0-2]\.(.*)$|7\.1\.[0-1]\.(.*)$
  adapters:
    - type: remote
      gav: org.jboss.as:jboss-as-arquillian-container-remote:${version}
      adapterClass: org.jboss.as.arquillian.container.remote.RemoteDeployableContainer
    - type: managed
      gav: org.jboss.as:jboss-as-arquillian-container-managed:${version}
      adapterClass: org.jboss.as.arquillian.container.managed.ManagedDeployableContainer
      configuration: &AS_CONFIG
        jbossHome: ${dist}
    - type: embedded
      gav: org.jboss.as:jboss-as-arquillian-container-embedded:${version}
      adapterClass: org.jboss.as.arquillian.container.embedded.EmbeddedDeployableContainer
      configuration: *AS_CONFIG
  dist: &AS_DIST
    gav: org.jboss.as:jboss-as-dist:zip:${version}
  exclude: &AS_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
    - "*:wildfly-arquillian-protocol-jmx"
    - "*:jboss-as-arquillian-testenricher-msc"
    - "*:jboss-as-arquillian-protocol-jmx"
- name: JBoss AS Domain
  versionExpression: 7\.0\.[0-2]\.(.*)$|7\.1\.[0-1]\.(.*)$
  adapters:
    - type: managed
      gav: org.jboss.as:jboss-as-arquillian-container-domain-managed:${version}
      adapterClass: org.jboss.as.arquillian.container.domain.managed.ManagedDomainDeployableContainer
      configuration: *AS_CONFIG
    - type: remote
      gav: org.jboss.as:jboss-as-arquillian-container-domain-remote:${version}
      adapterClass: org.jboss.as.arquillian.container.domain.remote.RemoteDomainDeployableContainer
  dist: &AS_DIST
    gav: org.jboss.as:jboss-as-dist:zip:${version}
  exclude: &AS_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
    - "*:wildfly-arquillian-protocol-jmx"
    - "*:jboss-as-arquillian-testenricher-msc"
    - "*:jboss-as-arquillian-protocol-jmx"
- name: WildFly
  versionExpression: 8.*
  adapters:
    - type: remote
      gav: org.wildfly:wildfly-arquillian-container-remote:${version}
      adapterClass: org.jboss.as.arquillian.container.remote.RemoteDeployableContainer
    - type: managed
      gav: org.wildfly:wildfly-arquillian-container-managed:${version}
      adapterClass: org.jboss.as.arquillian.container.managed.ManagedDeployableContainer
      configuration: &WF_CONFIG
        jbossHome: ${dist}
    - type: embedded
      gav: org.wildfly:wildfly-arquillian-container-embedded:${version}
      adapterClass: org.jboss.as.arquillian.container.embedded.EmbeddedDeployableContainer
      configuration: &WF_EMBEDD_CONFIG
        jbossHome: ${dist}
        modulePath: ${dist}/modules
      dependencies:
        - org.jboss.remotingjmx:remoting-jmx:2.0.1.Final
        - org.jboss.logging:jboss-logging:3.2.1.Final
  dist: &WF_DIST
    gav: org.wildfly:wildfly-dist:zip:${version}
  exclude: &WF_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
    - "*:wildfly-arquillian-protocol-jmx"
    - "*:jboss-as-arquillian-testenricher-msc"
    - "*:jboss-as-arquillian-protocol-jmx"
- name: WildFly Domain
  versionExpression: 8.*
  adapters:
    - type: managed
      gav: org.wildfly.arquillian:wildfly-arquillian-container-domain-managed:${version}
      adapterClass: org.jboss.as.arquillian.container.domain.managed.ManagedDomainDeployableContainer
      configuration: *WF_CONFIG
    - type: remote
      gav: org.wildfly.arquillian:wildfly-arquillian-container-domain-remote:${version}
      adapterClass: org.jboss.as.arquillian.container.domain.remote.RemoteDomainDeployableContainer
  dist: &WF_DIST
    gav: org.wildfly:wildfly-dist:zip:${version}
  exclude: &WF_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
    - "*:wildfly-arquillian-protocol-jmx"
    - "*:jboss-as-arquillian-testenricher-msc"
    - "*:jboss-as-arquillian-protocol-jmx"
- name: WildFly
  versionExpression: 9.*
  adapters:
    - type: remote
      gav: org.wildfly.arquillian:wildfly-arquillian-container-remote:1.1.0.Final
      adapterClass: org.jboss.as.arquillian.container.remote.RemoteDeployableContainer
    - type: managed
      gav: org.wildfly.arquillian:wildfly-arquillian-container-managed:1.1.0.Final
      adapterClass: org.jboss.as.arquillian.container.managed.ManagedDeployableContainer
      configuration: *WF_CONFIG
    - type: embedded
      gav: org.wildfly.arquillian:wildfly-arquillian-container-embedded:1.1.0.Final
      adapterClass: org.jboss.as.arquillian.container.embedded.EmbeddedDeployableContainer
      configuration: *WF_EMBEDD_CONFIG
      dependencies:
        - org.jboss.remotingjmx:remoting-jmx:2.0.1.Final
  dist: *WF_DIST
  exclude: &WF9_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
- name: WildFly Domain
  versionExpression: 9.*
  adapters:
    - type: managed
      gav: org.wildfly.arquillian:wildfly-arquillian-container-domain-managed:1.1.0.Final
      adapterClass: org.jboss.as.arquillian.container.domain.managed.ManagedDomainDeployableContainer
      configuration: *WF_CONFIG
    - type: remote
      gav: org.wildfly.arquillian:wildfly-arquillian-container-domain-remote:1.1.0.Final
      adapterClass: org.jboss.as.arquillian.container.domain.remote.RemoteDomainDeployableContainer
  dist: *WF_DIST
  exclude: &WF9_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
- name: WildFly
  versionExpression: 10.*
  adapters:
    - type: remote
      gav: org.wildfly.arquillian:wildfly-arquillian-container-remote:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.remote.RemoteDeployableContainer
    - type: managed
      gav: org.wildfly.arquillian:wildfly-arquillian-container-managed:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.managed.ManagedDeployableContainer
      configuration: *WF_CONFIG
    - type: embedded
      gav: org.wildfly.arquillian:wildfly-arquillian-container-embedded:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.embedded.EmbeddedDeployableContainer
      configuration: *WF_EMBEDD_CONFIG
  dist: *WF_DIST
  exclude: &WF10_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
- name: WildFly Domain
  versionExpression: 10.*
  adapters:
    - type: managed
      gav: org.wildfly.arquillian:wildfly-arquillian-container-domain-managed:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.domain.managed.ManagedDomainDeployableContainer
      configuration: *WF_CONFIG
    - type: remote
      gav: org.wildfly.arquillian:wildfly-arquillian-container-domain-remote:2.0.1.Final
      adapterClass: org.jboss.as.arquillian.container.domain.remote.RemoteDomainDeployableContainer
  dist: *WF_DIST
  exclude: &WF10_EXCLUDE
    - org.jboss.arquillian.test:*
    - org.jboss.arquillian.testenricher:*
    - org.jboss.arquillian.container:*
    - org.jboss.arquillian.core:*
    - org.jboss.arquillian.config:*
    - org.jboss.arquillian.protocol:*
    - org.jboss.shrinkwrap.api:*
    - org.jboss.shrinkwrap:*
    - org.jboss.shrinkwrap.descriptors:*
    - org.jboss.shrinkwrap.resolver:*
    - "*:wildfly-arquillian-testenricher-msc"
# Older versions of Glassfish (before 3.1.2) are no longer supported
- name: GlassFish
  versionExpression: ^3\.1\.[2-9]{1}(\.[0-9])*$
  adapters:
    - &GF_REMOTE
          type: remote
          gav: org.jboss.arquillian.container:arquillian-glassfish-remote-3.1:1.0.1
          adapterClass: org.jboss.arquillian.container.glassfish.remote_3_1.GlassFishRestDeployableContainer
    - &GF_MANAGED
      type: managed
      gav: org.jboss.arquillian.container:arquillian-glassfish-managed-3.1:1.0.1
      adapterClass: org.jboss.arquillian.container.glassfish.managed_3_1.GlassFishManagedDeployableContainer
      configuration:
        glassFishHome: ${dist}
        outputToConsole: true
    - &GF_EMBEDDED
      type: embedded
      gav: org.jboss.arquillian.container:arquillian-glassfish-embedded-3.1:1.0.1
      adapterClass: org.jboss.arquillian.container.glassfish.embedded_3_1.GlassFishContainer
      requireDist: false
      dependencies:
        - org.glassfish.main.extras:glassfish-embedded-all:${version}
  dist: &GF_DIST
    gav: org.glassfish.main.distributions:glassfish:zip:${version}

- name: GlassFish
  versionExpression: 4.*
  adapters:
    - *GF_REMOTE
    - *GF_MANAGED
    - *GF_EMBEDDED
  dist: *GF_DIST

- name: Payara
  versionExpression: 4.*
  adapters:
    - *GF_REMOTE
    - *GF_MANAGED
    - type: embedded
      gav: org.jboss.arquillian.container:arquillian-glassfish-embedded-3.1:1.0.1
      adapterClass: org.jboss.arquillian.container.glassfish.embedded_3_1.GlassFishContainer
      requireDist: false
      dependencies:
        - fish.payara.extras:payara-embedded-all:${version}
  dist:
    gav: fish.payara.distributions:payara:zip:${version}

- name: Tomcat
  versionExpression: 6.*
  adapters:
    - type: remote
      gav: org.jboss.arquillian.container:arquillian-tomcat-remote-6:1.0.0.CR9
      adapterClass: org.jboss.arquillian.container.tomcat.remote.Tomcat6RemoteContainer
    - type: managed
      gav: org.jboss.arquillian.container:arquillian-tomcat-managed-6:1.0.0.CR9
      adapterClass: org.jboss.arquillian.container.tomcat.managed.Tomcat6ManagedContainer
      configuration: &TOMCAT_MANAGED_CONFIG
        catalinaHome: ${dist}
        catalinaBase: ${dist}
  dist:
    gav: http://archive.apache.org/dist/tomcat/tomcat-6/v${version}/bin/apache-tomcat-${version}.zip

- name: Tomcat
  versionExpression: 7.*
  adapters:
    - type: remote
      gav: org.jboss.arquillian.container:arquillian-tomcat-remote-7:1.0.0.CR9
      adapterClass: org.jboss.arquillian.container.tomcat.remote.Tomcat7RemoteContainer
    - type: managed
      gav: org.jboss.arquillian.container:arquillian-tomcat-managed-7:1.0.0.CR9
      adapterClass: org.jboss.arquillian.container.tomcat.managed.Tomcat7ManagedContainer
      configuration: *TOMCAT_MANAGED_CONFIG
  dist: &TOMCAT_DIST
    gav: org.apache.tomcat:tomcat:zip:${version}

- name: Tomcat
  versionExpression: 8.0.*
  adapters:
    - type: remote
      gav: org.jboss.arquillian.container:arquillian-tomcat-remote-8:1.0.0.CR9
      adapterClass: org.jboss.arquillian.container.tomcat.remote.Tomcat8RemoteContainer
    - type: managed
      gav: org.jboss.arquillian.container:arquillian-tomcat-managed-8:1.0.0.CR9
      adapterClass: org.jboss.arquillian.container.tomcat.managed.Tomcat8ManagedContainer
      configuration: *TOMCAT_MANAGED_CONFIG
  dist: *TOMCAT_DIST
```

:::hint
By the time you read this post, Chameleon may be shipping with a version that has Tomcat already configured.
:::

## Running Tests

For the sake of simplicity the tests we run against Tomcat and WildFly will be a HTTP GET request against their root directories. Normally in these tests you would be making API calls, deploying applications or anything else your application needs to do against these application servers.

The format of these tests is much the same as those presented in the [previous blog post](/blog/2017-11/arquillian-testing.md). There are 2 big differences though.

First, we don't have a `@Deployment` method, because we are not running any code inside the application servers.

Second, the test method we do have has the `@RunAsClient` annotation, which means this code is run outside the application servers. We consider these tests to be clients of the application server, not testing code that was deployed to them. By looking from the outside in, we can test our code as if it were an external application working with the application servers managed by Arquillian.

```java
package tomcat8;

import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.jboss.arquillian.container.test.api.RunAsClient;
import org.jboss.arquillian.junit.Arquillian;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;

import java.io.IOException;

@RunWith(Arquillian.class)
public class TomcatTest {
    @Test
    @RunAsClient
    public void connectToTomcat() throws IOException {
        try (CloseableHttpClient httpclient = HttpClients.createDefault()) {
            final HttpGet httpGet = new HttpGet("http://localhost:8080");
            try (CloseableHttpResponse response1 = httpclient.execute(httpGet)) {
                final int responseCode = response1.getStatusLine().getStatusCode();
                Assert.assertTrue(responseCode >= 200);
                Assert.assertTrue(responseCode <= 399);
            }
        }
    }
}
```

## Running the Tests

To run a test against a specific Maven profile, supply the `-P` argument, like so:

```
mvn clean verify -PTomcat8
```

Arquillian will download the binary distribution for the specified application server, start the server up, and run your tests. This is what the output looks like.

```
/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/bin/java -Dmaven.multiModuleProjectDirectory=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting "-Dmaven.home=/Users/matthewcasperson/Library/Application Support/JetBrains/Toolbox/apps/IDEA-U/ch-0/172.4574.11/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3" "-Dclassworlds.conf=/Users/matthewcasperson/Library/Application Support/JetBrains/Toolbox/apps/IDEA-U/ch-0/172.4574.11/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3/bin/m2.conf" "-javaagent:/Users/matthewcasperson/Library/Application Support/JetBrains/Toolbox/apps/IDEA-U/ch-0/172.4574.11/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=54994:/Users/matthewcasperson/Library/Application Support/JetBrains/Toolbox/apps/IDEA-U/ch-0/172.4574.11/IntelliJ IDEA.app/Contents/bin" -Dfile.encoding=UTF-8 -classpath "/Users/matthewcasperson/Library/Application Support/JetBrains/Toolbox/apps/IDEA-U/ch-0/172.4574.11/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3/boot/plexus-classworlds-2.5.2.jar" org.codehaus.classworlds.Launcher -Didea.version=2017.2.6 test -P Tomcat8
objc[12174]: Class JavaLaunchHelper is implemented in both /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/bin/java (0x1057b94c0) and /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/lib/libinstrument.dylib (0x1077f54e0). One of the two will be used. Which one is undefined.
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.octopus:arquillian-infrastructure-testing:jar:1.0-SNAPSHOT
[WARNING] 'build.plugins.plugin.version' for org.apache.maven.plugins:maven-surefire-plugin is missing. @ line 75, column 29
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building arquillian-infrastructure-testing 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ arquillian-infrastructure-testing ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ arquillian-infrastructure-testing ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ arquillian-infrastructure-testing ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 4 resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ arquillian-infrastructure-testing ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ arquillian-infrastructure-testing ---
[INFO] Surefire report directory: /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running tomcat8.TomcatTest
Nov 28, 2017 6:59:24 PM org.jboss.arquillian.container.tomcat.managed.TomcatManagedContainer start
INFO: Starting Tomcat with: [/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre/bin/java, -Djava.util.logging.config.file=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/conf/logging.properties, -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager, -Dcom.sun.management.jmxremote.port=8089, -Dcom.sun.management.jmxremote.ssl=false, -Dcom.sun.management.jmxremote.authenticate=false, -Xmx512m, -XX:MaxPermSize=128m, -classpath, /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/bin/bootstrap.jar:/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/bin/tomcat-juli.jar, -Djava.endorsed.dirs=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/endorsed, -Dcatalina.base=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47, -Dcatalina.home=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47, -Djava.io.tmpdir=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/temp, org.apache.catalina.startup.Bootstrap, -config, /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/conf/../../../../../src/test/resources/tomcat8-server.xml, start]
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
28-Nov-2017 18:59:24.858 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version:        Apache Tomcat/8.0.47
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Sep 29 2017 13:46:41 UTC
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server number:         8.0.47.0
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Mac OS X
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            10.13.1
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          x86_64
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           1.8.0_151-b12
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor:            Oracle Corporation
28-Nov-2017 18:59:24.860 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:         /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47
28-Nov-2017 18:59:24.861 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:         /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47
28-Nov-2017 18:59:24.861 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.config.file=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/conf/logging.properties
28-Nov-2017 18:59:24.861 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
28-Nov-2017 18:59:24.861 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcom.sun.management.jmxremote.port=8089
28-Nov-2017 18:59:24.861 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcom.sun.management.jmxremote.ssl=false
28-Nov-2017 18:59:24.862 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcom.sun.management.jmxremote.authenticate=false
28-Nov-2017 18:59:24.862 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Xmx512m
28-Nov-2017 18:59:24.862 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -XX:MaxPermSize=128m
28-Nov-2017 18:59:24.862 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.endorsed.dirs=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/endorsed
28-Nov-2017 18:59:24.862 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.base=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47
28-Nov-2017 18:59:24.862 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.home=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47
28-Nov-2017 18:59:24.862 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/temp
28-Nov-2017 18:59:24.862 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: /Users/matthewcasperson/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.
28-Nov-2017 18:59:24.966 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
28-Nov-2017 18:59:24.985 INFO [main] org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a shared selector for servlet write/read
28-Nov-2017 18:59:24.987 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["ajp-nio-8009"]
28-Nov-2017 18:59:24.989 INFO [main] org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a shared selector for servlet write/read
28-Nov-2017 18:59:24.989 INFO [main] org.apache.catalina.startup.Catalina.load Initialization processed in 386 ms
28-Nov-2017 18:59:25.010 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service Catalina
28-Nov-2017 18:59:25.010 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet Engine: Apache Tomcat/8.0.47
28-Nov-2017 18:59:25.017 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/docs
28-Nov-2017 18:59:25.306 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/docs has finished in 289 ms
28-Nov-2017 18:59:25.306 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/manager
28-Nov-2017 18:59:25.336 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/manager has finished in 30 ms
28-Nov-2017 18:59:25.336 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/examples
28-Nov-2017 18:59:25.535 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/examples has finished in 199 ms
28-Nov-2017 18:59:25.536 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/ROOT
28-Nov-2017 18:59:25.552 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/ROOT has finished in 16 ms
28-Nov-2017 18:59:25.552 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/host-manager
28-Nov-2017 18:59:25.564 INFO [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory /Users/matthewcasperson/Development/ArquillianInfrastructureTesting/target/server/tomcat_8.0.47/apache-tomcat-8.0.47/webapps/host-manager has finished in 12 ms
28-Nov-2017 18:59:25.566 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
28-Nov-2017 18:59:25.571 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["ajp-nio-8009"]
28-Nov-2017 18:59:25.573 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in 583 ms
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.047 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.906 s
[INFO] Finished at: 2017-11-28T18:59:26+10:00
[INFO] Final Memory: 14M/309M
[INFO] ------------------------------------------------------------------------

Process finished with exit code 0
```

## Conclusion

By utilizing Arquillian to download, initialize and clean up the various application servers it supports, we can quite easily test code that interacts with these servers as clients. This means that with a little effort to set up Maven and configure the Arquillian containers, tests can be run against many different application servers and many different versions of those servers. This approach has work out quite well for us with the Java code that serves as the glue between Octopus and Java application servers.

You can download the source code to this blog post from [GitHub](https://github.com/OctopusDeploy/ArquillianInfrastructureTesting).

If you are interested in automating the deployment of your Java applications, [download a trial copy of Octopus Deploy](https://octopus.com/downloads), and take a look at [our documentation](https://octopus.com/docs/deployments/java/deploying-java-applications).
