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

## Required Review Procedure (Follow In Order)

Use this exact order for every review:

1. Read all provided evidence inputs (`changes.status`, `changes.diff`, unpacked XML/component files, and this rule file).
2. Build the changed-component list using path mapping rules in this document.
3. Classify each component action as `Added`, `Modified`, or `Deleted` from evidence.
4. Produce human-readable summary lines (Section 0), applying compact mode when thresholds are met.
5. Evaluate naming rules only for in-scope custom tables and custom attributes (Section 1 and Section 2).
7. If no in-scope items exist for a section, explicitly output the required empty-state sentence.

Do not skip steps. Do not reorder steps.

### Evidence Precedence

When evidence sources disagree, use this precedence:

1. `changes.status` for Added/Modified/Deleted file action detection.
2. `changes.diff` for change evidence details.
3. Unpacked XML/component files for display names, attribute details, flow trigger/actions, and descriptions.

### Hard Constraints (No Ambiguity)

- Never invent component names, physical names, flow names, or attributes.
- Never auto-correct or normalize physical names. Keep exact names from evidence.
- If evidence is insufficient, explicitly write `not available from current evidence`.
- Use only component names that appear in provided evidence inputs.
- Do not apply naming-rule checks outside custom tables/custom attributes.

---

## Workflow Prompt Directives

Apply these directives exactly when generating the review output:

- For change summary, include all component types defined in this rules document.
- Classify component type and action from changed file paths in `changes.status` first (per path mapping in this document); do not infer changed component types from name mentions in XML bodies.
- Only report relationship component changes when corresponding changed paths map to relationship files (`Other/Relationships*.xml` or `Other/Relationships/*.xml`, including solution-prefixed variants).
- If only table-related files changed under `Entities/<table>/...`, report those as table/table-metadata changes and do not emit relationship rows unless relationship-mapped files are changed.
- For naming-rule evaluation, only evaluate components this rules document marks as in scope.
- For naming rules output, print only WARNING/ERROR violations; suppress PASS component entries.
- If no components or rule findings are in scope, explicitly output the required empty-state sentence(s) for the affected section.

---

## Component Change Classification Rules

Use these rules to determine **component type** and **change action** in the human summary.

### Change action mapping

- `Added`: file path appears as untracked/added in change evidence.
- `Modified`: file path appears as modified in change evidence.
- `Deleted`: file path appears as deleted in change evidence.

### Component type mapping by unpacked path (best effort)

- `src/solutions/Entities/<name>/Entity.xml` or `src/solutions/<solution>/Entities/<name>/Entity.xml` -> Table
- `src/solutions/Entities/<name>/...` or `src/solutions/<solution>/Entities/<name>/...` -> Table-related metadata (forms/views/ribbon)
- `src/solutions/AppModules/<name>/AppModule.xml` or `src/solutions/<solution>/AppModules/<name>/AppModule.xml` -> App Module
- `src/solutions/AppModuleSiteMaps/<name>/AppModuleSiteMap.xml` or `src/solutions/<solution>/AppModuleSiteMaps/<name>/AppModuleSiteMap.xml` -> App Module Sitemap
- `src/solutions/Other/Relationships*.xml` and `src/solutions/Other/Relationships/*.xml` (or with `<solution>/Other/...`) -> Relationship
- `src/solutions/Workflows/*.xml` (or with `<solution>/Workflows/...`) -> Power Automate Flow / Process
- `src/solutions/WebResources/**` (or with `<solution>/WebResources/...`) -> Web Resource
- everything else -> Other Component

### Table and attribute extraction rules

- Table physical name: use folder name under `Entities/<table>`.
- Table display name: use DisplayName from `Entity.xml` when available, otherwise derive from physical name.
- Attribute rows: extract custom attributes from `Entity.xml` for the same publisher prefix as the table.
- Attribute data type: use attribute type metadata from XML; if unavailable, report `Unknown`.

---

## Naming Rules

---

### PhysicalName must be in lowercase

**Applies to:** Table physical names, attribute physical names

**Definition:**  
The full physical name — including the publisher prefix and the entity/attribute portion — must contain **no uppercase letters**. Every character must be either a lowercase letter (`a–z`), a digit (`0–9`), or an underscore (`_`).

**Rationale:**  
Dataverse stores physical names in lowercase. Inconsistent casing in schema names before saving causes confusion during solution comparisons, ALM pipelines, and code references.

**Examples:**

