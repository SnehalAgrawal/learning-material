# Architecture Styles: Monolith, Microservices, & EDA

### 1. Overview
Architecture styles define the high-level organization of a software system. A senior engineer must move beyond "Microservices are better" and understand that architecture is a response to **Organizational Structure** (Conway's Law) and **Domain Complexity**.

### 2. Key Concepts
*   **Monolith**: Single deployment unit. Easy to develop and test initially.
*   **Modular Monolith**: Code is logically separated into independent modules but deployed as one unit. Combines Monolith simplicity with Microservice boundaries.
*   **Microservices**: Distributed services grouped by business capability. Each has its own DB, deployment pipeline, and scaling policy.
*   **Event-Driven Architecture (EDA)**: Services communicate via asynchronous events (Pub/Sub). Decouples the "Producer" from the "Consumer."
*   **Serverless (FaaS)**: Execution model where the cloud provider manages infrastructure entirely; functions run based on triggers (HTTP, S3 events).

**Microservices vs Event-Driven Architecture**
| Feature           | Microservices                    | Event-Driven Architecture (EDA)  |
|-------------------|----------------------------------|----------------------------------|
| Communication     | Direct (REST, gRPC, queues)    | Indirect (Events via broker)     |
| Coupling          | Loose coupling                   | Very loose coupling              |
| Scalability       | Scale services independently   | Scale event processing           |
| Data Flow         | Request/Response (sync/async)  | Event streams (async)            |
| Complexity        | Moderate (service orchestration) | High (event tracing, ordering)   |
| Best For          | Business domains, workflows    | Real-time processing, IoT, logs  |

### 3. Real-World Usage
*   **Early Stage Startup**: A pure Monolith (Rails/Django) to iterate fast and find product-market fit.
*   **Scale-up (Uber/Netflix)**: Hundreds of Microservices to allow independent teams to ship features. Often requires the [Saga Pattern](../2-design-patterns/cloud-native-patterns.md) for data consistency.
*   **Log Processing**: EDA where a "Log Producer" emits events and multiple consumers (Elasticsearch, S3, Slack Alerts) process them independently.
*   **Image Resizing**: Using Serverless (AWS Lambda) to trigger a resize function whenever a user uploads a photo to S3.

### 4. Tradeoffs
*   **Microservices vs. Monolith**: Microservices offer team autonomy and scaling but introduce "Distributed System Tax" (Network latency, data consistency, observability nightmare).
*   **EDA vs. REST**: EDA makes a system more extensible but much harder to trace and debug "phantom" bugs triggered by event cascades.
*   **Serverless Cost**: Serverless is cheap for low, bursty traffic but significantly more expensive than reserved instances for steady-state, high-volume workloads.

### 5. When NOT to Use
*   **Microservices**: Do NOT use if your team is small (< 10-15 people). You'll spend more time managing infrastructure than writing business logic.
*   **EDA**: Avoid for synchronous requirements. If a user needs an immediate "Success/Fail" based on 5 external calls, a synchronous REST/gRPC approach is cleaner.

### 6. Interview Focus
*   **The "Why"**: "Why would you split this Monolith today? What specific pain points are we solving?"
    * I’d split a monolith only if it’s causing real pain — like slow deployments, tight coupling between teams, or scalability issues where one module limits the entire system.
    * Microservices help by enabling independent deployments, team ownership, and scaling specific components instead of the whole system.
*   **Data Strategy**: "How do you handle 'JOINS' in a microservice architecture where User data and Order data are in different DBs?" (API Composition vs. CQRS).
    * Since cross-service joins aren’t feasible, we use two main approaches:
    * API Composition — a service calls multiple services at runtime and aggregates results (simpler but higher latency).
    * CQRS with denormalization — we maintain a read model that combines User + Order data using events (faster reads, eventual consistency).
    * Choice depends on latency vs consistency tradeoffs.
*   **Failure Modes**: "What happens in our EDA if the Message Broker (Kafka/RabbitMQ) goes down?"
    * If the broker goes down, producers can’t publish events and consumers can’t receive them, so the system may stall.
    * To handle this, we use replication (clustered brokers), retries with backoff, durable message storage, and fallback mechanisms like local queues or temporary persistence.
    * The goal is to avoid data loss and recover gracefully once the broker is back.
*   **Monolith vs Microservices**: "When would you choose Monolith over Microservices?"
    * I’d choose a monolith when:
        * Team is small (under 10 developers)
        * Domain is simple and not rapidly evolving
        * Need to iterate quickly with few moving parts
        * Operational simplicity is prioritized over independent scaling
    * Microservices add overhead (CI/CD, monitoring, orchestration) that can slow down early-stage teams. A well-modularized monolith is often better than poorly implemented microservices.
*   **Serverless**: "When would you choose Serverless over traditional servers?"
    * Serverless shines when:
        * Workloads are unpredictable (spiky traffic)
        * Need to scale to zero (idle resources = zero cost)
        * Focus is on business logic, not infra management
        * Event-driven triggers (file uploads, webhooks) are common
    * Ideal for MVPs, APIs, data processing pipelines
    * Not great for steady, heavy workloads (more expensive) or long-running processes (timeouts)
*   **EDA vs REST**:
    * Use EDA when you need loose coupling, scalability, and real-time processing (e.g., notifications, logs, IoT).
    * Use REST when you need synchronous communication, immediate responses, and simpler debugging (e.g., CRUD operations, workflows).
    * Many systems use both: REST for commands (user actions) and EDA for events (system reactions).

### 7. Common Mistakes
*   **Distributed Monolith**: Building microservices that are so tightly coupled via synchronous HTTP calls that if one goes down, they ALL go down.
*   **Shared Databases**: Multiple microservices pointing to the same DB schema. This destroys the primary benefit of microservices (Independent Evolution).
*   **Ignoring Observability**: Moving to microservices without distributed tracing (Jaeger/Zipkin) and centralized logging.
