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

### compress_stone *(NEW — 2026-05-27)*
Context compression for NAVIR agent loops. Reduces tokens before LLM calls.
```python
execute("compress",                                   # compress all fields at once
    ocr_text="raw OCR ...", elements=[...], history=[...], messages=[...])
# → {ocr_text, elements, history, _engine, _ratio, _tokens_before, _tokens_after}

execute("ocr", text="raw OCR with noise ©®°•·...")   # OCR noise removal only
# → {text, before_chars, after_chars, saved_pct}

execute("history", history=[...], max_history=5)      # trim + summarize old entries
# → {history, before, after}

execute("stats", text="...")                          # token count estimate
# → {chars, approx_tokens, headroom_available, engine}
```
Engines: `navir_crush` (built-in, ~23% savings) → auto-upgrades to `headroom-ai` (60-95%) when installed.
Vendor path: `D:\clowd\Gauntlet\vendor` (add to sys.path)

---

## soul/ — Reasoning

### inferencesh *(NEW — 2026-05-27, VALIDATED)*
LLM reasoning inside the agent loop. Claude Haiku decides next UI action.
```python
execute("decide",                                         # next UI action
    goal="Navigate to login page",
    ocr_text="github.com | sign in | explore...",
    elements=[{"id":1,"type":"button","label":"Sign in","x":1755,"y":65}],
    history=[],
)
# → {"action":"click","target":"Sign in","reasoning":"...","confidence":0.99}

execute("judge",                                          # evaluate result
    criteria="User is on login page with password field",
    result_text="github.com/login ... password ... sign in button",
)
# → {"verdict":true,"reasoning":"...","confidence":0.95}

execute("plan",                                           # break goal into steps
    goal="Log in with email user@test.com password secret",
    ocr_text="username or email. password. sign in.",
    elements=[...],
)
# → {"steps":[{type:click,target:email},{type:type,text:user@test.com},...]}

execute("reflect",                                        # diagnose failure
    goal="...", ocr_text="...", history=[...]
)
# → {"diagnosis":"...","fix":{type:click,target:...},"confidence":0.9}
```
Model: `claude-haiku-4-5-20251001` (1-2s/call). Key: `D:\clowd\Gauntlet\secrets.json`
**Live test**: github.com → decide("click Sign in") → login page → decide("done") ✓

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


---

## soul/ � Reasoning

### vision_loop *(VALIDATED � 2026-05-27)*
See→Think→Act feedback loop. OCR + cv2, no API key. **Full cycle confirmed ✓**
```python
execute("windows")                                        # list visible windows
execute("see", window_title="Chrome")                     # {elements[], ocr_text}
execute("find", window_title="Chrome", label="Submit")    # {found, element}
execute("click", window_title="Chrome", label="Submit")   # click + SSIM verify
execute("check", window_title="Chrome", goal="text:OK")   # {reached: bool}
execute("loop",                                           # full autonomous loop
    window_title="Chrome",
    goal="text:Password",
    steps=[{"type": "click", "target": "Sign in", "wait_ms": 2500}],
    max_attempts=3,
    timeout_sec=30,
)
# LIVE TEST: status=success, attempts=1, ssim=changed px=2137 ✓
```
Goal DSL: `text:X` / `absent:X` / `type:X` / `ocr:X`
Cyrillic: `_cyr_to_ocr()` maps т→r, л→n, г→r (Tesseract Latin confusion)
Click: `SetCursorPos + mouse_event(0,0)` with `ClientToScreen + scale` coord mapping
Backend: `capture_window` + `detect_elements` (Tesseract+cv2) + SSIM `ClickVerifier`
NOTE: window must be maximized/restored — minimized returns 28x160 black bitmap
