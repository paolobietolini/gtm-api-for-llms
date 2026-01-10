# GTM API Workflows for LLMs

Algorithmic workflows and decision trees for LLM execution.

---

## Workflow: Create and Publish Tag

### Prerequisites Check

```
BEFORE starting workflow:
  ✓ Have valid OAuth token with scopes: [readonly, edit.containers, publish] # Consider an incremental auth workflow
  ✓ Know account ID (string)
  ✓ Know container ID (string)
```

### Algorithm

```
FUNCTION create_and_publish_tag(account_id, container_id, tag_config):

  STEP 1: Get or create workspace
    workspace = get_workspace(account_id, container_id)
    IF workspace is None:
      workspace = create_workspace(account_id, container_id)
    EXTRACT workspace_id FROM workspace.workspaceId
    EXTRACT workspace_fingerprint FROM workspace.fingerprint

  STEP 2: Verify/create trigger
    IF tag_config.trigger_id is None:
      trigger = create_trigger(account_id, container_id, workspace_id)
      tag_config.trigger_id = trigger.triggerId

  STEP 3: Verify/create variables (if needed)
    FOR EACH variable_name IN tag_config.required_variables:
      variable = find_variable_by_name(workspace_id, variable_name)
      IF variable is None:
        variable = create_variable(workspace_id, variable_name)

  STEP 4: Create tag
    tag = create_tag(workspace_id, tag_config)
    EXTRACT tag_id FROM tag.tagId

  STEP 5: Create version
    version = create_version_from_workspace(workspace_id)
    EXTRACT version_id FROM version.containerVersionId

  STEP 6: Publish version
    published = publish_version(container_id, version_id)
    RETURN published

  ERROR HANDLING:
    IF any step returns 409 Conflict:
      REFRESH workspace fingerprint
      RETRY from failed step
    IF any step returns 403:
      CHECK OAuth scopes
      RETURN error "Insufficient permissions"
    IF any step returns 404:
      VERIFY parent resource exists
      RETURN error "Parent resource not found"
```

---

## Workflow: List All Tags (with pagination)

### Algorithm

```
FUNCTION list_all_tags(workspace_path):

  all_tags = []
  page_token = None

  LOOP:
    request_params = {}
    IF page_token is not None:
      request_params["pageToken"] = page_token

    response = GET(workspace_path + "/tags", params=request_params)

    IF response.status_code != 200:
      RETURN error(response)

    tags = response.json.get("tag", [])
    all_tags.extend(tags)

    page_token = response.json.get("nextPageToken", None)

    IF page_token is None:
      BREAK

  RETURN all_tags
```

---

## Workflow: Update Tag (with conflict resolution)

### Algorithm

```
FUNCTION update_tag(tag_path, updates):

  max_retries = 3
  retry_count = 0

  LOOP:
    STEP 1: Get current tag
      current_tag = GET(tag_path)
      IF current_tag.status_code == 404:
        RETURN error "Tag does not exist"

      fingerprint = current_tag.json.fingerprint

    STEP 2: Merge updates with current tag
      updated_tag = {**current_tag.json, **updates}
      updated_tag.fingerprint = fingerprint

    STEP 3: Attempt update
      response = PUT(tag_path, body=updated_tag)

      IF response.status_code == 200:
        RETURN response.json

      IF response.status_code == 409:
        retry_count += 1
        IF retry_count >= max_retries:
          RETURN error "Max retries exceeded - workspace conflict"
        CONTINUE  # Retry loop

      IF response.status_code >= 400:
        RETURN error(response)
```

---

## Workflow: Handle Workspace Conflicts

### Algorithm

```
FUNCTION resolve_workspace_conflicts(workspace_path):

  STEP 1: Get workspace status
    status_response = GET(workspace_path + "/status")
    status = status_response.json

  STEP 2: Check for conflicts
    IF status.mergeConflict is empty:
      RETURN "No conflicts"

  STEP 3: Analyze conflicts
    conflicts = status.mergeConflict
    resolutions = []

    FOR EACH conflict IN conflicts:
      resolution = {
        "entityInWorkspace": {
          "tag": {"path": conflict.entityInWorkspace.tag.path}
        },
        "resolutionType": "renameConflict"  # Default strategy
      }
      resolutions.append(resolution)

  STEP 4: Resolve conflicts
    resolve_payload = {
      "fingerprint": workspace_status.fingerprint,
      "mergeConflict": resolutions
    }

    response = POST(workspace_path + ":resolve_conflict", body=resolve_payload)
    RETURN response.json
```

---

## Decision Tree: Choose HTTP Method

