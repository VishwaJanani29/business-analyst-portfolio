# Retail Sales Analysis — SQL Query Portfolio

**Project focus:** Retail sales performance analysis using SQL  
**Tools:** SQL  
**Dataset:** Customers, Products and Orders  
**Purpose:** Demonstrate how SQL can answer practical business questions about product performance, customer purchasing behaviour and regional sales.

---

## 1. Business context and data model

The analysis uses three related tables:

| Table | Purpose | Key fields |
|---|---|---|
| `CUSTOMERS` | Customer details and region | `customer_id`, `customer_name`, `region` |
| `PRODUCTS` | Product catalogue and selling price | `product_id`, `product_name`, `category`, `price` |
| `ORDERS` | Transaction-level purchase records | `order_id`, `customer_id`, `product_id`, `quantity`, `order_date` |

The `ORDERS` table connects customers and products through their respective IDs. The queries use joins to combine transaction quantities with customer and product attributes.

### Dataset scope

The supplied workbook contains 5 customers, 5 products and 8 order records. The analysis is therefore a small demonstration dataset, not a statistically representative retail sample.

---

## 2. Product performance analysis

### Query 1 — Rank products by total units sold

**Business question:** Which products are selling the most?

**Why this query is useful:** Product-level sales volume helps identify leading products and products that may need further investigation. The query aggregates order quantities by product and ranks the results from highest to lowest.

```sql
SELECT
    p.product_name,
    SUM(o.quantity) AS total_units_sold,
    RANK() OVER (ORDER BY SUM(o.quantity) DESC) AS sales_rank
FROM products AS p
JOIN orders AS o
    ON p.product_id = o.product_id
GROUP BY
    p.product_id,
    p.product_name
ORDER BY
    total_units_sold DESC;
```

**Expected result from the supplied dataset:**

| Product | Total units sold | Rank |
|---|---:|---:|
| Headphones | 5 | 1 |
| Desk | 3 | 2 |
| Keyboard | 2 | 3 |
| Laptop | 2 | 3 |
| Office Chair | 1 | 5 |

**Interpretation:** Headphones lead the dataset with 5 units sold. Keyboard and Laptop are tied at 2 units, so `RANK()` assigns both rank 3 and skips rank 4.

### Query 2 — Identify products selling more than two units

**Business question:** Which products have sold more than two units in total?

**Why this query is useful:** This applies a business threshold to aggregated sales. `HAVING` is used because the condition applies to the summed quantity for each product, not to an individual order row.

```sql
SELECT
    p.product_name,
    SUM(o.quantity) AS total_units_sold
FROM products AS p
JOIN orders AS o
    ON p.product_id = o.product_id
GROUP BY
    p.product_id,
    p.product_name
HAVING
    SUM(o.quantity) > 2
ORDER BY
    total_units_sold DESC;
```

**Expected result:**

| Product | Total units sold |
|---|---:|
| Headphones | 5 |
| Desk | 3 |

**Interpretation:** Headphones and Desk exceed the selected threshold. This is a simple way to isolate products meeting a defined sales-volume criterion.

### Query 3 — Identify low-performing products

**Business question:** Which products have sold fewer than two units in total?

**Why this query is useful:** The query applies the opposite threshold to identify products with low sales volume. This can help a business decide which products warrant further investigation. Low volume alone does not establish the reason for poor performance.

```sql
SELECT
    p.product_name,
    SUM(o.quantity) AS total_units_sold
FROM products AS p
JOIN orders AS o
    ON p.product_id = o.product_id
GROUP BY
    p.product_id,
    p.product_name
HAVING
    SUM(o.quantity) < 2
ORDER BY
    total_units_sold ASC;
```

**Expected result:**

| Product | Total units sold |
|---|---:|
| Office Chair | 1 |

**Interpretation:** Office Chair is the only product below the two-unit threshold in this dataset. Further analysis would be needed before recommending a pricing, promotion or inventory action.

### Query 4 — Classify products as Best Seller or Regular

**Business question:** How can products be grouped using a simple sales-volume rule?

**Why this query is useful:** A `CASE` expression converts a numerical measure into an understandable business category. The threshold is a demonstration rule and should be agreed with business stakeholders before being used operationally.

```sql
SELECT
    p.product_name,
    SUM(o.quantity) AS total_units_sold,
    CASE
        WHEN SUM(o.quantity) > 3 THEN 'Best Seller'
        ELSE 'Regular'
    END AS product_category
FROM products AS p
JOIN orders AS o
    ON p.product_id = o.product_id
GROUP BY
    p.product_id,
    p.product_name
ORDER BY
    total_units_sold DESC;
```

**Expected classification:**

| Product | Total units sold | Classification |
|---|---:|---|
| Headphones | 5 | Best Seller |
| Desk | 3 | Regular |
| Keyboard | 2 | Regular |
| Laptop | 2 | Regular |
| Office Chair | 1 | Regular |

**Interpretation:** Only Headphones cross the selected threshold of more than three units. The classification is rule-based, not a claim that the product is commercially successful in a wider market.

