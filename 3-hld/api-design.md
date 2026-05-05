# API Design & Internal Contracts

### 1. Overview
API design is about creating stable, predictable interfaces for others to consume. For senior engineers, it involves choosing the right protocol (REST, GraphQL, gRPC), managing versioning to avoid breaking changes, and ensuring reliability through idempotency.

### 2. Key Concepts
*   **REST**: Resource-based, using standard HTTP verbs (GET, POST, etc.). Best for public APIs.
*   **GraphQL**: Query language for your API. Client specifies exactly what data it needs. Reduces over-fetching.
*   **Versioning**: Strategic approach to API evolution (URI versioning `/v1/`, Header-based versioning).
*   **Idempotency**: An operation that can be performed multiple times without changing the result beyond the initial application (crucial for retries).
*   **Rate Limiting / Throttling**: Protecting the API from abuse and ensuring fair usage.

### 3. Real-World Usage
*   **Public Dev Platform (Stripe/Twilio)**: Using REST with clean versioning and extensive documentation to guide external developers.
*   **Mobile Apps**: Using GraphQL to fetch nested data (e.g., a "Post" with "Comments" and "Author Details") in a single round-trip over a slow cellular network.
*   **High-Perf Microservices**: Using **gRPC** (Protocol Buffers) for internal service-to-service communication to reduce serialization overhead and leverage bi-directional streaming.
*   **Payments**: Every `POST /payments` request includes an `Idempotency-Key` header so that a retried network failure doesn't charge the user twice.

### 4. Tradeoffs
*   **GraphQL Complexity**: Flexible queries are a dream for frontend but a "N+1" query nightmare for the backend and hard to cache at the CDN level.
*   **Versioning Fatigue**: Maintaining multiple API versions (`v1`, `v2`, `v3`) increases maintenance cost but provides the best user experience for consumers.
*   **REST vs. gRPC**: REST is human-readable and works anywhere; gRPC is binary, faster, but requires specialized clients and is harder to debug with a simple browser.

### 5. When NOT to Use
*   **GraphQL**: Don't use if your data is highly relational and public-facing with simple access patterns. The security risk of malicious complex queries is high.
*   **gRPC**: Avoid for public-facing web APIs where standard browser support is required without extra proxies like `grpc-web`.

### 6. Interview Focus
*   **Contract Design**: "Design an API for a YouTube-like service. How do you handle pagination, video uploading, and partial updates to metadata?"
    * Pagination: Use cursor-based (e.g., `next_cursor`) for infinite scroll, offset-based for fixed pages. Limit defaults to 20-50, max 100-200.
    * Video Upload: Multi-step process: 1) Initiate (get upload URL), 2) Upload (direct to S3/GCS), 3) Notify (webhook when done), 4) Transcode (background job), 5) Publish.
    * Partial Updates: Use PATCH with JSON Patch (RFC 6902) for structured updates, or field-level filtering if simpler. Avoid PUT for partial updates.
    * GraphQL: Return nested data (video + channel + comments) in single query, use DataLoader to batch database calls and prevent N+1.
*   **Reliability**: "A user clicks 'Pay' twice. How do you ensure the transaction only happens once? Explain Idempotency-Key implementation."
    * Use idempotency keys: client sends unique key per request, server stores and checks it before processing.
    * Implementation: use Redis or DB to store key → response mapping for short TTL (e.g., 24 hours).
    * Example flow: check key → if exists, return cached response; if not, process, store result, return.
    * Handle race conditions with pessimistic locks or atomic operations.
