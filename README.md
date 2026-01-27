# HR Workforce & Compensation Analytics — Power BI Sample

> **Developer:** Brittney N Workman **https://github.com/brittneybi**

A focused Power BI report for analyzing workforce composition and compensation. It answers practical questions such as:

- How many **active** employees do we have today and how has headcount changed?
- What is our **compensation run‑rate**, and how does it split across **salary vs. hourly** roles?
- How do **departments** and **locations** compare on **average/median compensation** and **headcount**?
- What are the **hire** and **exit** patterns (timing and reasons) of our employees, and how do they affect current cost?

> **Note:** This project uses entirely **synthetic** data.

---

## 📁 Dataset
- **Source:** Synthetic HR employee roster (Excel or embedded dataset)
- **Size:** ~1,000 rows and ~20 columns (hire/exit dates, status, pay type, annual/hourly pay, typical hours, department, job, location, performance rating)
- **Scope:** Includes both **active** and **separated** employees (terminations and resignations)

---

## 🧱 Data Model
- **Fact table:** `Employees` (demographic, job, pay, employment dates, status)
- **Dimension table:** A **Date** table used for point-in-time analysis and date/time windows via small helper measures
- **Style:** Simple layout centered on `Employees`; compensation is normalized so salary and hourly can be compared
- **Diagram:** See `/documentation/data-model.png`

---

## 📊 Report Highlights
- **Workforce Summary:** Headcount (as‑of date), hires/exits, average tenure, average annualized comp; slicers for Department, Location, Pay Type, Status
- **Compensation Intelligence:** Annualized Compensation (Active), Monthly Compensation (prorated), Salary/Hourly run‑rate, YTD variance to run‑rate, pay distribution
- **Hire/Exit & Lifecycle:** Trends and reasons for exits, tenure cohorts, performance distributions, and location views
- **Accessibility:** Color‑blind‑friendly palette and concise alt text on visuals/tooltips

---

## 🧮 Key DAX (selected)
_Path: `/scripts/dax/measures.dax`_

