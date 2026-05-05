---
doc_type: team-workflow
version: 2.7
status: approved
last_updated: 2026-05-05
audience: [PO, BA, Designer, Tech Lead, FE Dev, BE Dev, QA, Release Manager, Doc Maintainer]
purpose: Verbose handbook complementing the terse `WORKFLOW-STANDARD.md`. Tells humans + AI agents (Claude Code / Codex) how to collaborate so the product matches docs.
---

# Team Workflow (v2.7) — How we work, how AI helps

> Tài liệu này là handbook chi tiết bổ sung cho `WORKFLOW-STANDARD.md` (terse spec). Đối tượng đọc: người + AI agent. Mục tiêu duy nhất: đảm bảo **sản phẩm thật** (code + UI + DB) **khớp với docs** ở mọi thời điểm.
>
> Nếu mâu thuẫn xảy ra giữa file này và `WORKFLOW-STANDARD.md`, **`WORKFLOW-STANDARD.md` thắng**. File này chỉ giải thích "tại sao" và "làm thế nào".

---

## 0. Foundational principles — 5 nguyên tắc nền

Đây là 5 nguyên tắc **không thương lượng**. Tất cả skill, gate, naming convention bên dưới đều phái sinh từ đây.

### 0.1 Single source of truth (SSOT)

Mỗi loại thông tin chỉ tồn tại ở **đúng một nơi**:

| Thông tin | SSOT | Ai sở hữu |
|---|---|---|
| Goals, personas, scope của 1 phase | `docs/po/<phase>/00-prd.md` | PO |
| Business rules + field semantics | `docs/features/<slug>/01-srs.md` §3 + §5 | BA |
| User stories + Gherkin AC | `docs/features/<slug>/01-srs.md` §11 | BA |
| Component states + responsive | `docs/features/<slug>/02-design-spec.md` | Designer |
| Prisma schema (model, index, FK, ON DELETE) | `docs/features/<slug>/03-tech-design.md` §3 | Tech Lead |
| Endpoint summary (method, path, auth, body, status) | `docs/features/<slug>/03-tech-design.md` §11 | Tech Lead |
| Test cases + regression suite | `docs/features/<slug>/04-test-plan.md` | QA |
| Feature flag + rollout + rollback | `docs/features/<slug>/05-release-plan.md` | Release Manager |
| Architectural decisions cross-feature | `docs/_adr/<NNNN>-<slug>.md` | Tech Lead |
| Pre-PRD research / POC | `docs/_spikes/<NNNN>-<slug>.md` | bất kỳ ai |

Vi phạm điển hình: PO viết acceptance criteria trong PRD (đã có §11 SRS), BA viết Prisma schema trong SRS (đã có §3 Tech Design), Tech Lead viết business rule trong Tech Design (đã có §3 SRS).

### 0.2 AI-readable by design

Docs viết để **AI agent đọc được mà không cần hỏi lại**. Ràng buộc:

- Frontmatter YAML đầy đủ (`feature_slug`, `phase_slug`, `depends_on`, `consumed_by`, `status`).
- Section count cố định: PRD = 8, SRS = 11, Design Spec = 3, Tech Design = 12, Test Plan = 10, Release Plan = 7.
- Field name dùng `snake_case` ở DDL/JSON, `camelCase` ở TS code (qua serializer). **Không** lẫn lộn.
- Mọi cross-reference phải dùng đường dẫn tương đối (`./01-srs.md#5-data-model`).
- Mọi quyết định không chắc chắn → ghi vào **Open Questions** section. **Không** AI auto-fill.

Câu hỏi kiểm tra: "Nếu tôi đưa folder `docs/features/<slug>/` cho 1 AI agent fresh, nó có code đúng không cần hỏi gì không?". Nếu không → docs sai, không phải AI sai.

### 0.3 Traceability (truy vết hai chiều)

Mỗi dòng code phải truy được về 1 dòng docs, và ngược lại:

```
PRD goal → SRS rule → SRS field → Tech Design schema → Code field
                   → SRS story §11 → Test Plan TC-NNN → Release Plan metric
                   → Design Spec component state → FE component prop
```

Quy tắc skill `feature-*` đều enforce điều này (xem AGENTS.md §4.7).

### 0.4 One feature = one folder

Một feature slug → một folder `docs/features/<slug>/` chứa **đúng 6 file**: `README.md` + 5 doc đánh số 01..05.

