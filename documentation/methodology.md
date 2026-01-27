# Methodology

This report uses point-in-time logic for headcount and compensation. Employment overlap is evaluated within the target window (month, YTD, full year). Compensation is normalized across salary and hourly using **Annualized Pay**.

## Key Concepts
- Employment overlap: `Hire <= End` AND (`Exit` blank OR `Exit >= Start`)
- Proration by day count within the selected window
- Slicer-aware iteration using `VALUES(Employees[Employee ID])` + context transition
- Keep Department/Location slicers active by avoiding table-wide `ALL(Employees)`
