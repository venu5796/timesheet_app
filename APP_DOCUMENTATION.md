# Acquia Program Control Tower — Technical Documentation

> Version: 2.0 | Source: `index.html` | Format: Single-page HTML application

---

## 1. Overview

The **Acquia Program Control Tower** is a high-fidelity, interactive single-page dashboard designed for senior project managers and stakeholders. It provides real-time visibility into project finances, resource utilisation, and Jira-based development velocity across four dedicated tabs. By synthesising data from a Google Apps Script backend, it offers actionable insights through RAG (Red-Amber-Green) health indicators and predictive analytics.

---

## 2. Key Features by Tab

### 2.1 Program Dashboard

- **7 KPI Cards:** Burned (€), Remaining Budget, Forecast (EAC), Velocity (h/wk), Non-Billable %, Billable Efficiency, EAC Confidence
- **RAG Health Banner:** Instant Red/Amber/Green status for Budget, Schedule, Resources, and Delivery — each computed from thresholds in the data
- **Variance Flags panel:** resources >20% over or under their planned hours for a selected week
- **Delinquency & Missing panel:** resources scheduled but reporting 0 hours for the selected week
- **Cumulative Burn vs. Budget Cap** chart (line chart with budget cap reference line)
- **Non-Billable vs. Billable Time** chart (stacked bar)
- **Planned vs. Actual Hours S-Curve** (line chart)
- **Spend by Phase — Actual vs Planned** bar chart
- **Phase Cost Breakdown** table
- **Top 5 Spenders** (selectable by week)
- **Milestone Tracker** table (status badges: On Track, At Risk, Delayed, Complete)
- **Resource Utilisation Heatmap:** colour-coded per resource per week — Over >40h (red), Optimal 30–40h (green), Low 1–29h (amber), No entry (grey)
- **Weekly Performance Metrics** table: per-resource Planned, Actual, Variance, Spend for a selected week
- **Predictive Roll-off Schedule:** lists each resource with their projected end date and status indicator

### 2.2 Jira — High Level

- **Phase metric cards:** ticket counts for Qualification, Development, UAT, Done
- **Pipeline Flow chart:** horizontal bar showing issue distribution across phases
- **Sprint Completion Rate chart:** historical % of committed points delivered per sprint
- **Issues per Sprint Volume Trend** chart: total issue volume over time
- **Hour-Budget Breaches table:** tickets exceeding story-point SLA from the `Spend_Analysis` sheet — columns: Ticket, Pts, Hours Spent, Warn threshold, Over threshold, Summary, Assignee, Jira Status, SLA Status
- **All Issues table:** filterable by phase and sprint — columns: Key, Summary, Assignee, Status, Phase, Sprint, Points

### 2.3 Jira — Sprint View

- **Sprint selector** dropdown with date range display
- **Sprint Overview Bar:** Total, Done, Development, UAT, Qualification, Flagged, Days Left
- **Story Point Progress Bar:** done/total pts with colour-coded fill (green ≥80%, amber ≥60%, red <60%)
- **Burndown Chart:** Ideal vs Actual points remaining over sprint days
- **Status Breakdown:** per-status bar list with counts for all Jira workflow statuses
- **Label Distribution** doughnut: Blocked, Needs Client Input, Dependency, Needs Review, No Label
- **Assignee Workload** horizontal bar: story points per team member
- **Aging Issues table:** development tickets with no update >16 working hours (Mon–Fri, 9 am–6 pm, holidays excluded)
- **Flagged / Blocked Issues table:** tickets bearing labels `blocked`, `needs-client-input`, `dependency`, `needs-review`
- **Ready for Review Aging panel:** tickets in `READY FOR REVIEW` status beyond the configured threshold
- **Code Review Aging panel:** tickets in `CODE REVIEW` status beyond the configured threshold
- **Dev SLA Breaches panel:** tickets `IN PROGRESS` beyond their story-point working-hour SLA
- **Full Sprint Issues table:** all tickets sorted by workflow position

### 2.4 Capacity & Planning

- **Sprint Capacity chart:** available planned hours per sprint (from hard-coded `CAP_DATA` / ResourcePlan)
- **Story Points Capacity vs Committed vs Done** chart: three-bar comparison with configurable hours-per-point ratio
- **Team Availability breakdown:** per-person hours, role tag, bar indicator, story-point estimate; user-editable override input per person
- **Sprint Planner:** click-to-toggle interface moving issues between Backlog and Planned for Sprint, with a live capacity bar

---

## 3. Technical Architecture

### 3.1 Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, Vanilla CSS3 (custom CSS variables / design tokens), JavaScript ES6+ |
| Charts | Chart.js with `chartjs-plugin-datalabels` |
| PDF Export | `html2pdf.js` (A3 landscape) |
| Fonts | `DM Sans` (UI), `DM Mono` (financial/numeric data) |
| Data Source | Google Apps Script Web App returning JSON via `fetch()` |
| Persistence | `localStorage` (dark mode preference only) |

