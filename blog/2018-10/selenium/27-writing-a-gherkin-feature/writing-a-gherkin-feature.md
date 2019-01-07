---
title: Writing a Gherkin feature
description: In this post we write a complete test in Gherkin
author: matthew.casperson@octopus.com
visibility: private
bannerImage: webdriver.png
metaImage: webdriver.png
tags:
- Java
---

Now that we know how to construct regular expressions to map methods to Gherkin steps, we can go ahead and add annotations to all appropriate methods in the `AutomatedBrowserBase` class.

Notice that we don't add annotations for all the methods. Methods like `getTextFromElementWithId()`, which return a value, are not able to be used in Gherkin steps because Gherkin has no notion of variables, so the return values don't have any meaning. We also don't expose methods like `init()` and `destroy()`, as these lifecycle methods are called by the `openBrowser()` and `closeBrowser()` methods. There are some internal only methods like `getWebDriver()`, `getAutomatedBrowser()`, `setAutomatedBrowser()` and `getDesiredCapabilities()` that are only used by the decorators, and do not make any sense to expose as Gherkin steps.

The remaining steps have Cucumber annotations applied to them, assigning regular expressions that follow the same logic that we saw in the last post.

```java
package com.octopus.decoratorbase;

import com.octopus.AutomatedBrowser;
import com.octopus.AutomatedBrowserFactory;
import cucumber.api.java.en.And;
import cucumber.api.java.en.Given;
import cucumber.api.java.en.Then;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.DesiredCapabilities;

public class AutomatedBrowserBase implements AutomatedBrowser {

  // ...

  public AutomatedBrowserBase() {

  }

  public AutomatedBrowserBase(AutomatedBrowser automatedBrowser) {
    // ...
  }

  public AutomatedBrowser getAutomatedBrowser() {
    // ...
  }

  public void setAutomatedBrowser(AutomatedBrowser automatedBrowser) {
    // ...
  }

  @Given("^I open the browser \"([^\"]*)\"$")
  public void openBrowser(String browser) {
    // ...
  }

  @Given("^I close the browser$")
  public void closeBrowser() {
    // ...
  }

  @Override
  public WebDriver getWebDriver() {
    // ...
  }

  @Override
  public void setWebDriver(WebDriver webDriver) {
    // ...
  }

  @Override

  public DesiredCapabilities getDesiredCapabilities() {
    // ...
  }

  @Override
  public void init() {
    // ...
  }

  @Override
  public void destroy() {
    // ...
  }

  @And("^I open the URL \"([^\"]*)\"$")
  @Override
  public void goTo(String url) {
    // ...
  }

  @And("^I click the \\w+(?:\\s+\\w+)* with the id \"([^\"]*)\"$")
  @Override
  public void clickElementWithId(String id) {
    // ...
  }

  @And("^I click the \\w+(?:\\s+\\w+)* with the id \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void clickElementWithId(String id, int waitTime) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the id \"([^\"]*)\"$")
  @Override
  public void selectOptionByTextFromSelectWithId(String optionText, String id) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the id \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void selectOptionByTextFromSelectWithId(String optionText, String id, int waitTime) {
    // ...
  }

  @And("^I populate the \\w+(?:\\s+\\w+)* with the id \"([^\"]*)\" with the text \"([^\"]*)\"$")
  @Override
  public void populateElementWithId(String id, String text) {
    // ...
  }

  @And("^I populate the \\w+(?:\\s+\\w+)* with the id \"([^\"]*)\" with the text \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void populateElementWithId(String id, String text, int waitTime)
  {
    // ...
  }

  @Override
  public String getTextFromElementWithId(String id) {
    // ...
  }

  @Override
  public String getTextFromElementWithId(String id, int waitTime) {
    // ...
  }

  @And("^I click the \\w+(?:\\s+\\w+)* with the xpath \"([^\"]*)\"$")
  @Override
  public void clickElementWithXPath(String xpath) {
    // ...
  }

  @And("^I click the \\w+(?:\\s+\\w+)* with the xpath \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void clickElementWithXPath(String xpath, int waitTime) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the xpath \"([^\"]*)\"$")
  @Override
  public void selectOptionByTextFromSelectWithXPath(String optionText, String xpath) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the xpath \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void selectOptionByTextFromSelectWithXPath(String optionText, String xpath, int waitTime) {
    // ...
  }

  @And("^I populate the \\w+(?:\\s+\\w+)* with the xpath \"([^\"]*)\" with the text \"([^\"]*)\"$")
  @Override
  public void populateElementWithXPath(String xpath, String text) {
    // ...
  }

  @And("^I populate the \\w+(?:\\s+\\w+)* with the xpath \"([^\"]*)\" with the text \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void populateElementWithXPath(String xpath, String text, int waitTime) {
    // ...
  }

  @Override
  public String getTextFromElementWithXPath(String xpath) {
    // ...
  }

  @Override
  public String getTextFromElementWithXPath(String xpath, int waitTime) {
    // ...
  }

  @And("^I click the \\w+(?:\\s+\\w+)* with the css selector \"([^\"]*)\"$")
  @Override
  public void clickElementWithCSSSelector(String cssSelector) {
    // ...
  }

  @And("^I click the \\w+(?:\\s+\\w+)* with the css selector \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void clickElementWithCSSSelector(String cssSelector, int waitTime) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the css selector \"([^\"]*)\"$")
  @Override
  public void selectOptionByTextFromSelectWithCSSSelector(String optionText, String cssSelector) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the css selector \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void selectOptionByTextFromSelectWithCSSSelector(String optionText, String cssSelector, int waitTime) {
    // ...
  }

  @And("^I populate the \\w+(?:\\s+\\w+)* with the css selector
  \"([^\"]*)\" with the text \"([^\"]*)\"$")
  @Override
  public void populateElementWithCSSSelector(String cssSelector, String text) {
    // ...
  }

  @And("^I populate the \\w+(?:\\s+\\w+)* with the css selector \"([^\"]*)\" with the text \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void populateElementWithCSSSelector(String cssSelector, String text, int waitTime) {
    // ...
  }

  @Override
  public String getTextFromElementWithCSSSelector(String cssSelector) {
    // ...
  }

  @Override
  public String getTextFromElementWithCSSSelector(String cssSelector, int waitTime) {
    // ...
  }

  @And("^I click the \\w+(?:\\s+\\w+)* with the name \"([^\"]*)\"$")
  @Override
  public void clickElementWithName(String name) {
    // ...
  }

  @And("^I click the \\w+(?:\\s+\\w+)* with the name \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void clickElementWithName(String name, int waitTime) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the name \"([^\"]*)\"$")
  @Override
  public void selectOptionByTextFromSelectWithName(String optionText, String name) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the name \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void selectOptionByTextFromSelectWithName(String optionText, String name, int waitTime) {
    // ...
  }

  @And("^I populate the \\w+(?:\\s+\\w+)* with the name \"([^\"]*)\" with the text \"([^\"]*)\"$")
  @Override
  public void populateElementWithName(String name, String text) {
    // ...
  }

  @And("^I populate the \\w+(?:\\s+\\w+)* with the name \"([^\"]*)\" with the text \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void populateElementWithName(String name, String text, int waitTime) {
    // ...
  }

  @Override
  public String getTextFromElementWithName(String name) {
    // ...
  }

  @Override
  public String getTextFromElementWithName(String name, int waitTime) {
    // ...
  }

  @And("^I click the \"([^\"]*)\" \\w+(?:\\s+\\w+)*$")
  @Override
  public void clickElement(String locator) {
    // ...
  }

  @And("^I click the \"([^\"]*)\" \\w+(?:\\s+\\w+)* waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void clickElement(String locator, int waitTime) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \"([^\"]*)\" \\w+(?:\\s+\\w+)*$")
  @Override
  public void selectOptionByTextFromSelect(String optionText, String locator) {
    // ...
  }

  @And("^I select the option \"([^\"]*)\" from the \"([^\"]*)\" \\w+(?:\\s+\\w+)* waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void selectOptionByTextFromSelect(String optionText, String locator, int waitTime) {
    // ...
  }

  @And("^I populate the \"([^\"]*)\" \\w+(?:\\s+\\w+)* with the text \"([^\"]*)\"$")
  @Override
  public void populateElement(String locator, String text) {
    // ...
  }

  @And("^I populate the \"([^\"]*)\" \\w+(?:\\s+\\w+)* with the text \"([^\"]*)\" waiting up to \"(\\d+)\" seconds?$")
  @Override
  public void populateElement(String locator, String text, int waitTime) {
    // ...
  }

  @Override
  public String getTextFromElement(String locator) {
    // ...
  }

  @Override
  public String getTextFromElement(String locator, int waitTime) {
    // ...
  }

  @And("^I capture the HAR file$")
  @Override
  public void captureHarFile() {
    // ...
  }

  @And("^I capture the complete HAR file$")
  @Override
  public void captureCompleteHarFile() {
    // ...
  }

  @And("^I save the HAR file to \"([^\"]*)\"$")
  @Override
  public void saveHarFile(String file) {
    // ...
  }

  @And("^I block the request to \"([^\"]*)\" returning the HTTP code \"\\d+\"$")
  @Override
  public void blockRequestTo(final String url, final int responseCode) {
    // ...
  }

  @And("^I alter the response fron \"([^\"]*)\" returning the
  HTTP code \"\\d+\" and the response body:$")
  @Override
  public void alterResponseFrom(String url, int responseCode, String responseBody) {
    // ...
  }

  @And("^I maximize the window$")
  @Override
  public void maximizeWindow() {
    // ...
  }

}
```

