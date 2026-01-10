# GTM API Validation Rules for LLMs

Explicit validation rules and constraints for generating valid API requests.

---

## Required Fields by Operation

### Create Tag

```
REQUIRED:
  ✓ name (string, non-empty)
  ✓ type (string, valid tag type)
  ✓ firingTriggerId (array of strings, min 1 trigger ID)

OPTIONAL:
  - blockingTriggerId (array of strings)
  - parameter (array of Parameter objects)
  - notes (string)
  - parentFolderId (string)
  - liveOnly (boolean, default: false)
  - tagFiringOption (enum: "oncePerEvent" | "oncePerLoad" | "unlimited")
  - paused (boolean, default: false)
  - priority (object with type and value)
  - scheduleStartMs (string timestamp)
  - scheduleEndMs (string timestamp)
  - consentSettings (object)
  - monitoringMetadata (object)

MUST NOT INCLUDE:
  ✗ tagId (auto-generated)
  ✗ path (auto-generated)
  ✗ fingerprint (only for updates)
  ✗ accountId (derived from URL)
  ✗ containerId (derived from URL)
  ✗ workspaceId (derived from URL)
  ✗ tagManagerUrl (auto-generated)
```

### Update Tag

```
REQUIRED:
  ✓ ALL fields from current tag (GET first)
  ✓ fingerprint (from current tag)
  ✓ name
  ✓ type
  ✓ firingTriggerId

OPTIONAL:
  - Same as Create Tag

MUST NOT CHANGE:
  ✗ tagId (immutable)
  ✗ path (immutable)
  ✗ accountId (immutable)
  ✗ containerId (immutable)
  ✗ workspaceId (immutable)

PROCESS:
  1. GET current tag to get fingerprint
  2. Merge changes with current tag
  3. Include updated fingerprint in PUT body
```

### Create Trigger

```
REQUIRED:
  ✓ name (string, non-empty)
  ✓ type (string, valid trigger type)

CONDITIONAL:
  IF type == "customEvent":
    ✓ eventName (Parameter object with type and value)

  IF type == "timer":
    ✓ interval (Parameter object with integer value)

  IF type requires filtering (most types):
    ~ filter (array of Condition objects) - optional but common

OPTIONAL:
  - notes (string)
  - parentFolderId (string)
  - autoEventFilter (array)
  - checkValidation (Parameter)
  - waitForTags (Parameter)
  - waitForTagsTimeout (Parameter)
  - limit (Parameter) - for timer triggers

MUST NOT INCLUDE:
  ✗ triggerId
  ✗ path
  ✗ fingerprint (only for updates)
  ✗ uniqueTriggerId
```

### Create Variable

```
REQUIRED:
  ✓ name (string, non-empty)
  ✓ type (string, valid variable type)
  ✓ parameter (array, type-specific)

CONDITIONAL:
  IF type == "v" (Data Layer Variable):
    ✓ parameter MUST include:
      - {key: "name", type: "template", value: "dataLayerKey"}

  IF type == "c" (Constant):
    ✓ parameter MUST include:
      - {key: "value", type: "template", value: "constantValue"}

  IF type == "k" (Cookie):
    ✓ parameter MUST include:
      - {key: "name", type: "template", value: "cookieName"}

OPTIONAL:
  - notes (string)
  - parentFolderId (string)
  - formatValue (object for value transformation)
  - enablingTriggerId (array of trigger IDs)
  - disablingTriggerId (array of trigger IDs)
  - scheduleStartMs (string)
  - scheduleEndMs (string)

MUST NOT INCLUDE:
  ✗ variableId
  ✗ path
  ✗ fingerprint (only for updates)
```

### Create Workspace

```
REQUIRED:
  ✓ name (string, non-empty)

OPTIONAL:
  - description (string)

MUST NOT INCLUDE:
  ✗ workspaceId
  ✗ path
  ✗ fingerprint
```

### Create Version

```
REQUIRED:
  None (POST to workspace:create_version endpoint)

OPTIONAL:
  - name (string, version name)
  - notes (string, version description)

REQUEST BODY:
  {
    "name": "Version name",
    "notes": "Description of changes"
  }
```

