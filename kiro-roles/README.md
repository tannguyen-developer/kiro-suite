# 👥 Kiro Roles (`kiro-roles`)

[[Tiếng Việt] README.vi.md](README.vi.md)

> `kiro-roles` is the **Role Layer** of the `kiro-suite` ecosystem. It defines the complete taxonomy of engineering roles — their responsibilities, permissions, forbidden actions, and execution context for both human actors and AI agents. All role definitions inherit from and are constrained by `kiro-core`. No role may override, weaken, or contradict any `kiro-core` rule.

---

## 📌 Overview

The purpose of `kiro-roles` is to:

- **Define who does what** — each role has an explicit boundary, a set of permissions, and a list of forbidden actions.
- **Constrain AI agent behavior** — when an AI agent operates under a declared role, it inherits that role's full definition including all restrictions.
- **Enable `kiro-cli` role-scoped loading** — each role is a standalone `.ki` file that can be loaded independently without parsing the entire module.
- **Eliminate role ambiguity** — no two roles share the same decision authority in the same domain.

| Property | Value |
| :--- | :--- |
| **Layer** | Secondary — inherits from `kiro-core` |
| **Format** | `.ki` — Token-optimized key-value directive files |
| **Mode** | `machine_first` — Optimized for AI/LLM context parsing |
| **Permission model** | `deny_by_default_allow_by_explicit_grant` |
| **Conflict resolution** | `kiro-core` always wins; ambiguity → `r01` (ask) |
| **Versioning** | SemVer — pinned to `kiro-core` |

---

## 🗂 File Structure

```text
kiro-roles/
├── _index.ki          ← Manifest, role registry, system model, interaction rules
├── roles.ki           ← Monolithic reference file (all roles in one)
└── roles/
    ├── pm.ki          ├── po.ki          ├── ba.ki          ├── sm.ki
    ├── swe.ki         ├── fe_e.ki        ├── be_e.ki
    ├── tl.ki          ├── arch.ki
    ├── de.ki          ├── ml_e.ki        ├── ai_e.ki        ├── prompt_e.ki
    ├── qa.ki          ├── gate_k.ki
    ├── devops.ki      ├── sre.ki
    ├── sec.ki         ├── aud.ki
    ├── tech_w.ki      └── rel_m.ki
```

**`_index.ki`** is the entry point for `kiro-cli`. It contains:
- The role registry (`ROLE=<alias> FILE=roles/<alias>.ki CAT=<id>`)
- The role system model (RBAC axioms, permission model)
- Role categories and their members
- Role interaction rules
- AI agent execution context rules
- Conflict resolution rules
- Versioning

**Individual role files** (`roles/*.ki`) are self-contained — each file can be loaded in isolation by `kiro-cli` without reading any other file. Every role file includes a `META` header with `ROLE`, `ALIAS`, `PARENT`, `INDEX`, and `CATEGORY` fields.

---

## 🏗 Role System Model

`kiro-roles` uses **RBAC** (Role-Based Access Control) extended with execution context per role.

| Concept | Value |
| :--- | :--- |
| **Model** | `rbac` + responsibility and execution context |
| **Role unit** | Atomic named actor with a defined boundary |
| **Role scope** | Applies to human agents and AI agents equally |
| **Permission model** | `deny_by_default` — explicit grant required |
| **Permission source** | Inherited from `security.ki:SEC_CORE` + `principles.ki:SEC=least_privilege` |

**Four immutable axioms:**

1. No role may override `kiro-core` constraints.
2. No role may grant permissions beyond its own boundary.
3. Ambiguity in role boundary → apply `glossary.ki:r01` (ask first).
4. Role conflict → apply `principles.ki` priority order P01–P10.

---

## 🗃 Role Categories

