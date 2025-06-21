### **1. Designing Scalable Distributed Systems**

**Core Concepts:**
- Horizontal vs. vertical scaling
- Stateless vs. stateful services
- CAP Theorem (Consistency, Availability, Partition Tolerance)
	What it is:  
		In any distributed system, you can only guarantee 2 out of 3:
		- **C**onsistency: All nodes return the same data.
		- **A**vailability: Every request gets a response (success or failure).
		- **Partition Tolerance**: System continues to operate despite network partitions.
	Why it's useful:  
		Helps you make trade-offs in design — no system can guarantee all 3. You **must choose** based on your domain.
	Real-world example:		
	- **Banking system** (Consistency + Partition Tolerance): You’d prefer to reject a request rather than allow stale balance info.
	- **WhatsApp last-seen status** (Availability + Partition Tolerance): Might show slightly outdated status to avoid user delay.
- Microservices vs. monoliths
- Service discovery (e.g., Consul, etcd)
	**What it is:**  
		Mechanism for services to **find each other dynamically** without hardcoding IPs. Essential in **microservices or Kubernetes**.
	**Why it's useful:**  
		Scales well when instances change frequently (e.g., autoscaling). Keeps systems loosely coupled.
	**Real-world example:**  
		In your **voice bot** system:
		- Workflow executor needs to talk to STT/LLM services.
		- Instead of configuring IPs, each service **registers itself with Consul**.
		- The bot queries Consul: “Where’s STT?” and gets a current healthy instance.
- Communication (REST, gRPC, async messaging)
	- **REST** for internal admin tools or simple CRUD.
	- **gRPC** if you want faster structured comms between workflow executor and STT engine.
	- **Async messaging** (e.g., RabbitMQ/Kafka) if you want the bot to continue even if STT takes time.
- Fault tolerance, retries, circuit breakers (Hystrix pattern)
	- **What it is:**  
		Techniques to keep your system resilient when downstream services fail.
		- **Retries:** Try again after a failure (with backoff).
		- **Timeouts:** Don't hang forever waiting for response.
		- **Circuit Breaker:** If failures cross threshold, stop sending requests temporarily.
		**Real-world example:**  
		In your LLM + TTS flow:
		- If TTS API fails 3 times → circuit breaker trips for 1 min.
		- Meanwhile, fallback to a **cached TTS response** or say: “Sorry, please repeat.”
		- Helps keep call running instead of crashing due to one bad service.

**Typical Interview Questions:**
- Design a URL shortening service like Bit.ly that handles millions of requests daily.
	**Key Components:**
	- **API layer** to shorten/expand URLs
	- **Hash generator** (e.g., base62 of ID, or random string)
	- **Database** to store mappings (short → original)
	- **Caching** (e.g., Redis) to speed up redirect
	- **Rate limiting** to prevent abuse
	- **Analytics** pipeline (async via queue)
	**Scalability:**
	- Use **sharded DBs** for write scale
	- Cache hot URLs in Redis
	- Serve redirects via **CDN or edge servers**
- How would you design a system like Uber or WhatsApp? 
	- Uber (real-time geo & matching):
		- **Rider & driver apps** (mobile clients)
		- **Location service** (pub-sub system with Kafka or Redis Streams)
		- **Matching engine** (real-time ride assignment, queues)
		- **Pricing engine**, **payment service**
		- **Notification service** (push/SMS)
		**Tech stack:** Microservices, Kafka, Redis, PostGIS, gRPC, WebSockets
	* 💬 WhatsApp (messaging system):
		- **User service**, **contact service**
		- **Message queue system** (Kafka or custom)
		- **Delivery status tracking**, **storage** (S3, Cassandra)
		- **End-to-end encryption** (Signal protocol)
		- **Presence service** for "online/last seen"
		**Focus:** Low latency, offline sync, async retries, partition-tolerant message queues
- How do you ensure consistency in a distributed system?
- What happens when part of the system is down? How do you design for that?

**Your Edge:**
Highlight real examples from your voice bot system or Plivo integration — that’s a real-time distributed system already.

---

### **2. Event-Driven Architecture**

**Core Concepts:**
- Asynchronous communication via events
- Event producers, consumers, brokers (Kafka, RabbitMQ, etc.)    
- Event sourcing vs. state-based persistence
	-**Bank account:**
	- **State-based:** DB has one row → balance = ₹1,000
	- **Event-sourced:** DB has logs →
		- Deposit ₹500
		- Withdraw ₹200
		- Deposit ₹700
		- Replay events = balance ₹1,000
	#### 🧠 Why use event sourcing?
	- Complete **audit/history** (e.g., for financial, audit, logs)
	- Can **replay events** to rebuild state or debug
	- Supports **CQRS** well (next topic)
