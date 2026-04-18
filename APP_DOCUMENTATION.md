# Acquia Program Control Tower — Technical Documentation

> Version: 3.3 | Source: `index.html` | Format: Single-page HTML application

---

## 1. Overview

The **Acquia Program Control Tower** is a high-fidelity, interactive single-page dashboard designed for senior project managers and stakeholders. It provides real-time visibility into project finances, resource utilisation, and Jira-based development velocity across four dedicated tabs. Version 3.3 refines KPI logic, consolidates phase tables, updates Jira status taxonomy to a 7-stage workflow, and introduces allocated-vs-spend heatmap semantics with a UAT pipeline burndown layer.

---

## 2. Key Features & Layout

### 2.1 Persistent Sidebar (Global Overview)
The sidebar remains visible across all tabs, providing a constant "cockpit" view of the most critical health metrics:
- **Logo & Brand Area:** Program identity.
- **Critical Health (RAG):** Dynamic status dots for Budget, Schedule, Resources, and Delivery.
- **Overall Risk Badge:** Automated assessment (Low/Moderate/Critical) based on budget burn, EAC forecast, and planning depth.
- **Core Financials:** Real-time Burned (€), Remaining, and Forecast (EAC) values.
- **Budget Runway:** Dynamic calculation of remaining weeks based on recent 4-week average burn rate.
- **Projected Finish:** Date estimate based on the furthest scheduled task in the `plan` dataset.
- **Budget Cap Control:** Inline editable input to adjust the project's financial ceiling (debounced at 350 ms).

### 2.2 Program Dashboard (Main Tab)
- **Smart Insight Card:** AI-style human-readable summary of project health (e.g., "Budget Warning: EAC exceeds cap by €12k").
- **KPI Strip:**
    - **Velocity (Alloc vs Spend):** Average actual hours/week with a trend badge — Down (spend < 80% of leave-adjusted allocated), Optimum (80–110%), or Up/Over (> 110%) — compared week-on-week across the last 4 completed weeks.
    - **Non-Billable %:** Non-billable share of total logged hours with trend badge.
    - **Budget Burn Rate:** Average weekly spend (€/wk) over the last 4 weeks, compared against the weekly budget allocation. Trend: Accelerating / On Budget / Decelerating.
    - **EAC Confidence:** Gauge of data reliability based on the volume of future-planned weeks (No Data / Low / Medium / High).
- **Variance Flags:** List of resources whose billable hours deviate >20% above or below plan for the most recent completed week.
- **Charts:**
    - Cumulative Burn vs. Budget Cap (line) — data points annotated only for the current and previous week.
    - Non-Billable vs. Billable Time — stacked bar per week.
    - Planned vs. Actual Hours S-Curve — cumulative line.
    - Phase Spend — Actual vs. Planned grouped bar per phase.
    - Timesheet Compliance Trend — colour-coded bar chart (green ≥ 80%, amber ≥ 50%, red < 50%) showing % of planned resources who submitted per week.
    - Resource Load by Name — horizontal bar chart of total hours per resource (last 6 weeks).
- **Phase Performance & Cost Breakdown Table (merged):** Single table combining Actual (€), Planned (€), Delta (€), % of Planned, Phase EAC (€), Completion progress bar, and On Track / Near Cap / Overrun status per phase.
- **Top 5 Spenders:** Week-selectable feed of highest-cost resources.
- **Predictive Roll-off:** Feed of the 5 resources whose planned engagement ends soonest.
- **Milestone Tracker:** Table of milestones extracted from the ResourcePlan header row, showing due date and strict three-state status — **Done** (due date past), **In Progress** (due within 30 days), or **Upcoming** (due > 30 days away).
- **Resource Heatmap:**
    - **Search Filter:** Real-time resource lookup.
    - **Allocated vs Spend:** Each cell shows `spend/alloc` hours. Color-coded: Red (spend > allocated by > 10%), Green (within ±10% of allocated), Amber (spend < 80% of allocated). Tooltip shows raw values.

### 2.3 Jira — High Level Tab
- **Phase KPI Cards:** Ticket counts for Ready for Dev, In Progress, Review/QA (combined), Done, and Total.
- **Pipeline Flow Bar:** Numeric summary across Qual (Ready for Dev), Dev (In Progress), UAT (QA + Ready for QA), and Done.
- **Velocity Trends Chart:** Story points completed per calendar month (bar).
- **Category Distribution Chart:** Doughnut breakdown across all 8 pipeline stages (Ready for Dev, In Progress, Ready for Review, Code Review, Ready for QA, QA, Re-opened, Done).
- **Jira Spend Analysis Chart:** Story points per assignee, horizontal bar (top 10 by points).

