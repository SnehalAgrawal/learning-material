* [[Some Concepts|Some Concepts]]
* [[Design Patterns|Design Patterns]]
* [[SystemDesign|System Design]]
 ---
### 1. **Programming Languages**
#### [[Javascript Concepts]]
- Advanced ES6+ features (destructuring, spread, async/await, etc.)
- Closures, Hoisting, Event Loop, Promises
- Functional programming (map, reduce, filter)
- Memory management and performance optimization
- Browser APIs and the DOM
- [[TypeScript fundamentals]]
#### [[Python Concepts]]
- Pythonic constructs and idioms
- Data structures: list, dict, set, tuple
- Comprehensions and generators
- AsyncIO and multiprocessing
- Flask and FastAPI basics
- Scripting and automation patterns
---
### 2. **Databases**
#### SQL (PostgreSQL / MySQL)
- Joins, indexing, normalization/denormalization
- Transactions and isolation levels
	  Transaction is a unit of work that performs one or more operations on a database, ensuring that these operations are treated as a single, indivisible operation. This means that either all operations within the transaction are completed successfully, or none of them are applied. Transactions are crucial for maintaining data integrity and consistency in a database.
- Query optimization and EXPLAIN
- CTEs and window functions
	Common Table Expressions (CTEs) allow you to create temporary, named result sets within a single SQL query, improving readability and making complex queries easier to manage
	```sql
		WITH AverageSalary AS (
		  SELECT AVG(salary) AS avg_salary FROM employees
		)
		SELECT e.name, e.salary, AS.avg_salary
		FROM employees e
		CROSS JOIN AverageSalary AS;
	```
	Window functions perform calculations across a set of rows related to the current row, without altering the result set
	```sql
		SELECT
	    e.name,
	    e.salary,
	    RANK() OVER (ORDER BY salary DESC) AS salary_rank
		FROM employees e;
	```
#### NoSQL (MongoDB, Redis)
- Document modeling and schema design
- Aggregation pipelines
- TTL indexes and capped collections
	  TTL indexes expire and remove data from normal collections based on the value of a date-typed field and a TTL value for the index
	  Capped collections serialize write operations and therefore have worse concurrent insert, update, and delete performance than non-capped collections
- Redis data structures and caching strategies
---
### 3. **DevOps / Infra / Cloud**
#### Docker
- Dockerfile best practices
- Multi-stage builds, caching layers
- Volumes and networking
- Docker Compose for local orchestration
#### Kubernetes
- Pods, Services, Deployments, ConfigMaps, Secrets
- Helm charts
- Horizontal Pod Autoscaling
- Liveness/readiness probes
- Observability (Prometheus, Grafana)
	  Prometheus is a widely-used open-source monitoring and alerting toolkit, while Grafana is an open-source platform for monitoring and observability, particularly good at visualizing time-series data
#### Cloud (Azure + GCP)
- Identity and Access Management (IAM)
- Compute (VMs, App Services, Cloud Run)
- Storage (Blob/GCS, SQL, CosmosDB, Firestore)
- CI/CD (Azure Pipelines, GCP Cloud Build)
- Event-driven architectures (Pub/Sub, Event Grid)
---
### 4. **API Design & Architecture**
- REST API principles and anti-patterns
	API anti-patterns are design choices in API development that, while commonly used, are ultimately ineffective or counterproductive, leading to issues like decreased performance, security vulnerabilities, and increased complexity. Avoiding these anti-patterns is crucial for creating high-quality, maintainable, and secure APIs.
- GraphQL (queries, mutations, schema stitching, Apollo)
- WebSockets (real-time communication, socket lifecycle)
- Socket.IO (room management, retries, reconnection)
- API versioning and documentation (Swagger/OpenAPI)
- Rate limiting, caching, authentication (JWT/OAuth)
- Websocket V/s Socker.io V/s WS
	  **WebSocket** - built in to browsers (no library needed)
	  **ws** - a node package to emulate the WebSocket support for browsers but for node and usually used as a server (though it can do both)
	  **Socket.io** - a custom protocol/library on top of WebSockets for browsers and node (must be used both on client and server)
---
### 5. **System Design (Mid to Large Scale Systems)**
- Load balancing (reverse proxies, CDN)
- Scalability (horizontal vs vertical)
- Consistency models (eventual vs strong)
- Data partitioning/sharding
	Data partitioning splits a large dataset into smaller, more manageable pieces for storage and processing, potentially across multiple servers. Keeping a master table and creating partitioning table to keep data for certain duration.
	Sharding, a form of horizontal partitioning, distributes these pieces (shards) across different database instances. Creating multiple databases and on the basic of initials or id%2 keep the data in different tables and that group of table can do their own operations.
- Event-driven and CQRS patterns
- Caching layers and strategies (CDN, Redis, local)
- Message queues (Kafka, RabbitMQ)
- Database replication and failover
- Monitoring, alerting and observability
---
### 6. **Design Patterns**
- [[SOLID principles]]
- [[MVC, MVP, MVVM architecture]] (Design patten for UI driven Apps)
- [[Design Patterns]]
	- Module
	- Singleton
	- Factory
	- Function Factory
	- Observer
	- Strategy
	- Decorator
	- Proxy
	- Command
	- Dependency Injection
	- Anti-patterns
		- God object
		- Spaghetti code
---
### 7. **DSA (Data Structures & Algorithms)** ??
- Arrays, Strings, Linked Lists
- Trees (Binary, BST, Trie)
- Graphs (BFS, DFS, Dijkstra)
- Stacks, Queues, Heaps
- HashMaps and Sets
- Sorting algorithms (quick, merge, radix)
- Dynamic programming
- Greedy algorithms
- Sliding window and two pointer techniques
---
### 8. **Additional Important Areas**
#### Leadership & Engineering Management
- Team scaling and hiring strategy
- Project planning (Agile, Scrum, Kanban)
- Architecture decision documentation (ADR)
- Tech debt management
- Conflict resolution and mentorship
#### Testing
- Unit, integration, and E2E testing
- Jest, Mocha/Chai (JavaScript), PyTest
- Test pyramid
- Mocking/stubbing strategies
#### Security
- OWASP Top 10
- Secure coding practices (input validation, authZ/authN)
- TLS/SSL, HTTPS, CORS
- Secrets management (Vault, KMS)
#### Performance & Optimization
- Frontend performance (LCP, TTI, bundle size)
- Backend profiling and bottleneck resolution
- Load testing (k6, JMeter)
---
### Recommended Sequence to Study:
1. **JavaScript & Python Core + DSA**
2. **API Architecture + Design Patterns**
3. **Database Design + Query Optimization**
4. **Docker + Kubernetes + DevOps Basics**
5. **System Design + Scalability Concepts**
6. **Cloud Services (Azure/GCP)**
7. **Testing, Security, Observability**
8. **Engineering Leadership & Project Management**
---
### System Example
1. [[High Performance Socket Connection]]
2. 