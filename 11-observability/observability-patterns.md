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
    * **Step 1: Check the "Symptom" (User Impact)**
        * Don't start with CPU.
        * Check "Error Rate" and "User-facing Latency" (e.g., Apdex score).
        * *Observation*: "The Error Rate just spiked to 5% and Apdex dropped." (This confirms a real incident).
    * **Step 2: Check the "Vitals" (System Health)**
        * Look at "Infrastructure Metrics" (CPU, Memory, Disk I/O).
        * *Observation*: "CPU is at 40%, Memory is stable." (This rules out a noisy neighbor or OOM killing).
    * **Step 3: Check the "Network"**
        * Look at "Network Ingress/Egress" and "Connection Saturation."
        * *Observation*: "The API Gateway's connection count is maxed out."
    * **Step 4: Check the "Dependencies"**
        * Look at "External Service Latency" and "Database Load."
        * *Observation*: "The Database P95 latency doubled 5 minutes ago."
    * **Step 5: Check the "Logs" & "Traces"**
        * Correlate the time from Step 4 with Application Logs or Distributed Traces.
        * *Conclusion*: "The slow database queries started at the same time as the error spike. It's a DB issue."
*   **Metrics Design**: "What metrics would you track for a 'Background Job Worker' system?" (Hint: Queue depth, Job duration, Retry count).
    * **Queue-Level Metrics**:
        * `queue.depth`: Number of jobs waiting to be processed.
        * `queue.oldest_job_age`: How long the oldest job has been waiting.
        * `queue.delayed_jobs`: Number of jobs scheduled for future execution.
    * **Worker-Level Metrics**:
        * `worker.active_count`: Number of workers currently processing jobs.
        * `worker.idle_count`: Number of idle workers available.
        * `worker.cpu_usage`: CPU utilization of the worker process.
    * **Job-Level Metrics**:
        * `job.duration`: Time taken to process a single job (p50, p95, p99).
        * `job.success_rate`: Percentage of successfully processed jobs.
        * `job.failure_rate`: Percentage of failed jobs.
        * `job.retry_count`: Number of times a job has been retried.
    * **System-Level Metrics**:
        * `system.memory_usage`: Memory consumed by the worker process.
        * `system.disk_space`: Available disk space (important for temp files).
        * `system.network_errors`: Network-related errors during job processing.
*   **The Difference**: "Explain the difference between 'Monitoring' and 'Observability'."
    * **Monitoring**: "Do I know when something is wrong?"
        * **What it answers**: It tells you if the system is healthy or unhealthy based on pre-defined metrics and thresholds.
        * **How it works**: It relies on *known* failure modes. You proactively define what to watch (e.g., CPU > 90%, Error Rate > 5%).
        * **Analogy**: Checking the "Check Engine" light on your car. The car tells you *when* it's broken, but not *why*.
    * **Observability**: "Can I figure out what's wrong?"
        * **What it answers**: It allows you to ask *any* question about the system's internal state, even for failures you haven't seen before.
        * **How it works**: It relies on *unknown* failure modes. It collects rich, multi-dimensional data (logs, metrics, traces) that you can slice and dice to debug novel issues.
        * **Analogy**: A mechanic with a full diagnostic toolkit. They can hook up an OBD-II scanner, check the O2 sensor, read engine codes, and analyze fuel pressure to diagnose *any* engine problem.

### 7. Common Mistakes
*   **Alert Fatigue**: Setting too many noisy alerts that engineers start to ignore, causing them to miss a "real" critical outage.
*   **Unstructured Logs**: Logging raw text like `Error in user logic!` instead of structured JSON `{"evt": "error", "uid": 123, "msg": "invalid_login"}` that can be indexed and searched.
*   **Monitoring "Vitals" but not "Symptoms"**: Tracking CPU (Vital) but not Tracking "User Login Failure Rate" (Symptom). The user doesn't care if CPU is 10%, they care if they can't login.
