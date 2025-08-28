# Delta Lake Advanced Features Guide

## Table of Contents
- [Overview](#overview)
- [Setup and Prerequisites](#setup-and-prerequisites)
- [Advanced Features](#advanced-features)
  - [1. Table History and Metadata](#1-table-history-and-metadata)
  - [2. Delta Files and Transaction Log](#2-delta-files-and-transaction-log)
  - [3. File Optimization and Z-Ordering](#3-file-optimization-and-z-ordering)
  - [4. Time Travel Queries](#4-time-travel-queries)
  - [5. Version Rollback](#5-version-rollback)
  - [6. Vacuum Operations](#6-vacuum-operations)
- [Complete Example Workflow](#complete-example-workflow)
- [Best Practices](#best-practices)

## Overview

Delta Lake provides advanced features that go beyond basic CRUD operations, offering capabilities for data versioning, optimization, and maintenance. This guide explores these advanced features with practical examples using Databricks SQL.

## Setup and Prerequisites

```sql
-- Initial setup script for creating Databricks working directory and database
CREATE DATABASE IF NOT EXISTS delta_demo;
USE delta_demo;
```

**Requirements:**
- Databricks cluster attached to notebook
- Default language set to SQL
- Proper permissions for database operations

## Advanced Features

### 1. Table History and Metadata

Delta Lake maintains comprehensive metadata about tables and their structure.

#### Creating a Table with Transaction History

```sql
-- Create employee table
CREATE TABLE employee (
    id INT,
    name STRING,
    department STRING
) USING DELTA;

-- Insert initial records
INSERT INTO employee VALUES 
(1, 'John', 'Engineering'),
(2, 'Jane', 'Marketing'),
(3, 'Bob', 'Sales'),
(4, 'Alice', 'HR'),
(5, 'Charlie', 'Finance'),
(6, 'Diana', 'Operations');

-- Update operation
UPDATE employee SET department = 'IT' WHERE id = 3;

-- Delete operation
DELETE FROM employee WHERE id = 3;

-- Merge operation using temporary view
CREATE OR REPLACE TEMPORARY VIEW employee_updates AS
SELECT * FROM VALUES 
(7, 'Eve', 'Legal'),
(2, 'Jane Smith', 'Marketing'), 
(8, 'Frank', 'Support')
AS t(id, name, department);

MERGE INTO employee e
USING employee_updates u ON e.id = u.id
WHEN MATCHED THEN UPDATE SET name = u.name, department = u.department
WHEN NOT MATCHED THEN INSERT (id, name, department) VALUES (u.id, u.name, u.department);
```

#### Examining Table Details

```sql
-- Get extended table information
DESCRIBE EXTENDED employee;

-- Get detailed table information including partitioning and file details
DESCRIBE DETAIL employee;
```

**Output Information:**
- Schema details
- Partitioning information
- Table size
- Number of files
- Storage location
- Table properties

### 2. Delta Files and Transaction Log

Delta Lake stores data in Parquet files and maintains a transaction log for all operations.

#### Exploring Delta Lake Files

```python
%python
# Display contents of employee table directory
display(dbutils.fs.ls("/path/to/employee/table/"))

# Explore transaction log directory
display(dbutils.fs.ls("/path/to/employee/table/_delta_log/"))
```

**Directory Structure:**
```
employee/
├── _delta_log/
│   ├── 00000000000000000000.json
│   ├── 00000000000000000001.json
│   ├── 00000000000000000002.json
│   └── ...
├── part-00000-xxx.snappy.parquet
├── part-00001-xxx.snappy.parquet
└── ...
```

#### Reading Transaction Log

```sql
-- Query transaction log for merge operation details
SELECT 
    add.path as new_files,
    remove.path as removed_files
FROM 
    (SELECT * FROM json.`/path/to/employee/table/_delta_log/00000000000000000004.json`)
WHERE 
    add IS NOT NULL OR remove IS NOT NULL;
```

### 3. File Optimization and Z-Ordering

Optimize small files and create indexes for better query performance.

```sql
-- Optimize table and create Z-order index on employee_id
OPTIMIZE employee ZORDER BY (id);
```

**Benefits:**
- **File Compaction**: Combines small files into optimal sizes
- **Z-Ordering**: Creates multi-dimensional clustering for faster data retrieval
- **Improved Performance**: Reduces file scanning overhead

### 4. Time Travel Queries

Query historical versions of your data using version numbers or timestamps.

#### View Table History

```sql
-- Display complete history of table operations
DESCRIBE HISTORY employee;
```

**Sample Output:**
| Version | Timestamp | Operation | Details |
|---------|-----------|-----------|---------|
| 5 | 2024-01-15 10:30:00 | OPTIMIZE | File compaction |
| 4 | 2024-01-15 10:25:00 | MERGE | 3 rows affected |
| 3 | 2024-01-15 10:20:00 | DELETE | 1 row deleted |
| 2 | 2024-01-15 10:15:00 | UPDATE | 1 row updated |
| 1 | 2024-01-15 10:10:00 | INSERT | 6 rows inserted |
| 0 | 2024-01-15 10:05:00 | CREATE TABLE | Table created |

#### Time Travel Queries

```sql
-- Query specific version
SELECT * FROM employee VERSION AS OF 3;

-- Query using timestamp
SELECT * FROM employee TIMESTAMP AS OF '2024-01-15 10:20:00';
```

**Use Cases:**
- Data auditing and compliance
- Recovering from accidental changes
- A/B testing with historical data
- Debugging data pipeline issues

### 5. Version Rollback

Restore tables to previous versions when data is accidentally modified or deleted.

#### Simulating Accidental Data Loss

```sql
-- Accidentally delete all records
DELETE FROM employee;

-- Verify table is empty
SELECT COUNT(*) FROM employee; -- Returns 0
```

#### Restoring Previous Version

```sql
-- Restore table to version 5 (before deletion)
RESTORE TABLE employee TO VERSION AS OF 5;

-- Verify data is restored
SELECT * FROM employee;
```

**Important Notes:**
- Restore operations are tracked as new transactions
- You can restore to any available version
- Restore can also use timestamps instead of version numbers

### 6. Vacuum Operations

Remove old data files to manage storage costs while maintaining data integrity.

#### Basic Vacuum Operation

```sql
-- Attempt to vacuum with 0 hours retention (will show warning)
VACUUM employee RETAIN 0 HOURS;
```

#### Advanced Vacuum with Safety Checks Disabled

```sql
-- Disable retention duration check and enable dry run
SET spark.databricks.delta.retentionDurationCheck.enabled = false;
SET spark.databricks.delta.vacuum.logging.enabled = true;

-- Dry run to see what files would be deleted
VACUUM employee RETAIN 0 HOURS DRY RUN;

-- Actual vacuum operation
VACUUM employee RETAIN 0 HOURS;
```

#### Verifying Vacuum Results

```python
%python
# Check remaining data files after vacuum
display(dbutils.fs.ls("/path/to/employee/table/"))
```

**Vacuum Considerations:**
- **Default Retention**: 7 days (168 hours)
- **Safety Mechanism**: Prevents deletion of files that might be in use
- **Storage Optimization**: Removes orphaned files from old versions
- **Time Travel Impact**: Cannot query versions older than retention period

## Complete Example Workflow

```sql
-- 1. Create and populate table
CREATE TABLE sales_data (id INT, product STRING, amount DECIMAL(10,2), date DATE) USING DELTA;
INSERT INTO sales_data VALUES (1, 'Laptop', 1200.00, '2024-01-01');

-- 2. Perform updates and track changes
UPDATE sales_data SET amount = 1150.00 WHERE id = 1;
DESCRIBE HISTORY sales_data;

-- 3. Optimize for performance
OPTIMIZE sales_data ZORDER BY (date, product);

-- 4. Query historical data
SELECT * FROM sales_data VERSION AS OF 0; -- Original data
SELECT * FROM sales_data VERSION AS OF 1; -- After update

-- 5. Clean up old files
VACUUM sales_data RETAIN 168 HOURS; -- Standard 7-day retention
```

## Best Practices

### Performance Optimization
- **Regular Optimization**: Run `OPTIMIZE` on frequently updated tables
- **Z-Ordering**: Use on columns frequently used in WHERE clauses
- **Partitioning**: Consider partitioning large tables by date or category

### Data Management
- **Retention Policies**: Set appropriate vacuum retention based on business needs
- **Time Travel Usage**: Use for auditing and debugging, not for regular queries
- **Transaction Monitoring**: Regularly review table history for unusual patterns

### Security and Compliance
- **Access Control**: Implement proper permissions for time travel and restore operations
- **Audit Trails**: Leverage transaction logs for compliance reporting
- **Data Lineage**: Use Delta Lake's built-in versioning for data governance

### Storage Management
- **Regular Vacuuming**: Schedule vacuum operations to control storage costs
- **Monitor File Sizes**: Use `DESCRIBE DETAIL` to track table growth
- **Cleanup Procedures**: Implement automated cleanup for development/testing environments

---

**Note**: This guide covers advanced Delta Lake features available in Databricks. Some features may have different syntax or availability in other Delta Lake implementations.
