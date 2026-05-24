# 👥 Kiro Roles (`kiro-roles`)

[[English] README.md](README.md)

> `kiro-roles` là **Role Layer** của hệ sinh thái `kiro-suite`. Nó định nghĩa đầy đủ taxonomy của các engineering role — bao gồm trách nhiệm, quyền hạn, các hành vi bị cấm và execution context cho cả human actors và AI agents. Tất cả định nghĩa role đều kế thừa từ và bị ràng buộc bởi `kiro-core`. Không role nào được phép override, làm yếu đi hoặc mâu thuẫn với bất kỳ rule nào trong `kiro-core`.

---

## 📌 Tổng quan

Mục đích của `kiro-roles`:

* **Xác định ai làm gì** — mỗi role có boundary rõ ràng, tập quyền hạn và danh sách hành vi bị cấm.
* **Ràng buộc hành vi AI agent** — khi AI agent hoạt động dưới một role được khai báo, nó kế thừa toàn bộ định nghĩa role đó, bao gồm mọi constraint.
* **Hỗ trợ `kiro-cli` role-scoped loading** — mỗi role là một file `.ki` độc lập có thể load riêng mà không cần parse toàn module.
* **Loại bỏ ambiguity về role** — không tồn tại hai role cùng chia sẻ quyền quyết định trong cùng một domain.

| Thuộc tính              | Giá trị                                             |
| :---------------------- | :-------------------------------------------------- |
| **Layer**               | Secondary — kế thừa từ `kiro-core`                |
| **Format**              | `.ki` — file directive key-value tối ưu token       |
| **Mode**                | `machine_first` — tối ưu cho AI/LLM context parsing |
| **Permission model**    | `deny_by_default_allow_by_explicit_grant`           |
| **Conflict resolution** | `kiro-core` luôn thắng; ambiguity → `r01` (ask)     |
| **Versioning**          | SemVer — pinned theo `kiro-core`                  |

---

## 🗂 Cấu trúc file