```
DECISION: What HTTP method to use?

INPUT: operation_type

IF operation_type == "create":
  RETURN "POST"

ELSE IF operation_type == "read_one":
  RETURN "GET"

ELSE IF operation_type == "read_list":
  RETURN "GET"

ELSE IF operation_type == "update":
  RETURN "PUT"

ELSE IF operation_type == "delete":
  RETURN "DELETE"

ELSE IF operation_type IN ["revert", "publish", "sync", "resolve_conflict"]:
  RETURN "POST"  # Special operations use POST

ELSE:
  RETURN error "Unknown operation type"
```

---

## Decision Tree: Determine Required Scopes

```
DECISION: What OAuth scopes are needed?

INPUT: operation, resource_type

required_scopes = ["tagmanager.readonly"]  # Always need read

IF operation IN ["create", "update"]:
  required_scopes.append("tagmanager.edit.containers")

IF operation == "delete" AND resource_type == "container":
  required_scopes.append("tagmanager.delete.containers")

IF operation == "publish":
  required_scopes.append("tagmanager.edit.containers")
  required_scopes.append("tagmanager.publish")

IF operation IN ["create_permission", "update_permission", "delete_permission"]:
  required_scopes.append("tagmanager.manage.users")

IF operation == "update_account":
  required_scopes.append("tagmanager.manage.accounts")

RETURN required_scopes
```

---

## Decision Tree: Error Recovery

```
DECISION: How to handle API error?

INPUT: http_status_code, error_response

IF http_status_code == 400:
  ACTION: Log request body, validate JSON syntax
  RETURN "Invalid request - fix and retry"

ELSE IF http_status_code == 401:
  ACTION: Refresh OAuth token
  RETRY: Yes, with new token

ELSE IF http_status_code == 403:
  error_reason = error_response.error.errors[0].reason

  IF error_reason == "rateLimitExceeded":
    ACTION: Implement exponential backoff
    WAIT: min(2^retry_count, 32) seconds
    RETRY: Yes, up to 5 times

  ELSE IF error_reason == "insufficientPermissions":
    ACTION: Check OAuth scopes
    RETURN "Need additional scopes - cannot retry"

  ELSE:
    RETURN "Forbidden - check permissions"

ELSE IF http_status_code == 404:
  ACTION: Verify resource path/ID
  RETURN "Resource not found - verify IDs"

ELSE IF http_status_code == 409:
  ACTION: Get fresh fingerprint
  RETRY: Yes, with updated fingerprint

ELSE IF http_status_code == 429:
  ACTION: Exponential backoff
  WAIT: 60 seconds minimum
  RETRY: Yes

ELSE IF http_status_code >= 500:
  ACTION: Exponential backoff
  WAIT: min(2^retry_count, 32) seconds
  RETRY: Yes, up to 5 times

ELSE:
  RETURN "Unexpected error"
```

---

## State Machine: Tag Lifecycle

```
STATES:
  - NONEXISTENT
  - DRAFT (in workspace)
  - VERSIONED (in container version)
  - PUBLISHED (in live version)
  - DELETED (soft delete)

TRANSITIONS:

FROM NONEXISTENT:
  create_tag() → DRAFT

FROM DRAFT:
  update_tag() → DRAFT
  delete_tag() → DELETED
  create_version() → VERSIONED (tag now exists in version AND workspace)
  revert_tag() → DRAFT (with base version content)

FROM VERSIONED:
  NO DIRECT MUTATIONS (immutable)
  publish_version() → PUBLISHED
  delete_version() → removes from version (tag may still be DRAFT in workspace)

FROM PUBLISHED:
  NO DIRECT MUTATIONS (immutable)
  create_new_version_without_tag() → tag removed from new published version

FROM DELETED:
  revert_tag() → DRAFT (restores from base version)
  create_version() → tag stays DELETED in version (not included)

RULES:
  - Cannot modify VERSIONED or PUBLISHED tags
  - To change published tag: modify in workspace, create version, publish
  - Delete in workspace only affects workspace, not live version
  - Revert restores to base version state
```

---

## State Machine: Workspace States

```
STATES:
  - UP_TO_DATE (matches base version)
  - MODIFIED (has unpublished changes)
  - CONFLICTED (base version changed, conflicts exist)
  - SYNCED (just synchronized with latest version)

TRANSITIONS:

FROM UP_TO_DATE:
  modify_entity() → MODIFIED

FROM MODIFIED:
  create_version() → UP_TO_DATE (changes now in new version)
  base_version_changes() → CONFLICTED

FROM CONFLICTED:
  resolve_conflict() → SYNCED

FROM SYNCED:
  modify_entity() → MODIFIED

OPERATIONS BY STATE:

UP_TO_DATE:
  - Can create/modify/delete entities
  - Can sync (no effect)
  - Cannot create version (no changes)

MODIFIED:
  - Can create/modify/delete entities
  - Can create version
  - Can sync (may cause conflicts)

CONFLICTED:
  - Cannot create version
  - Must resolve_conflict() first
  - Can view status

SYNCED:
  - Can create/modify/delete entities
  - Can create version if has changes
```

