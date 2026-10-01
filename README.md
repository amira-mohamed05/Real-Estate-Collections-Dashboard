## 📁 Project Files

| File | Description |
|------|-------------|
| 📊 [Graduation Project.pbix](Graduation%20Project.pbix) | Power BI report (35 pages) |
| 📗 [Graduation Project Data Source.xlsm](Graduation%20Project%20Data%20Source.xlsm) | Source data (8 project worksheets) |
**Enterprise Real Estate Financial Surveillance, Installment Lifecycle DAX Engine & Bank Risk Analytics**

An interactive **Power BI** dashboard that tracks installment collections and credit risk across **8 large-scale real estate projects**, showing what has been collected, what is overdue, and whether the risk sits with banks or with cash customers.

> Graduation project — Data Analysis Diploma, Route Academy
> *Uncovering the Stories Hidden Beneath the Data*

---

## The Portfolio in Numbers

| Total Portfolio Units | Total Active Sold | Sold Rate | Invoiced Demand |
|:---:|:---:|:---:|:---:|
| **73,943** | **3,088** | **4.18%** | **13.88B EGP** |

- **Collected:** 13.09B EGP (~94% of the invoiced demand)
- **Overdue split:** Bank **521.9M EGP** (66%) | Cash **269.1M EGP** (34%)

**Global Summary KPIs (invoice level)**

| Sold % | Collection Rate | Completion % | Outstanding % |
|:---:|:---:|:---:|:---:|
| **4.18%** (3,088 / 73,943) | **89.86%** (12,241 / 13,622) | **70.55%** | **10.14%** (1,381 / 13,622) |

---

## 1. Business Problem

| Problem | Description |
|---------|-------------|
| **Disparate worksheets & inconsistent plans** | The developer manages 8 compounds across 8 separate Excel workbooks with different payment schedules, which fragments the data. |
| **Metric distortions from embedded totals** | The source sheets contain footer summary rows (*Summary Total*, *Paid Installments*, *Remaining Installments*) inside the customer tables, which distorts record counts and averages. |
| **Installment lifecycle blind spots** | There was no way to separate aged delinquencies from future dues. The wide layout (Installments 1-7 as columns) prevented time-series aging and lifecycle tracking. |
| **Obscured credit risk (Bank vs. Cash)** | Management could not separate the credit risk of mortgage banks from the risk of direct in-house cash financing, which delayed legal and collection workflows. |

---

## 2. Project Mission & Objectives

Build a **100% reconciled Power BI decision-support platform** that ingests the raw project schedules, applies an installment lifecycle model in DAX, and isolates the exposure across banks and cash customers.

- **Standardize ingestion:** an automated Power Query pipeline that separates control rows and normalizes column headers.
- **3-state lifecycle modeling:** classify every installment as `PAID`, `DUE - NOT PAID`, or `NOT DUE`.
- **Time-cutoff analytics (Old vs. New):** compare historic collections with newly matured dues in any date window.
- **Multi-level navigation:** drill from portfolio KPIs down to a single project, payment type, and bank.

### Projects Covered

One Kattameya Compound · Zahra North Coast · Degla Landmark · Skyline Katamya Compound · Degla Palms 6 October Compound · Lake Front 6 · Crystal Plaza Maadi Compound · Rihana

---

## 3. Data Transformation Pipeline (Power Query)

| Step | Action |
|------|--------|
| **1. Source normalization** | Connected to all 8 project workbooks, enforced a standard **21-column schema**, and standardized text encodings, contract numbers, and financing institution names. |
| **2. Control row isolation** | Separated the final 3 control rows from the customer data, leaving **3,088 authentic customer contracts**. |
| **3. Dimensional unpivoting** | Unpivoted *Installment 1 to 7* into vertical rows indexed by Customer ID, Unit Number, Installment Number, and Raw Value. |
| **4. Business state categorization** | Added conditional columns mapping each installment to `PAID` (collected), `DUE - NOT PAID` (overdue), or `NOT DUE` (future maturity). |

```
8 Project Workbooks (Excel)
        ↓
Source Normalization (21-column schema)
        ↓
Control Row Isolation (3 rows)
        ↓
Unpivot Installments 1-7
        ↓
3-State Classification
        ↓
Power BI Data Model + DAX Measures
        ↓
Interactive Dashboard
```

---

## 4. DAX Measures

Each project has its own table. In the examples below, replace `'Project'` with the table name and adjust column names, total units, and dates to match your model.

### Sales & Project Metrics

