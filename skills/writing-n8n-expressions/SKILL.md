---
name: writing-n8n-expressions
description: Validate n8n expression syntax and fix common errors. Use when writing n8n expressions, using {{}} syntax, accessing $json/$node variables, troubleshooting expression errors, or working with webhook data in workflows.
---

# n8n Expression Syntax

Expert guide for writing correct n8n expressions in workflow node fields.

## When to Use This Skill

- Writing `{{expression}}` in n8n node fields
- Accessing webhook body data from downstream nodes
- Referencing other nodes with `$node["Name"]`
- Debugging "undefined", literal text output, or expression errors

## Prerequisites

- n8n workflow with at least one upstream node to reference

## Workflow

### Step 1: Expression Format

All dynamic content uses double curly braces:
```
{{$json.fieldName}}
{{$json.body.name}}       ← Webhook data (always under .body)
{{$node["HTTP Request"].json.data}}
{{$now.toFormat('yyyy-MM-dd')}}
{{$env.API_KEY}}
```

### Step 2: Webhook Data Structure

> [!IMPORTANT]  
> Webhook data is **always** nested under `.body`

```javascript
// Webhook node output:
{
  "headers": {...},
  "body": {           // ← YOUR DATA IS HERE
    "name": "John",
    "email": "john@example.com"
  }
}

// ❌ WRONG: {{$json.name}}
// ✅ CORRECT: {{$json.body.name}}
```

### Step 3: Reference Other Nodes

```javascript
// Reference by exact node name (case-sensitive)
{{$node["HTTP Request"].json.data}}
{{$node["Webhook"].json.body.email}}

// Field names with spaces: bracket notation
{{$json['field name']}}
```

### Step 4: Common Patterns

```javascript
// Concatenation (automatic in text fields)
Hello {{$json.body.name}}!

// In a URL field
https://api.example.com/users/{{$json.body.user_id}}

// Conditional / default
{{$json.email || 'no-email@example.com'}}

// Date formatting
{{$now.toFormat('yyyy-MM-dd')}}
{{$now.plus({days: 7}).toISO()}}

// Array access
{{$json.users[0].email}}
{{$json.items.length}}
```

### Step 5: Where NOT to Use Expressions

| Context | Correct approach |
|---------|-----------------|
| Code node (JS) | `$json.field` (no `{{ }}`) |
| Code node (Python) | `_json["field"]` |
| Webhook path | Static string only |
| Credential fields | Use credential system |

## Validation

Quick checklist before saving:
- `{{ }}` wraps all dynamic fields in non-code nodes
- Webhook fields use `.body.fieldName`
- Node names exactly match (case-sensitive, quoted with spaces)
- No nested `{{{ }}}` triple braces

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| Field shows as literal text `$json.x` | Missing `{{ }}` | Wrap: `{{$json.x}}` |
| `undefined` on webhook data | Missing `.body` | Change to `{{$json.body.fieldName}}` |
| Expression fails in Code node | Wrong context | Remove `{{ }}`, use direct JS/Python |
| `Cannot read property of undefined` | Parent path missing | Check path exists via expression editor preview |
| Node name not found | Case mismatch | Match exact node name including capitalization |

## Resources

- See [ADVANCED.md](./ADVANCED.md) for: advanced Luxon patterns, $jmespath queries, string methods, regex
- n8n expression docs: https://docs.n8n.io/code/expressions/
---
*Antigravity Global Skills — n8n Expression Syntax*
