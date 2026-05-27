# NAVIR GitHub Scout Report
**Date:** 2026-05-27  
**Method:** GitHub API + inferencesh.decide + browser_gateway DOM extraction  
**Query:** mcp server python agent | autonomous agent memory | screenshot ocr automation

---

## TOP 5 ЗОЛОТО ДЛЯ NAVIR

### #1 chopratejas/headroom — 2041★ — CRITICAL
**Gap:** inferencesh витрачає токени на довгі OCR/history контексти  
**Що дає:** 60-95% менше токенів, 0% втрати точності (GSM8K ±0.000)  
**Install:** `pip install "headroom-ai[all]"`  
**Інтеграція:** обгортка над inferencesh — стискати ocr_text + history перед відправкою до Haiku  
**Stone:** `compress_stone` в slot=mind  
**Effort:** 2-3 дні  

### #2 Dataojitori/nocturne_memory — 1149★ — HIGH
**Gap:** memory_core не має rollback між сесіями  
**Що дає:** версіонована пам'ять з checkpoint/rewind, PostgreSQL backend  
**Install:** `pip install nocturne-memory`  
**Інтеграція:** upgrade memory_core.py — Nocturne як persistence layer  
**Stone:** upgrade існуючого memory_core  
**Effort:** 5-7 днів  

### #3 ArcadeAI/arcade-mcp — 901★ — MEDIUM
**Gap:** Gauntlet stones ізольовані, не сумісні з екосистемою MCP  
**Що дає:** стандартний MCP Tool interface → stones = MCP tools для будь-якого клієнта  
**Install:** `pip install arcade-mcp`  
**Інтеграція:** self_forge → генерує Arcade-сумісні stones  
**Stone:** upgrade self_forge templates  
**Effort:** 4-6 днів  

### #4 Cranot/roam-code — 466★ — MEDIUM  
**Gap:** немає code-aware indexing для автономного дебагу  
**Що дає:** 238 code intelligence команд, SQLite code graph, 28 мов  
**Install:** `pip install roam-code` (або `git clone`)  
**Інтеграція:** новий stone `codebase_intel` в slot=mind  
**Stone:** `codebase_intel` — новий stone  
**Effort:** 3-4 дні  

### #5 golf-mcp/golf — 829★ — MEDIUM
**Gap:** немає observability/трейсинг для production  
**Що дає:** Auth, structured logs, debugger, testing harness  
**Install:** `pip install golf-mcp`  
**Інтеграція:** `@traced` декоратор на всі stones  
**Stone:** `observability` middleware  
**Effort:** 3-5 днів  

---

## ВІДХИЛЕНО

| Repo | Причина |
|------|---------|
| rocketride-server 3.4k★ | C++ core, складна інтеграція, не Python stone |
| 0xSteph/pentest-ai 541★ | Security niche, не core NAVIR use case |
| coleam00/mcp-mem0 675★ | Mem0 потребує API ключ, nocturne_memory краще |

---

## HEADROOM KILLER FEATURE для NAVIR

```python
# До headroom: inferencesh відправляє 10k токенів
inf.execute("decide", goal=..., ocr_text=BIG_OCR, history=LONG_HISTORY)

# Після headroom: 1k токенів, та сама відповідь
from headroom import compress
compressed = compress({"ocr": ocr_text, "history": history})
inf.execute("decide", goal=..., **compressed)  # 90% дешевше
```

`headroom learn` автоматично мінить failed sessions і пише fix до CLAUDE.md — 
це частково вирішує проблему самовдосконалення без ручного налаштування.

---

## НАСТУПНІ ДІЇ

1. **ЗАРАЗ:** `pip install "headroom-ai[all]"` (довго, пустити overnight)  
2. **ДЕНЬ 1:** compress_stone — обгортка над inferencesh + headroom  
3. **ДЕНЬ 2:** тест — чи справді 70%+ economy на NAVIR mission  
4. **ТИЖДЕНЬ 1:** nocturne_memory upgrade для rollback між сесіями  
