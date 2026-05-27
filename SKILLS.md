# Combat Skills & Battle Doctrine

## The Six Skills

All skills follow `SkillBase` interface → return `SkillResult(status, data, next_action, elapsed_ms, notes)`.

```
status:      "ok" | "empty" | "error" | "skipped"
next_action: "continue" | "stop" | "retry"
```

### recall — Recon Phase
Search memory before acting. Injects results into context.
```python
skill.run(context, query="what do we know about playwright?", top_k=5)
# → context["recalled"] = [{key, value, score, tags}, ...]
```

### learn — Consolidate Phase
Save results to memory_core after completing work.
```python
skill.run(context, key="playwright_stone", value="implemented 2026-05-27...",
          tags=["stone", "browser"], outcome="success")
```
Auto-adds "success"/"error" tag from `outcome`.

### verify — Execute Phase
Visual confirmation via OrbisVision. Takes screenshot on "unchanged".
```python
skill.run(context, hwnd=12345, action_desc="clicked submit button")
# → context["verify_screenshot"] = "/path/to/screenshot.png" if unchanged
```

### patrol — Recon Phase
System health monitoring. Reads error_log.jsonl + stone_queue.jsonl.
```python
skill.run(context)
# → {threat_level: "LOW"|"MEDIUM"|"HIGH", patterns: {...}, draft_stones: N}
```
Threat thresholds: unknown_errors>5 → HIGH, rate_limit>10 → MEDIUM, draft_stones>5 → LOW

### scout — Execute Phase
Source discovery via source_scout.py. Wrapped with error_interceptor.
```python
skill.run(context, source="mcp_registry", limit=20)
# → {artifacts_found: N, top_candidates: [...]}
```

### forge — Consolidate Phase
Build new stone via self_forge. Complete new capability in one call.
```python
skill.run(context, stone_id="web_crawler", slot="power",
          description="Crawl URLs and extract text", template="scraper")
# → writes execute.py + stone_manifest.json to stones/power/web_crawler/
```

---

## Battle Protocols

```python
from skills.runner import runner

runner.protocol("before_task", context)          # recall + patrol
runner.protocol("after_action", context, hwnd=X) # verify
runner.protocol("after_task", context, key=..., value=...)  # learn
runner.protocol("scout_cycle", context)          # patrol + scout + learn
runner.protocol("full_cycle", context)           # recall + patrol + scout + learn
runner.protocol("new_capability", context, stone_id=..., slot=..., description=...)
```

## Battle Rhythm

```
BEFORE TASK     recall: "что мы знаем об этой задаче?"
                patrol: "система здорова?"
                         ↓
EXECUTE         scout: "что нового в мире?"
                verify: "визуальное подтверждение действия"
                         ↓
AFTER TASK      learn: "что узнали? сохранить."
                forge: "нужен новый stone? создать."
```

## SkillRunner API

```python
from skills.runner import runner

# Single skill
result = runner.run("patrol", context)

# Chain (sequential, stops on "stop")
results = runner.chain(["recall", "patrol", "scout"], context, limit=10)

# Named protocol
results = runner.protocol("full_cycle", context)
```

## COMBAT_DOCTRINE.md (summary)

**Forbidden actions:**
- Never forge a stone without recall first (check if it already exists)
- Never learn without a key + value (partial saves corrupt memory)  
- Never skip patrol in production missions
- Never promote artifact with gold_score < 72 (TAKE threshold)

**Threat response:**
- HIGH threat → stop scout, run learn, notify
- MEDIUM threat → proceed with caution, double verify
- LOW threat → note, continue mission
