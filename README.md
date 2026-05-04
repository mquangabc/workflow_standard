---
doc_type: workflow_standard
version: 1.0
last_updated: 2026-05-04
audience: [PO, BA, Designer, Dev, QA, Tech Lead]
purpose: Quy chuẩn tài liệu và bàn giao giữa các vai trò để AI (Claude Code / Codex) có thể đọc – hiểu – sinh sản phẩm bám sát thiết kế nhất.
---

# Quy Chuẩn Làm Việc PO → BA → Design → Dev → Test (AI-First)

## 0. Triết lý nền tảng

1. **Single source of truth**: Mọi tài liệu sống trong Git (GitLab), dạng `.md`. Confluence/Notion chỉ là nơi mirror, không phải nơi gốc.
2. **AI-readable by design**: Mỗi file có frontmatter chuẩn, tên file chuẩn, vị trí chuẩn → AI tìm được không cần hỏi.
3. **Traceability bắt buộc**: Mỗi tài liệu phải link đến Jira, Figma, Git branch tương ứng. Không có liên kết = không hợp lệ.
4. **One feature = one folder**: Toàn bộ vòng đời của một tính năng nằm trong một thư mục duy nhất.
5. **Người là gatekeeper, AI là drafter**: AI sinh bản nháp, con người review – không bao giờ ngược lại.

---

## 1. Cấu trúc repo tài liệu chuẩn

Đặt trong cùng repo code (mono-repo) hoặc repo `docs/` riêng:

```
docs/
├── README.md                       # Index toàn bộ feature đang sống
├── _templates/                     # Template cho mọi loại tài liệu
│   ├── 00-prd.template.md
│   ├── 01-srs.template.md
│   ├── 02-user-stories.template.md
│   ├── 03-design-spec.template.md
│   ├── 04-tech-design.template.md
│   ├── 05-api-contract.template.md
│   ├── 06-test-plan.template.md
│   └── README.template.md
├── _ai-prompts/                    # Prompt chuẩn cho từng giai đoạn
│   ├── po-to-prd.md
│   ├── prd-to-srs.md
│   ├── srs-to-design-spec.md
│   ├── srs-to-frontend.md
│   ├── srs-to-backend.md
│   └── srs-to-tests.md
├── _conventions/
│   ├── naming.md                   # Quy tắc đặt tên file/branch/Jira
│   ├── frontmatter.md              # Schema frontmatter
│   └── definition-of-done.md       # DoD cho từng vai trò
└── features/
    └── cart-upsell-campaign/       # Một feature = một folder
        ├── README.md               # Index của feature
        ├── 00-prd.md               # PO
        ├── 01-srs.md               # BA
        ├── 02-user-stories.md      # BA
        ├── 03-design-spec.md       # Designer
        ├── 04-tech-design.md       # Tech Lead
        ├── 05-api-contract.md      # BE Dev
        ├── 06-test-plan.md         # QA
        └── assets/                 # Hình minh hoạ, sơ đồ, JSON mẫu
```

**Lý do**: AI được trỏ vào `docs/features/<slug>/` là lập tức có toàn bộ context của feature, không cần lục tìm.

---

## 2. Frontmatter chuẩn (BẮT BUỘC cho mọi file `.md`)

```yaml
---
feature_slug: cart-upsell-campaign      # kebab-case, trùng với tên folder
doc_type: srs                            # prd | srs | user-stories | design-spec | tech-design | api-contract | test-plan
version: 1.2                             # SemVer cho tài liệu
status: approved                         # draft | in-review | approved | deprecated
owner: "@username.ba"                    # 1 người duy nhất chịu trách nhiệm
reviewers: ["@po", "@tech-lead"]
created_at: 2026-04-20
last_updated: 2026-05-03
links:
  jira_epic: PROJ-100
  jira_stories: [PROJ-101, PROJ-102]
  figma: https://figma.com/file/xxx/cart-upsell?node-id=12-345
  git_branch: feature/cart-upsell
  related_docs:
    - ./00-prd.md
    - ./03-design-spec.md
depends_on:                              # Tài liệu này phải đọc gì trước
  - ./00-prd.md
consumed_by:                             # Ai sẽ đọc tài liệu này
  - ./03-design-spec.md
  - ./04-tech-design.md
  - ./06-test-plan.md
---
```

