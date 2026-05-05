## ✅ **SOLID Principles (Short Notes for Interviews)**
### **1. Single Responsibility Principle (SRP)**
**What:** A class/module should have one reason to change — one job.  
**Why:** Easier to test, debug, and maintain.  
**Advantage:** Clean separation of concerns.
```js
// ❌ Mixed responsibility
class UserManager {
  createUser() { /* ... */ }
  sendEmail() { /* ... */ }
}

// ✅ Split into single responsibilities
class UserService {
  createUser() { /* ... */ }
}
class EmailService {
  sendEmail() { /* ... */ }
}
```
---
### **2. Open/Closed Principle (OCP)**
**What:** Code should be open to extension, closed to modification.  
**Why:** Avoids breaking existing code when adding features.  
**Advantage:** More flexible and future-proof design.

```js
// ❌ Need to modify the original method to support new types
function getDiscount(user) {
  if (user.type === 'premium') return 0.2;
  return 0.1;
}

// ✅ Extend without changing core
class Discount {
  get(user) { return 0.1; }
}
class PremiumDiscount extends Discount {
  get(user) { return 0.2; }
}
```
---
### **3. Liskov Substitution Principle (LSP)**
**What:** Subclasses should be replaceable without breaking the app.  
**Why:** Guarantees expected behavior when using inheritance.  
**Advantage:** Reliable polymorphism.
```js
// ✅ Bird should work even if replaced by any subtype
class Bird {
  fly() {}
}
class Eagle extends Bird {
  fly() { console.log("Flying"); }
}
// ❌ This violates LSP — can't fly
class Penguin extends Bird {
  fly() { throw new Error("Penguins can't fly!"); }
}
```
✅ Instead, use composition or separate interfaces if behavior differs.

---
### **4. Interface Segregation Principle (ISP)**
**What:** Don’t force classes to implement unused methods.  
**Why:** Avoids bloated, confusing interfaces.  
**Advantage:** More focused and flexible design.
```js
// ❌ Bad: One interface for everything
class AllInOnePrinter {
  print() {}
  fax() {}
  scan() {}
}

// ✅ Split responsibilities
class Printer {
  print() {}
}
class Scanner {
  scan() {}
}
```
---
### **5. Dependency Inversion Principle (DIP)**
**What:** High-level modules should depend on abstractions, not low-level implementations.  
**Why:** Makes code decoupled and testable.  
**Advantage:** Easier to mock, swap, or update dependencies.
```js
// ❌ Direct dependency on concrete class
class MySQL {
  query(sql) {}
}
class UserService {
  constructor() {
    this.db = new MySQL(); // tightly coupled
  }
}

// ✅ Depend on abstraction (e.g., DB interface)
class UserService {
  constructor(db) {
    this.db = db; // can inject MySQL, Mongo, Mock
  }
}
```
---

### ✨ Summary Table
| Principle | What It Means                             | Why It Helps                |
| --------- | ----------------------------------------- | --------------------------- |
| SRP       | One job per class                         | Separation, easier change   |
| OCP       | Extend without changing code              | Safer updates, plug-n-play  |
| LSP       | Subtypes must behave like base type       | Reliable inheritance        |
| ISP       | Don't force unused methods                | Simpler, cleaner interfaces |
| DIP       | Depend on abstraction, not implementation | Decoupled, testable code    |

---


# ✅ **SOLID Principles (JS-Focused 1-Pager)**

### 🔹 **1. Single Responsibility Principle (SRP)**
**📌 One job per class/module — one reason to change.**
#### ✅ Why?
- Easier to test, debug, maintain
- Improves modularity and readability
#### 🧠 Real-World:
> In React, I split data-fetching into hooks and UI into components.
#### 🔧 Example:
```js
class Logger {
  log(msg) { console.log(msg); }
}

class OrderService {
  constructor(logger) { this.logger = logger; }
  placeOrder(order) {
    this.logger.log("Order placed");
  }
}
```
---
### 🔹 **2. Open/Closed Principle (OCP)**
**📌 Open to extension, closed to modification.**
#### ✅ Why?
- Add new features without breaking old code
- Supports plugin-like architectures
#### 🧠 Real-World:
> I used polymorphism for payment types instead of adding more `if-else` cases.
#### 🔧 Example:
```js
class Discount {
  calculate(price) { return price; }
}
class PremiumDiscount extends Discount {
  calculate(price) { return price * 0.8; }
}
```

---
### 🔹 **3. Liskov Substitution Principle (LSP)**
**📌 Subclasses should behave like the parent class.**
#### ✅ Why?
- Guarantees reliable inheritance
- Prevents runtime surprises
#### 🧠 Real-World:
> I avoid extending classes when child behavior diverges — I use composition instead.
#### 🔧 Example:
```js
class Bird {
  fly() { console.log('Flying'); }
}
class Penguin extends Bird {
  fly() { throw new Error("Can't fly"); } // ❌ violates LSP
}
```

✅ Fix: Use separate interfaces or behaviors.

---
### 🔹 **4. Interface Segregation Principle (ISP)**
**📌 Don’t force classes to implement unused methods.**
#### ✅ Why?
- Smaller, focused interfaces
- Reduces confusion and accidental dependencies
#### 🧠 Real-World:
> I split `PrinterService` into smaller modules like Print, Scan, Fax based on use case.
#### 🔧 Example:
```js
class Printer {
  print(doc) { /* ... */ }
}
class Scanner {
  scan(doc) { /* ... */ }
}
```

---
### 🔹 **5. Dependency Inversion Principle (DIP)**
**📌 High-level modules should depend on abstractions.**
#### ✅ Why?
- Decouples business logic from low-level details
- Enables mocking for unit testing
#### 🧠 Real-World:
> I inject repositories into services so I can switch between real and mock DBs.
#### 🔧 Example:
```js
// Abstraction
class Database {
  query(sql) {}
}

// Injected into service
class UserService {
  constructor(db) { this.db = db; }
}

```
---
### 🧠 Summary Table

|Principle|Key Idea|Real-World Benefit|
|---|---|---|
|**SRP**|One responsibility per class|Easier changes, better modularity|
|**OCP**|Extend without modifying code|Add features safely|
|**LSP**|Subtypes must behave as expected|Safe inheritance|
|**ISP**|Keep interfaces lean|Focused design, avoids confusion|
|**DIP**|Depend on abstractions, not concretes|Easier testing, better flexibility|