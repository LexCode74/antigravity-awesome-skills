---
name: coding-n8n-javascript
description: Write JavaScript code in n8n Code nodes. Use when writing JavaScript in n8n, using $input/$json/$node syntax, making HTTP requests with $helpers, working with dates using DateTime, troubleshooting Code node errors, or choosing between Code node modes.
---

# JavaScript Code Node

Expert guidance for writing JavaScript code in n8n Code nodes.

## When to Use This Skill

- Writing JavaScript in an n8n Code node
- Using `$input`, `$json`, or `$node` to access workflow data
- Choosing between "Run Once for All Items" vs "Run Once for Each Item" mode
- Debugging `undefined`, return format, or webhook data errors

## Prerequisites

- n8n instance running
- Code node added to workflow with language set to JavaScript

## Workflow

### Step 1: Choose Execution Mode

| Mode | When to use | Data access |
|------|-------------|-------------|
| **Run Once for All Items** ✅ (95% of cases) | Aggregation, filtering, batch ops | `$input.all()` |
| **Run Once for Each Item** | Per-item independent logic | `$input.item` |

### Step 2: Access Data

```javascript
// All items (most common)
const items = $input.all();

// First item only
const first = $input.first().json;

// Webhook data - ALWAYS under .body
const email = $input.first().json.body.email;   // ✅
// const email = $input.first().json.email;     // ❌ undefined

// Reference another node
const fromNode = $node["HTTP Request"].json;
```

### Step 3: Return Correct Format

```javascript
// ✅ Required: Array of objects with json property
return [{json: {field: value}}];

// ✅ Multiple items
return items.map(item => ({
  json: {id: item.json.id, processed: true}
}));

// ❌ Wrong: No array wrapper
return {json: {field: value}};

// ❌ Wrong: No json wrapper
return [{field: value}];
```

### Step 4: Use Built-ins

```javascript
// HTTP request from code
const response = await $helpers.httpRequest({
  method: 'GET',
  url: 'https://api.example.com/data',
  headers: {'Authorization': 'Bearer token'}
});

// Date/time with Luxon
const now = DateTime.now();
const formatted = now.toFormat('yyyy-MM-dd');
const tomorrow = now.plus({days: 1}).toISO();

// JSON path queries
const adults = $jmespath(data, 'users[?age >= `18`]');
```

### Step 5: Guard Against Errors

```javascript
// Safe null check
const value = item.json?.user?.email || 'no-email@example.com';

// Try-catch for HTTP calls
try {
  const res = await $helpers.httpRequest({url: 'https://api.example.com'});
  return [{json: {success: true, data: res}}];
} catch (error) {
  return [{json: {success: false, error: error.message}}];
}

// Validate input exists
if (!items || items.length === 0) return [];
```

## Validation

```javascript
// Pre-deployment checklist (add as comment block):
// ✅ returns array format [{json: {...}}]
// ✅ webhook data accessed via .body
// ✅ null checks on optional fields
// ✅ all code paths have return statement
```

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `undefined` on webhook field | Missing `.body` | Use `$json.body.fieldName` |
| `Cannot return non-array` | Missing `[...]` wrapper | Wrap return in `[{json: ...}]` |
| Expression `{{ }}` in code | Wrong syntax context | Use direct JS: `$json.field` not `{{$json.field}}` |
| `item.json.x.y` crashes | Null parent | Use optional chaining: `item.json?.x?.y` |
| HTTP 401 inside code | Auth header missing | Add `Authorization` to headers object |

## Resources

- See [ADVANCED.md](./ADVANCED.md) for: 10 production patterns, aggregation, regex filter, data enrichment
- n8n docs: https://docs.n8n.io/code/code-node/
---
*Antigravity Global Skills — n8n JavaScript Code Node*
