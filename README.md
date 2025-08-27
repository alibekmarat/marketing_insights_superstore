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

Customer Activity

* **Unique Customers:** 793
* **Average Orders per Customer:** \~6.2
* **Most Active Customer:** 17 orders
