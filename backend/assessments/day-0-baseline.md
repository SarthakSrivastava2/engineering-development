# Backend & Distributed Systems — Day 0 Baseline

- **Date:** 2026-09-27
- **Type:** Diagnostic assessment, not a teaching session
- **Objective:** Establish a baseline across backend, networking, database, concurrency, distributed-systems, infrastructure, and production-engineering concepts.

## Overall diagnosis

> Practically experienced backend engineer with fragmented conceptual depth across networking, databases and distributed systems.

The central finding is that I often know a mechanism exists and roughly why it exists, but do not yet have a precise mental model of the layers involved or their failure modes. Day 0 should not be reduced to a programming-grade score.

The assessment showed practical experience and useful intuition in HTTP/API work, connection pools, caching, queues, Go HTTP servers, and infrastructure. It also exposed substantial conceptual gaps in networking layers, DNS, TLS, database internals, Kafka, distributed systems, reliability, tracing, containers, and systematic design. The diagnostic is preserved as given; later learning is not implied here.

## Assessment scope

Twenty questions covered HTTP request lifecycle; TCP/IP; DNS; TLS; HTTP status codes; indexes; transactions and ACID; isolation; connection pools; Redis/caching; queues; Kafka; Go concurrency; distributed systems; eventual consistency; reliability; observability; containers; Kubernetes; and URL-shortener design.

## Topic-by-topic findings

### 1. HTTP request lifecycle

**Answer:** Started the explanation at TCP because `http.ListenAndServe` listens on TCP. Described TCP as identifying HTTP and directing it to the listener, then mixed HTTP parsing, OAuth bearer-token validation, and request-body decoding together.

**Conceptual weakness:** Layering and ownership of responsibilities are blurred.

**Correct mental model:** TCP carries an ordered byte stream; HTTP is an application protocol carried over a transport. The Go HTTP server parses the request into an `http.Request`; handler/application code may decode `r.Body`. OAuth/bearer-token authentication is application logic, not a property of TCP or HTTP.

```text
Application: authentication, authorization, handler logic
HTTP:        request / response semantics
TLS:         connection security (when used)
TCP:         reliable ordered byte stream
IP:          network addressing and routing
Link:        Ethernet / Wi-Fi
```

### 2. TCP

**Strength:** Correctly understood that IP does not provide TCP-style delivery guarantees and identified SYN/SYN-ACK/ACK at a high level.

**Incorrect answer:** Included public-key cryptography in the TCP handshake.

**Correct mental model:** The TCP three-way handshake establishes connection state, including sequence-number-related state. TCP reliability does not come from cryptography; TLS is a separate security layer above TCP.

```text
Client                         Server
  SYN -------------------------->
      <------------------- SYN + ACK
  ACK -------------------------->
            connected
```

TCP fundamentals are partially understood; distinguish transport reliability from TLS security.

### 3. DNS

**Answer:** Described DNS as a hostname/IP database but conflated it with `/etc/hosts`, and treated Kubernetes DNS as a separate definition of DNS.

**Major weakness:** DNS resolution and hierarchy are not yet understood well enough.

**Correct mental model:** DNS is a distributed hierarchical naming system. `/etc/hosts` is a local static mapping. Recursive resolvers and authoritative servers have different roles; caches and TTLs affect resolution. Kubernetes uses DNS mechanisms for service discovery; it is an application of DNS concepts.

```text
api.example.com → resolver/cache → authoritative answer → IP address
                                                        ↓
                                                  TCP → TLS → HTTP
```

### 4. TLS

**Answer:** Recognized certificates and trusted CAs but conflated certificates, public/private keys, OAuth credentials, certificate verification, and application authentication. Described certificates as being decoded by a public key to validate the server.

**Major conceptual weakness:** TLS and OAuth/public-key application authentication are mixed together.

**Correct mental model:** A certificate binds an identity (such as a domain) to a public key and is signed by a CA. The client checks the chain, hostname, and validity as part of authenticating the server, then the TLS handshake establishes session keys. TLS secures the connection; OAuth/bearer tokens are application-level authentication/authorization.

```text
TCP: reliable transport
TLS: encryption, integrity, server authentication
HTTP: application request/response
OAuth / bearer token: application authentication/authorization
```

### 5. HTTP/API status codes

**Strength:** Correctly gave common semantics: creation as 200 or 201; malformed request 400; unauthenticated 401; unauthorized 403; missing resource 404; duplicate conflict 409; unexpected failure 500.

This is a relative strength in basic API semantics, not evidence of complete API-design knowledge.

### 6. Database indexes

**Answer:** Suggested indexing email and compared an index to a Go map: a way to avoid scanning every row.

**Partial understanding / major gap:** The intuition is useful, but B-trees, composite indexes, selectivity, covering indexes, index maintenance, query planners, and query plans were not understood.

For example, an appropriate index may help locate rows for:

