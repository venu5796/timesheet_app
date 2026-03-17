# Acquia Program Control Tower — Documentation

## 📊 Overview
The **Acquia Program Control Tower** is a high-fidelity, interactive dashboard designed for senior project managers and stakeholders. It provides real-time visibility into project finances, resource utilization, and Jira-based development velocity. By synthesizing data from multiple sources (via Google Apps Script), it offers actionable insights through "RAG" (Red-Amber-Green) health indicators and predictive analytics.

---

## 🚀 Key Features

### 1. Program Dashboard
A high-level view of the project's financial and operational status.
*   **7 KPI Cards:** Real-time tracking of Burned (€), Remaining Budget, Forecast (EAC), Velocity (h/wk), Non-Billable %, Billable Efficiency, and EAC Confidence.
*   **Project Health Banner:** Instant RAG status for Budget, Schedule, Resources, and Delivery.
*   **Resource Management:** 
    *   **Heatmap:** A detailed "Resource Utilization Heatmap" highlighting over-utilized (>40h) and under-utilized resources.
    *   **Predictive Roll-off:** A schedule predicting when resources will roll off the project based on planned allocations.
*   **Financial Visualization:** 
    *   Cumulative Burn vs. Budget Cap.
    *   Planned vs. Actual Hours (S-Curve).
    *   Spend by Phase breakdown.

### 2. Jira Integration (Sprint & High Level)
Granular tracking of development progress.
*   **Sprint View:**
    *   **Burndown Chart:** Tracks story points over the course of a sprint.
    *   **SLA Breach Tracking:** Automatically flags tickets that have stayed in development longer than their story-point SLA.
    *   **Aging & Blocked Issues:** Identifies tickets with no updates for >2 days or those marked as "Flagged."
    *   **Code Review Panel:** Monitors tickets awaiting reviewer feedback.
*   **High-Level View:**
    *   Pipeline flow across Qualification, Development, UAT, and Done phases.
    *   Historical sprint completion rates and volume trends.

### 3. Advanced Utilities
*   **Interactive Controls:** Switch between weeks/sprints, adjust the remaining budget cap on the fly, and filter tables dynamically.
*   **Dark Mode:** Full support for a professional dark theme with optimized chart colors.
*   **PDF Export:** Generate high-quality PDF reports for stakeholders using `html2pdf.js`.
*   **Auto-Sync:** Background data refresh every 15 minutes with a visual sync indicator.

---

## 🛠 Technical Architecture

### Tech Stack
*   **Frontend:** HTML5, Vanilla CSS3 (Custom Design Tokens), JavaScript (ES6+).
*   **Charts:** [Chart.js](https://www.chartjs.org/) with `chartjs-plugin-datalabels`.
*   **PDF Generation:** `html2pdf.js`.
*   **Data Source:** Google Apps Script (Web App) acting as a JSON API provider.
*   **Typography:** 'DM Sans' for UI and 'DM Mono' for financial/numerical data.

### Data Flow
1.  **Request:** The app fetches JSON data from a Google Apps Script URL.
2.  **Processing:** The script parses "Actuals" (from time entries) and "Plan" (from resource planning) data.
3.  **Mapping:** It intelligently maps task descriptions to specific project phases (e.g., "Sprint 0", "UAT").
4.  **Calculations:** Computes cumulative sums, EAC, variances, and working-hour-based SLAs.
5.  **Rendering:** Updates DOM elements and Chart.js instances.

---

## 🎨 UI & Design Tokens
The application uses a modern design system defined in `:root` CSS variables:
*   **Primary Blue:** `#003d82` (Acquia Brand)
*   **Success Green:** `#10b981`
*   **Danger Red:** `#ef4444`
*   **Warning Amber:** `#f59e0b`
*   **Transitions:** Smooth 0.2s fades and transforms for an "alive" feel.

---

## 📋 User Guide
*   **Syncing Data:** Click **🔄 Sync Data** to pull the latest entries. The "Last synced" time is displayed in the header.
*   **Setting Budget:** Enter a value in the **Remaining Budget Cap** field; all burn charts and EAC calculations will update instantly.
*   **Exporting:** Click **📄 Export PDF** to save the current dashboard view. The app automatically reformats the layout for A3 landscape PDF printing.
*   **Dark Mode:** Use the 🌙 toggle in the header for low-light environments.
