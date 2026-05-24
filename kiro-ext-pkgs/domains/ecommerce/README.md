# ecommerce-domain-pack

[[Tiếng Việt] README.vi.md](README.vi.md)

## Purpose

The `ecommerce-domain-pack` is a tertiary-layer AI steering extension within kiro-suite,
located at `kiro-ext-pkgs/domains/ecommerce/`. It produces machine-readable `.ki` directive
files that steer AI agent behavior in ecommerce contexts — it does not produce application code.

The pack encodes ecommerce-specific business rules, entity models, state machines, workflow
constraints, security extensions, and role-scoped permissions. When an AI agent loads this
pack, it inherits all constraints and applies them when generating code, schemas, or
architectural decisions for ecommerce systems.

Target platforms:

- **B2C storefronts** — customer-facing shopping experiences
- **B2B procurement portals** — business buyer workflows and approval chains
- **Marketplace platforms** — multi-seller order splitting and fulfillment routing
- **Headless commerce backends** — API-first commerce services decoupled from presentation

---

## Scope

### In-Scope System Types

| System | Description |
|---|---|
| `storefront` | Customer-facing UI and session management |
| `cart_service` | Cart lifecycle, item management, inventory soft-checks |
| `order_service` | Order creation, lifecycle, discounts, coupons |
| `payment_service` | Payment intents, capture, refunds, fraud signals |
| `fulfillment_service` | Fulfillment units, warehouse notifications, shipment tracking |
| `catalog_service` | Products, variants, categories, inventory records |

### Out-of-Scope Concerns

The following systems are explicitly excluded. AI agents SHALL NOT apply this pack's
constraints to these domains:

- `ERP_integration`
- `accounting_ledger`
- `HR_systems`
- `logistics_carrier_internals`
- `ad_tech_platforms`

---

## Inheritance Chain

The pack sits in a three-layer inheritance chain. Each layer may only add constraints —
it may never relax or redefine anything declared by a parent layer.

```
kiro-core                                       (primary,   LAYER=primary)
    └── kiro-ext-pkgs                           (secondary, LAYER=secondary)
            └── domains/ecommerce-domain-pack   (tertiary,  LAYER=tertiary)
```

| Property | Value |
|---|---|
| `OVERRIDE_POLICY` | `kiro-core_always_wins` |
| `CONFLICT_RESOLUTION` | `escalate_per_glossary.ki:r01` |

`OVERRIDE_POLICY=kiro-core_always_wins` applies at all levels of the chain. This pack may
only add domain-specific constraints on top of what kiro-core and kiro-roles declare. It may
never relax, redefine, or override anything already declared in kiro-core or kiro-roles.

When a domain directive key matches a kiro-core directive key, the pack emits a
`CONFLICT_SIGNAL` and halts. The operator must resolve the conflict explicitly — no silent
resolution is permitted.

---

## File Index

All `.ki` files must be registered in `_index.ki` FILE_REGISTRY before they are considered
active. Files are organized into three priority tiers that define implementation and review order.

| File | Priority | Purpose | Key Directives |
|---|---|---|---|
| `_index.ki` | P0 | Pack manifest, dependency declarations, load order, priority registry | `LOAD_ORDER`, `FILE_REGISTRY`, `PRIORITY`, `SEMVER` |
| `glossary.ki` | P0 | Domain-specific term aliases extending `kiro-core/glossary.ki` | `EXTENDS=kiro-core/glossary.ki`, `term=definition` aliases |
| `entities.ki` | P0 | 18 canonical entities with ownership, fields, cross-reference rules | `ENTITY`, `OWNER`, `REQUIRED_FIELDS`, `CROSS_REF_POLICY` |
| `states.ki` | P0 | State machine definitions for 7 stateful entities | `STATE_MACHINE`, `STATES`, `TERMINAL_STATES`, `TRANSITION` |
| `rules.ki` | P0 | Business rule constraints: pricing, inventory, discounts, tax, order lifecycle | `RULE=condition->action`, `DISCOUNT_ORDER` |
| `security.ki` | P0 | PCI-DSS and GDPR extensions on top of `kiro-core/security.ki` | `EXTENDS=kiro-core/security.ki`, `PII_FIELDS`, `RULE=raw_card_data_*` |
| `reliability.ki` | P1 | Saga patterns, idempotency, retry policies, circuit breakers | `SAGA_STEP`, `COMPENSATE`, `BACKOFF`, `MAX_RETRIES` |
| `workflows.ki` | P1 | 6 canonical ecommerce workflow flows and step sequences | `WORKFLOW`, `STEP`, `EMITS_EVENT`, `CONSTRAINT` |
| `roles.ki` | P1 | Domain-scoped role extensions for 13 kiro-roles | `ROLE_EXTENSION`, `BASE`, `DOMAIN_FORBIDDEN`, `DOMAIN_ESC` |
| `edge-cases.ki` | P2 | Failure mode and edge case constraints | `RULE=failure_condition->remediation_action` |

