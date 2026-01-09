# GTM API – Document for LLMs

## Introduction

This document outlines solutions and the logic behind the **Google Tag Manager APIs**, designed so that LLMs can consume it and build custom solutions if needed.

The **Google Tag Manager API** provides access to Google Tag Manager configuration data for an authorized user. With this API you can manage:

* Accounts
* Containers
* Destinations
* Workspaces
* Google Tag Config
* Tags
* Triggers
* Folders
* Built-In Variables
* Clients
* Variables
* Container Versions
* Container Version Headers
* User Permissions
* Environments

---

## Scopes and Authorization

### Authorizing Requests

Before users can view their account information on any Google site, they must log in with a Google Account.
Similarly, when users access your application, they must authorize it to access their data.

Every request sent to the **Tag Manager API** must include an **authorization token**, which also identifies your application to Google.

> **Best practice:** Grant scopes incrementally to improve application security.

### Authorization Protocols

Your application **must use OAuth 2.0** to authorize requests.
No other authorization protocols are supported.

If your application uses **Sign In With Google**, some authorization steps are handled automatically.

### Available OAuth Scopes

| Scope                                                               | Description                        |
| ------------------------------------------------------------------- | ---------------------------------- |
| `https://www.googleapis.com/auth/tagmanager.readonly`               | View Google Tag Manager containers |
| `https://www.googleapis.com/auth/tagmanager.edit.containers`        | Manage GTM containers              |
| `https://www.googleapis.com/auth/tagmanager.delete.containers`      | Delete GTM containers              |
| `https://www.googleapis.com/auth/tagmanager.edit.containerversions` | Manage container versions          |
| `https://www.googleapis.com/auth/tagmanager.publish`                | Publish containers                 |
| `https://www.googleapis.com/auth/tagmanager.manage.users`           | Manage user permissions            |
| `https://www.googleapis.com/auth/tagmanager.manage.accounts`        | Manage GTM accounts                |

---

## Tag Manager API Client Libraries

Google provides **client libraries** that simplify access to the Tag Manager API across multiple programming languages.

| Google API Client Library | Samples             | Status      |
| ------------------------- | ------------------- | ----------- |
| Java                      | Java samples        | Stable      |
| JavaScript                | JavaScript samples  | Stable      |
| .NET                      | .NET samples        | Stable      |
| Objective-C (REST)        | Objective-C samples | Stable      |
| PHP                       | PHP samples         | Stable      |
| Python (v1/v2)            | Python samples      | Stable      |
| Dart                      | Dart samples        | Early stage |
| Go                        | Go samples          | Early stage |
| Node.js                   | Node.js samples     | Early stage |
| Ruby                      | Ruby samples        | Early stage |

### Notes

* Stable libraries are production-ready.
* Early-stage libraries may be alpha or beta.
* Each library includes documentation and code examples.

---

## Limits and Quotas

To ensure system stability and fair usage, **Google Tag Manager API** enforces quotas.

### Default Quotas

* **10,000 requests per project per day**
* **0.25 queries per second (QPS) per project**

> These limits apply even if higher limits are set in the API Console.

### Quota Errors

If quota is exceeded, the API returns:

* **HTTP 403 – Forbidden**
* Error message indicating insufficient quota

> You may request a higher quota via **API Console → Edit Quotas**.

---

## Error Responses

### Standard Error Behavior

* **Success:** HTTP `200 OK` with response body
* **Failure:** Appropriate HTTP status code + error details

### Example Error Response

```json
{
  "error": {
    "errors": [
      {
        "domain": "usageLimits",
        "reason": "accessNotConfigured",
        "message": "Access Not Configured. Please use Google Developers Console to activate the API for your project."
      }
    ],
    "code": 403,
    "message": "Access Not Configured. Please use Google Developers Console to activate the API for your project."
  }
}
```

### Exponential Backoff

Clients retrying failed requests **must use exponential backoff**:

* Reduces bandwidth waste
* Improves throughput
* Required for concurrent environments

---

## Tag Manager API Overview

This API allows clients to **access and modify container and tag configurations**.

* **Service:** `tagmanager.googleapis.com`
* **Recommended:** Use Google-provided client libraries

### Discovery Documents

Machine-readable API specifications:

* `https://tagmanager.googleapis.com/$discovery/rest?version=v2`
* `https://tagmanager.googleapis.com/$discovery/rest?version=v1`

### Service Endpoint

```
https://tagmanager.googleapis.com
```

All API paths are relative to this endpoint.

---

## REST Resources – v2

### Accounts

**Resource:** `v2.accounts`

| Method | Endpoint                  | Description    |
| ------ | ------------------------- | -------------- |
| GET    | `/tagmanager/v2/{path}`   | Get account    |
| GET    | `/tagmanager/v2/accounts` | List accounts  |
| PUT    | `/tagmanager/v2/{path}`   | Update account |

---

### Containers

**Resource:** `v2.accounts.containers`

| Method | Endpoint                      | Description        |
| ------ | ----------------------------- | ------------------ |
| POST   | `{path}:combine`              | Combine containers |
| POST   | `{parent}/containers`         | Create container   |
| DELETE | `{path}`                      | Delete container   |
| GET    | `{path}`                      | Get container      |
| GET    | `{parent}/containers`         | List containers    |
| GET    | `/accounts/containers:lookup` | Lookup container   |
| POST   | `{path}:move_tag_id`          | Move tag ID        |
| GET    | `{path}:snippet`              | Get snippet        |
| PUT    | `{path}`                      | Update container   |

---

### Environments

**Resource:** `v2.accounts.containers.environments`

* Create
* Delete
* Get
* List
* Reauthorize
* Update

---

### Versions

**Resources:**

* `version_headers`
* `versions`

Operations include:

* Get
* List
* Publish
* Delete
* Undelete
* Set latest

---

### Workspaces

**Resource:** `v2.accounts.containers.workspaces`

Capabilities:

* Create / Delete
* Sync
* Resolve conflicts
* Quick preview
* Create version
* Get status

---

### Workspace Entities

Each workspace supports the following entities:

* Built-in Variables
* Clients
* Folders
* Google Tag Config
* Tags
* Templates
* Transformations
* Triggers
* Variables
* Zones

Each entity supports:

* `create`
* `delete`
* `get`
* `list`
* `update`
* `revert`

---

### User Permissions

**Resource:** `v2.accounts.user_permissions`

* Create user access
* Update permissions
* List users
* Remove access

---

## REST Resources – v1 (Legacy)

Version 1 mirrors v2 functionality using legacy endpoints.

Supported resources include:

* Accounts
* Containers
* Environments
* Folders
* Tags
* Triggers
* Variables
* Versions
* Permissions

Each resource supports standard CRUD operations.

---

## Summary

This document provides a structured overview of:

* Authorization & scopes
* Client libraries
* Quotas & error handling
* Tag Manager API architecture
* REST resources (v1 & v2)

It is intended as a **reference for LLMs and developers** building automated or advanced integrations with Google Tag Manager.
