# Cloud-Native Design Patterns

### 1. Overview
Cloud-native patterns address the challenges of building and running scalable, resilient applications in modern distributed environments (like Kubernetes or AWS). These patterns focus on maintaining data consistency, managing failures, and decoupling services in a world where "everything fails all the time."

### 2. Key Concepts
*   **Saga Pattern**: Manages failures in distributed transactions by using a sequence of local transactions. Each local transaction updates the database and publishes a message to trigger the next one. If one fails, "compensating transactions" are executed to undo the previous steps.
*   **Outbox Pattern**: Solves the "Dual Write" problem. It ensures that a database update and a message publication happen atomically by writing the message to a "Local Outbox" table in the same transaction as the business data.
*   **Circuit Breaker**: Prevents a failing service from causing a cascading failure across the system. It "trips" after a threshold of failures, immediately returning an error/fallback for subsequent calls until the service recovers.
*   **Bulkhead**: Isolates elements of an application into pools so that if one fails, the others will continue to function. (e.g., separating thread pools for critical vs. non-critical APIs).

### 3. Real-World Usage
*   **E-commerce Checkout**: A Saga is used to handle `Stock Reservation -> Payment -> Shipping`. If Payment fails, Stock is released via a compensating transaction.
*   **Reliable Messaging**: A microservice updates a "Order" status and writes an "OrderCreated" event to its `outbox` table. A separate process polls the outbox and sends it to Kafka.
*   **External API Integration**: Using a Circuit Breaker (like Resilience4j or Polly) when calling a third-party weather API that is frequently down.
*   **Search vs. Checkout**: Placing the "Product Search" and "Cart Checkout" on different thread pools to ensure a spike in search traffic doesn't crash the checkout flow.

### 4. Tradeoffs
*   **Saga Complexity**: Choreographed Sagas (events) are decentralized but hard to debug; Orchestrated Sagas (manager) are easier to trace but create a central point of logic.
*   **Outbox Latency**: Adds a small delay between the DB write and the message being visible on the bus, and requires extra infrastructure for the "relay" process.
*   **Circuit Breaker Fallbacks**: Designing meaningful fallbacks (e.g., returning cached data) requires extra effort and product-level decisions.

### 5. When NOT to Use
*   **Saga**: If you can use a single database with ACID transactions (e.g., a modular monolith), do NOT use Sagas. They introduce massive eventual consistency complexity.
*   **Circuit Breaker**: For extremely fast internal calls where failure is rare and the overhead of tracking metrics outweighs the benefits.

### 6. Interview Focus
*   **Distributed Transactions**: "How do you maintain data consistency between two microservices that own their own databases?"
*   **Resilience**: "What happens to your system if Service B starts responding slowly? How do you prevent it from taking down Service A?"
*   **Atomic Operations**: "Explain the problem of 'sending an email after a DB commit' and how the Outbox pattern fixes it."

### 7. Common Mistakes
*   **Missing Compensating Transactions**: In a Saga, forgetting to handle the "Cleanup" phase for one of the steps, leaving the system in an inconsistent state.
*   **Circuit Breaker Settings**: Setting failure thresholds too low (causing "flapping") or too high (ignoring real failures).
*   **Neglecting Monitoring**: Not setting up alerts when a Circuit Breaker trips, leaving the system in an "Open" (failing) state without the team knowing.
