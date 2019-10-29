---
title: "Selenium series: The Page Object Model design pattern"
description: In this post, we learn about the Page Object Model design pattern.
author: matthew.casperson@octopus.com
visibility: public
published: 2018-10-01
bannerImage: webdriver.png
metaImage: webdriver.png
tags:
- DevOps
---

This post is part of a series about [creating a Selenium WebDriver test framework](../0-toc/webdriver-toc.md).

While our previous test successfully verified the process of purchasing a ticket for an event in TicketMonster, this style of testing where we define each interaction with the page in a sequential order has some limitations.

The first limitation is that each of the interactions are not particularly descriptive. Someone with limited knowledge of the application being tested would quite understandably be confused by a line of code like:

```java
automatedBrowser.populateElement("tickets-1", "2", WAIT_TIME);
```

What is `tickets-1` in this case? This identifier does not describe the element that it locates particularly well. You would need to be intimately familiar with the application being tested to know what this line of code does.

The second, and perhaps most significant, limitation is that the code used to construct this test is not reusable. Imagine that in addition to this test that purchases a ticket, we wanted to write a second test that verified the prices of the tickets for each available section. You might write a test like this to ensure that pricing changes don’t result in unreasonably high or low ticket prices.

To write a second test, each of the interactions with the application would need to be copied and pasted into a second test method. However, copying and pasting is best avoided because it makes code much harder to maintain, since common functionality now lives in multiple methods and all have to be individually updated.

One solution to both of these limitations is to use the Page Object Model, or POM, design pattern. The POM design pattern encapsulates the interactions with an individual page inside a class. This means the interactions with a page are now exposed behind methods with friendly names, and the classes can be reused between tests.

Let’s take a look at how we can rewrite the test using the POM design pattern.

Each POM class that we create needs to have access to an `AutomatedBrowser` instance. In addition, we will define a common wait time for the explicit waits used when interacting with elements. Exposing these shared properties is done with a class called `BasePage`.

Note that the instance variables, static variables and constructor all have the protected scope. This means that they are only available to classes that extend `BasePage`, meaning `BasePage` is not something we can instantiate directly:

```java
package com.octopus.pages;

import com.octopus.AutomatedBrowser;

public class BasePage {

  protected static final int WAIT_TIME = 30;

  protected final AutomatedBrowser automatedBrowser;

  protected BasePage(AutomatedBrowser automatedBrowser) {
    this.automatedBrowser = automatedBrowser;
  }
}
```

We start all our tests on the main page of the TicketMonster application. To represent this main page, we create the class `MainPage`:

```java
package com.octopus.pages.ticketmonster;

import com.octopus.AutomatedBrowser;

import com.octopus.pages.BasePage;

public class MainPage extends BasePage {

  private static final String URL =
    "https://ticket-monster.herokuapp.com";

  private static final String BUY_TICKETS_NOW = "Buy tickets now";

  public MainPage(final AutomatedBrowser automatedBrowser) {

  super(automatedBrowser);

  }

  public MainPage openPage() {
    automatedBrowser.goTo(URL);
    return this;
  }

  public EventsPage buyTickets() {
    automatedBrowser.clickElement(BUY_TICKETS_NOW, WAIT_TIME);
    return new EventsPage(automatedBrowser);
  }
}
```

Let’s break this code down.

All of our POM classes will extend `BasePage`. Extending `BasePage` gives these classes access to an instance of `AutomatedBrowser` as well as using a shared default wait time:

```java
public class MainPage extends BasePage {
```

To make URLs and element identifiers more maintainable we assign the strings to constants. Using constant variables means we can give these strings some meaningful context, which will be important later on when dealing with some of the more unintuitive element locators:

```java
private static final String URL =
  "https://ticket-monster.herokuapp.com";

private static final String BUY_TICKETS_NOW = "Buy tickets now";
```

The constructor takes an instance of `AutomatedBrowser` and passes it to the `BasePage` constructor:

```java
public MainPage(final AutomatedBrowser automatedBrowser) {
  super(automatedBrowser);
}
```

The first method we define will open up the URL to the application main page. To accomplish this action we create a method called `openPage()`.

It is in methods like `openPage()` that the specific details of URLs to be opened or elements to be interacted with are encapsulated. Callers of this method do not need to know the URL that is being opened, and should the URL change, it only needs to be changed in one location making maintenance much easier.

To allow calls to be chained, we return an instance of `this` as the final statement. Each method in POM classes will return the POM class that additional interactions can be performed on. This way consumers of the POM classes can easily understand the flow of the application, which we will see when we write the test method:

```java
public MainPage openPage() {
  automatedBrowser.goTo(URL);
  return this;
}
```

The only action we are interested in on the main page is clicking the `Buy tickets now` link, which we do in a method called `buyTickets()`.

