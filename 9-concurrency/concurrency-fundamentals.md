# Concurrency Fundamentals: Threads, Processes, & Sync

### 1. Overview
Concurrency is the ability of different parts of a program to be executed out-of-order or in partial order without affecting the final outcome. For senior engineers, this is about managing shared state safely, choosing the right primitive for the job (Threads vs. Processes), and preventing catastrophic failures like Deadlocks or Race Conditions.

### 2. Key Concepts
*   **Process**: An independent unit of execution with its own memory space. Heavy to create/switch.
*   **Thread**: A "Lightweight Process" that exists within a process and shares its memory. Fast but dangerous (shared state).
*   **Race Condition**: When the outcome depends on the non-deterministic timing of two or more threads accessing shared data.
*   **Locks (Mutex)**: A primitive that allows only one thread to access a resource at a time.
*   **Semaphores**: A counter that allows a fixed number of threads to access a resource simultaneously.
*   **Deadlock**: When Thread A is waiting for Thread B, and Thread B is waiting for Thread A, causing the app to hang forever.

### 3. Real-World Usage
*   **Web Servers (Multi-threaded)**: Java or C# servers that spawn a new thread (or pull from a pool) for every incoming HTTP request.
*   **Background Workers (Multi-process)**: Python or Ruby task workers (Celery/Sidekiq) that spawn separate processes to bypass the GIL and use all CPU cores.
*   **Database Connection Pooling**: Using **Semaphores** to limit the number of concurrent connections to a database to prevent overwhelming it.
*   **Safe State Sharing**: Using "Concurrent Collections" (e.g., `ConcurrentHashMap` in Java) to safely store global config without manual lock management.

### 4. Tradeoffs
*   **Threads vs. Processes**: Threads use less memory and switch faster but can crash the entire process if one thread fails. Processes are isolated (safe) but use more RAM and have slow communication (IPC).
*   **Fine-grained vs. Coarse-grained Locking**: Locking a whole object is safe but slow (bottleneck). Locking specific fields is fast but significantly harder to implement without deadlocks.
*   **Optimistic vs. Pessimistic Locking**: Pessimistic (Locks) is better for high contention; Optimistic (Versioning/Check-and-Set) is better for low contention.

### 5. When NOT to Use
*   **Threads for Shared Logic**: Avoid manual thread management and locking if you can use **Message Passing** (e.g., Go Channels or Actor Model). Communicating by sharing memory is the source of 99% of concurrency bugs.
*   **Over-concurrency**: Spawning 10,000 OS threads on a machine with 4 cores. The performance will be destroyed by "Context Switching" overhead. Use "Green Threads" or "Coroutines" instead.

### 6. Interview Focus
*   **The Increment Problem**: "Why is `count += 1` not thread-safe? Describe the sequence of events at the CPU level."
*   **Deadlock Prevention**: "Explain the 4 necessary conditions for a deadlock and how to prevent them in a resource-sharing system." (Hint: Ordering of locks).
*   **Producer-Consumer**: "How would you implement a Producer-Consumer pattern using a thread-safe Queue?"

### 7. Common Mistakes
*   **The "Check-then-Act" Bug**: Checking if a value is null and then doing something with it, without realizing another thread could have made it null in between the check and the act.
*   **Missing 'volatile' or 'Atomic'**: Thinking a standard boolean/integer is enough for a cross-thread "Flag," failing to account for CPU cache visibility issues.
*   **Holding Locks during I/O**: Holding a Mutex while making a network call, which blocks every other thread in the system until the network responds (potentially seconds).
