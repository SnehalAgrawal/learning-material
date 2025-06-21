## MVC, MVP, MVVM architecture
**MVC**, **MVP**, and **MVVM** — all of which are architectural patterns that help organize code in UI-driven applications.

These patterns are **architecture-level ideas**, not bound to a language. But frameworks may lean toward one pattern:
- **React**: Encourages MVVM-ish thinking (state = ViewModel, JSX = View)
- **Angular**: MVVM by design (with services and data binding)
- **Vanilla JS**: You can implement any pattern manually
So you can use MVC, MVP, or MVVM in any JS app — but some are more natural depending on your stack.

We’ll implement a **counter app**:
- One button to increment
- One label to display the count
### 1. **MVC (Model-View-Controller)**
- **Model**: Manages the data and business logic.
- **View**: Handles the UI and presentation layer.
- **Controller**: Handles input, updates the Model, and may update the View.
**Flow**:  
User → Controller → Model → View (View can also pull from Model)
**Key Trait**:  
Controller is tightly coupled with both View and Model. Views may access Model directly.
**Used in**:  
Web frameworks like **Ruby on Rails**, **ASP.NET MVC**, **Django** (sort of), and **Express (loosely)**.
**Example:**
```js
// Model
class CounterModel {
  constructor() {
    this.count = 0;
  }

  increment() {
    this.count++;
  }

  getCount() {
    return this.count;
  }
}

// View
class CounterView {
  constructor() {
    this.button = document.getElementById('increment');
    this.label = document.getElementById('label');
  }

  bindClick(handler) {
    this.button.addEventListener('click', handler);
  }

  update(count) {
    this.label.textContent = count;
  }
}

// Controller
class CounterController {
  constructor(model, view) {
    this.model = model;
    this.view = view;

    this.view.bindClick(this.handleIncrement.bind(this));
    this.view.update(this.model.getCount());
  }

  handleIncrement() {
    this.model.increment();
    this.view.update(this.model.getCount());
  }
}

// Setup
const app = new CounterController(new CounterModel(), new CounterView());
```
---
### 2. **MVP (Model-View-Presenter)**
- **Model**: Business logic and data layer.
- **View**: UI, usually passive (just rendering and input forwarding).
- **Presenter**: Handles all UI logic; receives input from View, updates Model, and updates View.
**Flow**:  
User → View → Presenter → Model → Presenter → View
**Key Trait**:  
**Presenter is unit testable**, View is completely decoupled from the logic. The View is "dumb".
**Used in**:  
Legacy Android apps, desktop apps like **WinForms**, some **Swing** apps.
**Example:**
```js
// Model is same as MVC

// View
class CounterView {
  constructor() {
    this.button = document.getElementById('increment');
    this.label = document.getElementById('label');
  }

  bindClick(handler) {
    this.button.addEventListener('click', handler);
  }

  render(count) {
    this.label.textContent = count;
  }
}

// Presenter
class CounterPresenter {
  constructor(view, model) {
    this.view = view;
    this.model = model;

    this.view.bindClick(this.onClick.bind(this));
    this.view.render(this.model.getCount());
  }

  onClick() {
    this.model.increment();
    this.view.render(this.model.getCount());
  }
}

const presenter = new CounterPresenter(new CounterView(), new CounterModel());
```
---
### 3. **MVVM (Model-View-ViewModel)**
- **Model**: Data and business logic.
- **View**: UI that binds to the ViewModel.
- **ViewModel**: Exposes data and commands to the View. Acts as an abstraction of the View.
**Flow**:  
Two-way binding:  
User ↔ View ↔ ViewModel ↔ Model
**Key Trait**:  
**ViewModel exposes observable data**, so View automatically updates. Clean for data-binding-heavy UIs.
**Used in**:  
Modern **Android (Jetpack)**, **WPF**, **SwiftUI**, **React with Redux/MobX (similar in concept)**
**Example:**
```js
// ViewModel logic
import { useState } from 'react';

function useCounterViewModel() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(prev => prev + 1);
  }

  return { count, increment };
}

// View (React component)
function CounterView() {
  const { count, increment } = useCounterViewModel();

  return (
    <>
      <p>{count}</p>
      <button onClick={increment}>Increment</button>
    </>
  );
}
```

