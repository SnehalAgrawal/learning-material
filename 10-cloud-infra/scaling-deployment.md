# Deployment Strategies & Infrastructure Scaling

### 1. Overview
Modern software delivery is about "Zero Downtime." For senior engineers, this involves designing deployment pipelines that can ship code safely without affecting users, and using automated infrastructure scaling to handle traffic fluctuations while maintaining cost efficiency.

### 2. Key Concepts
*   **Blue-Green Deployment**: Maintaining two identical production environments. One is live (Blue); the new version is deployed to Green. Traffic is switched instantly at the Load Balancer level.
*   **Canary Deployment**: Releasing a new version to a small percentage (e.g., 5%) of users first. If metrics look good, it's rolled out to everyone.
*   **Auto-Scaling**:
    *   *Horizontal Pod Autoscaler (HPA)*: Adding more pods based on CPU/RAM usage.
    *   *Cluster Autoscaler*: Adding more physical nodes/VMs when there is no room for more pods.
*   **CI/CD**:
    *   *Continuous Integration*: Automating the build and test process.
    *   *Continuous Deployment*: Automatically shipping every "passing" change to production.

### 3. Real-World Usage
*   **Risk Mitigation**: Using **Canary Deployments** for a major database migration to ensure that if something breaks, only 1% of users are affected.
*   **Fast Rollback**: Using **Blue-Green** to roll back a "Breaking UI" change in seconds just by flipping a traffic switch.
*   **Demand Spikes**: An e-commerce site using **Auto-scaling** to go from 5 servers at midnight to 100 servers during a "Flash Sale" at 9 AM.
*   **Quality Gates**: A CI/CD pipeline that requires 90% test coverage and a successful "Security Scan" before code can be merged to `main`.

### 4. Tradeoffs
*   **Blue-Green vs. Canary**: Blue-Green is faster to switch but twice as expensive (requires double the infrastructure). Canary is cheaper but takes longer to "Prove" stability.
*   **Auto-scaling Lag**: It takes time (minutes) to boot new VMs or pods. If your traffic spikes in seconds (e.g., a Super Bowl ad), auto-scaling might be too slow. You need "Pre-warming."
*   **CD vs. Manual Approval**: Full Continuous Deployment is the goal, but for highly regulated industries (Banking/Medical), manual "Human-in-the-loop" approval is often a compliance requirement.

### 5. When NOT to Use
*   **Blue-Green for Databases**: Blue-Green is great for stateless apps, but managing two versions of a 10TB database in sync during a deployment is a recipe for data corruption. Use "Expand/Contract" migrations instead.
*   **Auto-scaling for Tiny Apps**: If you only have 2 servers, the overhead of setting up and monitoring auto-scaling rules is likely higher than the cost savings.

### 6. Interview Focus
*   **Rolling Updates**: "Explain how Kubernetes performs a 'Rolling Update.' What happens to active user connections?"
    * **The Concept**: "Rolling Update" is the default strategy for updating a Deployment in Kubernetes. Instead of taking down all old pods at once (which would cause downtime), it gradually replaces them.
    * **The Process**:
        1.  **Configuration**: You define the `maxUnavailable` (e.g., 1) and `maxSurge` (e.g., 1) in your Deployment manifest.
        2.  **Start New**: Kubernetes starts one (or more) *new* pods with the new image version.
        3.  **Wait for Readiness**: It waits for the new pod(s) to pass their `Readiness Probes` (health checks). This ensures the new version is actually ready to accept traffic.
        4.  **Terminate Old**: Once the new pod is ready, it terminates one of the *old* pods.
        5.  **Repeat**: It repeats this cycle (New -> Ready -> Terminate Old) until all old pods are gone and only new pods remain.
    * **Handling Active Connections**:
        * **Graceful Shutdown**: When a pod is marked for termination, Kubernetes sends a `SIGTERM` signal to the application container.
        * **The Grace Period**: The application has a `terminationGracePeriodSeconds` (default 30 seconds) to finish processing any in-flight requests and shut down cleanly.
        * **The Switch**: The Service load balancer stops sending traffic to the old pod as soon as it starts shutting down.
        * **The Result**: If your application is well-behaved (handles `SIGTERM` correctly), active connections are drained gracefully, and users don't experience errors. If the app ignores `SIGTERM`, those specific connections might be dropped.
