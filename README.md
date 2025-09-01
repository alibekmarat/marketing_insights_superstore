# marketing_insights_superstore
Project on Power BI that uses the Superstore dataset to deliver sales &amp; marketing insights. Focus: discount efficiency, customer loyalty, profit concentration etc.

**STEP 1** Project Goals 

Audience: **Head of Sales & Marketing** – responsible for driving revenue growth, optimising discount strategy, and improving customer loyalty.

Business Context: the company’s sales are growing, but questions remain about whether discounting, product mix, and customer engagement are truly sustainable. To support sales and marketing decisions, we need insights into the effectiveness of current strategies and guidance on where to focus future campaigns.

Main Business Questions:
1. **Customer Retention vs Acquisition**: Are we getting repeat purchases from customers, or relying too much on one-time buyers?
2. **Regional Growth Opportunities**: Which regions show untapped potential for growth, and where are marketing efforts failing?

Project Goal  is to deliver actionable insights for the Head of Sales & Marketing to strengthen customer retention and loyalty and identify regions with the highest growth potential.

**STEP 2** 📊 Used Dataset

1. **Source:** Kaggle – Superstore Sales Dataset:.
2. **Size:** 9,800 rows  and 18 columns.
3. **Coverage:** Customer orders across the United States.
4. **Key Fields:**
  * Customer: `Customer ID`, `Customer Name`, `Segment`
  * Orders: `Order ID`, `Order Date`, `Ship Date`, `Ship Mode`
  * Geography: `Region`, `State`, `City`, `Postal Code`
  * Products: `Category`, `Sub-Category`, `Product ID`, `Product Name`
  * Transactions: `Sales`

There is no Discount and Profit sections, so might limit the financial analysis(not critical to current Business Questions).

**STEP 3** Dataset Preprocessing and Audit
Missing Values: `Postal Code` has missing values (11 rows). No duplicates.
* No invalid shipments (all Ship Dates ≥ Order Dates).
 5. Sales Distributions
* **Min:** $0.44
* **Max:** $22,638.48
* **Mean:** $230.77

Regional Coverage (U.S)
* West: 3,140 orders
* East: 2,785 orders
* Central: 2,277 orders
* South: 1,598 orders

Customer activity
* **Unique Customers:** 793
* **Average Orders per Customer:** \~6.2
* **Most Active Customer:** 17 orders


**STEP 4** Star Schema 

The dataset was transformed into a star schema in Power BI. This separates transactional facts from descriptive dimensions, improving performance and making business questions easier to answer.

Fact Table
*	FactSales: Order-level data (Order ID, Customer ID, Product ID, Postal Code, Ship Mode, Order Date, Ship Date, Sales).

Dimension Tables
*	DimCustomer: Customer ID, Name, Segment.  
*	DimProduct: Product ID, Name, Sub-Category, Category.
*	DimGeo: Postal Code, City, State, Region.
*	DimShipMode: Shipping modes.
*	DimDate: Calendar table generated with DAX for time intelligence.

Relationships
*	One-to-many (1:*) relationships from each Dimension to FactSales.
*	Active link: DimDate[Date] → FactSales[Order Date].
*	Inactive link (for flexibility): DimDate[Date] → FactSales[Ship Date] (activated in DAX with USERELATIONSHIP).
*	All relationships set to single direction (Dim ➜ Fact) to avoid ambiguity.

Outcome
*	Clean separation of business entities.
*	Enables analysis of customer retention and regional growth with efficient slicing by customer, product, geography, and time.


**STEP 5** — Transformations in Power Query

Goal is make the model clean, consistent, and business-ready.

* Set data types

  * `Order Date`, `Ship Date` → **Date** (Using Locale: **English (UK)** for `dd/mm/yyyy`)
  * `Sales` → **Decimal** (renamed **Sales USD**)
  * IDs/Geo → **Text** (incl. `Postal Code`)
* Created columns in **FactSales**

  * **OrderToShipDays** = `Ship Date – Order Date` (Add Column ▸ Date ▸ Subtract Days)
  * **First Purchase Date (Col)** via **Merge** (grouped min Order Date per Customer and left-joined back)
  * **Months Since First Purchase** (custom column):

    ```
    (Date.Year([Order Date]) - Date.Year([First Purchase Date (Col)])) * 12
      + (Date.Month([Order Date]) - Date.Month([First Purchase Date (Col)]))
    ```
  * **Cohort Month** (Add Column ▸ Date ▸ Month ▸ Start of Month), formatted as `YYYY-MM`
* Text cleanup: Trim, Clean, Capitalize (City/State/Names)
* Geography choice (pragmatic): use **Region** directly from **FactSales**
  *(Postal Code had nulls; this keeps visuals accurate and removes `(Blank)` categories.)*

**Quick QA after Apply**

* Cards: **Sales USD ≈ 2.26M**, **Orders ≈ 4.9K**, **Customers = 793**
* Chart: Sales by **Region** shows 4 regions, no `(Blank)`

# Step 6 — Date Table & Relationships

