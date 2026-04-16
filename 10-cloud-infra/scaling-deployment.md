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
*   **Failover Logic**: "Design a deployment pipeline for a business-critical API. How do you handle 'Rollbacks' automatically?"
*   **Auto-scaling Metrics**: "Besides CPU and RAM, what are other 'Business Metrics' you could use to trigger auto-scaling?" (Hint: Request Queue Depth, Messages in SQS).

### 7. Common Mistakes
*   **Missing Health Checks**: Doing a deployment but not waiting for the "Readiness Probe" to pass before killing the old version, causing a total outage (The "Deployment Cliff").
*   **Ignoring Database Compatibility**: Deploying Code V2 that expects a DB column that hasn't been added yet. Deployments must be "Backward Compatible."
*   **Over-scaling**: Setting auto-scaling limits too high without a budget cap, leading to a $50,000 AWS bill on a single Friday night.
