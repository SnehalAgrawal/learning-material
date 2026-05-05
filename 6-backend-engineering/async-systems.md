# Async Systems: Job Queues & Backpressure

### 1. Overview
Asynchronous systems allow a backend to handle long-running or non-critical tasks without blocking the main request-response cycle. For senior engineers, this is about managing the complexity of "Background Work," ensuring durability, and handling systems that are overwhelmed (Backpressure).

### 2. Key Concepts
*   **Job Queues (Task Queues)**: A mechanism to offload work to background workers (e.g., Celery, BullMQ, Sidekiq).
*   **Message Broker**: The middleware that stores the jobs (Redis, RabbitMQ, SQS).
*   **Worker**: A separate process that polls the queue and executes the job.
*   **Backpressure**: When the number of incoming jobs exceeds the workers' capacity, the system must either slow down the producer, drop messages (Shedding Load), or scale workers.
*   **Visibility Timeout**: The amount of time a job is "hidden" from other workers after being picked up. If not acknowledged within this time, it becomes visible again.

### 3. Real-World Usage
*   **Report Generation**: A user clicks "Download Yearly PDF." The API returns `202 Accepted`, puts a job in the queue, and later notifies the user via Email/WebSocket when the PDF is ready in S3.
*   **Third-Party Webhooks**: When Stripe sends a payment notification, you save it to a queue and respond `200 OK` immediately to avoid Stripe retrying due to your slow internal processing.
*   **Email Campaigns**: Using a queue to send 100,000 emails. If the email provider (SES/SendGrid) throttles you, the queue naturally handles the "Backpressure" by holding the remaining jobs.
*   **Image/Video Processing**: Transcoding a 1GB video file is a high-CPU task that must be done asynchronously.

### 4. Tradeoffs
*   **Latency vs. Resilience**: Async systems provide high resilience (jobs aren't lost) but move the "Success" confirmation away from the user's initial interaction.
*   **Observability**: Debugging an "Async chain" (API -> Queue -> Worker A -> Queue -> Worker B) is significantly harder than a synchronous call stack.
*   **Resource Management**: Running 100 background workers uses a lot of RAM/CPU even when the queue is empty unless using Auto-scaling.

### 5. When NOT to Use
*   **Small, Fast Tasks**: If a task takes < 50ms (like updating a single SQL field), the overhead of serializing a message and putting it in Redis is higher than just doing it synchronously.
*   **Dependency on Immediate Result**: If the very next screen in the UI *must* have the data from the task, doing it async will cause a race condition where the UI loads before the data is ready.

### 6. Interview Focus
*   **Concurrency vs. Parallelism**: "How many workers can your system handle? Is it CPU-bound or I/O-bound?"
*   **Failure Handling**: "A worker crashes while generating a report. How do you ensure that report is eventually generated?" (Hint: Acknowledgments & Visibility Timeouts).
*   **Scaling the Producer**: "What happens if the queue size reaches 1 million? How do you implement Backpressure to prevent the producer from filling the disk?"

### 7. Common Mistakes
*   **The "Invisible" Failure**: Not having a DLQ (Dead Letter Queue). Jobs fail, go back to the head of the queue, fail again, and loop forever, consuming all worker resources.
*   **Missing Idempotency**: A worker processes a job, sends an email, but crashes *before* acknowledging the job. The job is retried, and the user gets the same email twice.
*   **Database Overload**: Scaling to 500 workers to process a queue, only to have all 500 workers overwhelm the single database with concurrent connections.