| PhysicalName      | Status | Reason                              |
|--------------------|--------|-------------------------------------|
| `tmy_order`        | ✅ PASS | All lowercase                       |
| `tmy_orderdetail`  | ✅ PASS | All lowercase                       |
| `tmy_Order`        | ❌ FAIL | Contains uppercase `O`              |
| `tmy_OrderDetail`  | ❌ FAIL | Contains uppercase `O` and `D`      |
| `TMY_order`        | ❌ FAIL | Prefix contains uppercase letters   |

**LLM Check Instruction:**  
Scan the physical name character by character. If any character falls outside `[a-z0-9_]`, flag a violation of **PhysicalName must be in lowercase**.

---

### Table physical names must use singular form

**Applies to:** Table physical names only (not attributes)

**Definition:**  
The entity portion of a custom table's physical name must represent a **single record concept**, not a collection. Use the singular form of the noun.

**Rationale:**  
Each table record represents one instance of the entity. Plural naming implies a collection and conflicts with standard Dataverse conventions (e.g., `account`, `contact`, `invoice`).

**Examples:**

| PhysicalName       | Status | Reason                                    |
|---------------------|--------|-------------------------------------------|
| `tmy_order`         | ✅ PASS | Singular                                  |
| `tmy_orderdetail`   | ✅ PASS | Singular                                  |
| `tmy_product`       | ✅ PASS | Singular                                  |
| `tmy_orders`        | ❌ FAIL | Plural — should be `tmy_order`            |
| `tmy_orderdetails`  | ❌ FAIL | Plural — should be `tmy_orderdetail`      |
| `tmy_deliveries`    | ❌ FAIL | Plural — should be `tmy_delivery`         |

**Common plural suffixes to flag:** `-s`, `-es`, `-ies` (converted from `-y`), `-ves`

**LLM Check Instruction:**  
Extract the entity portion of the name (everything after the first `_`). Check whether it ends with a common plural suffix (`s`, `es`, `ies`, `ves`). If yes, flag as a probable violation of **Table physical names must use singular form** and suggest the singular form. Note: some words are legitimately non-plural despite ending in `s` (e.g., `status`, `address`, `process`) — use context and common English to distinguish.

**Known non-violations (words ending in `s` that are singular):**

- `status`, `address`, `process`, `progress`, `access`, `class`, `canvas`

---

### Lookup attribute physical names must end with the suffix `id`

**Applies to:** Custom lookup (Many-to-One relationship) attributes only

**Definition:**  
Any attribute that stores a reference to another table (i.e., a lookup field) must have a physical name ending in `id`. The portion before `id` must meaningfully represent the target table or relationship purpose.

**Rationale:**  
Dataverse appends `id` to lookup physical names automatically when the schema name follows conventions. Enforcing this rule ensures schema names are set correctly before deployment and that lookup fields are immediately distinguishable from other field types during review.

**Examples:**

| PhysicalName         | Status | Reason                                                   |
|-----------------------|--------|----------------------------------------------------------|
| `tmy_orderid`         | ✅ PASS | Lookup to `tmy_order`, ends with `id`                   |
| `tmy_contactid`       | ✅ PASS | Lookup to `contact`, ends with `id`                     |
| `tmy_parentaccountid` | ✅ PASS | Lookup to `account` (parent), ends with `id`            |
| `tmy_order`           | ❌ FAIL | Lookup field missing `id` suffix                        |
| `tmy_contact_lookup`  | ❌ FAIL | Lookup field not ending with `id`                       |
| `tmy_ref_contact`     | ❌ FAIL | Lookup field not ending with `id`                       |

**LLM Check Instruction:**  
When reviewing an attribute that is of type **Lookup**, check that its physical name ends in `id`. If it does not, flag a violation of **Lookup attribute physical names must end with the suffix `id`**. This rule applies **only to lookup-type attributes** — do not apply to text, number, date, or other attribute types.

---

## Violation Severity Levels

| Severity | Description                                                                 |
|----------|-----------------------------------------------------------------------------|
| ERROR    | Must be fixed before deployment. Applies to all enforced naming rules in this document. |
| WARNING  | Should be reviewed. Used for ambiguous plural detection edge cases. |

---

## LLM Review Output Format

Section title format is strict:

- Use markdown headings for section titles (for example, `## RULE EVALUATION MATRIX`, `## FINDINGS (NAMING RULES)`).
- Do not render section titles as numbered list items (for example, `1. RULE EVALUATION MATRIX`, `2. FINDINGS (NAMING RULES)`).
- Do not prefix mandatory section titles with bullets, numbering, or other list markers.

