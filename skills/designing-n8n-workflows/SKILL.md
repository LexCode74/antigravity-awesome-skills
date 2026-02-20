---
name: designing-n8n-workflows
description: Proven workflow architectural patterns from real n8n workflows. Use when building new workflows, designing workflow structure, choosing workflow patterns, planning workflow architecture, or asking about webhook processing, HTTP API integration, database operations, AI agent workflows, or scheduled tasks.
---

# n8n Workflow Patterns

Proven architectural patterns for building n8n workflows from real template analysis.

## When to Use This Skill

- Starting a new workflow and need to choose an architecture
- Deciding between webhook, scheduled, or API integration patterns
- Building an AI agent workflow with tools and memory
- Planning error handling strategy for a workflow

## Prerequisites

- n8n instance running
- Use `search_templates` to find existing real examples before building from scratch

## Workflow

### Step 1: Select Your Pattern

| Pattern | Use When | Trigger |
|---------|----------|---------|
| **Webhook Processing** (35%) | Receive external events | Webhook |
| **Scheduled Tasks** (28%) | Recurring reports/maintenance | Schedule |
| **HTTP API Integration** | Sync with third-party APIs | Manual/Webhook |
| **Database Operations** | ETL, data sync | Schedule |
| **AI Agent Workflow** | Chat, reasoning, tool use | Webhook/Chat |

### Step 2: Apply the Pattern Template

**Webhook Processing**
```
Webhook → Validate → Transform (Set/Code) → Action → Respond
                                          → Error Trigger → Error Notify
```

**Scheduled Task**
```
Schedule → Fetch (HTTP/DB) → Process (Code/Set) → Deliver → Log
```

**HTTP API Integration**
```
Trigger → HTTP Request → Transform → Store/Act → Error Handler
```

**Database Operations**
```
Schedule → Query → IF (records exist?) → Write → Verify
```

**AI Agent**
```
Webhook → AI Agent [OpenAI Model + Tool Nodes + Window Buffer Memory] → Respond
```

### Step 3: Add Core Components

```javascript
// Branching (IF/Switch)
IF → True Path: action1
   → False Path: action2

// Batch processing
Split In Batches → Process → Loop

// Error handling (separate workflow)
Error Trigger → Get Error Details → Notify (Slack/Email)
```

### Step 4: Apply the Creation Checklist

**Planning**: Pattern selected → nodes identified → data flow mapped → error strategy defined

**Implementation**: Trigger → Source → Transform → Output → Error handler

**Validation**: `validate_node` per critical node → `n8n_validate_workflow` → test with sample data

**Deployment**: Activate via `activateWorkflow` operation → monitor first run

## Validation

- Use `search_templates({searchMode: "by_task", task: "webhook_processing"})` to verify patterns
- Run `n8n_validate_workflow` before activation
- Test edge cases: empty input, API errors, null fields

## Error Handling

| Issue | Cause | Resolution |
|-------|-------|------------|
| Wrong execution order | Legacy v0 mode | Set `executionOrder: "v1"` in settings |
| Webhook returns undefined | Missing `.body` | Use `{{$json.body.field}}` expressions |
| Nodes run in unexpected order | Multiple triggers | Remove extra triggers or split workflows |
| Rate limit from API | No throttle | Add `Wait` node between API calls |
| Large dataset crashes | No batching | Use `Split In Batches` (set chunk size) |

## Key Statistics (Real Templates)

- Most common trigger: **Webhook** (35%)
- Most common transform: **Set** (68%), **Code** (42%), **IF** (38%)
- Most common output: **HTTP Request** (45%), **Slack** (32%), **DB write** (28%)
- Simple 3-5 nodes: 42% | Medium 6-10 nodes: 38% | Complex 11+: 20%

## Resources

- See [ADVANCED.md](./ADVANCED.md): detailed examples, pattern files (webhook, API, DB, AI, scheduled)
- Quick start: `n8n_deploy_template({templateId: 2947})` for Weather → Slack example
- Find examples: `search_templates({query: "webhook slack integration"})`
---
*Antigravity Global Skills — n8n Workflow Patterns*
