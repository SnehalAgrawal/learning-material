# HLD Fundamentals: Scalability, CAP, & Fault Tolerance

### 1. Overview
High-Level Design (HLD) fundamentals are the "Laws of Physics" for distributed systems. For a senior engineer, these aren't just definitions but the primary constraints used to evaluate architectural proposals. They dictate how a system survives growth (Scalability), handles hardware failure (Fault Tolerance), and makes data trade-offs (CAP).

### 2. Key Concepts
*   **Scalability**: The ability of a system to handle increased load.
    *   *Vertical*: Adding more CPU/RAM to a single node.
    *   *Horizontal*: Adding more nodes to the pool.
*   **Latency vs. Throughput**:
    *   *Latency*: Time taken for a single request (ms).
    *   *Throughput*: Number of requests handled per unit of time (RPS).
*   **CAP Theorem**: In a distributed system, you can only pick two:
    *   *Consistency*: All nodes see the same data at the same time.
    *   *Availability*: Every request gets a response (success or failure).
    *   *Partition Tolerance*: System works despite network failures (MANDATORY in modern systems).
    
    CAP Theorem = During network failures, choose between correct data (C) or always responding (A).

    CAP theorem states that in a distributed system, when a network partition occurs, you must choose between consistency and availability. Since partitions are unavoidable, systems are typically designed as either CP (like databases) or AP (like social platforms), depending on business needs.

    When a network partition happens, you must choose:

    * **Option 1: CP (Consistency + Partition Tolerance)**
        * You ensure correct data
        * But might reject requests (downtime)

        Example: **Banking systems**: If balance can't be verified → transaction fails
            
    * **Option 2: AP (Availability + Partition Tolerance)**
        * Always respond
        * But data might be temporarily inconsistent

        Example: **Social media likes/comments**: Seeing slightly outdated data is acceptable
            
    * **Why Not CA?**
        * CA (Consistency + Availability) assumes no network failures
        * In reality → networks fail all the time

    So CA is not practical in distributed systems

*   **Fault Tolerance**: System continues to work even when parts fail. The "Availability" goal. Achieving it via Redundancy (No single point of failure) and Graceful Degradation. It include
    * **Replication**: Keeping multiple copies of data across different nodes.
    * **Failover**: Detect failure and redirect traffic to a healthy node.
    * **Redundancy**: Duplicate everything critical like load balancers, databases, application servers, and even entire data centers.
    * **Timeouts + retries**: If a service is slow → give up after a timeout and try again (or fail fast).
    * **Circuit breaker**: If a service is failing repeatedly → stop calling it for a while to let it recover.
    * **Idempotency**: Make sure the same operation can be repeated without causing issues (e.g., charging a customer twice).
    * **Data Partitioning + Isolation**: Split system into independent parts.
    * **Consensus Algorithms**: Used to maintain consistency across nodes. Leader election, Agreement on state. Examples: Raft, Paxos
    * **Monitoring + Health Checks**: Used to detect failures early and trigger recovery.Heartbeats, Metrics, Alerts.
    * **Graceful Degradation**: System still works with reduced functionality, Show cached data if DB is down, disable recommendations but keep checkout working.

    Tradeoffs:
    * **Strong fault tolerance**: often reduces co  nsistency or increases latency.
    * **More retries**: more load.
    * **More replication**: higher cost.


### 3. Real-World Usage
*   **Retail Peak (Black Friday)**: Using Horizontal Scaling + Auto-scaling groups to go from 10 servers to 1000 during a flash sale.
*   **Global Social Media**: Picking **AP** (Availability + Partition Tolerance) for Facebook/Instagram comments—seeing a comment 2 seconds late is better than the "Comment" button not working.
*   **Banking Systems**: Picking **CP** (Consistency + Partition Tolerance) for balance transfers. The system must fail the request rather than showing an incorrect balance.
*   **Video Streaming**: Optimizing for Throughput over Latency during buffering, but switching to low latency for live streams (WebRTC).

### 4. Tradeoffs
*   **Cost vs. Scalability**: Horizontal scaling requires complex service discovery and load balancing. Vertical scaling is cheaper and easier until you hit the hardware ceiling.
*   **Consistency vs. Performance**: Strong consistency (CP) requires network round-trips for consensus (e.g., Paxos/Raft), which increases latency. Eventual availability (AP) is faster but leads to "dirty reads."
*   **Complexity vs. Availability**: Adding more redundant layers (Active-Active regions) increases availability but makes deployment and state synchronization much harder.

### 5. When NOT to Use
*   **Microservices Everywhere**: Scaling horizontally across 50 microservices for a CRUD app with 10 users is an anti-pattern (Over-engineering).
*   **Strong Consistency**: Don't enforce it for non-critical data. If you can tolerate "Eventual Consistency" for a profile picture update, take the performance win.

### 6. Interview Focus
*   **Estimation**: "If our app has 10M DAU and each user uploads 2 photos/day, how much storage and bandwidth do we need?"
    * 10M users × 2 photos = 20M photos/day.
    * Assuming ~2MB per photo → ~40TB/day storage.
    * For bandwidth, uploads alone are ~40TB/day, and reads are typically 5–10× higher → ~200–400TB/day total bandwidth.
    * We’d use compression, CDN, and object storage to optimize.
*   **Scenario Choice**: "Why would you choose NoSQL (usually AP) over SQL (usually CP) for a global real-time chat app?"
    * In chat systems, availability and low latency matter more than strict consistency.
    * NoSQL (AP) allows messages to be delivered even during network partitions, with eventual consistency.
    * Users tolerate slight delays or ordering issues, but not message failures, so AP is preferred.
*   **Bottlenecks**: "The load balancer is healthy, but the system is slow. Where do you look first? (DB locks, CPU, or Network IO?)"
    * First, I’d check the database — especially for locks, slow queries, or connection pool exhaustion, since DB is the most common bottleneck.
    * Then CPU (high usage → inefficient code), and finally network I/O (latency, packet loss).
    * I’d validate using metrics, logs, and tracing before concluding.

### 7. Common Mistakes
*   **Ignoring Network Partitions**: Thinking "our network is stable," so CAP doesn't apply. Partition Tolerance is not optional.
*   **Optimizing the Wrong Metric**: Reducing Latency from 50ms to 20ms when the real problem is that the system crashes (Throughput) at 1000 users.
*   **Single Point of Failure (SPOF)**: Having a perfectly scalable API layer but a single, non-redundant primary database that can't be scaled or failed over.
