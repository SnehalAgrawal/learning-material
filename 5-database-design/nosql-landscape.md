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
*   **Schema Design**: "Design the schema for a social feed in DynamoDB. How do you handle 'Partition Keys' to avoid 'Hot Partitions'?"
*   **Consistency Tradeoffs**: "Explain why MongoDB might report a write as successful even if it hasn't reached all replicas. How do you fix this?" (Hint: Write Concerns).

### 7. Common Mistakes
*   **The "One Database for Everything" Trap**: Trying to fit highly relational data into MongoDB or using Redis as a primary, persistent data store for critical financial data.
*   **Schema-less = No Schema**: Failing to define a schema in the application layer, leading to "Document Rot" where 10 different versions of a "User" object exist in the same collection.
*   **Scanning instead of Querying**: Performing massive scans in NoSQL because you didn't create the right indexes or Partition Keys, leading to huge costs and slow performance.
