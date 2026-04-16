### 1. System design
**System Design** as an umbrella includes much more than just HLD and LLD. When you're designing a system **end-to-end**, especially in real-world scenarios (or senior-level interviews), you go through several **stages or dimensions**, not just two.
#### ✅ 1. **Requirements Clarification**
**Purpose:** Understand what you're solving.
- Functional: What should the system do? (e.g. voice-to-text, response generation)
- Non-Functional: Latency, availability, scale, security
- Constraints: Budget, deadlines, tech stack, existing systems
#### ✅ 2. **High-Level Design (HLD)**
**Purpose:** Define the system architecture and data flow.
- Identify components/modules
- Define interactions between services
- Choose key technologies
- Draw system-level diagrams (boxes and arrows)
- Define external APIs (public interfaces, protocols)
#### ✅ 3. **Low-Level Design (LLD)**
**Purpose:** Flesh out component internals.
- Define class-level design, data structures, and algorithms
- Method-level logic and responsibilities
- Detailed internal APIs or gRPC contracts
- Database schemas, table relationships
- Error-handling logic, retry flows
#### ✅ 4. **Scalability & Performance Design**
**Purpose:** Make the system handle real-world load.
- Horizontal scaling of stateless components
- Load balancing strategies (round-robin, least connection, etc.)
- Caching (Redis, CDN)
- Queues for async processing (e.g. Kafka, RabbitMQ)
- Rate limiting and throttling
#### ✅ 5. **Data Modeling & Storage**
**Purpose:** Choose the right data strategy.
- Relational vs NoSQL vs time-series DBs
- Partitioning/sharding strategy
- Indexing for performance
- Data retention, archival, versioning
#### ✅ 6. **Reliability & Fault Tolerance**
**Purpose:** Ensure uptime and graceful degradation.
- Redundancy and failover
- Retry strategies (exponential backoff, idempotency)
- Circuit breakers and bulkheads
- Graceful degradation paths
- Health checks and self-healing
#### ✅ 7. **Security Design**

**Purpose:** Protect data and system access.
- AuthN/AuthZ (JWT, OAuth2, mTLS)
- Secure APIs (rate limiting, input validation)
- Data encryption at rest and in transit
- Secrets management
#### ✅ 8. **Deployment & DevOps Planning**
**Purpose:** Make it operational and maintainable.
- Docker, Kubernetes, CI/CD pipelines
- Rollout strategies: blue/green, canary, feature flags
- Config management (Helm, env vars)
- Observability: logs, metrics, traces
#### ✅ 9. **Monitoring & Alerting**
**Purpose:** Know when things break.
- Logging (e.g. ELK stack)
- Metrics (Prometheus, Grafana)
- Tracing (Jaeger, OpenTelemetry)
- Alerts and dashboards
#### ✅ 10. **Trade-Off Analysis**
**Purpose:** Justify your choices.
- Latency vs cost
- Consistency vs availability (CAP Theorem)
- Complexity vs maintainability
- Open-source vs managed services
#### Visualizing the Flow:
```csharp
[Requirements Gathering]
         ↓
 [High-Level Design (HLD)]
         ↓
 [Low-Level Design (LLD)]
         ↓
[Scalability / Performance]
         ↓
[Reliability / Fault Tolerance]
         ↓
[Security, DevOps, Monitoring]
         ↓
  [Trade-off Justification]
```
### 2. High-Level Design (HLD) — The Blueprint
**Goal:** Understand the **overall structure** of the system.
**Key Characteristics:**
- **Bird’s-eye view**: Focuses on the _architecture_, _components_, and _interactions_ between systems.
- **Tech stack decisions**: What databases, queues, caches, frameworks, etc.
- **External interfaces**: How your system talks to users or other services.
- **Major components**: Services, modules, APIs, etc.
- **Scalability, availability, security** at a conceptual level.
**Example:**
Designing a voice bot platform
- Services: STT Service, LLM Executor, TTS Engine, Call Orchestrator
- Data flow: WebSocket ↔ STT ↔ LLM ↔ TTS ↔ Plivo
- Tech choices: Google STT, Azure TTS, OpenAI GPT, Node.js, Redis, PostgreSQL
- Communication: gRPC between services, WebSocket for real-time audio
### 3. Low-Level Design (LLD) — The Implementation Plan
**Goal:** Define the **internal logic** and structure of components.
**Key Characteristics:**
- **Class-level and module-level detail**
- **Function signatures**, **algorithms**, **data structures**
- **Database schema**
- Internal API contracts and flows
- Edge-case handling and error recovery
- Deployment configs (e.g. Helm charts, Dockerfiles if deeply detailed)
Example (continued):
For the STT Service:
- Class: `AudioStreamHandler`, `PartialTranscriptBuffer`
- Data Flow: Handle RSV1 from Plivo → Stream chunks → Assemble words with confidence
- Retry logic for failed STT segments
- Redis used for phrase-level caching
- Circuit breaker for fallback to backup STT engine