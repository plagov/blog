---
title: "Running Multiple Parallel Requests With RestAssured and TestNG"
date: 2018-09-13T21:13:42+02:00
draft: false
---
RestAssured can be used not only for sending HTTP requests and asserting response body and headers. It also can 
measure the response time and assert it against a condition you set. In this blog post, I describe the way how to 
send multiple parallel requests with RestAssured and a data provider from TestNG.
<!--more-->

# Introduction

Consider the following scenario. You have to execute the same test method with multiple test data values against a 
REST API endpoint. Moreover, you have to run all of them in parallel at the same time. A real-world scenario could be 
to check how a service under test will respond to multiple requests sent simultaneously, to test the response time, 
for example.

For REST API testing, a common tool in Java world is the [RestAssured](http://rest-assured.io/) library. I assume 
that you have already some experience with this library (if not, check its 
[getting started](https://github.com/rest-assured/rest-assured/wiki/GettingStarted) guide and 
[documentation](https://github.com/rest-assured/rest-assured/wiki/Usage)), so let's set up first request specification.

## RestAssured specification configuration

In this presentation I will solve the following task: having a list of certain codes/IDs, I need to send the same 
POST request towards a service endpoint each with its code/ID from the list.

Let's first set up RestAssured specification configuration. I want to reuse the same configuration for all test 
methods in my class, so I'll define first the specification as a class field and will put the builder configuration 
in `@BeforeMethod` of TestNG.

```java
RequestSpecification specification;

@BeforeMethod
public void setUpSpecification(){
  RequestSpecBuilder builder = new RequestSpecBuilder();
  builder.addQueryParam(parameterName, parameterValue); // define common parameters
  builder.setContentType(ContentType.JSON);
  specification = builder.build();
}
```

So, each of the tests in my test class will reuse the same specification.

## Using TestNG DataProvider

Since the task assumes that I will use the same test method with different data values, TestNG DataProvider feature 
will help to make this easy to set up without code duplication. What `DataProvider` is - it is a method that supplies 
test method with its values one by one so that test will continue to execute as long as `DataProvider` will continue 
to supply its values. You can hard code values in `DataProvider` method, or you can define an implementation of 
getting test values from another source (file, database or, another method or class).
`DataProvider` method should return an array of type `Object`, either a one-dimensional or multi-dimensional.

For simplicity, I will use a one-dimensional array with hard-coded test values.

```java
@DataProvider(parallel = true)
public Object[] getId() {
  return new Object[] {"KD3856", "DK8937", "EF7301"};
}
```

`@DataProvider` annotation accepts several arguments as configurations. One of them is for example `name` which will 
set a custom name for your DataProvider. If this argument is omitted, the method's name will be taken as a name.

Another argument is boolean `parallel`, which is set to `false` by default. Here's what happens when you set the 
value for this argument to `true`.

DataProvider will supply its test data values to test method in parallel, each in its own thread. Thus, the test 
method will be executed also in parallel, which will significantly improve the overall test execution time. If we 
would leave the default value for the `parallel` argument, then the test will be executed one by one.

## Set up a test method

Once DataProvider is set up, now it's time to set up test method and connect it to a DataProvider. Here's the whole 
test method that sends a POST request.

```java
@Test(dataProvider = "getId")
 public void responseTimeOfEndpoint(String id) {
   Map<String, String> map = new HashMap<>();
   map.put("id", id);

  given()
    .spec(specification)
    .body(map)
  .when()
    .log()
    .all()
    .post(ENDPOINT_ADDRESS)
  .then()
    .time(lessThan(2000L))
    .and()
    .statusCode(200);
}
```

Let's describe this test method in detail.

TestNG `@Test` annotation accepts argument `dataProvider` that points to a name of a DataProvider that we want to 
use. We set DataProvider's method name since we didn't specify a custom name for a DataProvider.

In order for the test method to get a test value from a DataProvider, test method should accept arguments of the same 
type as DataProvider's test value is. In our example, DataProvider returns an array of strings. Thus, the test method 
should accept an argument of a String type. If we would have a multi-dimensional String array in DataProvider, then 
the test method would accept two String arguments.

When we need to use the test data value, then we use this argument variable. So we did in `map.put("id", id);` 
expression when creating a body for a POST request.

Next goes a pretty common RestAssured DSL for a POST method that asserts that a response is received within 2 seconds 
and its status is 200 OK.

## Set up a number of threads for a DataProvider

When you set a `parallel` argument of `@DataProvider` annotation to `true` you also need to specify the number of 
parallel threads to be open.

That is done in two ways:

* as a command line argument passed to Maven
* as a configuration of Maven Surefire Plugin

Let's describe both options.

### Number of parallel threads passed as a command line argument

Just include the following parameter in your Maven command with the number of threads you need to open: 
`-Ddataproviderthreadcount=30`.

To run tests within IntelliJ IDEA, use the same syntax in "VM options" field of TestNG Run Configuration.

### Number of parallel threads as a configuration of Maven Surefire Plugin

Another option of setting up the number of threads is to specify this configuration in Maven Surefire Plugin if it is 
used it your project. This way you keep your configuration in `pom.xml` and don't add yet another argument to your 
pipeline command.

The configuration is pretty simple. Set the `dataproviderthreadcount` in property name and number of threads in 
property value. Find below the full plugin declaration:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.22.0</version>
  <configuration>
    <properties>
      <property>
        <name>dataproviderthreadcount</name>
        <value>30</value>
      </property>
    </properties>
  </configuration>
</plugin>
```

## Conclusion

In this blog post, I showed the way how you can easily set up TestNG and RestAssured to run the same test in parallel 
using a set of test data values. So this technique can be used for a sort of load testing or performance testing of 
your REST endpoints.

Just one drawback in the current implementation of parallel DataProvider execution is that you set the number of 
threads globally per project and you can't control it per test method or per test class.

In case if the number of the test data values in DataProvider is less than the number of threads set, TestNG will 
open the number of the threads that is equal to a number of test data values available in DataProvider.