**Quy tắc**:
- File không có frontmatter → MR bị reject ở CI (dùng linter `remark-frontmatter` + custom rule).
- `version` bump khi nội dung thay đổi đáng kể; `last_updated` bump mọi lần edit.
- `status: approved` chỉ được set bởi `owner` sau khi tất cả `reviewers` approve.

---

## 3. Vai trò – Input – Output – Cách dùng AI

### 3.1 PO (Product Owner)

| Mục | Nội dung |
|---|---|
| **Input** | Yêu cầu khách hàng, dữ liệu thị trường, OKR công ty, feedback support |
| **Output bắt buộc** | `00-prd.md`, Jira **Epic** với link đến `00-prd.md` |
| **Output không bắt buộc** | Wireframe phác thảo (chỉ để truyền ý tưởng, không phải design) |
| **Tools** | Jira, Claude (chat) để brainstorm, GitLab |
| **DoD** | PRD được BA và Tech Lead xác nhận "đủ thông tin để bắt đầu SRS" |

**Template `00-prd.md`:**

```markdown
---
feature_slug: <slug>
doc_type: prd
version: 1.0
status: draft
owner: "@po.name"
...
---

# PRD: <Feature name>

## 1. Bối cảnh & vấn đề
- Vấn đề kinh doanh đang giải quyết là gì?
- Bằng chứng (số liệu, ticket support, user research)

## 2. Mục tiêu
- Mục tiêu kinh doanh (đo bằng KPI gì?)
- Mục tiêu người dùng

## 3. User personas & use cases
- Persona chính
- Top 3 use case

## 4. Scope
### In scope
### Out of scope (rất quan trọng - AI và Dev sẽ bám vào để không tự suy diễn)

## 5. Success metrics
- KPI 1: ...
- KPI 2: ...

## 6. Constraints & assumptions
- Ràng buộc kỹ thuật (Shopify policy, GDPR, ...)
- Ràng buộc thời gian
- Giả định cần BA verify

## 7. Open questions
- [ ] Câu hỏi 1 (chờ ai trả lời)
```

**Cách dùng AI cho PO:**
- Mở Claude/ChatGPT, paste raw notes từ họp khách hàng.
- Dùng prompt chuẩn ở `_ai-prompts/po-to-prd.md` để chuyển raw → PRD draft.
- PO **luôn review lại bằng tay**, đặc biệt phần "Out of scope" và KPI.

**Người sau đọc gì**: BA đọc `00-prd.md` đầu tiên, sau đó họp clarification với PO trước khi viết SRS.

---

### 3.2 BA (Business Analyst)

| Mục | Nội dung |
|---|---|
| **Input** | `00-prd.md`, biên bản họp với PO, hệ thống hiện tại (legacy doc nếu có) |
| **Output bắt buộc** | `01-srs.md` (theo format file SRS Cart Upsell mẫu), `02-user-stories.md`, Jira **Stories** với Acceptance Criteria |
| **Tools** | Claude (để sinh draft SRS từ PRD), GitLab, Jira |
| **DoD** | Tech Lead, Designer, QA cùng ký "đọc hiểu được, không cần hỏi thêm" |

**Cấu trúc `01-srs.md`** dùng nguyên format file mẫu (Cart Upsell) – đây là chuẩn vàng:
1. Purpose
2. Scope (in/out)
3. Key business rules (validation, defaults, state transitions)
4. Domain model + Enums
5. Data model & validation rules (bảng có cột Field/Type/Required/Default/Rule)
6. State transitions
7. Runtime/behavior contract
8. DB schema gợi ý (PostgreSQL DDL nếu hệ thống dùng Postgres)
9. QA scenarios (đánh số 1, 2, 3...)
10. Implementation notes for AI
11. Open assumptions (cần Product/FE confirm)

**Template `02-user-stories.md`:**

```markdown
## Story: PROJ-101 – Tạo cross-sell campaign với manual selection

**As a** merchant
**I want to** tạo campaign cross-sell chọn sản phẩm thủ công
**So that** tôi có thể kiểm soát chính xác sản phẩm được upsell

### Acceptance Criteria (Gherkin)
- **Given** tôi đang ở màn tạo campaign
  **When** tôi chọn `offer_type = cross_sell`, `offer_source = manual_selection`, chọn 3 sản phẩm
  **And** nhấn Save với status = running
  **Then** campaign được lưu, status = running, hiển thị trong list

- **Given** tôi không chọn sản phẩm nào
  **When** tôi nhấn Save
  **Then** hệ thống báo lỗi "Phải chọn ít nhất 1 sản phẩm"

### Liên kết
- SRS: [`01-srs.md` mục 3.5, 5.3](./01-srs.md#35-offer-source-rules)
- Design: [`03-design-spec.md` screen S-02](./03-design-spec.md)
```

