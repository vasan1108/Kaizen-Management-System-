# Handoff: KIS Employee Mobile App

## Overview
Mobile app screens for the Employee role of the Kaizen Implementation System (KIS) — a shop-floor continuous-improvement suggestion tool. Covers the full employee flow: signing in, viewing a dashboard, submitting a suggestion (issue + idea, voice or text, with photos), reviewing before submit, confirmation, and tracking submitted suggestions through the approval/implementation pipeline.

Built against `build-spec.md` in the `vasan1108/Kaizen-Management-System-` GitHub repo (§4 state machine, §6 i18n, §10 Frontend Pages — Employee). That repo currently contains only the spec (no existing UI code) — this design is the first UI pass, not a recreation of existing screens.

## About the Design Files
The files in this bundle (`kaizen-employee-app.html`, `ios-frame.jsx`) are **design references built in HTML/React for prototyping only** — not production code to copy directly. The task is to recreate these designs in the target codebase's actual stack (per the build spec: React + TypeScript + Tailwind CSS + React Router, likely with a mobile-web or React Native shell) using that stack's own component patterns. `ios-frame.jsx` is only a phone-bezel mockup wrapper for presentation — do not port it; the real app has no iOS chrome.

## Fidelity
**High-fidelity.** Colors, type, spacing, and copy below are final. Recreate pixel-accurately using the target stack's styling approach (e.g. Tailwind classes matching the tokens below), not by embedding this HTML.

## Design System
Base system: **Modernist** (flat, architectural, zero corner radius, strong 2px rules, Archivo typeface, flush-left labels, no centered button text). This app **overrides Modernist's accent color**: primary is `#ffd6a8` (peach) instead of Modernist's default red, paired with a dark warm-ink contrast color. All structural rules (0 radius, 2px dividers, flush-left, Archivo, no decoration) still apply.

## Design Tokens

| Token | Value | Use |
|---|---|---|
| `--peach` (primary) | `#ffd6a8` | Brand fills: primary CTA banners, active tab background, selected chips, status pill "in review" fill, reference-code badge |
| `--ink` (contrast) | `#1c1a17` | Primary text, primary button fill (with peach text on it), icons, dividers at high opacity, active states |
| `--ink-70` | `rgba(28,26,23,.7)` | Secondary text |
| `--ink-55` | `rgba(28,26,23,.55)` | Meta text (dates, captions) |
| `--ink-45` | `rgba(28,26,23,.45)` | Placeholder text, disabled/pending steps |
| `--ink-15` | `rgba(28,26,23,.15)` | Dividers, borders |
| `--bg` (paper) | `#fdfbf8` | Screen background |
| `--surface` | `#f4efe7` | Input fields, photo placeholder fill |
| `--amber` (link) | `#a3591f` | Links, "Edit" affordances (default `a` color) |
| `--amber-hover` | `#7a4013` | Link hover |
| `--red-muted` | `#b3402a` | Rejected status only (semantic exception to the mono palette) |
| Font | `Archivo` (400 body / 800 headings), via Google Fonts | All text |
| Radius | `0px` everywhere | No rounded corners on any container, button, input, or chip |
| Divider weight | `2px solid` | All rules/borders (never 1px hairlines) |
| Button height | `52px` | Primary/secondary full-width buttons |
| Input height | `44–46px` | Text fields |

## Screens / Views

### 1. Sign In
- **Purpose:** Employee authentication; toggle to self-registration.
- **Layout:** Full-height column, `paddingTop:60px` (safe area). Header "Sign in" (26px/800). Below it a 2-tab segmented control (`LOGIN` / `REGISTER`, 2px ink border, active tab ink-filled with peach text). Form area: Employee ID field, Password field (masked dots), right-aligned "Forgot password?" link. Bottom-pinned full-width primary button "SIGN IN" (ink fill, peach text) + centered "New here? Register as employee" link.
- **Fields:** `.input`-style: 44–46px height, `--surface` fill, 2px `--ink-15` border, label above in 11px uppercase `--ink-70`.
- **Register tab (not fully mocked):** empid, name, mobile, department, plant, password — per build-spec §2 (self-register, no HR validation).