Folder không được chứa file khác. Cross-feature artifact đi vào `docs/_adr/` hoặc `docs/_spikes/`.

Phase scoping (multi-release): xem §5.6 phía dưới.

### 0.5 Human gatekeeper / AI drafter

AI **draft**, người **gate**:

- AI agent (Claude Code / Codex) viết draft thông qua skill `feature-*`.
- Người (owner trong frontmatter) review + chuyển status `draft → in-review → approved`.
- AI **không** được tự đặt `status: approved`. **Không** được tự bump major version. **Không** được tự fill Open Questions.

---

## 1. Repo doc structure

```
docs/
├── _conventions/
│   ├── WORKFLOW-STANDARD.md        ← terse spec (single source for the rules)
│   └── TEAM-WORKFLOW.md            ← this file (verbose handbook)
├── _adr/                           ← Architecture Decision Records (Michael Nygard format)
│   ├── README.md
│   └── <NNNN>-<slug>.md            ← e.g. 0001-choose-mongodb.md
├── _spikes/                        ← pre-PRD research / POC notes
│   ├── README.md
│   └── <NNNN>-<slug>.md
├── po/
│   └── <phase-slug>/               ← 1 phase = 1 PRD = nhiều feature
│       └── 00-prd.md               ← phase-level PRD (PO viết, 8 sections)
├── features/
│   └── <feature-slug>/             ← đúng 6 file, không hơn không kém
│       ├── README.md               ← lightweight index (~30 lines, status table)
│       ├── 01-srs.md               ← SRS 11 sections (BA, includes user stories §11)
│       ├── 02-design-spec.md       ← Design Spec 3 sections (Designer)
│       ├── 03-tech-design.md       ← Tech Design 12 sections (Tech Lead, Prisma §3, endpoints §11)
│       ├── 04-test-plan.md         ← Test Plan 10 sections (QA)
│       └── 05-release-plan.md      ← Release Plan 7 sections (Release Manager)
├── system-architecture.md          ← project-level (Doc Maintainer sync sau release)
├── codebase-summary.md             ← project-level
├── project-roadmap.md              ← project-level
└── code-standards.md               ← project-level
```

### Những gì KHÔNG còn (so với v1.0)

- ❌ `_templates/` — bị bỏ. Skills (`.claude/skills/feature-*/SKILL.md`) tự sinh template inline.
- ❌ `_ai-prompts/` — bị bỏ. Thay bằng 9 skill chuyên biệt.
- ❌ `02-user-stories.md` riêng — đã merge vào SRS §11.
- ❌ `05-api-contract.md` riêng — đã thu gọn thành Tech Design §11 (1 row = 1 endpoint).
- ❌ `00-prd.md` trong feature folder — PRD chuyển sang `docs/po/<phase>/`, 1 PRD bao nhiều feature.

### Những gì MỚI (v2.7)

- ✨ `_adr/` — quyết định kiến trúc cross-feature.
- ✨ `_spikes/` — research / POC trước PRD.
- ✨ `05-release-plan.md` — feature flag, rollout, rollback, monitoring.
- ✨ `README.md` per feature — re-add (đã có ở v1, bị bỏ ở v2.0, giờ cần lại vì folder đã có 5 file).
- ✨ Cross-feature deps ở PRD frontmatter (`depends_on_features`, `blocks_features`).

---

## 2. Frontmatter standard

YAML frontmatter là **bắt buộc** cho mọi doc. Spec đầy đủ ở `WORKFLOW-STANDARD.md` §4.2 — đây tóm tắt key rules:

```yaml
doc_type: <prd|srs|design-spec|tech-design|test-plan|release-plan>
version: <X.Y[.Z]>                  # semver, bump theo §7 WORKFLOW-STANDARD
status: <draft|in-review|approved|released|deprecated>
owner: "@<github-username>"
last_updated: <YYYY-MM-DD>          # update mỗi lần content thay đổi
feature_slug: <kebab-slug>          # ngoại trừ PRD (PRD dùng phase_slug + features[])
phase_slug: <phase-N-name>
depends_on: [./01-srs.md, ...]      # đường dẫn tương đối tới upstream
consumed_by: [./03-tech-design.md, ...]
links:
  jira_epic: <PROJ-XXX>             # OPTIONAL — bỏ qua nếu team không dùng Jira
  jira_stories: [<PROJ-XXX>, ...]   # OPTIONAL
  figma: <url>                      # khuyến nghị cho design-spec
  git_branch: feature/<slug>        # MANDATORY
```