With these annotations in place, we can now write a feature file to complete a test of a ticket purchase from TicketMonster.

Save the following code to the file
`src/test/resources/com/octopus/ticketmonster.feature`.

```gherkin
Feature: Test TicketMonster
  Scenario: Purchase Tickets
    Given I open the browser "ChromeNoImplicitWait"
    When I open the URL "https://ticket-monster.herokuapp.com"
    And I click the "Buy tickets now" button waiting up to "30" seconds
    And I click the "Concert" link waiting up to "30" seconds
    And I click the "Rock concert of the decade" link waiting up to "30" seconds
    And I select the option "Toronto : Roy Thomson Hall" from the "venueSelector" drop down list waiting up to "30" seconds
    And I click the "bookButton" button waiting up to "30" seconds
    And I select the option "A - Premier platinum reserve" from the "sectionSelect" drop down list waiting up to "30" seconds
    And I populate the "tickets-1" text box with the text "2" waiting up to "30" seconds
    And I click the "add" button waiting up to "30" seconds
    And I populate the "email" text box with the text "email@example.org" waiting up to "30" seconds
    And I click the "submit" button waiting up to "30" seconds
    Then I close the browser
```

Now either run the `CucumberTest` test class from IntelliJ, or commit the code to GitHub and let Travis CI run the test for you. We have just successfully replicated the journey through the TicketMonster application that we wrote in Java in a previous post.

