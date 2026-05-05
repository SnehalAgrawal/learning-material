# Reliability Patterns: Circuit Breakers, Retries, & Rate Limiting

### 1. Overview
Reliability patterns are the "Protective Gear" of a distributed system. Since network calls are unreliable, senior engineers design systems that can "fail gracefully" rather than crashing hard. These patterns prevent individual service failures from becoming global outages.

### 2. Key Concepts
*   **Circuit Breaker**: Detects failures and "opens" to stop calls to a failing service, allowing it time to recover and protecting the caller from hanging.
    * **App level**
        ```javascript
        const CircuitBreaker = require('opossum');

        const options = {
        timeout: 3000, // fail if slow
        errorThresholdPercentage: 50, // open circuit if failures > 50%
        resetTimeout: 10000 // try again after 10s
        };

        const breaker = new CircuitBreaker(callExternalService, options);
        breaker.fallback(() => "fallback response");
        breaker.fire();
        ```
        Why app-level?
            * Fine-grained control per API call
            * Custom fallback logic
            * Easier debugging
    * **Infrastructure level**
        
        You can also implement it outside your code using:
        * Istio
        * Linkerd
        * Envoy
*   **Retry with Exponential Backoff**: Automatically retries a failed operation but waits longer between each attempt (e.g., 100ms, 200ms, 400ms) to avoid "thundering herd" attacks on a struggling service.
*   **Bulkhead**: Isolates pools of resources (threads, connections) so that a failure in one area doesn't starve another.
*   **Rate Limiting**: Restricts the number of requests a client can make in a given timeframe (e.g., 100 requests/minute).
*   **Jitter**: Adding a small amount of randomness to retry intervals to prevent all failed clients from retrying at the exact same millisecond.

### 3. Real-World Usage
*   **API Gateway**: Implementing **Rate Limiting** based on API keys to ensure a single noisy customer doesn't degrade performance for others.
*   **Payment Processing**: Using **Retries** for transient 503 errors from a payment provider, but using a **Circuit Breaker** if they are returning 100% errors for 10 minutes.
*   **Search Service**: Using a **Bulkhead** implementation (like a separate thread pool) for "Standard Search" vs "Admin Search," so users don't block admins from viewing logs.
*   **Third-party SDKs**: AWS SDKs have built-in **Exponential Backoff and Jitter** for all service calls.

### 4. Tradeoffs
*   **Retries vs. Latency**: Retries increase the total time a user waits for a response. If a service is down, retrying 5 times at a 1-second timeout means the user waits 5 seconds for a failure message.
*   **Rate Limiting Complexity**: Global rate limiting (using Redis) adds network overhead; local rate limiting (per instance) is faster but less accurate in a cluster.
*   **Fallback Logic**: A Circuit Breaker is only as good as its fallback (e.g., "Return a cached list" vs "Return an empty list" vs "Return an error").

### 5. When NOT to Use
*   **Retries on Non-Idempotent Operations**: **CRITICAL**: Never retry a non-idempotent `POST` request (like `/charge_payment`) unless you have an `Idempotency-Key` or you risk charging the user twice.
*   **Exponential Backoff for Human UI**: If a user clicks a button, waiting 10 seconds for the 5th retry is a bad UX. Fail early and tell the user.

### 6. Interview Focus
*   **The "Thundering Herd"**: "What happens when 1,000 clients all retry their failed requests at the exact same moment? How do you fix it?" (Hint: Jitter).

    * 1000 clients fail simultaneously → all retry at same time (0ms)
    * Creates massive load spike on already struggling service
    * Can cause cascading failures
    * Fix: Exponential backoff with jitter:
    * Retry 1: 100ms (±random)
    * Retry 2: 200ms (±random)
    * Retry 3: 400ms (±random)
    * ...
    * Spreads out retries over time
    * Reduces peak load
    * Gives service time to recover
*   **Circuit Breaker States**: "Explain the 'Half-Open' state of a circuit breaker. How does it transition back to 'Closed'?"
    * Three states:
    * CLOSED: Normal operation, normal traffic
    * OPEN: Fail fast, stop sending traffic
    * HALF-OPEN: Allow limited test traffic
    * Transitions:
        * Closed → Open: Too many failures (error threshold)
        * Open → Half-Open: Timeout expires (e.g., 1 min)
        * Half-Open → Closed: Some tests succeed
        * Half-Open → Open: Tests fail again
*   **Rate Limiting Algorithms**: "Compare 'Leaky Bucket' vs 'Token Bucket' algorithms for rate limiting."
    * Leaky Bucket:
        * Fixed output rate (like a leaky faucet)
        * Smooths out traffic
        * Good for: API rate limiting, traffic shaping
    * Token Bucket:
        * Variable output rate (tokens added over time)
        * Allows bursts of traffic
        * Good for: Burst-tolerant systems, dynamic rate limiting

### 7. Common Mistakes
*   **Infinite Retries**: Not setting a maximum retry count, causing requests to loop forever and consume resources.
*   **Missing Jitter**: All servers in a cluster recovering and hitting the database at the exact same nanosecond.
*   **Single-Threaded Hangups**: Not setting an explicit **Timeout** on global calls. A slow service is often more dangerous than a dead service because it uses up all available connection slots.
