# Databricks dbutils - Quick Revision Notes

## 📋 Overview
**dbutils** = Databricks utility functions for file system, notebooks, widgets, secrets, and jobs.

---

## 🔍 Getting Help
```python
dbutils.help()                    # All available modules
dbutils.fs.help()                # File system help
dbutils.notebook.help()          # Notebook help
```

---

## 📁 dbutils.fs (File System Operations)

### Basic File Operations
| Command | Purpose | Example |
|---------|---------|---------|
| `dbutils.fs.ls(path)` | List files/directories | `dbutils.fs.ls("/mnt/data")` |
| `dbutils.fs.cp(src, dest)` | Copy files | `dbutils.fs.cp("/src/file.csv", "/dest/file.csv")` |
| `dbutils.fs.mv(src, dest)` | Move/rename files | `dbutils.fs.mv("/old/file.csv", "/new/file.csv")` |
| `dbutils.fs.rm(path)` | Delete files | `dbutils.fs.rm("/temp/file.csv")` |
| `dbutils.fs.put(path, content)` | Write to file | `dbutils.fs.put("/data/test.txt", "Hello World")` |
| `dbutils.fs.head(path)` | Read file content | `dbutils.fs.head("/data/file.txt", maxBytes=1000)` |
| `dbutils.fs.mkdirs(path)` | Create directories | `dbutils.fs.mkdirs("/new/folder/path")` |

### Important Parameters
- `recurse=True` - For recursive operations (copy/move/delete directories)
- `overwrite=True` - To overwrite existing files

### Mount Operations
| Command | Purpose | Example |
|---------|---------|---------|
| `dbutils.fs.mount()` | Mount external storage | Mount Azure Blob/S3 to DBFS |
| `dbutils.fs.mounts()` | List all mounts | See current mount points |
| `dbutils.fs.unmount(path)` | Remove mount | `dbutils.fs.unmount("/mnt/storage")` |
| `dbutils.fs.refreshMounts()` | Refresh mount cache | Update all nodes |
| `dbutils.fs.updateMount()` | Update mount config | Change credentials/settings |

### Mount Example (Azure Blob)
```python
dbutils.fs.mount(
  source="wasbs://container@account.blob.core.windows.net",
  mount_point="/mnt/data",
  extra_configs={"fs.azure.account.key.account.blob.core.windows.net": "key"}
)
```

---

## 🎛️ dbutils.widgets (Interactive Controls)

### Widget Types
| Type | Command | Purpose |
|------|---------|---------|
| **Text** | `dbutils.widgets.text(name, default, label)` | Free text input |
| **Dropdown** | `dbutils.widgets.dropdown(name, default, choices, label)` | Select one option |
| **Combobox** | `dbutils.widgets.combobox(name, default, choices, label)` | Select or type custom |
| **Multiselect** | `dbutils.widgets.multiselect(name, default, choices, label)` | Select multiple |

### Widget Management
```python
# Get widget value
value = dbutils.widgets.get("widget_name")

# Get all widget values
all_values = dbutils.widgets.getAll()

# Remove specific widget
dbutils.widgets.remove("widget_name")

# Remove all widgets
dbutils.widgets.removeAll()
```

### Quick Example
```python
# Create dropdown
dbutils.widgets.dropdown("env", "dev", ["dev", "test", "prod"], "Environment")

# Use value
environment = dbutils.widgets.get("env")
```

---

## 📓 dbutils.notebook (Notebook Management)

### Key Functions
| Function | Purpose | Syntax |
|----------|---------|---------|
| `run()` | Execute another notebook | `dbutils.notebook.run(path, timeout, args)` |
| `exit()` | Exit with return value | `dbutils.notebook.exit("SUCCESS")` |

### Notebook Run Example
```python
# Run notebook with parameters
result = dbutils.notebook.run(
    "/path/to/notebook", 
    300,  # timeout in seconds
    {"param1": "value1", "param2": "value2"}
)
print(f"Result: {result}")
```

### Exit Example
```python
try:
    # Process data
    dbutils.notebook.exit("SUCCESS")
except Exception as e:
    dbutils.notebook.exit(f"FAILED: {str(e)}")
```

---

## ▶️ %run Command (Code Reuse)

### Basic Usage
```python
# Run notebook
%run /path/to/notebook

# Run with parameters
%run /path/to/notebook $param1="value1" $param2="value2"

# Dynamic path
notebook_path = "/utils/common"
%run $notebook_path
```

### Parameter Handling
**In called notebook:**
```python
# Define parameters
dbutils.widgets.text("param1", "default")
param1 = dbutils.widgets.get("param1")
```

**Key Points:**
- All variables/functions from called notebook become available
- Use for utility functions and common code
- Parameters passed via widgets

