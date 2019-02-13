+++
title = "Metrics"
description = "Monitor Watermill in realtime"
date = 2019-02-12T21:00:00+01:00
weight = -200
draft = false
bref = "Monitor Watermill in realtime using Prometheus"
toc = true
+++

### Metrics

Monitoring of Watermill may be performed by using decorators for publishers/subscribers and middlewares for handlers. 
We provide a default implementation using Prometheus, based on the official [Prometheus client](https://github.com/prometheus/client_golang) for Go.

The `components/metrics` package exports `PrometheusMetricsBuilder`, which provides convenience functions to wrap publishers, subscribers and handlers so that they update the relevant Prometheus registry:

{{% render-md %}}
{{% load-snippet-partial file="content/src-link/components/metrics/builder.go" first_line_contains="// PrometheusMetricsBuilder" last_line_contains="func (b PrometheusMetricsBuilder)" %}}
{{% /render-md %}}

### Wrapping publishers, subscribers and handlers

If you are using Watermill's [router](/docs/messages-router) (which is recommended in most cases), you can use a single convenience function `AddPrometheusRouterMetrics` to ensure that all the handlers added to this router are wrapped to update the Prometheus registry, together with their publishers and subscribers:

{{% render-md %}}
{{% load-snippet-partial file="content/src-link/components/metrics/builder.go" first_line_contains="// AddPrometheusRouterMetrics" last_line_contains="AddMiddleware" padding_after="1" %}}
{{% /render-md %}}

Example use of `AddPrometheusRouterMetrics`:

{{% render-md %}}
{{% load-snippet-partial file="content/src-link/_examples/metrics/main.go" first_line_contains="prometheusRegistry :=" last_line_contains="AddPrometheusRouterMetrics(" %}}
{{% /render-md %}}

Standalone publishers and subscribers may also be decorated through the use of dedicated methods of `PrometheusMetricBuilder`:

{{% render-md %}}
{{% load-snippet-partial file="content/src-link/_examples/metrics/main.go" first_line_contains="subWithMetrics, err := " last_line_contains="pubWithMetrics, err := " padding_after="3" %}}
{{% /render-md %}}

### Exposing the /metrics endpoint

In accordance with how Prometheus works, the service needs to expose a HTTP endpoint for scraping. By convention, it is a GET endpoint, and its path is usually `/metrics`.

To serve this endpoint, there are two convenience functions, one using a previously created Prometheus Registry, while the other also creates a new registry:

{{% render-md %}}
{{% load-snippet-partial file="content/src-link/components/metrics/http.go" first_line_contains="// CreateRegistryAndServeHTTP" last_line_contains="func ServeHTTP(" %}}
{{% /render-md %}}

Here is an example of its use in practice:

{{% render-md %}}
{{% load-snippet-partial file="content/src-link/_examples/metrics/main.go" first_line_contains="prometheusRegistry :=" last_line_contains="defer closeMetricsServer()" %}}
{{% /render-md %}}

### Grafana dashboard

We have prepared a [Grafana dashboard](https://grafana.com/dashboards/9777) to use with the metrics implementation described above. It provides basic information about the throughput, failure rates and publish/handler durations.
For a more detailed description of the metrics, see [below](#exported-metrics).

**TODO: replace with files from `master`**

<a target="_blank" href="https://gitlab.com/threedotslabs/threedots.tech/raw/grafana-dashboard/static/watermill-io/grafana_dashboard.png"><img src="https://gitlab.com/threedotslabs/threedots.tech/raw/grafana-dashboard/static/watermill-io/grafana_dashboard_small.png" /></a>

### Exported metrics

Listed below are all the metrics that are registered on the Prometheus Registry by `PrometheusMetricsBuilder`.
 
For more information on Prometheus metric types, please refer to [Prometheus docs](https://prometheus.io/docs/concepts/metric_types).
 
<table>
  <tr>
    <th>Object</th>
    <th>Metric</th>
    <th>Description</th>
    <th>Labels/Values</th>
  </tr>
  <tr>
    <td rowspan="3">Subscriber</td>
    <td rowspan="3"><code>subscriber_messages_received_total</code></td>
    <td rowspan="3">A Prometheus Counter.<br>Counts the number of messages obtained by the subscriber.</td>
    <td><code>acked</code> is either "acked" or "nacked".</td>
  </tr>
  <tr>
    <td><code>handler_name</code> is set if the subscriber operates within a handler; "&lt;no handler&gt;" otherwise.</td>
  </tr>
  <tr>
    <td><code>subscriber_name</code> identifies the subscriber. If it implements <code>fmt.Stringer</code>, it is the result of `String()`, <code>package.structName</code> otherwise.</td>
  </tr>
  <tr>
    <td rowspan="2">Handler</td>
    <td rowspan="2"><code>handler_execution_time_seconds</code></td>
    <td rowspan="2">A Prometheus Histogram. <br>Registers the execution time of the handler function wrapped by the middleware.</td>
    <td><code>handler_name</code> is the name of the handler.</td>
  </tr>
  <tr>
    <td><code>success</code> is either "true" or "false", depending on whether the wrapped handler function returned an error or not.</td>
  </tr>
  <tr>
    <td rowspan="3">Publisher</td>
    <td rowspan="3"><code>publish_time_seconds</code></td>
    <td rowspan="3">A Prometheus Histogram.<br>Registers the time of execution of the Publish function of the decorated publisher.</td>
    <td><code>success</code> is either "true" or "false", depending on whether the decorated publisher returned an error or not.</td>
  </tr>
  <tr>
    <td><code>handler_name</code> is set if the publisher operates within a handler; "&lt;no handler&gt;" otherwise.</td>
  </tr>
  <tr>
    <td><code>publisher_name</code> identifies the publisher. If it implements <code>fmt.Stringer</code>, it is the result of `String()`, <code>package.structName</code> otherwise.</td>
  </tr>
</table>

Additionally, every metric has the `node` label, provided by Prometheus, with value corresponding to the instance that the metric comes from.

### Customization

If you feel like some metric is missing, you can easily expand this basic implementation. The best way to do so is to use the prometheus registry and register a metric according to [the documentation](https://godoc.org/github.com/prometheus/client_golang/prometheus). 

An elegant way to update these metrics would be through the use of decorators:

{{% render-md %}}
{{% load-snippet-partial file="content/src-link/message/decorator.go" first_line_contains="// MessageTransformSubscriberDecorator" last_line_contains="type messageTransformSubscriberDecorator" %}}
{{% /render-md %}}

and/or [router middlewares](/docs/messages-router/#middleware). 

A more simplistic approach would be to just update the metric that you want in the handler function.

### TODO: 
1. maybe something about the dev environment and/or how to configure Prometheus/Grafana?
1. replace the image with `master` after it is merged
1. how to import the dashboard (although it is described in detail in the link to the dashboard)?