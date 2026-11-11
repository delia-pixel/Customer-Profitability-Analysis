# E-Commerce Dataset Normalization Report

## 1. First Normal Form (1NF) Check

### 1.1 Order Items Table

We performed the following checks:

- **Missing values and duplicates:** We checked for missing values and duplicate records. None were found.
- **Primary key:** We checked whether `order_item_id` could be used as the primary key, but it was not unique.
- **Composite key:** We checked whether `order_item_id` together with `product_id` could be used as a composite primary key. However, the combination was still not unique.
- **Multiple values in one cell:** We checked whether any cells contained multiple values. Fortunately, none were found.

**Conclusion:** We needed to create a new unique identifier column to serve as the primary key so that the Order Items table could meet 1NF requirements.

### 1.2 Customer Master Table

We performed the following checks:

- **Missing values:** None were found.
- **Primary key:** We checked whether `customer_id` could be used as the primary key. It was unique.
- **Duplicates:** No duplicate records were found.
- **Multiple values in cells:** None were found.

**Conclusion:** The Customer Master table meets 1NF requirements.

### 1.3 Product Catalog Table

We performed the following checks:

- **Missing values and duplicates:** None were found.
- **Primary key:** We checked whether `product_id` could be used as the primary key. It was unique.
- **Multiple values in cells:** None were found.

**Conclusion:** The Product Catalog table meets 1NF requirements.

### 1.4 E-Commerce Sales Customer Analytics Table

We performed the following checks:

- **Primary key:** We checked whether `order_id` could be used as the primary key. It was unique.
- **Missing values:** We found missing values in three columns: `estimated_delivery_days`, `delivery_days`, and `return_status`. However, the primary key had no missing values, so this did not prevent the table from meeting 1NF requirements.
- **Duplicates:** No duplicate records were found.
- **Multiple values in cells:** We detected separators in the `customer_review` column. However, this was normal because the column contains sentences and written reviews.

**Conclusion:** The E-Commerce Sales Customer Analytics table meets 1NF requirements.

## 2. Second Normal Form (2NF) Check

### 2.1 Functional Dependency

We checked functional dependencies in the four tables to understand whether columns depended on their primary keys.

Our findings were:

- We checked whether each product had a single unit price. The result showed that each product had a consistent unit price.
- We checked whether product costs varied between transactions.
- We discovered that each product had multiple recorded costs in the Order Items table. This meant that `product_id` alone was not enough to determine the recorded total product cost.

### 2.2 Repeated Columns

We checked the four tables for repeated columns and compared their values.

#### A. E-Commerce Table and Customer Master Table

We discovered that the E-Commerce table contained 10 columns that were already available in the Customer Master table.

We decided to remove the repeated customer information from the E-Commerce table and keep only `customer_id` to connect both tables.

#### B. Order Items Table and Product Catalog Table

We discovered three similar columns:

**1. Product ID**

We kept `product_id` because it connects the Order Items table to the Product Catalog table.

**2. Unit Price**

We compared unit prices in both tables to check whether they were the same.

The results showed that the unit prices matched.

This meant that the unit price could be stored in the Product Catalog table instead of being repeated unnecessarily.

**3. Product Cost**

We investigated the meaning of `product_cost` in both tables.

We discovered that:

- In the Product Catalog table, `product_cost` represents the cost of one unit.
- In the Order Items table, `product_cost` represents the total cost of the purchased quantity.

Therefore, the product cost in Order Items is calculated based on the quantity purchased.

#### C. E-Commerce Table and Order Items Table

We compared the repeated financial columns in both tables.

**1. Quantity Differences**

We discovered that the `quantity` column in the E-Commerce table was not always equal to the sum of quantities in the Order Items table.

We initially noticed that some orders contained repeated product records, which could help explain the quantity differences.

**2. Gross Sales Differences**

We also compared the `gross_sales` column in both tables.

We discovered differences in gross sales for the same orders that had quantity differences.

**3. Investigation of an Individual Order**

We selected one order ID and investigated its records in both tables.

In the E-Commerce table, the order had a quantity of 7 and a recorded gross sales value.

However, the same order had eight product records in the Order Items table.

When we added the quantities and gross sales of the first four product records, their totals matched the values in the E-Commerce table exactly.

This suggested that additional product records might have been added to the Order Items table without updating the main E-Commerce table.

**4. Investigation of the Remaining Orders**

We continued investigating the other orders with quantity and gross sales differences.

We discovered that every one of these orders had an initial group of product records whose total quantity and gross sales matched the E-Commerce table.

The additional product records appeared after this matching group.

This finding suggests that the additional records could explain the differences between the two tables.

We decided to preserve these records rather than remove them without confirming why they existed.

#### D. E-Commerce Table and Product Catalog Table

We also examined the repeated `product_cost` column.

