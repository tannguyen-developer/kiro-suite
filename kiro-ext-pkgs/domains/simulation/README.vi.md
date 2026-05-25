# simulation-domain-pack

[[English] README.md](README.md)

## Mục đích

`simulation-domain-pack` là một AI steering extension lớp thứ ba trong hệ sinh thái kiro-suite, nằm tại `kiro-ext-pkgs/domains/simulation/`. Nó tạo ra các file `.ki` ở dạng machine-readable để điều hướng hành vi AI agent trong các ngữ cảnh simulation system — nó không sinh ra code ứng dụng.

Pack này ưu tiên **model-fidelity-first**: tính đúng đắn của simulation model được ưu tiên hơn execution performance, entertainment output, hoặc ML environment scaffolding. Khi một AI agent load pack này, nó kế thừa toàn bộ constraint và áp dụng chúng khi generate code, schema, hoặc architectural decision cho simulation system.

Các loại simulation mục tiêu:

* **Physics simulations** — computational model của physical system và phenomena
* **Traffic simulations** — transportation và mobility flow model
* **Economic simulations** — market, financial, và economic process model
* **System dynamics** — stock-and-flow feedback loop model của aggregate behavior
* **Agent-based simulations** — emergent behavior phát sinh từ individual agent rule
* **Discrete event simulations (DES)** — state change chỉ xảy ra tại discrete event time (MVP simulation type)

---

## Phạm vi

### Các loại System thuộc phạm vi

| System                      | Mô tả                                                                                                   |
| --------------------------- | ------------------------------------------------------------------------------------------------------- |
| `physics_simulation`        | Computational model của physical system được điều khiển bởi declared physical law và boundary condition |
| `traffic_simulation`        | Transportation và mobility flow model đại diện cho real-world movement pattern và infrastructure        |
| `economic_simulation`       | Market, financial, và economic process model với declared agent behavior và market rule                 |
| `system_dynamics`           | Stock-and-flow feedback loop model đại diện cho aggregate behavior theo thời gian                       |
| `agent_based_simulation`    | Model nơi emergent behavior phát sinh từ individual Agent rule và local interaction                     |
| `discrete_event_simulation` | Model nơi state change chỉ xảy ra tại discrete Event time; MVP simulation type                          |

Một system được xem là thuộc phạm vi khi **mục tiêu chính là fidelity đối với real-world
process model**, không phải entertainment output hoặc ML environment scaffolding.

### Các concern ngoài phạm vi

Các domain sau đây bị loại trừ rõ ràng. AI agent KHÔNG ĐƯỢC áp dụng constraint của pack này
cho các domain này:

| Domain                                                                 | Giải thích                                                                                    |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `game_engine_rendering`                                                | Mục tiêu chính là visual output và frame rate, không phải process model fidelity              |
| `real_time_graphics`                                                   | Rendering pipeline được tối ưu cho perceptual quality, không phải physical accuracy           |
| `game_ai_behavior_trees`                                               | Behavior tree được tối ưu cho entertainment challenge, không phải real-world process modeling |
| `entertainment_physics_approximations`                                 | Physics approximation được tinh chỉnh cho feel và responsiveness, không phải fidelity         |
| `ml_training_environments_without_explicit_model_fidelity_requirement` | RL environment không có declared real-world process model nằm ngoài phạm vi                   |

---

## Chuỗi kế thừa

Pack này nằm trong chuỗi kế thừa ba tầng. Mỗi tầng chỉ được phép thêm constraint —
không bao giờ được nới lỏng hoặc định nghĩa lại bất kỳ thứ gì được khai báo bởi parent layer.

```text
kiro-core                                             (primary,    LAYER=primary)
    └── kiro-ext-pkgs                                 (secondary,  LAYER=secondary)
            └── simulation-domain-pack                (tertiary,   LAYER=tertiary)
                    └── sub-domain packs (tương lai)  (quaternary, LAYER=quaternary)
                        simulation/physics
                        simulation/traffic
                        simulation/economic
```

| Property              | Giá trị                        |
| --------------------- | ------------------------------ |
| `OVERRIDE_POLICY`     | `kiro-core_always_wins`        |
| `CONFLICT_RESOLUTION` | `escalate_per_glossary.ki:r01` |

`OVERRIDE_POLICY=kiro-core_always_wins` áp dụng ở mọi cấp trong chuỗi. Pack này chỉ được
phép thêm domain-specific constraint dựa trên những gì kiro-core và kiro-roles khai báo. Nó
không bao giờ được phép nới lỏng, định nghĩa lại, hoặc override bất kỳ thứ gì đã tồn tại trong
kiro-core hoặc kiro-roles.

Khi một domain directive key trùng với một kiro-core directive key, pack sẽ phát ra
`CONFLICT_SIGNAL` và dừng. Operator phải resolve conflict một cách explicit — không cho phép
silent resolution.

---

## File Index

Mọi file `.ki` phải được đăng ký trong `_index.ki` `FILE_REGISTRY` trước khi được xem là active.
Các file được tổ chức thành ba priority tier xác định implementation order và review order.