### 2.4 Jira — Sprint View Tab
- **Sprint Selector:** Dropdown populated from all unique sprint names in `jiraIssues`; defaults to the **current sprint** (sprint whose `sprint_dates` range contains today), falling back to the last sprint alphabetically.
- **Sprint Date Label:** Displays `sprintStart → sprintEnd` resolved from `jiraConfig.sprint_dates` for the selected sprint.
- **Sprint Stats Bar:** Total, Done, Ready for Dev, In Progress, Ready for Review, Code Review, Ready for QA, QA, Re-opened, Scope Creep, and Blocked ticket counts.
- **Points Badge:** Points done vs. total points for the selected sprint.
- **Sprint Burndown Chart:** Day-by-day ideal vs. actual remaining points using `status_category_changed` timestamps, plus a third **In UAT Queue** dataset (cumulative points entering Ready for QA / QA status). Falls back to a Done vs. Remaining bar when sprint dates are absent.
- **Risk Labels Distribution:** Doughnut chart sourced from `jiraSummary` sheet label columns (top 8 labels), with fallback to sprint issue labels if the summary contains no label data.
- **Ready for Review Aging Table:** Issues in "READY FOR REVIEW" status ranked by working hours elapsed since `status_category_changed`, flagged against `AGING_THRESHOLD_WORK_HRS`.
- **Code Review Aging Table:** Issues in "CODE REVIEW" or "IN REVIEW" status, same aging logic.
- **Dev SLA Breaches Table:** Issues in Development, Ready for Review, Code Review, Ready for QA, or UAT whose elapsed working hours exceed the story-point-banded SLA threshold (with a 1-hour grace window).
- **Sprint Issue Detail Table:** Full issue list with Key, Status badge, Summary, Assignee, and Story Points.

### 2.5 Capacity & Planning Tab (Hidden by Default)
- **Velocity Simulation (What-If?):** Interactive slider (10–100 h/wk) showing Simulated EAC and Buffer against the budget cap.
- **Sprint Capacity Overview:** Table of resources with Available h, Planned h, and Load %.

---

## 3. Technical Architecture

### 3.1 Tab Engine & Navigation
- **`switchTab(name)`:** Toggles `.active` class on `.tab-panel` elements and nav buttons; triggers the appropriate render function (`processData`, `jRenderHighLevel`, `jRenderSprintView`) on activation.
- **CSS-Driven Visibility:** Optimized `.tab-panel` transitions with a `fadeIn` animation (0.2 s).
- **Persistent States:** Dark mode preference is persisted via `localStorage` (`pct_dark_mode`); system `prefers-color-scheme` is respected as a fallback.

### 3.2 Design System (SaaS Aesthetic)
- **Token System:** CSS custom properties under `:root` for all colors, radii, and fonts. A `[data-theme="dark"]` selector overrides surface and text tokens.
- **Legacy Variable Mapping:** A second `:root` block maps legacy variable names (e.g., `--primary`, `--surface`, `--bg`) to the new token system for JavaScript compatibility.
- **Typography:** DM Sans (400–800) for UI; DM Mono (400–500) for numeric values.
- **Night Mode (Dark Mode):**
    - Fully semantic variable mapping for surfaces, text, borders, and status colors.
    - All Chart.js instances are re-rendered on theme toggle to update gridlines, tick, and legend label colors.

### 3.3 Data Normalization & Processing
- **Sprint Resolution (`resolveLatestSprint`):** When a Jira issue carries a semicolon-delimited list of sprint names, the function extracts the highest-numbered sprint so each issue belongs to exactly one sprint.
- **Sprint Date Resolution:** Sprint start/end dates and working-day count are built into `jiraConfig.sprint_dates` (keyed by sprint name) during the Google Apps Script `doGet` call.
- **Monday-Snapping (`getMonday`):** All weekly aggregations snap to Monday using local-timezone arithmetic.
- **Robust Date Parsing (`parseDateSafe`):** Handles ISO (`YYYY-MM-DD`), `DD-Mon-YY`, `DD-Mon-YYYY`, and `Mon-DD` formats with India-timezone safety via `toLocalISO`.
- **Prorata Basis:** The current week's planned hours are scaled by `elapsedWorkDays / 5` before being added to the plan aggregation, ensuring Actuals are compared fairly against an incomplete plan target.
- **Phase Matching:** Task/Deliverable strings from `TimeEntries` are fuzzy-matched against known phase names (prefix and containment checks) with a `taskPhaseCache` for performance.
- **Velocity Status Logic:** Computed in `updateUI` by comparing actual spend hours vs. leave-adjusted allocated hours (from `plan`) over the last 4 completed weeks. Ratio < 0.8 → Down; 0.8–1.1 → Optimum; > 1.1 → Up/Over.
- **Heatmap Alloc Lookup:** `renderHeatmap` builds a per-person per-week plan map from `globalData.plan` at render time to derive allocated hours for each cell comparison.
- **Merged Phase Table (`renderPhaseMerged`):** Replaces the former `renderPhaseEAC` and `renderPhaseBreakdown` functions; renders a single consolidated table with Actual, Planned, Delta, % of Planned, Phase EAC, Completion bar, and Status.

