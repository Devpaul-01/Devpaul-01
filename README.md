# Toluwase Ogunsola

**Backend engineer: distributed systems, concurrency control, and AI infrastructure.**
TypeScript · Go · Python · PostgreSQL · Redis

Three years building backend systems where the hard part is staying correct under concurrency and failure: shared budgets that can't be overspent by racing processes, ledgers that can't be edited after the fact, and LLM traffic that keeps flowing when a provider goes down. Currently a backend engineering intern at BirthdayMessaging, and a Computer Science student at FUTA.

I write the failure cases down. Every project below has tests that run against real Redis or Postgres, architecture decision records for the trade-offs, and where I make a performance or correctness claim, a reproducible measurement behind it.

[Email](mailto:oluwaseyiogunsola90@gmail.com) · [LinkedIn](https://www.linkedin.com/in/ogunsola-toluwase-697727376/) · [Portfolio](https://portfolio-five-orcin-3go8vhmpm0.vercel.app/)

**Based in Nigeria (WAT, UTC+1). Open to global remote roles and contract engineering work.**

---

## Featured work

### [agent-broker](https://github.com/Devpaul-01/agent-broker): admission control for multi-process agent systems
`TypeScript` `Python` `Redis` `Lua` · [npm](https://www.npmjs.com/package/agent-broker) · in active development (v0.1)

When several OS processes share a rate-limited resource such as an LLM provider, no process can coordinate locally, because there is no "local". agent-broker answers *"is this call allowed right now?"* in one atomic Redis round trip, so independent processes, in either language, share one real decision.

```
$ npm run demo:race
30 processes racing for a budget that covers 3 calls...
admitted: 3   denied: 27   OK exactly 3
```

- **One script, not a check-then-act sequence.** A single Lua script checks circuit state, budget, and concurrency and reserves all three. I rejected a split-script design in [ADR-0023](https://github.com/Devpaul-01/agent-broker/blob/main/docs/adr/0023-single-combined-admission-script.md) because the gap between scripts would reopen the exact race the design exists to close.
- **Two languages, one protocol.** The TypeScript and Python clients run the same Lua against the same keys. A test extracts the script from the TypeScript source and diffs it against the Python copy, so drift fails CI instead of relying on a comment.
- **Failure semantics are deliberate.** Circuit recovery closes on a single probe success, and [ADR-0010](https://github.com/Devpaul-01/agent-broker/blob/main/docs/adr/0010-single-probe-circuit-recovery.md) records why I dropped the original three-state design. Agent identity and depth are derived server-side and never trusted from the caller, with a depth-spoofing test.
- **Verified against real processes.** 188 TypeScript tests (27 files) and 68 Python tests (14 files) run against real Redis, including tests that fork actual OS processes and kill them. 26 ADRs. Admission latency measured at a median of 0.22–0.30 ms and p99 under 5.3 ms (N=1000, local Redis). The measurement report documents three mistakes I made along the way instead of smoothing them over.

---

### [llm-gateway](https://github.com/Devpaul-01/llm-gateway): self-hostable multi-provider LLM gateway
`Go` `PostgreSQL` `Redis` `Lua` `Prometheus` `React` · [Live demo](https://llm-gateway-studyhubplatform.vercel.app/demo)

An OpenAI-compatible gateway in front of 12 providers. Apps change their `base_url` and get failover, caching, budgets, and observability underneath. A zero-key demo mode runs a mock provider with chaos switches (down, rate-limited, flaky). With the primary provider killed and 300 prompts at concurrency 20, the repo's measured run shows 100% success, with only 18 requests reaching the dead provider before its breaker opened.

- **Shared-state circuit breaker.** Five transient failures in 30 seconds open a provider's breaker (atomic Lua `INCR` + `EXPIRE`). An open provider moves to the back of the candidate chain instead of being removed, so it is retried once healthy. State lives in Redis, so every gateway instance agrees.
- **Cache that can't serve stale answers.** Only deterministic (temperature 0) text requests are cached, keyed by SHA-256. Bumping a per-project epoch invalidates every entry in O(1) with no key scan, and the write script re-checks the epoch so an in-flight response can't repopulate a cache that was just invalidated.
- **Atomic token budgets.** A Lua check-and-reserve gates each request against a daily budget, then settles against actual usage after the response, refunding over-estimates and charging overshoots.
- **Production details.** Provider credentials are encrypted at rest with AES-256-GCM. It supports streaming (SSE), Prometheus metrics with Grafana dashboards, and a request log in Postgres.
- **Tested for races.** 84 Go tests in 22 files. CI runs `go test -race ./...` against real Postgres 16 and Redis 7 service containers.
- **Used by my own products.** StudyHub and FounderSales both route their AI calls through it. Migrating them let me delete their hand-rolled provider-failover code.

---

### [Kith](https://github.com/Devpaul-01/Kith): multi-tenant financial ledger for family coordination
`Node.js` `Express` `PostgreSQL` `Redis` `BullMQ` `Supabase` · [Live demo](https://kith-self.vercel.app)

An append-only ledger with a 124-operation REST API (OpenAPI), 23 tables, 21 ADRs, and 73 test files.

- **Immutability enforced in the database.** A Postgres trigger freezes confirmed, disputed, and resolved ledger rows and whitelists the only legal status transitions. I added it because admin writes use the service-role key and bypass row-level security, so an app-layer rule alone wasn't enough. Corrections are new linked entries, never edits.
- **Two independent duplicate defenses.** A client idempotency key under a unique constraint, plus an RPC that takes a Postgres advisory lock and checks for near-duplicate submissions inside a time window. That closes the race where two concurrent submissions both pass the check before either commits.
- **Atomic multi-step writes as Postgres RPCs.** `raise_dispute_atomic` row-locks the ledger entry first, closing the double-dispute race.
- **Background pipeline built for failure.** Nine BullMQ queues, a notification outbox table, and a recovery scan that re-enqueues deliveries stuck in `pending` or `failed`.
- **Tenant isolation on the request path.** JWT, then workspace membership, then role. Membership is cached in Redis for 30 seconds and invalidated on change. A failed workspace check returns 404 rather than 403, so workspace IDs can't be enumerated.

---

### [StudyHub](https://github.com/Devpaul-01/Studyhub): real-time peer-learning platform with an AI study assistant
`Python` `Flask` `PostgreSQL` `Redis` `Socket.IO` · [Live demo](https://studyhub-two-psi.vercel.app/)

A social feed, live co-study sessions, and an AI assistant, built to run correctly across multiple gunicorn workers. 84 test files and over 1,370 test functions.

- **A scheduler lock that fails closed.** If Redis is unreachable, a cron job is skipped rather than risking duplicate execution across workers, the opposite default from the rest of the app and chosen on purpose. Benchmarked at 0 overlap violations across 90 trials with up to 25 concurrent processes, against a naive baseline that produced up to 4,750. The raw results and harness are in the repo.
- **AI calls classified, not just retried.** Gateway responses map to four outcomes (`KEY_FAULT`, `PROVIDER_TRANSIENT`, `BAD_MODEL`, `NON_RETRYABLE`) that decide whether to cool down, evict a model, or abort. A short Redis-backed cooldown keeps the app from hammering a gateway that is struggling. This replaced my original 6-provider failover once llm-gateway existed.
- **Real-time state that survives scaling out.** Presence, rate limits, and Socket.IO fan-out (`message_queue`) live in Redis so any worker can serve any client. An 8-thread pool keeps slow AI calls off the WebSocket path.
- **Refresh-token rotation with reuse detection.** Tokens are stored only as SHA-256 hashes and rotated on every use. A 10-second grace window absorbs legitimate multi-tab races, and reuse outside it revokes the whole token family.

---

### [FounderSales](https://github.com/Devpaul-01/Foundersales): AI sales coaching platform
`Node.js` `Express` `PostgreSQL` `Redis` `BullMQ`

Calendar-aware meeting prep, prospect research, voice-memo capture, post-meeting debriefs, and live buyer-simulation practice. 190 API endpoints and 48 tables.

- **A cost gate in front of every AI call.** `calendarAiGate.js` decides whether a call is worth making: it reuses research on the same prospect from the last 14 days, skips low-stakes events, and degrades when quota runs low. Every decision and its reason is written to an audit table, so "we cut AI spend" is a query you can run.
- **A distributed concurrency guard that self-heals.** It replaced a design whose per-process pending queue could leave a process waiting on a promise nothing would ever resolve. In-flight calls are slots in a Redis sorted set scored by timestamp, and stale slots from crashed holders are swept on the next acquire with no separate reaper job. It fails open by design, because a user should never be blocked by traffic shaping.
- **Gateway migration without touching call sites.** I replaced a 4-provider fallback chain with a single llm-gateway client, kept a cooldown on the one gateway key, and preserved the external contract so 20+ call sites didn't change.

---

## Experience

**Backend Engineering Intern, BirthdayMessaging** · Oct 2026 – present

Built the backend for a new mobile app on top of the company's published Chrome extension, as one of two engineers on the app, working directly with the founder.

- Async FastAPI and SQLAlchemy 2.0 on PostgreSQL, with JWT access tokens and single-use refresh tokens where reuse or expiry revokes the whole session.
- A 15-minute reminder scheduler guarded by a crash-safe Postgres advisory lock, with one per-reminder, per-year idempotency record so a reminder can't double-fire.
- Redis rate limiting adapted for mobile, Cloudflare R2 presigned uploads, and PayPal Orders-API payment capture with webhook-confirmed activation.
- Audited the published extension and fixed a silent DM failure, three unimplemented PayPal functions, and 32 tests that were passing without testing anything.

## Education

**B.Sc. Computer Science**, Federal University of Technology, Akure (FUTA) · expected 2028

## Writing

- [Designing agent-broker: Collision-Safe Redis Keys, Budget Integrity, and Broker-Derived Delegation Identities](https://dev.to/oluwaseyi_ogunsola_691294/designing-agent-broker-collision-safe-redis-keys-budget-integrity-and-zero-trust-delegation-npl)
- [Reengineering Family Finance: Building Financial Integrity Into Kith](https://dev.to/oluwaseyi_ogunsola_691294/reengineering-family-finance-building-financial-integrity-into-kith-1lf0)
- [Not All AI Provider Failures Are the Same: Building a Fault-Aware LLM Fallback System](https://dev.to/oluwaseyi_ogunsola_691294/not-all-ai-provider-failures-are-the-same-building-a-fault-aware-llm-fallback-system-31a3)

---

## Skills

**Languages:** TypeScript, JavaScript, Go, Python, SQL, Lua
**Backend:** Node.js, Express, FastAPI, Flask, REST API design, OpenAPI, idempotency, rate limiting, circuit breakers, admission control
**Data:** PostgreSQL (RPCs, triggers, advisory locks, row-level locking, RLS), Redis (Lua scripting, sorted sets, distributed locks), Supabase
**Async and real-time:** BullMQ, notification outbox pattern, Socket.IO / WebSockets, scheduled jobs
**AI infrastructure:** multi-provider routing and failover, response caching, token budgeting, streaming, cost gating
**Security:** JWT, refresh-token rotation, AES-256-GCM, RBAC, enumeration-resistant error handling
**Delivery:** Docker, GitHub Actions (including `go test -race` against real services), Jest, Vitest, Pytest, Prometheus, Grafana, Sentry, Vercel, Fly.io
