# System Design Interview Preparation Guide

A comprehensive study guide covering high-level system design, low-level design, production engineering, distributed systems, and common interview design problems. Use this as a revision + speaking-practice document.

> Recommended use: first learn the interview method, then study the core building blocks, then practice common app designs, then revise LLD and trade-offs.

## Index

- **Part 1 — Interview Method**
  - [System Design Interview Framework](#system-design-interview-framework)
  - [Requirement Gathering](#requirement-gathering)
  - [Back-of-the-Envelope Estimation](#back-of-the-envelope-estimation)
  - [Common Design Flow](#common-design-flow)
- **Part 2 — Core HLD Topics**
  - [Networking and Communication](#networking-and-communication)
  - [API Design](#api-design)
  - [Load Balancing](#load-balancing)
  - [Databases and Data Modeling](#databases-and-data-modeling)
  - [Indexes and Query Optimization](#indexes-and-query-optimization)
  - [Transactions and Consistency](#transactions-and-consistency)
  - [Replication, Partitioning, and Sharding](#replication-partitioning-and-sharding)
  - [Caching](#caching)
  - [CDN and Edge Delivery](#cdn-and-edge-delivery)
  - [Storage Systems](#storage-systems)
  - [Message Queues and Event Streaming](#message-queues-and-event-streaming)
  - [Distributed Systems Fundamentals](#distributed-systems-fundamentals)
  - [Rate Limiting and Throttling](#rate-limiting-and-throttling)
  - [Search Systems](#search-systems)
  - [Real-Time Systems](#real-time-systems)
  - [Authentication and Authorization](#authentication-and-authorization)
  - [Security](#security)
  - [Observability](#observability)
  - [Reliability and Fault Tolerance](#reliability-and-fault-tolerance)
  - [Performance Optimization](#performance-optimization)
  - [Architecture Patterns](#architecture-patterns)
  - [Backend Service Design](#backend-service-design)
  - [Frontend System Design](#frontend-system-design)
  - [Data Pipelines and Analytics](#data-pipelines-and-analytics)
  - [Recommendation Systems](#recommendation-systems)
  - [AI and ML System Design](#ai-and-ml-system-design)
- **Part 3 — Common HLD App Designs**
  - [URL Shortener](#url-shortener)
  - [Instagram](#instagram)
  - [Twitter / X](#twitter-x)
  - [Facebook / LinkedIn Feed](#facebook-linkedin-feed)
  - [WhatsApp / Messenger](#whatsapp-messenger)
  - [Slack / Discord](#slack-discord)
  - [YouTube / Netflix](#youtube-netflix)
  - [Spotify](#spotify)
  - [Amazon / Flipkart](#amazon-flipkart)
  - [Food Delivery App](#food-delivery-app)
  - [Uber / Ola](#uber-ola)
  - [Google Maps](#google-maps)
  - [BookMyShow / Ticketmaster](#bookmyshow-ticketmaster)
  - [Hotel / Flight Booking](#hotel-flight-booking)
  - [Payment Gateway](#payment-gateway)
  - [Wallet / UPI-like System](#wallet-upi-like-system)
  - [Stock Trading App](#stock-trading-app)
  - [Dropbox / Google Drive](#dropbox-google-drive)
  - [GitHub](#github)
  - [Google Docs](#google-docs)
  - [Notification System](#notification-system)
  - [Search Autocomplete](#search-autocomplete)
  - [Logging and Metrics System](#logging-and-metrics-system)
- **Part 4 — Low-Level Design**
  - [LLD Interview Framework](#lld-interview-framework)
  - [OOP Fundamentals](#oop-fundamentals)
  - [SOLID Principles](#solid-principles)
  - [Design Patterns](#design-patterns)
  - [Parking Lot](#parking-lot)
  - [Elevator System](#elevator-system)
  - [Library Management System](#library-management-system)
  - [BookMyShow LLD](#bookmyshow-lld)
  - [Splitwise](#splitwise)
  - [Chess](#chess)
  - [Snake and Ladder](#snake-and-ladder)
  - [ATM](#atm)
  - [Vending Machine](#vending-machine)
  - [Food Delivery LLD](#food-delivery-lld)
  - [Ride Sharing LLD](#ride-sharing-lld)
  - [Notification LLD](#notification-lld)
  - [Logging Framework](#logging-framework)
  - [Rate Limiter LLD](#rate-limiter-lld)
  - [Cache LLD](#cache-lld)
  - [File System LLD](#file-system-lld)
  - [Task Scheduler](#task-scheduler)
  - [Pub/Sub System](#pubsub-system)
- **Part 5 — Revision Material**
  - [System Design Trade-Offs](#system-design-trade-offs)
  - [Production Checklist](#production-checklist)
  - [Interview Question Bank](#interview-question-bank)
  - [Best Study Order](#best-study-order)

---

# Part 1 — Interview Method

## System Design Interview Framework

### Definition
System design is the process of defining the architecture, components, data flow, APIs, storage, scaling strategy, reliability model, and operational behavior of a software system. In interviews, the goal is not to produce a perfect architecture. The goal is to show structured thinking, sensible trade-offs, and practical production awareness.

### Standard Interview Flow
1. Clarify requirements: what exactly needs to be built and what is out of scope.
2. Define non-functional requirements: scale, latency, availability, consistency, durability, cost, security.
3. Estimate traffic and storage: requests per second, data size, read/write ratio, bandwidth.
4. Design APIs: core endpoints, request/response, idempotency, pagination.
5. Model data: tables/documents, indexes, keys, relationships, access patterns.
6. Draw high-level architecture: clients, gateway, services, databases, cache, queue, object storage, CDN.
7. Deep dive bottlenecks: feed fanout, search indexing, seat locking, payment idempotency, media processing, real-time messaging, etc.
8. Cover reliability and observability: retries, DLQ, monitoring, fallbacks, alerts, disaster recovery.
9. Summarize trade-offs and future improvements.

### Production Usage
- Engineering teams use system design to decide service boundaries, database choice, scaling model, operational cost, and failure behavior.
- Real production design is iterative. Teams usually start simple, measure bottlenecks, then add queues, caches, replicas, sharding, and specialized storage only when justified.
- Good design is not “use Kafka, Redis, Kubernetes everywhere.” Good design means each component solves a specific bottleneck.

### Example
For a URL shortener, a simple MVP can be one API service, one SQL database, and a cache for hot redirects. At larger scale, redirects become read-heavy, so the design adds Redis/CDN caching, database replicas, rate limiting, analytics pipeline, and abuse detection.

### Common Interview Questions
**Q1. How do you approach a system design interview?**  
I start by clarifying functional and non-functional requirements, estimate scale, design APIs and data models, propose a high-level architecture, then deep dive into bottlenecks and trade-offs. I avoid jumping to technologies before understanding access patterns.

**Q2. What does the interviewer usually evaluate?**  
They evaluate structure, requirement clarification, trade-off reasoning, scalability, reliability, database modeling, and whether the candidate can explain production failure modes clearly.

**Q3. What is the biggest mistake in system design interviews?**  
Jumping directly to tools like Kafka, Redis, or microservices without explaining the problem they solve. Another common mistake is ignoring non-functional requirements like consistency, availability, and failure handling.

## Requirement Gathering

### Definition
Requirement gathering means separating what the system must do from how the system should behave under scale and failure.

### Functional Requirements
- Core user actions: create, read, update, delete, search, follow, message, upload, pay, book, etc.
- Feature boundaries: what is in scope for the interview and what can be explicitly excluded.
- Actor types: user, admin, seller, driver, merchant, background worker, external partner.

### Non-Functional Requirements
- **Latency:** How fast should reads/writes be? Example: chat message delivery under 500 ms.
- **Availability:** How often should the system be up? Example: payment status APIs need high availability.
- **Consistency:** Is stale data acceptable? Example: social feed can be eventually consistent, wallet balance usually cannot.
- **Durability:** Can data be lost? Example: financial ledger entries must not be lost.
- **Scalability:** What happens when users, writes, or storage grow 10x?
- **Security:** Authentication, authorization, encryption, abuse prevention, privacy.
- **Cost:** A good architecture should not over-engineer early-stage traffic.

### Production Usage
Production teams convert requirements into SLOs, data contracts, service ownership, capacity planning, and incident priorities. For example, feed ranking may tolerate eventual consistency, but checkout and payment confirmation require stronger correctness guarantees.

### Example
For Instagram feed: functional requirements include uploading photos, following users, viewing feed, liking, commenting, and notifications. Non-functional requirements include low read latency, high availability, eventual consistency for likes, CDN delivery for media, and abuse/moderation controls.

### Common Interview Questions
**Q1. Why are non-functional requirements important?**  
They directly influence architecture. A read-heavy low-latency system needs caching and replicas. A strongly consistent financial system needs transactions, idempotency, and careful ledger design.

**Q2. How do you decide consistency requirements?**  
I classify data by business risk. Social likes and feeds can be eventually consistent. Payments, inventory locking, and wallet balances usually need strong consistency or carefully controlled transactional workflows.

**Q3. What should you clarify before designing?**  
Scope, user actions, expected traffic, read/write ratio, latency target, data size, consistency requirements, retention, security constraints, and whether global multi-region support is required.

## Back-of-the-Envelope Estimation

### Definition
Back-of-the-envelope estimation is rough capacity planning using assumptions about users, requests, storage, bandwidth, and growth. Interviewers use it to test whether your architecture matches the scale.

### Key Estimations
| Estimation | Formula / Thought Process | Example |
|---|---|---|
| Daily active users | Given or assumed | 10M DAU |
| Requests per second | daily requests / 86,400 | 100M requests/day ≈ 1,157 RPS average |
| Peak RPS | average RPS × 2 to 10 | 1,157 average ≈ 5k peak |
| Storage per day | objects/day × avg object size | 1M images × 2 MB = 2 TB/day |
| Bandwidth | RPS × response size | 10k RPS × 100 KB = 1 GB/s |
| Cache size | hot objects × object size | 10M hot profiles × 2 KB = 20 GB |

### Production Usage
- Capacity estimates determine number of service replicas, database partitions, cache memory, queue throughput, object storage cost, and CDN strategy.
- Estimations do not need to be exact; they need to be directionally correct.
- Always separate average load from peak load.

### Common Interview Questions
**Q1. Why estimate before drawing architecture?**  
Because architecture depends on scale. A system with 100 RPS can be a simple monolith, while 1M RPS likely requires partitioning, caching, load balancing, and asynchronous processing.

**Q2. How do you estimate storage for media-heavy systems?**  
Estimate uploads per day, average file size, number of derived versions such as thumbnails/transcoded formats, retention period, and replication overhead.

**Q3. How do you handle uncertainty in estimates?**  
I state assumptions clearly, keep numbers approximate, and design with headroom. The goal is to reason about order of magnitude, not exact billing-level precision.

## Common Design Flow

### Definition
The common design flow is a repeatable path to move from vague problem statement to a defendable architecture.

### Template
```text
1. Requirements
2. Scale assumptions
3. Core APIs
4. Data model
5. High-level architecture
6. Deep dives
7. Failure handling
8. Monitoring and operations
9. Trade-offs and summary
```

### Example Spoken Structure
“I will first clarify the core features and scale. Then I will design the main APIs and data model. After that I will propose a high-level architecture and deep dive into the main bottleneck, which for this system is feed generation. Finally, I will cover caching, failure handling, and observability.”

### Common Interview Questions
**Q1. How do you decide the deep-dive area?**  
I choose the component most central to the system’s difficulty: feed generation for Twitter, seat locking for ticket booking, matching for Uber, message ordering for chat, and idempotency for payments.

**Q2. How do you end a design interview well?**  
Summarize the architecture, call out key trade-offs, state current limitations, and mention what you would improve next if scale increased.

# Part 2 — Core HLD Topics

## Networking and Communication

### Definition
Networking topics explain how clients, services, and infrastructure communicate over the internet or internal networks.

### Subtopics
| Subtopic | What it means |
|---|---|
| **HTTP/HTTPS** | Request-response protocol used by browsers and APIs; HTTPS adds TLS encryption. |
| **TCP vs UDP** | TCP provides reliable ordered delivery; UDP is faster but does not guarantee delivery or ordering. |
| **DNS** | Maps domain names to IP addresses and enables routing users to nearby or healthy endpoints. |
| **TLS/SSL** | Encrypts communication in transit and verifies server identity. |
| **WebSockets** | Persistent bidirectional connection useful for chat, gaming, collaboration, and live dashboards. |
| **Server-Sent Events** | One-way server-to-client streaming over HTTP, useful for notifications and live updates. |
| **Long Polling** | Client repeatedly holds requests open until the server has new data. |
| **gRPC** | High-performance RPC framework using HTTP/2 and Protobuf; common for internal service communication. |
| **REST** | Resource-oriented API style using HTTP methods like GET, POST, PUT, PATCH, DELETE. |
| **GraphQL** | Client-driven query language where clients request exactly the fields they need. |
| **Reverse Proxy** | Sits in front of servers for routing, TLS termination, caching, compression, and security. |
| **API Gateway** | Central entry point for routing, auth, rate limiting, request validation, and observability. |
| **Connection Pooling** | Reuses network/database connections to avoid expensive connection setup. |
| **Timeouts and Retries** | Prevent requests from hanging forever and improve transient failure resilience. |

### Production Usage
- Use HTTP/REST for public APIs where simplicity matters.
- Use gRPC for internal high-throughput service calls.
- Use WebSockets for two-way real-time communication.
- Always configure timeouts; missing timeouts can cascade failures across services.

### Example
A chat application may use HTTPS REST APIs for login and chat history, WebSockets for real-time messages, DNS for global routing, and TLS for secure communication.

### Common Interview Questions and Answers
**Q. When would you use WebSockets instead of REST?**  
Use WebSockets when the server needs to push frequent low-latency updates to the client, such as chat, presence, live tracking, or collaborative editing. REST works better for simple request-response operations.

**Q. What is the difference between TCP and UDP?**  
TCP gives reliable ordered delivery with retransmission, making it suitable for web APIs. UDP is lower latency but unreliable, useful for real-time media, gaming, or protocols where occasional packet loss is acceptable.

**Q. Why are timeouts important?**  
Without timeouts, slow downstream services can block threads, exhaust connection pools, and cause cascading failures. Timeouts force failures to surface quickly so callers can retry, fallback, or degrade gracefully.

**Q. What is an API Gateway used for?**  
It centralizes cross-cutting concerns such as authentication, routing, rate limiting, TLS termination, request validation, and metrics before traffic reaches internal services.

---

## API Design

### Definition
API design defines how clients and services interact with the system through endpoints, payloads, errors, pagination, authentication, and versioning.

### Subtopics
| Subtopic | What it means |
|---|---|
| **REST APIs** | Expose resources using HTTP methods and status codes. |
| **GraphQL APIs** | Allow clients to request specific nested data shapes. |
| **gRPC APIs** | Strongly typed service contracts, efficient binary serialization, internal microservices. |
| **API Versioning** | Preserve backward compatibility while evolving contracts. |
| **Pagination** | Splits large result sets into pages using offset or cursor. |
| **Cursor Pagination** | Stable pagination using a cursor such as timestamp/id; better for large changing datasets. |
| **Idempotency** | Same request repeated multiple times produces the same result; critical for payments and retries. |
| **Error Handling** | Return clear error codes, messages, and retryability information. |
| **Request Validation** | Validate schema, types, size, permissions, and business constraints. |
| **Webhooks** | Server-to-server callbacks for asynchronous events. |
| **OpenAPI/Swagger** | Machine-readable API documentation and contract definition. |

### Production Usage
- Public APIs need stable contracts, versioning, auth, rate limits, and documentation.
- Internal APIs need strong ownership and backward compatibility.
- Idempotency keys are mandatory for payment/order APIs.
- Cursor pagination is preferred for feeds and timelines.

### Example
A payment API uses POST /payments with an Idempotency-Key header. If the client retries because of a timeout, the server returns the existing payment result instead of charging twice.

### Common Interview Questions and Answers
**Q. How do you design a good REST API?**  
Use resource-based URLs, correct HTTP methods, clear request/response schemas, validation, status codes, pagination, auth, and backward-compatible versioning.

**Q. Offset vs cursor pagination?**  
Offset is simple but slow and inconsistent for large changing datasets. Cursor pagination is more stable and scalable because it continues from a known item or timestamp.

**Q. What is idempotency and why is it important?**  
Idempotency ensures repeated requests have the same effect as one request. It is essential when clients retry operations like payments, booking, and order creation.

**Q. When would you use GraphQL?**  
Use GraphQL when clients need flexible nested data and want to avoid over-fetching or under-fetching, especially across multiple frontend surfaces. But it adds complexity in caching, authorization, and query cost control.

---

## Load Balancing

### Definition
Load balancing distributes traffic across multiple servers to improve availability, throughput, and fault tolerance.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Layer 4 Load Balancing** | Routes traffic based on IP/port without understanding HTTP. |
| **Layer 7 Load Balancing** | Routes HTTP traffic based on path, headers, host, cookies, or content. |
| **Round Robin** | Distributes requests sequentially across servers. |
| **Least Connections** | Sends traffic to the server with fewer active connections. |
| **Weighted Routing** | Sends more traffic to stronger or preferred servers. |
| **IP Hash** | Routes same client IP to same backend, useful for sticky sessions. |
| **Consistent Hashing** | Distributes keys across nodes with minimal remapping when nodes change. |
| **Health Checks** | Remove unhealthy instances from traffic rotation. |
| **Global Load Balancing** | Routes users to closest or healthiest region. |

### Production Usage
- Use load balancers in front of stateless services.
- Health checks prevent traffic from reaching bad instances.
- Global load balancing improves latency and disaster recovery.
- Avoid sticky sessions where possible; prefer stateless services.

### Example
An API service runs on 20 instances behind an L7 load balancer. The load balancer routes /api/* requests, terminates TLS, checks health endpoints, and removes failing instances automatically.

### Common Interview Questions and Answers
**Q. Layer 4 vs Layer 7 load balancer?**  
Layer 4 routes based on network-level information like IP and port. Layer 7 understands HTTP and can route by path, host, headers, or cookies.

**Q. Why are health checks important?**  
They allow the load balancer to stop sending traffic to unhealthy instances, improving availability during crashes, deploys, and partial failures.

**Q. What is sticky session and why avoid it?**  
Sticky sessions route the same client to the same server. They can simplify session handling but reduce flexibility and make scaling/failover harder. Prefer shared session storage or stateless JWT/session tokens.

**Q. What is consistent hashing used for?**  
It maps keys to nodes while minimizing key movement when nodes are added or removed. It is useful for distributed caches and partitioned storage.

---

## Databases and Data Modeling

### Definition
Database design defines how data is stored, related, indexed, queried, replicated, and scaled based on access patterns.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Relational Databases** | Structured tables with SQL, joins, constraints, and ACID transactions. |
| **NoSQL Databases** | Non-relational databases optimized for flexible schemas, high scale, or specific access patterns. |
| **Key-Value Stores** | Fast lookup by key; examples include Redis and DynamoDB-style systems. |
| **Document Databases** | Store JSON-like documents; useful for flexible nested data. |
| **Wide-Column Stores** | Handle high write throughput and large distributed datasets. |
| **Graph Databases** | Optimized for relationships and graph traversal. |
| **Time-Series Databases** | Optimized for timestamped metrics/events. |
| **Normalization** | Reduce duplication by splitting data into related tables. |
| **Denormalization** | Duplicate data intentionally to optimize reads. |
| **Schema Migrations** | Safe evolution of database schema over time. |
| **Soft Deletes** | Mark data deleted without physically removing it. |
| **Audit Logs** | Track who changed what and when. |

### Production Usage
- Choose SQL when relationships, transactions, and query flexibility matter.
- Choose NoSQL when access patterns are fixed and horizontal scale is more important than joins.
- Model data from access patterns, not just entities.
- Denormalize carefully when read performance justifies duplicated data.

### Example
For an e-commerce app, product catalog search may use Elasticsearch, order/payment data may use PostgreSQL, cart state may use Redis, and product images may use object storage.

### Common Interview Questions and Answers
**Q. SQL vs NoSQL?**  
SQL is better for relational data, joins, transactions, and consistency. NoSQL is better for flexible schema, very high scale, key-based access, or specialized workloads. The choice depends on access patterns and consistency needs.

**Q. What is normalization?**  
Normalization organizes data to reduce duplication and maintain consistency, usually by splitting related entities into separate tables.

**Q. Why denormalize data?**  
Denormalization improves read performance by avoiding expensive joins or cross-partition lookups, at the cost of extra storage and consistency complexity.

**Q. How do you choose a database?**  
I look at data shape, query patterns, read/write ratio, consistency needs, transaction requirements, scale, operational complexity, and team familiarity.

---

## Indexes and Query Optimization

### Definition
Indexes are auxiliary data structures that speed up reads by avoiding full scans, at the cost of extra storage and slower writes.

### Subtopics
| Subtopic | What it means |
|---|---|
| **B-tree Index** | General-purpose index for equality and range queries. |
| **Hash Index** | Fast equality lookup but not range queries. |
| **Composite Index** | Index on multiple columns; order matters. |
| **Unique Index** | Enforces uniqueness and speeds lookup. |
| **Covering Index** | Contains all columns needed by a query, avoiding table lookup. |
| **Partial Index** | Indexes only rows matching a condition. |
| **Full-Text Index** | Supports text search using tokenized inverted index. |
| **Query Plan** | Database execution strategy for a query. |
| **N+1 Query Problem** | Repeated queries in loops causing excessive database calls. |
| **Materialized View** | Precomputed query result stored for faster reads. |

### Production Usage
- Index columns used frequently in WHERE, JOIN, ORDER BY, and unique lookups.
- Avoid adding indexes blindly because every index slows writes.
- Use EXPLAIN plans to verify whether indexes are used.
- Composite index order must match query patterns.

### Example
If users log in by email, add a unique index on users.email. If orders are queried by user_id and created_at, add a composite index on (user_id, created_at DESC).

### Common Interview Questions and Answers
**Q. When should you add an index?**  
Add indexes based on actual query patterns, especially columns used in filters, joins, sorting, and uniqueness constraints.

**Q. Why can indexes slow down writes?**  
On every insert, update, or delete, the database must also update index structures, increasing write cost and storage usage.

**Q. What is a composite index?**  
An index on multiple columns. It is useful when queries filter or sort by those columns together. Column order matters because the leftmost prefix is most important.

**Q. How do you debug a slow query?**  
Check query plan, missing indexes, large scans, joins, sort operations, row estimates, N+1 patterns, and whether the query can be rewritten or precomputed.

---

## Transactions and Consistency

### Definition
Transactions group multiple operations into one logical unit, while consistency models define what users observe when data changes across replicas or services.

### Subtopics
| Subtopic | What it means |
|---|---|
| **ACID** | Atomicity, Consistency, Isolation, Durability. |
| **Atomicity** | All operations succeed or none are applied. |
| **Consistency** | Database moves from one valid state to another. |
| **Isolation** | Concurrent transactions do not interfere incorrectly. |
| **Durability** | Committed data survives crashes. |
| **Isolation Levels** | Read uncommitted, read committed, repeatable read, serializable. |
| **Strong Consistency** | Reads always return the latest committed value. |
| **Eventual Consistency** | Replicas converge over time. |
| **Read-your-writes** | A user sees their own latest writes. |
| **Distributed Transactions** | Transaction spanning multiple services/databases. |
| **Saga Pattern** | Sequence of local transactions with compensating actions. |

### Production Usage
- Use ACID transactions for orders, payments, inventory locks, and ledger updates.
- Use eventual consistency for feeds, analytics, counters, and notifications when stale data is acceptable.
- Avoid distributed transactions unless absolutely necessary; prefer sagas/outbox/event-driven workflows.
- Use idempotency and reconciliation for money flows.

### Example
In ticket booking, seat reservation and payment cannot be handled as a naive eventually consistent update. You usually lock the seat with an expiration, then confirm after payment, then release if payment fails or times out.

### Common Interview Questions and Answers
**Q. What are ACID properties?**  
Atomicity means all-or-nothing, consistency preserves valid state, isolation controls concurrent transactions, and durability ensures committed data survives crashes.

**Q. Strong vs eventual consistency?**  
Strong consistency returns the latest committed data immediately. Eventual consistency allows temporary stale reads but replicas converge later, improving availability and scalability.

**Q. What is a saga?**  
A saga coordinates a distributed workflow using multiple local transactions and compensating actions instead of one global transaction.

**Q. Where is eventual consistency acceptable?**  
Feeds, likes count, analytics, recommendations, notifications, and search indexes can often be eventually consistent because brief staleness is acceptable.

---

## Replication, Partitioning, and Sharding

### Definition
Replication copies data for availability/read scale, while partitioning and sharding split data across nodes for storage and write scalability.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Primary-Replica Replication** | Writes go to primary, reads can go to replicas. |
| **Multi-Primary Replication** | Multiple nodes accept writes; conflict resolution is harder. |
| **Synchronous Replication** | Write waits for replica acknowledgment; stronger durability but higher latency. |
| **Asynchronous Replication** | Primary returns before replicas catch up; lower latency but possible lag. |
| **Read Replicas** | Scale read traffic and reduce primary load. |
| **Partitioning** | Splits one logical dataset into smaller pieces. |
| **Sharding** | Distributes partitions across different machines. |
| **Range Sharding** | Split by ranges; can suffer hot ranges. |
| **Hash Sharding** | Distributes evenly using hash(key). |
| **Hot Partition** | One partition receives disproportionate traffic. |
| **Rebalancing** | Moving data when nodes are added/removed. |

### Production Usage
- Use replicas first for read-heavy systems before sharding.
- Shard only when one database cannot handle data size or write throughput.
- Choose shard key based on access pattern and distribution.
- Monitor replica lag and hot partitions.

### Example
A chat app can shard messages by conversation_id so all messages for a conversation are stored together and ordered efficiently.

### Common Interview Questions and Answers
**Q. Replication vs sharding?**  
Replication copies the same data to multiple nodes for availability and read scaling. Sharding splits data across nodes to scale storage and writes.

**Q. How do you choose a shard key?**  
Pick a key that matches access patterns, distributes traffic evenly, avoids hot partitions, and minimizes cross-shard queries.

**Q. What is replica lag?**  
Replica lag is the delay between data being written to primary and appearing on replicas. It can cause stale reads.

**Q. Why is sharding hard?**  
It complicates joins, transactions, rebalancing, migrations, cross-shard queries, and operational debugging.

---

## Caching

### Definition
Caching stores frequently accessed or expensive-to-compute data in faster storage to reduce latency and backend load.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Client-side Cache** | Browser/mobile stores data locally. |
| **CDN Cache** | Caches static/media content near users. |
| **Application Cache** | Service-level cache for computed or fetched data. |
| **Database Query Cache** | Cache repeated query results. |
| **Distributed Cache** | Shared cache such as Redis/Memcached across app instances. |
| **Cache-aside** | App checks cache first, loads DB on miss, then fills cache. |
| **Write-through** | Write goes to cache and DB together. |
| **Write-back** | Write goes to cache first and DB later; faster but riskier. |
| **TTL** | Time-to-live after which cached value expires. |
| **LRU/LFU** | Eviction strategies based on recency/frequency. |
| **Cache Invalidation** | Removing/updating stale cached data. |
| **Cache Stampede** | Many requests hit DB when a hot key expires. |
| **Hot Key** | One key receives extreme traffic. |

### Production Usage
- Use Redis for hot data, sessions, counters, rate limits, and short-lived computed results.
- Use CDN cache for static files, images, videos, and public content.
- Use TTL + invalidation for freshness.
- Protect against cache stampede with locks, jitter, request coalescing, and stale-while-revalidate.

### Example
Twitter timeline service may cache home timelines for active users. When a user opens the app, the service reads timeline IDs from Redis and fetches tweet details from cache or database.

### Common Interview Questions and Answers
**Q. What is cache-aside?**  
The application first checks the cache. On miss, it reads from the database, writes the result into cache, and returns it.

**Q. What is cache invalidation?**  
It is the process of removing or updating stale cached data when source data changes. It is hard because data may be cached at many layers.

**Q. How do you prevent cache stampede?**  
Use TTL jitter, request coalescing, distributed locks, stale-while-revalidate, and pre-warming for hot keys.

**Q. When should you not cache?**  
Avoid caching highly sensitive, rarely accessed, extremely volatile, or strongly consistent data unless invalidation is carefully designed.

---

## CDN and Edge Delivery

### Definition
A CDN stores and serves content from edge locations closer to users, reducing latency and origin load.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Edge Cache** | Distributed cache located near users. |
| **Static Asset Delivery** | JS, CSS, images, fonts, videos served through CDN. |
| **Origin Server** | Source server from which CDN fetches missing content. |
| **Cache-Control Headers** | HTTP headers controlling cache behavior. |
| **Signed URLs** | Temporary authenticated URLs for private assets. |
| **Image Optimization** | Resize/compress images near edge. |
| **Video Streaming** | Deliver segmented adaptive bitrate video through CDN. |
| **Geo Routing** | Route users based on geography/latency. |
| **Edge Compute** | Run small functions at edge for auth, redirects, personalization. |

### Production Usage
- Use CDN for media-heavy apps like Instagram, YouTube, and e-commerce.
- Private media should use signed URLs or signed cookies.
- Set correct cache headers to avoid stale or uncacheable assets.
- Use cache purge for urgent invalidation.

### Example
Instagram stores original images in object storage, generates multiple sizes, and serves them through a CDN so users fetch media from nearby edge nodes instead of central servers.

### Common Interview Questions and Answers
**Q. Why use a CDN?**  
To reduce latency, bandwidth cost, and origin load by serving content from edge locations closer to users.

**Q. How do you serve private files through CDN?**  
Use signed URLs/cookies with expiration and authorization checks before generating the signed access token.

**Q. What is cache-control?**  
HTTP headers that tell browsers and CDNs whether and how long to cache content.

**Q. CDN vs application cache?**  
CDN caches content near users, usually static/media content. Application cache stores data near backend services, usually dynamic or computed data.

---

## Storage Systems

### Definition
Storage systems handle persistent data such as objects, files, blocks, logs, backups, media, and archival data.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Object Storage** | Stores blobs with keys; ideal for images, videos, backups, documents. |
| **Block Storage** | Low-level disk volumes attached to compute instances. |
| **File Storage** | Hierarchical file system shared by applications. |
| **Blob Storage** | Unstructured binary object storage. |
| **Hot Storage** | Frequently accessed low-latency data. |
| **Cold Storage** | Rarely accessed cheaper archival storage. |
| **Multipart Upload** | Large files uploaded in chunks. |
| **Compression** | Reduces storage and bandwidth cost. |
| **Deduplication** | Avoid storing duplicate content. |
| **Backup and Restore** | Recover data after accidental deletion/corruption. |
| **Data Lifecycle Policy** | Move data from hot to cold storage or delete after retention period. |

### Production Usage
- Store large files in object storage, not relational database rows.
- Store metadata in database and binary content in object storage.
- Use background workers for virus scanning, compression, thumbnails, and transcoding.
- Apply lifecycle policies to control cost.

### Example
Google Drive stores file metadata in a database, file bytes in object storage, permissions in relational tables, and search index in a search engine.

### Common Interview Questions and Answers
**Q. Why not store videos in the database?**  
Databases are inefficient for large binary blobs. Object storage is cheaper, scalable, durable, and integrates well with CDN and lifecycle policies.

**Q. What is multipart upload?**  
Uploading a large file in chunks so failures can retry only failed parts and uploads can be parallelized.

**Q. Hot vs cold storage?**  
Hot storage is optimized for frequent low-latency access. Cold storage is cheaper but slower and used for archives/backups.

**Q. What metadata should be stored for files?**  
Owner, object key, size, MIME type, checksum, versions, permissions, timestamps, processing status, and retention state.

---

## Message Queues and Event Streaming

### Definition
Queues and streams decouple producers from consumers, absorb load spikes, and enable asynchronous processing.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Message Queue** | Point-to-point asynchronous task delivery. |
| **Event Stream** | Append-only event log that consumers read independently. |
| **Kafka** | Distributed event streaming platform with topics, partitions, offsets. |
| **RabbitMQ** | Broker for queues, routing, and task distribution. |
| **SQS/PubSub** | Managed cloud queue/pub-sub systems. |
| **Producer** | Service that publishes messages/events. |
| **Consumer** | Service that processes messages/events. |
| **Consumer Group** | Consumers sharing work across partitions. |
| **Offset** | Consumer position in a stream. |
| **Dead-Letter Queue** | Stores messages that repeatedly fail. |
| **Retry Queue** | Retries failed messages later. |
| **Ordering** | Guarantee messages with same key are processed in order. |
| **Idempotent Consumer** | Consumer safe to process same message multiple times. |
| **Outbox Pattern** | Store DB change and event in same transaction, publish later. |
| **CDC** | Change Data Capture reads database changes and emits events. |
| **CQRS** | Separate write model from read model. |
| **Event Sourcing** | Use append-only events as source of truth. |

### Production Usage
- Use queues for email sending, image processing, payment webhooks, notifications, and retries.
- Use streams for analytics, event-driven systems, activity feeds, and data pipelines.
- Always handle duplicates and poison messages.
- Track queue lag and processing failures.

### Example
When a user uploads a video, the API stores metadata and pushes a TranscodeVideo job to a queue. Workers process the video asynchronously, generate formats, and update status.

### Common Interview Questions and Answers
**Q. Queue vs stream?**  
A queue usually distributes tasks for processing, while a stream keeps an ordered log that multiple consumers can independently read and replay.

**Q. What is a dead-letter queue?**  
A DLQ stores messages that failed after retries so they can be inspected or manually reprocessed without blocking the main queue.

**Q. Why do consumers need to be idempotent?**  
Most queue systems provide at-least-once delivery, so duplicate messages can happen. Idempotent consumers prevent duplicate side effects.

**Q. What is the outbox pattern?**  
Write the business change and an outbox event in the same database transaction, then a separate publisher emits the event. This avoids losing events after a DB commit.

---

## Distributed Systems Fundamentals

### Definition
Distributed systems are systems where multiple machines coordinate to behave as one logical system despite network delays, failures, and partial partitions.

### Subtopics
| Subtopic | What it means |
|---|---|
| **CAP Theorem** | Under partition, choose consistency or availability. |
| **PACELC** | If partition choose A/C; else choose latency/consistency. |
| **Consensus** | Nodes agree on a value despite failures. |
| **Leader Election** | Select one node to coordinate writes/tasks. |
| **Quorum** | Require majority reads/writes for consistency. |
| **Distributed Locks** | Coordinate exclusive access across machines. |
| **Two-Phase Commit** | Distributed transaction protocol; blocking and complex. |
| **Logical Clocks** | Track event ordering without synchronized physical clocks. |
| **Vector Clocks** | Track causality across distributed nodes. |
| **Gossip Protocol** | Nodes spread state to peers gradually. |
| **Split-Brain** | Multiple nodes believe they are leader. |
| **Conflict Resolution** | Rules to merge conflicting writes. |
| **At-least-once** | May process duplicates but does not lose messages. |
| **At-most-once** | May lose messages but avoids duplicates. |
| **Exactly-once** | Hard semantic guarantee often implemented through idempotency + transactions. |

### Production Usage
- Assume networks fail, messages duplicate, clocks skew, and nodes crash.
- Prefer idempotency and retry-safe operations.
- Use managed databases/coordination systems when possible instead of building consensus yourself.
- Keep distributed transactions out of hot paths unless business-critical.

### Example
A distributed queue may deliver the same message twice if the worker processes it but crashes before acknowledging. The consumer must use idempotency keys to avoid duplicate side effects.

### Common Interview Questions and Answers
**Q. What is CAP theorem?**  
During a network partition, a distributed system must choose between consistency and availability. It cannot guarantee both perfectly under partition.

**Q. What is quorum?**  
A quorum requires a majority or configured number of nodes to acknowledge reads/writes, improving consistency and fault tolerance.

**Q. Why are distributed locks dangerous?**  
They can fail due to clock skew, lease expiry, network partitions, and client pauses. Use them carefully and prefer database constraints or idempotency where possible.

**Q. Exactly-once processing: real or myth?**  
True exactly-once is difficult in distributed systems. In practice, systems achieve exactly-once effects using idempotency, deduplication keys, transactions, and careful offset management.

---

## Rate Limiting and Throttling

### Definition
Rate limiting controls how many requests a user, IP, API key, or service can make in a time window to protect reliability and prevent abuse.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Fixed Window** | Count requests in fixed time buckets; simple but bursty at boundaries. |
| **Sliding Window** | Smoother limit using rolling window. |
| **Token Bucket** | Tokens refill over time; allows controlled bursts. |
| **Leaky Bucket** | Processes requests at constant rate, smoothing bursts. |
| **Per-user Limit** | Limit by authenticated user. |
| **Per-IP Limit** | Limit anonymous or suspicious traffic. |
| **Global Limit** | Protect entire service from overload. |
| **Distributed Rate Limit** | Consistent limit across multiple app instances. |
| **Quota** | Longer-term usage allowance. |
| **Retry-After Header** | Tell client when to retry. |

### Production Usage
- Implement rate limits at API gateway and service layer for sensitive actions.
- Use Redis or dedicated gateway counters for distributed rate limiting.
- Apply stricter limits to login, signup, OTP, payments, scraping-prone APIs, and expensive AI endpoints.
- Return 429 with Retry-After when limited.

### Example
Login API may allow 5 failed attempts per user per 15 minutes and 20 attempts per IP per hour to reduce credential stuffing.

### Common Interview Questions and Answers
**Q. Token bucket vs leaky bucket?**  
Token bucket allows bursts up to bucket size while enforcing average rate. Leaky bucket smooths traffic at a fixed output rate.

**Q. How do you implement distributed rate limiting?**  
Use a shared store like Redis with atomic increments/expiration or a gateway-level rate limiter. For very high scale, shard counters or use approximate algorithms.

**Q. What status code for rate limit?**  
HTTP 429 Too Many Requests, usually with Retry-After header.

**Q. Where should rate limiting be applied?**  
At multiple layers: edge/WAF for abusive traffic, API gateway for global/user limits, and service-level for business-specific expensive operations.

---

## Search Systems

### Definition
Search systems index and retrieve documents, products, users, locations, or content based on text, filters, ranking, and relevance.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Full-Text Search** | Search text using tokenized inverted indexes. |
| **Inverted Index** | Maps terms to documents containing them. |
| **Elasticsearch/OpenSearch** | Distributed search engines for text, filters, aggregations. |
| **Tokenization** | Break text into searchable tokens. |
| **Stemming** | Reduce words to root forms. |
| **Synonyms** | Map related terms for recall. |
| **Fuzzy Search** | Handle typos and approximate matches. |
| **Autocomplete/Typeahead** | Return suggestions as user types. |
| **Faceted Search** | Filter by categories/attributes. |
| **Geo Search** | Search by location/distance. |
| **Reindexing** | Rebuild search index from source data. |
| **Hybrid Search** | Combine keyword and vector/semantic search. |
| **Vector Search** | Search by embedding similarity. |
| **Reranking** | Reorder candidates using stronger relevance model. |

### Production Usage
- Use a database as source of truth and search index as eventually consistent read model.
- Use asynchronous indexing via queue/CDC.
- Monitor index freshness and failed indexing jobs.
- For e-commerce, combine lexical relevance, filters, business ranking, personalization, and availability.

### Example
Amazon search indexes product titles, descriptions, categories, prices, inventory status, reviews, and seller metadata. Ranking combines text match, popularity, conversion rate, price, and personalization.

### Common Interview Questions and Answers
**Q. Why not use SQL LIKE for large search?**  
LIKE queries do not scale well for rich full-text relevance, typo tolerance, ranking, faceting, and distributed search. Use a search engine for large-scale search.

**Q. What is an inverted index?**  
A structure mapping each term to the list of documents containing it, enabling fast full-text lookup.

**Q. How do you keep search index updated?**  
Emit events on data changes or use CDC, process updates asynchronously, retry failures, and periodically reindex from the source of truth.

**Q. What is hybrid search?**  
A combination of keyword search and semantic/vector search to improve both exact matching and conceptual matching.

---

## Real-Time Systems

### Definition
Real-time systems deliver updates to users quickly, often through persistent connections or push-based communication.

### Subtopics
| Subtopic | What it means |
|---|---|
| **WebSockets** | Persistent two-way communication. |
| **SSE** | Server-to-client streaming. |
| **Long Polling** | Fallback for near-real-time updates. |
| **Push Notifications** | Mobile/browser notifications through platform services. |
| **Presence** | Track online/offline state. |
| **Typing Indicators** | Ephemeral real-time events. |
| **Message Ordering** | Preserve logical order within conversation/channel. |
| **Pub/Sub** | Broadcast events to interested subscribers. |
| **Fan-out** | Send one event to many recipients. |
| **CRDTs** | Conflict-free data structures for collaborative editing. |
| **Operational Transformation** | Transform concurrent edits in collaborative docs. |

### Production Usage
- Use WebSockets for interactive two-way systems.
- Use pub/sub to route events across multiple WebSocket servers.
- Store durable events in database/log; do not rely only on in-memory sockets.
- Presence is often eventually consistent and TTL-based.

### Example
WhatsApp stores messages durably, delivers online messages over persistent connections, queues offline messages, and sends push notifications when users are offline.

### Common Interview Questions and Answers
**Q. WebSockets vs push notifications?**  
WebSockets require the app to be connected and allow real-time two-way communication. Push notifications work when the app is offline/backgrounded but are not suitable for high-frequency interactive updates.

**Q. How do you scale WebSocket servers?**  
Keep connection state on WebSocket servers, use load balancing, store user-to-connection mapping in distributed store, and use pub/sub to route messages to the right server.

**Q. How do you handle offline messages?**  
Persist messages first, mark delivery state, deliver when user reconnects, and optionally trigger push notification.

**Q. How is presence usually implemented?**  
Clients heartbeat periodically; server stores presence with TTL. If heartbeat stops, user is considered offline after expiry.

---

## Authentication and Authorization

### Definition
Authentication verifies who the user is; authorization determines what the user is allowed to access or do.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Registration/Login** | Create and authenticate user accounts. |
| **Password Hashing** | Store salted password hashes using secure algorithms. |
| **Session Auth** | Server stores session and client stores session cookie. |
| **JWT** | Signed token containing claims; often stateless. |
| **OAuth** | Delegated authorization with third-party providers. |
| **SSO** | Single sign-on across multiple apps. |
| **Refresh Tokens** | Longer-lived tokens used to get new access tokens. |
| **Access Tokens** | Short-lived tokens for API access. |
| **API Keys** | Identify calling applications/services. |
| **RBAC** | Role-based access control. |
| **ABAC** | Attribute-based access control. |
| **Scopes** | Fine-grained permission boundaries. |
| **MFA** | Extra verification factor. |
| **Password Reset** | Secure account recovery flow. |

### Production Usage
- Use HTTPS-only secure cookies for session-based web auth.
- Hash passwords with strong algorithms and never store plaintext.
- Keep access tokens short-lived and refresh tokens protected.
- Centralize permission checks for sensitive operations.

### Example
A Google Drive-like system checks authentication first, then authorization based on file ownership, sharing permissions, organization policy, and link visibility.

### Common Interview Questions and Answers
**Q. Authentication vs authorization?**  
Authentication verifies identity. Authorization checks permissions for a resource or action.

**Q. Session vs JWT?**  
Sessions store state server-side and are easier to revoke. JWTs are self-contained and scalable but revocation and claim freshness are harder.

**Q. What is RBAC?**  
Role-based access control grants permissions through roles like admin, editor, viewer. It is simple and common for business apps.

**Q. How should passwords be stored?**  
Store salted hashes using strong password hashing algorithms, never plaintext. Add rate limiting and MFA for sensitive systems.

---

## Security

### Definition
Security protects systems from unauthorized access, data leakage, abuse, injection attacks, fraud, and operational compromise.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Encryption in Transit** | Protect data moving over networks using TLS. |
| **Encryption at Rest** | Protect stored data using encryption keys. |
| **Secrets Management** | Store credentials securely outside source code. |
| **Input Validation** | Reject malformed, unexpected, or dangerous input. |
| **SQL Injection** | Attacker manipulates SQL queries through input. |
| **XSS** | Attacker injects malicious scripts into pages. |
| **CSRF** | Attacker tricks browser into unwanted authenticated request. |
| **SSRF** | Server is tricked into making internal network requests. |
| **CORS** | Controls cross-origin browser requests. |
| **DDoS Protection** | Absorb/filter massive traffic floods. |
| **Bot Detection** | Detect automation and abuse. |
| **PII Protection** | Minimize, encrypt, and restrict personal data. |
| **Audit Logging** | Record security-relevant actions. |
| **Least Privilege** | Grant minimum required permissions. |
| **Secure File Uploads** | Validate type, scan files, isolate storage. |

### Production Usage
- Use parameterized queries to prevent SQL injection.
- Use output escaping and CSP to reduce XSS.
- Keep secrets in a secret manager, not environment dumps or code.
- Log sensitive actions but avoid logging passwords, tokens, or PII unnecessarily.

### Example
A file upload system validates MIME type, size, and extension, stores uploaded files outside the web root, scans for malware, and serves them through signed URLs.

### Common Interview Questions and Answers
**Q. How do you prevent SQL injection?**  
Use parameterized queries/prepared statements, ORM-safe APIs, input validation, and least-privilege database users.

**Q. How do you protect APIs from abuse?**  
Authentication, authorization, rate limiting, quotas, bot detection, WAF, input validation, anomaly detection, and audit logs.

**Q. What is least privilege?**  
Every user/service gets only the minimum permissions required. This reduces blast radius if credentials are compromised.

**Q. How do you handle sensitive data?**  
Minimize collection, encrypt in transit/rest, restrict access, mask logs, define retention, support deletion, and audit access.

---

## Observability

### Definition
Observability is the ability to understand system behavior from logs, metrics, traces, alerts, and dashboards.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Logging** | Record structured events for debugging and audit. |
| **Metrics** | Numeric measurements over time: latency, RPS, errors. |
| **Tracing** | Follow a request across services. |
| **Distributed Tracing** | Trace calls through microservices with correlation IDs. |
| **Alerting** | Notify humans when SLOs or thresholds are violated. |
| **Dashboards** | Visualize system health and business metrics. |
| **Health Checks** | Endpoints for service readiness/liveness. |
| **SLO/SLI/SLA** | Reliability targets, indicators, and external commitments. |
| **Error Budget** | Allowed unreliability before prioritizing reliability work. |
| **Queue Lag** | Measure backlog in async systems. |
| **Cache Hit Ratio** | Measure cache effectiveness. |

### Production Usage
- Track golden signals: latency, traffic, errors, saturation.
- Use correlation IDs across services.
- Alert on user-impacting symptoms, not only infrastructure internals.
- Use dashboards for deploys, incidents, and capacity planning.

### Example
A payment service tracks payment success rate, provider latency, idempotency conflicts, webhook failures, queue lag, DB errors, and reconciliation mismatches.

### Common Interview Questions and Answers
**Q. Logs vs metrics vs traces?**  
Logs explain events, metrics show numerical trends, and traces show request path across services.

**Q. What are golden signals?**  
Latency, traffic, errors, and saturation. They give a quick view of service health.

**Q. What is an SLO?**  
A service-level objective is an internal reliability target, such as 99.9% of requests under 300 ms.

**Q. Why use correlation IDs?**  
They connect logs and traces from the same request across services, making debugging much easier.

---

## Reliability and Fault Tolerance

### Definition
Reliability engineering ensures systems continue operating correctly despite failures, overload, dependency issues, and disasters.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Redundancy** | Multiple components to avoid single point of failure. |
| **Failover** | Switch to backup component/region after failure. |
| **Circuit Breaker** | Stop calling failing dependency temporarily. |
| **Retries** | Repeat failed transient operations. |
| **Exponential Backoff** | Increase retry delay to reduce pressure. |
| **Jitter** | Randomize retries to avoid synchronized spikes. |
| **Bulkhead** | Isolate resources so one failure does not sink all services. |
| **Graceful Degradation** | Serve reduced functionality during partial failure. |
| **Fallback** | Alternative behavior when dependency fails. |
| **Disaster Recovery** | Recover from region/data loss. |
| **RPO/RTO** | Recovery point/time objectives. |
| **Chaos Testing** | Intentionally test failure scenarios. |
| **Load Shedding** | Drop lower-priority traffic under overload. |

### Production Usage
- Use timeouts, retries with backoff, and circuit breakers for external calls.
- Design idempotent retry-safe operations.
- Use multi-AZ databases and backups.
- Prefer graceful degradation over total outage.

### Example
If recommendation service fails, an e-commerce homepage can show trending products instead of failing the full page.

### Common Interview Questions and Answers
**Q. What is graceful degradation?**  
The system continues with reduced functionality instead of failing completely. Example: show cached feed if ranking service is unavailable.

**Q. Why can retries be dangerous?**  
Retries can amplify load and worsen outages if not bounded with timeouts, backoff, jitter, and idempotency.

**Q. What is circuit breaker?**  
A pattern that stops calls to a failing dependency temporarily, allowing it to recover and preventing cascading failures.

**Q. RPO vs RTO?**  
RPO is how much data loss is acceptable. RTO is how quickly service must be restored.

---

## Performance Optimization

### Definition
Performance optimization reduces latency, increases throughput, and improves resource efficiency across frontend, backend, database, network, and storage layers.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Latency Optimization** | Reduce time taken per request. |
| **Throughput Optimization** | Increase work handled per unit time. |
| **Connection Pooling** | Reuse connections. |
| **Batching** | Process multiple operations together. |
| **Async Processing** | Move slow work out of request path. |
| **Compression** | Reduce network payload size. |
| **Lazy Loading** | Load data/assets only when needed. |
| **Precomputation** | Compute expensive results ahead of time. |
| **Materialized Views** | Stored precomputed query results. |
| **Read Replicas** | Offload read traffic. |
| **Hot Path Optimization** | Optimize the most frequent or latency-sensitive path. |
| **Profiling** | Measure where time/resources are spent. |

### Production Usage
- Measure before optimizing.
- Optimize hot paths first.
- Move non-critical work to background jobs.
- Use caching, indexes, pagination, and CDN for common performance wins.

### Example
A feed service precomputes home timelines for active users so read requests do not need to rank thousands of posts synchronously.

### Common Interview Questions and Answers
**Q. How do you reduce API latency?**  
Use caching, database indexes, connection pooling, smaller payloads, async background jobs, parallel calls where safe, CDN, and avoid unnecessary network hops.

**Q. What is precomputation?**  
Compute expensive results before they are requested, such as timelines, recommendations, reports, or materialized views.

**Q. How do you optimize database performance?**  
Use indexes, query plan analysis, pagination, denormalization where needed, read replicas, partitioning, and avoid N+1 queries.

**Q. What is the first step in performance optimization?**  
Measure and profile. Without measurement, optimization is guesswork.

---

## Architecture Patterns

### Definition
Architecture patterns are reusable ways to structure services, data flow, dependencies, and deployment units.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Monolith** | Single deployable application. |
| **Modular Monolith** | Single deployable app with strong internal module boundaries. |
| **Microservices** | System split into independently deployable services. |
| **SOA** | Service-oriented architecture with shared services. |
| **Event-Driven Architecture** | Services communicate through events. |
| **Serverless** | Run functions/services without managing servers directly. |
| **Layered Architecture** | Controller/service/repository style separation. |
| **Hexagonal Architecture** | Domain isolated from external adapters. |
| **Clean Architecture** | Business rules independent of frameworks/infrastructure. |
| **CQRS** | Separate command/write model from query/read model. |
| **Event Sourcing** | Store changes as events. |
| **Saga** | Distributed workflow with compensation. |
| **Strangler Fig** | Gradually replace legacy system. |
| **BFF** | Backend for Frontend tailored to client needs. |
| **Sidecar** | Helper process alongside service for proxy/logging/security. |

### Production Usage
- Start with a modular monolith when team/product is small.
- Move to microservices when independent scaling, team ownership, or deployment boundaries justify complexity.
- Use event-driven design for decoupling and async workflows.
- Avoid microservices without observability and operational maturity.

### Example
An e-commerce platform may start as a modular monolith, then split catalog, checkout, payment, notification, and search into services as traffic and team size grow.

### Common Interview Questions and Answers
**Q. Monolith vs microservices?**  
Monolith is simpler to develop, deploy, and debug. Microservices enable independent scaling and ownership but add network, data consistency, deployment, and observability complexity.

**Q. What is CQRS?**  
Command Query Responsibility Segregation separates writes from reads, allowing different models optimized for mutation and querying.

**Q. What is event sourcing?**  
Persist state changes as an append-only sequence of events and rebuild current state from those events.

**Q. When should you split a service?**  
When there is clear domain ownership, independent scaling need, frequent independent deployments, or the module has become a bottleneck. Do not split only because microservices sound modern.

---

## Backend Service Design

### Definition
Backend service design covers how services expose APIs, own data, process jobs, handle dependencies, deploy safely, and operate in production.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Service Boundaries** | Define ownership around business capabilities. |
| **Internal APIs** | Service-to-service contracts. |
| **External APIs** | Public/client-facing contracts. |
| **Background Workers** | Async processing outside request path. |
| **Cron Jobs** | Scheduled recurring tasks. |
| **Job Scheduling** | Manage delayed/retried workloads. |
| **Service Discovery** | Find service instances dynamically. |
| **Config Management** | Manage environment-specific settings. |
| **Feature Flags** | Enable/disable features safely. |
| **Deployments** | Rolling, blue-green, canary. |
| **Rollback** | Revert bad deploy quickly. |
| **Stateless Service** | No local durable state; easy to scale horizontally. |

### Production Usage
- Keep services stateless where possible.
- Own data per service to reduce coupling.
- Use feature flags and canary deploys for risk control.
- Move slow work such as email, media processing, and analytics to workers.

### Example
A notification service exposes APIs to create notification requests, stores notification state, processes delivery in workers, retries failures, and tracks delivery metrics.

### Common Interview Questions and Answers
**Q. What makes a good service boundary?**  
A good boundary maps to business capability, has clear ownership, minimizes chatty dependencies, and owns its data model.

**Q. Why prefer stateless services?**  
They scale horizontally, recover easily, and can be replaced without losing local state.

**Q. What are feature flags used for?**  
Safe rollout, A/B testing, quick rollback, enabling features per user/team, and decoupling deploy from release.

**Q. Why use background workers?**  
To keep user-facing requests fast by moving slow, retryable, or non-critical tasks out of the synchronous path.

---

## Frontend System Design

### Definition
Frontend system design focuses on client architecture, performance, caching, state management, rendering, offline behavior, and API interaction.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Client-Server Architecture** | Frontend communicates with backend APIs. |
| **Browser Cache** | Cache static assets and responses. |
| **Local Storage** | Simple persistent key-value storage. |
| **Session Storage** | Tab/session-scoped storage. |
| **IndexedDB** | Structured browser database for offline data. |
| **Offline Support** | Allow partial function without network. |
| **Optimistic UI** | Update UI before server confirms. |
| **Infinite Scroll** | Load paginated content as user scrolls. |
| **Asset Optimization** | Bundle splitting, compression, image optimization. |
| **State Management** | Manage local/global/server state. |
| **Debouncing/Throttling** | Control high-frequency user events. |
| **Progressive Rendering** | Render useful content early. |
| **Error Boundaries** | Contain UI failures. |
| **API Batching** | Reduce network requests. |

### Production Usage
- Use CDN for static assets.
- Split bundles by route and lazy-load heavy components.
- Use optimistic UI carefully with rollback.
- Cache server state with invalidation rules.
- Support offline mode for apps like docs, notes, and email drafts.

### Example
A social feed uses cursor pagination, image lazy loading, skeleton UI, optimistic like updates, local cache, and retry on failed actions.

### Common Interview Questions and Answers
**Q. What is optimistic UI?**  
The UI updates immediately assuming success, then either confirms or rolls back if the server fails. It improves perceived speed but needs conflict/error handling.

**Q. How do you improve frontend performance?**  
Use code splitting, CDN, compression, caching, lazy loading, optimized images, avoid unnecessary re-renders, and minimize API round trips.

**Q. LocalStorage vs IndexedDB?**  
LocalStorage is simple synchronous key-value storage for small data. IndexedDB is async, larger, structured, and better for offline-first apps.

**Q. How do you design infinite scroll?**  
Use cursor pagination, loading states, duplicate prevention, virtualization for large lists, and preserve scroll position.

---

## Data Pipelines and Analytics

### Definition
Data pipelines collect, process, transform, store, and serve event data for analytics, reporting, ML, monitoring, and product decisions.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Event Tracking** | Capture user/system actions. |
| **Clickstream** | User interaction events. |
| **ETL** | Extract, transform, load. |
| **ELT** | Extract, load, transform in warehouse. |
| **Data Lake** | Store raw large-scale data cheaply. |
| **Data Warehouse** | Structured analytics store. |
| **Batch Processing** | Process large data periodically. |
| **Stream Processing** | Process events continuously. |
| **Metrics Aggregation** | Roll up raw data into metrics. |
| **Funnel Analysis** | Track user conversion steps. |
| **A/B Testing** | Compare product variants statistically. |
| **Experiment Tracking** | Measure experiments and guardrails. |

### Production Usage
- Use event schemas and versioning.
- Separate analytics from transactional workloads.
- Validate event quality and deduplicate.
- Use batch for historical reports and streams for near-real-time dashboards.

### Example
An e-commerce app emits product_viewed, added_to_cart, checkout_started, payment_success events. Pipelines aggregate conversion funnels and feed recommendation models.

### Common Interview Questions and Answers
**Q. Batch vs stream processing?**  
Batch processes data periodically in chunks. Streaming processes events continuously with lower latency but greater operational complexity.

**Q. Why use a data warehouse?**  
For analytical queries across large historical datasets without overloading production databases.

**Q. How do you ensure event quality?**  
Define schemas, validate producers, version events, monitor missing/invalid fields, deduplicate, and reconcile against source systems.

**Q. What is funnel analysis?**  
Measuring how users move through steps such as view product → add to cart → checkout → payment success to identify drop-offs.

---

## Recommendation Systems

### Definition
Recommendation systems select and rank items for users using behavior, content, collaborative signals, embeddings, business rules, and feedback loops.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Candidate Generation** | Fetch possible items from many sources. |
| **Ranking** | Order candidates by predicted relevance/value. |
| **Personalization** | Adapt results to user behavior/preferences. |
| **Collaborative Filtering** | Recommend based on similar users/items. |
| **Content-Based Filtering** | Recommend based on item/user features. |
| **Embeddings** | Dense vector representations for similarity. |
| **Feature Store** | Central store for ML features. |
| **Real-Time Signals** | Recent clicks, searches, purchases. |
| **Batch Training** | Train models periodically. |
| **Online Serving** | Serve recommendations with low latency. |
| **Cold Start** | Handle new users/items. |
| **Diversity/Freshness** | Avoid repetitive stale recommendations. |
| **A/B Testing** | Measure recommendation impact. |

### Production Usage
- Use multi-stage architecture: candidate generation → filtering → ranking → diversification.
- Precompute heavy candidate sets offline and personalize online.
- Track feedback events and guardrail metrics.
- Fallback to popular/trending items for cold start.

### Example
YouTube recommendations combine subscriptions, watch history, similar videos, trending content, freshness, and ranking models optimized for watch time and satisfaction.

### Common Interview Questions and Answers
**Q. How do recommendation systems work at high level?**  
They generate candidate items, filter invalid items, rank them using user/item/context features, diversify results, serve them with low latency, and learn from feedback.

**Q. How do you solve cold start?**  
Use onboarding preferences, demographics/context, popular items, trending items, content metadata, and exploration strategies until enough behavior is collected.

**Q. Why separate candidate generation and ranking?**  
Candidate generation retrieves a manageable set from millions of items; ranking uses more expensive models/features on that smaller set.

**Q. What metrics matter?**  
CTR, conversion, watch time, retention, satisfaction, diversity, freshness, latency, and business guardrails such as spam or revenue impact.

---

## AI and ML System Design

### Definition
AI/ML system design covers model serving, embeddings, vector search, RAG, evaluation, inference latency, cost, guardrails, monitoring, and feedback loops.

### Subtopics
| Subtopic | What it means |
|---|---|
| **Model Serving** | Expose ML/LLM models through APIs for inference. |
| **Feature Store** | Consistent features for training and serving. |
| **Embedding Generation** | Convert text/images/items into vectors. |
| **Vector Database** | Store/search embeddings by similarity. |
| **RAG** | Retrieve relevant context and pass it to an LLM. |
| **Prompt Orchestration** | Manage system prompts, tools, schemas, retries. |
| **LLM Gateway** | Centralize model routing, logging, rate limits, cost controls. |
| **Model Routing** | Choose model by task complexity/cost/latency. |
| **Batch Inference** | Run predictions offline at scale. |
| **Real-Time Inference** | Low-latency prediction on request path. |
| **Evaluation Pipeline** | Measure quality, safety, latency, cost, regressions. |
| **Guardrails** | Validate inputs/outputs and enforce safety/business rules. |
| **Caching LLM Responses** | Avoid repeated expensive model calls. |
| **Hallucination Reduction** | Use retrieval, citations, validation, constrained outputs. |

### Production Usage
- Use simpler deterministic logic before LLM calls when possible.
- Cache embeddings and repeated LLM responses.
- Evaluate with golden datasets before changing prompts/models.
- Add guardrails, schema validation, rate limits, fallbacks, and human approval for high-risk actions.

### Example
A support chatbot uses RAG: embed user query, retrieve relevant docs, rerank results, generate answer with citations, validate output, and fallback to human support when confidence is low.

### Common Interview Questions and Answers
**Q. What is RAG?**  
Retrieval-Augmented Generation retrieves relevant external context and gives it to an LLM so answers are grounded in source data.

**Q. How do you reduce LLM hallucinations?**  
Use high-quality retrieval, citations, constrained output schemas, verification steps, refusal behavior, evals, and avoid asking the model to guess when context is missing.

**Q. How do you control LLM cost?**  
Use routing to cheaper models, caching, batching, prompt compression, retrieval pruning, rate limits, and avoid LLM calls for deterministic tasks.

**Q. What should be monitored in ML systems?**  
Latency, cost, error rate, model quality, drift, feedback metrics, safety violations, retrieval quality, and business KPIs.

---

# Part 3 — Common HLD App Designs

Each app design below follows the same interview structure: requirements, APIs, data model, architecture, scaling concerns, production risks, and interview Q&A.

## URL Shortener

### Functional Requirements / Features
- Create short URLs
- Redirect short URL to long URL
- Custom aliases
- Expiry
- Analytics
- Rate limiting
- Abuse detection

### Example APIs
- `POST /urls`
- `GET /{shortCode}`
- `GET /urls/{id}/analytics`

### Core Data Model
- `url_mappings(id, short_code, long_url, user_id, expires_at, created_at)`
- `click_events(short_code, timestamp, referrer, geo, user_agent)`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Short code generation
- Hot redirect caching
- Database uniqueness
- Analytics ingestion
- Abuse/spam prevention

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you generate unique short codes?**  
Use base62 encoding of an auto-increment/sequence ID or generate random codes with uniqueness checks. Base62 is deterministic and compact; random codes avoid predictability but require collision handling.

**Q. How do you scale redirects?**  
Cache hot mappings in Redis/CDN, use read replicas, keep redirect path minimal, and process analytics asynchronously.

**Q. How do you prevent duplicate redirects or broken links?**  
Use unique constraints on short_code, validate long URLs, handle expiration, and return proper 404/410 responses.

---

## Instagram

### Functional Requirements / Features
- User profiles
- Follow/unfollow
- Photo/video upload
- Feed
- Stories
- Reels
- Likes
- Comments
- Search
- Explore
- Notifications
- DMs

### Example APIs
- `POST /media`
- `GET /feed?cursor=`
- `POST /follow`
- `POST /media/{id}/like`
- `GET /users/{id}/profile`

### Core Data Model
- `users`
- `media`
- `follows`
- `likes`
- `comments`
- `feed_items`
- `notifications`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Media storage + CDN
- Feed generation
- Fanout strategy
- Ranking
- Hot celebrity accounts
- Moderation
- Search indexing

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. Fanout on write vs fanout on read for Instagram feed?**  
Fanout on write precomputes feed entries when someone posts, making reads fast but writes expensive for high-follower users. Fanout on read computes feed at request time, making writes cheap but reads slower. A hybrid approach is common.

**Q. Where do you store images/videos?**  
Store binaries in object storage, metadata in DB, derived thumbnails/transcodes generated asynchronously, and deliver through CDN.

**Q. How do likes/comments scale?**  
Use write-optimized storage, cache counts, update counters asynchronously, and accept eventual consistency for displayed counts.

---

## Twitter / X

### Functional Requirements / Features
- Tweet creation
- Timeline
- Follow graph
- Like
- Retweet
- Reply
- Hashtags
- Trending
- Search
- Notifications
- DMs
- Media upload

### Example APIs
- `POST /tweets`
- `GET /timeline/home?cursor=`
- `POST /users/{id}/follow`
- `POST /tweets/{id}/retweet`

### Core Data Model
- `users`
- `tweets`
- `follows`
- `timeline_entries`
- `likes`
- `retweets`
- `hashtags`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Home timeline generation
- Celebrity problem
- Fanout write/read hybrid
- Search freshness
- Trending computation
- Abuse/rate limits

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. What is the celebrity user problem?**  
A user with millions of followers makes fanout-on-write very expensive. Handle by not fully pushing celebrity posts; instead pull them at read time or use special queues.

**Q. How do you design timeline generation?**  
Use hybrid fanout: precompute timelines for normal accounts, pull high-follower accounts at read time, cache timelines, and rank/filter before serving.

**Q. How do you compute trending topics?**  
Aggregate hashtag/keyword events in streaming windows, normalize by baseline, remove spam, and rank by velocity and region.

---

## Facebook / LinkedIn Feed

### Functional Requirements / Features
- Post creation
- Friend/connection graph
- Feed ranking
- Comments
- Reactions
- Shares
- Groups/pages
- Privacy settings
- Ads/recommendations

### Example APIs
- `POST /posts`
- `GET /feed`
- `POST /connections`
- `POST /posts/{id}/reaction`

### Core Data Model
- `users`
- `connections`
- `posts`
- `feed_candidates`
- `privacy_rules`
- `reactions`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Graph queries
- Privacy enforcement
- Feed ranking
- Ads insertion
- Recommendation signals
- Eventual consistency

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. What makes LinkedIn/Facebook feed harder than Twitter?**  
Richer privacy rules, multiple content types, ranking signals, groups/pages, ads, and relationship strengths make candidate generation and filtering more complex.

**Q. Where should privacy be enforced?**  
At write and read paths where relevant, but final enforcement must happen before serving content. Privacy bugs are critical.

**Q. How do you rank feed items?**  
Generate candidates from network/activity, filter privacy, score by relevance/freshness/engagement, diversify, insert ads if applicable, then cache.

---

## WhatsApp / Messenger

### Functional Requirements / Features
- One-to-one chat
- Group chat
- Message delivery
- Ordering
- Read receipts
- Typing indicators
- Presence
- Offline delivery
- Media messages
- Push notifications
- E2E encryption

### Example APIs
- `POST /messages`
- `GET /conversations/{id}/messages`
- `POST /messages/{id}/ack`
- `WS /connect`

### Core Data Model
- `users`
- `devices`
- `conversations`
- `conversation_members`
- `messages`
- `delivery_receipts`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- WebSocket connection management
- Offline queue
- Ordering per conversation
- Push notifications
- Multi-device sync
- Encryption

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you guarantee message ordering?**  
Assign monotonically increasing sequence numbers per conversation or use server timestamps plus per-conversation ordering rules. Store durably before delivery.

**Q. How do you deliver messages to offline users?**  
Persist messages, track undelivered state per recipient/device, deliver when they reconnect, and send push notifications.

**Q. How do you scale WebSocket connections?**  
Use many connection servers, store user-to-server mapping in distributed store, and route messages through pub/sub.

---

## Slack / Discord

### Functional Requirements / Features
- Workspaces/servers
- Channels
- Threads
- Mentions
- Reactions
- Files
- Notifications
- Search
- Permissions
- Bots/integrations

### Example APIs
- `POST /messages`
- `GET /channels/{id}/messages`
- `POST /reactions`
- `GET /search?q=`

### Core Data Model
- `workspaces`
- `channels`
- `members`
- `messages`
- `threads`
- `reactions`
- `permissions`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Channel fanout
- Permission checks
- Search indexing
- Notification rules
- Retention policies
- Bots/webhooks

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. Slack vs WhatsApp design difference?**  
Slack has workspace/channel permissions, search, retention, integrations, and threads; WhatsApp focuses more on private chat, device sync, encryption, and mobile delivery.

**Q. How do mentions work?**  
Parse message, identify mentioned users/groups, check permissions, create notification events, and respect user notification preferences.

**Q. How is message search implemented?**  
Index message content asynchronously into search engine with workspace/channel permissions used during query filtering.

---

## YouTube / Netflix

### Functional Requirements / Features
- Video upload
- Transcoding
- Streaming
- Adaptive bitrate
- Recommendations
- Watch history
- Comments/likes
- Subscriptions
- Search
- Content moderation

### Example APIs
- `POST /videos`
- `GET /videos/{id}/playback`
- `POST /watch-events`
- `GET /recommendations`

### Core Data Model
- `videos`
- `video_assets`
- `transcoding_jobs`
- `users`
- `watch_history`
- `comments`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Large object upload
- Async transcoding
- CDN delivery
- ABR streaming
- Recommendation ranking
- Copyright/moderation

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. Why transcode videos?**  
Different devices and network conditions need different resolutions/bitrates/codecs. Transcoding creates multiple renditions for adaptive streaming.

**Q. How does adaptive bitrate streaming work?**  
Video is split into segments at multiple quality levels. The client switches quality based on bandwidth and buffer health.

**Q. How do you handle upload processing?**  
Store original in object storage, enqueue transcoding job, generate renditions/thumbnails, update processing status, then publish.

---

## Spotify

### Functional Requirements / Features
- Music streaming
- Playlists
- Search
- Recommendations
- Offline downloads
- Artist pages
- Listening activity
- Audio CDN

### Example APIs
- `GET /tracks/{id}/stream`
- `POST /playlists`
- `GET /recommendations`
- `POST /listening-events`

### Core Data Model
- `tracks`
- `albums`
- `artists`
- `playlists`
- `playlist_tracks`
- `listening_history`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Low-latency streaming
- Rights/region restrictions
- Playlist consistency
- Recommendation pipeline
- Offline license handling

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How does Spotify differ from YouTube?**  
Audio files are smaller and playback latency/buffering is different, but rights management, recommendations, offline downloads, and playlist consistency are central.

**Q. How do offline downloads work?**  
Download encrypted content with licenses/expiry, sync play events later, and enforce subscription/region/device rules.

**Q. How do playlists scale?**  
Store playlist metadata separately from ordered playlist items, support pagination, versioning, and conflict handling for concurrent edits.

---

## Amazon / Flipkart

### Functional Requirements / Features
- Product catalog
- Search
- Inventory
- Cart
- Checkout
- Payments
- Orders
- Shipment tracking
- Reviews
- Recommendations
- Coupons
- Seller dashboard

### Example APIs
- `GET /products/search`
- `POST /cart/items`
- `POST /orders`
- `POST /payments`
- `GET /orders/{id}`

### Core Data Model
- `products`
- `inventory`
- `carts`
- `orders`
- `order_items`
- `payments`
- `shipments`
- `reviews`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Search relevance
- Inventory consistency
- Cart expiry
- Checkout idempotency
- Payment workflow
- Order state machine

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you avoid overselling inventory?**  
Use atomic inventory reservation or locking during checkout, expire reservations, and reconcile with order/payment state.

**Q. Cart consistency requirements?**  
Cart can usually be eventually consistent, but checkout/order/inventory must be more strongly controlled.

**Q. How do you design order creation?**  
Use idempotency key, create pending order, reserve inventory, initiate payment, confirm order on payment success, release inventory on failure/timeout.

---

## Food Delivery App

### Functional Requirements / Features
- Restaurant search
- Menu
- Cart
- Order placement
- Payment
- Delivery partner assignment
- Live tracking
- ETA
- Ratings
- Refunds/cancellations

### Example APIs
- `GET /restaurants`
- `POST /orders`
- `POST /payments`
- `GET /orders/{id}/track`

### Core Data Model
- `restaurants`
- `menus`
- `orders`
- `delivery_partners`
- `locations`
- `assignments`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Geo search
- Restaurant availability
- Order workflow
- Delivery matching
- Live location streaming
- ETA prediction

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you assign delivery partners?**  
Find nearby available partners using geo-index, score by distance/current load/vehicle/acceptance probability, send offer, timeout and retry.

**Q. How do live tracking updates work?**  
Driver app sends location periodically; backend stores latest location and pushes updates to customer via WebSocket/SSE or polling.

**Q. What parts need strong consistency?**  
Payment, order state transitions, refunds, and restaurant acceptance. Location and ETA can be eventually consistent.

---

## Uber / Ola

### Functional Requirements / Features
- Rider app
- Driver app
- Location tracking
- Nearby driver matching
- Ride booking
- ETA
- Dynamic pricing
- Trip management
- Payments
- Ratings
- Notifications

### Example APIs
- `POST /rides/request`
- `POST /drivers/location`
- `POST /rides/{id}/accept`
- `GET /rides/{id}/track`

### Core Data Model
- `riders`
- `drivers`
- `driver_locations`
- `rides`
- `ride_events`
- `payments`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- High-frequency location ingestion
- Geo indexing
- Matching
- Surge pricing
- State machine
- Real-time updates

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you find nearby drivers?**  
Use geospatial indexes such as geohash/S2/H3 cells, query nearby cells, filter available drivers, and rank by ETA and acceptance likelihood.

**Q. How do you handle two riders selecting same driver?**  
Use atomic compare-and-set or transactional state transition from available to assigned.

**Q. Why is location stored differently from ride records?**  
Location is high-frequency ephemeral data; ride records are durable transactional data. They have different storage/access patterns.

---

## Google Maps

### Functional Requirements / Features
- Location search
- Route calculation
- ETA
- Traffic updates
- Nearby places
- Map tiles
- Navigation
- Offline maps

### Example APIs
- `GET /places/search`
- `GET /routes`
- `GET /tiles/{z}/{x}/{y}`
- `POST /traffic-events`

### Core Data Model
- `places`
- `roads_graph`
- `traffic_segments`
- `map_tiles`
- `user_locations`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Geospatial indexing
- Graph algorithms
- Tile serving/CDN
- Traffic ingestion
- ETA prediction
- Offline caching

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How are map tiles served?**  
Pre-render or dynamically generate tiles by zoom/x/y coordinates, store/cache them, and serve through CDN.

**Q. How is route calculation modeled?**  
Road network is a graph; routing uses shortest path algorithms with edge weights based on distance, speed, restrictions, and traffic.

**Q. How do you handle traffic updates?**  
Ingest real-time location/speed/events, aggregate by road segment, update edge weights, and refresh ETA.

---

## BookMyShow / Ticketmaster

### Functional Requirements / Features
- Event listing
- Seat layout
- Seat locking
- Booking
- Payment
- Ticket generation
- Cancellation
- Waiting room
- Notifications

### Example APIs
- `GET /events`
- `GET /shows/{id}/seats`
- `POST /seats/hold`
- `POST /bookings`
- `POST /payments`

### Core Data Model
- `events`
- `venues`
- `shows`
- `seats`
- `seat_holds`
- `bookings`
- `payments`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Concurrent seat booking
- Temporary locks
- Payment timeout
- Queue/waiting room for high demand
- Idempotency

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you prevent double booking?**  
Use transactional seat holds with unique constraints/row locks and expiry. Confirm booking only after payment success.

**Q. Why use seat hold expiry?**  
Users may abandon payment. Expiry releases seats automatically and prevents permanent lock.

**Q. How do you handle high-demand events?**  
Use waiting rooms, rate limits, cache seat maps, queue booking attempts, and protect DB from spikes.

---

## Hotel / Flight Booking

### Functional Requirements / Features
- Search
- Availability
- Pricing
- Booking
- Payment
- Cancellation
- Partner integration
- Inventory sync
- Confirmation

### Example APIs
- `GET /search`
- `POST /availability/check`
- `POST /bookings`
- `POST /payments`

### Core Data Model
- `listings/flights`
- `availability`
- `bookings`
- `payments`
- `partner_sync_events`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- External partner APIs
- Availability freshness
- Price changes
- Reservation holds
- Cancellation policy

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. Why is booking travel hard?**  
Availability and prices can change frequently and may depend on external partners. The system must revalidate before payment/confirmation.

**Q. How do you handle stale search results?**  
Treat search as approximate, recheck availability and price before booking, then hold inventory if possible.

**Q. How do cancellations work?**  
Use policy engine, update booking state, initiate refund if applicable, notify partner/user, and reconcile payment state.

---

## Payment Gateway

### Functional Requirements / Features
- Payment initiation
- Authorization
- Capture
- Refunds
- Webhooks
- Idempotency
- Ledger
- Reconciliation
- Fraud detection
- Transaction status

### Example APIs
- `POST /payments`
- `POST /payments/{id}/capture`
- `POST /refunds`
- `POST /webhooks/provider`

### Core Data Model
- `payments`
- `payment_attempts`
- `ledger_entries`
- `refunds`
- `webhook_events`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Idempotency
- Provider retries
- Webhook dedupe
- Ledger correctness
- Reconciliation
- PCI/security

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. Why are idempotency keys critical?**  
Clients may retry payment requests due to timeouts. Idempotency prevents double charging.

**Q. How do webhooks work?**  
Providers send asynchronous status updates. The system verifies signature, deduplicates event, updates payment state, and records raw event.

**Q. Why maintain a ledger?**  
A ledger gives auditable double-entry record of money movement, useful for correctness, reconciliation, and compliance.

---

## Wallet / UPI-like System

### Functional Requirements / Features
- User wallet
- Bank linking
- Money transfer
- Transaction history
- Balance
- Ledger
- Limits
- Fraud checks
- Notifications
- Reconciliation

### Example APIs
- `POST /transfers`
- `GET /balance`
- `GET /transactions`
- `POST /bank-link`

### Core Data Model
- `wallets`
- `ledger_entries`
- `transfers`
- `limits`
- `fraud_checks`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Strong consistency
- Ledger immutability
- Idempotency
- Fraud/risk checks
- Reconciliation
- Limits

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you store wallet balance?**  
Use immutable ledger entries as source of truth and maintain balance as derived/cached value with transactional updates.

**Q. How do you prevent double spend?**  
Use transactions/locks/atomic updates on wallet balance or ledger constraints, plus idempotency keys.

**Q. Why not simply update balance column?**  
A single balance column lacks auditability. Ledger entries provide traceable history and reconciliation.

---

## Stock Trading App

### Functional Requirements / Features
- Market data
- Watchlist
- Order placement
- Order matching/routing
- Portfolio
- Real-time prices
- Risk checks
- Notifications

### Example APIs
- `GET /quotes`
- `POST /orders`
- `GET /portfolio`
- `WS /market-data`

### Core Data Model
- `users`
- `accounts`
- `orders`
- `executions`
- `positions`
- `market_ticks`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Low-latency market data
- Order state machine
- Risk validation
- Event sourcing
- Audit/compliance

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. What matters most in trading systems?**  
Correctness, auditability, low latency, ordering, risk checks, and regulatory compliance.

**Q. How do order states work?**  
Orders transition through states like pending, accepted, partially filled, filled, cancelled, rejected. State transitions must be durable and auditable.

**Q. How do you serve real-time prices?**  
Ingest market ticks, publish through streaming/pub-sub, cache latest quote, and deliver via WebSocket.

---

## Dropbox / Google Drive

### Functional Requirements / Features
- File upload/download
- Folder hierarchy
- Sharing permissions
- Sync
- Version history
- Conflict resolution
- Large file upload
- Search

### Example APIs
- `POST /files`
- `GET /files/{id}`
- `POST /share`
- `GET /sync/changes`

### Core Data Model
- `files`
- `file_versions`
- `folders`
- `permissions`
- `sync_events`
- `object_chunks`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Chunked upload
- Object storage
- Metadata DB
- Permission checks
- Delta sync
- Conflict handling

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How does file sync work?**  
Clients track local state and server change cursor. Server returns delta changes since cursor; conflicts create versions or conflict copies.

**Q. How do large uploads work?**  
Use chunked/multipart upload, resume failed chunks, verify checksums, then commit metadata.

**Q. How are permissions modeled?**  
Store ownership and ACL/shared link permissions separately from file metadata; enforce permission on every read/write/search.

---

## GitHub

### Functional Requirements / Features
- Repositories
- Commits
- Branches
- Pull requests
- Issues
- Code search
- Permissions
- Webhooks
- CI integration
- Notifications

### Example APIs
- `GET /repos`
- `POST /pulls`
- `POST /issues`
- `GET /search/code`
- `POST /webhooks`

### Core Data Model
- `repos`
- `branches`
- `commits`
- `pull_requests`
- `issues`
- `permissions`
- `webhooks`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Git object storage
- Permission model
- Code search indexing
- Webhook delivery retries
- Activity feed

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. What is hard about GitHub system design?**  
Repository storage, permission checks, code search, webhooks, CI integration, notifications, and high write/read activity around PRs/issues.

**Q. How do webhooks scale?**  
Store events, deliver asynchronously with retries/backoff, sign payloads, and expose delivery logs.

**Q. How do permissions work?**  
Check organization/repo/team roles and branch protection rules before allowing actions.

---

## Google Docs

### Functional Requirements / Features
- Real-time editing
- Comments
- Sharing permissions
- Version history
- Presence
- Offline edits
- Conflict resolution

### Example APIs
- `GET /documents/{id}`
- `POST /documents/{id}/ops`
- `WS /documents/{id}/collaborate`

### Core Data Model
- `documents`
- `document_ops`
- `document_versions`
- `comments`
- `permissions`
- `presence`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Operational transformation/CRDT
- Low-latency collaboration
- Versioning
- Permission checks
- Offline merge

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. OT vs CRDT?**  
Both handle concurrent edits. OT transforms operations based on concurrent changes; CRDTs use data structures that merge without conflicts. Each has complexity trade-offs.

**Q. How do you store document history?**  
Store snapshots plus operation logs. Snapshots speed loading; operation logs preserve history and support collaboration.

**Q. How is presence handled?**  
Use WebSockets and TTL heartbeats; presence is ephemeral and eventually consistent.

---

## Notification System

### Functional Requirements / Features
- In-app
- Email
- SMS
- Push
- Preferences
- Templates
- Delivery tracking
- Retries
- Deduplication
- Batching
- Priority

### Example APIs
- `POST /notifications`
- `GET /notifications`
- `POST /preferences`

### Core Data Model
- `notification_requests`
- `notification_deliveries`
- `templates`
- `preferences`
- `dedupe_keys`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Multi-channel routing
- User preferences
- Provider failures
- Retry/DLQ
- Deduplication
- Rate limiting

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you avoid duplicate notifications?**  
Use dedupe keys, idempotent requests, event IDs, and provider delivery tracking.

**Q. How do retries work?**  
Retry transient failures with exponential backoff, send permanent failures to DLQ, and avoid retrying invalid requests.

**Q. How are preferences applied?**  
Before delivery, evaluate user/channel preferences, quiet hours, unsubscribe status, and priority rules.

---

## Search Autocomplete

### Functional Requirements / Features
- Prefix search
- Typeahead
- Ranking
- Trending queries
- Personalization
- Typo tolerance
- Low latency

### Example APIs
- `GET /suggest?q=iph`

### Core Data Model
- `query_terms`
- `prefix_index`
- `popular_queries`
- `user_search_history`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- Trie/prefix index
- Precomputed suggestions
- Cache hot prefixes
- Ranking by popularity/freshness/personalization

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you build autocomplete?**  
Precompute prefix-to-suggestions mappings using tries or search indexes, cache hot prefixes, and rank by popularity/personalization.

**Q. How do you keep suggestions fresh?**  
Update counts from search logs through streaming/batch jobs and periodically rebuild prefix indexes.

**Q. Why cache autocomplete?**  
Many users type common prefixes like “i” or “iph”, so caching reduces latency and backend load.

---

## Logging and Metrics System

### Functional Requirements / Features
- Log ingestion
- Metrics collection
- Traces
- Storage
- Indexing
- Querying
- Dashboards
- Alerts
- Retention

### Example APIs
- `POST /logs`
- `POST /metrics`
- `GET /query`
- `POST /alerts`

### Core Data Model
- `log_events`
- `metric_points`
- `trace_spans`
- `alert_rules`
- `dashboards`

### High-Level Architecture
- Client → API Gateway / Load Balancer → domain services → databases/cache/object storage/search/queue as needed.
- Read path should be optimized separately from write path where traffic patterns differ.
- Expensive or slow work should move to async workers through queues/streams.
- Cache hot reads and serve static/media assets through CDN when relevant.

### Main Scaling / Deep-Dive Areas
- High-volume ingestion
- Buffering
- Time-series storage
- Index cost
- Retention/downsampling
- Alerting

### Production Notes
- Define clear state machines for workflows such as booking, payment, upload processing, delivery, or moderation.
- Use idempotency for retryable writes.
- Add observability for latency, error rate, queue lag, cache hit ratio, storage growth, and business success metrics.
- Prefer correctness for money, inventory, permissions, and security-sensitive paths; prefer eventual consistency for feeds, counters, analytics, and notifications when acceptable.

### Common Interview Questions and Answers
**Q. How do you ingest high-volume logs?**  
Use agents to batch/compress logs, send to queue/stream, process asynchronously, index selected fields, and store raw logs cheaply.

**Q. Metrics vs logs?**  
Metrics are numeric time-series for monitoring; logs are detailed events for debugging. Metrics are cheaper to query at scale.

**Q. How do retention policies work?**  
Keep high-resolution recent data, downsample older metrics, and archive/delete old logs to control cost.

---

# Part 4 — Low-Level Design

## LLD Interview Framework

### Definition
Low-level design focuses on classes, interfaces, objects, relationships, responsibilities, design patterns, extensibility, and concurrency behavior. It translates requirements into maintainable code structure.

### Standard Flow
1. Clarify requirements and constraints.
2. Identify core entities/classes.
3. Define relationships and responsibilities.
4. Define interfaces and service methods.
5. Handle edge cases, validation, and errors.
6. Discuss concurrency/thread safety if relevant.
7. Apply design patterns only where useful.

### Common Interview Questions
**Q. What is the difference between HLD and LLD?**  
HLD focuses on architecture, services, databases, scaling, and infrastructure. LLD focuses on code-level class design, interfaces, patterns, and object interactions.

**Q. What makes a good LLD answer?**  
Clear responsibilities, low coupling, high cohesion, extensibility, correct handling of edge cases, and sensible use of design patterns.

## OOP Fundamentals

| Topic | Definition | Example |
|---|---|---|
| **Class** | Blueprint for creating objects with data and behavior. | A ParkingLot class contains Floors, Slots, and TicketService. |
| **Object** | Runtime instance of a class. | A ParkingLot class contains Floors, Slots, and TicketService. |
| **Encapsulation** | Hide internal state and expose controlled methods. | A ParkingLot class contains Floors, Slots, and TicketService. |
| **Abstraction** | Expose essential behavior and hide implementation details. | A ParkingLot class contains Floors, Slots, and TicketService. |
| **Inheritance** | Child class reuses or extends parent behavior. | A ParkingLot class contains Floors, Slots, and TicketService. |
| **Polymorphism** | Same interface, different implementations. | PaymentProcessor interface with StripeProcessor and RazorpayProcessor implementations. |
| **Composition** | Build complex behavior by containing other objects; often preferred over inheritance. | A ParkingLot class contains Floors, Slots, and TicketService. |
| **Interface** | Contract that classes implement. | PaymentProcessor interface with StripeProcessor and RazorpayProcessor implementations. |
| **Abstract Class** | Base class with shared behavior and abstract methods. | A ParkingLot class contains Floors, Slots, and TicketService. |
| **Dependency Injection** | Provide dependencies from outside instead of creating them internally. | A ParkingLot class contains Floors, Slots, and TicketService. |

### Common Interview Questions
**Q. Explain polymorphism with example.**  
A PaymentProcessor interface can have StripeProcessor and RazorpayProcessor implementations. The checkout service calls processPayment() without knowing the internal provider details.

**Q. Composition vs inheritance?**  
Inheritance models “is-a” relationships but can create rigid hierarchies. Composition models “has-a” relationships and is usually more flexible.

## SOLID Principles

| Principle | Meaning | Example |
|---|---|---|
| **Single Responsibility Principle** | A class should have one reason to change. | Separate InvoiceCalculator from InvoicePrinter. |
| **Open/Closed Principle** | Open for extension, closed for modification. | Add new PaymentProcessor implementation without changing CheckoutService. |
| **Liskov Substitution Principle** | Subclasses should be usable wherever parent is expected. | A Square should not break behavior expected from Rectangle if setters differ. |
| **Interface Segregation Principle** | Prefer small focused interfaces. | Do not force Printer to implement scan() if it only prints. |
| **Dependency Inversion Principle** | Depend on abstractions, not concrete implementations. | OrderService depends on PaymentGateway interface. |

### Common Interview Questions
**Q. Why is SOLID useful?**  
It improves maintainability, testability, extensibility, and reduces coupling in object-oriented systems.

**Q. Which SOLID principle is most commonly violated?**  
Single Responsibility is often violated when one class handles validation, business logic, persistence, and formatting together.

## Design Patterns

| Pattern | Definition | Production Example |
|---|---|---|
| **Singleton** | Ensure only one instance exists. | Configuration manager; use carefully because it can hurt testing. |
| **Factory** | Create objects without exposing construction logic. | NotificationFactory creates Email/SMS/Push sender. |
| **Abstract Factory** | Create families of related objects. | UI component factory for Windows/Mac themes. |
| **Builder** | Construct complex objects step by step. | Build complex search query or HTTP request. |
| **Prototype** | Clone existing object. | Copy game objects or templates. |
| **Adapter** | Convert one interface to another. | Wrap third-party payment API behind internal interface. |
| **Decorator** | Add behavior dynamically. | Add compression/encryption around stream. |
| **Proxy** | Control access to another object. | Lazy loading, access control, remote proxy. |
| **Facade** | Simplify complex subsystem. | CheckoutFacade orchestrates cart, payment, inventory. |
| **Strategy** | Swap algorithms at runtime. | Different pricing or matching strategies. |
| **Observer** | Notify subscribers on state change. | Event listeners, notification subscriptions. |
| **Command** | Encapsulate request as object. | Undo/redo, job queues. |
| **State** | Change behavior based on state. | Order states: pending, paid, shipped, cancelled. |
| **Template Method** | Base algorithm with overridable steps. | Data import pipeline with custom parse step. |
| **Chain of Responsibility** | Pass request through handlers. | Middleware chain or approval workflow. |
| **Repository** | Abstract persistence logic. | UserRepository hides SQL details. |
| **Unit of Work** | Track changes and commit transaction together. | ORM transaction boundary. |
| **MVC** | Separate model, view, controller. | Classic web apps. |
| **MVVM** | Separate view from view model/state. | Frontend frameworks/mobile apps. |

### Common Interview Questions
**Q. When would you use Strategy pattern?**  
When you need interchangeable algorithms behind the same interface, such as different discount, pricing, matching, or routing strategies.

**Q. Factory vs Strategy?**  
Factory is about object creation. Strategy is about selecting behavior/algorithm.

**Q. Why avoid overusing design patterns?**  
Patterns add abstraction. Use them when they solve real extensibility or complexity problems, not to make simple code unnecessarily complicated.

## Parking Lot

### Core Classes / Entities
- `ParkingLot`
- `Floor`
- `ParkingSlot`
- `Vehicle`
- `Ticket`
- `Gate`
- `PaymentService`

### Useful Patterns
- Strategy for slot assignment
- Factory for vehicle types
- State for ticket/payment status

### Main Design Concerns
- Concurrent slot assignment
- Multiple vehicle types
- Pricing strategy
- Ticket lifecycle

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Parking Lot?**  
The core entities are usually: ParkingLot, Floor, ParkingSlot, Vehicle, Ticket, Gate, PaymentService. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Elevator System

### Core Classes / Entities
- `Elevator`
- `Floor`
- `Request`
- `Scheduler`
- `ButtonPanel`
- `Door`

### Useful Patterns
- Strategy for scheduling
- State for elevator movement

### Main Design Concerns
- Request scheduling
- Direction handling
- Emergency mode
- Concurrent requests

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Elevator System?**  
The core entities are usually: Elevator, Floor, Request, Scheduler, ButtonPanel, Door. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Library Management System

### Core Classes / Entities
- `Book`
- `BookCopy`
- `Member`
- `Loan`
- `Reservation`
- `Catalog`

### Useful Patterns
- Repository for catalog
- State for book copy availability

### Main Design Concerns
- Multiple copies
- Due dates/fines
- Search
- Reservation queue

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Library Management System?**  
The core entities are usually: Book, BookCopy, Member, Loan, Reservation, Catalog. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## BookMyShow LLD

### Core Classes / Entities
- `Show`
- `Seat`
- `SeatLock`
- `Booking`
- `Payment`
- `User`

### Useful Patterns
- State for booking
- Strategy for pricing

### Main Design Concerns
- Seat locking
- Expiry
- Concurrency
- Payment confirmation

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in BookMyShow LLD?**  
The core entities are usually: Show, Seat, SeatLock, Booking, Payment, User. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Splitwise

### Core Classes / Entities
- `User`
- `Group`
- `Expense`
- `Split`
- `BalanceSheet`
- `SettlementService`

### Useful Patterns
- Strategy for split type
- Command for expense creation

### Main Design Concerns
- Equal/exact/percentage splits
- Balance simplification
- Idempotent expense updates

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Splitwise?**  
The core entities are usually: User, Group, Expense, Split, BalanceSheet, SettlementService. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Chess

### Core Classes / Entities
- `Board`
- `Cell`
- `Piece`
- `Move`
- `Player`
- `Game`

### Useful Patterns
- Strategy per piece movement
- State for game status

### Main Design Concerns
- Move validation
- Check/checkmate
- Undo/history
- Special moves

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Chess?**  
The core entities are usually: Board, Cell, Piece, Move, Player, Game. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Snake and Ladder

### Core Classes / Entities
- `Game`
- `Board`
- `Cell`
- `Player`
- `Dice`
- `Snake`
- `Ladder`

### Useful Patterns
- Strategy for dice rolling

### Main Design Concerns
- Turn order
- Multiple players
- Win condition
- Configurable board

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Snake and Ladder?**  
The core entities are usually: Game, Board, Cell, Player, Dice, Snake, Ladder. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## ATM

### Core Classes / Entities
- `ATM`
- `Card`
- `Account`
- `Transaction`
- `CashDispenser`
- `BankService`

### Useful Patterns
- State for ATM flow
- Chain for cash denomination dispensing

### Main Design Concerns
- Authentication
- Cash availability
- Transaction rollback
- External bank failures

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in ATM?**  
The core entities are usually: ATM, Card, Account, Transaction, CashDispenser, BankService. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Vending Machine

### Core Classes / Entities
- `VendingMachine`
- `Inventory`
- `Product`
- `Payment`
- `Dispenser`

### Useful Patterns
- State for selection/payment/dispense
- Strategy for payment

### Main Design Concerns
- Change return
- Out of stock
- Refunds
- Inventory update

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Vending Machine?**  
The core entities are usually: VendingMachine, Inventory, Product, Payment, Dispenser. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Food Delivery LLD

### Core Classes / Entities
- `Restaurant`
- `Menu`
- `Cart`
- `Order`
- `DeliveryPartner`
- `AssignmentService`

### Useful Patterns
- Strategy for assignment
- State for order lifecycle

### Main Design Concerns
- Order state machine
- Partner assignment
- Menu availability
- Cancellation

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Food Delivery LLD?**  
The core entities are usually: Restaurant, Menu, Cart, Order, DeliveryPartner, AssignmentService. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Ride Sharing LLD

### Core Classes / Entities
- `Rider`
- `Driver`
- `Ride`
- `Location`
- `MatchingService`
- `PricingService`

### Useful Patterns
- Strategy for matching/pricing
- State for ride lifecycle

### Main Design Concerns
- Driver availability
- Location updates
- Ride state transitions
- Payment

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Ride Sharing LLD?**  
The core entities are usually: Rider, Driver, Ride, Location, MatchingService, PricingService. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Notification LLD

### Core Classes / Entities
- `Notification`
- `Template`
- `Preference`
- `ChannelSender`
- `DeliveryStatus`

### Useful Patterns
- Strategy for channel sending
- Factory for sender

### Main Design Concerns
- Preference filtering
- Retries
- Dedupe
- Templates

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Notification LLD?**  
The core entities are usually: Notification, Template, Preference, ChannelSender, DeliveryStatus. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Logging Framework

### Core Classes / Entities
- `Logger`
- `LogLevel`
- `Appender`
- `Formatter`
- `LogEvent`

### Useful Patterns
- Chain/appender pattern
- Strategy for formatting

### Main Design Concerns
- Async logging
- Level filtering
- Multiple sinks
- Thread safety

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Logging Framework?**  
The core entities are usually: Logger, LogLevel, Appender, Formatter, LogEvent. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Rate Limiter LLD

### Core Classes / Entities
- `RateLimiter`
- `Rule`
- `Bucket`
- `RequestContext`
- `Storage`

### Useful Patterns
- Strategy for algorithm

### Main Design Concerns
- Token bucket
- Sliding window
- Distributed counters
- Atomic updates

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Rate Limiter LLD?**  
The core entities are usually: RateLimiter, Rule, Bucket, RequestContext, Storage. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Cache LLD

### Core Classes / Entities
- `Cache`
- `Entry`
- `EvictionPolicy`
- `Storage`
- `TTLManager`

### Useful Patterns
- Strategy for eviction
- Decorator for metrics

### Main Design Concerns
- LRU/LFU
- TTL expiry
- Thread safety
- Capacity limit

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Cache LLD?**  
The core entities are usually: Cache, Entry, EvictionPolicy, Storage, TTLManager. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## File System LLD

### Core Classes / Entities
- `FileSystemNode`
- `File`
- `Directory`
- `Permission`
- `PathResolver`

### Useful Patterns
- Composite pattern
- Strategy for permissions

### Main Design Concerns
- Path traversal
- Permissions
- Large directories
- Metadata

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in File System LLD?**  
The core entities are usually: FileSystemNode, File, Directory, Permission, PathResolver. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Task Scheduler

### Core Classes / Entities
- `Task`
- `Scheduler`
- `Worker`
- `Trigger`
- `RetryPolicy`

### Useful Patterns
- Priority queue
- Strategy for retry

### Main Design Concerns
- Delayed tasks
- Recurring tasks
- Retries
- Worker failures

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Task Scheduler?**  
The core entities are usually: Task, Scheduler, Worker, Trigger, RetryPolicy. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

## Pub/Sub System

### Core Classes / Entities
- `Topic`
- `Publisher`
- `Subscriber`
- `Message`
- `Broker`
- `Subscription`

### Useful Patterns
- Observer pattern
- Queue per subscriber

### Main Design Concerns
- Message ordering
- Retries
- Fanout
- Backpressure

### Example Interview Answer Structure
1. Define entities and relationships.
2. Define service classes and key methods.
3. Explain state transitions and validations.
4. Discuss concurrency or extensibility concerns.

### Common Interview Questions and Answers
**Q. What are the core entities in Pub/Sub System?**  
The core entities are usually: Topic, Publisher, Subscriber, Message, Broker, Subscription. I would keep responsibilities separated so domain objects hold state while services coordinate workflows.

**Q. How would you make this design extensible?**  
Use interfaces for behavior that can vary, such as pricing, matching, payment, scheduling, notification channel, or eviction policy. Keep business rules isolated from persistence and UI concerns.

**Q. What concurrency issue can appear here?**  
Concurrent requests may modify the same resource, such as seat, slot, balance, inventory, cache key, or task. Use locks, transactions, atomic operations, or compare-and-set depending on the system.

---

# Part 5 — Revision Material

## System Design Trade-Offs

| Trade-off | How to explain in interview |
|---|---|
| **SQL vs NoSQL** | SQL gives strong relational modeling and transactions; NoSQL can scale specific access patterns with flexible schemas. |
| **Monolith vs Microservices** | Monolith is simpler; microservices add independent scaling/ownership but increase operational complexity. |
| **REST vs GraphQL vs gRPC** | REST is simple/public-friendly, GraphQL is client-flexible, gRPC is efficient/internal. |
| **WebSockets vs Long Polling** | WebSockets are efficient for continuous real-time; long polling is simpler fallback. |
| **Strong vs Eventual Consistency** | Strong improves correctness; eventual improves availability/latency/scale. |
| **Cache-aside vs Write-through** | Cache-aside is simple and common; write-through improves cache consistency but slows writes. |
| **Fanout on Write vs Fanout on Read** | Write fanout makes reads fast but writes expensive; read fanout keeps writes cheap but reads expensive. |
| **Offset vs Cursor Pagination** | Offset is simple but poor for large changing data; cursor is scalable and stable. |
| **Vertical vs Horizontal Scaling** | Vertical is simpler but limited; horizontal scales better but requires statelessness/partitioning. |
| **Batch vs Stream Processing** | Batch is simpler/cheaper for periodic work; streaming supports low-latency updates. |
| **Sync vs Async Processing** | Sync is immediate/simple; async improves resilience/latency but adds eventual consistency. |
| **Object Storage vs Database Storage** | Object storage is better for large blobs; DB is better for structured metadata and transactions. |
| **Sharding vs Replication** | Replication copies data; sharding splits data. |
| **Single-region vs Multi-region** | Single-region is simpler; multi-region improves latency/DR but complicates consistency. |
| **Simplicity vs Scalability** | Start simple until bottlenecks justify complexity. |

## Production Checklist

- [ ] Clear functional and non-functional requirements
- [ ] Back-of-envelope capacity estimate
- [ ] API contracts and idempotency
- [ ] Data model and indexes
- [ ] Caching strategy
- [ ] Async processing for slow tasks
- [ ] Load balancing and autoscaling
- [ ] Rate limits and abuse prevention
- [ ] Authentication and authorization
- [ ] Encryption and secrets management
- [ ] Observability: logs, metrics, traces, alerts
- [ ] Retries with timeout, backoff, jitter
- [ ] Circuit breakers and graceful degradation
- [ ] Queue DLQ and replay strategy
- [ ] Backup, restore, and disaster recovery
- [ ] Schema migration and backward compatibility
- [ ] Security and privacy review
- [ ] Cost and operational complexity review

## Interview Question Bank

**Q. How would you design a URL shortener?**  
Focus on short code generation, redirect latency, database uniqueness, cache, analytics, and abuse prevention.

**Q. How would you design Twitter timeline?**  
Focus on follow graph, tweet storage, fanout write/read hybrid, celebrity users, cache, ranking, and search/trends.

**Q. How would you design Instagram?**  
Focus on media upload/storage/CDN, feed generation, stories/reels, likes/comments, search, notifications, and moderation.

**Q. How would you design WhatsApp?**  
Focus on message persistence, WebSockets, offline delivery, ordering, read receipts, group chat, and push notifications.

**Q. How would you design YouTube?**  
Focus on video upload, transcoding, object storage, CDN streaming, adaptive bitrate, recommendations, and search.

**Q. How would you design Uber?**  
Focus on geo indexing, driver matching, location updates, ride state machine, pricing, and real-time tracking.

**Q. How would you design BookMyShow?**  
Focus on seat locking, booking flow, payment, concurrency control, wait rooms, and idempotency.

**Q. How would you design a payment system?**  
Focus on idempotency, state machine, ledger, provider webhooks, reconciliation, retries, and fraud checks.

**Q. How would you design Google Docs?**  
Focus on real-time collaboration, OT/CRDT, version history, permissions, offline edits, and presence.

**Q. How would you design a notification system?**  
Focus on channels, preferences, templates, queues, retries, dedupe, rate limits, and delivery tracking.

**Q. How would you design search autocomplete?**  
Focus on prefix index/trie, ranking, caching hot prefixes, freshness, typo tolerance, and personalization.

**Q. How would you design a recommendation system?**  
Focus on candidate generation, ranking, real-time/batch features, cold start, feedback, A/B testing, and serving latency.

**Q. How would you design a rate limiter?**  
Focus on algorithm selection, per-user/IP keys, Redis atomic counters, distributed consistency, and failure behavior.

**Q. How would you design a distributed cache?**  
Focus on key partitioning, replication, eviction, consistency, hot keys, and failure handling.

**Q. How would you design a logging system?**  
Focus on ingestion, buffering, indexing, storage, retention, query, dashboards, and alerts.

## Best Study Order

1. Learn the interview framework and requirement clarification.
2. Practice estimation: RPS, storage, bandwidth, peak traffic.
3. Study APIs, databases, indexing, caching, load balancing, queues, CDN.
4. Study distributed systems: consistency, replication, sharding, CAP, idempotency.
5. Practice common HLD problems: URL shortener, Twitter, Instagram, WhatsApp, YouTube, Uber, BookMyShow, Payment Gateway.
6. Practice LLD separately: Parking Lot, Elevator, Splitwise, Rate Limiter, Cache, Notification System.
7. Revise trade-offs and speak answers aloud in 2–5 minute formats.

## Final Interview Advice

- Do not over-engineer early. Start with a simple design, then scale bottlenecks.
- Always say why a component exists.
- State assumptions clearly.
- For money, inventory, permissions, and security: prioritize correctness.
- For feeds, counters, analytics, search, and notifications: eventual consistency is often acceptable.
- Prefer practical production trade-offs over textbook perfection.