```text
kiro-roles/
├── _index.ki          ← Manifest, role registry, system model, interaction rules
├── roles.ki           ← File tham chiếu monolithic (tất cả roles)
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

**`_index.ki`** là entry point của `kiro-cli`. Nó chứa:

* Role registry (`ROLE=<alias> FILE=roles/<alias>.ki CAT=<id>`)
* Role system model (RBAC axioms, permission model)
* Role categories và members
* Role interaction rules
* AI agent execution context rules
* Conflict resolution rules
* Versioning

**Các file role riêng lẻ** (`roles/*.ki`) là self-contained — mỗi file có thể được load độc lập bởi `kiro-cli` mà không cần file nào khác. Mỗi role file có header `META` gồm `ROLE`, `ALIAS`, `PARENT`, `INDEX`, `CATEGORY`.

---

## 🏗 Role System Model

`kiro-roles` sử dụng **RBAC** (Role-Based Access Control) mở rộng với execution context theo từng role.

| Khái niệm             | Giá trị                                                                 |
| :-------------------- | :---------------------------------------------------------------------- |
| **Model**             | `rbac` + responsibility + execution context                             |
| **Role unit**         | Actor nguyên tử có boundary xác định                                    |
| **Role scope**        | Áp dụng cho cả human agents và AI agents                                |
| **Permission model**  | `deny_by_default` — bắt buộc explicit grant                             |
| **Permission source** | Kế thừa từ `security.ki:SEC_CORE` + `principles.ki:SEC=least_privilege` |

**4 axioms bất biến:**

1. Không role nào được override constraint của `kiro-core`.
2. Không role nào được cấp quyền vượt quá boundary của chính nó.
3. Ambiguity trong boundary → áp dụng `glossary.ki:r01` (ask trước).
4. Role conflict → áp dụng priority order P01–P10 trong `principles.ki`.

---

## 🗃 Role Categories

| #  | Category                    | Alias    | Members (đã định nghĩa)                                                                              |
| :- | :-------------------------- | :------- | :--------------------------------------------------------------------------------------------------- |
| 01 | Strategy & Requirements     | `CAT_01` | `pm`, `po`, `ba`, `sm`, `agile_coach`, `ux_r`                                                        |
| 02 | Design & Experience         | `CAT_02` | `pd`, `uxui_e`, `ux_e`, `ui_e`, `ui_d`, `anim_d`                                                     |
| 03 | Core Engineering            | `CAT_03` | `swe`, `fe_e`, `be_e`, `fs_e`, `mob_e`, `ios_e`, `and_e`, `desktop_e`, `web3_e`, `game_e`, `embed_e` |
| 04 | Architecture & Leadership   | `CAT_04` | `arch`, `sol_arch`, `ent_arch`, `cloud_arch`, `tl`, `em`                                             |
| 05 | Data / AI / ML              | `CAT_05` | `de`, `ds`, `da`, `ml_e`, `ai_e`, `prompt_e`, `db_a`, `nlp_e`                                        |
| 06 | Quality & Testing           | `CAT_06` | `qa`, `qc`, `sdet`, `auto_qa`, `man_qa`, `perf_qa`, `rvw`, `gate_k`                                  |
| 07 | Infrastructure & Operations | `CAT_07` | `devops`, `sre`, `pe`, `sys_a`, `net_e`, `cloud_e`, `noc`                                            |
| 08 | Security & Compliance       | `CAT_08` | `sec`, `app_sec`, `cloud_sec`, `pen_t`, `aud`, `soc`, `iam_e`                                        |
| 09 | Specialized Support         | `CAT_09` | `rel_m`, `tech_w`, `dev_rel`, `loc_e`, `build_e`, `finops`                                           |

> Các role có file `.ki` trong `roles/` được **định nghĩa đầy đủ**. Các member còn lại chỉ được khai báo trong `_index.ki` và giữ trạng thái reserved cho triển khai tương lai.

---

## 📖 Role Reference

Mỗi role tuân theo template nghiêm ngặt:

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

* **Purpose:** Xác định product vision, ưu tiên backlog, align stakeholders.
* **Key permissions:** Tạo và prioritize ticket, approve scope change, define done state.
* **Key forbidden actions:** Bypass security review, approve breaking changes không có sign-off từ engineering, bỏ qua AC definition.
* **Core constraints:** `principles.ki:REQ,VALUE,DECIDE,COMM,PLAN` · `security.ki:DATA_SEC=minimize_pii` · `workflow.ki:TASK=no_ticket_no_code`
* **Execution tags:** `out_plan`, `ask`, `r01`, `r04`
* **Escalation:** Legal/compliance risk · Financial impact · Customer trust risk

---

#### `po` — Product Owner [`roles/po.ki`](roles/po.ki)

* **Purpose:** Đại diện business value trong agile team, quản lý sprint backlog.
* **Key permissions:** Accept/reject sprint deliverables, reprioritize sprint backlog.
* **Key forbidden actions:** Override security constraints, approve production deployment không qua gate, bỏ definition of done.
* **Core constraints:** `principles.ki:REQ,VALUE,PLAN` · `security.ki:SEC_CORE=deny_by_default` · `workflow.ki:TASK=definition_of_done`
* **Execution tags:** `out_plan`, `ask`, `r01`
* **Escalation:** Customer trust risk · Financial impact

---

#### `ba` — Business Analyst [`roles/ba.ki`](roles/ba.ki)

* **Purpose:** Chuyển business needs thành requirement có cấu trúc.
* **Key permissions:** Tạo requirement document, request clarification từ mọi role, flag requirement mâu thuẫn.
* **Key forbidden actions:** Đưa architecture decision, approve code change, bypass requirement review.
* **Core constraints:** `principles.ki:REQ,TRUTH,COMM` · `security.ki:DATA_SEC=classify_data_sensitivity` · `workflow.ki:TASK=definition_of_ready`
* **Execution tags:** `out_plan`, `ask`, `r01`, `r04`
* **Escalation:** Legal/compliance risk · Customer trust risk

---

#### `sm` — Scrum Master [`roles/sm.ki`](roles/sm.ki)

* **Purpose:** Facilitate agile ceremonies, remove blockers, bảo vệ team focus.
* **Key permissions:** Escalate blockers, thay đổi process, flag vi phạm workflow.
* **Key forbidden actions:** Product decision, override technical decision, approve code.
* **Core constraints:** `principles.ki:COMM,PLAN,CHANGE` · `workflow.ki:TASK,STATUS,REVIEW`
* **Execution tags:** `out_steps`, `ask`, `r01`
* **Escalation:** Blocker > 2 ngày · Velocity giảm > 30%

---

### CAT_03 — Core Engineering

#### `swe` — Software Engineer [`roles/swe.ki`](roles/swe.ki)

* **Purpose:** Thiết kế, implement, test và maintain hệ thống phần mềm.
* **Key permissions:** Viết code, merge qua PR, tạo branch, chạy CI, đề xuất refactor.
* **Key forbidden actions:** Commit trực tiếp main, bỏ qua test business logic, hardcode secrets, silent fail error.
* **Core constraints:** `principles.ki:P01–P10,CORRECT,MAINT,SEC,REL,TEST` · `security.ki:SECRET,IO_SEC,IAM` · `workflow.ki:VCS,PR,CI` · `coding.ki:FUNC,FLOW,STATE,ERR,MOD,ASYNC`
* **Execution tags:** `out_code`, `r03`, `r05`, `r09`, `r10`
* **Escalation:** Security breach signals · Data loss · Production outage

---

#### `fe_e` — Front-End Engineer [`roles/fe_e.ki`](roles/fe_e.ki)

* **Purpose:** Xây UI với correctness, performance và accessibility.
* **Key permissions:** Viết frontend code, đề xuất UX, request API change qua ticket.
* **Key forbidden actions:** Lưu secrets trong frontend, bỏ qua a11y, skip CSR/SSR review.
* **Core constraints:** `principles.ki:UX,CORRECT,MAINT,PERF` · `security.ki:IO_SEC=encode_data_on_output,SECRET=never_in_frontend` · `coding.ki:FUNC,FLOW,STATE,ASYNC`
* **Execution tags:** `out_code`, `r05`, `r10`
* **Escalation:** Customer trust risk · Critical a11y violation

---

#### `be_e` — Back-End Engineer [`roles/be_e.ki`](roles/be_e.ki)

* **Purpose:** Xây server-side logic, API, data access và integrations.
* **Key permissions:** Viết backend code, design DB query, schema change, service integration.
* **Key forbidden actions:** Skip input validation, dùng dynamic SQL concat, expose internal error, hardcode secrets.
* **Core constraints:** `principles.ki:P01–P10,CORRECT,SEC,REL` · `security.ki:IO_SEC,SECRET,IAM,DATA_SEC,AUDIT` · `coding.ki:FUNC,FLOW,ERR,MOD,ASYNC,DATA`
* **Execution tags:** `out_code`, `r03`, `r05`, `r08`, `r10`
* **Escalation:** Security breach signals · Data loss · Production outage

---

### CAT_04 — Architecture & Leadership

#### `tl` — Tech Lead [`roles/tl.ki`](roles/tl.ki)

* **Purpose:** Chịu trách nhiệm technical direction của team, đảm bảo quality và delivery.
* **Key permissions:** Approve architecture change, merge critical PR, define standards.
* **Key forbidden actions:** Override `kiro-core`, bypass security, merge khi CI fail.
* **Core constraints:** `principles.ki:P01–P10,DECIDE,PLAN,CHANGE,REVIEW` · `security.ki:SEC_CORE,IAM,AUDIT` · `workflow.ki:PR,CI,REVIEW`
* **Execution tags:** `out_plan`, `r02`, `r04`, `r05`, `r06`, `r10`
* **Escalation:** Security breach · Production outage · Financial impact

---

#### `arch` — Software Architect [`roles/arch.ki`](roles/arch.ki)

* **Purpose:** Thiết kế system architecture, đảm bảo scalability, security và alignment với `kiro-core`.
* **Key permissions:** Approve architecture lớn, define bounded context, mandate pattern.
* **Key forbidden actions:** Circular dependency, shared DB integration, bypass zero-trust model.
* **Core constraints:** `principles.ki:DECIDE,COMPLEXITY,MAINT,CHANGE,DEP` · `security.ki:SEC_ARCH,IAM,DATA_SEC` · `coding.ki:MOD=depend_on_abstractions`
* **Execution tags:** `out_plan`, `r02`, `r04`, `r05`, `r09`
* **Escalation:** Legal/compliance risk · Financial impact · Irreversible operation

---

### CAT_05 — Data / AI / ML

#### `de` — Data Engineer [`roles/de.ki`](roles/de.ki)

* **Purpose:** Xây và maintain data pipeline, warehouse và infrastructure data.
* **Key permissions:** Tạo pipeline, schema change, access non-prod data.
* **Key forbidden actions:** Access prod PII không phép, skip backup migration, dùng unvalidated data vào prod.
* **Core constraints:** `principles.ki:CORRECT,REL,PERF,SEC` · `security.ki:DATA_SEC,SECRET,AUDIT` · `coding.ki:FUNC,ERR,DATA`
* **Execution tags:** `out_code`, `r07`, `r08`, `r10`
* **Escalation:** Data loss · Financial impact

---

#### `ml_e` — Machine Learning Engineer [`roles/ml_e.ki`](roles/ml_e.ki)

* **Purpose:** Thiết kế, train, evaluate và deploy ML model.
* **Key permissions:** Access training data, deploy staging model, propose feature change.
* **Key forbidden actions:** Deploy model chưa validate, dùng PII không consent, skip bias evaluation.
* **Core constraints:** `principles.ki:TRUTH,CORRECT,SEC,PERF` · `security.ki:DATA_SEC=minimize_pii,AUDIT` · `coding.ki:FUNC,ERR,DATA`
* **Execution tags:** `out_code`, `r07`, `r08`, `r10`
* **Escalation:** Customer trust risk · Legal risk · Model bias

---

#### `ai_e` — AI Engineer [`roles/ai_e.ki`](roles/ai_e.ki)

* **Purpose:** Thiết kế và triển khai AI systems, integrations và agents.
* **Key permissions:** Access LLM API, deploy AI feature staging, propose model change.
* **Key forbidden actions:** Deploy AI không validation, skip safety evaluation, expose system prompt, dùng PII trong prompt không consent.
* **Core constraints:** `principles.ki:TRUTH,CORRECT,SEC,REL` · `security.ki:SECRET,IO_SEC,DATA_SEC,AUDIT` · `coding.ki:FUNC,ERR,ASYNC`
* **Execution tags:** `out_code`, `r05`, `r09`, `r10`
* **Escalation:** Customer trust risk · Legal risk · Security breach

---

#### `prompt_e` — Prompt Engineer [`roles/prompt_e.ki`](roles/prompt_e.ki)

* **Purpose:** Thiết kế, đánh giá và tối ưu prompt cho AI system.
* **Key permissions:** Tạo/version prompt, chạy evaluation, propose model change.
* **Key forbidden actions:** Deploy prompt không safety evaluation, public system prompt, skip versioning.
* **Core constraints:** `principles.ki:TRUTH,CORRECT,SEC` · `security.ki:SECRET,IO_SEC` · `coding.ki:DOC`
* **Execution tags:** `out_code`, `r05`, `r10`
* **Escalation:** Customer trust risk · Safety failure

---

### CAT_06 — Quality & Testing

#### `qa` — Quality Assurance [`roles/qa.ki`](roles/qa.ki)

* **Purpose:** Verify system behavior theo AC và quality gates.
* **Key permissions:** Block release khi fail quality, tạo bug ticket, request test env.
* **Key forbidden actions:** Approve release có P0/P1 bug, skip regression critical path, mark flaky test là pass.
* **Core constraints:** `principles.ki:CORRECT,TEST,REL` · `security.ki:DATA_SEC=mask_in_non_production` · `workflow.ki:TASK=definition_of_done,CI=GATE`
* **Execution tags:** `out_steps`, `r03`, `r10`
* **Escalation:** Customer trust risk · Production outage · P0 bug in prod

---

#### `gate_k` — Quality Gatekeeper [`roles/gate_k.ki`](roles/gate_k.ki)

* **Purpose:** Enforce quality gate, chặn deliverable không đạt chuẩn.
* **Key permissions:** Block merge PR, request changes, escalate violation.
* **Key forbidden actions:** Approve PR khi CI fail, bypass security gate, self-approve PR.
* **Core constraints:** `principles.ki:REVIEW,CORRECT,SEC` · `security.ki:SEC_CORE,AUDIT` · `workflow.ki:PR,CI,REVIEW`
* **Execution tags:** `out_steps`, `r05`, `r10`
* **Escalation:** Security breach · P0 vulnerability · CI bypass attempt

---

### CAT_07 — Infrastructure & Operations

#### `devops` — DevOps Engineer [`roles/devops.ki`](roles/devops.ki)

* **Purpose:** Xây CI/CD, infrastructure và automation delivery.
* **Key permissions:** Manage pipeline, provision non-prod infra, deploy staging, config monitoring.
* **Key forbidden actions:** Deploy prod không manual gate, lưu secrets plaintext, disable security scan.
* **Core constraints:** `principles.ki:OPS,REL,SEC` · `security.ki:SECRET,INFRA_SEC,AUDIT` · `workflow.ki:CI,CD,RELEASE,HOTFIX`
* **Execution tags:** `out_steps`, `r06`, `r08`, `r10`
* **Escalation:** Production outage · Data loss · Security breach

---

#### `sre` — Site Reliability Engineer [`roles/sre.ki`](roles/sre.ki)

* **Purpose:** Đảm bảo reliability, availability, performance production system.
* **Key permissions:** Trigger incident, escalate SEV0/SEV1, enforce SLO, request architecture change.
* **Key forbidden actions:** Silence alert không root cause, skip postmortem SEV1/SEV2, disable circuit breaker.
* **Core constraints:** `principles.ki:REL,OPS,PERF,TRUTH` · `security.ki:AUDIT,INFRA_SEC` · `workflow.ki:HOTFIX,INCIDENT`
* **Execution tags:** `out_steps`, `r06`, `r07`, `r10`
* **Escalation:** Production outage · Data loss · Financial impact

---

### CAT_08 — Security & Compliance

#### `sec` — Security Engineer [`roles/sec.ki`](roles/sec.ki)

* **Purpose:** Bảo vệ system, data và user thông qua security design và review.
* **Key permissions:** Block release khi có vuln, enforce security control, access audit log.
* **Key forbidden actions:** Custom crypto, secret trong code, bypass MFA, suppress audit log.
* **Core constraints:** `principles.ki:P03=security,SEC,TRUTH` · `security.ki:SEC_CORE,SECRET,IAM,DATA_SEC,IO_SEC,INFRA_SEC,AUDIT`
* **Execution tags:** `out_steps`, `r05`, `r06`, `r10`
* **Escalation:** Security breach · Data loss · Legal risk

---

#### `aud` — Compliance Auditor [`roles/aud.ki`](roles/aud.ki)

* **Purpose:** Kiểm tra compliance theo regulatory và internal policy.
* **Key permissions:** Read audit logs, review access control, request evidence.
* **Key forbidden actions:** Modify audit log, approve non-compliant system, skip evidence.
* **Core constraints:** `principles.ki:TRUTH,SEC,REVIEW` · `security.ki:AUDIT,IAM,DATA_SEC` · `workflow.ki:TASK=definition_of_done`
* **Execution tags:** `out_full`, `r05`, `r10`
* **Escalation:** Legal/compliance risk · Security breach · Audit tampering

---

### CAT_09 — Specialized Support

#### `tech_w` — Technical Writer [`roles/tech_w.ki`](roles/tech_w.ki)

* **Purpose:** Viết và maintain technical documentation.
* **Key permissions:** Create/update docs, request review, flag outdated docs.
* **Key forbidden actions:** Publish docs chưa review, document behavior chưa verify, skip versioning API docs.
* **Core constraints:** `principles.ki:TRUTH,COMM,DOC` · `security.ki:DATA_SEC=never_document_secrets` · `coding.ki:DOC`
* **Execution tags:** `out_full`, `r01`, `r10`
* **Escalation:** Doc conflict code · API breaking change không update docs

---

#### `rel_m` — Release Manager [`roles/rel_m.ki`](roles/rel_m.ki)

* **Purpose:** Điều phối và kiểm soát release process.
* **Key permissions:** Approve production deploy, trigger rollback, block release.
* **Key forbidden actions:** Deploy không rollback plan, skip checklist, override CI fail.
* **Core constraints:** `principles.ki:REL,CHANGE,OPS` · `security.ki:AUDIT,INFRA_SEC` · `workflow.ki:CD,RELEASE,HOTFIX`
* **Execution tags:** `out_steps`, `r02`, `r06`, `r10`
* **Escalation:** Production outage · Financial impact · Rollback required

---

## 🔗 Role Interaction Rules

Roles không hoạt động độc lập. Interaction rules:

| Trigger                   | Rule                                          |
| :------------------------ | :-------------------------------------------- |
| `pm` request scope change | `tl` phải đánh giá technical impact trước     |
| `arch` mandate pattern    | `tl` và `swe` phải implement                  |
| `sec` block release       | Chỉ `sec` hoặc cấp cao hơn được unblock       |
| `qa` block release        | `po` hoặc `pm` có thể accept risk có sign-off |
| `gate_k` block PR         | Author phải fix, không được bypass            |
| `sre` declare incident    | Tất cả role defer cho incident commander      |
| `aud` request evidence    | Tất cả role phải cung cấp trong SLA           |
| `ai_e` deploy model       | `sec` và `qa` phải sign-off                   |
| `de` migration            | `rel_m` và `sre` phải được notify             |
| `tl` approve ADR          | `arch` review nếu cross-domain                |

**Escalation path:** `role` → `tl` → `arch` → `em` → `sec` (nếu liên quan security)

**Permission boundaries:**

* Role không được cấp quyền vượt quá quyền của chính nó.
* Role không được truy cập ngoài scope.
* AI agent inherit nguyên trạng permissions + forbidden actions.

---

## 🤖 AI Agent Execution Context

Khi AI agent chạy dưới `kiro-roles`, áp dụng các rule:

| Rule                         | Behavior                      |
| :--------------------------- | :---------------------------- |
| Must declare active role     | Không có role → không execute |
| Inherit full role definition | Bao gồm forbidden_actions     |
| Không được self-elevate      | Role là hard ceiling          |
| Ambiguity → r01              | Phải hỏi trước khi làm        |
| TRUTH luôn active            | Theo `principles.ki`          |
| Conflict phải được surface   | Không tự resolve ngầm         |
| Vi phạm forbidden_actions    | Phải từ chối                  |
| Security-sensitive action    | Áp dụng r05 review            |
| Production change            | Áp dụng r06 rollback plan     |
| Output unverifiable          | Gắn nhãn `unverified`         |

---

## 🚀 Sử dụng với `kiro-cli`

### Load một role

```bash
kiro-cli role load swe
# đọc roles/swe.ki + áp dụng kiro-core constraints
```

### Tra cứu role

```bash
kiro-cli role info be_e
# resolve qua _index.ki rồi load file tương ứng
```

### Load theo category

```bash
kiro-cli role load --cat 08
```

---

### Cách `_index.ki` hoạt động

`kiro-cli` luôn đọc `_index.ki` trước:

```
_role registry → mapping alias → file → category
_role system model → RBAC + permission model
_interaction rules → cross-role constraints
_ai execution context → behavior rules
_conflict resolution → tie-breaking
_versioning → SemVer
```

---

## ⚖️ Conflict Resolution

| Conflict                        | Resolution                       |
| :------------------------------ | :------------------------------- |
| Role vs `kiro-core`             | `kiro-core` luôn thắng           |
| Ambiguity role boundary         | r01 (ask)                        |
| Two role same authority         | `tl` (technical), `pm` (product) |
| Business pressure vs constraint | `principles.ki` P01–P10          |
| Security vs delivery            | Reject nếu không có signed risk  |
| AI role conflict                | Refuse execution                 |

---

## 🛠 Đóng góp

Vì kế thừa `kiro-core`, mọi thay đổi phải tuân thủ:

1. **New role** → thêm file `.ki`, register trong `_index.ki` → bump `MINOR`
2. **Change role** → update file + changelog → security review nếu đổi permission
3. **Breaking change** → RFC + `arch` + `sec` approval → bump `MAJOR`
4. Không bao giờ override hoặc làm yếu `kiro-core`

**Versioning:**

| Change                            | Version |
| :-------------------------------- | :------ |
| Breaking permission/role boundary | MAJOR   |
| Add role / non-breaking change    | MINOR   |
| Fix/clarification                 | PATCH   |

---

*© 2026 Kiro Suite. Chuẩn hóa tương lai của collaboration giữa con người và máy móc.*