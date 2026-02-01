# Pre-Read Notes: Building Cost & Latency Dashboards

## 1) What You'll Learn

- Understand what a cost & latency dashboard is and what problems it solves
- Learn the core components you’ll need: instrumentation, a data store, a dashboard tool, and alerts
- Grasp latency concepts like P95 and P99, and how histograms help visualize tail latency
- Learn how to combine cost data with request data to get cost per query (and ROI)
- Explore practical steps to set up anomaly alerts for unexpected spend or latency spikes

---

## 2) Detailed Explanation

This section follows a friendly, step-by-step style. It uses simple analogies and concrete examples to build understanding without overwhelming you.

### A. Introduction: What Is a Cost & Latency Dashboard?

Analogy: Imagine you’re driving a car. Your dashboard shows fuel usage, speed, and engine health in real time. A cost & latency dashboard does the same for a software service: it shows how much money you’re spending per request (cost per query) and how long each request takes (latency). The idea is to give you a quick, real-time snapshot of both financial and performance health in one place.

One simple way to think about it in code (pseudo):
- Collect metrics: latency_ms per request, request_count, and cost_usd per service
- Store and query: time-series data by service/endpoints
- Visualize: dashboards with latency histograms and cost charts
- Alert: notify when latency or cost crosses thresholds

Technical note (brief): You’ll typically instrument code to emit metrics, export them to a time-series store (Prometheus or OpenTelemetry-friendly backends), and visualize with a dashboard tool (Grafana). Latency is often captured as a histogram to support percentile calculations like P95 and P99.

### B. Why This Matters

Three clear benefits:
- Better budgeting and cost control: you see which services drive spend and when costs spike.
- Improved user experience: by tracking latency, you can spot slow endpoints and fix bottlenecks before users notice.
- Faster troubleshooting: correlated signals (cost + latency + traces) help you pinpoint root causes quickly.

You’re solving three common problems with one view:
- Unclear cost attribution: you can see which service or endpoint drives spend.
- Tail latency surprises: P95/P99 show how many requests land in the tail, not just the average.
- Reactive firefighting: alerts give you early warnings before incidents escalate.

### C. Building Understanding: From Known to New

Painful, pre-dashboard pattern: You’d guess which endpoint is expensive or slow by looking at logs one by one. It’s noisy, slow, and misses the big picture.

New pattern with dashboards: You collect structured metrics from all services, unify them in a single view, and filter by service, endpoint, or time window. The dashboard lets you see, at a glance, which requests take the longest and which generate the most cost—both now and over time.

Technical intuition: start with histograms for latency, use percentile functions to derive P95/P99, and join that with cost signals by service to compute cost per query.

### D. Core Components

- Data Collection (OpenTelemetry instrumentation) — The basic source of truth. You emit metrics like:
  - Counter: requests_total
  - Histogram: request_duration_ms
  - Gauge: current_queue_length
  Simple micro-example: instrument your HTTP handler to record duration and increment request counts per endpoint.

- Data Storage & Export (Prometheus, OpenTelemetry backends) — Stores time-series data so dashboards can query it quickly. Histograms let you build percentiles without storing every sample.

- Visualization (Grafana) — The main UI. Panels show latency distributions, counts, costs, and per-endpoint breakdowns. Use shared dashboards and templates to stay consistent.

- Cost Signals (Cloud cost data + service tagging) — Bring in cost data from your cloud provider, tagging resources by service or tenant. This is how you show “cost per query” per service.

- Alerts & Anomalies (P95, P99 thresholds; cost anomalies) — Proactive notifications when latency tails widen or spend grows unexpectedly. Alerts keep you from chasing fires.

Micro-examples for each:
- Data Collection: http_server_request_duration_ms_histogram by endpoint
- Data Storage: Prometheus metric store with a histogram bucket layout
- Visualization: Grafana panel showing P95 latency by service
- Cost Signals: cloud_cost_usd_total by service
- Alerts: alert when p99_latency_ms > 1s or daily_cost_usd > budget_limit

### E. Step-by-Step Process

1) Instrument services with OpenTelemetry
- Add a histogram for latency (e.g., http.server.request.duration_ms)
- Increment a counter for total requests per endpoint/service
- Attach useful attributes like http_method, route, and service name

2) Export metrics to a time-series store
- Use Prometheus as a data source or an OTLP exporter to a backend that Grafana can query
- Ensure histogram data is bucketed so you can calculate quantiles efficiently

3) Bring in cost signals
- Tag cloud resources by service; export or feed cloud cost data as a metric (cost_usd) per service
- If available, use a cloud cost exporter or API to pull daily/ten-minute cost figures

