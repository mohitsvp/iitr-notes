# Building Cost & Latency Dashboards — Beginner Assignment Notes

Assignment: 8 MCQ/MSQ questions (Easy) + 1 Subjective question (Hard-style)

Note: The questions are designed for beginners. They focus on understanding concepts and simple calculations you can implement in a notebook or lightweight dashboard.

Easy: 8 MCQ/MSQ questions

1) What does P95 latency mean?
- A) The maximum latency observed
- B) The average latency across all requests
- C) The value below which 95% of observed latency values fall
- D) The latency of the 95th request
Answer: C
Explanation: P95 latency is the 95th percentile of latency values—95% of requests have latency at or below this value.

2) How do you compute cost per query from logs?
- A) total_cost / total_queries
- B) total_queries / total_cost
- C) average_latency / total_cost
- D) total_cost * total_queries
Answer: A
Explanation: Cost per query is total_cost divided by total_queries. This gives the average cost attributed to each request.

3) Which of the following is a good practice for a cost anomaly alert baseline?
- A) A fixed threshold that never changes
- B) A dynamic baseline using moving averages or percentiles
- C) Alert only when at least one request fails
- D) Alert only on daily totals
Answer: B
Explanation: Dynamic baselines (e.g., moving averages, percentiles) adapt to normal fluctuations and reduce alert fatigue.

4) Which data sources are most suitable for a cost & latency dashboard?
- A) Logs for latency and costs, and metrics for timestamps
- B) Randomly generated numbers
- C) Static configuration files with no runtime data
- D) Only user surveys
Answer: A
Explanation: Logs capture latency and cost per request; metrics provide reliable, time-series signals for dashboards.

5) In a serverless environment, which approach best attributes per-request cost?
- A) Attribute a fixed cost to all requests
- B) Attribute cost per request using request-level details (e.g., function invocations, duration)
- C) Attribute cost only at the end of the month
- D) Attribute cost based on the number of users
Answer: B
Explanation: Per-request cost attribution should consider invocation cost and duration per request to reflect true cost impact.

6) What does P99 latency measure?
- A) The 99th percentile latency
- B) The 99th request’s latency
- C) The median latency
- D) The peak latency observed
Answer: A
Explanation: P99 latency is the latency value below which 99% of requests fall.

7) Which visualization best communicates latency distribution across endpoints?
- A) A single line chart of average latency
- B) A box-and-whisker plot (or violin plot) per endpoint showing multiple percentiles
- C) A pie chart of endpoint names
- D) A stacked bar chart of status codes
Answer: B
Explanation: Box plots or violin plots show distribution and variability, making it easy to compare latency spread across endpoints.

8) For real-time dashboards, what is a practical data freshness target?
- A) Real-time with sub-second delay
- B) A delay of a few seconds to a few minutes, depending on SLA
- C) Daily refresh
- D) Weekly refresh
Answer: B
Explanation: Real-time dashboards usually balance freshness with system load, often aiming for seconds-to-minutes delay depending on SLAs.

Hard: 1 Subjective question

Subjective (design-and-explain style)

Question:
You are given a microservices-based application with three services: auth, search, and checkout. You need to design a cost & latency dashboard for this app that a product manager can use to identify performance and cost issues quickly. Provide:
- a) A concise data schema (fields and data types) you would instrument/collect
- b) The key metrics you would display (with rationale)
- c) A simple data-processing plan to compute:
  - i) cost_per_query per service
  - ii) P95 and P99 latency per service and per endpoint
  - iii) a basic ROI_per_query proxy (explain any assumptions)
- d) Three alerting rules you would implement (describe thresholds and what they trigger)
- e) A rough sketch of the dashboard layout (sections and widgets) and why
- f) Any trade-offs or potential pitfalls to watch for, and how you would mitigate them

Answer guidance:
- Provide clear, actionable details suitable for a beginner to implement in a notebook or a lightweight dashboard tool.
- Include lightweight example code snippets or pseudo-SQL where helpful.
- Emphasize decisions and rationale so a learner can defend their design.

Answer Key and Solutions

MCQ/MSQ questions — Solutions and explanations

1) Answer: C
- P95 latency is the value below which 95% of observed latency values fall. It is a percentile measure, not a single maximum or average.

2) Answer: A
- Cost per query = total_cost / total_queries. This yields the average resource cost attributed to each request.

3) Answer: B
- A dynamic baseline (moving average, percentiles, or similar) adapts to traffic changes and avoids frequent false positives.

4) Answer: A
- Logs plus metrics provide the necessary per-request details (latency, cost, endpoints) for a robust dashboard.