### Query 5 — Compare each product with average product sales

**Business question:** Which products sold more units than the average product?

**Why this query is useful:** Comparing each product's total units with the average total across products provides a relative performance benchmark. The inner query first calculates each product's total; the next level calculates the average of those product totals.

```sql
SELECT
    product_name,
    total_units_sold
FROM (
    SELECT
        p.product_id,
        p.product_name,
        SUM(o.quantity) AS total_units_sold
    FROM products AS p
    JOIN orders AS o
        ON p.product_id = o.product_id
    GROUP BY
        p.product_id,
        p.product_name
) AS product_totals
WHERE
    total_units_sold > (
        SELECT AVG(product_total)
        FROM (
            SELECT
                SUM(quantity) AS product_total
            FROM orders
            GROUP BY product_id
        ) AS average_product_totals
    )
ORDER BY
    total_units_sold DESC;
```

**Expected result:**

| Product | Total units sold |
|---|---:|
| Headphones | 5 |
| Desk | 3 |

**Benchmark:** Total units sold are 13 across 5 products, giving an average of **2.6 units per product**.

**Interpretation:** Headphones and Desk are above the dataset's average product sales volume. This is a relative comparison within the supplied dataset.

---

## 3. Customer purchasing analysis

### Query 6 — Calculate total units purchased by each customer

**Business question:** How many units has each customer purchased?

**Why this query is useful:** Customer-level aggregation helps compare purchasing activity and provides the foundation for customer segmentation and ranking.

```sql
SELECT
    c.customer_name,
    SUM(o.quantity) AS total_units_purchased
FROM customers AS c
JOIN orders AS o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name
ORDER BY
    total_units_purchased DESC,
    c.customer_name;
```

**Expected result:**

| Customer | Total units purchased |
|---|---:|
| Ananya | 3 |
| Priya | 3 |
| Rahul | 3 |
| Sneha | 3 |
| Arjun | 1 |

**Interpretation:** Four customers purchased 3 units each. Arjun purchased 1 unit. The dataset therefore has a four-way tie for the highest purchase quantity.

### Query 7 — Find customers purchasing more than two units

**Business question:** Which customers purchased more than two units in total?

**Why this query is useful:** This filters customer-level aggregates using `HAVING`, helping identify customers who meet a selected purchase-volume threshold.

```sql
SELECT
    c.customer_name,
    SUM(o.quantity) AS total_units_purchased
FROM customers AS c
JOIN orders AS o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name
HAVING
    SUM(o.quantity) > 2
ORDER BY
    total_units_purchased DESC,
    c.customer_name;
```

**Expected result:** Ananya, Priya, Rahul and Sneha, each with 3 units purchased.

**Interpretation:** Four of the five customers meet the greater-than-two-unit criterion.

### Query 8 — Segment customers by total purchase quantity

**Business question:** How can customers be grouped into High, Medium and Low purchase categories?

**Why this query is useful:** Customer segmentation turns a numeric measure into categories that are easier to interpret in business reporting. The thresholds below follow the stated rule: more than 3 units is High; 2–3 units is Medium; fewer than 2 units is Low.

```sql
SELECT
    c.customer_name,
    SUM(o.quantity) AS total_units_purchased,
    CASE
        WHEN SUM(o.quantity) > 3 THEN 'High'
        WHEN SUM(o.quantity) >= 2 THEN 'Medium'
        ELSE 'Low'
    END AS purchase_category
FROM customers AS c
JOIN orders AS o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name
ORDER BY
    total_units_purchased DESC,
    c.customer_name;
```

**Expected result using these thresholds:**

| Customer | Total units purchased | Purchase category |
|---|---:|---|
| Ananya | 3 | Medium |
| Priya | 3 | Medium |
| Rahul | 3 | Medium |
| Sneha | 3 | Medium |
| Arjun | 1 | Low |

**Interpretation:** Four customers fall into Medium and one into Low. No customer qualifies as High under the stated rule. These categories describe purchase quantity only; they are not a measure of customer profitability or lifetime value.

### Query 9 — Identify the top two customer ranking positions, including ties

**Business question:** Who are the top customers based on total units purchased?

**Why this query is useful:** A ranking function makes ties visible rather than arbitrarily excluding customers with the same purchase quantity. Here, `DENSE_RANK()` assigns the same rank to customers with equal totals.

```sql
WITH customer_totals AS (
    SELECT
        c.customer_id,
        c.customer_name,
        SUM(o.quantity) AS total_units_purchased
    FROM customers AS c
    JOIN orders AS o
        ON c.customer_id = o.customer_id
    GROUP BY
        c.customer_id,
        c.customer_name
),
customer_ranks AS (
    SELECT
        customer_name,
        total_units_purchased,
        DENSE_RANK() OVER (
            ORDER BY total_units_purchased DESC
        ) AS purchase_rank
    FROM customer_totals
)
SELECT
    customer_name,
    total_units_purchased,
    purchase_rank
FROM customer_ranks
WHERE purchase_rank <= 2
ORDER BY
    purchase_rank,
    customer_name;
```