### 2. Home / Dashboard
- **Purpose:** Landing screen after login; quick stats, primary action, recent activity.
- **Layout:** Header block: dept/plant line (12px `--ink-55`) + "Hi, {name}" (24px/800), bottom 2px divider.
- **Stat row:** 4 equal columns (Submitted / In review / Approved / Rejected), each with big number (26px/800) + uppercase label (11px `--ink-55`), separated by 2px vertical dividers.
- **Primary CTA banner:** full-bleed-within-margin peach block, "New Suggestion" (17px/800) + subtext, trailing arrow icon (Lucide `arrow-right`). Tap → Submit flow screen 1.
- **Recent list:** section label "RECENT" (13px/800 uppercase), rows of suggestion title + reference code/date (11px `--ink-55`), each with a status pill (top-right aligned): peach fill = in-progress/review states, ink fill = implementing, outlined red = rejected.
- **Bottom tab bar:** 3 tabs (Home / New / Mine), 2px top border, active tab has a peach-tint (`#fff2e2`) background block behind icon+label; icons from Lucide (`home`, `plus`, `file`).

### 3. Submit Suggestion — Step 1 (Issue)
- **Purpose:** Capture the problem: location, issue description, evidence photos.
- **Layout:** Header: back chevron + "STEP 1 OF 3" (right-aligned, 11px/800 uppercase), title "What's the issue?" (22px/800), then a **language selector**: label "SELECT YOUR LANGUAGE" (11px uppercase `--ink-70`) above a horizontally scrollable row of language chips (English, தமிழ், हिन्दी, मराठी, ਪੰਜਾਬੀ) — selected chip ink-filled/peach text, others outlined 2px `--ink-25`. This selection scopes the whole submission flow's speech input + rendered labels (per build-spec §6).
- **Progress bar:** 3 equal segments, filled = `--ink`, unfilled = `--ink-15`; step 1 → first segment filled.
- **Form:** "Line / Area" input, "Exact spot (optional)" input.
- **Issue field:** label row with a `TYPE` / `SPEAK` 2-way toggle (2px ink border pill group; active = ink fill + peach text) — SPEAK mode should invoke the Web Speech API per build-spec §6 (locale = selected language). Textarea below (110px), live word counter bottom-right "`n / 250 words`" (client + server validated on original-language text per spec).
- **Photo evidence:** row of 74×74px dashed-border (`2px dashed --ink-35`) upload tiles; first shows a camera icon, subsequent tiles show `+`.
- **Footer:** full-width "NEXT" primary button.

### 4. Submit Suggestion — Step 2 (Idea)
- **Purpose:** Optional improvement idea + solution sketch.
- Same header pattern, "STEP 2 OF 3", progress bar first 2 segments filled.
- "Your idea (optional)" field with the same TYPE/SPEAK toggle and textarea.
- "Solution sketch (optional)" — same dashed upload tile pattern.
- **Footer:** two buttons side by side — "BACK" (2px ink outline, ink text, transparent) and "NEXT" (ink fill, peach text), each `flex:1`.

### 5. Review & Submit — Step 3
- **Purpose:** Read-only summary before final submit, with per-section edit links.
- Progress bar fully filled (3/3).
- Sections: Location, Issue (+ photo thumbnails, 56×56px `--surface` tiles), Idea — each with an uppercase section label and a right-aligned "Edit" link (amber, underlined) that returns to the relevant step.
- **Footer:** full-width "SUBMIT SUGGESTION" primary button.

### 6. Confirmation
- **Purpose:** Success state, ties back into tracking.
- 56×56px ink square with a peach checkmark icon (Lucide `check`).
- "Submitted" (26px/800), body copy ("Your suggestion has been sent to your Department Head for review."), then the reference code as a peach-filled badge, e.g. `KIS-PLT03-000512` (per build-spec §3 `reference_code` format).
- **Footer:** "VIEW IN MY SUGGESTIONS" primary button + "Submit another suggestion" link.

