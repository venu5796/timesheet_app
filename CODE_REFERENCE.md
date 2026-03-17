# Code Reference

Technical reference for `index.html` and `Code.gs`. Use this alongside the README when making changes independently.

---

## Code.gs — function reference

### `doGet(e)`
The entry point. Called every time the Web App URL is fetched.

**Returns JSON with these keys:**
```json
{
  "actuals":      [],   // TimeEntries rows
  "plan":         [],   // ResourcePlan rows (flattened)
  "rawPhases":    [],   // Phase names in order
  "rawDates":     [],   // ISO week dates from ResourcePlan
  "milestones":   [],   // Auto-generated from ResourcePlan phase headers
  "unknownTasks": [],   // TimeEntries tasks that didn't match any phase
  "jiraSummary":  [],   // Jira_Summary rows {metric, value, rag}
  "jiraIssues":   [],   // Jira_CT rows — one object per issue
  "jiraConfig":   {}    // Derived sprint config {sprint_name, sprint_start, sprint_end, sprint_days, statuses[]}
}
```

**Append `?debug=1` to trigger `debugInfo()` instead** — returns raw cell values for date debugging.

---

### `normaliseDate(val)`
Converts any date format GAS produces to `YYYY-MM-DD` string.

Handles:
- GAS `Date` object
- GAS internal string: `"Mon Feb 23 2026 00:00:00 GMT+0530 (...)"`
- ISO with time: `"2026-03-11T18:30:00.000Z"`
- ISO date only: `"2026-03-11"`
- `DD-Mon-YY`: `"11-Mar-26"`
- `DD-Mon-YYYY`: `"11-Mar-2026"`
- `MM/DD/YY`: `"03/05/26"`

Returns `null` if the value cannot be parsed.

---

### `getUnknownTasks(actuals, weekToPhase)`
Scans TimeEntries for `Task/Deliverable` values that don't match any known phase name. Returns sorted array of unmatched task strings — useful for spotting data entry inconsistencies.

---

### Jira_CT header normalisation
Headers are lowercased and spaces replaced with underscores:
- `Story Points` → `story_points`
- `Sprint.startDate` → `sprint.startdate`
- `Sprint.endDate` → `sprint.enddate`

The sprint config derivation scans both `sprint.startdate` and `sprint_startdate` variants to handle different Jira Cloud for Sheets export formats.

---

### How `jiraConfig` is derived
```javascript
// Sprint name — first non-empty value in Sprint column
// Sprint end — first non-empty Sprint.endDate, normalised to YYYY-MM-DD
// Sprint days — calendar days between start and end (inclusive)
// Statuses — unique set of Status values across all issues
```

---

## index.html — structure

```
<head>
  CSS (design tokens, layout, component styles)
  External: Chart.js, chartjs-plugin-datalabels, html2pdf
</head>
<body>
  .container#master-report
    .header                    ← Title, budget input, sync/export buttons
    #loader                    ← Spinner shown during fetch
    #dashboard (hidden until data loads)
      .tab-nav                 ← Program / Jira High Level / Jira Sprint View
      #tab-program             ← Existing financial dashboard
      #tab-jira-high           ← High level Jira view
      #tab-jira-sprint         ← Sprint-level Jira view

<script>
  Constants
  globalData object
  Utility functions
  fetchData()
  processData()
  updateUI() + render functions (Program tab)
  switchTab()
  Jira shared helpers
  jRenderHighLevel() + sub-functions
  jRenderSprintView() + sub-functions
  renderJiraTab()
  fetchData() call
</script>
```

---

## index.html — key constants

```javascript
const API_URL = "...";        // Your Apps Script Web App URL — update this
const RATE_FALLBACK = 100;    // Default hourly rate if not set in ResourcePlan
const THRESH_OVER = 40;       // Heatmap: hours/week above this = overloaded (red)
const THRESH_GOOD = 30;       // Heatmap: hours/week above this = optimal (green)
const VARIANCE_FACTOR = 1.2;  // Flag if actual > planned × this factor
const VARIANCE_UNDER = 0.8;   // Flag if actual < planned × this factor
const ACTIVE_DAYS = 14;       // Days window for "active resources" count in RAG banner
const CHART_COLORS = [...];   // Colour palette for Program tab charts
```

---

## index.html — globalData shape

```javascript
let globalData = {
  actuals:      [],   // Raw TimeEntries from GS
  plan:         [],   // Flattened ResourcePlan from GS
  rawPhases:    [],   // Phase name array
  milestones:   [],   // [{name, due, notes}]
  unknownTasks: [],   // Task strings that didn't match a phase
  jiraSummary:  [],   // [{metric, value, rag}]
  jiraIssues:   [],   // [{key, summary, status, assignee, story_points, labels[], sprint, ...}]
  jiraConfig:   {},   // {sprint_name, sprint_start, sprint_end, sprint_days, statuses[]}

  // Computed by processData() — not from GS:
  _phasesActual:  {},
  _phasesPlanned: {},
  _phaseOrder:    [],
  _phaseNameMap:  {},
  _people:        {},
  _weeks:         {}
};
```

---

## index.html — Jira status mapping

```javascript
function jStatusCategory(s) {
  // Returns: "qualification" | "development" | "uat" | "done" | "other"
}

function jStatusColor(s) {
  // Returns hex color:
  // done          → #10b981 (green)
  // development   → #3b82f6 (blue)
  // uat           → #f59e0b (amber)
  // qualification → #94a3b8 (grey)
  // other/unknown → #8b5cf6 (purple — visible, not silently grey)
}
```

