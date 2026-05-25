# simulation-domain-pack

## Purpose

The `simulation-domain-pack` is a tertiary-layer AI steering extension within kiro-suite,
located at `kiro-ext-pkgs/domains/simulation/`. It produces machine-readable `.ki` directive
files that steer AI agent behavior in simulation system contexts — it does not produce
application code.

The pack is **model-fidelity-first**: correctness of the simulation model takes precedence
over execution performance, entertainment output, or ML environment scaffolding. When an AI
agent loads this pack, it inherits all constraints and applies them when generating code,
schemas, or architectural decisions for simulation systems.

Target simulation types:

- **Physics simulations** — computational models of physical systems and phenomena
- **Traffic simulations** — transportation and mobility flow models
- **Economic simulations** — market, financial, and economic process models
- **System dynamics** — stock-and-flow feedback loop models of aggregate behavior
- **Agent-based simulations** — emergent behavior arising from individual agent rules
- **Discrete event simulations (DES)** — state changes occurring only at discrete event times (MVP simulation type)

---

## Scope

### In-Scope System Types

| System | Description |
|---|---|
| `physics_simulation` | Computational models of physical systems governed by declared physical laws and boundary conditions |
| `traffic_simulation` | Transportation and mobility flow models representing real-world movement patterns and infrastructure |
| `economic_simulation` | Market, financial, and economic process models with declared agent behaviors and market rules |
| `system_dynamics` | Stock-and-flow feedback loop models representing aggregate behavior over time |
| `agent_based_simulation` | Models where emergent behavior arises from individual Agent rules and local interactions |
| `discrete_event_simulation` | Models where state changes occur only at discrete Event times; MVP simulation type |

A system qualifies as in-scope when its **primary objective is fidelity to a real-world
process model**, not entertainment output or ML environment scaffolding.

### Out-of-Scope Concerns

The following domains are explicitly excluded. AI agents SHALL NOT apply this pack's
constraints to these domains:

| Domain | Explanation |
|---|---|
| `game_engine_rendering` | Primary objective is visual output and frame rate, not process model fidelity |
| `real_time_graphics` | Rendering pipelines optimized for perceptual quality, not physical accuracy |
| `game_ai_behavior_trees` | Behavior trees optimized for entertainment challenge, not real-world process modeling |
| `entertainment_physics_approximations` | Physics approximations tuned for feel and responsiveness, not fidelity |
| `ml_training_environments_without_explicit_model_fidelity_requirement` | RL environments without a declared real-world process model are out of scope |

---

## Inheritance Chain

The pack sits in a three-layer inheritance chain. Each layer may only add constraints —
it may never relax or redefine anything declared by a parent layer.

```text
kiro-core                                          (primary,    LAYER=primary)
    └── kiro-ext-pkgs                              (secondary,  LAYER=secondary)
            └── simulation-domain-pack             (tertiary,   LAYER=tertiary)
                    └── sub-domain packs (future)  (quaternary, LAYER=quaternary)
                        simulation/physics
                        simulation/traffic
                        simulation/economic
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
| `_index.ki` | P0 | Pack manifest, dependency declarations, load order, scope, priority registry, versioning | `LOAD_ORDER`, `FILE_REGISTRY`, `PRIORITY`, `SCOPE_IN`, `SCOPE_OUT`, `SEMVER` |
| `glossary.ki` | P0 | Domain-specific term aliases extending `kiro-core/glossary.ki`; time semantics; conflict resolution alias | `EXTENDS=kiro-core/glossary.ki`, `term=definition` aliases, `r01` |
| `entities.ki` | P0 | 13 canonical entities with ownership, fields, immutability, and cross-reference rules | `ENTITY`, `OWNER`, `REQUIRED_FIELDS`, `CROSS_REF_POLICY`, `APPEND_ONLY`, `IMMUTABLE_AFTER_TERMINAL` |
| `states.ki` | P0 | State machine definitions for 5 stateful simulation entities | `STATE_MACHINE`, `STATES`, `TERMINAL_STATES`, `TRANSITION` |
| `rules.ki` | P0 | Correctness, fidelity, reproducibility, numerical precision, time semantics, and scope boundary rules | `RULE=condition->action`, `AXIOM`, `REUSE`, `run_hash_computation` |
| `reliability.ki` | P1 | Checkpointing, run isolation, observation durability, cancellation, and lifecycle observability | `CHECKPOINT_THRESHOLD`, `LIFECYCLE_EVENT`, `OBSERVATION_WRITE_CONSTRAINT` |
| `workflows.ki` | P1 | 6 canonical simulation lifecycle workflow phases and step sequences | `WORKFLOW`, `STEP`, `EMITS_EVENT`, `CONSTRAINT` |
| `roles.ki` | P1 | Domain-scoped role extensions for 11 kiro-roles with forbidden actions and escalation triggers | `ROLE_EXTENSION`, `BASE`, `DOMAIN_FORBIDDEN`, `DOMAIN_ESC`, `SIGNOFF_REQUIRED` |
| `edge-cases.ki` | P2 | Numerical safety rules, model divergence and drift constraints, error signal format table | `RULE=failure_condition->remediation_action`, `ERROR_SIGNAL` |

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
3. kiro-ext-pkgs/domains/simulation/_index.ki
4. Individual pack files as declared in _index.ki
```

