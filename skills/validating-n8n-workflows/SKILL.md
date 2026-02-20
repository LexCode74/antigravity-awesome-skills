---
name: validating-n8n-workflows
description: Interpret validation errors and guide fixing them. Use when encountering validation errors, validation warnings, false positives, operator structure issues, or need help understanding validation results. Also use when asking about validation profiles, error types, or the validation loop process.
---

# n8n Validation Expert

Expert guide for interpreting and fixing n8n validation errors.

## When to Use This Skill

- Encountering `missing_required`, `invalid_value`, or `type_mismatch` errors
- Seeing a validation warning and unsure if it matters
- IF/Switch node has broken operator structure
- Workflow fails `n8n_validate_workflow` before activation

## Prerequisites

- n8n-mcp MCP server connected
- Node or workflow configuration ready to validate

## Workflow

### Step 1: Choose the Right Profile

| Profile | Use When | Strictness |
|---------|----------|------------|
| `minimal` | Quick check while editing | Low |
| `runtime` ✅ | Pre-deployment (recommended) | Medium |
| `ai-friendly` | AI-generated configs, fewer false-positives | Medium-Low |
| `strict` | Production-critical workflows | High |

### Step 2: The Validation Loop (Normal!)

```
validate_node → read error (23s) → fix → validate again (58s) → repeat
```

Expect **2-3 iterations** — this is expected behavior, not failure.

```javascript
validate_node({
  nodeType: "nodes-base.slack",
  config: {resource: "channel", operation: "create", name: "general"},
  profile: "runtime"
})
// → {valid: true, errors: [], warnings: [...]}
```

### Step 3: Fix Errors by Type

```javascript
// missing_required → add the field
config.channel = "#general";

// invalid_value → use an allowed value
config.operation = "post";    // not "send"

// type_mismatch → fix the type
config.limit = 100;           // number, not "100"

// invalid_expression → add {{}}
config.text = "={{$json.name}}";   // not "$json.name"

// invalid_reference → fix node name (case-sensitive!)
config.expr = "={{$node['HTTP Request'].json.data}}";
```

### Step 4: Handle Auto-Sanitization

Auto-sanitization runs on every workflow save and fixes:
- Binary operators (equals, contains…) → removes stray `singleValue`
- Unary operators (isEmpty, isNotEmpty…) → adds `singleValue: true`
- IF/Switch metadata for v2.2+ / v3.2+

**Cannot fix**: broken connections, branch count mismatches → use `cleanStaleConnections` or `n8n_autofix_workflow`.

### Step 5: Validate Entire Workflow

```javascript
// After building all nodes
n8n_validate_workflow({id: "abc123", options: {
  validateNodes: true,
  validateConnections: true,
  validateExpressions: true,
  profile: "runtime"
}})
```

## Validation

Good validation result looks like:
```json
{"valid": true, "errors": [], "warnings": [...], "suggestions": [...]}
```

- `valid: true` → safe to activate
- `errors` present → **must fix before activation**
- `warnings` present → review; may be acceptable false-positives
- `suggestions` → optional improvements

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `missing_required` | Field not provided | Add field, check operation context |
| `invalid_value` | Bad enum value | Check allowed values via `get_node` |
| `type_mismatch` | String vs number | Fix JS type (e.g. `100` not `"100"`) |
| `invalid_expression` | Missing `={{}}` | Add expression syntax around variable |
| `invalid_reference` | Typo in node name | Match name exactly (case-sensitive) |
| `Connection target not found` | Broken connection | Use `cleanStaleConnections` operation |

## Resources

- See [ADVANCED.md](./ADVANCED.md): full error catalog, false-positive guide, recovery strategies
- Run `n8n_autofix_workflow({id, applyFixes: false})` to preview auto-fixes
---
*Antigravity Global Skills — n8n Validation Expert*
