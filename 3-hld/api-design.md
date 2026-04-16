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
*   **Reliability**: "A user clicks 'Pay' twice. How do you ensure the transaction only happens once? Explain Idempotency-Key implementation."
*   **Evolution**: "How do you deprecate a field in an API currently used by 100,000 active clients?"

### 7. Common Mistakes
*   **Breaking Changes**: Removing a field or changing its data type on a `v1` API without a migration path.
*   **Poor Status Codes**: Returning `200 OK` for an error with a body `{ "error": "failed" }`. Use semantic HTTP codes (400, 401, 403, 404, 500).
*   **Under-filtering**: Fetching an entire User object (including password hash/address) when only the `username` was needed.
