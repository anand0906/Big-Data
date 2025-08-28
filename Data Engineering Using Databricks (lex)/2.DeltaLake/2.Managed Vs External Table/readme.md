# Managed vs External Delta Tables in Databricks

## Overview

In Databricks, there are two primary types of tables you can create:
1. **Managed Tables**
2. **External Tables**

Both table types consist of **data** and **metadata**, but they differ in where this information is stored and how it's managed.

## Table Creation Methods

You can create tables in Databricks using two approaches:
- **DataFrame Style**: Using DataFrame API
- **SQL Style**: Using SQL commands

## Managed Tables

### Definition
A **Managed Table** is a table where both data and metadata are fully managed by Databricks.

### Creation Syntax (DataFrame Style)
```python
# Basic managed table creation
df.write.saveAsTable("table_name")

# With database/schema specification
df.write.saveAsTable("database_name.table_name")
```

### Key Characteristics

| Aspect | Details |
|--------|---------|
| **Default Format** | Delta Table (if no format specified) |
| **Data Storage** | Data Explorer |
| **Metadata Storage** | DBFS (`/user/hive/warehouse/`) |
| **Management** | Fully managed by Databricks |
| **Path Specification** | No path required |

### Storage Structure
```
/user/hive/warehouse/ (in DBFS)
├── schema_name.db/
    └── table_name/
        ├── data.parquet
        └── _delta_log/
            ├── 00000000000000000000.json
            └── 00000000000000000000.crc
```

### Drop Behavior
⚠️ **Important**: When you drop a managed table, **both data and metadata are permanently deleted**.

```sql
DROP TABLE database_name.managed_table_name;
-- Result: Complete deletion - no recovery possible
```

## External Tables

### Definition
An **External Table** is a table where data is stored in a user-specified external location, while the table definition remains in the Databricks catalog.

### Creation Syntax (DataFrame Style)
```python
# External table creation with path specification
df.write \
  .option("path", "/mnt/storage_account/container/path/") \
  .saveAsTable("database_name.table_name")
```

### Key Characteristics

| Aspect | Details |
|--------|---------|
| **Default Format** | Delta Table (if no format specified) |
| **Data Storage** | Data Explorer |
| **Metadata Storage** | User-specified external path (e.g., ADLS) |
| **Management** | Data managed by external storage |
| **Path Specification** | Required - specifies metadata location |

### Storage Structure
```
/mnt/storage_account/container/your_path/
├── data.parquet
└── _delta_log/
    ├── 00000000000000000000.json
    └── 00000000000000000000.crc
```

### Drop Behavior
✅ **Advantage**: When you drop an external table, only the table definition is removed. **Metadata remains in the external location**.

```sql
DROP TABLE database_name.external_table_name;
-- Result: Table definition deleted, but metadata persists in external storage
```

## Comparison Table

| Feature | Managed Table | External Table |
|---------|---------------|----------------|
| **Data Location** | Data Explorer | Data Explorer |
| **Metadata Location** | DBFS (`/user/hive/warehouse/`) | External Path (ADLS/S3/etc.) |
| **Path Required** | ❌ No | ✅ Yes |
| **Drop Behavior** | Deletes everything | Preserves metadata |
| **Recovery After Drop** | ❌ Not possible | ✅ Possible |
| **Management** | Full Databricks control | Hybrid control |

## Practical Examples

### Creating a Managed Table
```python
# Assuming you have a DataFrame called 'circuits_df'
circuits_df.write.saveAsTable("formula1.circuits_managed")
```

### Creating an External Table
```python
# External table with ADLS path
circuits_df.write \
  .option("path", "/mnt/adls_storage/raw/formula1/circuits/") \
  .saveAsTable("formula1.circuits_external")
```

### Database/Schema Creation
```sql
-- Create a database first
CREATE SCHEMA formula1;

-- Use the database
USE formula1;

-- Show all tables in current database
SHOW TABLES;
```

## Data Recovery from External Tables

If you drop an external table, you can recover it by querying the external metadata directly:

### Query External Delta Files
```sql
-- Query the external Delta files directly
SELECT * FROM DELTA.`/mnt/adls_storage/raw/formula1/circuits/`;
```

### Recreate Table Using CTAS
```sql
-- Create Table As Select (CTAS) to recreate the table
CREATE TABLE formula1.circuits_recovered AS
SELECT * FROM DELTA.`/mnt/adls_storage/raw/formula1/circuits/`;
```

## Best Practices

### Use Managed Tables When:
- Working with temporary or intermediate data
- Full Databricks management is preferred
- Data lifecycle is tied to the workspace
- Simplified management is priority

### Use External Tables When:
- Data needs to persist beyond table lifecycle
- Multiple systems need access to the same data
- Disaster recovery is important
- Data governance requires external storage control
- Working with existing external data sources

## Delta Lake Features

Both managed and external tables support full Delta Lake functionality:
- **ACID Transactions**
- **Time Travel**
- **Schema Evolution**
- **Merge Operations**
- **Version History**

### Checking Table History
```sql
-- View table history (works for both managed and external)
DESCRIBE HISTORY formula1.circuits_managed;
DESCRIBE HISTORY formula1.circuits_external;
```

## File System Commands

### Explore Managed Table Storage
```python
# List files in DBFS hive warehouse location
%fs ls /user/hive/warehouse/formula1.db/circuits_managed/

# View Delta logs in DBFS
%fs ls /user/hive/warehouse/formula1.db/circuits_managed/_delta_log/
```

### Explore External Table Storage
```python
# List files in external location
%fs ls /mnt/adls_storage/raw/formula1/circuits/

# View external Delta logs
%fs ls /mnt/adls_storage/raw/formula1/circuits/_delta_log/
```

## Summary

The choice between managed and external tables depends on your specific use case:

- **Managed Tables**: Simpler to use, fully managed by Databricks, but data is lost when dropped
- **External Tables**: More flexible, data persists after table drop, suitable for production workloads requiring data durability

Understanding these differences is crucial for designing robust data architectures in Databricks environments.