```sql
SELECT * FROM users WHERE email = 'sarthak@example.com';
```

The database may still choose a table scan depending on the query and data. Indexes have write and storage costs; the unanswered question “Why not index every column?” is an important next concept.

### 7. Transactions and ACID

**Answer:** Admitted not knowing ACID and misremembered its expansion. The bank-transfer rollback example showed useful atomicity intuition.

**Correct mental model:** ACID means Atomicity, Consistency, Isolation, Durability. A transaction groups operations such that a failed transfer does not leave only the debit committed:

```text
BEGIN
  debit A
  credit B
COMMIT
```

Transactions, commit/rollback, constraints, locking, MVCC, and concurrent transactions are major learning gaps.

### 8. Isolation

**Answer:** Reasoned that concurrent transactions may both read the same initial value and overwrite one another, but did not know formal anomalies or isolation levels.

**Major weakness:** Dirty reads, non-repeatable reads, phantom reads, lost updates, READ COMMITTED, REPEATABLE READ, and SERIALIZABLE were unknown. Teach this after transaction foundations.

### 9. Connection pools

**Working knowledge:** Correctly explained that connection setup/teardown is expensive and a pool reuses connections. Also understood that poor sizing can waste resources, exceed connection limits, queue queries, and raise latency/timeouts.

Future depth: pool sizing, database limits, contention, application concurrency, connection lifetime/idle connections, and transaction interaction.

### 10. Redis and caching

**Practical intuition:** Described a cache-aside-like read path and considered updating or invalidating Redis after a database modification.

```text
GET → Redis hit → return
       miss → database → cache with TTL → return
```

**Conceptual gaps:** Write-through/write-behind, invalidation, staleness, TTL, stampedes, penetration, eviction, consistency, Redis failure, and distributed caching.

```text
Database: Alice     Cache: Bob
```

The system needs a defined source of truth and an invalidation/update policy. Caching is a moderate practical strength with important conceptual gaps.

### 11. Queues

**Basic intuition:** Understood that `Service A → Queue → Service B` decouples producer and consumer work so the producer need not wait synchronously. Also recognized that failure and rollback become more complicated.

**Gap:** Could not clearly distinguish a queue, pub/sub, and Kafka. Learn buffering, backpressure, acknowledgements, retries, duplicate delivery, idempotency, dead-letter queues, ordering, and producer/consumer failures before Kafka.

### 12. Kafka

**Answer:** Explicitly did not know Kafka. Partitions, offsets, consumer groups, ordering, replication, and delivery semantics were all unknown.

**Major gap:** Do not begin with Kafka implementation details; first establish generic messaging and asynchronous-processing concepts.

### 13. Go concurrency

**Answer:** Recognized concurrent increments as problematic but attributed goroutine termination to garbage collection and suggested `WaitGroup` to solve shared-counter safety.

**Correction:** `count++` is a read-modify-write and is not automatically safe across goroutines. A `WaitGroup` waits for completion; it does not protect shared memory. If `main` returns, the process exits; the garbage collector does not terminate goroutines because a function ended.

```text
G1 reads 0       G2 reads 0
G1 writes 1      G2 writes 1
Possible result: 1, although two increments were attempted
```

Use an appropriate mutex, atomic operation, or channel-based ownership for memory safety, and a `WaitGroup` separately when waiting for goroutines. **Lifecycle synchronization != memory synchronization.** This is a significant weakness despite practical Go familiarity.

### 14. Distributed systems

**Answer:** Explicitly said the question was not understood.

**Major gap:** Start from processes communicating across a network, where machines and links can fail, packets can be delayed/lost/duplicated/reordered, partitions can occur, replicas can diverge, and clocks differ. Do not jump straight to CAP or consensus.

```text
Process A  ←── network (delay / loss / partition) ──→  Process B
```

### 15. Eventual consistency

**Answer:** Explicitly did not know the concept.

**Major gap:** Learn strong and eventual consistency, read-after-write behavior, replication lag, stale reads, conflicts, and tradeoffs through concrete examples.

### 16. Reliability

**Answer:** Tried reasoning about stale Redis data and a database queue, showing practical intuition but not a broader reliability model.

**Major gap:** Timeouts, retries, exponential backoff, jitter, circuit breakers, bulkheads, rate limits, backpressure, graceful degradation, load shedding, idempotency, and failure isolation remain to learn.

```text
API → PostgreSQL / Redis / Kafka
```

A slow dependency can be more dangerous than a dead one: it can hold application resources, raise latency, exhaust pools, and cause cascading failure.

### 17. Observability

**Strength:** Understood logs as timestamped events and metrics as aggregates such as request counts, connection stats, and latency. Distributed tracing was not understood.

```text
Logs    — What happened?
Metrics — How much, how often, how bad?
Traces  — Where did this request spend its time?
```