| # | Category | Alias | Members (defined) |
| :- | :--- | :--- | :--- |
| 01 | Strategy & Requirements | `CAT_01` | `pm`, `po`, `ba`, `sm`, `agile_coach`, `ux_r` |
| 02 | Design & Experience | `CAT_02` | `pd`, `uxui_e`, `ux_e`, `ui_e`, `ui_d`, `anim_d` |
| 03 | Core Engineering | `CAT_03` | `swe`, `fe_e`, `be_e`, `fs_e`, `mob_e`, `ios_e`, `and_e`, `desktop_e`, `web3_e`, `game_e`, `embed_e` |
| 04 | Architecture & Leadership | `CAT_04` | `arch`, `sol_arch`, `ent_arch`, `cloud_arch`, `tl`, `em` |
| 05 | Data / AI / ML | `CAT_05` | `de`, `ds`, `da`, `ml_e`, `ai_e`, `prompt_e`, `db_a`, `nlp_e` |
| 06 | Quality & Testing | `CAT_06` | `qa`, `qc`, `sdet`, `auto_qa`, `man_qa`, `perf_qa`, `rvw`, `gate_k` |
| 07 | Infrastructure & Operations | `CAT_07` | `devops`, `sre`, `pe`, `sys_a`, `net_e`, `cloud_e`, `noc` |
| 08 | Security & Compliance | `CAT_08` | `sec`, `app_sec`, `cloud_sec`, `pen_t`, `aud`, `soc`, `iam_e` |
| 09 | Specialized Support | `CAT_09` | `rel_m`, `tech_w`, `dev_rel`, `loc_e`, `build_e`, `finops` |

> Roles with a `.ki` file in `roles/` are **fully defined**. Remaining members in each category are declared in `_index.ki` and reserved for future definition.

---

## 📖 Role Reference

Each role follows this strict template:

```
purpose=
responsibilities=
inputs=
outputs=
permissions=
forbidden_actions=
required_core_constraints:
  principles.ki=
  security.ki=
  workflow.ki=
  coding.ki=
execution_tags=
escalation_triggers=
```

---

### CAT_01 — Strategy & Requirements

#### `pm` — Product Manager [`roles/pm.ki`](roles/pm.ki)

- **Purpose:** Define product vision, prioritize backlog, align stakeholders.
- **Key permissions:** Create and prioritize tickets, approve scope changes, define done state.
- **Key forbidden actions:** Bypass security review, approve breaking changes without eng sign-off, skip AC definition.
- **Core constraints:** `principles.ki:REQ,VALUE,DECIDE,COMM,PLAN` · `security.ki:DATA_SEC=minimize_pii` · `workflow.ki:TASK=no_ticket_no_code`
- **Execution tags:** `out_plan`, `ask`, `r01`, `r04`
- **Escalation:** Legal/compliance risk · Financial impact · Customer trust risk

---

#### `po` — Product Owner [`roles/po.ki`](roles/po.ki)

- **Purpose:** Represent business value in agile team, own sprint backlog.
- **Key permissions:** Accept or reject sprint deliverables, reprioritize sprint backlog.
- **Key forbidden actions:** Override security constraints, approve prod deployments without gate, skip definition of done.
- **Core constraints:** `principles.ki:REQ,VALUE,PLAN` · `security.ki:SEC_CORE=deny_by_default` · `workflow.ki:TASK=definition_of_done`
- **Execution tags:** `out_plan`, `ask`, `r01`
- **Escalation:** Customer trust risk · Financial impact

---

#### `ba` — Business Analyst [`roles/ba.ki`](roles/ba.ki)

- **Purpose:** Translate business needs into structured requirements.
- **Key permissions:** Create req documents, request clarification from any role, flag contradictory requirements.
- **Key forbidden actions:** Make architecture decisions, approve code changes, bypass req review.
- **Core constraints:** `principles.ki:REQ,TRUTH,COMM` · `security.ki:DATA_SEC=classify_data_sensitivity` · `workflow.ki:TASK=definition_of_ready`
- **Execution tags:** `out_plan`, `ask`, `r01`, `r04`
- **Escalation:** Legal/compliance risk · Customer trust risk

---

#### `sm` — Scrum Master [`roles/sm.ki`](roles/sm.ki)

- **Purpose:** Facilitate agile ceremonies, remove blockers, protect team focus.
- **Key permissions:** Escalate blockers, facilitate process changes, flag workflow violations.
- **Key forbidden actions:** Make product decisions, override technical decisions, approve code.
- **Core constraints:** `principles.ki:COMM,PLAN,CHANGE` · `workflow.ki:TASK,STATUS,REVIEW`
- **Execution tags:** `out_steps`, `ask`, `r01`
- **Escalation:** Blocker unresolved > 2 days · Team velocity drop > 30%

---

### CAT_03 — Core Engineering

#### `swe` — Software Engineer [`roles/swe.ki`](roles/swe.ki)

