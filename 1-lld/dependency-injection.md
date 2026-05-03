# Dependency Injection (DI)

### 1. Overview
Dependency Injection is a design pattern in which an object receives other objects that it depends on. It is the primary way to achieve **Dependency Inversion**, allowing objects to be decoupled and making systems more testable and modular.

#### 1.1 Without Dependency Injection
The class is responsible for creating its own dependencies, leading to tight coupling.

```python
# Python
class Engine:
    def start(self):
        print("Engine started")

class Car:
    def __init__(self):
        self.engine = Engine() # Tightly coupled: Car "knows" how to create Engine

    def drive(self):
        self.engine.start()
```

```javascript
// JavaScript
class Engine {
  start() {
    console.log("Engine started");
  }
}

class Car {
  constructor() {
    this.engine = new Engine(); // Tightly coupled
  }

  drive() {
    this.engine.start();
  }
}
```

#### 1.2 With Dependency Injection
Dependencies are "injected" from the outside, usually through the constructor.

```python
# Python
class Car:
    def __init__(self, engine):
        self.engine = engine # Dependency is provided, not created

    def drive(self):
        self.engine.start()

# We can now swap implementations easily
class ElectricEngine:
    def start(self):
        print("Electric engine started")

engine = ElectricEngine()
my_car = Car(engine)
my_car.drive()
```

```javascript
// JavaScript
class Car {
  constructor(engine) {
    this.engine = engine; // Injected dependency
  }

  drive() {
    this.engine.start();
  }
}

// Swapping implementations
class ElectricEngine {
  start() {
    console.log("Electric engine started");
  }
}

const engine = new ElectricEngine();
const myCar = new Car(engine);
myCar.drive();
```

### Framework-Level DI
Most modern frameworks provide a "Container" to automate this.

```javascript
// JavaScript/TypeScript (NestJS Example)
@Injectable()
class Engine {}

@Injectable()
class Car {
  constructor(private engine: Engine) {} // Auto-injected by framework
}
```

```python
# Python (FastAPI Example)
from fastapi import Depends

def get_engine():
    return Engine()

@app.get("/drive")
def drive(engine: Engine = Depends(get_engine)):
    return engine.start()
```

#### Types of Dependency Injection
* Constructor Injection – dependencies passed via constructor
* Setter Injection – dependencies set via setter methods
* Field Injection – dependencies assigned directly to fields (common in frameworks like Spring)

#### Why use DI?
* Loose coupling → components are independent
* Easier testing → you can inject mocks or stubs
* Better maintainability → easier to modify or extend
* Flexibility → swap implementations without changing code

#### Real-world analogy

Think of a phone charger:
* Without DI: the phone builds its own charger
* With DI: you plug in any compatible charger

#### Common frameworks using DI
* Java: Spring, Guice
* .NET: Built-in DI container
* JavaScript/TypeScript: Angular, NestJS

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
    
    Inversion of Control (IoC) is a principle where the control of object creation and program flow is shifted from your code to an external system (like a framework).
    
    Dependency Injection (DI) is a design pattern used to implement IoC, where dependencies are provided to a class from the outside instead of being created inside it.
    
    👉 In one line: DI is a way to achieve IoC.
*   **Lifetimes**: "Explain the difference between Singleton and Scoped dependencies in a web request context."
    
    Singleton and Scoped define how long a dependency instance lives in a web app.

    Singleton [global (app-wide)]:
    One instance for the entire application lifetime. Every request and user shares the same object. Example: database connection pool, logging service

    Scoped [per request]:
    One instance per web request. Each request gets its own instance, but within that request it’s shared. Example: user session data, request-specific context
*   **Circular Dependencies**: "What happens when Class A needs Class B, and Class B needs Class A? How do you solve it?"
    
    A circular dependency occurs when two classes depend on each other, causing resolution issues in DI. It’s usually fixed by refactoring to remove the cycle, often by introducing a third abstraction or using lazy/setter injection.

    ```
    1. Break the cycle by introducing a third class: Move shared logic into C.
    A → C ← B

    2. Use an interface / abstraction: Depend on abstractions instead of concrete classes.

    3. Lazy injection
    class A {
        constructor(getB) {
            this.getB = getB;
        }
    }

    4. Setter injection (instead of constructor)
    a = A()
    b = B()
    a.set_b(b)
    b.set_a(a)

    5. Use events or callbacks: Instead of direct dependency, communicate via events.
    ```

### 7. Common Mistakes
*   **Service Locator Anti-pattern**: Passing the DI container itself into a class so the class can "resolve" its own dependencies. This hides dependencies.
*   **Constructor Over-injection**: A constructor having 10+ arguments is a code smell that the class has too many responsibilities (SRP violation).
*   **Manual Wire-up in Large Apps**: Trying to manually inject 100+ dependencies without using a container, leading to a massive, unmaintainable main file.