Start Section 0 with a compact summary table (no sentence-style intro lines).

Use this exact header order:

```
| Action | Display Name | PhysicalName |
|---|---|---|
| Added | Order Detail | tmy_oderdetail |
| Added | Order | tmy_order |
| Added | Blog App | tmy_blogapp |
```

Rules for this table:

- Prioritize display name first, then physical name.
- Use `Added`, `Modified`, or `Deleted` in `Action`.
- Include all in-scope changed components (tables, apps, flows/processes, relationships, web resources, others) as separate rows.
- If display name is unavailable, derive a readable display name from physical name/path.
- If physical name is unavailable, use best-effort path-derived identifier.
- For relationship components, format `Display Name` as `Relationship: <logical name>`.
- If there are no component changes, output one row:
  - `| None | No component changes found | N/A |`

### Markdown Table Formatting Requirements

Use strict markdown table formatting for all tables in the output:

- Use this exact header in Section 0:
  | Action | Display Name | PhysicalName |
  |---|---|---|
  | <Action> | <Display Name> | <PhysicalName> |
- Use `Added`, `Modified`, or `Deleted` in `Action`.
- Do not pad cells with alignment spaces for visual width.
- Do not emit tabs in table rows.
- Trim leading/trailing spaces inside each cell value.
- For relationship rows, do not use file names such as `Relationships.xml` as `Display Name` when a logical name is available from evidence.

### Large Change Set Summary Mode

When the number of changed components is large, prefer compact summary output.

Use compact mode when either of these is true:

- 10 or more changed components overall, or
- 5 or more changed tables.

In compact mode:

- Summarize at table level (physical names) for added/updated/deleted tables.
- Do not expand to attribute-level details unless a specific naming-rule finding requires it.
- If available from XML, include table display name in short form: `<physical name> (<display name>)`.
- Use short grouped lines such as:
  - `Added tables: tmy_order (Order), tmy_oderdetail (Order Detail), ...`
  - `Updated tables: ...`
  - `Deleted tables: ...`
  - `Added flows/processes: Flow A, Flow B, ...`
  - `Updated flows/processes: ...`
  - `Deleted flows/processes: ...`

In compact mode, attribute-level rows are optional unless needed to explain a naming-rule finding.

For flows/processes, include best-effort high-level step result summaries when available from workflow metadata.
Use concise lines like:

- `Flow <name>: trigger <trigger>; then <action 1>; then <action 2>; result <outcome summary>.`

Do not invent step names. If step details are not available in evidence, write:

- `Flow <name>: step-level details not available from current unpacked evidence.`

After the compact summary table, continue with concise component-level lines in this style:

```
- Added table <table physical name> - <table display name>
  | Attribute | Display Name | Data Type |
  |---|---|---|
  | <attribute physical name> | <attribute display name> | <attribute data type> |

- Added app module <component physical name>
- Modified flow/process <component physical name>
- Deleted relationship <component physical name>
```

Use `Added`, `Updated`, or `Deleted` based on diff evidence.
If a changed table has no in-scope attribute changes, include the table line and write: `No in-scope attribute changes`.
If no component changes are found, write: `No component changes found.`

### Mandatory Naming Section: RULE EVALUATION (NAMING RULES)

Use a single merged naming section only.
Do not output separate sections named `RULE EVALUATION MATRIX` and `FINDINGS (NAMING RULES)`.
Include only components with WARNING/ERROR violations. Do not print PASS-only entries.

If no naming-rule violations exist, output only:

```
## RULE EVALUATION (NAMING RULES)
No naming-rule violations found.
```

When violations exist, output each finding in this structure:

```
## RULE EVALUATION (NAMING RULES)
COMPONENT: <physical name>
TYPE: <Table | Attribute — Lookup | Attribute — Other>
RULE VIOLATED: <PhysicalName must be in lowercase | Table physical names must use singular form | Lookup attribute physical names must end with the suffix `id`>
SEVERITY: <ERROR | WARNING>
REASON: <one-line explanation>
SUGGESTED FIX: <corrected logical name, if applicable>
```

Use `N/A` only when explicitly needed inside the reason text.

Do not apply naming-rule checks to non-table/non-attribute components (apps, flows, relationships, web resources, and other components).

**Example output:**

```
COMPONENT: tmy_Orders
TYPE: Table
RULE VIOLATED: PhysicalName must be in lowercase; Table physical names must use singular form
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
RULE VIOLATED: Lookup attribute physical names must end with the suffix `id`
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