**Tier rules:**

- P1 files SHALL NOT be merged before all P0 files have passed review.
- P2 files SHALL NOT be merged before all P1 files have passed review.
- A P0 file that references a P1 or P2 file not yet implemented SHALL declare the reference
  as `TBD=<description> TICKET=<ref>` rather than leaving it undefined.

---

## Usage

### Load Order

An AI agent consuming this pack MUST load dependencies in this exact order:

```
1. kiro-core          (all files)
2. kiro-roles         (all files)
3. kiro-ext-pkgs/domains/ecommerce/_index.ki
4. Individual pack files as declared in _index.ki
```

The `_index.ki` file enforces this via `LOAD_ORDER` and `REQUIRES` directives. Loading pack
files out of order or before their declared position will cause `_index.ki` to emit
`MISSING_DEP` and halt.

### What the Pack Does

The pack steers AI agent behavior — it does not produce application code. When loaded, it
constrains the agent to:

- Use the canonical entity model and ownership boundaries
- Enforce valid state machine transitions
- Apply business rules for pricing, inventory, discounts, and tax
- Extend kiro-core security rules with PCI-DSS and GDPR constraints
- Follow saga patterns and idempotency requirements for distributed operations
- Emit domain events at every workflow step

### Missing Dependency Behavior

If `kiro-core` or `kiro-roles` is not loaded before this pack is activated, `_index.ki`
emits `MISSING_DEP` and halts. The agent must load the missing dependency and retry.

```
MISSING_DEP=kiro-core_required_before_pack_activation
MISSING_DEP=kiro-roles_required_before_pack_activation
```

### Conflict Signal Behavior

When the pack's directive parser encounters a directive key that also exists in kiro-core,
it classifies the situation as a conflict and emits `CONFLICT_SIGNAL` rather than silently
applying either rule. The operator must resolve the conflict explicitly before the pack is
considered active.

```
CONFLICT_SIGNAL=<domain_file>:<key> conflicts with kiro-core/<file>:<key>
```

### File Registry Requirement

A `.ki` file must be registered in `_index.ki` FILE_REGISTRY before it is considered active.
Unregistered files are ignored by the pack loader, even if they are present on disk.

```
FILE_REGISTRY=<filename>.ki  PRIORITY=<tier>  STATUS=active
```

---

## Versioning

```
SEMVER=major.minor.patch
VERSION=1.0.0
```

| Change Type | Version Bump | Sign-off Required |
|---|---|---|
| Breaking change to entity definitions or state machines | `MAJOR` | `arch` sign-off |
| New entities or rules added (non-breaking) | `MINOR` | `tl` review |
| Clarifications or documentation fixes | `PATCH` | — |

Additional rules:

- Any change to `security.ki` requires `sec` role sign-off before merge, regardless of
  version bump type.
- No P1 or P2 file may be merged before all files in the tier above it have passed review.
- A version bump always requires a changelog entry.
- When `kiro-core` version is bumped, this pack must be reviewed for compatibility.

---

## Extension Guide

Sub-domain packs extend this pack for specialized ecommerce verticals. They follow the same
override model used by this pack relative to kiro-core.

### Directory Structure

Sub-domain packs live under:

```
kiro-ext-pkgs/domains/ecommerce/<sub-domain>/
```

Examples:

- `kiro-ext-pkgs/domains/ecommerce/marketplace/`
- `kiro-ext-pkgs/domains/ecommerce/subscription/`
- `kiro-ext-pkgs/domains/ecommerce/b2b/`

### Required Declarations

Every sub-domain pack `_index.ki` MUST declare:

```
PARENT=kiro-ext-pkgs/domains/ecommerce@<version>
LAYER=quaternary
OVERRIDE_POLICY=parent_always_wins
```

Child packs follow the same META block conventions and file registration requirements as
this pack. Every `.ki` file in a child pack must be registered in the child pack's own
`_index.ki` FILE_REGISTRY before it is considered active.

### Override Policy

`OVERRIDE_POLICY=parent_always_wins` applies: this pack's rules take precedence over child
pack rules. Sub-domain packs may only add constraints; they may never relax a parent rule.

```
RULE=subdomain_pack_conflicts_with_parent_pack_rule->apply_parent_always_wins_and_surface_conflict
RULE=subdomain_pack_relaxes_parent_pack_rule->block_not_permitted
```

This mirrors the same model used by this pack relative to kiro-core.
