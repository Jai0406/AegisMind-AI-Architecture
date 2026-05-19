# AegisMind AI ⌘
### Enterprise-Ready Document Intelligence Engine & Secure Multi-Silo RAG Pipeline

AegisMind AI is a production-grade, hardware-optimized Retrieval-Augmented Generation (RAG) architecture engineered for secure enterprise environments. Operating entirely via local compute, it implements a decoupled microservices design—utilizing a high-performance FastAPI backend matched with an analytics-driven system telemetry interface.

By eliminating all third-party cloud API dependencies, AegisMind AI guarantees absolute data privacy while maintaining low-latency execution through sub-millisecond pre-auth semantic caching, multi-format vector routing, and cross-encoder reranking.

---

## ⚖️ Legal & Proprietary Status
**AegisMind AI is the exclusive proprietary property of YesMaa.**

* **Architect & Lead Developer:** Jai Verma
* **License State:** **ALL RIGHTS RESERVED (PROPRIETARY).** No open-source distribution, replication, or modification rights are granted.
* **Purpose:** This repository serves strictly as an architectural specification and engineering portfolio showcase. For corporate inquiries, technical walkthroughs, or licensing requests, please contact the developer via formal professional channels.

---

## 🛡️ Core Security & Governance Features

The system implements a rigorous enterprise governance model directly within the pipeline layer:

- **Zero-Trust IAM Framework** — Implements automated, token-based validation passed via custom request headers (`X-Sat-Token`) to authenticate sessions securely without persistent sessions.
- **Role-Based Access Control (RBAC)** — Enforces strict role differentiation. `Admin` identities retain full system orchestration rights (user onboarding, document decommissioning, cache purging), while `Employee` contexts are securely sandboxed.
- **Granular Document-Level ACLs** — Access Control Lists are bound to the document metadata registry. Employees can query only the specific vector spaces and documents they have been explicitly authorized to view by an administrator.
- **Query Interception & Compliance Guardrails** — Automated text-matching layers intercept unauthorized queries mapping to restricted organizational domains (e.g., payroll, termination, layoffs, legal disputes), preventing data exposure.
- **Live Compliance Auditing** — Every network transaction writes directly to an append-only, thread-safe SQLite auditing ledger (`query_audit`) capturing exact employee telemetry, query text, accessed source footprints, and security-blocked events.

---

## ⚙️ Core Architecture & Pipeline Execution

### Multi-Silo Data Ingestion (`POST /upload`)
1. **Memory-Mapped Ingestion:** Large payloads (PDF, DOCX, CSV, JSON) are streamed directly to storage targets via disk-chunking, capping temporary RAM footprint at ~10MB regardless of massive file sizes.
2. **Byte & Semantic Deduplication:** Dual-stage validation checks incoming data against a standard SHA-256 byte hash and a semantic text hash (computed via sorted token sets). Overlapping versions trigger automated in-place versioning and replacements.
3. **Isolated Vector Embeddings:** Extracted chunks are vectorized using local embeddings and routed dynamically into **5 distinct ChromaDB collections** (PDF, DOCX, CSV, System Logs, Reports) to completely eliminate cross-format vector contamination.

### Multi-Layer Query Processing (`POST /query`)

```text
                        User Query Input (with Header SAT Validation)
                                              │
                                              ▼
     [Layer 1: Semantic Cache] ──▶ Local FAISS L2 Index Search (Distance < 0.15)
                                              │
                             ┌────────────────┴────────────────┐
                          (HIT)                             (MISS)
                             │                                 │
                 ACL Source Re-Verification                    ▼
                             │                  [Layer 2: Intent-Based Router]
                             ▼                  (Parses keywords & targets specific DB silo)
                 Instant ⚡ Return (<1s)                        │
                                                               ▼
                                                 [Layer 3: Vector Similarity Search]
                                                 (Retrieves top 16 candidate chunks)
                                                               │
                                                               ▼
                                                 [Layer 4: Cross-Encoder Reranking]
                                                 (FlashRank filters space down to top 4)
                                                               │
                                                               ▼
                                                 [Layer 5: Context Window Expansion]
                                                 (SQLite maps child chunks to parent text)
                                                               │
                                                               ▼
                                                 [Layer 6: Local LLM Generation]
                                                 (Phi-3.5 Local Synthesis & Audit Log)
```

---

## 🛠️ Production Tech Stack

