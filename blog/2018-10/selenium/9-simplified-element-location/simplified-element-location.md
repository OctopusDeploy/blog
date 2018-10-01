## Simplified element location

In the last lecture we looked at the various ways elements could be
located in a web page in order for a test to interact with them. We
built methods that allowed us to interact with elements found by their
ID, XPath and CSS Selector, and the homework assignment was to add
support for locate elements based on their name attribute.

But wouldn't it be nice if we could call a single set of methods with
the element identifier and let WebDriver work out which elements
matched?

Imagine you were developing a web application using a test driven
development approach. You might write a test like the following.

```java
public void formTestWithSimpleBy() throws URISyntaxException {

  final AutomatedBrowser automatedBrowser =
  AUTOMATED_BROWSER_FACTORY.getAutomatedBrowser("ChromeNoImplicitWait");

  // Populate these variables when it is known how to locate the elements
  final String formButtonLocator = "";
  final String formTextBoxLocator = "";
  final String formTextAreaLocator = "";
  final String formDropDownListLocator = "";
  final String formCheckboxLocator = "";
  final String messageLocator = "";

  try {

    automatedBrowser.init();

    automatedBrowser.goTo(FormTest.class.getResource("/form.html").toURI().toString());

    automatedBrowser.clickElement(formButtonLocator, 10);
    assertEquals("Button Clicked", automatedBrowser.getTextFromElement(messageLocator));

    automatedBrowser.populateElement(formTextBoxLocator, "test text", 10);
    assertEquals("Text Input Changed", automatedBrowser.getTextFromElement(messageLocator));

    automatedBrowser.populateElement(formTextAreaLocator, "test text",
    10);
    assertEquals("Text Area Changed", automatedBrowser.getTextFromElement(messageLocator));

    automatedBrowser.selectOptionByTextFromSelect("Option 2.1", formDropDownListLocator, 10);
    assertEquals("Select Changed", automatedBrowser.getTextFromElement(messageLocator));

    automatedBrowser.clickElement(formCheckboxLocator, 10);
    assertEquals("Checkbox Changed", automatedBrowser.getTextFromElement(messageLocator));
  } finally {
    automatedBrowser.destroy();
  }
}
```

Notice that the test calls method like `automatedBrowser.clickElement()`
or `automatedBrowser.populateElement()`. These methods do not specify that
the elements are to be found by their ID, an XPath or CSS Selector.
Indeed, using a test driven development approach, you will most likely
not know the best way to select these elements, because the web page
will not have been written yet. As a tester all we are concerned about
is that there is *some* way to locate the elements, with the actual
locators being something we fill in later when these values as known.

With a few tricks it is possible to write tests like this.

We'll start by creating an interface called `SimpleBy`. It will have one
method called `getElement()`. This method will return the first element to
match the locator string, whether that locator is an ID, an XPath, a CSS
Selector, a name or any other means of searching for elements.

```java
package com.octopus.utils;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;

public interface SimpleBy {
  WebElement getElement(WebDriver webDriver,
    String locator,
    int waitTime,
    ExpectedConditionCallback expectedConditionCallback);
}
```

The `ExpectedConditionCallback` interface defines a single method that
will take a By object, and return a `ExpectedCondition` object. We'll
make use of this to build an explicit wait condition.

```java
package com.octopus.utils;

import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedCondition;

@FunctionalInterface
public interface ExpectedConditionCallback {
  ExpectedCondition<WebElement> getExpectedCondition(By by);
}
```

To represents errors encountered when locating elements with a generic
locator, we create the `WebElementException` class.

```java
package com.octopus.exceptions;

public class WebElementException extends RuntimeException {

  public WebElementException() {

  }

  public WebElementException(final String message) {
    super(message);
  }

  public WebElementException(final String message, final Throwable ex) {
    super(message, ex);
  }

  public WebElementException(final Exception ex) {
    super(ex);
  }
}
```

Implementing the `SimpleBy` interface is the `SimpleByImpl` class. This is
where the magic happens.

