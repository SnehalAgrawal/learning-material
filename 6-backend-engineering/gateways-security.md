# Backend Security: AuthN, AuthZ, & API Gateways

### 1. Overview
Secure backend engineering is about building multiple layers of defense. For senior engineers, this involves understanding the difference between "Who are you?" (Authentication) and "What are you allowed to do?" (Authorization), and using an API Gateway as a centralized gatekeeper for the entire system.

### 2. Key Concepts
*   **Authentication (AuthN)**: Identifying the user (Passwords, SSO/OAuth2, MFA).
*   **Authorization (AuthZ)**: Permission management.
    *   *RBAC*: Role-Based Access Control (e.g., "Admin", "User").
    *   *ABAC*: Attribute-Based Access Control (e.g., "Allow if user is owner AND time is before 5PM").
*   **JWT (JSON Web Tokens)**: Stateless, self-contained tokens used to pass user identity between microservices.
*   **API Gateway**: A single entry point that handles Request Routing, Auth, Rate Limiting, and SSL Termination.

### 3. Real-World Usage
*   **OAuth2 / OIDC**: Using Google/GitHub SSO to authenticate users without storing passwords in your own database.
*   **Resource Protection**: A `PostService` that checks if the `user_id` in the JWT matches the `author_id` of the post before allowing an `UPDATE` (ABAC).
*   **Cross-Cutting Concerns**: Using an **API Gateway (Kong, AWS API Gateway, NGINX)** to reject unauthorized requests. See [Reliability Patterns](../4-distributed-systems/reliability-patterns.md) for rate-limiting algorithms.
*   **Internal Security**: Using "mTLS" (Mutual TLS) between microservices to ensure that Service A can ONLY talk to Service B.

### 4. Tradeoffs
*   **Stateless (JWT) vs. Stateful (Sessions)**: JWTs are easy to scale horizontally but impossible to "Revoke" instantly if a user's account is compromised (unless you add a blacklist/DB check, making it stateful again).
*   **Gateway Centralization**: API Gateways simplify architecture but become a "Single Point of Failure" and can introduce unneeded latency if not tuned correctly.
*   **RBAC vs. ABAC**: RBAC is simple to implement but brittle for complex business rules. ABAC is extremely flexible but much harder to audit and slow to evaluate.

### 5. When NOT to Use
*   **JWT for Web Apps**: If you don't have multiple domains/microservices, standard Secure HttpOnly Sessions are often safer and easier to manage than JWTs in LocalStorage (XSS risk).
*   **Complex Gateway Logic**: Avoid putting business logic in the API Gateway. It should only handle infrastructure concerns (Routing, Auth, Rate Limiting).

### 6. Interview Focus
*   **The Invalidation Problem**: "A user's laptop is stolen. How do you invalidate their active JWTs immediately?"
*   **Security Vulnerabilities**: "Explain CSRF (Cross-Site Request Forgery) and how to prevent it."
*   **OAuth Flows**: "Which OAuth2 flow should a React SPA (Single Page App) use vs. a Backend-to-Backend service?" (Hint: Authorization Code with PKCE vs. Client Credentials).

### 7. Common Mistakes
*   **Storing Secrets in Frontend**: Putting API keys or JWT signing secrets in the frontend code.
*   **IDOR (Insecure Direct Object Reference)**: Authenticating a user but failing to authorize them (e.g., a user can access `GET /api/orders/999` just by changing the ID in the URL).
*   **JWT without Verification**: Passing a JWT between services and assuming it's valid without checking the signature against the public key.
