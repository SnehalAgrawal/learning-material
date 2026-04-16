# System Design Framework: The 45-Minute Blueprint

### 1. Overview
The System Design interview is a test of your ability to handle ambiguity and drive a technical conversation. For senior/staff roles, you are expected to take the lead, clarify requirements, and build a cohesive architecture. This framework ensures you cover all critical components without getting lost in low-level details too early.

### 2. Key Concepts (The 6-Step Blueprint)
1.  **Requirements Clarification (5m)**: Define boundaries.
    *   *Functional*: What does it do? (e.g., "Post a tweet," "Follow users").
    *   *Non-Functional*: Scalability (DAU/MAU), Availability (99.99%), Latency (p95 < 200ms).
2.  **Back-of-the-envelope Estimation (5m)**: Determine scale.
    *   How much storage for 10 years? How many requests per second?
3.  **API Design & Contract (5m)**: Define how the world talks to your system (e.g., `POST /v1/tweets`).
4.  **Database Schema & Data Model (5m)**: Choose SQL vs. NoSQL and define tables/fields.
5.  **High-Level Design (10m)**: Draw the boxes (Load Balancer, App Servers, DB, Cache).
6.  **Deep Dive & Bottlenecks (15m)**: Focus on the hard parts (Scaling DB, Caching strategy, Failure modes).

### 3. Real-World Usage
*   **The "Design WhatsApp" Question**: Start by asking if we need "Read Receipts." Estimate media storage vs. text storage. Deep dive into "WebSockets vs. Long Polling" for real-time delivery.
*   **The "Design Netflix" Question**: Requirements: Video upload vs. Viewers. Deep dive into "CDN distribution" and "Encoding pipelines."
*   **The "Design TicketMaster" Question**: Deep dive into "Handling concurrency and distributed locks" to prevent double-booking the same seat.

### 4. Tradeoffs
*   **Breadth vs. Depth**: Spending too much time on Estimations can leave no time for the High-Level design. Use estimations only to justify your database choice.
*   **Ideal vs. Real**: Design for the "100 million user" scale requested, but mention how the system would look on Day 1 (Monolith) vs. Day 1000 (Microservices).
*   **Complexity vs. Reliability**: Every box you add (Kafka, Redis, K8s) adds complexity. Justify every piece of the puzzle.

### 5. When NOT to Use
*   **Coding/LLD Interviews**: Do not use this framework for "Design a Parking Lot." That requires Class Diagrams and SOLID, not Load Balancers and Partitions.
*   **Deep Research Topics**: If the interviewer wants to focus *only* on the Database, skip the HLD and go straight to Sharding/Indexes.

### 6. Interview Focus
*   **Driving the Session**: "Don't wait for permission. Say: 'I'll start by clarifying requirements, then I'll move to API design. Does that sound good?'"
*   **Handling Constraints**: "What if the storage is limited? How would you change your schema?"
*   **Failure Thinking**: "What happens if this database node goes down right now?"

### 7. Common Mistakes
*   **Silence**: Thinking in your head for 2 minutes. Always think out loud.
*   **Jumping to the DB immediately**: Choosing "Couchbase" or "Kafka" before you even know what the functional requirements are.
*   **Ignoring Non-Functional Requirements**: Designing a functional system that can only handle 10 users per second when the prompt asked for 10 million.