4) Build dashboards in Grafana
- Latency panels: show P95 and P99 by service/endpoint
- Cost panels: show cost per query (cost_usd / requests) by service
- Use template variables to filter by service, environment, or time range
- Apply transformations to join latency data with cost data where helpful

5) Set up alerts
- Latency alerts: alert if P99 latency exceeds a target for a sustained period
- Cost alerts: alert if daily spend or cost-per-query spikes unexpectedly

6) Iterate and refine
- Add more panels as you learn which signals matter most
- Downsample long time ranges for speed; limit panels to avoid overloading dashboards
- Consider using exemplars or traces to deepen context around outliers

### F. Key Features

- Percentile-based latency visualization (P95, P99) by service
  - Why it matters: users care about tail latency more than average latency
  - Simple example: p95_latency_ms = percentile(0.95, latency_ms_histogram)

- Cost per query (and ROI) calculations
  - Why it matters: shows how much each request “costs” and helps justify optimizations
  - Simple example: cost_per_query = total_cost_usd / total_requests

- Cost anomaly alerts
  - Why it matters: catches runaway spend before it surprises you
  - Simple example: alert when cost_usd > budget_limit or when rate of cost growth exceeds a threshold

### G. Putting It All Together

ONE complete example (concise, code-ish):

- Prerequisites
  - OpenTelemetry-instrumented HTTP service emitting:
    - http_server_request_duration_ms_histogram
    - requests_total counter
  - Cloud cost signal by service: cloud_cost_usd_total{service="checkout"}
  - Grafana with Prometheus data source connected

- Minimal complete workflow (10–15 lines of code-style guidance)
  - PromQL for latency: histogram_quantile(0.95, sum(rate(http_server_request_duration_ms_bucket{service="checkout"}[5m])) by (le)))
  - PromQL for cost: sum(rate(cloud_cost_usd_total{service="checkout"}[5m])) 
  - Cost per query panel: cost_rate / request_rate
  - Example in Grafana: 
    - Panel A: P95 latency by service (using histogram_quantile)
    - Panel B: Cost per query by service (cost_per_time_unit / requests_per_time_unit)
  - Simple alert rule (pseudo):
    - If latency_p95 > 0.8s for 10 minutes, trigger alert
    - If cost_per_query spikes by >20% day-over-day, trigger alert

This compact example shows how you tie together the latency histogram, per-service cost signal, and a per-query metric into a single, coherent view. It’s the essence of “cost per request” dashboards: you’re measuring how much each request costs while watching how long it takes to handle that request.

---

## 3) Practice Exercises

1) Pattern Recognition
- Look at a sample dashboard that has latency by endpoint and total cost per service. Identify which panel would most likely reveal tail-latency issues and why.

2) Concept Detective
- Given the terms: histogram, percentile, P95, P99, and PromQL. Explain in simple terms what each one means and how they help you understand latency.

3) Real-Life Application
- List 3 real-world scenarios where a cost-per-query dashboard would help you decide where to optimize (e.g., a particular endpoint, a payment service, a data-fetching API).

4) Spot the Error
- You see a panel showing P99 latency but the data source is a simple counter (requests_total) with no latency histogram. Why would that be wrong, and how would you fix it?

5) Planning Ahead
- Imagine you’re adding a new microservice to your system. What are the first three signals you would instrument and visualize to ensure you can track cost and latency from day one?

---

## Quick Reference: Key Terms Defined

- **Latency**: The time it takes to finish a single request or operation.
- **P95 / P99**: The 95th and 99th percentile latency. They represent the value below which 95% or 99% of requests fall.
- **Histogram**: A data structure that groups observations into buckets, helping you calculate quantiles like P95/P99.
- **OpenTelemetry**: A standard for collecting telemetry data (metrics, logs, traces) from applications.
- **Prometheus**: A time-series database and query engine often used for metrics collection and alerting.
- **Grafana**: A visualization tool that uses data from Prometheus and other sources to build dashboards.
- **Cost per query**: A computed metric that divides total cost by the number of queries, giving a per-request cost.
- **Anomaly alerts**: Alerts triggered by unusual or unexpected changes in metrics (e.g., sudden cost spikes, tail latency increases).

---

## Tips for Beginners

- Start simple: instrument just a few key endpoints and a single latency histogram.
- Use templates: Grafana dashboards often have starter templates you can adapt.
- Keep labels clean: use stable names for services and endpoints to avoid messy dashboards.
- Limit data volume: for long time ranges, downsample or reduce the number of panels to keep dashboards responsive.
- Think in terms of user impact: tail latency (P95/P99) often correlates with user-visible slowdowns more than averages.

---