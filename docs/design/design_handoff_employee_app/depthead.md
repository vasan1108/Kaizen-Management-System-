# Handoff: KIS Department Head Web App (Desktop)

## Overview
Desktop/laptop screens for the Department Head role of the Kaizen Implementation System (KIS). Dept Heads review suggestions submitted by employees in their department, set priority, and approve/reject with remarks before escalation to the Plant Head. Built against `build-spec.md` in `vasan1108/Kaizen-Management-System-` (§4 state machine, §10 Frontend Pages — Dept Head).

Unlike the Employee role (mobile, shop floor), Dept Head and all subsequent roles (Plant Head, KPO, Finance, Admin) use laptop/desktop screens — table-based dashboards for readability at scale.

## About the Design Files
`kaizen-depthead-app.html` and `browser-window.jsx` are **design references** — `browser-window.jsx` is only a browser-chrome mockup wrapper for presentation; the real app has no browser bezel. Recreate in the target stack (React + TypeScript + Tailwind, per build spec) rather than embedding this HTML.

## Fidelity
High-fidelity. Colors, type, spacing, copy, and the table column set below are final.

## Design System
Same as Employee handoff: **Modernist** base (flat, 0 radius, 2px rules, Archivo), with brand override: primary `#ffd6a8` (peach), contrast `#1c1a17` (ink). See `employee.md` in this folder for the full token table — unchanged here.

## Screens / Views

### 1. Dashboard (Desktop)
- **Purpose:** Department-wide suggestion queue overview + quick stats.
- **Nav bar:** 64px, "KIS" wordmark, "Dashboard" (active, underlined) / "New Suggestion" nav items, right-aligned user identity ("Priya Menon · Dept Head · Assembly Line 2, Plant 3").
- **Stat row:** 4-column grid (Pending / High priority / Approved / Rejected), each cell: big number (30px/800) + uppercase label. 2px dividers between cells (grid gap rendered as background line).
- **Search bar:** left-aligned, 320px, icon + placeholder "Search by ref ID, employee, or keyword" — filters the table client-side or via `GET /suggestions?search=`.
- **Priority filter chips:** ALL (active, ink-filled) / HIGH / MEDIUM / LOW, right-aligned next to search.
- **Table** — columns, in order:
  1. **Ref ID** — monospace, muted (`KIS-PLT03-000512`)
  2. **Employee** — name (14px/600) + department/line as a secondary line (11px muted)
  3. **Issue** — single-line summary, ellipsis-truncated (`table-layout:fixed`, no fixed pixel width so it flexes with the container)
  4. **Priority** — colored square swatch (ink=High, peach outline=Medium, outline only=Low) + label
  5. **Time in queue** — relative duration (e.g. `2h 10m`, `1d 3h`)
  - Header row: 2px bottom border under `#1c1a17`, uppercase 11px/800 labels. Row dividers 2px `--ink-15`. Row click → Suggestion Detail.
- Content area is constrained to a **960px centered column**, not full window width — table stays readable, doesn't stretch edge-to-edge on wide monitors.

### 2. Suggestion Detail — Review (Desktop)
- **Purpose:** Full suggestion content + Dept Head decision (priority, remarks, approve/reject).
- **Nav bar:** back arrow + "Back to queue", user identity right-aligned.
- **Two-column layout** (960px max-width, centered): left column (flex, ~620px) = suggestion content; right column (340px) = decision panel. 2px divider between them.
- **Left column:**
  - Ref ID (monospace), title (24px/800), meta line (employee name + ID, line/area, submitted timestamp).
  - "Issue" section: EN/ORIGINAL language toggle (top-right of section), issue text (14px/1.6), photo thumbnails (100×100px, `--surface` fill).
  - "Idea" section below a 2px divider: idea text only (no photos in this mock — add a sketch thumbnail row if the idea includes one).
- **Right column (decision panel):**
  - "Set priority" — 3-way segmented control (LOW / MEDIUM / HIGH), selected segment peach-filled.
  - "Remarks (required)" — textarea, placeholder "Add a note for the Plant Head…". Required before either action per build-spec §4 (remarks travel with the review record).
  - Action row: "REJECT" (outlined red `#b3402a`) and "APPROVE" (ink-filled, peach text), equal width.

## Interactions & Behavior (to implement)
- **Search:** debounced filter against ref ID / employee name / issue text.
- **Priority filter chips:** single-select filter on the table; combines with search.
- **Row click:** navigates to Suggestion Detail for that ref ID.
- **Priority segmented control (detail):** sets `SuggestionReview.priority` before submission; required field.
- **Remarks:** required textarea — disable both Approve/Reject until non-empty (per spec, remarks are stored on the `SuggestionReview` record regardless of decision).
- **Approve:** transitions suggestion status to next stage (Plant Head review) per the build-spec §4 state machine; **Reject:** transitions to `REJECTED`, remarks shown to the employee in their Suggestion Detail timeline.
- **EN/ORIGINAL toggle:** swaps issue text between translated (per submitting employee's language) and original — same pattern as Employee detail screen.

## State Management
- **Queue data:** fetched from `GET /suggestions?status=DH_REVIEW&department=...` — not client-derived.
- **Decision draft:** priority + remarks held locally until Approve/Reject submitted (`PATCH`/`POST` per build-spec §9), then queue refetches.
- **Auth/session:** same JWT pattern as Employee (build-spec §8/§9); Dept Head role gates access to this dashboard (role-based routing).

## Assets
Inline SVG icons approximating **Lucide** (`search`, `chevron-left`). Recreate with the real Lucide package.

## Files
- `kaizen-depthead-app.html` — full design source (2 screens, self-contained design component).
- `browser-window.jsx` — browser-chrome wrapper used only for presentation; not part of the app.
- See `employee.md` (same folder) for the shared design token table and Sign In screen — Dept Head reuses Employee's Sign In as-is.
