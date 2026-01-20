# GTM API Resource Schemas

Complete schema definitions for all Google Tag Manager API resources.

---

## Table of Contents

- [Accounts](#accounts)
- [Containers](#containers)
- [Workspaces](#workspaces)
- [Tags](#tags)
- [Triggers](#triggers)
- [Variables](#variables)
- [Built-in Variables](#built-in-variables)
- [Container Versions](#container-versions)
- [Environments](#environments)
- [Folders](#folders)
- [Clients](#clients)
- [Templates](#templates)
- [Transformations](#transformations)
- [Zones](#zones)
- [User Permissions](#user-permissions)

---

## Accounts

GTM Account resource represents the top-level entity in the hierarchy.

### JSON Representation

```json
{
  "path": string,
  "accountId": string,
  "name": string,
  "shareData": boolean,
  "fingerprint": string,
  "tagManagerUrl": string,
  "features": {
    "supportUserPermissions": boolean,
    "supportMultipleContainers": boolean
  }
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | `string` | Output only | GTM Account's API relative path (format: `accounts/{accountId}`). |
| `accountId` | `string` | Output only | The Account ID uniquely identifies the GTM Account. |
| `name` | `string` | Required | Account display name. |
| `shareData` | `boolean` | Optional | Whether the account shares data anonymously with Google for benchmarking. |
| `fingerprint` | `string` | Output only | Fingerprint computed at storage time, recomputed on modification. |
| `tagManagerUrl` | `string` | Output only | Auto-generated link to the tag manager UI. |
| `features` | `object` | Output only | Read-only Account feature set. |

---

## Containers

Container resource represents a deployment target (web, mobile, server).

### JSON Representation

```json
{
  "path": string,
  "accountId": string,
  "containerId": string,
  "name": string,
  "domainName": [string],
  "publicId": string,
  "tagIds": [string],
  "features": {
    "supportUserPermissions": boolean,
    "supportEnvironments": boolean,
    "supportWorkspaces": boolean,
    "supportGtagConfigs": boolean,
    "supportBuiltInVariables": boolean,
    "supportClients": boolean,
    "supportFolders": boolean,
    "supportTags": boolean,
    "supportTemplates": boolean,
    "supportTriggers": boolean,
    "supportVariables": boolean,
    "supportVersions": boolean,
    "supportZones": boolean,
    "supportTransformations": boolean
  },
  "notes": string,
  "usageContext": [enum],
  "fingerprint": string,
  "tagManagerUrl": string,
  "taggingServerUrls": [string]
}
```

### Usage Context Enum

| Value | Description |
|-------|-------------|
| `web` | Web container (standard GTM) |
| `android` | Android mobile container |
| `ios` | iOS mobile container |
| `androidSdk5` | Android SDK v5 container |
| `iosSdk5` | iOS SDK v5 container |
| `amp` | AMP container |
| `server` | Server-side container |

---

## Workspaces

Workspace is an isolated environment for making changes to a container.

### JSON Representation

```json
{
  "path": string,
  "workspaceId": string,
  "accountId": string,
  "containerId": string,
  "name": string,
  "description": string,
  "fingerprint": string,
  "tagManagerUrl": string
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | `string` | Output only | Workspace API path. |
| `workspaceId` | `string` | Output only | Unique workspace identifier. |
| `accountId` | `string` | Output only | Parent account ID. |
| `containerId` | `string` | Output only | Parent container ID. |
| `name` | `string` | Required | Workspace name. |
| `description` | `string` | Optional | Workspace description. |
| `fingerprint` | `string` | Output only | Computed fingerprint. |
| `tagManagerUrl` | `string` | Output only | Link to workspace in UI. |

---

## Tags

Tags define tracking pixels, scripts, or server-side logic to execute.

### JSON Representation

```json
{
  "path": string,
  "tagId": string,
  "accountId": string,
  "containerId": string,
  "workspaceId": string,
  "name": string,
  "type": string,
  "notes": string,
  "firingTriggerId": [string],
  "blockingTriggerId": [string],
  "liveOnly": boolean,
  "parameter": [
    {
      "type": enum,
      "key": string,
      "value": string,
      "list": [Parameter],
      "map": [Parameter]
    }
  ],
  "fingerprint": string,
  "parentFolderId": string,
  "tagFiringOption": enum,
  "monitoringMetadata": {
    "type": string,
    "parameter": [Parameter]
  },
  "consentSettings": {
    "consentStatus": enum,
    "consentType": [
      {
        "consentType": string
      }
    ]
  },
  "paused": boolean,
  "priority": {
    "type": enum,
    "value": string
  },
  "scheduleStartMs": string,
  "scheduleEndMs": string,
  "tagManagerUrl": string
}
```

### Tag Type Field

The `type` field is a **string** with no predefined enum in the API specification. Tag types include:

1. **Built-in Google tags** (GA4, Google Ads, etc.)
2. **Custom HTML/Image tags**
3. **Custom template tags** (user-defined)
4. **Community Gallery templates**

**Note**: The API does not enumerate all possible tag types. Commonly observed types include:

| Type Code | Observed Name | Common Parameters |
|-----------|---------------|-------------------|
| `html` | Custom HTML tag | `html`, `supportDocumentWrite` |
| `img` | Image tag | `url`, `cacheBusterQueryParam` |
| `gaawe` | Google Analytics 4 Config | `measurementId`, `configParameter` |
| `googtag` | Google Tag (gtag.js) | `tagId` |
| `sp` | Google Ads Remarketing | `conversionId` |
| `flc` | Floodlight Counter | `activityTag`, `groupTag`, `countingMethod` |
| `fls` | Floodlight Sales | `activityTag`, `groupTag`, `revenue`, `orderId` |
| `gclidw` | Google Ads Conversion Linker | (no params) |
| `awct` | Google Ads Conversion Tracking | `conversionId`, `conversionLabel`, `conversionValue` |

**Important**: Reference existing tags in your container or GTM UI for correct type strings and parameters.

### Tag Firing Options

| Value | Description |
|-------|-------------|
| `oncePerEvent` | Fire once per event (default) |
| `oncePerLoad` | Fire once per page load |
| `unlimited` | Fire every time trigger conditions met |

### Parameter Types

| Type | Description |
|------|-------------|
| `template` | Simple string value |
| `integer` | Integer value |
| `boolean` | Boolean value |
| `list` | List of parameters |
| `map` | Key-value map of parameters |
| `tagReference` | Reference to another tag |

### Consent Status Enum

| Value | Description |
|-------|-------------|
| `notSet` | No consent requirement |
| `notNeeded` | Consent not needed |
| `needed` | Consent needed |

---

## Triggers

Triggers define when tags should fire.

### JSON Representation

```json
{
  "path": string,
  "triggerId": string,
  "accountId": string,
  "containerId": string,
  "workspaceId": string,
  "name": string,
  "type": enum,
  "notes": string,
  "filter": [
    {
      "type": enum,
      "parameter": [Parameter]
    }
  ],
  "autoEventFilter": [
    {
      "type": enum,
      "parameter": [Parameter]
    }
  ],
  "checkValidation": {
    "type": enum,
    "parameter": [Parameter]
  },
  "waitForTags": {
    "type": enum,
    "parameter": [Parameter]
  },
  "waitForTagsTimeout": {
    "type": enum,
    "parameter": [Parameter]
  },
  "uniqueTriggerId": string,
  "eventName": {
    "type": enum,
    "parameter": [Parameter]
  },
  "interval": {
    "type": enum,
    "parameter": [Parameter]
  },
  "limit": {
    "type": enum,
    "parameter": [Parameter]
  },
  "fingerprint": string,
  "parentFolderId": string,
  "tagManagerUrl": string
}
```

### Trigger Type Field

The `type` field is a **string** with no predefined enum in the API specification. Trigger types vary by container type.

**Note**: The API does not enumerate all trigger types. Commonly observed types include:

#### Web Container Triggers

| Type | Description | Usage |
|------|-------------|-------|
| `pageview` | Page View | Fires on page load |
| `domReady` | DOM Ready | Fires when DOM is ready |
| `windowLoaded` | Window Loaded | Fires when window is fully loaded |
| `customEvent` | Custom Event | Fires on custom event name |
| `linkClick` | Link Click | Fires on link clicks |
| `formSubmission` | Form Submission | Fires on form submit |
| `timer` | Timer | Fires on interval |
| `scrollDepth` | Scroll Depth | Fires at scroll percentage |
| `youTubeVideo` | YouTube Video | Fires on video events |
| `elementVisibility` | Element Visibility | Fires when element visible |
| `triggerGroup` | Trigger Group | Logical combination of triggers |
| `historyChange` | History Change | Fires on history API events |
| `javascriptError` | JavaScript Error | Fires on JS errors |
| `clickAll` | All Elements Click | Fires on any click |

#### Server Container Triggers

| Type | Description | Usage |
|------|-------------|-------|
| `serverPageview` | Server-side Pageview | Server container pageview |
| `serverCustomEvent` | Server-side Custom Event | Server container custom event |

**Important**: Reference existing triggers in your container or GTM UI documentation for correct type strings.

### Filter Types

| Type | Description |
|------|-------------|
| `equals` | Value equals |
| `contains` | Value contains substring |
| `startsWith` | Value starts with |
| `endsWith` | Value ends with |
| `matchRegex` | Value matches regex |
| `greater` | Value greater than |
| `less` | Value less than |
| `css` | CSS selector matches |

### Trigger Groups (type: `triggerGroup`)

Trigger groups combine multiple triggers using logical AND/OR operations. The member triggers are specified using the `parameter` field with a specific structure.

**Important**: The parameter structure for trigger groups is non-obvious. You must use the exact format below.

#### Parameter Structure for Trigger Groups

```json
{
  "name": "My Trigger Group",
  "type": "triggerGroup",
  "parameter": [
    {
      "key": "triggerIds",
      "type": "list",
      "list": [
        {"type": "triggerReference", "value": "14"},
        {"type": "triggerReference", "value": "15"}
      ]
    }
  ]
}
```

#### Key Points

1. **`parameter`** must contain an array with a single object
2. **`key`** must be exactly `"triggerIds"` (case-sensitive)
3. **`type`** must be `"list"` at the outer level
4. **`list`** contains the member triggers
5. Each member trigger uses:
   - `"type": "triggerReference"` (not `template` or `tagReference`)
   - `"value": "<triggerId>"` - the trigger ID as a string

#### Example: Creating a Trigger Group via API

```json
{
  "name": "Pageview AND Click Group",
  "type": "triggerGroup",
  "parameter": [
    {
      "key": "triggerIds",
      "type": "list",
      "list": [
        {"type": "triggerReference", "value": "123"},
        {"type": "triggerReference", "value": "456"}
      ]
    }
  ]
}
```

#### Common Mistakes

- **Wrong key**: Using `"triggers"` instead of `"triggerIds"`
- **Wrong list item type**: Using `"template"` or `"tagReference"` instead of `"triggerReference"`
- **Wrong nesting**: Putting triggers directly in `parameter` instead of inside `list`
- **Using `key` in list items**: List items should only have `type` and `value`

---

## Variables

Variables store and retrieve values for use in tags and triggers.

### JSON Representation

```json
{
  "path": string,
  "variableId": string,
  "accountId": string,
  "containerId": string,
  "workspaceId": string,
  "name": string,
  "type": string,
  "notes": string,
  "scheduleStartMs": string,
  "scheduleEndMs": string,
  "parameter": [Parameter],
  "enablingTriggerId": [string],
  "disablingTriggerId": [string],
  "fingerprint": string,
  "parentFolderId": string,
  "formatValue": {
    "caseConversionType": enum,
    "convertNullToValue": {
      "type": enum,
      "value": string
    },
    "convertUndefinedToValue": {
      "type": enum,
      "value": string
    },
    "convertTrueToValue": {
      "type": enum,
      "value": string
    },
    "convertFalseToValue": {
      "type": enum,
      "value": string
    }
  },
  "tagManagerUrl": string
}
```

### Variable Type Field

The `type` field is a **string** with no predefined enum in the API specification. Variable types are determined by:

1. **Built-in GTM variable types** (created via GTM UI)
2. **Custom template types** (user-defined templates)
3. **Community template types** (from Template Gallery)

**Note**: The API does not provide an exhaustive list of variable types. Types are implementation-specific strings that GTM recognizes internally. Common types observed in GTM containers include:

| Type Code | Observed Name | Typical Use |
|-----------|---------------|-------------|
| `c` | Constant | Static value |
| `jsm` | Custom JavaScript | Execute JS function |
| `v` | Data Layer Variable | Read from dataLayer |
| `k` | First-Party Cookie | Read cookie value |
| `u` | URL | Parse URL components |
| `f` | Referrer | Get referrer URL |
| `aev` | Auto-Event Variable | Click/form data |
| `r` | Random Number | Generate random number |
| `e` | Custom Event | Event variable |
| `gas` | Google Analytics Settings | GA settings variable |

**Important**: This list is not authoritative or complete. When creating variables via API, reference existing variables in your container or GTM UI documentation for correct type strings.

### Case Conversion Types

| Value | Description |
|-------|-------------|
| `lowercase` | Convert to lowercase |
| `uppercase` | Convert to uppercase |
| `none` | No conversion |

---

## Built-in Variables

Pre-configured variables available in containers.

### Web Container Built-in Variables

| Variable | Type | Description |
|----------|------|-------------|
| `pageUrl` | URL | Page URL |
| `pageHostname` | URL | Page hostname |
| `pagePath` | URL | Page path |
| `referrer` | URL | Referrer URL |
| `clickElement` | Element | Clicked element |
| `clickClasses` | Element | Click element classes |
| `clickId` | Element | Click element ID |
| `clickTarget` | Element | Click target |
| `clickUrl` | Element | Click URL |
| `clickText` | Element | Click text |
| `formElement` | Form | Form element |
| `formClasses` | Form | Form classes |
| `formId` | Form | Form ID |
| `formTarget` | Form | Form target |
| `formUrl` | Form | Form URL |
| `formText` | Form | Form text |
| `errorMessage` | Error | JS error message |
| `errorUrl` | Error | Error URL |
| `errorLine` | Error | Error line number |
| `newHistoryState` | History | History state object |
| `oldHistoryState` | History | Previous history state |
| `historySource` | History | History change source |
| `randomNumber` | Utility | Random number |
| `containerId` | Container | Container public ID |
| `containerVersion` | Container | Container version |
| `debugMode` | Container | Debug mode status |
| `videoProvider` | Video | Video provider (YouTube) |
| `videoStatus` | Video | Video status |
| `videoUrl` | Video | Video URL |
| `videoTitle` | Video | Video title |
| `videoDuration` | Video | Video duration |
| `videoPercent` | Video | Video % watched |
| `videoVisible` | Video | Video visibility |
| `scrollDepthThreshold` | Scroll | Scroll depth % |
| `scrollDepthUnits` | Scroll | Scroll depth units |
| `scrollDirection` | Scroll | Scroll direction |

### Server Container Built-in Variables

| Variable | Type | Description |
|----------|------|-------------|
| `eventName` | Event | Event name |
| `clientName` | Client | Client name |
| `requestPath` | Request | Request path |
| `requestMethod` | Request | HTTP method |
| `requestHost` | Request | Request host |
| `ipAddress` | Request | Client IP address |
| `userAgent` | Request | User agent string |

---

## Container Versions

Immutable snapshots of container configuration.

### JSON Representation

```json
{
  "path": string,
  "containerVersionId": string,
  "accountId": string,
  "containerId": string,
  "name": string,
  "description": string,
  "container": Container,
  "tag": [Tag],
  "trigger": [Trigger],
  "variable": [Variable],
  "folder": [Folder],
  "builtInVariable": [enum],
  "client": [Client],
  "transformation": [Transformation],
  "zone": [Zone],
  "customTemplate": [CustomTemplate],
  "fingerprint": string,
  "tagManagerUrl": string,
  "deleted": boolean
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `path` | `string` | Version API path |
| `containerVersionId` | `string` | Version ID |
| `name` | `string` | Version name |
| `description` | `string` | Version description |
| `tag` | `Tag[]` | All tags in version |
| `trigger` | `Trigger[]` | All triggers in version |
| `variable` | `Variable[]` | All variables in version |
| `builtInVariable` | `enum[]` | Enabled built-in variables |
| `deleted` | `boolean` | Whether version is deleted |

---

## Environments

Deployment targets for container versions.

### JSON Representation

```json
{
  "path": string,
  "environmentId": string,
  "accountId": string,
  "containerId": string,
  "name": string,
  "description": string,
  "type": enum,
  "url": string,
  "authorizationCode": string,
  "authorizationTimestamp": string,
  "enableDebug": boolean,
  "fingerprint": string,
  "containerVersionId": string,
  "tagManagerUrl": string
}
```

### Environment Types

| Type | Description |
|------|-------------|
| `user` | User-created environment |
| `live` | Production environment |
| `latest` | Latest (preview) environment |

---

## Folders

Organizational units for tags, triggers, and variables.

### JSON Representation

```json
{
  "path": string,
  "folderId": string,
  "accountId": string,
  "containerId": string,
  "workspaceId": string,
  "name": string,
  "notes": string,
  "fingerprint": string,
  "tagManagerUrl": string
}
```

---

## Clients

Server-side container clients that receive and process incoming requests.

### JSON Representation

```json
{
  "path": string,
  "clientId": string,
  "accountId": string,
  "containerId": string,
  "workspaceId": string,
  "name": string,
  "type": string,
  "notes": string,
  "parameter": [Parameter],
  "fingerprint": string,
  "parentFolderId": string,
  "priority": integer,
  "tagManagerUrl": string
}
```

### Common Client Types

| Type | Description |
|------|-------------|
| `gaaw` | GA4 Web Client |
| `sp` | Google Ads Client |

---

## Templates

Custom tag, variable, or client templates.

### JSON Representation

```json
{
  "path": string,
  "templateId": string,
  "accountId": string,
  "containerId": string,
  "workspaceId": string,
  "name": string,
  "templateData": string,
  "fingerprint": string,
  "tagManagerUrl": string,
  "galleryReference": {
    "host": string,
    "owner": string,
    "repository": string,
    "signature": string,
    "version": string
  }
}
```

---

## Transformations

Server-side transformations that modify event data.

### JSON Representation

```json
{
  "path": string,
  "transformationId": string,
  "accountId": string,
  "containerId": string,
  "workspaceId": string,
  "name": string,
  "type": string,
  "notes": string,
  "parameter": [Parameter],
  "fingerprint": string,
  "parentFolderId": string,
  "tagManagerUrl": string
}
```

---

## Zones

Security zones that restrict where tags can fire.

### JSON Representation

```json
{
  "path": string,
  "zoneId": string,
  "accountId": string,
  "containerId": string,
  "workspaceId": string,
  "name": string,
  "notes": string,
  "childContainer": [
    {
      "publicId": string,
      "nickname": string
    }
  ],
  "boundary": {
    "condition": [Condition],
    "customEvaluationTriggerId": [string]
  },
  "typeRestriction": {
    "enable": boolean,
    "whitelistedTypeId": [string]
  },
  "fingerprint": string,
  "tagManagerUrl": string
}
```

---

## User Permissions

Access control for GTM accounts and containers.

### JSON Representation

```json
{
  "path": string,
  "accountId": string,
  "containerId": string,
  "emailAddress": string,
  "accountAccess": {
    "permission": enum
  },
  "containerAccess": [
    {
      "containerId": string,
      "permission": enum
    }
  ]
}
```

### Permission Levels

| Permission | Description |
|------------|-------------|
| `read` | Read-only access |
| `edit` | Edit containers and tags |
| `approve` | Approve and publish |
| `publish` | Publish versions |
| `admin` | Full administrative access |
| `noAccess` | No access |

---

## Common Fields Across Resources

All mutable resources share these common fields:

| Field | Type | Description |
|-------|------|-------------|
| `path` | `string` | Full API resource path |
| `accountId` | `string` | Parent account ID |
| `containerId` | `string` | Parent container ID |
| `workspaceId` | `string` | Parent workspace ID (if applicable) |
| `name` | `string` | Display name |
| `notes` | `string` | User notes |
| `fingerprint` | `string` | Version control fingerprint |
| `tagManagerUrl` | `string` | UI link |
| `parentFolderId` | `string` | Containing folder ID |

---

## Parameter Object Structure

Parameters are used throughout tags, triggers, variables, and other entities.

```json
{
  "type": enum,
  "key": string,
  "value": string,
  "list": [Parameter],
  "map": [Parameter]
}
```

### Parameter Types

- `template` - String template with variable interpolation
- `integer` - Integer value
- `boolean` - Boolean value
- `list` - Array of parameters
- `map` - Key-value parameter pairs
- `tagReference` - Reference to tag ID

---

## Notes for LLMs

1. **All IDs are strings**, even if they look numeric
2. **Fingerprints are required** for updates to prevent conflicts
3. **Path format** is consistent: `accounts/{accountId}/containers/{containerId}/workspaces/{workspaceId}/tags/{tagId}`
4. **Array notation** `[]` indicates repeated fields
5. **References between entities** use IDs, not paths
6. **Built-in variables** must be explicitly enabled before use
7. **Server-side resources** (clients, transformations) only exist in server containers
8. **Zone restrictions** only work in web containers
9. **Template data** uses GTM's sandboxed JavaScript dialect
10. **Parameters can be nested** - lists can contain maps, maps can contain lists
