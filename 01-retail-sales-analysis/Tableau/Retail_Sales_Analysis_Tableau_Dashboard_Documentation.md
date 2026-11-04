# Retail Sales Performance Dashboard — Tableau Documentation

**Project:** Retail Sales Analysis  
**Visualization tool:** Tableau  
**Workbook:** `BA1_RetailSales.twb`  
**Dashboard:** `Retail Sales Performance Dashboard`  
**Data source:** Excel workbook containing Customers, Orders and Products tables

---

## 1. Dashboard purpose

The dashboard brings together key retail performance measures so that a user can review revenue, sales volume, product performance, regional performance and customer revenue contribution in one place.

The workbook contains 10 worksheets and one assembled dashboard. The dashboard uses five of those worksheets/indicators:

1. Total Revenue
2. Total Units Sold
3. Revenue By Product
4. Sales By Region
5. Customer Revenue Analysis

A Region dropdown filter is also included on the dashboard.

## 2. Data source and preparation

The Tableau workbook connects to an Excel data source named **Retail _ Sales Analysis**. It references three tables:

| Table | Fields used | Analytical purpose |
|---|---|---|
| Customers | `customer_id`, `customer_name`, `region` | Customer identity and regional grouping |
| Orders | `order_id`, `customer_id`, `product_id`, `quantity`, `order_date` | Transaction and quantity information |
| Products | `product_id`, `product_name`, `category`, `price` | Product details and unit price |

The workbook stores these tables in Tableau's logical data model as a related table collection. The common identifiers are `customer_id` between Customers and Orders, and `product_id` between Products and Orders. This allows customer and product attributes to be analysed alongside order quantities.

**Data connection note:** The `.twb` file records a local Excel path from the original authoring computer. A `.twb` file is a workbook definition and does not package the Excel source file. To make the project portable for recruiters, publish the source workbook alongside it or save/export a Tableau Packaged Workbook (`.twbx`) with the data included, after checking that the data is safe to share.

## 3. Calculated fields

### 3.1 Revenue

**Business question:** What is the monetary value of the products sold?

**Tableau calculation:**

```tableau
[price] * [quantity]
```

**How it works:** Multiplies the product's unit price by the number of units in the order. Tableau then aggregates this calculated measure as `SUM(Revenue)` in the revenue worksheets and KPI.

**Why it is used:** Quantity alone shows sales volume but not monetary contribution. Revenue allows comparison of products, customers and regions in value terms.

### 3.2 Customer Segment

**Business question:** How can customers be grouped by the number of units they purchased?

**Tableau calculation saved in the workbook:**

```tableau
IF SUM([quantity]) > 3 THEN "High Value"
ELSEIF SUM([quantity]) >= 2 THEN "Medium Value"
ELSE "Low Value"
END
```

**How it works:** The calculation evaluates each customer's aggregated purchase quantity and assigns a segment:
- High Value: more than 3 units
- Medium Value: 2 or 3 units
- Low Value: fewer than 2 units

**Important interpretation:** In the supplied data, four customers purchased 3 units each and one customer purchased 1 unit. Therefore, this calculation assigns four customers to Medium Value and one to Low Value; no customer falls into High Value. The label is based only on units purchased, not revenue, margin or lifetime value.

## 4. Worksheet design and analytical intent

The workbook contains the following worksheets. The first five listed below are used in the final dashboard; the remaining sheets are additional views saved in the workbook.

| Worksheet | Visual setup recorded in workbook | Why it is useful |
|---|---|---|
| Total Revenue | `SUM(Revenue)` displayed as a text KPI | Gives an at-a-glance monetary performance measure |
| Total Units Sold | `SUM(Quantity)` displayed as a text KPI | Shows overall sales volume independently of revenue |
| Revenue By Product | Product Name against `SUM(Revenue)` | Compares the monetary contribution of individual products |
| Sales By Region | Region against `SUM(Quantity)` | Compares sales volume across regions |
| Customer Revenue Analysis | Customer Name against `SUM(Revenue)` | Shows how much revenue is associated with each customer |
| Customer Purchase Analysis | Customer Name against `SUM(Quantity)` with Customer Segment on colour | Compares customer purchase volume and segment |
| Product Sales By Unit | Product Name against `SUM(Quantity)`, with quantity labels | Identifies products by units sold |
| Region By Revenue | Region against `SUM(Revenue)` | Provides a regional comparison by monetary value |
| Revenue By Category | Pie mark; Category by colour and `SUM(Revenue)` by wedge size | Shows the revenue mix across product categories |
| Sales By Category | Pie mark; Category by colour and `SUM(Quantity)` by wedge size and label | Shows the unit-sales mix across product categories |

