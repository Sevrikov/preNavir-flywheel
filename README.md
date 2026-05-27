# preNavir Flywheel

> An autonomous tool-discovery and integration engine — the self-reinforcing loop that powers NAVIR.

```
SCOUT ──→ SCORE ──→ FORGE ──→ TEST
  ↑                              │
  └──── RECALL ←── LEARN ←───────┘
```

Each revolution of the flywheel: the system **finds** a new tool, **evaluates** it, **builds** a stone (integration layer), **tests** it, **remembers** the result — and uses that memory to make the next revolution smarter.

---

## What is this?

**preNavir Flywheel** is the foundational infrastructure layer for an autonomous AI agent system built on two engines:

| Engine | Role |
|--------|------|
| **Gauntlet** | Rust MCP server with hot-reloadable Python "stones" in named slots |
| **NAVIR** | Python CLI scavenger — finds tools, scores them, promotes to stones |

Together they form a flywheel: NAVIR discovers capabilities and feeds them into Gauntlet. Gauntlet gives NAVIR more powerful tools. The loop accelerates.

---

## Architecture

```
GAUNTLET (Rust MCP server, SIP/1 protocol)
├── stones/
│   ├── power/          ← Executors (browser, terminal, sandbox)
│   │   ├── browser_gateway     Chrome CDP: navigate/screenshot/click/eval
│   │   ├── terminal_mcp        subprocess: run/run_bg/which/env
│   │   ├── browser_mcp         Multi-session Chrome + CAPTCHA detection
│   │   └── open_computer_use   Docker sandbox (isolated execution)
│   ├── time/           ← Memory & persistence
│   │   └── memory_core         SQLite + HNSW (all-MiniLM-L6-v2, 384-dim)
│   ├── reality/        ← Self-modification
│   │   └── self_forge          Stone scaffolding engine (6 templates)
│   ├── space/          ← Filesystem
│   │   └── fs_ops              Read/write/search/tree with safety guards
│   ├── mind/           ← Knowledge & understanding
│   │   └── doc_reader          PDF/DOCX/XLSX/TXT reader (no API key)
│   └── soul/           ← Inference & reasoning
│       └── inferencesh         (planned: LLM API call inside the loop)

NAVIR (Python CLI, D:\Поиск золотишка\navir\)
├── core/
│   ├── source_scout.py     MCP registry scanner (25 domain queries)
│   └── artifact_to_stone.py  Promotion pipeline
├── skills/
│   ├── recall.py           Recon: semantic memory lookup before task
│   ├── learn.py            Consolidate: save results to memory_core
│   ├── verify.py           Execute: visual confirmation via OrbisVision
│   ├── patrol.py           Recon: system health + threat detection
│   ├── scout.py            Execute: source_scout wrapper
│   ├── forge.py            Consolidate: self_forge wrapper
│   └── runner.py           SkillRunner + 6 battle protocols
└── test_protocol.py        5-tier validation (96.2% idealism score)
```

---

## The Stone System (SIP/1 Protocol)

Each **stone** = a directory with two files:
- `execute.py` — Python module with `execute(**kwargs) -> str` (returns JSON)
- `stone_manifest.json` — metadata (stone_id, slot, status, schema)

Gauntlet hot-reloads stones without restart. Status: `draft` → `active`.

### Stone lifecycle

```
1. NAVIR scouts artifact (GitHub / MCP Registry / HuggingFace)
2. Score with navir_score (0–100): TAKE ≥ 72 / WATCH ≥ 45 / GRAVEYARD < 45
3. artifact_to_stone.py generates draft execute.py + manifest
4. Human or self_forge fills the implementation
5. Smoke test (python execute.py)
6. Manifest status: draft → active
7. test_protocol.py Tier 4 validates all active stones
```

---

## Combat Skills & Battle Protocols

Skills follow a three-phase battle rhythm:

