# 🧠 Kiro Core (`kiro-core`)

[[English] README.md](README.md)

> `kiro-core` là **Foundation Layer** của hệ sinh thái `kiro-suite`. Nó định nghĩa các nguyên tắc bất biến, ngôn ngữ dùng chung và các tiêu chuẩn phổ quát áp dụng toàn cục cho mọi project, role, workflow và tác vụ có hỗ trợ AI. Tất cả module khác — `kiro-rules`, `kiro-roles` và các module mở rộng — đều kế thừa và hoạt động dựa trên những gì được thiết lập tại đây.

---

## 📌 Tổng quan

Mục đích của `kiro-core`:

* **Loại bỏ sự mơ hồ** — mọi thuật ngữ, rule và directive chỉ có đúng một nghĩa.
* **Giảm token consumption của AI** — định dạng `.ki` nhỏ gọn mã hóa lượng signal tối đa với số token tối thiểu.
* **Chuẩn hóa tư duy engineering** — từ đặt tên biến đến thiết kế distributed system, cùng một tập nguyên tắc được áp dụng.
* **Làm single source of truth** — không rule nào được định nghĩa ở nơi khác có thể override những gì định nghĩa tại đây trừ khi được version hóa rõ ràng.

| Thuộc tính              | Giá trị                                                                                                                 |
| :---------------------- | :---------------------------------------------------------------------------------------------------------------------- |
| **Format**              | `.ki` — file directive key-value tối ưu token                                                                           |
| **Mode**                | `machine_first` — tối ưu cho AI/LLM context parsing                                                                     |
| **Scope**               | Global — áp dụng cho mọi ngôn ngữ, role và project                                                                      |
| **Versioning**          | SemVer (Major.Minor.Patch)                                                                                              |
| **Conflict resolution** | Pillar có priority cao hơn sẽ thắng; nếu cùng priority → chọn giải pháp đơn giản hơn, rồi reversible hơn, rồi nhanh hơn |

---

## 🗂 Cấu trúc: 9 Pillars

