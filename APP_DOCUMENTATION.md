# Acquia Program Control Tower — Technical Documentation

> Version: 4.5 | Source: `index.html` | Format: Single-page HTML application

---

## 1. Overview

The **Acquia Program Control Tower** is a premium, executive-grade dashboard designed for high-fidelity project management. v4.5 refines the **Capacity Analysis Engine** to exclusively target development roles while calculating targeted PTO deductions, and includes **Dual-Threshold SLA Warnings** for development work items, enabling early visibility into at-risk tickets before hard SLA breaches. The application combines real-time financial tracking, Jira development velocity analytics, sophisticated resource modelling, and proactive risk flagging to provide a "single source of truth" for program stakeholders.

---

## 2. Interface Architecture

### 2.1 Fixed Executive Sidebar (Desktop)
A permanent 180px left-hand sidebar providing high-level program vitals:
- **Health Indicators:** Compact RAG cards (Budget, Schedule, Resources, Delivery) using high-fidelity status dots (Green, Amber, Red).
- **Financial Pulse:** Quick-view mini-cards for **Total Burn**, **Forecast (EAC)**, and **Remaining Budget**.
- **Projections:** Real-time calculation of **Budget Runway** (weeks) and **Projected Finish** date based on plan data.
- **Risk Assessment:** A high-contrast "Overall Risk" badge (Low, Moderate, Critical) that updates dynamically.

### 2.2 Functional Topbar & Responsive Design
The command center for global project controls:
- **Responsive Layout:** Automatically collapses the sidebar and stacks grid components on tablets and mobile devices.
- **Integrated Navigation:** Seamless switching between Program, Jira High Level, Sprint View, and Capacity tabs.
- **Live Sync Engine:** Status indicator showing last-synced timestamp and a pulsing background fetch dot.
- **Scenario Modeller:** A topbar-integrated **Budget Cap** input that triggers pro-rata adjustments across all financial charts.

---

## 3. Core Analytics & Planning

### 3.1 Capacity Analysis Engine (Planning Tab)
The Planning tab allows for granular resource modeling for specific sprints:
- **Developer-Only Filtering:** Automatically filters out non-development roles (e.g., PM, QA, Design) so the capacity model exclusively loads resources mapped as Developers (Frontend, Backend, Dev).
- **Individual Productivity Quotients:** Every resource has a dedicated range slider (10% to 100%) to model individual efficiency levels.
- **FE/BE Categorization:** Stakeholders can manually override roles to tag resources as **Frontend (FE)** or **Backend (BE)**. These tags instantly aggregate into team-specific KPI cards.
- **Dynamic Resource Selection:** Checkboxes allow for inclusion/exclusion of individuals to see the impact on total team throughput.
- **Advanced Calculation Logic:**
    - **Ideal Sprint Base:** Normalised to 10 working days (Monday Week 1 to Friday Week 2).
    - **Targeted PTO Deduction:** PTO and holidays are calculated and subtracted *only* if they fall within the specific weeks a resource is actively allocated to the sprint.
    - **Raw Capacity Formula:** `Total Planned Hours (within sprint weeks) - PTO (within allocated weeks)`.
    - **Net Available Formula:** `Raw Capacity × Individual Productivity Quotient`.
- **Explanation Sub-text:** The "Raw Capacity" column provides a transparent mathematical breakdown beneath the value: `Planned: Xh - PTO (allocated weeks): Yh`.
- **Team KPI Cards:** Real-time totals for **Frontend Net**, **Backend Net**, and **Total Net Capacity**.

### 3.2 Program Performance (Dashboard Tab)
- **Insight Banner:** A Glassmorphism-styled card providing proactive summaries (e.g., "EAC exceeds cap by €12k").
- **Analytical KPI Strip:**
    - **Velocity (Alloc vs Spend):** Lead card showing actual hours/week with an integrated sparkline trend.
    - **Non-Billable Share:** Executive gauge of non-revenue time.
    - **Budget Burn Rate:** Average weekly spend vs. pro-rata target.
- **Advanced Charts:**
    - **Cumulative S-Curve:** Tracks cumulative Actual vs. Planned hours over time.
    - **Utilization Heatmap:** A per-resource, per-week grid with **Search/Filter** and color-coded allocation matching.

