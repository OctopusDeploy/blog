---
title: An Introduction to Arquillian Testing
description: Testing Java EE code requires more than just a unit test. This blog post looks at how Arquillian solves the problem of testing Java EE apps.
author: matthew.casperson@octopus.com
visibility: public
published: 2017-11-28
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - DevOps
---

Let's create a simple EJB application with two classes. The first, called `EnterpriseJavaBean`, writes a UUID to the console, sleeps for a period of time, and writes the same UUID to the console again.

```java
package org.example.arquilliantest;

import javax.ejb.Asynchronous;
import javax.ejb.Singleton;
import java.util.Random;
import java.util.UUID;

@Singleton
public class EnterpriseJavaBean {
    @Asynchronous
    public void writeToConsole() {
        final UUID uuid = UUID.randomUUID();
        System.out.println(uuid.toString());

        try {
            Thread.sleep(Math.abs(new Random().nextLong()) % 100);
        } catch (InterruptedException e) {
            // ignored
        }

        System.out.println(uuid.toString());
    }
}
```

The second class, called `StartupService`, is constructed on startup, and after construction calls `EnterpriseJavaBean.writeToConsole()` 10 times.

```java
package org.example.arquilliantest;

import javax.annotation.PostConstruct;
import javax.ejb.EJB;
import javax.ejb.Singleton;
import javax.ejb.Startup;

@Startup
@Singleton
public class StartupService {
    @EJB
    private EnterpriseJavaBean enterpriseJavaBean;

    @PostConstruct
    public void postConstruct() {
        for (int i = 0; i < 10; ++i) {
            enterpriseJavaBean.writeToConsole();
        }
        System.out.println("All Done");
    }
}
```

The code here is run of the mill EJB logic that you would find in any Java EE application. Once compiled and run, something like the following would be printed to the console.

```
12:56:19,359 INFO  [stdout] (EJB default - 4) 2a3c2709-c556-49be-8ca7-25778b54310d
12:56:19,442 INFO  [stdout] (EJB default - 4) 2a3c2709-c556-49be-8ca7-25778b54310d
12:56:19,444 INFO  [stdout] (EJB default - 9) 4be307b4-9e4b-4f37-b1b4-a54f234a591c
12:56:19,471 INFO  [stdout] (EJB default - 9) 4be307b4-9e4b-4f37-b1b4-a54f234a591c
12:56:19,472 INFO  [stdout] (EJB default - 10) ef7a63ae-1e65-4d9e-aed0-97ddf6681956
12:56:19,519 INFO  [stdout] (EJB default - 10) ef7a63ae-1e65-4d9e-aed0-97ddf6681956
12:56:19,520 INFO  [stdout] (EJB default - 2) 8dc989b8-48e2-4128-9b3a-d80a883033a7
12:56:19,615 INFO  [stdout] (EJB default - 2) 8dc989b8-48e2-4128-9b3a-d80a883033a7
12:56:19,618 INFO  [stdout] (EJB default - 1) 4ae8e179-6b3c-44c3-9cee-62a93c069bba
12:56:19,719 INFO  [stdout] (EJB default - 1) 4ae8e179-6b3c-44c3-9cee-62a93c069bba
12:56:19,721 INFO  [stdout] (EJB default - 6) 3cf93b22-dd92-45ea-95d3-73873322579e
12:56:19,784 INFO  [stdout] (EJB default - 6) 3cf93b22-dd92-45ea-95d3-73873322579e
12:56:19,786 INFO  [stdout] (EJB default - 3) b38ce136-78dc-49b5-85db-c11f491a5f14
12:56:19,856 INFO  [stdout] (default task-1) All Done
12:56:19,877 INFO  [stdout] (EJB default - 3) b38ce136-78dc-49b5-85db-c11f491a5f14
12:56:19,878 INFO  [stdout] (EJB default - 7) f0bb9c74-fdec-4994-b66d-296e310da24d
12:56:19,945 INFO  [stdout] (EJB default - 7) f0bb9c74-fdec-4994-b66d-296e310da24d
12:56:19,946 INFO  [stdout] (EJB default - 5) e8ef8311-dd2d-45c1-af07-a3059d92f680
12:56:19,972 INFO  [stdout] (EJB default - 5) e8ef8311-dd2d-45c1-af07-a3059d92f680
12:56:19,973 INFO  [stdout] (EJB default - 8) b40cf503-c05c-4282-95be-3bf03875bc2f
12:56:20,045 INFO  [stdout] (EJB default - 8) b40cf503-c05c-4282-95be-3bf03875bc2f
12:56:20,046 INFO  [stdout] (EJB default - 4) a0a70b9f-dd13-41fd-95a9-f8b38a062377
12:56:20,085 INFO  [stdout] (EJB default - 4) a0a70b9f-dd13-41fd-95a9-f8b38a062377
12:56:20,086 INFO  [stdout] (EJB default - 9) f8069c90-c980-4593-805f-352c7fcb13ae
12:56:20,110 INFO  [stdout] (EJB default - 9) f8069c90-c980-4593-805f-352c7fcb13ae
12:56:20,111 INFO  [stdout] (EJB default - 10) e0fe386b-18e5-4883-a14f-a8af75b010ef
12:56:20,116 INFO  [stdout] (EJB default - 10) e0fe386b-18e5-4883-a14f-a8af75b010ef
12:56:20,118 INFO  [stdout] (EJB default - 2) 9d057b61-cb3f-4da4-b401-9f1e8d251669
12:56:20,215 INFO  [stdout] (EJB default - 2) 9d057b61-cb3f-4da4-b401-9f1e8d251669
12:56:20,217 INFO  [stdout] (EJB default - 1) 17b80b1e-6885-4a38-8051-7246854fd54c
12:56:20,227 INFO  [stdout] (EJB default - 1) 17b80b1e-6885-4a38-8051-7246854fd54c
12:56:20,229 INFO  [stdout] (EJB default - 6) 81dcd134-e704-4c77-b8b0-04a15e163705
12:56:20,302 INFO  [stdout] (EJB default - 6) 81dcd134-e704-4c77-b8b0-04a15e163705
12:56:20,303 INFO  [stdout] (EJB default - 3) 4e32ecb9-9e95-42b0-8077-3cb80f40faca
12:56:20,398 INFO  [stdout] (EJB default - 3) 4e32ecb9-9e95-42b0-8077-3cb80f40faca
```

