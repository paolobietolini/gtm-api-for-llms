---
name: gtm-api
description: Execute Google Tag Manager API operations - create, update, delete, and manage tags, triggers, variables, and containers. Use when working with GTM programmatically, generating API calls, validating GTM configurations, publishing container versions, or automating tag management workflows.
---

# Google Tag Manager API

Execute GTM API operations with validation, error handling, and proper workflow patterns.

## Quick Start

### 1. Understand the Hierarchy

```
Account
└── Container
    ├── Environments (deployment targets)
    ├── Versions (immutable snapshots)
    └── Workspaces (mutable, where changes happen)
        ├── Tags
        ├── Triggers
        └── Variables
```

**Critical rule:** All changes happen in workspaces; publishing creates versions.

### 2. Four Steps for Every Operation

1. **Read algorithm** → [instructions.md](references/instructions.md)
2. **Copy template** → [request-templates.md](references/request-templates.md)
3. **Validate** → [validation-rules.md](references/validation-rules.md)
4. **Execute** with error handling → [workflows.md](references/workflows.md)

---

## Core Execution Workflow

### Step 1: Validate Inputs

```
BEFORE any API call:
  1. Validate required fields present
  2. Validate field types (strings, arrays, etc.)
  3. Verify entity references exist (trigger IDs, variable names)
  4. Check OAuth scopes sufficient
  5. Confirm container type supports operation
```

### Step 2: Get or Create Workspace

```
workspaces = GET /accounts/{account_id}/containers/{container_id}/workspaces

IF workspaces is empty:
  workspace = POST /workspaces with {"name": "Default Workspace"}
ELSE:
  workspace = workspaces[0]

Store: workspace_id, workspace_path
```

### Step 3: Execute Operation

Use the appropriate reference file:
- **Creating entities** → [request-templates.md](references/request-templates.md)
- **Step-by-step algorithms** → [instructions.md](references/instructions.md)
- **Complex flows** → [workflows.md](references/workflows.md)

### Step 4: Handle Response

```
IF status 200/201: Extract IDs, return success
IF status 400: Check JSON syntax, validate fields
IF status 401: Refresh OAuth token
IF status 403: Check scopes OR rate limit (backoff)
IF status 404: Verify resource path/ID
IF status 409: Get fresh fingerprint, retry (max 3)
IF status 429: Exponential backoff (1s → 2s → 4s → 8s → 16s → 32s)
IF status 500: Retry with backoff
```

---

## Critical Rules

### Fingerprints

```
CREATE:  Don't include fingerprint
UPDATE:  MUST include current fingerprint (GET first)
DELETE:  Not needed
```

### IDs vs Paths

```
API endpoints:     Use full path
                   "accounts/123/containers/456/workspaces/10/tags/5"

Entity references: Use ID only
                   firingTriggerId: ["5"]
```

### Boolean Parameters

```
WRONG:  {"type": "boolean", "value": true}
RIGHT:  {"type": "boolean", "value": "true"}  // String!
```

### Rate Limits

```
10,000 requests/day
0.25 QPS (1 request per 4 seconds)
On limit: Exponential backoff
```

---

## Common Operations

### Create a Tag

```
1. GET workspace
2. Verify triggers exist (GET /triggers/{id})
3. Build tag body with:
   - name (required)
   - type (required)
   - firingTriggerId (required, array of IDs)
   - parameter (array of typed params)
4. POST {workspace_path}/tags
5. Return tag_id
```

Template: [request-templates.md](references/request-templates.md) → "Tag Templates"

### Update a Tag

```
1. GET {tag_path} → extract fingerprint
2. Merge updates with current state
3. Include fingerprint in body
4. PUT {tag_path}
5. On 409: Get fresh fingerprint, retry
```

### Publish Changes

```
1. GET {workspace_path}/status
   - Check for conflicts (resolve first)
   - Check for changes (nothing to publish if empty)
2. POST {workspace_path}:create_version with name/notes
3. POST {version_path}:publish
4. Return version_id
```

Algorithm: [instructions.md](references/instructions.md) → "Instruction: Publish Changes"

### List Resources with Pagination

```
all_items = []
page_token = None

LOOP:
  response = GET {path}/{resource_type}?pageToken={page_token}
  all_items.extend(response.items)
  page_token = response.nextPageToken
  IF page_token is None: BREAK

RETURN all_items
```

---

## OAuth Scopes

| Action | Scope |
|--------|-------|
| Read anything | `tagmanager.readonly` |
| Create/edit tags | `tagmanager.edit.containers` |
| Publish | `tagmanager.publish` |
| Delete containers | `tagmanager.delete.containers` |
| Manage users | `tagmanager.manage.users` |

Scopes are additive. Publishing requires: `readonly` + `edit.containers` + `publish`

---

## Path Patterns

| Resource | Path |
|----------|------|
| Container | `accounts/{id}/containers/{id}` |
| Workspace | `.../containers/{id}/workspaces/{id}` |
| Tag | `.../workspaces/{id}/tags/{id}` |
| Trigger | `.../workspaces/{id}/triggers/{id}` |
| Variable | `.../workspaces/{id}/variables/{id}` |
| Version | `.../containers/{id}/versions/{id}` |

Special paths:
- `/versions/live` - Currently published version
- `/workspaces/{id}:create_version` - Create version
- `/versions/{id}:publish` - Publish version

---

## Tag Parameter Structure

Parameters use typed objects:

```json
{
  "parameter": [
    {"type": "template", "key": "measurementId", "value": "G-XXXXX"},
    {"type": "boolean", "key": "sendPageView", "value": "true"},
    {"type": "list", "key": "eventParameters", "list": [...]}
  ]
}
```

Types: `template` (string), `boolean` (string "true"/"false"), `integer`, `list`, `map`

---

## Reference Documentation

### Execution (When Doing)

| File | Use When |
|------|----------|
| [instructions.md](references/instructions.md) | Step-by-step algorithms for operations |
| [request-templates.md](references/request-templates.md) | Copy-paste JSON templates |
| [validation-rules.md](references/validation-rules.md) | Validating before sending |
| [workflows.md](references/workflows.md) | Complex flows, decision trees |

### Understanding (When Learning)

| File | Use When |
|------|----------|
| [context.md](references/context.md) | Mental model, architecture |
| [schemas.md](references/schemas.md) | Complete entity structures |
| [api-reference.md](references/api-reference.md) | All endpoints, methods |
| [examples.md](references/examples.md) | Real-world request/response pairs |
| [quick-reference.md](references/quick-reference.md) | Lookup tables, type codes |

---

## Common Mistakes to Avoid

### Including auto-generated fields in CREATE

```
WRONG: {"tagId": "5", "path": "...", "name": "My Tag"}
RIGHT: {"name": "My Tag", "type": "html", "firingTriggerId": ["5"]}
```

### Missing fingerprint in UPDATE

```
WRONG: PUT /tags/5 with {"name": "Updated"}
RIGHT: GET /tags/5 first, then PUT with current fingerprint
```

### Using paths in entity references

```
WRONG: {"firingTriggerId": ["accounts/123/.../triggers/5"]}
RIGHT: {"firingTriggerId": ["5"]}
```

---

## Execution Template

When given a GTM API task:

```
1. IDENTIFY operation type (create/update/delete/list/publish)
2. FIND algorithm in instructions.md
3. GET template from request-templates.md (if creating/updating)
4. VALIDATE using validation-rules.md
5. EXECUTE with proper auth headers
6. HANDLE errors using workflow from workflows.md
7. RETURN result with IDs or error with fix suggestion
```