5) Answer: B
- Per-request cost attribution should be based on per-invocation details (duration, resources used) to reflect actual cost.

6) Answer: A
- P99 latency is the 99th percentile; 99% of requests observe latency at or below this value.

7) Answer: B
- Box/violin plots per endpoint show distribution and variability, enabling cross-endpoint latency comparisons beyond averages.

8) Answer: B
- Real-time dashboards commonly target a modest freshness window (seconds to a few minutes) aligned with the system and business SLAs.

Subjective question — Ideal approach outline

- a) Data schema
  - request_id: string
  - service: string
  - endpoint: string
  - timestamp: timestamp
  - latency_ms: integer
  - cost_usd: float
  - status: string
  - user_id (optional): string
  - resource_units (optional): float
- b) Key metrics
  - Latency distributions (P50, P90, P95, P99) by service/endpoint
  - Average cost per query by service/endpoint
  - Queries per second (throughput) per service
  - ROI_per_query proxy (see c)
  - Error rate by service/endpoint
  Rationale: captures performance, cost, reliability, and business value signals
- c) Data-processing plan
  - i) cost_per_query per service
    - Aggregate: total_cost_per_service / total_queries_per_service
  - ii) P95 and P99 latency
    - Compute percentiles over a sliding window (e.g., last 5–15 minutes) per service/endpoint
    - Use a streaming or batch pipeline; simple notebook approach: group_by service/endpoint and apply percentile_approx for latency_ms
  - iii) ROI_per_query proxy
    - Define Value_per_query proxy (e.g., revenue_per_query or cost_savings_per_query)
    - ROI_per_query = (Value_per_query - cost_per_query) / cost_per_query
    - If Value_per_query is not known, discuss scenario analysis: compute only cost_per_query trends and annotate with qualitative ROI bands (e.g., favorable, neutral, unfavorable)
- d) Alerting rules
  - Rule 1: Cost spike alert
    - Trigger if current cost_per_query or total_cost_over_window > baseline_cost_per_query * 1.5 for any service
  - Rule 2: Latency tail alert
    - Trigger if P99_latency for a service/endpoint > threshold (e.g., 95th percentile latency exceeds 2x baseline)
  - Rule 3: Error rate alert
    - Trigger if error_rate > 1% for any service in a rolling window
- e) Dashboard layout sketch
  - Top bar: time window selector, environment (prod/staging), and last refresh time
  - Section 1: Latency overview
    - Multi-line or grouped box plots showing P95/P99 by service/endpoint
  - Section 2: Cost and ROI
    - Bar chart: cost_per_query by service
    - ROI per_query (proxy) as colored sparkline or gauge
  - Section 3: Throughput and reliability
    - Line chart: queries per second by service
    - Error rate by service (area chart or stacked bar)
  - Section 4: Anomalies and alerts
    - List of active alerts with severity and suggested actions
- f) Trade-offs and pitfalls
  - Data granularity vs. latency: finer granularity yields better fidelity but higher ingestion/processing load; mitigate with rolling windows and sampling
  - Attribution accuracy: per-request cost in serverless can be noisy; mitigate by amortized costing and consistent attribution rules
  - False positives in alerts: tune baselines and incorporate drift handling; use two-stage alerts (warning then critical)
  - Data privacy and security: ensure logs don’t expose sensitive PII; redaction policies
  - Extensibility: start simple and add services/endpoints iteratively to avoid complexity

What learners will likely implement (starter steps)
- Generate synthetic data or use a small demo dataset
- Compute cost_per_query and basic latency percentiles using Python/pandas or SQLite
- Sketch a simple dashboard layout (even in a notebook with matplotlib or seaborn)
- Write one or two short scripts showing how to calculate and print ROI_per_query proxy values and thresholds for alerting

Notes for instructors or mentors
- Encourage learners to connect the dots between operational metrics (latency, error rates) and business metrics (cost, ROI).
- Emphasize practical constraints: data freshness, ease of setup, and maintainability.
- Provide optional extension tasks for learners who want deeper practice (e.g., implement a minimal Grafana dashboard with a Prometheus-like data source, or a notebook that generates a PDF report with plots).

References and further reading (recommended)
- Basics of latency percentiles and tail latency (P95, P99) in distributed systems
- OpenTelemetry data collection, Prometheus-style time-series metrics, and Grafana dashboards
- Simple approaches to cost attribution in serverless environments and common pitfalls
- Lightweight alerting concepts (baseline-based thresholds, moving averages, and drift-aware checks)