---
name: using-n8n-mcp-tools
description: Expert guide for using n8n-mcp MCP tools effectively. Use when searching for nodes, validating configurations, accessing templates, managing workflows, or using any n8n-mcp tool. Provides tool selection guidance, parameter formats, and common patterns.
---

# n8n MCP Tools Expert

Master guide for n8n-mcp MCP tools to discover, build, and manage workflows.

## When to Use This Skill

- Searching for n8n nodes with `search_nodes`
- Validating node or workflow configuration
- Creating or updating workflows via API
- Deploying templates from the n8n library
- Debugging MCP tool errors or choosing the right tool

## Prerequisites

- n8n-mcp MCP server connected
- For workflow create/update/deploy: `N8N_API_URL` and `N8N_API_KEY` configured

## Workflow

### Step 1: Discover the Right Node

```
search_nodes(query: "slack")
→ get_node(nodeType: "nodes-base.slack")               # detail="standard" (default)
→ get_node(nodeType: "nodes-base.slack", mode: "docs") # readable docs
```

### Step 2: Know the nodeType Format

> [!IMPORTANT]  
> Two formats exist — use the correct one for each tool!

| Context | Format | Example |
|---------|--------|---------|
| `search_nodes`, `get_node`, `validate_node` | Short | `nodes-base.slack` |
| `n8n_create_workflow`, `n8n_update_partial_workflow` | Full | `n8n-nodes-base.slack` |

`search_nodes` returns **both** formats:
```
nodeType: "nodes-base.slack"           # for search/validate
workflowNodeType: "n8n-nodes-base.slack"  # for workflow tools
```

### Step 3: Validate Before Deploying

```javascript
validate_node({
  nodeType: "nodes-base.slack",
  config: {resource: "channel", operation: "create", name: "general"},
  profile: "runtime"   // recommended: minimal | runtime | ai-friendly | strict
})
```

### Step 4: Build Workflows Iteratively

```javascript
// 1. Create
n8n_create_workflow({name: "My Flow", nodes: [...], connections: {...}})

// 2. Validate
n8n_validate_workflow({id: "abc123"})

// 3. Update incrementally (~56s avg between edits)
n8n_update_partial_workflow({
  id: "abc123",
  intent: "Add error handling",
  operations: [{type: "addNode", node: {...}}]
})

// 4. Activate
n8n_update_partial_workflow({
  id: "abc123",
  operations: [{type: "activateWorkflow"}]
})
```

### Step 5: Use Templates as Starting Points

```javascript
search_templates({query: "slack webhook", limit: 10})
n8n_deploy_template({templateId: 2947, autoFix: true, autoUpgradeVersions: true})
```

## Validation

Pre-deployment checklist:
- `validate_node` passes with `profile: "runtime"`
- `n8n_validate_workflow` shows no errors
- All credential fields configured (not hardcoded)
- Workflow activated via `activateWorkflow` operation

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `Node not found` | Wrong prefix | Use `nodes-base.*` for search/validate |
| `API not available` | No API key | Set `N8N_API_URL` + `N8N_API_KEY` |
| Large payload, slow response | `detail: "full"` | Use `detail: "standard"` (default) |
| Validation false-positives | Wrong profile | Use `profile: "ai-friendly"` |
| Connection sourceIndex confusion | Multi-output nodes | Use `branch: "true"/"false"` or `case: 0` |

## Resources

- See [ADVANCED.md](./ADVANCED.md): full tool reference, smart parameters, AI agent guide, version checking
- n8n-mcp docs: `tools_documentation({topic: "search_nodes", depth: "full"})`
---
*Antigravity Global Skills — n8n MCP Tools Expert*
