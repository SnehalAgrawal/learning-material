# Core Object-Oriented Programming (OOP)

### 1. Overview
Object-Oriented Programming is a paradigm centered around "objects" rather than "actions." For senior engineers, OOP is not just about syntax (classes/methods) but about managing complexity through encapsulation, modularity, and establishing clear boundaries between system components.

### 2. Key Concepts
*   **Encapsulation**: Bundling data and methods that operate on that data into a single unit (class), while restricting direct access to some components to prevent accidental state mutation.
```python
# Good Encapsulation
class BankAccount:
    def __init__(self, balance):
        self._balance = balance  # Protected
    
    def deposit(self, amount):
        if amount > 0:
            self._balance += amount
    
    def get_balance(self):
        return self._balance

# Bad Encapsulation
class BadAccount:
    def __init__(self, balance):
        self.balance = balance  # Public

account = BadAccount(100)
account.balance = -50  # Anyone can do this!
```
```javascript
// Good Encapsulation
class BankAccount {
    #balance; // Private field
    
    constructor(balance) {
        this.#balance = balance;
    }
    
    deposit(amount) {
        if (amount > 0) {
            this.#balance += amount;
        }
    }
    
    getBalance() {
        return this.#balance;
    }
}
```
*   **Abstraction**: Hiding complex implementation details and showing only the necessary features of an object. It reduces cognitive load by allowing engineers to interact with higher-level interfaces.
```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount):
        pass

class StripeProcessor(PaymentProcessor):
    def process_payment(self, amount):
        print(f"Processing ${amount} via Stripe")
```
```javascript
// JavaScript does NOT have true abstract classes or interfaces (built-in).
// Everything you’ve seen so far is a pattern, not a language feature.
// Interface-like Abstraction
class PaymentProcessor {
    constructor() {
        if (this.executePayment === PaymentProcessor.prototype.executePayment) {
            throw new Error("executePayment must be overridden");
        }
    }

    processPayment(amount) {
        this.validate(amount);       // common
        this.logStart(amount);       // common
        
        this.executePayment(amount); // 🔁 child-specific
        
        this.logSuccess(amount);     // common
    }

    validate(amount) {
        if (amount <= 0) {
            throw new Error("Invalid amount");
        }
    }

    logStart(amount) {
        console.log("Starting payment...");
    }

    logSuccess(amount) {
        console.log("Payment successful!");
    }

    executePayment(amount) {
        throw new Error("Must implement");
    }
}

class StripeProcessor extends PaymentProcessor {
    executePayment(amount) {
        console.log(`Processing ₹${amount} via Stripe`);
    }
}

class RazorpayProcessor extends PaymentProcessor {
    executePayment(amount) {
        console.log(`Processing ₹${amount} via Razorpay`);
    }
}

function getProcessor(type) {
    switch (type) {
        case "stripe": return new StripeProcessor();
        case "razorpay": return new RazorpayProcessor();
        default: throw new Error("Unsupported payment method");
    }
}

function checkout(processor, amount) {
    processor.processPayment(amount);
}

const userChoice = "razorpay";
const processor = getProcessor(userChoice);

checkout(processor, 1200);
// JavaScript looks for executePayment in this order:
// On the object itself (child class)
// Then up the prototype chain (parent class)
```
**Interface v/s Abstract class**
- Interface → defines only what to do (no implementation)
- Abstract class → defines what + some how (partial implementation)

*   **Inheritance**: A mechanism where a new class derives properties and behaviors from an existing class.
```python
class BaseService:
    def __init__(self, db_connection):
        self.db = db_connection

class UserService(BaseService):
    def get_user(self, user_id):
        return self.db.find("users", user_id)
```
```javascript
// JavaScript does NOT have true abstract classes or interfaces (built-in).
// Everything you’ve seen so far is a pattern, not a language feature.
class BaseService {
    constructor(dbConnection) {
        this.db = dbConnection;
    }
}

class UserService extends BaseService {
    getUser(userId) {
        return this.db.find("users", userId);
    }
}
```

*   **Polymorphism**: The ability of different types to be treated as a common base type, often achieved through method overriding or interfaces.
```python
class Logger(ABC):
    @abstractmethod
    def log(self, message): pass

class CloudLogger(Logger):
    def log(self, message): print(f"Cloud: {message}")

class LocalLogger(Logger):
    def log(self, message): print(f"Local: {message}")

def notify_admin(logger: Logger, msg):
    logger.log(msg) # Polymorphic call
```
```javascript
class Logger {
    log(message) { throw new Error("Method not implemented"); }
}

class CloudLogger extends Logger {
    log(message) { console.log(`Cloud: ${message}`); }
}

class LocalLogger extends Logger {
    log(message) { console.log(`Local: ${message}`); }
}

function notifyAdmin(logger, msg) {
    logger.log(msg); // Polymorphic call
}
```

*   **Composition**: Building complex objects by combining simpler objects (the "has-a" relationship), which is often more flexible than inheritance.
```python
class Validator:
    def validate(self, data): return True

class Repository:
    def save(self, data): print("Saved to DB")

class CreateUserUseCase:
    def __init__(self, validator, repository):
        self.validator = validator # Composition
        self.repository = repository
    
    def execute(self, data):
        if self.validator.validate(data):
            self.repository.save(data)
```
```javascript
class Validator {
    validate(data) { return true; }
}

class Repository {
    save(data) { console.log("Saved to DB"); }
}

class CreateUserUseCase {
    constructor(validator, repository) {
        this.validator = validator; // Composition
        this.repository = repository;
    }

    execute(data) {
        if (this.validator.validate(data)) {
            this.repository.save(data);
        }
    }
}
```


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