### 7. My Suggestions
- **Purpose:** Employee's own suggestion list with status filtering (build-spec §10: "status timeline with timestamps").
- Header "My Suggestions" + horizontally scrollable filter chips: ALL (active, ink-filled) / IN REVIEW / APPROVED / REJECTED (outlined).
- List rows: reference code (11px monospace `--ink-50`), title (14px/600), date + status pill per row (2px divider between rows). Status vocabulary matches the state machine in build-spec §4: `DH REVIEW`, `PH REVIEW`, `IMPLEMENTING`, `REJECTED` (outlined red), `CLOSED` (outlined ink).
- Bottom tab bar, "Mine" tab active.

### 8. Suggestion Detail
- **Purpose:** Full status timeline + submitted content for one suggestion, mirrors the approval chain in build-spec §4.
- Header: back chevron, reference code (monospace), title, line/area + submitted date.
- **Vertical stepper** (2px `--ink-15` connecting line, left-aligned): Submitted → Dept Head review → Plant Head review → Implementation (KPO) → Cost monitoring (Finance) → Closed. Each step: 12×12px square marker (ink-filled = done, peach-filled with ink border = current/in-progress, outlined `--ink-30` = pending), bold 13px label, 11px meta line (decision, priority, timestamp). The in-progress KPO step nests its implementation stages (e.g. "Procurement") as sub-entries.
- **"Your submission" card:** section label + an EN/ORIGINAL 2-way toggle (ink border pill, matches build-spec §6's translation toggle), issue text (13px), photo thumbnails (52×52px).
- If a suggestion is rejected, this is where DH/PH remarks should render as a callout (not shown in current mock — add a bordered quote block using `--ink-15` border + italic text, sourced from `SuggestionReview.remarks`).

## Interactions & Behavior (to implement)
- **Language chips** (screen 3): tapping sets the active language for the whole submission flow — persists across steps 1–3, drives Web Speech API locale and field labels.
- **TYPE/SPEAK toggle:** switching to SPEAK starts/stops Web Speech API capture into the same textarea; show a recording/waveform state while listening (not mocked — needs a real state, e.g. pulsing mic icon).
- **Word counter:** live-updates on input; disable "Next" or show inline error past 250 words (client-side; server re-validates per spec).
- **Photo upload tiles:** tapping opens camera/file picker; filled tiles should show a thumbnail + remove (×) affordance.
- **Step navigation:** Back/Next preserve entered data (draft state) when moving between steps 1–3.
- **Submit:** on tap, POST `/suggestions` (per build-spec §9), transition to Confirmation with the returned `reference_code`.
- **Status pills / stepper markers:** color and label driven by `Suggestion.status` (state machine, build-spec §4) — do not hardcode per screen.
- **Filter chips (My Suggestions):** filter the list client-side or via query param against `GET /suggestions/mine`.

## State Management
- **Submission draft:** language, line/area, exact spot, issue text (+ language + word count), photos[], idea text, solution sketches[] — held across steps 1–3, cleared on submit or discard.
- **Auth state:** JWT access/refresh (per build-spec §8/§9), `must_reset_password` flag should redirect to a forced reset flow (not mocked — add per build-spec §2).
- **Suggestion list/detail:** fetched from API, not client-derived; status/timeline rendering is a pure function of the `Suggestion` + `SuggestionReview` + `ImplementationStage` records (build-spec §3).

## Assets
No external image assets — all icons are hand-drawn inline SVGs approximating **Lucide** icons (`chevron-left`, `mic`, `camera`, `check`, `arrow-right`, `plus`, `home`, `file`). Recreate using the actual Lucide icon set/package in the target codebase rather than these inline SVGs.

## Files
- `kaizen-employee-app.html` — the full design source (all 8 screens, self-contained HTML/React design component). View source for exact markup/inline styles per screen.
- `ios-frame.jsx` — phone bezel wrapper used only for presentation in the design tool; not part of the app itself.
