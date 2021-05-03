---
title: "Load Testing With Gatling"
date: 2019-03-12T21:18:54+02:00
draft: false
---
In the [previous blog post]({{< ref "2018-09-13-running-multiple-parallel-requests-with-restassured-and-testng.md" >}})
I've described how to use Dataprovider of TestNG to assert the response time of the REST API endpoint when sending 
multiple parallel requests. In this blog post, I will describe how to do the same with Gatling, which is a more 
suitable tool for such kind of task.
<!--more-->

# About Gatling

[Gatling](https://gatling.io/) is an open-source tool that allows you to make load testing of web applications. It 
provides a flexible DSL that makes possible to emulate real production user scenarios. For example, consider the 
following scenario: 100 users are sending requests to the service for 3 minutes, then the number of users decreases 
and stays for the next 2 minutes and then again increases for 5 minutes. The load that users put on the service is 
not constant, it is floating. This is a very common scenario, there are peaks and lows. Thus, just throwing a fixed 
number of requests to the service at the same time during a certain period of time - this is not how real production 
scenario looks like.

Gatling runs on JVM and is written in Scala. But there's no need in Scala programming experience to write tests. The 
DSL that Gatling provides is self-expressive.

# Initialize project with Gatling and Maven

Gatling has good integration with build tools like SBT and Maven. Though SBT is a popular choice for Scala, I have 
more experience with Maven and since there's a plugin developed by Gatling developers, I decided to go with the tool 
which I know better.

Let's create a new Maven project. To add Gatling to the project, add the following dependency:

```xml
<dependencies>
  <dependency>
    <groupId>io.gatling.highcharts</groupId>
    <artifactId>gatling-charts-highcharts</artifactId>
    <version>3.5.1</version>
  </dependency>
</dependencies>
```

Since we're using Maven, a `gatling-maven-plugin` should be added. Put this block inside the `build` section of 
`pom.xml` file.

```xml
<plugin>
  <groupId>io.gatling</groupId>
  <artifactId>gatling-maven-plugin</artifactId>
  <version>3.1.2</version>
</plugin>
```

And the final thing, to be able to run Scala with Maven, add `scala-maven-plugin` plugin.

```xml
<plugin>
  <groupId>net.alchim31.maven</groupId>
  <artifactId>scala-maven-plugin</artifactId>
  <version>4.5.1</version>
</plugin>
```

# Define service under test and testing scenario

Now, when all needed dependencies are set up. Next, we need to define the testing scenario.

For this demo, I will use one of the endpoints provided by [Postman Echo](https://docs.postman-echo.com/) service 
which is a free service that provides HTTP access for testing and demo purposes.

The scenario that we're going to emulate will be as follows: during the period of 2 minutes, we will send 200 
requests that will be distributed evenly during that period, next we will increase the load to 300 users during one 
minute and finally will finish the scenario with 200 users during one minute. And we will set an assert that the 
response time of each request should be no more than 800 ms and the status code of the response should be equal to 
`200 OK`.

This scenario is used just for demo purposes. In a real project, before starting to automate the scenario, analyze 
the potential load that is expected on your service and find out what scenario will suit your case.

Create a class and make this class to extend the Gatling's `Simulation` class.
That is mandatory for all classes that are expected to run the scenario.

Next, define some constants, like the base URL, query parameters and protocol
configuration.

```scala
private val url = "https://postman-echo.com/get"
private val queryParams = Map("foo1" -> "bar1", "foo2" -> "bar2")

private def httpConfBuilder(envUrl: String): HttpProtocolBuilder = {
  http
    .baseUrl(envUrl)
    .acceptHeader("application/json")
}
```

After some constants are set up, next define the scenario itself that will
represent the actual business logic. Define here all the query and path params
that are accepted by the service, request body and asserts (for response time,
for example).

```scala
private val sendGetRequest = scenario("Get date from endpoint")
  .exec(http("GET method")
    .get("/")
    .queryParamMap(queryParams)
    .check(status is 200, responseTimeInMillis lte 800))
```

In this example, I'm sending a GET request to the particular endpoint of the
service (root in the example below). Define your custom name for the whole
scenario in `scenario` method, next, set the name of the particular request in
`exec` method and all the configuration of the request. Since the business logic
of a real project might assume sending different type of requests to different
endpoints with different asserts, it is possible to chain multiple requests
inside the one scenario.

Next, after scenario is defined, the set up should be done. The set up of the
scenario is the actual logic of how many request should be sent, how much time
the scenario should be executed, how the user load should be spread during the
execution. All of that is done using the `setUp` method.

```scala
setUp(sendGetRequest.inject(
  rampUsers(200) during(2 minutes),
  rampUsers(300) during (2 minutes),
  rampUsers(200) during(1 minutes)
)).protocols(httpConfBuilder(url))
```

Pass the scenario name as a method argument and define the load logic inside the
`inject` method. When finish with defining the load logic, set the HTTP
configration to be used during the execution.

That's it with the basic GET request with Gaatling. Run the following command
to start the scenario execution.

```bash
mvn gatling:test
```

Upon the finish, Gatling will generate the HTML report with statistics and
graphics under `target/gatling/${class_name}-${timestamp_of_execution}`.

# Parameterized scenarios with POST request

In the previous section, I've described the simple scenario with the GET
request. In this section I'm going to describe the scenario for the
parameterized POST request.

Let's first define a service under test and HTTP configuration. As an endpoint
I will use another service from Postman.

```scala
private val url = "https://postman-echo.com/post"

private def httpConfBuilder(envUrl: String): HttpProtocolBuilder = {
  http
    .baseUrl(envUrl)
    .acceptHeader("application/json")
}
```

I have a CSV file with a column that contains random ID numbers that will be
used as a parameter to a test scenario. Gatling has a `feeder` feature that
feeds the test scenario with values from a defined feeder. There are available
several built-in feeders, check
[official documentation](https://gatling.io/docs/3.0/session/feeder/) for more
details.

To define a feeder for the CSV file, just use a `csv` method from Gatling core
and provide a filename, located under `src/main/resources` or
`src/test/resources`:

```scala
val feeder = csv("random_strigs.csv")
```

Given the CSV file has a column header, then we can refer to this header to
use values from this column. In the next example, I will use values from a CSV
file to insert them into the query parameter map. Each request will be sent
with its own value taken from the feeder.

```scala
private val sendPostRequest = scenario("Send POST request to endpoint")
  .feed(feeder)
  .exec(
    http("POST method")
      .post(url)
      .queryParamMap(Map("id" -> "${id}"))
      .check(status is 200, responseTimeInMillis lte 800))
```

`"${id}"` in the query map points to the column header in CSV file. That will
take a value from the feeder and compose a URL of the following format:
`/post?id=781f5efc84`.

As in the previous example, we will check for status code 200 and that the
response time will be less than or equal to 800 ms.

And the set up of the scenario will be the same as with the GET method test.

Run `mvn gatling:test` command and observe result upon completion.

# Run multiple scenarios within one project

It is very often when you have multiple scenarios within the same project and
you want them to be executed one-by-one on a CI server. By default, Gatling
doesn't allow more than one `setUp` method per project. But that is configurable
in `gatling-maven-plugin`.

Add the following configuration to the maven plugin:

```xml
<configuration>
  <runMultipleSimulations>true</runMultipleSimulations>
</configuration>
```

Run the `mvn gatling:test` command. That will execute all tests in the
project sorted by alphabetic order.

# Conclusion

In this blog post, I showed how quickly and easily you can create your first
test for a load testing with Gatling.

All code examples for this blog post can be found over on
[GitHub](https://github.com/plagov/gatling-blogpost-demo).
