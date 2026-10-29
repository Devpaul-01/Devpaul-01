# Toluwase Ogunsola
**Backend Systems & Distributed Infrastructure Engineer**  
*TypeScript · Node.js · Python · PostgreSQL · Redis · Distributed Systems & Concurrency*

I engineer backend systems where **correctness under concurrency and failure** is the core requirement—building append-only financial ledgers, distributed coordination primitives, fault-tolerant multi-provider AI infrastructure, and crash-resilient background processing pipelines.

📫 **Email:** [oluwaseyiogunsola90@gmail.com](mailto:oluwaseyiogunsola90@gmail.com) | 💼 **LinkedIn:** [in/dev-paul-697727376](https://linkedin.com/in/dev-paul-697727376) | 🌐 **Portfolio:** [portfolio-five-orcin-3go8vhmpm0.vercel.app](https://portfolio-five-orcin-3go8vhmpm0.vercel.app/)

> **Open to Global Remote Roles & Contract Engineering Opportunities**

---

### 🔥 Active Focus & Infrastructure Highlight

#### ⚙️ [agent-broker](https://github.com/Devpaul-01/agent-broker) — *Redis-Backed Admission Gate for Multi-Process Agent Systems*
`TypeScript` `Python` `Node.js` `Redis` `Lua` `Distributed Systems` · [Published on npm](https://www.npmjs.com/package/agent-broker)

A library that answers one question under real concurrency: *"is this call allowed right now?"* It coordinates shared budget, delegation depth, in-flight concurrency and circuit-breaker state across **independent processes** through Redis. TypeScript and Python implementations run the same Lua scripts against the same keys, so processes in both languages can share one budget pool.

```
$ npm run demo:race
30 processes racing for a budget that covers 3 calls...
admitted: 3   denied: 27   OK exactly 3
```

* **Atomic Lua Gatekeeping:** Agent, circuit, budget and concurrency checks and reservations happen in a single Redis Lua script, so there is no check-then-act window for racing callers.
* **Reservation & Reconciliation:** TTL reservations settle on `reportOutcome`; a lazy sweep recovers reservations from crashed callers and refuses late reports, so nothing is refunded twice.
* **Sliding-Window Circuit Breaker:** Redis sorted-set failure window per target, with probe-based recovery. One successful probe closes the circuit (the original 3-state design was retired, with the reasoning recorded in [ADR-0010](https://github.com/Devpaul-01/agent-broker/blob/main/docs/adr/0010-single-probe-circuit-recovery.md)).
* **Verified, not asserted:** 188 tests across 27 files against a real Redis, with failure-mode suites that race real forked processes; cross-language tests with TypeScript and Python workers on one Redis; 26 ADRs; local admission latency measured at a median of about 0.25 ms (benchmark, not a production figure).

---

### 🚀 Core Engineering Projects

#### 🏦 [Kith](https://github.com/Devpaul-01/Kith) — *Financial Ledger & Coordination Platform*
`Node.js` `TypeScript` `Express` `PostgreSQL` `Redis` `BullMQ` `Supabase`

A multi-tenant financial infrastructure platform designed for immutable state and high concurrency. **199 API endpoints, ~30 core tables, 20 Architecture Decision Records (ADRs).**

* **Immutable Append-Only Ledger:** Historical state is immutable; corrections are executed as new linked ledger entries. Idempotency keys + a duplicate-submission heuristic prevent double-spends and network retry bugs.
* **Atomic PostgreSQL RPCs:** Moved race-sensitive operations (disputes, invite acceptances, recurring target conversions) into atomic database transactions, eliminating check-then-write race conditions.
* **Durable 9-Queue Pipeline:** Engineered a BullMQ async pipeline with per-queue rate limits and a PostgreSQL notification outbox pattern with recovery scans for stuck jobs (99.9% delivery reliability).
* **Zero-Trust Multi-Tenant Auth:** Enforced JWT → Workspace → Role verification cached in Redis (30s actively invalidated TTL) with enumeration-resistant 404s on unauthorized access.
* 🔗 **[Live Demo](https://kith-self.vercel.app)**

---

#### 🧠 [StudyHub](https://github.com/Devpaul-01/Studyhub) — *Resilient AI & Real-Time Collaboration Infrastructure*
`Python` `Flask` `PostgreSQL` `Redis` `WebSockets`

An asynchronous, multi-worker peer-learning backend engineered for zero single-points-of-failure under concurrent scale.

* **Multi-Provider AI Resilience:** Built a 6-provider LLM failover system with structured error classification (`KEY_FAULT`, `PROVIDER_TRANSIENT`, `BAD_MODEL`) to guarantee near-100% uptime during upstream API outages.
* **Distributed State Scaling:** Shifted WebSocket presence, rate limiting, and cron scheduler locks to Redis, enabling horizontal scaling across multi-worker setups (`gunicorn -w N`).
* **Latency Isolation:** Offloaded LLM execution to a bounded 8-worker thread pool, preventing slow AI responses from blocking the main WebSocket event loop handling real-time messaging.
* **Security & Token Rotation:** Implemented SHA-256 hashed refresh token rotation with an automated 10-second grace window to handle multi-tab browser races while detecting token-reuse attacks.
* 🔗 **[Live Demo](https://studyhub-two-psi.vercel.app/)**

---

#### 💼 [FounderSales](https://github.com/Devpaul-01/Foundersales) — *AI Sales Coaching Platform*
`Node.js` `TypeScript` `Express` `PostgreSQL` `Redis` `BullMQ`

An AI coaching and sales analysis engine optimized for latency reduction and API cost control.

* **Distributed Concurrency Guard:** Engineered a generic Redis Sorted Set slot registry to replace deadlocking local promise queues across distributed nodes, featuring automated staleness sweeps and fail-open degradation.
* **Distributed Fallback Chain:** Built a 4-provider LLM fallback system (Cerebras → Groq → Mistral → OpenRouter) with Redis-backed cooldown states across instances to prevent cascading API rate limits.
* **Payload Consolidation (75% Latency Reduction):** Consolidated 4 sequential LLM calls into a single structured payload, dropping response latency from 3.2s to 800ms and cutting API costs by ~60%.
* **Pre-Call Token Gating:** Engineered an audit-logged caching module that reuses existing prospect research within a 14-day sliding window, preventing token waste on recurring workflows.

---

### 🛠️ Technical Capabilities

| Category | Technologies & Concepts |
| :--- | :--- |
| **Languages** | TypeScript, JavaScript, Python, SQL |
| **Backend & Distributed Systems** | Node.js, Express, Flask, REST APIs, Concurrency Control, Race Conditions, Idempotency, Circuit Breakers |
| **Databases & Caching** | PostgreSQL (RPCs, Indexes, Transactions, Row-locking), Redis (Lua Scripting, Sorted Sets, Pipelines), Supabase |
| **Queues & Asynchronous Processing** | BullMQ, RQ, Background Workers, Notification Outbox Patterns |
| **AI Infrastructure** | Multi-Provider LLM Orchestration, Fallback Chains, Cost Optimization, Rate-Limit Cooldowns |
| **Security & Observability** | JWT, Refresh Token Rotation, RBAC, Rate Limiting, Request Trace IDs (`X-Trace-Id`), Sentry Alerting |
| **Infra & Testing** | Docker, GitHub Actions (CI/CD), Jest, Pytest, AWS, Railway, Vercel |