| File             | Priority | Mục đích                                                                                             | Key Directive                                                                                       |
| ---------------- | -------- | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `_index.ki`      | P0       | Pack manifest, dependency declaration, load order, scope, priority registry, versioning              | `LOAD_ORDER`, `FILE_REGISTRY`, `PRIORITY`, `SCOPE_IN`, `SCOPE_OUT`, `SEMVER`                        |
| `glossary.ki`    | P0       | Domain-specific term alias mở rộng `kiro-core/glossary.ki`; time semantic; conflict resolution alias | `EXTENDS=kiro-core/glossary.ki`, `term=definition` alias, `r01`                                     |
| `entities.ki`    | P0       | 13 canonical entity với ownership, field, immutability, và cross-reference rule                      | `ENTITY`, `OWNER`, `REQUIRED_FIELDS`, `CROSS_REF_POLICY`, `APPEND_ONLY`, `IMMUTABLE_AFTER_TERMINAL` |
| `states.ki`      | P0       | State machine definition cho 5 stateful simulation entity                                            | `STATE_MACHINE`, `STATES`, `TERMINAL_STATES`, `TRANSITION`                                          |
| `rules.ki`       | P0       | Correctness, fidelity, reproducibility, numerical precision, time semantic, và scope boundary rule   | `RULE=condition->action`, `AXIOM`, `REUSE`, `run_hash_computation`                                  |
| `reliability.ki` | P1       | Checkpointing, run isolation, observation durability, cancellation, và lifecycle observability       | `CHECKPOINT_THRESHOLD`, `LIFECYCLE_EVENT`, `OBSERVATION_WRITE_CONSTRAINT`                           |
| `workflows.ki`   | P1       | 6 canonical simulation lifecycle workflow phase và step sequence                                     | `WORKFLOW`, `STEP`, `EMITS_EVENT`, `CONSTRAINT`                                                     |
| `roles.ki`       | P1       | Domain-scoped role extension cho 11 kiro-roles với forbidden action và escalation trigger            | `ROLE_EXTENSION`, `BASE`, `DOMAIN_FORBIDDEN`, `DOMAIN_ESC`, `SIGNOFF_REQUIRED`                      |
| `edge-cases.ki`  | P2       | Numerical safety rule, model divergence và drift constraint, error signal format table               | `RULE=failure_condition->remediation_action`, `ERROR_SIGNAL`                                        |

**Quy tắc tier:**

* File P1 KHÔNG ĐƯỢC merge trước khi tất cả file P0 đã pass review.
* File P2 KHÔNG ĐƯỢC merge trước khi tất cả file P1 đã pass review.
* Một file P0 tham chiếu tới file P1 hoặc P2 chưa được implement PHẢI khai báo reference đó
  dưới dạng `TBD=<description> TICKET=<ref>` thay vì để undefined.

---

## Cách sử dụng

### Load Order

Một AI agent sử dụng pack này PHẢI load dependency theo đúng thứ tự sau:

```text
1. kiro-core          (all files)
2. kiro-roles         (all files)
3. kiro-ext-pkgs/domains/simulation/_index.ki
4. Individual pack files as declared in _index.ki
```

File `_index.ki` enforce điều này thông qua directive `LOAD_ORDER` và `REQUIRES`. Việc load
pack file sai thứ tự hoặc trước vị trí được khai báo sẽ khiến `_index.ki` phát ra
`MISSING_DEP` và dừng.

### Pack này làm gì

Pack này điều hướng hành vi AI agent — nó không tạo ra application code. Khi được load, nó
constraint agent để:

* Áp dụng nguyên tắc model-fidelity-first: correctness luôn ưu tiên hơn performance
* Sử dụng canonical 13-entity model với ownership boundary và cross-reference rule
* Enforce valid state machine transition cho mọi stateful simulation entity
* Yêu cầu explicit `Assumption` declaration trước khi bất kỳ `Simulation_Run` nào được khởi tạo
* Enforce reproducibility thông qua `Seed`, `run_hash`, và pinned model version
* Áp dụng numerical precision, divergence detection, và NaN/infinity halt rule
* Tuân theo 6-phase simulation lifecycle workflow với event emission tại mọi step
* Merge base kiro-roles definition với domain-scoped forbidden action và escalation trigger

### Hành vi khi thiếu dependency

Nếu `kiro-core` hoặc `kiro-roles` chưa được load trước khi pack này được activate, `_index.ki`
sẽ phát ra `MISSING_DEP` và dừng. Agent phải load dependency bị thiếu và retry.

```text
MISSING_DEP=kiro-core_required_before_pack_activation
MISSING_DEP=kiro-roles_required_before_pack_activation
```

### Hành vi Conflict Signal

Khi directive parser của pack gặp một directive key cũng tồn tại trong kiro-core
hoặc kiro-roles, nó phân loại tình huống đó là conflict và phát ra `CONFLICT_SIGNAL` thay vì
silent apply bất kỳ rule nào. Operator phải resolve conflict một cách explicit trước khi
pack được xem là active.

```text
CONFLICT_SIGNAL=<domain_file>:<key> conflicts with <parent_pack>/<file>:<key>
```

### Yêu cầu File Registry

