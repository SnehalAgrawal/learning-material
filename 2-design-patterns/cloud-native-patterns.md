# Cloud-Native Design Patterns

### 1. Overview

Cloud-native patterns address the challenges of building and running scalable, resilient applications in modern distributed environments (like Kubernetes or AWS). 

For senior roles, these patterns are about:
*   **Decoupling Services** → Ensuring one service's failure doesn't crash the whole system.
*   **Data Consistency** → Managing transactions across multiple databases without distributed locks.

👉 Think of Cloud-Native Patterns as *“The Survival Guide”* — they help your system stay alive in a world where network partitions and service failures are the norm.

### 2. Key Concepts

#### 2.1. **Circuit Breaker**

Prevents a failing service from causing a cascading failure across the system.

💡 **Bad Example (Endless waiting/Retry storm)**

```javascript
async function callService() {
  try {
    return await fetch("http://flaky-service/api");
  } catch (e) {
    return callService(); // ❌ Infinite retry on failure
  }
}
```

```python
import requests

def call_service():
    try:
        return requests.get("http://flaky-service/api")
    except:
        return call_service() # ❌ Infinite retry on failure
```

❌ Problem: If the service is down, the caller will keep trying, exhausting threads and resources on both sides.

✅ **Better (Circuit Breaker)**

```javascript
class CircuitBreaker {
  constructor() { this.state = "CLOSED"; this.failures = 0; }
  async call(fn) {
    if (this.state === "OPEN") return "Fallback Data";
    try {
      return await fn();
    } catch (e) {
      this.failures++;
      if (this.failures > 5) this.state = "OPEN";
      throw e;
    }
  }
}
```

```python
class CircuitBreaker:
    def __init__(self):
        self.state = "CLOSED"
        self.failures = 0
    def call(self, func):
        if self.state == "OPEN": return "Fallback Data"
        try:
            return func()
        except:
            self.failures += 1
            if self.failures > 5: self.state = "OPEN"
            raise
```

#### 2.2. **Outbox Pattern**

Ensures that a database update and a message publication happen atomically.

💡 **Bad Example (The "Dual Write" Problem)**

```javascript
async function updateOrder(order) {
  await db.save(order);
  await kafka.send("order_updated", order); // ❌ What if this fails?
}
```

```python
def update_order(order):
    db.save(order)
    kafka.send("order_updated", order) # ❌ DB saved but event might be lost
```

❌ Problem: If the message broker is down, the DB is updated but the rest of the system never knows.

✅ **Better (Outbox Pattern)**

```javascript
async function updateOrder(order) {
  const trx = await db.transaction();
  await trx.save(order);
  await trx.saveToOutboxTable({ event: "order_updated", data: order });
  await trx.commit(); // ✅ Both succeed or both fail
}
```

```python
def update_order(order):
    with db.transaction() as trx:
        trx.save(order)
        trx.save_to_outbox({"event": "order_updated", "data": order})
```

#### 2.3. **Saga Pattern**

Manages distributed transactions by using a sequence of local transactions with "compensating actions" for failures.

#### 2.4. **Bulkhead**

Isolates elements of an application into pools so that if one fails, others continue to function. (e.g., separate thread pools for critical vs non-critical APIs).

### 3. Real-World Usage

*   **E-commerce Checkout** → Saga pattern for `Inventory -> Payment -> Shipping`.
*   **Reliable Events** → Outbox pattern to ensure every "UserCreated" event actually reaches Kafka.
*   **Resilience** → Using libraries like `Resilience4j` (Java), `Polly` (.NET), or `Opossum` (Node.js) for Circuit Breakers.
*   **Resource Isolation** → Using separate Docker containers or Kubernetes namespaces to prevent resource exhaustion.

### 4. Tradeoffs

*   **Saga Complexity** → Hard to debug and trace; introduces eventual consistency.
*   **Outbox Latency** → Adds a small delay between DB write and message availability.
*   **Circuit Breaker Fallbacks** → Designing meaningful fallbacks (e.g., cached data) requires extra effort.

### 5. When NOT to Use

*   **Saga** → If you can use a single database with ACID transactions (Modular Monolith), do not use Sagas.
*   **Outbox** → If message loss is acceptable (e.g., logging/analytics), the overhead might not be worth it.

### 6. Interview Focus

*   **Distributed Transactions** → "How do you maintain consistency between two services that own their own databases?" (Saga).
*   **The Dual Write Problem** → "What happens if your DB commit succeeds but your message send fails?" (Outbox).
*   **Failure Propagation** → "How do you prevent a slow downstream service from crashing your entire system?" (Circuit Breaker/Bulkhead).

### 7. Common Mistakes

*   **Missing Compensating Transactions** → Forgetting to "undo" a step in a Saga when a later step fails.
*   **Circuit Breaker Flapping** → Setting thresholds too low, causing the breaker to open/close too frequently.
*   **Polling the Outbox** → Not using Change Data Capture (CDC) like Debezium, leading to high DB load from polling.