**Quy tắc thép:**

- **Jira OPTIONAL** (v2.6+). Nếu không dùng Jira: dùng story ID stable `US-01`, `STORY-01`. KHÔNG dùng `PROJ-XXX` kiểu fake.
- **Git branch MANDATORY**: `feature/<slug>` cho code, `docs/<slug>` cho doc-only.
- **Doc files MANDATORY** không liên quan tới việc dùng Jira hay không.
- AI agent **KHÔNG** tự set `status: approved`. Status chỉ owner đổi.

---

## 3. Roles — Input / Output / Skill / DoD

9 vai trò, mỗi vai trò có **đúng 1 skill** ánh xạ. Người không skill = không artifact = không gate.

### 3.1 PO (Product Owner) — `feature-prd-author`

| | |
|---|---|
| **Input** | Customer feedback, OKR, support tickets, meeting notes, spike (`docs/_spikes/`) |
| **Output** | `docs/po/<phase-slug>/00-prd.md` (1 PRD bao **nhiều feature** trong cùng phase) |
| **Sections** | 8 — Context · Goals · Personas · Scope · Metrics · Constraints · Open questions · Sub-features (mapping mỗi sub-feature → SRS + Tech Design path) |
| **Tools** | Claude Code, Notion (raw notes only — KHÔNG là SSOT), Git mandatory |
| **DoD** | Frontmatter đủ (`features: [...]`, optional `depends_on_features`, `blocks_features`); 8 section đủ; Open Questions không trống nếu có uncertainty; status `approved` bởi PO + Tech Lead reviewer |
| **Sample prompt** | "Viết PRD cho phase-2-upsell-campaigns gồm 3 feature: upsell-product-page, upsell-cart, upsell-checkout. Notes ở `docs/_spikes/0007-upsell-research.md`." |

### 3.2 BA (Business Analyst) — `feature-srs-author`

| | |
|---|---|
| **Input** | PRD `approved` của phase chứa feature |
| **Output** | `docs/features/<feature-slug>/01-srs.md` |
| **Sections** | 11 — Purpose · Scope · Business rules · Domain model · **Data model & validation rules (§5)** · State transitions · Runtime · QA scenarios · Implementation notes · Open assumptions · **User stories với Gherkin AC (§11)** |
| **Tools** | Claude Code, Git mandatory |
| **DoD** | §5 chỉ chứa **field semantics** (name, conceptual type, required, default, business rule) — **KHÔNG** Prisma/DDL; §11 mọi story có AC dạng Given/When/Then; status `approved` bởi BA + PO + Tech Lead |
| **Critical rule** | BA **KHÔNG** viết Prisma schema. Schema design là việc của Tech Lead ở Tech Design §3. |
| **Sample prompt** | "Viết SRS cho feature upsell-campaign từ `docs/po/phase-2-upsell-campaigns/00-prd.md`." |

### 3.3 Designer — `feature-design-spec-author`

| | |
|---|---|
| **Input** | SRS `draft` (cho phép parallel) hoặc `approved` (cho gate sequential); Figma file (Figma MCP) |
| **Output** | `docs/features/<feature-slug>/02-design-spec.md` |
| **Sections** | 3 (lightweight) — Screen inventory · Component spec (top 2-3 components, props ánh xạ SRS §5) · Responsive behavior |
| **Tools** | Claude Code + **Figma MCP** (`use_figma`), Figma file, Git mandatory |
| **DoD** | Mọi component state + edge case Figma không express được phải viết text; mọi prop trace tới SRS §5 field; status `approved` sau khi SRS `approved` |
| **Parallel rule (v2.7)** | Designer có thể **draft** parallel với BA viết SRS. `approved` vẫn sequential — Design Spec không thể `approved` trước SRS `approved`. |
| **Sample prompt** | "Viết design spec cho upsell-campaign, Figma node 1:9695." |

### 3.4 Tech Lead — `feature-tech-design-author`