*   **Evolution**: "How do you deprecate a field in an API currently used by 100,000 active clients?"
    * Deprecation strategy:
        * Deprecate with warning header: "Field X is deprecated, will be removed in 3 months"
        * Stop returning the field in new responses (but don't delete it yet)
        * Allow reads but ignore writes for deprecated fields
        * Monitor usage via logs/metrics
        * Set a firm removal date
        * Remove only after significant traffic drop
    * Provide migration path (new field, alternate endpoint)
    * Communicate clearly with clients (blog post, deprecation guide)
    * Offer migration window (3-6 months typical)
*   **Designing for Scale**: "How do you design a Twitter feed service that handles 1M writes/sec and 10M reads/sec?"
    * Architecture: Fan-out on write (push model) for active users, fan-out on read (pull model) for celebrities/inactive users.
    * Storage: Separate hot (Redis/memcached for recent tweets) and cold (Cassandra/DynamoDB for timeline storage).
    * Redis: Maintain per-user timeline caches (e.g., last 500 tweets). Use sorted sets for O(log N) adds/retrieves.
    * Cassandra: Partition by user_id (or user_id + timestamp for time-based retrieval). Replication across availability zones.
    * Fan-out on write: When user tweets, push to timelines of all followers (async via message queue).
    * Fan-out on read: For celebrities (1M+ followers), don't push; merge their tweets into user timelines at read time.
    * Hybrid approach: Push for 99% of users, fan-out on read for top 1% (celebrities).
    * Caching: Aggressive caching at multiple layers (CDN, Redis, in-memory).
    * Monitoring: Track write/read throughput, cache hit rates, fan-out latency, queue depth.
    * Rate limiting: Implement per-user rate limits on tweets, follows, etc.
*   **Real-time vs. Batch**: "When to use real-time APIs (WebSockets/SSE) vs. batch APIs?"
    * Real-time (WebSockets/SSE): Use for live notifications, chat, stock tickers, collaborative editing where low latency (<200ms) and continuous updates are critical.
    * Batch APIs: Use for bulk operations (e.g., monthly billing, data exports), periodic syncs, non-urgent updates. Simpler to implement, easier to rate limit, more fault-tolerant.
    * Hybrid: Use batch for base data sync, real-time for delta updates and events.
*   **Rate Limiting Strategy**: "How would you implement rate limiting for a public API?"
    * Token bucket algorithm: 
        * Each user has a bucket that refills at a constant rate (e.g., 100 tokens/minute).
        * Each request consumes tokens (e.g., 1 token/request).
        * If bucket is empty, reject (429 Too Many Requests).
    * Implementation: 
        * Use Redis with atomic operations (INCR/EXPIRE) to store token counts per user/API key.
        * Set sliding window (last N minutes) or fixed window (per calendar minute).
    * Tiers: Free tier (100/min), Pro (1000/min), Enterprise (10000/min).
    * Burst capacity: Allow brief bursts above average rate (e.g., 2x average) to handle traffic spikes.
    * Local vs. Global: For distributed systems, use centralized Redis to track global counts across all instances.
    * Error handling: Return 429 with Retry-After header indicating when client can retry.
*   **Designing for GraphQL**: "How would you handle N+1 queries in a GraphQL API?"
    * Use DataLoader pattern: 
        * Batch multiple requests into single database query (e.g., fetch 100 user_ids in one query instead of 100 separate queries).
        * Implement caching per batch to avoid duplicate work.
        * Use memoization to cache results within a single request.
    * Query cost analysis: Assign "cost" to fields based on complexity (e.g., deep joins cost more). Enforce total cost limit per request.
    * Field-level batching: Group identical field requests across the query tree and resolve them together.
    * Persisted queries: Allow clients to use pre-defined query IDs to avoid sending query strings, reducing parsing overhead.
    * Query complexity analysis: Reject queries that exceed complexity thresholds (e.g., max depth, max nodes).
*   **Designing for REST**: "How would you design a REST API for a URL shortener like TinyURL?"
    * Resources: 
        * `POST /shorten`: Create a new short URL (request: original URL; response: short_code, full_url)
        * `GET /{short_code}`: Redirect to original URL
        * `DELETE /{short_code}`: Delete a short URL (admin only)
        * `GET /analytics/{short_code}`: Get click stats
    * Unique constraints: short_code must be unique; use collision detection with random suffix generation
    * Redirection: 301 (permanent) vs 302 (temporary) based on request parameter
    * Expiration: TTL on short URLs (e.g., 30 days) for cleanup
    * API versioning: `/v1/shorten`, `/v1/analytics`
    * Error codes: 400 (invalid URL), 404 (not found), 409 (collision), 429 (rate limit), 500 (server error)
    * Idempotency: `POST /shorten` with `Idempotency-Key` header to handle retries
    * Caching: Cache redirects at CDN (301) and application level
*   **Versioning Strategies**: "Compare URI versioning vs. Header versioning vs. Content versioning"
    * URI versioning (`/v1/resource`): 
        * Pros: Simple, explicit, cacheable per version, easy to route
        * Cons: Pollutes URI namespace, hard to manage many versions
    * Header versioning (`Accept: application/vnd.api.v1+json`): 
        * Pros: Cleaner URIs, version in metadata
        * Cons: Harder to cache (requires custom cache key), tooling support varies
    * Content versioning (`?version=1`): 
        * Pros: Easy to implement, no URI pollution
        * Cons: Query params affect cache keys, can't version file-based resources
    * Hybrid: URI version for major breaking changes, header for minor changes
    * Recommendation: URI versioning for major changes, header for minor improvements
*   **Error Handling**: "How do you design a robust error handling strategy for a REST API?"
    * Consistent error format: 
        ```json
        { "error": { "code": "invalid_request", "message": "Email is required", "field": "email" } }
        ```
    * HTTP status codes: 
        * 400: Client error (validation, bad request)
        * 401: Unauthorized (missing/invalid auth)
        * 403: Forbidden (access denied)
        * 404: Not found
        * 409: Conflict
        * 429: Rate limit
        * 500: Server error
    * Error codes: Machine-readable codes for client-side logic
    * Error messages: User-friendly descriptions
    * Logging: Log errors with request context for debugging
    * Alerting: Monitor error rates and trigger alerts on anomalies
    * Graceful degradation: Return partial data with error indicators instead of failing completely when possible
    * Idempotency: Idempotent operations should return same error/result for same inputs

### 7. Common Mistakes
*   **Breaking Changes**: Removing a field or changing its data type on a `v1` API without a migration path.
*   **Poor Status Codes**: Returning `200 OK` for an error with a body `{ "error": "failed" }`. Use semantic HTTP codes (400, 401, 403, 404, 500).
*   **Under-filtering**: Fetching an entire User object (including password hash/address) when only the `username` was needed.
