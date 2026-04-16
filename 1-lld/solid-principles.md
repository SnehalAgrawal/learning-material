# SOLID Principles

### 1. Overview
SOLID is a set of five design principles intended to make software designs more understandable, flexible, and maintainable. For senior roles, SOLID is about reducing the "Cost of Change" and preventing "Rigidity" (code that is hard to change) and "Fragility" (code that breaks in many places when changed).

### 2. Key Concepts
*   **Single Responsibility (SRP)**: A class should have one, and only one, reason to change. It's about high cohesion.
*   **Open/Closed (OCP)**: Software entities should be open for extension but closed for modification. Use interfaces or abstract classes to add behavior without touching existing code.
*   **Liskov Substitution (LSP)**: Subtypes must be substitutable for their base types without altering the correctness of the program.
*   **Interface Segregation (ISP)**: Clients should not be forced to depend on methods they do not use. Prefer many small, specific interfaces over one large, general one.
*   **Dependency Inversion (DIP)**: High-level modules should not depend on low-level modules; both should depend on abstractions.

### 3. Real-World Usage
*   **Plugin Architectures**: OCP is the foundation of IDE plugins (like VS Code). You extend the editor's functionality without modifying the core editor source code.
*   **Abstracting Third-Party Services**: Using DIP to depend on an `INotificationService` interface instead of a concrete `TwilioSmsProvider`, allowing seamless switching to AWS SNS or SendGrid.
*   **Unit Testing**: ISP allows you to mock only the specific behaviors required for a test case, rather than implementing a massive interface.

### 4. Tradeoffs
*   **Over-Engineering**: Strictly following SOLID 100% of the time can lead to "Interface-itis," where simple tasks require navigating 10 files.
*   **Boilerplate**: SRP often leads to more files and classes, which can increase the overhead of the build system and initial setup.
*   **Tracing Difficulty**: Dependency Inversion can make it harder to find the entry point or current implementation during a debugging session without an IDE.

### 5. When NOT to Use
*   **Prototyping/MVPs**: When speed is the only metric and the code is expected to be thrown away or rewritten.
*   **Hyper-Performance Systems**: In extremely low-latency code (e.g., high-frequency trading), the extra layers of abstraction and indirection can introduce unacceptable cache misses.

### 6. Interview Focus
*   **Identifying Violations**: Interviewers often show a code snippet and ask, "Which SOLID principle is being violated here?"
*   **Practical Application**: "Refactor this `ReportGenerator` that currently handles DB fetching, formatting, and emailing."
*   **DIP vs. DI**: "Explain the difference between Dependency Inversion (the principle) and Dependency Injection (the technique)."

### 7. Common Mistakes
*   **Thinking SRP means "One Method"**: SRP is about "one reason to change" (e.g., business logic change), not just "one function."
*   **LSP Violations**: Throwing `NotImplementedException` in a subclass method because that subclass doesn't support a parent class behavior.
*   **OCP Abuse**: Trying to make *every* private method extensible "just in case," leading to speculative complexity.
