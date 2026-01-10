# LLM Quick Start Guide

**Purpose:** Get started executing GTM API operations immediately.

---

## Step 1: Understand What You're Working With

Google Tag Manager API lets you programmatically manage:
- **Tags** - Tracking code that fires on your site
- **Triggers** - Conditions that determine when tags fire
- **Variables** - Dynamic values used in tags and triggers
- **Versions** - Immutable snapshots of container configuration
- **Workspaces** - Isolated environments for making changes

**Key Concept:** All changes happen in **workspaces**, then get published via **versions**.

---

## Step 2: Your Essential Files

### When Executing Tasks (Use These First)

1. **[llm-instructions.md](llm-instructions.md)**
   - Start here for any operation
   - Contains step-by-step algorithms
   - Tells you exactly what to do

2. **[llm-request-templates.md](llm-request-templates.md)**
   - Copy-paste JSON templates
   - Replace marked variables
   - Avoid syntax errors

3. **[llm-validation-rules.md](llm-validation-rules.md)**
   - Check before sending request
   - Required vs optional fields
   - Prevent common mistakes

4. **[llm-workflows.md](llm-workflows.md)**
   - Decision trees for complex flows
   - Error recovery procedures
   - State machine definitions

### When Learning Context (Read These Second)

5. **[llm-context.md](llm-context.md)**
   - Mental model of GTM
   - Core concepts
   - Critical rules

6. **[schemas.md](schemas.md)**
   - Complete data structures
   - All available fields
   - Type definitions

---

## Step 3: Common Operations Quick Reference

### Create a Tag

```
1. Go to: llm-instructions.md → "Instruction: Create a Tag"
2. Copy template: llm-request-templates.md → "GA4 Configuration Tag"
3. Validate: llm-validation-rules.md → "Create Tag" section
4. Execute: POST {workspace_path}/tags
```

### Update a Tag

```
1. Go to: llm-instructions.md → "Instruction: Update a Tag"
2. GET current tag first (to get fingerprint)
3. Validate: llm-validation-rules.md → "Update Tag" section
4. Execute: PUT {tag_path} with fingerprint
```

### Publish Changes

```
1. Go to: llm-instructions.md → "Instruction: Publish Changes"
2. Check workspace has changes
3. Create version: POST {workspace_path}:create_version
4. Publish version: POST {version_path}:publish
```

### Handle Error

```
1. Go to: llm-workflows.md → "Decision Tree: Error Recovery"
2. Match HTTP status code
3. Apply recovery procedure
4. Retry if appropriate
```

---

## Step 4: Your First Operation

Let's create a simple tag:

### Input Requirements
- Account ID: "123456"
- Container ID: "7890"
- OAuth token with scopes: `tagmanager.readonly`, `tagmanager.edit.containers`

### Algorithm to Follow

```
STEP 1: Get workspace
  GET https://tagmanager.googleapis.com/tagmanager/v2/accounts/123456/containers/7890/workspaces
  Extract: workspace_id and workspace_path from response

STEP 2: Create "All Pages" trigger
  Template from: llm-request-templates.md → "All Pages Trigger"

  POST {workspace_path}/triggers
  Body: {
    "name": "All Pages",
    "type": "pageview"
  }

  Extract: trigger_id from response

STEP 3: Create tag
  Template from: llm-request-templates.md → "Custom HTML Tag"

  POST {workspace_path}/tags
  Body: {
    "name": "Test Tag",
    "type": "html",
    "firingTriggerId": ["{trigger_id}"],
    "parameter": [
      {
        "type": "template",
        "key": "html",
        "value": "<script>console.log('Hello from GTM');</script>"
      },
      {
        "type": "boolean",
        "key": "supportDocumentWrite",
        "value": "false"
      }
    ]
  }

  Extract: tag_id from response

STEP 4: Done!
  Return tag_id to user
```

### Validation Checklist

Before STEP 3:
- ☐ workspace_id is not empty
- ☐ trigger_id is not empty
- ☐ tag name is not empty
- ☐ tag type is "html"
- ☐ firingTriggerId contains trigger_id
- ☐ parameter array has html and supportDocumentWrite

---

## Step 5: Common Patterns

### Pattern: List All Resources

```
Algorithm: llm-instructions.md → "List All Resources"

all_items = []
page_token = None

LOOP:
  response = GET {parent_path}/{resource_type}?pageToken={page_token}
  all_items.extend(response.items)
  page_token = response.nextPageToken

  IF page_token is None:
    BREAK

RETURN all_items
```

### Pattern: Get or Create Workspace

```
workspaces = GET /workspaces

IF workspaces is empty:
  workspace = POST /workspaces
    Body: {"name": "Default Workspace"}
ELSE:
  workspace = workspaces[0]

RETURN workspace
```

### Pattern: Handle 409 Conflict

```
max_retries = 3
retry_count = 0

WHILE retry_count < max_retries:
  response = PUT {path} with body

  IF response.status == 409:
    current = GET {path}
    fingerprint = current.fingerprint
    body.fingerprint = fingerprint
    retry_count += 1
    CONTINUE

  RETURN response

RETURN error "Max retries exceeded"
```