### 3.4 Jira Status Taxonomy (`jStatusCategory`)
Maps raw Jira status strings to internal categories used across all sprint views, KPI counts, burndown, and SLA checks:

| Raw Status | Internal Category |
|---|---|
| Ready for Dev, New, To Do | `readydev` |
| In Progress | `development` |
| Ready for Review | `readyreview` |
| Code Review, In Review | `codereview` |
| Ready for QA | `readyqa` |
| QA, UAT | `uat` |
| Re-opened_ch, Re-opened, Reopened | `reopened` |
| Done | `done` |

### 3.5 Working Hours Engine (`workingHoursElapsed_`)
Calculates elapsed time between two `Date` objects, excluding:
- Weekends (Saturday / Sunday).
- Hours outside 09:00–18:00.
- **Public Holidays:** Defined in `WORK_HOURS.holidays` (India 2026, updated to 2027-01-01 boundary).
- **Personal Leave Days:** If a `person` argument is supplied and `globalData._processedLeaves` contains leave dates for that person, those dates are also excluded. Leave data is sourced from the `Teamleaves` / `Team Leaves` sheet.

### 3.6 Background Sync
- **Auto-refresh:** A 15-minute countdown (`SYNC_INTERVAL_MS`) is tracked client-side; the header shows the last successful sync time and a pulsing dot during active fetches.
- **Error Resilience:** Individual chart failures or missing fields do not crash the dashboard; fallback logic provides `—` or `0` values. A debug panel appears if the API returns an error object.

---

## 4. Configuration & Thresholds (`CONFIG`)

| Constant | Default | Purpose |
|---|---|---|
| `THRESH_OVER` | `40` h | Legacy heatmap threshold (superseded by alloc vs spend logic) |
| `THRESH_GOOD` | `30` h | Legacy heatmap threshold (superseded by alloc vs spend logic) |
| `RATE_FALLBACK` | `100` | €/h used for simulations when no rate is found |
| `VARIANCE_FACTOR` | `1.2` | Variance Flag trigger (20% over plan) |
| `VARIANCE_UNDER` | `0.8` | Variance Flag lower trigger (20% under plan) |
| `ACTIVE_DAYS` | `14` | Window for considering a resource "Active" |
| `SYNC_INTERVAL_MS` | `15 min` | Background data refresh cadence |
| `AGING_THRESHOLD_WORK_HRS` | `16` | Working-hour threshold for Ready-for-Review / Code Review aging badges |
| `SLA_GRACE_HOURS` | `1` h | Grace window before a Dev SLA breach is flagged |

### Dev SLA Thresholds (`DEV_SLA`)
Applied to issues in Development, Ready for Review, Code Review, Ready for QA, and UAT stages.

| Story Points | Warn (h) | Over (h) |
|---|---|---|
| 1 | 2 | 2 |
| 2 | 4 | 4 |
| 3 | 8 | 12 |
| 5 | 16 | 24 |
| 8 | 24 | 24 |
| default | 16 | 24 |

---

## 5. Data Sources (Google Apps Script — `doGet`)

| Dataset | Sheet | Key Fields Served |
|---|---|---|
| `actuals` | `TimeEntries` | Date, Person, Role, Task/Deliverable, Time in Hours, Subtotal, Rate |
| `plan` | `ResourcePlan` | role, person, plannedRate, dateStr, planH, phase |
| `rawPhases` / `rawDates` | `Phases` | Phase name order, week-to-phase mapping |
| `milestones` | `ResourcePlan` (row 2 headers) | name, due |
| `jiraSummary` | `Jira_Summary` | All columns (lowercased headers); label columns used as primary source for Risk Labels chart |
| `jiraIssues` | `Jira_CT` | key, summary, status, assignee, story_points, sprint, labels, status_category_changed, sprint.startdate, sprint.enddate |
| `jiraConfig` | Derived from `Jira_CT` | sprint_name, sprint_start, sprint_end, sprint_days, sprint_dates (map), statuses |
| `spendAnalysis` | `Spend_Analysis` | All columns |
| `leaves` | `Teamleaves` / `Team Leaves` | Resource → [leave date, …] |
| `unknownTasks` | Derived | Tasks in TimeEntries not matched to any phase |

---

## 6. PDF Export
- **A3 Landscape:** Optimized layout for executive status reports via `html2pdf.js`.
- **Print Mode:** CSS `@media print` hides `.no-print` elements (sidebar, nav, controls) to maximize data visibility.
- **Canvas Snapshots:** `html2pdf` converts Chart.js canvases to static images during export for cross-browser compatibility.