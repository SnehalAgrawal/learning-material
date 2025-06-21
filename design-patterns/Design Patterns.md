✅ Design Patterns in JavaScript

| Purpose                    | Covered Pattern                       |
| -------------------------- | ------------------------------------- |
| Component/module isolation | Module, Singleton                     |
| Object creation            | Factory, Function Factory             |
| Extending behavior         | Decorator, Proxy                      |
| Flexible logic             | Strategy                              |
| Event systems / pub-sub    | Observer                              |
| UI commands & undo         | Command                               |
| Dependency management      | Dependency Injection                  |
| Common red flags           | God Object, Spaghetti Code (anti-pat) |
|                            |                                       |
**Design Patterns Are Language-Agnostic**?
They are **general solutions to common software design problems**, described at a conceptual level. You can implement them in **any language** that supports basic object-oriented or functional principles — including Java, Python, C++, TypeScript, C#, Go, Ruby, etc.

✅ But Each Language Has Different Idioms
JavaScript makes some patterns:
- **easier or more concise** (due to closures, first-class functions)
- or **less necessary** (because the language is flexible and dynamic)
- ✅ Design patterns are **not JavaScript-specific**
- ✅ You can apply them in any modern language
- ✅ JavaScript gives you **flexibility** to implement them in creative or functional ways
### 🚀 Why You See These More in JS Interviews
- JS doesn’t have strict interfaces/classes, so **you apply patterns more dynamically**
- JS (especially with React, Node, etc.) **leans on patterns like Factory, Observer, Strategy, Decorator** because they match the async/event-driven nature of the web
- Frontend engineers often use patterns to manage **state, UI behavior, and modularity**

| Pattern                       | JS Style                              | Other Languages Style                        |
| ----------------------------- | ------------------------------------- | -------------------------------------------- |
| **Singleton**                 | Closure or class-based                | Static class instance in Java/C#             |
| **Factory**                   | Function returns object               | Factory classes in Java/C#                   |
| **Strategy**                  | Pass functions dynamically            | Interface-based strategy classes             |
| **Observer**                  | Event emitter pattern / callback list | EventBus / Pub-Sub in Java/C#                |
| **Decorator**                 | Higher-order functions                | Annotation-based or wrapper classes          |
| **Command**                   | Functions as commands                 | Command interface and concrete classes       |
| **DI (Dependency Injection)** | Manual injection, no native container | Spring (Java), NestJS (TS), .NET built-in DI |

