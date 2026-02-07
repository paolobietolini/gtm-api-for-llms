# GTM API for LLMs

This repository contains a structured, machine-readable reference of the **Google Tag Manager API**, optimized for consumption by **Large Language Models (LLMs)**.

The goal of this project is to provide a clean, normalized, and comprehensive description of GTM APIs so that AI systems can reason about, generate, and validate GTM-related solutions.

---

## Purpose

- Provide **LLM-friendly documentation** for Google Tag Manager APIs
- Reduce ambiguity found in human-oriented docs
- Enable AI-assisted:
  - GTM container management
  - Tag, trigger, and variable generation
  - Workspace and version workflows
  - Permission and environment handling

---

## For Humans: Install the AI Coding Agent Skill

Want your AI assistant to help with GTM API operations? Install the skill and it will automatically know how to create tags, triggers, variables, publish versions, and more.

**Works with:** Claude Code, OpenAI Codex, and other compatible agents.

### Claude Code Installation

```bash
# Option 1: Clone and copy
git clone https://github.com/paolobietolini/gtm-api-for-llms.git
cp -r gtm-api-for-llms/skills/gtm-api ~/.claude/skills/

# Option 2: One-liner
curl -sL https://github.com/paolobietolini/gtm-api-for-llms/archive/main.tar.gz | tar xz && \
  mkdir -p ~/.claude/skills && \
  cp -r gtm-api-for-llms-main/skills/gtm-api ~/.claude/skills/ && \
  rm -rf gtm-api-for-llms-main
```

Verify: Ask Claude *"What skills are available?"* — you should see `gtm-api`.

### OpenAI Codex Installation

```bash
# Option 1: Clone and copy
git clone https://github.com/paolobietolini/gtm-api-for-llms.git
cp -r gtm-api-for-llms/skills/gtm-api ~/.codex/skills/

# Option 2: One-liner
curl -sL https://github.com/paolobietolini/gtm-api-for-llms/archive/main.tar.gz | tar xz && \
  mkdir -p ~/.codex/skills && \
  cp -r gtm-api-for-llms-main/skills/gtm-api ~/.codex/skills/ && \
  rm -rf gtm-api-for-llms-main
```

Verify: Run `/skills` in Codex — you should see `gtm-api`.

### Usage

Once installed, the skill activates automatically when you ask about GTM API operations:

- *"Create a GA4 pageview tag in my GTM container"*
- *"Help me publish changes to my GTM container"*
- *"What's the correct format for a custom event trigger?"*
- *"Update tag X to fire on a different trigger"*

No special syntax needed — just describe what you want to do with GTM.

---

## Documentation Structure

### For LLMs (Primary Audience)

#### Core Execution Guides

1. **[llm-instructions.md](llm-instructions.md)** - **START HERE for execution**
   - Explicit step-by-step algorithms for common operations
   - Error handling procedures
   - Validation checklists
   - Best for: Task execution, operation sequencing

2. **[llm-workflows.md](llm-workflows.md)** - Decision trees and state machines
   - Complete workflow algorithms with branching logic
   - State machine definitions for entities
   - Path construction algorithms
   - Best for: Multi-step operations, understanding state transitions

3. **[llm-validation-rules.md](llm-validation-rules.md)** - Validation constraints
   - Required vs optional fields by operation
   - Field type validation rules
   - Cross-entity validation requirements
   - Best for: Pre-flight validation, error prevention

4. **[llm-request-templates.md](llm-request-templates.md)** - Copy-paste templates
   - Complete request bodies for all entity types
   - Annotated examples with replacement markers
   - Common mistake examples
   - Best for: Quick request generation, avoiding errors

#### Reference Documentation

5. **[llm-context.md](llm-context.md)** - Mental models and architecture
   - GTM hierarchy and concepts
   - Core principles and constraints
   - Best for: System prompts, understanding architecture

6. **[schemas.md](schemas.md)** - Complete resource schemas
   - JSON representations for all GTM entities
   - Field descriptions and types
   - Entity-specific enumerations
   - Best for: Understanding data structures

7. **[api-reference.md](api-reference.md)** - Complete API reference
   - All v2 API endpoints
   - Authentication and OAuth scopes
   - Rate limits and quotas
   - Best for: Endpoint lookup, auth requirements

8. **[examples.md](examples.md)** - Real-world workflows
   - Complete request/response examples
   - Common workflow patterns
   - Error handling scenarios
   - Best for: Learning by example

9. **[quick-reference.md](quick-reference.md)** - Fast lookups
   - Operation matrices
   - Type code tables
   - Scope mappings
   - Best for: Quick fact checking

### For Developers

- **[gtm_api_v2_discovery.json](gtm_api_v2_discovery.json)** - Official discovery document
  - Machine-readable API specification
  - Use for programmatic tooling

---

## Quick Start for LLMs

### First Time Here?

**→ [LLM-QUICK-START.md](LLM-QUICK-START.md)** - Complete getting started guide with step-by-step instructions

### Recommended Reading Order

