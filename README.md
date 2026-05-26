<img width="1536" height="1024" alt="746937c2a77947e9cd68e983d84d1ffb" src="https://github.com/user-attachments/assets/97768df7-13d1-4f00-95c0-931295a05c59" />

# PlantCo Quantity Performance Analysis

Power BI Dashboard — Dynamic YTD vs PYTD Performance Report.

# Background

This project was completed for PlantCo to give stakeholders a clear, actionable view of commercial performance across markets, products, and customer accounts. The core business need was a compact, interactive YTD vs PYTD performance dashboard that surfaces underperforming countries and accounts, highlights margin performance, and lets users switch instantly between Sales, Gross Profit, and Quantity.

**Dataset (summary)**

* Source: PlantCo_data.xlsx (Excel workbook).
* Period covered: January 2022 → April 2024.

**Main tables:**

* Fact_Sales — transactional invoice-line data (Invoice_Date, Product_ID, Account_ID, Sales, Quantity, COGS).
* Dim_Account — one row per customer account (country, geographic coords, postal/street fields).
* Dim_Product — product hierarchy (Product_Family → Product_Group → Product_Name → Product_Size → Product_Type).

**Model additions:**

* Dim_Date — DAX calendar (2022–2024) with an In_Past flag to correctly compute PYTD for partial years.
* Slicer_Values — manual table with rows Sales, Gross Profit, Quantity to power SWITCH measures.

---

# Project narrative

*Objective*

Build a dynamic Power BI report that answers: Which countries/products/accounts are underperforming vs prior year, how are monthly trends evolving, which accounts are high-margin vs high-value, and what is the overall GP% profile?

*Approach (4-phase workflow)*

# Data preparation (Power Query)

Loaded the three sheets from the Excel workbook, standardized table names (Fact_Sales, Dim_Account, Dim_Product), fixed types (Invoice_Date → Date), removed obvious duplicates and redundant columns, and trimmed noise (e.g., duplicate latitude/country columns).

# Data modelling

* Created Dim_Date using CALENDAR() and added an In_Past boolean to avoid misleading PYTD comparisons for months not yet present in the current year.
* Created Slicer_Values to let users pick the metric shown across visuals.
* Built relationships: Fact_Sales[Invoice_Date] → Dim_Date[Date], Fact_Sales[Account_ID] → Dim_Account[Account_ID], and Fact_Sales[Product_ID] → Dim_Product[Product_Name].
* Organized measures into logical folders (Base, YTD, PYTD, Switches, Comparison, Titles) to keep the model tidy for report consumers.

# DAX measure development

1 Base measures: Sales, Quantity, COGS, Gross Profit, GP%.

* Time-intelligence *:
* *YTD_* measures using TOTALYTD() (anchored to Invoice_Date so YTD stops at the latest invoice).*
* *PYTD_* using SAMEPERIODLASTYEAR() combined with the In_Past flag to prevent comparing to empty months.*

2 Switch logic: SelectedMetric = SELECTEDVALUE(Slicer_Values[Value]) then SWITCH() to return the corresponding YTD or PYTD measure across visuals (keeps report consistent when the user toggles metric).

3 Comparison measures: YTD_vs_PYTD = Switch_YTD - Switch_PYTD and percentage change calculations for conditional formatting.

4 Dynamic titles: small DAX text measures to keep charts readable and context-aware.

# Report design & visuals

* Canvas styled with a PowerPoint-exported background for a professional, consistent look.
* Header KPIs (cards) for YTD, PYTD, variance, and GP% (conditional color formatting: green for positive, red for negative).
 
* *Core visuals:*
* *Treemap showing bottom 10 countries by YTD vs PYTD to quickly identify trouble markets.*
* *Waterfall chart illustrating month-by-month variance contributions (drillable by country/product).*
* *Line + stacked columns plotting YTD (columns) vs PYTD (line).*
* *Scatter chart with a zoom slider to segment accounts by YTD value (x) and GP% (y), including benchmark lines to identify high-margin/low-volume upsell opportunities.*

* UX considerations: enforce single-select on the values slicer to avoid ambiguous SWITCH outputs; recommend single-year selection for clear PYTD comparisons.

# Techniques & why they were chosen

* Power Query for reliable cleaning and repeatable ETL.
* DAX TOTALYTD() / SAMEPERIODLASTYEAR() plus an In_Past flag to handle partial-year data accurately — this prevents misleading PYTD numbers early in the year.
* SWITCH-driven measures to give users immediate, consistent metric changes across all visuals without duplicating visuals or measures.
* Conditional formatting and dynamic titles to reduce cognitive load for non-technical stakeholders.

# Interesting findings / surprises during analysis

* Canada surfaced as the largest contributor to the gross profit decline (not just sales decline), driven by a weakness in the Landscape product type in March–April 2024 — a specific, time-bound insight that suggests product- and month-level causes.
* February 2024 outperformed the prior year, indicating something replicable (promotion, channel change, or product mix) worth investigating.
* The account landscape is highly skewed: many accounts cluster at common volume levels while a small set drive most invoice volume ideal for targeted account management and resource prioritization.
* Identification of a high-margin, low-volume customer segment revealed clear upsell opportunities that are low-risk (high margin) and high-reward.

---

# Conclusions

# Key takeaways

* The dynamic YTD vs PYTD dashboard meets PlantCo’s need for a concise, actionable performance view across countries, products, and accounts.
* The In_Past flag plus careful time-intelligence measures produced trustworthy PYTD comparisons despite partial-year data.
* The SWITCH-driven UX (Sales / Gross Profit / Quantity) allows non-technical stakeholders to explore multiple lenses without changing the report structure.
* Concrete commercial actions are visible: targeted reviews for bottom countries (Canada, Colombia, Croatia, Germany), investigation into February marketing & product mix, and an upsell program for high-margin, low-volume accounts.

# How to iterate & improve

* Add SKU-level and channel-level data (if available) to diagnose whether declines are product-specific or channel-driven.
* Enrich account data with customer tier, industry, and contact owner to translate insights into CRM actions (targeted outreach, promotions).
* Automate data refresh via Power BI Service gateway and schedule to provide near-real-time monitoring and remove manual update steps.
* Add anomaly detection / alerts for sudden month-on-month GP% drops so commercial teams can react quickly.
* Create action pages: a recommended-actions page listing top 10 accounts/countries with suggested next steps (promotions, pricing review, supply checks).
* Performance tuning: if the dataset grows, migrate heavy transformations into a database or Azure Data Factory for faster refresh and model performance.
