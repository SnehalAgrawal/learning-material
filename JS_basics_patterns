# JavaScript Interview Prep – Core Concepts & Patterns

This document summarizes key JavaScript concepts and design patterns frequently asked in interviews. It’s tailored for developers and engineering leaders who want a deep, practical understanding of how JavaScript works under the hood.

---

## ✅ Language Fundamentals

### 1. Lexical Scope
- Variable scope is determined by **where functions are written**, not where they're called.
```js
function outer() {
  let x = 10;
  function inner() {
    console.log(x); // 10
  }
  return inner;
}
```

### 2. Closures
- A closure is a function that **remembers** its lexical environment even after its parent function has finished executing.
```js
function makeCounter() {
  let count = 0;
  return () => ++count;
}
```

---

## ✅ `let`, `const`, `var` Differences

| Feature      | `var`           | `let`         | `const`         |
|--------------|------------------|----------------|------------------|
| Scope        | Function          | Block          | Block            |
| Hoisting     | Yes (undefined)   | Yes (TDZ)      | Yes (TDZ)        |
| Reassignable | Yes              | Yes            | ❌               |
| Redeclarable | Yes              | ❌             | ❌               |

---

## ✅ Arrow Functions

- Short syntax for functions.
- **Lexically binds `this`** (doesn't rebind).
```js
const greet = () => console.log('Hello');
```

**Key Differences vs Regular Functions**:
- No `this`, `arguments`, `new`, or `super`.
- Not suitable as constructors or methods needing dynamic `this`.

---

## ✅ Async/Await vs Promises

```js
// Promises
fetch('/api').then(res => res.json()).then(data => console.log(data));

// Async/Await
async function getData() {
  try {
    const res = await fetch('/api');
    const data = await res.json();
  } catch (err) {
    console.error(err);
  }
}
```

- Use `Promise.all()` for concurrency
- Avoid using `await` inside `.forEach()` — prefer `for...of`.

---

## ✅ Module Systems

| Feature          | CommonJS (`require`) | ES Modules (`import`) |
|------------------|----------------------|------------------------|
| Sync/Async       | Sync                 | Async                 |
| Export Syntax    | `module.exports`     | `export`              |
| Scope            | File                 | File                  |
| Used In          | Node.js              | Modern JS (Browser & Node) |

---

## ✅ Memory Leaks in JavaScript

Common sources:
- Global variables
- Forgotten timers/event listeners
- Closures holding onto DOM references

Tools: Chrome DevTools > Memory tab

---

## ✅ Design Patterns in JavaScript

### 1. **Module Pattern**
Encapsulate private data and expose public methods.
```js
const Counter = (() => {
  let count = 0;
  return {
    increment: () => ++count,
    reset: () => (count = 0)
  };
})();
```

### 2. **Singleton Pattern**
Ensure one instance exists.
```js
class Singleton {
  constructor() {
    if (Singleton.instance) return Singleton.instance;
    Singleton.instance = this;
  }
}
```

### 3. **Factory Pattern**
Encapsulate object creation logic.
```js
function createUser(name) {
  return { name, role: 'user' };
}
```

### 4. **Function Factory**
Returns a function, usually customized.
```js
function multiplier(factor) {
  return num => num * factor;
}
```

### 5. **Observer Pattern**
Allows subscription to changes.
```js
class Observable {
  constructor() { this.subs = []; }
  subscribe(fn) { this.subs.push(fn); }
  notify(data) { this.subs.forEach(fn => fn(data)); }
}
```

### 6. **Strategy Pattern**
Switch logic dynamically.
```js
class PayPal { pay(a) { console.log(`PayPal: ${a}`); } }
class Stripe { pay(a) { console.log(`Stripe: ${a}`); } }

class PaymentContext {
  setStrategy(s) { this.strategy = s; }
  pay(amount) { this.strategy.pay(amount); }
}
```

### 7. **Decorator Pattern**
Wrap functions for additional behavior.
```js
function withLogging(fn) {
  return (...args) => {
    console.log('Args:', args);
    return fn(...args);
  };
}
```

---

## ✅ Functional Concepts

- **Debounce / Throttle**: Rate-limit function execution
- **Currying**: Transform `f(a, b)` → `f(a)(b)`
- **Memoization**: Cache result of pure functions
- **Event Delegation**: Attach fewer listeners
- **Composition**: `compose(f, g)(x)` = `f(g(x))`

---

## ✅ Factory Function vs Function Factory

| Feature         | Factory Function                  | Function Factory                      |
|------------------|------------------------------------|----------------------------------------|
| Returns         | An object                         | A function                             |
| Used For        | Object creation                   | Dynamic function generation            |
| Example         | `createUser(name)`                | `makeMultiplier(factor)`              |

---

## ✅ To-Do: Practice Mock Interviews

- [ ] Deep dive into system design with Node/React
- [ ] Build mini apps using these patterns
- [ ] Practice real-world scenario Q&A

---
