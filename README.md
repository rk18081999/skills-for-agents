# .agent — VBCS Page Builder Framework

A scalable, metadata-driven framework for AI coding agents to generate Oracle VBCS
pages following Redwood Design System patterns. Supports Claude Code, Cursor, Windsurf,
and other IDE-based AI assistants.

## Architecture

```
.agent/
├── README.md                          ← You are here
├── SKILL.md                           ← Orchestrator: routes metadata to template skills
├── metadata-schema.json               ← JSON schema for developer input
├── sample-metadata.md                 ← Ready-to-use metadata samples for all templates
│
├── knowledge/                         ← Core VBCS reference (01–09, unchanged)
│   ├── 01-project-structure.md
│   ├── 02-action-chains.md
│   ├── 03-notifications-and-adp.md
│   ├── 04-rest-and-data.md
│   ├── 05-jet-components.md
│   ├── 06-best-practices-and-pitfalls.md
│   ├── 07-lifecycle-templates-forms.md
│   ├── 08-debugging-navigation-variables.md
│   └── 09-redwood-layout-adp-patterns-components.md
│
├── rules/                             ← Always-on guardrails (unchanged)
│   ├── checklist.md
│   ├── redwood-components-reference-1.md
│   ├── rules.md
│   └── vbcs-best-practices.md
│
├── workflows/                         ← Step-by-step task templates (unchanged)
│   └── service-connection.md
│
└── skills/
    ├── VBCS_Expert/SKILL.md           ← Foundation skill (unchanged)
    ├── VBCS_Prompt_Templates.md       ← General prompt templates (unchanged)
    ├── VBCS_Prompting_Playbook.md     ← Prompting guide (unchanged)
    ├── VBCS_Validation_Checklist.md   ← Validation rules (unchanged)
    │
    └── templates/                     ← Template-specific skills (scalable)
        ├── smart-search-page/         ← ✅ READY
        │   ├── SKILL.md              ← Template instructions + ADF/non-ADF paths
        │   ├── references/
        │   │   ├── code-templates.md  ← All HTML/JSON/JS templates with placeholders
        │   │   ├── filter-config-guide.md ← Filter metadata configuration
        │   │   └── redwood-spec.md    ← Redwood design spec (anatomy, responsive, a11y)
        │   ├── demos/                 ← Oracle VB Cookbook verified working code
        │   │   ├── page.html
        │   │   ├── page.js
        │   │   ├── page.json
        │   │   └── employeeSearchConfig.json
        │   └── prompts/
        │       └── smart-search-prompts.md ← CREATE/ADD/MODIFY/DEBUG prompts
        │
        ├── data-management-page/      ← ✅ READY
        │   ├── SKILL.md              ← Template instructions + cell-edit/row-edit/read-only paths
        │   ├── references/
        │   │   ├── code-templates.md  ← Data grid, table, drawer, tab templates
        │   │   └── redwood-spec.md    ← DMPT design spec (anatomy, drawers, messaging)
        │   ├── demos/                 ← Oracle VB Cookbook verified working code
        │   │   ├── data-grid-editable-page.*    ← Cell edit with BDP
        │   │   ├── batch-editable-table-page.*  ← Row edit with BDP
        │   │   ├── table-complex-filters-page.* ← Server-side filtering
        │   │   ├── data-grid-data-transfer-demo.* ← Cut/Copy/Paste/Fill
        │   │   └── data-grid-hide-show-demo.*    ← Column/Row Hide/Show
        │   └── prompts/
        │       └── data-management-prompts.md ← CREATE/ADD/MODIFY/DEBUG prompts
        │
        ├── welcome-page/              ← 🔜 PLANNED
        ├── general-overview-page/     ← 🔜 PLANNED
        └── item-overview-page/        ← 🔜 PLANNED
```

## How It Works

### For Developers (Users of the Agent)

1. **Create a metadata JSON** describing your page:
   - Which template (`smart-search-page`, `data-management-page`, `welcome-page`, etc.)
   - Page name, title, flow, REST endpoint, response shape
   - Template-specific blocks: e.g. `search` + `collection` (smart search), or `dataGrid` (data management)
   - See `metadata-schema.json` for the full specification

2. **Give it to the agent** with "Build this page" or "Generate from this metadata"

3. **Receive complete page files** — HTML, JS, JSON, action chains, and any extra artifacts (e.g. smart-search config JSON, BDP/grid wiring for data management)

### For the AI Agent

1. Read `SKILL.md` (orchestrator) to identify the template
2. Route to `skills/templates/{template}/SKILL.md`
3. Read the template's `references/` for code patterns
4. Consult `demos/` for Oracle-verified working code
5. Use `prompts/` for the right action type (CREATE/ADD/MODIFY/DEBUG)
6. Generate files, validate against `rules/`, deliver

