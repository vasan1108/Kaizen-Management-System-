# KIS — Kaizen Implementation System
### Build Specification for Claude Code

This document is the authoritative build spec. It translates the source requirements
(`KIS_flow.docx`) into a concrete, production-grade web application design. Where the
source document was silent or ambiguous, an explicit **Assumption** is stated — treat
these as defaults to implement, and flag to the user if they need to change.

---

## 1. Product Summary

KIS is an internal web application for a multi-plant manufacturing organization that
lets shop-floor **Employees** submit Kaizen (continuous-improvement) suggestions in
their own language, and routes each suggestion through a fixed approval hierarchy —
**Department Head → Plant Head → KPO (implementation) → Finance (cost validation)** —
with SLA timers, email escalation, and full audit history.

---

## 2. Roles & Permissions Matrix

| Capability | Employee | Dept Head (DH) | Plant Head (PH) | KPO | Finance | Admin |
|---|---|---|---|---|---|---|
| Register self | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Created by Admin | ❌ | ✅ | ✅ | ✅ | ✅ | (seed/superuser) |
| Submit suggestion | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| View own suggestions | ✅ | — | — | — | — | — |
| View dept-wide queue | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| View plant-wide queue | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| View queue by department/priority filters | ❌ | ✅ | ✅ | ✅ | ✅ | — |
| Set priority | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Approve/Reject | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Add implementation stages/photos | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Toggle Horizontal Deployment | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Enter approx. cost savings | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Enter actual monitored savings | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| Manage plants/departments | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Create/activate/deactivate any non-employee account | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Reassign roles | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| View/export MIS reports (org-wide analytics) | ❌ | ❌ | ❌ | ❌ | ✅ (cost-focused) | ✅ (full) |

**Confirmed A1:** Employees self-register freely (empid, name, mobile, dept, plant,
password) — no HR master list validation against `empid`. All other roles (DH, PH,
KPO, Finance) are created only by Admin with a system-generated default password and
**must reset on first login** (confirmed).

**Assumption A2:** One user has exactly one role per plant. A person needing DH role
in two plants gets two user records (kept simple; revisit if this becomes a real need).

**Assumption A6:** The source document scopes Admin to org-structure/user
administration only. Since the user has now asked for Admin-facing MIS reports,
Admin's read access is extended to **aggregated, org-wide suggestion analytics**
(counts, rates, trends) — not individual suggestion content/remarks, which stay
scoped to the DH/PH/KPO/Finance chain per the original design. See §11.

---

## 3. Data Model

```
Plant
  id, name, code, is_active, created_at

Department
  id, plant_id (FK), name, is_active

User
  id, empid (unique), name, mobile_number, email, plant_id (FK), department_id (FK, nullable for PH/KPO/Finance),
  role (enum: EMPLOYEE, DEPT_HEAD, PLANT_HEAD, KPO, FINANCE, ADMIN),
  password_hash, must_reset_password (bool), is_active (bool),
  preferred_language (enum: en, ta, hi, mr, pa),
  created_at, updated_at

Suggestion
  id, reference_code (human-readable, e.g. KIS-PLT01-000123),
  employee_id (FK -> User),
  plant_id (FK), department_id (FK),
  line_area (text), exact_spot (text, nullable),
  issue_text_original (text, max 250 words), issue_text_language (enum),
  issue_text_en (text, nullable — populated by translation job),
  idea_text_original (text, nullable), idea_text_en (text, nullable),
  issue_photo_urls (text[]), solution_sketch_urls (text[]),
  priority (enum: LOW, MEDIUM, HIGH, nullable until set by DH),
  status (enum — see State Machine §5),
  parent_suggestion_id (FK -> Suggestion, nullable — set on horizontally-deployed clones),
  is_horizontal_deployment_source (bool, default false),
  created_at, updated_at

SuggestionReview
  id, suggestion_id (FK), reviewer_id (FK -> User), reviewer_role (enum: DEPT_HEAD, PLANT_HEAD),
  decision (enum: APPROVED, REJECTED), remarks (text, required), decided_at,
  sla_due_at, sla_breached (bool)

ImplementationStage
  id, suggestion_id (FK), kpo_id (FK -> User),
  stage_title (text — custom label, e.g. "Procurement", "Installed", "Trial Run", "Implemented"),
  remarks (text), photo_urls (text[]), stage_order (int), created_at

CostRecord
  id, suggestion_id (FK),
  approx_savings_amount (decimal, set by KPO),
  approx_savings_currency (default org currency),
  actual_savings_amount (decimal, nullable, set by Finance),
  monitoring_status (enum: MONITORING, MONITORED),
  monitoring_period_start, monitoring_period_end,
  recorded_by_finance_id (FK -> User), updated_at

NotificationLog
  id, suggestion_id (FK), recipient_id (FK -> User), type (enum: DH_NUDGE, PH_SLA_BREACH),
  sent_at, channel (enum: EMAIL)

AuditLog
  id, entity_type, entity_id, actor_id (FK -> User), action, diff (jsonb), created_at

ReportExportLog
  id, requested_by_id (FK -> User), report_type (enum — see §11), filters (jsonb),
  export_format (enum: CSV, XLSX, PDF), generated_at
```

