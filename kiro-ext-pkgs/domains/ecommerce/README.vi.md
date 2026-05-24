# ecommerce-domain-pack

[[English] README.md](README.md)

## Mục đích

`ecommerce-domain-pack` là một extension AI steering lớp thứ ba trong hệ sinh thái kiro-suite, nằm tại `kiro-ext-pkgs/domains/ecommerce/`. Nó tạo ra các file `.ki` dạng machine-readable để điều hướng hành vi của AI agent trong bối cảnh thương mại điện tử — nó không sinh ra code ứng dụng.

Gói này mã hóa các business rules đặc thù ecommerce, entity models, state machines, workflow constraints, security extensions và role-scoped permissions. Khi một AI agent load gói này, nó sẽ kế thừa toàn bộ constraint và áp dụng chúng khi sinh code, schema hoặc các quyết định kiến trúc cho hệ thống ecommerce.

Các nền tảng mục tiêu:

* **B2C storefronts** — trải nghiệm mua sắm hướng khách hàng
* **B2B procurement portals** — luồng mua hàng doanh nghiệp và chuỗi phê duyệt
* **Marketplace platforms** — tách đơn hàng multi-seller và routing fulfillment
* **Headless commerce backends** — dịch vụ commerce API-first tách rời presentation

---

## Phạm vi (Scope)

### Các loại hệ thống trong phạm vi

| Hệ thống              | Mô tả                                                  |
| --------------------- | ------------------------------------------------------ |
| `storefront`          | UI phía khách hàng và quản lý session                  |
| `cart_service`        | vòng đời giỏ hàng, quản lý item, kiểm tra tồn kho mềm  |
| `order_service`       | tạo đơn hàng, lifecycle, giảm giá, coupon              |
| `payment_service`     | payment intents, capture, hoàn tiền, tín hiệu gian lận |
| `fulfillment_service` | đơn vị fulfillment, thông báo kho, tracking vận chuyển |
| `catalog_service`     | sản phẩm, biến thể, danh mục, bản ghi tồn kho          |

### Các thành phần ngoài phạm vi

Các hệ thống sau bị loại trừ rõ ràng. AI agent KHÔNG được áp dụng constraint của gói này cho các domain sau:

* `ERP_integration`
* `accounting_ledger`
* `HR_systems`
* `logistics_carrier_internals`
* `ad_tech_platforms`

---

## Chuỗi kế thừa (Inheritance Chain)

Gói nằm trong chuỗi kế thừa 3 lớp. Mỗi lớp chỉ được phép thêm constraint — không được nới lỏng hoặc định nghĩa lại bất kỳ thứ gì từ lớp cha.

```
kiro-core                                       (primary,   LAYER=primary)
    └── kiro-ext-pkgs                           (secondary, LAYER=secondary)
            └── domains/ecommerce-domain-pack   (tertiary,  LAYER=tertiary)
```

| Thuộc tính            | Giá trị                        |
| --------------------- | ------------------------------ |
| `OVERRIDE_POLICY`     | `kiro-core_always_wins`        |
| `CONFLICT_RESOLUTION` | `escalate_per_glossary.ki:r01` |

`OVERRIDE_POLICY=kiro-core_always_wins` áp dụng xuyên suốt chuỗi. Gói này chỉ được thêm constraint đặc thù domain trên nền các khai báo từ kiro-core và kiro-roles. Không được nới lỏng, định nghĩa lại hoặc override bất kỳ thứ gì đã có.

Khi một directive key của domain trùng với key của kiro-core, gói sẽ emit `CONFLICT_SIGNAL` và dừng. Operator phải tự xử lý conflict một cách rõ ràng — không được tự động giải quyết.

---

## Danh mục file (File Index)

Tất cả file `.ki` phải được đăng ký trong `_index.ki` FILE_REGISTRY trước khi được coi là active. File được chia thành 3 tier ưu tiên.

| File             | Priority | Mục đích                                                          | Key Directives                                                        |
| ---------------- | -------- | ----------------------------------------------------------------- | --------------------------------------------------------------------- |
| `_index.ki`      | P0       | manifest gói, dependency, load order, registry ưu tiên            | `LOAD_ORDER`, `FILE_REGISTRY`, `PRIORITY`, `SEMVER`                   |
| `glossary.ki`    | P0       | alias thuật ngữ domain mở rộng từ `kiro-core/glossary.ki`         | `EXTENDS=kiro-core/glossary.ki`, `term=definition`                    |
| `entities.ki`    | P0       | 18 entity canonical với ownership, fields, cross-reference rules  | `ENTITY`, `OWNER`, `REQUIRED_FIELDS`, `CROSS_REF_POLICY`              |
| `states.ki`      | P0       | state machine cho 7 entity có state                               | `STATE_MACHINE`, `STATES`, `TERMINAL_STATES`, `TRANSITION`            |
| `rules.ki`       | P0       | business rule: pricing, inventory, discount, tax, order lifecycle | `RULE=condition->action`, `DISCOUNT_ORDER`                            |
| `security.ki`    | P0       | mở rộng PCI-DSS và GDPR trên `kiro-core/security.ki`              | `EXTENDS=kiro-core/security.ki`, `PII_FIELDS`, `RULE=raw_card_data_*` |
| `reliability.ki` | P1       | saga pattern, idempotency, retry policy, circuit breaker          | `SAGA_STEP`, `COMPENSATE`, `BACKOFF`, `MAX_RETRIES`                   |
| `workflows.ki`   | P1       | 6 workflow ecommerce canonical và chuỗi step                      | `WORKFLOW`, `STEP`, `EMITS_EVENT`, `CONSTRAINT`                       |
| `roles.ki`       | P1       | role extension domain cho 13 kiro-roles                           | `ROLE_EXTENSION`, `BASE`, `DOMAIN_FORBIDDEN`, `DOMAIN_ESC`            |
| `edge-cases.ki`  | P2       | failure mode và edge case constraints                             | `RULE=failure_condition->remediation_action`                          |

