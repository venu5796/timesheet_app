# Acquia Program Control Tower

A single-file web dashboard that combines **financial project data** (burn, resource utilisation, milestones) with **Jira sprint tracking** (velocity, blockers, aging issues, phase pipeline) — all served from a Google Apps Script backend connected to Google Sheets.

---

## Architecture overview

```
Google Sheets
  ├── TimeEntries        ← Actual hours logged per person per week
  ├── ResourcePlan       ← Planned hours per person per week
  ├── Phases             ← Week-to-phase mapping
  ├── Jira_Summary       ← Computed Jira metrics (velocity %, blockers, RAG)
  └── Jira_CT            ← Raw Jira issues (synced via Jira Cloud for Sheets)
         └── Columns: Key · Summary · Status · Assignee · Story Points
                      Created · Updated · Labels · Sprint
                      Sprint.startDate · Sprint.endDate
        │
        ▼
  Google Apps Script (Code.gs)
  └── doGet() → returns single JSON payload
        │
        ▼
  index.html (standalone, no build step)
  ├── 📊 Program tab     ← Budget burn, resource heatmap, milestones
  ├── 🔵 Jira High Level ← Phase pipeline, sprint velocity trend, all issues
  └── 🎯 Jira Sprint View← Burndown, status breakdown, aging, blocked issues
```

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire frontend — HTML, CSS, JS in one file |
| `Code.gs` | Google Apps Script backend — reads Sheets, returns JSON |

---

## Setup guide

### Step 1 — Google Sheets structure

Your spreadsheet needs these tabs with exact names:

**TimeEntries** — one row per time entry
| Column | Description |
|---|---|
| Person | Resource name |
| Role | Their role |
| Date | Week start date |
| Task/Deliverable | Phase-mapped task name |
| Time in Hours | Hours logged |
| Subtotal | Billable amount (hours × rate) |
| Rate | Hourly rate |

**ResourcePlan** — planned hours grid
- Row 2: phase names in columns D onwards
- Row 3: week start dates in columns D onwards
- Row 4+: person rows with `Role | Person | Rate | hours per week...`

**Phases** — week-to-phase mapping
- Column A: phase name
- Columns B+: ISO week dates belonging to that phase

**Jira_Summary** — computed metrics (formulas, not manual)
| metric | value | rag |
|---|---|---|
| velocity_pct | `=IFERROR(COUNTIF(Jira_CT!C:C,"Done")/COUNTA(Jira_CT!A2:A)*100,0)` | `=IF(B2>=80,"GREEN",IF(B2>=60,"AMBER","RED"))` |
| blocked | `=COUNTIF(Jira_CT!H:H,"*blocked*")` | `=IF(B3=0,"GREEN",IF(B3<=2,"AMBER","RED"))` |
| needs_client_input | `=COUNTIF(Jira_CT!H:H,"*needs-client-input*")` | `=IF(B4=0,"GREEN",IF(B4>=1,"AMBER","RED"))` |
| dependency | `=COUNTIF(Jira_CT!H:H,"*dependency*")` | `=IF(B5=0,"GREEN",IF(B5<=1,"AMBER","RED"))` |
| needs_review | `=COUNTIF(Jira_CT!H:H,"*needs-review*")` | `=IF(B6=0,"GREEN",IF(B6<=2,"AMBER","RED"))` |
| aging_issues | `=COUNTIFS(Jira_CT!C:C,"In Progress",Jira_CT!G:G,"<"&NOW()-2)` | `=IF(B7=0,"GREEN",IF(B7<=2,"AMBER","RED"))` |
| total_issues | `=COUNTA(Jira_CT!A2:A)` | |
| total_points | `=SUM(Jira_CT!E:E)` | |
| completed_points | `=SUMIF(Jira_CT!C:C,"Done",Jira_CT!E:E)` | |
| last_synced | `=TEXT(NOW(),"yyyy-mm-dd hh:mm")` | |

**Jira_CT** — synced via Jira Cloud for Sheets add-on
| Column | Jira field |
|---|---|
| Key | `issuekey` |
| Summary | `summary` |
| Status | `status` |
| Assignee | `assignee` |
| Story Points | `story_points` |
| Created | `created` |
| Updated | `updated` |
| Labels | `labels` |
| Sprint | `sprint` |
| Sprint.startDate | `sprint.startdate` |
| Sprint.endDate | `sprint.enddate` |

JQL for Jira Cloud for Sheets: `project = "YOUR_PROJECT_KEY"` (broad — all issues, filtered in dashboard)

Set auto-refresh to hourly.

---

### Step 2 — Deploy the Apps Script

1. Open your Google Sheet
2. Go to **Extensions → Apps Script**
3. Delete any existing code and paste the full contents of `Code.gs`
4. Click **Deploy → New deployment**
5. Type: **Web app**
6. Execute as: **Me**
7. Who has access: **Anyone**
8. Click **Deploy** → copy the Web App URL

> Every time you change `Code.gs`, create a **new deployment** — do not update existing. The URL changes each time.

---

### Step 3 — Configure index.html

Open `index.html` and find the `API_URL` constant near line 598:

```javascript
const API_URL = "https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec";
```

Replace with your Web App URL from Step 2.

