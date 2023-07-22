---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Smart BDD
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
transition: slide-left
title: Smart BDD - Less code, higher quality tests and better documentation!
---

## Smart BDD - Less code, higher quality tests and better documentation!

An introduction to Smart BDD

This is a 10 - 15min presentation followed by QA

JAMES BAYLISS

---
layout: image-right
image: './images/me.jpg'
---

James Bayliss

Software developer since 2000

Author of Smart BDD 
 - https://github.com/bit-smart-io/smart-bdd
 - mejrbayliss@gmail.com 

---

### What's the landscape and what problem is Smart BDD solving?

* Functional, integration, end-to-end etc... test codebases can become too complicated
  * Painful to maintain
  * This can lead to re-writes of re-writes etc...
* Frameworks themselves get in the way - increase code complexity and reduce productivity and coverage
* They lack diagrams, a diagram is very useful for communication

---

With Smart BDD you write the code first using best practices and this generates:
* Interactive feature files that serve as documentation
* Diagrams to better document the product

The barrier to entry is super low, you start with one annotation or add a file to resources/META-INF! That's it you're
generating documentation.

---
 
### Smart BDD interactive HTML page

![Local Image](/images/get-book.jpg)
---

### Smart BDD code that generated the interactive HTML page

```java
@ExtendWith(SmartReport.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class BookControllerIT {
    // skipped setup...
    @Order(0)
    @Test
    public void getBookBy13DigitIsbn_returnsTheCorrectBook() {
        whenGetBookByIsbnIsCalledWith(VALID_13_DIGIT_ISBN_FOR_BOOK_1);
        thenTheResponseIsEqualTo(BOOK_1);
    }

    public void whenGetBookByIsbnIsCalledWith(String isbn) {
        HttpHeaders headers = new HttpHeaders();
        headers.setAccept(singletonList(MediaType.APPLICATION_JSON));
    }

    private void thenTheResponseIsEqualTo(IsbnBook book) {
        assertThat(bookFromJson(response.getBody())).isEqualTo(book);
    }
    // skipped helper classes...
}
```

----

### Correct abstraction for the problem
* Why? Because below does it all and not good for code re-use!
* Tests should have good quality
* Measure of quality could be considered to be the ability to change

```java
@Test
public void getBookBy13DigitIsbn_returnsTheCorrectBook() {
    final IsbnBook book = new IsbnBook(VALID_13_DIGIT_ISBN_FOR_BOOK_1, "book 1 title", singletonList("book 1 author"));
    
    stubFor(get(urlEqualTo("/isbn-db/"+VALID_13_DIGIT_ISBN_FOR_BOOK_1))
        .withPort(PORT)
        .willReturn(aResponse()
        .withHeader("Content-Type","application/json")
        .withBody(bookAsString(book))));

    HttpHeaders headers = new HttpHeaders();
    headers.setAccept(singletonList(MediaType.APPLICATION_JSON));
    ResponseEntity<String> response=template.getForEntity("/book/"+VALID_13_DIGIT_ISBN_FOR_BOOK_1, String.class, headers);

    assertThat(bookFromJson(response.getBody())).isEqualTo(book);
}
```

----

### Let's take an example
* Take the latest cucumber code Cucumber - https://github.com/cucumber/cucumber-jvm/tree/main/examples/calculator-java-junit5
* Create a repo with one example calculator-java-junit5 copy and pasted it in to a new project -
https://github.com/bit-smart-io/smart-bdd-examples


---

### Cucumber example - Page 1

```gherkin
Feature: Shopping

  Scenario: Give correct change
    Given the following groceries:
      | name  | price |
      | milk  | 9     |
      | bread | 7     |
      | soap  | 5     |
    When I pay 25
    Then my change should be 4
```

----

### Cucumber example - Page 2

```java
public class ShoppingSteps {

    private final RpnCalculator calc = new RpnCalculator();

    @Given("the following groceries:")
    public void the_following_groceries(List<Grocery> groceries) {
        for (Grocery grocery : groceries) {
            calc.push(grocery.price.value);
            calc.push("+");
        }
    }

    @When("I pay {}")
    public void i_pay(int amount) {
        calc.push(amount);
        calc.push("-");
    }

    @Then("my change should be {}")
    public void my_change_should_be_(int change) {
        assertEquals(-calc.value().intValue(), change);
    }
    // omitted Grocery and Price class
}
```

----

### Cucumber example - Page 3

```java
/**
 * Work around. Surefire does not use JUnits Test Engine discovery
 * functionality. Alternatively execute the
 * org.junit.platform.console.ConsoleLauncher with the maven-antrun-plugin.
 */
@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("io/cucumber/examples/calculator")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "io.cucumber.examples.calculator")
public class RunCucumberTest {
}
```

```java
public class ParameterTypes {

    private final ObjectMapper objectMapper = new ObjectMapper();

    @DefaultParameterTransformer
    @DefaultDataTableEntryTransformer
    @DefaultDataTableCellTransformer
    public Object transformer(Object fromValue, Type toValueType) {
        return objectMapper.convertValue(fromValue, objectMapper.constructType(toValueType));
    }
}
```

