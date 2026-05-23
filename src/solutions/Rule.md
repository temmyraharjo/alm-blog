# Power Platform Naming Convention Rules

## Overview

This document defines naming conventions for **custom tables and attributes** in Microsoft Power Platform (Dataverse). These rules are enforced by LLM-based review and apply **only to custom components** — i.e., those with a publisher prefix (e.g., `tmy_`). Out-of-the-box system tables and attributes (e.g., `account`, `contact`, `createdon`) are exempt.

---

## Scope

| Component Type     | In Scope | Notes                                      |
|--------------------|----------|--------------------------------------------|
| Custom tables      | ✅       | Must carry a publisher prefix (e.g., `tmy_`) |
| Custom attributes  | ✅       | Must carry a publisher prefix (e.g., `tmy_`) |
| System tables      | ❌       | Exempt — do not apply these rules           |
| System attributes  | ❌       | Exempt — do not apply these rules           |
| Choice (option set)| ❌       | Not yet in scope                            |
| Relationships      | ❌       | Not yet in scope                            |

---

## Rules

---

### RULE-001 — Logical names must be in lowercase

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
Scan the logical name character by character. If any character falls outside `[a-z0-9_]`, flag as **RULE-001 VIOLATION**.

---

### RULE-002 — Table logical names must use singular form

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
Extract the entity portion of the name (everything after the first `_`). Check whether it ends with a common plural suffix (`s`, `es`, `ies`, `ves`). If yes, flag as a **probable RULE-002 VIOLATION** and suggest the singular form. Note: some words are legitimately non-plural despite ending in `s` (e.g., `status`, `address`, `process`) — use context and common English to distinguish.

**Known non-violations (words ending in `s` that are singular):**

- `status`, `address`, `process`, `progress`, `access`, `class`, `canvas`

---

### RULE-003 — Lookup attribute logical names must end with the suffix `id`

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
When reviewing an attribute that is of type **Lookup**, check that its logical name ends in `id`. If it does not, flag as **RULE-003 VIOLATION**. This rule applies **only to lookup-type attributes** — do not apply to text, number, date, or other attribute types.

---

## Violation Severity Levels

| Severity | Description                                                                 |
|----------|-----------------------------------------------------------------------------|
| ERROR    | Must be fixed before deployment. Applies to RULE-001, RULE-002, RULE-003.  |
| WARNING  | Should be reviewed. Used for ambiguous plural detection (RULE-002 edge cases). |

---

## LLM Review Output Format

When reviewing a set of changes, output findings in the following structure:

```
COMPONENT: <logical name>
TYPE: <Table | Attribute — Lookup | Attribute — Other>
RULE VIOLATED: <RULE-001 | RULE-002 | RULE-003 | NONE>
SEVERITY: <ERROR | WARNING | PASS>
REASON: <one-line explanation>
SUGGESTED FIX: <corrected logical name, if applicable>
```

**Example output:**

```
COMPONENT: tmy_Orders
TYPE: Table
RULE VIOLATED: RULE-001, RULE-002
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
RULE VIOLATED: RULE-003
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

*Last updated: 2026-05-23 | Scope: Custom tables and attributes only*