| Measure | DAX Expression | Purpose |
|---------|----------------|---------|
| **# Sold** | `CALCULATE(DISTINCTCOUNT('Project'[Unit Number]), NOT('Project'[Customer Name] IN {"Summary Total","Paid Installments","Remaining Installments"}))` | Counts real customers only, excluding control rows. |
| **Sold %** | `DIVIDE([# Sold], <Total Units>, 0)` | Sold units as a share of the project's fixed total units. |
| **Project Completion %** | `AVERAGE('Project'[POC])` | Average construction progress. |

### Installment Lifecycle

| Measure | DAX Expression | Purpose |
|---------|----------------|---------|
| **Issued Invoices** | `CALCULATE(COUNTROWS('Project'), 'Project'[State] IN {"PAID","DUE - NOT PAID"})` | Installments that are payable now. |
| **Collected Invoices** | `CALCULATE(COUNTROWS('Project'), 'Project'[State] = "PAID")` | Installments already paid. |
| **Outstanding Invoices** | `[Issued Invoices] - [Collected Invoices]` | Payable but unpaid installments. |
| **Collection Rate** | `DIVIDE([Collected Invoices], [Issued Invoices])` | Share of issued invoices that were collected. |
| **Outstanding %** | `DIVIDE([Outstanding Invoices], [Issued Invoices])` | Share of issued invoices still unpaid. |
| **Collected Amount** | `CALCULATE(SUM('Project'[Amount]), 'Project'[State] = "PAID")` | Money collected. |
| **Outstanding Amount** | `CALCULATE(SUM('Project'[Amount]), 'Project'[State] = "DUE - NOT PAID")` | Money overdue. |
| **Invoiced Amount** | `[Collected Amount] + [Outstanding Amount]` | Total demand issued. |

### Time Cutoff Segmentation (Old vs. New)

The cutoff dates are hardcoded inside each measure. Change them to your own start and end dates.

| Measure | DAX Expression | Purpose |
|---------|----------------|---------|
| **Collected OLD** | `CALCULATE([Collected Amount], 'Project'[Date] < DATE(2026,7,1))` | Collections before the start date. |
| **Collected NEW** | `CALCULATE([Collected Amount], 'Project'[Date] >= DATE(2026,7,1), 'Project'[Date] <= DATE(2026,8,3))` | Collections inside the date range, inclusive. |
| **Outstanding OLD** | `CALCULATE([Outstanding Amount], 'Project'[Date] < DATE(2026,7,1))` | Overdue amounts before the start date. |
| **Outstanding NEW** | `CALCULATE([Outstanding Amount], 'Project'[Date] >= DATE(2026,7,1), 'Project'[Date] <= DATE(2026,8,3))` | Overdue amounts inside the date range. |

The same pattern is repeated with `COUNTROWS` for the `OLD Rows` and `NEW Rows` measures.

**Why it is written this way**
- `DIVIDE` returns BLANK (or the alternate result) instead of an error when the denominator is zero.
- `CALCULATE` changes the filter context, which is how each measure isolates one state or one time window.
- Control rows are excluded in the sales measures so they never count as customers.

---

## 5. Report Architecture

- **Home / Navigation** page
- **Global pages:** Global Summary, and Collection By Date Summary (old vs. new comparison)
- **Project pages:** 4 pages per project — Overall, Cash, Bank, Bank-wise

Every analytical page carries a Home button, a Project Completion % indicator, and a No. of Units card.

**Custom HTML visuals:** HTML was used on the Home page to style the company name "معمار المرشدي" (Memar Al Morshedy) with a glowing title effect, and the same branding appears as a custom card on the Global Summary page.

---

## 6. Key Insights

- **Collection is strong:** about 94% of the invoiced demand has been collected (13.09B of 13.88B EGP).
- **Invoice-level view:** 12,241 of 13,622 issued invoices were collected (89.86%), leaving 1,381 invoices (10.14%) outstanding.
- **Banks hold most of the overdue exposure:** 66% of overdue amounts sit with banks (521.9M EGP) versus 34% with cash customers (269.1M EGP), so follow-up should prioritize bank relationships.
- **Low sold rate:** only 4.18% of the 73,943 portfolio units are sold, which points to a large sales opportunity.

> Add more findings from your final dashboard here.

---

## Tools & Skills

**Power BI** · **Power Query** · **DAX** · **Excel** · **HTML** · Data Modeling · Dashboard Design · UI/UX

## Author

**Amira Mohamed Ali Abdelhamid**
Data Analyst | Tanta, Egypt
GitHub: [@Amia-Mohamed05](https://github.com/Amia-Mohamed05)
