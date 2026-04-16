# Structural Design Patterns

### 1. Overview
Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient. They help build relationships between entities without causing tight coupling.

### 2. Key Concepts
*   **Adapter**: Allows objects with incompatible interfaces to collaborate (the "Wrapper").
*   **Decorator**: Lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors.
*   **Facade**: Provides a simplified interface to a complex library, framework, or any other complex set of classes.
*   **Proxy**: Provides a surrogate or placeholder for another object to control access to it (e.g., lazy loading, logging, caching).

### 3. Real-World Usage
*   **Third-Party Integration**: Using an Adapter to map a legacy SOAP API response into a modern REST internal format.
*   **Middleware/Streams**: Node.js streams or Express.js response wrappers are essentially Decorators (adding `res.json` or gzip compression to a standard response).
*   **BFF (Backend-for-Frontend)**: A Facade that aggregates 5 microservice calls into 1 API response for a mobile app.
*   **Virtual Proxies**: Hibernated entities or heavy images that only load when actually accessed in the code.

### 4. Tradeoffs
*   **Adapter vs. Facade**: Adapter changes an interface to match another; Facade simplifies an interface. Choosing the wrong one can lead to confusion about intent.
*   **Decorator Complexity**: Can result in a large number of small, nested objects which are difficult to debug and trace (the "Matryoshka doll" effect).
*   **Proxy Overhead**: Every method call through a Proxy adds a small layer of indirection, which can add up in high-frequency loops.

### 5. When NOT to Use
*   **Decorator**: If you only need to add behavior once, inheritance or a simple `if` block is often cleaner than building a Decorator architecture.
*   **Facade**: Don't build a Facade that masks *too much* complexity, preventing developers from using necessary advanced features of the underlying library.

### 6. Interview Focus
*   **Decorator vs. Inheritance**: "Why would you use a Decorator instead of subclasses to add logging or encryption features?"
*   **External Libs**: "How do you handle a library that your team uses which has a messy, poorly named API?" (Facade/Adapter).
*   **Security/Efficiency**: "How would you implement access control for a heavy Resource object without modifying the object's code?" (Proxy).

### 7. Common Mistakes
*   **Over-adapting**: Creating Adapters for things that are already consistent, adding unnecessary layers of mapping code.
*   **Leaking Abstractions**: A Proxy or Facade that forces the consumer to handle exceptions specific to the "hidden" inner classes.
*   **Confusing Proxy with Decorator**: Decorator adds *behavior*; Proxy manages *access* or provides a *placeholder*.
