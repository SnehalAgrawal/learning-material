# Behavioral Design Patterns

### 1. Overview

Behavioral patterns focus on **how objects communicate and interact**. They describe how responsibilities are assigned between objects and how they collaborate to achieve a task.

For senior roles, these patterns are about:
*   **Decoupling Sender/Receiver** → The object asking for something shouldn't need to know who handles it.
*   **Extending Logic at Runtime** → Swapping algorithms or behaviors without changing the host class.

👉 Think of Behavioral Patterns as *“The Team Meeting”* — they define the rules of engagement and how information flows between different players.

### 2. Key Concepts

#### 2.1. **Observer**

Defines a subscription mechanism to notify multiple objects about any events that happen to the object they’re observing.

💡 **Bad Example (Tight coupling)**

```javascript
class Store {
  saleStarted() {
    new EmailService().send();
    new SMSService().send();
    // Every new service requires modifying Store
  }
}
```

```python
class Store:
    def sale_started(self):
        EmailService().send()
        SMSService().send()
        # Every new service requires modifying Store
```

❌ Problem: The `Store` is tightly coupled to specific notification services.

✅ **Better (Observer)**

```javascript
class Store {
  constructor() { this.observers = []; }
  subscribe(obs) { this.observers.push(obs); }
  notify() { this.observers.forEach(obs => obs.update()); }
}
```

```python
class Store:
    def __init__(self):
        self.observers = []
    def subscribe(self, obs):
        self.observers.append(obs)
    def notify(self):
        for obs in self.observers:
            obs.update()
```

#### 2.2. **Strategy**

Defines a family of algorithms, puts each of them into a separate class, and makes their objects interchangeable.

💡 **Bad Example (Massive switch/if-else)**

```javascript
function getPrice(price, type) {
  if (type === "member") return price * 0.9;
  if (type === "guest") return price;
}
```

```python
def get_price(price, user_type):
    if user_type == "member":
        return price * 0.9
    elif user_type == "guest":
        return price
```

❌ Problem: Adding a new discount type violates OCP.

✅ **Better (Strategy)**

```javascript
class DiscountStrategy {
  apply(price) { /* abstract */ }
}

class MemberDiscount extends DiscountStrategy {
  apply(price) { return price * 0.9; }
}
```

```python
class DiscountStrategy:
    def apply(self, price): pass

class MemberDiscount(DiscountStrategy):
    def apply(self, price):
        return price * 0.9
```

#### 2.3. **State**

Lets an object alter its behavior when its internal state changes.

💡 **Bad Example (State logic leak)**

```javascript
class Order {
  cancel() {
    if (this.state === "shipped") throw Error("Too late!");
    if (this.state === "pending") this.state = "cancelled";
  }
}
```

```python
class Order:
    def cancel(self):
        if self.state == "shipped":
            raise Exception("Too late!")
        elif self.state == "pending":
            self.state = "cancelled"
```

❌ Problem: As states grow, the `cancel()` method (and others) becomes a mess of conditionals.

✅ **Better (State Pattern)**

```javascript
class ShippedState {
  cancel() { throw Error("Cannot cancel shipped order"); }
}

class PendingState {
  cancel(order) { order.setState(new CancelledState()); }
}
```

```python
class ShippedState:
    def cancel(self):
        raise Exception("Cannot cancel shipped order")

class PendingState:
    def cancel(self, order):
        order.set_state(CancelledState())
```

#### 2.4. **Command**

Turns a request into a stand-alone object. This lets you parameterize methods with different requests and support undo.

#### 2.5. **Chain of Responsibility**

Lets you pass requests along a chain of handlers. Each handler decides to process it or pass it to the next.

### 3. Real-World Usage

*   **Frontend State Management** → Redux/Vuex use the Observer pattern (Store -> UI).
*   **Payment Gateways** → Using Strategy to switch between Stripe, PayPal, and Crypto.
*   **Undo/Redo** → Using the Command pattern in text editors.
*   **Middleware** → Express.js or ASP.NET Core middleware is a classic **Chain of Responsibility**.
*   **Workflows** → Order processing (Pending -> Paid -> Shipped) using the **State** pattern.

### 4. Tradeoffs

*   **Observer Memory Leaks** → Forgetting to unsubscribe can lead to "Zombies" in memory.
*   **Command Overhead** → Creating a class for every tiny action can bloat the codebase.
*   **Strategy vs State** → Strategy is usually set once by the client; State transitions automatically based on logic.

### 5. When NOT to Use

*   **Chain of Responsibility** → If the chain is too long, debugging "who changed the request" becomes a nightmare.
*   **Strategy** → If you only have two options that never change, a simple `if` is better than three classes.

### 6. Interview Focus

*   **Eliminating Switch Statements** → "How do you refactor a 50-line `if/else` block that calculates shipping?" (Strategy).
*   **Event-Driven Systems** → "How do you decouple a `CheckoutService` from a `EmailService` and `AnalyticsService`?" (Observer).
*   **Workflow Design** → "Design a document approval system where actions change based on status." (State).

### 7. Common Mistakes

*   **Hardcoding the Chain** → Hardcoding the next link inside the handler instead of letting the client configure it.
*   **Observer Spaghetti** → Observers triggering other Observers, creating an untraceable tree of side effects.
*   **Using Strategy for Params** → Using Strategy for simple value changes instead of actual behavior changes.