- **Purpose:** Design, implement, test, and maintain software systems.
- **Key permissions:** Write and merge code via PR, create branches, run CI pipelines, propose refactors.
- **Key forbidden actions:** Commit directly to main, skip tests on business logic, hardcode secrets, swallow errors silently.
- **Core constraints:** `principles.ki:P01–P10,CORRECT,MAINT,SEC,REL,TEST` · `security.ki:SECRET,IO_SEC,IAM` · `workflow.ki:VCS,COMMIT,PR,CI` · `coding.ki:FUNC,FLOW,STATE,ERR,MOD,ASYNC`
- **Execution tags:** `out_code`, `r03`, `r05`, `r09`, `r10`
- **Escalation:** Security breach signals · Possible data loss · Production outage

---

#### `fe_e` — Front-End Engineer [`roles/fe_e.ki`](roles/fe_e.ki)

- **Purpose:** Implement user-facing interfaces with correctness, performance, and accessibility.
- **Key permissions:** Write frontend code, propose UX improvements, request API changes via ticket.
- **Key forbidden actions:** Store secrets in frontend code, bypass a11y requirements, skip CSR/SSR decision review.
- **Core constraints:** `principles.ki:UX,CORRECT,MAINT,PERF` · `security.ki:IO_SEC=encode_data_on_output,SECRET=never_in_frontend` · `coding.ki:FUNC,FLOW,STATE,ASYNC`
- **Execution tags:** `out_code`, `r05`, `r10`
- **Escalation:** Customer trust risk · Critical a11y violation

---

#### `be_e` — Back-End Engineer [`roles/be_e.ki`](roles/be_e.ki)

- **Purpose:** Implement server-side logic, APIs, data access, and integrations.
- **Key permissions:** Write backend code, design DB queries, propose schema changes, configure service integrations.
- **Key forbidden actions:** Skip input validation, use dynamic SQL concatenation, expose internal errors to client, hardcode secrets.
- **Core constraints:** `principles.ki:P01–P10,CORRECT,SEC,REL` · `security.ki:IO_SEC,SECRET,IAM,DATA_SEC,AUDIT` · `coding.ki:FUNC,FLOW,ERR,MOD,ASYNC,DATA`
- **Execution tags:** `out_code`, `r03`, `r05`, `r08`, `r10`
- **Escalation:** Security breach signals · Possible data loss · Production outage

---

### CAT_04 — Architecture & Leadership

#### `tl` — Tech Lead [`roles/tl.ki`](roles/tl.ki)

- **Purpose:** Own technical direction of team, ensure quality and delivery.
- **Key permissions:** Approve architecture changes, merge critical PRs, escalate to arch, define team standards.
- **Key forbidden actions:** Override `kiro-core` constraints, approve security bypasses, merge without CI pass.
- **Core constraints:** `principles.ki:P01–P10,DECIDE,PLAN,CHANGE,REVIEW` · `security.ki:SEC_CORE,IAM,AUDIT` · `workflow.ki:PR,REVIEW,CI,RELEASE`
- **Execution tags:** `out_plan`, `r02`, `r04`, `r05`, `r06`, `r10`
- **Escalation:** Security breach signals · Production outage · Financial impact

---

#### `arch` — Software Architect [`roles/arch.ki`](roles/arch.ki)

- **Purpose:** Define system architecture, ensure scalability, security, and alignment with `kiro-core`.
- **Key permissions:** Approve major architecture changes, define bounded contexts, mandate patterns.
- **Key forbidden actions:** Approve circular dependencies, allow shared database integration, bypass zero-trust model.
- **Core constraints:** `principles.ki:DECIDE,COMPLEXITY,MAINT,CHANGE,DEP` · `security.ki:SEC_ARCH,IAM,DATA_SEC` · `coding.ki:MOD=depend_on_abstractions`
- **Execution tags:** `out_plan`, `r02`, `r04`, `r05`, `r09`
- **Escalation:** Legal/compliance risk · Financial impact · Irreversible operation

---

### CAT_05 — Data / AI / ML

#### `de` — Data Engineer [`roles/de.ki`](roles/de.ki)

- **Purpose:** Build and maintain data pipelines, warehouses, and data infrastructure.
- **Key permissions:** Create and modify data pipelines, propose schema changes, access non-prod data.
- **Key forbidden actions:** Access prod PII without approval, skip backup before migration, use unvalidated data in prod.
- **Core constraints:** `principles.ki:CORRECT,REL,PERF,SEC` · `security.ki:DATA_SEC,SECRET,AUDIT` · `coding.ki:FUNC,ERR,DATA`
- **Execution tags:** `out_code`, `r07`, `r08`, `r10`
- **Escalation:** Possible data loss · Financial impact

---

#### `ml_e` — Machine Learning Engineer [`roles/ml_e.ki`](roles/ml_e.ki)