**Expected result:**

| Customer | Total units purchased | Dense rank |
|---|---:|---:|
| Ananya | 3 | 1 |
| Priya | 3 | 1 |
| Rahul | 3 | 1 |
| Sneha | 3 | 1 |
| Arjun | 1 | 2 |

**Interpretation:** Four customers share first place. Arjun occupies the second distinct ranking position. This is why the query returns five customers even though it asks for the top two ranking positions.

---

## 4. Regional sales analysis

### Query 10 — Calculate sales quantity by region and filter regions above two units

**Business question:** How many units were sold in each region, and which regions exceeded two units?

**Why this query is useful:** Regional aggregation supports geographical comparison. The `HAVING` clause filters regions after their quantities have been summed.

```sql
SELECT
    c.region,
    SUM(o.quantity) AS sales_quantity
FROM customers AS c
JOIN orders AS o
    ON c.customer_id = o.customer_id
GROUP BY
    c.region
HAVING
    SUM(o.quantity) > 2
ORDER BY
    sales_quantity DESC;
```

**Expected result:**

| Region | Sales quantity |
|---|---:|
| South | 6 |
| East | 3 |
| North | 3 |

**Interpretation:** South contributes 6 of the 12 units included in these regional results. West is excluded because its customer purchased only 1 unit.

---

## 5. Additional aggregate analysis

### Query 11 — Calculate average quantity per order

**Business question:** What is the average number of units in an order?

**Why this query is useful:** This provides a simple measure of average order quantity. It is different from average units sold per product.

```sql
SELECT
    AVG(quantity) AS average_quantity_per_order
FROM orders;
```

**Expected result:** **1.625 units per order** (13 units across 8 orders).

### Query 12 — Calculate average total units sold per product

**Business question:** What is the average sales volume across products?

**Why this query is useful:** This query first calculates total quantity for each product and then averages those product-level totals. It is the benchmark used in Query 5.

```sql
SELECT
    AVG(product_total) AS average_units_per_product
FROM (
    SELECT
        product_id,
        SUM(quantity) AS product_total
    FROM orders
    GROUP BY product_id
) AS product_totals;
```

**Expected result:** **2.6 units per product** (13 units across 5 products).

---

## 6. SQL concepts demonstrated

| SQL concept | How it is used in this project |
|---|---|
| `SELECT` | Choose the fields and calculated measures to display |
| `JOIN` | Combine customers or products with their order records |
| `GROUP BY` | Aggregate quantities by product, customer or region |
| `SUM()` | Calculate total units purchased or sold |
| `AVG()` | Calculate average order quantity and average product sales |
| `HAVING` | Filter grouped results using aggregate conditions |
| `CASE` | Classify customers and products using business rules |
| `ORDER BY` | Sort results by sales or purchase quantity |
| `RANK()` | Rank products while preserving ties |
| `DENSE_RANK()` | Rank customers while preserving ties without gaps |
| Subqueries | Compare product totals against an aggregate benchmark |
| CTEs (`WITH`) | Break customer aggregation and ranking into readable steps |

---

## 7. Key findings from the supplied dataset

- **Total units sold:** 13 across 8 orders.
- **Total revenue:** ₹1,96,000, calculated as product price multiplied by order quantity.
- **Top-selling product:** Headphones, with 5 units.
- **Lowest-selling product:** Office Chair, with 1 unit.
- **Top customer purchase quantity:** Four customers tied at 3 units each.
- **Leading region by quantity:** South, with 6 units.
- **Average order quantity:** 1.625 units.
- **Average product sales quantity:** 2.6 units.

These findings describe the supplied sample data. They should not be interpreted as evidence of broader market or customer behaviour without a larger dataset and additional context.

---

## 8. Data and interpretation notes

1. **Customer segmentation:** The documented threshold used here is High: more than 3 units; Medium: 2–3 units; Low: fewer than 2 units. Under this rule, customers with 3 units are Medium. This resolves an inconsistency in the working notes where the same 3-unit total was labelled High in one result and Medium in another.
2. **Top-two customers:** Four customers tie at 3 units. The ranking query deliberately retains ties and returns the top two distinct ranking positions.
3. **Average calculations:** Average quantity per order (1.625) and average total units per product (2.6) answer different business questions and should not be interchanged.
4. **Revenue:** Revenue is not calculated in the query set above because the focus of these exercises is unit-volume analysis. The reported ₹1,96,000 is based on the supplied product prices and order quantities.
5. **SQL compatibility:** The queries use common SQL features. Window functions and CTEs require a database version that supports them. Minor syntax adjustments may be needed for a particular database platform.

---

## 9. Project outcome

This SQL analysis demonstrates a structured approach to retail sales questions: joining transactional data to business entities, aggregating performance measures, applying business thresholds, ranking results, segmenting customers and comparing results against averages.

The purpose is not simply to write SQL statements, but to select an appropriate SQL technique for each business question and explain how the resulting information can support business analysis and decision-making.
