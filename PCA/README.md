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

## Understand logs and events

## Tracing and Spans

## Push vs Pull

## Service Discovery

## Basics of SLOs, SLAs, and SLIs

</details>

<details>

  <summary>Prometheus Fundamentals (20%)</summary>

* System Architecture
* Configuration and Scraping
* Understanding Prometheus Limitations
* Data Model and Labels
* Exposition Format

</details>

<details>

  <summary>PromQL (28%)</summary>

* Selecting Data
* Rates and Derivatives
* Aggregating over time
* Aggregating over dimensions
* Binary operators
* Histograms
* Timestamp Metrics

</details>

<details>

  <summary>Instrumentation and Exporters (16%)</summary>

* Client Libraries
* Instrumentation
* Exporters
* Structuring and naming metrics

</details>

<details>

  <summary>Alerting and Dashboarding (18%)</summary>

* Dashboarding basics
* Configuring Alerting rules
* Understand and Use Alertmanager
* Alerting basics (when, what, and why)

</details>