### Publish Version

```
REQUIRED:
  None (POST to version:publish endpoint)

OPTIONAL:
  - fingerprint (version fingerprint)

REQUEST BODY:
  Can be empty {}
```

---

## Field Type Validation

### String Fields

```
VALIDATION RULES:

name:
  - Min length: 1
  - Max length: 256
  - Cannot be only whitespace
  - Example: "GA4 - Page View"

notes:
  - Max length: unlimited
  - Can be empty
  - Can contain newlines

type (tag):
  - Must match known tag type OR custom template type
  - Case-sensitive
  - Examples: "gaawe", "html", "img", "awct"
  - Cannot be empty

type (trigger):
  - Must match valid trigger type for container
  - Case-sensitive
  - Examples: "pageview", "customEvent", "timer"

type (variable):
  - Must match known variable type OR custom template type
  - Case-sensitive
  - Examples: "c", "v", "k", "jsm"
```

### Array Fields

```
firingTriggerId:
  - Type: array of strings
  - Min length: 1 (at least one trigger)
  - Each element: trigger ID from SAME workspace
  - Example: ["5", "12", "23"]

blockingTriggerId:
  - Type: array of strings
  - Min length: 0 (optional)
  - Each element: trigger ID from SAME workspace
  - Example: ["8"]

parameter:
  - Type: array of Parameter objects
  - Min length: 0 (depends on tag/variable type)
  - Structure: see Parameter Object Validation
```

### Boolean Fields

```
liveOnly:
  - Type: boolean
  - Values: true | false
  - Default: false

paused:
  - Type: boolean
  - Values: true | false
  - Default: false

shareData (Account):
  - Type: boolean
  - Values: true | false

enableDebug (Environment):
  - Type: boolean
  - Values: true | false
```

### Enum Fields

```
tagFiringOption:
  - Values: "oncePerEvent" | "oncePerLoad" | "unlimited"
  - Default: "oncePerEvent"

usageContext (Container):
  - Values: ["web"] | ["android"] | ["ios"] | ["amp"] | ["server"]
  - Can be array with one element

environmentType:
  - Values: "user" | "live" | "latest"

consentStatus:
  - Values: "notSet" | "notNeeded" | "needed"
```

---

## Parameter Object Validation

### Parameter Structure

```
VALID STRUCTURES:

1. Simple template:
   {
     "type": "template",
     "key": "paramName",
     "value": "paramValue"
   }

2. Boolean:
   {
     "type": "boolean",
     "key": "paramName",
     "value": "true"  // Note: string "true" or "false"
   }

3. Integer:
   {
     "type": "integer",
     "key": "paramName",
     "value": "42"  // Note: string representation
   }

4. List:
   {
     "type": "list",
     "key": "paramName",
     "list": [
       {Parameter}, {Parameter}, ...
     ]
   }

5. Map:
   {
     "type": "map",
     "key": "paramName",
     "map": [
       {Parameter}, {Parameter}, ...
     ]
   }

6. Tag reference:
   {
     "type": "tagReference",
     "key": "paramName",
     "value": "tagId"
   }
```

### Parameter Validation Rules

```
RULE 1: Type field is always required
  parameter.type MUST be one of:
    - "template"
    - "boolean"
    - "integer"
    - "list"
    - "map"
    - "tagReference"

RULE 2: Key field is always required
  parameter.key MUST be:
    - Non-empty string
    - Matches expected parameter key for tag/variable type

RULE 3: Value XOR List XOR Map
  IF type == "template" | "boolean" | "integer" | "tagReference":
    MUST have: value (string)
    MUST NOT have: list, map

  IF type == "list":
    MUST have: list (array of Parameters)
    MUST NOT have: value, map

  IF type == "map":
    MUST have: map (array of Parameters)
    MUST NOT have: value, list

RULE 4: Value format by type
  IF type == "boolean":
    value MUST be "true" or "false" (string)

  IF type == "integer":
    value MUST be string representation of integer

  IF type == "template":
    value can be:
      - Literal string: "some text"
      - Variable reference: "{{Variable Name}}"
      - Mixed: "Value is {{Variable Name}}"

  IF type == "tagReference":
    value MUST be tag ID (string)
```

