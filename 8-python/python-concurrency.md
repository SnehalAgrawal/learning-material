# Python Concurrency: Asyncio & Event Loop

### 1. Overview
`asyncio` is a library to write concurrent code using the `async/await` syntax. Unlike multi-threading, which relies on the OS to switch between threads, `asyncio` uses "Cooperative Multitasking" where the application decides when to yield control back to the event loop.

### 2. Key Concepts
*   **Event Loop**: The core of every `asyncio` application. It manages and distributes the execution of different tasks.
*   **Coroutines**: Functions defined with `async def`. They can be "paused" and "resumed" using `await`.
*   **Tasks**: Wrappers for coroutines that schedule them to run on the event loop.
*   **Futures**: Low-level objects that represent an eventual result of an asynchronous operation.
*   **Awaitable**: Any object that can be used in an `await` expression (Coroutines, Tasks, Futures).

### 3. Real-World Usage
*   **High-Concurrency Web Servers**: Using **FastAPI** or **Sanic** (based on `asyncio`) to handle thousands of concurrent JSON requests on a single thread.
*   **Scraping/Batch I/O**: Fetching data from 500 different URLs in parallel without the overhead of 500 OS threads.
*   **Websocket Servers**: Handling persistent, long-lived connections for real-time chat or notifications.
*   **Database Queries**: Using async drivers like `asyncpg` to avoid blocking the event loop while waiting for a slow SQL query.

### 4. Tradeoffs
*   **No Parallelism**: Since it's all on one thread (and governed by the GIL), `asyncio` doesn't speed up CPU-bound code. It only helps with I/O-bound code.
*   **"Color of Functions"**: You can only `await` inside an `async` function. This means you often have to make your entire call stack asynchronous (The "Async Infection").
*   **Debugging Complexity**: Stack traces in `asyncio` can be difficult to read as they often stop at the event loop boundary.

### 5. When NOT to Use
*   **CPU-Bound Tasks**: Using `asyncio` for image processing or heavy math will block the *entire* event loop, making your whole app unresponsive. Use `multiprocessing`.
*   **Blocking Libraries**: If you use a blocking library (like standard `requests`) inside an `async` function, it will block the event loop for all users. You MUST use async-native libraries (like `httpx` or `aiohttp`).

### 6. Interview Focus
*   **Blocking the Loop**: "What happens if I call `time.sleep(10)` inside an `async def` function? How do I fix it?" (Hint: `await asyncio.sleep(10)`).
*   **Concurrency Control**: "How do you limit the number of concurrent tasks in `asyncio` to prevent overwhelming a backend API?" (Hint: `asyncio.Semaphore`).
*   **Thread Integration**: "How do you run a legacy synchronous function inside an async application without blocking the loop?" (Hint: `loop.run_in_executor`).

### 7. Common Mistakes
*   **Defining but not Awaiting**: Defining an `async def` function and calling it without `await`, which doesn't actually execute the body of the function.
*   **Mixing Sync and Async**: Using `asyncio` but then calling a standard, blocking `db.commit()` in the middle, negating all the benefits of the event loop.
*   **Ignoring Exceptions in Tasks**: Spawning tasks as "fire and forget" but not checking their results or catching exceptions, leading to silent failures.
