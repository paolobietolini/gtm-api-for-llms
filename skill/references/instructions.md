# LLM Instruction Set for GTM API

Explicit instructions for LLMs to follow when executing GTM API operations.

---

## Core Execution Principles

### PRINCIPLE 1: Always validate before execute

```
BEFORE making ANY API call:
  1. Validate all required fields are present
  2. Validate field types and formats
  3. Validate cross-entity references exist
  4. Check container type compatibility
  5. Verify OAuth scopes are sufficient
```

### PRINCIPLE 2: Handle errors deterministically

```
AFTER receiving API response:
  1. Check HTTP status code
  2. IF status >= 400: Parse error response
  3. Apply error recovery algorithm
  4. Retry with exponential backoff if applicable
  5. If retry fails, return detailed error to user
```

### PRINCIPLE 3: Track state across operations

```
MAINTAIN state for:
  - Current workspace ID and fingerprint
  - Created entity IDs (for references)
  - OAuth token expiry
  - Retry count for rate limits
  - Last successful GET timestamps
```

### PRINCIPLE 4: Never assume - always verify

```
DO NOT assume:
  - Default workspace exists
  - Trigger IDs from user input are valid
  - Variables referenced in templates exist
  - Container type supports requested feature

ALWAYS verify by listing/getting resources first
```

---

## Instruction: Create a Tag

```
TASK: Create tag in GTM container

INPUTS:
  - account_id: string
  - container_id: string
  - tag_config: object with {name, type, trigger_ids, parameters}

ALGORITHM:

STEP 1: Validate inputs
  IF account_id is empty OR container_id is empty:
    RETURN error "Account ID and Container ID required"

  IF tag_config.name is empty:
    RETURN error "Tag name is required"

  IF tag_config.type is empty:
    RETURN error "Tag type is required"

  IF tag_config.trigger_ids is empty OR length == 0:
    RETURN error "At least one firing trigger required"

STEP 2: Get or create workspace
  workspaces = GET /accounts/{account_id}/containers/{container_id}/workspaces

  IF workspaces.workspace is empty:
    workspace = POST /accounts/{account_id}/containers/{container_id}/workspaces
      Body: {"name": "Default Workspace"}
  ELSE:
    workspace = workspaces.workspace[0]  // Use first workspace

  workspace_id = workspace.workspaceId
  workspace_path = workspace.path

STEP 3: Verify triggers exist
  FOR EACH trigger_id IN tag_config.trigger_ids:
    trigger = GET {workspace_path}/triggers/{trigger_id}

    IF trigger.status_code == 404:
      RETURN error "Trigger {trigger_id} does not exist in workspace"

STEP 4: Verify variable references
  IF tag_config.parameters contains "{{":
    Extract all variable references from parameters

    FOR EACH variable_name IN extracted_references:
      variables = GET {workspace_path}/variables

      found = False
      FOR EACH var IN variables.variable:
        IF var.name == variable_name:
          found = True
          BREAK

      IF NOT found:
        // Check if it's a built-in variable
        built_in_vars = GET {workspace_path}/built_in_variables

        IF variable_name NOT IN built_in_vars:
          RETURN error "Variable {{variable_name}} does not exist"

STEP 5: Build request body
  tag_body = {
    "name": tag_config.name,
    "type": tag_config.type,
    "firingTriggerId": tag_config.trigger_ids
  }

  IF tag_config.parameters is not None:
    tag_body["parameter"] = tag_config.parameters

  IF tag_config.notes is not None:
    tag_body["notes"] = tag_config.notes

  IF tag_config.blocking_trigger_ids is not None:
    tag_body["blockingTriggerId"] = tag_config.blocking_trigger_ids

STEP 6: Create tag
  response = POST {workspace_path}/tags
    Headers:
      Authorization: Bearer {access_token}
      Content-Type: application/json
    Body: tag_body

  IF response.status_code == 201 OR response.status_code == 200:
    tag = response.json
    RETURN {
      "success": True,
      "tag_id": tag.tagId,
      "tag_path": tag.path,
      "tag": tag
    }

  ELSE IF response.status_code == 409:
    // Conflict - refresh workspace and retry once
    workspace = GET {workspace_path}
    RETRY from STEP 5 (once only)

  ELSE IF response.status_code == 403:
    RETURN error "Insufficient permissions - need tagmanager.edit.containers scope"

  ELSE IF response.status_code == 400:
    RETURN error "Invalid request - " + response.json.error.message

  ELSE:
    RETURN error "API error: " + response.json.error.message

OUTPUT:
  Return tag ID and path to user
  Store tag_id for potential future reference
```

