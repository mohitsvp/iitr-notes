# Lecture Notes: Building Cost & Latency Dashboards

## 1. What You’ll Learn

In this lesson, you’ll learn to:
- Explain how to instrument services to collect per-request latency and cost data, and visualize those data together in a real-time dashboard.
- Build latency dashboards that show percentile-based metrics (P95, P99) and explain why percentiles matter for user experience.
- Compute cost-per-query (cost per request) using logs and metrics, and translate that into a practical ROI view.
- Set up cost anomaly alerts and latency anomaly alerts so you can act quickly on outliers.
- Integrate OpenTelemetry for cohesive traces, metrics, and logs, and connect them to Grafana/Prometheus dashboards.
- Create a scalable dashboard design that can grow with new services, regions, or pricing models (tags, quotas, budgets).

---

## 2. Detailed Explanation

### a. Intro: What Is a Cost & Latency Dashboard?

Think of a dashboard as the dashboard of a car. You want to see two crucial gauges at a glance: speed (how fast you are delivering requests) and fuel (how much money you’re spending to operate those services). Your cost dashboard tells you how much each request costs to run, while your latency dashboard tells you how long each request takes. When you combine them, you can answer questions like: “If our API handles 1,000 requests per second, what is the daily cost, and what’s the worst latency users might experience?”

In practice, a cost & latency dashboard collects data from multiple sources:
- Telemetry about requests: which service handled the request, which endpoint, and how long it took.
- Cost signals: how much the underlying compute, storage, network, and third-party services cost in the same time window.
- Contextual data: which region, environment (prod/stage), and which customer or plan is involved.

The ingestion path typically looks like this: instrumented services emit traces and metrics → a collector (OpenTelemetry Collector or similar) exports metrics and traces to a time-series database (Prometheus, or Prometheus-compatible storage) and a tracing backend → Grafana or another visualization layer reads from these sources and builds dashboards. Alerts can be configured to trigger when metrics breach thresholds or when anomalies are detected.

Key ideas to remember:
- Latency measurements should be distributed, not just averages. Percentiles (P50, P95, P99) reveal tail behavior where a small portion of requests can dominate user perception.
- Cost data must be tied to a clear unit of analysis, such as cost per query or cost per endpoint, so you can understand financial impact per user experience.
- Correlation is king. When you see high latency, you want to know if the cost also changes at the same time, and if a particular service or endpoint is driving both.

#### Why percentiles rather than only averages?
- Averages can mask tail latency. If most requests are fast but a small percentage are very slow, users experience those delays. Percentiles tell you what 95% or 99% of requests look like, which maps more directly to user experience.

---

### b. Why It Matters

- Real-time visibility helps you detect and respond to issues before users are affected.
- Per-request cost visibility helps teams optimize for cost efficiency without sacrificing performance.
- Percentile latency dashboards help you prioritize optimizations for the most impactful tail cases (the slowest requests).
- Anomaly alerts reduce MTTR (mean time to resolve) by surfacing unusual patterns in cost and latency early.

Practical use cases:
- A sudden spike in latency alongside a spike in cost signals a potential misconfiguration (e.g., a new feature causing more work per request) or a runaway dependency (e.g., a downstream API behaving abnormally).
- A region experiencing higher per-request costs may indicate capacity constraints, poor autoscaling behavior, or egress pattern changes.
- You want to ensure service-level objectives (SLOs) for latency are met while keeping cost within budget, and you need automatic alerts when either dimension goes off track.

---

### c. Detailed Walkthrough

Below is a practical, end-to-end walkthrough you can follow, with concrete decisions, data models, and example queries. The goal is a repeatable pattern you can apply to many services.

1) Instrumentation: collect metrics and traces per request
- What to collect
  - Request metadata: service, endpoint, region, environment, user tier (if permissible).
  - Latency: a histogram bucketized by latency in milliseconds (or microseconds).
  - Throughput: requests per second or per minute (QPS or RPS).
  - Cost signals: estimated cost per request derived from underlying resource usage (compute time, memory, storage, external service usage), possibly using pricing data and resource usage counters.
  - Optional: user satisfaction proxies (e.g., error rate, retry rate).

- How to instrument
  - Use OpenTelemetry (OTel) for unified instrumentation: traces, metrics, and logs.
  - For latency, prefer histograms. Histograms let you compute P95, P99 later.
  - For cost per request, attach a cost tag to the metric, so you can filter and aggregate by service, endpoint, or region.

