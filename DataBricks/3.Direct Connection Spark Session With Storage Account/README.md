# Azure Databricks Direct Spark Session Connectivity Guide

This guide provides step-by-step instructions for connecting Azure Databricks to Azure Storage services using direct Spark session configuration, without mounting storage volumes.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup Steps](#setup-steps)
- [Connection Methods](#connection-methods)
- [Advanced Configuration](#advanced-configuration)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Overview

The direct Spark session method configures storage access at the Spark level, allowing you to read and write data directly using storage URLs without creating persistent mounts. This approach is ideal for:

- Ad-hoc data analysis
- Notebooks that need temporary storage access
- Scenarios where mounting is not preferred
- Dynamic storage configuration

---

## Prerequisites

- Azure subscription with appropriate permissions
- Azure Databricks workspace
- Basic understanding of Spark and Databricks concepts

---

## Setup Steps

### Step 1: Create a Storage Account

1. Navigate to [Azure Portal](https://portal.azure.com)
2. Go to **Storage Accounts** → **Create**
3. Configure the storage account:
   - **Subscription**: Select your subscription
   - **Resource Group**: Create new or select existing
   - **Storage Account Name**: Provide globally unique name
   - **Region**: Choose nearby region
   - **Performance**: Select **Standard**
   - **Replication**: Choose **Locally-Redundant Storage (LRS)**
4. Review and create the storage account

### Step 2: Create Azure Key Vault

1. Search for **Key Vault** in Azure Portal → **Create**
2. Select resource group and provide unique name for the Key Vault
3. Once created, configure **Access Policies**:
   - Click **Add Access Policy**
   - Select **Key Vault Administrator** role
   - Assign to both:
     - Databricks service principal
     - Your user account
4. Save the changes

### Step 3: Store Storage Access Key in Key Vault

1. In Key Vault, navigate to **Secrets** under **Settings**
2. Click **Generate/Import** to create a new secret:
   - **Upload Options**: Manual
   - **Name**: `storage-account-access-key`
   - **Value**: Copy the **Access Key** from your storage account
     - Find this in **Storage Account** → **Access Keys** section
3. Save the secret

### Step 4: Setup Azure Databricks

1. Navigate to **Azure Databricks** in Azure Portal
2. Create a new Databricks workspace (if not exists)
3. Launch the workspace and create a **Cluster**
4. Create a secret scope in Databricks:
   - Navigate to: `https://<databricks-instance>#secrets/createScope`
   - Configure the scope:
     - **Scope Name**: `storage-scope`
     - **Key Vault**: Select the Key Vault you created
     - Link secrets from Key Vault
5. Click **Create**

---

## Connection Methods

### Method 1: Blob Storage Connection

Configure Spark session to access Azure Blob Storage:

```python
# Set Spark configuration for Blob Storage
spark.conf.set(
    "fs.azure.account.key.<storage-account-name>.blob.core.windows.net",
    dbutils.secrets.get(scope="storage-scope", key="storage-account-access-key")
)

# Define path to Blob Storage container
path = "wasbs://<container-name>@<storage-account-name>.blob.core.windows.net/"

# List files in the container
files = dbutils.fs.ls(path)
for file in files:
    print(file.path, file.size)

# Read data directly
df = spark.read.csv(f"{path}/data/sample.csv", header=True, inferSchema=True)
display(df)

# Write data
df.write.mode("overwrite").csv(f"{path}/output/processed_data.csv")
```

### Method 2: Azure Data Lake Storage (ADLS Gen2)

Configure Spark session for Azure Data Lake Storage:

```python
# Set Spark configuration for Data Lake Storage (ADLS Gen2)
spark.conf.set(
    "fs.azure.account.key.<storage-account-name>.dfs.core.windows.net",
    dbutils.secrets.get(scope="storage-scope", key="storage-account-access-key")
)

# Define path to Data Lake container
path = "abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/"

# List files in Data Lake container
files = dbutils.fs.ls(path)
for file in files:
    print(f"Path: {file.path}, Size: {file.size}, Modified: {file.modificationTime}")

# Read Parquet data
df = spark.read.parquet(f"{path}/data/transactions/")
display(df)

# Write Delta table
df.write.format("delta").mode("overwrite").save(f"{path}/delta/transactions")
```

---

## Advanced Configuration

### Reusable Connection Setup

Create a shared notebook for connection configuration to avoid repetition:

**Notebook: `connection_setup`**
```python
def setup_blob_connection(storage_account_name, scope_name="storage-scope", key_name="storage-account-access-key"):
    """Setup Spark configuration for Blob Storage access"""
    spark.conf.set(
        f"fs.azure.account.key.{storage_account_name}.blob.core.windows.net",
        dbutils.secrets.get(scope=scope_name, key=key_name)
    )
    print(f"✓ Blob Storage connection configured for: {storage_account_name}")

def setup_adls_connection(storage_account_name, scope_name="storage-scope", key_name="storage-account-access-key"):
    """Setup Spark configuration for ADLS Gen2 access"""
    spark.conf.set(
        f"fs.azure.account.key.{storage_account_name}.dfs.core.windows.net",
        dbutils.secrets.get(scope=scope_name, key=key_name)
    )
    print(f"✓ ADLS Gen2 connection configured for: {storage_account_name}")

def get_storage_path(storage_type, storage_account, container, path=""):
    """Generate storage path based on storage type"""
    if storage_type.lower() == "blob":
        base_path = f"wasbs://{container}@{storage_account}.blob.core.windows.net"
    elif storage_type.lower() == "adls":
        base_path = f"abfss://{container}@{storage_account}.dfs.core.windows.net"
    else:
        raise ValueError("storage_type must be 'blob' or 'adls'")
    
    return f"{base_path}/{path}" if path else base_path
```

**Using the shared setup in other notebooks:**
```python
# Include the shared setup
%run /Shared/connection_setup

# Configure connections
setup_blob_connection("mystorageaccount")
setup_adls_connection("mydatalake")

# Use helper function
blob_path = get_storage_path("blob", "mystorageaccount", "mycontainer", "data/input")
adls_path = get_storage_path("adls", "mydatalake", "analytics", "processed")

# Read data
df1 = spark.read.csv(f"{blob_path}/sales.csv", header=True)
df2 = spark.read.parquet(adls_path)
```

### Cluster-Level Configuration

Configure storage access at the cluster level for automatic availability across all notebooks:

1. Go to **Clusters** page in Databricks
2. Edit your cluster configuration
3. Under **Spark Config**, add:

```
# For Blob Storage
fs.azure.account.key.<storage-account-name>.blob.core.windows.net {{secrets/storage-scope/storage-account-access-key}}

# For ADLS Gen2
fs.azure.account.key.<storage-account-name>.dfs.core.windows.net {{secrets/storage-scope/storage-account-access-key}}
```

4. Restart the cluster

**Benefits of cluster-level configuration:**
- All notebooks automatically have storage access
- No need to set configuration in each notebook
- Consistent configuration across the cluster
- Simplified notebook code

### Multiple Storage Accounts Configuration

Handle multiple storage accounts in a single cluster:

```python
# Dictionary of storage configurations
storage_configs = {
    "dev_storage": {
        "account": "devstorageaccount",
        "scope": "dev-scope",
        "key": "dev-storage-key"
    },
    "prod_storage": {
        "account": "prodstorageaccount", 
        "scope": "prod-scope",
        "key": "prod-storage-key"
    }
}

def configure_multiple_storage(configs):
    """Configure multiple storage accounts"""
    for name, config in configs.items():
        # Blob Storage
        spark.conf.set(
            f"fs.azure.account.key.{config['account']}.blob.core.windows.net",
            dbutils.secrets.get(scope=config['scope'], key=config['key'])
        )
        # ADLS Gen2
        spark.conf.set(
            f"fs.azure.account.key.{config['account']}.dfs.core.windows.net",
            dbutils.secrets.get(scope=config['scope'], key=config['key'])
        )
        print(f"✓ Configured storage: {name} ({config['account']})")

# Configure all storage accounts
configure_multiple_storage(storage_configs)
```

---

## Best Practices

### Security Best Practices

1. **Use Key Vault for Secrets**
   - Never hardcode access keys in notebooks
   - Always use secret scopes linked to Key Vault
   - Regularly rotate storage account keys

2. **Principle of Least Privilege**
   - Grant minimal required permissions
   - Use different storage accounts for different environments
   - Implement proper access controls

3. **Secret Scope Management**
   - Use descriptive scope names
   - Document scope purposes and contents
   - Regularly audit secret usage

### Performance Optimization

1. **Connection Reuse**
   - Configure connections once per session
   - Use cluster-level configuration when possible
   - Avoid repeated configuration calls

2. **Data Format Selection**
   - Use Parquet for analytical workloads
   - Consider Delta Lake for transactional data
   - Use appropriate compression

3. **Path Structure**
   - Organize data with proper partitioning
   - Use consistent naming conventions
   - Implement proper folder structures

### Code Organization

```python
# Example of well-organized storage configuration
class StorageManager:
    def __init__(self, scope_name="storage-scope"):
        self.scope_name = scope_name
        self.configured_accounts = set()
    
    def configure_storage(self, account_name, key_name, storage_types=["blob", "adls"]):
        """Configure storage account for specified types"""
        access_key = dbutils.secrets.get(scope=self.scope_name, key=key_name)
        
        if "blob" in storage_types:
            spark.conf.set(
                f"fs.azure.account.key.{account_name}.blob.core.windows.net",
                access_key
            )
        
        if "adls" in storage_types:
            spark.conf.set(
                f"fs.azure.account.key.{account_name}.dfs.core.windows.net", 
                access_key
            )
        
        self.configured_accounts.add(account_name)
        print(f"✓ Configured {account_name} for {storage_types}")
    
    def get_path(self, storage_type, account_name, container, path=""):
        """Generate storage path"""
        if account_name not in self.configured_accounts:
            raise ValueError(f"Storage account {account_name} not configured")
        
        if storage_type == "blob":
            base = f"wasbs://{container}@{account_name}.blob.core.windows.net"
        elif storage_type == "adls":
            base = f"abfss://{container}@{account_name}.dfs.core.windows.net"
        else:
            raise ValueError("storage_type must be 'blob' or 'adls'")
        
        return f"{base}/{path}" if path else base

# Usage
storage = StorageManager()
storage.configure_storage("mystorageaccount", "storage-account-access-key")

# Read data
df = spark.read.csv(storage.get_path("blob", "mystorageaccount", "data", "input/sales.csv"), header=True)
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Authentication Errors

**Error**: `java.lang.IllegalArgumentException: Account key not found`

**Solutions**:
```python
# Verify secret retrieval
try:
    secret = dbutils.secrets.get(scope="storage-scope", key="storage-account-access-key")
    print("Secret retrieved successfully")
except Exception as e:
    print(f"Error retrieving secret: {e}")

# Check scope configuration
scopes = dbutils.secrets.listScopes()
print("Available scopes:", [scope.name for scope in scopes])

# Verify scope contents
keys = dbutils.secrets.list("storage-scope")
print("Keys in scope:", [key.key for key in keys])
```

#### 2. Path Format Errors

**Error**: `Path does not exist` or `Invalid path format`

**Solutions**:
```python
# Verify path format
def validate_path(path):
    """Validate storage path format"""
    patterns = {
        "blob": r"wasbs://[^/]+@[^/]+\.blob\.core\.windows\.net/?",
        "adls": r"abfss://[^/]+@[^/]+\.dfs\.core\.windows\.net/?"
    }
    
    import re
    for storage_type, pattern in patterns.items():
        if re.match(pattern, path):
            return storage_type, True
    return None, False

# Example usage
path = "wasbs://mycontainer@mystorage.blob.core.windows.net/"
storage_type, is_valid = validate_path(path)
print(f"Path valid: {is_valid}, Type: {storage_type}")
```

#### 3. Permission Issues

**Error**: `This request is not authorized to perform this operation`

**Solutions**:
```python
# Test basic connectivity
def test_storage_access(path):
    """Test if storage path is accessible"""
    try:
        files = dbutils.fs.ls(path)
        print(f"✓ Successfully accessed {path}")
        print(f"Found {len(files)} items")
        return True
    except Exception as e:
        print(f"✗ Failed to access {path}: {e}")
        return False

# Test your storage
test_storage_access("wasbs://mycontainer@mystorage.blob.core.windows.net/")
```

### Debug Configuration

```python
def debug_spark_config(storage_account):
    """Debug Spark storage configuration"""
    configs = spark.sparkContext.getConf().getAll()
    
    # Filter storage-related configurations
    storage_configs = [
        (key, value) for key, value in configs 
        if storage_account in key and 'azure' in key
    ]
    
    print("Storage-related Spark configurations:")
    for key, value in storage_configs:
        # Mask sensitive values
        masked_value = value[:10] + "..." if len(value) > 10 else value
        print(f"  {key}: {masked_value}")
    
    return len(storage_configs) > 0

# Usage
debug_spark_config("mystorageaccount")
```



## Summary

The direct Spark session configuration method provides a flexible approach to accessing Azure Storage from Databricks without persistent mounts. Key