- **Purpose:** Design, train, evaluate, and deploy machine learning models.
- **Key permissions:** Access training datasets, deploy models to staging, propose feature changes.
- **Key forbidden actions:** Deploy unvalidated models to prod, use PII without consent, skip bias evaluation.
- **Core constraints:** `principles.ki:TRUTH,CORRECT,SEC,PERF` · `security.ki:DATA_SEC=minimize_pii,AUDIT` · `coding.ki:FUNC,ERR,DATA`
- **Execution tags:** `out_code`, `r07`, `r08`, `r10`
- **Escalation:** Customer trust risk · Legal/compliance risk · Model bias detected

---

#### `ai_e` — AI Engineer [`roles/ai_e.ki`](roles/ai_e.ki)

- **Purpose:** Design and implement AI-powered systems, integrations, and agents.
- **Key permissions:** Access LLM APIs, deploy AI features to staging, propose model changes.
- **Key forbidden actions:** Deploy AI without output validation, skip safety evaluation, expose system prompts to users, use PII in prompts without consent.
- **Core constraints:** `principles.ki:TRUTH,CORRECT,SEC,REL` · `security.ki:SECRET,IO_SEC,DATA_SEC,AUDIT` · `coding.ki:FUNC,ERR,ASYNC`
- **Execution tags:** `out_code`, `r05`, `r09`, `r10`
- **Escalation:** Customer trust risk · Legal/compliance risk · Security breach signals

---

#### `prompt_e` — Prompt Engineer [`roles/prompt_e.ki`](roles/prompt_e.ki)

- **Purpose:** Design, evaluate, and optimize prompts for AI systems.
- **Key permissions:** Create and version prompts, run evaluations, propose model changes.
- **Key forbidden actions:** Deploy prompts without safety evaluation, expose system prompts publicly, skip versioning.
- **Core constraints:** `principles.ki:TRUTH,CORRECT,SEC` · `security.ki:SECRET,IO_SEC` · `coding.ki:DOC=document_public_apis`
- **Execution tags:** `out_code`, `r05`, `r10`
- **Escalation:** Customer trust risk · Safety failure detected

---

### CAT_06 — Quality & Testing

#### `qa` — Quality Assurance [`roles/qa.ki`](roles/qa.ki)

- **Purpose:** Verify system behavior meets AC and quality gates.
- **Key permissions:** Block release on quality failure, create bug tickets, request test environments.
- **Key forbidden actions:** Approve release with open P0/P1 bugs, skip regression on critical paths, mark flaky tests as passing.
- **Core constraints:** `principles.ki:CORRECT,TEST,REL` · `security.ki:DATA_SEC=mask_in_non_production` · `workflow.ki:TASK=definition_of_done,CI=GATE`
- **Execution tags:** `out_steps`, `r03`, `r10`
- **Escalation:** Customer trust risk · Production outage · P0 bug found in prod

---

#### `gate_k` — Quality Gatekeeper [`roles/gate_k.ki`](roles/gate_k.ki)

- **Purpose:** Enforce quality gates and prevent substandard deliverables from progressing.
- **Key permissions:** Block PR merge, request changes, escalate critical violations.
- **Key forbidden actions:** Approve PR with failing CI, waive security gates without sign-off, approve own PR.
- **Core constraints:** `principles.ki:REVIEW,CORRECT,SEC` · `security.ki:SEC_CORE,AUDIT` · `workflow.ki:PR,CI=GATE,REVIEW`
- **Execution tags:** `out_steps`, `r05`, `r10`
- **Escalation:** Security breach signals · P0 vuln in PR · CI gate bypass attempted

---

### CAT_07 — Infrastructure & Operations

#### `devops` — DevOps Engineer [`roles/devops.ki`](roles/devops.ki)

- **Purpose:** Build and maintain CI/CD pipelines, infrastructure, and delivery automation.
- **Key permissions:** Manage pipeline configs, provision non-prod infra, deploy to staging, configure monitoring.
- **Key forbidden actions:** Deploy to prod without manual gate, store secrets in pipeline plaintext, disable security scans.
- **Core constraints:** `principles.ki:OPS,REL,SEC` · `security.ki:SECRET,INFRA_SEC,AUDIT` · `workflow.ki:CI,CD,RELEASE,HOTFIX`
- **Execution tags:** `out_steps`, `r06`, `r08`, `r10`
- **Escalation:** Production outage · Possible data loss · Security breach signals

---

#### `sre` — Site Reliability Engineer [`roles/sre.ki`](roles/sre.ki)