- Simple example (Go-style pseudocode)
  - Instrument a request handler to record start time and duration, plus tags.

  Code sketch (Go-like):
  - func handler(w, r) {
      start := time.Now()
      ctx := r.Context()
      // process request
      // after processing
      durMs := time.Since(start).Milliseconds()
      // emit metric
      meter := otel.Meter("service-a")
      hist := metric.Must(meter).NewFloat64Histogram("request_latency_ms")
      hist.Record(ctx, float64(durMs), attribute.String("endpoint", r.URL.Path), attribute.String("service", "service-a"), attribute.String("region", "us-west-1"))

      // estimate cost per request (example)
      // assume you have a function that assigns cost per ms of compute
      costPerMs := estimateCostPerMs("service-a", region)
      c := float64(durMs) * costPerMs
      histCost := metric.Must(meter).NewFloat64Histogram("request_cost_usd")
      histCost.Record(ctx, c, attribute.String("endpoint", r.URL.Path))
  }

- Notes
  - The per-request cost should reflect the actual resources consumed. If you are serverless, you can map invocation duration and memory usage to a cost proxy. If you are in a Kubernetes cluster, map CPU-core-seconds, memory-seconds, and any external services’ costs into a per-request proxy.

2) Data collection and transport
- OpenTelemetry Collector
  - Use the OpenTelemetry Collector to receive traces and metrics, process them (batching, attribute normalization), and export them to the data sinks you choose (Prometheus-compatible metric storage, Loki for logs, Tempo for traces, etc.).
  - Typical pipeline: OTLP -> Collector -> Prometheus (metrics), Tempo (traces), Loki (logs).
- Data sinks
  - Metrics: Prometheus or a Prometheus-compatible time-series store.
  - Traces: Tempo or Jaeger, depending on your stack.
  - Logs: Loki or a similar log aggregation system.
- Data shaping
  - Use consistent semantic conventions for naming: latency_ms_bucket (histogram), request_latency_ms (metric name), and cost_usd (cost per request metric).
  - Use labels (tags) sparingly but meaningfully: service, endpoint, region, environment, version, and perhaps tenant or customer segment if privacy and governance allow.

3) Visualization: Grafana as the cockpit
- Data sources
  - Prometheus for metrics
  - Tempo for traces
  - Loki for logs
- Panels to create
  - Latency distribution panel
    - Type: histogram or heatmap
    - Data: request_latency_ms_bucket (histogram)
    - Purpose: visualize latency distribution and compute percentile values
  - P95 and P99 panels
    - Data: histogram_quantile(0.95, sum(rate(request_latency_ms_bucket[5m])) by (le))
      and histogram_quantile(0.99, sum(rate(request_latency_ms_bucket[5m])) by (le))
  - Cost per request panel
    - Data: request_cost_usd histogram
    - Derived metric: cost_per_request = sum(request_cost_usd) / sum(requests) (where requests can be a separate metric)
  - Throughput panel
    - Data: rate(requests_total[1m])
  - ROI/Cost-Per-Query panel
    - Data: cost_per_request vs throughput (to visualize cost efficiency)
    - Optionally, plot ROI proxy: value_per_request_savings / cost_per_request
  - Per-service cost panel
    - Data: sum by(service) of request_cost_usd
  - Time-to-first-byte (TTFB) or upstream latency panel
    - If you have trace data, you can surface TTFB as a derived metric
  - Anomaly status panel
    - Integrate anomaly alerts (thresholds or ML-based) and display color-coded status

4) Calculating cost per query (per-request cost)
- Core idea
  - You want a simple, defensible unit: how much does each request cost to serve?
  - Approach: assign a cost metric per request by aggregating resource usage across the request path, and/or by attributing cloud spend to requests by tags.
- How to do it
  - Tag each metric with service, endpoint, region, and deployment tag.
  - Attach a cost attribution map per resource:
    - Compute: CPU-time, memory usage
    - Storage: data transferred, storage footprint
    - Network: egress/ingress per region
    - External services: API call costs (e.g., third-party APIs)
  - Estimate cost per resource usage unit using pricing data from cloud providers.
  - For serverless or auto-scaled environments, map price per millisecond or per 100ms with memory size to compute a per-request cost.