---

## 🔐 dbutils.secrets (Secret Management)

### Core Functions
| Function | Purpose | Example |
|----------|---------|---------|
| `get(scope, key)` | Retrieve secret value | `dbutils.secrets.get("db", "password")` |
| `getBytes(scope, key)` | Get secret as bytes | For binary secrets |
| `list(scope)` | List secrets in scope | `dbutils.secrets.list("api-keys")` |
| `listScopes()` | List all scopes | See available scopes |

### Common Usage Patterns
```python
# Database connection
db_user = dbutils.secrets.get("database", "username")
db_pass = dbutils.secrets.get("database", "password")

# API keys
api_key = dbutils.secrets.get("apis", "openai-key")

# Cloud storage
storage_key = dbutils.secrets.get("azure", "storage-key")
```

### Error Handling
```python
try:
    secret = dbutils.secrets.get("scope", "key")
except Exception as e:
    print(f"Secret not found: {e}")
```

---

## 🆕 Creating Secret Scopes

### Method 1: UI (Easiest)
1. Go to workspace URL + `#secrets/createScope`
2. Fill form: Scope name, permissions
3. Click "Create"

### Method 2: CLI
```bash
# Create scope
databricks secrets create-scope --scope my-secrets

# Add secret
databricks secrets put --scope my-secrets --key api-key
```

### Method 3: Azure Key Vault
```bash
databricks secrets create-scope --scope keyvault-scope \
  --scope-backend-type AZURE_KEYVAULT \
  --resource-id /subscriptions/.../vaults/vault-name
```

---

## ⚡ Quick Reference Commands

### Most Used File Operations
```python
# List files
files = dbutils.fs.ls("/mnt/data")

# Copy file
dbutils.fs.cp("/src/data.csv", "/dest/data.csv")

# Delete directory
dbutils.fs.rm("/temp/", recurse=True)
```

### Most Used Widget Pattern
```python
# Create and use widget
dbutils.widgets.text("file_path", "/default/path")
path = dbutils.widgets.get("file_path")
```

### Most Used Secret Pattern
```python
# Get secret for database
password = dbutils.secrets.get("db-secrets", "password")
```

### Most Used Notebook Pattern
```python
# Run child notebook
result = dbutils.notebook.run("/etl/process_data", 600, {"date": "2023-12-01"})
```

---

## 🎯 Key Concepts to Remember

### File System
- **DBFS paths**: Start with `/` or `dbfs:/`
- **Mount points**: External storage as `/mnt/name`
- **Recursive operations**: Use `recurse=True`

### Widgets
- **Create once**: Define at notebook start
- **Get values**: Use `dbutils.widgets.get()`
- **Clean up**: Use `removeAll()` when done

### Notebooks
- **Timeout**: Always set reasonable timeout
- **Parameters**: Pass as dictionary
- **Return values**: Use `exit()` to return results

### Secrets
- **Never hardcode**: Always use secret scopes
- **Scope organization**: Separate by environment/purpose
- **Access control**: Manage permissions carefully

---

## 🚨 Common Mistakes to Avoid

1. **Forgetting `recurse=True`** when deleting directories
2. **Not handling timeouts** in notebook.run()
3. **Hardcoding secrets** instead of using scopes
4. **Not cleaning up widgets** after use
5. **Using wrong paths** (local vs DBFS)
6. **Not checking if files exist** before operations

---

## 💡 Pro Tips

1. **Use `%fs ls /path`** for quick file listing in cells
2. **Mount external storage** for better performance than direct access
3. **Create utility notebooks** with common functions and `%run` them
4. **Use widgets** to make notebooks reusable
5. **Store all secrets** in scopes, never in code
6. **Test with short timeouts** first, then increase
7. **Use meaningful scope names** like `prod-db`, `dev-apis`

---

## 📊 Command Priority for Exams/Interviews

### Must Know (High Priority)
- `dbutils.fs.ls()`, `dbutils.fs.cp()`, `dbutils.fs.rm()`
- `dbutils.widgets.text()`, `dbutils.widgets.get()`
- `dbutils.secrets.get()`
- `%run` command basics

### Should Know (Medium Priority)
- `dbutils.fs.mount()`, `dbutils.fs.mounts()`
- `dbutils.notebook.run()`, `dbutils.notebook.exit()`
- `dbutils.widgets.dropdown()`, `dbutils.widgets.removeAll()`
- Secret scope creation

### Good to Know (Low Priority)
- `dbutils.fs.head()`, `dbutils.fs.put()`
- `dbutils.widgets.multiselect()`, `dbutils.widgets.combobox()`
- `dbutils.secrets.listScopes()`, `dbutils.secrets.list()`
- Advanced mount configurations