- CQRS (Command Query Responsibility Segregation)
- Idempotency and retries

**Typical Interview Questions:**
- What is event-driven architecture and when would you use it?
- Design a stock trading system using an event-driven approach.
- How do you handle message ordering or deduplication?

**Pro Tip:**  
Tie this back to your use of queues or CDC in data sync, especially if you're already using something like Kafka or Bull for jobs.

Kafka:
- Distributed, durable **event streaming platform**
- **High throughput**, low latency (millions of msgs/sec)
- **Message replay** via offsets (great for debugging or reprocessing)
- Stores data **persistently** on disk (append-only log)
- **Scalable** via partitions and brokers
- Supports **multiple independent consumers**
- Used for **real-time data pipelines**, **event sourcing**, **analytics**
Use Kafka When:
- You need **replayable**, high-volume **event streams**
- Systems must **scale massively**
- You’re implementing **event-driven or microservice architectures**
Use RabbitMQ When:
- Simpler **task queueing**
- You need **built-in retries/DLQ**
- Lower throughput and **tighter delivery guarantees**

---
### **3. API Design**

**Core Concepts:**
- REST vs. GraphQL vs. gRPC
	- **REST**: Resource-based, widely used, easy to cache/debug
	- **GraphQL**: Flexible queries, fetch only needed data, good for mobile/UI apps
	- **gRPC**: Fast, binary (Protobuf), good for internal microservice-to-microservice calls
- Versioning (URI-based, header-based)
	- **URI-based**: `/v1/rides` — simple, clear
	- **Header-based**: `Accept: application/vnd.company.v2+json` — cleaner URLs
	  ```javascript
		app.use((req, res, next) => {
		  const acceptHeader = req.headers['accept'] || '';
		  const versionMatch = acceptHeader.match(/vnd\.myapp\.v(\d+)\+json/);
		  req.apiVersion = versionMatch ? versionMatch[1] : '1'; // Default to v1
		  next();
		});
		
		app.get('/users', (req, res) => {
		  if (req.apiVersion === '1') {
			// Do what you want
		  } else if (req.apiVersion === '2') {
			// Do what you want
		  } else {
			// Do what you want
		  }
		});
			```
- Pagination, filtering, rate limiting
- OpenAPI/Swagger
- Authentication and authorization (OAuth2, JWT)
	- **OAuth2**: Authorization for third-party apps (Google Login, etc.)
	- **JWT**: Token-based auth (stateless, signed tokens with claims)
	- Use **scopes/roles** for RBAC (e.g., `admin`, `driver`, `user`)

**Typical Interview Questions:**
- Design an API for a ride-hailing app.
- How would you handle versioning of APIs in a microservices setup?
- How do you secure APIs?

**Good to Mention:**  
How you’ve designed APIs for internal microservices (Plivo, STT, LLM trigger) and how you ensure backwards compatibility or low latency.

---

### **4. System Performance, Load Balancing**

**Core Concepts:**
- Throughput, latency, p99 response times
- Horizontal scaling with load balancers (NGINX, HAProxy, Azure Load Balancer)
- Autoscaling, health checks
- Caching layers (CDN, API gateway caching)
- Backpressure and rate limiting

**Typical Interview Questions:**
- How would you scale a real-time chat system to handle 1M concurrent users?
	- Use **WebSockets**, split users across **sharded servers**
	- Load balance via **reverse proxy** or L7 gateway
	- Use **Redis pub/sub** or Kafka for cross-node messaging
- What are your strategies for reducing latency?
	- Use **CDNs**, **local caches**
	- Avoid N+1 DB queries, optimize SQL (The **N+1 problem** happens when your code makes **1 query to fetch a list**, and then **N additional queries**—**one per item in the list**.)
	- **Async processing** for non-critical work
	- Deploy near users (edge/CDN regions)
- Explain how load balancing works and where it fits in a cloud-native app.
	- Traffic hits **cloud load balancer** → routes to healthy app instances
	- Supports **health checks**, SSL termination, session affinity
	- Works across **zones/regions** for resiliency

**Good to Add:**  
Share how you handled 200+ concurrent voice bot calls with 55+ pods and what bottlenecks you hit.

---

### **5. Database Design, Caching Strategies**

