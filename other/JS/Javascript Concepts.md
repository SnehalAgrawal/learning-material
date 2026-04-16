
1. **Variables (let, const, var)**

| Feature      | `var`           | `let`     | `const`   |
| ------------ | --------------- | --------- | --------- |
| Scope        | Function        | Block     | Block     |
| Hoisting     | Yes (undefined) | Yes (TDZ) | Yes (TDZ) |
| Reassignable | Yes             | Yes       | ❌         |
| Redeclarable | Yes             | ❌         | ❌         |
```js
let a = 1;       // Block scoped
const b = 2;     // Block scoped + immutable
var c = 3;       // Function scoped
```

2. **Data Types**
- **Primitive**: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`
- **Reference**: `object`, `array`, `function`
```js
let x = "hello"; // string
let y = { name: "JS" }; // object
```

3. **Functions**
```js
function sum(a, b) { return a + b; }     // Regular
const mul = (a, b) => a * b;             // Arrow
(function greet() { console.log("Hi"); })(); // IIFE
```

4. **Hoisting**
let/const are hoisted but not initialized
```js
console.log(x); // undefined
var x = 5;
```

5. **Closures**
Lexical Scope: Variable scope is determined by **where functions are written**, not where they're called.
```js
function outer() {
  let x = 10;
  function inner() {
    console.log(x); // 10
  }
  return inner;
}
```
A closure is a function that **remembers** its lexical environment even after its parent function has finished executing.
```js
function outer() {
  let count = 0;
  return function inner() {
    return ++count;
  }
}
const counter = outer();
console.log(counter()); // 1
console.log(counter()); // 2
```

6. **This Keyword**
```js
const obj = {
  val: 10,
  getVal() {
    return this.val;
  }
};
obj.getVal(); // 10


// Normal function Vs arrow function this
const obj = {
      name: 'Object',
      regularFunction: function() {
        console.log(this.name); // "this" refers to obj
      },
      arrowFunction: () => {
        console.log(this.name); // "this" refers to window/global
      }
};

obj.regularFunction(); // Outputs: Object
obj.arrowFunction();    // Outputs: undefined (window/global)
```
Arrow functions **don’t bind** their own `this`.

7. **Prototypes & Inheritance**
```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function() {
  return `Hi, ${this.name}`;
};
```

8. **Call, Apply, Bind**
```js
function greet(msg) { return `${msg}, ${this.name}`; }
const user = { name: "Alice" };

greet.call(user, "Hello");
greet.apply(user, ["Hi"]);
const bound = greet.bind(user);
bound("Hey");
```

9. **Promises & Async/Await**
```js
const fetchData = () =>
  new Promise((res, rej) => setTimeout(() => res("done"), 1000));

async function getData() {
  const result = await fetchData();
  console.log(result);
}
```

10. **Event Loop (Microtasks vs Macrotasks)**
Micro task queue: Promise, Event observer, executeMicroTask()
Task queue: all the other jobs then Micro task queue tasks
```js
console.log("Start");
setTimeout(() => console.log("Timeout"), 0);
Promise.resolve().then(() => console.log("Promise"));
console.log("End");
// Output: Start → End → Promise → Timeout
```

11. **Destructuring & Spread/Rest**
```js
const [a, b] = [1, 2];
const { x, y } = { x: 5, y: 6 };

const nums = [1, 2, 3];
const more = [...nums, 4];
function log(...args) { console.log(args); }
```

12. **Array Methods**
```js
[1, 2, 3].map(x => x * 2);
[1, 2, 3].filter(x => x > 1);
[1, 2, 3].reduce((acc, x) => acc + x, 0);
```

13. **Optional Chaining & Nullish Coalescing**
`??` works for "", null, undefined but || for falsy value
```js
const user = { name: "Alice", address: null };
user?.address?.city;        // undefined
user.address ?? "No address"; // "No address"
```

14. **Modules (ESM)**
```js
// export.js
export const val = 5;
export default function greet() { return "Hi"; }

// import.js
import greet, { val } from './export.js';
```

15. **Classes**
```js
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes noise`; }
}
```

16. **Debounce & Throttle (Essentials in DOM)**
```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
```

17. **Deep vs Shallow Copy**
```js
const obj = { a: 1, b: { c: 2 } };
const shallow = { ...obj };
const deep = JSON.parse(JSON.stringify(obj));
```

18. **Equality**
```js
0 == false     // true (loose)
0 === false    // false (strict)
NaN === NaN    // false → use Number.isNaN(NaN)
```

19. **Truthy / Falsy**
```js
if ("") console.log("won't run");
```

20. **Set, Map, WeakMap, WeakSet**
```js
const set = new Set([1, 2, 2]); // unique
const map = new Map([[1, "one"]]);
```

21. **Arrow Functions**
- Short syntax for functions.
- **Lexically binds `this`** (doesn't rebind).
```js
const greet = () => console.log('Hello');
```
**Key Differences vs Regular Functions**:
- No `this`, `arguments`, `new`, or `super`.
- Not suitable as constructors or methods needing dynamic `this`.

22. **Async/Await vs Promises**
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
- Use `Promise.all()` for concurrency
- Avoid using `await` inside `.forEach()` — prefer `for...of`.

23. **Memory Leaks in JavaScript**
    Common sources:
    - Global variables
    - Forgotten timers/event listeners
    - Closures holding onto DOM references
	Tools: Chrome DevTools > Memory tab
---
#### ✅ Functional Concepts
- **Debounce / Throttle**: Rate-limit function execution
- **Currying**: Transform `f(a, b)` → `f(a)(b)`
- **Memoization**: Cache result of pure functions
- **Event Delegation**: Attach fewer listeners
- **Composition**: `compose(f, g)(x)` = `f(g(x))`
#### ✅ Factory Function vs Function Factory

| Feature  | Factory Function   | Function Factory            |
| -------- | ------------------ | --------------------------- |
| Returns  | An object          | A function                  |
| Used For | Object creation    | Dynamic function generation |
| Example  | `createUser(name)` | `makeMultiplier(factor)`    |