| #  | File                                                    | Vai trò                      | Trọng tâm                                                                                                                                            |
| :- | :------------------------------------------------------ | :--------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | [`glossary.ki`](#1-glossaryki--shared-language)         | **Shared Language**          | Canonical aliases, priority levels (P0–P4), severity levels (SEV0–SEV4), delivery states, work types, role definitions và universal execution rules. |
| 2  | [`principles.ki`](#2-principleski--constitution)        | **Constitution**             | Framework ra quyết định cấp cao nhất: priority order, truth/honesty rules, value delivery, complexity control, anti-patterns và escalation triggers. |
| 3  | [`architecture.ki`](#3-architectureki--system-design)   | **System Design**            | Architectural drivers, domain boundaries (DDD), sync vs. async communication, data management, scalability, resilience và tuân thủ 12-Factor.        |
| 4  | [`coding.ki`](#4-codingki--engineering-standards)       | **Engineering Standards**    | Clean code rules cho functions, control flow, state, error handling, comments, modules, concurrency và data structures.                              |
| 5  | [`naming.ki`](#5-namingki--naming-conventions)          | **Naming Conventions**       | Intent-revealing names, semantic prefixes, standardized action verbs, casing rules và banned anti-patterns.                                          |
| 6  | [`quality.ki`](#6-qualityki--quality-assurance)         | **Quality Assurance**        | Testing pyramid (unit/integration/e2e), CI/CD quality gates, performance baselines và test maintenance standards.                                    |
| 7  | [`security.ki`](#7-securityki--security-standards)      | **Security Standards**       | Zero Trust mindset, secrets management, IAM, data protection, input validation, infrastructure security và audit logging.                            |
| 8  | [`workflow.ki`](#8-workflowki--collaboration--delivery) | **Collaboration & Delivery** | Git branching strategy (Trunk-based), Conventional Commits, PR/MR standards, code review culture, CI/CD pipeline rules và incident workflow.         |
| 9  | [`docs.ki`](#9-docski--documentation-standards)         | **Documentation Standards**  | Docs-as-code, README requirements, quy trình ADR/RFC, API documentation, inline comments, runbooks và maintenance lifecycle.                         |

---

## ⚖️ Core Priority Order

Khi các rule xung đột, `kiro-core` giải quyết bằng precedence chain nghiêm ngặt sau:

| Rank | Principle           | Ý nghĩa                                                                                                       |
| :--- | :------------------ | :------------------------------------------------------------------------------------------------------------ |
| P01  | **Truth**           | Không fabricated data, fake metrics hoặc false completion claims. Phải nêu rõ những gì chưa biết.             |
| P02  | **Correctness**     | Logic phải đáp ứng requirement và xử lý edge cases. Reproduce bug trước khi fix.                              |
| P03  | **Security**        | Không đánh đổi safety lấy tốc độ nếu chưa có explicit, signed risk acceptance.                                |
| P04  | **Reliability**     | System phải degrade gracefully. Luôn giả định failure sẽ xảy ra. Production change phải có rollback plan.     |
| P05  | **Maintainability** | Code phải phục vụ người đọc trong tương lai. Ưu tiên readability hơn cleverness. Low coupling, high cohesion. |
| P06  | **Clarity**         | Naming, structure và communication phải rõ ràng, không mơ hồ.                                                 |
| P07  | **Testability**     | Critical paths phải verify được. Test behavior, không test implementation.                                    |
| P08  | **Performance**     | Đo lường trước khi optimize. Fix bottleneck thực sự, không phải bottleneck tưởng tượng.                       |
| P09  | **Delivery Speed**  | Ship từng phần nhỏ nhưng chạy được. Hoàn thành trước rồi mới mở rộng.                                         |
| P10  | **Aesthetics**      | Priority thấp nhất. Không được block delivery chỉ vì style.                                                   |

**Tie-breaking rules (theo thứ tự):** complexity thấp hơn → reversible hơn → delivery nhanh hơn.

---

## 📖 Tham chiếu Pillar

### 1. `glossary.ki` — Shared Language

Glossary là từ điển phổ quát cho toàn bộ `kiro-suite`. Nó định nghĩa các alias ngắn mà AI và con người sử dụng giống hệt nhau, loại bỏ ambiguity ở mức token.

**Các section chính:**

* **Priority / Severity:** `p0=critical_now`, `p1=high` … `p4=backlog`; `sev0=service_down` … `sev4=cosmetic`; `must=mandatory`, `should=recommended`, `may=optional`.
* **Delivery States:** `todo`, `prog`, `blk`, `rvw`, `test`, `done`, `hold`, `drop`.
* **Work Types:** `feat`, `bug`, `ref`, `perf`, `sec`, `ops`, `mig`, `spk`, `doc`, `hotfix`.
* **Requirements / Planning:** `req`, `fr`, `nfr`, `ac`, `uc`, `ec`, `dep`, `scope`, `eta`, `tbd`.
* **Engineering Terms:** `sys`, `mod`, `cmp`, `svc`, `api`, `cfg`, `env`, `job`, `queue`, `worker`, `cron`.
* **Data / Database:** `db`, `tbl`, `col`, `pk`, `fk`, `idx`, `txn`, `cache`.
* **Frontend / UI:** `ui`, `ux`, `a11y`, `resp`, `state`, `route`, `ssr`, `csr`.
* **Backend / Integration:** `rest`, `rpc`, `ctrl`, `mw`, `dto`, `repo`.
* **Security:** `authn`, `authz`, `rbac`, `mfa`, `sess`, `tok`, `secret`, `vuln`, `threat`, `least`.
* **DevOps / Delivery:** `ci`, `cd`, `pipe`, `img`, `ctr`, `iac`, `roll`.
* **Observability:** `log`, `metric`, `trace`, `alert`, `dash`, `slo`, `sli`, `sla`, `mttr`, `rto`, `rpo`.
* **Testing / Quality:** `ut`, `it`, `e2e`, `smoke`, `reg`, `mock`, `stub`, `cov`, `flake`, `lint`, `fmt`.
* **Design / Code Quality:** `clean`, `dry`, `kiss`, `solid`, `soc`, `srp`, `cc`, `smell`, `debt`.
* **Performance:** `lat`, `thr`, `mem`, `cpu`, `io`, `ttl`, `pool`, `bneck`.
* **Review / Governance:** `pr`, `mr`, `rv`, `apr`, `gate`, `adr`, `policy`, `std`, `chk`.
* **Documentation:** `readme`, `runbook`, `playbook`, `spec`, `diag`, `notes`, `kb`.
* **Decision Flags:** `ok`, `no`, `risk`, `ask`, `block`, `exp`, `trade`.
* **Universal Execution Rules:** `r01=req_unclear->ask` đến `r10=done->verify`.
* **Output Tags:** `out_plan`, `out_code`, `out_diff`, `out_steps`, `out_table`, `out_short`, `out_full`.
* **Roles:** 9 category bao phủ hơn 60 engineering roles từ `pm` (product manager) đến `finops` (cloud financial operations).

---

### 2. `principles.ki` — Constitution

Framework ra quyết định cấp cao nhất. Mọi pillar khác đều xuất phát từ các nguyên tắc này.

**Các section chính:**

* **Mission:** `deliver_correct_secure_maintainable_value` với `high_signal_low_waste_execution`.
* **Truth / Honesty:** Không fabrication, không fake APIs, không fake metrics, không trình bày suy đoán như fact. Phải nêu rõ unknowns và assumptions. `req_missing_info->ask`.
* **Requirement Handling:** Hiểu requirement trước khi thực thi. Xác định inputs, outputs và done-state trước khi bắt đầu. `unclear_goal->ask_goal`.
* **Value Delivery:** Giải quyết root problem, không chỉ surface request. Ship từng phần nhỏ nhưng hoạt động được. `if_scope_bloated->reduce_scope`.
* **Decision Making:** Ưu tiên solution đơn giản, explicit, reversible, observable, composable và standard. `multiple_options->compare_tradeoffs`.
* **Complexity Control:** `duplicate_twice_ok_three_times_extract`. `abstraction_without_2plus_real_use_cases->avoid`.
* **Correctness:** `bugfix->reproduce_first->identify_root_cause->add_regression_guard`.
* **Security:** `secret_in_code->block`. Các thay đổi liên quan auth, payment, admin và user data cần review bổ sung.
* **Reliability:** `external_dependency->handle_timeout`. `prod_change->rollback_plan`.
* **Testing Mindset:** Test behavior, không test implementation. `high_risk_change->increase_test_depth`.
* **Performance:** `perf_claim_without_measurement->challenge`. `minor_gain_major_complexity->reject`.
* **Anti-Patterns (bị cấm):** `guessing_requirements`, `gold_plating`, `premature_optimization`, `overengineering`, `copy_paste_sprawl`, `hidden_magic`, `silent_failures`, `unsafe_shortcuts`, `scope_creep`, `rewriting_working_system_without_case`, `changing_many_variables_at_once`.
* **Escalation Triggers:** Legal/compliance risk, tín hiệu security breach, khả năng mất dữ liệu, financial impact, production outage, customer trust risk, irreversible operations.
* **Done Definition:** Đạt objective + xử lý major risks + verify behavior + không còn critical issues đã biết + docs được cập nhật + next steps rõ ràng.

---

### 3. `architecture.ki` — System Design

Tiêu chuẩn thiết kế system theo hướng modular, observable, resilient và secure-by-default.

**Các section chính:**

* **Architectural Drivers:** `modularity_over_granularity`, `decoupling_over_reuse`, `observability_by_default`, `stateless_preferred`, `single_source_of_truth`, `api_first_design`.
* **System Boundaries:** Ưu tiên DDD. Bounded contexts phải explicit. `monolith_first_unless_scale_demands_microservices`. `circular_dependency_found->break_via_inversion_of_control`.
* **Communication (Sync vs. Async):** Async event-driven cho state changes; sync REST/RPC chỉ dùng cho queries. `choreography_over_orchestration_for_loose_coupling`. `idempotency_required_for_all_mutations`. `sync_call_to_external_system->require_circuit_breaker+timeout`.
* **Data Management:** Một database cho mỗi service. Không shared database integration. `distributed_transaction_needed->use_saga_pattern`. `data_schema_change->require_backward_compatibility`.
* **Scalability & Resilience:** Horizontal scaling thay vì vertical scaling. Stateless compute nodes. `single_point_of_failure_identified->design_redundancy`. `retry_logic_implemented->require_exponential_backoff_and_jitter`.
* **Security in Architecture:** Giả định Zero Trust network. TLS cho toàn bộ traffic transit. Secrets lưu trong vault, không nằm trong code hoặc plaintext env. `internal_service_to_service->require_mutual_tls_or_auth`.
* **State Management:** Đẩy state ra edge hoặc storage layer. `session_data_required->use_distributed_cache`.
* **12-Factor App Compliance:** Config trong environment. Stateless processes. Logs dưới dạng event streams. `config_hardcoded_in_source->move_to_env_vars`.

---

### 4. `coding.ki` — Engineering Standards

Rule độc lập ngôn ngữ cho việc viết code sạch, robust và maintainable.

**Các section chính:**

* **Functions & Methods:** Chỉ làm một việc. Ưu tiên pure function. Tối đa 3 parameters hoặc dùng object. Command-query separation. `func_length_gt_50_lines->consider_refactor`. `boolean_flag_argument->split_into_two_functions`.
* **Control Flow & Logic:** Ưu tiên early return. Tránh nesting sâu. Fail fast với invalid input. `nesting_level_gt_3->extract_to_function`. `complex_conditional->extract_to_variable_with_clear_name`.
* **Variables & State:** Ưu tiên immutability. Giảm tối đa scope của variable. Không mutate global state. `magic_number_found->extract_to_named_constant`. `magic_string_found->extract_to_enum_or_constant`.
* **Error Handling:** Không swallow errors một cách im lặng. Throw exception cụ thể. Error message phải có context. `empty_catch_block->reject_and_block`. `user_facing_error->sanitize_internal_details`.
* **Comments & Documentation:** Code giải thích *what*; comments giải thích *why*. `comment_repeats_code_logic->remove_comment`. `hack_or_workaround->require_comment_with_ticket_link`.
* **Modules & Dependencies:** Chỉ import explicit. Depend vào abstraction, không depend vào concrete implementation. `wildcard_import_used->replace_with_explicit_imports`. `tight_coupling_to_external_lib->wrap_in_adapter`.
* **Concurrency & Async:** Không block main thread. Handle mọi promise rejection. `async_call_without_timeout->add_timeout`. `race_condition_risk->use_locks_mutexes_or_atomic_ops`.
* **Data Structures:** Chọn structure phù hợp cho lookup. `array_lookup_in_large_loop->convert_to_set_or_map`.

---

### 5. `naming.ki` — Naming Conventions

Tên phải thể hiện intent, dễ đọc, dễ tìm kiếm và nhất quán trong toàn bộ codebase.

**Các section chính:**

* **General Principles:** Thể hiện intent, không phải implementation. Độ dài tương ứng với scope. Consistency quan trọng hơn sở thích cá nhân. `cryptic_abbreviation->expand_to_full_word`.
* **Semantic Prefixes:** Boolean → `is_`, `has_`, `can_`, `should_`, `will_`. Array → plural nouns. Function → action verb mạnh. Event → past tense verb. `boolean_without_prefix->add_is_has_can_prefix`.
* **Standardized Action Verbs:**

  * `get` = trả về local/fast computed value
  * `fetch` = lấy dữ liệu qua network hoặc I/O
  * `compute` = tính toán CPU nặng
  * `create` = instantiate object mới trong memory
  * `build` = construct object phức tạp theo từng bước
  * `parse` = chuyển raw data sang structured format
  * `format` = chuyển structured data sang raw string
  * `save` = persist vào storage
  * `delete` = xóa vĩnh viễn khỏi storage
  * `remove` = loại khỏi collection hoặc relation
  * `update` = sửa existing record
  * `upsert` = update hoặc insert nếu chưa tồn tại
* **Casing Standards:** `PascalCase` cho class. `UPPER_SNAKE_CASE` cho constants và env vars. `kebab-case` cho URL và web assets. Luôn ưu tiên standard của ngôn ngữ.
* **Banned Anti-Patterns:** `manager`, `util`, `helper`, `data`, `info` dưới dạng suffix mơ hồ. `temp`, `val`, `res`, `obj` làm variable name. Hungarian notation. Từ tục tĩu hoặc inside jokes.

---

### 6. `quality.ki` — Quality Assurance

Quality là trách nhiệm của tất cả mọi người. Shift left — test sớm, test thường xuyên, automate mọi thứ.

**Các section chính:**

* **Test Pyramid:**

  * **Unit (base):** Nhanh, isolated, volume lớn. Không I/O, network hoặc DB. Chạy ở mức milliseconds. Structure Arrange-Act-Assert rõ ràng. Một logical assertion cho mỗi test. `unit_test_makes_network_call->block_and_require_mock`.
  * **Integration (middle):** Verify giao tiếp giữa modules. Ưu tiên DB thật qua containers hơn mocks. Verify migrations và queries. `integration_test_mocks_database->warn_prefer_ephemeral_db`.
  * **E2E (top):** Chỉ dành cho critical user journeys. Chạy trên environment gần production. Phải cleanup test data. `e2e_test_used_for_simple_edge_case->move_down_to_unit_test`.
* **CI/CD Quality Gates (bắt buộc pass):** Build compilation, toàn bộ active tests, static analysis, linting, formatting, security dependency scan, không có critical/high vulnerabilities mới.
* **Gate Rules:** `overall_test_coverage_drops_below_threshold->fail_build`. `linter_reports_error->fail_build_no_exceptions`. `vulnerability_found_in_new_dependency->block_merge`.
* **Performance & Observability:** Load test critical read/write paths. Định nghĩa latency và throughput baselines. `api_response_time_exceeds_sla_in_test->flag_performance_regression`.
* **Test Maintenance:** Đối xử với test code như production code. `flaky_test_unfixed_for_7_days->delete_or_rewrite`. `test_suite_takes_longer_than_15_mins->require_optimization_or_parallelization`.

---

### 7. `security.ki` — Security Standards

Secure-by-default. Defense in depth. Zero Trust. Deny-by-default, allow-by-exception.

**Các section chính:**

* **Core Mindset:** `secure_by_default`, `defense_in_depth`, `least_privilege_access`, `zero_trust_architecture_assumption`, `deny_by_default_allow_by_exception`. `security_is_continuous_not_a_state`.
* **Secrets Management:** Không hardcode secrets. Không log credentials. Dùng environment variables hoặc secret managers. Rotate secrets định kỳ và sau mọi lần exposure. `secret_found_in_commit->revoke_immediately_and_rewrite_history`. `api_key_stored_in_frontend_code->block_and_move_to_backend_proxy`.
* **Identity & Access Management (IAM):** Authenticate mọi request tại perimeter. Authorize mọi action ở resource level. Dùng OAuth2/OIDC/SAML. MFA bắt buộc cho admin/sensitive access. `custom_crypto_or_auth_implementation->block_and_require_standard_library`. `missing_authorization_check_on_mutation->block_as_critical_vulnerability`.
* **Data Protection:** TLS 1.2 tối thiểu cho toàn bộ data transit. Encrypt dữ liệu nhạy cảm khi lưu trữ. Classify data: public / internal / confidential / restricted. Giảm tối đa việc thu thập và retention PII. `http_traffic_allowed_in_production->block_and_force_https`. `pii_logged_in_application_logs->mask_immediately_and_purge_logs`.
* **Input Validation & Output Encoding:** Không trust client input. Validate chặt type, length, format và range. Dùng parameterized queries. `dynamic_sql_concatenation_detected->block_and_require_prepared_statements`. `raw_html_rendered_from_user_input->block_and_require_sanitization`.
* **Infrastructure & Dependency Security:** Giữ dependencies và OS luôn được patch. Chạy services dưới non-root user. Implement rate limiting. Scan container images. `container_configured_to_run_as_root->block`. `dependency_has_known_critical_cve->fail_build_and_require_update`.
* **Logging, Auditing & Incident Response:** Log mọi auth success/failure events. Log authorization failures. Log critical state changes cùng actor identity. Ưu tiên append-only audit logs. `system_lacks_audit_trail_for_financial_transaction->block_release`.

---

### 8. `workflow.ki` — Collaboration & Delivery

Chuẩn hóa Git practices, commit discipline, PR culture và CI/CD pipeline rules.

**Các section chính:**

* **Branching Strategy:** Ưu tiên Trunk-based development. Chỉ dùng GitFlow cho legacy hoặc regulated releases. Feature branch ngắn hạn (tối đa vài ngày). `main`/`master` luôn phải deployable. Format branch: `type/ticket-id-short-description`. `direct_commit_to_main->block_and_require_pr`.
* **Commit Standards:** Bắt buộc Conventional Commits (`feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `test`). Atomic commits — một logical change cho mỗi commit. Subject line dùng imperative mood. Phần body phải giải thích *why* nếu *what* phức tạp. `commit_message_lacks_conventional_prefix->reject_commit`.
* **Pull Request / Merge Request:** Nhỏ và tập trung. Phải link issue tracker ticket. Title và description phải rõ ràng. Dùng draft cho WIP. Tối thiểu một approval từ code owner. `pr_diff_gt_400_lines->warn_and_suggest_splitting_pr`. `pr_ci_checks_failed->block_merge`.
* **Code Review Culture:** Review correctness, security và maintainability. Automate style checks — không tranh cãi syntax trong PR. Giả định thiện chí. Prefix nitpick bằng `nit:`, blocker bằng `blocker:`. `review_turnaround_gt_24_hours->escalate_to_team_chat`.
* **CI/CD:** Chạy tests và linters trên mọi push. Build artifact một lần, deploy mọi nơi. Auto deploy staging khi merge vào main. Production deploy cần manual gate trừ khi pipeline đã đủ mature. Immutable artifacts — không overwrite published version. `build_breaks_on_main->highest_priority_fix_drop_everything`.
* **Issue Tracking:** Không ticket thì không code. Phải đạt Definition of Ready trước khi bắt đầu. Phải đạt Definition of Done trước khi đóng ticket. `ticket_in_progress_gt_5_days->flag_as_blocked_or_too_large`.
* **Incident & Hotfix Workflow:** Branch trực tiếp từ production tag. Ưu tiên mitigation hơn perfect fix. Phải backport về main sau khi deploy prod. Bắt buộc blameless post-mortem cho SEV1/SEV2. `hotfix_bypasses_ci->reject_unless_system_is_completely_down`.

---

### 9. `docs.ki` — Documentation Standards

Documentation là first-class deliverable. Docs lỗi thời còn tệ hơn không có docs.

**Các section chính:**

* **Core Mindset:** Docs-as-code — lưu trong version control. Single source of truth. Tối ưu cho reader, không phải writer. `doc_conflicts_with_code->code_is_truth_update_doc`. `information_duplicated_in_wiki_and_repo->delete_wiki_link_to_repo`.
* **Repository Level (README):** Mọi repository phải có README. Phải mô tả project trong một câu. Phải liệt kê prerequisites và setup steps. Phải có local execution command. Phải xác định code owners hoặc maintainer team. `readme_lacks_setup_instructions->block_pr_for_new_repo`.
* **Architecture & Decision Records:** Ghi lại decision quan trọng bằng ADR (Architecture Decision Record). ADR phải gồm: context, decision và consequences. Trạng thái ADR: `proposed` → `accepted` → `deprecated` → `superseded`. Dùng RFC (Request for Comments) cho proposal trước ADR. `major_dependency_added->require_adr`. `database_technology_changed->require_adr`.
* **API & Integration Documentation:** Public/internal APIs phải được document. REST → OpenAPI/Swagger. GraphQL → inline schema descriptions. gRPC → Protobuf comments. Ưu tiên generate docs từ code. `new_api_endpoint_added->require_openapi_spec_update`.
* **Inline Comments:** Document *why*, không document *what*. Document hacks và tech debt kèm ticket links. Dùng standard docstrings (JSDoc, PyDoc, GoDoc). `comment_explains_basic_language_syntax->delete_comment`. `todo_comment_without_ticket_link->warn_and_request_link`.
* **Operations & Runbooks:** Mọi production service phải có runbook. Runbook phải liệt kê: critical alerts và ý nghĩa của chúng, hướng dẫn troubleshooting từng bước, escalation contacts. `service_alert_fires_but_not_in_runbook->add_to_runbook_post_incident`.
* **Maintenance & Lifecycle:** Review core docs theo quý. Markdown là format mặc định. Dùng Mermaid.js hoặc PlantUML cho diagrams-as-code. `doc_last_updated_gt_365_days->flag_for_freshness_review`. `feature_removed_from_code->remove_feature_from_docs`.

---

## 🚀 Sử dụng với `kiro-cli`

Khi `kiro-cli` được dùng trong project, cấu trúc local sẽ như sau:

```text
my-project/
└── .kiro/
    ├── kiro.yaml          # Khai báo sử dụng kiro-core (và version)
    └── .cache/
        └── kiro-core/     # Nội dung tải về từ kiro-suite
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

### AI consume context như thế nào

Khi bạn yêu cầu AI thực hiện task trong project, `kiro-cli` sẽ tự động load toàn bộ file `.ki` trong `.kiro/` làm input context trước khi AI xử lý request.

**Hiệu ứng thực tế:**

* AI hiểu `must=mandatory` và `should=recommended` — không xem chúng là tương đương.
* AI tuân theo priority chain `truth > correctness > security > reliability > maintainability` khi generate hoặc review code.
* AI sẽ từ chối request tạo hidden side effect vì `coding.ki` cấm `hidden_magic`.
* AI sẽ yêu cầu clarification khi phát hiện `req_unclear`, theo `r01` trong `glossary.ki`.
* AI sẽ flag `secret_in_code` như blocker theo `security.ki`.
* AI sẽ đề xuất tách PR có diff lớn hơn 400 dòng theo `workflow.ki`.

---

## 🛠 Đóng góp

Vì `kiro-core` là foundation layer nên mọi thay đổi phải đi qua quy trình nghiêm ngặt:

1. **Submit RFC** (Request for Comments) — mô tả vấn đề, thay đổi đề xuất và impact.
2. **Verify backward compatibility** — không thay đổi nào được âm thầm phá vỡ behavior hiện có của các module phụ thuộc.
3. **Cập nhật versioning** theo SemVer:

   * `MAJOR` — breaking change cho rule hoặc directive hiện có.
   * `MINOR` — thêm rule hoặc directive mới theo cách backward-compatible.
   * `PATCH` — clarification, sửa typo hoặc update không ảnh hưởng behavior.
4. **Promote RFC thành ADR** sau khi được chấp nhận — ghi lại context, decision và consequences.

---

*© 2026 Kiro Suite. Chuẩn hóa tương lai của collaboration giữa con người và máy móc.*