# NoSQL Landscape: Key-Value, Document, Column, & Graph

### 1. Overview
NoSQL (Not Only SQL) databases provide a mechanism for storage and retrieval of data modeled in means other than the tabular relations used in relational databases. For senior engineers, picking a NoSQL store is about matching the **Data Model** and **Access Pattern** to the right storage engine.

### 2. Key Concepts
*   **Key-Value**: Simplest form (e.g., Redis, DynamoDB). Map a unique key to a value. Best for high-speed caching and simple session state.
*   **Document**: Stores data as JSON-like documents (e.g., MongoDB, CouchDB). Schema-less and flexible. Great for content management.
*   **Column Store / Wide Column**: Stores data in columns instead of rows (e.g., Cassandra, HBase). Optimized for reading massive amounts of data in specific columns (Analytics/Time-series).
*   **Graph DB**: Focuses on relationships between entities (e.g., Neo4j). Entities are "Nodes" and relationships are "Edges." Perfect for social networks or fraud detection.

### 3. Real-World Usage
*   **Leaderboards/Session Stores**: Using **Redis** (Key-Value) for sub-millisecond access to millions of active sessions.
*   **Product Catalogs**: Using **MongoDB** (Document) where different products have wildly different attributes (e.g., a "Shirt" has size/color; a "Laptop" has CPU/RAM).
*   **IoT Aggregation**: Using **Cassandra** (Wide Column) to store millions of sensor readings per second and querying by time range.
*   **Fraud Detection/Recommendations**: Using **Neo4j** (Graph) to find "friends of friends who also bought this item" or "money transfers connecting 5 suspicious accounts."

### 4. Tradeoffs
*   **Flexibility vs. Integrity**: NoSQL is easy to evolve because there is no fixed schema, but you lose the "Safety Valve" of DB-level constraints (Joins/Foreign Keys). You must handle integrity in the application code.
*   **Eventual Consistency vs. Availability**: Most NoSQL DBs prioritize Availability over Strict Consistency (The "AP" in CAP).
*   **Query Power**: NoSQL usually has limited query capabilities compared to SQL. You often have to "duplicate" data to satisfy different query patterns (Denormalization).

### 5. When NOT to Use
*   **Complex Financial Transactions**: If you need strict ACID across multiple records, SQL is almost always a better, safer choice.
*   **Unknown Access Patterns**: NoSQL requires you to design your schema based on your queries (Query-First Design). If you don't know how you'll query the data yet, stick to SQL.

### 6. Interview Focus
*   **The Model Choice**: "Given a scenario like 'LinkedIn connection graph,' why would a Graph DB be 100x faster than a SQL DB with a 'Connections' table?"
    * **SQL Approach**: To find "Friends of Friends," you would need to join the `Connections` table with itself multiple times (recursive join). This becomes extremely slow (exponential complexity) as the depth of the search increases (e.g., friends of friends of friends).
    * **Graph DB Approach**: A Graph DB (like Neo4j) stores these connections as direct edges. Finding connections is a simple graph traversal operation. It has constant time complexity (O(1)) for traversing a single relationship, regardless of the total dataset size, making it vastly superior for connected data.
    * **Example**: In Neo4j, finding friends of friends is a single Cypher query that traverses 2 levels of relationships. In SQL, it requires multiple self-joins, which is computationally expensive.
    
    ```cypher
    // Neo4j: Find friends of friends
    MATCH (p:Person)-[:FRIEND]->()-[:FRIEND]->(fof:Person)
    WHERE p.name = 'Alice'
    RETURN fof
    ```

*   **Schema Design**: "Design the schema for a social feed in DynamoDB. How do you handle 'Partition Keys' to avoid 'Hot Partitions'?"
    * **Hot Partition**: When one partition key receives a disproportionately high volume of requests compared to others, causing a bottleneck. In DynamoDB, this can happen if you use a non-random partition key (e.g., `userId`) and a few users are very active.
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

    Query all shards in parallel, Merge + sort in application
    
*   **Consistency Tradeoffs**: "Explain why MongoDB might report a write as successful even if it hasn't reached all replicas. How do you fix this?" (Hint: Write Concerns).
    * **MongoDB Write Concerns**: By default, MongoDB's write concern is `w:1`, which means the write is acknowledged by the primary node. It does not wait for replication to other nodes.
    * **Fix**: To ensure data durability, you can increase the write concern to `w: 'majority'` or `w: <number_of_replicas>`. This requires the write to be acknowledged by a majority of the replica set members before it is considered successful.
    ```
    // MongoDB Write Concern Example
    db.collection.insertOne(
        { name: "Alice" },
        { writeConcern: { w: "majority", wtimeout: 5000 } } // Wait for majority, with 5s timeout
    );
    ```
    * **CAP Theorem**: "In a distributed system, you can only have two out of three: Consistency, Availability, and Partition Tolerance. Which two would you pick for a banking system?"
        * **Answer**: **Consistency and Partition Tolerance (CP)**.
        * **Reason**: In a banking system, data accuracy is paramount. It's better to make the system unavailable during a network partition (preventing inconsistent writes) than to allow incorrect balances.
        * **Tradeoff**: You sacrifice Availability during network partitions. Users might see "System Unavailable" messages instead of potentially incorrect data.
        * **Example**: If a bank's primary node fails, the system should stop accepting writes (Consistency) rather than allowing writes to a secondary node that might have stale data (Availability). This ensures that once a transaction is committed, it is consistent across the system.

### 7. Common Mistakes
*   **The "One Database for Everything" Trap**: Trying to fit highly relational data into MongoDB or using Redis as a primary, persistent data store for critical financial data.
*   **Schema-less = No Schema**: Failing to define a schema in the application layer, leading to "Document Rot" where 10 different versions of a "User" object exist in the same collection.
*   **Scanning instead of Querying**: Performing massive scans in NoSQL because you didn't create the right indexes or Partition Keys, leading to huge costs and slow performance.