| | |
|---|---|
| **Input** | SRS + Design Spec `approved` (draft cho phép parallel) |
| **Output** | `docs/features/<feature-slug>/03-tech-design.md` |
| **Sections** | 12 — Architecture · Module breakdown · **Database schema (Prisma, §3)** · File structure · Data flow · Validation · Error handling · Performance budget · Security · Migration plan · **API Contract lightweight (§11, 1-row-per-endpoint table)** · Open questions |
| **Tools** | Claude Code, Prisma, Git mandatory; reference ADR theo ID (`see ADR-0003`) |
| **DoD** | §3 chứa Prisma model đầy đủ (CHECK constraint cho Postgres / Zod refines cho Mongo, indexes, FK, ON DELETE); §11 endpoint table không paste full OpenAPI; mỗi field §3 trace về SRS §5; status `approved` bởi Tech Lead + 1 senior reviewer |
| **Critical rule** | KHÔNG paste full OpenAPI YAML. KHÔNG bỏ §3 (schema). Endpoint table format: `| Method | Path | Auth | Request body (Zod ref) | Response status | User story |`. |
| **Sample prompt** | "Viết tech design cho upsell-campaign, dùng Prisma + Mongo, theo SRS + Design Spec đã approved." |

### 3.5 QA (QA Engineer / Test Lead) — `feature-test-plan-author`

| | |
|---|---|
| **Input** | SRS + Design Spec + Tech Design `approved` |
| **Output** | `docs/features/<feature-slug>/04-test-plan.md` |
| **Sections** | 10 (lightweight) — Scope · Test approach · Test environment · Test cases (`TC-NNN`) · Edge cases & boundary · State transition tests · Non-functional · Regression suite · Exit criteria · Bug severity guideline |
| **Tools** | Claude Code, Jira Xray/Zephyr (CSV import) nếu dùng Jira, Git mandatory |
| **DoD** | Mỗi test case cite source: SRS §+AC, Design Spec component state, hoặc Tech Design endpoint/constraint; Exit criteria measurable; Test ID stable (`TC-NNN`); status `approved` |
| **Parallel rule** | QA **chạy parallel** với FE/BE Impl. Không chờ code xong mới viết test. |
| **Sample prompt** | "Viết test plan cho upsell-campaign từ SRS + Design Spec + Tech Design đã approved, format Jira Xray import-ready." |

### 3.6 FE Dev — `feature-frontend-impl`

| | |
|---|---|
| **Input** | SRS + Design Spec + Tech Design (đặc biệt §2 module, §4 file structure, §11 endpoints) + Figma (qua MCP) + Test Plan §4 (test cases để code đúng từ đầu) |
| **Output** | Code dưới `app/modules/<feature-slug>/` (admin app) hoặc `extensions/<slug>/` (storefront Preact) |
| **Stack** | **React Router 7** + TypeScript + **Shopify Polaris** + **App Bridge** + **TanStack Query** + **Zod** (admin) / **Preact** + Zod (storefront, đọc shop metafield, không API roundtrip) |
| **Tools** | Claude Code, Figma MCP, Git mandatory (`feature/<slug>`) |
| **DoD** | Mọi component prop trace về Design Spec §2 / SRS §5; Zod schema khớp SRS §5; unit test cạnh component; lint + typecheck pass; PR link tới feature folder README |
| **Sample prompt** | "Implement FE cho upsell-campaign theo Tech Design + Design Spec, Figma node 1:9695." |

### 3.7 BE Dev — `feature-backend-impl`

| | |
|---|---|
| **Input** | SRS + Tech Design (đặc biệt §3 Prisma schema, §2 module, §4 file structure, §7 error handling, §11 endpoints) + Test Plan §4 |
| **Output** | Code dưới `app/services/<feature>*.server.ts` + `app/api/v1/<feature-slug>/` + `prisma/schema/<feature>.prisma` + migration |
| **Stack** | **React Router 7 server functions** + TypeScript + **Prisma** (Mongo or Postgres) + **Zod** (DTO validation) — Template A (RR7 monolith). |
| **Tools** | Claude Code, Prisma CLI, Git mandatory |
| **DoD** | Schema khớp Tech Design §3 (KHÔNG rename field, KHÔNG skip constraint); validation khớp SRS §5; runtime behavior khớp SRS §7; soft-fail (empty result không là 500); unit + integration test; migration reversible |
| **Sample prompt** | "Implement BE cho upsell-campaign theo Tech Design §3 (Prisma) + §11 (endpoints)." |

### 3.8 Release Manager — `feature-release-author`