If you read this test out aloud it almost sounds like instructions you would give a colleague if you were instructing them to complete a ticket purchase. But the format is still a bit clunky. Most of the steps end with the phrase `waiting up to "30" seconds`, and some of the locators like `tickets-1` don't give a lot of context.

Let's address the needless repetition of the phrase `waiting up to "30" seconds`.

We start by adding a new method called `setDefaultExplicitWaitTime()` to the `AutomatedBrowser` interface. We'll use this method to set a default time to be used with an explicit wait on all the steps.

```java
package com.octopus;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.DesiredCapabilities;

public interface AutomatedBrowser {
  // ...
  void setDefaultExplicitWaitTime(int waitTime);
  // ...
}
```

This method is then implemented in the `AutomatedBrowserBase` class, and exposed as a Gherkin step.

```java
package com.octopus.decoratorbase;

import com.octopus.AutomatedBrowser;
import com.octopus.AutomatedBrowserFactory;
import cucumber.api.java.en.And;
import cucumber.api.java.en.Given;
import cucumber.api.java.en.Then;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.DesiredCapabilities;

public class AutomatedBrowserBase implements AutomatedBrowser {

  // ...

  @And("^I set the default explicit wait time to \"(\\d+)\" seconds?$")
  @Override
  public void setDefaultExplicitWaitTime(int waitTime) {
    if (getAutomatedBrowser() != null) {
      getAutomatedBrowser().setDefaultExplicitWaitTime(waitTime);
    }
  }

  // ...

}
```

Then in the `WebDriverDecorator` class we capture the default wait time in the `setDefaultExplicitWaitTime()` method, and use the default wait time if it is greater than 0 for any of the methods that previously did not accept a `waitTime` parameter.

