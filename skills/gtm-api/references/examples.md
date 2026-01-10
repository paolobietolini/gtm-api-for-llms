# GTM API Workflow Examples

This document provides complete, executable examples of common Google Tag Manager API workflows.

All examples assume you have a valid OAuth 2.0 access token with appropriate scopes.

---

## Example 1: Create Workspace → Add Tag → Publish Version

### Scenario
Add a new Google Analytics 4 tag to a container and publish it to production.

### Step 1: List Containers

Find the container you want to modify.

**Request:**
```http
GET https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers
Authorization: Bearer {access_token}
```

**Response:**
```json
{
  "container": [
    {
      "path": "accounts/12345/containers/67890",
      "accountId": "12345",
      "containerId": "67890",
      "name": "My Website Container",
      "publicId": "GTM-XXXXX",
      "usageContext": ["web"],
      "fingerprint": "1234567890"
    }
  ]
}
```

### Step 2: Create a Workspace

Create a new workspace for your changes.

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "Add GA4 Tag",
  "description": "Adding Google Analytics 4 measurement tag"
}
```

**Response:**
```json
{
  "path": "accounts/12345/containers/67890/workspaces/10",
  "workspaceId": "10",
  "name": "Add GA4 Tag",
  "description": "Adding Google Analytics 4 measurement tag",
  "fingerprint": "9876543210"
}
```

### Step 3: Create a Trigger

Create an "All Pages" trigger for the tag.

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/triggers
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "All Pages",
  "type": "pageview",
  "filter": []
}
```

**Response:**
```json
{
  "path": "accounts/12345/containers/67890/workspaces/10/triggers/5",
  "triggerId": "5",
  "name": "All Pages",
  "type": "pageview",
  "fingerprint": "1111111111"
}
```

### Step 4: Create a GA4 Configuration Variable

Create a Google Analytics 4 Configuration variable.

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/variables
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "GA4 Config",
  "type": "gaawe",
  "parameter": [
    {
      "type": "template",
      "key": "measurementId",
      "value": "G-XXXXXXXXXX"
    }
  ]
}
```

**Response:**
```json
{
  "path": "accounts/12345/containers/67890/workspaces/10/variables/8",
  "variableId": "8",
  "name": "GA4 Config",
  "type": "gaawe",
  "fingerprint": "2222222222"
}
```

### Step 5: Create the GA4 Tag

Create the Google Analytics 4 tag that references the trigger and variable.

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/tags
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "GA4 - Page View",
  "firingTriggerId": ["5"],
  "type": "gaawe",
  "parameter": [
    {
      "type": "template",
      "key": "measurementId",
      "value": "{{GA4 Config}}"
    }
  ]
}
```

**Response:**
```json
{
  "path": "accounts/12345/containers/67890/workspaces/10/tags/15",
  "tagId": "15",
  "name": "GA4 - Page View",
  "firingTriggerId": ["5"],
  "type": "gaawe",
  "fingerprint": "3333333333"
}
```

### Step 6: Create Container Version

Create a version from the workspace.

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10:create_version
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "Add GA4 tracking",
  "notes": "Added Google Analytics 4 page view tag"
}
```

**Response:**
```json
{
  "containerVersion": {
    "path": "accounts/12345/containers/67890/versions/3",
    "containerVersionId": "3",
    "name": "Add GA4 tracking",
    "container": {
      "path": "accounts/12345/containers/67890",
      "containerId": "67890"
    },
    "tag": [...],
    "trigger": [...],
    "variable": [...]
  }
}
```

### Step 7: Publish the Version

Publish the version to make it live.

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/versions/3:publish
Authorization: Bearer {access_token}
```

**Response:**
```json
{
  "containerVersion": {
    "path": "accounts/12345/containers/67890/versions/3",
    "containerVersionId": "3",
    "name": "Add GA4 tracking",
    "fingerprint": "4444444444"
  }
}
```

---

## Example 2: List All Tags in Production

### Scenario
Get all tags currently published in the live container.

**Request:**
```http
GET https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/versions/live
Authorization: Bearer {access_token}
```

**Response:**
```json
{
  "path": "accounts/12345/containers/67890/versions/3",
  "containerVersionId": "3",
  "tag": [
    {
      "tagId": "15",
      "name": "GA4 - Page View",
      "type": "gaawe",
      "firingTriggerId": ["5"]
    },
    {
      "tagId": "20",
      "name": "Facebook Pixel",
      "type": "html",
      "firingTriggerId": ["5"]
    }
  ],
  "trigger": [...],
  "variable": [...]
}
```

---

## Example 3: Update Existing Tag

### Scenario
Modify an existing tag in a workspace.

### Step 1: Get Current Tag

**Request:**
```http
GET https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/tags/15
Authorization: Bearer {access_token}
```

**Response:**
```json
{
  "path": "accounts/12345/containers/67890/workspaces/10/tags/15",
  "tagId": "15",
  "name": "GA4 - Page View",
  "type": "gaawe",
  "firingTriggerId": ["5"],
  "fingerprint": "3333333333",
  "parameter": [...]
}
```

### Step 2: Update Tag

**Request:**
```http
PUT https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/tags/15
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "GA4 - Page View + Scroll",
  "type": "gaawe",
  "firingTriggerId": ["5", "7"],
  "fingerprint": "3333333333",
  "parameter": [...]
}
```