**Quy tắc tier:**

* P1 KHÔNG được merge trước khi toàn bộ P0 được review xong.
* P2 KHÔNG được merge trước khi toàn bộ P1 được review xong.
* File P0 mà reference P1/P2 chưa tồn tại phải dùng `TBD=<description> TICKET=<ref>`.

---

## Cách sử dụng (Usage)

### Thứ tự load

AI agent khi dùng gói này PHẢI load theo đúng thứ tự:

```
1. kiro-core          (all files)
2. kiro-roles         (all files)
3. kiro-ext-pkgs/domains/ecommerce/_index.ki
4. Các file trong pack theo _index.ki
```

File `_index.ki` enforce bằng `LOAD_ORDER` và `REQUIRES`. Load sai thứ tự hoặc thiếu dependency sẽ tạo `MISSING_DEP` và dừng.

---

### Chức năng của gói

Gói này điều hướng hành vi AI agent — không sinh code ứng dụng. Khi được load, nó ép agent:

* Dùng canonical entity model và ownership boundaries
* Enforce state machine transition hợp lệ
* Áp dụng business rules cho pricing, inventory, discount, tax
* Mở rộng security rules từ kiro-core với PCI-DSS và GDPR
* Tuân thủ saga pattern và idempotency cho distributed operations
* Emit domain events tại mọi bước workflow

---

### Hành vi khi thiếu dependency

Nếu `kiro-core` hoặc `kiro-roles` chưa được load trước, `_index.ki` sẽ emit `MISSING_DEP` và dừng.

```
MISSING_DEP=kiro-core_required_before_pack_activation
MISSING_DEP=kiro-roles_required_before_pack_activation
```

---

### Hành vi conflict signal

Khi parser gặp directive trùng key với kiro-core, hệ thống coi là conflict và emit `CONFLICT_SIGNAL` thay vì tự quyết.

```
CONFLICT_SIGNAL=<domain_file>:<key> conflicts with kiro-core/<file>:<key>
```

---

### Yêu cầu file registry

File `.ki` chỉ được coi là active nếu đã đăng ký trong `_index.ki` FILE_REGISTRY. File không đăng ký sẽ bị bỏ qua.

```
FILE_REGISTRY=<filename>.ki  PRIORITY=<tier>  STATUS=active
```

---

## Phiên bản (Versioning)

```
SEMVER=major.minor.patch
VERSION=1.0.0
```

| Loại thay đổi                        | Tăng version | Yêu cầu sign-off |
| ------------------------------------ | ------------ | ---------------- |
| Breaking change entity/state machine | MAJOR        | arch             |
| Thêm entity/rule (non-breaking)      | MINOR        | tl review        |
| Sửa documentation                    | PATCH        | —                |

Quy tắc bổ sung:

* Mọi thay đổi `security.ki` cần sign-off từ role `sec`
* Không merge P1/P2 khi tier trên chưa hoàn tất review
* Mỗi version bump bắt buộc có changelog
* Khi `kiro-core` bump version, pack này phải review tương thích

---

## Hướng mở rộng (Extension Guide)

### Cấu trúc thư mục

```
kiro-ext-pkgs/domains/ecommerce/<sub-domain>/
```

Ví dụ:

* `marketplace/`
* `subscription/`
* `b2b/`

---

### Khai báo bắt buộc

Mỗi sub-domain `_index.ki` PHẢI khai báo:

```
PARENT=kiro-ext-pkgs/domains/ecommerce@<version>
LAYER=quaternary
OVERRIDE_POLICY=parent_always_wins
```

---

### Quy tắc override

`OVERRIDE_POLICY=parent_always_wins` áp dụng: rule của pack cha luôn ưu tiên hơn sub-domain. Sub-domain chỉ được thêm constraint, không được nới lỏng rule cha.

```
RULE=subdomain_pack_conflicts_with_parent_pack_rule->apply_parent_always_wins_and_surface_conflict
RULE=subdomain_pack_relaxes_parent_pack_rule->block_not_permitted
```