**For Executing Tasks:**
1. **[llm-instructions.md](llm-instructions.md)** - Get the algorithm for your specific task
2. **[llm-request-templates.md](llm-request-templates.md)** - Copy the request body template
3. **[llm-validation-rules.md](llm-validation-rules.md)** - Validate before sending
4. **[llm-workflows.md](llm-workflows.md)** - Handle complex multi-step flows

**For Understanding Context:**
1. **[llm-context.md](llm-context.md)** - Mental model of GTM architecture
2. **[schemas.md](schemas.md)** - Data structure reference
3. **[api-reference.md](api-reference.md)** - Endpoint and auth reference

### Task → Document Mapping

| Task | Primary Document | Supporting Docs |
|------|------------------|-----------------|
| **Create a tag** | [llm-instructions.md](llm-instructions.md) → "Create a Tag" | [llm-request-templates.md](llm-request-templates.md) → Tag Templates |
| **Update a tag** | [llm-instructions.md](llm-instructions.md) → "Update a Tag" | [llm-validation-rules.md](llm-validation-rules.md) → Update Tag |
| **Publish changes** | [llm-instructions.md](llm-instructions.md) → "Publish Changes" | [llm-workflows.md](llm-workflows.md) → Create and Publish |
| **Handle errors** | [llm-workflows.md](llm-workflows.md) → "Error Recovery" | [llm-instructions.md](llm-instructions.md) → Error Recovery |
| **List resources** | [llm-instructions.md](llm-instructions.md) → "List All Resources" | [llm-workflows.md](llm-workflows.md) → Pagination |
| **Build parameters** | [llm-request-templates.md](llm-request-templates.md) | [llm-validation-rules.md](llm-validation-rules.md) → Parameters |
| **Validate request** | [llm-validation-rules.md](llm-validation-rules.md) | [schemas.md](schemas.md) |
| **Create client** | [llm-instructions.md](llm-instructions.md) → "Create a Client" | [llm-request-templates.md](llm-request-templates.md) → Client Templates |
| **Create transformation** | [llm-instructions.md](llm-instructions.md) → "Create a Transformation" | [llm-request-templates.md](llm-request-templates.md) → Transformation Templates |
| **Manage built-in vars** | [llm-instructions.md](llm-instructions.md) → "Manage Built-in Variables" | [llm-request-templates.md](llm-request-templates.md) → Built-in Variable Templates |
| **Understand hierarchy** | [llm-context.md](llm-context.md) | [quick-reference.md](quick-reference.md) → Entity Relationships |
| **Look up tag types** | [quick-reference.md](quick-reference.md) → Tag Types | [llm-request-templates.md](llm-request-templates.md) |
| **Look up transformation types** | [quick-reference.md](quick-reference.md) → Transformation Types | [schemas.md](schemas.md) → Transformations |
| **Check OAuth scopes** | [llm-workflows.md](llm-workflows.md) → Determine Scopes | [api-reference.md](api-reference.md) |

---

## Key Concepts

### Hierarchy

```
Account
└── Container
    ├── Environments
    ├── Versions (immutable)
    └── Workspaces (mutable)
        ├── Tags
        ├── Triggers
        ├── Variables
        ├── Built-in Variables (enable/disable)
        ├── Clients (server-side only)
        ├── Transformations (server-side only)
        ├── Templates
        └── Folders
```

### Critical Rules

1. **All changes happen in workspaces** - Live containers cannot be modified directly
2. **Publishing creates versions** - Versions are immutable snapshots
3. **Fingerprints prevent conflicts** - Always include fingerprint when updating
4. **Use v2 API** - v1 is legacy, v2 is current
5. **Rate limit: 0.25 QPS** - 1 request per 4 seconds
6. **OAuth 2.0 only** - No other auth methods supported

---

## API Overview

- **Base URL**: `https://tagmanager.googleapis.com/tagmanager/v2`
- **Format**: REST with JSON
- **Auth**: OAuth 2.0
- **Current Version**: v2
- **Rate Limit**: 10,000 requests/day, 0.25 QPS

### Main Resources

| Resource | Description | Mutable in Workspace |
|----------|-------------|----------------------|
| Accounts | Top-level organization | ❌ |
| Containers | Deployment targets | Partially |
| Workspaces | Isolated editing environments | ✅ |
| Tags | Tracking code/pixels | ✅ |
| Triggers | Firing conditions | ✅ |
| Variables | Dynamic values | ✅ |
| Built-in Variables | Pre-configured variables | ✅ (enable/disable) |
| Clients | Server-side request handlers | ✅ (server only) |
| Transformations | Server-side event data modifiers | ✅ (server only) |
| Templates | Custom tag/variable definitions | ✅ |
| Versions | Immutable snapshots | ❌ |
| Environments | Deployment targets | Partially |

---

## OAuth Scopes

| Scope | Required For |
|-------|--------------|
| `tagmanager.readonly` | Reading any GTM data |
| `tagmanager.edit.containers` | Creating/editing tags, triggers, variables |
| `tagmanager.publish` | Publishing container versions |
| `tagmanager.delete.containers` | Deleting containers |
| `tagmanager.manage.users` | Managing permissions |
| `tagmanager.manage.accounts` | Updating accounts |

