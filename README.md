# GitMind — Persistent Engineering Intelligence
### Powered by IBM Db2 Cloud & LangGraph Multi-Agent Orchestration

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-brightgreen.svg)](https://python.org)
[![IBM Db2](https://img.shields.io/badge/IBM%20Db2-v12.1.2%2B-blue.svg)](https://www.ibm.com/products/db2)
[![LangGraph](https://img.shields.io/badge/Orchestration-LangGraph-purple.svg)](https://github.com/langchain-ai/langgraph)
[![Tests: 28 Passing](https://img.shields.io/badge/Tests-28%20Passing%20(100%25)-success.svg)](tests/)

---

## 1. The Real-World Problem

Modern engineering teams suffer from acute institutional amnesia:
- **Lost Context:** Why was an unusual architectural decision or workaround introduced? Commit messages are frequently cryptic (`fix bug`, `updates`), and the original developers may have left the organization.
- **Repeated Regressions:** Developers inadvertently re-introduce flaws that were previously investigated and reverted months or years earlier.
- **Fragmented Tooling:** Git commits, Tree-sitter AST syntax, pull requests, issues, and code reviews live in disconnected silos.
- **Conventional RAG Limitations:** Generic RAG chatbots lack repository knowledge graphs, invent commit SHAs and line numbers, lack repository isolation, and forget past engineering investigations across sessions.

---

## 2. The Solution: GitMind

**GitMind** transforms software repositories into a persistent, searchable, and auditable engineering intelligence system powered by **IBM Db2**.

Rather than treating code as loose text snippets, GitMind constructs a unified **Repository Knowledge Graph** and **Multi-Agent Memory** inside IBM Db2, connecting:
- Commit histories and diffs
- Tree-sitter function and class AST symbols
- Pull requests and code reviews
- GitHub issues and bug discussions
- Prior agent findings and regression alerts

Every AI conclusion is verified by a dedicated **Evidence Verification Layer** against actual Db2 records, distinguishing verified facts from historical inferences.

---

## 3. Key Differentiators (Built on IBM Db2)

1. **Repository Knowledge Graph (`GITMIND_RELATIONS`):**  
   Deterministic relational mapping connecting `COMMIT -> TOUCHES_FILE`, `COMMIT -> MODIFIES_FUNCTION`, `COMMIT -> PARENT_COMMIT`, and `PR -> ISSUE` inside IBM Db2.
2. **Persistent Multi-Agent Engineering Memory (`GITMIND_FINDINGS`):**  
   Findings discovered by agents during past investigations are persisted in Db2 with supporting evidence, allowing subsequent runs to immediately recall institutional precedents.
3. **Signature Differentiator: Historical Change-Risk Intelligence (`gitmind review`):**  
   Analyzes a developer's current branch/working diff, extracts AST-modified functions, searches Db2 for past reverts, bug fixes, and prior findings, and computes an explainable, deterministic risk assessment before merging.

---

## 4. The Seven Specialist Agents

GitMind coordinates seven specialized LangGraph agents using `@entrypoint` and `@task` functional workflow patterns:

| Agent | Responsibility | Key Tools | Primary Persistence |
|---|---|---|---|
| **History Agent** | Reconstructs commit timelines, feature evolution, and author blame | `get_commit_history`, `get_commit_details`, `semantic_search` | `GITMIND_ARTIFACTS` + `GITMIND_VECTORS` |
| **Code Agent** | Semantic AST analysis of functions, classes, and dependency changes | `tree-sitter` AST parsers, `CODE_TOOLS` | `GITMIND_ARTIFACTS` (`function`/`class`) |
| **Recovery Agent** | Scans reflog and dangling objects to construct safe recovery plans | `get_reflog`, `get_dangling_objects`, `get_stash_list` | Read-only with human confirmation |
| **GitHub Agent** | Traces human intent across commits, PRs, reviews, and issues | `PyGithub`, `get_pr_context`, `search_issues` | `GITMIND_ARTIFACTS` (`pr`/`issue`) |
| **Suggest Agent** | Proactive health audit: unpushed commits, stale branches, push risks | `detect_force_push_risk`, `get_unpushed_commits` | Dynamic repo state + Db2 |
| **Project Agent** | Explains repository architecture, tech stack, and onboarding | `get_project_description`, `semantic_search` | `GITMIND_ARTIFACTS` (`doc`) |
| **Docs Agent** | Surfaces verified Git command documentation and syntax help | `get_git_command_help`, `get_git_config_docs` | Contextual documentation |

---

## 5. IBM Db2 Integration Architecture

GitMind leverages IBM Db2 as its central operational backbone:

```
Developer CLI / PR
        │
        ▼
LangGraph Supervisor / Router
        │
┌───────┴───────────────────────────────────────┐
│ Seven Specialist Agents (History, Code, etc.) │
└───────┬───────────────────────────────────────┘
        │
        ▼
Evidence Verification Layer (Cross-checks Db2)
        │
        ▼
Shared Repository Intelligence Interface
        ├── Relational Query Service (ibm_db_dbi)
        ├── Db2 Vector Retrieval Service (langchain-db2 DB2VS)
        ├── Agent Memory Service (GITMIND_FINDINGS)
        └── Append-Only Audit Service (GITMIND_AUDIT_EVENTS)
        │
        ▼
IBM Db2 Database (Cloud / Community Edition v12.1.2+)
        ├── GITMIND_REPOSITORIES
        ├── GITMIND_ARTIFACTS
        ├── GITMIND_RELATIONS (Knowledge Graph)
        ├── GITMIND_VECTORS (Native VECTOR + VECTOR_DISTANCE)
        ├── GITMIND_INDEX_RUNS
        ├── GITMIND_AGENT_RUNS
        ├── GITMIND_FINDINGS
        ├── GITMIND_FINDING_EVIDENCE
        └── GITMIND_AUDIT_EVENTS
```

### Db2 Features Utilized:
- **Official `langchain-db2` DB2VS Vector Store:** Native vector operations using `DB2VS(embedding_function, table_name, client, distance_strategy=DistanceStrategy.COSINE)`.
- **Native Vector Data Types:** Tables use native Db2 `vector(dim, FLOAT32)` columns.
- **Server-Side Distance:** Cosine similarity is calculated in-database via `VECTOR_DISTANCE(..., COSINE)`.
- **Binary JSON Metadata:** Metadata is stored via `SYSTOOLS.JSON2BSON` and retrieved via `SYSTOOLS.BSON2JSON`.
- **Multi-Tenant Isolation:** All relational queries and vector lookups are strictly filtered by repository ID (`REPO_ID`), guaranteeing zero data leakage.

---

## 6. Installation & Quick Start

### 6.1 Prerequisites
- Python 3.10, 3.11, 3.12, or 3.13
- IBM Db2 Cloud or Community Edition instance (v12.1.2+)
- Git installed on PATH

### 6.2 Setup
```bash
# 1. Clone the repository
git clone https://github.com/vastavikadi/gitmind-va.git
cd gitmind-va

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment
cp .env.example .env
# Edit .env with your IBM Db2 host, credentials, and LLM API keys
```

### 6.3 Configuration (`.env`)
```env
STORAGE_BACKEND=db2

DB2_DATABASE=BLUDB
DB2_HOST=your-db2-host.databases.appdomain.cloud
DB2_PORT=50001
DB2_USERNAME=your-username
DB2_PASSWORD=your-password
DB2_SSL=true

DB2_SCHEMA=GITMIND
DB2_VECTOR_TABLE=GITMIND_VECTORS

LLM_PROVIDER=gemini
GEMINI_API_KEY=your-api-key
GEMINI_MODEL=gemini-2.5-flash

EMBEDDING_PROVIDER=gemini
EMBEDDING_MODEL=models/gemini-embedding-001
EMBEDDING_DIMENSION=768

GITHUB_TOKEN=your-github-token
```

---

## 7. CLI Commands & Usage

### 7.1 Database Management (`db2`)
```bash
# Run diagnostics on Db2 connection and native vector capabilities
gitmind db2 doctor

# Test round-trip latency to Db2
gitmind db2 test

# Apply versioned relational migrations
gitmind db2 migrate

# View indexed repository intelligence statistics
gitmind db2 status
```

### 7.2 Indexing (`index`)
```bash
# Incremental indexing (indexes new commits since last indexed HEAD)
gitmind index

# Full rebuild
gitmind index --full

# Index custom history depth
gitmind index --days 30
```

### 7.3 Signature Feature: Historical Change-Risk Review (`review`)
```bash
# Review current working tree against main branch
gitmind review --base main

# Review against origin/main
gitmind review --base origin/main
```
**Example Output:**
```
┌───────────────────── Change Risk Review (Base: main) ──────────────────────┐
│                                                                             │
│                 GITMIND CHANGE REVIEW — HIGH HISTORICAL RISK                │
│                                                                             │
│  Repository: vastavikadi/gitmind | Base: main -> Head: f49be67c |           │
│  Risk Score: 45.0/100                                                       │
│                                                                             │
│  Changed Components                                                         │
│   • src/auth/token_service.py                                               │
│   • Modified Functions/Classes: rotate_token, authenticate                  │
│                                                                             │
│  Detected Risk Signals (Deterministic Model)                                │
│   • Security-Sensitive Logic Modified (+25.0 pts): Authentication touched   │
│   • Past Bugfix Precedent in Component (+20.0 pts): File had prior fixes    │
│                                                                             │
│  Grounded Historical Evidence (IBM Db2)                                     │
│   1 [VERIFIED] Commit abc1234 — Verified bugfix for token expiration        │
│   2 [VERIFIED] PR #67 — Enforced rotation boundary limits                   │
│                                                                             │
│  Actionable Recommendations                                                 │
│   • Require peer security review before merging.                            │
│   • Compare refresh-token validation against historical fix in abc1234.     │
│                                                                             │
│  Review ID: 9A3E517D-7A4D-41 | Backed by IBM Db2 Multi-Agent Memory         │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.4 Evidence-Grounded Querying (`ask`)
```bash
# Ask single-shot question
gitmind ask "Why was authentication introduced and how did it evolve?"

# Interactive chat session
gitmind ask --chat
```

### 7.5 Repository Storytelling & Explanation
```bash
# Tell the story of what was built over the last 14 days
gitmind story --days 14 --detailed

# Explain a specific commit hash, file, or feature
gitmind explain f49be67
gitmind explain cli.py

# Project onboarding overview
gitmind project
```

### 7.6 Persistent Memory Search
```bash
# Search previous engineering findings stored in Db2
gitmind memory search "token expiration regression"
```

---

## 8. Benchmark Evaluation Results

The system was evaluated against a 20-question golden evaluation benchmark covering historical reasoning, regression recall, and multi-tenancy boundaries:

| Evaluation Metric | Target Threshold | Measured Score | Status |
|---|---|---|---|
| **Retrieval Hit@5** | >= 80.0% | **100.0%** (20/20) | **PASS** |
| **Mean Reciprocal Rank (MRR)** | >= 0.700 | **0.858** | **PASS** |
| **Factual Claim Grounding** | 100.0% | **100.0%** | **PASS** |
| **Cross-Repository Leakage** | 0.0% | **0.0% (Zero)** | **PASS** |
| **Secret Redaction Rate** | 100.0% | **100.0%** | **PASS** |
| **Automated Test Suite** | 100% Pass | **28/28 Passing** | **PASS** |

Run tests:
```bash
python -m unittest discover -s tests -p "test_*.py" -v
```

---

## 9. Security & Trust Architecture

- **Zero Secret Ingestion:** Scans and redacts credentials (API keys, JWT tokens, private keys) before embedding or LLM transmission.
- **File Blacklisting:** Excludes `.env`, credentials, certificates, binaries, and build artifacts from indexing.
- **Non-Destructive Git Guardrails:** Destructive Git operations (`reset --hard`, `push --force`, `branch -D`) are blocked by default.
- **Parameterized SQL:** 100% of queries use parameter binding to eliminate SQL injection risks.
- **Immutable Audit Trail:** `GITMIND_AUDIT_EVENTS` logs all indexing, agent executions, and reviews for governance.

---

## 10. Limitations

- GitHub PR and Issue intent tracing requires an optional `GITHUB_TOKEN`. If not configured, GitMind gracefully operates on local Git and Tree-sitter intelligence.
- Very large binary assets or minified JavaScript bundles are excluded by design to prevent noise in the vector store.
- Change-risk estimation relies on indexed commit depth; repositories with shallow clones (`--depth 1`) provide limited historical comparison.

---

## 11. License & Attribution

GitMind is licensed under the [MIT License](LICENSE).  
Original GitMind created by [Aditya Shukla](https://github.com/vastavikadi/gitmind-va).  
Enhanced for the **IBM Db2 TechX Build Challenge 2026**.
