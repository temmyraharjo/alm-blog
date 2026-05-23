# Power Platform Review Rules

## Overview

This document defines the review logic for unpacked Power Platform solution changes.

- **Change summary scope:** All supported solution component types (tables, attributes, app modules, flows/processes, relationships, web resources, etc.).
- **Naming rule scope:** Custom tables and custom attributes only (publisher-prefixed components such as `tmy_`).
- **System component naming:** System tables/attributes are excluded from naming-rule violations.

---

## Scope

| Scope Area | Component Type | In Scope | Notes |
|---|---|---|---|
| Change Summary | Tables (Entities) | ✅ | Include added/modified/deleted table components |
| Change Summary | Attributes (Columns) | ✅ | Include attribute changes when detectable from Entity.xml and related metadata |
| Change Summary | App Modules / Model-driven apps | ✅ | Include app module changes |
| Change Summary | App Module Site Maps | ✅ | Include sitemap changes |
| Change Summary | Power Automate Flows / Processes (Workflows) | ✅ | Include flow/process changes when present in unpacked solution |
| Change Summary | Relationships | ✅ | Include relationship changes |
| Change Summary | Web Resources | ✅ | Include web resource changes |
| Change Summary | Other solution components | ✅ | Include with best-effort type classification |
| Naming Rules | Custom tables | ✅ | Must carry publisher prefix (for example `tmy_`) |
| Naming Rules | Custom attributes | ✅ | Must carry publisher prefix (for example `tmy_`) |
| Naming Rules | System tables | ❌ | Exempt from naming-rule checks |
| Naming Rules | System attributes | ❌ | Exempt from naming-rule checks |

---

## Component Change Classification Rules

Use these rules to determine **component type** and **change action** in the human summary.

### Change action mapping

- `Added`: file path appears as untracked/added in change evidence.
- `Modified`: file path appears as modified in change evidence.
- `Deleted`: file path appears as deleted in change evidence.

### Component type mapping by unpacked path (best effort)

- `src/solutions/<solution>/Entities/<name>/Entity.xml` -> Table
- `src/solutions/<solution>/Entities/<name>/...` -> Table-related metadata (forms/views/ribbon)
- `src/solutions/<solution>/AppModules/<name>/AppModule.xml` -> App Module
- `src/solutions/<solution>/AppModuleSiteMaps/<name>/AppModuleSiteMap.xml` -> App Module Sitemap
- `src/solutions/<solution>/Other/Relationships*.xml` and `.../Other/Relationships/*.xml` -> Relationship
- `src/solutions/<solution>/Workflows/*.xml` or workflow/process paths -> Power Automate Flow / Process
- `src/solutions/<solution>/WebResources/**` -> Web Resource
- everything else -> Other Component

### Table and attribute extraction rules

- Table logical name: use folder name under `Entities/<table>`.
- Table display name: use DisplayName from `Entity.xml` when available, otherwise derive from logical name.
- Attribute rows: extract custom attributes from `Entity.xml` for the same publisher prefix as the table.
- Attribute data type: use attribute type metadata from XML; if unavailable, report `Unknown`.

---

## Naming Rules

---

### Logical names must be in lowercase

**Applies to:** Table logical names, attribute logical names

**Definition:**  
The full logical name — including the publisher prefix and the entity/attribute portion — must contain **no uppercase letters**. Every character must be either a lowercase letter (`a–z`), a digit (`0–9`), or an underscore (`_`).

**Rationale:**  
Dataverse stores logical names in lowercase. Inconsistent casing in schema names before saving causes confusion during solution comparisons, ALM pipelines, and code references.

**Examples:**

| Logical Name       | Status | Reason                              |
|--------------------|--------|-------------------------------------|
| `tmy_order`        | ✅ PASS | All lowercase                       |
| `tmy_orderdetail`  | ✅ PASS | All lowercase                       |
| `tmy_Order`        | ❌ FAIL | Contains uppercase `O`              |
| `tmy_OrderDetail`  | ❌ FAIL | Contains uppercase `O` and `D`      |
| `TMY_order`        | ❌ FAIL | Prefix contains uppercase letters   |