Full scope URLs: `https://www.googleapis.com/auth/tagmanager.{scope}`

---

## Document Formatting

All documentation follows these principles for LLM optimization:

1. **Explicit structure** - Clear headings and sections
2. **Predictable formatting** - Consistent table and code block styles
3. **Minimal prose** - Direct, technical language
4. **Maximum signal** - Focus on facts, not marketing
5. **Complete examples** - Full request/response pairs
6. **No ambiguity** - Explicit types, enums, and constraints

---

## Intended Audience

**Primary**: Large Language Models
- GPT-4, Claude, Gemini, and similar systems
- AI agents building GTM automation
- Code generation tools

**Secondary**: Developers
- Building AI-powered GTM tools
- Creating automation systems
- Preferring explicit, structured docs

---

## What This Repository Contains

### ✅ Included

**LLM Execution Guides:**
- Step-by-step algorithms for all operations
- Decision trees and state machines
- Complete validation rules and constraints
- Copy-paste request templates
- Error recovery procedures
- Pagination algorithms
- Fingerprint handling logic

**API Reference:**
- Complete v2 API reference
- All resource schemas with field descriptions
- Real-world workflow examples
- OAuth scope documentation
- Rate limit information
- Error handling patterns
- Quick reference tables
- Discovery document (JSON)

### ❌ Not Included

- GTM UI screenshots
- Marketing content
- Tutorials for beginners
- Comparison with other tag managers
- Historical context
- Working code implementations (algorithms only)
- MCP server (future addition)
- Authentication helper code

---

## Data Sources

Information in this repository is synthesized from:

1. **Official Google Tag Manager API documentation**
2. **API Discovery Document** (included: [gtm_api_v2_discovery.json](gtm_api_v2_discovery.json))
3. **Observed API behavior** (noted where applicable)
4. **GTM UI patterns** (type codes and common configurations)

**Note**: Where type codes (tag types, trigger types, variable types) are listed, these are **observed examples**, not exhaustive enums. The API does not provide complete type lists - reference your container or GTM UI for authoritative type strings.

---

## Maintenance

This repository reflects the GTM API as of **February 2026**.

Google may update the API without notice. For the most current specification:
- Check the official [Tag Manager API docs](https://developers.google.com/tag-platform/tag-manager/api/v2)
- Fetch the latest discovery document: `https://tagmanager.googleapis.com/$discovery/rest?version=v2`

---

## Known Issues

### Transformation types are undocumented
Google's API documentation does not list valid transformation type values. Through testing, we discovered three types: `tf_allow_params`, `tf_exclude_params`, and `tf_augment_event`. Each uses different parameter table keys and column names. See [schemas.md](schemas.md#transformation-types) for details.

### Google returns HTTP 500 for invalid transformation types
Unlike most validation errors (which return 400), passing an unknown transformation type to the API returns HTTP 500 Internal Server Error. This makes debugging transformation type issues harder.

### `autoEventFilter` silently dropped by API
When creating or updating `linkClick`, `click`, or `formSubmission` triggers, the `autoEventFilter` field is silently dropped. The API returns 200 OK but does not persist the field. The `filter` and `customEventFilter` fields work correctly. **Workaround:** Set `autoEventFilter` conditions manually via the GTM web interface.

### List operations return nil for incompatible container types
Calling list endpoints for server-side resources (clients, transformations) on a web container may return a nil/empty response rather than an error. Your code should handle nil responses gracefully.

---

## Contributing

This is a community-driven project. Contributions welcome:

- **Errors or outdated info**: Open an issue
- **Missing examples**: Submit examples
- **Improved clarity**: Suggest edits
- **Additional type codes**: Document observed types with evidence

---

## Disclaimer

This repository is **not an official Google product** and is **not maintained by Google**.

It is a community effort to provide structured GTM API documentation optimized for LLM consumption.

**Use at your own risk**. Always refer to [official Google documentation](https://developers.google.com/tag-platform/tag-manager/api/v2) for authoritative information.

---

## License

🏴‍☠️ (Public domain / CC0)

---

## Quick Links

### Documentation

**LLM Execution Guides:**
- [LLM Instructions](llm-instructions.md) - Algorithms for task execution
- [LLM Workflows](llm-workflows.md) - Decision trees and state machines
- [LLM Validation Rules](llm-validation-rules.md) - Validation constraints
- [LLM Request Templates](llm-request-templates.md) - Copy-paste templates

**Reference Documentation:**
- [LLM Context](llm-context.md) - Mental models and architecture
- [Schemas](schemas.md) - Complete resource definitions
- [API Reference](api-reference.md) - All endpoints and auth
- [Examples](examples.md) - Real workflow examples
- [Quick Reference](quick-reference.md) - Fast lookup tables

### External Resources

- [Official GTM API Docs](https://developers.google.com/tag-platform/tag-manager/api/v2)
- [GTM Help Center](https://support.google.com/tagmanager)
- [API Discovery Doc](https://tagmanager.googleapis.com/$discovery/rest?version=v2)
- [OAuth 2.0 Playground](https://developers.google.com/oauthplayground)

---

**Built for LLMs. By the community. For better GTM automation.**