---

## Cross-Entity Validation

### Tag References

```
VALIDATION:

firingTriggerId:
  FOR EACH trigger_id IN firingTriggerId:
    ✓ trigger_id MUST exist in SAME workspace
    ✓ trigger_id MUST be string
    ✓ trigger MUST NOT be deleted

blockingTriggerId:
  FOR EACH trigger_id IN blockingTriggerId:
    ✓ trigger_id MUST exist in SAME workspace
    ✓ trigger_id MUST be string
    ✓ Can reference same trigger as firingTriggerId

parentFolderId:
  IF parentFolderId is not None:
    ✓ folder MUST exist in SAME workspace
    ✓ folder MUST NOT be deleted
```

### Variable References

```
VALIDATION:

In parameter values:
  IF value contains "{{" and "}}":
    variable_name = extract text between {{ and }}
    ✓ Variable with name==variable_name MUST exist in workspace OR
    ✓ Built-in variable with that name MUST be enabled

enablingTriggerId / disablingTriggerId:
  FOR EACH trigger_id:
    ✓ trigger MUST exist in SAME workspace
```

### Workspace References

```
VALIDATION:

When creating entities in workspace:
  ✓ workspace MUST exist
  ✓ workspace MUST NOT be deleted
  ✓ workspace MUST be in same container

When creating version from workspace:
  ✓ workspace MUST have changes (status != UP_TO_DATE)
  ✓ workspace MUST NOT have unresolved conflicts
```

---

## Container Type Constraints

### Web Container

```
ALLOWED:
  ✓ Tags (all types except server-side)
  ✓ Triggers (all web types)
  ✓ Variables (all web types)
  ✓ Zones
  ✓ Google Tag Config

NOT ALLOWED:
  ✗ Clients
  ✗ Transformations
  ✗ Server-side trigger types
```

### Server Container

```
ALLOWED:
  ✓ Tags (server-side types)
  ✓ Triggers (server-side types)
  ✓ Variables (server-side types)
  ✓ Clients
  ✓ Transformations

NOT ALLOWED:
  ✗ Zones
  ✗ Google Tag Config
  ✗ Web-specific trigger types
  ✗ Web-specific built-in variables
```

### Validation Algorithm

```
FUNCTION validate_entity_for_container(entity_type, container):

  container_usage = container.usageContext[0]  // "web", "server", etc.

  IF entity_type == "zone":
    RETURN container_usage == "web"

  IF entity_type == "client" OR entity_type == "transformation":
    RETURN container_usage == "server"

  IF entity_type == "gtag_config":
    RETURN container_usage == "web"

  // Tags, triggers, variables: check specific type
  RETURN True  // Most entities allowed in all containers
```

---

## Fingerprint Validation

### When Required

```
OPERATION → FINGERPRINT REQUIRED?

POST (create):     ✗ NO  - Do not include
GET (read):        ✗ NO  - Returned in response
PUT (update):      ✓ YES - Must match current
DELETE:            ✗ NO  - Not needed
POST (special):    ~ MAYBE - Check specific operation
```

### Fingerprint Rules

```
RULE 1: Fingerprint must be current
  fingerprint_in_request == latest_fingerprint_from_GET

RULE 2: Fingerprint changes after update
  After successful PUT:
    old_fingerprint != new_fingerprint

RULE 3: 409 Conflict means stale fingerprint
  IF response.status == 409:
    fingerprint_in_request != current_fingerprint
    ACTION: GET fresh copy, retry with new fingerprint

RULE 4: Workspace operations need workspace fingerprint
  When updating workspace entities:
    Some operations may need workspace.fingerprint
    Check API response for fingerprint requirements
```

---

## Path Format Validation

### Path Construction Rules