| Operational Domain | Underlying Technology | Integration Framework |
|---|---|---|
| **API Framework** | FastAPI + Uvicorn Core | High-concurrency asynchronous network layer |
| **System Orchestrator** | Thread-Safe Multi-Worker Wrapper | Decoupled cross-process execution driver |
| **Relational / Audit Store** | SQLite3 Engine | Thread-locked relational persistence system |
| **Vector Indexing Silos** | ChromaDB Collection Array | Multi-format isolated semantic storage |
| **Sub-Millisecond Cache** | FAISS (FlatL2 CPU Index) | Pre-auth sub-millisecond response layer |
| **Localized Embedding Core** | FastEmbed (`BAAI/bge-small-en-v1.5`) | Thread-optimized local feature extractor |
| **High-Precision Reranker** | FlashRank Engine | Ultra-lightweight cross-encoder ranker |
| **Deterministic LLM Engine** | ChatOllama (Phi-3.5 Core Node) | Constrained local synthesis transformer |

---

## 📂 System Project Topology

```text
AegisMind AI/
│
├── run.py                 # System Orchestrator (Launches FastAPI Backend & Streamlit UI)
├── app.py                 # Streamlit System Dashboard & Telemetry Analytics UI
├── main.py                # Core FastAPI Engine, RAG Pipeline & Security Governance Layer
├── requirements.txt       # Hardened system dependencies
│
├── document_store.db      # SQLite3 File — Audit Logs & Structural Parent-Text Registry
├── chroma_db/             # ChromaDB Data — Multi-collection isolated vector spaces
├── faiss_cache.index      # FAISS Matrix — High-speed semantic cache index
└── cache_texts.pkl        # Pickle Interface — Pre-validated response data reference array
```

---

## 📡 API Integration Specification

AegisMind AI operates via a fully documented RESTful API layer. Every restricted interaction requires an active `X-Sat-Token` injected into the network request context.

| HTTP Method | Route Endpoint | Target Consumer Scope | Functional Execution |
|---|---|---|---|
| `GET` | `/health` | System Monitors | Pulls real-time engine health, collection counts, and thread allocation maps. |
| `GET` | `/me` | Identity Layer | Extracts user identity, operational metadata, and active role scope from token. |
| `POST` | `/users/create` | Admin Only | Onboards employees via strict regex filters and generates a unique secure SAT. |
| `GET` | `/users/list` | Admin Only | Pulls full registry data from directory store for corporate directory tracking. |
| `DELETE` | `/users/delete/{id}` | Admin Only | Decommissions access, wiping employee references from all active document ACLs. |
| `POST` | `/upload` | Admin Only | Streams multi-format payloads, enforces deduplication, and maps access matrices. |
| `POST` | `/query` | Authenticated | Executes deep semantic query traversal across cache, routed vector DB, and reranker. |
| `GET` | `/documents` | Authenticated | Pulls system inventory filtering for records explicitly matching user access. |
| `PATCH` | `/documents/{id}/access` | Admin Only | Updates real-time ACL matrix configuration settings for a given target file. |
| `DELETE` | `/cache` | Admin Only | Purges FAISS indexing layers to invalidate active cache memories on demand. |
| `GET` | `/audit` | Admin Only | Pulls chronologically tracked event trails from compliance ledger. |

---

## ⚡ Architectural Performance Profile & Hardware Tuning

- **On-Device Inference Optimization:** Running entirely on-device with zero cloud dependencies to ensure data privacy, raw pipeline inference execution on local CPU completes in 1.5–3 minutes under standard execution configurations. However, performance can be heavily accelerated by setting the `AEGISMIND_THREADS` environment variable to match the exact physical performance cores of the system, optimizing execution threading via Ollama without CPU thrashing.
- **Sub-Second Semantic Reuse:** Repeated or semantically equivalent questions (FAISS L2 boundary < 0.15) completely bypass the LLM processing chain, serving secure cached answers in <1 second.
- **Context Footprint Optimization:** By utilizing targeted multi-silo collection routing and precise parent-window text slicing rather than passing bloated document segments, global vector scanning overhead is reduced by roughly ~40%, drastically conserving local compute cycles.
- **Hardware Lifecycle Safeguards:** Disk serialization transactions for the FAISS cache index are bound strictly to application shutdown lifecycles, eliminating continuous per-query I/O writes and preventing premature SSD degradation.

---

## 🤝 Enterprise Access & Collaboration

Because AegisMind AI is a proprietary intellectual property built for enterprise compliance, the core implementation source code (`main.py`, `app.py`, vectors, and database configurations) is maintained within a secure, audited private repository.

**How to Request Access for Evaluation:**

If you are a technical recruiter, engineering lead, or prospective enterprise partner wishing to evaluate the source code, inspect the underlying database schemas, or discuss commercial integration:

- **Submit a Formal Request:** Please reach out through professional channels (LinkedIn/Email) stating your organization and purpose.
- **Repository Invitation:** Upon verification, your GitHub identity will be provisioned with temporary/permanent read-access as an authorized collaborator to the private production repository here: [Secure Private Repository Link]
- **Enterprise Inquiries:** For custom enterprise feature development or localized RAG deployment strategies, please submit a formal business brief.


### Engineered and Developed by Jai Verma