### 3.2 Data Fields Consumed from Google Apps Script

| JSON key | Description |
|---|---|
| `actuals` | Array of time-entry rows (resource, date, hours, task, billability, cost) |
| `plan` | Array of planned-hour rows (resource, phase, week, planned hours) |
| `rawPhases` | Ordered array of phase name strings from the Phases tab |
| `milestones` | Milestone rows with name, due date, status, and notes |
| `unknownTasks` | Tasks that could not be mapped to a known phase |
| `jiraSummary` | Key-value metric rows including SLA thresholds and sprint config |
| `jiraIssues` | Array of Jira issue objects: `key`, `summary`, `status`, `assignee`, `sprint`, `story_points`, `labels`, `updated`, `total_hours` |
| `jiraConfig` | Object with `sprint_start`, `sprint_end`, `sprint_days`, `ready_for_review_max_hrs`, `code_review_max_hrs` |
| `spendAnalysis` | Array from Spend_Analysis sheet: Ticket, Story Points, Total Hours, Warn/Over thresholds, Status |

### 3.3 Data Flow

1. On page load, `fetchData()` calls the Google Apps Script URL via `fetch()`.
2. The JSON response is parsed and stored in the `globalData` object.
3. `processData()` builds week/person aggregates from `actuals` and `plan`, computes cumulative burn, EAC, variances, and RAG thresholds.
4. Phase matching uses a multi-stage algorithm: exact match → starts-with prefix → phase-name-starts-with-attempt. Intentionally does **not** use `includes()` to avoid false positives (e.g. `"Pre Kick Off"` matching `"Kick Off"`).
5. `renderJiraTab()` populates all Jira and Capacity sub-sections from `jiraIssues` and `jiraSummary`.
6. All Chart.js instances are stored in `chartRefs` / `jiraChartRefs` / `sparkRefs` for proper destroy-before-recreate on re-renders.
7. `silentFetch()` runs every 15 minutes in the background without displaying a loader or hiding the dashboard.

### 3.4 Working Hours Engine

All elapsed-time calculations (Aging Issues, Dev SLA Breaches) use `workingHoursElapsed_()`, which:

- Counts only Monday–Friday hours between **09:00 and 18:00** (9 hours/day)
- Skips all dates listed in `WORK_HOURS.holidays` (India public holidays for 2026)
- Returns a decimal number of working hours — displayed as working days (`wd`) if ≥9 hours or working hours (`wh`) if less

> The holiday list covers India 2026 only and requires a yearly update.

### 3.5 Dev SLA Thresholds

All thresholds are in **working hours**. Configured in the `DEV_SLA` constant; can also be overridden via `jiraSummary` metric rows.

| Story Points | Warn (working hrs) | Over (working hrs) | Equivalent |
|---|---|---|---|
| 1 pt | 2 h | 2 h | ~¼ working day |
| 2 pt | 4 h | 4 h | ~½ working day |
| 3 pt | 8 h | 12 h | 1 day warn / 1.5 days over |
| 5 pt | 16 h | 24 h | 2 days warn / 3 days over |
| 8 pt | 24 h | 24 h | >3 working days |
| Default | 16 h | 24 h | 2 days warn / 3 days over |

**Grace window:** 1 working hour after the Over threshold before a ticket is escalated to `Over (within grace)` status (`SLA_GRACE_HOURS = 1`).

### 3.6 Capacity Data (`CAP_DATA`)

Sprint capacity is hard-coded in the `CAP_DATA` constant, mapping sprint phase names to per-person hour allocations. The `CAP_SPRINT_MAP` object translates Jira sprint labels (e.g. `"VCOM Sprint 1"`) to ResourcePlan phase labels (e.g. `"Dev Sprint 1"`). Currently covers **Dev Sprint 1–5**. Sprint 4 includes an additional Tech Arch (Jayakrishnan Jayabal, 40 h).

User-entered overrides are stored in the in-memory `_capOverrides` object (key: `"${sprint}__${person}"`) and reset on page refresh.

---

## 4. UI & Design System

### 4.1 Design Tokens

Defined as CSS custom properties on `:root` (light) and `[data-theme="dark"]` (dark):

| Token | Light value | Purpose |
|---|---|---|
| `--primary` | `#003d82` | Acquia brand blue — card titles, KPI values |
| `--primary-light` | `#0057b8` | Header gradient end |
| `--accent` | `#3b82f6` | Interactive highlights, Jira ticket keys |
| `--success` | `#10b981` | Green RAG, Done status, optimal utilisation |
| `--danger` | `#ef4444` | Red RAG, over-utilisation, SLA breach |
| `--warning` | `#f59e0b` | Amber RAG, low utilisation, at-risk milestones |
| `--bg` | `#eef2f7` | Page background |
| `--surface` | `#ffffff` | Card background |
| `--surface-2` | `#f8fafc` | Alternate row / input background |
| `--border` | `#dde3ed` | Card and table borders |
| `--text` | `#1a2535` | Primary body text |
| `--muted` | `#64748b` | Secondary text, labels, metadata |