If you recall from the previous post, how we interacted with elements like this link was not as easy as it might have appeared, because these elements could either be styled links (`<a>` elements), or form buttons (`<input>` elements). Depending on which kind of element was used, our first test had to use a different locater. Links could be identified by their text, while form buttons had to be identified by an `id` or `name` attribute.

This distinction between elements is no longer the concern of those writing these tests, but instead has been encapsulated inside this POM class. Calling the `buyTickets()` method will result in the desired element being clicked, regardless of how that element has been located.

Because clicking this link directs the browser to the events page, we return an instance of the `EventsPage` class. Callers of the `buyTickets()` can understand that this return value indicates that a page navigation has taken place, and that further interactions with the page now have to be performed using the `EventsPage` class:

```java
public EventsPage buyTickets() {
  automatedBrowser.clickElement(BUY_TICKETS_NOW, WAIT_TIME);
  return new EventsPage(automatedBrowser);
}
```

Let’s take a look at the `EventsPage` class:

```java
package com.octopus.pages.ticketmonster;

import com.octopus.AutomatedBrowser;

import com.octopus.pages.BasePage;

public class EventsPage extends BasePage {

  public EventsPage(final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
  }

  public VenuePage selectEvent(final String category, final String event)
  {
    automatedBrowser.clickElement(category, WAIT_TIME);
    automatedBrowser.clickElement(event, WAIT_TIME);
    return new VenuePage(automatedBrowser);
  }
}
```

As with the `MainPage` class, `EventsPage` extends the `BasePage` class and passes an instance of `AutomatedBrowser` from its constructor to the `BaseClass` constructor:

```java
public class EventsPage extends BasePage {
  public EventsPage(final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
  }
```

The only thing we want to do on the events page is select the event we’re purchasing tickets for. This involves clicking links in the left hand collapsible menu.

To allow this method to select any of the options in the menu, we pass in the menu category and event name as parameters.

Selecting an event will trigger the application to load the venue page. We represent this navigation by returning an instance of the `VenuePage` class:

```java
public VenuePage selectEvent(final String category, final String event)
{
  automatedBrowser.clickElement(category, WAIT_TIME);
  automatedBrowser.clickElement(event, WAIT_TIME);
  return new VenuePage(automatedBrowser);
}
```

Let’s take a look at the `VenuePage` class:

```java
package com.octopus.pages.ticketmonster;

import com.octopus.AutomatedBrowser;
import com.octopus.pages.BasePage;

public class VenuePage extends BasePage {

  private static final String VENUE_DROP_DOWN_LIST = "venueSelector";
  private static final String BOOK_BUTTON = "bookButton";

  public VenuePage(final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
  }

  public VenuePage selectVenue(final String venue) {
    automatedBrowser.selectOptionByTextFromSelect(venue, VENUE_DROP_DOWN_LIST, WAIT_TIME);
    return this;
  }

  public CheckoutPage book() {
    automatedBrowser.clickElement(BOOK_BUTTON, WAIT_TIME);
    return new CheckoutPage(automatedBrowser);
  }
}
```

This class extends `BasePage`, passes `AutomatedBrowser` to the `BasePage` constructor and defines some constants for the locators of the venue drop-down list and the book button:

```java
public class VenuePage extends BasePage {
  private static final String VENUE_DROP_DOWN_LIST = "venueSelector";
  private static final String BOOK_BUTTON = "bookButton";

  public VenuePage(final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
  }
```

Selecting a venue is performed by the `selectVenue()` method, with the venue name passed in as an argument.

Selecting a venue does not trigger any page navigation, so we return `this` to indicate that future interactions will still be performed on the same page:

```java
public VenuePage selectVenue(final String venue) {
  automatedBrowser.selectOptionByTextFromSelect(venue,
  VENUE_DROP_DOWN_LIST, WAIT_TIME);

  return this;
}
```

Moving to the booking page is performed by the `book()` method.

Clicking the book button results in the application navigating to the checkout page, which we represent by returning an instance of the `CheckoutPage` class:

```java
public CheckoutPage book() {
  automatedBrowser.clickElement(BOOK_BUTTON, WAIT_TIME);
  return new CheckoutPage(automatedBrowser);
}
```

Let’s take a look at the `CheckoutPage` class:

```java
package com.octopus.pages.ticketmonster;

import com.octopus.AutomatedBrowser;

import com.octopus.pages.BasePage;

public class CheckoutPage extends BasePage {

    private static final String SECTION_DROP_DOWN_LIST = "sectionSelect";
    private static final String ADULT_TICKET_COUNT = "tickets-1";
    private static final String ADD_TICKETS_BUTTON = "add";
    private static final String EMAIL_ADDRESS = "email";
    private static final String CHECKOUT_BUTTON = "submit";

    public CheckoutPage(final AutomatedBrowser automatedBrowser) {
        super(automatedBrowser);
    }

    public CheckoutPage buySectionTickets(final String section, final
    Integer adultCount) {
        automatedBrowser.selectOptionByTextFromSelect(section, SECTION_DROP_DOWN_LIST, WAIT_TIME);
        automatedBrowser.populateElement(ADULT_TICKET_COUNT, adultCount.toString(), WAIT_TIME);
        automatedBrowser.clickElement(ADD_TICKETS_BUTTON, WAIT_TIME);
        return this;
    }

    public ConfirmationPage checkout(final String email) {
        automatedBrowser.populateElement(EMAIL_ADDRESS, email, WAIT_TIME);
        automatedBrowser.clickElement(CHECKOUT_BUTTON, WAIT_TIME);
        return new ConfirmationPage(automatedBrowser);
    }
}
```

This class extends `BasePage` and passes `AutomatedBrowser` to the `BasePage` constructor.

The constant variables here are a good example of why you should use variables to provide context to locator strings. In particular, the locators `tickets-1` and `submit` don’t have any obvious link to the elements they identify. However, we can provide some meaningful context to these locators through their variable names of `ADULT_TICKET_COUNT` and `CHECKOUT_BUTTON`:

```java
public class CheckoutPage extends BasePage {

  private static final String SECTION_DROP_DOWN_LIST =
  "sectionSelect";
  private static final String ADULT_TICKET_COUNT = "tickets-1";
  private static final String ADD_TICKETS_BUTTON = "add";
  private static final String EMAIL_ADDRESS = "email";
  private static final String CHECKOUT_BUTTON = "submit";

  public CheckoutPage(final AutomatedBrowser automatedBrowser) {
    super(automatedBrowser);
  }
```

To buy tickets in a given section we use the `buySectionTickets()` method. This method selects the desired section from the drop-down list, adds the number of tickets to be bought, and clicks the `Add` button.

This action does not result in any page navigation, so we return `this`:

```java
  public CheckoutPage buySectionTickets(final String section, final Integer adultCount) {

  automatedBrowser.selectOptionByTextFromSelect(section,
  SECTION_DROP_DOWN_LIST, WAIT_TIME);

  automatedBrowser.populateElement(ADULT_TICKET_COUNT,
  adultCount.toString(), WAIT_TIME);

  automatedBrowser.clickElement(ADD_TICKETS_BUTTON, WAIT_TIME);

  return this;
}
```

To purchase the tickets we use the `checkout()` method. This method accepts the email address to be associated with the purchase, enters that email address into the appropriate field, and clicks the `Checkout` button.

Clicking the `Checkout` button navigates us to the confirmation page, so we return an instance of the `ConfirmationPage` class:

```java
public ConfirmationPage checkout(final String email) {
  automatedBrowser.populateElement(EMAIL_ADDRESS, email, WAIT_TIME);
  automatedBrowser.clickElement(CHECKOUT_BUTTON, WAIT_TIME);
  return new ConfirmationPage(automatedBrowser);
}
```

Let’s take a look at `the ConfirmationPage` class:

```java
package com.octopus.pages.ticketmonster;

import com.octopus.AutomatedBrowser;
import com.octopus.pages.BasePage;

public class ConfirmationPage extends BasePage {

    private static final String EMAIL_ADDRESS = "div.col-md-6:nth-child(1) > div:nth-child(1) > p:nth-child(2)";
    private static final String EVENT_NAME = "div.col-md-6:nth-child(1) > div:nth-child(1) > p:nth-child(3)";
    private static final String VENUE_NAME = "div.col-md-6:nth-child(1) > div:nth-child(1) > p:nth-child(4)";

    public ConfirmationPage(final AutomatedBrowser automatedBrowser) {
        super(automatedBrowser);
    }

    public String getEmail() {
        return automatedBrowser.getTextFromElement(EMAIL_ADDRESS, WAIT_TIME);
    }

    public String getEvent() {
        return automatedBrowser.getTextFromElement(EVENT_NAME, WAIT_TIME);
    }

    public String getVenue() {
        return automatedBrowser.getTextFromElement(VENUE_NAME, WAIT_TIME);
    }
}
```

As has been the case for every other POM class, this class extends `BasePage` and passes `AutomatedBrowser` to the `BasePage` constructor.

The elements that we want to interact with on this page have no attributes that we could use to identify them, forcing us to use CSS selectors. The use of constant variables here is particularly important in giving these locators some context as strings like `"div.col-md-6:nth-child(1) > div:nth-child(1) > p:nth-child(2)"` are difficult to decipher:

