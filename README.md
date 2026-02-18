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

<div class="grid">
    <div class="body">
      <h3>1. HR Workforce &amp; Compensation Analytics</h3>
      <p>Power BI sample project with .pbix report file, documentation, scripts, and visuals.</p>
        <a class="card" href="https://github.com/brittneybi/pbi-HR-workforce-analytics-sample" target="_blank" rel="noopener">
<img
    src="https://raw.githubusercontent.com/brittneybi/pbi-HR-workforce-analytics-sample/main/documentation/Page1-EXECUTIVE.png"
    alt="HR Workforce &amp; Compensation Analytics — Executive page"
    loading="lazy"
/>
<img
    src="https://raw.githubusercontent.com/brittneybi/pbi-HR-workforce-analytics-sample/main/documentation/Page2-HIRES&SEPARATIONS.png"
    alt="HR Workforce &amp; Compensation Analytics — Hires and Separations page"
    loading="lazy"
/>
<img
    src="https://raw.githubusercontent.com/brittneybi/pbi-HR-workforce-analytics-sample/main/documentation/Page3-COMPENSATION&RUN-RATE.png"
    alt="HR Workforce &amp; Compensation Analytics — Compensation and Run Rate page"
    loading="lazy"
/>
<img
    src="https://raw.githubusercontent.com/brittneybi/pbi-HR-workforce-analytics-sample/main/documentation/Page4-LOCATION.png"
    alt="HR Workforce &amp; Compensation Analytics — Locations page"
    loading="lazy"
/>
<img
    src="https://raw.githubusercontent.com/brittneybi/pbi-HR-workforce-analytics-sample/main/documentation/Page5-PERFORMANCE RATINGS.png"
    alt="HR Workforce &amp; Compensation Analytics — Performance Ratings Analysis page"
    loading="lazy"
/>
<img
    src="https://raw.githubusercontent.com/brittneybi/pbi-HR-workforce-analytics-sample/main/documentation/employee-drillthrough-page.png"
    alt="HR Workforce &amp; Compensation Analytics — Employee drillthrough page"
    loading="lazy"
/>
<img
    src="https://raw.githubusercontent.com/brittneybi/pbi-HR-workforce-analytics-sample/main/documentation/data-model.png"
    alt="HR Workforce &amp; Compensation Analytics — Data Model"
    loading="lazy"
/>
    <p>https://github.com/brittneybi/pbi-HR-workforce-analytics-sample</p></a>
    </div>
</div>

## 🧮 Key DAX
_Path: `/scripts/dax/measures.dax`_

### **Headcount**
```DAX
// Active Headcount (point-in-time)
Active Headcount = 
    VAR AnchorDate = MAX ( 'dimDate'[Date] )
    RETURN
    CALCULATE (
        DISTINCTCOUNT ( Employees[Employee ID] ),
        KEEPFILTERS (
            FILTER (
                ALL ( Employees[Hire Date], Employees[Exit Date] ),
                Employees[Hire Date] <= AnchorDate
                    && ( ISBLANK ( Employees[Exit Date] ) || Employees[Exit Date] >= AnchorDate )
            )
        )
    )

// New Hires 
New Hires =
    CALCULATE (    -- Count hires in the current filter period (month/quarter/year/rolling)
        DISTINCTCOUNT ( Employees[Employee ID] ),
        USERELATIONSHIP ( Employees[Hire Date], 'dimDate'[Date] )
    )

// Separations
Separations = 
    CALCULATE (  -- Count exits in the current filter period (inclusive of the exit date)
        DISTINCTCOUNT ( Employees[Employee ID] ),
        USERELATIONSHIP ( Employees[Exit Date], 'dimDate'[Date] )
    )

// FTE Headcount
FTE Headcount = 
    VAR AnchorDate = [_AsOfDate]
    VAR ActiveSet =
        FILTER (
            ALL ( Employees[Hire Date], Employees[Exit Date] ),
            Employees[Hire Date] <= AnchorDate
                && ( ISBLANK ( Employees[Exit Date] ) || Employees[Exit Date] >= AnchorDate )
        )
    RETURN
    SUMX (
        ActiveSet,
        DIVIDE ( COALESCE ( SUM(Employees[Typical Hours]), [Standard Weekly Hours] ), [Standard Weekly Hours] )
    )

// Average Headcount
Average Headcount = 
    VAR DatesInScope =
        CALCULATETABLE ( VALUES ( dimDate[Date] ), ALLSELECTED ( dimDate[Date] ) )
    RETURN
    AVERAGEX ( DatesInScope, CALCULATE ( [Active Headcount], dimDate[Date] ) )
```

