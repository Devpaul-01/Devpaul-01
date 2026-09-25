# Toluwase Ogunsola
**Backend Systems & Distributed Infrastructure Engineer**  
*TypeScript · Node.js · Python · PostgreSQL · Redis · LLM Orchestration*

I engineer backend systems where **correctness under concurrency and failure** is the core requirement—building append-only financial ledgers, fault-tolerant multi-provider AI infrastructure, and crash-resilient background execution pipelines.

📫 **Email:** [oluwaseyiogunsola90@gmail.com](mailto:oluwaseyiogunsola90@gmail.com) | 💼 **LinkedIn:** [in/dev-paul-697727376](https://linkedin.com/in/dev-paul-697727376) | 🌐 **Portfolio:** [portfolio-five-orcin-3go8vhmpm0.vercel.app](https://portfolio-five-orcin-3go8vhmpm0.vercel.app/)

> **Open to Global Remote Roles & Contract Engineering Opportunities**

---

### 🚀 Core Engineering Projects

#### 🏦 [Kith](https://github.com/Devpaul-01/Kith) — *Financial Ledger & Coordination Platform*
`Node.js` `TypeScript` `Express` `PostgreSQL` `Redis` `BullMQ` `Supabase`

A multi-tenant financial infrastructure platform designed for immutable state and high concurrency. **199 API endpoints, ~30 core tables, 18 Architecture Decision Records (ADRs).**

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
* 🔗 **[Live Demo](https://web-production-cd6dc0.up.railway.app)**

---

#### 💼 [FounderSales](https://github.com/Devpaul-01/Foundersales) — *AI Sales Coaching Platform*
`Node.js` `TypeScript` `Express` `PostgreSQL` `Redis` `BullMQ`

An AI coaching and sales analysis engine optimized for latency reduction and API cost control.

* **Distributed Fallback Chain:** Built a 4-provider LLM fallback system (Cerebras → Groq → Mistral → OpenRouter) with Redis-backed cooldown states across instances to prevent cascading API rate limits.
* **Payload Consolidation (75% Latency Reduction):** Consolidated 4 sequential LLM calls into a single structured payload, dropping response latency from 3.2s to 800ms and cutting API costs by ~60%.
* **Pre-Call Token Gating:** Engineered an audit-logged caching module that reuses existing prospect research within a 14-day sliding window, preventing token waste on recurring workflows.

---

### 🛠️ Technical Capabilities

| Category | Technologies & Concepts |
| :--- | :--- |
| **Languages** | TypeScript, JavaScript, Python, SQL |
| **Backend & Architecture** | Node.js, Express, Flask, REST APIs, Distributed Systems, Asynchronous Processing, Idempotency |
| **Databases & Queues** | PostgreSQL (RPCs, Indexes, Transactions), Redis, BullMQ, RQ, SQLAlchemy, Supabase |
| **AI Systems** | Multi-Provider LLM Orchestration, Streaming APIs, Fallback Chains, Cost Optimization |
| **Security & Auth** | JWT, Refresh Token Rotation, RBAC, OAuth, Rate Limiting, CSRF Protection |
| **Infra & Testing** | Docker, GitHub Actions (CI/CD), Jest, Pytest, AWS, Railway, Vercel |