| | |
|---|---|
| **Input** | SRS + Tech Design + Test Plan (§9 exit criteria phải green) + FE/BE Impl đã merge |
| **Output** | `docs/features/<feature-slug>/05-release-plan.md` |
| **Sections** | 7 — Release summary · **Feature flag strategy** · **Rollout plan (canary % phases)** · Monitoring & alerts · **Rollback runbook** · Post-launch checklist · Open questions |
| **Tools** | Claude Code, LaunchDarkly/Unleash/internal flag system, monitoring (Datadog/Grafana), Git mandatory |
| **DoD** | Feature flag tên + default state rõ; canary % phases có time-based gate (e.g. "5% × 24h → 25% × 48h → 100%"); rollback runbook có command thật + rollback time SLA; mọi metric §4 trace về Tech Design §8 perf budget; status `approved` bởi Release Manager + Tech Lead + on-call |
| **Multi-release** | Naming: `05-release-plan-phase-2-1.md`, `05-release-plan-phase-2-2.md` (xem §5.6) |
| **Sample prompt** | "Viết release plan cho upsell-campaign-phase-2-1, canary 5%/25%/100%, flag tên `upsell_pdp_v1`." |

### 3.9 Doc Maintainer — `feature-docs-sync`

| | |
|---|---|
| **Input** | Feature ở status `released` (flag 100%) |
| **Output** | **KHÔNG** file mới — edit project-level docs: `docs/system-architecture.md`, `docs/codebase-summary.md`, `docs/project-roadmap.md`, `docs/code-standards.md` |
| **Tools** | Claude Code, Git mandatory |
| **DoD** | system-architecture phản ánh module/schema/endpoint mới; roadmap đánh dấu phase complete; code-standards cập nhật convention mới (nếu có); link tới feature folder README |
| **Sample prompt** | "Sync project docs sau khi upsell-campaign-phase-2-1 released." |

---

## 4. Per-feature README

Mỗi feature folder **bắt buộc** có `README.md` lightweight (~30 lines max). Re-add ở v2.7 vì folder giờ chứa 5 file.

Người tạo: ai chạm folder đầu tiên (thường là BA viết SRS). Người update: mọi role sau đó.

### Sample

```markdown
---
doc_type: feature-readme
feature_slug: upsell-campaign
phase_slug: phase-2-upsell-campaigns
last_updated: 2026-05-05
---

# upsell-campaign — Feature Index

Phase: [phase-2-upsell-campaigns](../../po/phase-2-upsell-campaigns/00-prd.md)
Git branch: `feature/upsell-campaign`
Jira epic: — (not used)

## Documents

| # | Doc | Owner | Status | Version |
|---|---|---|---|---|
| 01 | [SRS](./01-srs.md) | @ba-jane | ✅ approved | 1.2 |
| 02 | [Design Spec](./02-design-spec.md) | @design-tom | ✅ approved | 1.0 |
| 03 | [Tech Design](./03-tech-design.md) | @lead-mike | ✅ approved | 1.1 |
| 04 | [Test Plan](./04-test-plan.md) | @qa-li | 🔄 in-review | 1.0 |
| 05 | [Release Plan](./05-release-plan.md) | @rm-anna | 📝 draft | 0.3 |

## Code

- FE: `app/modules/upsell-campaign/`
- BE: `app/services/upsell-campaign.server.ts`, `app/api/v1/upsell-campaign/`
- Schema: `prisma/schema/upsell-campaign.prisma`
- Storefront: `extensions/upsell-storefront/`

## Related

- ADR-0007 (cart discount via Shopify Functions)
- Spike-0012 (upsell research notes)
```

---

## 5. Naming conventions

### 5.1 Phase slug

Format: `phase-N-<short-name>` kebab-case.
- `phase-1-free-gift-campaigns`
- `phase-2-upsell-campaigns`

### 5.2 Feature slug

Format: kebab-case noun, **khớp domain language trong code**.
- `upsell-campaign` (matches `app/modules/upsell-campaign/`, `prisma/schema/upsell-campaign.prisma`)
- `free-gift-campaign`

Slug **không bao giờ rename** sau khi PRD `approved`. Rename = MAJOR bump tất cả docs liên quan + notify downstream.

### 5.3 Doc file names