**LLM Check Instruction:**  
Scan the logical name character by character. If any character falls outside `[a-z0-9_]`, flag a violation of **Logical names must be in lowercase**.

---

### Table logical names must use singular form

**Applies to:** Table logical names only (not attributes)

**Definition:**  
The entity portion of a custom table's logical name must represent a **single record concept**, not a collection. Use the singular form of the noun.

**Rationale:**  
Each table record represents one instance of the entity. Plural naming implies a collection and conflicts with standard Dataverse conventions (e.g., `account`, `contact`, `invoice`).

**Examples:**

| Logical Name        | Status | Reason                                    |
|---------------------|--------|-------------------------------------------|
| `tmy_order`         | ✅ PASS | Singular                                  |
| `tmy_orderdetail`   | ✅ PASS | Singular                                  |
| `tmy_product`       | ✅ PASS | Singular                                  |
| `tmy_orders`        | ❌ FAIL | Plural — should be `tmy_order`            |
| `tmy_orderdetails`  | ❌ FAIL | Plural — should be `tmy_orderdetail`      |
| `tmy_deliveries`    | ❌ FAIL | Plural — should be `tmy_delivery`         |

**Common plural suffixes to flag:** `-s`, `-es`, `-ies` (converted from `-y`), `-ves`

**LLM Check Instruction:**  
Extract the entity portion of the name (everything after the first `_`). Check whether it ends with a common plural suffix (`s`, `es`, `ies`, `ves`). If yes, flag as a probable violation of **Table logical names must use singular form** and suggest the singular form. Note: some words are legitimately non-plural despite ending in `s` (e.g., `status`, `address`, `process`) — use context and common English to distinguish.

**Known non-violations (words ending in `s` that are singular):**

- `status`, `address`, `process`, `progress`, `access`, `class`, `canvas`

---

### Lookup attribute logical names must end with the suffix `id`

**Applies to:** Custom lookup (Many-to-One relationship) attributes only

**Definition:**  
Any attribute that stores a reference to another table (i.e., a lookup field) must have a logical name ending in `id`. The portion before `id` must meaningfully represent the target table or relationship purpose.

**Rationale:**  
Dataverse appends `id` to lookup logical names automatically when the schema name follows conventions. Enforcing this rule ensures schema names are set correctly before deployment and that lookup fields are immediately distinguishable from other field types during review.

**Examples:**

| Logical Name          | Status | Reason                                                   |
|-----------------------|--------|----------------------------------------------------------|
| `tmy_orderid`         | ✅ PASS | Lookup to `tmy_order`, ends with `id`                   |
| `tmy_contactid`       | ✅ PASS | Lookup to `contact`, ends with `id`                     |
| `tmy_parentaccountid` | ✅ PASS | Lookup to `account` (parent), ends with `id`            |
| `tmy_order`           | ❌ FAIL | Lookup field missing `id` suffix                        |
| `tmy_contact_lookup`  | ❌ FAIL | Lookup field not ending with `id`                       |
| `tmy_ref_contact`     | ❌ FAIL | Lookup field not ending with `id`                       |

**LLM Check Instruction:**  
When reviewing an attribute that is of type **Lookup**, check that its logical name ends in `id`. If it does not, flag a violation of **Lookup attribute logical names must end with the suffix `id`**. This rule applies **only to lookup-type attributes** — do not apply to text, number, date, or other attribute types.

---

## Violation Severity Levels

| Severity | Description                                                                 |
|----------|-----------------------------------------------------------------------------|
| ERROR    | Must be fixed before deployment. Applies to all enforced naming rules in this document. |
| WARNING  | Should be reviewed. Used for ambiguous plural detection edge cases. |

---

## LLM Review Output Format

Before listing naming-rule findings, include a mandatory detailed inventory section so reviewers can see exactly what changed.

### Mandatory Section 0: HUMAN-READABLE CHANGE SUMMARY (ALL COMPONENTS)

Start the output with a concise human-readable summary in this style:

```
- Added table <table logical name> - <table display name>
  | Attribute | Display Name | Data Type |
  | <attribute logical name> | <attribute display name> | <attribute data type> |

- Added app module <component logical name>
- Modified flow/process <component logical name>
- Deleted relationship <component logical name>
```