### 3.3 Jira Pipeline & Sprint Health
- **Cross-Sprint Pipeline:** Visual summary of the "Dev to UAT" queue status with story point totals and completion percentage.
- **Burndown Analytics:** High-fidelity chart tracking Ideal trajectory, Remaining points, and the "UAT Queue" pressure.
- **Risk Signals:** Dedicated tracking for **Scope Creep**, **Re-opened Tickets**, and **Blocker Pressure**.

#### 3.3.1 At-Risk Queue (Sprint View)
A real-time risk alerting system that flags development items at intermediate and critical SLA thresholds:
- **Dual-Threshold SLA Warnings:** Development items trigger two levels of alerts based on elapsed working hours since status change:
  - **SLA Warning** (Orange Badge): Item has exceeded the `warn` threshold but is still within the `over` threshold. Provides early visibility to allow corrective action.
  - **SLA Breach** (Red Badge): Item has exceeded the `over` threshold + grace period. Indicates hard SLA violation requiring immediate escalation.
- **Aging Flags:** Items in "Ready for Review" or "Code Review" statuses longer than 16 working hours are flagged as aging (Orange Badge) to prevent review bottlenecks.
- **Story Point Bands:** SLA thresholds vary by story point complexity:
  - **1 point:** warn=2h, over=2h
  - **2 points:** warn=4h, over=4h
  - **3 points:** warn=8h, over=12h
  - **5 points:** warn=16h, over=24h
  - **8 points:** warn=24h, over=24h
  - **Default (>8 points):** warn=16h, over=24h
- **Working Hours Basis:** All time calculations respect business hours (10:00–18:00), exclude weekends/holidays, and account for person-specific PTO.
- **Sorted Display:** At-Risk items sorted by hours elapsed (descending) to prioritize most critical issues.

---

## 4. Technical Engine

### 4.1 UI/UX & Theming
- **Executive Palette:** Pure Black/Orange for Dark mode; Linen/Dark Tangerine for Light mode.
- **Adaptive UI:** Media queries ensure all charts, sliders, and tables are fully functional on screens down to 320px wide.

### 4.2 Working Hours Engine (`workingHoursElapsed_`)
- **Standard Schedule:** Hardcoded to **10:00 – 18:00** (8 hours/day base).
- **India-Specific Holidays:** Integrated 2026-2027 holiday set.
- **Fuzzy PTO Matching:** Case-insensitive resource matching between `Teamleaves` and `ResourcePlan`.

---

## 5. Thresholds & Configuration (`CONFIG`)

### 5.1 Performance Guards & SLA Configuration
| Parameter | Value | Purpose |
|---|---|---|
| **Variance Factor** | 1.2 | Flags resources billing 20% over plan |
| **Aging Threshold** | 16 Working Hours | Flags items stalled in review/QA |
| **SLA Grace Period** | 1 Hour | Grace period after `over` threshold before flagging breach |
| **Work Hours Start** | 10:00 | Business day start time |
| **Work Hours End** | 18:00 | Business day end time |

### 5.2 Development SLA Thresholds by Story Point
| Story Points | Warn Threshold | Over Threshold | Risk Level |
|---|---|---|---|
| 1 | 2h | 2h | Single-threshold (no intermediate warning) |
| 2 | 4h | 4h | Single-threshold (no intermediate warning) |
| 3 | 8h | 12h | Dual-threshold (4h warning window) |
| 5 | 16h | 24h | Dual-threshold (8h warning window) |
| 8 | 24h | 24h | Single-threshold (no intermediate warning) |
| Default (>8) | 16h | 24h | Dual-threshold (8h warning window) |

**SLA Logic:** When an item enters "In Progress" status, elapsed working hours are measured from `status_category_changed`. If hours exceed `warn` + grace period, a SLA Warning is triggered (orange badge). If hours exceed `over` + grace period, a SLA Breach is triggered (red badge).

---

## 6. Data Sources

- **`actuals`**: `TimeEntries` sheet (Hours, Rates, Tasks).
- **`plan`**: `ResourcePlan` sheet (Phase allocation, planned hours).
- **`jiraIssues`**: `Jira_CT` sheet (Live ticket status, Points, Sprints).
- **`leaves`**: `Teamleaves` sheet (Individual holiday dates).
- **`jiraSummary`**: Live Jira board metadata and labels.
- **`milestones`**: Key project deliverables and due dates.
- **`spendAnalysis`**: Aggregated actual hours vs story points mapping.
- **`rawPhases`**: Raw Phase breakdown configurations.