---
title: Open Liberty
---

### Prerequisites

* Connection to the internet (docker.io, github.com, maven repo, npm registry)
* [Docker for Desktop](https://www.docker.com/products/docker-desktop)
* [Docker-Compose](https://docs.docker.com/compose/install)
* Code Editor - We recommend [VSCode](https://code.visualstudio.com)

### General Instructions

1. Clone the git repository

    ```
    git clone https://github.com/ibm-cloud-architecture/learning-distributed-tracing-101.git
    ```

2. Change to the lab directory

    ```
    cd lab-jaeger-ol
    ```

<InlineNotification>

**NOTE:** The solution to the lab is located [here](href:/lab-jaeger-ol/solution).

</InlineNotification>

<InlineNotification>

**NOTE:** `docker-compose` is already configured to run `Jaeger`.

In case that stopping docker-compose doesn't work with `Ctrl-C`, you can stop it using `docker-compose stop`

</InlineNotification>

## Test your environment

1. Change directory to the lab:

    ```
    cd lab-jaeger-ol
    ```


2. Test the service without tracing enable

    ```
    docker-compose build
    docker-compose up
    ```

3. Try the service

    ```
    curl http://localhost:9080/sayHello/Carlos
    Hello Carlos!
    ```

4. End the service

Use `Ctrl-C` on the window where you started `docker-compose` or use `docker-compose stop` .


## Add client libraries

1. Add the tracing client library for the java `service-a`, editing the file `service-a/pom.xml` and adding the dependency for `jaeger-client`. The `pom.xml` section should look like the following:

    ```
    <dependency>
        <groupId>io.jaegertracing</groupId>
        <artifactId>jaeger-client</artifactId>
        <version>0.34.0</version>
    </dependency>
    ```

## Tracing every HTTP request

1. Inspect the following environment variables already added to the `services->service-a->environment` and `services->service-b->environment` blocks of the Docker Composer file `docker-compose.yaml`:

    ```
    JAEGER_REPORTER_LOG_SPANS: "true"
    JAEGER_SAMPLER_TYPE: const
    JAEGER_SAMPLER_PARAM: 1
    ```

2. Add the `<feature>mpOpenTracing-1.3</feature>` element to the `<featureManager>` block of the Open Liberty server configuration file `service-a/src/main/liberty/config/server.xml`. The `<featureManager>` element should look exactly like this:

    ```
        <featureManager>
            <feature>microProfile-3.2</feature>
            <feature>mpOpenTracing-1.3</feature>
        </featureManager>
    ```

3. Modify the `<webApplication>` element of the Open Liberty server configuration file `service-a/src/main/liberty/config/server.xml` to instruct Open Liberty to use the OpenTracing dependencies included in the application WAR file. The `<webApplication>` element should look exactly like this:

    ```
    <webApplication location="service-a.war" contextRoot="/" >
            <classloader apiTypeVisibility="+third-party" />
        </webApplication>
    ```


4. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

    ```
    docker-compose build
    docker-compose up
    ```

5. Call the same API endpoint, but now is instrumented with tracing

    ```
    curl http://localhost:9080/sayHello/Carlos
    Hello Carlos!
    ```

6. Open the Jaeger UI using the web browser

    ```
    open http://localhost:16686/jaeger
    ```

7. Select the Service `service-a` from the drop-down options and click `Find Traces`

![find-trace](../../images/ol-service-a-find-trace.png)

8. Click on one of the traces, then expand the trace's `Tags` and `Process`. You should see information about the HTTP request such as `http.method` set to `GET` and `http.status_code` set to `200`. The `Process` section shows information about the process generating the entry, such as its hostname, IP address, and the version of the Jaeger client library. 

![trace-details](../../images/ol-service-a-trace-details.png)

9. Force an error in the service by calling the `/error` endpoint.

    ```
    curl http://localhost:9080/error -I -s | head -n 1
    HTTP/1.1 500 Internal Server Error
    ```


10. Click `Find Traces` now it should show a trace with the error endpoint.

![serv-err](../../images/ol-service-a-error.png)


11. Click on the trace with the `/error`, then expand the trace's `Tags` and `Process`. You should see information about the trace such as the `http.status_code` set to `500`.

![err-details](../../images/ol-service-a-error-details.png)

## Finding slow HTTP requests

In the `service-a` we have the API endpoint `/sayHello`, we used this endpoint in the previous section but called it only once. This endpoint has some strange behavior that not all responses are fast, very often the response is slower than 100ms.

1. Stop docker-compose with `Ctrl+C` and start it again.

    ```
    docker-compose up
    ```

2. Run the following code to call the API multiple times or open the URL endpoint \http://localhost:9080/sayHello/Carlos on the web browser and click refresh multiple times.

    ```
    for i in {1..20}
    do 
    curl http://localhost:9080/sayHello/Carlos -I -s | head -n 1
    sleep 1
    done
    ```

3. Open the Jaeger UI using the web browser

    ```
    open http://localhost:16686/jaeger
    ```

4. Select the Service `service-a` from the drop-down options and click `Find Traces`

![slow-a](../../images/ol-service-a-slow.png)

In the picture above, you can see a timeline graph with each trace represented with a circle, in this case, we have 20 traces in the result set when we clicked `Find Traces`.
Note that the span names are defined as `<HTTP method>:<@Path value of endpoint’s class>/<@Path value of endpoint’s method>`, which is defined in the property `mp.opentracing.server.operation-name-provider` of the `liberty-maven-plugin` plugin. The other alternatives are listed in the [Eclipse MicroProfile OpenTracing specification](https://github.com/eclipse/microprofile-opentracing/blob/master/spec/src/main/asciidoc/microprofile-opentracing.asciidoc#server-span-name).
Some traces are taking approximately 100ms and others are taking approximately 2ms.
You can see the pattern that only every 3rd request the response is slow.
When troubleshooting we are interested first on the slowest requests, you can click on one of the traces on the graph, or you can sort in the table by `Longest First`.

5. Select the trace that took the longest time 103ms, expand all the information for the single-span operation `/sayHello` including tags and logs.

![slow-details](../../images/ol-service-a-slow-details.png)

6. The handler has a sleep step in the method `sayHello` that delays the response every 3rd request. Open the file `src/main/java/com/example/servicea/HelloController.java` and locate the culprit code.

    ```
    // simulate a slow request every 3 requests
    try {
        if (counter++ % 3 == 0) {
            Thread.sleep(100);
        }
    } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    ```

7. Remove the `try/catch` block and save the file `HelloController.java`.

8. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

    ```
    docker-compose build
    docker-compose up
    ```

9. Run again the following code to call the API multiple times or open the URL endpoint \http://localhost:9080/sayHello/Carlos on the web browser and click refresh multiple times.

    ```
    for i in {1..20}
    do 
    curl http://localhost:9080/sayHello/Carlos -I -s | head -n 1
    sleep 1
    done
    ```

10. Open the Jaeger UI using the web browser

    ```
    open http://localhost:16686/jaeger
    ```

11. Select the Service `service-a` from the drop-down options and click `Find Traces`

![serv-fast](../../images/ol-service-a-fast.png)

You can see now that all HTTP requests are fast and the problem is fixed

Cloud Native applications can be composed of microservices and each microservice handling multiple endpoints. Having the ability to have observability allows you to narrow down to a specific service and within that service a specific endpoint having problems, starting with a single trace and span you can increase the observability of your applications.

## Tracing an HTTP handler

In the previous example, we were able to identify the endpoint `/sayHello` as one of interest in our service. Let's see how can we add tracing instrumentation to the function that is handling this endpoint.

1. Add the following imports at the top of the file `HelloController.java`

    ```
    import java.util.LinkedHashMap;
    import java.util.Map;

    import javax.inject.Inject;

    import io.opentracing.Scope;
    import io.opentracing.Span;
    import io.opentracing.Tracer;
    ```

2. In the class `HelloController` add the following Autowire to have access to the global tracer

    ```
        @Inject
        private Tracer tracer;
    ```

3. Locate the method `sayHello` and wrap the code in a try with a scope, this will create a new child span.

    ```
    public String sayHello(@PathParam("name") String name) {
        try (Scope scope = tracer.buildSpan("say-hello-handler").startActive(true)) {
            String response = formatGreeting(name);
            return response;
        }
    }
    ```

4. Get a reference to the new child span `say-hello-handler` using the method `scope.span()`

    ```
    public String sayHello(@PathParam("name") String name) {
        try (Scope scope = tracer.buildSpan("say-hello-handler").startActive(true)) {
            Span span = scope.span();
            String response = formatGreeting(name);
            return response;
        }
    }
    ```

5. The OpenTracing API supports the method `log` you can log an event with a name and an object. Add a log to the span with a message that contains the value of the name.

    ```
    public String sayHello(@PathParam("name") String name) {
        try (Scope scope = tracer.buildSpan("say-hello-handler").startActive(true)) {
            Span span = scope.span();
            Map<String, String> fields = new LinkedHashMap<>();
            fields.put("event", name);
            fields.put("message", "this is a log message for name " + name);
            span.log(fields);
            // you can also log a string instead of a map, key=event value=<stringvalue>
            // span.log("this is a log message for name " + name);
            String response = formatGreeting(name);
            return response;
        }
    }
    ```

6. The OpenTracing API supports the method `setTag` you can tag the span with a key and any value. Add a tag that contains the response, in normal use cases you would not log the entire response and instead key values that are useful for later searching for spans. Since we are initiating the scope within a `try` statement there is no need to call explicit `span.finish()`. 

    ```
    public String sayHello(@PathParam("name") String name) {
        try (Scope scope = tracer.buildSpan("say-hello-handler").startActive(true)) {
            Span span = scope.span();
            Map<String, String> fields = new LinkedHashMap<>();
            fields.put("event", name);
            fields.put("message", "this is a log message for name " + name);
            span.log(fields);
            // you can also log a string instead of a map, key=event value=<stringvalue>
            // span.log("this is a log message for name " + name);
            String response = formatGreeting(name);
            span.setTag("response", response);
            return response;
        }
    }
    ```

7. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

    ```
    docker-compose build
    docker-compose up
    ```


8. Call the API endpoint.

    ```
    curl http://localhost:9080/sayHello/Carlos
    Hello Carlos!
    ```


9. Open the Jaeger UI using the web browser

    ```
    open http://localhost:16686/jaeger
    ```

10. Select the Service `service-a` from the drop-down options and click `Find Traces`

![2-span](../../images/ol-service-a-2-spans.png)

Notice in the result items table, for the trace item that the trace indicates that there are a total of two spans `2 Spans` and that service-a contains two spans `service-a (2)`

11. Click the trace, expand the spans `say-hello`, and then expand the `Tags` and `Logs` sections.

![span-details](../../images/ol-service-a-2-span-details.png)

Notice in the Tags section the tag is located with key `name` and the string value `Hello Carlos!`.
Notice in the Logs section the log event with the name `name` and the message `this is a log message for name Carlos`

## Tracing a function

The HTTP handler usually calls other functions to perform the business logic, when calling another function within the same service you can create a child span.

1. The `sayHello` handler calls the function `formatGreeting` to process the input `name`. In the method `formatGreeting` create a new span using `tracer.buildSpan` and name the span `format-greeting`. 

    ```
    private String formatGreeting(String name) {
        try (Scope scope = tracer.buildSpan("format-greeting").startActive(true)) {
            Span span = scope.span();
            span.log("formatting message locally for name " + name);
            String response = "Hello " + name + "!";
            return response;
        }
    }
    ```

2. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

    ```
    docker-compose build
    docker-compose up
    ```

3. Call the API endpoint.

    ```
    curl http://localhost:9080/sayHello/Carlos
    Hello Carlos!
    ```

4. Open the Jaeger UI using the web browser

    ```
    open http://localhost:16686/jaeger
    ```

5. Select the Service `service-a` from the drop-down options and click `Find Traces`

![3-span](../../images/ol-service-a-3-spans.png)

Notice that the trace now contains three spans.

6. Click the trace, expand the spans `say-hello` and `format-greeting`, and then expand the `Logs` sections.

![span-form](../../images/ol-service-a-span-formatter.png)

Notice the cascading effect between the three spans, the span `format-greeting` contains the message `formatting message locally for name Carlos` that we instrumented.

## Distributing Tracing

You can have a single trace that goes across multiple services, this allows you to distribute tracing and better observability on the interactions between services.

In the previous example, we instrumented a single service `service-a`, and created a span when calling a local function to format the greeting message.

For the following example, we are going to use a remote service `service-b` to format the message, and returning the formatted greeting message to the HTTP client.

1. In the file `HelloController.java` locate the handler function `sayHello` and replace the function call `formatGreeting(name)` with `formatGreetingRemote(name)`.

    ```
    public String sayHello(@PathParam("name") String name) {
        try (Scope scope = tracer.buildSpan("say-hello-handler").startActive(true)) {
            Span span = scope.span();
            Map<String, String> fields = new LinkedHashMap<>();
            fields.put("event", name);
            fields.put("message", "this is a log message for name " + name);
            span.log(fields);
            String response = formatGreetingRemote(name);
            span.setTag("response", response);
            return response;
        }
    }
    ```

2. In the method `formatGreetingRemote` the HTTP request is automatically instrumented, and the tracing headers inserted when calling the remote service `service-b` endpoint `/formatGreeting`.

    ```
    private String formatGreetingRemote(String name) {
        String serviceName = System.getenv("SERVICE_FORMATTER");
        if (serviceName == null) {
            serviceName = "localhost";
        }
        String urlPath = "http://" + serviceName + ":9081/formatGreeting";
        URI uri = UriBuilder //
                .fromPath(urlPath)
                .queryParam("name", name).build();
        Client client = ClientBuilder.newClient();
        String responseStr = null;
        try {
            Response response = client.target(uri.toASCIIString()).request().get();
            responseStr = response.readEntity(String.class);
        } finally {
            client.close();
        }
        return responseStr;
    }
    ```

3. The service `service-b` is already instrumented to trace every HTTP request using the same procedure <<tracing-every-http-request, Trace every HTTP request>> that we did for service `service-a`.

4. Import at the top of the file `service-b/src/main/java/com/example/serviceb/FormatController.java` the OpenTracing libraries.

    ```
    import javax.inject.Inject;

    import io.opentracing.Scope;
    import io.opentracing.Span;
    import io.opentracing.Tracer;
    ```

5. In the class `FormatController` add the following Autowire to have access to the global tracer

    ```
        @Inject
        private Tracer tracer;
    ```

6. Located the HTTP handler function `formatGreeting` in the file `FormatController.java`

    ```
    public String formatGreeting(@QueryParam("name") String name) {
        String response = "Hello, from service-b " + name + "!";
        return response;
    }
    ```

7. Create a new child span using the parent span located in the `req` object as context.
This will allow the trace to have an additional child span. Use the function `tracer.startSpan` and name the span `format-greeting`.

    ```
    public String formatGreeting(@QueryParam("name") String name) {
        try (Scope scope = tracer.buildSpan("format-greeting").startActive(true)) {
            Span span = scope.span();
            String response = "Hello, from service-b " + name + "!";
            return response;
        }
    }
    ```

8. Add a log event to the new span using the method `span.log`.

    ```
    public String formatGreeting(@QueryParam("name") String name) {
        try (Scope scope = tracer.buildSpan("format-greeting").startActive(true)) {
            Span span = scope.span();
            span.log("formatting message remotely for name " + name);
            String response = "Hello, from service-b " + name + "!";
            return response;
        }
    }
    ```


9. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

    ```
    docker-compose build
    docker-compose up
    ```


10. Call the API endpoint.

    ```
    curl http://localhost:9080/sayHello/Carlos
    Hello Carlos!
    ```


11. Open the Jaeger UI using the web browser

    ```
    open http://localhost:16686/jaeger
    ```

12. Select the Service `service-a` from the drop-down options and click `Find Traces`

![b-trace](../../images/ol-services-b-trace.png)


Notice that the trace contains a total of four spans `5 Spans` two for `service-a(3)` and two for `service-b(2)`

13. Click the trace to drill down to get more details.

![b-span](../../images/ol-services-b-spans.png)

Notice in the top section, the summary which includes the `Trace Start`, `Duration: 117.81ms`, `Services: 2`, `Depth: 5` and `Total Spans: 5`.

Notice the bottom section on how the total duration of 117.81ms is broken down per span, and at which time each span started and ended. You can see that the time spent in `service-b` was 3.36ms, meaning that for this single HTTP request `service-a` spent the difference (117.81-3.36) 114.45ms and `service-b` spent 3.36ms.

14. Expand the `Logs` sections for both spans `say-hello` from `service-a` and  `format-greeting` from `service-b`.

![b-logs](../../images/ol-services-b-logs.png)


Notice on the right side, each span has a summary each with the associated `Service`, `Duration`, and `Start Time`. The `Start Time` of a span marks the end time from the previous span.

Notice the time for the first log message `this is a log message for name Carlos` in `service-a` is of 1ms, this means this log event happened 1ms after the trace started.

Notice the time for the second log message `formatting message remotely for name Carlos` in `service-b` is of 93ms, this means this log event happened 93ms after the trace started in `service-a`.

It is very useful to see the log events we instrumented in our endpoint handlers across services in this manner because it provides full observability of the lifecycle of the HTTP request across multiple services.

## Baggage propagation

Imagine a scenario where you want to redirect all Safari users to a specific version of a service using the `User-Agent` HTTP header. This is useful in canary deployments when a new version is rolled out for a specific subset of users. However, the header is present only at the first service. If the routing rule is for a service lower in a call graph then the header has to be propagated through all intermediate services. This is a great use-case for distributed context propagation which is a feature of many tracing systems.

Baggage items are key:value string pairs that apply to the given Span, its SpanContext, and all Spans which directly or transitively reference the local Span. That is, baggage items propagate in-band along with the trace itself.

Baggage items enable powerful functionality given a full-stack OpenTracing integration (for example, arbitrary application data from a mobile app can make it, transparently, all the way into the depths of a storage system), and with it some powerful costs: use this feature with care.

Use this feature thoughtfully and with care. Every key and value is copied into every local and remote child of the associated Span, and that can add up to a lot of network and CPU overhead.

1. Locate the HTTP handler `sayHello` in the file `HelloController.java`. Use the method `span.setBaggageItem('my-baggage', name)` before the method call `formatGreetingRemote(name)` to set the baggage with key `my-baggage` to the value of the `name` parameter.

    ```
    public String sayHello(@PathParam("name") String name) {
        try (Scope scope = tracer.buildSpan("say-hello-handler").startActive(true)) {
            Span span = scope.span();
            Map<String, String> fields = new LinkedHashMap<>();
            fields.put("event", name);
            fields.put("message", "this is a log message for name " + name);
            span.log(fields);
            span.setBaggageItem("my-baggage", name);
            String response = formatGreetingRemote(name);
            span.setTag("response", response);
            return response;
        }
    }
    ```

2. Locate the HTTP handler `formatGreeting` in the file `FormatController.java`. Use the method `span.getBaggageItem('my-baggage')` to get the value of the name parameter at `service-a`. For convenience log the value using `span.log` to see the value in the Jaeger UI.

    ```
    public String formatGreeting(@QueryParam("name") String name) {
        try (Scope scope = tracer.buildSpan("format-greeting").startActive(true)) {
            Span span = scope.span();
            span.log("formatting message remotely for name " + name);
            String response = "Hello, from service-b " + name + "!";
            String myBaggage = span.getBaggageItem("my-baggage");
            span.log("this is baggage " + myBaggage);
            return response;
        }
    }
    ```

3. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

    ```
    docker-compose build
    docker-compose up
    ```

4. Call the same API endpoint, but now is instrumented with tracing

    ```
    curl http://localhost:9080/sayHello/Carlos
    Hello Carlos!
    ```

5. Open the Jaeger UI using the web browser

    ```
    open http://localhost:16686/jaeger
    ```

6. Select the Service `service-a` from the drop-down options and click `Find Traces`. Expand the section `Logs` for the spans `say-hello` and `format-greeting`

![serv-bag](../../images/ol-service-b-baggage.png)

Notice that the baggage is set in the `service-a` with the value `Carlos` this baggage is propagated to all spans local or remote. In the `server-b` span you can see the baggage value `Carlos` is propagated.


## Searching Traces

If you have a specific trace id you can search for it by putting the trace id on the top left search box.

You can also use a tag to search for example searching traces that have a specific HTTP status code, or one of the custom tags we added to a span.

1. To search for traces using HTTP method `GET` and status code `200`, enter `http.status_code=200  http.method=GET` on the `Tags` field in the search form, and then click `Find Traces`.

![ui-search](../../images/ol-ui-search.png)

## Dependency graph


The Jaeger UI has a view for service dependencies, it shows a visual Directed acyclic graph (DAG).

1. Click the tab `Dependencies` (this tab is renamed `System Architecture` starting with Jaeger 1.17), then click the `DAG` tab.

![ui-dep](../../images/jaeger-ui-dependencies-dag-1.jpg)

Notice that the graph shows the direction with an arrow flowing from `service-a` to `service-b`. It also shows the number of traces between the services.

This is is a simple example and there is not much value for a small set of services, but when a large number of services each with multiple endpoints then the graph becomes more interesting like the following example:

![ui-dep-2](../../images/jaeger-ui-dependencies-dag-2.jpg)


## Eclipse MicroProfile annotations

The [Eclipse MicroProfile](https://projects.eclipse.org/projects/technology.microprofile) project is designed to simplify the creation of microservices and it [defines annotations](https://github.com/eclipse/microprofile-opentracing/blob/master/spec/src/main/asciidoc/microprofile-opentracing.asciidoc) that are specific for further customizing traces generated with OpenTracing libraries.