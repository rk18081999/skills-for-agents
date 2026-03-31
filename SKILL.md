---
name: vbcs-page-builder
description: >
  Orchestrator skill for building Oracle VBCS pages from developer metadata. This is
  the entry point for all page generation. Trigger whenever a developer provides a
  page metadata JSON, asks to "build a page", "create a page", "generate a page",
  or references any Redwood page template (smart-search, data-management, welcome,
  general-overview, item-overview). Also trigger when the user mentions "metadata-driven",
  "page builder",
  "page generator", or wants to scaffold a new VBCS page from a specification. This
  skill routes to the correct template-specific skill based on the metadata. Even if
  the user just says "build me a search page" or "I need a new page", use this skill.
---

# VBCS Page Builder — Orchestrator

This skill reads developer-provided metadata and generates complete, production-ready
VBCS pages following Oracle Redwood Design System patterns.

## How It Works

1. Developer provides metadata JSON (or answers questions to build one)
2. This orchestrator validates the metadata and identifies the Redwood template
3. Routes to the correct template-specific skill in `skills/templates/`
4. The template skill generates all page files using its code templates + demo code
5. Output is validated against shared rules and template-specific checklists

Before generating any code, **always** read:
- This file (routing + metadata schema)
- The target template's `SKILL.md` in `skills/templates/{template-name}/`
- The target template's `references/` files as directed by its SKILL.md
- The relevant knowledge docs from `knowledge/` as directed by `skills/VBCS_Expert/SKILL.md`

---

## Step 0: Receive or Build Metadata

If the developer provides a metadata JSON file, validate it against the schema in
`metadata-schema.json`. If the developer describes their requirements conversationally,
build the metadata JSON by asking for the required fields.

### Required Fields (All Templates)

| Field | Type | Description |
|-------|------|-------------|
| `template` | string | Redwood template identifier |
| `pageName` | string | VBCS page ID (kebab-case) |
| `pageTitle` | string | Display title |
| `flowName` | string | VBCS flow name |
| `appType` | string | `"standalone"` or `"extension"` |
| `backendType` | string | `"adf"` or `"non-adf"` |
| `rest.serviceName` | string | VBCS service connection name |
| `rest.operationId` | string | GET list operation ID from OpenAPI spec |

### REST Endpoint Location (All Templates)

Every template needs to know the physical data store location. Always capture these fields
alongside `serviceName` and `operationId`:

| Field | Description | Example |
|-------|-------------|---------|
| `rest.baseUrl` | Physical API base URL | `https://myapp.oraclecloud.com/ic/api/v1` |
| `rest.endpointPath` | Resource path on the API | `/Employee`, `/projects` |

If the service connection does not yet exist, `baseUrl` + `endpointPath` gives the agent
everything needed to bootstrap an `openapi3.json` and `catalog.json` — run the
`workflows/service-connection.md` workflow before page generation.

For **data management pages** (create/update/delete), also capture:

| Field | Description | Example |
|-------|-------------|---------|
| `rest.operations.create` | POST operation ID | `create_Employee` |
| `rest.operations.update` | PATCH/PUT operation ID | `update_Employee` |
| `rest.operations.delete` | DELETE operation ID | `delete_Employee` |
| `rest.operations.getById` | GET single record ID | `get_Employee` |
| `rest.errorResponsePath` | Error message path in response | `o:errorDetails[0].detail` |

### Template-Specific Fields

Each template defines additional required/optional fields. See the metadata schema
for the full specification.

---

## Step 1: Route to Template Skill

Read the `template` field and route to the correct skill:

| Template Value | Skill Path | Status |
|---------------|------------|--------|
| `smart-search-page` | `skills/templates/smart-search-page/SKILL.md` | **Ready** |
| `data-management-page` | `skills/templates/data-management-page/SKILL.md` | **Ready** |
| `welcome-page` | `skills/templates/welcome-page/SKILL.md` | Planned |
| `general-overview-page` | `skills/templates/general-overview-page/SKILL.md` | Planned |
| `item-overview-page` | `skills/templates/item-overview-page/SKILL.md` | Planned |

If the template is "Planned", inform the developer and offer to scaffold a basic
page using the `knowledge/09-redwood-layout-adp-patterns-components.md` patterns
as a fallback.

---

## Step 2: Execute Template Skill

The template skill handles:
1. Reading its `references/` for code templates and configuration guides
2. Consulting `demos/` for Oracle-verified working code
3. Reading the appropriate `prompts/` for the action type
4. Generating all page files (HTML, JS, JSON, chains, config files)
5. Applying template-specific validation rules

---

## Step 3: Validate Output

After generation, validate against:
1. `rules/checklist.md` — universal VBCS rules
2. `rules/vbcs-best-practices.md` — coding standards
3. `skills/VBCS_Validation_Checklist.md` — comprehensive validation
4. Template-specific do's/don'ts from the template SKILL.md

---

## Step 4: Deliver Files

Present the generated files organized by VBCS project structure:

```
flows/{flowName}/
├── {flowName}-flow.js
├── {flowName}-flow.json
└── pages/
    ├── {pageName}-page.html
    ├── {pageName}-page.js
    ├── {pageName}-page.json
    └── {pageName}-page-chains/    (if non-ADF)
        ├── {Entity}InitChain.js
        └── {Entity}FilterChain.js
resources/
└── data/
    └── {configFileName}.json      (if smart-search)
```

---

## Adding New Templates

To add a new Redwood page template to the framework:

1. Create folder: `skills/templates/{template-name}/`
2. Add `SKILL.md` — template-specific instructions, anatomy, do's/don'ts
3. Add `references/` — code-templates.md, any config guides
4. Add `demos/` — real Oracle demo files (HTML, JS, JSON)
5. Add `prompts/` — pre-built prompts for CREATE, MODIFY, DEBUG
6. Update the routing table in this file
7. Extend `metadata-schema.json` with template-specific fields

The template SKILL.md should follow this structure:
- Prerequisites
- Step 0: Gather requirements (or read from metadata)
- Architecture decision (if template has variants like ADF vs non-ADF)
- Step-by-step implementation (PATH A / PATH B if needed)
- Common mistakes
- Cross-references to shared knowledge docs

---

## Cross-Reference: Shared Knowledge

The template skills reference these shared docs as needed:

| Need | Read |
|------|------|
| Page structure, flow config | `knowledge/01-project-structure.md` |
| Action chains | `knowledge/02-action-chains.md` |
| ADP patterns, notifications | `knowledge/03-notifications-and-adp.md` |
| REST calls, ORDS, caching | `knowledge/04-rest-and-data.md` |
| JET components | `knowledge/05-jet-components.md` |
| Common pitfalls | `knowledge/06-best-practices-and-pitfalls.md` |
| Lifecycle, forms, templates | `knowledge/07-lifecycle-templates-forms.md` |
| Debugging, variables | `knowledge/08-debugging-navigation-variables.md` |
| Redwood layouts, ADP+table | `knowledge/09-redwood-layout-adp-patterns-components.md` |