---

## Instruction: Update a Tag

```
TASK: Update existing tag

INPUTS:
  - tag_path: string (full path to tag)
  - updates: object with fields to update

ALGORITHM:

STEP 1: Get current tag state
  current_tag = GET {tag_path}

  IF current_tag.status_code == 404:
    RETURN error "Tag does not exist at path: {tag_path}"

  IF current_tag.status_code != 200:
    RETURN error "Failed to get tag: " + current_tag.json.error.message

  fingerprint = current_tag.json.fingerprint

STEP 2: Validate updates
  IF updates.name is not None AND updates.name is empty:
    RETURN error "Tag name cannot be empty"

  IF updates.firingTriggerId is not None AND length(updates.firingTriggerId) == 0:
    RETURN error "At least one firing trigger required"

  // Validate new trigger references if changing
  IF updates.firingTriggerId is not None:
    FOR EACH trigger_id IN updates.firingTriggerId:
      // Extract workspace path from tag_path
      workspace_path = extract_workspace_path(tag_path)
      trigger = GET {workspace_path}/triggers/{trigger_id}

      IF trigger.status_code == 404:
        RETURN error "Trigger {trigger_id} not found"

STEP 3: Merge updates with current tag
  updated_tag = {**current_tag.json, **updates}
  updated_tag["fingerprint"] = fingerprint

  // Ensure immutable fields not changed
  updated_tag["tagId"] = current_tag.json.tagId
  updated_tag["path"] = current_tag.json.path
  updated_tag["accountId"] = current_tag.json.accountId
  updated_tag["containerId"] = current_tag.json.containerId
  updated_tag["workspaceId"] = current_tag.json.workspaceId

STEP 4: Update tag
  max_retries = 3
  retry_count = 0

  WHILE retry_count < max_retries:
    response = PUT {tag_path}
      Headers:
        Authorization: Bearer {access_token}
        Content-Type: application/json
      Body: updated_tag

    IF response.status_code == 200:
      RETURN {
        "success": True,
        "tag": response.json
      }

    ELSE IF response.status_code == 409:
      // Fingerprint conflict - get fresh copy
      current_tag = GET {tag_path}
      fingerprint = current_tag.json.fingerprint
      updated_tag = {**current_tag.json, **updates}
      updated_tag["fingerprint"] = fingerprint

      retry_count += 1
      CONTINUE

    ELSE IF response.status_code == 403:
      RETURN error "Insufficient permissions"

    ELSE:
      RETURN error "Update failed: " + response.json.error.message

  RETURN error "Max retries exceeded - workspace conflict"

OUTPUT:
  Return updated tag to user
```

---

## Instruction: Publish Changes

```
TASK: Create version from workspace and publish

INPUTS:
  - account_id: string
  - container_id: string
  - workspace_id: string (optional, uses first if not provided)
  - version_name: string
  - version_notes: string

ALGORITHM:

STEP 1: Verify workspace
  IF workspace_id is None:
    workspaces = GET /accounts/{account_id}/containers/{container_id}/workspaces
    IF workspaces.workspace is empty:
      RETURN error "No workspaces found in container"
    workspace_id = workspaces.workspace[0].workspaceId

  workspace_path = build_path("workspace", {
    account_id: account_id,
    container_id: container_id,
    workspace_id: workspace_id
  })

STEP 2: Check workspace status
  status = GET {workspace_path}/status

  IF status.json.mergeConflict is not empty:
    RETURN error "Workspace has conflicts - resolve before publishing"

  IF status.json.workspaceChange is empty:
    RETURN error "No changes in workspace - nothing to publish"

STEP 3: Create version
  version_payload = {
    "name": version_name OR "Version created by API",
    "notes": version_notes OR "Automated publish"
  }

  version_response = POST {workspace_path}:create_version
    Headers:
      Authorization: Bearer {access_token}
      Content-Type: application/json
    Body: version_payload

  IF version_response.status_code != 200:
    RETURN error "Failed to create version: " + version_response.json.error.message

  version = version_response.json.containerVersion
  version_id = version.containerVersionId
  version_path = version.path

STEP 4: Publish version
  publish_response = POST {version_path}:publish
    Headers:
      Authorization: Bearer {access_token}
      Content-Type: application/json
    Body: {}

  IF publish_response.status_code != 200:
    RETURN error "Failed to publish version: " + publish_response.json.error.message

  published_version = publish_response.json.containerVersion

STEP 5: Return result
  RETURN {
    "success": True,
    "version_id": published_version.containerVersionId,
    "version_name": published_version.name,
    "message": "Version {version_id} published successfully"
  }

OUTPUT:
  Confirm to user that version is live
  Include version ID and name
```