### 1. **Module Pattern**
Encapsulate private data and expose public methods.  
**Use When**: You want to create reusable, self-contained modules with private state.
```js
const Counter = (() => {
  let count = 0;
  return {
    increment: () => ++count,
    reset: () => (count = 0)
  };
})();
```
---
### 2. **Singleton Pattern**
Ensure one instance exists across the application.  
**Use When**: You need a global shared resource (e.g., config, logging, DB connection).
```js
class Singleton {
  constructor() {
    if (Singleton.instance) return Singleton.instance;
    Singleton.instance = this;
  }
}
```
---
### 3. **Factory Pattern**
Encapsulate object creation logic.  
**Use When**: You want to abstract the creation of different types of similar objects.
```js
function createUser(name) {
  return { name, role: 'user' };
}
```
---
### 4. **Function Factory**
Returns a function, usually customized.  
**Use When**: You want to generate functions with pre-configured behavior.
```js
function multiplier(factor) {
  return num => num * factor;
}
```
---
### 5. **Observer Pattern**
Allows subscription to changes and notifies observers.  
**Use When**: You want one-to-many relationships like event systems or reactive state.
```js
class Observable {
  constructor() { this.subs = []; }
  subscribe(fn) { this.subs.push(fn); }
  notify(data) { this.subs.forEach(fn => fn(data)); }
}
```
---
### 6. **Strategy Pattern**
Switch behavior dynamically using interchangeable strategies.  
**Use When**: You want to change an algorithm at runtime without altering code.
```js
class PayPal { pay(a) { console.log(`PayPal: ${a}`); } }
class Stripe { pay(a) { console.log(`Stripe: ${a}`); } }

class PaymentContext {
  setStrategy(s) { this.strategy = s; }
  pay(amount) { this.strategy.pay(amount); }
}
```
---
### 7. **Decorator Pattern**
Wrap objects or functions to add new behavior without modifying the original.  
**Use When**: You want to add responsibilities dynamically.
```js
function withLogging(fn) {
  return (...args) => {
    console.log('Args:', args);
    return fn(...args);
  };
}
```
---
### 8. **Proxy Pattern**
Intermediary for accessing an object — can add caching, logging, access control.  
**Use When**: You want to control access to or extend the behavior of an object.
```js
const target = { msg: "hello" };

const proxy = new Proxy(target, {
  get: (obj, prop) => {
    console.log(`Accessing ${prop}`);
    return obj[prop];
  }
});

console.log(proxy.msg); // logs "Accessing msg" then "hello"
```
---
### 9. **Command Pattern**
Encapsulates a request as an object, allowing parameterization and queuing.  
**Use When**: You need undo/redo, macro recording, or command history.
```js
class Command {
  constructor(execute, undo) {
    this.execute = execute;
    this.undo = undo;
  }
}

const add = x => x + 1;
const subtract = x => x - 1;

const cmd = new Command(add, subtract);
console.log(cmd.execute(5)); // 6
console.log(cmd.undo(6));    // 5
```
---
### 10. **Dependency Injection**
Pass dependencies instead of creating them inside classes/modules.  
**Use When**: You want to decouple components and improve testability.
```js
class Logger {
  log(msg) { console.log(msg); }
}

class UserService {
  constructor(logger) {
    this.logger = logger;
  }

  createUser(user) {
    this.logger.log(`User ${user} created`);
  }
}

const logger = new Logger();
const userService = new UserService(logger);
userService.createUser("Snehal");
```
---
### Anti-Patterns
#### 11. **God Object**
An object that knows too much or does too much.  
**Why Avoid**: Hard to test, understand, or modify.
```js
const god = {
  handleAuth: () => {},
  sendEmail: () => {},
  connectToDB: () => {},
  manageUI: () => {}
};
// Split into services: AuthService, MailService, DBService, UIController
```
#### 12. **Spaghetti Code**
Code with no structure, heavy coupling, and tangled flow.  
**Why Avoid**: Becomes unmaintainable very quickly, especially in JS apps with too many inline handlers.
```js
// Example of bad event handling flow
button.onclick = function() {
  doA();
  if (someCondition) {
    doB(); doC();
  } else {
    doD();
  }
  doE();
};
```
**Fix**: Break into small named functions, use controller layers, follow SOLID principles.

---
### Optional
Here are the ones you can keep in your mental back pocket — _mention if needed_, skip otherwise:
#### 1. **Builder Pattern**
Breaks complex object creation into steps.  
**When?** When constructor has too many params or nested config.
```js
class CarBuilder {
  setColor(color) { this.color = color; return this; }
  setWheels(wheels) { this.wheels = wheels; return this; }
  build() { return new Car(this.color, this.wheels); }
}
```
👉 **Rare in frontend**, more common in backend config-heavy logic.

---
#### 2. **Adapter Pattern**
Converts one interface to another expected by the client.  
**When?** You integrate third-party APIs or legacy systems.
```js
class OldAPI {
  getUser() { return { fname: 'John', lname: 'Doe' }; }
}

class UserAdapter {
  constructor(oldApi) { this.oldApi = oldApi; }
  getFullName() {
    const u = this.oldApi.getUser();
    return `${u.fname} ${u.lname}`;
  }
}
```
👉 **Useful in integration-heavy systems** — APIs, legacy support.

---
#### 3. **Mediator Pattern**
Centralizes communication between components.  
**When?** You want to reduce direct dependency between UI parts.
```js
class ChatRoom {
  register(user) {
    user.room = this;
    this.users = this.users || [];
    this.users.push(user);
  }

  send(from, msg) {
    this.users.forEach(u => u !== from && u.receive(msg));
  }
}
```
👉 Used more in **UI messaging systems**, games, or pub-sub layers.