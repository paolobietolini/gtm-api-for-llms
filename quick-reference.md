# GTM API Quick Reference

Fast lookup tables for common GTM API operations and mappings.

---

## Table of Contents

- [Resource Operations Matrix](#resource-operations-matrix)
- [OAuth Scopes by Action](#oauth-scopes-by-action)
- [Container Type Features](#container-type-features)
- [HTTP Methods by Operation](#http-methods-by-operation)
- [Tag Types Reference](#tag-types-reference)
- [Trigger Types Reference](#trigger-types-reference)
- [Variable Types Reference](#variable-types-reference)
- [Built-in Variables by Container Type](#built-in-variables-by-container-type)
- [Error Codes](#error-codes)
- [Path Format Patterns](#path-format-patterns)
- [Entity Relationships](#entity-relationships)
- [Publishing Workflow States](#publishing-workflow-states)

---

## Resource Operations Matrix

| Resource | Create | Read | Update | Delete | List | Revert | Other Operations |
|----------|--------|------|--------|--------|------|--------|------------------|
| Account | ❌ | ✅ | ✅ | ❌ | ✅ | ❌ | - |
| Container | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | combine, lookup, snippet, move_tag_id |
| Workspace | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | sync, resolve_conflict, create_version, quick_preview, status |
| Tag | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | - |
| Trigger | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | - |
| Variable | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | - |
| Built-in Variable | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | create (enable), delete (disable) |
| Folder | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | move_entities_to_folder |
| Version | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | publish, undelete, set_latest, live, latest |
| Version Header | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | latest |
| Environment | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | reauthorize |
| Client | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | - |
| Transformation | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | - |
| Zone | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | - |
| Template | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | - |
| Google Tag Config | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | - |
| User Permission | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | - |

---

## OAuth Scopes by Action

| Action | Required Scope | Scope URL |
|--------|----------------|-----------|
| View containers, tags, triggers | `tagmanager.readonly` | `https://www.googleapis.com/auth/tagmanager.readonly` |
| Create/edit containers | `tagmanager.edit.containers` | `https://www.googleapis.com/auth/tagmanager.edit.containers` |
| Delete containers | `tagmanager.delete.containers` | `https://www.googleapis.com/auth/tagmanager.delete.containers` |
| Create/edit versions | `tagmanager.edit.containerversions` | `https://www.googleapis.com/auth/tagmanager.edit.containerversions` |
| Publish versions | `tagmanager.publish` | `https://www.googleapis.com/auth/tagmanager.publish` |
| Manage users | `tagmanager.manage.users` | `https://www.googleapis.com/auth/tagmanager.manage.users` |
| Manage accounts | `tagmanager.manage.accounts` | `https://www.googleapis.com/auth/tagmanager.manage.accounts` |

### Scope Hierarchy

Scopes are **additive**. You need multiple scopes for complex operations:

- **Read only**: `readonly`
- **Edit tags**: `readonly` + `edit.containers`
- **Publish changes**: `readonly` + `edit.containers` + `publish`
- **Full admin**: All scopes

---

## Container Type Features

| Feature | Web | Android | iOS | AMP | Server |
|---------|-----|---------|-----|-----|--------|
| Tags | ✅ | ✅ | ✅ | ✅ | ✅ |
| Triggers | ✅ | ✅ | ✅ | ✅ | ✅ |
| Variables | ✅ | ✅ | ✅ | ✅ | ✅ |
| Built-in Variables | ✅ | ✅ | ✅ | ✅ | ✅ |
| Folders | ✅ | ✅ | ✅ | ✅ | ✅ |
| Workspaces | ✅ | ✅ | ✅ | ✅ | ✅ |
| Environments | ✅ | ✅ | ✅ | ✅ | ✅ |
| Versions | ✅ | ✅ | ✅ | ✅ | ✅ |
| Google Tag Config | ✅ | ❌ | ❌ | ❌ | ❌ |
| Clients | ❌ | ❌ | ❌ | ❌ | ✅ |
| Transformations | ❌ | ❌ | ❌ | ❌ | ✅ |
| Zones | ✅ | ❌ | ❌ | ❌ | ❌ |
| Templates | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## HTTP Methods by Operation

| Operation | HTTP Method | Body Required | Fingerprint Required |
|-----------|-------------|---------------|---------------------|
| Create | `POST` | ✅ Yes | ❌ No |
| Read (get) | `GET` | ❌ No | ❌ No |
| Read (list) | `GET` | ❌ No | ❌ No |
| Update | `PUT` | ✅ Yes | ✅ Yes |
| Delete | `DELETE` | ❌ No | ❌ No |
| Revert | `POST` | ❌ No | ❌ No |
| Publish | `POST` | ❌ No | ❌ No |

---

## Tag Types Reference

| Type Code | Display Name | Category | Container Type |
|-----------|--------------|----------|----------------|
| `html` | Custom HTML | Custom | Web, AMP |
| `img` | Custom Image | Custom | Web, AMP |
| `gaawe` | Google Analytics 4 Config | Google | Web, Server |
| `googtag` | Google Tag | Google | Web |
| `sp` | Google Ads Remarketing | Google Ads | Web |
| `awct` | Google Ads Conversion | Google Ads | Web |
| `gclidw` | Conversion Linker | Google Ads | Web |
| `flc` | Floodlight Counter | Floodlight | Web |
| `fls` | Floodlight Sales | Floodlight | Web |
| `ua` | Universal Analytics (deprecated) | Google | Web |
| `gaaw` | GA4 Event | Google | Server |

---

## Trigger Types Reference

### Web Container Triggers

| Type Code | Display Name | When It Fires |
|-----------|--------------|---------------|
| `pageview` | Page View | Page loads |
| `domReady` | DOM Ready | DOM is ready |
| `windowLoaded` | Window Loaded | Window fully loaded |
| `customEvent` | Custom Event | Custom event pushed to dataLayer |
| `linkClick` | Link Click | User clicks a link |
| `linkClickAll` | All Elements | User clicks any element |
| `formSubmission` | Form Submission | Form is submitted |
| `timer` | Timer | At specified interval |
| `scrollDepth` | Scroll Depth | Page scrolled to threshold |
| `elementVisibility` | Element Visibility | Element becomes visible |
| `youTubeVideo` | YouTube Video | YouTube video event |
| `historyChange` | History Change | Browser history changes |
| `javascriptError` | JavaScript Error | JS error occurs |
| `triggerGroup` | Trigger Group | Combination of triggers |

**Note on Trigger Groups**: Creating trigger groups requires a specific `parameter` structure. See [schemas.md](schemas.md#trigger-groups-type-triggergroup) for the correct format:
```json
{"parameter": [{"key": "triggerIds", "type": "list", "list": [{"type": "triggerReference", "value": "<triggerId>"}]}]}
```

### Server Container Triggers

| Type Code | Display Name | When It Fires |
|-----------|--------------|---------------|
| `serverPageview` | Pageview | Server receives pageview |
| `serverCustomEvent` | Custom Event | Server receives custom event |

---

## Variable Types Reference

### Web Container Variables

| Type Code | Display Name | Returns |
|-----------|--------------|---------|
| `c` | Constant | Static value |
| `jsm` | Custom JavaScript | JS function result |
| `v` | Data Layer Variable | dataLayer value |
| `k` | 1st Party Cookie | Cookie value |
| `u` | URL | URL component |
| `f` | Referrer | Referrer URL |
| `aev` | Auto-Event Variable | Event data (click, form) |
| `r` | Random Number | Random number |
| `smm` | RegEx Table | Lookup table value |
| `e` | Custom Event | Event name |
| `gas` | GA Settings | GA configuration |
| `ctv` | Container Version | Version number |
| `dbg` | Debug Mode | Debug status |
| `d` | DOM Element | Element property |
| `vis` | Visibility | Element visibility % |

### Server Container Variables

| Type Code | Display Name | Returns |
|-----------|--------------|---------|
| `gev` | Event Data | Event parameter value |
| `remoteConfig` | Remote Config | Remote config value |

---

## Transformation Types Reference (Server-Side Only)

| Type Code | Display Name | Purpose | Table Key | Column(s) |
|-----------|-------------|---------|-----------|-----------|
| `tf_allow_params` | Allow Parameters | Only allow specified params | `allowedParamsTable` | `allowedParams` |
| `tf_exclude_params` | Exclude Parameters | Remove specified params | `excludedParamsTable` | `excludedParams` |
| `tf_augment_event` | Augment Event | Add/modify event params | `augmentEventTable` | `paramName`, `paramValue` |

**Important:** These types are undocumented in official Google API docs. Google returns HTTP 500 (not 400) for unknown types.

### Common Transformation Parameters (All Types)

| Parameter Key | Type | Description |
|---------------|------|-------------|
| `matchingConditionsEnabled` | boolean | Whether conditions must match |
| `allTagsExcept` | boolean | Apply to all tags except listed |
| `affectedTags` | list | Specific tags to target |
| `affectedTagTypes` | list | Tag types to target |
| `matchingConditionsTable` | list | Matching conditions |

## Client Types Reference (Server-Side Only)

| Type Code | Display Name | Description |
|-----------|-------------|-------------|
| `gaaw` | GA4 Web Client | Receives GA4 web measurement data |
| `gaaw_client` | GA4 Client | GA4 client (template-based) |
| `sp` | Google Ads Client | Receives Google Ads data |

**Note:** Client types are template-based strings, not a fixed enum. The `type` field references the template used.

---

## Built-in Variables by Container Type

### Web Container

| Category | Variables |
|----------|-----------|
| **Pages** | Page URL, Page Hostname, Page Path, Referrer |
| **Clicks** | Click Element, Click Classes, Click ID, Click Target, Click URL, Click Text |
| **Forms** | Form Element, Form Classes, Form ID, Form Target, Form URL, Form Text |
| **Errors** | Error Message, Error URL, Error Line |
| **History** | New History State, Old History State, History Source |
| **Video** | Video Provider, Video Status, Video URL, Video Title, Video Duration, Video Percent, Video Visible |
| **Scroll** | Scroll Depth Threshold, Scroll Depth Units, Scroll Direction |
| **Utility** | Random Number, Container ID, Container Version, Debug Mode |

### Server Container

| Category | Variables |
|----------|-----------|
| **Event** | Event Name |
| **Client** | Client Name |
| **Request** | Request Path, Request Method, Request Host, IP Address, User Agent |

---

## Error Codes

| HTTP Code | Error Type | Cause | Solution |
|-----------|------------|-------|----------|
| `400` | Bad Request | Malformed request | Check JSON syntax |
| `401` | Unauthorized | Missing/invalid token | Refresh OAuth token |
| `403` | Forbidden | Quota exceeded or insufficient permissions | Check quota, verify scopes |
| `404` | Not Found | Resource doesn't exist | Verify resource path/ID |
| `409` | Conflict | Fingerprint mismatch | Get latest fingerprint, retry |
| `429` | Too Many Requests | Rate limit exceeded | Implement exponential backoff |
| `500` | Internal Error | Server error | Retry with backoff |
| `503` | Service Unavailable | Temporary outage | Retry with backoff |

---

## Path Format Patterns

| Resource | Path Format |
|----------|-------------|
| Account | `accounts/{accountId}` |
| Container | `accounts/{accountId}/containers/{containerId}` |
| Workspace | `accounts/{accountId}/containers/{containerId}/workspaces/{workspaceId}` |
| Tag | `accounts/{accountId}/containers/{containerId}/workspaces/{workspaceId}/tags/{tagId}` |
| Trigger | `accounts/{accountId}/containers/{containerId}/workspaces/{workspaceId}/triggers/{triggerId}` |
| Variable | `accounts/{accountId}/containers/{containerId}/workspaces/{workspaceId}/variables/{variableId}` |
| Version | `accounts/{accountId}/containers/{containerId}/versions/{containerVersionId}` |
| Environment | `accounts/{accountId}/containers/{containerId}/environments/{environmentId}` |
| User Permission | `accounts/{accountId}/permissions/{permissionId}` |

### Special Paths

| Path | Description |
|------|-------------|
| `/versions/live` | Currently published version |
| `/versions/latest` | Latest created version |
| `/workspaces/{id}:create_version` | Create version from workspace |
| `/workspaces/{id}:sync` | Sync workspace with container |
| `/versions/{id}:publish` | Publish version |

---

## Entity Relationships

```
Account
└── Container
    ├── Environments
    │   └── (points to Version)
    ├── Versions (immutable)
    │   ├── Tags
    │   ├── Triggers
    │   ├── Variables
    │   ├── Folders
    │   └── ...
    └── Workspaces (mutable)
        ├── Tags
        │   ├── references → Triggers (firing)
        │   ├── references → Triggers (blocking)
        │   └── references → Variables
        ├── Triggers
        │   └── references → Variables (in filters)
        ├── Variables
        │   ├── references → Triggers (enabling)
        │   └── references → Triggers (disabling)
        ├── Folders
        │   └── contains → Tags, Triggers, Variables
        ├── Clients (server only)
        ├── Transformations (server only)
        ├── Zones (web only)
        └── Templates
```

---

## Publishing Workflow States

| State | Description | Can Edit | Can Publish |
|-------|-------------|----------|-------------|
| **Draft** | Workspace with unpublished changes | ✅ Yes | ✅ Yes |
| **Up to date** | Workspace matches published version | ✅ Yes | ❌ No changes |
| **Conflicted** | Base version changed, conflicts exist | ❌ Must resolve | ❌ Must resolve |
| **Synced** | Workspace updated to latest version | ✅ Yes | ✅ If has changes |
| **Version Created** | Version created but not published | ❌ Immutable | ✅ Yes |
| **Published** | Version is live | ❌ Immutable | ❌ Already live |

---

## Quota Limits

| Limit Type | Default Value | Can Increase |
|------------|---------------|--------------|
| Requests per day | 10,000 | ✅ Via quota request |
| Queries per second (QPS) | 0.25 (1 per 4 seconds) | ✅ Via quota request |
| Containers per account | 200 | ❌ Hard limit |
| Workspaces per container | 3 (Free), Unlimited (360) | ✅ Upgrade to 360 |
| Tags per container | ~500 (recommended) | ⚠️ Performance degrades |
| Environments per container | 15 | ❌ Hard limit |
| Versions per container | Unlimited | - |

---

## API Versions Comparison

| Feature | v1 | v2 | Recommendation |
|---------|----|----|----------------|
| Accounts | ✅ | ✅ | Use v2 |
| Containers | ✅ | ✅ | Use v2 |
| Workspaces | ❌ | ✅ | Use v2 |
| Tags | ✅ | ✅ | Use v2 |
| Triggers | ✅ | ✅ | Use v2 |
| Variables | ✅ | ✅ | Use v2 |
| Versions | ✅ | ✅ | Use v2 |
| Environments | ✅ | ✅ | Use v2 |
| Clients | ❌ | ✅ | Use v2 |
| Transformations | ❌ | ✅ | Use v2 |
| Zones | ❌ | ✅ | Use v2 |
| Templates | ❌ | ✅ | Use v2 |
| **Status** | Legacy | Current | **Always use v2** |

---

## Common Parameter Keys

### Tag Parameters

| Key | Type | Used In | Description |
|-----|------|---------|-------------|
| `html` | template | Custom HTML | HTML code to execute |
| `measurementId` | template | GA4 | GA4 Measurement ID |
| `tagId` | template | Google Tag | Tag ID (e.g., GT-XXXXX) |
| `conversionId` | template | Google Ads | Conversion ID |
| `conversionLabel` | template | Google Ads | Conversion label |
| `url` | template | Image tag | Image URL |

### Trigger Parameters

| Key | Type | Used In | Description |
|-----|------|---------|-------------|
| `arg0` | template | Filters | First comparison value |
| `arg1` | template | Filters | Second comparison value |
| `interval` | integer | Timer | Interval in milliseconds |
| `limit` | integer | Timer | Max number of fires |
| `eventName` | template | Custom Event | Event name to match |

### Variable Parameters

| Key | Type | Used In | Description |
|-----|------|---------|-------------|
| `name` | template | Data Layer Variable | dataLayer key |
| `defaultValue` | template | Data Layer Variable | Fallback value |
| `trackingId` | template | GA Settings | GA tracking ID |
| `component` | template | URL Variable | URL component (e.g., HOST, PATH) |

---

## Field Type Reference

| Type | Example Value | Notes |
|------|---------------|-------|
| `string` | `"My Tag"` | UTF-8 text |
| `integer` | `42` | Whole number |
| `boolean` | `true` or `false` | Boolean value |
| `enum` | `"web"` | Predefined set of values |
| `object` | `{"key": "value"}` | Nested structure |
| `array` | `["item1", "item2"]` | List of items |

---

## Tips for Quick Lookups

1. **Finding resource paths**: Use list operation → extract `path` field
2. **Getting fingerprints**: Always GET before PUT
3. **Checking permissions**: Review `features` object in container/account
4. **Validating scopes**: Match operation to table above
5. **Resolving conflicts**: GET workspace status → resolve_conflict
6. **Testing changes**: Use quick_preview before publish
7. **Rollback**: Publish previous version by ID
8. **Batch operations**: No native batch API - must loop sequentially
9. **Searching entities**: No search API - must list all and filter
10. **Rate limiting**: Max 1 request per 4 seconds (0.25 QPS)