```java
package com.octopus.utils.impl;

import com.octopus.exceptions.WebElementException;
import com.octopus.utils.ExpectedConditionCallback;
import com.octopus.utils.SimpleBy;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedCondition;
import java.util.concurrent.TimeUnit;

public class SimpleByImpl implements SimpleBy {

private static final int MILLISECONDS_PER_SECOND = 1000;
private static final int TIME_SLICE = 100;

@Override
public WebElement getElement(

  WebDriver webDriver,

  String locator,

  int waitTime,

  ExpectedConditionCallback expectedConditionCallback) {

    final By[] byInstances = new By[] {
      By.id(locator),
      By.xpath(locator),
      By.cssSelector(locator),
      By.className(locator),
      By.linkText(locator),
      By.name(locator)
    };

    long time = -1;

    while (time < waitTime * MILLISECONDS_PER_SECOND) {
      for (final By by : byInstances) {
        try {
          final WebDriverWaitEx wait = new WebDriverWaitEx(
          webDriver,
          TIME_SLICE,
          TimeUnit.MILLISECONDS);
          final ExpectedCondition<WebElement> condition =
          expectedConditionCallback.getExpectedCondition(by);
          return wait.until(condition);
        } catch (final Exception ignored) {
          /*
            Do nothing
          */
        }

        time += TIME_SLICE;
      }
    }

    throw new WebElementException("All attempts to find element failed");
  }
}
```

Let's break this class down.

Inside the `getElement()` method, we create a number of By instances,
passing in the generic locator string. The locator string could be
anything: an XPath, an ID, a CSS Selector. We don't know what kind of
locator we have, but by constructing multiple different implementations
of the By class with it, we have a multiple different ways to attempt to
find a matching element.

```java
@Override
public WebElement getElement(
  WebDriver webDriver,
  String locator,
  int waitTime,

  ExpectedConditionCallback expectedConditionCallback) {
    final By[] byInstances = new By[] {
      By.id(locator),
      By.xpath(locator),
      By.cssSelector(locator),
      By.className(locator),
      By.linkText(locator),
      By.name(locator)
  };
```

We then enter a `while` loop. This loop will always run at least once,
because we start the `time` variable at `-1`. The loop will continue while
the time spent in the loop is less than the time we have allocated to
find a matching element.

```java
long time = -1;

while (time < waitTime * MILLISECONDS_PER_SECOND) {
```

Next we loop over each instance of the `By` class that we created earlier.
Our intention here is to find the one `By` instance that can use the
supplied locator to actually match an element.

```java
for (final By by : byInstances) {
```

Inside the `try` block we execute an explicit wait, only this time we wait
for a short amount of time. `TIME_SLICE` is set to 0.1 of a second, which
means each implementation of By that this iteration of the loop is
working with has a fraction of a second in which to find a matching
element.

Notice that we call the functional interface passed into the
`expectedConditionCallback` parameter to convert the instance of the `By`
class into the desired expected condition. This allows the caller of the
method to decide what state the element should be in in order to be
considered a match.

It is worth noting that the WebDriver API is not thread safe, so we have
to execute our code in a sequential fashion like this instead of running
each test in a separate thread.

The catch block does nothing, because we expect that most attempts to
find an element with a given implementation of the By class will fail.
For example, if the locator string was set to an XPath like
`//*[@name="button_element"]`, the locator `By.cssSelector(locator)`
will never find a match because it only works with CSS Selectors.

However, the locator `By.xpath(locator)` could well find a match, in which
case return `wait.until(condition)` will exit the loop and return the
matching element.

```java
try {
  final WebDriverWaitEx wait = new WebDriverWaitEx(
  webDriver,
  TIME_SLICE,
  TimeUnit.MILLISECONDS);
  final ExpectedCondition<WebElement> condition = expectedConditionCallback.getExpectedCondition(by);

  return wait.until(condition);
} catch (final Exception ignored) {
  /*
    Do nothing
  */
}
```

If the current implementations of `By` does not successfully find a
matching element, we increment time (the total time spent finding the
element) by `TIME_SLICE` (the time we allocated to this implementation of
By), and move onto the next search.

```java
time += TIME_SLICE;
```

If all attempts to find the element failed within the allotted time, we
throw an exception.

```java
throw new WebElementException("All attempts to find element failed");
```

The end result of this code is to give each implementation of `By` a short
amount of time to find something that matches the supplied locator
string. The first one to find a match returns it, or an exception is
thrown.

![](image1.tmp)

You may wonder what happens if two implementations of `By` could return a
match. This can happen if the `id` attribute of an element is the same as
the `name` element of another element. In this case, the first match wins
and is returned.

In practise though such conflicts are rare, and can be easily resolved
by passing in an XPath or CSS Selector as the locator. The odds of your
web page having a XPath or CSS Selector embedded in an `id`, `name` or `class`
attribute is exceptionally small, and so they will not match more than
one element.

