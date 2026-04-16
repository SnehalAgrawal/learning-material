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
*   **Fault Tolerance**: The "Availability" goal. Achieving it via Redundancy (No single point of failure) and Graceful Degradation.

### 3. Real-World Usage
*   **Retail Peak (Black Friday)**: Using Horizontal Scaling + Auto-scaling groups to go from 10 servers to 1000 during a flash sale.
*   **Global Social Media**: Picking **AP** (Availability + Partition Tolerance) for Facebook/Instagram comments—seeing a comment 2 seconds late is better than the "Comment" button not working.
*   **Banking Systems**: Picking **CP** (Consistency + Partition Tolerance) for balance transfers. The system must fail the request rather than showing an incorrect balance.
*   **Video Streaming**: Optimizing for Throughput over Latency during buffering, but switching to low latency for live streams (WebRTC).

### 4. Tradeoffs
*   **Cost vs. Scalability**: Horizontal scaling requires complex service discovery and load balancing. Vertical scaling is cheaper and easier until you hit the hardware ceiling.
*   **Consistency vs. Performance**: Strong consistency (CP) requires network round-trips for consensus (e.g., Paxos/Raft), which increases latency. Eventual consistency (AP) is faster but leads to "dirty reads."
*   **Complexity vs. Availability**: Adding more redundant layers (Active-Active regions) increases availability but makes deployment and state synchronization much harder.

### 5. When NOT to Use
*   **Microservices Everywhere**: Scaling horizontally across 50 microservices for a CRUD app with 10 users is an anti-pattern (Over-engineering).
*   **Strong Consistency**: Don't enforce it for non-critical data. If you can tolerate "Eventual Consistency" for a profile picture update, take the performance win.

### 6. Interview Focus
*   **Estimation**: "If our app has 10M DAU and each user uploads 2 photos/day, how much storage and bandwidth do we need?"
*   **Scenario Choice**: "Why would you choose NoSQL (usually AP) over SQL (usually CP) for a global real-time chat app?"
*   **Bottlenecks**: "The load balancer is healthy, but the system is slow. Where do you look first? (DB locks, CPU, or Network IO?)"

### 7. Common Mistakes
*   **Ignoring Network Partitions**: Thinking "our network is stable," so CAP doesn't apply. Partition Tolerance is not optional.
*   **Optimizing the Wrong Metric**: Reducing Latency from 50ms to 20ms when the real problem is that the system crashes (Throughput) at 1000 users.
*   **Single Point of Failure (SPOF)**: Having a perfectly scalable API layer but a single, non-redundant primary database that can't be scaled or failed over.