- **Purpose:** Ensure reliability, availability, and performance of production systems.
- **Key permissions:** Trigger incident response, escalate SEV0/SEV1, enforce SLO gates, request architecture changes.
- **Key forbidden actions:** Silence alerts without root cause, skip post-mortem for SEV1/SEV2, disable circuit breakers.
- **Core constraints:** `principles.ki:REL,OPS,PERF,TRUTH` · `security.ki:AUDIT,INFRA_SEC` · `workflow.ki:HOTFIX,INCIDENT`
- **Execution tags:** `out_steps`, `r06`, `r07`, `r10`
- **Escalation:** Production outage · Possible data loss · Financial impact

---

### CAT_08 — Security & Compliance

#### `sec` — Security Engineer [`roles/sec.ki`](roles/sec.ki)

- **Purpose:** Protect systems, data, and users through security design, review, and response.
- **Key permissions:** Block release on critical vuln, mandate security controls, access audit logs, escalate incidents.
- **Key forbidden actions:** Approve custom crypto, allow secrets in code, bypass MFA for admin access, suppress audit logs.
- **Core constraints:** `principles.ki:P03=security,SEC,TRUTH` · `security.ki:SEC_CORE,SECRET,IAM,DATA_SEC,IO_SEC,INFRA_SEC,AUDIT`
- **Execution tags:** `out_steps`, `r05`, `r06`, `r10`
- **Escalation:** Security breach signals · Possible data loss · Legal/compliance risk

---

#### `aud` — Compliance Auditor [`roles/aud.ki`](roles/aud.ki)

- **Purpose:** Verify compliance with regulatory and internal policy requirements.
- **Key permissions:** Read audit logs, review access control configs, request evidence from any role.
- **Key forbidden actions:** Modify audit logs, approve non-compliant systems, skip evidence collection.
- **Core constraints:** `principles.ki:TRUTH,SEC,REVIEW` · `security.ki:AUDIT,IAM,DATA_SEC` · `workflow.ki:TASK=definition_of_done`
- **Execution tags:** `out_full`, `r05`, `r10`
- **Escalation:** Legal/compliance risk · Security breach signals · Audit log tampering detected

---

### CAT_09 — Specialized Support

#### `tech_w` — Technical Writer [`roles/tech_w.ki`](roles/tech_w.ki)

- **Purpose:** Create and maintain technical documentation for all audiences.
- **Key permissions:** Create and update documentation, request technical review, flag outdated docs.
- **Key forbidden actions:** Publish docs without technical review, document unverified behavior, skip versioning on API docs.
- **Core constraints:** `principles.ki:TRUTH,COMM,DOC` · `security.ki:DATA_SEC=never_document_secrets` · `coding.ki:DOC`
- **Execution tags:** `out_full`, `r01`, `r10`
- **Escalation:** Doc conflicts with code · API breaking change undocumented

---

#### `rel_m` — Release Manager [`roles/rel_m.ki`](roles/rel_m.ki)

- **Purpose:** Coordinate and govern the software release process.
- **Key permissions:** Approve production deployments, trigger rollbacks, block release on gate failure.
- **Key forbidden actions:** Deploy without rollback plan, skip release checklist, override CI failures.
- **Core constraints:** `principles.ki:REL,CHANGE,OPS` · `security.ki:AUDIT,INFRA_SEC` · `workflow.ki:CD,RELEASE,HOTFIX`
- **Execution tags:** `out_steps`, `r02`, `r06`, `r10`
- **Escalation:** Production outage · Financial impact · Rollback required

---

## 🔗 Role Interaction Rules

Roles do not operate in isolation. Cross-role interactions follow these rules:

| Trigger | Rule |
| :--- | :--- |
| `pm` requests scope change | `tl` must assess technical impact first |
| `arch` mandates a pattern | `tl` and `swe` must implement it |
| `sec` blocks a release | Only `sec` or above can unblock |
| `qa` blocks a release | `po` or `pm` may accept risk with written sign-off |
| `gate_k` blocks a PR | Author must resolve — bypass is forbidden |
| `sre` declares an incident | All roles defer to the incident commander |
| `aud` requests evidence | All roles must comply within SLA |
| `ai_e` deploys a model | `sec` and `qa` must sign off |
| `de` runs a migration | `rel_m` and `sre` must be notified |
| `tl` approves an ADR | `arch` must review if cross-domain |

**Escalation path:** `role` → `tl` → `arch` → `em` → `sec` (if security-related)