```java
package com.octopus.decorators;

import com.octopus.AutomatedBrowser;
import com.octopus.decoratorbase.AutomatedBrowserBase;
import com.octopus.utils.SimpleBy;
import com.octopus.utils.impl.SimpleByImpl;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;

public class WebDriverDecorator extends AutomatedBrowserBase {

  // ...

  private int defaultExplicitWaitTime;

  // ...

  @Override
  public void setDefaultExplicitWaitTime(final int waitTime) {
    defaultExplicitWaitTime = waitTime;
  }

  // ...

  @Override
  public void clickElementWithId(final String id) {
    if (defaultExplicitWaitTime <= 0) {
        webDriver.findElement(By.id(id)).click();
      } else {
        clickElementWithId(id, defaultExplicitWaitTime);
    }
  }

  // ...

  @Override
  public void clickElement(final String locator) {
    clickElement(locator, defaultExplicitWaitTime);
  }

  // ...

}
```

In the `setDefaultExplicitWaitTime()` method the `defaultExplicitWaitTime` instance variable is set to the default wait time.

```java
private int defaultExplicitWaitTime;

@Override
public void setDefaultExplicitWaitTime(final int waitTime) {
  defaultExplicitWaitTime = waitTime;
}
```

We then use this default value in any method that interacts with an element, but does not accept a wait time parameter. For example, the `clickElementWithId()` method below does not accept a wait time parameter, and will by default attempt to click the element immediately with no wait.

With the change we are making, if `defaultExplicitWaitTime` is greater
than zero we instead call the overloaded `clickElementWithId()` method
that does accept a wait time parameter, passing in the value of
`defaultExplicitWaitTime`. This means that if `defaultExplicitWaitTime` has
been defined, methods that did not accept a wait time parameter now
defer to those overloaded version of the method that do, and will in
turn wait for a period of time for the element that is being interacted
with to be available and in the correct state.

All the methods that do no accept a wait time parameter have been
rewritten with this new `if` statement.

```java
@Override
public void clickElementWithId(final String id) {
  if (defaultExplicitWaitTime <= 0) {
    webDriver.findElement(By.id(id)).click();
  } else {
    clickElementWithId(id, defaultExplicitWaitTime);
  }
}
```

The only methods that don't use the same logic of checking to see if `defaultExplicitWaitTime` is greater than 0 are those that use the simplified locator strings. These methods already deferred to their overloaded siblings with a wait time of zero, which is now replaced with the `defaultExplicitWaitTime` variable.

```java
@Override
public void clickElement(final String locator) {
  clickElement(locator, defaultExplicitWaitTime);
}
```

Now we can write the Gherkin feature like this. We call the step `And I set the default explicit wait time to "30" seconds`, and remove the phrase `waiting up to "30" seconds` from all the other steps.

```gherkin
Feature: Test TicketMonster With Default Wait
  Scenario: Purchase Tickets with default wait time
    Given I open the browser "ChromeNoImplicitWait"
    And I set the default explicit wait time to "30" seconds
    When I open the URL "https://ticket-monster.herokuapp.com"
    And I click the "Buy tickets now" button
    And I click the "Concert" link
    And I click the "Rock concert of the decade" link
    And I select the option "Toronto : Roy Thomson Hall" from the "venueSelector" drop down list
    And I click the "bookButton" button
    And I select the option "A - Premier platinum reserve" from the "sectionSelect" drop down list
    And I populate the "tickets-1" text box with the text "2"
    And I click the "add" button
    And I populate the "email" text box with the text "email@example.org"
    And I click the "submit" button
    Then I close the browser
```

We are now very close to having a test that can be written and read in something close to plain English. The last remaining hurdle are the locators like `tickets-1`, which are hard to read.

Gherkin does not have any native notions of constants, meaning we need to introduce something we'll call aliases. Aliases are nothing more than key value pairs, but they allow us to assign a meaningful key like `Adult Ticket Count` to the value `tickets-1`. We can then use the key `Adult Ticket Count` in the Gherkin step, making the step much more readable.

To store these key/value pairs, we create a new instance variable called `aliases`, and a new method called `setAliases()` to save them.

