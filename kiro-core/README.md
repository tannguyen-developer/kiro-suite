# 🧠 Kiro Core (`kiro-core`)

[[Tiếng Việt] README.vi.md](README.vi.md)

> `kiro-core` is the **Foundation Layer** of the `kiro-suite` ecosystem. It defines immutable principles, shared language, and universal standards that apply globally across every project, role, workflow, and AI-assisted task. All other modules — `kiro-rules`, `kiro-roles`, and beyond — inherit from and operate on top of what is established here.

---

## 📌 Overview

The purpose of `kiro-core` is to:

- **Eliminate ambiguity** — every term, rule, and directive has exactly one meaning.
- **Reduce AI token consumption** — compact `.ki` format encodes maximum signal in minimum tokens.
- **Standardize engineering thinking** — from naming a variable to designing a distributed system, the same principles apply.
- **Serve as the single source of truth** — no rule defined elsewhere overrides what is defined here unless explicitly versioned.

| Property | Value |
| :--- | :--- |
| **Format** | `.ki` — Token-optimized key-value directive files |
| **Mode** | `machine_first` — Optimized for AI/LLM context parsing |
| **Scope** | Global — applies to all languages, roles, and projects |
| **Versioning** | SemVer (Major.Minor.Patch) |
| **Conflict resolution** | Higher priority pillar wins; equal priority → choose simpler, then more reversible, then faster |

---

## 🗂 Structure: 9 Pillars

