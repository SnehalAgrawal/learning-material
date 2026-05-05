# Backend Performance: Pagination, N+1, & Lazy Loading

### 1. Overview
Performance in the backend is primarily about reducing "Waste"—wasteful network calls, wasteful CPU cycles, and wasteful database scans. For senior engineers, this involves mastering technical patterns that prevent a system from slowing down as data volume grows linearly.

### 2. Key Concepts
*   **Pagination**: Splitting large data sets into smaller "Pages."
    *   *Offset Pagination*: `LIMIT 10 OFFSET 100`. Simple but slow for large offsets.
    *   *Cursor/Keyset Pagination*: `WHERE id > last_seen_id`. Faster and more stable for live feeds.
*   **N+1 Query Problem**: When an application makes one query to fetch `N` parents, and then `N` more queries to fetch children for each parent. Fixed by "Eager Loading" (Joins or IN clauses).
*   **Lazy Loading**: Delaying the initialization of an object or data until it is actually needed.
*   **Response Compression**: Using Gzip or Brotli to reduce the size of JSON payloads over the wire.

### 3. Real-World Usage
*   **Social Media Feeds**: Using **Cursor-based Pagination** (pointing to the timestamp of the last post) to ensure that even if new posts arrive, the user doesn't see duplicates when they scroll down.
*   **ORM Optimization**: Using Django's `select_related` or Eloquent's `with()` to solve the **N+1 problem** in a single DB round-trip.
*   **Product Listings**: **Lazy Loading** the "Customer Reviews" and "Related Products" only when the user scrolls to those sections of the page.
*   **Global APIs**: Compressing 1MB of JSON data down to 100KB using Gzip, significantly improving TTI (Time to Interactive) for users on slow mobile networks.

### 4. Tradeoffs
*   **Eager vs. Lazy Loading**: Eager loading (fetching everything upfront) increases memory usage; Lazy loading (fetching on demand) increases the number of DB round-trips.
*   **Offset vs. Cursor Pagination**: Offset allows "Jump to Page 50" but is slow; Cursor is efficient but only allows "Next/Previous" navigation.
*   **Object Pooling**: Reusing heavy objects (like DB connections) reduces CPU spikes during initialization but consumes "idle" memory.

### 5. When NOT to Use
*   **Lazy Loading in Background Tasks**: Never lazy load inside a loop that processes millions of records (like an export job). You'll destroy your database with thousands of tiny queries. Use Eager Loading + Chunking.
*   **Compression for Tiny Payloads**: The CPU cost of compressing/decompressing a 100-byte JSON object is higher than the network savings.

### 6. Interview Focus
*   **The Log Test**: "I show you an application log with 101 SQL queries for one page load. What is the likely problem and how do you fix it?" (N+1).
*   **Scaling Pagination**: "Why does `OFFSET 1,000,000` make a SQL database slow? How do you fix it for a massive dataset?"
*   **Latency Debugging**: "The API is slow for users in Europe but fast in the US. What backend performance patterns (excluding CDNs) would you look at?" (Hint: Payload size, serialization time, DB location).

### 7. Common Mistakes
*   **Ignoring Serialization Cost**: Thinking "The DB is fast, so the API is fast," while the server spends 200ms converting 10,000 DB rows into JSON.
*   **Over-Pagination**: Using a page size of 10, forcing the frontend to make 50 requests to show a simple list. Use a reasonable default (50-100).
*   **Deep Joins as Eager Loading**: Attempting to "Eager Load" 10 levels of nested relationships in one SQL query, resulting in a Cartesian Product that crashes the DB memory.
