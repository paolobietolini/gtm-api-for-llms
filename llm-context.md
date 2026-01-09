# LLM Context — Google Tag Manager API

This document provides a **compact, explicit, and machine-oriented context** for reasoning about the **Google Tag Manager (GTM) API**.

It is intended to be injected directly into a Large Language Model context window.

---

## Domain Overview

Google Tag Manager (GTM) is a configuration-based system for managing tracking, tags, and client-side / server-side integrations.

The GTM API allows **programmatic access** to:
- Accounts
- Containers
- Workspaces
- Tags, Triggers, Variables
- Versions and Environments
- Permissions and Users

The API is **stateful**, **hierarchical**, and **versioned**.

---

## API Structure (Mental Model)

Hierarchy (top → bottom):

```

Account
└─ Container
├─ Destinations
├─ Environments
├─ Versions
└─ Workspaces
├─ Tags
├─ Triggers
├─ Variables
├─ Built-In Variables
├─ Clients
├─ Templates
├─ Transformations
├─ Zones
└─ Folders

```

Key rule:
> **All mutable entities live inside a Workspace.**  
> Publishing happens at the **Container Version** level.

---

## API Versions

- **v2** → current, recommended
- **v1** → legacy, still supported

Both expose similar resources but with different endpoint shapes.

---

## Authentication & Authorization

- OAuth 2.0 **only**
- Every request requires an **access token**
- Scopes are **granular and additive**

Common scopes:
- `tagmanager.readonly`
- `tagmanager.edit.containers`
- `tagmanager.publish`
- `tagmanager.manage.users`
- `tagmanager.manage.accounts`

---

## Core Concepts

### Account
Top-level entity. Owns containers and users.

### Container
Logical unit deployed on a site or server.
- Has environments
- Has versions
- Has workspaces

### Workspace
Editable sandbox.
- All changes happen here
- Can diverge from live version
- Can be synced, merged, or conflicted

### Container Version
Immutable snapshot of a container.
- Created from a workspace
- Can be published
- One version is “live”

### Environment
Deployment target (e.g. live, staging).
- Linked to container versions
- Has authorization codes/snippets

---

## Entity Lifecycle (Simplified)

1. Create / select Workspace
2. Create or update entities (tags, triggers, etc.)
3. Resolve conflicts (if any)
4. Create Container Version
5. Publish Container Version
6. Version becomes Live

---

## Common Operations Pattern

Most GTM entities support:

- `create`
- `get`
- `list`
- `update`
- `delete`
- `revert` (workspace-only)

Revert restores the entity to the base container version.

---

## Error Handling

- Success → HTTP 200
- Errors → HTTP 4xx / 5xx
- Quota errors → HTTP 403

The API **expects exponential backoff** for retries.

---

## Quotas (Default)

- 10,000 requests / project / day
- 0.25 QPS / project

Hard limit enforced by GTM, regardless of API Console settings.

---

## Deterministic Rules (Important for LLMs)

- You cannot modify live containers directly
- Publishing always creates or uses a container version
- Deleting entities in a workspace does not affect live until publish
- Workspaces may conflict if base version changes
- v1 and v2 endpoints are **not interchangeable**

---

## What This Context Is For

This context enables an LLM to:

- Reason about GTM architecture
- Generate valid API call sequences
- Understand dependencies between entities
- Avoid illegal state transitions
- Explain GTM workflows deterministically

---

## What This Context Is NOT

- Not a replacement for official Google docs
- Not guaranteed to reflect future API changes
- Not a runtime schema or OpenAPI spec

---

## Usage Recommendation

Inject this document:
- As **system or developer context**
- Before asking for GTM API workflows
- Before generating GTM automation code
- Before validating GTM configurations

---

End of context.