**Cách dùng AI cho BA:**

```
# Prompt sinh SRS từ PRD (lưu trong _ai-prompts/prd-to-srs.md)

Bạn là Business Analyst senior cho Shopify app.
Đọc file PRD đính kèm: <00-prd.md>
Tham khảo SRS mẫu: docs/features/cart-upsell-campaign/01-srs.md

Sinh ra `01-srs.md` cho feature mới với:
1. Cấu trúc 11 mục giống SRS mẫu
2. Mọi field trong data model phải có: Type, Required, Default, Rule
3. Mọi enum phải liệt kê đầy đủ giá trị
4. Mọi business rule phải kiểm chứng được (testable)
5. Section "Open assumptions" liệt kê những điểm PRD chưa rõ
6. DB schema dạng PostgreSQL DDL với CHECK constraint đầy đủ
7. Tối thiểu 15 QA scenarios

Cấm tự suy diễn business logic. Phần nào PRD không nói rõ → đưa vào "Open assumptions".
```

**Người sau đọc gì**:
- Designer đọc `01-srs.md` mục 3 (business rules) + mục 4 (domain model) + `02-user-stories.md`
- Dev đọc toàn bộ `01-srs.md`
- QA đọc `01-srs.md` mục 9 (QA scenarios) + `02-user-stories.md` (AC)

---

### 3.3 Designer

| Mục | Nội dung |
|---|---|
| **Input** | `01-srs.md`, `02-user-stories.md`, design system hiện có |
| **Output bắt buộc** | File Figma (đặt tên frame chuẩn), `03-design-spec.md` |
| **Tools** | Figma (+ Figma MCP để AI đọc được), GitLab |
| **DoD** | Dev có thể nhìn `03-design-spec.md` + Figma là biết phải code gì, không cần hỏi |

**Quy tắc đặt tên Figma (BẮT BUỘC để AI qua MCP đọc được):**

```
[FeatureSlug] / [Screen-ID] - [Screen Name] / [State]

Ví dụ:
cart-upsell-campaign / S-01 - Campaign List / Default
cart-upsell-campaign / S-01 - Campaign List / Empty
cart-upsell-campaign / S-02 - Create Campaign / Step 1 - Basic Info
cart-upsell-campaign / S-02 - Create Campaign / Step 1 - Validation Error
```

**Component trong Figma cũng cần đặt tên chuẩn:**

```
Component / Button / Primary / Default
Component / Input / Text / Error
Component / Card / Product / Selected
```

**Template `03-design-spec.md`:**

```markdown
---
doc_type: design-spec
...
---

# Design Spec: <Feature>

## 1. Screen inventory
| ID | Tên | Figma frame | User story liên quan |
|---|---|---|---|
| S-01 | Campaign List | [link] | PROJ-101 |
| S-02 | Create Campaign | [link] | PROJ-102 |

## 2. Design tokens được dùng
- Color: dùng từ `design-system/tokens.json`
- Typography: ...
- Spacing: ...
(Không tạo token mới mà không bàn với DS lead)

## 3. Component spec
### 3.1 ProductSelectorCard
- **Figma**: [link đến component]
- **States**: default, hover, selected, disabled, loading
- **Props (theo SRS)**:
  - `productId: string`
  - `title: string`
  - `price: Money`
  - `discount?: { type, value }`
  - `selected: boolean`
- **Behavior**:
  - Click → toggle selected
  - Khi `disabled = true` (sản phẩm đã trong giỏ và `hide_products_already_in_cart = true`) → không render
- **Edge cases**:
  - Title quá dài: ellipsis 2 dòng
  - Không có ảnh: hiện placeholder

## 4. Interaction & flow
- Sơ đồ flow (link đến Figma flow hoặc Mermaid inline)

## 5. Empty / Error / Loading states
| Screen | Empty | Error | Loading |
|---|---|---|---|

## 6. Responsive
- Mobile (< 768px): ...
- Tablet: ...
- Desktop: ...

## 7. Accessibility
- Keyboard nav order
- ARIA labels
- Color contrast đã đạt WCAG AA

## 8. Animation & motion
- Mô tả những thứ Figma không truyền tải được (timing, easing, sequence)

## 9. Dynamic content rules
- Khi data từ API thay đổi runtime, behavior là gì?
- (Phần này QUAN TRỌNG vì Figma không thể hiện được)
```