### For Framework Maintainers (Adding New Templates)

1. Create `skills/templates/{template-name}/`
2. Add `SKILL.md` — anatomy, steps, do's/don'ts
3. Add `references/` — code-templates.md + any config guides
4. Add `demos/` — real working Oracle demo files
5. Add `prompts/` — template-specific prompt library
6. Update the routing table in the top-level `SKILL.md`
7. Extend `metadata-schema.json` with template-specific fields

## Quick Start

1. Place this `.agent/` folder in your VBCS project root
2. Your IDE agent picks up rules and skills automatically
3. Provide metadata JSON or describe your page requirements
4. The agent generates production-ready Redwood pages

## Metadata examples

### Smart search (`smart-search-page`)

```json
{
  "template": "smart-search-page",
  "pageName": "employee-search",
  "pageTitle": "Employees",
  "flowName": "employee-management",
  "appType": "standalone",
  "backendType": "adf",
  "entity": { "name": "Employee", "keyField": "id" },
  "rest": {
    "baseUrl": "https://myapp.oraclecloud.com/ic/api/v1",
    "endpointPath": "/Employee",
    "serviceName": "businessObjects",
    "operationId": "getall_Employee",
    "responseShape": { "itemsPath": "items" }
  },
  "search": {
    "keywordFields": ["firstName", "lastName"],
    "configFileName": "employeeSearchConfig.json",
    "filters": [
      { "name": "DepartmentsFilter", "label": "Departments", "type": "SelectSingle", "field": "department" },
      { "name": "SalaryFilter", "label": "Salary", "type": "NumericRange", "field": "salary", "range": { "min": 1, "max": 30000 } }
    ]
  },
  "collection": { "displayType": "dynamic-table" }
}
```

### Data management (`data-management-page`)

Optional filtering is expressed in two ways:

- **`dataGrid.columnHeaderFiltering`** — column header filter icons on `oj-data-grid` (LDPV / `filterCriterion`).
- **Top-level `search`** (same shape as smart-search) — keyword search + chip filters on the page toolbar, combinable with the grid per `data-management-page/references/code-templates.md` (Smart Search + table).

```json
{
  "template": "data-management-page",
  "pageName": "operations-grid",
  "pageTitle": "Operations",
  "flowName": "operations",
  "appType": "standalone",
  "backendType": "adf",
  "entity": { "name": "Operation", "keyField": "id" },
  "rest": {
    "baseUrl": "https://myapp.oraclecloud.com/ic/api/v1",
    "endpointPath": "/Operation",
    "serviceName": "businessObjects",
    "operationId": "getall_Operation",
    "operations": {
      "create": "create_Operation",
      "update": "update_Operation",
      "delete": "delete_Operation",
      "getById": "get_Operation"
    },
    "errorResponsePath": "o:errorDetails[0].detail",
    "responseShape": { "itemsPath": "items" }
  },
  "dataGrid": {
    "editMode": "cell-edit",
    "pattern": "data-grid-pattern",
    "columnHeaderFiltering": true,
    "dataTransfer": {
      "cut": "enable",
      "copy": "enable",
      "paste": "enable",
      "fill": "enable"
    },
    "columns": [
      { "field": "name", "headerText": "Name", "type": "text", "editable": true },
      { "field": "status", "headerText": "Status", "type": "select", "editable": true },
      { "field": "budget", "headerText": "Budget", "type": "number", "editable": true }
    ]
  },
  "search": {
    "keywordFields": ["name", "code"],
    "configFileName": "operationsSearchConfig.json",
    "filters": [
      {
        "name": "StatusFilter",
        "label": "Status",
        "type": "SelectSingle",
        "field": "status",
        "options": {
          "source": "static",
          "staticValues": [
            { "value": "OPEN", "label": "Open" },
            { "value": "CLOSED", "label": "Closed" }
          ]
        }
      },
      {
        "name": "BudgetFilter",
        "label": "Budget",
        "type": "NumericRange",
        "field": "budget",
        "range": { "min": 0, "max": 500000 }
      }
    ]
  },
  "transactional": {
    "saveAction": true,
    "cancelAction": true,
    "draftAction": false
  },
  "drawer": {
    "enabled": false,
    "position": "end",
    "type": "inline"
  }
}
```

Larger VB Cookbook samples for read-only grids (merged cells, styling, nested headers) live under the repository root `demos/` directory (e.g. `demos/data-grid-merged-2510/`). The template skill’s `demos/` folder holds trimmed page triplets aligned with this framework.
