# Distributed Core: Consensus & Coordination

### 1. Overview
The core of distributed systems involves getting a group of independent machines to agree on a single value or state despite network failures. For a senior engineer, this is about understanding how "Source of Truth" is established and maintained across a cluster through Consensus and Leader Election.

### 2. Key Concepts
*   **Consensus Basics**: The process of reaching agreement among a group of nodes. Crucial for keeping distributed logs or state machines consistent.
*   **Leader Election**: Automatically choosing one node as the "Primary" coordinator. If the leader fails, a new one is elected.
*   **Replication**: Copying data across multiple nodes to ensure durability and availability.
*   **Quorum**: The minimum number of nodes that must agree for an operation to be considered successful (Usually `(N/2) + 1`).

### 3. Real-World Usage
*   **Service Discovery (Consul/Etcd)**: Using the **Raft** consensus algorithm to ensure that every service in the cluster sees the same set of available backend URLs.
*   **Distributed Locking**: Using **ZooKeeper** to ensure that only one "Worker Node" processes a specific background job at a time.
*   **Database Clusters**: MongoDB or PostgreSQL with "Automatic Failover"—the nodes use a heartbeat and election protocol to promote a secondary to primary if the original fails.
*   **Log Management**: Kafka uses a "Controller" node that is elected to manage partitions and offsets.

### 4. Tradeoffs
*   **Consistency vs. Latency**: Consensus algorithms require multiple network round-trips (Propose -> Vote -> Commit), which is significantly slower than writing to a single local disk.
*   **Availability vs. Partition Size**: If you lose a majority of nodes (e.g., in a 5-node cluster, you lose 3), the system becomes read-only or shuts down entirely to prevent split-brain.
*   **Raft vs. Paxos**: Raft is designed to be understandable and is the industry standard today; Paxos is the original but notoriously difficult to implement correctly.

### 5. When NOT to Use
*   **Single-Region Apps**: If you only run one instance of a database or service, you don't need consensus algorithms.
*   **High-Write Low-Consistency**: For things like "Video view counts," using a Quorum-based consensus is overkill and slow. Use simple asynchronous incrementing.

### 6. Interview Focus
*   **Split-Brain Problem**: "What happens if two nodes both think they are the Leader? How does your system prevent data corruption?" (Hint: Fencing Tokens / Generation clock).
*   **Quorum Math**: "Why do we typically use an odd number of nodes (3, 5, 7) for consensus clusters?"
*   **Algorithmic Insight**: "Explain high-level how the Raft algorithm ensures that a new leader has all the committed logs from the previous leader."

### 7. Common Mistakes
*   **Manual Failover**: Relying on human intervention to promote a DB secondary to primary, leading to massive downtime during off-hours.
*   **Ignoring Network Jitter**: Setting "Heatbeat" timeouts too low, causing "Election Storms" where the cluster spends all its time electing new leaders instead of doing work.
*   **Thinking Replication is Backup**: Replication handles *Availability*; if you accidentally run `DROP TABLE`, that command is replicated instantly. You still need snapshots/backups.
