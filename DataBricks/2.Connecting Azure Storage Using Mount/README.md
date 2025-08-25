# Azure Databricks Storage Connectivity Guide

This guide provides step-by-step instructions for connecting Azure Databricks to Azure Storage services using two different authentication methods.

## Table of Contents
- [Method 1: Blob Storage with Access Keys](#method-1-blob-storage-with-access-keys)
- [Method 2: Data Lake with App Registration (Recommended)](#method-2-data-lake-with-app-registration-recommended)
- [Comparison Between Methods](#comparison-between-methods)
- [Common Operations](#common-operations)

---

## Method 1: Blob Storage with Access Keys

### Prerequisites
- Azure subscription
- Azure Databricks workspace
- Appropriate permissions to create Azure resources

### Step 1: Create a Storage Account

1. Navigate to **Azure Portal** → **Storage accounts**
2. Click **Create** and configure:
   - Enter required details (name, resource group, region)
   - Choose appropriate performance tier
3. Once created, go to the storage account
4. Navigate to **Access keys** and copy the key for later use

### Step 2: Create a Key Vault

1. Search for **Key Vault** in Azure Portal
2. Click **Create** and enter:
   - Key Vault Name
   - Resource Group
   - Region
3. After creation, assign roles:
   - Assign yourself **Key Vault Administrator** role
   - Assign **azuredatabricks** **Key Vault Administrator** role
4. Navigate to **Secrets** → **Generate/Import**
5. Store your **Storage Account Access Key** as a secret

### Step 3: Create Secret Scope in Databricks

1. Access the secret scope creation page:
   ```
   https://<databricks-instance>#secrets/createScope
   ```
2. Configure the scope:
   - **Scope Name**: `my-secret-scope`
   - **Key Vault DNS Name**: (from Key Vault properties)
   - **Resource ID**: (from Key Vault properties)
3. Click **Create**

### Step 4: Mount Azure Blob Storage

```python
# Configuration parameters
storage_account_name = "<your-storage-account-name>"
container_name = "<your-container-name>"
mount_point = "/mnt/<your-mount-name>"

# Retrieve access key from Key Vault
access_key = dbutils.secrets.get(scope="my-secret-scope", key="my-storage-access-key")

# Mount the storage
dbutils.fs.mount(
    source=f"wasbs://{container_name}@{storage_account_name}.blob.core.windows.net/",
    mount_point=mount_point,
    extra_configs={f"fs.azure.account.key.{storage_account_name}.blob.core.windows.net": access_key}
)
```

### Step 5: Verify the Mount

```python
# List contents to verify mount
display(dbutils.fs.ls(mount_point))
```

---

## Method 2: Data Lake with App Registration (Recommended)

### Step 1: Create a Storage Account

1. Navigate to **Azure Portal** → **Storage accounts**
2. Click **Create** and configure:
   - **Storage Account Type**: `StorageV2`
   - **Performance**: Standard or Premium
3. Create a **Container** (e.g., `my-container`)

### Step 2: Create App Registration

1. Search for **App Registrations** in Azure Portal
2. Click **New Registration**:
   - **Name**: `databricks-app`
   - Click **Register**
3. Copy the following values:
   - **Application (client) ID**
   - **Directory (tenant) ID**
4. Navigate to **Certificates & Secrets**
5. Create a new **Client Secret** and copy it

### Step 3: Assign Storage Permissions

1. Go to your **Storage Account**
2. Navigate to **Access Control (IAM)**
3. Click **Add Role Assignment**
4. Assign **Storage Blob Data Contributor** role to your App Registration

### Step 4: Create and Configure Key Vault

1. Create a new **Key Vault** in Azure Portal
2. Assign roles:
   - Assign yourself **Key Vault Administrator** role
   - Assign **azuredatabricks** **Key Vault Administrator** role
3. Navigate to **Secrets** and create:
   - **app-id**: Application Client ID
   - **client-secret**: Generated Client Secret
   - **directory-id**: Tenant ID

### Step 5: Create Databricks Secret Scope

1. Navigate to:
   ```
   https://<databricks-instance>#secrets/createScope
   ```
2. Create scope:
   - **Name**: `my-keyvault-scope`
   - **Key Vault URI**: (from Key Vault properties)
   - **Resource ID**: (from Key Vault properties)

### Step 6: Mount Azure Data Lake Storage

```python
# OAuth configuration
configs = {
    "fs.azure.account.auth.type": "OAuth",
    "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
    "fs.azure.account.oauth2.client.id": dbutils.secrets.get(scope="my-keyvault-scope", key="app-id"),
    "fs.azure.account.oauth2.client.secret": dbutils.secrets.get(scope="my-keyvault-scope", key="client-secret"),
    "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/" + dbutils.secrets.get(scope="my-keyvault-scope", key="directory-id") + "/oauth2/token"
}

# Mount the storage
dbutils.fs.mount(
    source="abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/",
    mount_point="/mnt/<mount-name>",
    extra_configs=configs
)
```

---

## Common Operations

Once your storage is mounted, you can perform various file operations:

### List Files
```python
display(dbutils.fs.ls("/mnt/<mount-name>"))
```

### Read Data
```python
# Read CSV file
df = spark.read.csv("/mnt/<mount-name>/path/to/file.csv", header=True)
display(df)
```

### Write Data
```python
# Write DataFrame to CSV
df.write.csv("/mnt/<mount-name>/path/to/output.csv")
```

### File Management
```python
# Delete a file
dbutils.fs.rm("/mnt/<mount-name>/path/to/file.csv", True)

# Move a file
dbutils.fs.mv(
    "/mnt/<mount-name>/path/to/source.csv",
    "/mnt/<mount-name>/path/to/destination.csv"
)

# Copy a file
dbutils.fs.cp(
    "/mnt/<mount-name>/path/to/source.csv",
    "/mnt/<mount-name>/path/to/copy.csv"
)
```

### Unmount Storage
```python
dbutils.fs.unmount("/mnt/<mount-name>")
```

---

## Comparison Between Methods

| Aspect | Access Keys Method | App Registration Method |
|--------|-------------------|------------------------|
| **Authentication** | Direct access keys | OAuth-based tokens |
| **Security** | Less secure (static keys) | More secure (temporary tokens) |
| **Scalability** | Limited to specific resources | Multi-resource access |
| **Management** | Manual key rotation | Automatic token refresh |
| **Complexity** | Simpler setup | More complex but flexible |
| **Enterprise Ready** | Basic use cases | Enterprise applications |

### Authentication Methods

#### Access Keys Method
- Uses storage account access keys directly
- Secrets stored in Key Vault for secure retrieval
- Authentication via Databricks-to-Key Vault integration
- No separate identity management

#### App Registration Method
- Uses OAuth-based authentication through Azure AD
- Creates service principal identity with specific permissions
- Role-based access control (RBAC)
- Centralized identity management

### Security Considerations

#### Access Keys Method
- **Pros**: Simple to implement and understand
- **Cons**: Static keys with full storage access, manual rotation required

#### App Registration Method
- **Pros**: Fine-grained permissions, automatic token refresh, enterprise-grade security
- **Cons**: More complex setup and configuration

### Use Case Recommendations

#### Choose Access Keys Method When:
- Simple applications with basic storage needs
- Limited scope of access required
- Quick prototyping or development environments

#### Choose App Registration Method When:
- Enterprise-level applications
- Multiple Azure resource access needed
- Enhanced security requirements
- Production environments

---

## Best Practices

1. **Security**
   - Always use Key Vault for storing sensitive information
   - Regularly rotate secrets and keys
   - Use least privilege principle for role assignments

2. **Monitoring**
   - Monitor access patterns and unusual activities
   - Set up alerts for unauthorized access attempts

3. **Documentation**
   - Document your secret scope names and Key Vault configurations
   - Maintain an inventory of mounted storage accounts

4. **Testing**
   - Test connectivity before deploying to production
   - Verify permissions are working as expected

---

## Troubleshooting

### Common Issues

1. **Mount Failed**: Check permissions and secret values
2. **Access Denied**: Verify role assignments and scope configuration
3. **Secret Not Found**: Ensure secret names match exactly
4. **Connection Timeout**: Check network connectivity and firewall rules

### Debug Commands

```python
# Check existing mounts
display(dbutils.fs.mounts())

# Test secret retrieval
test_secret = dbutils.secrets.get(scope="my-scope", key="my-key")
print("Secret retrieved successfully" if test_secret else "Failed to retrieve secret")

# List available secret scopes
dbutils.secrets.listScopes()
```
