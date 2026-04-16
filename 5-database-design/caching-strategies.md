# Caching Strategies & Invalidation

### 1. Overview
Caching is the ultimate "Speed Hack" for high-performance systems. It involves storing copies of data in a high-speed layer (usually RAM) to serve future requests faster. For senior engineers, the challenge is not "how to cache" but "how to keep the cache consistent with the database" (Invalidation).

### 2. Key Concepts
*   **Cache-aside (Lazy Loading)**: App checks the cache; if not found (Miss), fetches from DB and updates cache. Most common pattern.
*   **Write-through**: App writes to the cache and the cache immediately writes to the DB. Hard consistency.
*   **Write-back (Write-behind)**: App writes to the cache; cache writes to the DB after a delay (batching). High performance but risk of data loss.
*   **TTL (Time To Live)**: Expiration timer for a cache key.
*   **Cache Eviction Policies**: How to decide what to delete when the cache is full (LRU - Least Recently Used, LFU - Least Frequently Used).

### 3. Real-World Usage
*   **Session Management**: Storing user tokens in **Redis** with a 24-hour TTL for sub-millisecond authentication.
*   **Product Details**: Using **Cache-aside** for e-commerce products. The data changes rarely, and a "Miss" once in a while is acceptable.
*   **Aggregated Analytics**: Using **Write-back** for "View Counts." Increment a counter in Redis every second and push the total to SQL every hour.
*   **Content Delivery (CDN)**: Caching static assets (Images, CSS, JS) at the "Edge" (Cloudflare/Akamai) near the user.

### 4. Tradeoffs
*   **Performance vs. Staleness**: A longer TTL means a faster app but users might see old data (e.g., an updated price).
*   **Complexity**: Adding a cache introduces a "Distributed Systems Problem." You now have two sources of truth that can get out of sync.
*   **Cache Stampede**: When a popular key expires and 10,000 requests all hit the database at once trying to "re-fill" the cache.

### 5. When NOT to Use
*   **Highly Dynamic/Unique Data**: If every query is unique (e.g., a "Search" for specific random strings), the cache hit rate will be 0% and the cache just adds latency.
*   **Critical Real-time Data**: If you cannot tolerate even 1 second of "stale" data (e.g., a bank balance), caching is dangerous unless using **Write-through**.

### 6. Interview Focus
*   **Invalidation Strategies**: "The user updated their name. How do you ensure the cache is updated across 10 regions?" (Hint: Pub/Sub or Event-driven invalidation).
*   **Cache Stampede Solution**: "How do you prevent a massive DB spike when a 'Hot Key' (like a celebrity profile) expires?" (Hint: Locking/Leasing or Probabilistic Expiring).
*   **Hit Rate Optimization**: "How do you monitor if a cache is actually helping? What constitutes a 'Good' hit rate?"

### 7. Common Mistakes
*   **The "Leaky" Implementation**: Not setting a TTL on keys, causing Redis memory to grow until it crashes the server.
*   **Ignoring Cold Starts**: Deploying a new app version that clears the whole cache, causing the database to melt under the immediate load.
*   **Caching Sensitive Data**: Storing PII (Personal Identifiable Information) in an unencyrpted, shared cache that can be accessed by other internal teams.
