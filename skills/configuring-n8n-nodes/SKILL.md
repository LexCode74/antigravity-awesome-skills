---
name: configuring-n8n-nodes
description: Operation-aware node configuration guidance. Use when configuring nodes, understanding property dependencies, determining required fields, choosing between get_node detail levels, or learning common configuration patterns by node type.
---

# n8n Node Configuration

Expert guidance for operation-aware node configuration with property dependencies.

## When to Use This Skill

- Configuring a node and unsure which fields are required
- Field visibility changes based on operation (e.g., POST vs GET)
- Choosing between `detail: "standard"`, `"full"`, or `mode: "search_properties"`
- Understanding why validation says a field is missing

## Prerequisites

- Node type identified (use `search_nodes` if needed)
- n8n-mcp MCP server connected

## Workflow

### Step 1: Get Node Info (Standard First)

```javascript
// Default: detail="standard" — covers 95% of use cases
get_node({nodeType: "nodes-base.slack"})

// Only if standard is insufficient
get_node({nodeType: "nodes-base.slack", detail: "full"})

// Search for a specific property
get_node({nodeType: "nodes-base.httpRequest", mode: "search_properties", propertyQuery: "auth"})
```

### Step 2: Respect Operation Context

> [!IMPORTANT]  
> **Resource + operation determine which fields are required.**  
> Don't copy configs between operations without checking!

```javascript
// Slack post message
{resource: "message", operation: "post", channel: "#general", text: "Hi!"}

// Slack update message — DIFFERENT fields!
{resource: "message", operation: "update", messageId: "1234", text: "Updated!"}
```

### Step 3: Respect Property Dependencies

Fields appear/disappear based on other field values:

```javascript
// HTTP Request GET — no body fields shown
{method: "GET", url: "https://api.example.com"}

// HTTP Request POST — sendBody + body now required
{method: "POST", url: "https://api.example.com", sendBody: true,
 body: {contentType: "json", content: {name: "John"}}}
```

### Step 4: Validate Iteratively

```
Configure minimal → validate_node → read error → add field → validate again
(expect 2-3 cycles — this is normal)
```

```javascript
validate_node({nodeType: "nodes-base.slack",
               config: {resource: "channel", operation: "create", name: "general"},
               profile: "runtime"})
```

### Step 5: Trust Auto-Sanitization

For IF/Switch nodes, `singleValue` is auto-fixed on save:
- Binary operators (equals, contains) → `singleValue` removed
- Unary operators (isEmpty, isNotEmpty) → `singleValue: true` added

Do **not** manually set `singleValue` — let the system handle it.

## Validation

Pre-deployment checklist:
- Started with `detail: "standard"` (not "full")
- Correct resource/operation pair chosen
- All conditional dependencies satisfied (e.g., `sendBody: true` when POST)
- `validate_node` passes with `profile: "runtime"`

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `field X is required` | Missing conditional dep | Check displayOptions with `mode: "search_properties"` |
| Config from different operation | Copied wrong config | `get_node` for new operation, start fresh |
| `singleValue` wrong | IF operator mismatch | Don't set manually; let auto-sanitization fix |
| Large payload | Used `detail: "full"` | Switch to `detail: "standard"` (default) |
| Auth field missing | `sendAuth: true` not set | Follow dependency chain from GET → auth |

## Resources

- See [ADVANCED.md](./ADVANCED.md): complete configuration examples by node type, all dependency patterns
- Companion skill: `using-n8n-mcp-tools` for `get_node` API details
---
*Antigravity Global Skills — n8n Node Configuration*