```java
public class ConfirmationPage extends BasePage {

    private static final String EMAIL_ADDRESS = "div.col-md-6:nth-child(1) > div:nth-child(1) > p:nth-child(2)";
    private static final String EVENT_NAME = "div.col-md-6:nth-child(1) > div:nth-child(1) > p:nth-child(3)";
    private static final String VENUE_NAME = "div.col-md-6:nth-child(1) > div:nth-child(1) > p:nth-child(4)";

    public ConfirmationPage(final AutomatedBrowser automatedBrowser) {
        super(automatedBrowser);
    }
```

Unlike the other POM classes, we are not clicking, selecting, or populating any elements on this page. We are however interested in getting some text from the page, which we can then use to verify the tickets we purchased have the correct values.

The getter functions here return the text content of 3 paragraph (or `<p>`) elements:

```java
public String getEmail() {
    return automatedBrowser.getTextFromElement(EMAIL_ADDRESS, WAIT_TIME);
}

public String getEvent() {
    return automatedBrowser.getTextFromElement(EVENT_NAME, WAIT_TIME);
}

public String getVenue() {
    return automatedBrowser.getTextFromElement(VENUE_NAME, WAIT_TIME);
}
```

Let’s now take a look at the test method that uses the POM classes:

```java
@Test
public void purchaseTicketsPageObjectModel() {

    final AutomatedBrowser automatedBrowser =
            AUTOMATED_BROWSER_FACTORY.getAutomatedBrowser("ChromeNoImplicitWait");

    try {

        automatedBrowser.init();

        final EventsPage eventsPage = new MainPage(automatedBrowser)
                .openPage()
                .buyTickets();

        final VenuePage venuePage = eventsPage
                .selectEvent("Concert", "Rock concert of the decade");

        final CheckoutPage checkoutPage = venuePage
                .selectVenue("Toronto : Roy Thomson Hall")
                .book();

        final ConfirmationPage confirmationPage = checkoutPage
                .buySectionTickets("A - Premier platinum reserve", 2)
                .checkout("email@example.org");

        Assert.assertTrue(confirmationPage.getEmail().contains("email@example.org"));
        Assert.assertTrue(confirmationPage.getEvent().contains("Rock concert of the decade"));
        Assert.assertTrue(confirmationPage.getVenue().contains("Roy Thomson Hall"));

    } finally {
        automatedBrowser.destroy();
    }
}
```

The code to initialize the `AutomatedBrowser` instance remains the same as our previous test:

```java
@Test
public void purchaseTicketsPageObjectModel() {
  final AutomatedBrowser automatedBrowser =
  AUTOMATED_BROWSER_FACTORY.getAutomatedBrowser("ChromeNoImplicitWait");

  try {
    automatedBrowser.init();
```

Our test then starts with the main page, which is now represented by the `MainPage` class. We start by creating a new instance of the `MainPage` class, and then chain calls to the `openPage()` and `buyTickets()` methods.

An instance of the `EventsPage` class is returned by the `buyTickets()` method. We save this value in a variable called `eventsPage`.

Notice that at no point in this code did we make any reference to the URL that was used to open the page, or the locators that were used to click the `Buy Tickets Now` link. These details are now handled by the POM classes, freeing up the test code from any specific knowledge of how the web application works:

```java
final EventsPage eventsPage = new MainPage(automatedBrowser)
  .openPage()
  .buyTickets();
```

Navigating the venue, checkout and confirmation pages follows the same pattern. The only values that need to be defined in the test are the names of the concerts, venues, sections, the email address, and the number of tickets to buy. At no point are we defining locators or making distinctions between links and form buttons:

```java
final VenuePage venuePage = eventsPage
  .selectEvent("Concert", "Rock concert of the decade");

final CheckoutPage checkoutPage = venuePage
  .selectVenue("Toronto : Roy Thomson Hall")
  .book();

final ConfirmationPage confirmationPage = checkoutPage
  .buySectionTickets("A - Premier platinum reserve", 2)
  .checkout("email@example.org");
```

Validating the details of the purchased tickets is also now much more streamlined. The `ConfirmationPage` class exposes the values we’re interested in through getter methods, and the test code no longer has to deal with the obtuse CSS selectors required to find the paragraph elements that hold this information:

```java
Assert.assertTrue(confirmationPage.getEmail().contains("email@example.org"));
Assert.assertTrue(confirmationPage.getEvent().contains("Rock concert of the decade"));
Assert.assertTrue(confirmationPage.getVenue().contains("Roy Thomson Hall"));
```

Once the test is completed, we clean up the resources in the `finally` block:

```java
} finally {
  automatedBrowser.destroy();
}
```

By using the POM design pattern we have made our test much more readable and abstracted away many of the details required to interact with pages like URL or locators, allowing tests to be written against a descriptive and fluent API.

This post is part of a series about [creating a Selenium WebDriver test framework](../0-toc/webdriver-toc.md).
