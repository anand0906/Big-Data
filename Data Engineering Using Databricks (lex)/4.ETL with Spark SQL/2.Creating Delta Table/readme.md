# Creating Delta Tables - Simple Step-by-Step Guide

## What Are Delta Tables?

Delta tables are like super-smart Excel sheets that:
- Keep track of all changes you make
- Handle millions of rows efficiently
- Let multiple people work on them at the same time
- Automatically fix errors and optimize performance

Think of them as Google Docs for data - everyone can see changes, and you can go back to previous versions.

## Getting Started

### Step 1: Basic Setup

```sql
-- Create your workspace (like creating a new project folder)
CREATE DATABASE IF NOT EXISTS my_data_project;
USE my_data_project;

-- Set up for best performance
SET spark.databricks.delta.autoOptimize.autoCompact = true;
```

### Step 2: Prepare Your Notebook
- Create a new notebook
- Set language to SQL
- Connect to your cluster (the computers that will do the work)

## Method 1: CTAS (Create Table As Select)

CTAS is like saying "make me a new table that looks exactly like this data."

### Simple Example with Parquet Files

```sql
-- Create a table from a parquet file (easiest method)
CREATE TABLE customers_table
AS SELECT * 
FROM parquet.`/data/customers.parquet`;

-- Check what you created
DESCRIBE EXTENDED customers_table;
```

**What happened**: Spark automatically figured out the column names and types from the file.

### Real Example: Sales Data

