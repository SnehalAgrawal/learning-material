# Cost Engineering in the Cloud

### 1. Overview
Cost is a first-class engineering metric for senior architects. Cost Engineering involves designing systems that are not just technically efficient but economically sustainable. In the cloud (AWS/Azure/GCP), this means understanding the pricing models for different compute and storage tiers.

### 2. Key Concepts
*   **On-Demand vs. Reserved Instances (RI)**:
    *   *On-Demand*: Pay by the second. Use for bursty/unpredictable load.
    *   *RI / Savings Plans*: Commit to 1-3 years for up to 70% discount. Use for steady-state baseline load.
*   **Serverless Pricing**: Pay only for execution time. No idle cost. Best for low/bursty traffic.
*   **Spot Instances**: Excess cloud capacity at a massive discount (90%) but can be terminated with a 2-minute notice.
*   **Egress Costs**: The "Hidden Tax." Cloud providers charge heavily for data leaving their network to the public internet or between regions.

### 3. Real-World Usage
*   **Stateless Workloads**: Running a fleet of "Search Indexers" on **Spot Instances** with a fallback to On-Demand if capacity disappears.
*   **Database Baselining**: Using **Reserved Instances** for your primary database since you know it will be running 24/7.
*   **Development Environments**: Using **Serverless** or auto-shutdown scripts for dev/staging environments to avoid paying for them over the weekend.
*   **Multiregion Strategy**: Placing "Read Replicas" in the same region as the App servers to minimize inter-region data transfer costs.

### 4. Tradeoffs
*   **Serverless vs. Containers**: Serverless scales to zero cost but becomes more expensive than Kubernetes (EKS/GKE) once you hit a certain constant RPS threshold.
*   **Performance vs. Tiering**: Using Amazon S3 "Intelligent Tiering"—moving infrequently accessed data to "Cold" storage saves money but adds retrieval latency.
*   **Operational Burden vs. Discount**: Spot instances are cheap but require complex "interrupt-handling" logic in your code/orchestrator.

### 5. When NOT to Use
*   **Reserved Instances**: Don't buy them for a new product where you don't know the baseline traffic. Wait 3 months to see the usage pattern.
*   **Serverless**: Avoid for long-running processes (e.g., video transcoding that takes 30 mins) as costs will skyrocket and you'll hit timeout limits.

### 6. Interview Focus
*   **The "Build vs. Buy" Cost**: "Should we build our own Auth system or use Auth0? How do you calculate the TCO (Total Cost of Ownership)?"
    * Build: Higher initial dev cost, full control, no vendor lock-in, but ongoing maintenance burden.
    * Buy: Lower upfront cost, faster to market, provider handles updates/security, but recurring subscription fees and less flexibility.
    * TCO Calculation:
        * Build: (dev salaries × time) + infra + maintenance + opportunity cost.
        * Buy: (subscription fees × time) + migration cost + potential customization costs.
    * I'd choose Buy for non-core features (like auth) to speed up time-to-market, but Build for core IP that differentiates us.
*   **Scaling Economics**: "At what point in our growth does it make sense to move from AWS Lambda to a Kubernetes cluster?"
    * Depends on cost vs. control.
    * Lambda is great for spiky traffic, but can get expensive at high, steady volume.
    * Kubernetes gives more control and can be cheaper at scale (better resource utilization), but requires operational overhead (DevOps team, cluster management).
    * Breakeven point varies, but typically: Lambda for <100 RPM baseline (with spiky peaks); Kubernetes for >500 RPM consistent load.
*   **Hidden Costs**: "How does choosing a multi-region Active-Active architecture impact our monthly AWS bill?"
    * Multi-region = 2x+ costs for compute, databases, and data transfer. Active-Active adds complexity (data sync, latency) and higher egress costs.
    * Need to factor in:
        * Duplicate infrastructure in each region
        * Data replication costs (cross-region replication)
        * Load balancing across regions
        * Higher bandwidth/egress charges
    * Only justify when required for disaster recovery or low-latency global user access.

### 7. Common Mistakes
*   **Ignoring Idle Resources**: Leaving a huge DB instance running for a staging environment that nobody uses.
*   **The S3 "Trap"**: Keeping terabytes of data in "Standard" storage that has never been accessed in 2 years. Use Lifecycle Policies.
*   **Provider Lock-in Pricing**: Relying on proprietary cloud services that are cheap today but have no alternatives if the provider raises prices later.
