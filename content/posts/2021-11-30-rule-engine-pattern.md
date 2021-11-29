+++
title = "How I learned about the rule engine pattern with Java"
date = "2021-11-30"
author = "Vitali Plagov"
draft = true
+++

In this blog post I would like to describe how I learned about the rule engine pattern while contributing to the
open source project.
<!--more-->

Software developers work with 3rd party open source libraries on a daily basis. The more you use a library in your 
day-to-day work, the more familiar you become with it. In my job as a test automation engineer, I work with 
[Selenide](https://github.com/selenide/selenide) all the time. So, when I had to accomplish a certain task, I found 
out that Selenide doesn't have a solution to help me with. So, I thought it could be a good opportunity to contribute 
a new functionality into the open source library.

## Problem

Here's the problem I had. The Selenide library has a function that finds an ancestor in the HTML DOM of the current 
element. The ancestor is the same HTML element, so it has different attributes the one can use to locate it on the 
page - a tag name, a class name, an attribute, an attribute with a value. So, the task is to build an XPath 
expression for each of options.

A very straightforward solution would be to make an `if - else if - else` statement. But  with four 
options mentioned above that construct would look too sloppy. There must be a better and more clean approach of 
accomplishing this.

## Rule Engine pattern

There's indeed a better approach for such kind of problem - Rule Engine pattern.
The essence of this pattern is to split each of `if - else if - else` branches in its own rule class. Then, the main 
rule engine class will hold all the rules and will find the one that matches the client's request.

### Define a rule class

To make sure that all rule classes will implement the same method, let's define an interface that each class will
implement:

```java
public interface AncestorRule {
  Optional<AncestorResult> evaluate(String selector);
}
```

Next, let's define the first rule class. That class wil hold the logic defined in the `if - else` branch:

```java
public class AncestorWithClassRule implements AncestorRule {

  @Override
  public Optional<AncestorResult> evaluate(String selector) {
    if (isCssClass(selector)) {
      String xpath = format(
        "ancestor::*[contains(concat(' ', normalize-space(@class), ' '), ' %s ')][%s]",
        selector.substring(1)
      );
      return Optional.of(new AncestorResult(xpath));
    }
    return Optional.empty();
  }
}
```

So, here's the single logic that checks if the given selector matches the given condition - if it is a CSS class. 
The `isCssClass()` is a function defined in the supper class (not shown here for brevity reasons). If the selector 
indeed is a CSS class, then it builds an XPath expression and returns it as an Optional of a `AncestorResult`, 
otherwise an empty Optional.

This rule class is clean, short, easy to understand. It is written once and there's no need to modify it often, 
unless the business logic of the rule is updated.

We define other rules the same way. Validate if the input matches the given condition, build and return a respective 
XPath expression. Otherwise, an empty result.

### Rule result

The above code has a usage of the `AncestorResult`. The purpose of this class is to wrap the result of the 
successful evaluation. This class looks as follows:

```java
public class AncestorResult {

  private final String value;

  public AncestorResult(String value) {
    this.value = value;
  }

  public String getValue() {
    return value;
  }
}
```

Just one class field that we set via constructor and access it with a getter.

### Rule Engine class

Now, let's finally get to the class that holds the rule engine logic.

```java
public class AncestorRuleEngine {

    private static final List<AncestorRule> rules = Arrays.asList(
        new AncestorWithTagRule(),
        new AncestorWithClassRule(),
        new AncestorWithAttributeRule(),
        new AncestorWithAttributeAndValueRule()
    );

    public AncestorResult process(String selector) {
        return rules
            .stream()
            .map(rule -> rule.evaluate(selector))
            .flatMap(optional -> optional.map(Stream::of).orElseGet(Stream::empty))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("Selector does not match any rule"));
    }
}
```

First thing that is done in this class is a static list of all the rules that are applicable to this domain. If we 
have a new rule - we implement a new class and add it to this list of rules.

Second thing is the processing of the client's input across all rules. It streams the list of rules, evaluates each 
of them. The first non-empty result of the rule is being returned to the client. Otherwise, the rule engine will 
throw an exception.

A bit about the `flatMap()` operation. The previous `map()` function returns a stream of Optionals. Then, the 
`flatMap()` converts a stream of empty optionals to an empty stream. Otherwise, to a stream of non-empty 
AncestorResults encapsulated into Optional. That construct is compatible with Java 8 and looks verbose. Luckily, 
starting with Java 9, that could be simplified. Check more in this 
[Baeldung](https://www.baeldung.com/java-filter-stream-of-optional) article.

### Usage of rule engine

Now, when we have all or rules implemented, the rule engine defined, let's see how to call and use this engine.

```java
public class ClientSideThatCallsTheRuleEngine {

    public void executeClientCode() {
        // some executions

        AncestorRuleEngine ruleEngine = new AncestorRuleEngine();
        String xpath = ruleEngine.process(selector).getValue();
        
        // other executions
    }
}
```

It is as simple as that. Instantiate a rule engine. Pass in the client's input and get the result. It's clean, short 
and precise. We hide all the low-level logic of validating the input, building a respective result, processing it. 
Compare it with the straightforward approach with multiple `if - else` branches. The more logic we add, the more 
this `if - else` monster will grow.

## Conclusion

In this blog post I described an example of how to implement the rule engine pattern in Java. I'm glad I learned 
about this pattern. And I'm sure that I will have more opportunities to use this pattern.