----
### Smart BDD example - Page 1 of 1

```java
@ExtendWith(SmartReport.class)
public class ShoppingTest { 
  private final RpnCalculator calculator = new RpnCalculator();

  @Test
  void giveCorrectChange() {
    givenTheFollowingGroceries(item("milk", 9), item("bread", 7), item("soap", 5));
    whenIPay(25);
    myChangeShouldBe(4);
  }

  public void whenIPay(int amount) {
    calculator.push(amount);
    calculator.push("-");
  }

  public void myChangeShouldBe(int change) {
      assertThat(-shoppingService.calculatorValue().intValue()).isEqualTo(change);
  }

  public void givenTheFollowingGroceries(Grocery... groceries) {
    for (Grocery grocery : groceries) {
        calculator.push(grocery.getPrice());
        calculator.push("+");
    }
  }
  // omitted Grocery class 
}
```

----

### Cheated a little
The font size is epically big, so I put items on the same line

```
Scenario: Give correct change (PASSED)
Given the following groceries
  item "milk" 9
  item "bread" 7
  item "soap" 5
When I pay 25
My change should be 4
```

Future dev would allow item method to have `@HideMethodName` and that would yield better results
```
Scenario: Give correct change (PASSED)
Given the following groceries
  "milk" 9
  "bread" 7
  "soap" 5
When I pay 25
My change should be 4
```
----

### I'll try to demonstrate the complexity of Cucumber

Let's dive in something more advanced:

* A dollar is 2 of the currency below
* Visa payments take 1 currency processing fee

```gherkin
When I pay 25 "Dollars"
Then my change should be 29
```

It is reasonable to think that we can add this method

```java
@When("I pay {int} {string}")
public void i_pay(int amount,String currency){
  calc.push(amount*exchangeRate(currency));
  calc.push("-");
}
```

----
#### Big cup of nope!

This is the output

```text
Step failed
io.cucumber.core.runner.AmbiguousStepDefinitionsException: "I pay 25 "Dollars"" matches more than one step definition:
"I pay {int} {string}" in io.cucumber.examples.calculator.ShoppingSteps.i_pay(int,java.lang.String)
```

* Here is where the tail starts to wag the dog, you embark of investing time and more code to work around the framework
* We should always strive for simplicity, additional code and in a boarder sense additional features will always make code
harder to maintain

----
#### More pain
We have 3 options:

1. Mutate `i_pay` method to handle a currency. If we had 10's or 100's occurrences of `When I pay ..` this would be
   risky and time-consuming. If we add "Visa" payment method, we are starting to add complexity to an existing method.
2. Create a new method doesn't start with `I pay`. It could be `With currency I pay 25 "Dollars"`. Not ideal as this
   isn't really what wanted. It looses discoverability. How would we add "Visa" payment method?
3. Use multiple steps `I pay` and `with currency`. This is the most maintainable solution. For discoverability, you'd
   need a consistent naming convention. With a large codebase, good luck with discoverability, as they are loosely
   coupled in the feature file, but coupled in code.
----

### Simple solution in Smart BDD

```java
@ExtendWith(SmartReport.class)
public class ShoppingTest { 
  private final ShoppingService shoppingService = new ShoppingService();
  private final PayBuilder payBuilder = new PayBuilder();
    
  @Test
  void giveCorrectChangeWhenCurrencyIsDollars() {
    givenTheFollowingGroceries(item("milk", 9), item("bread", 7), item("soap", 5));
    whenIPay(25).withCurrency("Dollars");
    myChangeShouldBe(29);
  }

  public PayBuilder whenIPay(int amount) {
    return payBuilder.withAmount(amount);
  }
  
  public void myChangeShouldBe(int change) {
    pay();
      assertThat(-shoppingService.calculatorValue().intValue()).isEqualTo(change);
  }
  
  private void pay() {
    final Pay pay = payBuilder.build();
    shoppingService.calculatorPushWithCurrency(pay.getAmount(), pay.getCurrency());
    shoppingService.calculatorPush("-");
  }
  // builders and classes omitted
}
```

----

### Compare the complexity

Let's count the number of lines:

| Cucumber         | Lines | Smart BDD        | Lines |
|:-----------------|:------|:-----------------|:------|  
| ShoppingSteps    | 123   | ShoppingTest     | 114   |
| ParameterTypes   | 21    |                  |       |
| runCucumberTest  | 16    |                  |       |
| shopping.feature | 20    |                  |       |
| **Total**        | 180   | **Total**        | 114   |

----

### The difference in approach leads to Smart BDD

* To having less code and higher quality code
* Therefore, less complexity
* Therefore, lowering the cost of maintaining and adding testing
* Therefore, increasing productivity
* Oh, and you get sequence diagrams plus many new features are in the pipeline
----

### What's next for Smart BDD
* Better diagrams
* Ability to generate and store alongside tests
  * Markdown
  * Request/response data
* Interactive HTML
  * Rerun tests
  * Change values 
* Store and measure performance of tests

Looking for some adoption and would love to hear if anybody is interested in using.

----

That's my very quick demo, questions please

----