Cố định, không sáng tạo:
- `00-prd.md` (chỉ ở `docs/po/<phase>/`)
- `README.md`
- `01-srs.md`
- `02-design-spec.md`
- `03-tech-design.md`
- `04-test-plan.md`
- `05-release-plan.md` (hoặc `05-release-plan-phase-N-M.md` cho multi-release)

### 5.4 ADR / Spike

- `docs/_adr/<NNNN>-<slug>.md` — `0001-choose-mongodb.md`, `0002-shopify-functions-for-cart-discount.md`.
- `docs/_spikes/<NNNN>-<slug>.md` — `0007-upsell-research.md`.
- Numbering monotonic 4 digit, không tái sử dụng số khi xoá.

### 5.5 Git branch (MANDATORY)

- Code: `feature/<feature-slug>` — e.g. `feature/upsell-campaign`.
- Doc-only: `docs/<feature-slug>` — e.g. `docs/upsell-campaign`.
- ADR-only: `adr/<NNNN>-<slug>` — e.g. `adr/0007-cart-discount-strategy`.

### 5.6 Phase scoping (multi-release feature)

Khi 1 feature ship qua nhiều phase (e.g. upsell-campaign Phase 2.1 PDP, 2.2 Cart, 2.3 Checkout):

- `feature_slug` **không đổi** giữa các release.
- Mỗi iteration có phase folder riêng: `docs/po/phase-2-1-upsell-pdp/`, `docs/po/phase-2-2-upsell-cart/`.
- SRS + Tech Design trong feature folder **MAJOR bump** mỗi release (1.0 → 2.0 cho Phase 2.2).
- Release Plan đặt tên per phase: `05-release-plan-phase-2-1.md`, `05-release-plan-phase-2-2.md`.
- Test Plan: 1 file duy nhất, hoặc thêm section per phase, hoặc MAJOR bump per phase — chọn 1 và nhất quán.

### 5.7 Field naming

- DDL + JSON: `snake_case` (`product_id`, `discount_percent`).
- TS code: `camelCase` (`productId`, `discountPercent`) qua serializer.
- Field name là **CONTRACT** giữa SRS §5 ↔ Tech Design §3 ↔ Zod ↔ code.
- Rename = MAJOR bump SRS + notify downstream owners (PR + Slack).

### 5.8 Figma frame naming

- Top frame match feature slug: `upsell-campaign / pdp / desktop`.
- Component naming match Design Spec §2: `UpsellOfferCard`, `UpsellAcceptButton`.

---

## 6. Handoff gates

### 6.1 Linear flow + parallel allowances

```
PO: PRD approved
  │
  ├─→ BA: SRS approved ─────────────────────────────┐
  │                                                  │
  │   (Designer có thể DRAFT parallel ở đây — §0.1)  │
  │                                                  ↓
  └────────────────→ Designer: Design Spec approved
                       │
                       │   (Tech Lead có thể DRAFT parallel)
                       ↓
                     Tech Lead: Tech Design approved
                       │   (Prisma §3 + endpoints §11)
                       │
              ┌────────┼────────────────┐
              ↓        ↓                ↓
        FE Dev:    BE Dev:           QA: Test Plan approved
        code       code              (parallel với FE/BE Impl)
        merged     merged
              │        │                │
              └────────┴────────────────┘
                       ↓
                  Release Manager: Release Plan approved
                       │   (gate trước khi flip flag)
                       ↓
                  Ship → flag 100%
                       ↓
                  Doc Maintainer: project-level docs sync
```

### 6.2 Gate rule chi tiết

| Gate | Pass condition | Ai gate |
|---|---|---|
| PRD → SRS | PRD `status: approved`, frontmatter đủ, Open Questions không block | PO + Tech Lead |
| SRS → Design Spec / Tech Design | SRS `status: approved`, §5 không có Prisma, §11 mọi story có AC | BA + PO + Tech Lead |
| Design Spec → Tech Design | Design Spec `status: approved` | Designer + Tech Lead |
| Tech Design → Impl | Tech Design `status: approved`, §3 Prisma đầy đủ, §11 endpoints đầy đủ | Tech Lead + senior reviewer |
| Impl → Release Plan | FE/BE merged, lint+test pass, **Test Plan §9 exit criteria green** | FE Lead + BE Lead + QA |
| Release Plan → Ship | Release Plan `approved`, flag tạo, monitoring sẵn, rollback drill done | Release Manager + Tech Lead + on-call |
| Ship → Docs Sync | Flag 100%, status `released`, no incident 24h | Tech Lead → Doc Maintainer |