```
RULE: Path segments must be in order

✓ VALID:
  accounts/123/containers/456
  accounts/123/containers/456/workspaces/10
  accounts/123/containers/456/workspaces/10/tags/5

✗ INVALID:
  containers/456/accounts/123  (wrong order)
  accounts/123/workspaces/10   (missing container)
  accounts//containers/456     (empty segment)

RULE: IDs must be non-empty strings

✓ VALID:
  accounts/12345
  accounts/6789

✗ INVALID:
  accounts/      (empty ID)
  accounts/null  (literal "null")

RULE: No trailing slashes

✓ VALID:
  accounts/123/containers/456/workspaces/10/tags

✗ INVALID:
  accounts/123/containers/456/workspaces/10/tags/
```

### ID Extraction

```
FUNCTION extract_id_from_path(path, resource_type):

  // Example path: "accounts/123/containers/456/workspaces/10/tags/5"

  segments = path.split("/")

  IF resource_type == "account":
    RETURN segments[1]  // "123"

  IF resource_type == "container":
    RETURN segments[3]  // "456"

  IF resource_type == "workspace":
    RETURN segments[5]  // "10"

  IF resource_type == "tag":
    RETURN segments[7]  // "5"

  IF resource_type == "trigger":
    RETURN segments[7]

  IF resource_type == "variable":
    RETURN segments[7]

  IF resource_type == "version":
    RETURN segments[5]  // versions are at container level

  IF resource_type == "environment":
    RETURN segments[5]
```

---

## Common Validation Errors

### Error: Missing required field

```
SYMPTOM:
  HTTP 400 Bad Request
  Error: "Required field missing"

CAUSES:
  - name is empty or missing
  - type is empty or missing
  - firingTriggerId is empty array or missing

FIX:
  Review Required Fields by Operation section
  Ensure all REQUIRED fields are present
```

### Error: Invalid parameter type

```
SYMPTOM:
  HTTP 400 Bad Request
  Error: "Invalid parameter"

CAUSES:
  - parameter.type is not valid enum value
  - boolean value is not "true" or "false" string
  - integer value is not numeric string

FIX:
  Review Parameter Object Validation
  Check parameter.type is valid
  Check value format matches type
```

### Error: Entity reference not found

```
SYMPTOM:
  HTTP 400 Bad Request
  Error: "Referenced entity does not exist"

CAUSES:
  - firingTriggerId references non-existent trigger
  - Variable reference "{{VarName}}" doesn't exist
  - parentFolderId references deleted folder

FIX:
  List entities in workspace to verify IDs
  Check entity names match exactly (case-sensitive)
  Verify entities are in same workspace
```

### Error: Fingerprint mismatch

```
SYMPTOM:
  HTTP 409 Conflict

CAUSES:
  - Using stale fingerprint from old GET request
  - Entity was modified by another process
  - Workspace was synchronized

FIX:
  GET entity again to get fresh fingerprint
  Merge your changes with current state
  Retry PUT with new fingerprint
```

---

## Validation Checklist for LLMs

Before making API call:

```
CREATE TAG:
  ☐ name is non-empty string
  ☐ type is valid tag type string
  ☐ firingTriggerId is array with at least one trigger ID
  ☐ All trigger IDs exist in workspace
  ☐ All variable references exist or are built-in
  ☐ parameters match tag type requirements
  ☐ No fingerprint included
  ☐ No auto-generated fields included

UPDATE TAG:
  ☐ GET tag first to get current state
  ☐ Extract fingerprint from GET response
  ☐ Merge changes with current tag
  ☐ Include fingerprint in PUT body
  ☐ All required fields still present
  ☐ No immutable fields changed

CREATE TRIGGER:
  ☐ name is non-empty string
  ☐ type is valid trigger type
  ☐ Required conditional fields present (e.g., eventName for customEvent)
  ☐ filter array structure is valid
  ☐ No fingerprint included

CREATE VARIABLE:
  ☐ name is non-empty string
  ☐ type is valid variable type
  ☐ parameters match variable type
  ☐ No fingerprint included

PUBLISH VERSION:
  ☐ Version was created from workspace
  ☐ Version ID is valid
  ☐ Have tagmanager.publish scope
```
