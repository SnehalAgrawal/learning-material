# Behavioral Design Patterns

### 1. Overview
Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects. They describe not just patterns of objects or classes but also the patterns of communication between them.

### 2. Key Concepts
*   **Observer**: Defines a subscription mechanism to notify multiple objects about any events that happen to the object they’re observing.
*   **Strategy**: Defines a family of algorithms, puts each of them into a separate class, and makes their objects interchangeable.
*   **Command**: Turns a request into a stand-alone object that contains all information about the request. This lets you pass requests as a method arguments, delay or queue a request’s execution, and support undoable operations.
*   **Chain of Responsibility**: Lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain.
*   **State**: Lets an object alter its behavior when its internal state changes. It appears as if the object changed its class.

### 3. Real-World Usage
*   **Frontend State Management**: Redux or Vuex use the Observer pattern (Store -> Components) to update the UI.
*   **Payment Gateways**: Using Strategy to switch between `StripePayment`, `PayPalPayment`, and `CryptoPayment` at runtime based on user choice.
*   **Undo/Redo**: Using the Command pattern in text editors or graphic design tools (Figma/Photoshop).
*   **Request Middleware**: Authentication -> Validation -> Rate Limiting is a classic **Chain of Responsibility**. In modern backends, this is often handled by an [API Gateway](../6-backend-engineering/gateways-security.md).
*   **Order Workflows**: A `Order` object transitioning from `Pending` -> `Paid` -> `Shipped` -> `Delivered` using the **State** pattern to change behavior (e.g., `cancel()` only works in `Pending`).

### 4. Tradeoffs
*   **Strategy vs. State**: Strategy is a one-time choice for an algorithm; State is a dynamic transition between behaviors.
*   **Observer Memory Leaks**: Subscriptions that aren't properly cleaned up can lead to memory leaks (Zombies).
*   **Command Overhead**: Requires a lot of "Command" classes for every simple action, which can bloat the codebase.

### 5. When NOT to Use
*   **Chain of Responsibility**: If the chain is very long, it can be hard to debug which handler touched the request last.
*   **Strategy**: If you only have two algorithms that rarely change, a simple `switch` or `if/else` is easier to maintain than three classes.

### 6. Interview Focus
*   **Refactoring Conditionals**: "How would you eliminate a massive `switch` statement that grows every time we add a new marketing discount type?" (Strategy).
*   **Event-Driven Communication**: "How do you decouple a `UserService` from a `EmailService` so that user registration can trigger emails, analytics, and rewards without the UserService knowing about them?" (Observer).
*   **Lifecycle Management**: "Design a document approval workflow where different users can 'Approve', 'Reject', or 'Request Changes' based on the document's current status." (State).

### 7. Common Mistakes
*   **Hardcoding the Chain**: In Chain of Responsibility, hardcoding the next link inside the handler instead of letting the client configure the chain.
*   **Using Strategy for Everything**: Using Strategy for simple parameter changes instead of behavioral changes.
*   **Observer Spaghetti**: Having Observers that trigger other Observers, creating an untraceable tree of side effects.
