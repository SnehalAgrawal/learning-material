 ### 1. **Types**

TypeScript adds **static types** to JavaScript. Common types include:
- `string`
- `number`
- `boolean`
- `null`, `undefined`
- `any` (escape hatch — avoid it unless necessary)
- `unknown` (safe counterpart to `any`)
- `void` (no return value)
- `never` (function never returns, e.g. throws error)
```ts
let name: string = 'Snehal';
let age: number = 30;
let isActive: boolean = true;
```
### 2. **Type Inference**

TS often infers types, so you don’t always need annotations.

```ts
let city = 'Pune'; // inferred as string`
```
---
### 3. **Functions and Return Types**
You can type arguments and return values:

```ts
function greet(name: string): string {   
  return `Hello, ${name}`; 
}
```
---
### 4. **Interfaces and Type Aliases**
Define object shapes:
```ts
interface User {   id: number;   name: string; }
type Status = 'active' | 'inactive';
```
- Use `interface` for **extending and describing object shapes**. 
- Use `type` for **unions, primitives, complex compositions, and advanced type operations**.
- In modern TS, either can describe object types, but **`interface` is still preferred for public APIs and extendable object types**.
---
### 5. **Optional and Readonly Properties**

Mark properties as optional or read-only:
```ts
interface Profile {   
	username: string;   
	bio?: string;         // optional   
	readonly id: number;  // cannot be changed
}
```
---
### 6. **Union and Intersection Types**
Combine types for flexibility:
```ts
type Result = 'success' | 'error';
type Admin = User & { isAdmin: boolean };
```
---
### 7. **Generics**
Write reusable, type-safe code:
```ts
function wrapInArray<T>(value: T): T[] {   
	return [value]; 
}
```
---
### 8. **Enums**
Represent named constants:
```ts
enum Role {   Admin,   User,   Guest }
```
---
### 9. **Type Assertions**
Tell TS the type explicitly:
```ts
let input = document.getElementById('myInput') as HTMLInputElement;
```
---
### 10. **Type Narrowing**
Use `typeof`, `instanceof`, or control flow to narrow types:
```ts
function printId(id: string | number) {
	if (typeof id === 'string') {
		console.log(id.toUpperCase());
	} else {
		console.log(id.toFixed(2));
	}
}
```
---
### 11. **interface Vs type**
You can model shapes using both `interface` and `type` (with unions), but they serve different purposes and shine in different scenarios. `interface` supports extend key word to extend the model instantly Vs with `type` need to use intersection/union for the same.

Here's a breakdown:
#### 1. `interface` is for describing object shapes and extending them
Interfaces are structural contracts. They are designed to describe the shape of objects and classes.
```ts
interface User {
  id: number;
  name: string;
}
```
interfaces support:
- Extension / inheritance
- Declaration merging: You can add fields to the same interface in multiple declarations — great for library authors.
```ts
// Extension / inheritance
interface Admin extends User {
  isAdmin: boolean;
}
```
#### 2. `type` is more flexible and can represent unions, primitives, and more
```ts
type Status = 'active' | 'inactive' | 'pending'; // union of string literals
type ID = number | string;                       // union of primitives

type User = {
  id: number;
  name: string;
};
```

`type` can be used for:
- Union & intersection types
- Function types
- Tuple types
- Conditional types
- Mapping types

| Use Case                         | Use `interface`      | Use `type`                 |
| -------------------------------- | -------------------- | -------------------------- |
| Describing object/class shape    | Yes                  | Also OK                    |
| Extending shapes via inheritance | Yes (native support) | Harder (use intersections) |
| Creating unions of types         | No                   | Yes (ideal)                |
| Representing functions/tuples    | Not ideal            | Yes                        |
| Declaration merging              | Yes                  | No                         |
| Conditional or mapped types      | No                   | Yes                        |
