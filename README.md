# Hi, I'm Toluwase 👋

**Backend Software Engineer** | Node.js · TypeScript · Python

I build systems where correctness matters — financial ledgers, real-time messaging, AI orchestration, and distributed coordination. I care about failure modes, race conditions, and documenting *why* decisions were made, not just *what* was built.

---

## 🚀 What I'm Working On

### [FounderSales](https://github.com/Devpaul-01/Foundersales) — AI Sales Coaching Platform

**Node.js · TypeScript · Express · PostgreSQL · Redis · BullMQ · 4-Provider AI Fallback**

Sales enablement platform for founders selling without a sales team. Real outreach outcomes and AI roleplay practice are scored on the same rubric, feeding the same coaching loop.

**Key engineering decisions:**
- **4-provider LLM fallback** (Cerebras → Groq → Mistral → OpenRouter) — failures classified by HTTP status, not string matching
- **AI buyer psychology** — each practice session has a persona with hidden motivations and live interest/trust/confusion scores
- **Two-layer idempotency for meeting prep** — BullMQ `jobId` dedup + DB-state re-check for cross-trigger races
- **Cost-gated AI actions** — prep generation, discovery, and research check if generation is worth the spend first
- **Rolling chat summarization** — bounds token growth so long coaching conversations stay cost-effective
- **Namespaced rate limiting** — one limiter per route sized to actual AI cost

---

### [Kith](https://github.com/Devpaul-01/Kith) — Family Financial Coordination Platform

**Node.js · TypeScript · Express · PostgreSQL · Redis · BullMQ**

Multi-tenant family financial coordination platform with **199 API endpoints** and **25–30 core database tables**. Built around:

- **Append-only ledger** — confirmed contributions are never edited in place
- **Atomic PostgreSQL RPCs** — race-sensitive workflows run as single transactions
- **9-queue BullMQ system** — with Postgres-backed outbox for reliable notifications
- **18 Architecture Decision Records** documenting every major engineering trade-off

🔗 [Live Demo](https://kith-self.vercel.app)

---

### [StudyHub](https://github.com/Devpaul-01/Studyhub) — Real-Time Learning Platform

**Python · Flask · PostgreSQL · Redis · WebSockets · Multi-Provider AI**

Real-time academic collaboration platform with:

- **Redis-backed distributed coordination** — presence, rate limiting, scheduled-job state
- **Distributed lock that fails closed** — prevents duplicate cron executions across workers
- **6-provider AI fallback layer** — with Redis-backed health state and failure classification
- **8-worker ThreadPoolExecutor** — isolates slow AI calls from blocking real-time messaging

---

## 🛠️ Technologies I Work With

| Category | Technologies |
|---|---|
| **Languages** | TypeScript, JavaScript, Python |
| **Backend** | Node.js, Express.js, Flask, REST API Design |
| **Databases & Distributed Systems** | PostgreSQL, Redis, BullMQ, asynchronous job processing, distributed coordination |
| **Real-Time** | WebSockets, Socket.IO, Server-Sent Events |
| **Security** | JWT, OAuth, refresh-token rotation, CSRF protection, rate limiting, RBAC, multi-tenant authorization |
| **AI Systems** | Multi-provider LLM orchestration, streaming, provider failover, error classification |
| **Testing** | Jest, Pytest, CI |
| **Infrastructure & Services** | Docker, Git, GitHub, Supabase, AWS, Vercel, Firebase, Sentry |

---

## 📫 How to Reach Me

- 📧 [oluwaseyiogunso1a90@gmail.com](mailto:oluwaseyiogunsola90@gmail.com)
- 🔗 [LinkedIn](https://linkedin.com/in/dev-paul-697727376)
- 🌐 [Portfolio](https://portfolio-five-orcin-3go8vhmpm0.vercel.app/)

---

*Currently open to internship / entry-level backend engineering roles.*
