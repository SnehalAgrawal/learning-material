# Stream Processing: Exactly-Once & Backpressure

### 1. Overview
Stream processing is the continuous processing of data as it arrives (unbounded data), as opposed to "Batch Processing" (bounded data). For senior engineers, the focus is on handling high-velocity data, maintaining internal state safely, and ensuring correctness through exactly-once processing.

### 2. Key Concepts
*   **Unbounded Data**: Data that has no defined end (events, sensor data, logs).
*   **Exactly-Once Processing**: The promise that even if a system fails, the final *state* (e.g., total count of clicks) reflects each message being processed only once.
*   **Backpressure**: A mechanism where a slow consumer tells the producer to slow down, preventing the consumer from being overwhelmed and crashing.
*   **Windowing**: Grouping events by time intervals (e.g., "Total sales in the last 10 minutes").
    *   *Tumbling Windows*: Fixed size, non-overlapping blocks.
    *   *Sliding Windows*: Overlapping blocks.
*   **Stateful Processing**: The system remembers previous events (e.g., calculating a running average).

### 3. Real-World Usage
*   **Fraud Detection**: Analyzing a stream of credit card transactions to find patterns (e.g., 3 transactions in 3 different countries within 1 hour).
*   **Real-time Dashboards**: Stock tickers or sports betting platforms where every millisecond of delay costs money.
*   **IoT Analytics**: Processing millions of data points from smart meters to detect power grid anomalies.
*   **Ad Tech**: Counting ad impressions and clicks in real-time to adjust bidding strategies.

### 4. Tradeoffs
*   **Throughput vs. Latency**: Batching events increases throughput (more data processed per second) but increases latency for individual events.
*   **Correctness vs. Performance**: Exactly-once processing requires checkpoints or distributed transactions, which slows down the processing speed compared to "at-least-once."
*   **State Management**: Storing state in the stream processor (e.g., Apache Flink) makes the system complex to scale and recover from failures.

### 5. When NOT to Use
*   **Historical Analysis**: If you need to analyze data from 3 years ago and don't need the result in milliseconds, use a Batch processor (Spark, SQL).
*   **Transactional CRUD**: Standard business logic (creating a user) should happen in a database, not a stream processor.

### 6. Interview Focus
*   **Late Data**: "How do you handle events that arrive out of order or late due to network delays?" (Hint: Watermarks).
*   **Scaling Stateful Systems**: "How do you scale a stream processor that is currently calculating a 24-hour moving average for 1 million keys?"
*   **Handling Spikes**: "Your stream processor is falling behind the input data. What are your first 3 actions?" (Scale horizontally, check for backpressure, optimize state access).

### 7. Common Mistakes
*   **Assuming Clock Sync**: Thinking all messages arrive in the order they were created. Never rely on the system clock of the stream processor; use the "Event Time" from the message itself.
*   **Ignoring Checkpoints**: Not configuring robust state checkpoints/backups. If the processor crashes, you lose your running totals (the "State").
*   **Memory Overload**: Using a "Sliding Window" that is too large (e.g., 1 week of data), causing the stream processor to run out of RAM trying to keep all those events in memory.