The workbook records the mark type as Pie for Revenue By Category and Sales By Category. The other chart worksheets use Tableau's Automatic mark setting with the listed row/column or text encodings. The exact visual appearance can depend on the Tableau rendering environment and formatting.

## 5. Dashboard composition

The dashboard is named **Retail Sales Performance Dashboard**.

The saved dashboard layout places its components in the following order:

### KPI area
- **Total Revenue** — monetary value of sales.
- **Total Units Sold** — total number of units sold.

These KPIs provide two complementary views of performance: value and volume.

### Product and regional comparison area
- **Revenue By Product** — compares products by revenue.
- **Sales By Region** — compares regions by units sold.

This combination helps distinguish products contributing more revenue from regions generating higher sales volume.

### Customer analysis area
- **Customer Revenue Analysis** — compares customer contribution by revenue.

This adds a customer-level perspective to the product and regional views.

### Interactive control
- **Sales By Region** — a dropdown filter is saved on the dashboard.

The filter gives the viewer a way to select a region and examine the dashboard in that regional context. The workbook stores the filter as a dashboard filter control associated with the Region field.

## 6. How to read the dashboard

A recruiter or business user can read the dashboard in this sequence:

1. Start with Total Revenue and Total Units Sold to understand overall value and volume.
2. Review Revenue By Product to see which products contribute most to revenue.
3. Review Sales By Region to compare regional unit volume.
4. Review Customer Revenue Analysis to understand the distribution of revenue across customers.
5. Use the Region dropdown to narrow the analysis to a selected region.

The dashboard combines summary indicators with product, regional and customer-level views, allowing the user to move from an overall performance snapshot to more specific comparisons.

## 7. Results from the supplied sample data

The Excel source contains 5 customer records, 5 product records and 8 order records. Based on the listed product prices and order quantities:

| Measure | Result |
|---|---:|
| Total units sold | 13 |
| Total revenue | ₹1,96,000 |
| Number of customers | 5 |
| Number of products | 5 |
| Number of order records | 8 |

These are sample-dataset results, not general market conclusions. The dashboard's region filter can change the displayed measures when a region is selected.

## 8. Tableau skills demonstrated

This workbook provides evidence of the following Tableau capabilities:

- Connecting Tableau to an Excel data source.
- Working with multiple related tables in a logical data model.
- Using dimensions such as Customer Name, Product Name, Category and Region.
- Using measures such as Quantity, Price and calculated Revenue.
- Creating calculated fields using arithmetic and conditional logic.
- Aggregating measures with `SUM`.
- Building KPI-style text worksheets.
- Building product, regional and customer comparison views.
- Using pie charts to represent category mix.
- Assembling multiple worksheets into a dashboard.
- Adding a dropdown filter to support interactive exploration.

## 9. Limitations and interpretation cautions

1. The source dataset is small: 5 customers, 5 products and 8 orders. The dashboard demonstrates analytical and visualization techniques but does not support broad market-level conclusions.
2. The Customer Segment calculation uses quantity purchased as its only criterion. The term “High Value” should not be interpreted as customer monetary value.
3. Revenue is calculated as unit price multiplied by quantity. The workbook does not show discounts, returns, taxes, shipping, cost of goods or profit.
4. The workbook contains additional worksheets beyond those placed on the dashboard. The documentation distinguishes the dashboard components from the saved supporting worksheets.
5. The uploaded `.twb` is not a packaged workbook. Its recorded Excel connection points to a local path. For public sharing, provide the source Excel file or a `.twbx` package with appropriate data included, and verify that the workbook opens on another machine.

---

## 10. Project outcome

The Tableau workbook translates the retail dataset into a compact performance dashboard. It combines revenue and unit KPIs with product, regional and customer comparisons, and provides a Region dropdown for interactive exploration.

The dashboard complements the SQL analysis: SQL demonstrates how the business questions are answered through queries, while Tableau communicates selected performance measures visually for easier review and decision-making.
