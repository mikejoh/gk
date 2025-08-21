# OpenTelemetry Certified Associate

_This exam is an online, proctored, multiple-choice exam._

## Resources

## Topics

<details>
  <summary>Fundamentals of Observability (18%)</summary>

* Telemetry Data
* Semantic Conventions
* Instrumentation
* Analysis and Outcomes

## Telemetry Data

The purpose of OpenTelemtry is to:

* collect
* process
* export

signals. Signals are _categories of telemetry_. Metrics, logs, traces and baggage are examples of signals.

Each signal represents a _coherent_, _stand-alone_ set of functionality. Each signal follows a separate lifecycle.

Signals are system outputs that describe the underlying activity of the operating system and applications running on a platform, a measurement at a specific point in time:

* temperature
* memory usage
* an event that you want to trace

OpenTelemetry supports:

* Traces
* Metrics
* Logs
* Baggage

### Traces

The path of a request through your application.

Gives you the big picture of what happens when a request is made to an application. A _root_ span is identified by the lack of a `parent_id`, `null`. All spans in a trace will have the same `trace_id`. All spans shares the same `trace_id` which makes it into a trace!

* _Tracer Provider_, sometimes called `TracerProvider` is a factor for Tracers. In most apps a Tracer Provider is initialized once and its lifecycle matches the applications lifecycle. The initialization also includes Resource and Exporter initialization.
* _Tracer_, creates spans containing information about what is happening for a given operation, such as a request in a service.
* _Trace_ Exporter, send traces to a consumer. Example: OpenTelemetry Collector.
* _Context Propagation_, is the core concept that enables Distributed Tracing. With _context propagation_ spans can be correlated with each other and assembled into a trace.
* _Span_, unit of work or operation. Building block of traces. In OTEL they include the following information:
  * _Name_
  * _Parent Span ID_
  * _Start and End Timestamps_
  * _Span Context_, an immutable object on every span that contains:
    * _Trace ID_
    * _Span ID_
    * _Trace Flags,_ binary encoding containing information about the trace
    * _Trace State_
  * _Attributes_, key-value pairs that contain metadata that you can use to annotate a Span to carry information about the operation it is tracking. You can add attributes to spans _during_ or _after_ span creation. Rules:
    * Keys must be non-null string values
    * Values mush be a non-null string, boolean, floating point value, integer or an array of these values
  * _Span Events_, a structured log message, or annotation on a Span.
  * _Span Links_, associate on span with one or more spans, casual relationship.
  * _Span Status_
    * _Unset_ - default, completed without an error.
    * _Ok_ - explicitly mareked as error-free by the dev or application.
    * _Error_ - Some error occurred in the operation it tracks.
  * _Span Kind_, is ahint to the tracing backend as to how the trace should be assembled. Can be:
    * _Internal_ - Span is used for internal operations.
    * _Server_ - Span is used to track a request to a server.
    * _Client_ - Span is used to track a request to a client.
    * _Producer_ - Span is used to track an event produced by the application.
    * _Consumer_ - Span is used to track an event consumed by the application.

When to use Span Events vs Span Attributes:

Consider wheter a specific timestamp is meaningful, if its meaningful add it as data to a span event if it isnt meaningful, attach the data as span attributes.

### Metrics

A measurement captured at runtime.

The moment of capture is known as a _metric event_.

MEter Provider is factory for Meter instaces. A meter creates metric instruments, capturing measurements about a service at runtime.

Metric Exporter send metric data to a consumer.

Metric instruments are defined by:

* Name
* Kind
* Unit (optional)
* Description (optional)

These are chosen by the dev.

The instrument kind can be one of the following:

* _Counter_ - Value that accumulates over time.
* _Asynchronous Counter_ - Value that accumulates over time, but is updated asynchronously.
* _UpDownCounter_ - Value that can increase or decrease over time.
* _Asynchronous UpDownCounter_ - Value that can increase or decrease over time, but is updated asynchronously.
* _Histogram_ - A representation of the distribution of values.
* _Gauge_ - Measures a current value at the time it is read.
* _Asynchronous Gauge_ - Measures a current value at the time it is read, but is updated asynchronously.

Aggregation is the technique whereby a large number of measurements are combined into either exact or estimated statistics about metric events that took place during a time window.

OTLP API provides a default aggregation for each instrument _which can be overridden by views_.

A view provides SDK users with the flexibility to customize the metrics output by the SDK. You can customize which metric instruments are to be processed or ignored.

### Logs

A recording of an event.

A log is a timestamped text record, struvtured or unstructured with optional metadata.

OTLP SDKs and autoinstrumentation utlizie several components to automatically correlate logs with traces.

A structured log is log whose textual data format follows a consistent, machine-readable structure. E.g. JSON.

For infrastructure components, CLF is commonly used:

```
127.0.0.1 - johndoe [04/Aug/2024:12:34:56 -0400] "POST /api/v1/login HTTP/1.1" 200 1234
```

Unstructured logs are logs that dont follow a consistent structure, more human readable and used in development. Difficult to parse. Might need pre-processing them to be human readaable.

Semistructured logs are logs that does use some self-consistent pattern but may not use the same formatting and delimiters across different systems.

Log record respresents the recording of an event, a log record in OTLP contains two kinds of fields:

* Named top-level fields of specific type and meaning
* Resource and attributes fields of arbitrary valu e and type

| Field Name | Description |
|---|---|
| Timestamp | Time when the event occurred. |
| ObservedTimestamp | Time when the event was observed. |
| TraceId | Request trace ID. |
| SpanId | Request span ID. |
| TraceFlags | W3C trace flag. |
| SeverityText | The severity text (also known as log level). |
| SeverityNumber | Numerical value of the severity. |
| Body | The body of the log record. |
| Resource | Describes the source of the log. |
| InstrumentationScope | Describes the scope that emitted the log. |
| Attributes | Additional information about the event. |

### Baggage

Contextual information that is passed between signals.

Propagate any data you like alongside context. Resides next to context. Key-value store.

Baggate means you can pass data across services and processes making it available to all signals.

By using Contextual Propagation to pass baggage across these services the clientId is available to any additional spans, metrics or logs.

Use Baggage to include information typically available only at the start of a request further downstream.

Keep in mind that downstream services could propagate Baggage outside your network.

Baggage is not the same as attributes, a common use case for Baggage is to add data to Span Attributes across a whole trace.

## Semantic Conventions

OpenTelemetry deinfes semantic conventions or _sematic attributes_ that specify common names for different kind of operations and data.

[The Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/) define a common set of (semantic) attributes which provide meaning to data when collecting, producing and consuming it.

Metrics Conventions:

* Metric Naming
  * Use snake case
* Labeling
  * Attach contextual metadata to metrics
  * Avoid high cardinality labels, this can hurt performance

Traces Conventions:

* Span Naming
  * Use clear, descriptive names
  * Use snake case
* Attributes
  * Similar to labels in metrics, span attributes provide context such as HTTP status codes, method names or error messages
* Context Propagation
  * Ensure that semantic conventions are followed to maintain consistent trace context across services

Logs Conventions:

* Structured logging
  * Utilize key-value pairs in log messages to enable better parsing and searching
  * Instead of plain text, format logs with keys like `timestamp`, `severity`, and `message`
* Consistent Field Names
  * Adopt a standard naming schema for common log fields to facilitate correlation with metrics and traces
* Log Levels
  * Standardize log severity levels to maintain uniformity across different application and services

## Instrumentation

For a system to be observable it muse be **instrumented**. Code from the systems components must emit signals such as traces, metrics and logs.

Theres two ways to instrument your code:

* Code based solutions via official APIs and SDKs
  * Deeper insight and rich telemetry from your application itself.
* Zero-code solutions
  * Great to get started, or when you cant modify the application you need to get telemetry out of.

Other OTEL benefits:

* Libraries can leverage the OpenTelemetry API as a dependency, which will have no impact on applications using that library

Zero-code instrumentation adds the OTEL API and SDK capabilities to your application typically as an agent or agent-like installation. The mechanism is typically one of:

* bytecode manipulation
* monkey patching
* eBPF

## Analysis and Outcomes

The goal of observability is to provide insights into the behavior and performance of systems, enabling teams to:

* Identify and troubleshoot issues quickly
* Optimize system performance and resource utilization
* Enhance user experience through better reliability and responsiveness
* Make informed decisions based on data-driven insights

Data aggregation, `sum()`, `avg_over_time()` and `max_over_time()` are examples of analysis functions that can be used to analyze telemetry data.

Visualization tools like Grafana, Kibana, and Jaeger can be used to create dashboards and visual representations of telemetry data.

Outcome based analysis involves using telemetry data to measure the impact of changes, such as:

* Performance improvements
* SLO, SLI and SLAs

Analysis transforms raw telemetry data into insights that drive decision-making, helping to ensure systems meet performance and reliability targets.

</details>

<details>
  <summary>The OpenTelemetry API and SDK (46%)</summary>

* Data Model
* Composability and Extension
* Configuration
* Signals (Tracing, Metric, Log)
* SDK Pipelines
* Context Propagation
* Agents

The OpenTelemtry data model defines standardized representations for telemetry data, ensuring consistency across different languages and systems.



</details>

<details>
  <summary>The OpenTelemetry Collector (26%)</summary>

* Configuration
* Deployment
* Scaling
* Pipelines
* Transforming Data

</details>

<details>
  <summary>Maintaining and Debugging Observability Pipelines (10%)</summary>

* Context Propagation
* Debugging Pipelines
* Error Handling
* Schema Management

</details>
