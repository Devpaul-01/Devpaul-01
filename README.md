# Toluwase Ogunsola

**Backend Software Engineer** — Node.js · TypeScript · Python

I build systems where correctness under concurrency and failure matters more than surface features — financial ledgers that can't silently lose data, multi-provider AI pipelines that degrade instead of breaking, and background job systems designed to survive crashes and races rather than just handle the happy path.

Open to **Global Remote Roles & Contractor Contracts**, remote or relocation.

---

## Projects

### [Kith](https://github.com/Devpaul-01/Kith) — Family Financial Coordination Platform
`Node.js` `TypeScript` `Express` `PostgreSQL` `Redis` `BullMQ` `React` `Supabase`

A multi-tenant platform for tracking shared family money and responsibilities — recurring contribution pools, disputes, tasks — built the way a fintech backend would be, not a typical CRUD app. **199 API endpoints, ~25–30 core tables, 18 documented Architecture Decision Records.**

- **Financial integrity by construction** — confirmed ledger entries are never edited in place; corrections are new linked entries, so history is always reconstructable. Idempotency keys plus a 10-minute duplicate-submission heuristic guard against both network retries and genuine double-submits.
- **Atomicity via Postgres RPCs** — dispute resolution, invite acceptance, contributor-target updates, and event→recurring-pool conversion all run as single-transaction stored procedures, closing check-then-write race windows that sequential application code can't safely close.
- **9-queue BullMQ system** with per-queue concurrency tuning and a Postgres-backed notification outbox — failed push/email deliveries are marked, not dropped, and a recovery scan re-enqueues anything stuck.
- **Layered authorization** — JWT → workspace membership → role, with a 30-second Redis membership cache (actively invalidated on writes) and enumeration-resistant 404s instead of 403s on unauthorized workspace access.
- **Real test coverage** — unit, integration, and rate-limit suites, with CI running against actual PostgreSQL and Redis rather than mocks.

🔗 [Live demo](https://kith-self.vercel.app)

---

### [FounderSales](https://github.com/Devpaul-01/Foundersales) — AI Sales Coaching Platform
`Node.js` `TypeScript` `Express` `PostgreSQL` `Redis` `BullMQ`

A coaching platform where real outreach outcomes and AI-simulated practice conversations are scored on the same rubric and feed one coaching loop.

- **4-provider LLM fallback chain** (Cerebras → Groq → Mistral → OpenRouter), with failures classified by structured HTTP status/error body — a bad key, a down provider, and a dead model each trigger a different recovery action instead of one generic retry.
- **AI cost gating** — a dedicated module decides whether an AI call is worth making *before* spending it (research reuse, low-stakes skip, quota checks), with every decision logged to an audit table.
- **One bundled AI call instead of four** for practice-session replies — reply text, buyer internal monologue, outcome classification, and coaching feedback all come back from a single request.
- **Redis-coordinated cross-instance state** — key cooldowns and model-discovery caching are shared across horizontally-scaled instances, with a documented kill switch back to in-memory behavior.

---

### [StudyHub](https://github.com/Devpaul-01/Studyhub) — Real-Time Learning Platform
`Python` `Flask` `PostgreSQL` `Redis` `WebSockets`

A peer academic collaboration platform (group chat, live study sessions, AI tutoring) built to run correctly under multiple worker processes, not just a single instance.

- **Redis-coordinated multi-instance state** — WebSocket presence, rate limiting, and scheduled-job locking all moved off process-local memory so `gunicorn -w N` is actually safe.
- **Fail-closed distributed locking** — the one deliberate exception to an otherwise fail-open Redis usage pattern, because a missed scheduler tick is cheaper than a duplicate cron run.
- **6-provider AI fallback** with Redis-backed health state and the same status-code-driven failure classification approach as FounderSales.
- **Latency isolation** — AI dispatch runs on a bounded 8-worker thread pool so a slow model call can't block the WebSocket event loop handling real-time messages.
- **Refresh-token rotation with reuse detection** — a 10-second grace window distinguishes a legitimate multi-tab race from an actual stolen-token replay.

🔗 [Live demo](https://web-production-cd6dc0.up.railway.app)

---

## Experience

**Freelance Web Developer** — local salon booking platform (Jun–Aug 2025)
Designed and shipped a Node.js/Express + PostgreSQL booking API end-to-end — requirements through deployment (Vercel/Railway, custom domain + SSL) — and maintained it in production for 3 months post-launch.

---

## Stack

| | |
|---|---|
| **Languages** | TypeScript, JavaScript, Python |
| **Backend** | Node.js, Express, Flask, REST API design |
| **Data** | PostgreSQL, Redis, BullMQ, SQLAlchemy, MongoDB |
| **Real-time** | WebSockets, Socket.IO |
| **Auth & Security** | JWT, OAuth, refresh-token rotation, CSRF, rate limiting, RBAC |
| **AI Systems** | Multi-provider LLM orchestration, failure classification, streaming |
| **Testing / Infra** | Jest, Pytest, CI, Docker, Supabase, AWS, Vercel, Railway |

---

## Contact

[oluwaseyiogunsola90@gmail.com](mailto:oluwaseyiogunsola90@gmail.com) · [LinkedIn](https://linkedin.com/in/dev-paul-697727376) · [Portfolio](https://portfolio-five-orcin-3go8vhmpm0.vercel.app/)