---
### MVC Vs MVP Vs MVVM

| Feature          | MVC                     | MVP                    | MVVM                   |
| ---------------- | ----------------------- | ---------------------- | ---------------------- |
| JS Framework Fit | Express, Vanilla        | Vanilla, older Angular | React, Angular, Vue    |
| Best For         | Clear separation        | Testable UI            | Data-binding-heavy UI  |
| Event Flow       | Controller ↔ Model/View | Presenter ↔ Model/View | ViewModel ↔ Model/View |

---
## Other Architecture

There are many UI and application architecture patterns exist beyond MVC, MVP, and MVVM. Which one you choose depends on your project’s complexity, real-time needs, testability, and team structure.
Here are some **popular architecture patterns** used in frontend and full-stack development:
### 1. **Flux / Redux (Unidirectional Data Flow)**
- **State container pattern**
- Data flows: `Action → Dispatcher → Store → View`
- Encourages **pure functions** and a **single source of truth**
**Used in**: React apps, especially large-scale SPAs
```txt
User clicks → Action → Reducer → New State → UI updates
```
---
### 2. **Clean Architecture**
- Separation of concerns into concentric circles:
    - **Entities (core logic)**
    - **Use Cases (application rules)**
    - **Interface Adapters (views/controllers)**
    - **Frameworks/Drivers (UI, DB, APIs)**
**Used in**: Scalable backend or frontend (e.g., NestJS, Angular, advanced React)
```txt
Dependency rule: Inner layers never depend on outer ones.
```
---
### 3. **Component-Based Architecture**
- Break app into reusable, self-contained components
- Each component manages its own state, behavior, and rendering
**Used in**: React, Vue, Svelte — de facto pattern in modern frontend frameworks
```txt
<Header /> <Sidebar /> <ProductCard /> — all independent
```
---
### 4. **Micro Frontends**
- Split frontend app into multiple **independently deployable** apps
- Each team owns its own piece: code, pipeline, and deployment
**Used in**: Large orgs with multiple teams, like Spotify, Amazon
---
### 5. **Hexagonal Architecture (Ports and Adapters)**
- Separate the core logic from outside world (UI, DB, APIs)
- Input/output goes through ports; core logic plugs into them
**Used in**: Backend-heavy apps, domain-driven design (DDD), but adaptable to frontend
---
### 6. **Observer Pattern / Reactive Programming**
- UI reacts to **observable streams of data** (e.g., RxJS, signals)
- View auto-updates when data changes
**Used in**: Angular (RxJS), SolidJS, Svelte, MobX
---
### 7. **State Machines / Statecharts**
- Model UI behavior using **states and transitions**
- Helps with complex flows like forms, wizards, or chatbots
**Used in**: XState (React, Vue), embedded systems
---
## Summary (When to Use What?)

| Pattern             | Best For                                | Stack Fit                       |
| ------------------- | --------------------------------------- | ------------------------------- |
| MVC / MVP / MVVM    | Traditional separation, testable UI     | Vanilla JS, Android, .NET, etc. |
| Redux / Flux        | Predictable state management            | React, complex SPAs             |
| Clean Architecture  | Scalable, layered design                | Angular, NestJS, full-stack     |
| Component-based     | Modular UI development                  | React, Vue, Svelte              |
| Micro Frontends     | Large teams, domain ownership           | Webpack Module Federation       |
| Observer / Reactive | Real-time UIs, streams, auto reactivity | RxJS, MobX, Solid, Svelte       |
| State Machines      | Finite workflows, UI consistency        | XState, form wizards            |