If you had a keen eye you may have noticed that we created an instance
of `WebDriverWaitEx` instead of the usual `WebDriverWait`. `WebDriverWaitEx`
extends `WebDriverWait`, and adds an additional constructor that allow it
to be configured to wait for sub-second amounts of time (`WebDriverWait`
can wait for no less than 1 second). This is important to us, because if
each instance of the `By` class that we were testing took 1 second to
complete, the entire loop would take at least 6 seconds to process,
which is too long.

```java
package com.octopus.utils.impl;

import org.openqa.selenium.WebDriver;

import org.openqa.selenium.support.ui.Clock;

import org.openqa.selenium.support.ui.Sleeper;

import org.openqa.selenium.support.ui.SystemClock;

import org.openqa.selenium.support.ui.WebDriverWait;

import java.util.concurrent.TimeUnit;

public class WebDriverWaitEx extends WebDriverWait {

public static final long DEFAULT_SLEEP_TIMEOUT = 10;

public WebDriverWaitEx(final WebDriver driver, final long
timeOutInSeconds) {

this(driver, new SystemClock(), Sleeper.SYSTEM_SLEEPER,
timeOutInSeconds, DEFAULT_SLEEP_TIMEOUT);

}

public WebDriverWaitEx(final WebDriver driver, final long timeOut, final
TimeUnit time) {

this(driver, new SystemClock(), Sleeper.SYSTEM_SLEEPER, timeOut,
DEFAULT_SLEEP_TIMEOUT, time);

}

public WebDriverWaitEx(final WebDriver driver, final long
timeOutInSeconds, final long sleepInMillis) {

this(driver, new SystemClock(), Sleeper.SYSTEM_SLEEPER,
timeOutInSeconds, sleepInMillis);

}

public WebDriverWaitEx(

final WebDriver driver,

final Clock clock,

final Sleeper sleeper,

final long timeOutInSeconds,

final long sleepTimeOut) {

this(driver, clock, sleeper, timeOutInSeconds, sleepTimeOut,
TimeUnit.SECONDS);

}

public WebDriverWaitEx(

final WebDriver driver,

final Clock clock,

final Sleeper sleeper,

final long timeOut,

final long sleepTimeOut,

final TimeUnit time) {

// Call the WebDriverWait constructor with a timeout of 0

super(driver, clock, sleeper, 0, sleepTimeOut);

// Now set the timeout, possibly as a sub-second duration

withTimeout(timeOut, time);

}

}

We now have the ability to find elements based on any kind of locator.
To take advantage of this, we add the following methods to the
AutomatedBrowser interface.

void clickElement(String locator);

void clickElement(String locator, int waitTime);

void selectOptionByTextFromSelect(String optionText, String locator);

void selectOptionByTextFromSelect(String optionText, String locator, int
waitTime);

void populateElement(String locator, String text);

void populateElement(String locator, String text, int waitTime);

String getTextFromElement(String locator);

String getTextFromElement(String locator, int waitTime);

The usual default methods are added to AutomatedBrowserBase.

@Override

public void clickElement(final String locator) {

if (getAutomatedBrowser() != null) {

getAutomatedBrowser().clickElement(locator);

}

}

@Override

public void clickElement(final String locator, final int waitTime) {

if (getAutomatedBrowser() != null) {

getAutomatedBrowser().clickElement(locator, waitTime);

}

}

@Override

public void selectOptionByTextFromSelect(final String optionText, final
String locator) {

if (getAutomatedBrowser() != null) {

getAutomatedBrowser().selectOptionByTextFromSelect(optionText, locator);

}

}

@Override

public void selectOptionByTextFromSelect(final String optionText, final
String locator, final int waitTime) {

if (getAutomatedBrowser() != null) {

getAutomatedBrowser().selectOptionByTextFromSelect(optionText, locator,
waitTime);

}

}

@Override

public void populateElement(final String locator, final String text) {

if (getAutomatedBrowser() != null) {

getAutomatedBrowser().populateElement(locator, text);

}

}

@Override

public void populateElement(final String locator, final String text,
final int waitTime) {

if (getAutomatedBrowser() != null) {

getAutomatedBrowser().populateElement(locator, text, waitTime);

}

}

@Override

public String getTextFromElement(final String locator) {

if (getAutomatedBrowser() != null) {

return getAutomatedBrowser().getTextFromElement(locator);

}

return null;

}

@Override

public String getTextFromElement(final String locator, final int
waitTime) {

if (getAutomatedBrowser() != null) {

return getAutomatedBrowser().getTextFromElement(locator, waitTime);

}

return null;

}

And the following implementations are added to WebDriverDecorator.

private static final SimpleBy SIMPLE_BY = new SimpleByImpl();

@Override

public void clickElement(final String locator) {

clickElement(locator, 0);

}

@Override

public void clickElement(final String locator, final int waitTime) {

SIMPLE_BY.getElement(

getWebDriver(),

locator,

waitTime,

by -> ExpectedConditions.elementToBeClickable(by)).click();

}

@Override

public void selectOptionByTextFromSelect(final String optionText, final
String locator) {

selectOptionByTextFromSelect(optionText, locator, 0);

}

@Override

public void selectOptionByTextFromSelect(final String optionText, final
String locator, final int waitTime) {

new Select(SIMPLE_BY.getElement(

getWebDriver(),

locator,

waitTime,

by ->
ExpectedConditions.elementToBeClickable(by))).selectByVisibleText(optionText);

}

@Override

public void populateElement(final String locator, final String text) {

populateElement(locator, text, 0);

}

@Override

public void populateElement(final String locator, final String text,
final int waitTime) {

SIMPLE_BY.getElement(

getWebDriver(),

locator,

waitTime,

by -> ExpectedConditions.elementToBeClickable(by)).sendKeys(text);

}

@Override

public String getTextFromElement(final String locator) {

return getTextFromElement(locator, 0);

}

@Override

public String getTextFromElement(final String locator, final int
waitTime) {

return SIMPLE_BY.getElement(

getWebDriver(),

locator,

waitTime,

by -> ExpectedConditions.presenceOfElementLocated(by)).getText();

}
```

