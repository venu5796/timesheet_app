# Acquia Program Control Tower — Technical Documentation

> Version: 4.3 | Source: `index.html` | Format: Single-page HTML application

---

## 1. Overview

The **Acquia Program Control Tower** is a premium, executive-grade dashboard designed for high-fidelity project management. v4.3 introduces a **Mobile-Optimized Interface** and granular resource modelling through **Per-Resource Productivity Quotients**. The application combines real-time financial tracking, Jira development velocity analytics, and sophisticated resource modelling to provide a "single source of truth" for program stakeholders.

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
- **Individual Productivity Quotients:** Every resource now has a dedicated range slider (10% to 100%) to model individual efficiency levels.
- **FE/BE Categorization:** Stakeholders can manually override roles to tag resources as **Frontend (FE)** or **Backend (BE)**. These tags instantly aggregate into team-specific KPI cards.
- **Dynamic Resource Selection:** Checkboxes allow for inclusion/exclusion of individuals to see the impact on total team throughput.
- **Advanced Calculation Logic:**
    - **Ideal Sprint Base:** Normalised to 10 working days (Monday Week 1 to Friday Week 2).
    - **Raw Capacity Formula:** `(80h - Holidays - PTO) × Allocation %`.
    - **Net Available Formula:** `Raw Capacity × Individual Productivity Quotient`.
- **Explanation Sub-text:** The "Raw Capacity" column provides a mathematical breakdown: `[Base - Holidays - PTO] × Allocation %`.
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
- **Cross-Sprint Pipeline:** Visual summary of the "Dev to UAT" queue status.
- **Burndown Analytics:** High-fidelity chart tracking Ideal trajectory, Remaining points, and the "UAT Queue" pressure.
- **Risk Signals:** Dedicated tracking for **Scope Creep**, **Re-opened Tickets**, and **Blocker Pressure**.

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

### Performance Guards
| Guard | Value |
|---|---|
| **Variance Factor** | 1.2 (Flags resources billing 20% over plan) |
| **Aging Threshold** | 16 Working Hours (Flags review/QA stagnation) |
| **SLA Grace** | 1 Hour |

---

## 6. Data Sources

- **`actuals`**: `TimeEntries` sheet (Hours, Rates, Tasks).
- **`plan`**: `ResourcePlan` sheet (Phase allocation, planned hours).
- **`jiraIssues`**: `Jira_CT` sheet (Live ticket status, Points, Sprints).
- **`leaves`**: `Teamleaves` sheet (Individual holiday dates).
