# Stone Inventory

Active stones as of 2026-05-27. Each stone is a Python module `execute(**kwargs) -> str (JSON)`.

## power/ — Executors

### browser_gateway
Chrome DevTools Protocol bridge.
```python
execute("navigate", url="https://example.com")
execute("screenshot")                             # base64 PNG
execute("click", selector="#submit-button")
execute("eval", expression="document.title")
execute("get_text", max_chars=4000)
execute("tabs")
```
Backend: `websocket-client` + Chrome CDP at `:9222`

### terminal_mcp_server  
subprocess wrapper with safety guards.
```python
execute("run", cmd="dir /b", cwd="D:\\path")
execute("run_bg", cmd="python server.py")         # returns pid
execute("which", name="python")
execute("env")                                    # safe keys only
execute("cwd")
```

### browser_mcp
Multi-session Chrome control + CAPTCHA detection.
```python
execute("sessions")                               # list active tabs as sessions
execute("new_session", url="https://...", name="task1")
execute("close_session", session_id="...")
execute("fill_form", fields={"#email": "x@y.z", "#pass": "secret"})
execute("captcha_check")                          # {found, signals, confidence}
execute("wait_for", selector=".loaded", timeout_sec=15)
```
Extends `browser_gateway`. Key fix: `/json/new` requires PUT not POST.

### open_computer_use
Docker sandbox for isolated code execution.
```python
execute("status")                                 # Docker availability check
execute("create", name="task1", image="ubuntu:22.04")
execute("exec", sandbox_id="task1", cmd="python3 -c 'print(42)'")
execute("list")
execute("destroy", sandbox_id="task1")
```
Graceful no-Docker fallback. `--network none` isolation, 512MB RAM cap.

---

## time/ — Memory & Persistence

### memory_core
Semantic memory with SQLite + HNSW.
```python
execute("save", key="lesson", value="text in any language", tags=["tag"], confidence=0.9)
execute("search", query="any language query", top_k=5, threshold=0.40)
execute("get", key="lesson")
execute("forget", key="lesson")
execute("stats")
```
Model: `all-MiniLM-L6-v2` (384-dim). **Embed value only, not key+value.**

---

## reality/ — Self-modification

### self_forge
Stone scaffolding engine.
```python
execute("templates")                              # list 6 templates
execute("generate", stone_id="my_tool", description="...", template="scraper")
execute("write", stone_id="my_tool", slot="power", description="...", template="bridge")
```
Templates: `transformer`, `monitor`, `scraper`, `writer`, `bridge`, `bare`

---

## space/ — Filesystem

### fs_ops
Safe filesystem operations.
```python
execute("read", path="/abs/path/file.txt", offset=0, limit=100)
execute("write", path="/abs/path/out.txt", content="data", append=True)
execute("search", path="/dir", pattern="TODO", recursive=True)
execute("tree", path="/dir", depth=3)
execute("exists", path="/abs/path")
execute("delete", path="/abs/path/file.txt", confirm=True)
```
Blocked: C:/Windows, C:/Program Files, system directories.

---

## mind/ — Knowledge

### doc_reader
Multi-format document reader. **No API key required.**
```python
execute("read", path="/abs/path/file.pdf", max_chars=8000)
execute("read", path="/abs/path/report.docx")
execute("read", path="/abs/path/data.xlsx")
execute("search", path="/abs/path/file.pdf", pattern="revenue")
execute("info", path="/abs/path/file.pdf")       # pages, author, size
execute("formats")                               # backend availability check
```
Backends: `pdftotext` (system) / `python-docx` / `openpyxl` / direct read

### vault_read / vault_append
Obsidian vault integration (read/write markdown notes).

---

## soul/ — Reasoning

### inferencesh
*(planned)* LLM API call inside the agent loop. Gives NAVIR autonomous reasoning.

---

## Stone Manifest Schema

```json
{
  "stone_id": "my_stone",
  "slot": "power",
  "version": "1.0.0",
  "protocol": "SIP/1",
  "runtime": "python3",
  "entry": "execute.py",
  "status": "active",
  "description": "...",
  "schema": {
    "input": {"type": "object", "properties": {"action": {"type": "string"}}, "required": ["action"]},
    "output": {"type": "string", "description": "JSON string"}
  }
}
```