*   **Failover Logic**: "Design a deployment pipeline for a business-critical API. How do you handle 'Rollbacks' automatically?"
    * **The "Undo" Button**: A rollback is triggered when a new deployment fails or causes errors. Kubernetes stores a history of previous revisions for each Deployment.
    * **Automatic Rollback (The Safety Net)**: In Kubernetes, you can configure `Recreate` or `Rolling` strategies. For a business-critical API, you should use a `RollingUpdate` with strict `Readiness Probes`.
    * **The Scenario**: Let's say you have 5 pods. You update the image. 
        1.  New Pod 1 starts. If it fails its Health Check, Kubernetes **stops** the rollout immediately. It does not touch the other 4 old pods.
        2.  **Automatic Rollback**: Because the new version is unhealthy, the Deployment Controller automatically reverts the change. It keeps the old Pods and kills the new one(s) that failed to start.
    * **Manual Rollback (The "I want to go back to V1")**:
        * If the new version *seems* okay initially but starts failing later (e.g., a slow memory leak), the auto-rollback might not trigger.
        * You manually trigger a rollback using: `kubectl rollout undo deployment/my-api`.
        * **What happens?**: Kubernetes looks at the revision history and re-applies the YAML configuration of the previous version (e.g., Revision 3). It then performs another "Rolling Update" to replace the bad version (V4) with the old good version (V3).
*   **Auto-scaling Metrics**: "Besides CPU and RAM, what are other 'Business Metrics' you could use to trigger auto-scaling?" (Hint: Request Queue Depth, Messages in SQS).
    * **The Standard (Why CPU/RAM aren't enough)**:
        * **The Problem**: Your CPU might be at 100%, but if your application is "I/O Bound" (waiting for the database), adding more CPU won't help. Conversely, you could have low CPU usage but be getting hammered with requests (High Latency).
    * **Business Metrics for Scaling**:
        1.  **Queue Depth (The "Backlog" Metric)**:
            * **For Background Workers**: If you use a Queue (like RabbitMQ, Kafka, or SQS), the number of messages waiting in the queue is the *perfect* indicator of load.
            * **Logic**: If `QueueDepth > 1000`, scale up. If `QueueDepth < 100`, scale down.
        2.  **Request Latency (The "User Experience" Metric)**:
            * **The Logic**: If the 95th percentile latency (p95) goes above your SLA (e.g., > 200ms), the system is slow. Trigger scaling immediately.
        3.  **Active Connections / Concurrent Users**:
            * **The Logic**: Some platforms (like connection-pooled apps) have a hard limit on concurrent connections. If `Connections > 80% of Max`, scale up.
        4.  **Custom Business Metrics (The "Sales" Metric)**:
            * **The Logic**: E-commerce sites often scale based on "Items Added to Cart" or "Completed Orders" (e.g., "Scale up if we are processing > 100 orders per minute"). This is the most accurate but hardest to implement.
    * **Implementation (KEDA)**:
        * For Kubernetes, the standard way to do this is using **KEDA** (Kubernetes Event-Driven Autoscaling).
        * KEDA can connect directly to SQS, Kafka, RabbitMQ, etc., and scale your pods from 0 to N based on the queue depth, without needing CPU metrics.

### 7. Common Mistakes
*   **Missing Health Checks**: Doing a deployment but not waiting for the "Readiness Probe" to pass before killing the old version, causing a total outage (The "Deployment Cliff").
*   **Ignoring Database Compatibility**: Deploying Code V2 that expects a DB column that hasn't been added yet. Deployments must be "Backward Compatible."
*   **Over-scaling**: Setting auto-scaling limits too high without a budget cap, leading to a $50,000 AWS bill on a single Friday night.
