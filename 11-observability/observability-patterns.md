# Observability: Logs, Metrics, & Tracing

### 1. Overview
Observability is the measure of how well you can understand the internal state of a system by looking only at its external outputs. For senior engineers, this is the "Black Box" debugging toolset. It goes beyond simple monitoring to provide the context needed to find the "needle in the haystack" during a production outage.

### 2. Key Concepts
*   **The Three Pillars**:
    *   *Logs*: Discrete events (e.g., "User clicked button," "Error in DB"). Great for specific context but slow to query and expensive to store.
    *   *Metrics*: Aggregated numerical data (e.g., "CPU usage is 80%," "Error rate is 2%"). Fast, cheap, and perfect for alerting.
    *   *Tracing*: Following a single request as it jumps between 10 microservices. Essential for finding latency bottlenecks in distributed systems.
*   **SLIs, SLOs, & SLAs**:
    *   *SLI*: Service Level Indicator (The metric you measure).
    *   *SLO*: Service Level Objective (The target you want to hit).
    *   *SLA*: Service Level Agreement (The legal contract with users).

### 3. Real-World Usage
*   **Alerting**: Setting a **Metric-based alert** on "5XX error rate" to ping an engineer on PagerDuty before users start calling support.
*   **Latency Debugging**: Using a **Distributed Trace** (e.g., Datadog, Jaeger) to find that a single request is slow because `Service B` is waiting for a slow `Service C` database query.
*   **Audit Trails**: Using **Structured Logging** (JSON logs) to search for all actions taken by a specific `user_id` when investigating a security incident.
*   **Capacity Planning**: Looking at "Memory Usage Metrics" over the last 30 days to decide when to upgrade the database instance size.

### 4. Tradeoffs
*   **Logging Level vs. Cost**: Logging `DEBUG` level in production provides great detail but can cost more than the app's hosting bill and slow down the app due to Disk I/O.
*   **Tracing Overhead**: Adding tracing spans to every function call can add 5-10% latency to a high-performance app. It needs to be "Sampled" (e.g., trace only 1% of requests).
*   **Build vs. Buy**: Building your own ELK stack (Elasticsearch, Logstash, Kibana) gives you control but requires a dedicated infra team. Buying SaaS (New Relic/Datadog) is expensive but "just works."

### 5. When NOT to Use
*   **Detailed Tracing for Simple Apps**: A single-process monolith doesn't need distributed tracing. Simple application logs and profiling are enough.
*   **Polling for Metrics**: For high-volume systems, metrics should be "Pushed" (UDP/StatsD) rather than "Polled" by the monitoring server to avoid creating a performance bottleneck.

### 6. Interview Focus
*   **Production Incident Simulation**: "The website is slow. You have a dashboard with CPU and Error Rate. What is your process to find the root cause?"
*   **Metrics Design**: "What metrics would you track for a 'Background Job Worker' system?" (Hint: Queue depth, Job duration, Retry count).
*   **The Difference**: "Explain the difference between 'Monitoring' and 'Observability'."

### 7. Common Mistakes
*   **Alert Fatigue**: Setting too many noisy alerts that engineers start to ignore, causing them to miss a "real" critical outage.
*   **Unstructured Logs**: Logging raw text like `Error in user logic!` instead of structured JSON `{"evt": "error", "uid": 123, "msg": "invalid_login"}` that can be indexed and searched.
*   **Monitoring "Vitals" but not "Symptoms"**: Tracking CPU (Vital) but not Tracking "User Login Failure Rate" (Symptom). The user doesn't care if CPU is 10%, they care if they can't login.