---

## Algorithm: Build Tag Parameter List

```
FUNCTION build_tag_parameters(tag_type, config):

  parameters = []

  IF tag_type == "gaawe":  # GA4 Tag
    parameters.append({
      "type": "template",
      "key": "measurementId",
      "value": config.measurement_id
    })

    IF config.config_parameters is not None:
      param_list = []
      FOR EACH param IN config.config_parameters:
        param_list.append({
          "type": "template",
          "key": "name",
          "value": param.name
        })
        param_list.append({
          "type": "template",
          "key": "value",
          "value": param.value
        })

      parameters.append({
        "type": "list",
        "key": "configParameter",
        "list": param_list
      })

  ELSE IF tag_type == "html":  # Custom HTML
    parameters.append({
      "type": "template",
      "key": "html",
      "value": config.html_code
    })

    parameters.append({
      "type": "boolean",
      "key": "supportDocumentWrite",
      "value": "false"
    })

  ELSE IF tag_type == "img":  # Image tag
    parameters.append({
      "type": "template",
      "key": "url",
      "value": config.image_url
    })

  ELSE IF tag_type == "awct":  # Google Ads Conversion
    parameters.append({
      "type": "template",
      "key": "conversionId",
      "value": config.conversion_id
    })

    parameters.append({
      "type": "template",
      "key": "conversionLabel",
      "value": config.conversion_label
    })

    IF config.conversion_value is not None:
      parameters.append({
        "type": "template",
        "key": "conversionValue",
        "value": config.conversion_value
      })

  ELSE:
    RETURN error "Unknown tag type - consult container for type-specific parameters"

  RETURN parameters
```

---

## Algorithm: Build Trigger Filter

```
FUNCTION build_trigger_filter(filter_type, variable_name, comparison_value):

  VALID_FILTER_TYPES = [
    "equals", "contains", "startsWith", "endsWith",
    "matchRegex", "greater", "less", "css"
  ]

  IF filter_type NOT IN VALID_FILTER_TYPES:
    RETURN error "Invalid filter type"

  filter = {
    "type": filter_type,
    "parameter": [
      {
        "type": "template",
        "key": "arg0",
        "value": "{{" + variable_name + "}}"
      },
      {
        "type": "template",
        "key": "arg1",
        "value": comparison_value
      }
    ]
  }

  RETURN filter
```

---

## Path Construction Algorithm

```
FUNCTION build_resource_path(resource_type, ids):

  base = "accounts/" + ids.account_id

  IF resource_type == "account":
    RETURN base

  IF resource_type == "container":
    RETURN base + "/containers/" + ids.container_id

  IF resource_type == "workspace":
    RETURN base + "/containers/" + ids.container_id + "/workspaces/" + ids.workspace_id

  IF resource_type IN ["tag", "trigger", "variable"]:
    workspace_path = base + "/containers/" + ids.container_id + "/workspaces/" + ids.workspace_id

    IF resource_type == "tag":
      RETURN workspace_path + "/tags/" + ids.tag_id
    ELSE IF resource_type == "trigger":
      RETURN workspace_path + "/triggers/" + ids.trigger_id
    ELSE IF resource_type == "variable":
      RETURN workspace_path + "/variables/" + ids.variable_id

  IF resource_type == "version":
    RETURN base + "/containers/" + ids.container_id + "/versions/" + ids.version_id

  IF resource_type == "environment":
    RETURN base + "/containers/" + ids.container_id + "/environments/" + ids.environment_id

  RETURN error "Unknown resource type"
```

---

## Notes for LLM Execution

### When executing workflows:

1. **Always validate inputs** before making API calls
2. **Extract IDs from response paths** - responses include full paths, extract just the ID
3. **Store fingerprints** when reading resources - needed for updates
4. **Check fingerprint in response** after update - it changes
5. **Handle pagination** for all list operations
6. **Implement backoff** for rate limit errors
7. **Never skip error handling** - every API call can fail
8. **Workspace isolation** - changes don't affect live until published

### Variable references:

- In parameters, reference variables by name: `"{{Variable Name}}"`
- Variable name must match exactly (case-sensitive)
- Built-in variables must be enabled before use
- Custom variables must exist in same workspace

### ID vs Path rules:

- **API endpoints**: Use full path
- **Entity references**: Use ID only (firingTriggerId, blockingTriggerId, etc.)
- **Response parsing**: Extract both path (for API calls) and ID (for references)

### Fingerprint rules:

- **Required for**: PUT operations (update)
- **Not required for**: POST (create), GET (read), DELETE
- **Get from**: Latest GET request before update
- **Changes after**: Every successful update
- **Conflict**: 409 error means fingerprint is stale