Now we can complete the test, using a mix of IDs, CSS Selectors and
XPaths. The new methods will automatically find any matching elements,
and we don't need to worry about matching the correct locator to the
correct method.

The use of the `ChromeNoImplicitWait` configuration is very important
here. If you recall from the last lecture, mixing implicit and explicit
waits can lead some undesirable results, and this is once such case. Had
we used a configuration that had implicit waits enabled, calls to the
methods we implemented above could conceivably take nearly a minute to
complete, as each of the 6 explicit waits end up waiting the 10 seconds
we configured for the implicit wait. By using the `ChromeNoImplicitWait`
configuration, we ensure that the explicit waits in the `SimpleByImpl`
class take only a fraction of a second.

```java
@Test
public void formTestWithSimpleBy() throws URISyntaxException {
  final AutomatedBrowser automatedBrowser = AUTOMATED_BROWSER_FACTORY.getAutomatedBrowser("ChromeNoImplicitWait");

  final String formButtonLocator = "button_element";
  final String formTextBoxLocator = "text_element";
  final String formTextAreaLocator = "textarea_element";
  final String formDropDownListLocator = "[name=select_element]";

  final String formCheckboxLocator =
  "//*[@name=\\"checkbox1_element\\"]";

  final String messageLocator = "message";

  try {
    automatedBrowser.init();

    automatedBrowser.goTo(FormTest.class.getResource("/form.html").toURI().toString());

    automatedBrowser.clickElement(formButtonLocator, 10);
    assertEquals("Button Clicked",
    automatedBrowser.getTextFromElement(messageLocator));

    automatedBrowser.populateElement(formTextBoxLocator, "test text", 10);
    assertEquals("Text Input Changed",
    automatedBrowser.getTextFromElement(messageLocator));

    automatedBrowser.populateElement(formTextAreaLocator, "test text", 10);
    assertEquals("Text Area Changed",
    automatedBrowser.getTextFromElement(messageLocator));

    automatedBrowser.selectOptionByTextFromSelect("Option 2.1", formDropDownListLocator, 10);
    assertEquals("Select Changed", automatedBrowser.getTextFromElement(messageLocator));

    automatedBrowser.clickElement(formCheckboxLocator, 10);
    assertEquals("Checkbox Changed", automatedBrowser.getTextFromElement(messageLocator));
  } finally {
    automatedBrowser.destroy();
  }
}
```

In my own experience, these new methods we have added to the
`AutomatedBrowser` interface are far more convenient than methods that are
tied to a specific locator. They remove the need to manually keep
locators and the methods they are passed to in sync, and the code is
more readable too. For this reason future lectures will uses these new
methods almost exclusively.
