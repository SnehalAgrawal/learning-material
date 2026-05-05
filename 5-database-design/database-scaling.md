# Database Scaling: Replicas, Sharding, & Partitioning

### 1. Overview
Database scaling is the process of expanding a database to handle increasing amounts of data and traffic. For a senior engineer, this is about moving from a single instance to a distributed system, managing the complexity of data distribution, and ensuring consistency across nodes.

### 2. Key Concepts
*   **Read Replicas**: Creating "Follower" nodes that mirror the "Leader." The Leader handles all writes; Followers handle only reads.
*   **Sharding (Horizontal Partitioning)**: Splitting a single large dataset across multiple independent database instances. Each instance (Shard) holds a fraction of the data.
*   **Vertical Partitioning**: Splitting a table by columns. (e.g., putting "Big BLOBS" like `profile_image` in one table and `username/email` in another).
*   **Partition Key / Shard Key**: The unique field used to decide which shard/node a piece of data belongs to.
*   **Federation**: Splitting databases by business function (e.g., a "Users DB," a "Orders DB," a "Products DB").

### 3. Real-World Usage
*   **Social Media Feed**: 99% of traffic is reading. Using 1 Leader for writes and **10 Read Replicas** to handle the massive read volume.
*   **Global SaaS (Slack/Discord)**: Using **Sharding** where each "Workspace" or "Guild" has its own shard. A user in Workspace A never touches the database for Workspace B.
*   **E-Commerce Search**: Moving "Product Descriptions" and "Reviews" to a separate **Vertical Partition** or even a different DB (Elasticsearch) to keep the core "Order" table lean and fast.
*   **Multi-Region Failover**: Maintaining a "Standby" replica in a different geographic region to recover from a total region data-center outage.

### 4. Tradeoffs
*   **Read Replicas vs. Consistency**: Replicas are usually asynchronous. A user might write to the Leader and immediately read from a Follower that hasn't received the update yet ([Replication Lag](../4-distributed-systems/consistency-models.md)).
*   **Sharding vs. Complexity**: Sharding is the "End Game" of scaling but it makes across-shard `JOINs` and "Aggregations" nearly impossible. It also makes "Re-sharding" (moving data as shards grow) extremely difficult.
*   **Shard Key Choice**: A bad shard key leads to "Hot Shards" (one server doing all the work while the others are idle).

### 5. When NOT to Use
*   **Sharding**: **Avoid at all costs** until you have exhausted every other option (Vertical scaling, SQL optimization, Caching, Read Replicas). Sharding adds immense architectural overhead.
*   **Read Replicas**: If your app is write-heavy (1:1 read/write ratio), adding read replicas won't help; it will actually add more load to the Leader to manage replication.

### 6. Interview Focus
*   **The Scaling Strategy**: "Our SQL database is reaching its limits. Walk me through the steps you would take to scale it, starting with the easiest."
    * **Vertical Scaling**: Increase RAM/CPU of the existing server.
    * **Read Replicas**: Add read replicas to handle read traffic.
    * **Caching**: Implement Redis or Memcached to reduce database load.
    * **Sharding**: Partition data across multiple servers if vertical scaling is not enough.
    * **Database Optimization**: Optimize queries and add indexes.
    * **Connection Pooling**: Use connection pooling to reduce connection overhead.
    * **Denormalization**: Duplicate data to reduce joins and improve query performance.
    * **Materialized Views**: Precompute and store query results for faster access.
*   **Shard Key Design**: "If we shard our 'Uber-like' app by `city_id`, what happens when New York City has 100x more traffic than Buffalo? How do you fix the hot shard?"
    * **Hot Shard**: When one partition key receives a disproportionately high volume of requests compared to others, causing a bottleneck. In DynamoDB, this can happen if you use a non-random partition key (e.g., `userId`) and a few users are very active.
    * **Solution**: Use **Composite Keys** with a **Randomized Partition Key** or **Sharding**.
    ```
    // DynamoDB Schema for Social Feed
    {PartitionKey: 'USER#123', SortKey: 'TIMESTAMP#2024-01-01T10:00:00Z', data: {...}} // Actual Post
    {PartitionKey: 'USER#123', SortKey: 'FEED#2024-01-01T10:00:00Z', data: {...}} // Aggregated Feed Item
    ```
    To avoid hot partitions, you can hash the partition key or use a random prefix to distribute data across multiple physical partitions.
    ```
    // Randomized Partition Key Example
    {PartitionKey: 'USER#123#abc', SortKey: 'TIMESTAMP#2024-01-01T10:00:00Z', ...} 
    ```
*   **Replication Lag**: "How do you handle the case where a user updates their profile and the 'Profile View' page still shows old data?"
    * **Replication Lag**: When a user updates their profile, the changes are written to the primary database. However, due to network latency or replication delay, the read replicas may not have received the updated data yet. As a result, when a user views their profile page, they might see stale data from a replica that hasn't been updated.
    * **Solution**: Use **Read-Your-Writes Consistency** or **Synchronous Replication**.
    ```
    // Read-Your-Writes Consistency Example
    After a successful write to the primary database, explicitly read from the primary for the next request.
    ```

### 7. Common Mistakes
*   **The "Big Shard" Mistake**: Waiting too long to shard. By the time the DB is too big, moving it to a sharded architecture requires days of downtime.
*   **Ignoring Failover Latency**: Having replicas but not testing the "Switchover" process. A failover that takes 30 minutes is not a high-availability solution.
*   **Sharding by Incremental ID**: Sharding by `user_id % 10`. This works initially but creates a nightmare when you need to add an 11th shard. Use **Consistent Hashing**.
