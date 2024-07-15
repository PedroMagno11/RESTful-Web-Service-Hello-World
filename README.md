# RESTful-Web-Service-Hello-World

## What you will build
You will build a service that will accept HTTP GET requests at `http://localhost:8080/greeting`.

It will respond with a JSON representation of a greeting, as the following listing shows:
```json
{
  "id": 1,
  "content":"Hello, World!"
}
```

You can customize the greeting with an optional `name` parameter in the query string, as the following listing shows:
```http request
http://localhost:8080/greeting?name=User
```

The `name` parameter value overrides the default value of `World` and is reflected in the response, as the following listing shows:
```json
{
  "id": 1,
  "content": "Hello, User!"
}
```

## What you need

- Java 17 or later
- Gradle 7.5+ or Maven 3.5+
- Your favorite IDE:
  - Intellij IDEA
  - VSCode
  - NetBeans
  - Spring Tool Suite (STS)

## Starting with Spring Initializr

1 - Navigate to [https://start.spring.io](https://start.spring.io). This service pulls in all the dependencies you need for an application and does most of the setup for you.

2 - Choose either Gradle or Maven and the language you want to use. This guide assumes that you chose Java.

3 - Click Dependencies and select Spring Web.

4 - Click Generate.

5 - Download the resulting ZIP file, which is an archive of a web application that is configured with your choices.

```
If your IDE has the Spring Initalizr integration, you can complete this process from your IDE. 
```

## Create a Resource Representation Class

Now that you have set up the project and build system, you can create your web service.

Begin the process by thinking about service interactions.

The service will handle `GET` requests for `/greeting`, optionally with a `name` parameter in the query string. The `GET` requests should return a `200 OK` response with JSON in the body that represents a greeting. It should resemble the following output: 
```json
{
  "id": 1,
  "content": "Hello, World!"
}
```
The `id` field is a unique identifier for the greeting, and `content` is the textual representation of the greeting.

To model the greeting representation, create a resource representation class. To do so, provide a Java record class for the `id` and `content` data, as the following listing (from 
`src\main\java\br\com\pedromagno\simpleRESTFulWebService\dto\Greeting.java`) shows:

```java
package br.com.pedromagno.simpleRESTFulWebService.dto;

public record Greeting(long id, String content) { }
```
## Create a Resource Controller

In Spring's approach to building RESTful web 
service, HTTP requests are handled by a controller. 
These components are identified by the `@RestController`
shown in the following listing (from `src\main\java\br\com\pedromagno\simpleRESTFulWebService\controller\GreetingController.java`) 
handles `GET` requests for `/greeting` by returning a new 
instance of the `Greeting` class:

````java
package br.com.pedromagno.simpleRESTFulWebService.controller;

import br.com.pedromagno.simpleRESTFulWebService.dto.Greeting;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import java.util.concurrent.atomic.AtomicLong;

@RestController
public class GreetingController {
    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @GetMapping("/greeting")
    public ResponseEntity<Greeting> greeting(@RequestParam(value = "name", defaultValue = "World") String name){
        Greeting greeting = new Greeting(counter.incrementAndGet(), String.format(template, name));
        return ResponseEntity.ok(greeting);
    }
}
````
This controller is concise and simple, but there is plenty going on under the hood. We break it down step by step.

The `@GetMapping` annotation ensures that HTTP GET requests to `/greeting` are mapped to the `greeting()` method.


OBS:
There are companion annotations for other HTTP verbs (e.g. `@PostMapping` for `POST`). There is also a `@RequestMapping` annotation that they all derive from, and can serve as a synonym (e.g. `@RequestMapping(method=GET)`).

`@RequestParam` binds the value of 
the query string parameter `name` 
into the `name` parameter of 
the `greeting()` method. 
If the `name` parameter is 
absent in the request, 
the `defaultValue` of `World` 
is used.

The implementation of the 
method body creates and 
returns a new `Greeting` object 
with id and content 
attributes based on the 
next value from the `counter` 
and formats the given `name` 
by using the greeting `template`.

A key difference between a 
traditional MVC controller 
and the RESTful web service 
controller shown earlier 
is the way that the HTTP 
response body is created. 
Rather than relying on a 
view technology to 
perform server-side rendering
of the greeting data to 
HTML, this RESTful web 
service controller populates 
and returns a Greeting 
object. 
The object data will be 
written directly to the 
HTTP response as JSON.

This code uses 
Spring @RestController 
annotation, which marks 
the class as a controller 
where every method 
returns a domain object 
instead of a view. 
It is shorthand for 
including both 
@Controller and 
@ResponseBody.

The Greeting object must 
be converted to JSON. 
Thanks to Spring’s HTTP 
message converter support, 
you need not do this 
conversion manually. 
Because Jackson 2 
is on the classpath, 
Spring’s MappingJackson2HttpMessageConverter 
is automatically chosen 
to convert the Greeting 
instance to JSON.

## Run the Service
The Spring Initializr 
creates an application class 
for you. In this case, you 
do not need to further 
modify the class. 
The following listing
(from src\main\java\br\com\pedromagno\simpleRESTFulWebService\SimpleRestFulWebServiceApplication.java) 
shows the application class:

```java
package br.com.pedromagno.simpleRESTFulWebService;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SimpleRestFulWebServiceApplication {
    
	public static void main(String[] args) {
		SpringApplication.run(SimpleRestFulWebServiceApplication.class, args);
	}
}
```
@SpringBootApplication is a 
convenience annotation 
that adds all of the following:

- @Configuration: Tags the class as a source of bean definitions for the application context.

- @EnableAutoConfiguration:
Tells Spring Boot to start 
adding beans based on classpath settings, other beans, and various property settings. For example, if spring-webmvc is on the classpath, this annotation flags the application as a web application and activates key behaviors, such as setting up a DispatcherServlet.

- @ComponentScan: Tells Spring to look for other components, configurations, and services in the com/example package, letting it find the 
controllers.

The main() method uses Spring Boot’s SpringApplication.run() method to launch an application. Did you notice that there was not a single line of XML? There is no web.xml file, either. This web application is 100% pure Java and you did not have to deal with configuring any plumbing or infrastructure.



## Test the Service

With the service running, visit [http://localhost:8080/greeting](http://localhost:8080/greeting), where you should see:
```json
{
  "id": 1,
  "content": "Hello, World!"
}
```

Provide a `name` query string parameter by visiting [http://localhost:8080/greeting?name=User](http://localhost:8080/greeting?name=User). Notice how the value of the `content`
attribute changes from `Hello, World!` to `Hello, User!`, as the following listing shows:
````json
{
  "id": 2,
  "content": "Hello, User!"
}
````

This change demonstrates that the `@RequestParam` arrangement in `GreetingController` is working as expected. The `name` parameter has been given a default value of `World` but can be explicitly overridden through the query string.

Notice also how the id attribute has changed from `1` to `2`. This proves that you are working against the same `GreetingController` instance across multiple requests and that its counter field is being incremented on each call as expected.