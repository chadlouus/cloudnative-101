---
title: Distributed Tracing
---

## What is Distributed Tracing?

Distributed tracing, also called distributed request tracing, is a method used to profile and monitor applications, especially those built using a microservices architecture. Distributed tracing helps pinpoint where failures occur and what causes poor performance.

## Who Uses Distributed Tracing?

IT and DevOps teams can use distributed tracing to monitor applications.  Distributed tracing is particularly well-suited to debugging and monitoring modern distributed software architectures, such as microservices.

Developers can use distributed tracing to help debug and optimize their code.

## What is OpenTracing?

It is probably easier to start with what OpenTracing is NOT.

* OpenTracing is not a download or a program.  Distributed tracing requires that software developers add instrumentation to the code of an application, or to the frameworks used in the application.

* OpenTracing is not a standard. The Cloud Native Computing Foundation (CNCF) is not an official standards body.  The OpenTracing API project is working towards creating more standardized APIs and instrumentation for distributed tracing.

OpenTracing is comprised of an API specification, frameworks and libraries that have implemented the specification, and documentation for the project.   OpenTracing allows developers to add instrumentation to their application code using APIs that do not lock them into any one particular product or vendor.

For more information about where OpenTracing has already been implemented, see the [list of languages](/docs/supported-languages) and the  [list of tracers](/docs/supported-tracers) that support the OpenTracing specification.


## What is a Span?

The "**span**" is the primary building block of a distributed trace, representing an individual unit of work done in a distributed system.

Each component of the distributed system contributes a span - a named, timed operation representing a piece of the workflow.

Spans can (and generally do) contain "References" to other spans, which allows multiple Spans to be assembled into one complete **Trace** - a visualization of the life of a request as it moves through a distributed system.

Each span encapsulates the following state according to the OpenTracing specification:

- An operation name
- A start timestamp and finish timestamp
- A set of key:value span **Tags**
- A set of key:value span **Logs**
- A **SpanContext**

### Tags

**Tags** are key:value pairs that enable user-defined annotation of spans in order to query, filter, and comprehend trace data.

Span tags should apply to the _whole_ span. There is a list available at [semantic_conventions.md](https://github.com/opentracing/specification/blob/master/semantic_conventions.md) listing conventional span tags for common scenarios. Examples may include tag keys like `db.instance` to identify a database host, `http.status_code` to represent the HTTP response code, or `error` which can be set to True if the operation represented by the Span fails.

### Logs

**Logs** are key:value pairs that are useful for capturing _timed_ log messages and other debugging or informational output from the application itself.  Logs may be useful for documenting a specific moment or event within the span (in contrast to tags that should apply to the span regardless of time).

### Baggage Items

The **SpanContext** carries data across process boundaries. Specifically, it has two major components:

* An implementation-dependent state to refer to the distinct span within a trace
** i.e., the implementing Tracer's definition of spanID and traceID  
* Any **Baggage Items**
** These are key:value pairs that cross process-boundaries.
** These may be useful to have some data available for access throughout the trace.

## Tracer

A `Tracer` is the actual implementation that will record the `Spans` and publish them somewhere. How an application handles the actual `Tracer` is up to the developer: either consume it directly throughout the application or store it in the `GlobalTracer` for easier usage with instrumented frameworks.

Different `Tracer` implementations vary in how and what parameters they receive at initialization time, such as:

- Component name for this application's traces.
- Tracing endpoint.
- Tracing credentials.
- Sampling strategy.

Once a `Tracer` instance is obtained, it can be used to manually create `Span`, or pass it to existing instrumentation for frameworks and libraries.

In order to not force the user to keep around a `Tracer`, the `io.opentracing.util` artifact includes a helper `GlobalTracer` class implementing the `io.opentracing.Tracer` interface, which, as the name implies, acts as a global instance that can be used from anywhere. It works by forwarding all operations to another underlying `Tracer`, that will get registered at some future point.

By default, the underlying `Tracer` is a `no-nop` implementation.

### Starting a new Trace

A new trace is started whenever a new `Span` is created without references to a parent `Span`. When creating a new `Span`, you need to specify an "operation name", which is a free-format string that you can use to help you identify the code this `Span` relates to.
The next `Span` from our new trace will probably be a child `Span` and can be seen as a representation of a sub-routine that is executed "within" the main `Span`. This child `Span` has, therefore, a `ChildOf` relationship with the parent.
Another type of relationship is the `FollowsFrom` and is used in special cases where the new `Span` is independent of the parent `Span`, such as in asynchronous processes.


### Accessing the Active Span

`Tracer` can be used for enabling access to the `ActiveSpan`. `ActiveSpans` can also be accessed through a `scopeManager` in some languages. Refer to the specific language guide for more implementation details.

### Propagating a Trace with Inject/Extract

In order to trace across process boundaries in distributed systems, services need to be able to continue the trace injected by the client that sent each request. OpenTracing allows this to happen by providing inject and extract methods that encode a span's context into a carrier.
The `inject` method allows for the `SpanContext` to be passed on to a carrier. For example, passing the trace information into the client's request so that the server you send it to can continue the trace. The `extract` method does the exact opposite. It extracts the `SpanContext` from the carrier. For example, if there was an active request on the client side, the developer must extract the `SpanContext` using the `io.opentracing.Tracer.extract` method.

![Trace Propagation](./images/Extract.png)

NOTE: Content extracted from http://opentracing.io

## Presentations


## Documentation

| Topics                            | Description         | Link        |
| --------------------------------  | ------------------  |:----------- |
| Open Tracing | OpenTracing WebPage. | See [Open Tracing](http://opentracing.io) |

## Activities

| Topics                            | Description         | Link        |
| --------------------------------| ------------------  |:----------- |
| NodeJS | Implementing Distributed Tracing in NodeJS | [NodeJS](./activities/lab1) |
| Java | Implementing Distributed Tracing in Java | [Java](./activities/lab2) |
| Open Liberty | Implementing Distributed Tracing in Open Liberty | [Open Liberty](./activities/lab3) |
| Jaeger in OpenShift & Kubernetes | Implementing Istio on Kubernetes & OpenShift | [Jaeger in Kubernetes](./activities/lab4) |
| Jaeger with Istio - NodeJS | Implementing Istio with NodeJS | [Istio NodeJS](./activities/lab5) |
| Jaeger with Istio - Java | Implementing Istio with Java | [Istio Java](./activities/lab6) |
| Jaeger with Istio - Open Liberty | Implementing Istio with Open Liberty | [Istio Open Liberty](./activities/lab7) |

