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
    src="https://raw.githubusercontent.com/brittneybi/pbi-HR-workforce-analytics-sample/main/documentation/Page5-employee-drillthrough.png"
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

## 🧮 Key DAX (selected)
_Path: `/scripts/dax/measures.dax`_

```DAX
// *Measures Table
*Measures = 
    GENERATESERIES(1, 0)

// *Date Table
dimDate = 
    VAR FiscalStartMonth = 7  // 7 = July
    VAR FirstDateVal     = DATE(2020,1,1)
    VAR LastDateVal      = DATE(2025,12,31)

    // Tip: for rolling horizons, you can use:
    // VAR FirstDate = EDATE ( DATE(YEAR(TODAY())-6, 1, 1), 0 )   // ~6 years back
    // VAR LastDate  = EOMONTH ( TODAY(), 24 )                    // ~24 months forward

    RETURN
    ADDCOLUMNS (
        CALENDAR ( FirstDateVal, LastDateVal ),

        // ============================
        // Basic calendar attributes
        // ============================

        "Year",               YEAR ( [Date] ),
        "MonthNumber",        MONTH ( [Date] ),
        "MonthNameShort",     FORMAT ( [Date], "MMM" ),
        "MonthName",          FORMAT ( [Date], "MMMM" ),
        "DayOfMonth",         DAY ( [Date] ),
        "QuarterNumber",      QUARTER ( [Date] ),
        "QuarterLabel",       "Q" & QUARTER ( [Date] ),

        // Friendly composite labels

        "YearMonth",          FORMAT ( [Date], "yyyy-MM" ),   // text label for axes
        "YearMonthLabel",     FORMAT ( [Date], "MMM yyyy" ), // e.g., "Dec 2025"
        "YearQuarter",        "Q" & QUARTER ( [Date] ) & " " & YEAR ( [Date] ), // "Q4 2025"

        // ============================
        // Period start/end helpers
        // ============================
        "MonthStart",         DATE ( YEAR([Date]), MONTH([Date]), 1 ),
        "MonthEnd",           EOMONTH ( [Date], 0 ),
        "QuarterStart",
            DATE ( YEAR([Date]),
                ( (QUARTER([Date]) - 1) * 3 ) + 1,
                1 ),
        "QuarterEnd",
            EOMONTH (
                DATE ( YEAR([Date]),
                    ( (QUARTER([Date]) - 1) * 3 ) + 3,
                    1 ),
                0
            ),
        "YearStart",          DATE ( YEAR([Date]), 1, 1 ),
        "YearEnd",            DATE ( YEAR([Date]), 12, 31 ),

        // ============================
        // ISO week/year attributes (Mon-based)
        // ============================
        // ISOWeekNum: 1..53; ISOYear may differ from calendar year at year boundaries

        "ISOWeekNum",         WEEKNUM ( [Date], 21 ),  // 21 = ISO week, Monday start
        "ISOYear",
            YEAR ( [Date] + 4 - WEEKDAY ( [Date], 2 ) ) // standard ISO trick
            + ( MONTH([Date]) = 1 && WEEKNUM([Date], 21) > 50 ) 
            - ( MONTH([Date]) = 12 && WEEKNUM([Date], 21) = 1 ),
        "ISOYearWeek",
            VAR _isoY = YEAR ( [Date] + 4 - WEEKDAY ( [Date], 2 ) )
                + ( MONTH([Date]) = 1 && WEEKNUM([Date], 21) > 50 )
                - ( MONTH([Date]) = 12 && WEEKNUM([Date], 21) = 1 )
            VAR _isoW = WEEKNUM ( [Date], 21 )
            RETURN FORMAT ( _isoY, "0000" ) & "-W" & FORMAT ( _isoW, "00" ),
        "WeekStartMonday",    [Date] - WEEKDAY([Date],2) + 1,  // Monday
        "WeekEndSunday",      [Date] - WEEKDAY([Date],2) + 7,  // Sunday

        // ============================
        // Fiscal calendar (configurable FiscalStartMonth)
        // ============================

        "FiscalYear",
            YEAR ( EDATE ( [Date], 12 - FiscalStartMonth + 1 ) ),
        "FiscalMonthNumber",
            MOD ( MONTH([Date]) - FiscalStartMonth + 12, 12 ) + 1, // 1..12 starting at fiscal start
        "FiscalQuarterNumber",
            INT ( ( MOD ( MONTH([Date]) - FiscalStartMonth + 12, 12 ) ) / 3 ) + 1, // 1..4
        "FiscalYearLabel",
            "FY" & FORMAT (
                YEAR ( EDATE ( [Date], 12 - FiscalStartMonth + 1 ) ),
                "0000"
            ),
        "FiscalYearMonthNumber",
            ( YEAR ( EDATE ( [Date], 12 - FiscalStartMonth + 1 ) ) * 100 )
            + ( MOD ( MONTH([Date]) - FiscalStartMonth + 12, 12 ) + 1 ),
        "FiscalYearMonth",
            "FY" & YEAR ( EDATE ( [Date], 12 - FiscalStartMonth + 1 ) )
            & "-" & FORMAT ( MOD ( MONTH([Date]) - FiscalStartMonth + 12, 12 ) + 1, "00" ),
        "FiscalQuarter",
            "FQ" & (
                INT ( ( MOD ( MONTH([Date]) - FiscalStartMonth + 12, 12 ) ) / 3 ) + 1
            ) & " " &
            YEAR ( EDATE ( [Date], 12 - FiscalStartMonth + 1 ) ),
        "FiscalMonthStart",
            DATE (
                YEAR ( EDATE ( [Date], 12 - FiscalStartMonth + 1 ) ),
                MOD ( FiscalStartMonth + ( MOD ( MONTH([Date]) - FiscalStartMonth + 12, 12 ) ), 12 ) + 1,
                1
            ),
        "FiscalMonthEnd",
            EOMONTH (
                DATE (
                    YEAR ( EDATE ( [Date], 12 - FiscalStartMonth + 1 ) ),
                    MOD ( FiscalStartMonth +
                    ( MOD ( MONTH([Date]) - FiscalStartMonth + 12, 12 ) ), 12 ) + 1,
                    1
                ),
                0
            ),

        // ============================
        // Day-of-week & working-day flags
        // ============================

        "DayOfWeekNumber",    WEEKDAY ( [Date], 2 ),           // 1 = Monday .. 7 = Sunday
        "DayOfWeekName",      FORMAT ( [Date], "dddd" ),
        "IsWeekend",          IF ( WEEKDAY([Date],2) >= 6, TRUE(), FALSE() ),

        // Optional holiday flags (uncomment after adding a Holidays table)
        // Expect a separate table 'Holidays' with a column Holidays[Date]
        // "IsHoliday",          NOT ISEMPTY ( FILTER ( Holidays, Holidays[Date] = [Date] ) ),
        // "IsBusinessDay",
        //     VAR _hol = NOT ISEMPTY ( FILTER ( Holidays, Holidays[Date] = [Date] ) )
        //     RETURN IF ( _hol || WEEKDAY([Date],2) >= 6, FALSE(), TRUE() ),

        // ============================
        // Relative offsets (helpful for slicers/filters)
        // ============================

        "DateOffset_Days",    DATEDIFF ( [Date], TODAY(), DAY ),     // 0 = today
        "MonthOffset",        DATEDIFF ( EOMONTH([Date],0), EOMONTH(TODAY(),0), MONTH ),
        "QuarterOffset",
            DATEDIFF (
                DATE ( YEAR([Date]), (QUARTER([Date])-1)*3 + 1, 1 ),
                DATE ( YEAR(TODAY()), (QUARTER(TODAY())-1)*3 + 1, 1 ),
                QUARTER
            ),
        "YearOffset",         YEAR ( [Date] ) - YEAR ( TODAY() ),

        // ============================
        // Period membership flags (current selections relative to TODAY)
        // ============================

        "IsCurrentMonth",     IF ( EOMONTH([Date],0) = EOMONTH(TODAY(),0), TRUE(), FALSE() ),
        "IsCurrentQuarter",
            VAR _qs = DATE ( YEAR([Date]), (QUARTER([Date])-1)*3 + 1, 1 )
            VAR _qc = DATE ( YEAR(TODAY()), (QUARTER(TODAY())-1)*3 + 1, 1 )
            RETURN IF ( _qs = _qc, TRUE(), FALSE() ),
        "IsCurrentYear",      IF ( YEAR([Date]) = YEAR(TODAY()), TRUE(), FALSE() ),

        // Convenience flags for time intelligence visuals

        "IsMTD", IF ( [Date] >= DATE(YEAR(TODAY()), MONTH(TODAY()), 1) && [Date] <= TODAY(), TRUE(), FALSE() ),
        "IsQTD", IF ( [Date] >= DATE(YEAR(TODAY()), (QUARTER(TODAY())-1)*3 + 1, 1) && [Date] <= TODAY(), TRUE(),
                        FALSE()  ),
        "IsYTD", IF ( [Date] >= DATE(YEAR(TODAY()), 1, 1) && [Date] <= TODAY(), TRUE(), FALSE() ),

        // Fiscal YTD flag relative to today

        "IsFYTD",
            VAR _fyStart =
                DATE (
                    YEAR ( EDATE ( TODAY(), 12 - FiscalStartMonth + 1 ) ),
                    FiscalStartMonth,
                    1
                )
            RETURN IF ( [Date] >= _fyStart && [Date] <= TODAY(), TRUE(), FALSE() )
    )

// Helpers (example)
_AsOfDate = 
    VAR AnchorDate = MAX ( 'dimDate'[Date] )
    RETURN
        IF (
            NOT ISBLANK ( AnchorDate ),
            -- Anchor to end-of-month for consistency in month/quarter views
            EOMONTH ( AnchorDate, 0 ),
            -- Fallback when the page has no date filter at all
            EOMONTH ( TODAY (), 0 )
        )
Y := YEAR([_AsOfDate])

// Active Headcount (point-in-time)
Active Headcount =
    VAR AnchorDate = [_AsOfDate]
    RETURN
    CALCULATE (
        DISTINCTCOUNT ( Employees[Employee ID] ),
        KEEPFILTERS (
            FILTER (
                ALL ( Employees[Hire Date], Employees[Exit Date] ),
                Employees[Hire Date] <= AnchorDate
                    && ( ISBLANK ( Employees[Exit Date] ) ||
                            Employees[Exit Date] >= AnchorDate )
            )
        )
    )

// New Hires & Separations
New Hires = 
    -- Count hires in the current filter period (month/quarter/year/rolling)
    CALCULATE (
        DISTINCTCOUNT ( Employees[Employee ID] ),
        -- USERELATIONSHIP is optional but makes intent explicit and survives future model changes.
        USERELATIONSHIP ( Employees[Hire Date], 'dimDate'[Date] )
    )

Separations = 
    -- Count exits in the current filter period (inclusive of the exit date)
    CALCULATE (
        DISTINCTCOUNT ( Employees[Employee ID] ),
        USERELATIONSHIP ( Employees[Exit Date], 'dimDate'[Date] )
    )

// FTEs
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
        DIVIDE ( COALESCE ( SUM(Employees[Typical Hours]), [Standard Weekly Hours] ),
                    [Standard Weekly Hours] )
    )

// Annualized Compensation (Active)
Annualized Compensation (Active) =
    VAR d = [_AsOfDate]
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
                    SELECTEDVALUE ( Employees[Salary or Hourly] ) = "Salary",
                    COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ),
                    COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 )
                        * COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 )
                        * 52
                )
            )
        RETURN IF ( Active, Annualized, 0 )
    )

// Monthly Compensation (Prorated for exits and hires)
Monthly Compensation =
    VAR StartDate = [_MonthStart]
    VAR EndDate   = [_MonthEnd]
    VAR YearStart = DATE ( YEAR ( StartDate ), 1, 1 )
    VAR YearEnd   = DATE ( YEAR ( StartDate ), 12, 31 )
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
                    SELECTEDVALUE ( Employees[Salary or Hourly] ) = "Salary",
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

// Run‑Rates (Active)
Hourly Run-Rate (Active) =
    VAR d = [_AsOfDate]
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire     = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit     = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR IsHourly = CALCULATE ( SELECTEDVALUE ( Employees[Salary or Hourly] ) = "Hourly" )
        VAR Active   = Hire <= d && ( ISBLANK ( Exit ) || Exit >= d )
        VAR Rate     = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 ) )
        VAR Hours    = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 ) )
        RETURN IF ( Active && IsHourly, Rate * Hours * 52, 0 )
    )

Salary Run-Rate (Active) =
    VAR d = [_AsOfDate]
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire     = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit     = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR IsSalary = CALCULATE ( SELECTEDVALUE ( Employees[Salary or Hourly] ) = "Salary" )
        VAR Active   = Hire <= d && ( ISBLANK ( Exit ) || Exit >= d )
        VAR Annual   = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ) )
        RETURN IF ( Active && IsSalary, Annual, 0 )
    )

Total Compensation Run-Rate (Active) =
    [Hourly Run-Rate (Active)] + [Salary Run-Rate (Active)]

// Annual Compensation (Prorated, Full Year)
Annual Compensation =
    VAR YearStart = [_YearStart]
    VAR YearEnd   = [_YearEnd]
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
                    SELECTEDVALUE ( Employees[Salary or Hourly] ) = "Salary",
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
