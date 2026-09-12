# Handoff: KIS Admin Web App (Desktop)

## Overview
Desktop screens for the Admin role: org-structure management (plants, departments, users) and org-wide MIS reporting. Built against `build-spec.md` in `vasan1108/Kaizen-Management-System-` (§2 roles/permissions, §3 data model, §7.2 Admin reports, §10 Frontend Pages — Admin).

Same structural pattern as other back-office roles: desktop-only, table-driven, 960px centered content.

## About the Design Files
`kaizen-admin-app.html` and `browser-window.jsx` are design references; `browser-window.jsx` is a presentation-only browser-chrome wrapper. Recreate in the target stack, not by embedding this HTML.

## Fidelity
High-fidelity — final colors, type, spacing, copy.

## Design System
Same as other roles: **Modernist** base, primary `#ffd6a8` (peach), contrast `#1c1a17` (ink). Full token table in `employee.md`.

## Screens / Views

### 1. Plant List
- Nav: **Plants** (active) / **MIS Reports** tabs.
- "Plants" header + "NEW PLANT" button (ink-filled).
- Table: Code, Plant name, Depts (count), Users (count), Status (ACTIVE ink-filled / INACTIVE outlined). Row click → Plant Detail.

### 2. Plant Detail (Departments & Users)
- Back-to-plants nav. Plant code + name header.
- "Departments (n)" section: chip row (outlined pills) + "+ Add department" link.
- "Users (n)" section: "NEW USER" button, role filter chips (ALL/DEPT HEAD/PLANT HEAD/KPO/FINANCE), table (Name, Role, Department (— if plant-scoped role), Status — ACTIVE / **PENDING RESET** for admin-created accounts awaiting forced password reset per build-spec §2 confirmed A1, Edit link).

### 3. User Create / Edit
- Centered form (520px), narrower window (900px) than the table screens.
- Note under the title: "System-generated password, forced reset on first login" — per spec, all non-Employee accounts are Admin-created with a default password and must reset.
- Fields: Full name, Employee ID (2-col grid), Mobile number, Email (2-col grid), Role (4-way segmented: DEPT HEAD/PLANT HEAD/KPO/FINANCE), Department (dropdown, only relevant for Dept Head — KPO/PH/Finance have no department per data model `department_id` nullable for those roles).
- Actions: CANCEL (outlined) / CREATE USER (ink-filled).

### 4. MIS Reports — 5 report types (build-spec §7.2)
Shared shell: nav with MIS Reports tab active, 5 report-type tabs (VOLUME / SLA COMPLIANCE / PARTICIPATION / HORIZONTAL DEPLOYMENT / USERS & ACCOUNTS — active tab ink-filled), a filter row (date range + plant/department/role dropdowns as relevant per report) with CSV/XLSX/PDF export buttons on the right, then report-specific stats/charts/tables:

- **4a Volume:** 4-stat row (Submitted/Approved/Rejected/Pending) + grouped bar chart (submissions by plant, monthly).
- **4b SLA Compliance:** 3-stat row (DH nudges sent/PH SLA breaches/Breach rate) + breach-rate table by plant × priority (High/Medium/Low/Overall columns).
- **4c Participation:** 3-stat row (Active submitters/Active employees/Participation rate) + org-wide top-contributors table (rank, employee, plant, department, suggestion count).
- **4d Horizontal Deployment:** 3-stat row (Suggestions cloned/Avg. plants per clone/Clone approval rate) + recent-deployments table (source ref ID, issue, cloned-to plants, outcomes).
- **4e Users & Accounts:** 4-stat row (Active accounts/Inactive accounts/Pending first-login reset/Employees self-registered) + accounts-by-role×plant matrix table + a searchable **all-employees table** (Emp ID, Name, Plant, Department, Status) filtered by the same plant/role controls.

Per build-spec §7.1: Admin reports are aggregated only — never expose individual issue/idea text or reviewer remarks. The employees table on 4e is a legitimate exception (user/account admin data, not suggestion content).

## Interactions & Behavior (to implement)
- **Plant/user CRUD:** standard create/edit/activate-deactivate flows per `POST/PATCH /admin/plants`, `/admin/departments`, `/admin/users`, `/admin/users/:id/status` (build-spec §9).
- **Role reassignment:** edit an existing user's role via the same form (§2 permissions matrix drives which fields show — e.g. Department only for Dept Head).
- **Report tabs:** swap stats/chart/table per type; filters persist across tab switches.
- **Export:** `POST /reports/admin/:reportType/export` with filters + format; logged to `ReportExportLog`.
- **Employees table (4e):** filtered by the plant/role dropdowns already on that report; search narrows further client-side or via query param.

## State Management
All data server-fetched (plants/departments/users via `/admin/*`, reports via `/reports/admin/:reportType`) — not client-derived. Filters are query params shared across a report's stat/chart/table sections.

## Files
- `kaizen-admin-app.html` — full design source (screens 1–3 plus 4a–4e report variants).
- `browser-window.jsx` — browser-chrome wrapper, presentation only.
- See `employee.md` for shared tokens + Sign In (reused as-is).
