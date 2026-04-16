# JS Advanced Concepts: Memory & Optimization

### 1. Overview
Advanced JavaScript development involves moving beyond building features to building *sustainable* and *performant* systems. This includes understanding the garbage collector to prevent memory leaks, optimizing user interactions with debouncing/throttling, and leveraging functional programming concepts to reduce side effects.

### 2. Key Concepts
*   **Memory Leaks**: When objects are no longer needed but are still reachable from the "Root." (e.g., forgotten global variables, Uncleared timers, closures).
*   **Garbage Collection (GC)**: V8 uses the "Mark-and-Sweep" algorithm. It starts from the root and marks all reachable objects; anything not marked is deleted.
*   **Debounce**: Ensures a function is called only *after* a certain amount of time has passed since it was last called. (e.g., search-as-you-type).
*   **Throttle**: Ensures a function is called at most *once* every X milliseconds. (e.g., scroll event handlers).
*   **Higher-Order Functions**: Functions that take other functions as arguments or return them. (e.g., `map`, `filter`, `reduce`).

### 3. Real-World Usage
*   **Search UI**: Using **Debounce** (e.g., 300ms) on a search input to prevent firing 50 API requests while the user types "JavaScript."
*   **Infinite Scroll**: Using **Throttle** on the `scroll` event to calculate pagination logic at most every 200ms, preventing jank.
*   **Production Debugging**: Using `heap snapshots` in Chrome DevTools to find why an SPA's memory usage grows from 50MB to 500MB after the user navigates between pages.
*   **Data Transformation**: Using **Functional Programming** to transform a messy API response into a clean UI model without mutating the original object.

### 4. Tradeoffs
*   **Functional vs. Imperative**: Functional code is more predictable and easier to test, but it can be less performant in "Hot Loops" due to the creation of many temporary objects.
*   **Debounce vs. Throttle**: Use Debounce for "Wait until stop"; use Throttle for "Fixed execution rate." Using the wrong one can lead to "UI Lag" or "API Spam."
*   **Immutability**: Striving for 100% immutability (`Object.freeze`) is great for reliability but can cause performance issues in apps with massive state objects (like a 3D editor).

### 5. When NOT to Use
*   **In-Place Sorting**: For very large arrays (1M+ items), using immutable `.sort()` (which creates a copy) might crash the browser. Use the in-place `.sort()` but only after careful consideration.
*   **Global Variables**: Never use `var` or declare variables without `let/const`. They attach to the `window` object and are the #1 cause of accidental memory leaks.

### 6. Interview Focus
*   **Leak Identification**: "You notice your React app slows down after navigating back and forth 10 times. How do you find the leak?" (Hint: Event listeners, Timers, or global Stores).
*   **Implementation Logic**: "Write a basic `debounce` function from scratch."
*   **Functional Thinking**: "How do you explain the concept of 'Purity' in functions and why it matters in a React/Redux environment?"

### 7. Common Mistakes
*   **Forgotten Event Listeners**: Adding an event listener on `window` inside a component and not removing it in `componentWillUnmount` (or `useEffect` cleanup).
*   **Closure Leaks**: Holding onto a reference to a large DOM element inside a closure, preventing the entire DOM tree from being garbage collected.
*   **Improper Throttle Settings**: Setting a throttle time too high (e.g., 1000ms for a drag-and-drop), making the UI feel "broken" and laggy.
