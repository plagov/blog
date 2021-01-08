---
title: "Test automation with Cucumber"
date: 2020-01-08T21:18:54+02:00
draft: true
---
In this blog post I will describe and show practical examples on how to start a test automation project with Cucumber
and Kotlin. [Cucumber](https://cucumber.io/) is a great tool to use when starting an end-to-end test automation for a
project with a complex business logic. Here I will show several important technical things that should be done to
have everything working properly.
<!--more-->

# Scenarios to cover

In this blog post I will cover two scenarios. I will use [Goodreads](https://www.goodreads.com/) web-site for this
purpose. First, I will search for Ernest Hemingway using the search field on the home page and will select a random
book from first page of search results. Then I will assert that this book is Hemingway's book. Thus, the test will
check that the search returns correct results.

Second scenario will check "b categories". Under the search box, there's a section with books' genres. I will
select a random genre, open it and on the next page will select a random book under the "New Releases" section. Next,
on the book's page I will check that book's genres include the genre I've selected on the homepage.

# Declaring required dependencies

For this project I decided to use Kotlin. I will take Cucumber for Java as a testing framework and Selenide to work
with the browser. So, let's create a Gradle project with Kotlin DSL and declare above mentioned dependencies.

```kotlin
plugins {
  id("org.jetbrains.kotlin.jvm") version "1.4.10"
}

val cucumberVersion = "6.7.0"

dependencies {
  implementation(platform("org.jetbrains.kotlin:kotlin-bom"))
  implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")

  implementation("io.cucumber:cucumber-java:$cucumberVersion")
  implementation("io.cucumber:cucumber-junit:$cucumberVersion")
  implementation("io.cucumber:cucumber-guice:$cucumberVersion")

  implementation("com.codeborne:selenide:5.15.1")
}
```
