# Collection & Financial Risk Intelligence Dashboard

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

Measures are grouped in a dedicated measure table per project (for example `Lake Front Dax`), plus `Global Summary Dax`, `Collection Dax`, and `Measurse HTML`. The tables below use **Lake Front 6** as the example; the other 7 projects follow the same pattern with their own table name and suffix (for example `Sold Rihana`, `Collection Rate Skyline`). Most lifecycle measures are wrapped in `VAR Result = ... RETURN IF(ISBLANK(Result), 0, Result)` so blanks show as 0; it is omitted below for readability.

### Sales & Project Metrics

| Measure | DAX Expression | Purpose |
|---------|----------------|---------|
| **Adopted Total Units Lake** | `1203` | Fixed total units of the project, used as the Sold % denominator. |
| **Sold Lake** | `CALCULATE(DISTINCTCOUNT('Lake Front 6'[رقم الوحدة]), 'Lake Front 6'[اسم العميل] <> "Summary Total", 'Lake Front 6'[اسم العميل] <> "Paid Installments", 'Lake Front 6'[اسم العميل] <> "Remaining Installments")` | Counts real customers only, excluding the 3 control rows. |
| **Sold % Lake** | `DIVIDE([Sold Lake], [Adopted Total Units Lake], 0)` | Sold units as a share of the project's total units. |
| **Project Completion % Lake** | `CALCULATE(AVERAGE('Lake Front 6'[POC]), 'Lake Front 6'[اسم العميل] <> "Summary Total", 'Lake Front 6'[اسم العميل] <> "Paid Installments", 'Lake Front 6'[اسم العميل] <> "Remaining Installments")` | Average construction progress across customers. |

### Installment Lifecycle

| Measure | DAX Expression | Purpose |
|---------|----------------|---------|
| **Issued Invoices Lake** | `CALCULATE(COUNTROWS('Lake Front 6'), 'Lake Front 6'[Installment State] <> "NOT DUE")` | Installments that are payable now (paid or overdue). |
| **Collected Invoices Lake** | `CALCULATE(COUNTROWS('Lake Front 6'), 'Lake Front 6'[Installment State] = "PAID")` | Installments already paid. |
| **Outstanding Invoices Lake** | `[Issued Invoices Lake] - [Collected Invoices Lake]` | Payable but unpaid installments. |
| **Collection Rate Lake** | `DIVIDE([Collected Invoices Lake], [Issued Invoices Lake], 0)` | Share of issued invoices that were collected. |
| **Outstanding % Lake** | `DIVIDE([Outstanding Invoices Lake], [Issued Invoices Lake], 0)` | Share of issued invoices still unpaid. |
| **Collected Amount Lake** | `CALCULATE(SUM('Lake Front 6'[Amount]), 'Lake Front 6'[Installment State] = "PAID")` | Money collected. |
| **Total Invoice Amount Lake** | `CALCULATE(SUM('Lake Front 6'[Amount]), 'Lake Front 6'[Installment State] <> "NOT DUE")` | Total demand issued so far. |
| **Outstanding Amount Lake** | `[Total Invoice Amount Lake] - [Collected Amount Lake]` | Money overdue. |

### Bank vs. Cash Exposure

| Measure | DAX Expression | Purpose |
|---------|----------------|---------|
| **Bank Outstanding Lake** | `CALCULATE(SUM('Lake Front 6'[Amount]), 'Lake Front 6'[اسم العميل] <> "Summary Total", 'Lake Front 6'[اسم العميل] <> "Paid Installments", 'Lake Front 6'[اسم العميل] <> "Remaining Installments", 'Lake Front 6'[Installment State] = "DUE - NOT PAID", 'Lake Front 6'[Payment Type] = "Bank")` | Overdue amount owed through banks. |
| **Cash Outstanding Lake** | Same as above with `'Lake Front 6'[Payment Type] = "Cash"` | Overdue amount owed by cash customers. |
| **Outstanding Balance Lake** | `[Bank Outstanding Lake] + [Cash Outstanding Lake]` | Total overdue balance. |

### Time Cutoff Segmentation (Old vs. New)

The cutoff dates are hardcoded inside each measure.