The `_index.ki` file enforces this via `LOAD_ORDER` and `REQUIRES` directives. Loading pack
files out of order or before their declared position will cause `_index.ki` to emit
`MISSING_DEP` and halt.

### What the Pack Does

The pack steers AI agent behavior — it does not produce application code. When loaded, it
constrains the agent to:

- Apply the model-fidelity-first principle: correctness over performance at all times
- Use the canonical 13-entity model with ownership boundaries and cross-reference rules
- Enforce valid state machine transitions for all stateful simulation entities
- Require explicit `Assumption` declarations before any `Simulation_Run` is initiated
- Enforce reproducibility via `Seed`, `run_hash`, and pinned model version
- Apply numerical precision, divergence detection, and NaN/infinity halt rules
- Follow the 6-phase simulation lifecycle workflow with event emission at every step
- Merge base kiro-roles definitions with domain-scoped forbidden actions and escalation triggers

### Missing Dependency Behavior

If `kiro-core` or `kiro-roles` is not loaded before this pack is activated, `_index.ki`
emits `MISSING_DEP` and halts. The agent must load the missing dependency and retry.

```
MISSING_DEP=kiro-core_required_before_pack_activation
MISSING_DEP=kiro-roles_required_before_pack_activation
```

### Conflict Signal Behavior

When the pack's directive parser encounters a directive key that also exists in kiro-core
or kiro-roles, it classifies the situation as a conflict and emits `CONFLICT_SIGNAL` rather
than silently applying either rule. The operator must resolve the conflict explicitly before
the pack is considered active.

```
CONFLICT_SIGNAL=<domain_file>:<key> conflicts with <parent_pack>/<file>:<key>
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

- Any state machine transition added or removed requires `arch` sign-off and a `MAJOR` bump.
- Any new entity added to `entities.ki` requires `tl` review and a `MINOR` bump.
- No P1 or P2 file may be merged before all files in the tier above it have passed review.
- A version bump always requires a changelog entry.
- When `kiro-core` version is bumped, this pack must be reviewed for compatibility.

---

## Extension Guide

Sub-domain packs extend this pack for specialized simulation verticals. They follow the same
override model used by this pack relative to kiro-core.

### Directory Structure

Sub-domain packs live under:

```
kiro-ext-pkgs/domains/simulation/<sub-domain>/
```

Examples:

- `kiro-ext-pkgs/domains/simulation/physics/`
- `kiro-ext-pkgs/domains/simulation/traffic/`
- `kiro-ext-pkgs/domains/simulation/economic/`

### Required Declarations

Every sub-domain pack `_index.ki` MUST declare:

```
PARENT=kiro-ext-pkgs/domains/simulation@<version>
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

### Deferred Complexity

The following items are deferred to post-MVP and are candidates for sub-domain pack
implementation in future versions. They MUST NOT be referenced in P0 or P1 files without
a `TBD` declaration:

| Deferred Item | Notes |
|---|---|
| `multi_agent_learning` | Reinforcement learning within simulation agents; requires explicit model fidelity requirement to qualify as in-scope |
| `continuous_physics_integration` | Continuous ODE/PDE solvers; deferred due to coupling to continuous math not covered by DES MVP rules |
| `real_time_co_simulation` | Synchronized execution across multiple simulation engines in real time; requires distributed coordination rules |
| `distributed_multi_node_simulation` | Multi-node execution with distributed state; requires distributed transaction and consensus rules beyond MVP scope |

---

## Deferred Complexity

Post-MVP items deferred from `VERSION=1.0.0`:

- **`multi_agent_learning`** — Reinforcement learning and adaptive agent behavior within
  simulation contexts. Requires explicit model fidelity requirement to remain in-scope.
  Candidate for a `simulation/agent-learning` sub-domain pack.

- **`continuous_physics_integration`** — Continuous ODE/PDE numerical solvers (e.g.,
  Runge-Kutta, finite element methods). Deferred due to coupling to continuous mathematics
  not covered by the DES-focused MVP rules. Candidate for a `simulation/physics` sub-domain pack.

- **`real_time_co_simulation`** — Synchronized execution across multiple heterogeneous
  simulation engines in real time (e.g., FMI/FMU co-simulation). Requires distributed
  coordination and synchronization rules beyond MVP scope.

- **`distributed_multi_node_simulation`** — Multi-node execution with distributed state
  management, partitioned State Vectors, and cross-node consistency guarantees. Requires
  distributed transaction and consensus rules. Candidate for a `simulation/distributed`
  sub-domain pack.