- Example approach
  - Per-request cost = (compute_time_ms × cost_per_ms) + (memory_usage_MB × cost_per_MB) + (data_transferred_MB × data_cost_per_MB) + (external_service_cost_per_request)
  - Expose a metric: request_cost_usd
  - Use a histogram to capture distribution of per-request costs for anomaly detection and percentile-based dashboards
- Simple example (pseudo)
  - cost_per_ms_by_service := function(service, region) { return price_per_cpu_ms(region) * cpu_seconds_per_ms(...) }
  - request_cost_usd = sum(cost_per_ms_by_service(service, region) for all resources used by this request)

5) Visualizing P95 and P99 latency
- Why histogram_quantile?
  - A histogram captures the distribution of latency with bucket boundaries, letting you compute percentiles across a cluster of instances.
- PromQL-ish queries (Prometheus-compatible)
  - P95: histogram_quantile(0.95, sum(rate(request_latency_ms_bucket[5m])) by (le, service, endpoint))
  - P99: histogram_quantile(0.99, sum(rate(request_latency_ms_bucket[5m])) by (le, service, endpoint))
  - If you need global percentiles across services/endpoints, you can aggregate by fewer labels:
    - histogram_quantile(0.95, sum(rate(request_latency_ms_bucket[5m])) by (le))
- Visualization tips
  - Show a main time-series panel for P95 and P99 for the entire API, with a second panel showing per-endpoint percentiles to spot hotspots.
  - Use color to indicate health: green for healthy, yellow for warning, red for critical.
  - Add a density/heatmap panel to show latency distribution at a glance.
  - Keep the time window reasonable (5–15 minutes for real-time, longer for historical trend analysis).

6) OpenTelemetry: the glue for traces, metrics, and logs
- Why OpenTelemetry?
  - It provides a standard, vendor-agnostic way to collect traces, metrics, and logs, and it helps you correlate signals through a common trace identifier.
- Key components
  - Instrumentation libraries in your languages (instrument code to emit spans and metrics)
  - OpenTelemetry Collector to collect and export
  - Semantic conventions for naming and attributes
- Correlation patterns
  - Ensure trace_id, span_id propagate into metrics and logs so you can link a timing metric to a specific trace and the related log events.
- Simple OTLP ingestion example
  - Instrument your app to export metrics to the collector via OTLP
  - Collector forwards to Prometheus (metrics) and Tempo (traces), and Loki (logs)
- Practical benefits
  - You can drill from a failing request’s trace to the exact latency panel and to the per-request cost signal, enabling faster root cause analysis.

7) Alerting: cost and latency anomalies
- Cost anomaly alerts
  - Goal: catch unusual spikes in cost, not just high cost in general
  - Approaches
    - Threshold-based alerting: alert when cost_per_request or total_cost crosses a fixed threshold
    - ML-based anomaly detection: use anomaly detectors (e.g., Random Cut Forest) available in cloud offerings to detect unusual patterns
  - Tools
    - Grafana Cloud: supports usage and cost alerts, with forecasting and anomaly detection
    - AWS Managed Prometheus: anomaly detection capabilities for metrics
- Latency anomaly alerts
  - Alert on P95 or P99 percentile exceeding a threshold
  - Use rolling baselines: detect when percentile deviates from historical norms
  - Consider separate rules for different endpoints or services
- Practical alerting workflow
  - Alert rules should be actionable and include clues (which service, endpoint, region)
  - Tie alerts to a runbook or incident response plan
  - Use multi-channel notifications (pager, email, chat) and auto-assign to on-call teams
- Example alert idea
  - If P95 latency for /checkout exceeds 1.5x historical P95 for 10 minutes, trigger alert
  - If total daily cost exceeds budget, trigger a cost alert
- References to a few approaches
  - Anomaly detection in Prometheus-based setups is supported by ML-based detectors
  - Cost anomaly detection can be provided by cloud-native tools or third-party platforms, including solutions designed to monitor multi-provider spend

8) Data integration patterns: per-service tagging and cost allocation
- Cost allocation tags
  - Attach tags to cloud resources (e.g., cost center, service name, environment) and propagate these tags to the metrics
  - Use tag-based aggregation in dashboards to split cost by service or by endpoint
- Per-service dashboards
  - Build dashboards that show cost per service, per endpoint, and per region
  - Drill down to per-request cost on demand
