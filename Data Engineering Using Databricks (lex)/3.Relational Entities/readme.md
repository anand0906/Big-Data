# Databricks Relational Entities - Complete Guide

## Overview

Databricks Lakehouse is a modern data platform that combines the best features of data warehouses and data lakes. It organizes your data using a hierarchical structure with six main components that work together to manage and store your information efficiently.

Think of it like organizing files on your computer - you have folders within folders, and each level serves a specific purpose.

<img src="https://docs.azure.cn/en-us/databricks/_static/images/unity-catalog/object-model.png" />
## The Six Primary Objects

### 1. 🗄️ Metastore

**What it is:** The "brain" of your Databricks Lakehouse that stores all the information about your data.

**Simple explanation:** Imagine a giant catalog or index that keeps track of where everything is stored, what it contains, and how it's organized. The Metastore doesn't store the actual data, but it knows everything about the data.

**What it contains:**
- Location of data files
- Data structure information (columns, data types)
- Access permissions
- Data lineage (where data comes from)
- Table statistics

**Example:**
```
Metastore contains information like:
- "customers" table is located at s3://mybucket/customers/
- It has columns: customer_id (integer), name (string), email (string)
- It was created on 2024-01-15
- Only marketing team can access it
```

---

### 2. 📚 Catalog

**What it is:** A high-level grouping that contains multiple databases. Think of it as a major category or department in your organization.

**Simple explanation:** Like different sections in a library - you might have a "Science" section, "History" section, etc. Each catalog represents a major business area or data domain.

**Common use cases:**
- Separating data by business unit (Sales, Marketing, Finance)
- Separating environments (Development, Testing, Production)
- Separating data by geography (US, Europe, Asia)

**Example structure:**
```
production_catalog/
├── sales_database/
├── marketing_database/
└── finance_database/

development_catalog/
├── sales_database/
├── marketing_database/
└── finance_database/
```

**Real-world example:**
```sql
-- Accessing data from different catalogs
SELECT * FROM production_catalog.sales_database.customers;
SELECT * FROM development_catalog.sales_database.customers;
```

---

### 3. 🗂️ Database (Schema)

**What it is:** A collection of related tables, views, and functions grouped together. In Databricks, "database" and "schema" mean the same thing.

**Simple explanation:** Like folders within a catalog section. If a catalog is like a department store, then databases are like specific sections within that store (Electronics, Clothing, Home & Garden).

**Organization benefits:**
- Groups related data together
- Makes data easier to find and manage
- Helps with security and access control
- Keeps things organized and logical

**Example structure:**
```
sales_catalog/
├── customer_database/
│   ├── customers table
│   ├── customer_preferences table
│   └── customer_feedback table
├── product_database/
│   ├── products table
│   ├── categories table
│   └── inventory table
└── orders_database/
    ├── orders table
    ├── order_items table
    └── payments table
```

**SQL example:**
```sql
-- Creating a database
CREATE DATABASE IF NOT EXISTS sales_catalog.customer_database;

-- Using the database
USE sales_catalog.customer_database;
```

---

### 4. 📋 Table

**What it is:** The actual place where your data lives - organized in rows and columns, just like a spreadsheet.

**Simple explanation:** Think of an Excel spreadsheet where each row is a record (like one customer) and each column is a specific piece of information (like name, email, age).

**Key characteristics:**
- Data is stored as files in cloud storage (like Amazon S3)
- Uses Delta Lake format for reliability and performance
- Can handle massive amounts of data
- Supports ACID transactions (data integrity)

**Types of tables:**
- **Managed tables:** Databricks manages both data and metadata
- **External tables:** You manage the data location, Databricks manages metadata

**Example - Customer Table:**
```
+-------------+------------------+----------------------+-----+
| customer_id | name             | email                | age |
+-------------+------------------+----------------------+-----+
| 1           | John Smith       | john@email.com       | 32  |
| 2           | Sarah Johnson    | sarah@email.com      | 28  |
| 3           | Mike Davis       | mike@email.com       | 45  |
+-------------+------------------+----------------------+-----+
```

**SQL examples:**
```sql
-- Creating a table
CREATE TABLE sales_catalog.customer_database.customers (
    customer_id INT,
    name STRING,
    email STRING,
    age INT,
    created_date DATE
);

-- Inserting data
INSERT INTO sales_catalog.customer_database.customers VALUES
(1, 'John Smith', 'john@email.com', 32, '2024-01-15'),
(2, 'Sarah Johnson', 'sarah@email.com', 28, '2024-01-16');

-- Querying data
SELECT name, email FROM sales_catalog.customer_database.customers WHERE age > 30;
```

---

### 5. 👁️ View

