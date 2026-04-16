# Core Object-Oriented Programming (OOP)

### 1. Overview
Object-Oriented Programming is a paradigm centered around "objects" rather than "actions." For senior engineers, OOP is not just about syntax (classes/methods) but about managing complexity through encapsulation, modularity, and establishing clear boundaries between system components.

### 2. Key Concepts
*   **Encapsulation**: Bundling data and methods that operate on that data into a single unit (class), while restricting direct access to some components to prevent accidental state mutation.
*   **Abstraction**: Hiding complex implementation details and showing only the necessary features of an object. It reduces cognitive load by allowing engineers to interact with higher-level interfaces.
*   **Inheritance**: A mechanism where a new class derives properties and behaviors from an existing class.
*   **Polymorphism**: The ability of different types to be treated as a common base type, often achieved through method overriding or interfaces.
*   **Composition**: Building complex objects by combining simpler objects (the "has-a" relationship), which is often more flexible than inheritance.

### 3. Real-World Usage
*   **Middleware Chains**: In Express.js or ASP.NET Core, polymorphism is used to treat every middleware as a standard interface `(req, res, next)`.
*   **SDK Development**: Abstraction is used to provide a simple `client.upload()` method that hides the complexity of multipart uploads, retries, and checksums.
*   **Database ORMs**: Classes representing tables (Encapsulation) and using Inheritance to handle common fields like `id`, `created_at`, and `updated_at`.

### 4. Tradeoffs
*   **Inheritance vs. Composition**: Inheritance creates a tight coupling between parent and child (fragile base class problem). Composition allows for runtime flexibility and easier testing through dependency injection.
*   **Complexity**: Over-abstracting can lead to "Boilerplate Hell," making it difficult for new engineers to trace the execution flow.
*   **Performance**: Abstraction and polymorphism (virtual method tables) add a tiny runtime overhead, though usually negligible compared to I/O costs.

### 5. When NOT to Use
*   **Functional-First Environments**: In high-concurrency systems (like Elixir) or data-pipeline heavy apps, OOP state mutation can lead to race conditions. Prefer immutable data structures.
*   **Simple Scripts**: For small utility scripts, the overhead of defining classes and relationships is overkill.

### 6. Interview Focus
*   **Deep Understanding**: "Explain why you would prefer Composition over Inheritance in a payment gateway system."
*   **Problem Patterns**: "How would you design a bird simulation where some birds can fly and others can't without using deep inheritance?"
*   **Practical**: "What is the difference between an Abstract Class and an Interface in [Language]?"

### 7. Common Mistakes
*   **Deep Inheritance Hierarchies**: Creating 5+ levels of inheritance, making the system impossible to refactor.
*   **Violating Encapsulation**: Making all fields `public` or providing "God Getters/Setters" that expose internal state.
*   **Premature Abstraction**: Creating interfaces for things that will only ever have one implementation.
