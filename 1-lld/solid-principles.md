# SOLID Principles

### 1. Overview

SOLID is a set of five design principles intended to make software designs more understandable, flexible, and maintainable. For senior roles, SOLID is about reducing the **"Cost of Change"** and preventing:

* **Rigidity** → small change = large refactor
* **Fragility** → one change breaks unrelated parts

👉 Think of SOLID as *“designing code that survives change without drama.”*


### 2. Key Concepts


#### 2.1. **Single Responsibility (SRP)**

  A class should have one, and only one, reason to change (one responsibility = one axis of change).

  💡 **Bad Example (doing too much)**

  ```javascript
  class UserService {
    saveUser(user) {
      // DB logic
    }

    sendEmail(user) {
      // Email logic
    }
  }
  ```

  ```python
  class UserService:
      def save_user(self, user):
          pass  # DB logic

      def send_email(self, user):
          pass  # Email logic
  ```

  ❌ Problem: DB + Email changes affect same class

  ✅ **Better (split responsibilities)**

  ```javascript
  class UserRepository {
    save(user) {}
  }

  class EmailService {
    send(user) {}
  }
  ```

  ```python
  class UserRepository:
      def save(self, user):
          pass

  class EmailService:
      def send(self, user):
          pass
  ```


#### 2.2. **Open/Closed (OCP)**

Open for extension, closed for modification. Use interfaces or abstract classes to add behavior without touching existing code.

💡 Add new behavior without changing existing code.

❌ **Bad Example**

```javascript
function getDiscount(type) {
  if (type === "regular") return 10;
  if (type === "premium") return 20;
}
```

```python
def get_discount(user_type):
    if user_type == "regular":
        return 10
    elif user_type == "premium":
        return 20
```

❌ Every new type = modify function

✅ **Better (extend via polymorphism)**

```javascript
class Discount {
  get() {}
}

class RegularDiscount extends Discount {
  get() { return 10; }
}

class PremiumDiscount extends Discount {
  get() { return 20; }
}
```

```python
class Discount:
    def get(self):
        pass

class RegularDiscount(Discount):
    def get(self):
        return 10

class PremiumDiscount(Discount):
    def get(self):
        return 20
```


#### 2.3. **Liskov Substitution (LSP)**

Subtypes should behave like their parent without breaking expectations.

💡 “If it looks like a duck, it should behave like a duck.”

❌ **Bad Example**

```javascript
class Bird {
  fly() {}
}

class Penguin extends Bird {
  fly() {
    throw new Error("Can't fly");
  }
}
```

```python
class Bird:
    def fly(self):
        pass

class Penguin(Bird):
    def fly(self):
        raise Exception("Can't fly")
```

❌ Violates expectation

✅ **Better (correct abstraction)**

```javascript
class Bird {}

class FlyingBird extends Bird {
  fly() {}
}

class Penguin extends Bird {}
```

```python
class Bird:
    pass

class FlyingBird(Bird):
    def fly(self):
        pass

class Penguin(Bird):
    pass
```


#### 2.4. **Interface Segregation (ISP)**

Don’t force classes to implement unused methods. Clients should not be forced to depend on methods they do not use. Prefer many small, specific interfaces over one large, general one.

❌ **Bad Example**

```javascript
class Worker {
  work() {}
  eat() {}
}
```

```python
class Worker:
    def work(self): pass
    def eat(self): pass
```

❌ What about robots?

✅ **Better (split interfaces)**

```javascript
class Workable {
  work() {}
}

class Eatable {
  eat() {}
}
```

```python
class Workable:
    def work(self): pass

class Eatable:
    def eat(self): pass
```


#### 2.5. **Dependency Inversion (DIP)**

Depend on abstractions, not concrete implementations. High-level modules should not depend on low-level modules; both should depend on abstractions.

💡 High-level logic shouldn’t care about low-level details.

❌ **Bad Example**

```javascript
class EmailService {
  send() {}
}

class Notification {
  constructor() {
    this.service = new EmailService();
  }
}
```

```python
class EmailService:
    def send(self): pass

class Notification:
    def __init__(self):
        self.service = EmailService()
```

❌ Tight coupling

✅ **Better (inject dependency)**

```javascript
class Notification {
  constructor(service) {
    this.service = service;
  }
}
```

```python
class Notification:
    def __init__(self, service):
        self.service = service
```

### 3. Real-World Usage

* **Plugin Architectures** → OCP (e.g., extending editors without modifying core)
* **Service Abstraction** → DIP (swap SMS/email providers easily)
* **Testing** → ISP + DIP (mock only what you need)
* **Microservices** → SRP at system level (each service = one responsibility)

### 4. Tradeoffs

* **Over-Engineering** → Too many abstractions for simple problems
* **Boilerplate** → More files, interfaces, setup
* **Debugging Complexity** → Harder to trace due to indirection

💡 Rule of thumb: *Apply SOLID progressively, not blindly.*

### 5. When NOT to Use

* **Prototyping / MVPs** → Speed > design
* **Short-lived scripts** → No long-term maintenance
* **Ultra low-latency systems** → Abstraction overhead matters

