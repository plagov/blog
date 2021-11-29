+++
title = "How I learned about the rule engine pattern with Java"
date = "2021-10-11"
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