**Cách dùng AI cho Designer:**
- Designer **không dùng AI để generate UI** (đó là việc của con người), nhưng dùng AI để:
  - Sinh `03-design-spec.md` từ Figma + SRS: prompt Claude đọc SRS + screenshot Figma → ra spec draft.
  - Check accessibility tự động.
  - Đảm bảo không thiếu state nào (so chéo với QA scenarios ở SRS).

**Người sau đọc gì**:
- **FE Dev** đọc `03-design-spec.md` để biết component breakdown, sau đó dùng Figma MCP truy cập Figma file để extract style/layout.
- **QA** đọc `03-design-spec.md` mục 5 (states) và mục 7 (a11y) để viết test case UI.

---

### 3.4 Dev (Tech Lead + FE + BE)

#### 3.4.1 Tech Lead viết `04-tech-design.md` TRƯỚC khi Dev code

| Mục | Nội dung |
|---|---|
| **Input** | `01-srs.md`, `03-design-spec.md`, kiến trúc hiện tại |
| **Output bắt buộc** | `04-tech-design.md`, `05-api-contract.md` (OpenAPI), DB migration plan |
| **Tools** | Claude Code (đọc cả repo để đề xuất kiến trúc nhất quán), GitLab |

**Template `04-tech-design.md`:**

```markdown
## 1. Architecture overview
(Sơ đồ Mermaid: client → API → service → DB → external)

## 2. Module breakdown
- Module A: trách nhiệm gì, file ở đâu
- Module B: ...

## 3. File/folder structure (cho cả FE và BE)
src/
  features/
    cart-upsell/
      api/
      components/
      hooks/
      types/
      utils/

## 4. Data flow
- Khi merchant submit form → ...
- Khi storefront resolve campaign → ...

## 5. Dependencies
- Library mới cần thêm: tên, lý do, license
- Internal package: ...

## 6. Error handling strategy
- Validation errors: format JSON nào
- Runtime errors: log ở đâu, alert ai

## 7. Performance budget
- API p95 < 300ms (theo SRS mục 10)
- Bundle size FE tăng tối đa: 50KB gzip

## 8. Security
- Authn/Authz check cho mỗi endpoint
- Rate limit
- PII handling

## 9. Migration & rollout plan
- DB migration script
- Feature flag tên gì
- Rollout: % traffic
```

#### 3.4.2 FE Dev

**Input**: `01-srs.md`, `03-design-spec.md`, `04-tech-design.md`, `05-api-contract.md`, Figma file qua MCP.

**Output**: Code trong branch `feature/<slug>`, MR có link Jira story, unit test ≥ 80% coverage cho logic phức tạp.

**Cách dùng Claude Code / Codex tối ưu:**

```bash
# Trong Claude Code, từ root repo:
claude "Đọc các file sau theo thứ tự:
1. docs/features/cart-upsell-campaign/01-srs.md
2. docs/features/cart-upsell-campaign/03-design-spec.md
3. docs/features/cart-upsell-campaign/04-tech-design.md
4. docs/features/cart-upsell-campaign/05-api-contract.md

Sau đó dùng Figma MCP đọc frame: 
cart-upsell-campaign / S-02 - Create Campaign / Step 1 - Basic Info

Nhiệm vụ: triển khai component <CampaignBasicInfoForm /> theo:
- Tech stack: React + TypeScript + Tailwind + react-hook-form + zod
- File structure tuân theo 04-tech-design.md mục 3
- Validation schema bám sát 01-srs.md mục 5
- Component breakdown bám sát 03-design-spec.md mục 3
- Viết test song song bằng Vitest + Testing Library
- Mỗi commit theo Conventional Commits

Output:
- Tạo các file mới trong src/features/cart-upsell/
- Không sửa file ngoài thư mục feature
- Cuối cùng tạo MR description theo template MR_TEMPLATE.md"
```

#### 3.4.3 BE Dev

**Input**: `01-srs.md` (đặc biệt mục 5 data model, mục 8 DB schema), `04-tech-design.md`, `05-api-contract.md`.

**Output**: Code BE, migration script, OpenAPI spec đồng bộ với code (dùng decorator hoặc generate từ code).