```java
public class AutomatedBrowserBase implements AutomatedBrowser {
  // ...

  private Map<String, String> aliases = new HashMap<>();

  // ...

  @Given("^I set the following aliases:$")
  public void setAliases(Map<String, String> aliases) {
    this.aliases.putAll(aliases);
  }

  // ...

}
```

We then make use of a feature in Cucumber called data tables to populate the aliases map.

Notice that the regular expression `^I set the following aliases:$` has no capture groups. Traditionally we use capture groups as a way of passing values to the method parameters. But in this case the data table is supplied after the step, and passed into the method as a `Map` object.

```java
@Given("^I set the following aliases:$")
public void setAliases(Map<String, String> aliases) {
  this.aliases.putAll(aliases);
}
```

We call this step like so. The table underneath the step is passed to the method as a `Map`, with the first column being the key and the second column is the value.

```gherkin
And I set the following aliases:
  | Venue | venueSelector |
  | Book | bookButton |
  | Section | sectionSelect |
  | Adult Ticket Count | tickets-1 |
  | Add Tickets | add |
  | Checkout | submit |
```

Now that we can populate this map, we need a way to read it.

Java 8 has a handy method called `getOrDefault()` that allows us to get a value from a map, or return a default value. Now inside every method of the `AutomatedBrowserBase` class we pass string parameters to the child `AuotomatedBrowser` instance using the string value as a key of the aliases map, or if the aliases map does not contain the string as a key, the parameter is passed as is.

For example, instead of calling:

```java
getAutomatedBrowser().selectOptionByTextFromSelectWithId(optionText, id)
```

which passes the parameters `optionText` and `id` directly through to the child `AuotomatedBrowser` instance, we instead call:

```java
getAutomatedBrowser().selectOptionByTextFromSelectWithId(
  aliases.getOrDefault(optionText, optionText),
  aliases.getOrDefault(id, id))
```

The code `aliases.getOrDefault(optionText, optionText)` means "Get the value assigned to the key `optionText` from the `aliases` map, or if that key does not exist, return `optionText` as the default value".

The code below shows how the methods in the `AutomatedBrowserBase` class now look as they first try to look up the aliases map for an aliased value. Every method has been updated to look up the aliases map, and the code below shows how the `selectOptionByTextFromSelectWithId()`
method was updated.

```java
@And("^I select the option \"([^\"]*)\" from the \\w+(?:\\s+\\w+)* with the id \"([^\"]*)\"$")
@Override
public void selectOptionByTextFromSelectWithId(String optionText, String
id) {
  if (getAutomatedBrowser() != null) {
    getAutomatedBrowser().selectOptionByTextFromSelectWithId(
    aliases.getOrDefault(optionText, optionText),
    aliases.getOrDefault(id, id));
  }
}
```

These changes mean we can now write the test like this. The aliases map now gives obscure locators like `tickets-1` a readable name like `Adult Ticket Count`.

```gherkin
Feature: Test TicketMonster With Aliases
  Scenario: Purchase Tickets with default wait time and aliases
    Given I open the browser "ChromeNoImplicitWait"
    And I set the following aliases:
      | Venue | venueSelector |
      | Book | bookButton |
      | Section | sectionSelect |
      | Adult Ticket Count | tickets-1 |
      | Add Tickets | add |
      | Checkout | submit |
    And I set the default explicit wait time to "30" seconds
    When I open the URL "https://ticket-monster.herokuapp.com"
    And I click the "Buy tickets now" button
    And I click the "Concert" link
    And I click the "Rock concert of the decade" link
    And I select the option "Toronto : Roy Thomson Hall" from the "Venue" drop down list
    And I click the "Book" button
    And I select the option "A - Premier platinum reserve" from the "Section" drop down list
    And I populate the "Adult Ticket Count" text box with the text "2"
    And I click the "Add Tickets" button
    And I populate the "email" text box with the text "email@example.org"
    And I click the "Checkout" button
    Then I close the browser
```

Now that we have aliases exposing the element ids and names with friendly names like `Add Tickets` and `Checkout`, the test fulfils the requirement of providing the implementation details required to execute the test while also being easy to read. Anyone familiar with the TicketMonster web application would be able to follow these instructions to purchase tickets for the concert. This is the beauty of the Gherkin language, and the power of the Cucumber library.
