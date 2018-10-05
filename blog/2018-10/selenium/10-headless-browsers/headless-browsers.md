---
title: Headless Browsers
description: In this post we learn how to run tests against headless browsers.
author: matthew.casperson@octopus.com
visibility: private
bannerImage: webdriver.png
metaImage: webdriver.png
tags:
- Java
---

You will have noticed by now that running tests with WebDriver results in a browser window being opened and the web pages being loaded and interacted with as if by some invisible mouse pointer. While it can be useful to watch the progression of a test in the browser, there are times when it is desirable to have the tests complete off-screen. For example, running tests as part of a continuous deployment process does not require anyone to watch the browser as the tests are executed. Indeed, sometimes there is not even a monitor attached to the systems that are running the tests -  this is known as a headless environment. So how can we run tests in such headless environments?

This is a problem that projects like [PhantomJS](http://phantomjs.org/) were created to solve. PhantomJS is a web browser based on WebKit, which is the library that powers browsers like Apple Safari. Unlike a traditional browser though, PhantomJS has no GUI, and is designed to be controlled by technologies like WebDriver. Because it has no GUI, PhantomJS can be run on continuous integration servers that are traditionally hosted on headless servers. This means you can run WebDriver tests on a central server in response to application changes without having to launch a browser window in a desktop environment.

Recently browsers like Firefox and Chrome have added native support for headless browsing. This is a great benefit to anyone writing WebDriver tests, as it means that the tests can be run on the very same browsers that end users have installed, while still allowing tests to be run on a headless server.

These days development of PhantomJS has stalled. One of the maintainers of the project has [stepped down](https://groups.google.com/forum/#!topic/phantomjs/9aI5d-LDuNE), and the latest release of PhantomJS is over 2 years old. But the good news is that it is quite easy to configure Chrome and Firefox to run tests in a headless environment.

Before we start configuring headless browsers, we need to add some additional support for configuring the driver classes.

WebDriver uses a class called `DesiredCapabilities` that serves as a generic container for browser driver settings. The `DesiredCapabilities` class is essentially a container for key value pairs, with some convenience methods for configuring commonly used settings.

First we add the method `getDesiredCapabilities()` to the `AutomatedBrowser` interface.

```java
public interface AutomatedBrowser {
  // ...
  DesiredCapabilities getDesiredCapabilities();
  // ...
}
```

Then we add a default method in the `AutomatedBrowserBase` class.

This method differs a little from the typical default decorator method implementation in that if there is no parent `AutomatedBrowser` instance to return an instance of the `DesiredCapabilities` class from, we return a new instance of `DesiredCapabilities` instead of `null`. This ensures that if no decorator has provided any `DesiredCapabilities`, we can always rely on a default instance being returned.

```java
@Override
public DesiredCapabilities getDesiredCapabilities() {
  if (getAutomatedBrowser() != null) {
    return getAutomatedBrowser().getDesiredCapabilities();
  }

  return new DesiredCapabilities();
}
```

The `DesiredCapabilities` class is used for configuration settings that are common to all browsers. Each driver then has a corresponding "options" class that is used to configure browser specific settings. These two objects are merged together to build up the complete set of configuration settings.

Here is the code for the `ChromeDecorator` class updated to support these two configuration classes. We create an instance of the `ChromeOptions` class, `merge()` it with the common settings returned by `getDesiredCapabilities()`, and pass the merged result to the `ChromeDriver()` constructor.

This code does not configure any additional settings yet, but does demonstrate how the `DesiredCapabilities` class is used in conjunction with the browser specific options class.

```java
package com.octopus.decorators;

import com.octopus.AutomatedBrowser;
import com.octopus.decoratorbase.AutomatedBrowserBase;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

public class ChromeDecorator extends AutomatedBrowserBase {

    public ChromeDecorator(final AutomatedBrowser automatedBrowser) {
        super(automatedBrowser);
    }

    @Override
    public void init() {
        final ChromeOptions options = new ChromeOptions();
        options.merge(getDesiredCapabilities());
        final WebDriver webDriver = new ChromeDriver(options);
        getAutomatedBrowser().setWebDriver(webDriver);
        getAutomatedBrowser().init();
    }
}
```

We follow the same pattern for the `FirefoxDecorator` class, merging the `FirefoxDriver` class with the `DesiredCapabilities` class.

```java
package com.octopus.decorators;

import com.octopus.AutomatedBrowser;
import com.octopus.decoratorbase.AutomatedBrowserBase;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;

public class FirefoxDecorator extends AutomatedBrowserBase {
  public FirefoxDecorator(final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
  }

  @Override
  public void init() {
    final FirefoxOptions options = new FirefoxOptions();
    options.merge(getDesiredCapabilities());
    final WebDriver webDriver = new FirefoxDriver(options);
    getAutomatedBrowser().setWebDriver(webDriver);
    getAutomatedBrowser().init();
  }
}
```

Starting a browser in headless mode is done by configuring either the `ChromeOptions` or `FirefoxOptions` instances.

To launch Chrome in headless mode, we pass some arguments to the chrome executable. The `ChromeOptions` class provides a simple way to configure these arguments through the method `setHeadless()`.

Let's take a look at the code for the `ChromeDecorator` class to allow us to run Chrome in headless mode.

```java
package com.octopus.decorators;

import com.octopus.AutomatedBrowser;
import com.octopus.decoratorbase.AutomatedBrowserBase;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

public class ChromeDecorator extends AutomatedBrowserBase {

  final boolean headless;

  public ChromeDecorator(final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
    this.headless = false;
  }

  public ChromeDecorator(final boolean headless, final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
    this.headless = headless;
  }

  @Override
  public void init() {
    final ChromeOptions options = new ChromeOptions();
    options.setHeadless(headless);
    options.merge(getDesiredCapabilities());
    final WebDriver webDriver = new ChromeDriver(options);
    getAutomatedBrowser().setWebDriver(webDriver);
    getAutomatedBrowser().init();

  }
}
```

First we provide a instance variable called `headless` to track whether this instance of the Chrome browser should be run in headless mode or not. To set this variable we overload the constructor.

```java
final boolean headless;

public ChromeDecorator(final AutomatedBrowser automatedBrowser) {
  super(automatedBrowser);
  this.headless = false;
}

public ChromeDecorator(final boolean headless, final AutomatedBrowser automatedBrowser) {
  super(automatedBrowser);
  this.headless = headless;
}
```

In the `init()` method we make a call to `setHeadless()` to enable or disable headless mode (although given headless mode is disabled by default, calling `setHeadless(false)` doesn't change anything).

```java
options.setHeadless(headless);
```

Taking a look at the `ChomeOptions.setHeadless()` method we can see that headless mode is enabled by passing the `--headless` and `--disable-gpu` arguments to Chrome.

```java
public ChromeOptions setHeadless(boolean headless) {

  args.remove("--headless");
  if (headless) {
    args.add("--headless");
    args.add("--disable-gpu");
  }

  return this;
}
```

We then update the `AutomatedBrowserFactory getChromeBrowser()` method with a parameter to define if the Chrome browser should be headless or not.

```java
private AutomatedBrowser getChromeBrowser(final boolean headless) {
  return new ChromeDecorator(headless,
    new ImplicitWaitDecorator(10,
      new WebDriverDecorator()
    )
  );
}
```

Finally we update the `getAutomatedBrowser()` method to allow a headless instance of Chrome to be created.

```java
public AutomatedBrowser getAutomatedBrowser(String browser) {
  if ("Chrome".equalsIgnoreCase(browser)) {
    return getChromeBrowser(false);
  }

  if ("ChromeHeadless".equalsIgnoreCase(browser)) {
    return getChromeBrowser(true);
  }

  // ...
}
```

With these changes in place, we can update the tests to run them against a headless instance of Chrome.

When the tests are run, you will not see the browser window be displayed. But the test will execute in the background and pass as before.

```java
@Test
public void formTestByIDHeadless() throws URISyntaxException {
  final AutomatedBrowser automatedBrowser =
    AUTOMATED_BROWSER_FACTORY.getAutomatedBrowser("ChromeHeadless");

  // ...
}
```

The process for creating a headless instance of Firefox is almost exactly the same as for Chrome.

First the `FirefoxDecorator` class is updated with a constructor that sets the headless instance variable, and a call to `setHeadless()` in the options class configures the headless mode on the driver.

```java
package com.octopus.decorators;

import com.octopus.AutomatedBrowser;
import com.octopus.decoratorbase.AutomatedBrowserBase;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;

public class FirefoxDecorator extends AutomatedBrowserBase {

  final boolean headless;

  public FirefoxDecorator(final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
    this.headless = false;
  }

  public FirefoxDecorator(final boolean headless, final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
    this.headless = headless;
  }

  @Override
  public void init() {
    final FirefoxOptions options = new FirefoxOptions();
    options.setHeadless(headless);
    options.merge(getDesiredCapabilities());
    final WebDriver webDriver = new FirefoxDriver(options);
    getAutomatedBrowser().setWebDriver(webDriver);
    getAutomatedBrowser().init();
  }
}
```

Looking at the `FirefoxOptions.setHeadless()` method, we can see that headless mode is enabled by passing the `-headless` argument to Firefox.

```java
public FirefoxOptions setHeadless(boolean headless) {
  args.remove("-headless");
  if (headless) {
    args.add("-headless");
  }
  return this;
}
```

The `AutomatedBrowserFactory getFirefoxBrowser()` method is then updated to support setting headless mode.

```java
private AutomatedBrowser getFirefoxBrowser(final boolean headless) {
  return new FirefoxDecorator(headless,
    new ImplicitWaitDecorator(10,
      new WebDriverDecorator()
    )
  );
}
```

And the `getAutomatedBrowser()` method is updated to support creating headless instances of Firefox.

```java
public AutomatedBrowser getAutomatedBrowser(String browser) {
  //...
  if ("Firefox".equalsIgnoreCase(browser)) {
    return getFirefoxBrowser(false);
  }

  if ("FirefoxHeadless".equalsIgnoreCase(browser)) {
    return getFirefoxBrowser(true);
  }

  //...
}
```

Then, just as with the Chrome browser, the tests can be updated to use the headless version of Firefox.

```java
@Test
public void formTestByIDHeadlessFirefox() throws URISyntaxException {
  final AutomatedBrowser automatedBrowser =
    AUTOMATED_BROWSER_FACTORY.getAutomatedBrowser("FirefoxHeadless");

  // ...
}
```

Running tests on specialized browsers like PhantomJS that didn't quite behave like "real" browsers used to be a pain point for testers, but was a necessary evil. By supporting headless browsing, browsers like Chrome and Firefox have paved the way for testers to utilize the same browsers used by end users in automated tests on headless servers. We'll take advantage of these headless browsers in later lectures as we integrate with platforms like Travis CI and AWS Lambda.

In addition, by exposing the ability to configure browsers via the `DesiredCapabilities` class we have provided a hook that we can take advantage of with new decorators to add functionality such as custom proxies, which is exactly what we'll be doing in the next lecture.