**Prompt mẫu cho Claude Code (BE):**

```bash
claude "Implement Cart Upsell Campaign API.

Đọc:
- docs/features/cart-upsell-campaign/01-srs.md (toàn bộ)
- docs/features/cart-upsell-campaign/04-tech-design.md
- docs/features/cart-upsell-campaign/05-api-contract.md
- src/shared/ (để học pattern hiện tại)

Yêu cầu:
1. Tạo migration tương ứng SRS mục 8 (giữ nguyên CHECK constraints)
2. Tạo entity, DTO, validation theo SRS mục 5 — KHÔNG ĐỔI tên field
3. Tạo service: CRUD + state transitions theo SRS mục 6
4. Tạo resolver runtime theo SRS mục 7 (chú ý filtering order ở 7.3)
5. Mỗi method có unit test cover 100% nhánh validation
6. Tuân thủ SRS mục 7.4, 7.5: empty offer / no recommendation = không lỗi 500
7. Performance: p95 < 300ms với cache (theo mục 10)

Không tự ý:
- Thêm field không có trong SRS
- Đổi enum value
- Bỏ qua CHECK constraint
"
```

**Người sau đọc gì**:
- **QA**: đọc `04-tech-design.md` để hiểu edge case kỹ thuật, đọc `05-api-contract.md` để viết API test.
- **Maintainer tương lai**: đọc `README.md` của feature → `04-tech-design.md`.

---

### 3.5 QA / Test

| Mục | Nội dung |
|---|---|
| **Input** | `01-srs.md` (mục 9), `02-user-stories.md` (AC), `03-design-spec.md` (states), `04-tech-design.md`, `05-api-contract.md` |
| **Output bắt buộc** | `06-test-plan.md`, test cases trong Jira/Xray, automation script trong GitLab |
| **Tools** | Claude (sinh test case), Playwright/Cypress, k6 (perf), Postman/Newman |
| **DoD** | Tất cả AC pass, regression pass, performance budget đạt, không còn bug Critical/High |

**Template `06-test-plan.md`:**

```markdown
## 1. Scope
- In scope
- Out of scope

## 2. Test approach
| Layer | Tool | Trách nhiệm |
|---|---|---|
| Unit | Vitest/Jest | Dev |
| Integration | Supertest | Dev |
| API | Postman/Newman | QA |
| E2E | Playwright | QA |
| Performance | k6 | QA |
| Manual UAT | - | QA + PO |

## 3. Test environment
- Staging URL
- Test accounts
- Seed data

## 4. Test cases (functional)
| ID | Tiêu đề | Liên kết SRS | Liên kết AC | Steps | Expected | Priority |
|---|---|---|---|---|---|---|
| TC-001 | Tạo cross-sell với manual | SRS 3.5, 3.6 | PROJ-101 | ... | ... | High |

## 5. Edge cases & negative tests
(map từ SRS mục 5 - mọi validation rule = 1 negative test)

## 6. Non-functional
- Performance test scenario
- Security: SQLi, XSS, authz bypass
- Accessibility: axe-core scan

## 7. Regression suite
- List feature liên quan cần regression

## 8. Exit criteria
- 100% test case High priority pass
- 0 bug Critical, ≤ 2 bug Medium đã có workaround
```

**Cách dùng AI cho QA:**

```
Prompt (lưu ở _ai-prompts/srs-to-tests.md):

Đọc:
- docs/features/<slug>/01-srs.md
- docs/features/<slug>/02-user-stories.md
- docs/features/<slug>/03-design-spec.md

Sinh test cases dạng bảng Markdown với cột:
TC-ID | Title | Type (positive/negative/edge) | SRS link | AC link | Preconditions | Steps | Expected | Priority

Yêu cầu:
1. Mỗi business rule trong SRS mục 3 → tối thiểu 1 positive + 1 negative
2. Mỗi validation rule trong SRS mục 5 → 1 negative test (boundary value)
3. Mỗi state transition trong SRS mục 6 → 1 test
4. Mỗi screen state trong design-spec mục 5 → 1 UI test
5. Resolver behavior (SRS mục 7) → 1 test/scenario, đặc biệt mục 7.3 filtering order
6. Mọi QA scenario trong SRS mục 9 → 1 test case riêng
7. Output dùng được để import vào Jira Xray (CSV-friendly)
```

---

## 4. README.md của mỗi feature (BẮT BUỘC)

