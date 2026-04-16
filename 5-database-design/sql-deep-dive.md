# SQL Deep Dive: Indexing, Transactions, & Normalization

### 1. Overview
Relational Databases (SQL) are the bedrock of transaction systems. For a senior engineer, SQL mastery is not about writing basic `SELECT` statements, but about understanding the cost of storage models (B-Trees), the trade-offs of the ACID properties, and how to optimize for high-concurrency environments.

### 2. Key Concepts
*   **Indexing**: Using B-Trees or Hash indexes to speed up reads.
    *   *Clustered Index*: The physical ordering of data on disk (usually the Primary Key).
    *   *Non-Clustered Index*: A separate structure pointing back to the data.
*   **Transactions (ACID)**:
    *   *Atomic*: All or nothing.
    *   *Consistent*: DB moves from one valid state to another.
    *   *Isolated*: Transactions don't interfere (Isolation levels: Read Uncommitted, Read Committed, Repeatable Read, Serializable).
    *   *Durable*: Committed data survives crashes.
*   **Normalization (1NF to 3NF)**: Structuring data to reduce redundancy and maintain integrity.
*   **Denormalization**: Intentionally introducing redundancy to speed up specific complex read queries.

### 3. Real-World Usage
*   **Financial Transactions**: Moving money between accounts requires a single **ACID Transaction** to ensure no money is "created" or "lost" during the move.
*   **Search Optimization**: Creating a **Compound Index** (e.g., `(user_id, created_at)`) to optimize the query for a user's recent activity feed.
*   **Data Consistency**: Using **Foreign Keys** and **Check Constraints** to prevent "orphan" data (e.g., an Order without a valid User).
*   **Analytics Dashboards**: Using a **Star Schema** (Denormalization) to provide fast aggregations without joining 20 tables.

### 4. Tradeoffs
*   **Read vs. Write Speed**: Adding an index makes reads faster but makes every `INSERT`/`UPDATE` slower as the index must be updated too.
*   **Isolation vs. Concurrency**: "Serializable" is perfectly safe but causes lots of "Deadlocks" in high-traffic apps. "Read Committed" is fast but allows "Non-repeatable reads."
*   **Normalization vs. Joins**: Highly normalized data is easy to maintain but requires complex, slow `JOIN` operations.

### 5. When NOT to Use
*   **Unstructured Data**: Don't use SQL if you need to store massive, variable JSON blobs that change daily. Use a Document store.
*   **High-Volume Write-Only Data**: For things like "Server Logs" or "Sensor Data," the overhead of ACID and Indexing is too high. Use a Time-Series or Columnar DB.

### 6. Interview Focus
*   **Execution Plans**: "How do you use `EXPLAIN ANALYZE` to find why a 1-second query is taking 10 seconds?"
*   **Deadlock Resolution**: "Two users try to update each other's status at the same time. A deadlock occurs. How does the DB handle it? How do you prevent it?"
*   **Design Question**: "Design the schema for a School Management System. How do you handle a student having multiple classes and a class having multiple students?" (Many-to-Many).

### 7. Common Mistakes
*   **The "Select *" Trap**: Fetching 50 columns when you only need one, bloating memory usage and preventing the database from using "Covering Indexes."
*   **Indexing Every Column**: Thinking "more indexes = more speed." This crashes write performance and wastes disk space.
*   **N+1 Query Problem**: Executing a query inside a loop (e.g., fetching 100 posts, then making 100 separate DB calls to fetch authors).
