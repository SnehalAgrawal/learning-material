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
    * **Watermarks** : A watermark is a threshold that represents the latest timestamp that the system expects to have seen. Any event with a timestamp earlier than the watermark is considered "late" and is dropped or handled separately. Example:
        ```javascript
        const watermark = eventTime => eventTime - 5000; // 5 second grace period
        ```
*   **Scaling Stateful Systems**: "How do you scale a stream processor that is currently calculating a 24-hour moving average for 1 million keys?"
    * **Horizontal Scaling**: Use a distributed stream processing framework (e.g., Apache Flink, Apache Spark Streaming) that supports horizontal scaling. Each instance can process a subset of the keys, and the keys can be repartitioned across instances as needed.
    * **Partitioning**: Partition the data by a key (e.g., user ID, device ID) to distribute the state across multiple instances. Each instance maintains the state for its assigned partition(s).
    * **State Management**: Use a fault-tolerant state management system (e.g., RocksDB, HDFS) to store the state. Regular checkpoints of the state should be taken to enable recovery from failures.
    * **Example**: Flink's Keyed State and Checkpointing mechanisms allow for horizontal scaling of stateful stream processing applications. Flink's Keyed State allows you to maintain state that is partitioned by a key, and Flink's Checkpointing mechanism allows you to take periodic snapshots of the state. These snapshots can be used to recover from failures and to scale the application horizontally.
        ```
        // Flink Keyed State Example
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Event> stream = env.addSource(new FlinkKafkaConsumer<>("events", new EventSchema(), properties));

        // Key the stream by user ID
        KeyedStream<Event, String> keyedStream = stream.keyBy(event -> event.getUserId());

        // Maintain state per user (e.g., count of events)
        DataStream<UserEventCount> result = keyedStream
            .mapWithState(new CountEventsStateFunction())
            .keyBy(result -> result.getUserId());
        ```
*   **Handling Spikes**: "Your stream processor is falling behind the input data. What are your first 3 actions?" (Scale horizontally, check for backpressure, optimize state access).
    * **Scale Horizontally**: Increase the number of worker nodes/instances processing the stream. The load will be redistributed across the increased capacity.
    * **Check for Backpressure**: Identify where the bottleneck is. Is the source (e.g., Kafka) producing data faster than the processor can consume? Is the sink (e.g., database) unable to keep up with the write volume?
    * **Optimize State Access**: If the processor is stateful, check if state access is optimized. For example, using RocksDB as the state backend can handle large state sizes more efficiently than in-memory state.
    * **Example**: If a stream processor handling clickstream data starts falling behind, you can scale horizontally by increasing the number of Kafka partitions and assigning more processor instances to consume from those partitions. You can also optimize state access by using RocksDB as the state backend and configuring it to use disk storage instead of memory.
    ```
    // Flink backpressure example
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    DataStream<Event> stream = env.addSource(new FlinkKafkaConsumer<>("events", new EventSchema(), properties));

    // Key the stream by user ID
    KeyedStream<Event, String> keyedStream = stream.keyBy(event -> event.getUserId());

    // Maintain state per user (e.g., count of events)
    DataStream<UserEventCount> result = keyedStream
        .mapWithState(new CountEventsStateFunction())
        .keyBy(result -> result.getUserId());
    ```

### 7. Common Mistakes
*   **Assuming Clock Sync**: Thinking all messages arrive in the order they were created. Never rely on the system clock of the stream processor; use the "Event Time" from the message itself.
*   **Ignoring Checkpoints**: Not configuring robust state checkpoints/backups. If the processor crashes, you lose your running totals (the "State").
*   **Memory Overload**: Using a "Sliding Window" that is too large (e.g., 1 week of data), causing the stream processor to run out of RAM trying to keep all those events in memory.