### 6. Interview Focus
*   **Identifying Violations**: Interviewers often show a code snippet and ask, "Which SOLID principle is being violated here?"

    - samples at last section
*   **DIP vs. DI**: "Explain the difference between Dependency Inversion (the principle) and Dependency Injection (the technique)."
    
    DIP is the rule, DI is the tool.
    * Dependency Inversion (DIP)
      - A design principle
      - Says: High-level modules should depend on abstractions, not concrete implementations
      - Focus: What to achieve (decoupling)

    * Dependency Injection (DI)
      - An implementation technique
      - Says: *Provide dependencies from outside instead of creating them inside*
      - Focus: How to achieve DIP


### 7. Common Mistakes
*   **Thinking SRP means "One Method"**: SRP is about "one reason to change" (e.g., business logic change), not just "one function."
*   **LSP Violations**: Throwing `NotImplementedException` in a subclass method because that subclass doesn't support a parent class behavior.
*   **OCP Abuse**: Trying to make *every* private method extensible "just in case," leading to speculative complexity.

---

### SOLID Violations – Identify in Interviews

#### 1. Single Responsibility Principle (SRP)

##### ❌ Example 1

```javascript
class ReportManager {
  fetchData() {
    // DB logic
  }

  generateReport() {
    // formatting logic
  }

  sendEmail() {
    // email logic
  }
}
```

🎯 **Violation**: SRP
💡 One class = multiple responsibilities (DB, formatting, communication)


##### ❌ Example 2

```javascript
class User {
  constructor(name) {
    this.name = name;
  }

  saveToDatabase() {
    // DB logic
  }
}
```

🎯 **Violation**: SRP
💡 Entity + persistence logic mixed together


#### 2. Open/Closed Principle (OCP)

##### ❌ Example 1

```javascript
function calculateSalary(employee) {
  if (employee.type === "fulltime") {
    return employee.salary;
  } else if (employee.type === "contract") {
    return employee.hourly * employee.hours;
  }
}
```

🎯 **Violation**: OCP
💡 Adding new employee type = modifying this function


##### ❌ Example 2

```javascript
class PaymentProcessor {
  process(type) {
    if (type === "card") {
      // card logic
    } else if (type === "paypal") {
      // paypal logic
    }
  }
}
```

##### Solution
```javascript
class Employee {
  calculateSalary() {}
}

class FullTimeEmployee extends Employee {
  constructor(salary) {
    super();
    this.salary = salary;
  }

  calculateSalary() {
    return this.salary;
  }
}

class ContractEmployee extends Employee {
  constructor(hourly, hours) {
    super();
    this.hourly = hourly;
    this.hours = hours;
  }

  calculateSalary() {
    return this.hourly * this.hours;
  }
}

const emp = new ContractEmployee(50, 20);
console.log(emp.calculateSalary());
```

🎯 **Violation**: OCP
💡 New payment method = change existing code


#### 3. Liskov Substitution Principle (LSP)

##### ❌ Example 1

```javascript
class Bird {
  fly() {
    console.log("Flying");
  }
}

class Ostrich extends Bird {
  fly() {
    throw new Error("Cannot fly");
  }
}
```

🎯 **Violation**: LSP
💡 Subclass breaks expected behavior


##### ❌ Example 2

```javascript
class Rectangle {
  setWidth(w) { this.width = w; }
  setHeight(h) { this.height = h; }
}

class Square extends Rectangle {
  setWidth(w) {
    this.width = this.height = w;
  }
}
```

🎯 **Violation**: LSP
💡 Square changes behavior → breaks assumptions of Rectangle


#### 4. Interface Segregation Principle (ISP)

##### ❌ Example 1

```javascript
class Machine {
  print() {}
  scan() {}
  fax() {}
}

class Printer extends Machine {
  fax() {
    throw new Error("Not supported");
  }
}
```

🎯 **Violation**: ISP
💡 Printer forced to implement unused methods


##### ❌ Example 2

```javascript
class Worker {
  work() {}
  eat() {}
}

class Robot extends Worker {
  eat() {
    throw new Error("Robots don't eat");
  }
}
```

🎯 **Violation**: ISP
💡 Robot forced to implement irrelevant behavior


#### 5. Dependency Inversion Principle (DIP)

##### ❌ Example 1

```javascript
class MySQLDatabase {
  connect() {}
}

class UserService {
  constructor() {
    this.db = new MySQLDatabase();
  }
}
```

🎯 **Violation**: DIP
💡 High-level module depends on concrete class


##### ❌ Example 2

```javascript
class EmailSender {
  send() {}
}

class Notification {
  sendNotification() {
    const email = new EmailSender();
    email.send();
  }
}
```

🎯 **Violation**: DIP
💡 Hard-coded dependency → no flexibility


### 🧠 Quick Recognition Tricks (Interview Gold)

* **SRP** → “Does this class have multiple reasons to change?”
* **OCP** → “Do I need to modify existing code to add new behavior?”
* **LSP** → “Can I replace parent with child safely?”
* **ISP** → “Is this class forced to implement unused stuff?”
* **DIP** → “Is this class tightly coupled to a concrete implementation?”