| # | File | Role | Core Focus |
| :- | :--- | :--- | :--- |
| 1 | [`glossary.ki`](#1-glossaryki--shared-language) | **Shared Language** | Canonical aliases, priority levels (P0–P4), severity levels (SEV0–SEV4), delivery states, work types, role definitions, and universal execution rules. |
| 2 | [`principles.ki`](#2-principlesky--constitution) | **Constitution** | The highest-level decision framework: priority order, truth/honesty rules, value delivery, complexity control, anti-patterns, and escalation triggers. |
| 3 | [`architecture.ki`](#3-architectureki--system-design) | **System Design** | Architectural drivers, domain boundaries (DDD), sync vs. async communication, data management, scalability, resilience, and 12-Factor compliance. |
| 4 | [`coding.ki`](#4-codingki--engineering-standards) | **Engineering Standards** | Clean code rules for functions, control flow, state, error handling, comments, modules, concurrency, and data structures. |
| 5 | [`naming.ki`](#5-namingki--naming-conventions) | **Naming Conventions** | Intent-revealing names, semantic prefixes, standardized action verbs, casing rules, and banned anti-patterns. |
| 6 | [`quality.ki`](#6-qualityki--quality-assurance) | **Quality Assurance** | Testing pyramid (unit/integration/e2e), CI/CD quality gates, performance baselines, and test maintenance standards. |
| 7 | [`security.ki`](#7-securityki--security-standards) | **Security Standards** | Zero Trust mindset, secrets management, IAM, data protection, input validation, infrastructure security, and audit logging. |
| 8 | [`workflow.ki`](#8-workflowki--collaboration--delivery) | **Collaboration & Delivery** | Git branching strategy (Trunk-based), Conventional Commits, PR/MR standards, code review culture, CI/CD pipeline rules, and incident workflow. |
| 9 | [`docs.ki`](#9-docski--documentation-standards) | **Documentation Standards** | Docs-as-code, README requirements, ADR/RFC process, API documentation, inline comments, runbooks, and maintenance lifecycle. |

---

## ⚖️ Core Priority Order

When rules conflict, `kiro-core` resolves them using this strict precedence chain:

| Rank | Principle | Meaning |
| :--- | :--- | :--- |
| P01 | **Truth** | No fabricated data, fake metrics, or false completion claims. State unknowns explicitly. |
| P02 | **Correctness** | Logic must satisfy requirements and handle edge cases. Reproduce bugs before fixing. |
| P03 | **Security** | Never trade safety for speed without explicit, signed risk acceptance. |
| P04 | **Reliability** | Systems must degrade gracefully. Expect failures. Require rollback plans for production changes. |
| P05 | **Maintainability** | Code must serve future readers. Readability over cleverness. Low coupling, high cohesion. |
| P06 | **Clarity** | Names, structures, and communication must be unambiguous. |
| P07 | **Testability** | Critical paths must be verifiable. Test behavior, not implementation. |
| P08 | **Performance** | Measure before optimizing. Fix real bottlenecks, not imagined ones. |
| P09 | **Delivery Speed** | Ship small, working slices. Finish before expanding. |
| P10 | **Aesthetics** | Lowest priority. Never block delivery for style alone. |

**Tie-breaking rules (in order):** lower complexity → more reversible → faster delivery.

---

## 📖 Pillar Reference

### 1. `glossary.ki` — Shared Language

The glossary is the universal dictionary for the entire `kiro-suite`. It defines short aliases that AI and humans use identically, eliminating ambiguity at the token level.

**Key sections:**

- **Priority / Severity:** `p0=critical_now`, `p1=high` … `p4=backlog`; `sev0=service_down` … `sev4=cosmetic`; `must=mandatory`, `should=recommended`, `may=optional`.
- **Delivery States:** `todo`, `prog`, `blk`, `rvw`, `test`, `done`, `hold`, `drop`.
- **Work Types:** `feat`, `bug`, `ref`, `perf`, `sec`, `ops`, `mig`, `spk`, `doc`, `hotfix`.
- **Requirements / Planning:** `req`, `fr`, `nfr`, `ac`, `uc`, `ec`, `dep`, `scope`, `eta`, `tbd`.
- **Engineering Terms:** `sys`, `mod`, `cmp`, `svc`, `api`, `cfg`, `env`, `job`, `queue`, `worker`, `cron`.
- **Data / Database:** `db`, `tbl`, `col`, `pk`, `fk`, `idx`, `txn`, `cache`.
- **Frontend / UI:** `ui`, `ux`, `a11y`, `resp`, `state`, `route`, `ssr`, `csr`.
- **Backend / Integration:** `rest`, `rpc`, `ctrl`, `mw`, `dto`, `repo`.
- **Security:** `authn`, `authz`, `rbac`, `mfa`, `sess`, `tok`, `secret`, `vuln`, `threat`, `least`.
- **DevOps / Delivery:** `ci`, `cd`, `pipe`, `img`, `ctr`, `iac`, `roll`.
- **Observability:** `log`, `metric`, `trace`, `alert`, `dash`, `slo`, `sli`, `sla`, `mttr`, `rto`, `rpo`.
- **Testing / Quality:** `ut`, `it`, `e2e`, `smoke`, `reg`, `mock`, `stub`, `cov`, `flake`, `lint`, `fmt`.
- **Design / Code Quality:** `clean`, `dry`, `kiss`, `solid`, `soc`, `srp`, `cc`, `smell`, `debt`.
- **Performance:** `lat`, `thr`, `mem`, `cpu`, `io`, `ttl`, `pool`, `bneck`.
- **Review / Governance:** `pr`, `mr`, `rv`, `apr`, `gate`, `adr`, `policy`, `std`, `chk`.
- **Documentation:** `readme`, `runbook`, `playbook`, `spec`, `diag`, `notes`, `kb`.
- **Decision Flags:** `ok`, `no`, `risk`, `ask`, `block`, `exp`, `trade`.
- **Universal Execution Rules:** `r01=req_unclear->ask` through `r10=done->verify`.
- **Output Tags:** `out_plan`, `out_code`, `out_diff`, `out_steps`, `out_table`, `out_short`, `out_full`.
- **Roles:** 9 categories covering 60+ engineering roles from `pm` (product manager) to `finops` (cloud financial operations).

---

### 2. `principles.ki` — Constitution

The highest-level decision framework. Every other pillar derives from these principles.

**Key sections:**

- **Mission:** `deliver_correct_secure_maintainable_value` with `high_signal_low_waste_execution`.
- **Truth / Honesty:** No fabrication, no fake APIs, no fake metrics, no guesses presented as facts. State unknowns and assumptions explicitly. `req_missing_info->ask`.
- **Requirement Handling:** Understand before executing. Define inputs, outputs, and done-state before starting. `unclear_goal->ask_goal`.
- **Value Delivery:** Solve the root problem, not the surface request. Ship small working slices. `if_scope_bloated->reduce_scope`.
- **Decision Making:** Prefer simple, explicit, reversible, observable, composable, and standard solutions. `multiple_options->compare_tradeoffs`.
- **Complexity Control:** `duplicate_twice_ok_three_times_extract`. `abstraction_without_2plus_real_use_cases->avoid`.
- **Correctness:** `bugfix->reproduce_first->identify_root_cause->add_regression_guard`.
- **Security:** `secret_in_code->block`. Extra review required for auth, payment, admin, and user data changes.
- **Reliability:** `external_dependency->handle_timeout`. `prod_change->rollback_plan`.
- **Testing Mindset:** Test behavior, not implementation. `high_risk_change->increase_test_depth`.
- **Performance:** `perf_claim_without_measurement->challenge`. `minor_gain_major_complexity->reject`.
- **Anti-Patterns (banned):** `guessing_requirements`, `gold_plating`, `premature_optimization`, `overengineering`, `copy_paste_sprawl`, `hidden_magic`, `silent_failures`, `unsafe_shortcuts`, `scope_creep`, `rewriting_working_system_without_case`, `changing_many_variables_at_once`.
- **Escalation Triggers:** Legal/compliance risk, security breach signals, possible data loss, financial impact, production outage, customer trust risk, irreversible operations.
- **Done Definition:** Objective met + major risks addressed + behavior verified + no known critical issues + docs updated + next steps clear.

---

### 3. `architecture.ki` — System Design

Standards for designing systems that are modular, observable, resilient, and secure by default.

**Key sections:**

- **Architectural Drivers:** `modularity_over_granularity`, `decoupling_over_reuse`, `observability_by_default`, `stateless_preferred`, `single_source_of_truth`, `api_first_design`.
- **System Boundaries:** DDD preferred. Bounded contexts must be explicit. `monolith_first_unless_scale_demands_microservices`. `circular_dependency_found->break_via_inversion_of_control`.
- **Communication (Sync vs. Async):** Async event-driven for state changes; sync REST/RPC for queries only. `choreography_over_orchestration_for_loose_coupling`. `idempotency_required_for_all_mutations`. `sync_call_to_external_system->require_circuit_breaker+timeout`.
- **Data Management:** One database per service. No shared database integration. `distributed_transaction_needed->use_saga_pattern`. `data_schema_change->require_backward_compatibility`.
- **Scalability & Resilience:** Horizontal scaling over vertical. Stateless compute nodes. `single_point_of_failure_identified->design_redundancy`. `retry_logic_implemented->require_exponential_backoff_and_jitter`.
- **Security in Architecture:** Zero Trust network assumption. TLS for all transit. Secrets in vault, never in code or plaintext env. `internal_service_to_service->require_mutual_tls_or_auth`.
- **State Management:** Push state to edges or storage. `session_data_required->use_distributed_cache`.
- **12-Factor App Compliance:** Config in environment. Stateless processes. Logs as event streams. `config_hardcoded_in_source->move_to_env_vars`.

---

### 4. `coding.ki` — Engineering Standards

Language-agnostic rules for writing clean, robust, and maintainable code.

**Key sections:**

- **Functions & Methods:** Do one thing well. Prefer pure functions. Limit parameters to 3 or use an object. Command-query separation. `func_length_gt_50_lines->consider_refactor`. `boolean_flag_argument->split_into_two_functions`.
- **Control Flow & Logic:** Prefer early returns. Avoid deep nesting. Fail fast on invalid input. `nesting_level_gt_3->extract_to_function`. `complex_conditional->extract_to_variable_with_clear_name`.
- **Variables & State:** Prefer immutability. Minimize variable scope. No global state mutations. `magic_number_found->extract_to_named_constant`. `magic_string_found->extract_to_enum_or_constant`.
- **Error Handling:** Never swallow errors silently. Throw specific exceptions. Include context in error messages. `empty_catch_block->reject_and_block`. `user_facing_error->sanitize_internal_details`.
- **Comments & Documentation:** Code explains *what*; comments explain *why*. `comment_repeats_code_logic->remove_comment`. `hack_or_workaround->require_comment_with_ticket_link`.
- **Modules & Dependencies:** Explicit imports only. Depend on abstractions, not concretions. `wildcard_import_used->replace_with_explicit_imports`. `tight_coupling_to_external_lib->wrap_in_adapter`.
- **Concurrency & Async:** Never block the main thread. Handle all promise rejections. `async_call_without_timeout->add_timeout`. `race_condition_risk->use_locks_mutexes_or_atomic_ops`.
- **Data Structures:** Use appropriate structure for lookup. `array_lookup_in_large_loop->convert_to_set_or_map`.

---

### 5. `naming.ki` — Naming Conventions

Names must reveal intent, be pronounceable, searchable, and consistent across the entire codebase.

**Key sections:**

- **General Principles:** Reveal intent, not implementation. Length proportional to scope. Consistency over personal preference. `cryptic_abbreviation->expand_to_full_word`.
- **Semantic Prefixes:** Booleans → `is_`, `has_`, `can_`, `should_`, `will_`. Arrays → plural nouns. Functions → strong action verb. Events → past tense verb. `boolean_without_prefix->add_is_has_can_prefix`.
- **Standardized Action Verbs:**
  - `get` = return local/fast computed value
  - `fetch` = retrieve over network or I/O
  - `compute` = heavy CPU calculation
  - `create` = instantiate new object in memory
  - `build` = construct complex object step by step
  - `parse` = convert raw data to structured format
  - `format` = convert structured data to raw string
  - `save` = persist to storage
  - `delete` = remove permanently from storage
  - `remove` = take out from collection or relation
  - `update` = modify existing record
  - `upsert` = update or insert if not exists
- **Casing Standards:** `PascalCase` for classes (universal). `UPPER_SNAKE_CASE` for constants and env vars. `kebab-case` for URLs and web assets. Consistency with language standard first.
- **Banned Anti-Patterns:** `manager`, `util`, `helper`, `data`, `info` as vague suffixes. `temp`, `val`, `res`, `obj` as variable names. Hungarian notation. Profanity or inside jokes.

---

### 6. `quality.ki` — Quality Assurance

Quality is everyone's responsibility. Shift left — test early, test often, automate everything.

**Key sections:**

- **Test Pyramid:**
  - **Unit (base):** Fast, isolated, high volume. No I/O, no network, no DB. Millisecond execution. Clear Arrange-Act-Assert structure. One logical assertion per test. `unit_test_makes_network_call->block_and_require_mock`.
  - **Integration (middle):** Verify communication between modules. Prefer real DB via containers over mocks. Verify migrations and queries. `integration_test_mocks_database->warn_prefer_ephemeral_db`.
  - **E2E (top):** Critical user journeys only. Run against production-like environment. Must clean up test data. `e2e_test_used_for_simple_edge_case->move_down_to_unit_test`.
- **CI/CD Quality Gates (all must pass):** Build compilation. All active tests. Static analysis and linting. Code formatting. Security dependency scan. No new critical/high vulnerabilities.
- **Gate Rules:** `overall_test_coverage_drops_below_threshold->fail_build`. `linter_reports_error->fail_build_no_exceptions`. `vulnerability_found_in_new_dependency->block_merge`.
- **Performance & Observability:** Load test critical read/write paths. Define latency and throughput baselines. `api_response_time_exceeds_sla_in_test->flag_performance_regression`.
- **Test Maintenance:** Treat test code with the same respect as production code. `flaky_test_unfixed_for_7_days->delete_or_rewrite`. `test_suite_takes_longer_than_15_mins->require_optimization_or_parallelization`.

---

### 7. `security.ki` — Security Standards

Secure by default. Defense in depth. Zero Trust. Deny by default, allow by exception.

**Key sections:**

- **Core Mindset:** `secure_by_default`, `defense_in_depth`, `least_privilege_access`, `zero_trust_architecture_assumption`, `deny_by_default_allow_by_exception`. `security_is_continuous_not_a_state`.
- **Secrets Management:** Never hardcode secrets. Never log credentials. Use environment variables or secret managers. Rotate secrets periodically and after any exposure. `secret_found_in_commit->revoke_immediately_and_rewrite_history`. `api_key_stored_in_frontend_code->block_and_move_to_backend_proxy`.
- **Identity & Access Management (IAM):** Authenticate every request at the perimeter. Authorize every action at the resource level. Use OAuth2/OIDC/SAML. Require MFA for admin/sensitive access. `custom_crypto_or_auth_implementation->block_and_require_standard_library`. `missing_authorization_check_on_mutation->block_as_critical_vulnerability`.
- **Data Protection:** TLS 1.2 minimum for all data in transit. Encrypt highly sensitive data at rest. Classify data: public / internal / confidential / restricted. Minimize PII collection and retention. `http_traffic_allowed_in_production->block_and_force_https`. `pii_logged_in_application_logs->mask_immediately_and_purge_logs`.
- **Input Validation & Output Encoding:** Trust no client input. Validate type, length, format, and range strictly. Use parameterized queries. `dynamic_sql_concatenation_detected->block_and_require_prepared_statements`. `raw_html_rendered_from_user_input->block_and_require_sanitization`.
- **Infrastructure & Dependency Security:** Keep all dependencies and OS patched. Run services as non-root. Implement rate limiting. Scan container images. `container_configured_to_run_as_root->block`. `dependency_has_known_critical_cve->fail_build_and_require_update`.
- **Logging, Auditing & Incident Response:** Log all auth success/failure events. Log all authorization failures. Log all critical state changes with actor identity. Append-only audit logs preferred. `system_lacks_audit_trail_for_financial_transaction->block_release`.

---

### 8. `workflow.ki` — Collaboration & Delivery

Standardized Git practices, commit discipline, PR culture, and CI/CD pipeline rules.

**Key sections:**

- **Branching Strategy:** Trunk-based development preferred. GitFlow only for legacy or highly regulated releases. Short-lived feature branches (max a few days). `main`/`master` must always be deployable. Branch name format: `type/ticket-id-short-description`. `direct_commit_to_main->block_and_require_pr`.
- **Commit Standards:** Conventional Commits required (`feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `test`). Atomic commits — one logical change per commit. Imperative mood in subject line. Include *why* in body when *what* is complex. `commit_message_lacks_conventional_prefix->reject_commit`.
- **Pull Request / Merge Request:** Small and focused. Must link to issue tracker ticket. Must have clear title and description. Draft status for WIP. Minimum one approval from code owner. `pr_diff_gt_400_lines->warn_and_suggest_splitting_pr`. `pr_ci_checks_failed->block_merge`.
- **Code Review Culture:** Review for correctness, security, and maintainability. Automate style checks — do not argue syntax in PRs. Assume good intent. Prefix nitpicks with `nit:`, blockers with `blocker:`. `review_turnaround_gt_24_hours->escalate_to_team_chat`.
- **CI/CD:** Run tests and linters on every push. Build artifacts once, deploy everywhere. Automated deploy to staging on merge to main. Production deploy requires manual gate unless fully mature. Immutable artifacts — never overwrite a published version. `build_breaks_on_main->highest_priority_fix_drop_everything`.
- **Issue Tracking:** No ticket, no code. Definition of Ready must be met before starting. Definition of Done must be met before closing. `ticket_in_progress_gt_5_days->flag_as_blocked_or_too_large`.
- **Incident & Hotfix Workflow:** Branch directly from production tag. Prioritize mitigation over perfect fix. Must backport to main after deploying to prod. Blameless post-mortem required for SEV1/SEV2. `hotfix_bypasses_ci->reject_unless_system_is_completely_down`.

---

### 9. `docs.ki` — Documentation Standards

Documentation is a first-class deliverable. Outdated docs are worse than no docs.

**Key sections:**

- **Core Mindset:** Docs-as-code — store in version control. Single source of truth. Optimize for the reader, not the writer. `doc_conflicts_with_code->code_is_truth_update_doc`. `information_duplicated_in_wiki_and_repo->delete_wiki_link_to_repo`.
- **Repository Level (README):** Every repository must have a README. Must state what the project does in one sentence. Must list prerequisites and setup steps. Must provide local execution command. Must identify code owners or maintainer team. `readme_lacks_setup_instructions->block_pr_for_new_repo`.
- **Architecture & Decision Records:** Record significant decisions via ADR (Architecture Decision Record). ADR format must include: context, decision, and consequences. ADR states: `proposed` → `accepted` → `deprecated` → `superseded`. Use RFC (Request for Comments) for proposals before ADR. `major_dependency_added->require_adr`. `database_technology_changed->require_adr`.
- **API & Integration Documentation:** Public and internal APIs must be documented. REST → OpenAPI/Swagger. GraphQL → inline schema descriptions. gRPC → Protobuf comments. Generate docs from code wherever possible. `new_api_endpoint_added->require_openapi_spec_update`.
- **Inline Comments:** Document *why*, not *what*. Document hacks and tech debt with ticket links. Use standard docstrings (JSDoc, PyDoc, GoDoc). `comment_explains_basic_language_syntax->delete_comment`. `todo_comment_without_ticket_link->warn_and_request_link`.
- **Operations & Runbooks:** Every production service must have a runbook. Runbook must list: critical alerts and their meaning, step-by-step troubleshooting guides, escalation contacts. `service_alert_fires_but_not_in_runbook->add_to_runbook_post_incident`.
- **Maintenance & Lifecycle:** Review core docs quarterly. Use Markdown as the default format. Use Mermaid.js or PlantUML for diagrams-as-code. `doc_last_updated_gt_365_days->flag_for_freshness_review`. `feature_removed_from_code->remove_feature_from_docs`.

---

## 🚀 Usage with `kiro-cli`

When `kiro-cli` is used inside a project, the local structure looks like this:

```text
my-project/
└── .kiro/
    ├── kiro.yaml          # Declares usage of kiro-core (and version)
    └── .cache/
        └── kiro-core/     # Downloaded content from kiro-suite
            ├── glossary.ki
            ├── principles.ki
            ├── architecture.ki
            ├── coding.ki
            ├── naming.ki
            ├── quality.ki
            ├── security.ki
            ├── workflow.ki
            └── docs.ki
```

### How AI consumes context

When you request an AI task inside the project, `kiro-cli` automatically loads all `.ki` files inside `.kiro/` as input context before the AI processes your request.

**Practical effects:**

- AI knows `must=mandatory` and `should=recommended` — it will not treat them as equivalent.
- AI follows the priority chain `truth > correctness > security > reliability > maintainability` when generating or reviewing code.
- AI will refuse a request that introduces a hidden side effect, because `coding.ki` bans `hidden_magic`.
- AI will ask for clarification when `req_unclear` is detected, per `r01` in `glossary.ki`.
- AI will flag a `secret_in_code` as a blocker, per `security.ki`.
- AI will suggest splitting a PR with more than 400 lines of diff, per `workflow.ki`.

---

## 🛠 Contributing

Because `kiro-core` is the foundation layer, all changes must go through a strict process:

1. **Submit an RFC** (Request for Comments) — describe the problem, proposed change, and impact.
2. **Verify backward compatibility** — no change may silently break existing behavior in dependent modules.
3. **Update versioning** according to SemVer:
   - `MAJOR` — breaking change to an existing rule or directive.
   - `MINOR` — new rule or directive added in a backward-compatible way.
   - `PATCH` — clarification, typo fix, or non-behavioral update.
4. **Promote RFC to ADR** once accepted — record context, decision, and consequences.

---

*© 2026 Kiro Suite. Standardizing the future of human-machine collaboration.*
