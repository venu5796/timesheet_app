# Refactoring Guide: Program Control Tower

This document summarizes the architectural recommendations and data structure changes proposed on March 31, 2026.

## 1. Data Structure Changes (Excel)
The project has been migrated from a "Matrix" layout to a "Vertical/Relational" layout in `VEOLIA_UK_Optimized.xlsx`.

### Key Sheets:
- **`Settings`**: Centralizes app constants (SLA thresholds, Rates, Holidays).
- **`Phases`**: Defines the timeline with explicit `Start` and `End` dates.
- **`ResourcePlan`**: Vertical format (`Person | Role | Start | End | Hours/Week`). 
  - *Benefit*: Fill in 1 row for a multi-week assignment. Only add rows when hours change.

## 2. Recommended App Refactoring (index.html)
- **Replace Hardcoded Capacity**: Remove the `CAP_DATA` constant and fetch it from the new `ResourcePlan` sheet.
- **Shared Fetch Logic**: Refactor `fetchData` and `silentFetch` into a single `loadData(isSilent)` function.
- **Dynamic Configuration**: Update the `CONFIG` object to populate itself from the `Settings` sheet instead of hardcoded values.
- **PDF Export**: Add a `.no-print` CSS class to the sidebar and buttons to ensure a clean report export.

## 3. New Apps Script Logic (Suggested)
To support the vertical `ResourcePlan`, your Google Apps Script `doGet()` should perform a "Date Expansion":
1. Read the `ResourcePlan` rows.
2. For each row, loop through the weeks between `Start Date` and `End Date`.
3. Output a flat JSON array of weekly objects that the app's `processData()` can easily consume.

## 4. Maintenance
The transformation script used to generate the new format is located at:
`~/.gemini/tmp/documents/transform_excel.py`
