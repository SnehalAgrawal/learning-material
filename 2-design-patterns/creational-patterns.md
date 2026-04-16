# Creational Design Patterns

### 1. Overview
Creational patterns abstract the instantiation process. They help make a system independent of how its objects are created, composed, and represented. For senior roles, the choice depends on where you want the "creation logic" to live and how much flexibility you need at runtime.

### 2. Key Concepts
*   **Singleton**: Ensures a class has only one instance and provides a global point of access.
*   **Factory Method**: Defines an interface for creating an object, but lets subclasses decide which class to instantiate.
*   **Abstract Factory**: Provides an interface for creating families of related or dependent objects without specifying their concrete classes.
*   **Builder**: Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.
*   **Prototype**: Creates new objects by copying an existing object (cloning).

### 3. Real-World Usage
*   **Database Connections**: The Singleton pattern is often used for connection pools or configuration managers.
*   **UI Frameworks**: Abstract Factory is used to create "Themed" components (e.g., a `MaterialUIFactory` vs. an `iOSUIFactory`).
*   **Form Builders / SQL Query Builders**: The Builder pattern is used to construct complex queries or nested JSON objects incrementally.
*   **Logging Libraries**: The Factory pattern allows a logger to instantiate different "sinks" (File, Console, CloudWatch) based on environment vars.

### 4. Tradeoffs
*   **Singleton Complexity**: Global state is hard to test and can lead to hidden dependencies and race conditions in multi-threaded environments.
*   **Factory Boilerplate**: Adding a new product type requires updating the factory or creating new factory subclasses.
*   **Builder vs. Constructor**: Builders are more readable for objects with 5+ parameters, but they require writing an extra class and increase total lines of code.

### 5. When NOT to Use
*   **Singleton**: Avoid using Sinton for state that *should* be isolated for testing or multiple concurrent sessions. Dependency Injection is almost always a better alternative.
*   **Factory**: Don't use a Factory for simple `new User()` calls where no logic or abstraction is needed. Over-using Factories leads to "YAGNI" (You Ain't Gonna Need It) complexity.

### 6. Interview Focus
*   **Singleton vs. Dependency Injection**: "If Singleton is a pattern, why is it often called an anti-pattern today? How does DI solve it?"
*   **Complex Object Construction**: "How would you design a system to build a multi-step Checkout process with optional steps?" (Hint: Builder).
*   **Abstract Factory vs. Factory Method**: "Explain when you would choose one over the other."

### 7. Common Mistakes
*   **Non-Thread-Safe Singletons**: Implementing a Singleton in Java/C# without handling multi-thread access (e.g., double-checked locking).
*   **The "One Giant Factory"**: Creating a single factory that knows about every class in the system, violating SRP.
*   **Cloning without Deep Copy**: In the Prototype pattern, performing a "shallow copy" of an object that contains nested references, causing shared state bugs.
