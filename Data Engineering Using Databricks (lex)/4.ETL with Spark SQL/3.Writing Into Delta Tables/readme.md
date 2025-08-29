# Writing Into Delta Tables - Simple Guide

## What Does "Writing Into Delta Tables" Mean?

Writing into Delta tables is like updating a smart spreadsheet. You can:
- **Add new rows** (like adding new customer records)
- **Update existing rows** (like changing a customer's address)
- **Delete rows** (like removing cancelled orders)
- **Replace all data** (like importing this month's fresh data)

The magic is that Delta tables remember every change, so you can always undo mistakes!

## Basic Writing Operations

### 1. INSERT - Adding New Data

Think of INSERT like adding new entries to your address book.

```sql
-- Create a simple customers table first
CREATE TABLE customers (
    customer_id INT,
    name STRING,
    email STRING,
    city STRING,
    signup_date DATE
) USING DELTA;

-- Add individual customers (like typing in new contacts)
INSERT INTO customers VALUES 
(1, 'John Smith', 'john@email.com', 'New York', '2024-08-01'),
(2, 'Sarah Johnson', 'sarah@email.com', 'Los Angeles', '2024-08-02'),
(3, 'Mike Wilson', 'mike@email.com', 'Chicago', '2024-08-03');

-- Check what we added
SELECT * FROM customers;
```

### 2. INSERT FROM SELECT - Adding Data from Other Tables

This is like copying contacts from one address book to another.

```sql
-- Add customers from a CSV file
INSERT INTO customers
SELECT 
    customer_id,
    full_name as name,
    email_address as email,
    city,
    registration_date as signup_date
FROM csv.`/data/new_customers.csv`
OPTIONS (header "true", inferSchema "true")
WHERE email_address IS NOT NULL;  -- Only add customers with valid emails
```

### 3. UPDATE - Changing Existing Data

UPDATE is like editing someone's phone number in your contacts.

```sql
-- Update a single customer's city
UPDATE customers 
SET city = 'San Francisco'
WHERE customer_id = 1;

-- Update multiple customers at once
UPDATE customers 
SET city = UPPER(city)  -- Make all cities uppercase
WHERE signup_date >= '2024-08-01';

-- Update with calculated values
UPDATE customers 
SET email = LOWER(TRIM(email))  -- Clean up email formatting
WHERE email IS NOT NULL;
```

### 4. DELETE - Removing Data

DELETE is like removing contacts you no longer need.

```sql
-- Delete a specific customer
DELETE FROM customers 
WHERE customer_id = 3;

-- Delete customers who haven't been active
DELETE FROM customers 
WHERE signup_date < '2023-01-01'
AND customer_id NOT IN (
    SELECT DISTINCT customer_id 
    FROM orders 
    WHERE order_date >= '2024-01-01'
);
```

## Advanced Writing Operations

### 5. MERGE - Smart Updates (UPSERT)

MERGE is like having a smart assistant that:
- Adds new contacts if they don't exist
- Updates existing contacts if they do exist
- All in one command!

```sql
-- Create a staging table with new/updated customer data
CREATE OR REPLACE TEMPORARY VIEW new_customer_data AS
SELECT *
FROM csv.`/data/daily_customer_updates.csv`
OPTIONS (header "true", inferSchema "true");

-- Smart merge operation
MERGE INTO customers AS target
USING new_customer_data AS source
ON target.customer_id = source.customer_id

-- If customer exists, update their info
WHEN MATCHED THEN UPDATE SET
    target.name = source.name,
    target.email = source.email,
    target.city = source.city

-- If customer is new, add them
WHEN NOT MATCHED THEN INSERT (
    customer_id, name, email, city, signup_date
) VALUES (
    source.customer_id, 
    source.name, 
    source.email, 
    source.city, 
    current_date()
);
```

### 6. OVERWRITE - Replacing All Data

This is like clearing your entire address book and starting fresh.

```sql
-- Replace all data in the table
INSERT OVERWRITE customers
SELECT 
    customer_id,
    name,
    email,
    city,
    signup_date
FROM csv.`/data/customers_complete_refresh.csv`
OPTIONS (header "true", inferSchema "true");
```

## Real-World Example: E-commerce Order Processing

Let's build a complete example with an online store's order system.

### Step 1: Create Your Tables

```sql
-- Orders table
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    order_date DATE,
    status STRING,
    total_amount DECIMAL(10,2)
) USING DELTA
PARTITIONED BY (order_date);  -- Organize by date for faster queries

-- Order status tracking table
CREATE TABLE order_status_log (
    order_id INT,
    old_status STRING,
    new_status STRING,
    changed_by STRING,
    changed_at TIMESTAMP
) USING DELTA;
```

### Step 2: Daily Order Loading

```sql
-- Load today's new orders
INSERT INTO orders
SELECT 
    order_id,
    customer_id,
    product_id,
    quantity,
    unit_price,
    order_date,
    'pending' as status,
    quantity * unit_price as total_amount
FROM csv.`/data/daily_orders/orders_2024_08_29.csv`
OPTIONS (header "true", inferSchema "true")
WHERE quantity > 0 AND unit_price > 0;  -- Quality checks
```

### Step 3: Processing Order Updates

```sql
-- Update order statuses (like when orders get shipped)
MERGE INTO orders AS target
USING (
    SELECT 
        order_id,
        new_status,
        current_timestamp() as update_time
    FROM csv.`/data/order_updates/status_updates.csv`
    OPTIONS (header "true", inferSchema "true")
) AS updates
ON target.order_id = updates.order_id

WHEN MATCHED THEN UPDATE SET
    target.status = updates.new_status;

-- Keep track of status changes
INSERT INTO order_status_log
SELECT 
    o.order_id,
    'pending' as old_status,
    o.status as new_status,
    'system' as changed_by,
    current_timestamp() as changed_at
FROM orders o
WHERE o.status != 'pending';
```

### Step 4: Handling Returns and Cancellations

```sql
-- Process returned orders
UPDATE orders 
SET status = 'returned',
    total_amount = 0
WHERE order_id IN (
    SELECT order_id 
    FROM csv.`/data/returns/returned_orders.csv`
    OPTIONS (header "true", inferSchema "true")
);

-- Delete cancelled orders (if business rules allow)
DELETE FROM orders 
WHERE status = 'cancelled' 
AND order_date < current_date() - INTERVAL 90 DAYS;
```

## Performance Tips for Writing

### 1. Batch Operations

```sql
-- Good: Insert many rows at once
INSERT INTO customers
SELECT * FROM csv.`/data/1000_new_customers.csv`
OPTIONS (header "true", inferSchema "true");

-- Avoid: Inserting one row at a time in a loop
-- This would be very slow!
```

### 2. Partitioning for Large Tables

```sql
-- Create partitioned table for better performance
CREATE TABLE sales_data (
    sale_id INT,
    customer_id INT,
    product_name STRING,
    amount DECIMAL(10,2),
    sale_date DATE,
    region STRING
) USING DELTA
PARTITIONED BY (region, sale_date);  -- Partition by region and date
```

### 3. Using Write Options

```sql
-- Configure how data is written for better performance
INSERT INTO sales_data
SELECT * FROM staging_sales
OPTIONS (
    'delta.autoOptimize.optimizeWrite' = 'true',  -- Write faster
    'delta.autoOptimize.autoCompact' = 'true'     -- Keep files organized
);
```

## Common Patterns and Use Cases

### 1. Daily Data Processing Pipeline

```sql
-- Step 1: Load raw daily data
CREATE OR REPLACE TEMPORARY VIEW daily_raw AS
SELECT *
FROM csv.`/data/daily_extracts/transactions_${current_date()}.csv`
OPTIONS (header "true", inferSchema "true");

-- Step 2: Clean and validate
CREATE OR REPLACE TEMPORARY VIEW daily_clean AS
SELECT 
    transaction_id,
    customer_id,
    amount,
    transaction_date,
    CASE 
        WHEN amount < 0 THEN 'refund'
        WHEN amount > 1000 THEN 'large_purchase'
        ELSE 'regular'
    END as transaction_type
FROM daily_raw
WHERE amount IS NOT NULL 
AND customer_id IS NOT NULL
AND transaction_date = current_date();

-- Step 3: Load into main table
INSERT INTO transactions
SELECT * FROM daily_clean;
```

### 2. Slowly Changing Dimensions (Keeping History)

```sql
-- Keep history of customer changes
CREATE TABLE customer_history (
    customer_id INT,
    name STRING,
    email STRING,
    city STRING,
    effective_date DATE,
    end_date DATE,
    is_current BOOLEAN
) USING DELTA;

-- Add new version when customer info changes
INSERT INTO customer_history
SELECT 
    customer_id,
    name,
    email,
    city,
    current_date() as effective_date,
    '9999-12-31' as end_date,  -- Far future date
    true as is_current
FROM new_customer_updates;

-- Mark old records as no longer current
UPDATE customer_history 
SET end_date = current_date() - 1,
    is_current = false
WHERE customer_id IN (SELECT customer_id FROM new_customer_updates)
AND is_current = true;
```

## Troubleshooting Common Issues

### Problem 1: Data Already Exists Error
```sql
-- Solution: Use INSERT OVERWRITE or MERGE instead
INSERT OVERWRITE customers
SELECT * FROM new_data;
```

### Problem 2: Schema Mismatch
```sql
-- Solution: Make sure column names and types match
-- Check your table structure first
DESCRIBE customers;

-- Then match the columns exactly
INSERT INTO customers (customer_id, name, email, city, signup_date)
SELECT 
    id as customer_id,
    full_name as name,
    email_address as email,
    location as city,
    registered_on as signup_date
FROM source_data;
```

### Problem 3: Constraint Violations
```sql
-- Check what data violates constraints before inserting
SELECT *
FROM source_data
WHERE email IS NULL 
   OR email NOT LIKE '%@%.%'  -- Invalid emails
   OR customer_id IS NULL;    -- Missing IDs

-- Clean the data first
INSERT INTO customers
SELECT *
FROM source_data
WHERE email IS NOT NULL 
  AND email LIKE '%@%.%'
  AND customer_id IS NOT NULL;
```

## Best Practices (Simple Rules)

1. **Always test first**: Use a small sample before processing large amounts of data
2. **Check your data**: Count records before and after to make sure nothing was lost
3. **Use transactions**: Group related changes together
4. **Add constraints**: Set up rules to keep bad data out
5. **Monitor performance**: Large updates might take time
6. **Keep backups**: Clone important tables before major changes

## Cleanup Commands

```sql
-- Remove test data
DELETE FROM customers WHERE name LIKE 'test%';

-- Drop temporary tables
DROP TABLE IF EXISTS temp_customer_data;

-- Optimize table after many changes
OPTIMIZE customers;

-- Clean up old file versions
VACUUM customers RETAIN 168 HOURS;  -- Keep 7 days of history
```

This guide shows you how to write data into Delta tables efficiently and safely. Start with simple INSERT statements, then move to more advanced operations like MERGE as you get comfortable!