```DAX
// Helpers (example)
AnchorDate := COALESCE( MAX('Calendar'[Date]), TODAY() )
Y := YEAR([AnchorDate])

// Active Headcount (point-in-time)
Active Headcount :=
SUMX(
    VALUES(Employees[Employee ID]),
    VAR Hire = CALCULATE(SELECTEDVALUE(Employees[Hire Date]))
    VAR Exit = CALCULATE(SELECTEDVALUE(Employees[Exit Date]))
    VAR d    = [AnchorDate]
    RETURN IF( Hire <= d && ( ISBLANK(Exit) || Exit >= d ), 1 )
)

// Annualized Pay (row-level normalization)
Annualized Pay :=
VAR IsSalary = SELECTEDVALUE(Employees[Salary or Hourly]) = "Salary"
VAR Annual   = COALESCE(SELECTEDVALUE(Employees[Annual Salary]),0)
VAR Rate     = COALESCE(SELECTEDVALUE(Employees[Hourly Rate]),0)
VAR Hours    = COALESCE(SELECTEDVALUE(Employees[Typical Hours]),40)
RETURN IF(IsSalary, Annual, Rate*Hours*52)

// Annualized Compensation (Active)
Annualized Compensation (Active) :=
SUMX(
    VALUES(Employees[Employee ID]),
    VAR Hire = CALCULATE(SELECTEDVALUE(Employees[Hire Date]))
    VAR Exit = CALCULATE(SELECTEDVALUE(Employees[Exit Date]))
    VAR d    = [_AsOfDate]
    VAR Active = Hire <= d && ( ISBLANK(Exit) || Exit >= d )
    VAR Pay = CALCULATE([Annualized Pay])
    RETURN IF( Active, Pay )
)

// Monthly Compensation (Prorated for exits and hires)
Monthly Compensation :=
VAR StartDate = [_MonthStart]
VAR EndDate   = [_MonthEnd]
VAR YearStart = DATE( YEAR(StartDate), 1, 1 )
VAR YearEnd   = DATE( YEAR(StartDate), 12, 31 )
VAR DaysInYear = DATEDIFF(YearStart, YearEnd, DAY) + 1
RETURN
SUMX(
    VALUES(Employees[Employee ID]),
    VAR Hire = CALCULATE(SELECTEDVALUE(Employees[Hire Date]))
    VAR Exit = CALCULATE(SELECTEDVALUE(Employees[Exit Date]))
    VAR InWindow = Hire <= EndDate && ( ISBLANK(Exit) || Exit >= StartDate )
    VAR Annualized = CALCULATE([Annualized Pay])
    VAR EffStart = MAX(Hire, StartDate)
    VAR EffEnd   = MIN( COALESCE(Exit, EndDate), EndDate )
    VAR ActiveDays = IF( InWindow, MAX(0, DATEDIFF(EffStart, EffEnd, DAY) + 1), 0 )
    RETURN DIVIDE( Annualized * ActiveDays, DaysInYear )
)

// Run‑Rates (Active)
Hourly Run-Rate (Active) :=
VAR d = [_AsOfDate]
RETURN
SUMX(
    VALUES(Employees[Employee ID]),
    VAR Hire     = CALCULATE(SELECTEDVALUE(Employees[Hire Date]))
    VAR Exit     = CALCULATE(SELECTEDVALUE(Employees[Exit Date]))
    VAR IsHourly = CALCULATE(SELECTEDVALUE(Employees[Salary or Hourly]) = "Hourly")
    VAR Active   = Hire <= d && ( ISBLANK(Exit) || Exit >= d )
    VAR Rate     = CALCULATE(COALESCE(SELECTEDVALUE(Employees[Hourly Rate]),0))
    VAR Hours    = CALCULATE(COALESCE(SELECTEDVALUE(Employees[Typical Hours]),0))
    RETURN IF( Active && IsHourly, Rate*Hours*52, 0 )
)

Salary Run-Rate (Active) :=
VAR d = [_AsOfDate]
RETURN
SUMX(
    VALUES(Employees[Employee ID]),
    VAR Hire     = CALCULATE(SELECTEDVALUE(Employees[Hire Date]))
    VAR Exit     = CALCULATE(SELECTEDVALUE(Employees[Exit Date]))
    VAR IsSalary = CALCULATE(SELECTEDVALUE(Employees[Salary or Hourly]) = "Salary")
    VAR Active   = Hire <= d && ( ISBLANK(Exit) || Exit >= d )
    VAR Annual   = CALCULATE(COALESCE(SELECTEDVALUE(Employees[Annual Salary]),0))
    RETURN IF( Active && IsSalary, Annual, 0 )
)

Total Compensation Run-Rate (Active) :=
[Hourly Run-Rate (Active)] + [Salary Run-Rate (Active)]

// Annual Compensation (Prorated, Full Year)
Annual Compensation (Prorated, Full Year) :=
VAR YearStart = DATE([Y],1,1)
VAR YearEnd   = DATE([Y],12,31)
VAR DaysInYear = DATEDIFF(YearStart, YearEnd, DAY) + 1
RETURN
SUMX(
    VALUES(Employees[Employee ID]),
    VAR Hire = CALCULATE(SELECTEDVALUE(Employees[Hire Date]))
    VAR Exit = CALCULATE(SELECTEDVALUE(Employees[Exit Date]))
    VAR InYear = Hire <= YearEnd && ( ISBLANK(Exit) || Exit >= YearStart )
    VAR Pay = CALCULATE([Annualized Pay])
    VAR EffStart = MAX(Hire, YearStart)
    VAR EffEnd   = MIN( COALESCE(Exit, YearEnd), YearEnd )
    VAR ActiveDays = IF( InYear, MAX(0, DATEDIFF(EffStart, EffEnd, DAY) + 1), 0 )
    RETURN DIVIDE( Pay * ActiveDays, DaysInYear )
)
