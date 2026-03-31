# Sample Metadata Catalog

Copy any sample below, paste it to the agent, and say **"Generate this page"**.
Swap the `baseUrl`, `endpointPath`, `serviceName`, `operationId`, and field names for your
own REST endpoint.

---

## Contents

1. [Employee Search — Smart Search, ADF](#1-employee-search--smart-search-adf)
2. [Project List — Smart Search, Non-ADF (ORDS)](#2-project-list--smart-search-non-adf-ords)
3. [Operations Grid — Data Management, Cell Edit, Full CRUD + Hide/Show + Data Transfer](#3-operations-grid--data-management-cell-edit-full-crud)
4. [Invoice Manager — Data Management, Row Edit + End Drawer](#4-invoice-manager--data-management-row-edit--end-drawer)
5. [Resource Planner — Data Management, Tabs (Data Collections Organizer)](#5-resource-planner--data-management-tabs)
6. [Report Viewer — Data Management, Read-Only Grid](#6-report-viewer--data-management-read-only)

---

## 1. Employee Search — Smart Search, ADF

**Use when:** Browsing a large employee list with keyword search and filter chips.
ADF/Oracle Business Objects backend — server-side filtering via `ListDataProviderView`.

```json
{
  "template": "smart-search-page",
  "pageName": "employee-search",
  "pageTitle": "Employees",
  "pageSubtitle": "Search and browse employees",
  "flowName": "employee-management",
  "appType": "standalone",
  "backendType": "adf",
  "layout": "fullWidth",

  "entity": {
    "name": "Employee",
    "keyField": "id",
    "displayFields": ["firstName", "lastName", "email", "department", "jobTitle", "status"]
  },

  "rest": {
    "baseUrl": "https://myapp.oraclecloud.com/ic/api/v1",
    "endpointPath": "/Employee",
    "serviceName": "businessObjects",
    "operationId": "getall_Employee",
    "responseShape": {
      "itemsPath": "items",
      "fields": [
        { "name": "id",         "type": "number" },
        { "name": "firstName",  "type": "string", "label": "First Name" },
        { "name": "lastName",   "type": "string", "label": "Last Name" },
        { "name": "email",      "type": "string" },
        { "name": "department", "type": "number", "label": "Department" },
        { "name": "jobTitle",   "type": "string", "label": "Job Title" },
        { "name": "salary",     "type": "number" },
        { "name": "status",     "type": "string" }
      ],
      "paginationStyle": "offset-limit"
    }
  },

  "search": {
    "keywordFields": ["firstName", "lastName", "email"],
    "placeholder": "Search employees",
    "configFileName": "employeeSearchConfig.json",
    "filters": [
      {
        "name": "DepartmentFilter",
        "label": "Department",
        "type": "SelectSingle",
        "field": "department",
        "options": {
          "source": "dynamic",
          "dynamicEndpoint": {
            "serviceName": "businessObjects",
            "operationId": "getall_Department",
            "labelField": "departmentName",
            "valueField": "id"
          }
        }
      },
      {
        "name": "StatusFilter",
        "label": "Status",
        "type": "SelectMultiple",
        "field": "status",
        "options": {
          "source": "static",
          "staticValues": [
            { "label": "Active",     "value": "ACTIVE" },
            { "label": "Inactive",   "value": "INACTIVE" },
            { "label": "On Leave",   "value": "ON_LEAVE" }
          ]
        }
      },
      {
        "name": "SalaryFilter",
        "label": "Salary Range",
        "type": "NumericRange",
        "field": "salary",
        "range": { "min": 0, "max": 200000, "defaultStart": 0, "defaultEnd": 100000 }
      }
    ]
  },

  "collection": {
    "displayType": "dynamic-table",
    "showCollectionContainer": false,
    "selectionMode": "single",
    "scrollPolicy": "loadMoreOnScroll",
    "fetchSize": 25,
    "columns": [
      { "field": "firstName",  "headerText": "First Name",  "sortable": "enabled", "width": "140px" },
      { "field": "lastName",   "headerText": "Last Name",   "sortable": "enabled", "width": "140px" },
      { "field": "email",      "headerText": "Email",       "sortable": "enabled" },
      { "field": "department", "headerText": "Department",  "sortable": "enabled", "width": "140px" },
      { "field": "jobTitle",   "headerText": "Job Title",   "sortable": "enabled" },
      { "field": "status",     "headerText": "Status",      "sortable": "enabled", "width": "110px", "template": "status-badge" }
    ]
  },

  "navigation": {
    "drillDownPage": "employee-detail",
    "drillDownParam": "employeeId",
    "drillDownField": "id"
  },

  "actions": {
    "primary": { "label": "Create Employee", "action": "navigate", "target": "employee-create" }
  }
}
```

---

## 2. Project List — Smart Search, Non-ADF (ORDS)

**Use when:** Browsing projects on a custom REST/ORDS backend. Filtering is done
client-side in the browser (ADP + JavaScript filter chain).

```json
{
  "template": "smart-search-page",
  "pageName": "project-list",
  "pageTitle": "Projects",
  "pageSubtitle": "Browse and filter active projects",
  "flowName": "project-management",
  "appType": "standalone",
  "backendType": "non-adf",
  "layout": "fullWidth",

  "entity": {
    "name": "Project",
    "keyField": "projectId",
    "displayFields": ["projectName", "owner", "startDate", "endDate", "priority", "status"]
  },

  "rest": {
    "baseUrl": "https://myords.example.com/ords/myschema",
    "endpointPath": "/projects",
    "serviceName": "ordsApi",
    "operationId": "getProjects",
    "responseShape": {
      "itemsPath": "items",
      "fields": [
        { "name": "projectId",   "type": "number" },
        { "name": "projectName", "type": "string", "label": "Project Name" },
        { "name": "owner",       "type": "string" },
        { "name": "startDate",   "type": "string", "label": "Start Date" },
        { "name": "endDate",     "type": "string", "label": "End Date" },
        { "name": "priority",    "type": "string" },
        { "name": "status",      "type": "string" },
        { "name": "budget",      "type": "number" }
      ],
      "paginationStyle": "offset-limit"
    }
  },

  "search": {
    "keywordFields": ["projectName", "owner"],
    "placeholder": "Search projects",
    "configFileName": "projectSearchConfig.json",
    "filters": [
      {
        "name": "PriorityFilter",
        "label": "Priority",
        "type": "SelectMultiple",
        "field": "priority",
        "options": {
          "source": "static",
          "staticValues": [
            { "label": "Critical", "value": "CRITICAL" },
            { "label": "High",     "value": "HIGH" },
            { "label": "Medium",   "value": "MEDIUM" },
            { "label": "Low",      "value": "LOW" }
          ]
        }
      },
      {
        "name": "StatusFilter",
        "label": "Status",
        "type": "SelectSingle",
        "field": "status",
        "options": {
          "source": "static",
          "staticValues": [
            { "label": "Planning",   "value": "PLANNING" },
            { "label": "Active",     "value": "ACTIVE" },
            { "label": "On Hold",    "value": "ON_HOLD" },
            { "label": "Completed",  "value": "COMPLETED" }
          ]
        }
      },
      {
        "name": "BudgetFilter",
        "label": "Budget",
        "type": "NumericRange",
        "field": "budget",
        "range": { "min": 0, "max": 5000000, "defaultStart": 0, "defaultEnd": 1000000 }
      }
    ]
  },

  "collection": {
    "displayType": "dynamic-table",
    "showCollectionContainer": false,
    "selectionMode": "single",
    "scrollPolicy": "loadMoreOnScroll",
    "fetchSize": 20,
    "columns": [
      { "field": "projectName", "headerText": "Project",    "sortable": "enabled" },
      { "field": "owner",       "headerText": "Owner",      "sortable": "enabled", "width": "150px" },
      { "field": "startDate",   "headerText": "Start",      "sortable": "enabled", "width": "120px", "template": "date" },
      { "field": "endDate",     "headerText": "End",        "sortable": "enabled", "width": "120px", "template": "date" },
      { "field": "priority",    "headerText": "Priority",   "sortable": "enabled", "width": "110px", "template": "status-badge" },
      { "field": "status",      "headerText": "Status",     "sortable": "enabled", "width": "120px", "template": "status-badge" }
    ]
  },

  "navigation": {
    "drillDownPage": "project-detail",
    "drillDownParam": "projectId",
    "drillDownField": "projectId"
  }
}
```

---

## 3. Operations Grid — Data Management, Cell Edit, Full CRUD

**Use when:** Inline cell editing on a data grid with batch save/cancel. Includes
full CRUD operations, column hide/show, data transfer (cut/copy/paste/fill), and
column header filters.

```json
{
  "template": "data-management-page",
  "pageName": "operations-grid",
  "pageTitle": "Operations",
  "pageSubtitle": "Manage operational data inline",
  "flowName": "operations",
  "appType": "standalone",
  "backendType": "adf",
  "layout": "fullWidth",

  "entity": {
    "name": "Operation",
    "keyField": "id"
  },

  "rest": {
    "baseUrl": "https://myapp.oraclecloud.com/ic/api/v1",
    "endpointPath": "/Operation",
    "serviceName": "businessObjects",
    "operationId": "getall_Operation",
    "operations": {
      "create":  "create_Operation",
      "update":  "update_Operation",
      "delete":  "delete_Operation",
      "getById": "get_Operation"
    },
    "errorResponsePath": "o:errorDetails[0].detail",
    "responseShape": {
      "itemsPath": "items",
      "fields": [
        { "name": "id",          "type": "number" },
        { "name": "name",        "type": "string" },
        { "name": "status",      "type": "string" },
        { "name": "priority",    "type": "string" },
        { "name": "assignee",    "type": "string" },
        { "name": "startDate",   "type": "string" },
        { "name": "dueDate",     "type": "string" },
        { "name": "budget",      "type": "number" },
        { "name": "notes",       "type": "string" }
      ],
      "paginationStyle": "offset-limit"
    }
  },

  "dataGrid": {
    "editMode": "cell-edit",
    "pattern": "data-grid-pattern",
    "columnHeaderFiltering": true,

    "columns": [
      { "field": "name",      "headerText": "Operation",  "type": "text",   "editable": true,  "width": "200px",
        "validation": { "required": true } },
      { "field": "status",    "headerText": "Status",     "type": "select", "editable": true,  "width": "130px",
        "lov": { "serviceName": "businessObjects", "operationId": "getall_OperationStatus",
                 "displayField": "label", "valueField": "value" } },
      { "field": "priority",  "headerText": "Priority",   "type": "select", "editable": true,  "width": "120px",
        "lov": { "serviceName": "businessObjects", "operationId": "getall_Priority",
                 "displayField": "label", "valueField": "value" } },
      { "field": "assignee",  "headerText": "Assignee",   "type": "text",   "editable": true,  "width": "150px" },
      { "field": "startDate", "headerText": "Start Date", "type": "date",   "editable": true,  "width": "130px" },
      { "field": "dueDate",   "headerText": "Due Date",   "type": "date",   "editable": true,  "width": "130px" },
      { "field": "budget",    "headerText": "Budget",     "type": "number", "editable": true,  "width": "120px",
        "validation": { "min": 0 } },
      { "field": "notes",     "headerText": "Notes",      "type": "text",   "editable": true }
    ],

    "hideShow": {
      "columnHidable": "enable",
      "initialHiddenColumns": [7, 8],
      "programmaticControl": true
    },

    "dataTransfer": {
      "cut":   "enable",
      "copy":  "enable",
      "paste": "enable",
      "fill":  "enable"
    }
  },

  "transactional": {
    "saveAction":             true,
    "cancelAction":           true,
    "draftAction":            false,
    "insertRow":              true,
    "deleteRow":              true,
    "unsavedChangesDialog":   true
  },

  "actions": {
    "primary": { "label": "Save Changes", "action": "custom" },
    "secondary": [
      { "label": "Discard", "action": "custom" }
    ]
  }
}
```

---

## 4. Invoice Manager — Data Management, Row Edit + End Drawer

**Use when:** Row-by-row editing of invoices with a detail side drawer that opens on
row click. End drawer is preferred for grids that scroll vertically.

```json
{
  "template": "data-management-page",
  "pageName": "invoice-manager",
  "pageTitle": "Invoices",
  "pageSubtitle": "Review and edit invoice records",
  "flowName": "finance",
  "appType": "standalone",
  "backendType": "adf",
  "layout": "fullWidth",

  "entity": {
    "name": "Invoice",
    "keyField": "invoiceId"
  },

  "rest": {
    "baseUrl": "https://myapp.oraclecloud.com/ic/api/v1",
    "endpointPath": "/Invoice",
    "serviceName": "businessObjects",
    "operationId": "getall_Invoice",
    "operations": {
      "create":  "create_Invoice",
      "update":  "update_Invoice",
      "delete":  "delete_Invoice",
      "getById": "get_Invoice"
    },
    "errorResponsePath": "o:errorDetails[0].detail",
    "responseShape": {
      "itemsPath": "items",
      "fields": [
        { "name": "invoiceId",    "type": "number" },
        { "name": "invoiceNo",    "type": "string",  "label": "Invoice #" },
        { "name": "vendor",       "type": "string" },
        { "name": "issueDate",    "type": "string",  "label": "Issue Date" },
        { "name": "dueDate",      "type": "string",  "label": "Due Date" },
        { "name": "amount",       "type": "number" },
        { "name": "currency",     "type": "string" },
        { "name": "status",       "type": "string" },
        { "name": "description",  "type": "string" }
      ],
      "paginationStyle": "offset-limit"
    }
  },

  "dataGrid": {
    "editMode": "row-edit",
    "pattern": "data-grid-pattern",
    "columnHeaderFiltering": false,

    "columns": [
      { "field": "invoiceNo",   "headerText": "Invoice #",  "type": "text",   "editable": false, "width": "130px" },
      { "field": "vendor",      "headerText": "Vendor",     "type": "text",   "editable": true,  "width": "180px",
        "validation": { "required": true } },
      { "field": "issueDate",   "headerText": "Issue Date", "type": "date",   "editable": true,  "width": "130px" },
      { "field": "dueDate",     "headerText": "Due Date",   "type": "date",   "editable": true,  "width": "130px" },
      { "field": "amount",      "headerText": "Amount",     "type": "number", "editable": true,  "width": "130px",
        "validation": { "required": true, "min": 0 } },
      { "field": "currency",    "headerText": "Currency",   "type": "select", "editable": true,  "width": "100px",
        "lov": { "serviceName": "businessObjects", "operationId": "getall_Currency",
                 "displayField": "code", "valueField": "code" } },
      { "field": "status",      "headerText": "Status",     "type": "select", "editable": true,  "width": "120px",
        "lov": { "serviceName": "businessObjects", "operationId": "getall_InvoiceStatus",
                 "displayField": "label", "valueField": "value" } }
    ]
  },

  "drawer": {
    "enabled":  true,
    "position": "end",
    "type":     "inline",
    "trigger":  "row-click",
    "size":     "medium"
  },

  "transactional": {
    "saveAction":           true,
    "cancelAction":         true,
    "draftAction":          false,
    "insertRow":            true,
    "deleteRow":            true,
    "unsavedChangesDialog": true
  },

  "navigation": {
    "parentPage":  "finance-dashboard",
    "parentLabel": "Finance"
  },

  "actions": {
    "primary": { "label": "New Invoice", "action": "drawer" },
    "secondary": [
      { "label": "Export", "action": "export" }
    ]
  }
}
```

---

## 5. Resource Planner — Data Management, Tabs

**Use when:** Multiple related data collections organized by tabs — each tab has its
own data grid. Uses the Data Collections Organizer Pattern.

```json
{
  "template": "data-management-page",
  "pageName": "resource-planner",
  "pageTitle": "Resource Planner",
  "pageSubtitle": "Plan and track resources by category",
  "flowName": "planning",
  "appType": "standalone",
  "backendType": "adf",
  "layout": "fullWidth",

  "entity": {
    "name": "Resource",
    "keyField": "resourceId"
  },

  "rest": {
    "baseUrl": "https://myapp.oraclecloud.com/ic/api/v1",
    "endpointPath": "/Resource",
    "serviceName": "businessObjects",
    "operationId": "getall_Resource",
    "operations": {
      "create":  "create_Resource",
      "update":  "update_Resource",
      "delete":  "delete_Resource",
      "getById": "get_Resource"
    },
    "errorResponsePath": "o:errorDetails[0].detail",
    "responseShape": {
      "itemsPath": "items",
      "fields": [
        { "name": "resourceId",  "type": "number" },
        { "name": "name",        "type": "string" },
        { "name": "type",        "type": "string" },
        { "name": "capacity",    "type": "number" },
        { "name": "allocated",   "type": "number" },
        { "name": "region",      "type": "string" },
        { "name": "status",      "type": "string" }
      ],
      "paginationStyle": "offset-limit"
    }
  },

  "dataGrid": {
    "editMode": "cell-edit",
    "pattern": "data-collections-organizer-pattern",
    "columnHeaderFiltering": false,

    "tabs": [
      { "id": "human",     "label": "Human Resources",  "dataSource": "humanResourcesSDP" },
      { "id": "equipment", "label": "Equipment",         "dataSource": "equipmentSDP" },
      { "id": "facilities","label": "Facilities",         "dataSource": "facilitiesSDP" }
    ],

    "columns": [
      { "field": "name",      "headerText": "Resource",   "type": "text",   "editable": true,  "width": "180px",
        "validation": { "required": true } },
      { "field": "region",    "headerText": "Region",     "type": "select", "editable": true,  "width": "130px",
        "lov": { "serviceName": "businessObjects", "operationId": "getall_Region",
                 "displayField": "name", "valueField": "code" } },
      { "field": "capacity",  "headerText": "Capacity",   "type": "number", "editable": true,  "width": "110px",
        "validation": { "min": 0 } },
      { "field": "allocated", "headerText": "Allocated",  "type": "number", "editable": true,  "width": "110px",
        "validation": { "min": 0 } },
      { "field": "status",    "headerText": "Status",     "type": "select", "editable": true,  "width": "120px",
        "lov": { "serviceName": "businessObjects", "operationId": "getall_ResourceStatus",
                 "displayField": "label", "valueField": "value" } }
    ]
  },

  "transactional": {
    "saveAction":           true,
    "cancelAction":         true,
    "draftAction":          true,
    "insertRow":            true,
    "deleteRow":            true,
    "unsavedChangesDialog": true
  },

  "contextSwitcher": {
    "enabled": true,
    "options": [
      { "value": "2025", "label": "FY 2025" },
      { "value": "2026", "label": "FY 2026" }
    ],
    "defaultValue": "2026"
  }
}
```

---

## 6. Report Viewer — Data Management, Read-Only

**Use when:** Displaying a read-only data grid — reports, audit logs, reference data.
No edit controls, no save/cancel. Includes hide/show and data transfer for
export-like clipboard behaviour.

```json
{
  "template": "data-management-page",
  "pageName": "audit-log",
  "pageTitle": "Audit Log",
  "pageSubtitle": "System activity log — read only",
  "flowName": "admin",
  "appType": "standalone",
  "backendType": "adf",
  "layout": "fullWidth",

  "entity": {
    "name": "AuditLog",
    "keyField": "logId"
  },

  "rest": {
    "baseUrl": "https://myapp.oraclecloud.com/ic/api/v1",
    "endpointPath": "/AuditLog",
    "serviceName": "businessObjects",
    "operationId": "getall_AuditLog",
    "responseShape": {
      "itemsPath": "items",
      "fields": [
        { "name": "logId",      "type": "number" },
        { "name": "timestamp",  "type": "string" },
        { "name": "user",       "type": "string" },
        { "name": "action",     "type": "string" },
        { "name": "entity",     "type": "string" },
        { "name": "entityId",   "type": "number",  "label": "Record ID" },
        { "name": "oldValue",   "type": "string",  "label": "Before" },
        { "name": "newValue",   "type": "string",  "label": "After" },
        { "name": "ipAddress",  "type": "string",  "label": "IP Address" },
        { "name": "result",     "type": "string" }
      ],
      "paginationStyle": "offset-limit"
    }
  },

  "dataGrid": {
    "editMode": "read-only",
    "pattern": "data-grid-pattern",
    "columnHeaderFiltering": true,

    "columns": [
      { "field": "timestamp",  "headerText": "Timestamp",  "editable": false, "width": "170px" },
      { "field": "user",       "headerText": "User",        "editable": false, "width": "150px" },
      { "field": "action",     "headerText": "Action",      "editable": false, "width": "120px" },
      { "field": "entity",     "headerText": "Entity",      "editable": false, "width": "130px" },
      { "field": "entityId",   "headerText": "Record ID",   "editable": false, "width": "100px" },
      { "field": "oldValue",   "headerText": "Before",      "editable": false },
      { "field": "newValue",   "headerText": "After",       "editable": false },
      { "field": "result",     "headerText": "Result",      "editable": false, "width": "110px" }
    ],

    "hideShow": {
      "columnHidable": "enable",
      "initialHiddenColumns": [6, 7],
      "programmaticControl": false
    },

    "dataTransfer": {
      "cut":   "disable",
      "copy":  "enable",
      "paste": "disable",
      "fill":  "disable"
    }
  },

  "navigation": {
    "parentPage":  "admin-dashboard",
    "parentLabel": "Admin"
  }
}
```

---

## Quick Reference — Which Template?

| Scenario | Template | backendType | editMode |
|---|---|---|---|
| Browse / search large list with filters | `smart-search-page` | `adf` or `non-adf` | — |
| Inline cell editing on a grid (spreadsheet-like) | `data-management-page` | `adf` | `cell-edit` |
| Row-by-row editing (form per row) | `data-management-page` | `adf` | `row-edit` |
| Multiple related collections in tabs | `data-management-page` | `adf` | `cell-edit` |
| Read-only display / report / audit log | `data-management-page` | `adf` or `non-adf` | `read-only` |

## Quick Reference — `backendType`

| Value | When to use | Filtering |
|---|---|---|
| `adf` | Oracle Business Objects, Fusion REST, any ADF backend | Server-side via `ListDataProviderView` + `mapFilterCriterion` |
| `non-adf` | Spring Boot, Node.js, ORDS custom REST, any non-Oracle backend | Client-side via `ArrayDataProvider2` + JavaScript filter chain |