**Core Concepts:**
- RDBMS vs. NoSQL vs. NewSQL
- Indexing, normalization, denormalization
- Caching layers: in-memory (Redis, Memcached), DB-level caching, CDN
- Cache invalidation, cache-aside, write-through
- Sharding and partitioning
	- Partitioning: Within a single database
		Partitioning means **splitting a large table into smaller, manageable parts** (partitions), stored **within the same database/server**.
		- Types of Partitioning:
			- **Range**: Based on ranges of values, e.g., `created_at < 2024-01-01` in one partition, `>= 2024-01-01` in another
			- **List**: Based on predefined values, e.g., customers from `'US'`, `'IN'`, `'EU'` in different partitions
			- **Hash**: Based on a hash of a key (e.g., user ID)
			- **Composite**: Combines range + hash
		You have a `transactions` table with **500M rows**. Partitioning it **by month** keeps each segment smaller, making queries like:
		`SELECT * FROM transactions WHERE created_at >= '2024-01-01'`
		- **PostgreSQL**: native table partitioning (`PARTITION BY`)
		- **MySQL**: support via `PARTITION BY RANGE/LIST/HASH`
		- **BigQuery / Athena**: heavily rely on partitioning
		- How to do partition
			```sql
			CREATE TABLE transactions (
			    id SERIAL PRIMARY KEY,
			    user_id INT NOT NULL,
			    amount NUMERIC(10, 2),
			    created_at DATE NOT NULL
			) PARTITION BY RANGE (created_at);
			
			CREATE TABLE transactions_2024_01 PARTITION OF transactions
			FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');		
			
			CREATE TABLE transactions_2024_02 PARTITION OF transactions
			FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
			
			SELECT * FROM transactions WHERE created_at BETWEEN '2024-01-10' AND '2024-01-31';
			```
	- Sharding: Across multiple databases/servers**
		Sharding is **horizontal scaling of the database**, where data is split **across different servers/instances**. Each shard holds a **subset of the data**.
		The key you shard by must ensure even distribution. Common:
		- **User ID** 
		- **Tenant ID** (in multi-tenant apps)
		- **Geographic region**
		Drawbacks:
		- Cross-shard joins are hard (or impossible)
		- Requires **custom routing logic**
		- Backup/restore gets more complex
		An app with 100 million users might store:
		- Users with `user_id % 4 === 0` in DB shard A
		- `% 1 === 1` in DB shard B, and so on...
		Apps like Instagram, Uber, and YouTube use sharding to handle global scale.
		Supported Tools: MongoDB (native sharding), Vitess (MySQL), CockroachDB, custom app logic with Postgres/MySQL
**Typical Interview Questions:**
- Design the schema for an e-commerce app.
- How would you design a high-read, low-write system?
- Where would you place caching in your system and why?

**Anchor Points:**  
Talk about your MongoDB to PostgreSQL sync (CDC project) and how you approached caching or read optimization.

---

### **6. CI/CD Pipelines**

**Core Concepts:**
- Build automation (Jenkins, GitHub Actions, Azure DevOps)
- Testing stages (unit, integration, e2e)
- Rollbacks, canary deployments, blue-green deployments
	- **Canary**: Release to a small % of users → monitor → full rollout
	- **Blue-Green**: Two environments (blue & green); switch traffic when green is stable
- Secrets management
- Containerization (Docker), orchestration (Kubernetes)

**Typical Interview Questions:**
- Walk me through your CI/CD setup.
- How do you ensure safe deployments in production?
- What is your rollback strategy if a deployment fails?

**Relevant Projects:**  
Describe your Kubernetes + Docker setup and how you deploy microservices — bonus if you’ve done blue-green or canary releases.

---

### **7. Cloud Platform (especially Azure)**

**Core Concepts:**
- Azure App Services, Functions, AKS (Kubernetes)
	- **App Services**: Managed platform for hosting web apps & APIs, auto-scaling, built-in CI/CD, ideal for web workloads.
	- **Azure Functions**: Serverless compute, event-driven, scales automatically, pay-per-execution — great for small, stateless tasks or event triggers.
	- **AKS (Azure Kubernetes Service)**: Managed Kubernetes for container orchestration, suitable for complex microservices and stateful workloads requiring full container control.
- Azure Storage, Cosmos DB, Azure SQL
	- **Azure Storage**: Blob, Queue, Table storage for unstructured data, messaging, and NoSQL-like key-value.
	- **Cosmos DB**: Globally distributed multi-model NoSQL DB with low-latency, multi-region writes, supports SQL, MongoDB, Cassandra APIs.
	- **Azure SQL**: Fully managed relational DB service (SQL Server engine) with scaling, backups, and security.
- Azure Event Hubs, Service Bus
	- **Event Hubs**: Big data streaming platform & event ingestion service — ideal for telemetry, logs, high-throughput event streams.
	- **Service Bus**: Enterprise messaging service supporting queues, topics, and advanced messaging patterns for decoupling microservices.
- Monitoring (Application Insights, Log Analytics)
	- **Application Insights**: Application performance monitoring (APM), request rates, failures, dependency tracking.
	- **Log Analytics**: Aggregates logs and metrics across resources, supports powerful queries with Kusto Query Language (KQL).
