# Security Fundamentals: OWASP & Zero Trust

### 1. Overview
Security is not a feature but a foundation. For senior engineers, security involves moving beyond "fixing bugs" to implementing a **Zero Trust Architecture**—the principle of "Never trust, always verify." It requires a deep understanding of the most common web vulnerabilities (OWASP Top 10) and how to design systems that are secure by default.

### 2. Key Concepts
*   **OWASP Top 10**: The standard awareness document for developers and web application security.
    *   *Injection (SQLi, XSS)*: Treating untrusted data as executable code.
    *   *Broken Access Control*: Users accessing data they shouldn't.
    *   *SSRF (Server-Side Request Forgery)*: Tricking a server into making requests to internal or external resources.
*   **Zero Trust Architecture**: No user or service is trusted by default, even if they are inside the "Corporate Network." Every request must be authenticated and authorized.
*   **Encryption**:
    *   *At-Rest*: Encrypting data on disk (AES-256).
    *   *In-Transit*: Encrypting data over the network (TLS 1.3).
*   **Secrets Management**: Safely storing and rotating API keys, DB passwords, and SSH keys (e.g., AWS Secrets Manager, HashiCorp Vault).

### 3. Real-World Usage
*   **Data Protection**: Hashing user passwords using **Argon2** or **BCrypt** with a salt to protect against rainbow table attacks.
*   **SSRF Protection**: Using a "Network Proxy" or strict "Allow-lists" for any service that has to make external HTTP calls based on user input (e.g., a "URL Preview" generator).
*   **Access Control**: Implementing "Least Privilege"—a microservice that only needs to *read* from an S3 bucket should not have *delete* permissions.
*   **Identity Management**: Using **OAuth2/OpenID Connect** so that you don't have to manage sensitive user credentials yourself.

### 4. Tradeoffs
*   **Security vs. Convenience**: Adding MFA (Multi-Factor Auth) or short-lived sessions increases security but adds friction for users.
*   **Runtime Scans vs. Speed**: Running daily "Security Vulnerability Scans" on your dependencies (using Snyk or NPM Audit) takes time in CI but prevents shipping infected libraries.
*   **Centralized Secrets vs. Fragility**: Using a "Secrets Vault" is more secure than ENV files but creates a dependency—if the Vault is down, the whole system might fail to boot.

### 5. When NOT to Use
*   **Manual Encryption Logic**: **NEVER** write your own encryption algorithm. Always use standard, peer-reviewed libraries (like OpenSSL or NaCL).
*   **Hidden 'Deep' URLs**: Don't rely on "Security through Obscurity" (e.g., `api.myapp.com/admin-732-hidden`). If it exists, an attacker will find it. Use real Auth.

### 6. Interview Focus
*   **Vulnerability Resolution**: "I show you an API that takes a `userId` from the URL and returns their data. What are the 3 security risks and how do you fix them?" (Hint: IDOR, SQLi, and Lack of Auth).
*   **Zero Trust**: "How do you implement Zero Trust for a suite of 20 microservices communicating with each other?" (Hint: mTLS + JWT for service identity).
*   **Data Breach mitigation**: "Our database was leaked. If we followed best practices, what data should be useless to the attacker?" (Hint: Properly salted and hashed passwords).

### 7. Common Mistakes
*   **Logging PII/Secrets**: Accidnetally logging a user's password or credit card number to a centralized logging system (Sentry/Splunk/CloudWatch).
*   **Implicit Trust**: Trusting an internal service just because it's "behind the firewall." If an attacker gets into one weak service, they can easily pivot to everything else (Lateral Movement).
*   **Default Credentials**: Shipping a system with `admin/admin` or `root/password` anywhere in the configuration.