**Response:**
```json
{
  "path": "accounts/12345/containers/67890/workspaces/10/tags/15",
  "tagId": "15",
  "name": "GA4 - Page View + Scroll",
  "firingTriggerId": ["5", "7"],
  "fingerprint": "5555555555"
}
```

---

## Example 4: Handle Workspace Conflicts

### Scenario
Resolve conflicts when workspace base version is outdated.

### Step 1: Check Workspace Status

**Request:**
```http
GET https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/status
Authorization: Bearer {access_token}
```

**Response:**
```json
{
  "workspaceChange": [
    {
      "tag": {
        "tagId": "15",
        "name": "GA4 - Page View"
      },
      "status": "updated"
    }
  ],
  "mergeConflict": [
    {
      "entityInWorkspace": {
        "tag": {
          "tagId": "20",
          "name": "Facebook Pixel"
        }
      },
      "entityInBaseVersion": {
        "tag": {
          "tagId": "20",
          "name": "Meta Pixel"
        }
      }
    }
  ]
}
```

### Step 2: Resolve Conflict

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10:resolve_conflict
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "fingerprint": "workspace_fingerprint",
  "mergeConflict": [
    {
      "entityInWorkspace": {
        "tag": {
          "path": "accounts/12345/containers/67890/workspaces/10/tags/20"
        }
      },
      "resolutionType": "renameConflict"
    }
  ]
}
```

---

## Example 5: Delete Tag Safely

### Scenario
Remove a tag from a workspace and publish the change.

### Step 1: Delete Tag from Workspace

**Request:**
```http
DELETE https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/tags/20
Authorization: Bearer {access_token}
```

**Response:**
```http
HTTP/1.1 204 No Content
```

### Step 2: Create and Publish Version

Follow steps 6-7 from Example 1.

---

## Example 6: Create Custom HTML Tag with Trigger Exception

### Scenario
Create a custom HTML tag that fires on all pages except checkout.

### Step 1: Create "Checkout Page" Trigger

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/triggers
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "Checkout Page",
  "type": "pageview",
  "filter": [
    {
      "type": "contains",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "{{Page URL}}"
        },
        {
          "type": "template",
          "key": "arg1",
          "value": "/checkout"
        }
      ]
    }
  ]
}
```

### Step 2: Create Custom HTML Tag with Exception

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/tags
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "Custom Tracking Script",
  "type": "html",
  "firingTriggerId": ["5"],
  "blockingTriggerId": ["8"],
  "parameter": [
    {
      "type": "template",
      "key": "html",
      "value": "<script>console.log('Custom tracking');</script>"
    }
  ]
}
```

**Note:** `blockingTriggerId` contains the checkout page trigger, preventing the tag from firing on checkout pages.

---

## Example 7: Error Handling - Quota Exceeded

### Request:
```http
GET https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers
Authorization: Bearer {access_token}
```

### Response (Error):
```json
{
  "error": {
    "errors": [
      {
        "domain": "usageLimits",
        "reason": "rateLimitExceeded",
        "message": "Rate Limit Exceeded"
      }
    ],
    "code": 403,
    "message": "Rate Limit Exceeded"
  }
}
```

### Handling:
- Implement exponential backoff
- Wait time: 1s → 2s → 4s → 8s...
- Max wait: 32 seconds
- Respect daily quota: 10,000 requests/day

---

## Example 8: Copy Tag Between Workspaces

### Scenario
Copy a tag from one workspace to another within the same container.

### Step 1: Get Tag from Source Workspace

**Request:**
```http
GET https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/10/tags/15
Authorization: Bearer {access_token}
```

### Step 2: Create Tag in Destination Workspace

**Request:**
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/workspaces/11/tags
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "GA4 - Page View (Copy)",
  "type": "gaawe",
  "firingTriggerId": ["5"],
  "parameter": [...]
}
```

**Note:** You must manually ensure referenced triggers and variables exist in the destination workspace.

---

## Common Patterns

### Pattern 1: Batch Operations
When creating multiple entities, create them sequentially to avoid race conditions with fingerprints.

### Pattern 2: Fingerprint Validation
Always include the current fingerprint when updating to prevent conflicts:
```json
{
  "name": "Updated Name",
  "fingerprint": "current_fingerprint_value"
}
```

### Pattern 3: Testing Before Publishing
1. Create workspace
2. Make changes
3. Use `quick_preview` to generate preview URL
4. Test in browser
5. Publish when validated

### Pattern 4: Rollback Strategy
Keep track of previous version IDs. To rollback:
```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/12345/containers/67890/versions/2:publish
```

---

## Required OAuth Scopes by Operation

| Operation | Minimum Scope |
|-----------|---------------|
| List containers | `tagmanager.readonly` |
| Create workspace | `tagmanager.edit.containers` |
| Create/update tags | `tagmanager.edit.containers` |
| Publish version | `tagmanager.publish` |
| Delete container | `tagmanager.delete.containers` |
| Manage users | `tagmanager.manage.users` |

---

## Tips for LLMs

1. **Always get fingerprints** before updating entities
2. **Check workspace status** before creating versions
3. **Validate entity references** (triggers, variables) exist before referencing them
4. **Use descriptive names** for workspaces to track changes
5. **Handle 403 errors** with exponential backoff
6. **Publish is destructive** - cannot undo, only roll forward or backward to a previous version
7. **Workspaces are isolated** - changes don't affect live until published
8. **Built-in variables** must be enabled before use
9. **Tag firing order** matters for dependent tags
10. **Custom templates** must be created before custom tag types can be used