```sql
-- Create sales table with some modifications
CREATE TABLE sales_data AS
SELECT 
    order_id,
    customer_id,
    product_name,
    quantity,
    price,
    order_date,
    input_file_name() as source_file,      -- Track where data came from
    current_timestamp() as loaded_time     -- Track when we loaded it
FROM parquet.`/data/sales/*.parquet`
WHERE price > 0;  -- Only include valid prices
```

## Method 2: Working with CSV Files

CSV files need more help because they're just text files.

### Step 1: Create a Temporary View First

```sql
-- Create a temporary view (like a preview) from CSV
CREATE OR REPLACE TEMPORARY VIEW csv_preview AS
SELECT *
FROM csv.`/data/products.csv`
OPTIONS (
  header "true",              -- First row has column names
  inferSchema "true",         -- Figure out data types
  delimiter ",",              -- Columns separated by commas
  quote '"'                   -- Text wrapped in quotes
);

-- Check if it looks right
SELECT * FROM csv_preview LIMIT 5;
```

### Step 2: Create the Delta Table

```sql
-- Now create the real table
CREATE TABLE products_table AS
SELECT 
    product_id,
    product_name,
    category,
    price,
    stock_quantity
FROM csv_preview
WHERE product_id IS NOT NULL;  -- Remove any bad rows
```

## Adding Smart Features to Your Tables

### Adding Data Quality Rules

```sql
-- Create table with built-in quality checks
CREATE TABLE customer_orders AS
SELECT 
    order_id,
    customer_id,
    total_amount,
    order_date
FROM parquet.`/data/orders.parquet`
WHERE total_amount > 0;  -- Basic quality check

-- Add constraints (rules) to keep data clean
ALTER TABLE customer_orders 
ADD CONSTRAINT valid_amount CHECK (total_amount >= 0);

-- Add requirement that certain fields cannot be empty
ALTER TABLE customer_orders 
ADD CONSTRAINT customer_required CHECK (customer_id IS NOT NULL);
```

### Adding Helpful Information

```sql
-- Create table with descriptions and organization
CREATE TABLE inventory_data (
    product_id STRING COMMENT 'Unique identifier for products',
    product_name STRING COMMENT 'Name of the product',
    price DECIMAL(10,2) COMMENT 'Price in USD',
    category STRING COMMENT 'Product category'
)
USING DELTA
LOCATION '/data/delta/inventory'  -- Where to store the table
PARTITIONED BY (category)         -- Organize by category for faster searches
COMMENT 'Daily inventory data for all products';
```

## Transforming Data While Creating Tables

### Example: Cleaning Customer Data

```sql
-- Create clean customer table with transformations
CREATE TABLE clean_customers AS
SELECT 
    customer_id,
    UPPER(TRIM(first_name)) as first_name,           -- Clean names
    UPPER(TRIM(last_name)) as last_name,             -- Remove extra spaces
    LOWER(TRIM(email)) as email,                     -- Standardize email
    CASE 
        WHEN state = 'CA' THEN 'California'
        WHEN state = 'NY' THEN 'New York'
        ELSE state 
    END as state_full_name,                          -- Expand abbreviations
    registration_date,
    YEAR(registration_date) as registration_year     -- Add calculated column
FROM csv.`/data/raw_customers.csv`
OPTIONS (header "true", inferSchema "true")
WHERE email IS NOT NULL 
AND email LIKE '%@%.%';  -- Only valid email formats
```

### Example: Creating Summary Tables

```sql
-- Create a summary table for reporting
CREATE TABLE monthly_sales_summary AS
SELECT 
    YEAR(order_date) as sales_year,
    MONTH(order_date) as sales_month,
    COUNT(*) as total_orders,
    SUM(total_amount) as total_revenue,
    AVG(total_amount) as average_order_value,
    COUNT(DISTINCT customer_id) as unique_customers
FROM parquet.`/data/orders.parquet`
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY sales_year, sales_month;
```

## Copying Tables (Cloning)

Sometimes you want to make copies of tables for testing or backup.

### Deep Clone (Full Copy)

```sql
-- Make a complete copy (like copying a folder with all files)
CREATE TABLE customers_backup 
DEEP CLONE customers_table;
```
- **Use when**: You need a completely separate copy
- **Takes**: More time and storage space
- **Good for**: Backups, testing changes

### Shallow Clone (Reference Copy)

```sql
-- Make a lightweight copy (like creating a shortcut)
CREATE TABLE customers_test 
SHALLOW CLONE customers_table;
```
- **Use when**: You want to test queries without copying all data
- **Takes**: Very little time and space
- **Good for**: Quick testing, development work

## Complete Real-World Example

Let's create a complete customer analytics table:

```sql
-- Step 1: Create view from CSV with proper options
CREATE OR REPLACE TEMPORARY VIEW raw_customer_data AS
SELECT *
FROM csv.`/data/customer_exports/customers_2024.csv`
OPTIONS (
    header "true",
    inferSchema "true",
    delimiter ",",
    multiline "true"
);

-- Step 2: Create enriched delta table
CREATE TABLE customer_analytics (
    customer_id STRING COMMENT 'Unique customer identifier',
    full_name STRING COMMENT 'Customer full name',
    email STRING COMMENT 'Customer email address',
    city STRING COMMENT 'Customer city',
    state STRING COMMENT 'Customer state',
    registration_year INT COMMENT 'Year customer registered',
    customer_tier STRING COMMENT 'Customer value tier',
    loaded_date TIMESTAMP COMMENT 'When this record was loaded'
)
USING DELTA
LOCATION '/data/delta/customer_analytics'
PARTITIONED BY (state)
COMMENT 'Enriched customer data for analytics and reporting'
AS
SELECT 
    customer_id,
    CONCAT(first_name, ' ', last_name) as full_name,
    LOWER(TRIM(email)) as email,
    INITCAP(city) as city,
    UPPER(state) as state,
    YEAR(registration_date) as registration_year,
    CASE 
        WHEN total_spent >= 1000 THEN 'Premium'
        WHEN total_spent >= 500 THEN 'Standard'
        ELSE 'Basic'
    END as customer_tier,
    current_timestamp() as loaded_date
FROM raw_customer_data
WHERE email IS NOT NULL 
AND email LIKE '%@%.%'
AND customer_id IS NOT NULL;

-- Step 3: Add quality rules
ALTER TABLE customer_analytics 
ADD CONSTRAINT valid_email CHECK (email LIKE '%@%.%');

ALTER TABLE customer_analytics 
ADD CONSTRAINT valid_tier CHECK (customer_tier IN ('Basic', 'Standard', 'Premium'));

-- Step 4: Check your work
SELECT 
    state,
    customer_tier,
    COUNT(*) as customer_count
FROM customer_analytics
GROUP BY state, customer_tier
ORDER BY state, customer_tier;
```

## Quick Reference Commands

```sql
-- Check table structure
DESCRIBE EXTENDED table_name;

-- See table constraints
SHOW TBLPROPERTIES table_name;

-- Count records
SELECT COUNT(*) FROM table_name;

-- See sample data
SELECT * FROM table_name LIMIT 10;

-- Check for problems
SELECT 
    COUNT(*) as total_rows,
    COUNT(DISTINCT customer_id) as unique_customers,
    SUM(CASE WHEN email IS NULL THEN 1 ELSE 0 END) as missing_emails
FROM table_name;
```

## Key Takeaways

1. **CTAS is powerful**: Creates tables automatically from existing data
2. **CSV needs help**: Always create a temporary view first for CSV files  
3. **Add quality rules**: Use constraints to keep your data clean
4. **Transform while loading**: Clean and enrich data as you create tables
5. **Choose clone type wisely**: Deep clone for backups, shallow clone for testing
6. **Always test first**: Use LIMIT and check your data before creating large tables

Remember: Delta tables are designed to make your data reliable and fast. Start simple, then add more features as you get comfortable!
