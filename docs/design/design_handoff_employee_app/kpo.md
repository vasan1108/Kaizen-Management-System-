# Handoff: KIS KPO Web App (Desktop)

## Overview
Desktop screens for the KPO (implementation) role. KPO receives suggestions approved by the Plant Head, tracks implementation via free-form stages (custom labels, remarks, photos), enters approx. cost savings, and can toggle Horizontal Deployment to clone the suggestion to other plants. Built against `build-spec.md` in `vasan1108/Kaizen-Management-System-` (§3 data model — `ImplementationStage`/`CostRecord`, §4 state machine, §10 Frontend Pages — KPO).

Same structural pattern as Dept Head / Plant Head: desktop-only, table dashboard, two-column detail layout.

## About the Design Files
`kaizen-kpo-app.html` and `browser-window.jsx` are design references; `browser-window.jsx` is a presentation-only browser-chrome wrapper. Recreate in the target stack, not by embedding this HTML.

## Fidelity
High-fidelity — final colors, type, spacing, copy.

## Design System
Same as other roles: **Modernist** base, primary `#ffd6a8` (peach), contrast `#1c1a17` (ink). Full token table in `employee.md`.

## Screens / Views

### 1. Dashboard (Desktop)
- Nav bar, 4-column stat row: **In progress / High priority / Implemented / Horizontal deploy** (different metric set from DH/PH — reflects KPO's own funnel).
- Search bar + **department dropdown** + priority filter chips (ALL/HIGH/MEDIUM/LOW) — same controls as Plant Head.
- **Table columns:** Ref ID, Employee (name + dept/line), Issue (truncated), Priority (swatch + label), **Stage** (replaces "Time in queue" — shows current implementation stage as a pill: `PROCUREMENT`/`TRIAL RUN` (peach fill, in progress), `NOT STARTED` (outlined), `IMPLEMENTED` (ink fill, done)).
- Content constrained to 960px centered column.

### 2. Suggestion Detail — Implementation (Desktop)
- Two-column layout (960px max, left ~580px content / right 380px action panel).
- **Left column:** Ref ID, title, meta (employee, line, "Approved by Plant Head" + date). Issue + Idea text (no photo/idea toggle needed here — KPO isn't the reviewer). Below a divider: **"Implementation stages"** section header with an "ADD STAGE" affordance (top-right), then a **vertical stepper** of existing stages — each stage: marker (ink-filled = done, peach-filled/ink-border = current), custom title (KPO-authored, e.g. "Procurement", "Trial run"), remarks text, date, and optional photo thumbnails (52×52px) — mirrors the Employee/DH detail stepper pattern but stages are free-form/custom-labeled per spec (not a fixed set).
- **Right column (action panel):**
  - "Approx. cost savings" — currency-prefixed input (₹ shown; use org default currency per open question in spec §13).
  - "Horizontal Deployment" toggle — label + description ("Clone this suggestion to other plants"), switch control (on = peach fill).
  - "New stage" mini-form: title input, remarks textarea, photo upload tile (dashed border, `+`).
  - "SAVE STAGE" (ink-filled) and "MARK IMPLEMENTED" (outlined) actions, stacked full-width.

## Interactions & Behavior (to implement)
- **Department dropdown + priority chips + search:** same filtering pattern as other role dashboards.
- **ADD STAGE / new-stage form:** `POST /suggestions/:id/stages` — appends to the stepper; stage_order per spec §3, no fixed vocabulary (KPO writes free-form titles).
- **Horizontal Deployment toggle:** `PATCH /suggestions/:id/horizontal-deployment` — per build-spec §4, this clones the suggestion into `DH_PENDING` in every other active plant's matching department; clones are visible only to DH/PH in target plants, never reach KPO/Finance there.
- **Approx. cost savings input:** `POST /suggestions/:id/cost/approx` — becomes the "approx" half of the CostRecord Finance later compares against actual.
- **MARK IMPLEMENTED:** transitions suggestion to `KPO_IMPLEMENTED` → auto `FINANCE_MONITORING` per the state machine (§4); should likely be disabled/confirm-gated until at least one stage exists.

## State Management
- Queue fetched via API scoped to plant + `status IN (PH_APPROVED, KPO_IN_PROGRESS)`.
- Stage list, cost record, and horizontal-deployment state are server-persisted (`ImplementationStage`, `CostRecord` tables) — not client-derived; each stage save/toggle should refetch or optimistically update the stepper.

## Files
- `kaizen-kpo-app.html` — full design source (2 screens).
- `browser-window.jsx` — browser-chrome wrapper, presentation only.
- See `employee.md` for shared tokens + Sign In (reused as-is), `planthead.md` for the Plant Head approval this role consumes.
