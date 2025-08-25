# Databricks dbutils - Complete Guide

## Table of Contents
- [Overview](#overview)
- [Getting Help](#getting-help)
- [dbutils.fs - File System Operations](#dbutilsfs---file-system-operations)
  - [File System Utils](#file-system-utils)
  - [Mount Operations](#mount-operations)
- [dbutils.widgets - Interactive Widgets](#dbutilswidgets---interactive-widgets)
- [dbutils.notebook - Notebook Management](#dbutilsnotebook---notebook-management)
- [%run Command](#run-command)
- [dbutils.secrets - Secret Management](#dbutilssecrets---secret-management)
- [Creating Secret Scopes](#creating-secret-scopes)
- [Best Practices](#best-practices)

---

## Overview

**dbutils** is a set of utility functions provided by Databricks to interact with the Databricks File System (DBFS), manage jobs, and work with data and notebooks in the Databricks environment. It allows users to interact with the workspace, notebooks, data, secrets, and more programmatically within a Databricks notebook or job.

---

## Getting Help

### `dbutils.help()`

Get an overview of all available dbutils methods and submodules:

```python
dbutils.help()
```

**Output:**
```
This module provides various utilities for users to interact with the rest of Databricks.
  credentials: DatabricksCredentialUtils -> Utilities for interacting with credentials within notebooks
  data: DataUtils -> Utilities for understanding and interacting with datasets (EXPERIMENTAL)
  fs: DbfsUtils -> Manipulates the Databricks filesystem (DBFS) from the console
  jobs: JobsUtils -> Utilities for leveraging jobs features
  library: LibraryUtils -> Utilities for session isolated libraries
  meta: MetaUtils -> Methods to hook into the compiler (EXPERIMENTAL)
  notebook: NotebookUtils -> Utilities for the control flow of a notebook (EXPERIMENTAL)
  preview: Preview -> Utilities under preview category
  secrets: SecretUtils -> Provides utilities for leveraging secrets within notebooks
  widgets: WidgetsUtils -> Methods to create and get bound value of input widgets inside notebooks
```

---

## dbutils.fs - File System Operations

### Getting Help for File System Operations

```python
dbutils.fs.help()
```

### File System Utils

#### 1. `dbutils.fs.ls()` - List Files and Directories

Lists all files and subdirectories in a specified directory on DBFS.

**Syntax:**
```python
dbutils.fs.ls(dir: str)
```

**Example:**
```python
files = dbutils.fs.ls("/mnt/data")
for file in files:
    print(f"Name: {file.name}, Path: {file.path}, Size: {file.size} bytes")
```

**Expected Output:**
```
Name: sales_data.csv, Path: dbfs:/mnt/data/sales_data.csv, Size: 2048 bytes
Name: customer_data.csv, Path: dbfs:/mnt/data/customer_data.csv, Size: 1024 bytes
Name: reports/, Path: dbfs:/mnt/data/reports/, Size: 0 bytes
```

#### 2. `dbutils.fs.cp()` - Copy Files or Directories

Copies a file or directory from one location to another in DBFS.

**Syntax:**
```python
dbutils.fs.cp(source: str, destination: str, recurse: bool = False)
```

**Parameters:**
- `source`: Path of the source file/directory
- `destination`: Path of the destination file/directory
- `recurse`: If True, copies directories recursively

**Example:**
```python
# Copy a single file
dbutils.fs.cp("/mnt/data/sales_data.csv", "/mnt/archive/sales_data.csv")

# Copy directory recursively
dbutils.fs.cp("/mnt/data/reports/", "/mnt/archive/reports/", recurse=True)
```

#### 3. `dbutils.fs.mv()` - Move/Rename Files or Directories

Moves or renames a file or directory in DBFS.

**Syntax:**
```python
dbutils.fs.mv(source: str, destination: str, recurse: bool = False)
```

**Example:**
```python
# Move a file
dbutils.fs.mv("/mnt/data/customer_data.csv", "/mnt/processed/customer_data.csv")

# Rename a directory
dbutils.fs.mv("/mnt/data/old_reports/", "/mnt/data/new_reports/", recurse=True)
```

#### 4. `dbutils.fs.rm()` - Remove Files or Directories

Deletes a specified file or directory.

**Syntax:**
```python
dbutils.fs.rm(dir_or_file: str, recurse: bool = False)
```

**Example:**
```python
# Remove a single file
dbutils.fs.rm("/mnt/archive/sales_data.csv")

# Remove directory and all contents
dbutils.fs.rm("/mnt/old_data/", recurse=True)
```

#### 5. `dbutils.fs.put()` - Write Data to a File

Writes specified content to a file.

**Syntax:**
```python
dbutils.fs.put(file: str, contents: str, overwrite: bool = False)
```

**Example:**
```python
# Create a new file with content
content = "This is a sample file created using dbutils."
dbutils.fs.put("/mnt/data/example.txt", content, overwrite=True)
```

#### 6. `dbutils.fs.head()` - Read File Content

Returns the first bytes of a file as a string.

**Syntax:**
```python
dbutils.fs.head(file: str, maxBytes: int = 65536)
```

**Example:**
```python
# Read first 1000 bytes of a file
content = dbutils.fs.head("/mnt/data/example.txt", maxBytes=1000)
print(content)
```

#### 7. `dbutils.fs.mkdirs()` - Create Directories

Creates directories if they don't exist.

**Syntax:**
```python
dbutils.fs.mkdirs(dir: str)
```

**Example:**
```python
# Create nested directories
dbutils.fs.mkdirs("/mnt/data/year=2023/month=12/day=31")
```

### Mount Operations

#### 1. `dbutils.fs.mount()` - Mount External Storage

Mounts external storage systems to DBFS.

**Syntax:**
```python
dbutils.fs.mount(
    source: str, 
    mountPoint: str, 
    encryptionType: str = "", 
    owner: str = None, 
    extraConfigs: dict = {}
)
```

**Example - Azure Blob Storage:**
```python
dbutils.fs.mount(
    source="wasbs://mycontainer@myaccount.blob.core.windows.net",
    mountPoint="/mnt/mydata",
    extraConfigs={"fs.azure.account.key.myaccount.blob.core.windows.net": "your-access-key"}
)
```

**Example - AWS S3:**
```python
dbutils.fs.mount(
    source="s3a://my-bucket/path/",
    mountPoint="/mnt/s3data",
    extraConfigs={
        "fs.s3a.access.key": "your-access-key",
        "fs.s3a.secret.key": "your-secret-key"
    }
)
```

#### 2. `dbutils.fs.mounts()` - List Mount Points

Displays all current mount points.

**Example:**
```python
mount_points = dbutils.fs.mounts()
for mount in mount_points:
    print(f"Mount Point: {mount.mountPoint}, Source: {mount.source}")
```

#### 3. `dbutils.fs.unmount()` - Unmount Storage

Removes a mount point.

**Example:**
```python
dbutils.fs.unmount("/mnt/mydata")
print("Mount point removed successfully.")
```

#### 4. `dbutils.fs.refreshMounts()` - Refresh Mount Cache

Forces all cluster nodes to refresh their mount cache.

**Example:**
```python
dbutils.fs.refreshMounts()
print("Mount cache refreshed across all nodes.")
```

#### 5. `dbutils.fs.updateMount()` - Update Mount Configuration

Updates an existing mount point.

**Example:**
```python
dbutils.fs.updateMount(
    source="wasbs://mycontainer@myaccount.blob.core.windows.net",
    mountPoint="/mnt/mydata",
    extraConfigs={"fs.azure.account.key.myaccount.blob.core.windows.net": "new-access-key"}
)
```

---

## dbutils.widgets - Interactive Widgets

Create interactive input controls within notebooks for parameterization.

### Widget Types and Methods

#### 1. `dbutils.widgets.text()` - Text Input Widget

**Syntax:**
```python
dbutils.widgets.text(name: str, defaultValue: str, label: str)
```

**Example:**
```python
# Create text widget
dbutils.widgets.text("file_path", "/mnt/data/input.csv", "Enter File Path")

# Get value
file_path = dbutils.widgets.get("file_path")
print(f"File Path: {file_path}")
```

#### 2. `dbutils.widgets.dropdown()` - Dropdown Widget

**Syntax:**
```python
dbutils.widgets.dropdown(name: str, defaultValue: str, choices: list, label: str)
```

**Example:**
```python
# Create dropdown widget
dbutils.widgets.dropdown("processing_mode", "fast", ["fast", "medium", "slow"], "Processing Mode")

# Get selected value
mode = dbutils.widgets.get("processing_mode")
print(f"Selected Mode: {mode}")
```

#### 3. `dbutils.widgets.combobox()` - Combobox Widget

**Syntax:**
```python
dbutils.widgets.combobox(name: str, defaultValue: str, choices: list, label: str)
```

**Example:**
```python
# Create combobox (allows custom input)
dbutils.widgets.combobox("country", "USA", ["USA", "Canada", "Mexico"], "Select Country")

# Get value
country = dbutils.widgets.get("country")
print(f"Country: {country}")
```

#### 4. `dbutils.widgets.multiselect()` - Multi-Select Widget

**Syntax:**
```python
dbutils.widgets.multiselect(name: str, defaultValue: str, choices: list, label: str)
```

**Example:**
```python
# Create multiselect widget
dbutils.widgets.multiselect("columns", "age", ["age", "salary", "department"], "Select Columns")

# Get selected values (comma-separated string)
selected_cols = dbutils.widgets.get("columns")
columns_list = selected_cols.split(",")
print(f"Selected Columns: {columns_list}")
```

### Widget Management

#### Get Widget Values

```python
# Get single widget value
value = dbutils.widgets.get("widget_name")

# Get all widget values
all_values = dbutils.widgets.getAll()
print("All widget values:", all_values)
```

#### Remove Widgets

```python
# Remove specific widget
dbutils.widgets.remove("widget_name")

# Remove all widgets
dbutils.widgets.removeAll()
```

---

## dbutils.notebook - Notebook Management

Utilities for running and managing notebooks within workflows.

### Available Methods

```python
dbutils.notebook.help()
```

**Output:**
```
The notebook module.
exit(value: String): void -> Exit a notebook with a value
run(path: String, timeoutSeconds: int, arguments: Map): String -> Run a notebook and return its exit value
```

#### 1. `dbutils.notebook.run()` - Run Another Notebook

**Syntax:**
```python
dbutils.notebook.run(path: str, timeout_seconds: int, arguments: dict = None)
```

**Parameters:**
- `path`: Path to the notebook to run
- `timeout_seconds`: Maximum execution time (0 = no timeout)
- `arguments`: Parameters to pass to the notebook

**Example:**
```python
# Run notebook with parameters
result = dbutils.notebook.run(
    "/Users/example@company.com/DataProcessing", 
    300, 
    {
        "inputPath": "/mnt/data/input", 
        "outputPath": "/mnt/data/output",
        "environment": "production"
    }
)
print(f"Notebook result: {result}")
```

#### 2. `dbutils.notebook.exit()` - Exit Notebook

**Syntax:**
```python
dbutils.notebook.exit(value: str)
```

**Example:**
```python
# Process data
try:
    # Your data processing logic here
    print("Data processing completed successfully.")
    dbutils.notebook.exit("SUCCESS")
except Exception as e:
    print(f"Error occurred: {e}")
    dbutils.notebook.exit("FAILED")
```

---

## %run Command

Execute notebooks as if they were part of the current notebook.

### Basic Usage

```python
# Run another notebook
%run /path/to/notebook

# Run notebook with parameters
%run /path/to/notebook $param1="value1" $param2="value2"

# Dynamic path
notebook_path = "/path/to/utility_notebook"
%run $notebook_path
```

### Parameter Handling

**In the called notebook (receiving parameters):**
```python
# Define widget parameters
dbutils.widgets.text("param1", "default_value1")
dbutils.widgets.text("param2", "default_value2")

# Get parameter values
param1 = dbutils.widgets.get("param1")
param2 = dbutils.widgets.get("param2")

print(f"Param1: {param1}")
print(f"Param2: {param2}")
```

**In the calling notebook:**
```python
# Call notebook with parameters
%run /path/to/child_notebook $param1="production" $param2="2023-12-31"
```

### Practical Example

**Utility Notebook (`/utils/common_functions`):**
```python
def clean_data(df):
    """Clean and standardize dataframe"""
    return df.dropna().drop_duplicates()

def calculate_metrics(df):
    """Calculate business metrics"""
    return {
        "total_rows": df.count(),
        "avg_value": df.select("value").mean()
    }

# Configuration
DATABASE_NAME = "analytics_db"
TABLE_PREFIX = "clean_"
```

**Main Notebook:**
```python
# Import utility functions
%run /utils/common_functions

# Now use the imported functions
df = spark.table("raw_data")
clean_df = clean_data(df)
metrics = calculate_metrics(clean_df)

print(f"Metrics: {metrics}")
print(f"Saving to: {DATABASE_NAME}.{TABLE_PREFIX}processed_data")
```

---

## dbutils.secrets - Secret Management

Securely store and retrieve sensitive information like passwords, API keys, and tokens.

### Key Methods

#### 1. `dbutils.secrets.get()` - Retrieve Secret

**Syntax:**
```python
dbutils.secrets.get(scope: str, key: str) -> str
```

**Example:**
```python
# Get API key
api_key = dbutils.secrets.get(scope="api-secrets", key="openai-api-key")

# Use in API call
import requests
headers = {"Authorization": f"Bearer {api_key}"}
response = requests.get("https://api.openai.com/v1/models", headers=headers)
```

#### 2. `dbutils.secrets.getBytes()` - Retrieve Secret as Bytes

**Syntax:**
```python
dbutils.secrets.getBytes(scope: str, key: str) -> bytes
```

#### 3. `dbutils.secrets.list()` - List Secrets in Scope

**Syntax:**
```python
dbutils.secrets.list(scope: str)
```

**Example:**
```python
# List all secrets in scope (metadata only, not values)
secrets = dbutils.secrets.list(scope="db-secrets")
for secret in secrets:
    print(f"Secret name: {secret.key}")
```

#### 4. `dbutils.secrets.listScopes()` - List All Secret Scopes

**Example:**
```python
# List all available secret scopes
scopes = dbutils.secrets.listScopes()
for scope in scopes:
    print(f"Scope: {scope.name}, Backend: {scope.backendType}")
```

### Practical Examples

#### Database Connection

```python
# Retrieve database credentials
db_host = dbutils.secrets.get(scope="database", key="host")
db_user = dbutils.secrets.get(scope="database", key="username")
db_password = dbutils.secrets.get(scope="database", key="password")

# Create connection string
connection_string = f"postgresql://{db_user}:{db_password}@{db_host}:5432/analytics"

# Use with Spark
df = spark.read \
    .format("jdbc") \
    .option("url", connection_string) \
    .option("dbtable", "customers") \
    .load()
```

#### Cloud Storage Access

```python
# Get cloud storage credentials
storage_key = dbutils.secrets.get(scope="azure-storage", key="account-key")

# Configure Spark to use the credentials
spark.conf.set(
    "fs.azure.account.key.mystorageaccount.blob.core.windows.net",
    storage_key
)

# Now read from Azure Blob Storage
df = spark.read.parquet("abfss://container@mystorageaccount.dfs.core.windows.net/data/")
```

---

## Creating Secret Scopes

### Method 1: Using UI with #secrets/createScope

1. Navigate to your Databricks workspace
2. Append `#secrets/createScope` to your workspace URL:
   ```
   https://your-databricks-instance#secrets/createScope
   ```
3. Fill in the form:
   - **Scope Name**: Choose a unique name (e.g., `my-api-secrets`)
   - **Manage Principal**: Set access permissions
4. Click "Create"

### Method 2: Using Databricks CLI

```bash
# Create secret scope
databricks secrets create-scope --scope my-api-secrets

# Add secret to scope
databricks secrets put --scope my-api-secrets --key openai-api-key

# List secrets in scope
databricks secrets list --scope my-api-secrets
```

### Method 3: Azure Key Vault-Backed Scope

```bash
# Create Azure Key Vault-backed scope
databricks secrets create-scope --scope my-keyvault-scope \
    --scope-backend-type AZURE_KEYVAULT \
    --resource-id /subscriptions/<subscription-id>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault-name> \
    --dns-name https://<vault-name>.vault.azure.net/
```

---

## Best Practices

### File System Operations
- Always use `/dbfs/` prefix when accessing DBFS from local file operations
- Use `recurse=True` carefully with `rm()` operations
- Regularly clean up temporary files and directories
- Use appropriate mount points for external storage access

### Widget Usage
- Remove widgets after use to keep notebooks clean
- Use descriptive labels for better user experience
- Validate widget inputs before processing
- Use appropriate widget types for better UX

### Notebook Management
- Use meaningful exit values for workflow orchestration
- Handle timeouts appropriately in production workflows
- Pass parameters efficiently using dictionaries
- Implement proper error handling in called notebooks

### Secret Management
- **Never hardcode** sensitive information in notebooks
- Use specific secret scopes for different environments (dev, test, prod)
- Regularly rotate secrets and update scopes
- Implement proper access controls for secret scopes
- Use Azure Key Vault-backed scopes for enterprise security

### Error Handling
```python
try:
    # dbutils operations
    result = dbutils.notebook.run("/path/to/notebook", 300)
    secret_value = dbutils.secrets.get("scope", "key")
except Exception as e:
    print(f"Operation failed: {e}")
    # Implement appropriate error handling
```

### Performance Tips
- Use `recurse=False` when not needed to improve performance
- Batch file operations when possible
- Use appropriate timeout values for notebook runs
- Cache frequently accessed secrets in variables (but be mindful of security)

---

## Summary Table

| Module | Primary Use Case | Key Methods |
|--------|------------------|-------------|
| `dbutils.fs` | File system operations | `ls()`, `cp()`, `mv()`, `rm()`, `put()`, `head()` |
| `dbutils.fs.mount` | External storage integration | `mount()`, `unmount()`, `mounts()` |
| `dbutils.widgets` | Interactive notebook parameters | `text()`, `dropdown()`, `get()`, `removeAll()` |
| `dbutils.notebook` | Workflow orchestration | `run()`, `exit()` |
| `dbutils.secrets` | Secure credential management | `get()`, `list()`, `listScopes()` |
| `%run` | Code reuse and modularity | Execute notebooks with parameters |

This comprehensive guide covers all essential dbutils functionality for effective Databricks development and operations.