```markdown
---
feature_slug: cart-upsell-campaign
doc_type: feature-index
status: in-development
---

# Feature: Cart Page Upsell Campaign

## Trạng thái tài liệu
| # | Doc | Owner | Status | Last updated | Link |
|---|---|---|---|---|---|
| 1 | PRD | @po | ✅ Approved | 2026-04-20 | [00-prd.md](./00-prd.md) |
| 2 | SRS | @ba | ✅ Approved | 2026-04-25 | [01-srs.md](./01-srs.md) |
| 3 | User Stories | @ba | ✅ Approved | 2026-04-26 | [02-user-stories.md](./02-user-stories.md) |
| 4 | Design Spec | @designer | 🔄 In Review | 2026-04-30 | [03-design-spec.md](./03-design-spec.md) |
| 5 | Tech Design | @tl | 📝 Draft | 2026-05-02 | [04-tech-design.md](./04-tech-design.md) |
| 6 | API Contract | @be | 📝 Draft | 2026-05-02 | [05-api-contract.md](./05-api-contract.md) |
| 7 | Test Plan | @qa | ⏳ Pending | - | [06-test-plan.md](./06-test-plan.md) |

## Liên kết ngoài
- **Jira Epic**: [PROJ-100](https://...)
- **Figma**: [Cart Upsell File](https://...)
- **Git branch**: `feature/cart-upsell`
- **Staging**: https://staging.../admin/apps/...

## Đọc theo thứ tự (cho người mới vào project)
1. PRD → hiểu vì sao
2. SRS → hiểu cái gì
3. Design Spec + Figma → hiểu trông như nào
4. Tech Design → hiểu code ở đâu
5. Test Plan → hiểu kiểm thử thế nào

## Open questions
- [ ] @po: confirm KPI baseline
- [ ] @designer: empty state cho mobile

## Changelog
- 2026-05-02: Tech Lead hoàn thành tech design draft
- 2026-04-30: Designer xong Figma main flow
- 2026-04-25: BA approve SRS v1.0
```

---

## 5. Naming convention tổng hợp

| Loại | Quy tắc | Ví dụ |
|---|---|---|
| Feature slug | kebab-case, danh từ | `cart-upsell-campaign` |
| Folder feature | Trùng feature slug | `docs/features/cart-upsell-campaign/` |
| File doc | `<order>-<type>.md` | `01-srs.md` |
| Git branch | `feature/<feature-slug>` hoặc `feature/<feature-slug>-<sub>` | `feature/cart-upsell-campaign` |
| MR title | `[<JIRA-ID>] <type>: <summary>` | `[PROJ-101] feat: campaign create form` |
| Commit | Conventional Commits | `feat(cart-upsell): add validation` |
| Figma frame | `<feature-slug> / <Screen-ID> - <Name> / <State>` | `cart-upsell-campaign / S-02 - Create / Default` |
| Jira Epic | Trùng feature_slug ở field custom | `PROJ-100` |

---

## 6. Quy tắc bàn giao (handoff gates)

Một giai đoạn chỉ được bắt đầu khi giai đoạn trước đạt **Definition of Done**:

```
PO done            → status PRD = approved + Epic Jira tạo
   ↓
BA done            → status SRS + User Stories = approved + Stories Jira tạo + Tech Lead ký
   ↓
Designer done      → status Design Spec = approved + Figma frame chuẩn + Dev ký "đọc hiểu"
   ↓
Tech Lead done     → status Tech Design + API Contract = approved
   ↓
Dev done           → MR merge + unit test pass + tài liệu cập nhật nếu lệch SRS
   ↓
QA done            → Test Plan execute xong + exit criteria đạt
```

**Quy tắc lệch chuẩn**: Khi Dev phát hiện SRS sai/thiếu, **không được tự sửa code lệch SRS**. Phải:
1. Mở Jira sub-task "SRS update needed".
2. BA cập nhật SRS, bump version.
3. Dev mới code theo bản mới.

---

## 7. Cách AI (Claude Code / Codex) tận dụng cấu trúc này

### 7.1 Lệnh "khởi động" tiêu chuẩn cho mọi phiên AI

Lưu trong `.claude/CLAUDE.md` (hoặc `AGENTS.md` cho Codex):

