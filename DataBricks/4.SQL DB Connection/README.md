# Azure SQL Database Connectivity Guide

This comprehensive guide covers connecting to Azure SQL Database from both Azure Databricks and Azure Data Studio, including setup, configuration, and operations.

## Table of Contents
- [Overview](#overview)
- [Part 1: Azure SQL Database Setup](#part-1-azure-sql-database-setup)
- [Part 2: Databricks Integration](#part-2-databricks-integration)
- [Part 3: Azure Data Studio Connection](#part-3-azure-data-studio-connection)
- [CRUD Operations](#crud-operations)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Overview

This guide demonstrates two primary methods for connecting to Azure SQL Database:
1. **Azure Databricks**: For big data processing and analytics
2. **Azure Data Studio**: For database management and SQL query execution

Both methods use secure authentication through Azure Key Vault for credentials management.

---

## Part 1: Azure SQL Database Setup

### Step 1: Create SQL Server

1. Navigate to [Azure Portal](https://portal.azure.com)
2. Search for **SQL Server** → **SQL Servers** → **Create**
3. Configure the server:
   - **Subscription**: Choose your subscription
   - **Resource Group**: Select or create resource group
   - **Server Name**: Provide globally unique name (e.g., `anand-sql-server2`)
   - **Region**: Select nearby region for performance
   - **Admin Login**: Set administrator username
   - **Password**: Set secure administrator password
4. Click **Review + Create** to finalize setup

### Step 2: Create SQL Database with Sample Data

1. Search for **SQL Databases** → **Create**
2. Configure the database:
   - **Server**: Select the SQL Server created above
   - **Database Name**: Provide name (e.g., `anand-sql-db`)
   - **Sample Data**: Select **Sample Database (AdventureWorks LT)**
3. Review settings and click **Create**

### Step 3: Configure Firewall Rules

1. Navigate to your SQL Database → **Overview**
2. Click **Set server firewall**
3. Add firewall rules:
   - **Rule Name**: `AllowMyIP`
   - **Start IP**: Your current IP address
   - **End IP**: Your current IP address
4. Save the firewall configuration

### Step 4: Get Connection Information

1. In SQL Database **Overview**, note:
   - **Server name**: `<server-name>.database.windows.net`
   - **Database name**: Your database name
2. Under **Connection strings**, copy the JDBC URL:
   ```
   jdbc:sqlserver://<server-name>.database.windows.net:1433;database=<database-name>
   ```

---

## Part 2: Databricks Integration

### Step 1: Create Azure Key Vault

1. Search for **Key Vault** → **Create**
2. Configure Key Vault:
   - **Resource Group**: Same as SQL Server
   - **Key Vault Name**: Provide unique name
   - **Region**: Same as SQL Server
3. After creation, configure access policies:
   - Assign **Key Vault Administrator** role to yourself
   - Assign **Key Vault Administrator** role to **azuredatabricks**

### Step 2: Store Database Credentials

1. Navigate to Key Vault → **Secrets** → **Generate/Import**
2. Create two secrets:
   
   **Secret 1: Database Username**
   - **Name**: `dbusername`
   - **Value**: Your SQL Server admin username
   
   **Secret 2: Database Password**
   - **Name**: `dbuserpassword`
   - **Value**: Your SQL Server admin password

### Step 3: Setup Databricks Secret Scope

1. Create or navigate to your Databricks workspace
2. Create a cluster in Databricks
3. Create secret scope:
   - Navigate to: `https://<databricks-instance>#secrets/createScope`
   - **Scope Name**: `anand-scope` (or your preferred name)
   - **Key Vault**: Link to your Key Vault
   - Get Key Vault DNS name and Resource ID from Key Vault properties

### Step 4: Connect Databricks to Azure SQL Database

```python
# Retrieve credentials from Key Vault
username = dbutils.secrets.get(scope="anand-scope", key="dbusername")
password = dbutils.secrets.get(scope="anand-scope", key="dbuserpassword")

# Define JDBC connection URL
jdbc_url = "jdbc:sqlserver://anand-sql-server2.database.windows.net:1433;database=anand-sql-db;user=anand@anand-sql-server2"

# Read data from SQL database
customer_table = (spark.read
    .format("jdbc")
    .option("url", jdbc_url)
    .option("dbtable", "SalesLT.Customer")
    .option("user", username)
    .option("password", password)
    .load()
)

# Display data
customer_table.show()
```

### Step 5: Create Reusable Connection Setup

Create a common notebook (`db_connection`) for connection logic:

```python
# db_connection notebook - Reusable connection setup
def get_sql_connection():
    """Return SQL connection parameters"""
    username = dbutils.secrets.get(scope="anand-scope", key="dbusername")
    password = dbutils.secrets.get(scope="anand-scope", key="dbuserpassword")
    jdbc_url = "jdbc:sqlserver://anand-sql-server2.database.windows.net:1433;database=anand-sql-db"
    
    connection_properties = {
        "user": username,
        "password": password,
        "driver": "com.microsoft.sqlserver.jdbc.SQLServerDriver"
    }
    
    return jdbc_url, connection_properties

def read_sql_table(table_name):
    """Read table from SQL database"""
    jdbc_url, properties = get_sql_connection()
    
    return (spark.read
        .format("jdbc")
        .option("url", jdbc_url)
        .option("dbtable", table_name)
        .option("user", properties["user"])
        .option("password", properties["password"])
        .load()
    )

def write_sql_table(dataframe, table_name, mode="append"):
    """Write DataFrame to SQL database"""
    jdbc_url, properties = get_sql_connection()
    
    dataframe.write.jdbc(
        url=jdbc_url,
        table=table_name,
        mode=mode,
        properties=properties
    )

# Test connection
print("✓ Connection utilities loaded successfully")
```

**Using in other notebooks:**
```python
# Include connection utilities
%run /Shared/db_connection

# Read data
customers = read_sql_table("SalesLT.Customer")
customers.show()

# Read with custom query
query = "(SELECT TOP 10 * FROM SalesLT.Customer ORDER BY CustomerID) as top_customers"
top_customers = read_sql_table(query)
top_customers.display()
```

---

## Part 3: Azure Data Studio Connection

### Step 1: Install Azure Data Studio

1. Download from [Microsoft's official page](https://learn.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio)
2. Choose appropriate version for your OS (Windows, macOS, Linux)
3. Follow installation instructions

### Step 2: Configure New Connection

1. Open Azure Data Studio
2. Click **New Connection**
3. Configure connection parameters:
   - **Server**: `<server-name>.database.windows.net`
   - **Database**: `<database-name>` (optional)
   - **Authentication Type**: SQL Login
   - **Username**: SQL Server admin username
   - **Password**: SQL Server admin password
   - **Remember Password**: Check for convenience
4. Click **Connect**

### Step 3: Explore Database

1. Expand server node in **Connections** pane
2. Browse **Tables**, **Views**, **Stored Procedures**
3. Right-click tables for operations:
   - **Select Top 1000**
   - **Edit Data**
   - **Generate Script**

### Step 4: Execute SQL Queries

```sql
-- View all customers
SELECT * FROM SalesLT.Customer;

-- Get customer count by city
SELECT 
    City, 
    COUNT(*) as CustomerCount
FROM SalesLT.Customer
GROUP BY City
ORDER BY CustomerCount DESC;

-- Join customers with orders
SELECT 
    c.FirstName,
    c.LastName,
    c.EmailAddress,
    COUNT(oh.SalesOrderID) as OrderCount
FROM SalesLT.Customer c
LEFT JOIN SalesLT.SalesOrderHeader oh ON c.CustomerID = oh.CustomerID
GROUP BY c.CustomerID, c.FirstName, c.LastName, c.EmailAddress
ORDER BY OrderCount DESC;
```

---

## CRUD Operations

### Create (Insert) Operations

**Databricks:**
```python
# Include connection utilities
%run /Shared/db_connection

# Create new customer data
new_customers = [
    (1001, "Individual", "John", "Doe", "johndoe@example.com", "123-456-7890", "2024-01-01"),
    (1002, "Individual", "Jane", "Smith", "janesmith@example.com", "987-654-3210", "2024-01-02")
]

# Create DataFrame
columns = ["CustomerID", "NameStyle", "FirstName", "LastName", "EmailAddress", "Phone", "ModifiedDate"]
new_df = spark.createDataFrame(new_customers, columns)

# Insert into database
write_sql_table(new_df, "SalesLT.Customer", mode="append")
print("✓ New customers inserted successfully")
```

**Azure Data Studio:**
```sql
-- Insert single customer
INSERT INTO SalesLT.Customer (NameStyle, FirstName, LastName, EmailAddress, Phone)
VALUES ('Individual', 'John', 'Doe', 'johndoe@example.com', '123-456-7890');

-- Insert multiple customers
INSERT INTO SalesLT.Customer (NameStyle, FirstName, LastName, EmailAddress, Phone)
VALUES 
    ('Individual', 'Alice', 'Johnson', 'alice.johnson@example.com', '111-222-3333'),
    ('Individual', 'Bob', 'Wilson', 'bob.wilson@example.com', '444-555-6666');
```

### Read Operations

**Databricks:**
```python
# Read all customers
all_customers = read_sql_table("SalesLT.Customer")
all_customers.show(20)

# Read with filtering
filtered_customers = read_sql_table("""
    (SELECT * FROM SalesLT.Customer 
     WHERE City = 'Seattle' AND CompanyName IS NOT NULL) as filtered
""")

# Read specific columns
customer_summary = read_sql_table("""
    (SELECT CustomerID, FirstName, LastName, EmailAddress, City 
     FROM SalesLT.Customer) as summary
""")
```

**Azure Data Studio:**
```sql
-- Basic read operations
SELECT * FROM SalesLT.Customer;

-- Read with conditions
SELECT CustomerID, FirstName, LastName, City
FROM SalesLT.Customer
WHERE City = 'Seattle'
ORDER BY LastName;

-- Read with joins
SELECT 
    c.FirstName + ' ' + c.LastName as FullName,
    c.EmailAddress,
    COUNT(soh.SalesOrderID) as TotalOrders,
    SUM(soh.TotalDue) as TotalSpent
FROM SalesLT.Customer c
LEFT JOIN SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
GROUP BY c.CustomerID, c.FirstName, c.LastName, c.EmailAddress
HAVING COUNT(soh.SalesOrderID) > 0
ORDER BY TotalSpent DESC;
```

### Update Operations

**Databricks:**
```python
# Note: Direct SQL updates from Databricks require special configuration
# Alternative: Read, modify, and write back

# Read existing data
customers = read_sql_table("SalesLT.Customer")

# Filter and modify
updated_customers = customers.filter(customers.CustomerID == 1001) \
    .withColumn("LastName", lit("UpdatedLastName")) \
    .withColumn("ModifiedDate", current_timestamp())

# Write back (this would typically require a more complex merge operation)
# For production, consider using Delta Lake or implementing proper merge logic
```

**Azure Data Studio:**
```sql
-- Update single record
UPDATE SalesLT.Customer 
SET LastName = 'Johnson-Updated', 
    ModifiedDate = GETDATE()
WHERE CustomerID = 1001;

-- Update multiple records
UPDATE SalesLT.Customer 
SET Phone = '000-000-0000'
WHERE Phone IS NULL;

-- Conditional update
UPDATE SalesLT.Customer 
SET CompanyName = FirstName + ' ' + LastName + ' Consulting'
WHERE CompanyName IS NULL AND NameStyle = 0;
```

### Delete Operations

**Databricks:**
```python
# For delete operations from Databricks, you typically need to:
# 1. Use Delta Lake for better support
# 2. Or use JDBC with raw SQL execution
# 3. Or read-filter-write pattern

from pyspark.sql import functions as F

# Example: Remove records and rewrite table (not recommended for large tables)
customers = read_sql_table("SalesLT.Customer")
filtered_customers = customers.filter(customers.CustomerID != 1001)

# Note: This approach requires careful consideration for production use
# write_sql_table(filtered_customers, "SalesLT.Customer", mode="overwrite")
```

**Azure Data Studio:**
```sql
-- Delete specific record
DELETE FROM SalesLT.Customer 
WHERE CustomerID = 1001;

-- Delete with conditions
DELETE FROM SalesLT.Customer 
WHERE EmailAddress LIKE '%example.com' 
AND ModifiedDate > '2024-01-01';

-- Delete with subquery
DELETE FROM SalesLT.Customer 
WHERE CustomerID IN (
    SELECT CustomerID 
    FROM SalesLT.Customer 
    WHERE FirstName = 'TestUser'
);
```

---

## Best Practices

### Security

1. **Credential Management**
   ```python
   # Always use Key Vault for credentials
   # Never hardcode passwords in notebooks
   username = dbutils.secrets.get(scope="secure-scope", key="db-username")
   password = dbutils.secrets.get(scope="secure-scope", key="db-password")
   ```

2. **Connection Pooling**
   ```python
   # Use connection properties for better performance
   connection_properties = {
       "user": username,
       "password": password,
       "driver": "com.microsoft.sqlserver.jdbc.SQLServerDriver",
       "loginTimeout": "30",
       "queryTimeout": "300"
   }
   ```

3. **Firewall Configuration**
   - Use specific IP ranges instead of allowing all IPs
   - Regularly review and update firewall rules
   - Consider using VNet integration for enhanced security

### Performance Optimization

1. **Efficient Data Reading**
   ```python
   # Use column selection to reduce data transfer
   specific_columns = read_sql_table("""
       (SELECT CustomerID, FirstName, LastName, EmailAddress 
        FROM SalesLT.Customer) as subset
   """)
   
   # Use partitioning for large datasets
   large_dataset = (spark.read
       .format("jdbc")
       .option("url", jdbc_url)
       .option("dbtable", "SalesLT.Customer")
       .option("partitionColumn", "CustomerID")
       .option("lowerBound", "1")
       .option("upperBound", "1000")
       .option("numPartitions", "4")
       .option("user", username)
       .option("password", password)
       .load()
   )
   ```

2. **Batch Operations**
   ```python
   # Write in batches for better performance
   def write_in_batches(df, table_name, batch_size=1000):
       total_rows = df.count()
       num_batches = (total_rows + batch_size - 1) // batch_size
       
       for i in range(num_batches):
           start_idx = i * batch_size
           end_idx = min((i + 1) * batch_size, total_rows)
           
           batch_df = df.limit(end_idx).offset(start_idx)
           write_sql_table(batch_df, table_name, mode="append")
           print(f"✓ Batch {i+1}/{num_batches} completed")
   ```

### Error Handling

```python
def safe_sql_operation(operation_func, *args, **kwargs):
    """Safely execute SQL operations with error handling"""
    try:
        result = operation_func(*args, **kwargs)
        print("✓ Operation completed successfully")
        return result
    except Exception as e:
        print(f"✗ Operation failed: {str(e)}")
        # Log error details
        import traceback
        traceback.print_exc()
        return None

# Usage
customers = safe_sql_operation(read_sql_table, "SalesLT.Customer")
```

---

## Troubleshooting

### Common Connection Issues

1. **Firewall Problems**
   ```python
   # Test connectivity
   def test_sql_connection():
       try:
           test_query = "(SELECT 1 as test) as connectivity_test"
           result = read_sql_table(test_query)
           result.show()
           print("✓ Connection successful")
           return True
       except Exception as e:
           print(f"✗ Connection failed: {e}")
           return False
   
   test_sql_connection()
   ```

2. **Authentication Issues**
   ```python
   # Verify secrets are accessible
   def verify_secrets():
       try:
           username = dbutils.secrets.get(scope="anand-scope", key="dbusername")
           password = dbutils.secrets.get(scope="anand-scope", key="dbuserpassword")
           print("✓ Secrets retrieved successfully")
           print(f"Username: {username[:3]}...")  # Show first 3 chars only
           return True
       except Exception as e:
           print(f"✗ Secret retrieval failed: {e}")
           return False
   
   verify_secrets()
   ```

3. **Network Connectivity**
   ```sql
   -- Test from Azure Data Studio
   SELECT @@VERSION as SQLVersion, GETDATE() as CurrentTime;
   
   -- Check connection details
   SELECT 
       connection_id,
       session_id,
       client_net_address,
       auth_scheme
   FROM sys.dm_exec_connections 
   WHERE session_id = @@SPID;
   ```

### Performance Issues

1. **Query Optimization**
   ```sql
   -- Use indexes effectively
   CREATE INDEX IX_Customer_City ON SalesLT.Customer(City);
   
   -- Monitor query execution
   SET STATISTICS TIME ON;
   SET STATISTICS IO ON;
   
   SELECT * FROM SalesLT.Customer WHERE City = 'Seattle';
   ```

2. **Connection Monitoring**
   ```python
   # Monitor Spark job execution
   def monitor_sql_read(table_name):
       import time
       start_time = time.time()
       
       df = read_sql_table(table_name)
       row_count = df.count()
       
       end_time = time.time()
       duration = end_time - start_time
       
       print(f"Table: {table_name}")
       print(f"Rows: {row_count}")
       print(f"Duration: {duration:.2f} seconds")
       
       return df
   
   customers = monitor_sql_read("SalesLT.Customer")
   ```

---

## Additional Resources

- [Azure SQL Database Documentation](https://docs.microsoft.com/en-us/azure/azure-sql/)
- [Azure Data Studio Documentation](https://docs.microsoft.com/en-us/sql/azure-data-studio/)
- [Databricks JDBC Documentation](https://docs.databricks.com/external-data/jdbc.html)
- [Azure Key Vault Integration](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes)

---

## Summary

This guide covered comprehensive connectivity options for Azure SQL Database:

- **Setup**: Complete Azure SQL Database and Key Vault configuration
- **Databricks Integration**: Secure connection using Key Vault secrets
- **Azure Data Studio**: Direct database management and querying
- **CRUD Operations**: Full create, read, update, delete examples
- **Best Practices**: Security, performance, and error handling
- **Troubleshooting**: Common issues and solutions

Both connection methods provide robust options for different use cases, from big data analytics in Databricks to direct database management in Azure Data Studio.