- Data fusion
  - Merge cost data with latency data on common keys (service, endpoint, region) to correlate performance with cost

9) Real-world design patterns and considerations
- Data retention and downsampling
  - Keep recent data at high resolution; downsample older data to moderate resolution to keep dashboards responsive
- Query performance
  - Keep panel queries efficient: downsample time ranges for long histories, limit the number of series per panel
- Privacy and governance
  - Avoid leaking sensitive user identifiers in metrics or logs; use aggregated or pseudonymized data
- Scale and resilience
  - Plan for multi-region deployments: ensure time synchronization, consistent dashboards, and data availability
- Security and access control
  - Restrict who can view cost dashboards and who can modify alert rules
  - Use role-based access control and audit trails

10) Practical step-by-step setup outline
- Step 1: Instrument your services with OpenTelemetry
  - Add histograms for latency: request_latency_ms_bucket with appropriate bucket boundaries
  - Add a per-request cost metric: request_cost_usd
  - Add labels/tacts: service, endpoint, region, environment
- Step 2: Set up the OpenTelemetry Collector
  - Configure to receive OTLP data from instrumented services
  - Export metrics to Prometheus, traces to Tempo, logs to Loki
- Step 3: Set up data stores
  - Prometheus for metrics
  - Tempo for traces
  - Loki for logs
- Step 4: Connect Grafana
  - Add data sources: Prometheus, Tempo, Loki
  - Create dashboards for latency (P95, P99), cost per request, per-service cost, and ROI
- Step 5: Define alerting rules
  - Latency alerts based on P95 and P99
  - Cost alerts for spikes or anomalies
  - Ensure alert routing and escalation policies
- Step 6: Validate end-to-end
  - Generate synthetic load and verify latency percentiles, cost signals, and alerts respond correctly
- Step 7: Iterate
  - Add more endpoints, refine histograms, tune bucket boundaries, adjust alert thresholds

---

### d. Other Add-Ons

- Common confusions
  - Confusing average latency with percentile latency. Always supplement with P50, P95, P99 views.
  - Treating cost per request as a single global value. Costs differ by service, region, and resource mix; use tagged aggregation to reveal the real story.
- Tips for better dashboards
  - Keep a small number of panels per page to avoid browser performance issues.
  - Use transformations in Grafana to join panels when necessary (e.g., combine latency and cost panels by common labels).
  - Use templating and variables for service, region, and environment to make dashboards reusable across teams.
- Cautions
  - When downsampling time series, ensure you do not lose critical percentile accuracy. Bucket boundaries should reflect typical latency ranges.
  - Ensure anomaly detection models have enough historical data to learn normal behavior.

---

## 3. Key Takeaways

- Build a real-time cockpit for your services by combining latency (percentiles) with cost metrics at per-request granularity.
- Use histograms to capture latency distributions and compute P50, P95, and P99. Histograms enable accurate percentile calculations across a fleet of services.
- Attach a cost metric per request and aggregate by service/endpoint/region to understand cost per query and ROI. This requires careful tagging and a thoughtful model tying resource usage to dollars.
- OpenTelemetry provides a cohesive path to collect traces, metrics, and logs, enabling rich correlation between latency and cost signals.
- Grafana, Prometheus, Tempo, and Loki form a powerful stack to visualize, alert, and act on both performance and cost signals in real time.
- Anomaly alerts (both cost and latency) should be proactive, actionable, and integrated with runbooks or incident response workflows.
- Start simple: instrument a few core services, build a minimal latency and cost dashboard, then incrementally add endpoints, regions, and more complex cost attribution.

---

## 4. Practical Examples and Snippets

To help you translate theory into practice, here are compact, beginner-friendly examples you can adapt to your stack.

- Example 1: Simple latency histogram metric (pseudo-Go)
  - Purpose: record per-request latency into a histogram
  - Metric: request_latency_ms_bucket
  - Labels: service, endpoint, region
  - Pseudo-code:
    - latency := measureRequestLatency()
    - recordHistogram("request_latency_ms_bucket", latency, labels={"service": "orders", "endpoint": "/checkout", "region": "us-west-1"})

- Example 2: Simple per-request cost metric
  - Purpose: attribute a dollar value to each request
  - Metric: request_cost_usd
  - Pseudo-code:
    - cost := estimateCostForRequest(...) // based on compute, storage, network, and external calls
    - recordHistogram("request_cost_usd", cost, labels={"service": "orders", "endpoint": "/checkout", "region": "us-west-1"})