To add a new Jira status, find `jStatusCategory()` and add it to the correct array:
```javascript
if (["READY FOR DEVELOPMENT", "RE-OPENED_CH", ..., "YOUR_NEW_STATUS"].includes(sl))
    return "development";
```

---

## index.html — Jira render call chain

```
renderJiraTab()                    ← called after fetchData() completes
  ├── jPopulateSprintSelector()    ← builds sprint dropdown from Jira_CT data
  ├── jRenderHighLevel()           ← renders High Level tab
  │     ├── jRenderHighLevelCharts()
  │     └── jRenderHighLevelTable()
  └── jRenderSprintView()          ← renders Sprint View tab for selected sprint
        ├── jRenderSprintBar()
        ├── jRenderSprintPts()
        ├── jRenderSprintStatusBreakdown()
        ├── jRenderSprintAgingBlocked()
        ├── jRenderSprintIssuesTable()
        └── jRenderSprintCharts()
```

`jRenderSprintView()` is also called directly by the sprint selector `onchange` event.
`jRenderHighLevelTable()` is called directly by the phase/sprint filter `onchange` events.

---

## index.html — chart ID reference

| Chart ID | Tab | What it shows |
|---|---|---|
| `hl-chart-pipeline` | High Level | Horizontal stacked bar — phase proportions |
| `hl-chart-velocity` | High Level | Sprint completion rate — committed vs done |
| `hl-chart-trend` | High Level | Issues per sprint stacked by phase |
| `sv-chart-burndown` | Sprint View | Ideal vs actual burndown line chart |
| `sv-chart-labels` | Sprint View | Label distribution doughnut |
| `sv-chart-workload` | Sprint View | Assignee story points horizontal bar |
| `chart-burn` | Program | Cumulative burn vs budget cap |
| `chart-bench` | Program | Non-billable vs billable time bar |
| `chart-scurve` | Program | Planned vs actual hours S-curve |
| `chart-phase` | Program | Spend by phase actual vs planned |

All charts are stored in `jiraChartRefs` (Jira) or `chartRefs` (Program) and destroyed before redraw to prevent canvas conflicts.

---

## index.html — HTML element ID reference (Jira tabs)

### High Level tab
| ID | Element |
|---|---|
| `hl-phase-cards` | Phase metric cards container |
| `hl-chart-pipeline` | Pipeline flow canvas |
| `hl-chart-velocity` | Sprint completion canvas |
| `hl-chart-trend` | Issues trend canvas |
| `hl-filter-phase` | Phase filter select |
| `hl-filter-sprint` | Sprint filter select |
| `hl-issues-body` | Issues table tbody |

### Sprint View tab
| ID | Element |
|---|---|
| `sprint-selector` | Sprint dropdown |
| `sv-sprint-dates` | Sprint date range label |
| `jsb-total` | Sprint bar — total issues |
| `jsb-done` | Sprint bar — done count |
| `jsb-dev` | Sprint bar — development count |
| `jsb-uat` | Sprint bar — UAT count |
| `jsb-qual` | Sprint bar — qualification count |
| `jsb-blocked` | Sprint bar — flagged count |
| `jsb-days` | Sprint bar — days remaining |
| `jira-pts-label` | Points label (X / Y pts) |
| `jira-pts-pct` | Points percentage |
| `jira-pts-bar` | Points progress bar fill |
| `jira-pts-done` | Points done label |
| `jira-pts-rem` | Points remaining label |
| `sv-chart-burndown` | Burndown canvas |
| `sv-status-breakdown` | Status breakdown bars container |
| `sv-chart-labels` | Label doughnut canvas |
| `sv-chart-workload` | Workload bar canvas |
| `sv-aging-body` | Aging issues table tbody |
| `sv-blocked-body` | Blocked issues table tbody |
| `jira-issues-body` | Sprint issues table tbody |

---

## Common changes

### Change the budget cap default
In `index.html`, find:
```html
<input type="number" id="budgetLimit" value="251392" ...>
```
Update the `value` attribute.

### Change heatmap thresholds
```javascript
const THRESH_OVER = 40;  // hours/week → red
const THRESH_GOOD = 30;  // hours/week → green (between GOOD and OVER)
```

### Change variance alert thresholds
```javascript
const VARIANCE_FACTOR = 1.2;  // actual > planned × 1.2 → over flag
const VARIANCE_UNDER  = 0.8;  // actual < planned × 0.8 → under flag
```

### Add a new Jira label
1. Add the label convention to your team's Jira workflow
2. In `index.html`, find `jLabelClass()` and add the new label → CSS class mapping
3. Add a corresponding CSS class in the style section following the `.jl-*` pattern
4. The label will now appear in the issues table and aging/blocked panels

### Change RAG thresholds for Jira metrics
Edit the formulas in the `Jira_Summary` tab in Google Sheets — the RAG column formulas control the thresholds. No code change needed.

---

## Deployment checklist (new sprint)

- [ ] Jira Cloud for Sheets JQL is pulling the right project
- [ ] `Jira_CT` has refreshed with current sprint data
- [ ] `Sprint`, `Sprint.startDate`, `Sprint.endDate` columns are populated
- [ ] `Jira_Summary` formulas are returning values (check for `#REF!` errors)
- [ ] Apps Script is deployed (no code changes needed unless you edited `Code.gs`)
- [ ] Click 🔄 Sync Data and verify sprint selector shows the new sprint name