```markdown
# Working agreement với AI assistant

Khi user yêu cầu làm việc với một feature, LUÔN làm theo thứ tự:

1. Đọc `docs/features/<slug>/README.md` để biết feature ở giai đoạn nào.
2. Đọc các file theo `depends_on` trong frontmatter.
3. Nếu là task code:
   - Đọc `01-srs.md`, `03-design-spec.md`, `04-tech-design.md`, `05-api-contract.md`
   - Truy cập Figma qua MCP nếu là FE task
4. Nếu là task viết tài liệu: đọc template tương ứng trong `docs/_templates/`
5. Tuân thủ `docs/_conventions/`

KHÔNG:
- Tự suy diễn business logic không có trong SRS
- Đổi tên field, enum value
- Bỏ qua validation rule
- Tạo file ngoài thư mục feature đang làm việc

KHI gặp mâu thuẫn giữa các tài liệu: dừng lại, báo cho user, KHÔNG tự chọn.
```

### 7.2 Bộ prompt mẫu cần có sẵn trong `_ai-prompts/`

| File | Mục đích | Người dùng |
|---|---|---|
| `po-to-prd.md` | Raw notes → PRD draft | PO |
| `prd-to-srs.md` | PRD → SRS draft theo format chuẩn | BA |
| `srs-to-design-spec.md` | SRS + screenshot Figma → Design spec | Designer |
| `srs-to-frontend.md` | SRS + Design + Figma MCP → React component | FE Dev |
| `srs-to-backend.md` | SRS → Entity/Service/Migration | BE Dev |
| `srs-to-tests.md` | SRS + AC → Test case bảng | QA |
| `review-doc.md` | Review một tài liệu xem có thiếu/mâu thuẫn không | Reviewer |

### 7.3 Mẹo tối ưu kết quả AI

1. **Luôn cho AI xem SRS mẫu (Cart Upsell) làm reference** khi sinh SRS mới — kết quả nhất quán hơn ~3 lần.
2. **Cho AI đọc nguyên thư mục feature** thay vì từng file rời — Claude Code làm tốt việc này.
3. **Yêu cầu AI list ra "Open assumptions"** ở mọi output — không cho AI tự suy diễn ngầm.
4. **Yêu cầu AI viết test song song với code** trong cùng một MR.
5. **Dùng Figma MCP** ở giai đoạn FE — đừng paste screenshot vì AI sẽ bỏ sót spacing/typography.
6. **Bật CI lint cho frontmatter** — nếu một file `.md` thiếu frontmatter chuẩn, build fail.

---

## 8. Bảng tóm tắt "Ai đọc gì khi làm gì"

| Khi bạn là... | Bắt đầu task → đọc | Trong khi làm → tham khảo | Trước khi handoff → cập nhật |
|---|---|---|---|
| **PO** | Customer feedback, Roadmap | PRD template | `00-prd.md` |
| **BA** | `00-prd.md` + họp PO | SRS mẫu, `_templates/01-srs.template.md` | `01-srs.md`, `02-user-stories.md` |
| **Designer** | `01-srs.md`, `02-user-stories.md` | Design system, `03-design-spec.template.md` | Figma + `03-design-spec.md` |
| **Tech Lead** | `01-srs.md`, `03-design-spec.md` | Kiến trúc hiện tại | `04-tech-design.md`, `05-api-contract.md` |
| **FE Dev** | `01-srs.md`, `03-design-spec.md`, `04-tech-design.md` | Figma qua MCP, `05-api-contract.md` | Code + cập nhật `04-tech-design.md` nếu lệch |
| **BE Dev** | `01-srs.md`, `04-tech-design.md`, `05-api-contract.md` | Code base hiện tại | Code + migration + cập nhật `05-api-contract.md` |
| **QA** | `01-srs.md` (mục 9), `02-user-stories.md`, `03-design-spec.md` | `04-tech-design.md`, `05-api-contract.md` | `06-test-plan.md` + test cases Jira |

---



## 9. Nguyên tắc cuối cùng

> **"Nếu AI đọc xong tài liệu mà vẫn phải hỏi lại — tài liệu sai, không phải AI sai."**

Tất cả nỗ lực chuẩn hoá ở đây là để khi một task được giao cho Claude Code hoặc Codex, AI có **đủ context, không mâu thuẫn, có thể trace ngược về mọi quyết định**. Khi đạt được điều đó, sản phẩm cuối sẽ bám sát thiết kế, vai trò rõ ràng, và bàn giao mượt — bất kể người làm là human hay AI.
