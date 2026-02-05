---
title: HR Workforce & Compensation Analytics
description: A focused Power BI sample for workforce and compensation analysis using synthetic data.
---

# HR Workforce & Compensation Analytics — Power BI Sample

This project showcases practical HR analytics in Power BI: **point‑in‑time headcount**, **salary vs. hourly normalization**, **run‑rate compensation**, and **prorated monthly/annual costs**—all on synthetic employee data.

> **Data:** ~1K rows, ~20 columns (hire/exit dates, pay type, annual/hourly, typical hours, department, job, location, performance rating, status).  
> **Privacy:** No real or proprietary data.

---

## 🔎 What’s inside

- **Power BI report:** `pbix/HR-Workforce-Analytics-Report.pbix`  
- **Dataset (Excel):** `data/fake_hr_dataset.xlsx`  
- **DAX measures:** `scripts/dax/measures.dax`  
- **Docs & images:** `documentation/`  
  - `data-model.png` – model diagram  
  - `Page1-EXECUTIVE.png` – report overview
  - `Page2-HIRES&SEPARATIONS.png` – new hires & separations (terminations/resignations)
  - `Page3-COMPENSATION&RUN-RATE.png` – financial trends and key metrics
  - `Page4-LOCATION.png` – employee metrics by location
  - `Page5-PERFORMANCE RATINGS.png` – analysis of performance ratings
  - `employee-drillthrough-page.png` – drill-through page to view table of detailed employee data
  - `methodology.md` – approach & assumptions

---

## 📊 Report highlights

- **Workforce Summary:** Headcount (as‑of), hires/exits, avg tenure, avg annualized comp  
- **Compensation Intelligence:** Annualized comp (active), monthly comp, salary/hourly run‑rate, YTD variance to run‑rate
- **Hire/Exit & Lifecycle:** Exit reasons, tenure cohorts, location and performance views

---

## 🧮 DAX measures

All key measures live in **`/scripts/dax/measures.dax`** (ready to paste into your model).  
They include row‑level **Annualized Pay**, point‑in‑time **Active Headcount**, **Monthly Compensation**, and **Salary/Hourly Run‑Rate**.

---

## 🚀 Quick start

1. Open **Power BI Desktop** and load `data/fake_hr_dataset.xlsx`.  
2. Create (or confirm) a **Date** table if you need as‑of filters.  
3. Paste the measures from **`/scripts/dax/measures.dax`** into your model.  
4. Save the report as **`pbix/HR-Workforce-Analytics-Report.pbix`**.  
5. Review the documentation images in **`/documentation`** and replace placeholders as needed.

---

## 🔗 Handy links

- **README:** [./README.md](./README.md)  
- **Measures:** [./scripts/dax/measures.dax](./scripts/dax/measures.dax)  
- **Methodology:** [./documentation/methodology.md](./documentation/methodology.md)

---

*Tip: You can publish this site with GitHub Pages using the repo root as the Pages source.*