We discovered that the product cost in the E-Commerce table represents the total product cost of the order.

This is different from the Product Catalog table, where product cost represents the cost of a single unit.

## 3. Third Normal Form (3NF) Check

The purpose of this stage was to remove unnecessary dependencies and organize the data into separate related tables.

### 3.1 Separating Customer Information

We removed all repeated customer information from the E-Commerce table except `customer_id`.

We kept `customer_id` because it connects orders to customers.

We then renamed the E-Commerce table to `Orders`.

### 3.2 Creating a Primary Key for Order Items

We added a new unique `order_item_id` column to the Order Items table.

This column serves as the primary key and allows each order item record to be identified individually.

### 3.3 Creating Copies of the Tables

We created copies of the four original tables before applying the remaining transformations.

We saved the normalized tables in a new folder named `3NF_Tables`.

This allowed us to preserve the original datasets while working on the normalized versions.

### 3.4 Separating Customer-Level Metrics

We separated customer-level metrics from individual orders because these values describe customers rather than specific transactions.

The columns were:

- `customer_lifetime_value`
- `is_repeat_customer`
- `customer_order_count`

We created a separate table called `CustomerMetrics` and connected it to the Customers table using `customer_id`.

### 3.5 Creating the Order Reviews Table

We created a separate table called `OrderReviews` to store customer reviews.

The table contains:

- `order_id`
- `customer_rating`
- `customer_review`
- `review_sentiment`

We connected the Order Reviews table to the Orders table using `order_id`.

This allowed us to separate review information from the main order records.

### 3.6 Checking Marketing Columns

We checked the relationships between the marketing columns to determine whether they needed to be separated into another table.

These columns included:

- `marketing_channel`
- `campaign_name`
- `coupon_code`

We did not find a consistent relationship that justified creating another table.

Therefore, we decided to keep the marketing columns in the Orders table.

### 3.7 Removing Profit Margin Percentage

We removed `profit_margin_percentage` because it can be calculated directly from profit and net sales.

The formula is:

**Profit Margin (%) = (Profit / Net Sales) × 100**

Removing this column helped reduce redundant calculated data.

### 3.8 Removing Profit

We removed the `profit` column from the Orders table because it could be calculated using the other financial columns.

The formula is:

**Profit = Net Sales − Product Cost − Shipping Cost**

This reduced unnecessary duplication in the table.

### 3.9 Removing Other Calculated Financial Columns

We checked the remaining financial columns and verified their formulas before removing redundant calculated attributes.

We removed:

- `net_sales` from the Orders table because it can be calculated from gross sales, discount amount, tax amount, and shipping cost.
- `gross_sales` from the Order Items table because it can be calculated from quantity and unit price.

We preserved the other original transaction values needed for future analysis.

## 4. Final Normalized Tables

After completing the normalization process, we obtained six tables.

| Table           | Number of Records |
| --------------- | ----------------: |
| Customers       |            25,000 |
| CustomerMetrics |            24,911 |
| Products        |             1,175 |
| Orders          |           138,116 |
| OrderItems      |           397,569 |
| OrderReviews    |           113,559 |

All six tables were saved as CSV files in the `3NF_Tables` folder.

## 5. Primary Key Validation

We checked the primary keys of the six normalized tables.

The purpose was to ensure that each record had a unique identifier and that no primary key contained missing values.

The validation showed that:

- All six tables had unique primary keys.
- No duplicate primary keys were found.
- No missing primary keys were found.

**Conclusion:** All six tables passed primary key validation.

## 6. Foreign Key Validation

We checked the foreign key relationships between the normalized tables.

The relationships tested were:

- Orders → Customers
- CustomerMetrics → Customers
- OrderItems → Orders
- OrderItems → Products
- OrderReviews → Orders

All five relationships returned **0 invalid references**.

We also confirmed that `order_item_id` contained no missing values.

**Conclusion:** All foreign key relationships passed validation, confirming that the records referenced valid entries in their related tables.

## 7. Entity Relationship Diagram (ERD)

We created an Entity Relationship Diagram to show the six normalized tables, their primary keys, foreign keys, and relationships.

The diagram helps us understand how the tables are connected and how they can be joined during analysis.

### ER Diagram

**In the 3NF_Tables Folder**

## 8. Conclusion

We checked the original four datasets using 1NF, 2NF, and 3NF principles.

During this process, we checked missing values, duplicates, primary keys, functional dependencies, repeated columns, and calculated financial attributes.

We also investigated the differences between the E-Commerce and Order Items tables and discovered that some orders contained additional product records that were not reflected in their order-level totals.

We preserved these records and documented the differences for the group members who will perform the analysis.

After reorganizing the data, we obtained six related tables and validated their primary and foreign keys.

Finally, we created an ER diagram to show the relationships between the tables.

