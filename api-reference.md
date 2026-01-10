# GTM API Reference

Complete API reference for Google Tag Manager v2 API.

---

## Table of Contents

- [API Overview](#api-overview)
- [Authentication](#authentication)
- [Endpoints](#endpoints)
- [Rate Limits](#rate-limits)
- [Error Handling](#error-handling)

---

## API Overview

The **Google Tag Manager API** provides programmatic access to GTM configuration.

- **Service Endpoint**: `https://tagmanager.googleapis.com`
- **Current Version**: `v2` (recommended)
- **Legacy Version**: `v1` (still supported)
- **Protocol**: REST over HTTPS
- **Data Format**: JSON

### Discovery Documents

Machine-readable API specifications:

- v2: `https://tagmanager.googleapis.com/$discovery/rest?version=v2`
- v1: `https://tagmanager.googleapis.com/$discovery/rest?version=v1`

---

## Authentication

### Authorization Protocol

The Tag Manager API requires **OAuth 2.0** authentication. No other protocols are supported.

Every request must include an access token in the Authorization header:

```http
Authorization: Bearer {access_token}
```

### OAuth 2.0 Scopes

| Scope | Description | Use Cases |
|-------|-------------|-----------|
| `https://www.googleapis.com/auth/tagmanager.readonly` | Read-only access | View containers, tags, triggers, variables |
| `https://www.googleapis.com/auth/tagmanager.edit.containers` | Edit containers | Create/modify tags, triggers, variables, workspaces |
| `https://www.googleapis.com/auth/tagmanager.delete.containers` | Delete containers | Remove containers |
| `https://www.googleapis.com/auth/tagmanager.edit.containerversions` | Manage versions | Create container versions |
| `https://www.googleapis.com/auth/tagmanager.publish` | Publish containers | Publish versions to production |
| `https://www.googleapis.com/auth/tagmanager.manage.users` | Manage permissions | Add/remove users, modify permissions |
| `https://www.googleapis.com/auth/tagmanager.manage.accounts` | Manage accounts | Update account settings |

### Scope Best Practices

1. **Request minimum scopes** - Only request scopes your application needs
2. **Incremental authorization** - Request additional scopes as needed
3. **Scopes are additive** - Multiple scopes can be combined
4. **Publish requires edit** - Publishing requires both `edit.containers` and `publish` scopes

---

## Endpoints

All endpoints are relative to: `https://tagmanager.googleapis.com/tagmanager/v2`

### Accounts

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/accounts/{accountId}` | Get account details |
| `GET` | `/accounts` | List all accessible accounts |
| `PUT` | `/accounts/{accountId}` | Update account |

### Containers

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/accounts/{accountId}/containers` | Create container |
| `GET` | `/accounts/{accountId}/containers/{containerId}` | Get container |
| `GET` | `/accounts/{accountId}/containers` | List containers |
| `PUT` | `/accounts/{accountId}/containers/{containerId}` | Update container |
| `DELETE` | `/accounts/{accountId}/containers/{containerId}` | Delete container |
| `POST` | `/accounts/{accountId}/containers/{containerId}:combine` | Combine containers |
| `GET` | `/accounts/containers:lookup` | Lookup container by destination/tag ID |
| `POST` | `/accounts/{accountId}/containers/{containerId}:move_tag_id` | Move tag ID |
| `GET` | `/accounts/{accountId}/containers/{containerId}:snippet` | Get container snippet |

### Workspaces

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/workspaces` | Create workspace |
| `GET` | `/{parent}/workspaces/{workspaceId}` | Get workspace |
| `GET` | `/{parent}/workspaces` | List workspaces |
| `PUT` | `/{parent}/workspaces/{workspaceId}` | Update workspace |
| `DELETE` | `/{parent}/workspaces/{workspaceId}` | Delete workspace |
| `POST` | `/{parent}/workspaces/{workspaceId}:sync` | Sync workspace |
| `POST` | `/{parent}/workspaces/{workspaceId}:resolve_conflict` | Resolve conflicts |
| `POST` | `/{parent}/workspaces/{workspaceId}:create_version` | Create version from workspace |
| `POST` | `/{parent}/workspaces/{workspaceId}:quick_preview` | Generate preview |
| `GET` | `/{parent}/workspaces/{workspaceId}/status` | Get workspace status |

**Note**: `{parent}` = `accounts/{accountId}/containers/{containerId}`

### Tags

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/tags` | Create tag |
| `GET` | `/{parent}/tags/{tagId}` | Get tag |
| `GET` | `/{parent}/tags` | List tags |
| `PUT` | `/{parent}/tags/{tagId}` | Update tag |
| `DELETE` | `/{parent}/tags/{tagId}` | Delete tag |
| `POST` | `/{parent}/tags/{tagId}:revert` | Revert tag |

**Note**: `{parent}` = `accounts/{accountId}/containers/{containerId}/workspaces/{workspaceId}`

### Triggers

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/triggers` | Create trigger |
| `GET` | `/{parent}/triggers/{triggerId}` | Get trigger |
| `GET` | `/{parent}/triggers` | List triggers |
| `PUT` | `/{parent}/triggers/{triggerId}` | Update trigger |
| `DELETE` | `/{parent}/triggers/{triggerId}` | Delete trigger |
| `POST` | `/{parent}/triggers/{triggerId}:revert` | Revert trigger |

### Variables

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/variables` | Create variable |
| `GET` | `/{parent}/variables/{variableId}` | Get variable |
| `GET` | `/{parent}/variables` | List variables |
| `PUT` | `/{parent}/variables/{variableId}` | Update variable |
| `DELETE` | `/{parent}/variables/{variableId}` | Delete variable |
| `POST` | `/{parent}/variables/{variableId}:revert` | Revert variable |

### Built-in Variables

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/built_in_variables` | Enable built-in variables |
| `DELETE` | `/{parent}/built_in_variables` | Disable built-in variables |
| `GET` | `/{parent}/built_in_variables` | List built-in variables |
| `POST` | `/{parent}/built_in_variables:revert` | Revert built-in variables |

### Folders

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/folders` | Create folder |
| `GET` | `/{parent}/folders/{folderId}` | Get folder |
| `GET` | `/{parent}/folders` | List folders |
| `PUT` | `/{parent}/folders/{folderId}` | Update folder |
| `DELETE` | `/{parent}/folders/{folderId}` | Delete folder |
| `POST` | `/{parent}/folders/{folderId}:revert` | Revert folder |
| `POST` | `/{parent}/folders/{folderId}:move_entities_to_folder` | Move entities |

### Container Versions

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/{parent}/versions/{versionId}` | Get version |
| `GET` | `/{parent}/versions` | List versions |
| `PUT` | `/{parent}/versions/{versionId}` | Update version |
| `DELETE` | `/{parent}/versions/{versionId}` | Delete version |
| `POST` | `/{parent}/versions/{versionId}:publish` | Publish version |
| `POST` | `/{parent}/versions/{versionId}:undelete` | Undelete version |
| `POST` | `/{parent}/versions/{versionId}:set_latest` | Set as latest |
| `GET` | `/{parent}/versions/live` | Get live version |
| `GET` | `/{parent}/versions/latest` | Get latest version |

**Note**: `{parent}` = `accounts/{accountId}/containers/{containerId}`

### Version Headers

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/{parent}/version_headers` | List version headers |
| `GET` | `/{parent}/version_headers/latest` | Get latest version header |

### Environments

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/environments` | Create environment |
| `GET` | `/{parent}/environments/{environmentId}` | Get environment |
| `GET` | `/{parent}/environments` | List environments |
| `PUT` | `/{parent}/environments/{environmentId}` | Update environment |
| `DELETE` | `/{parent}/environments/{environmentId}` | Delete environment |
| `POST` | `/{parent}/environments/{environmentId}:reauthorize` | Reauthorize environment |

### Clients (Server-side only)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/clients` | Create client |
| `GET` | `/{parent}/clients/{clientId}` | Get client |
| `GET` | `/{parent}/clients` | List clients |
| `PUT` | `/{parent}/clients/{clientId}` | Update client |
| `DELETE` | `/{parent}/clients/{clientId}` | Delete client |
| `POST` | `/{parent}/clients/{clientId}:revert` | Revert client |

### Transformations (Server-side only)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/transformations` | Create transformation |
| `GET` | `/{parent}/transformations/{transformationId}` | Get transformation |
| `GET` | `/{parent}/transformations` | List transformations |
| `PUT` | `/{parent}/transformations/{transformationId}` | Update transformation |
| `DELETE` | `/{parent}/transformations/{transformationId}` | Delete transformation |
| `POST` | `/{parent}/transformations/{transformationId}:revert` | Revert transformation |

### Zones (Web only)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/zones` | Create zone |
| `GET` | `/{parent}/zones/{zoneId}` | Get zone |
| `GET` | `/{parent}/zones` | List zones |
| `PUT` | `/{parent}/zones/{zoneId}` | Update zone |
| `DELETE` | `/{parent}/zones/{zoneId}` | Delete zone |
| `POST` | `/{parent}/zones/{zoneId}:revert` | Revert zone |

### Templates

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/templates` | Create template |
| `GET` | `/{parent}/templates/{templateId}` | Get template |
| `GET` | `/{parent}/templates` | List templates |
| `PUT` | `/{parent}/templates/{templateId}` | Update template |
| `DELETE` | `/{parent}/templates/{templateId}` | Delete template |
| `POST` | `/{parent}/templates/{templateId}:revert` | Revert template |

### Google Tag Config (Web only)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/{parent}/gtag_config` | Create Google Tag config |
| `GET` | `/{parent}/gtag_config/{gtagConfigId}` | Get config |
| `GET` | `/{parent}/gtag_config` | List configs |
| `PUT` | `/{parent}/gtag_config/{gtagConfigId}` | Update config |
| `DELETE` | `/{parent}/gtag_config/{gtagConfigId}` | Delete config |

### User Permissions

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/accounts/{accountId}/user_permissions` | Create permission |
| `GET` | `/accounts/{accountId}/user_permissions/{permissionId}` | Get permission |
| `GET` | `/accounts/{accountId}/user_permissions` | List permissions |
| `PUT` | `/accounts/{accountId}/user_permissions/{permissionId}` | Update permission |
| `DELETE` | `/accounts/{accountId}/user_permissions/{permissionId}` | Delete permission |

---

## Rate Limits

### Default Quotas

| Limit Type | Value | Notes |
|------------|-------|-------|
| Requests per day | 10,000 | Per project |
| Queries per second (QPS) | 0.25 | 1 request per 4 seconds |

### Important Notes

- Quotas apply **per Google Cloud project**, not per user
- GTM enforces these limits even if higher quotas are set in API Console
- Exceeding quota returns `HTTP 403` error
- Request quota increases via Google Cloud Console

### Quota Error Response

```json
{
  "error": {
    "errors": [{
      "domain": "usageLimits",
      "reason": "rateLimitExceeded",
      "message": "Rate Limit Exceeded"
    }],
    "code": 403,
    "message": "Rate Limit Exceeded"
  }
}
```

### Best Practices

1. **Implement exponential backoff** for retries
2. **Batch operations** where possible (though no native batch API exists)
3. **Cache responses** when data doesn't change frequently
4. **Monitor quota usage** in Google Cloud Console
5. **Request quota increases** proactively for production use

---

## Error Handling

### HTTP Status Codes

| Code | Status | Meaning | Action |
|------|--------|---------|--------|
| `200` | OK | Success | Process response |
| `400` | Bad Request | Invalid request | Fix request format |
| `401` | Unauthorized | Invalid/missing token | Refresh OAuth token |
| `403` | Forbidden | Insufficient permissions or quota exceeded | Check scopes or quota |
| `404` | Not Found | Resource doesn't exist | Verify resource ID |
| `409` | Conflict | Fingerprint mismatch | Get latest resource, retry |
| `429` | Too Many Requests | Rate limit exceeded | Implement backoff |
| `500` | Internal Server Error | Server error | Retry with backoff |
| `503` | Service Unavailable | Temporary outage | Retry with backoff |

### Error Response Format

```json
{
  "error": {
    "errors": [
      {
        "domain": string,
        "reason": string,
        "message": string,
        "locationType": string,
        "location": string
      }
    ],
    "code": integer,
    "message": string
  }
}
```

### Common Error Reasons

| Reason | Domain | Description |
|--------|--------|-------------|
| `accessNotConfigured` | `usageLimits` | API not enabled in Cloud Console |
| `rateLimitExceeded` | `usageLimits` | QPS quota exceeded |
| `quotaExceeded` | `usageLimits` | Daily quota exceeded |
| `authError` | `global` | Authentication failure |
| `insufficientPermissions` | `global` | Missing required scope |
| `invalidParameter` | `global` | Invalid request parameter |
| `notFound` | `global` | Resource not found |

### Exponential Backoff Algorithm

Recommended retry logic for rate limit errors:

```
wait_time = min(2^retry_count, 32) seconds
max_retries = 5
```

Example sequence:
1. First retry: 1 second
2. Second retry: 2 seconds
3. Third retry: 4 seconds
4. Fourth retry: 8 seconds
5. Fifth retry: 16 seconds

---

## Request/Response Format

### Content Type

All requests and responses use JSON:

```http
Content-Type: application/json
```

### Common Headers

**Request:**
```http
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Response:**
```http
Content-Type: application/json; charset=UTF-8
```

### Pagination

List endpoints support pagination via query parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `pageToken` | string | Token for next page |

**Example:**
```http
GET /tagmanager/v2/accounts/123/containers/456/workspaces/789/tags?pageToken=abc123
```

**Response includes:**
```json
{
  "tag": [...],
  "nextPageToken": "def456"
}
```

### Field Masks

Update operations support partial updates via field masks:

```http
PUT /tagmanager/v2/{path}?updateMask=name,notes
```

---

## Client Libraries

Google provides official client libraries:

| Language | Package | Status |
|----------|---------|--------|
| Java | `google-api-services-tagmanager` | Stable |
| JavaScript | `googleapis` npm package | Stable |
| .NET | `Google.Apis.TagManager` | Stable |
| PHP | `google/apiclient` | Stable |
| Python | `google-api-python-client` | Stable |
| Go | `google.golang.org/api/tagmanager` | Early stage |
| Node.js | `googleapis` npm package | Early stage |
| Ruby | `google-api-client` | Early stage |

---

## Notes for Developers

1. **Always use v2** - v1 is legacy, use v2 for all new development
2. **Fingerprints are critical** - Always include current fingerprint when updating
3. **Workspace isolation** - Changes in workspaces don't affect live until published
4. **No undo for publish** - Can only roll forward or back to previous version
5. **Rate limits are strict** - 0.25 QPS is enforced, plan accordingly
6. **IDs are strings** - Even if they look numeric, always treat as strings
7. **References use IDs** - Entity references use IDs, not full paths
8. **Server containers differ** - Some resources only exist in specific container types