| Phase | Skills | Purpose |
|-------|--------|---------|
| RECON | `recall` + `patrol` | Memory lookup + health check before acting |
| EXECUTE | `scout` + `verify` | Find new tools + visual confirmation |
| CONSOLIDATE | `learn` + `forge` | Persist knowledge + build new capability |

### Protocols (predefined skill chains)

```python
PROTOCOLS = {
    "before_task":    ["recall", "patrol"],
    "after_action":   ["verify"],
    "after_task":     ["learn"],
    "scout_cycle":    ["patrol", "scout", "learn"],
    "full_cycle":     ["recall", "patrol", "scout", "learn"],
    "new_capability": ["recall", "forge", "learn"],
}
```

---

## Memory Layer

`memory_core` uses **SQLite + HNSW** (usearch library) for semantic search:

```python
store.save("key", "value in any language", tags=["tag1"], confidence=0.9)
results = store.search("query in any language", top_k=5, threshold=0.40)
```

**Key insight:** embed only the `value`, never `key + value`. Mixing languages in one embedding vector degrades cosine similarity below threshold.

---

## Gold Score (navir_score)

Multi-signal scoring for artifact evaluation:

```
navir_score = leverage×2.2 + maturity×1.8 + strategic×2.4 
            + (11-risk)×1.4 + (11-difficulty)×1.2
```

Signals that boost score:
- Matches `_HIGH_VALUE_KW`: browser, automation, vision, sandbox, terminal, memory, embedding, scraper…
- GitHub stars: ≥1000 → +5pts, ≥200 → +3pts, ≥50 → +1pt
- Has GitHub repo, MIT/Apache license
- MCP protocol present in description

Signals that lower score:
- Matches `_LOW_VALUE_KW`: google ads, forex, crm, salesforce…
- No repo URL → risk +2

---

## Test Protocol (5-Tier Validation)

```
T0 Infrastructure    ORBIS :8000, Chrome CDP :9222, Gauntlet binary, Python deps
T1 Stone Isolation   memory_core, vision, self_forge, fs_ops — import + smoke
T2 Skill Isolation   Each skill runs independently
T3 Protocol Chain    Skill chains execute correctly
T4 End-to-End        All manifests valid, all execute.py compile, ORBIS health
─────────────────────────────────────────────────────────────────────────────
IDEALISM SCORE: 96.2%  (25 PASS / 0 FAIL / 1 WARN / 2 SKIP)
```

---

## Key Lessons

| Bug | Root Cause | Fix |
|-----|-----------|-----|
| Cross-lingual memory search → 0 results | `embed(key + value)` mixed languages | `embed(value)` only |
| All artifacts score 68.4 | 5 binary signals → no differentiation | keyword + popularity scoring |
| MCP registry search → 0 results | multi-word queries not supported | single-word queries only |
| Stone files at wrong path | `gauntlet_dir=root` vs `stones/` | corrected path nesting |
| Generated stones IndentationError | `textwrap.dedent(f'...')` + `textwrap.fill()` inside f-string | explicit string concat |
| browser_mcp new_session HTTP 405 | Chrome CDP `/json/new` needs PUT not POST | `method="PUT"` |

---

## Current Status (2026-05-27)

- **17 active stones** across 5 slots
- **74 artifacts** in discovery database
- **6 skills** + SkillRunner + COMBAT_DOCTRINE
- **96.2% idealism score** — infrastructure is real, not theoretical
- **One completed mission:** PDF tools → doc_reader stone (end-to-end autonomy test)

---

## What's Next

1. `browse` skill — wraps `browser_gateway` for autonomous web navigation
2. Scheduled `full_cycle` — true autonomous loop without human trigger
3. `inferencesh` soul stone — LLM API call inside the agent loop (give the system a brain)
4. `gnosis_memory` — alternative vector store, compare with memory_core

---

## Related

- [NAVIR source](D:\Поиск золотишка\navir\) — scavenger engine
- [Gauntlet stones](D:\clowd\Gauntlet\stones\) — integration layer
- [COMBAT_DOCTRINE.md](D:\Поиск золотишка\navir\COMBAT_DOCTRINE.md) — battle manual