### **Rates**
```DAX
Hire Rate % = DIVIDE ( [New Hires], [Average Headcount] )

Turnover Rate % = DIVIDE ( [Separations], [Average Headcount] )
```

### **Compensation**
```DAX
// Annual Compensation
Annual Compensation = 
    VAR YearStart = MINX ( STARTOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR YearEnd   = MAXX ( ENDOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR DaysInYear = DATEDIFF ( YearStart, YearEnd, DAY ) + 1
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR InWindow =
            Hire <= YearEnd && ( ISBLANK ( Exit ) || Exit >= YearStart )
        VAR Annualized =
            CALCULATE (
                IF (
                    SELECTEDVALUE ( Employees[Pay Type] ) = "Salary",
                    COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ),
                    COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 )
                        * COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 ) * 52
                )
            )
        VAR EffStart = MAX ( Hire, YearStart )
        VAR EffEnd   = MIN ( COALESCE ( Exit, YearEnd ), YearEnd )
        VAR ActiveDaysInYear =
            IF ( InWindow, MAX ( 0, DATEDIFF ( EffStart, EffEnd, DAY ) + 1 ), 0 )
        RETURN DIVIDE ( Annualized * ActiveDaysInYear, DaysInYear )
    )

// Annualized Pay (row-level normalization)
Annualized Compensation (Active) = 
    VAR d = MAX( dimDate[Date] )
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR Active =
            Hire <= d && ( ISBLANK ( Exit ) || Exit >= d )
        VAR Annualized =
            CALCULATE (
                IF (
                    SELECTEDVALUE ( Employees[Pay Type] ) = "Salary",
                    COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ),
                    COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 )
                        * COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 )
                        * 52
                )
            )
        RETURN IF ( Active, Annualized, 0 )
    )

// Monthly Compensation
Monthly Compensation = -- (Pro-Rated for exits and hires)
    VAR StartDate = MINX ( STARTOFMONTH ( dimDate[Date] ), dimDate[Date] )
    VAR EndDate   = MAXX(  ENDOFMONTH( dimDate[Date] ), dimDate[Date] )
    VAR YearStart = MINX ( STARTOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR YearEnd   = MAXX ( ENDOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR DaysInYear = DATEDIFF ( YearStart, YearEnd, DAY ) + 1
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR InWindow =
            Hire <= EndDate && ( ISBLANK ( Exit ) || Exit >= StartDate )
        VAR Annualized =
            CALCULATE (
                IF (
                    SELECTEDVALUE ( Employees[Pay Type] ) = "Salary",
                    COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ),
                    COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 )
                        * COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 ) * 52
                )
            )
        VAR EffStart = MAX ( Hire, StartDate )
        VAR EffEnd   = MIN ( COALESCE ( Exit, EndDate ), EndDate )
        VAR ActiveDaysInMonth =
            IF ( InWindow, MAX ( 0, DATEDIFF ( EffStart, EffEnd, DAY ) + 1 ), 0 )
        RETURN DIVIDE ( Annualized * ActiveDaysInMonth, DaysInYear )
    )

// Salary Run-Rate
Salary Run-Rate (Active) = 
    VAR d = MAX ( 'dimDate'[Date] )
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire     = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit     = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR IsSalary = CALCULATE ( SELECTEDVALUE ( Employees[Pay Type] ) = "Salary" )
        VAR Active   = Hire <= d && ( ISBLANK ( Exit ) || Exit >= d )
        VAR Annual   = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ) )
        RETURN IF ( Active && IsSalary, Annual, 0 )
    )

// Hourly Run-Rate
Hourly Run-Rate (Active) = 
    VAR d = MAX( dimDate[Date] )
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire     = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit     = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR IsHourly = CALCULATE ( SELECTEDVALUE ( Employees[Pay Type] ) = "Hourly" )
        VAR Active   = Hire <= d && ( ISBLANK ( Exit ) || Exit >= d )
        VAR Rate     = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 ) )
        VAR Hours    = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 ) )
        RETURN IF ( Active && IsHourly, Rate * Hours * 52, 0 )
    )

// Variance to Run-Rate 
Variance to Run-Rate (YTD) = 
    VAR StartDate = MINX ( STARTOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR EndDate   = MAX ( dimDate[Date] )
    VAR MonthsElapsed = DATEDIFF ( StartDate, EndDate, MONTH ) + 1   // Jan=1, Feb=2, ...
    RETURN
    [Annual Compensation]
        - DIVIDE ( [Annualized Compensation (Active)], 12 ) * MonthsElapsed
```
---
