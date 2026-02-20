---
name: coding-n8n-python
description: Write Python code in n8n Code nodes. Use when writing Python in n8n, using _input/_json/_node syntax, working with standard library, or need to understand Python limitations in n8n Code nodes.
---

# Python Code Node (Beta)

Expert guidance for writing Python code in n8n Code nodes.

## When to Use This Skill

- Writing Python in an n8n Code node
- Need `statistics`, `re`, `datetime`, or other standard library modules
- Understanding the difference between Python (Beta) and Python (Native) modes
- Debugging `KeyError`, return format, or webhook data errors in Python

## Prerequisites

- n8n instance running
- Code node → language set to **Python (Beta)** (recommended over Native)

> [!IMPORTANT]  
> **Use JavaScript for 95% of cases.** Python has no external libraries (no `requests`, `pandas`, `numpy`). Switch to JS if you need HTTP or date/time helpers.

## Workflow

### Step 1: Choose Mode

| Mode | Variables | Use when |
|------|-----------|----------|
| **Python (Beta)** ✅ | `_input`, `_json`, `_now` | Most use cases |
| **Python (Native)** | `_items`, `_item` | Pure Python only |

### Step 2: Access Data

```python
# All items
all_items = _input.all()

# First item
first = _input.first()["json"]

# Webhook data — ALWAYS under ["body"]
email = _json["body"]["email"]          # ✅
# email = _json["email"]              # ❌ KeyError

# Safe access
email = _json.get("body", {}).get("email", "no-email")

# Reference another node
from_node = _node["HTTP Request"]["json"]
```

### Step 3: Return Correct Format

```python
# ✅ Required: List of dicts with "json" key
return [{"json": {"field": value}}]

# ✅ List comprehension
return [{"json": {"id": item["json"]["id"]}} for item in _input.all()]

# ❌ Wrong: dict without list
return {"json": {"field": value}}

# ❌ Wrong: no "json" key
return [{"field": value}]
```

### Step 4: Standard Library Only

```python
# ✅ Available
import json, re, base64, hashlib, math, random, statistics
from datetime import datetime, timedelta
import urllib.parse

# ❌ NOT available (ModuleNotFoundError)
import requests, pandas, numpy, scipy
```

**Need HTTP?** → Use HTTP Request node before Code, or switch to JavaScript.

### Step 5: Guard Against Errors

```python
# Safe dict access
name = item["json"].get("name", "Unknown")

# None guard
amount = item["json"].get("amount") or 0

# Validate input first
items = _input.all()
if not items:
    return []
```

## Validation

Run before deploying:
```python
# Checklist (add as comment):
# ✅ returns list format [{"json": {...}}]
# ✅ webhook data via ["body"]
# ✅ .get() used for optional fields
# ✅ no external library imports
# ✅ all code paths have return
```

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `ModuleNotFoundError: requests` | External lib not available | Use HTTP Request node or switch to JS |
| `KeyError` on webhook field | Missing `["body"]` | Use `_json["body"]["field"]` |
| Return not a list | Missing `[...]` wrapper | Wrap with `return [{"json": ...}]` |
| `_input` not defined | Using Native mode | Switch to Python (Beta) mode |
| `NoneType is not subscriptable` | `None` dict access | Use `.get()` with default value |

## Resources

- See [ADVANCED.md](./ADVANCED.md) for: 10 production patterns, stats analysis, regex, data validation
- n8n Python docs: https://docs.n8n.io/code/builtin/python-modules/
---
*Antigravity Global Skills — n8n Python Code Node*
