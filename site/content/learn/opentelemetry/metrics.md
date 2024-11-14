---
title: What are OpenTelemetry Metrics?
displayTitle: An introduction to OpenTelemetry Metrics
navTitle: Metrics
description: OpenTelemetry Metrics play a critical role in monitoring applications by offering a way to capture and analyze key metrics in a standardized, scalable manner. Whether you're managing a complex microservices architecture or a simpler system, OpenTelemetry helps track essential statistics that reveal the health and performance of your services.
date: 2024-10-18
author: Nocnica Mellifera
githubUser: serverless-mom
displayDescription: 
  Learn more about OpenTelemetry & Monitoring with Checkly. Explore metrics, one of the three pillars of observability.
menu:
  learn:
    parent: "OpenTelemetry"
weight: 3
---
**OpenTelemetry Metrics** play a critical role in monitoring applications by offering a way to capture and analyze key metrics in a standardized, scalable manner. Whether you're managing a complex microservices architecture or a simpler system, OpenTelemetry helps track essential statistics that reveal the health and performance of your services.

---

## What are Metrics?

Metrics represent **quantitative measurements** of your system’s health and behavior. They provide insights into performance trends, such as:

- **CPU usage** over time
- **Request rates** per endpoint
- **Error counts** or failure rates
- **Latency** in handling requests

Metrics are lightweight and highly efficient to collect, aggregate, and query. They help identify patterns and anomalies without burdening storage, making them suitable for continuous monitoring at scale.

### Types of Metrics in OpenTelemetry:

- **Counter**: Measures occurrences or events, such as the number of requests handled.
- **Gauge**: Captures values that fluctuate, like memory usage.
- **Histogram**: Measures the distribution of values, such as response time percentiles.

Explore further in the [OpenTelemetry Metrics Documentation](https://opentelemetry.io/docs/concepts/signals/metrics/).


## Why Metrics Matter

In a **microservices** environment, metrics are indispensable for:

- **Performance monitoring**: Identifying bottlenecks or degraded performance.
- **Capacity planning**: Forecasting when additional resources are required.
- **Incident detection**: Alerting teams about abnormal system behavior.

Metrics are often **the first step** in identifying that something has gone wrong. If a metric shows unusual values (e.g., a spike in response time), you can investigate further by drilling into traces or logs to find the root cause.

## Metrics vs. Traces

Metrics have a number of advantages over tracing. Metrics are much more data efficient, generally at the collector level it’s possible to compress hundreds of individual metrics reported to a single packet of data sent on to the metrics backend. Further, metrics show broad trends whereas a trace, no matter how interesting, will always cover only a single request.

Should you use metrics instead of traces to monitor your service? Absolutely not. Metrics will always present average performance, and the specific information needed to really understand root causes will be elusive. Further, even with high resolution timeseries metrics it’s very hard to go from worrying metrics to find matching log data of a problem. Finally, modern traces can effectively show information about asynchronous requests as they contribute to overall request time, something that’s very hard to tease out of bare metrics.

## Setting up OpenTelemetry Metrics

### Auto-Instrumentation vs. Manual Instrumentation

1. **Auto-Instrumentation**: Many popular frameworks and libraries come with automatic OpenTelemetry instrumentation, requiring minimal setup.
2. **Manual Instrumentation**: Developers can manually add metrics within the application code by using SDKs to track specific business metrics (e.g., purchases per hour).

Learn more about instrumentation options in the [OpenTelemetry SDK Guide](https://opentelemetry.io/docs/instrumentation/).

## Example Metric Pipeline

With OpenTelemetry, you can collect, process, and export metrics using **Collectors**. Here’s a high-level example of a typical metric pipeline:

1. **Data Collection**: Metrics are generated by instrumented services.
2. **Processing**: The OpenTelemetry Collector aggregates and processes the data (e.g., batching or filtering metrics).
3. **Exporting**: Metrics are sent to observability platforms like **Prometheus** or **Grafana**.

Learn how to configure a collector in the [OpenTelemetry Collector Guide](/learn/opentelemetry/what-is-the-otel-collector/).



## Best Practices for Metrics in OpenTelemetry

- **Optimize cardinality**: Avoid creating too many distinct labels, as this can overwhelm storage and query systems.
- **Set appropriate aggregation intervals**: Batch data intelligently to balance between real-time insights and system load.
- **Use meaningful names**: Clearly describe the purpose of each metric to make dashboards and alerts easier to understand.
- **Standardize naming early**: While OpenTelemetry defines standard language for a number of concepts, actual metric naming is not standardized. As such it's possible to report `total-web-shop-checkout-time` and `webShopCheckoutTime_total` as two totally separate metrics even though they should be aggregated. No standard is perfect, of course, and to normalize data before it's stored, use the [filtering tools in the OpenTelemetry collector](/learn/opentelemetry/otel-filtering/).


OpenTelemetry metrics provide a robust foundation for observability, helping teams proactively monitor performance and detect issues before they escalate. With the right setup and tooling, you can gain comprehensive insights into your applications, enabling faster resolution times and improved reliability.