Một file `.ki` phải được đăng ký trong `_index.ki` `FILE_REGISTRY` trước khi được xem là active.
Các file chưa đăng ký sẽ bị pack loader bỏ qua, ngay cả khi chúng tồn tại trên disk.

```text
FILE_REGISTRY=<filename>.ki  PRIORITY=<tier>  STATUS=active
```

---

## Versioning

```text
SEMVER=major.minor.patch
VERSION=1.0.0
```

| Change Type                                                  | Version Bump | Sign-off Required |
| ------------------------------------------------------------ | ------------ | ----------------- |
| Breaking change đối với entity definition hoặc state machine | `MAJOR`      | `arch` sign-off   |
| Thêm entity hoặc rule mới (non-breaking)                     | `MINOR`      | `tl` review       |
| Clarification hoặc documentation fix                         | `PATCH`      | —                 |

Quy tắc bổ sung:

* Bất kỳ state machine transition nào được thêm hoặc xóa đều yêu cầu `arch` sign-off và `MAJOR` bump.
* Bất kỳ entity mới nào được thêm vào `entities.ki` đều yêu cầu `tl` review và `MINOR` bump.
* Không file P1 hoặc P2 nào được merge trước khi mọi file ở tier phía trên đã pass review.
* Một version bump luôn yêu cầu changelog entry.
* Khi version `kiro-core` được bump, pack này phải được review để kiểm tra compatibility.

---

## Extension Guide

Sub-domain pack mở rộng pack này cho các simulation vertical chuyên biệt. Chúng tuân theo cùng
override model được pack này sử dụng tương đối với kiro-core.

### Cấu trúc thư mục

Sub-domain pack nằm dưới:

```text
kiro-ext-pkgs/domains/simulation/<sub-domain>/
```

Ví dụ:

* `kiro-ext-pkgs/domains/simulation/physics/`
* `kiro-ext-pkgs/domains/simulation/traffic/`
* `kiro-ext-pkgs/domains/simulation/economic/`

### Required Declaration

Mọi `_index.ki` của sub-domain pack PHẢI khai báo:

```text
PARENT=kiro-ext-pkgs/domains/simulation@<version>
LAYER=quaternary
OVERRIDE_POLICY=parent_always_wins
```

Child pack tuân theo cùng META block convention và file registration requirement như
pack này. Mọi file `.ki` trong child pack phải được đăng ký trong chính
`_index.ki FILE_REGISTRY` của child pack trước khi được xem là active.

### Override Policy

`OVERRIDE_POLICY=parent_always_wins` được áp dụng: rule của pack này có precedence cao hơn
rule của child pack. Sub-domain pack chỉ được phép thêm constraint; chúng không bao giờ
được phép nới lỏng parent rule.

```text
RULE=subdomain_pack_conflicts_with_parent_pack_rule->apply_parent_always_wins_and_surface_conflict
RULE=subdomain_pack_relaxes_parent_pack_rule->block_not_permitted
```

Điều này phản chiếu cùng model được pack này sử dụng tương đối với kiro-core.

### Deferred Complexity

Các item sau được defer sang post-MVP và là candidate cho sub-domain pack
implementation trong future version. Chúng KHÔNG ĐƯỢC tham chiếu trong file P0 hoặc P1 nếu không có
khai báo `TBD`:

| Deferred Item                       | Ghi chú                                                                                                                      |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `multi_agent_learning`              | Reinforcement learning bên trong simulation agent; yêu cầu explicit model fidelity requirement để đủ điều kiện thuộc phạm vi |
| `continuous_physics_integration`    | Continuous ODE/PDE solver; bị defer do coupling với continuous math không được cover bởi DES MVP rule                        |
| `real_time_co_simulation`           | Synchronized execution trên nhiều simulation engine theo thời gian thực; yêu cầu distributed coordination rule               |
| `distributed_multi_node_simulation` | Multi-node execution với distributed state; yêu cầu distributed transaction và consensus rule vượt ngoài MVP scope           |

---

## Deferred Complexity

Các item post-MVP được defer khỏi `VERSION=1.0.0`:

* **`multi_agent_learning`** — Reinforcement learning và adaptive agent behavior bên trong
  simulation context. Yêu cầu explicit model fidelity requirement để tiếp tục thuộc phạm vi.
  Candidate cho `simulation/agent-learning` sub-domain pack.

* **`continuous_physics_integration`** — Continuous ODE/PDE numerical solver (ví dụ:
  Runge-Kutta, finite element method). Bị defer do coupling với continuous mathematics
  không được cover bởi DES-focused MVP rule. Candidate cho `simulation/physics` sub-domain pack.

* **`real_time_co_simulation`** — Synchronized execution trên nhiều heterogeneous
  simulation engine theo thời gian thực (ví dụ: FMI/FMU co-simulation). Yêu cầu distributed
  coordination và synchronization rule vượt ngoài MVP scope.

* **`distributed_multi_node_simulation`** — Multi-node execution với distributed state
  management, partitioned State Vector, và cross-node consistency guarantee. Yêu cầu
  distributed transaction và consensus rule. Candidate cho `simulation/distributed`
  sub-domain pack.