**Major gap:** Learn traces, request/correlation IDs, spans, and how a trace identifies a slow downstream service. Logs and metrics are working knowledge; tracing is not.

### 18. Containers

**Incorrect answer:** Described containers as small VMs.

**Correct mental model:** VMs normally include a guest OS; containers generally share the host kernel and use isolation mechanisms such as namespaces and cgroups.

```text
VMs:        Host → guest OS → application
Containers: Host kernel → isolated userspace + application
```

Container fundamentals are a weakness.

### 19. Kubernetes

**Practical familiarity, conceptual gaps:** Understood a rough Service → Pods → Containers relationship, but treated a Pod too much like a container with metadata and a Service too much like an application process.

**Correct model:** A Pod is Kubernetes' basic execution unit and can contain one or more containers that share certain resources, including networking. A Service provides stable networking/service discovery for a set of Pods; traffic behavior depends on Service type and cluster networking.

```text
Client → Service (stable address) → selected Pod → container(s)
```

Learn Pods, Deployments, ReplicaSets, readiness/liveness, scheduling, resource requests/limits, ConfigMaps/Secrets, rollouts, and autoscaling. Do not claim deep Kubernetes understanding from the practical exposure observed.

### 20. URL shortener

**Answer:** Suggested generating a short ID, storing the mapping in a database, and optionally caching lookups in Redis, but did not complete the exercise.

**Finding:** Some component intuition exists; a systematic system-design method is a major gap. Build from requirements, API, data model, architecture, scaling, consistency, failure modes, observability, and tradeoffs.

## Strengths

- Practical backend and Go HTTP-server experience.
- HTTP/API basics and common status-code semantics.
- Connection-pool intuition.
- Basic cache-aside and queue-decoupling intuition, including awareness of stale data and rollback complexity.
- Some Kubernetes and infrastructure exposure.
- Useful reasoning attempts even where the formal model was incomplete.

## Major weaknesses

- Network-layer separation, TCP internals, DNS, and TLS.
- Database indexes, transactions, and isolation.
- Kafka, distributed systems, and consistency.
- Reliability engineering and distributed tracing.
- Container fundamentals and systematic system design.
- Concurrency safety distinctions between waiting and protecting shared state.

## Recommended progression

These phases are recommendations only; none is marked complete by this assessment.

1. **Networking foundations:** OSI/TCP-IP, packets/segments/streams, IP, TCP/UDP, ports/sockets, connection lifecycle, reliability, flow/congestion control concepts, DNS/caching, TLS/certificates/handshake, HTTP over TCP/TLS.
2. **HTTP and backend servers:** request lifecycle, methods, status, headers/body/cookies, authentication/authorization, REST/API design, middleware, timeouts, cancellation, Go servers, graceful shutdown.
3. **Databases:** relational modeling, SQL, keys, indexes/B-trees/composites, query plans, transactions/ACID, locks/MVCC/isolation, pools, PostgreSQL.
4. **Caching and Redis:** cache patterns, TTL/eviction/invalidation, stale data, stampedes/penetration, Redis data structures/atomicity, distributed locks, rate limiting.
5. **Queues and Kafka:** generic messaging, producers/consumers, acknowledgements, retries, DLQs, idempotency, ordering/backpressure/pub-sub; then topics, partitions, offsets, groups, replication, delivery semantics.
6. **Concurrency in backend systems:** concurrent requests, shared state/races/locks, pools, worker pools, queues, backpressure, cancellation, timeouts, graceful shutdown.
7. **Distributed systems:** replication, partitioning/sharding, leaders/followers, lag, consistency, quorum/CAP, coordination, consensus concepts, distributed transactions, idempotency, failure detection.
8. **Reliability:** timeouts, retries/backoff/jitter, circuit breakers, bulkheads, rate limiting, load shedding, graceful degradation, backpressure, SLOs/SLIs/error budgets, isolation.
9. **Observability:** structured logs, latency metrics/percentiles, tracing, request IDs, dashboards, alerting, production debugging.
10. **Containers and Kubernetes:** Linux processes, containers/namespaces/cgroups/images, Pods, Deployments, ReplicaSets, Services, discovery, probes, resources, scheduling, rollouts, autoscaling.
11. **System design:** after foundations, practice URL shortener, rate limiter, notifications, file upload, and job scheduler using requirements → API → data model → architecture → scaling → consistency → failure modes → observability → tradeoffs.

## Progress record and next task

**Backend & Distributed Systems — Day 0: Completed.**

Current state: Practically experienced backend engineer with fragmented conceptual depth.

Strongest areas: HTTP/API basics; connection pools; basic caching; basic asynchronous-processing intuition; practical infrastructure exposure.

Major weaknesses: Networking fundamentals; DNS; TLS; database internals; transactions/isolation; Kafka; distributed systems; consistency; reliability; tracing; container fundamentals; systematic system design.

**Next task:** Networking Fundamentals — Layering, TCP/IP and sockets.