- Example 3: P95 and P99 PromQL-like queries (Prometheus-compatible)
  - P95 latency across all services:
    - histogram_quantile(0.95, sum(rate(request_latency_ms_bucket[5m])) by (le))
  - P99 latency for a specific endpoint:
    - histogram_quantile(0.99, sum(rate(request_latency_ms_bucket{service="orders", endpoint="/checkout"}[5m])) by (le))

- Example 4: Basic anomaly rule (conceptual)
  - If avg(request_cost_usd) > baseline + threshold for a 1-hour window, trigger a cost anomaly alert
  - If histogram_quantile(0.95, request_latency_ms_bucket) > baseline_p95 + delta for 10 minutes, trigger a latency anomaly alert

- Example 5: OpenTelemetry integration snippet (conceptual)
  - Instrument request paths with trace IDs and attach latency and cost metrics as attributes
  - Ensure OTLP exporters push metrics and traces to the collector
  - Collector forwards to Prometheus (metrics) and Tempo (traces)

Note: The exact code syntax will depend on your language and framework. The key patterns are consistent: collect latency in a histogram, collect per-request cost, attach meaningful labels, and export to a standard collector.

---

## 5. Quick Start Checklist

- [ ] Decide on the data strategy: Prometheus for metrics, Tempo for traces, Loki for logs (or equivalent).
- [ ] Instrument a minimal set of core services with OpenTelemetry: latency histograms and per-request cost metrics.
- [ ] Set up the OpenTelemetry Collector to receive OTLP data and export to your chosen backends.
- [ ] Build a basic Grafana dashboard with panels for:
  - Latency percentiles (P95, P99)
  - Request rate (throughput)
  - Cost per request and total cost
  - Per-service cost and per-endpoint latency hotspots
- [ ] Implement alerting rules for cost spikes and latency anomalies. Define escalation paths.
- [ ] Validate with load testing to ensure percentile calculations, cost attribution, and alert triggers behave as expected.
- [ ] Plan for growth: templated dashboards, multi-region support, additional endpoints, and more granular cost attribution.

---

## 6. Additional Reading and Best Practices (Conceptual)

- Latency measurement best practices emphasize histograms and proper bucket sizing to ensure percentile accuracy and meaningful insights into tail latency.
- Cost visibility should be anchored in a robust tagging strategy that enables per-service and per-endpoint cost attribution, often aligned with cloud cost management tools.
- OpenTelemetry is a flexible, vendor-agnostic approach to observability that enables seamless correlation between traces, metrics, and logs, which is essential for diagnosing complex interactions in modern microservices architectures.
- Anomaly detection for cost and performance can be achieved with ML-based detectors provided by cloud platforms or specialized tools, enabling proactive alerts beyond static thresholds.
- A real-time dashboard is most effective when data granularity aligns with user needs. Start with a compact, fast dashboard and gradually add depth as you validate insights and user feedback.

---

## 7. Bonus: Real-World Scenarios to Practice

- Scenario 1: A user-facing API shows rising P95 latency in region us-east-2. The per-request cost also spikes there. Practice tracing the request path to identify whether a downstream dependency or a particular endpoint drives both latency and cost. Use a per-service cost panel to confirm if a specific service is the main contributor.
- Scenario 2: A sudden cost spike without a corresponding latency change. Investigate whether background tasks, data processing, or external API usage increased. Check cost anomaly alerts and look for abnormal resource usage patterns across services.
- Scenario 3: A steady latency but cost per query increases over time. Investigate whether resource efficiency has declined (e.g., memory usage increasing, inefficient caching), or whether new features are consuming more compute per request.

---

## 8. Final Thoughts

Building cost and latency dashboards is not just about collecting data; it’s about turning data into insight that guides action. Strive for dashboards that are:
- Understandable at a glance (clear visuals and labeled axes)
- Actionable (alerts that point to concrete mitigation steps)
- Scalable (you can add services, regions, or pricing models without a complete redesign)
- Correlated (latency, cost, and traces share a common context so you can answer “why” quickly)

With a well-structured approach—instrumentation, data collection, visualization, and alerting—you’ll have a powerful platform to monitor, optimize, and communicate the cost and performance health of your services in real time.

---