| Measure | DAX Expression | Purpose |
|---------|----------------|---------|
| **Collected OLD Lake Front** | `CALCULATE(SUM('Lake Front 6'[Amount]), 'Lake Front 6'[Installment State] = "PAID", 'Lake Front 6'[Date] < DATE(2026,7,1))` | Collections before the start date. |
| **Collected NEW Lake Front** | `CALCULATE(SUM('Lake Front 6'[Amount]), 'Lake Front 6'[Installment State] = "PAID", 'Lake Front 6'[Date] >= DATE(2026,7,1), 'Lake Front 6'[Date] <= DATE(2026,8,3))` | Collections inside the date range, inclusive. |
| **Outstanding OLD Lake Front** | `CALCULATE(SUM('Lake Front 6'[Amount]), 'Lake Front 6'[Installment State] = "DUE - NOT PAID", 'Lake Front 6'[Date] < DATE(2026,7,1))` | Overdue amounts before the start date. |
| **OLD Rows Lake Front** | `CALCULATE(DISTINCTCOUNT('Lake Front 6'[اسم العميل]), 'Lake Front 6'[Date] < DATE(2026,7,1), NOT ISBLANK('Lake Front 6'[اسم العميل]))` | Customers with records before the start date. |

`Outstanding NEW` and `NEW Rows` follow the same pattern with the inclusive date range.

### Global Summary & Cross-Project Measures

| Measure | DAX Expression | Purpose |
|---------|----------------|---------|
| **Total Sold** | `[Sold Kattameya] + [Sold Zahra] + [Sold Degla Landmark] + [Sold Skyline] + [Sold Degla Palms] + [Sold Lake] + [Sold Crystal] + [Sold Rihana]` | Sold customers across all 8 projects. |
| **Sold %** | `DIVIDE([Total Sold], [Adopted Total Units], 0)` | Portfolio sold rate. |
| **Total Issued Invoices** | `[Issued Invoices Kattameya] + [Issued Invoices Zahra] + ... + [Issued Invoices Rihana]` | Issued invoices across all projects. |
| **Total Collected Invoices** | `[Collected Invoices Kattameya] + [Collected Invoices Zahra] + ... + [Collected Invoices Rihana]` | Collected invoices across all projects. |
| **Collection Rate** | `DIVIDE([Total Collected Invoices], [Total Issued Invoices], 0)` | Portfolio collection rate. |
| **Total Collected Amount** | `[Collected Amount Kattameya] + [Collected Amount Zahra] + ... + [Collected Amount Rihana]` | Total money collected. |
| **Total Outstanding Balance** | `[Outstanding Balance Kattameya] + [Outstanding Balance Zahra] + ... + [Outstanding Balance Rihana]` | Total overdue balance. |
| **Collected OLD by Project** | `SWITCH(SELECTEDVALUE(Project[Project Name]), "One Kattameya Compound", [Collected OLD Kattameya], "Zahra North Coast", [Collected OLD Zahra], ... , "Rihana", [Collected OLD Rihana], <sum of all 8 projects>)` | Shows the selected project, or the portfolio total when nothing is selected. |

### HTML Measures (Custom KPI Cards)

| Measure | Purpose |
|---------|---------|
| **HTML Sold %**, **HTML Collection Rate**, **HTML Completion**, **HTML Outstanding %** | Build the animated KPI cards on the Global Summary page. Each one assembles an HTML string (title, `value / total`, and a progress bar with a CSS animation) from the underlying measures and renders it in the HTML Content visual. |
| **معمار المرشدي** | Home page title with a glowing, fading text effect written in HTML and CSS. |

**Why it is written this way**
- `DIVIDE` returns the alternate result (0) instead of an error when the denominator is zero.
- `CALCULATE` changes the filter context, which is how each measure isolates one state, one payment type, or one time window.
- The 3 control rows stay in the table and are excluded inside the sales and bank/cash measures, so they never count as customers.
- Separate per-project measures keep every project page independent, and the `SWITCH` measures bring them together on the global page.

---

## 5. Report Architecture

- **Home / Navigation** page
- **Global pages:** Global Summary, and Collection By Date Summary (old vs. new comparison)
- **Project pages:** 4 pages per project — Overall, Cash, Bank, Bank-wise

Every analytical page carries a Home button, a Project Completion % indicator, and a No. of Units card.

The report contains **35 pages**: Home, Global Summary, Collection By Date Summary, and 32 project pages (Overall, Cash, Bank, and Bank-wise for each of the 8 projects).

**Custom HTML visuals:** the report uses the *HTML Content* custom visual. HTML measures (`HTML Sold %`, `HTML Collection Rate`, `HTML Completion`, `HTML Outstanding %`) build the styled KPI cards on the Global Summary page, and HTML also styles the company name "معمار المرشدي" (Memar Al Morshedy) on the Home page.

**Model organization:** the measures are grouped in dedicated measure tables — one per project (for example `Kattameya Dax`, `Lake Front Dax`), plus `Collection Dax` (old vs. new), `Global Summary Dax`, and `Measurse HTML`. Measure names follow the pattern `<Measure> <Project>` (for example `Collection Rate Rihana`).

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

└── README.md
---

## Author

**Amira Mohamed Ali Abdelhamid**
Data Analyst | Tanta, Egypt
GitHub: [@amira-mohamed05](https://github.com/amira-mohamed05)