### 6.3 Cross-feature dependency gate

PRD frontmatter có thể declare:

```yaml
depends_on_features: [free-gift-campaign]   # phase này cần feature kia ship trước
blocks_features: [checkout-personalization] # feature kia không thể proceed Tech Design tới khi PRD này ship
```

Gate: Tech Lead validate dependency graph khi review Tech Design. Nếu `depends_on_features` chưa `released` → Tech Design **không** approve.

### 6.4 Versioning bump (tóm tắt §7 WORKFLOW-STANDARD)

| Bump | Khi nào |
|---|---|
| **MAJOR** (1.x → 2.0) | Field rename · Business rule change · Breaking API shape · Removed section · State machine change · AC change |
| **MINOR** (1.0 → 1.1) | Add field · Add story · Add rule · Fix typo có ý nghĩa |
| **PATCH** (1.0.0 → 1.0.1) | Clarify wording, no semantic change |

`last_updated` MUST update mỗi content change. MAJOR bump → notify downstream owners (PR comment + Slack).

---

## 7. AI agreement — Claude Code / Codex

`AGENTS.md` (root repo) là contract chính cho AI agent. Tóm tắt trigger phrase per role:

| Role | Trigger phrase mẫu (VN/EN) | Skill kích hoạt |
|---|---|---|
| PO | "viết PRD", "create PRD", "phase requirements", "turn meeting notes into PRD" | `feature-prd-author` |
| BA | "viết SRS", "phân tích yêu cầu", "create acceptance criteria", "write user stories" | `feature-srs-author` |
| Designer | "viết design spec", "document Figma", "describe UI states", "spec for FE Dev" | `feature-design-spec-author` |
| Tech Lead | "viết tech design", "design DB schema", "module breakdown", "thiết kế kiến trúc" | `feature-tech-design-author` |
| QA | "viết test plan", "create QA test cases", "regression suite", "tạo test cases" | `feature-test-plan-author` |
| Release Manager | "viết release plan", "lập kế hoạch release", "feature flag plan", "kế hoạch rollback" | `feature-release-author` |
| Doc Maintainer | "đồng bộ docs sau release", "sync project docs", "update system-architecture" | `feature-docs-sync` |
| FE Dev | "implement FE", "viết React component", "convert Figma to code", "build the UI" | `feature-frontend-impl` |
| BE Dev | "implement BE", "viết API endpoint", "create migration", "build the service" | `feature-backend-impl` |

### Quy tắc AI:

1. **Match trigger** → invoke skill, **không** viết freehand. Skill enforce frontmatter + section count + cross-ref.
2. **Read upstream trước khi write**. Mọi skill đều mở docs tham chiếu trước.
3. **Không invent**. Field, endpoint, KPI, rule không có ở source → vào Open Questions.
4. **Không tự approve**. Status chuyển bởi human owner.
5. **Multiple skill match** → chạy sequential theo pipeline (PRD → SRS → … → Docs Sync).

### Minimal invocation

Không cần long bash prompt. Chỉ cần input tối thiểu — skill đọc context tự động:

- "viết SRS cho `upsell-campaign` từ `docs/po/phase-2-upsell-campaigns/00-prd.md`."
- "viết design spec cho `upsell-campaign`, Figma node `1:9695`."
- "implement BE cho `upsell-campaign` theo Tech Design §3 + §11."

---

## 8. "Ai đọc gì khi làm gì" — reading map

