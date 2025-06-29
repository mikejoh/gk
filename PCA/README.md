# Prometheus Certified Professional

_This exam is an online, proctored, multiple-choice exam._

## Resources

* <https://training.linuxfoundation.org/certification/prometheus-certified-associate/>
* <https://prometheus.io/docs/introduction/overview/>
* <https://github.com/hugoprudente/pca-preparation-guide>

## Topics

<details>
  <summary>Observability Concepts (18%)</summary>

Prometheus is an open-source systems monitoring and aletring toolkit.

Features:

* Multi-dimensional data model with time series data identified by metric name and key/value pairs
* PromQL, flexible query language to leverage this dimensionality
* No reliance on distributed storage, single nodes are autonomous
* Time series collection happens via a _pull model over HTTP_
* Pushing time series is supported
* Targets are discovered via service discovery or static configuration

## Metrics

Metrics are _numerical measurements_ in layperson terms. The term _time series_ refers to the recording of changes over time.

Metric types:

* **Counter**: A cumulative metric that represents a single monotonically increasing counter. It can only increase or be reset to zero. Do **not** use counter for values that can decrease, example do not use a counter to track the number of currenlty running processes.
* **Gauge**: A metric that represents a single numerical value that can arbitrarily fo up and down. Ususally used for values like temperatures, current memory, concurrent requests.
* **Historgram**: Samples observeations, usually things like request duration or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values.
  
  A histogram with a base metric of `<basename>` exposes multiple time series during a scrape:
  * Cumulative counters for the observations buckets, exposed as `<basename>_bucket{le=<upper_inclusive_bound>}`
  * The **total sum** of all observed values, exposed as `<basename>_sum`
  * The count of events that have been observed, exposed as `<basename>_count`

  Use the `histogram_quantile()` function to calculate quantiles from histograms or even aggregations of histograms.

  Suitable for calculate an Apdex score. Application Performance Index.

* **Summmary**: Similar to a histogram, samples observations, like request durations and response sizes. W

## Understand logs and events

Using the Prometheus query log, it has the ability to log all the queries run by the engine to a log file, as of `2.16.0`. This guide demonstrates how to use that log file.

To enable it:

1. Adapt the configuration add or remove query log configurations in the `prometheus.yml` file:

   ```yaml
   global:
     query_log_file: "/var/log/prometheus/query.log"
     # Additional configurations can be added here
   ```

2. Reload the Proemtheus server configuration by `POST`ing to the `/-/reload` endpoint. Needs the `--web.enable-lifecycle` flag to be set when starting the Prometheus server. Use `SIGHUP` to the Prometheus process.

The format:

* `params`: The query. The start and end timestamp, the step and the actual query statement.
* `stats`: Statictics. Currently, it contains internal engine timers.
* `ts`: The timestamp.

Add logrotation since Prometheus will not do this.

### Events

Prometheus is not an event-based system. Some monitoring systems are event-based, they report each individual event (HTTP request, an exception) to a central monitoring system immediately **as it happens**. This system aggregates the events into metrics (StatsD is the prime example of this). Or stores it for later processing, like ELK stack.

In such a system, pulling would be problematic indeed: the instrumented service would have to buffer events between pulls, and the pull would have to happen incredibly frequently.

Prometheus is only interested in reguralily collecting the **current state** of a given set of metrics, not the underlying events that led to the generation of those metrics.

The resulting monitoring traffic is low, and pull-based approach does not create problems in this case.

## Tracing and Spans

Prometheus supports OpenTelemetry protocol ingestion through HTTP.

`tracing_config` configures exporting traces from Prometheus to a tracing backend via the OTLP protocol. Tracing is currently an experimental feature and could change in the future.

### Traces

A trace represents the whole journey of a request or an action as it moved through all the nodes of a distributed system.

### Spans

A span is an **operation** of work taking place on a service. E.g. a web server responding to an HTTP request.

## Push vs Pull

Pulling over HTTP offers a number of advantages:

* You can start extra monitoring instances as needed, e.g. on your laptop when developing changes.
* You can more easily and reliably tell if a target is down.
* You can manually go to a target and inspect its health with a web browser.

## Service Discovery

## Basics of SLOs, SLAs, and SLIs

</details>

<details>

  <summary>Prometheus Fundamentals (20%)</summary>

## System Architecture

![alt text](image.png)

Prometheus **scrapes** metrics from **instrumented jobs**, either directly or via an intermediary push gateway for **short-lived jobs**.

Prometheus works well for recording any purely numeric time series, it fits both machine-cetric monitoring as well as monitoring of highly dynamic service oriented archtectures.

Designed for readability.