- Identity & Access Management (Azure AD, RBAC)
	- **Azure AD**: Identity provider for users and apps, supports OAuth2, OpenID Connect.
	- **RBAC (Role-Based Access Control)**: Fine-grained access permissions for Azure resources, minimizes blast radius.

**Typical Interview Questions:**
- How would you architect your app on Azure to be scalable and cost-effective?
- Compare Azure Functions vs App Services for a given use case.
- How do you handle logging, metrics, and alerting in Azure?

**Highlight:**  
If you’ve integrated Azure TTS or streaming, that’s a clear bonus — they’ll want to hear about that experience.

---

### **8. Data Pipelines**

**Core Concepts:**
- ETL vs ELT    
- Real-time streaming (Kafka, Flink, Azure Stream Analytics)
	- Use real-time streaming for logs, clickstreams, fraud detection, IoT telemetry.
- Batch processing (Spark, Azure Data Factory)
	- Azure Data Factory
	- Airpflow (Directed Acyclic Graph)
- Schema evolution, data lineage
	- **Schema evolution**: Ability to handle changes in data schema over time without breaking pipelines.
	    - Tools: Apache Avro, Protobuf, Schema Registry (Kafka)
	- **Data lineage**: Tracking how data flows and transforms through the pipeline.
	    - Tools: OpenLineage, Azure Purview, dbt (with docs)
- Observability and backpressure
	- **Observability**: Metrics, logs, tracing of data flows (e.g., lag in Kafka, processing time, errors)
	- **Backpressure**: When downstream systems can't keep up → buffer or throttle inputs (handled by Flink, Kafka, Akka Streams, etc.)

**Typical Interview Questions:**
- How would you design a data pipeline for processing user clickstreams?
	- Use **Kafka** to ingest click events, **Flink** or **Stream Analytics** to process/aggregate in real-time, and sink to **Data Lake**, **Cosmos DB**, or **Synapse Analytics** for querying. Add **Data Factory** for nightly aggregation jobs.
- How do you handle schema changes in streaming pipelines?
	- Use **Schema Registry** (e.g., Confluent for Kafka) with Avro or Protobuf for versioned schema enforcement. Set compatibility mode (e.g., backward or full). Add fallback logic in consumers.
- What's your approach to ensuring data integrity in a multi-stage pipeline?
	- Add checkpoints and retries, enforce idempotency, use **exactly-once semantics** where supported (e.g., Flink), monitor using metrics (lag, drop rate), and validate with data quality checks (e.g., dbt tests, Great Expectations).

**You Could Mention:**  
Your MongoDB to PostgreSQL pipeline, and if you’re doing analytics or async workflows from the voice bot data.

---

### **9. Prompt Engineering and LLM Integration**

**Core Concepts:**
- Prompt templates, few-shot vs zero-shot
	- **Prompt templates**: Reusable input patterns (e.g., `"You are a helpful assistant. Given: {input}, Respond with:"`)
	- **Zero-shot**: Model responds without examples.
	- **Few-shot**: Prompt includes examples to guide the model (e.g., Q&A pairs).
- Context window, tokens, embeddings
	- **Context window**: The max tokens (input + output) an LLM can "see" at once (e.g., GPT-4-turbo supports ~128K tokens).
	- **Tokens**: Chunks of text; 100 tokens ≈ 75 words.
	- **Embeddings**: Numeric vector representation of text for semantic search, clustering, etc.
- Handling hallucination, grounding responses
	- **Hallucination**: LLM makes up facts.
	- **Fixes**:
	    - Use RAG (Retrieval Augmented Generation) with trusted sources.
	    - Limit creativity (temperature ↓).
	    - Include citations or references in prompts.
- Streaming responses (real-time apps)
- Tools: LangChain, OpenAI API, vector DBs
	- **LangChain**: Framework for chaining LLMs with tools, memory, agents, etc.
	- **OpenAI API**: Core access to GPT models.
	- **Vector DBs**: Pinecone, Weaviate, FAISS – store embeddings for fast similarity search (RAG setups).

**Typical Interview Questions:**
- How do you integrate an LLM into a production app?
	- Use OpenAI API or LangChain. Apply RAG via vector DB. Wrap it in REST/gRPC, secure it (e.g., API keys), monitor latency and failure rates.
- How do you deal with prompt reliability and hallucinations?
	- Use retrieval-based grounding with embeddings from trusted docs. Add few-shot examples. Validate outputs. Reduce temperature. Test prompts with edge cases.
- How do you manage latency with real-time LLM use?
	- Use streaming API, partial STT/ASR + early LLM triggers (if voice). Cache embeddings + vector search. Preload prompt context. Offload post-processing.

**Your Edge:**  
You're already integrating OpenAI with Google STT and Azure TTS — mention your streaming use case, early triggering with partial STT, and flush logic. That shows real production-grade AI handling.