| Khi làm việc... | Đọc trước (bắt buộc) | Đọc tham khảo |
|---|---|---|
| **PO viết PRD** | OKR, customer feedback, spike (`docs/_spikes/`), `WORKFLOW-STANDARD.md` §3.1 | PRD phase trước, ADR liên quan |
| **BA viết SRS** | PRD `approved`, `WORKFLOW-STANDARD.md` §3.2, AGENTS.md §4.3 | SRS feature liên quan trong cùng phase |
| **Designer viết Design Spec** | SRS (draft OK), Figma file, Design Spec feature trước | Polaris docs, Brand guideline |
| **Tech Lead viết Tech Design** | SRS `approved`, Design Spec (draft OK), `_adr/`, `system-architecture.md`, Prisma schema hiện tại | Tech Design feature liên quan |
| **QA viết Test Plan** | SRS + Design Spec + Tech Design `approved` (đặc biệt SRS §3, §5, §6, §8, §11; Design Spec §2, §3; Tech Design §3, §7, §8, §9, §11) | Test plan feature trước, bug history |
| **FE Dev code** | SRS, Design Spec (full), Tech Design §2 + §4 + §11, Test Plan §4, Figma | `code-standards.md`, Polaris components |
| **BE Dev code** | SRS §3 + §5 + §7, Tech Design §2 + §3 + §4 + §7 + §11, Test Plan §4 | `code-standards.md`, Prisma docs |
| **Release Manager viết Release Plan** | Tech Design §8 + §10, Test Plan §9, FE/BE PR list | Release plan release trước |
| **Doc Maintainer sync** | Released feature folder, project-level docs hiện tại | ADR, roadmap |
| **AI agent (Claude Code)** | `AGENTS.md` + `WORKFLOW-STANDARD.md` + skill SKILL.md tương ứng + upstream doc(s) | feature folder gần nhất `approved` để mirror structure |

---

## 9. ADR + Spike conventions

### 9.1 ADR (Architecture Decision Records)

Dùng cho **quyết định kiến trúc cross-feature** (DB choice, lib choice, infra pattern).

- Path: `docs/_adr/<NNNN>-<slug>.md`.
- Format: **Michael Nygard** — Context · Decision · Status · Consequences.
- Numbering monotonic 4 digit (`0001`, `0002`, ...).
- **Immutable** sau khi `Accepted` — supersede bằng ADR mới với `Status: Supersedes ADR-NNNN`.
- Tech Design reference theo ID: "see ADR-0003".

### 9.2 Spike

Dùng cho **research / POC trước PRD**.

- Path: `docs/_spikes/<NNNN>-<slug>.md`.
- Format: free-form (problem · approach · findings · recommendation).
- Có thể update tới khi promote thành PRD.
- PRD reference theo ID nếu spike informed decision.

---

## 10. Final principle

> **"If AI reads the docs and still has to ask — the docs are wrong, not the AI."**

Triết lý kết: docs tồn tại để **eliminate ambiguity**. Mỗi câu hỏi từ AI agent là 1 lỗ hổng trong docs. Thay vì hardcode tri thức vào prompt, hardcode vào doc. Skill `feature-*` là wrapper tự động hoá việc enforce điều này.

Mọi review docs nên đặt câu hỏi:
1. AI agent có thể code 100% feature từ folder này mà không hỏi không?
2. Field name nào có rủi ro rename không?
3. Open Questions còn unresolved nào block downstream không?

Nếu cả 3 câu trả lời "yes / no / no" → docs sẵn sàng `approved`. Nếu không → bump version, update, re-review.

---

## Appendix A — File checklist before merge

Trước khi merge `docs/<feature-slug>` branch:

- [ ] `README.md` tồn tại, status table cập nhật
- [ ] Frontmatter mọi doc đủ field (xem §2)
- [ ] Section count đúng (PRD=8, SRS=11, DS=3, TD=12, TP=10, RP=7)
- [ ] BA không viết Prisma trong SRS §5
- [ ] Tech Design §3 Prisma đầy đủ (model + index + FK + ON DELETE)
- [ ] Tech Design §11 endpoint table 1-row-per-endpoint, không paste OpenAPI
- [ ] Mọi Open Questions có owner gán
- [ ] `last_updated` cập nhật
- [ ] Version bump đúng rule (§6.4)
- [ ] Git branch khớp convention (`feature/<slug>` hoặc `docs/<slug>`)
- [ ] Cross-feature deps (nếu có) khai báo ở PRD frontmatter

## Appendix B — Reference

- Workflow standard (terse): [`WORKFLOW-STANDARD.md`](./WORKFLOW-STANDARD.md)
- Agent contract: [`/AGENTS.md`](../../AGENTS.md)
- 9 skills: [`/.claude/skills/feature-*/SKILL.md`](../../.claude/skills/)
- Sample phase PRD: [`docs/po/phase-2-upsell-campaigns/00-prd.md`](../po/phase-2-upsell-campaigns/00-prd.md)
- Sample feature folder: [`docs/features/upsell-campaign/`](../features/upsell-campaign/)
- ADR index: [`docs/_adr/README.md`](../_adr/README.md)
- Spike index: [`docs/_spikes/README.md`](../_spikes/README.md)