If you need 100% accuracy, such as per request biliing, Prometheus is not a good choice as the collected data will likely not be detailed and complete enough. In that case it's better to use some other system to collect and analyze the data for billig, and Prometheus for the rest of your monitoring.

Components:

* Main Prometheus server
* Client libraries
* Push gateway
* Special purpose exporters
* an Alertmanager to handle alerts
* various support tools

## Configuration and Scraping

### Prometheus Configuration

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'node'
    static_configs:
    - targets: ['localhost:9100']
```

### Scrape configuration basics

```yaml
scrape_configs:
  - job_name: 'myapp'
    metrics_path: '/custom_metrics'
    scrape_interval: 10s
    static_configs:
      - targets: ['10.0.0.1:8080', '10.0.0.2:8080']
```

### Example: Scraping Kubernetes Pods

```yaml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        target_label: __address__
        regex: (.+):(?:\d+);(\d+)
        replacement: $1:$2
        action: replace
```

```yaml
[ Pod Annotation ]
  |
  |  __meta_kubernetes_pod_annotation_prometheus_io_scrape = "true"
  |--> KEEP
  |
  |  __meta_kubernetes_pod_annotation_prometheus_io_path = "/metrics"
  |--> SET --> __metrics_path__ = "/metrics"
  |
  |  __address__ = "10.42.0.15:8080"
  |  __meta_kubernetes_pod_annotation_prometheus_io_port = "9100"
  |--> SET --> __address__ = "10.42.0.15:9100"
  |
[ Scrape Target Ready ]
  --> GET http://10.42.0.15:9100/metrics
```

## Understanding Prometheus Limitations

## Data Model and Labels

### Metric names

* Metric names SHOULD specify the **general fature** of a system that is being measured e.g. `http_requests_total` - the total number of HTTP requests received.
* Metric names MAY use any UTF-8 characters.
* Metric names SHOULD match the regex: `[a-zA-Z_:][a-zA-Z0-9_:]*` which means they can contain letters, digits, underscores, and colons.

### Labels

Labels let you capture different instances of the same metric. For example: all HTTP requests that used the method `POST` to the `/api/tracks` handler. This is the **"dimensional data model"** of Prometheus.

The query language allows for filtering and aggregation based on these dimensions.

Any **change** of *_any_  label value, including adding or removing labels, will create a new time series.

Naming convention:

* Labels names MAY use any UTF-8 characters.
* Label names beginning with `__` (underscores) MUST be reserved for internal Prometheus use.
* Label names SHOULD match the regex `[a-zA-Z_][a-zA-Z0-9_]*` for the best experience and compatibility with other tools. Same as metric names.
* Label values MAY contain any UTF-8 characters.
* Labels with an empty label value are considered equivalent to labels that do not exist.

### Notation

Given a metric name and a set of labels, time series are frequently identified using this notation:

`<metric_name>{<label_name>=<label_value>, <label_name>=<label_value>, ...}`

For example, a time series with the metric name `api_http_requests_total` and the labels `method="POST"` and `handler="/messages"` would be represented as:

`api_http_requests_total{method="POST", handler="/messages"}`

Same notation that **OpenTSDB** uses.

## Exposition Format

_Metrics exposition in the classic Prometheus use case is dominated by strings because all the metric names, label names, and label values take **much more space than the float64 sample values**, even if the latter are represented in a potentially more verbose text form._

OpenMetrics is a specification built upon and carefully extending the Prometheus exposition format. OpenMetrics v2 is ongoing.

Metrics can be exposed to Prometheus using a **simple text-based** exposition format. There are various client libraries that implement this format for you.

Basic info:

* Transmission is done over HTTP.
* Encoding is UTF-8.
* HTTP `Content-Type` is `text/plain; version=0.0.4`. A missing version will lead to **fall-back to the most recent text format version.
* HTTP `Content-Encoding` (optional): `gzip`
* Advantages:
  * Human-readable
  * Easy to assemble
  * Readable line by line
* Limitations:
  * Verbose
  * Docstrings not integral part of the syntax no or little metric contract validation
  * Parsing cost
* Supported metric primitives
  * Counter
  * Gauge
  * Histogram
  * Summary
  * Untyped

</details>

<details>

  <summary>PromQL (28%)</summary>

## Selecting Data

## Rates and Derivatives

## Aggregating over time

## Aggregating over dimensions

## Binary operators

## Histograms

## Timestamp Metrics

</details>

<details>

  <summary>Instrumentation and Exporters (16%)</summary>

## Client Libraries

## Instrumentation

## Exporters

## Structuring and naming metrics

</details>

<details>

  <summary>Alerting and Dashboarding (18%)</summary>

## Dashboarding basics

## Configuring Alerting rules

## Understand and Use Alertmanager

## Alerting basics (when, what, and why)

</details>
