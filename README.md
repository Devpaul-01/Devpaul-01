# Toluwase Ogunsola

**Backend Engineer — Distributed Systems, Concurrency Control & AI Infrastructure**
`TypeScript` · `Go` · `Python` · `Node.js` · `PostgreSQL` · `Redis` · `Lua`

I build backend systems where **correctness under concurrency and failure** is the actual requirement — atomic admission control for multi-process agents, immutable financial ledgers, multi-provider LLM gateways with circuit breakers, and crash-resilient background pipelines. Verification over assertion: every project below ships with tests that exercise the failure paths, not just the happy path.

📫 [oluwaseyiogunsola90@gmail.com](mailto:oluwaseyiogunsola90@gmail.com) · 💼 [linkedin.com/in/dev-paul-697727376](https://linkedin.com/in/dev-paul-697727376) · 🌐 [Portfolio](https://portfolio-five-orcin-3go8vhmpm0.vercel.app/)

> **Open to global remote roles & contract engineering work**

---

## 🔥 Currently building: [agent-broker](https://github.com/Devpaul-01/agent-broker)
*A Redis-backed admission gate for multi-process agent systems* · [npm](https://www.npmjs.com/package/agent-broker) · `TypeScript` `Python` `Redis` `Lua`

Multiple OS processes sharing a rate-limited resource (LLM providers) can't fix check-then-spend races by coordinating locally — there's no "local" to coordinate. `agent-broker` answers *"is this call allowed right now?"* in one atomic Redis round trip, so independent processes — in either language — share one real admission decision.

```
$ npm run demo:race
30 processes racing for a budget that covers 3 calls...
admitted: 3   denied: 27   OK exactly 3
```

- **One atomic script, not a check-then-act sequence.** `REQUEST_PERMISSION` checks circuit state, budget, and concurrency and reserves all three in a single Lua script — explicitly rejected a split-script design (ADR-0023) because the gap *between* scripts would reopen the exact race the single script closes.
- **Dual-language, byte-identical protocol.** TypeScript and Python implementations are kept honest by an automated test (`test_script_equivalence.py`) that regex-extracts both Lua scripts and diffs them — not a comment promising they match.
- **Single-probe circuit recovery** on a Redis sorted-set failure window, with the earlier 3-state design and the reasoning for retiring it written up in [ADR-0010](https://github.com/Devpaul-01/agent-broker/blob/main/docs/adr/0010-single-probe-circuit-recovery.md).
- **TOCTOU-safe identity.** Agent depth, root ID, and budget key are always re-derived server-side inside the registration script — never trusted from caller input (ADR-0012, backed by an explicit depth-spoofing test).
- **Verified, not asserted:** 188 tests across 27 files (TS) + 68 tests across 14 files (Python) against real Redis, including tests that fork actual OS processes to check cross-process visibility and crash recovery. 26 ADRs. Admission latency benchmarked at a median of ~0.25ms, p99 under 5.3ms (N=1000) — benchmark doc includes its own repro commands and documents 3 mistakes found mid-measurement rather than smoothing them over.

---

## 🚀 Core Engineering Projects

### 🌐 [llm-gateway](https://github.com/Devpaul-01/llm-gateway) — *Self-Hostable Multi-Provider LLM Gateway*
`Go` `PostgreSQL` `Redis` `Lua` `Prometheus`

An OpenAI-API-compatible gateway in front of 12 LLM providers — drop-in `base_url` swap, with the reliability and cost controls a production app actually needs underneath.

- **Redis-backed circuit breaker per provider credential:** 5 failures in a 30s window opens it (atomic Lua `INCR`+`EXPIRE`); open providers fall to the back of the candidate chain rather than being removed, so they're retried once healthy.
- **Candidate-chain failover:** each request resolves into an ordered list across the model's fallback chain × every credential the project owns, filtered live by breaker and per-key cooldown state.
- **Exact-match response cache with O(1) bulk invalidation:** SHA-256-keyed, epoch-counter based — bumping a project's epoch invalidates every prior cache entry instantly with no key scan, and a Lua script re-checks the epoch at write time so an in-flight response can't repopulate a cache that was just invalidated.
- **Atomic token-budget gating:** daily per-project budget enforced by a Lua check-and-reserve script, settled against actual usage after the response completes (refunds over-estimates, charges overshoots).
- **Credentials encrypted at rest** with AES-256-GCM; all hot-path coordination (rate limiting, budget, cache, concurrency) is single-round-trip Lua, not multi-command sequences.
- 84 tests across 22 files, CI runs `go test -race ./...` against real Postgres and Redis service containers, not mocks.

---

### 🏦 [Kith](https://github.com/Devpaul-01/Kith) — *Financial Ledger & Coordination Platform*
`Node.js` `Express` `PostgreSQL` `Redis` `BullMQ` `Supabase`

Multi-tenant infrastructure for family financial coordination, built around an append-only ledger. **122 API endpoints, 23 tables, 21 ADRs, 73 test files.**

- **Immutability enforced at the database, not just the app layer.** A Postgres trigger (`enforce_ledger_entry_immutability`) blocks column changes on confirmed/disputed/resolved ledger rows — added specifically because admin writes use the service-role key and bypass RLS. Corrections are new linked entries, never edits.
- **Two independent idempotency defenses:** a client-supplied key under a DB unique constraint, plus a separate atomic RPC (`create_ledger_entry_checked`) that takes a Postgres advisory lock and checks for near-duplicate submissions within a configurable window — closing the race where two concurrent submissions both pass the check before either commits.
- **Atomic multi-step writes via Postgres RPCs**, not app-level transactions: `raise_dispute_atomic` row-locks the ledger entry before creating a dispute, closing the double-dispute race; `resolve_dispute_atomic` does the equivalent for resolution.
- **9-queue BullMQ pipeline** with a notification outbox table and a recovery scan that re-enqueues stuck deliveries.
- **Zero-trust auth path:** JWT → workspace membership → role, with membership cached in Redis for 30s and actively invalidated on mutation — and every authorization failure returns a uniform 404, never 403, to stay enumeration-resistant.
- 🔗 [Live demo](https://kith-self.vercel.app)

---

### 🧠 [StudyHub](https://github.com/Devpaul-01/Studyhub) — *Resilient AI & Real-Time Collaboration Backend*
`Python` `Flask` `PostgreSQL` `Redis` `WebSockets`

A peer-learning platform backend — social feed, live co-study sessions, and an AI study assistant — engineered for multi-worker horizontal scaling with no single point of failure.

- **6-provider LLM failover** (Gemini → Groq → Cohere → Cloudflare → Mistral → OpenRouter) with 4-way error classification (`KEY_FAULT`, `PROVIDER_TRANSIENT`, `BAD_MODEL`, `NON_RETRYABLE`) deciding whether to cool a key, evict a model, or abort — Redis-backed so cooldown state is shared across `gunicorn -w N` workers, fail-open to in-process state if Redis is unavailable.
- **WebSocket presence, thread rate limits, and scheduler locks moved to Redis** so any of N workers can serve any client; Socket.IO's `message_queue` is wired to Redis for cross-instance delivery.
- **Distributed scheduler lock that fails closed** — the opposite default of everything else in the app: if Redis is unreachable, a cron job is skipped rather than risking duplicate execution across workers. Benchmarked at 0 overlap violations across 90 trials (up to 25 concurrent processes) against a naive baseline that produced up to 4,750.
- **8-worker thread pool isolates AI latency** from the WebSocket event loop, so a slow provider call never blocks real-time message delivery.
- **Refresh-token rotation with reuse detection:** tokens stored only as SHA-256 hashes, rotated on every use, with a 10-second grace window to tolerate legitimate multi-tab races without treating them as theft — reuse outside that window revokes the entire token family.
- 87 test files, 1,400+ tests. 🔗 [Live demo](https://studyhub-two-psi.vercel.app/)

---

### 💼 [FounderSales](https://github.com/Devpaul-01/Foundersales) — *AI Sales Coaching Platform*
`Node.js` `Express` `PostgreSQL` `Redis` `BullMQ`

AI-driven sales coaching and outreach: voice-profile-driven opportunity discovery, live simulated buyer practice sessions, and calendar-aware meeting prep.

- **Redis sorted-set concurrency guard** replaced a local promise-queue design that could deadlock a process with no in-flight work waiting on a promise nothing would resolve. Stale slots self-heal via a timestamp-scored sweep on every check — no separate reaper job — and the guard fails open on Redis errors rather than blocking real requests.
- **4-provider LLM fallback** (Cerebras → Groq → Mistral → OpenRouter) with Redis-backed per-key cooldowns, generalized enough that the same module also handles external API key rotation elsewhere in the app.
- **Consolidated multi-call AI flows into single structured calls** (e.g. merging reply generation, outcome classification, and coaching tip into one request instead of several sequential ones) to cut both latency and cost.
- **14-day research-reuse gate:** before paying for new prospect research, checks for existing research on the same prospect within a 14-day window and logs every gate decision (proceed/skip + reason) to an audit table — so the cost-saving claim is independently checkable, not just asserted.
- 190 API endpoints across 48 tables.

---

## 🛠️ Technical Capabilities

| Category | Technologies & Concepts |
| :--- | :--- |
| **Languages** | TypeScript, JavaScript, Go, Python, SQL |
| **Backend & Distributed Systems** | Node.js, Express, Flask, net/http, REST APIs, Concurrency Control, Race Conditions, Idempotency, Circuit Breakers, Admission Control |
| **Databases & Caching** | PostgreSQL (RPCs, triggers, advisory locks, row-locking), Redis (Lua scripting, sorted sets, pub/sub), Supabase |
| **Queues & Async Processing** | BullMQ, RQ, Background Workers, Notification Outbox Pattern, Distributed Scheduler Locks |
| **AI Infrastructure** | Multi-Provider LLM Orchestration (4–12 providers across projects), Fallback Chains, Response Caching, Token Budgeting, Cost Optimization |
| **Security & Observability** | JWT, Refresh Token Rotation, AES-256-GCM Encryption at Rest, RBAC, Rate Limiting, Prometheus/Grafana, Sentry |
| **Infra & Testing** | Docker, GitHub Actions (CI, incl. `-race` against real Postgres/Redis), Jest, Pytest, Go testing, Vercel, Railway |