Use `Added`, `Updated`, or `Deleted` based on diff evidence.
If a changed table has no in-scope attribute changes, include the table line and write: `No in-scope attribute changes`.
If no component changes are found, write: `No component changes found.`

### Mandatory Section 1: CHANGE INVENTORY (ALL COMPONENTS)

Use this format:

```
## CHANGE INVENTORY

TABLE: <table logical name>
TABLE ACTION: <Created | Modified | Deleted>
EVIDENCE: <short diff-based evidence>

ATTRIBUTES:
- <attribute logical name> | ACTION: <Created | Modified | Deleted> | TYPE: <Lookup | Other> | LOOKUP TARGET: <target table or N/A>
- <attribute logical name> | ACTION: <Created | Modified | Deleted> | TYPE: <Lookup | Other> | LOOKUP TARGET: <target table or N/A>

COMPONENT: <component logical or path-derived name>
COMPONENT TYPE: <App Module | App Module Sitemap | Flow/Process | Relationship | Web Resource | Other Component>
COMPONENT ACTION: <Created | Modified | Deleted>
EVIDENCE: <short diff-based evidence>
```

If no component changes are found, output:

```
## CHANGE INVENTORY
No component changes were found in the diff.
```

### Mandatory Section 2: NAMING RULE EVALUATION MATRIX (CUSTOM TABLES/ATTRIBUTES ONLY)

For every custom table/custom attribute listed in CHANGE INVENTORY, include naming-rule evaluation:

```
## RULE EVALUATION MATRIX
COMPONENT: <logical name>
TYPE: <Table | Attribute — Lookup | Attribute — Other>
Logical names must be in lowercase: <PASS | ERROR>
Table logical names must use singular form: <PASS | ERROR | WARNING | N/A>
Lookup attribute logical names must end with the suffix `id`: <PASS | ERROR | N/A>
REASON: <short explanation>
SUGGESTED FIX: <logical name or N/A>
```

Use `N/A` when a rule does not apply (for example, singular-form rule on attributes, lookup-suffix rule on non-lookup attributes and tables).

Do not apply naming-rule checks to non-table/non-attribute components (apps, flows, relationships, web resources, and other components).

### Mandatory Section 3: FINDINGS (NAMING RULES)

When reviewing a set of changes, output findings in the following structure:

```
COMPONENT: <logical name>
TYPE: <Table | Attribute — Lookup | Attribute — Other>
RULE VIOLATED: <Logical names must be in lowercase | Table logical names must use singular form | Lookup attribute logical names must end with the suffix `id` | NONE>
SEVERITY: <ERROR | WARNING | PASS>
REASON: <one-line explanation>
SUGGESTED FIX: <corrected logical name, if applicable>
```

**Example output:**

```
COMPONENT: tmy_Orders
TYPE: Table
RULE VIOLATED: Logical names must be in lowercase; Table logical names must use singular form
SEVERITY: ERROR
REASON: Contains uppercase letter 'O'; entity name is plural.
SUGGESTED FIX: tmy_order

COMPONENT: tmy_contactid
TYPE: Attribute — Lookup
RULE VIOLATED: NONE
SEVERITY: PASS
REASON: Lowercase, lookup field correctly ends with 'id'.

COMPONENT: tmy_contact_ref
TYPE: Attribute — Lookup
RULE VIOLATED: Lookup attribute logical names must end with the suffix `id`
SEVERITY: ERROR
REASON: Lookup attribute does not end with 'id'.
SUGGESTED FIX: tmy_contactid
```

---

## Out of Scope (Future Rules)

The following are **not yet enforced** but are candidates for future rules:

- Publisher prefix validation (ensuring all custom components use the correct approved prefix)
- Choice (option set) naming conventions
- Relationship schema name conventions
- Attribute naming conventions by data type (e.g., boolean fields prefixed with `is` or `has`)
- Maximum logical name length

---

*Last updated: 2026-05-24 | Change summary scope: all component types | Naming-rule scope: custom tables and custom attributes only*