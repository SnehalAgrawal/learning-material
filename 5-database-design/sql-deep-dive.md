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
    * **EXPLAIN**: Shows the query plan (how the DB *intends* to execute the query).
    * **ANALYZE**: Actually executes the query and shows the *actual* runtime statistics.
    * Key things to look for: Sequential Scans on large tables, Nested Loop joins that could be Hash Joins, High Row Estimates vs Actuals (indicating bad statistics or missing indexes).
    
    
    // Example Output Analysis
    ```
    Seq Scan on users  (cost=0.00..2500.00 rows=1000 width=100)
    (actual time=0.050..50.000 rows=1000 loops=1)

    ```
    **Analysis**: We are doing a "Sequential Scan" on the `users` table, which means we are reading every single row. If the table has 10 million rows, this will be very slow. We should add an index on the column we are filtering by.
    
    // With Index
    ```
    Index Scan using idx_users_email on users (cost=0.00..8.00 rows=1 width=100)
    (actual time=0.030..0.040 rows=1 loops=1)
    Filter: (email = 'test@example.com')
    ```
    **Analysis**: Now we are using an index scan, which is much faster as it only reads the specific row we need.

*   **Deadlock Resolution**: "Two users try to update each other's status at the same time. A deadlock occurs. How does the DB handle it? How do you prevent it?"
    * **How DB handles it**: The database detects the deadlock (usually via a "Deadlock Detector" thread that checks for circular dependencies in the wait-for graph) and aborts one of the transactions (the "victim"). The victim transaction is rolled back, and the lock it held is released, allowing the other transaction to proceed.
    * **Prevention Techniques**:
        * **Consistent Lock Ordering**: Always acquire locks in the same order. For example, always update User A before User B. If both transactions try to do this, one will succeed and the other will wait, preventing a cycle.
        * **Lock Timeout**: Set a `lock_timeout` (e.g., `SET lock_timeout = '10s'`). If a transaction has to wait longer than this for a lock, it fails automatically, breaking the deadlock.
        * **Deadlock Monitoring**: Use `pg_stat_activity` (Postgres) or `SHOW ENGINE INNODB STATUS` (MySQL) to monitor for deadlocks and analyze the queries involved.
    ```sql
    -- Example: Setting a lock timeout in PostgreSQL
    SET LOCAL lock_timeout = '1s';

    BEGIN;
    -- User A updates their status
    UPDATE users SET status = 'online' WHERE id = 1;
    -- User B updates their status (trying to lock User A's row)
    UPDATE users SET status = 'online' WHERE id = 1;
    COMMIT;
    ```
    **Scenario**: User A and User B try to update each other's status simultaneously. Without a timeout, they wait forever. With `lock_timeout = '1s'`, one of them will fail after 1 second, allowing the other to proceed.

*   **Design Question**: "Design the schema for a School Management System. How do you handle a student having multiple classes and a class having multiple students?" (Many-to-Many).
    * **Solution**: Use a **Many-to-Many Relationship** with a **Junction Table**.
    ```sql
    -- Students Table
    CREATE TABLE students (
        id INT PRIMARY KEY,
        name VARCHAR(100)
    );

    -- Classes Table
    CREATE TABLE classes (
        id INT PRIMARY KEY,
        name VARCHAR(100)
    );

    -- Junction Table (Many-to-Many)
    CREATE TABLE enrollments (
        student_id INT,
        class_id INT,
        PRIMARY KEY (student_id, class_id),
        FOREIGN KEY (student_id) REFERENCES students(id),
        FOREIGN KEY (class_id) REFERENCES classes(id)
    );
    ```
    **Explanation**:
    * A student can have many classes: `student_id` appears multiple times in `enrollments` with different `class_id`s.
    * A class can have many students: `class_id` appears multiple times with different `student_id`s.
    * The `PRIMARY KEY` on `(student_id, class_id)` ensures a student can't enroll in the same class twice.
    
    // Querying for a student's classes
    ```sql
    SELECT c.name 
    FROM enrollments e
    JOIN classes c ON e.class_id = c.id
    WHERE e.student_id = 1;
    ```

### 7. Common Mistakes
*   **The "Select *" Trap**: Fetching 50 columns when you only need one, bloating memory usage and preventing the database from using "Covering Indexes."
*   **Indexing Every Column**: Thinking "more indexes = more speed." This crashes write performance and wastes disk space.
*   **N+1 Query Problem**: Executing a query inside a loop (e.g., fetching 100 posts, then making 100 separate DB calls to fetch authors).
