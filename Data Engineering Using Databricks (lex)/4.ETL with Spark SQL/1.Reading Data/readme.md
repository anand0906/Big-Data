## Reading Data from Files

### Reading Excel-like Files (CSV)

```sql
-- Read a CSV file (like opening an Excel file)
SELECT *
FROM csv.`/path/to/your/sales_data.csv`
OPTIONS (
  header "true",           -- First row contains column names
  inferSchema "true"       -- Figure out data types automatically
)
LIMIT 10;                  -- Show only first 10 rows
```

**Real Example**: Reading customer data
```sql
SELECT 
    customer_id,
    first_name,
    last_name,
    email
FROM csv.`/data/customers.csv`
OPTIONS (header "true", inferSchema "true")
WHERE email IS NOT NULL;
```

### Reading Web Data (JSON)

```sql
-- Read JSON files (like reading data from a website)
SELECT 
    product_name,
    price,
    category,
    reviews.rating
FROM json.`/data/products.json`
WHERE price > 100;
```

### Reading Optimized Files (Parquet)

```sql
-- Parquet files are like compressed, organized data
SELECT 
    state,
    COUNT(*) as total_loans,
    AVG(loan_amount) as average_loan
FROM parquet.`/data/loans.parquet`
GROUP BY state
ORDER BY total_loans DESC;
```

## Reading Data from Databases

### Connecting to PostgreSQL

```sql
-- Connect to a PostgreSQL database (like connecting to a library catalog)
CREATE OR REPLACE TEMPORARY VIEW customer_database AS
SELECT *
FROM jdbc
OPTIONS (
  url "jdbc:postgresql://database-server:5432/company_db",
  dbtable "customers",
  user "your_username",
  password "your_password",
  driver "org.postgresql.Driver"
);

-- Now query like any table
SELECT 
    customer_name,
    total_purchases,
    last_purchase_date
FROM customer_database
WHERE last_purchase_date >= '2024-01-01';
```

### Getting Only New Data (Incremental Loading)

```sql
-- Only get customers who were updated recently
CREATE OR REPLACE TEMPORARY VIEW new_customers AS
SELECT *
FROM jdbc
OPTIONS (
  url "jdbc:postgresql://database-server:5432/company_db",
  dbtable "(SELECT * FROM customers WHERE updated_date > '2024-08-01') as new_data",
  user "your_username",
  password "your_password"
);
```

This is like only downloading new emails instead of all emails every time.

## Reading Data from the Cloud

### Amazon S3
```sql
-- Read from Amazon's cloud storage
SELECT *
FROM json.`s3a://my-company-bucket/sales-data/*.json`
LIMIT 100;
```

### Microsoft Azure
```sql
-- Read from Microsoft's cloud storage
SELECT *
FROM csv.`abfss://container@company.dfs.core.windows.net/data/file.csv`
OPTIONS (header "true", inferSchema "true");
```

### Google Cloud
```sql
-- Read from Google's cloud storage
SELECT *
FROM parquet.`gs://my-bucket/data/`
LIMIT 100;
```

## Real-World Example: Processing Sales Data

Let's say you have sales data in different formats and want to combine them:

### Step 1: Extract from Different Sources

```sql
-- Get online sales from JSON files
CREATE OR REPLACE TEMPORARY VIEW online_sales AS
SELECT 
    order_id,
    customer_id,
    product_name,
    quantity,
    price,
    'online' as sales_channel,
    order_date
FROM json.`/data/online_sales/*.json`;

-- Get store sales from CSV files
CREATE OR REPLACE TEMPORARY VIEW store_sales AS
SELECT 
    order_id,
    customer_id,
    product_name,
    quantity,
    price,
    'store' as sales_channel,
    order_date
FROM csv.`/data/store_sales.csv`
OPTIONS (header "true", inferSchema "true");

-- Get customer info from database
CREATE OR REPLACE TEMPORARY VIEW customers AS
SELECT 
    customer_id,
    customer_name,
    email,
    city,
    state
FROM jdbc
OPTIONS (
  url "jdbc:postgresql://db-server:5432/crm",
  dbtable "customers",
  user "analyst",
  password "secret123"
);
```

### Step 2: Combine Everything

```sql
-- Combine all sales data with customer information
SELECT 
    s.order_id,
    c.customer_name,
    c.city,
    c.state,
    s.product_name,
    s.quantity,
    s.price,
    s.sales_channel,
    s.order_date
FROM (
    -- Combine online and store sales
    SELECT * FROM online_sales
    UNION ALL
    SELECT * FROM store_sales
) s
JOIN customers c ON s.customer_id = c.customer_id
WHERE s.order_date >= '2024-01-01'
ORDER BY s.order_date DESC;
```

## Best Practices (Simple Tips)

1. **Start Small**: Always test with `LIMIT 10` first
2. **Use Views**: Create temporary views for reusable data sources
3. **Check Your Data**: Count records and check for missing values
4. **Choose the Right Format**: Parquet is usually fastest, CSV is most common
5. **Name Things Clearly**: Use descriptive names for your views and tables

## Common Mistakes to Avoid

1. **Forgetting Headers**: Always check if your CSV has headers
2. **Wrong File Paths**: Double-check your file locations
3. **Missing Passwords**: Store database credentials securely
4. **Reading Too Much**: Use LIMIT when exploring data
5. **Ignoring Data Types**: Let Spark infer schemas or define them properly

## Quick Troubleshooting

**Problem**: "File not found"
**Solution**: Check the exact file path and permissions

**Problem**: "Connection failed" 
**Solution**: Verify database credentials and network access

**Problem**: "Schema mismatch"
**Solution**: Use `inferSchema "true"` or define schema manually

**Problem**: "Out of memory"
**Solution**: Use filters (WHERE clauses) to reduce data size

This guide gives you everything you need to start extracting data from various sources using Spark SQL in Databricks!