**What it is:** A saved query that looks and acts like a table, but doesn't store data itself. It's like a window that shows you specific data from one or more tables.

**Simple explanation:** Think of it as a custom filter or report that you save so you don't have to write the same complex query over and over again.

**Benefits:**
- Simplifies complex queries
- Provides security (hide sensitive columns)
- Creates reusable business logic
- Makes data easier to understand

**Types of views:**
- **Standard views:** Query is executed every time you use the view
- **Materialized views:** Results are stored and refreshed periodically (faster but uses more storage)

**Example scenario:**
Instead of writing this complex query every time:
```sql
SELECT 
    c.name,
    c.email,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.created_date >= '2024-01-01'
GROUP BY c.customer_id, c.name, c.email;
```

You create a view:
```sql
-- Creating the view
CREATE VIEW customer_summary AS
SELECT 
    c.name,
    c.email,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.created_date >= '2024-01-01'
GROUP BY c.customer_id, c.name, c.email;

-- Now you can simply use:
SELECT * FROM customer_summary WHERE total_orders > 5;
```

---

### 6. ⚙️ Function

**What it is:** Custom logic that you save and reuse. Functions take input, process it, and return output - either a single value or multiple rows.

**Simple explanation:** Like creating your own custom calculator or tool that you can use repeatedly in your queries.

**Types of functions:**

#### Scalar Functions (return single value):
```sql
-- Function to calculate tax
CREATE FUNCTION calculate_tax(amount DOUBLE, tax_rate DOUBLE)
RETURNS DOUBLE
RETURN amount * tax_rate;

-- Usage
SELECT 
    product_name,
    price,
    calculate_tax(price, 0.08) as tax_amount,
    price + calculate_tax(price, 0.08) as total_price
FROM products;
```

#### Table Functions (return multiple rows):
```sql
-- Function to get top customers by region
CREATE FUNCTION top_customers_by_region(region_name STRING, limit_count INT)
RETURNS TABLE(customer_name STRING, total_spent DOUBLE)
RETURN SELECT 
    c.name,
    SUM(o.total_amount) as total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.region = region_name
GROUP BY c.name
ORDER BY total_spent DESC
LIMIT limit_count;

-- Usage
SELECT * FROM top_customers_by_region('North', 10);
```

## How Everything Works Together

### Hierarchical Structure
```
Metastore
└── Catalog (e.g., production_data)
    └── Database (e.g., sales_db)
        ├── Table (e.g., customers)
        ├── View (e.g., active_customers)
        └── Function (e.g., calculate_discount)
```

### Complete Example
```sql
-- 1. Create a catalog
CREATE CATALOG ecommerce_data;

-- 2. Create a database
CREATE DATABASE ecommerce_data.sales;

-- 3. Create tables
CREATE TABLE ecommerce_data.sales.customers (
    id INT,
    name STRING,
    email STRING,
    region STRING,
    signup_date DATE
);

CREATE TABLE ecommerce_data.sales.orders (
    order_id INT,
    customer_id INT,
    amount DOUBLE,
    order_date DATE
);

-- 4. Create a function
CREATE FUNCTION ecommerce_data.sales.customer_lifetime_value(customer_id INT)
RETURNS DOUBLE
RETURN (
    SELECT COALESCE(SUM(amount), 0)
    FROM ecommerce_data.sales.orders
    WHERE customer_id = customer_id
);

-- 5. Create a view
CREATE VIEW ecommerce_data.sales.valuable_customers AS
SELECT 
    c.name,
    c.email,
    c.region,
    ecommerce_data.sales.customer_lifetime_value(c.id) as lifetime_value
FROM ecommerce_data.sales.customers c
WHERE ecommerce_data.sales.customer_lifetime_value(c.id) > 1000;

-- 6. Use everything together
SELECT * FROM ecommerce_data.sales.valuable_customers
WHERE region = 'North America'
ORDER BY lifetime_value DESC;
```

## Key Benefits

### 🎯 **Organization**
- Clear hierarchy makes data easy to find
- Logical grouping improves data governance
- Separation of concerns (dev/test/prod)

### 🔒 **Security**
- Granular permissions at each level
- Views can hide sensitive data
- Centralized access control

### 📈 **Performance**
- Delta Lake format for fast queries
- Optimized storage and indexing
- Caching and materialized views

### 🔧 **Flexibility**
- Standard SQL interface
- Works with existing tools (ODBC/JDBC)
- Scalable from small to massive datasets

## Best Practices

1. **Naming conventions:** Use clear, descriptive names
2. **Organization:** Group related objects together
3. **Security:** Apply least privilege access
4. **Documentation:** Comment your views and functions
5. **Performance:** Use appropriate data types and partitioning

This structure provides a robust, scalable way to organize and manage your data while maintaining performance and security.
