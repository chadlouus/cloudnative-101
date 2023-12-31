---
title: Lab 1 - Node.js
---

### Prerequisites

* Connection to the internet (docker.io, github.com, maven repo, npm registry)
* [Docker for Desktop](https://www.docker.com/products/docker-desktop)
* [Docker-Compose](https://docs.docker.com/compose/install)
* Code Editor - We recommend [VSCode](https://code.visualstudio.com)

## General Instructions

1. Clone the git repository

  ```
  git clone https://github.com/ibm-cloud-architecture/learning-distributed-tracing-101.git
  ```

2. Change to the lab directory

  ```
  cd lab-jaeger-nodejs
  ```

<InlineNotification>

**NOTE:** The solution to the lab is located [here](./solution).

</InlineNotification>

<InlineNotification>

**NOTE:** `docker-compose` is already configured to run `Jaeger`.

In case that stopping docker-compose doesn't work with `Ctrl-C`, you can stop it using `docker-compose stop`

</InlineNotification>

## Test your environment

1. Change directory to the lab

  ```
  cd lab-jaeger-nodejs
  ```

2. Test the service without tracing enable

  ```
  docker-compose build
  docker-compose up
  ```

3. Try the service

  ```
  curl http://localhost:8080/sayHello/Carlos
  Hello Carlos!
  ```

## Add client libraries

1. Add the tracing client library for the node.js `service-a`, edit the file `service-a/package.json` and add the dependency for `jaeger-client`
The json section should look like the following:

  ```
  {
  "dependencies": {
      "express": "^4.17.1",
      "jaeger-client": "^3.15.0",
      "bent": "^6.0.5"
    }
  }
  ```

## Tracing every HTTP request

1. Create a function at the end of the file `service-a/app.js` with the following code to initialize the Jaeger tracer.

  ```
  function initTracer (serviceName) {
    const initJaegerTracer = require('jaeger-client').initTracerFromEnv

    // Sampler set to const 1 to capture every request, do not do this for production
    const config = {
      serviceName: serviceName
    }
    // Only for DEV the sampler will report every span
    // Other sampler types are described here: https://www.jaegertracing.io/docs/1.7/sampling/
    config.sampler = { type: 'const', param: 1 }

    return initJaegerTracer(config)
  }
  ```

2. At line `7` add the following code to use the function `initTracer` to initialize the Tracer and set the tracer in `opentracing` .

  ```
  const tracer = initTracer(serviceName)
  const opentracing = require('opentracing')
  opentracing.initGlobalTracer(tracer)
  ```

3. Create a function at the end of the file `service/app.js` with the following code to trace every incoming HTTP request.

  ```
  function tracingMiddleWare (req, res, next) {
    const tracer = opentracing.globalTracer();
    // Extracting the tracing headers from the incoming http request
    const wireCtx = tracer.extract(opentracing.FORMAT_HTTP_HEADERS, req.headers)
    // Creating our span with context from incoming request
    const span = tracer.startSpan(req.path, { childOf: wireCtx })
    // Use the log api to capture a log
    span.log({ event: 'request_received' })

    // Use the setTag api to capture standard span tags for http traces
    span.setTag(opentracing.Tags.HTTP_METHOD, req.method)
    span.setTag(opentracing.Tags.SPAN_KIND, opentracing.Tags.SPAN_KIND_RPC_SERVER)
    span.setTag(opentracing.Tags.HTTP_URL, req.path)

    // include trace ID in headers so that we can debug slow requests we see in
    // the browser by looking up the trace ID found in response headers
    const responseHeaders = {}
    tracer.inject(span, opentracing.FORMAT_HTTP_HEADERS, responseHeaders)
    res.set(responseHeaders)

    // add the span to the request object for any other handler to use the span
    Object.assign(req, { span })

    // finalize the span when the response is completed
    const finishSpan = () => {
      if (res.statusCode >= 500) {
        // Force the span to be collected for http errors
        span.setTag(opentracing.Tags.SAMPLING_PRIORITY, 1)
        // If error then set the span to error
        span.setTag(opentracing.Tags.ERROR, true)

        // Response should have meaning info to futher troubleshooting
        span.log({ event: 'error', message: res.statusMessage })
      }
      // Capture the status code
      span.setTag(opentracing.Tags.HTTP_STATUS_CODE, res.statusCode)
      span.log({ event: 'request_end' })
      span.finish()
    }
    res.on('finish', finishSpan)
    next()
  }
  ```

The `tracer.extract(opentracing.FORMAT_HTTP_HEADERS, req.headers)` function above will attempt to extract the tracing headers from the incoming HTTP request. If the incoming request contains trace information, it will be used to create a child span from the previous service; the current service will then be correctly associated with the tracing dependency graph.

The new span is created using the function `tracer.startSpan(req.path, { childOf: wireCtx })`

The first activity captured is a log event of `request_end` with the function `span.log({ event: 'request_received' })`

The new span context is added to the HTTP response, this way the HTTP client can have this information in case of troubleshooting a particular HTTP request.

  ```
  const responseHeaders = {}
  tracer.inject(span, opentracing.FORMAT_HTTP_HEADERS, responseHeaders)
  res.set(responseHeaders)
  ```


The `span` is stored in the `req` object, this way the main endpoint handler can use it in case of attaching information into the same span or creating a new child span using this top-level `span` as a parent.

  ```
  Object.assign(req, { span })
  ```

When the request is finished by listening on the event `finish` in `res.on('finish', finishSpan)`, the response is analyzed to check if there was an error. If it is an error then the span is set to be sampled and marked as error using the following functions which includes a log event:

  ```
  span.setTag(opentracing.Tags.SAMPLING_PRIORITY, 1)
  span.setTag(opentracing.Tags.ERROR, true)
  span.log({ event: 'error', message: res.statusMessage })
  ```

For every HTTP response the status Code and a log event are captured. With log event `request_end` you can will easily see the time spent since the log event `request_start` . Finaly the span needs to be finished.

  ```
  span.setTag(opentracing.Tags.HTTP_STATUS_CODE, res.statusCode)
  span.log({ event: 'request_end' })
  span.finish()
  ```

4. At line `12` add the following code to use the function `tracingMiddleWare` as the first middleware to handle every HTTP request.

  ```
  app.use(tracingMiddleWare)
  ```

5. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

  ```
  docker-compose build
  docker-compose up
  ```

6. Now that the code is instrumented, call the same API endpoint a few times to capture the traces.

  ```
  curl http://localhost:8080/sayHello/Carlos
  Hello Carlos!
  ```

7. Open the Jaeger UI using the web browser

  ```
  open http://localhost:16686/jaeger
  ```

8. Select the Service `service-a` from the drop-down options and click `Find Traces`

![find-trace](../../images/nodejs-service-a-find-trace.jpg)

9. Click on one of the traces, then expand the trace's `Tags` and `Logs`. You should see information about the HTTP request such as `http.method` set to `GET` and `http.status_code` set to `200`. The Logs section has two log events; one with `request_received` and the final log `request_end`. The timestamps for each of the log events give you how much time the request took to be processed by your service business logic. In this example, it took `4ms`.

![trace-details](../../images/nodejs-service-a-trace-details.jpg)

10. Force an error in the service by calling the `/error` endpoint.

  ```
  curl http://localhost:8080/error
  some error (ノ ゜Д゜)ノ ︵ ┻━┻
  ```


11. Click `Find Traces` now it should show a trace mark with `Error` in red.

![node-service](../../images/nodejs-service-a-error.jpg)

12. Click on the trace with the `Error`, then expand the trace's `Tags` and `Logs`. You should see information about the trace such as `error` set to `true` and `http.status_code` set to `500`. The Logs section has an additional log event with `message = Internal Server Error`. Expand the log event.

![service-error](../../images/nodejs-service-a-error-details.jpg)

## Finding slow HTTP requests

In the `service-a`, we have the API endpoint `/sayHello`. We used this endpoint in the previous section but called it only once. This endpoint has some strange behavior. Not all responses are fast (=~ 2ms) and often the response is slow (=~ 100ms).

13. Stop docker-compose with `Ctrl+C` and start it again.

  ```
  docker-compose up
  ```

14. Run  the following code to call the API multiple times or open the URL endpoint http://localhost:8080/sayHello/Carlos on the web browser and click refresh multiple time.

  ```
  i=0;
  while [ $i -lt 15 ];
  do curl http://localhost:8080/sayHello/Carlos -I -s | head -n 1; i=$((i+1));
  done;
  ```

15. Open the Jaeger UI using the web browser

  ```
  open http://localhost:16686/jaeger
  ```

16. Select the Service `service-a` from the drop-down options and click `Find Traces`

![serv-slow](../../images/nodejs-service-a-slow.jpg)

In the picture above, you can see a timeline graph with each trace represented with a circle. In this case, we have 15 traces in the result set when we clicked `Find Traces`.
Some traces are taking approximately 100ms and others are taking approximately 2ms.
You can see the pattern where only every 3rd request the response is slow.
When troubleshooting, we are interested first in the slowest requests. You can click on one of the traces on the graph, or you can sort in the table by `Longest First`.

17. Select the trace that took the longest time (103ms). Expand all the information for the single-span operation `/sayHello` including tags and logs.

![slow-details](../../images/nodejs-service-a-slow-details.jpg)

18. The handler has a sleep step in the function `sayHello` that delays the response every 3rd request. Open the file `service-a/hello.js` and located the culprit code.

  ```
  // simulate a slow request every 3 requests
  setTimeout(async () => {
    const response = await formatGreeting(name);
    res.send(response)
  }, counter++ % 3 === 0 ? 100 : 0)
  ```

19. Remove the `setTimeout` function and replace it with the two functions `formatGreeting` and `res.send`.

  ```
  const response = await formatGreeting(name);
  res.send(response)
  ```

20. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

  ```
  docker-compose build
  docker-compose up
  ```

21. Run again the following code to call the API multiple times or open the URL endpoint http://localhost:8080/sayHello/Carlos on the web browser and click refresh multiple time.

  ```
  i=0;
  while [ $i -lt 15 ];
  do curl http://localhost:8080/sayHello/Carlos -I -s | head -n 1; i=$((i+1));
  done;
  ```

22. Open the Jaeger UI using the web browser.

  ```
  open http://localhost:16686/jaeger
  ```

23. Select the Service `service-a` from the drop-down options and click `Find Traces`.

![node-service](../../images/nodejs-service-a-fast.jpg)

You can see now that all HTTP requests are fast and the problem is fixed.


Cloud Native applications can be composed of microservices and each microservice handling multiple endpoints. Instrumenting the code of these microservices enables us to obverse their behavior and quickly narrow down to a specific service; and within that service, to further narrow down to a specific endpoint having problems. 

As demonstrated in this exercise, you can increase the observability of your application starting with a single trace and span. In the next sections of this lab, you will create additional spans to further observe the behavior of a specific HTTP handler and function. 

## Tracing an HTTP handler

In the previous example, we were able to identify the endpoint `/sayHello` as one of interest in our service. Let's see how can we add tracing instrumentation to the function that is handling this endpoint.

24. Import at the top of the file `service-a/hello.js` the `opentracing` module, and get the global tracer

  ```
  const opentracing = require('opentracing')
  const tracer = opentracing.globalTracer()
  ```

25. Open the file `service-a/hello.js` and locate the function `sayHello`

  ```
  const sayHello = async (req, res) => {
    const name = req.params.name
    const response = await formatGreeting(name);
    res.send(response)
  }
  ```

26. Create a new child span using the parent span located in the `req` object as context.
This will allow the trace to have an additional child span. Use the function `tracer.startSpan` and name the span `say-hello`.

  ```
  const sayHello = async (req, res) => {
    const span = tracer.startSpan('say-hello', { childOf: req.span })
    const name = req.params.name
    const response = await formatGreeting(name);
    res.send(response)
  }
  ```

27. The OpenTracing API supports the method `log`. You can log an event with a name and an object. Add a log to the span with a message that contains the value of the name.

  ```
  const sayHello = async (req, res) => {
    const span = tracer.startSpan('say-hello', { childOf: req.span })
    const name = req.params.name
    span.log({ event: 'name', message: `this is a log message for name ${name}` })
    const response = await formatGreeting(name);
    res.send(response)
  }
  ```

28. The OpenTracing API supports the method `setTag`. You can tag the span with a key and any value. Add a tag that contains the response, in normal use cases you would not log the entire response and instead key values that are useful for later searching for spans. Remember to call the `span.finish()` when you are done instrumenting the span.

  ```
  const sayHello = async (req, res) => {
    const span = tracer.startSpan('say-hello', { childOf: req.span })
    const name = req.params.name
    span.log({ event: 'name', message: `this is a log message for name ${name}` })
    const response = await formatGreeting(name);
    span.setTag('response', response)
    span.finish()
    res.send(response)
  }
  ```

29. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

  ```
  docker-compose build
  docker-compose up
  ```


30. Call the API endpoint.

  ```
  curl http://localhost:8080/sayHello/Carlos
  Hello Carlos!
  ```


31. Open the Jaeger UI using the web browser

  ```
  open http://localhost:16686/jaeger
  ```

32. Select the Service `service-a` from the drop-down options and click `Find Traces`

![2-spans](../../images/nodejs-service-a-2-spans.jpg)


Notice in the result items table, for the trace item that the trace indicates that there are a total of two spans `2 Spans` and that service-a contains two spans `service-a (2)`

33. Click the trace, expand the spans `say-hello`, and then expand the `Tags` and `Logs` sections.

![span-details](../../images/nodejs-service-a-span-details.jpg)


<InlineNotification>

**Notice:**  
* in the Tags section the tag is located with key `name` and the string value `Hello Carlos!`.
* in the Logs section the log event with the name `name` and the message `this is a log message for name Carlos`

</InlineNotification>


## Tracing a function

The HTTP handler usually calls other functions to perform the business logic, when calling another function within the same service you can create a child span.

34. The `sayHello` handler calls the function `formatGreeting` to process the input `name`. Pass the current span as an additional parammeter `formatGreeting(name, span)`

  ```
  const sayHello = async (req, res) => {
    const span = tracer.startSpan('say-hello', { childOf: req.span })
    const name = req.params.name
    span.log({ event: 'name', message: `this is a log message for name ${name}` })
    const response = await formatGreeting(name, span)
    span.setTag('response', response)
    span.finish()
    res.send(response)
  }
  ```

35. In the function `formatGreeting` create a new span using `tracer.startSpan`.
Use the span from the HTTP handler as `parent` span, name the span `format-greeting`. Remember to finish the span before returning with `span.finish()`.

  ```
  function formatGreeting(name, parent) {
    const span = tracer.startSpan('format-greeting', { childOf: parent })
    span.log({ event: 'format', message: `formatting message locally for name ${name}` })
    const response = `Hello ${name}!`
    span.finish()
    return response
  }
  ```

36. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

  ```
  docker-compose build
  docker-compose up
  ```


37. Call the API endpoint.

  ```
  curl http://localhost:8080/sayHello/Carlos
  Hello Carlos!
  ```


38. Open the Jaeger UI using the web browser

  ```
  open http://localhost:16686/jaeger
  ```

39. Select the Service `service-a` from the drop-down options and click `Find Traces`

![3-spans](../../images/nodejs-service-a-3-spans.jpg)

Notice that the trace now contains three spans.

40. Click the trace, expand the spans `say-hello` and `format-greeting`, and then expand the `Logs` sections.

![span-form](../../images/nodejs-service-a-span-formatter.jpg)

Notice the cascading effect between the three spans, the span `format-greeting` contains the message `formatting message locally for name Carlos` that we instrumented.

## Distributing Tracing

You can have a single trace that goes across multiple services, this allows you to distribute tracing and better observability on the interactions between services.

In the previous example, we instrumented a single service `service-a`, and created a span when calling a local function to format the greeting message.

For the following example, we are going to use a remote service `service-b` to format the message, and returning the formatted greeting message to the HTTP client.

41. In the file `service-a/hello.js` locate the handler function `sayHello` and replace the function call `formatGreeting(name, span)` with `formatGreetingRemote(name, span)`.

  ```
  const sayHello = async (req, res) => {
    const span = tracer.startSpan('say-hello', { childOf: req.span })
    const name = req.params.name
    span.log({ event: 'name', message: `this is a log message for name ${name}` })
    const response = await formatGreetingRemote(name, span)
    span.setTag('response', response)
    span.finish()
    res.send(response)
  }
  ```

42. In the function `formatGreetingRemote` use the function `tracer.inject` to extract the span context and inject them into the `headers` of the HTTP request when calling the remote service `service-b` endpoint `/formatGreeting`.

  ```
  const bent = require('bent')

  const formatGreetingRemote = async (name, span) => {
    const service = process.env.SERVICE_FORMATTER || 'localhost'
    const servicePort = process.env.SERVICE_FORMATTER_PORT || '8081'
    const url = `http://${service}:${servicePort}/formatGreeting?name=${name}`
    const headers = {}
    tracer.inject(span, opentracing.FORMAT_HTTP_HEADERS, headers)
    const request = bent('string', headers)
    const response = await request(url)
    return response
  }
  ```

43. The service `service-b` is already instrumented to trace every HTTP request using the same procedure <<tracing-every-http-request, Trace every HTTP request>> that we did for service `service-a`.

44. Locate the file `service-b/formatter.js` and add.

45. Import at the top of the file `service-b/formatter.js` the `opentracing` module, and get the global tracer

  ```
  const opentracing = require('opentracing')
  const tracer = opentracing.globalTracer()
  ```

46. Located the HTTP handler function `formatGreeting` in the file `service-b/formatter.js`

  ```
  function formatGreeting(req, res) {
    const name = req.query.name
    const response = `Hello from service-b ${name}!`
    res.send(response)
  }
  ```

47. Create a new child span using the parent span located in the `req` object as context.
This will allow the trace to have an additional child span. Use the function `tracer.startSpan` and name the span `format-greeting`.

  ```
  function formatGreeting(req, res) {
    const span = tracer.startSpan('format-greeting', { childOf: req.span })
    const name = req.query.name
    const response = `Hello from service-b ${name}!`
    res.send(response)
  }
  ```

48. Add a log event `format` to the new span using the method `span.log`. Remember to call the `span.finish()` when you are done instrumenting the span.

  ```
  function formatGreeting(req, res) {
    const span = tracer.startSpan('format-greeting', { childOf: req.span })
    const name = req.query.name
    span.log({ event: 'format', message: `formatting message remotely for name ${name}` })
    const response = `Hello from service-b ${name}!`
    span.finish()
    res.send(response)
  }
  ```


49. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

  ```
  docker-compose build
  docker-compose up
  ```


50. Call the API endpoint.

  ```
  curl http://localhost:8080/sayHello/Carlos
  Hello Carlos!
  ```


51. Open the Jaeger UI using the web browser

  ```
  open http://localhost:16686/jaeger
  ```

52. Select the Service `service-a` from the drop-down options and click `Find Traces`

![b-trace](../../images/nodejs-services-b-trace.jpg)

Notice that the trace contains a total of four spans `4 Spans` two for `service-a(2)` and two for `service-b(2)`

53. Click the trace to drill down to get more details.

![b-spans](../../images/nodejs-services-b-spans.jpg)

Notice in the top section, the summary which includes the `Trace Start`, `Duration: 19ms`, `Services: 2`, `Depth: 4` and `Total Spans: 4`.

Notice the bottom section on how the total duration of 19ms is broken down per span, and at which time each span started and ended. You can see that the time spent in `service-b` was 4ms, meaning that for this single HTTP request `service-a` spent 15ms and `service-b` spent 4ms.

54. Expand the `Logs` sections for both spans `say-hello` from `service-a` and  `format-greeting` from `service-b`.

![b-logs](../../images/nodejs-services-b-logs.jpg)


Notice on the right side, each span has a summary each with the associated `Service`, `Duration`, and `Start Time`. The `Start Time` of a span marks the end time from the previous span.

Notice the time for the first log message `this is a log message for name Carlos` in `service-a` is of 1ms, this means this log event happened 1ms after the trace started.

Notice the time for the second log message `formatting message remotely for name Carlos` in `service-b` is of 12ms, this means this log event happened 12ms after the trace started in `service-a`.

It is very useful to see the log events we instrumented in our endpoint handlers across services in this manner because it provides full observability of the lifecycle of the HTTP request across multiple services.

## Baggage propagation

Imagine a scenario where you want to redirect all Safari users to a specific version of a service using the User-Agent HTTP header. This is useful in canary deployments when a new version is rolled out for a specific subset of users. However, the header is present only at the first service. If the routing rule is for a service lower in a call graph then the header has to be propagated through all intermediate services. This is a great use-case for distributed context propagation which is a feature of many tracing systems.

Baggage items are key:value string pairs that apply to the given Span, its SpanContext, and all Spans which directly or transitively reference the local Span. That is, baggage items propagate in-band along with the trace itself.

Baggage items enable powerful functionality given a full-stack OpenTracing integration (for example, arbitrary application data from a mobile app can make it, transparently, all the way into the depths of a storage system), and with it some powerful costs: use this feature with care.

Use this feature thoughtfully and with care. Every key and value is copied into every local and remote child of the associated Span, and that can add up to a lot of network and CPU overhead.

55. Locate the HTTP handler `sayHello` in the file `service-a/hello.js`. Use the method `span.setBaggageItem('my-baggage', name)` before the function call `formatGreetingRemote(name, span)` to set the baggage with key `my-baggage` to the value of the `name` parameter.

  ```
  const sayHello = async (req, res) => {
    const span = tracer.startSpan('say-hello', { childOf: req.span })
    const name = req.params.name
    span.log({ event: 'name', message: `this is a log message for name ${name}` })
    span.setBaggageItem('my-baggage', name)
    const response = await formatGreetingRemote(name, span)
    span.setTag('response', response)
    span.finish()
    res.send(response)
  }
  ```

56. Locate the HTTP handler `formatGreeting` in the file `service-b/formatter.js`. Use the method `span.getBaggageItem('my-baggage')` to get the value of the name parameter at `service-a`. For convenience log the value using `span.log` to see the value in the Jaeger UI.

  ```
  function formatGreeting(req, res) {
    const span = tracer.startSpan('format-greeting', { childOf: req.span })
    const name = req.query.name
    span.log({ event: 'format', message: `formatting message remotely for name ${name}` })
    const response = `Hello from service-b ${name}!`
    const baggage = span.getBaggageItem('my-baggage')
    span.log({ event: 'baggage', message: `this is baggage ${baggage}` })
    span.finish()
    res.send(response)
  }
  ```

57. Build and run the service. If docker-compose is already running in the terminal enter `Ctrl+C` to exit and stop the containers.

  ```
  docker-compose build
  docker-compose up
  ```

58. Call the same API endpoint, but now is instrumented with tracing

  ```
  curl http://localhost:8080/sayHello/Carlos
  Hello Carlos!
  ```

59. Open the Jaeger UI using the web browser

  ```
  open http://localhost:16686/jaeger
  ```

60. Select the Service `service-a` from the drop-down options and click `Find Traces`. Expand the section `Logs` for the spans `say-hello` and `format-greeting`

![b-baggage](../../images/nodejs-service-b-baggage.jpg)

Notice that the baggage is set in the `service-a` with the value `Carlos` this baggage is propagated to all spans local or remote. In the `server-b` span you can see the baggage value `Carlos` is propagated.


## Searching Traces

If you have a specific trace id you can search for it by putting the trace id on the top left search box.

You can also use a tag to search for example searching traces that have a specific HTTP status code, or one of the custom tags we added to a span.

61. To search for traces using HTTP method `GET` and status code `200`, enter `http.status_code=200  http.method=GET` on the `Tags` field in the search form, and then click `Find Traces`.

![ui-search](../../images/jaeger-ui-search.png)

## Dependency graph


The Jaeger UI has a view for service dependencies, it shows a visual Directed acyclic graph (DAG).

62. Click the tab `Dependencies`, then click the `DAG` tab.

![ui-dep1](../../images/jaeger-ui-dependencies-dag-1.jpg)

Notice that the graph shows the direction with an arrow flowing from `service-a` to `service-b`. It also shows the number of traces between the services.

This is is a simple example and there is not much value for a small set of services, but when a large number of services each with multiple endpoints then the graph becomes more interesting like the following example:

![ui-depp](../../images/jaeger-ui-dependencies-dag-2.jpg)