Let's assume that the sequence of output above is the valid output, and that we want to verify this behavior as part of a unit test.

## Testing POJOs With JUnit

One of the selling points of the EJB 3 spec is that it allows you to write POJOs. With a few additional annotations, these POJOs become fully fledged EJBs. Or at least they will become EJBs in the right environment. But more on that later.

Because these classes are POJOs, we can easily incorporate them into unit tests.

Ignoring the work required to actually capture and validate console output, this is what our test might look like. It replicates
the same logic found in the `StartupService` class, so you might think that it would produce much the same output.

```java
@Test
public void plainTest() {
    final EnterpriseJavaBean enterpriseJavaBean = new EnterpriseJavaBean();
    for (int i = 0; i < 10; ++i) {
        enterpriseJavaBean.writeToConsole();
    }
    System.out.println("All Done");
}
```

When the test is run, we come close to the original output.

```
dc1d071a-18f9-4cd4-9f90-70768c818165
dc1d071a-18f9-4cd4-9f90-70768c818165
983ade5f-f514-4530-bd3e-e5c71910b3b4
983ade5f-f514-4530-bd3e-e5c71910b3b4
1ce137d0-1d9e-4f26-9b00-75ae72fb4cee
1ce137d0-1d9e-4f26-9b00-75ae72fb4cee
cd8bc751-6021-4a92-ab0a-03021f66c353
cd8bc751-6021-4a92-ab0a-03021f66c353
916e331c-2b54-4c9c-b3e6-f2625d816f7b
916e331c-2b54-4c9c-b3e6-f2625d816f7b
706bf787-d35d-49f2-8963-39010b53245d
706bf787-d35d-49f2-8963-39010b53245d
afa3d953-0599-4f8d-8e58-c22f79a7c789
afa3d953-0599-4f8d-8e58-c22f79a7c789
9ac2dc9b-dc8c-4f96-ba98-f38b299d9be8
9ac2dc9b-dc8c-4f96-ba98-f38b299d9be8
07e9bf94-6e38-4420-a175-81f436451832
07e9bf94-6e38-4420-a175-81f436451832
9027ca83-690d-4d45-beda-4ae599844f44
9027ca83-690d-4d45-beda-4ae599844f44
All Done
```

A keen observer would note that the `All Done` message is always printed at the end of the unit test output, but is printed at random points during the execution of the `EnterpriseJavaBean` class when deployed to a server.

This behavior is due to the fact that the `writeToConsole()` method has been marked as `@Asynchronous`.

These EJB annotations are ignored by any code that does not recognise them, and our JUnit test is one example of an execution environment that does not recognise EJB annotations. This means the unit tests will call this method synchronously, which in turn means the `All Done` message will always be displayed last.

This is the first indication that testing EJBs is not quite as straight forward as it might seem. But so far the differences are obvious; the method we are testing is clearly marked as `@Asynchronous`, so we can replicate this behavior easily enough. Lets call the `writeToConsole()` method inside an Executor, which will make the calls in an asynchronous manner.

```
@Test
public void plainThreadTest() throws InterruptedException {
    final EnterpriseJavaBean enterpriseJavaBean = new EnterpriseJavaBean();

    try {
        final ExecutorService executor = Executors.newFixedThreadPool(10);

        final List<Callable<Void>> tasks = new ArrayList<>();
        for (int i = 0; i < 10; ++i) {
            executor.submit(() -> {
                enterpriseJavaBean.writeToConsole();
                return null;
            });
        }

        System.out.println("All Done");
        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
    } catch (Exception e) {
        // ignored
    }
}
```

This produces the output:

```
All Done
b887112d-d92e-4a1c-8986-0b1e25916c99
4ded12d5-a837-495a-a199-059cb58a2f11
63e12097-6c62-416c-ae60-bd873554e376
18a722fe-548a-48c1-bc6c-eaec651522fa
8e2273c0-73cc-4dfc-9964-a1c6c40055b4
c41acb02-61fc-4a4b-bd68-45545b4ed734
b74acccf-1b38-45c8-9375-1d7380698214
85b71c13-9b3c-4e4b-bdb7-32064a6a9818
c2fa0677-291a-4066-8dcb-13f4a6896489
3ed6dc3a-590e-4350-a6a4-fc0e3e799505
b74acccf-1b38-45c8-9375-1d7380698214
4ded12d5-a837-495a-a199-059cb58a2f11
18a722fe-548a-48c1-bc6c-eaec651522fa
85b71c13-9b3c-4e4b-bdb7-32064a6a9818
63e12097-6c62-416c-ae60-bd873554e376
c41acb02-61fc-4a4b-bd68-45545b4ed734
b887112d-d92e-4a1c-8986-0b1e25916c99
c2fa0677-291a-4066-8dcb-13f4a6896489
8e2273c0-73cc-4dfc-9964-a1c6c40055b4
3ed6dc3a-590e-4350-a6a4-fc0e3e799505
```

Now we are calling the `writeToConsole()` method from a thread pool, which gets us closer to the way EJBs would call `@Asynchronous` methods. The `All Done` message is now no longer always at the end of the output, but the UUIDs are all jumbled up. This is not the behavior that we see when executing the `writeToConsole()` on the server, which will always print matching UUID pairs one after the other.

The issue here is that when the methods on the `EnterpriseJavaBean` class are called when it is injected as an EJB, the methods default to the semantics defined by the `@Lock(WRITE)` annotation. This means methods can only be called one at a time, which ensures the UUID pairs are always printed one after the other.

But this behavior is not immediately obvious when you look at the code for the `EnterpriseJavaBean` class. There is no `@Lock(WRITE)` annotation anywhere; it is a default that is assumed by an EJB container.

We could then go ahead and try to add some synchronization around the `writeToConsole()` method, but at this point we have spent more time replicating the environment in which an EJB is executed than verifying the business logic in these methods.

## Arquillian to the Recuse

This situation is resolved by the [Arquillian](http://arquillian.org/) project. Arquillian allows you to write tests against EJBs (or any kind of enhanced object) taking advantage of any functionality bestowed on the objects by the server environment.

It is actually quite easy to write an Arquillian test. Here is an example that replicates the behavior of the `EnterpriseJavaBean` class.

```java
package org.example.arquilliantest;

import org.jboss.arquillian.container.test.api.Deployment;
import org.jboss.arquillian.junit.Arquillian;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.spec.JavaArchive;
import org.junit.Test;
import org.junit.runner.RunWith;

import javax.ejb.EJB;

@RunWith(Arquillian.class)
public class ArquillianTest {

    @EJB
    private EnterpriseJavaBean enterpriseJavaBean;

    @Deployment
    public static JavaArchive createDeployment() {
        JavaArchive jar = ShrinkWrap.create(JavaArchive.class)
                .addClass(EnterpriseJavaBean.class);
        return jar;
    }

    @Test
    public void plainThreadTest() throws InterruptedException {
        for (int i = 0; i < 10; ++i) {
            enterpriseJavaBean.writeToConsole();
        }
        System.out.println("All Done");
    }
}
```

There are few important aspects to this test.

The `@RunWith(Arquillian.class)` annotation enriches the JUnit test with the functionality provided by Arquillian.

The `createDeployment()` method with the `@Deployment` annotation is used to create the artifact which is deployed as the test environment. In this case we are constructing an jar file that includes the `EnterpriseJavaBean` class. This is done using the [ShrinkWrap](http://arquillian.org/guides/shrinkwrap_introduction/) library, which allows us to create Java artifacts in code in much the same way a build tool like Maven would at build time.

Inside the test we have injected an instance of the `EnterpriseJavaBean` class using the `@EJB` annotation. The object that this variable references when the test is run is a real, live EJB. This means that instead of testing a POJO with EJB annotations that are ignored (and therefore testing an object with none of the functionality of an EJB), we are testing an object that behaves in the same way that it would when deployed to an application server.

Finally the `@Test` method itself runs the same test code that we originally tried to write, but this time the output is exactly what we see when the code is deployed to a real server.

## Conclusion

By running tests with Arquillian, it is possible to test the actual functionality of an object as it would provide when deployed to an application server. This frees you from trying to replicate the functionality bestowed by an application server, and means your tests reflect your production code.

You can get the source code to this project from [GitHub](https://github.com/OctopusDeploy/ArquillianTest).

If you are interested in automating the deployment of your Java applications, [download a trial copy of Octopus Deploy](https://octopus.com/downloads), and take a look at [our documentation](https://octopus.com/docs/deployments/java/deploying-java-applications).