---

## Instruction: List All Resources (with pagination)

```
TASK: List all resources of a type from a parent

INPUTS:
  - parent_path: string (e.g., workspace path)
  - resource_type: string ("tags" | "triggers" | "variables")

ALGORITHM:

STEP 1: Initialize collection
  all_resources = []
  page_token = None
  max_pages = 100  // Prevent infinite loop
  page_count = 0

STEP 2: Fetch pages
  WHILE page_count < max_pages:
    params = {}
    IF page_token is not None:
      params["pageToken"] = page_token

    response = GET {parent_path}/{resource_type}
      Headers:
        Authorization: Bearer {access_token}
      Params: params

    IF response.status_code == 403:
      wait_and_retry_with_backoff()
      CONTINUE

    IF response.status_code != 200:
      RETURN error "Failed to list {resource_type}: " + response.json.error.message

    // Extract resources based on type
    IF resource_type == "tags":
      resources = response.json.get("tag", [])
    ELSE IF resource_type == "triggers":
      resources = response.json.get("trigger", [])
    ELSE IF resource_type == "variables":
      resources = response.json.get("variable", [])
    ELSE:
      resources = response.json.get(resource_type, [])

    all_resources.extend(resources)

    page_token = response.json.get("nextPageToken", None)
    page_count += 1

    IF page_token is None:
      BREAK

  IF page_count >= max_pages:
    RETURN error "Exceeded max pages - possible pagination issue"

STEP 3: Return collection
  RETURN {
    "success": True,
    "count": length(all_resources),
    "resources": all_resources
  }

OUTPUT:
  Return complete list of resources
```

---

## Instruction: Handle Rate Limit

```
TASK: Execute request with exponential backoff for rate limits

INPUTS:
  - request_function: function that makes the API call
  - max_retries: integer (default 5)

ALGORITHM:

STEP 1: Initialize retry logic
  retry_count = 0
  base_wait = 1  // seconds

STEP 2: Execute with retry
  WHILE retry_count <= max_retries:
    response = request_function()

    IF response.status_code == 200 OR response.status_code == 201:
      RETURN response

    IF response.status_code == 403:
      error_reason = response.json.error.errors[0].reason

      IF error_reason == "rateLimitExceeded":
        wait_time = min(base_wait * (2 ** retry_count), 32)

        LOG "Rate limit exceeded, waiting {wait_time} seconds"
        SLEEP(wait_time)

        retry_count += 1
        CONTINUE

      ELSE:
        // Not rate limit - different 403 error
        RETURN response

    IF response.status_code == 429:
      wait_time = min(base_wait * (2 ** retry_count), 64)

      LOG "Too many requests, waiting {wait_time} seconds"
      SLEEP(wait_time)

      retry_count += 1
      CONTINUE

    // Other error - don't retry
    RETURN response

  RETURN error "Max retries exceeded for rate limit"

OUTPUT:
  Return successful response or final error
```

---

## Instruction: Build Tag with Parameters

