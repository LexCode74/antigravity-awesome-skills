---
name: using-notebooklm-mcp
description: Expert guide for all NotebookLM MCP tools. Use when creating notebooks, adding sources, querying AI, generating audio/video/reports/flashcards/quizzes, running deep research, or managing NotebookLM via the MCP server. Covers all 32 available tools.
---

# NotebookLM MCP Tools

Comprehensive guide for the `notebooklm` MCP server — 32 tools organized by task.

## When to Use This Skill

- Creating, listing, renaming or deleting notebooks via MCP
- Adding sources (URLs, text, Google Drive docs)
- Querying AI about existing sources in a notebook
- Running deep web research and importing discovered sources
- Generating audio overviews, videos, infographics, reports, flashcards, quizzes, or mind maps

## Prerequisites

- NotebookLM MCP server connected (`notebooklm` server)
- Auth tokens valid — if tools fail with 401, run `mcp_notebooklm_refresh_auth`

---

## Tool Reference by Category

### 🔐 Authentication

| Tool | Purpose |
|------|---------|
| `mcp_notebooklm_refresh_auth` | Reload tokens from disk or re-authenticate headlessly |
| `mcp_notebooklm_save_auth_tokens` | FALLBACK: save cookies manually from Chrome DevTools |

**Auth workflow:**
```
1. Run notebooklm-mcp-auth in terminal  (preferred)
2. Call refresh_auth()                   (pick up new tokens)
3. If step 1 fails → save_auth_tokens(cookies="...")
```

---

### 📓 Notebook Management

| Tool | Key Args | Notes |
|------|----------|-------|
| `mcp_notebooklm_notebook_list` | `max_results=100` | Lists all notebooks |
| `mcp_notebooklm_notebook_create` | `title=""` | Returns `notebook_id` |
| `mcp_notebooklm_notebook_get` | `notebook_id` | Returns sources list |
| `mcp_notebooklm_notebook_describe` | `notebook_id` | AI summary + suggested topics |
| `mcp_notebooklm_notebook_rename` | `notebook_id`, `new_title` | |
| `mcp_notebooklm_notebook_delete` | `notebook_id`, `confirm=True` | ⚠️ IRREVERSIBLE |

```python
# Pattern: Create → Get ID → Add sources
nb = mcp_notebooklm_notebook_create(title="Physics EDOs")
notebook_id = nb["notebook_id"]
```

---

### 📎 Source Management

| Tool | Key Args | Notes |
|------|----------|-------|
| `mcp_notebooklm_notebook_add_url` | `notebook_id`, `url` | Websites or YouTube |
| `mcp_notebooklm_notebook_add_text` | `notebook_id`, `text`, `title=""` | Paste raw text |
| `mcp_notebooklm_notebook_add_drive` | `notebook_id`, `document_id`, `title`, `doc_type` | `doc_type`: doc\|slides\|sheets\|pdf |
| `mcp_notebooklm_source_describe` | `source_id` | AI summary + **bold** keywords |
| `mcp_notebooklm_source_get_content` | `source_id` | Raw indexed text (fast export) |
| `mcp_notebooklm_source_list_drive` | `notebook_id` | See Drive freshness status |
| `mcp_notebooklm_source_sync_drive` | `source_ids`, `confirm=True` | Sync stale Drive docs |
| `mcp_notebooklm_source_delete` | `source_id`, `confirm=True` | ⚠️ IRREVERSIBLE |

---

### 🤖 AI Querying

```python
# Query existing sources (NOT for finding new sources)
result = mcp_notebooklm_notebook_query(
    notebook_id="...",
    query="Explica el modelo de decaimiento radiactivo",
    source_ids=["id1", "id2"],    # Default: all sources
    conversation_id=None,          # For follow-up questions
    timeout=120.0
)
```

> [!IMPORTANT]  
> `notebook_query` only works on **existing** sources. For web search or finding new sources → use `research_start`.

```python
# Configure chat behavior
mcp_notebooklm_chat_configure(
    notebook_id="...",
    goal="custom",                 # default | learning_guide | custom
    custom_prompt="Act as a physics tutor...",  # max 10000 chars
    response_length="longer"       # default | longer | shorter
)
```