**Assumption A3:** Reference codes and audit logging are not explicitly requested but
are treated as required for a "production-grade" system handling approvals and money
(cost savings) — include them.

---

## 4. Suggestion Lifecycle — State Machine

```
DRAFT
  → SUBMITTED                          (employee submits)
  → DH_PENDING                         (auto, on submission)
  → DH_REJECTED  | DH_APPROVED         (DH sets priority + decision + remarks)
DH_APPROVED
  → PH_PENDING                         (auto; SLA timer starts based on priority)
  → PH_REJECTED  | PH_APPROVED
PH_APPROVED
  → KPO_IN_PROGRESS                    (KPO adds implementation stages freely, any order/labels)
  → KPO_IMPLEMENTED                    (KPO marks final stage complete)
KPO_IMPLEMENTED
  → FINANCE_MONITORING                 (auto; Finance can now log actual savings)
  → FINANCE_MONITORED                  (Finance closes out monitoring period)
  → CLOSED
(any DH/PH stage) → REJECTED           (terminal; employee notified, remains visible in employee dashboard)
```

Horizontal deployment: when KPO toggles "Horizontal Deployment" on a suggestion in
`KPO_IN_PROGRESS` or later, the system clones the suggestion (issue/idea text,
translations, photos) into a **new Suggestion row per other active plant**, each with
`parent_suggestion_id` set to the original, `status = DH_PENDING`, and its own
`department_id` resolved by matching department name/type across plants
(**Assumption A4** — fallback: clone into every department of the target plant and
let each plant's DH triage).

**Confirmed:** Horizontal-deployment clones are visible only to **DH and PH** in the
target plants — they follow the normal DH → PH approval chain independently, but do
**not** flow onward to KPO or Finance in those plants. KPO/Finance visibility and
cost tracking remain scoped to the original suggestion only.

---

## 5. SLA & Notification Engine

- Runs as a scheduled background job (every 15–30 min), not on-request.
- **DH nudge:** if a suggestion has been `DH_PENDING` for > 2 days, email the DH once
  per day until actioned.
- **PH SLA:** on entering `PH_PENDING`, compute `sla_due_at` = now + (5d Low / 3d
  Medium / 2d High). If now > `sla_due_at` and still pending, email PH and mark
  `sla_breached = true` (log to `NotificationLog`, avoid duplicate sends same day).
- All emails link directly to the suggestion detail view.

**Assumption A5:** No SLA is specified for KPO or Finance stages in the source
document — none is implemented for those stages initially; flag as a possible V2 add.

---

## 6. Internationalization & Translation

- Supported UI languages: English, Tamil, Hindi, Marathi, Punjabi.
- Employee selects language once; entire suggestion-creation flow (labels, speech
  input, submitted text) renders/operates in that language.
- **Speech-to-text:** browser-native Web Speech API for the Issue and Idea fields,
  configured to the selected language's locale code.
- **Translation to English:** on submission, an async job calls a **self-hosted
  LibreTranslate instance** (open-source, runs as its own Docker container) to
  populate `issue_text_en` / `idea_text_en`. DH/PH/KPO/Finance views always show the
  English version with a toggle to see the original.
- Word-count validation (250-word cap on Issue) is enforced client- and server-side
  on the **original-language** text.

---

## 7. MIS Reporting (Admin & Finance)

Both Admin and Finance get a dedicated **Reports** area, built as read-only
aggregation queries over the existing tables (§3) — no new transactional logic,
just indexed queries + export.

### 7.1 Common capabilities
- **Filters:** date range, plant, department, priority, status — combinable.
- **Export formats:** CSV, Excel (`.xlsx`), and PDF — generated server-side and
  self-hosted (no cloud reporting SaaS): `exceljs` for Excel, `pdfkit` for PDF.
- Every export is written to `ReportExportLog` (who ran it, what filters, when) —
  consistent with the audit-first approach already used for suggestion decisions.
- Reports are **aggregated only**: Admin reports never expose individual issue/idea
  text or reviewer remarks; Finance reports never expose non-financial suggestion
  content beyond what Finance already sees in its normal queue (§9).

### 7.2 Admin reports
- **Suggestion Volume Report:** submitted / approved / rejected / pending counts,
  sliceable by plant, department, and time period (daily/weekly/monthly rollups).
- **SLA Compliance Report:** DH nudge counts and PH SLA-breach counts and rates, by
  plant and priority.
- **Employee Participation Report:** submissions per employee/department, top
  contributors, participation rate (submitters ÷ active employees) by plant.
- **Horizontal Deployment Report:** how many suggestions were cloned, to how many
  plants, and their independent approval outcomes.
- **User & Account Report:** active/inactive counts by role and plant, accounts
  still pending first-login password reset.

### 7.3 Finance reports
- **Cost Savings Report:** approx. (KPO) vs actual (Finance) savings, by plant,
  department, and time period.
- **Savings Realization Rate:** actual ÷ approx. ratio, to see how accurate KPO
  estimates are over time — by plant/department.
- **Monitoring Status Report:** counts of suggestions in `MONITORING` vs
  `MONITORED`, and how many have exceeded their `monitoring_period_end` without
  being closed out.
- **Plant-wise Savings Trend:** actual savings over time per plant, for
  period-over-period comparison.

**Assumption A7:** No specific report list was in the source document — the above is
a reasonable default set covering the metrics implied by the workflow (SLA
compliance, participation, and the approx-vs-actual cost split). Treat this list as a
starting point to confirm/trim with the user rather than final.

---

## 8. Technology Stack

Locked to the team's preferred stack: **Express, React, PostgreSQL** — and, per
requirement, **no cloud/SaaS services**. Everything below is self-hosted, typically
as sibling containers in the same Docker Compose / server deployment.

| Layer | Choice | Why |
|---|---|---|
| Frontend | React (Vite) + TypeScript + Tailwind CSS + React Router | Familiar stack, fast dev server, client-rendered dashboards are fine here (no SEO need — internal tool) |
| i18n | `react-i18next` | Standard, well-supported, works cleanly with the Web Speech API's locale codes |
| Backend | Express (Node.js/TypeScript) | Team's comfort zone; structure it as feature modules (routes/controllers/services per domain: auth, admin, suggestions, uploads) with explicit RBAC middleware rather than relying on a framework's DI |
| Database | PostgreSQL (self-hosted container) | Relational integrity for the approval hierarchy; JSONB for audit diffs |
| ORM | Prisma | Type-safe schema matching §3, easy migrations |
| Queue/Jobs | Redis + BullMQ (self-hosted container) | SLA scanner, translation jobs, email dispatch — run as a separate worker process from the Express API |
| File storage | **MinIO** (self-hosted, S3-compatible object storage) | Photos and solution sketches; same S3 SDK/API as cloud S3 so the storage-client code stays swappable, but the running service is entirely on-prem |
| Reporting/Export | `exceljs` (Excel), `pdfkit` (PDF), plain CSV writer | Self-hosted MIS report generation (§7) — no external BI/reporting SaaS |
| Auth | JWT (access + refresh), bcrypt password hashing, Express middleware guards | Stateless, scalable, no external identity provider |
| Email | Self-hosted SMTP relay (e.g. Postfix, or a self-hosted transactional stack like Postal/Mailu) | Transactional SLA/nudge emails without a third-party mail API |
| Translation | **LibreTranslate** (self-hosted, open-source, Docker container) | Supports English, Tamil, Hindi, Marathi; confirm Punjabi model coverage — see note below |
| Deployment | Docker Compose on organization-owned server(s)/VMs | No managed cloud platform; all services (api, worker, web, postgres, redis, minio, libretranslate, smtp) run as containers on infrastructure the org controls |
| CI/CD | Self-hosted runner (Jenkins, or Gitea/GitLab CE with local runners) | Keeps build/deploy pipeline off third-party cloud CI |
| Monitoring | Self-hosted logging/metrics (Grafana + Loki + Promtail, or plain structured logs via `pino` shipped to local disk) | Production visibility without an external SaaS APM |

**Note on LibreTranslate language coverage:** LibreTranslate's supported-language set
depends on which Argos Translate models are installed. English, Hindi, Marathi, and
Tamil are generally available; **Punjabi coverage should be verified when setting up
the LibreTranslate instance** — if a direct Punjabi↔English model isn't available,
the fallback is pivoting through Hindi, or documenting Punjabi as a known gap for a
first release.

**Suggested repo layout:**
```
/apps
  /api        (Express app: routes, controllers, services, middleware, prisma schema)
  /worker     (BullMQ processors: SLA scanner, translation jobs, email dispatch)
  /web        (React/Vite app: pages per role, shared components, i18n locale files)
/packages
  /shared     (shared TypeScript types/DTOs between api and web, if using a monorepo tool like Turborepo/pnpm workspaces)
/infra
  docker-compose.yml   (api, worker, web, postgres, redis, minio, libretranslate, smtp relay)
```

---

## 9. API Surface (high-level)

```
Auth
  POST   /auth/register                 (employee self-register)
  POST   /auth/login
  POST   /auth/reset-password           (forced reset for admin-created accounts)
  POST   /auth/refresh

Admin
  POST   /admin/plants
  POST   /admin/departments
  POST   /admin/users                    (create DH/PH/KPO/Finance)
  PATCH  /admin/users/:id                (update role/assignment)
  PATCH  /admin/users/:id/status         (activate/deactivate)
  GET    /admin/plants/:id/users

Suggestions
  POST   /suggestions                    (employee submit)
  GET    /suggestions/mine               (employee dashboard)
  GET    /suggestions/:id
  GET    /suggestions?role-scoped-filters (DH/PH/KPO/Finance queues; filters: department, priority, time-in-queue)
  PATCH  /suggestions/:id/priority       (DH only)
  POST   /suggestions/:id/review         (DH/PH decision + remarks)
  POST   /suggestions/:id/stages         (KPO adds implementation stage)
  PATCH  /suggestions/:id/horizontal-deployment
  POST   /suggestions/:id/cost/approx    (KPO)
  POST   /suggestions/:id/cost/actual    (Finance)
  GET    /suggestions/:id/audit

Uploads
  POST   /uploads                        (signed URL or direct upload for photos/sketches)

Reports
  GET    /reports/admin/:reportType      (volume, sla-compliance, participation, horizontal-deployment, users; query params = filters from §7.1)
  GET    /reports/finance/:reportType    (cost-savings, realization-rate, monitoring-status, savings-trend)
  POST   /reports/:reportType/export     (body: format = csv|xlsx|pdf, filters; returns file, logs to ReportExportLog)
```

All non-auth, non-register routes are protected by a role guard resolved from the
JWT; queue endpoints are additionally scoped to the caller's `plant_id`/`department_id`.

---

## 10. Frontend Pages (per role)

- **Employee:** Language picker → Submit Suggestion form (voice + text) → My
  Suggestions (status timeline with timestamps)
- **Dept Head:** Queue (counts: approved/rejected/pending; filter by priority) →
  Suggestion Detail (English translation, priority selector, approve/reject + remarks)
- **Plant Head:** Queue (filters: department, priority, time-in-queue, SLA status) →
  Suggestion Detail (same review pattern, SLA countdown visible)
- **KPO:** Queue (filters: department, priority) → Suggestion Detail (stage tracker,
  add stage + remarks + photos, approx. cost input, Horizontal Deployment toggle)
- **Finance:** Queue (filters: plant, department, priority, time-in-queue) →
  Suggestion Detail (monitoring period selector, actual savings input, monitoring
  status toggle) → **MIS Reports** tab (cost-savings, realization-rate,
  monitoring-status, savings-trend; filter panel + export buttons)
- **Admin:** Plant list → Plant detail (departments + all users by role) → User
  create/edit/activate-deactivate → Role reassignment → **MIS Reports** tab (volume,
  SLA compliance, participation, horizontal deployment, user/account report; filter
  panel + export buttons)

---

## 11. Non-Functional Requirements

- **Security:** RBAC enforced server-side (never trust client role claims beyond the
  JWT); rate-limit auth endpoints; sanitize/validate all uploads (image type/size
  limits); password policy + forced reset for admin-issued accounts.
- **Auditability:** every status transition and review decision written to
  `AuditLog` with actor and diff.
- **Performance:** queue list endpoints must be paginated and indexed on
  `(plant_id, department_id, status, priority)`; report aggregation queries should
  use dedicated indexes on `(plant_id, department_id, created_at, status)` and, for
  cost reports, on `CostRecord(plant via suggestion, monitoring_status)` — consider
  materialized views if report queries get slow against live tables at scale.
- **Reliability:** translation and email jobs run via BullMQ with retry/backoff;
  failures logged, not silently dropped.
- **Testing:** unit tests for the state machine transitions and SLA calculation
  (these are the highest-risk business logic); integration tests for role-scoped API
  access; e2e happy-path test per role.
- **Accessibility/i18n:** all 5 locales must have complete translation files before
  a language is enabled for selection.

---

## 12. Phased Build Plan

**Phase 1 — Foundation**
Auth (register/login/reset), Plant/Department/User admin CRUD, RBAC guards, base
DB schema + migrations.

**Phase 2 — Core Suggestion Flow**
Employee submission form (incl. speech-to-text, photo upload, word-count
validation), DH review + priority setting, PH review + SLA timer, translation job.

**Phase 3 — Implementation & Finance**
KPO stage tracker + approx. cost, Horizontal Deployment cloning, Finance monitoring
+ actual cost, suggestion closure.

**Phase 4 — Notifications & Hardening**
SLA scanner job, email nudges/breach alerts, audit log surfacing in UI, dashboards
with counts/filters polished, load/security testing.

**Phase 5 — MIS Reporting**
Report aggregation queries (§7.2/§7.3), filter UI, CSV/Excel/PDF export, export
audit logging — built once §2–§4 data is flowing so reports have real data to
validate against.

**Phase 6 — Production Readiness**
Dockerization, CI/CD pipeline, monitoring/alerting, staging deploy, UAT with real
plant/department data, go-live.

---

## 13. Open Questions to Confirm Before/During Build

1. What is the organization's default currency for cost savings?
2. Any existing SSO (e.g. Azure AD) to integrate instead of local password auth?
3. Is the report list in §7.2/§7.3 complete, or are there specific MIS reports
   already in use (e.g. an existing Excel template) that the new reports should
   match in structure?

---
*Generated from `KIS_flow.docx`. Assumptions A1, A2, and A4's visibility scope have
been confirmed by the user. A3 (audit logging), A5 (no SLA for KPO/Finance stages),
A6 (Admin's extended reporting access), and A7 (the specific MIS report list) remain
open defaults — revise this file directly if any should change before implementation
begins.*