* **DimDate** (DAX):

  ```DAX
  DimDate =
  ADDCOLUMNS (
      CALENDAR ( DATE(2015,1,1), DATE(2019,12,31) ),
      "Year", YEAR([Date]),
      "MonthNo", MONTH([Date]),
      "Month", FORMAT([Date], "MMM"),
      "YearMonth", FORMAT([Date], "YYYY-MM"),
      "Quarter", "Q" & FORMAT([Date], "Q"),
      "Week", WEEKNUM([Date],2),
      "IsWeekend", WEEKDAY([Date],2) >= 6
  )
  ```
* Mark **DimDate** as Date table (column `Date`)
* Relationships (single direction Dim ➜ Fact):

  * `DimDate[Date] → FactSales[Order Date]` (active)
  * `DimCustomer[Customer ID] → FactSales[Customer ID]`
  * `DimProduct[Product ID] → FactSales[Product ID]`
  * `DimShipMode[Ship Mode] → FactSales[Ship Mode]`
* Sorting: `DimDate[YearMonth]` **Sort by** `MonthNo`
* (Optional) Turn off **Auto Date/Time** (Options ▸ Data Load)


# Step 7 — Core KPI Measures (DAX)

Create these in **FactSales**.

```DAX
Sales USD := SUM('FactSales'[Sales USD])
Orders    := DISTINCTCOUNT('FactSales'[Order ID])
Customers := DISTINCTCOUNT('FactSales'[Customer ID])
AOV       := DIVIDE([Sales USD], [Orders])
```

**New vs Returning**

```DAX
First Purchase Date :=
CALCULATE(
  MIN('FactSales'[Order Date]),
  ALLEXCEPT('FactSales','FactSales'[Customer ID])
)

New Customers :=
CALCULATE(
  DISTINCTCOUNT('FactSales'[Customer ID]),
  FILTER(VALUES('FactSales'[Customer ID]),
    [First Purchase Date] >= MIN('DimDate'[Date]) &&
    [First Purchase Date] <= MAX('DimDate'[Date])
  )
)

Returning Customers := [Customers] - [New Customers]

Sales USD (New) :=
CALCULATE([Sales USD],
  FILTER(VALUES('FactSales'[Customer ID]),
    [First Purchase Date] >= MIN('DimDate'[Date]) &&
    [First Purchase Date] <= MAX('DimDate'[Date])
  )
)

Sales USD (Returning) := [Sales USD] - [Sales USD (New)]
```

**Regional share & rank**

```DAX
Region Sales (All) :=
CALCULATE([Sales USD], ALL('FactSales'[Region]))

Region Share of Sales :=
DIVIDE([Sales USD], [Region Sales (All)])

Region Rank (by Sales) :=
RANKX(ALL('FactSales'[Region]), [Sales USD], , DESC, DENSE)
```


# Step 8 — Dashboards

## 8.1 Retention (Customer Dynamics)

* **Cards:** Customers, New Customers, Returning Customers
* **Stacked column:** `DimDate[YearMonth]` vs **Sales USD (New)** & **Sales USD (Returning)**
* **Cohort matrix (counts):**

  * Rows = `FactSales[Cohort Month]`
  * Columns = `FactSales[Months Since First Purchase]`
  * Values = **DistinctCount(Customer ID)**
    *(Retention % optional; counts already included.)*

## 8.2 Regional Growth

* **Cards:** Sales USD, Orders, Customers, AOV
* **Column chart:** Sales by **Region**
* **Bar chart:** Sales by **State** (Region slicer optional)
* **Table:** Region | Sales USD | Orders | Customers | AOV | **Region Share %** | **Region Rank**

## 8.3 Executive Page (one-pager)

* Top KPIs: Sales, Orders, Customers, AOV
* Left: Sales by Region (column)
* Right: New vs Returning Sales by `YearMonth` (stacked column)
* Bottom: Region performance table (with **Share %** and **Rank**)
* Slicers: `DimDate[YearMonth]`, Region

## 8.4 Recommendations (business takeaways)

* Returning customers drive \~50%+ of sales → **loyalty program & cross-sell**
* **East** shows higher **AOV** → focus upsell/mix strategy
* **Central** underperforms vs. size → **diagnose pricing/logistics/targeting**
* **South** smallest share → **test & learn** with controlled spend


# Step 9 — QA, Performance & Hygiene

* Totals reconcile (Sales, Orders, Customers)
* All slicers propagate; no `(Blank)` Region in visuals
* Hide unused technical columns in Report view
* Currency formatting (USD) + whole-number counts
* Optional: Performance Analyzer check (no slow visuals)

# Step 10 — Assets & How to Run

**Repo structure**

```
/pbix/        Superstore_Analytics.pbix
/assets/      screenshots_exec.png, screenshots_retention.png, screenshots_regional.png
/docs/        Executive_Summary.pdf  (optional)
```

**Open & use**

1. Open PBIX in Power BI Desktop
2. Use slicers (YearMonth, Region) to filter pages
3. Pages: **Executive → Regional Growth → Retention → Recommendations**

**At load (reference values)**

* Sales USD ≈ **\$2.26M**, Orders ≈ **4.9K**, Customers = **793**, AOV ≈ **\$459**


## Notes / Limitations

* Geography modeled at **Region/State** level using fields from Fact (postal codes contain nulls)
* Profit-related analysis intentionally out of scope for focus (marketing/sales brief)