---

### Step 4 — Open the dashboard

Open `index.html` directly in any browser. No server needed — it fetches data from the Apps Script URL on load.

Click **🔄 Sync Data** to refresh manually at any time.

---

## Jira labels convention

The dashboard recognises these four labels. Apply them directly on Jira tickets:

| Label | When to use |
|---|---|
| `blocked` | Issue cannot progress — any blocker |
| `needs-client-input` | Waiting on client decision or asset |
| `dependency` | Blocked by another team or ticket |
| `needs-review` | Ready for review, no reviewer assigned |

---

## Status workflow mapping

The dashboard maps your 16 Jira statuses to four categories:

| Category | Statuses |
|---|---|
| Qualification | NEW · DISCOVERY_CH · STORY APPROVAL |
| Development | READY FOR DEVELOPMENT · RE-OPENED_CH · IN PROGRESS · READY FOR REVIEW · CODE REVIEW · READY FOR QA · QA · UAT RELEASE QUEUE |
| UAT | READY FOR UAT · UAT · RE-OPENED IN UAT · PROD RELEASE QUEUE |
| Done | DONE |

If your team adds a new status, add it to the `jStatusCategory()` function in `index.html`.

---

## Sprint configuration

Sprint name, start date, end date, and sprint length are all **derived automatically** from the `Sprint`, `Sprint.startDate`, and `Sprint.endDate` columns in `Jira_CT`. No manual updates needed when a new sprint starts — just refresh Jira Cloud for Sheets.

---

## Tabs reference

### 📊 Program tab
Sourced from TimeEntries, ResourcePlan, and Phases sheets.

| Section | What it shows |
|---|---|
| RAG Health Banner | Budget · Schedule · Resources · Delivery status |
| KPI Cards | Burned · Remaining · EAC · Velocity · Non-billable % · Billable efficiency |
| Variance Flags | Resources over/under plan by >20% |
| Delinquency Panel | Scheduled resources who submitted 0 hours |
| Burn vs Budget | Cumulative spend line vs budget cap |
| Roll-off Schedule | Predictive end dates per resource |
| Phase Breakdown | Actual vs planned spend per phase |
| Heatmap | Weekly hours per resource (over/optimal/low) |
| Weekly Spotlight | Hour-by-hour breakdown for selected week |

### 🔵 Jira — High Level tab
Sourced from Jira_CT (all sprints).

| Section | What it shows |
|---|---|
| Phase metric cards | Issue count + % per phase across whole project |
| Pipeline flow | Stacked horizontal bar — proportion in each phase |
| Sprint completion rate | Committed vs done story points per sprint |
| Issues per sprint | Stacked bar — volume and phase breakdown by sprint |
| All issues table | Filterable by phase and sprint |

### 🎯 Jira — Sprint View tab
Sourced from Jira_CT, filtered to selected sprint.

| Section | What it shows |
|---|---|
| Sprint selector | Auto-populated from Sprint column — switches all data |
| Overview bar | Total · Done · Dev · UAT · Qual · Flagged · Days left |
| Points progress | Story points completed vs total |
| Burndown | Ideal vs actual points remaining over sprint days |
| Status breakdown | All 16 statuses ordered by workflow position |
| Label distribution | Doughnut — blocked / needs-client / dependency / needs-review |
| Assignee workload | Story points per person horizontal bar |
| Aging issues | Development issues not updated in >48h |
| Flagged/blocked | Issues carrying blocked, needs-client-input, or dependency labels |
| Sprint issues table | All issues sorted by workflow stage, with age warning |

---

## Debug mode

Append `?debug=1` to your Apps Script Web App URL to see raw data diagnostics:

```
https://script.google.com/macros/s/YOUR_ID/exec?debug=1
```

Returns JSON showing raw vs normalised dates from TimeEntries, ResourcePlan, Phases, Jira_Summary, and Jira_CT. Use this if data isn't appearing correctly.

---

## PDF export

Click **📄 Export PDF** in the Program tab header. Exports the Program tab as an A3 landscape PDF. The Jira tabs are not included in the export — they are interactive views not designed for static print.

---

## Updating for a new sprint

Nothing to change in the code. When Jira Cloud for Sheets refreshes `Jira_CT`:
- Sprint selector auto-populates with the new sprint name
- Burndown recalculates from the new start/end dates
- High Level charts update to include the new sprint bar
- All metrics filter correctly to the selected sprint

---

## Known limitations

| Limitation | Workaround |
|---|---|
| Jira Cloud for Sheets max refresh is 1 hour | Manually trigger a sheet refresh for real-time data |
| Burndown is estimated from current done points | Not a true day-by-day historical — accurate shape only after sprint ends |
| PDF export covers Program tab only | Screenshot Jira tabs manually if needed for reports |
| Single-file architecture (~2,100 lines) | Functional for current scale — split into CSS/JS files if team grows |

---

## Roadmap (future enhancements)

- [ ] Master Dashboard — unified RAG view across all 5 goals
- [ ] CR/SOW Builder integration
- [ ] Sprint velocity trend (multi-sprint historical view)
- [ ] Email digest automation via Apps Script triggers
- [ ] Mobile-responsive Jira tab layout