### 4.2 Responsive Breakpoints

| Max-width | Layout change |
|---|---|
| 1200px | KPI row → 4 columns; main, equal, and alert grids → single column; header stacks vertically |
| 900px | Jira two-column and capacity grids → single column |
| 768px | KPI row → 2 columns; alert grid → single column |
| 480px | KPI row → 1 column; buttons go full-width |

### 4.3 Dark Mode

Activated by the 🌙 toggle in the header. The toggle:

- Sets `data-theme="dark"` on the `<html>` element, which activates the dark token overrides
- Persists the preference in `localStorage` under key `pct_dark_mode`
- Respects `prefers-color-scheme: dark` if no saved preference exists
- Re-renders all Chart.js instances on toggle to apply updated tick and grid colours

### 4.4 Animations

- `fadeUp` — cards and KPI tiles animate in from below on load (0.4–0.5 s ease)
- `bgPulse` — the background sync dot pulses during a silent background fetch
- `spin` — spinner during initial load and manual sync

---

## 5. Configuration & Thresholds

| Constant | Default | Purpose |
|---|---|---|
| `THRESH_OVER` | `40` h | Heatmap: over-utilisation threshold |
| `THRESH_GOOD` | `30` h | Heatmap: lower bound of optimal range |
| `RATE_FALLBACK` | `€100/h` | Fallback hourly rate when none is specified |
| `VARIANCE_FACTOR` | `1.2` | EAC upper-bound multiplier (120% of plan) |
| `VARIANCE_UNDER` | `0.8` | EAC lower-bound multiplier (80% of plan) |
| `ACTIVE_DAYS` | `14` | Days a resource is considered recently active |
| `SLA_GRACE_HOURS` | `1` h | Grace window after SLA Over threshold |
| `AGING_THRESHOLD_WORK_HRS` | `16` h | Working hours before a dev ticket is flagged as aging |
| Silent fetch interval | `15 min` | Background refresh cadence (`setInterval(silentFetch, 15 * 60 * 1000)`) |
| `budgetLimit` default | `€251,392` | Pre-populated remaining budget cap in the header input |

---

## 6. PDF Export

Clicking **Export PDF** adds the class `pdf-export-mode` to `<body>`, which:

- Hides header controls, week selectors, and all `.no-print` elements
- Removes `overflow: hidden` and `max-height` constraints from all cards, panels, and table wrappers so charts and tables render in full
- Disables sticky positioning on the heatmap's first column (required for canvas capture)
- Sets `box-shadow: none` and `animation: none` for clean print output
- Forces the KPI row to 4 columns and main/equal/alert grids to 2 columns for A3 landscape layout

`html2pdf.js` then captures the full DOM at **A3 landscape** dimensions.

---

## 7. Sync Behaviour

| Trigger | Function | Behaviour |
|---|---|---|
| Page load | `fetchData()` | Shows spinner, hides dashboard, full reload |
| Manual "Sync Data" button | `fetchData()` | Same as page load |
| Background timer (every 15 min) | `silentFetch()` | No spinner, no dashboard hide; shows only a brief pulse dot and updates the sync timestamp |
| Budget cap input change | `debouncedProcess()` → `processData()` | Re-renders charts and KPIs only (no network call); debounced at 350 ms |

On successful sync, `_lastSyncedAt` is updated and a 15-minute countdown is displayed under the header.

---

## 8. Error Handling

- If `fetchData()` receives a non-200 HTTP response or a JSON `{ error: "..." }` payload, the loader is hidden and an `error-box` div is rendered with the error message and a **Retry** button.
- `silentFetch()` failures are caught silently (`console.warn` only) and never disrupt the visible dashboard.
- All date parsing is handled by `parseDateSafe()`, which supports four formats: `YYYY-MM-DD`, `DD-Mon-YY`, `DD-Mon-YYYY`, and `Mon-DD`, with a browser `new Date()` fallback.

---

## 9. Known Limitations

- `CAP_DATA` is hard-coded in JavaScript; capacity changes require an `index.html` edit.
- Sprint planner state (`_plannerState`) is in-memory only; changes are lost on page refresh.
- Capacity hour overrides (`_capOverrides`) are also in-memory only.
- The holiday list covers India 2026 only and requires a yearly update in the `WORK_HOURS.holidays` Set.
- Phase matching does not use `includes()` to avoid substring false positives.
- The `API_URL` constant is hard-coded to a specific Google Apps Script deployment URL; changing the backend requires an `index.html` edit.
