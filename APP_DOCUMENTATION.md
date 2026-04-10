# Acquia Program Control Tower — Technical Documentation

> Version: 3.1 | Source: `index.html` | Format: Single-page HTML application

---

## 1. Overview

The **Acquia Program Control Tower** is a high-fidelity, interactive single-page dashboard designed for senior project managers and stakeholders. It provides real-time visibility into project finances, resource utilisation, and Jira-based development velocity across four dedicated tabs. Version 3.1 introduces a more robust tab-switching engine and enhanced sprint-aware data normalization.

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
- **Budget Cap Control:** Inline editable input to adjust the project's financial ceiling.

### 2.2 Program Dashboard (Main Tab)
- **Smart Insight Card:** AI-style human-readable summary of project health (e.g., "Budget Warning: EAC exceeds cap by €12k").
- **Efficiency KPIs:** Velocity (h/wk), Non-Billable %, and Billable Efficiency with 6-week trend sparklines.
- **EAC Confidence:** Gauge of data reliability based on the volume of future-planned weeks.
- **Variance & Delinquency:** 
    - **Split-Bar Variance:** Visual comparison of Prorated Plan (Ghost bar) vs. Actual (Solid bar).
    - **Delinquency:** List of resources with 0 hours, showing "Days since last entry."
- **Charts:** Cumulative Burn vs. Cap, Stacked Billability, S-Curve (Plan vs Actual), and Phase Spend.
- **Resource Heatmap:** 
    - **Search Filter:** Real-time resource lookup.
    - **Pinned Headers:** Sticky top row and first column for large datasets.
    - **Color-Coded Tiers:** Over >40h (Red), Optimal 30-40h (Green), Low 1-29h (Amber).

### 2.3 Jira High Level & Sprint View
- **Multi-Sprint Awareness:** Automatically detects and displays dates for the selected sprint, ensuring burndown and scope-creep metrics are accurate across historical and current sprints.
- **Scope Creep Indicator:** Tracks tickets added to the sprint after its start date.
- **SLA Hour-Budget Breaches:** Flags tickets exceeding their story-point working-hour allocation.
- **Aging Issues:** Highlights tickets stuck in Development or Review for >16 working hours.
- **Pipeline Flow:** Distribution of tickets across Qualification, Development, UAT, and Done.

### 2.4 Capacity & Planning
- **Velocity Simulation (What-If?):** Interactive slider allowing PMs to model how team velocity changes impact the final project EAC and buffer.
- **Hours-to-Points Ratio:** Configurable conversion factor for story point estimation.
- **Sprint Planner:** Visual interface to move tickets between Backlog and Planned Sprint buckets.

---

## 3. Technical Architecture

### 3.1 Tab Engine & Navigation
- **Asynchronous Rendering:** Tabs can render independently even if specific datasets (e.g., timesheet actuals) are still loading or unavailable.
- **CSS-Driven Visibility:** Optimized `.tab-panel` transitions with CSS animations for a smoother user experience.
- **Persistent States:** Dark mode and selected filters are preserved across tab switches.

### 3.2 Design System (SaaS Aesthetic)
- **Glassmorphism:** Blur-filtered headers and persistent UI elements for depth.
- **Modern Indigo Palette:** High-contrast indigo and slate scheme optimized for data density.
- **Night Mode (Dark Mode):** 
    - Fully semantic variable mapping (`--surface`, `--surface-2`, `--st-ok-bg`, etc.).
    - Automatic Chart.js re-rendering to update gridlines and axis labels on theme toggle.
    - Persisted via `localStorage` and respects system preferences.

### 3.3 Data Normalization & Processing
- **Sprint Date Resolution:** The app maps Jira issues to specific sprint windows stored in the `jiraConfig.sprint_dates` map, resolving the "Latest Sprint" bug from Version 3.0.
- **Monday-Snapping:** The `getMonday()` helper ensures all weekly aggregations are consistent, regardless of the date format provided by the API.
- **Robust Date Parsing:** `parseDateSafe()` handles ISO, DD-Mon-YY, DD-Mon-YYYY, and Mon-DD formats with India-timezone safety.
- **Prorata Basis:** Current week metrics (Variance, etc.) are scaled based on the number of elapsed working days (Mon-Fri) to ensure "Actuals" are compared fairly against incomplete "Plan" targets.

### 3.4 Working Hours Engine
Calculates elapsed time excluding:
- Weekends (Saturday/Sunday).
- Non-working hours (outside 09:00 - 18:00).
- **Public Holidays:** Hardcoded in `CONFIG.WORK_HOURS.holidays` (India 2026).

---

## 4. Configuration & Thresholds (`CONFIG`)

| Constant | Default | Purpose |
|---|---|---|
| `THRESH_OVER` | `40` h | Heatmap: Over-utilisation trigger |
| `THRESH_GOOD` | `30` h | Heatmap: Lower bound of optimal range |
| `RATE_FALLBACK` | `100` | €/h used for simulations when no rate is found |
| `VARIANCE_FACTOR` | `1.2` | "Red Flag" trigger (20% over plan) |
| `VARIANCE_UNDER` | `0.8` | "Green Flag" trigger (20% under plan) |
| `ACTIVE_DAYS` | `14` | Window for considering a resource "Active" |
| `SYNC_INTERVAL_MS` | `15 min` | Background data refresh cadence |

---

## 5. Sync & Error Handling
- **Data Monitor:** A hidden debug panel appears if the API returns 0 rows, assisting in troubleshooting.
- **Sync Status:** The header displays the last successful sync time and a countdown.
- **Error Resilience:** Individual chart failures or missing fields do not crash the entire dashboard; fallback logic provides "—" or "0" values.

---

## 6. PDF Export
- **A3 Landscape:** Optimized layout for executive status reports.
- **Print Mode:** Automatically hides the sidebar, navigation, and interactive controls to maximize data visibility.
- **Canvas Snapshots:** Converts interactive Chart.js canvases to static images during export to ensure cross-browser compatibility.