**Permission boundaries:**
- A role may not grant permissions it does not hold.
- A role may not access systems outside its defined scope.
- An AI agent executing as a role inherits that role's permissions and forbidden actions exactly.

---

## 🤖 AI Agent Execution Context

When an AI agent operates under `kiro-roles`, the following rules apply unconditionally:

| Rule | Behavior |
| :--- | :--- |
| Must declare active role before execution | No role = no execution |
| Inherits full role definition | Including all `forbidden_actions` |
| May not self-elevate permissions | Role boundary is a hard ceiling |
| Ambiguous instruction received | Apply `r01` — ask before proceeding |
| `TRUTH` principle always active | Per `principles.ki:TRUTH` |
| Conflicts must be surfaced | Never silently resolved |
| Instruction violates `forbidden_actions` | Refuse and explain |
| Security-sensitive action detected | Apply `r05` — review required |
| Production change detected | Apply `r06` — rollback plan required |
| Output is unverifiable | Label as `unverified` |

---

## 🚀 Usage with `kiro-cli`

### Loading a single role

```bash
kiro-cli role load swe
# Reads: kiro-roles/roles/swe.ki
# Applies: kiro-core constraints + swe-specific definition
```

### Looking up a role by alias

```bash
kiro-cli role info be_e
# Reads: kiro-roles/_index.ki → resolves FILE=roles/be_e.ki → loads be_e.ki
```

### Loading all roles in a category

```bash
kiro-cli role load --cat 08
# Reads: _index.ki → finds all ROLE entries with CAT=08 → loads sec.ki, aud.ki
```

### How `_index.ki` is used

`kiro-cli` reads `_index.ki` as the manifest before any role operation:

```text
_index.ki
  └── ROLE REGISTRY          → maps alias → file path → category
  └── ROLE SYSTEM MODEL      → RBAC axioms, permission model
  └── ROLE INTERACTION RULES → cross-role constraints
  └── AI AGENT EXEC CONTEXT  → AI-specific behavioral rules
  └── CONFLICT RESOLUTION    → tie-breaking rules
  └── VERSIONING             → SemVer + compatibility rules
```

Each individual role file is self-contained and includes a back-reference to `_index.ki` via `INDEX=kiro-roles/_index.ki`, so the CLI can always resolve the full system context from either entry point.

### Project integration

```text
my-project/
└── .kiro/
    ├── kiro.yaml
    └── .cache/
        ├── kiro-core/
        │   ├── glossary.ki
        │   ├── principles.ki
        │   └── ...
        └── kiro-roles/
            ├── _index.ki
            └── roles/
                ├── swe.ki
                ├── be_e.ki
                └── ...
```

When an AI task is requested with a role context, `kiro-cli` loads:
1. All `kiro-core` `.ki` files (global constraints)
2. `kiro-roles/_index.ki` (system model + interaction rules)
3. The specific role file (e.g., `roles/swe.ki`)

This gives the AI the minimum necessary context — no more, no less.

---

## ⚖️ Conflict Resolution

| Conflict | Resolution |
| :--- | :--- |
| `kiro-core` rule vs. any role rule | `kiro-core` always wins |
| Ambiguous role boundary | Escalate per `glossary.ki:r01` (ask) |
| Two roles claim same decision authority | `tl` arbitrates technical · `pm` arbitrates product |
| Role constraint vs. business pressure | Apply `principles.ki` P01–P10 priority order |
| Security constraint vs. delivery speed | Reject unless explicit risk acceptance is signed |
| AI agent role conflict | Refuse execution, surface conflict to human |

---

## 🛠 Contributing

Because `kiro-roles` inherits from `kiro-core`, all changes must respect the parent layer:

1. **New role** — add a `.ki` file in `roles/`, register it in `_index.ki` under `ROLE REGISTRY`. Bump `MINOR` version.
2. **Role definition change** — update the role's `.ki` file. Add a changelog entry. If permission boundary changes, `sec` review is required.
3. **Breaking change** (role boundary or permission model) — submit an RFC, get `arch` + `sec` sign-off, bump `MAJOR` version.
4. **Never** redefine, paraphrase, or weaken a `kiro-core` rule inside any role file.

**Versioning rules:**

| Change type | Version bump |
| :--- | :--- |
| Breaking change to role boundary or permission model | `MAJOR` |
| New role added or non-breaking responsibility change | `MINOR` |
| Clarification or documentation fix | `PATCH` |

---

*© 2026 Kiro Suite. Standardizing the future of human-machine collaboration.*