```
TASK: Construct complete tag object with proper parameter structure

INPUTS:
  - tag_type: string (e.g., "gaawe", "html", "awct")
  - config: object with tag-specific configuration

ALGORITHM:

STEP 1: Initialize tag object
  tag = {
    "name": config.name,
    "type": tag_type,
    "firingTriggerId": config.trigger_ids,
    "parameter": []
  }

STEP 2: Build parameters based on type
  IF tag_type == "gaawe":  // GA4 Tag
    tag.parameter.append({
      "type": "template",
      "key": "measurementId",
      "value": config.measurement_id
    })

    IF config.event_name is not None:
      tag.parameter.append({
        "type": "template",
        "key": "eventName",
        "value": config.event_name
      })

    IF config.event_parameters is not None:
      param_list = []
      FOR EACH param IN config.event_parameters:
        param_list.append({
          "type": "map",
          "map": [
            {
              "type": "template",
              "key": "name",
              "value": param.name
            },
            {
              "type": "template",
              "key": "value",
              "value": param.value
            }
          ]
        })

      tag.parameter.append({
        "type": "list",
        "key": "eventParameters",
        "list": param_list
      })

  ELSE IF tag_type == "html":  // Custom HTML
    tag.parameter.append({
      "type": "template",
      "key": "html",
      "value": config.html_code
    })

    tag.parameter.append({
      "type": "boolean",
      "key": "supportDocumentWrite",
      "value": "false"
    })

  ELSE IF tag_type == "awct":  // Google Ads Conversion
    tag.parameter.append({
      "type": "template",
      "key": "conversionId",
      "value": config.conversion_id
    })

    tag.parameter.append({
      "type": "template",
      "key": "conversionLabel",
      "value": config.conversion_label
    })

    IF config.conversion_value is not None:
      tag.parameter.append({
        "type": "template",
        "key": "conversionValue",
        "value": config.conversion_value
      })

  ELSE:
    RETURN error "Unknown tag type: {tag_type}"

STEP 3: Add optional fields
  IF config.blocking_trigger_ids is not None:
    tag["blockingTriggerId"] = config.blocking_trigger_ids

  IF config.notes is not None:
    tag["notes"] = config.notes

  IF config.paused is not None:
    tag["paused"] = config.paused

STEP 4: Validate structure
  IF tag.parameter is empty AND tag_type != "gclidw":
    RETURN error "Tag parameters required for type {tag_type}"

  FOR EACH param IN tag.parameter:
    IF param.type is None:
      RETURN error "Parameter type is required"
    IF param.key is None:
      RETURN error "Parameter key is required"

STEP 5: Return tag object
  RETURN tag

OUTPUT:
  Return complete, valid tag object ready for API submission
```

---

## Common Task Instruction Sets

### Create GA4 Page View Tag

```
EXECUTE:
  1. Get/create workspace
  2. Create "All Pages" trigger if doesn't exist
  3. Create GA4 tag with:
     - type: "gaawe"
     - measurementId parameter
     - Reference to All Pages trigger
  4. Return tag ID
```

### Create Conversion Tracking Setup

```
EXECUTE:
  1. Get/create workspace
  2. Create "Thank You Page" trigger (pageview with filter)
  3. Create Google Ads Conversion tag
  4. Create Google Ads Conversion Linker tag (all pages)
  5. Create version
  6. Publish version
  7. Return version ID and tag IDs
```

### Audit Container Tags

```
EXECUTE:
  1. Get live version
  2. Extract all tags from version
  3. For each tag:
     - Extract type
     - Extract triggers
     - Extract parameters
  4. Generate report:
     - Total tags
     - Tags by type
     - Tags without notes
     - Paused tags
  5. Return report object
```

---

## Error Recovery Instructions

### 409 Conflict

```
ON ERROR 409:
  1. GET resource again to get fresh fingerprint
  2. Merge your changes with current state
  3. Retry PUT with new fingerprint
  4. Max 3 retries
  5. If still failing, return conflict error to user
```

### 403 Rate Limit

```
ON ERROR 403 with "rateLimitExceeded":
  1. Calculate wait time: min(2^retry_count, 32) seconds
  2. Sleep for wait time
  3. Retry request
  4. Max 5 retries
  5. If still failing, return rate limit error
```

### 404 Not Found

```
ON ERROR 404:
  1. Verify path/ID is correct
  2. If referencing entity: list parent to see available IDs
  3. If parent exists: entity was deleted
  4. Return "Resource not found" with suggested action
```

### 401 Unauthorized

```
ON ERROR 401:
  1. OAuth token is invalid or expired
  2. Attempt token refresh if refresh token available
  3. If refresh succeeds: retry request
  4. If refresh fails: return "Re-authentication required"
```

---

## Notes for LLM Implementation

1. **Always extract and store IDs from responses** - you'll need them for future operations
2. **Never skip validation steps** - invalid requests waste API quota
3. **Handle pagination for ALL list operations** - never assume single page
4. **Respect rate limits proactively** - space out requests if making many calls
5. **Provide detailed errors to users** - include what went wrong and how to fix
6. **Track workspace state** - know if you created it or used existing
7. **Clean up on failure** - if multi-step operation fails, consider reverting changes
8. **Log all API calls** - helps debug issues later
