# Dependency Injection (DI)

### 1. Overview
Dependency Injection is a design pattern in which an object receives other objects that it depends on. It is the primary way to achieve **Dependency Inversion**, allowing objects to be decoupled and making systems more testable and modular.

### 2. Key Concepts
*   **The Dependency**: An object that another object needs to function (e.g., a Database Service).
*   **Injection**: The act of passing the dependency to a dependent object (the client) rather than letting the client create it.
*   **Constructor Injection**: Passing dependencies through the class constructor (preferred for required dependencies).
*   **Property/Setter Injection**: Assigning dependencies through public properties or setter methods (useful for optional dependencies).
*   **Inversion of Control (IoC) Container**: A framework that manages the creation and lifecycle (scoping) of dependencies.

### 3. Real-World Usage
*   **Unit Testing**: Injecting a `MockPaymentGateway` into an `OrderService` instead of the real one, allowing you to test the service without making HTTP calls.
*   **Environment-Specific Config**: Injecting an `S3Storage` provider in production and a `LocalStorage` provider in development.
*   **Frameworks**: Angular, Spring Boot, and .NET Core use built-in DI containers as their core architectural foundation.

### 4. Tradeoffs
*   **Setup Complexity**: Requires a "Composition Root" where all dependencies are wired together.
*   **Indirection**: It can sometimes be difficult to see exactly which implementation of an interface is being used at runtime without checking the container configuration.
*   **Learning Curve**: Modern DI libraries have complex concepts like Scoped, Singleton, and Transient lifetimes that can cause bugs if misunderstood.

### 5. When NOT to Use
*   **Small Libraries**: Adding a DI container dependency to a small utility library can be unnecessary overhead for consumers.
*   **Value Objects**: Objects like `Money`, `DateRange`, or `UserDTO` should not be "injected"; they should be created directly as they don't have complex behaviors or side effects.

### 6. Interview Focus
*   **Conceptual Difference**: "What is the difference between IoC and DI?"
*   **Lifetimes**: "Explain the difference between Singleton and Scoped dependencies in a web request context."
*   **Circular Dependencies**: "What happens when Class A needs Class B, and Class B needs Class A? How do you solve it?"

### 7. Common Mistakes
*   **Service Locator Anti-pattern**: Passing the DI container itself into a class so the class can "resolve" its own dependencies. This hides dependencies.
*   **Constructor Over-injection**: A constructor having 10+ arguments is a code smell that the class has too many responsibilities (SRP violation).
*   **Manual Wire-up in Large Apps**: Trying to manually inject 100+ dependencies without using a container, leading to a massive, unmaintainable main file.