---

## Step 6: Critical Rules to Remember

### Rule 1: Fingerprints
```
CREATE:  ✗ Don't include fingerprint
UPDATE:  ✓ MUST include current fingerprint
GET:     ✗ Returned, not sent
DELETE:  ✗ Not needed
```

### Rule 2: IDs vs Paths
```
API endpoints:    Use full path
                  "accounts/123/containers/456/workspaces/10/tags/5"

Entity references: Use ID only
                   firingTriggerId: ["5"]
```

### Rule 3: Workspace Isolation
```
Changes in workspace → NOT live
Create version → Still not live
Publish version → NOW live
```

### Rule 4: OAuth Scopes
```
Read anything:        tagmanager.readonly
Create/edit entities: tagmanager.edit.containers
Publish version:      tagmanager.publish (plus edit.containers)
Delete container:     tagmanager.delete.containers
```

### Rule 5: Rate Limits
```
10,000 requests/day
0.25 QPS (1 request per 4 seconds)

If exceeded: HTTP 403 with "rateLimitExceeded"
Recovery: Exponential backoff (1s → 2s → 4s → 8s → 16s → 32s)
```

---

## Step 7: Error Handling Quick Ref

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | Success | Process response |
| 400 | Bad request | Check JSON, validate fields |
| 401 | Unauthorized | Refresh OAuth token |
| 403 | Forbidden | Check scopes OR rate limit (backoff) |
| 404 | Not found | Verify resource path/ID |
| 409 | Conflict | Get fresh fingerprint, retry |
| 429 | Too many requests | Exponential backoff |
| 500 | Server error | Retry with backoff |

---

## Step 8: Validation Before Sending

### Pre-Flight Checklist

```
☐ Required fields present
☐ No auto-generated fields (for CREATE)
☐ Fingerprint included (for UPDATE)
☐ Field types correct (string, boolean as "true"/"false", etc.)
☐ Entity references valid (triggers, variables exist)
☐ Parameter structure valid
☐ OAuth scopes sufficient
☐ Container type supports feature
```

---

## Step 9: Common Mistakes to Avoid

### Mistake 1: Including auto-generated fields in CREATE
```
❌ WRONG:
{
  "tagId": "5",      // Remove
  "path": "...",     // Remove
  "name": "My Tag"
}

✅ CORRECT:
{
  "name": "My Tag",
  "type": "html",
  "firingTriggerId": ["5"]
}
```

### Mistake 2: Missing fingerprint in UPDATE
```
❌ WRONG:
PUT /tags/5
{ "name": "Updated" }

✅ CORRECT:
current = GET /tags/5
PUT /tags/5
{
  ...current,
  "name": "Updated",
  "fingerprint": current.fingerprint
}
```

### Mistake 3: Boolean as boolean instead of string
```
❌ WRONG:
{
  "type": "boolean",
  "value": true
}

✅ CORRECT:
{
  "type": "boolean",
  "value": "true"  // String!
}
```

### Mistake 4: Using paths in entity references
```
❌ WRONG:
{
  "firingTriggerId": ["accounts/123/.../triggers/5"]
}

✅ CORRECT:
{
  "firingTriggerId": ["5"]  // Just the ID
}
```

---

## Step 10: Your Execution Template

When given a GTM API task:

```
TASK: {user_request}

STEP 1: Identify operation type
  □ Create entity
  □ Update entity
  □ Delete entity
  □ List entities
  □ Publish version
  □ Other: ___________

STEP 2: Find algorithm
  File: llm-instructions.md
  Section: "Instruction: {operation}"

STEP 3: Get template (if creating/updating)
  File: llm-request-templates.md
  Section: "{entity_type} Templates"

STEP 4: Validate
  File: llm-validation-rules.md
  Section: "{operation} {entity_type}"
  Checklist: All required fields present

STEP 5: Execute
  Make API call with proper auth
  Headers:
    Authorization: Bearer {token}
    Content-Type: application/json

STEP 6: Handle response
  IF success (200/201):
    Extract IDs and paths
    Return to user

  IF error (4xx/5xx):
    Apply error recovery from llm-workflows.md
    Decision tree: "Error Recovery"

STEP 7: Return result
  Success: Include created IDs
  Failure: Include error message and suggested fix
```

---

## Summary

### The 3 Files You Need Most

1. **llm-instructions.md** - WHAT to do
2. **llm-request-templates.md** - WHAT to send
3. **llm-validation-rules.md** - WHAT to check

### The 4 Steps for Every Operation

1. **Read algorithm** (llm-instructions.md)
2. **Copy template** (llm-request-templates.md)
3. **Validate** (llm-validation-rules.md)
4. **Execute** (with error handling from llm-workflows.md)

### The 5 Things to Always Remember

1. **Fingerprints for updates** - Always GET first
2. **IDs not paths** - In entity references
3. **Workspace isolation** - Changes not live until published
4. **Validate before sending** - Prevent wasted API calls
5. **Handle rate limits** - Exponential backoff on 403/429

---

**You're ready to execute GTM API operations!**

Start with a simple operation (create tag), validate carefully, handle errors properly, and build from there.