---

### 🔬 Deep Research

```python
# Workflow: research_start → research_status → research_import

# Step 1: Start research
task = mcp_notebooklm_research_start(
    query="Runge-Kutta methods for stiff ODEs",
    source="web",           # web | drive
    mode="fast",            # fast (~30s, ~10 sources) | deep (~5min, ~40 sources)
    notebook_id="...",      # optional — creates new if omitted
    title="New Notebook"    # for new notebook
)

# Step 2: Poll until complete
status = mcp_notebooklm_research_status(
    notebook_id="...",
    max_wait=300,           # seconds
    poll_interval=30,
    compact=True            # False for full detail
)

# Step 3: Import sources
mcp_notebooklm_research_import(
    notebook_id="...",
    task_id=task["task_id"],
    source_indices=None     # None = import all
)
```

---

### 🎨 Studio / Content Generation

All generation tools require `confirm=True` (get user approval first).

| Tool | Key Args | Notes |
|------|----------|-------|
| `mcp_notebooklm_audio_overview_create` | `notebook_id`, `format`, `length`, `language`, `focus_prompt` | `format`: deep_dive\|brief\|critique\|debate |
| `mcp_notebooklm_video_overview_create` | `notebook_id`, `format`, `visual_style`, `language` | `visual_style`: auto_select\|classic\|whiteboard\|kawaii\|anime\|watercolor\|retro_print\|heritage\|paper_craft |
| `mcp_notebooklm_infographic_create` | `notebook_id`, `orientation`, `detail_level` | `orientation`: landscape\|portrait\|square |
| `mcp_notebooklm_slide_deck_create` | `notebook_id`, `format`, `length` | `format`: detailed_deck\|presenter_slides |
| `mcp_notebooklm_report_create` | `notebook_id`, `report_format`, `custom_prompt` | `report_format`: "Briefing Doc"\|"Study Guide"\|"Blog Post"\|"Create Your Own" |
| `mcp_notebooklm_flashcards_create` | `notebook_id`, `difficulty` | `difficulty`: easy\|medium\|hard |
| `mcp_notebooklm_quiz_create` | `notebook_id`, `question_count`, `difficulty` | |
| `mcp_notebooklm_data_table_create` | `notebook_id`, `description` | |
| `mcp_notebooklm_mind_map_create` | `notebook_id`, `title` | |
| `mcp_notebooklm_studio_status` | `notebook_id` | Poll generation progress + URLs |
| `mcp_notebooklm_studio_delete` | `notebook_id`, `artifact_id`, `confirm=True` | ⚠️ IRREVERSIBLE |

```python
# Generation pattern
mcp_notebooklm_audio_overview_create(
    notebook_id="...",
    format="deep_dive",
    length="default",
    language="es",
    focus_prompt="Focus on RK4 method",
    confirm=True           # Only after user approval!
)

# Poll for result
mcp_notebooklm_studio_status(notebook_id="...")
```

---

## Validation

Before calling any tool:
- `notebook_id` and `source_id` are UUIDs — copy from `notebook_list` or `notebook_get`
- `confirm=True` tools require **explicit user approval** before calling
- `notebook_query` ≠ research — use `research_start` for web search
- After adding sources, wait for indexing before querying

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| 401 / Unauthorized | Expired auth tokens | Call `refresh_auth()` or re-run CLI auth |
| `notebook_id not found` | Wrong UUID | Call `notebook_list()` for correct IDs |
| `research_status` timeout | Deep mode slow | Increase `max_wait=600`, use `mode="fast"` |
| `confirm=False` error | Missing explicit approval | Get user approval first, then pass `confirm=True` |
| Query returns empty | Sources not indexed | Wait 30-60s after `add_url`/`add_text`, then retry |
| `source_id not found` | Wrong UUID | Call `notebook_get(notebook_id)` for source list |

## Resources

- See [ADVANCED.md](./ADVANCED.md) for: full parameter tables per tool, BCP-47 language codes, Drive doc_type guide
- Companion skill: `grounding-with-notebooklm` for scientific grounding workflow
---
*Antigravity Global Skills — NotebookLM MCP*
