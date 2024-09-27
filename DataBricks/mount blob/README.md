<h1>Connecting With Blob Using Scope and Keyvault</h1>
<p><strong>Step 1: Create a Storage Account</strong></p>
<ol>
  <li>Go to Azure Portal and search for <strong>Storage accounts</strong>.</li>
  <li>Click <strong>Create</strong> and follow the prompts to set up your storage account.</li>
  <li>Once created, go to the storage account and copy the <strong>Access keys</strong> for later use.</li>
</ol>

<p><strong>Step 2: Create a Key Vault</strong></p>
<ol>
  <li>Search for <strong>Key Vault</strong> in Azure Portal and click <strong>Create</strong>.</li>
  <li>Enter the required details like <strong>Key Vault Name</strong> and <strong>Resource Group</strong>, then click <strong>Create</strong>.</li>
  <li>Once created, go to your Key Vault and select <strong>Secrets</strong> under the Settings section.</li>
  <li>Click <strong>Generate/Import</strong> and store your <strong>Storage Account Access Key</strong>.</li>
</ol>

<p><strong>Step 3: Create a Secret Scope in Databricks</strong></p>
<ol>
  <li>In Databricks, use the following URL to access the Secret Scope creation page: 
    <code>https://<databricks-instance>#secrets/createScope</code></li>
  <li>Enter a name for the secret scope, for example, <strong>my-secret-scope</strong>.</li>
  <li>If you're using Azure Key Vault, provide the <strong>Key Vault DNS Name</strong> and <strong>Resource ID</strong>.</li>
  <li>Click <strong>Create</strong> to finish setting up the scope.</li>
</ol>

<p><strong>Step 4: Mount Azure Blob Storage in Databricks</strong></p>
<ol>
  <li>In a Databricks notebook, use the following code to mount the Azure Blob Storage:</li>
</ol>

<code>
  storage_account_name = "<your-storage-account-name>"<br>
  container_name = "<your-container-name>"<br>
  mount_point = "/mnt/<your-mount-name>"<br>
  access_key = dbutils.secrets.get(scope="my-secret-scope", key="my-storage-access-key")<br>
  dbutils.fs.mount(<br>
  &nbsp;&nbsp;source=f"wasbs://{container_name}@{storage_account_name}.blob.core.windows.net/",<br>
  &nbsp;&nbsp;mount_point=mount_point,<br>
  &nbsp;&nbsp;extra_configs={f"fs.azure.account.key.{storage_account_name}.blob.core.windows.net": access_key}<br>
  )
</code>

<p><strong>Step 5: Verify and Perform Operations</strong></p>
<ol>
  <li>To verify the mount, list the contents using:</li>
</ol>

<code>display(dbutils.fs.ls(mount_point))</code>

<p><strong>Step 6: Perform Operations on Mounted Storage (DBFS vs Blob)</strong></p>
<p>Now you can perform various file operations. Here are examples of common operations for both Blob Storage and DBFS:</p>

<ul>
  <li><strong>Read a file from Blob Storage (DBFS):</strong></li>
  <code>
    file_path = f"{mount_point}/path/to/file.csv"<br>
    df = spark.read.csv(file_path)<br>
    display(df)
  </code>

  <li><strong>Write a file to Blob Storage (DBFS):</strong></li>
  <code>
    output_path = f"{mount_point}/path/to/save/output.csv"<br>
    df.write.csv(output_path)
  </code>

  <li><strong>List files in Blob Storage:</strong></li>
  <code>
    dbutils.fs.ls(mount_point)
  </code>

  <li><strong>Delete a file from Blob Storage:</strong></li>
  <code>
    file_to_delete = f"{mount_point}/path/to/file.csv"<br>
    dbutils.fs.rm(file_to_delete, True)
  </code>

  <li><strong>Move a file within Blob Storage:</strong></li>
  <code>
    source_path = f"{mount_point}/path/to/source.csv"<br>
    destination_path = f"{mount_point}/path/to/destination.csv"<br>
    dbutils.fs.mv(source_path, destination_path)
  </code>

  <li><strong>Unmount the storage:</strong></li>
  <code>
    dbutils.fs.unmount(mount_point)
  </code>
</ul>

<p>By mounting Blob Storage to Databricks, you can perform all types of operations on files just like working on a local file system. These operations make it easy to manage data stored in Azure Blob Storage directly from Databricks.</p>


<h1>Connecting Data Lake To Databricks using scope,app registration,keyvault</h1>
<p><strong>Step 1: Create a Storage Account</strong></p>
<ol>
  <li>Go to the <strong>Azure Portal</strong> and search for <strong>Storage accounts</strong>.</li>
  <li>Click <strong>Create</strong> and follow the prompts to set up a new storage account.
    <ul>
      <li><strong>Storage Account Type</strong>: Choose <code>StorageV2</code> for general-purpose use.</li>
      <li><strong>Performance</strong>: Select between <strong>Standard</strong> or <strong>Premium</strong>.</li>
    </ul>
  </li>
  <li>Create a <strong>Container</strong> in your storage account (e.g., <code>my-container</code>).</li>
</ol>

<p><strong>Step 2: Create an App Registration in Azure Active Directory</strong></p>
<ol>
  <li>In the Azure Portal, search for <strong>App Registrations</strong>.</li>
  <li>Click <strong>New Registration</strong>, provide a name (e.g., <code>databricks-app</code>), and click <strong>Register</strong>.</li>
  <li>Copy the following values:
    <ul>
      <li><strong>Application (client) ID</strong></li>
      <li><strong>Directory (tenant) ID</strong></li>
    </ul>
  </li>
  <li>Navigate to <strong>Certificates & Secrets</strong> and create a new <strong>Client Secret</strong>. Copy the secret for later use.</li>
</ol>

<p><strong>Step 3: Assign Roles for App Registration</strong></p>
<ol>
  <li>Go to the <strong>Storage Account</strong> created in Step 1.</li>
  <li>Under the <strong>Access Control (IAM)</strong> section, click <strong>Add Role Assignment</strong>.</li>
  <li>Assign the <strong>Storage Blob Data Contributor</strong> role to the App Registration.</li>
</ol>

<p><strong>Step 4: Create a Key Vault</strong></p>
<ol>
  <li>In the Azure Portal, search for <strong>Key Vault</strong> and create a new one.</li>
  <li>Once created, go to <strong>Secrets</strong> and store the following values:
    <ul>
      <li><strong>App ID</strong> (Application Client ID)</li>
      <li><strong>Client Secret</strong> (Generated in App Registration)</li>
      <li><strong>Directory ID</strong> (Tenant ID)</li>
    </ul>
  </li>
</ol>

<p><strong>Step 5: Store Secrets in Key Vault</strong></p>
<ol>
  <li>Add the App Registration's <strong>Client ID</strong>, <strong>Client Secret</strong>, and <strong>Directory ID</strong> as secrets in the Key Vault.</li>
  <li>For example:
    <ul>
      <li>Secret Name: <code>app-id</code>, <code>client-secret</code>, <code>directory-id</code></li>
    </ul>
  </li>
</ol>

<p><strong>Step 6: Create a Databricks Secret Scope to Connect to Key Vault</strong></p>
<ol>
  <li>In Databricks, navigate to the URL: <code>https://&lt;databricks-instance&gt;#secrets/createScope</code>.</li>
  <li>Create a scope (e.g., <code>my-keyvault-scope</code>) and provide the Key Vault URI and resource ID.</li>
</ol>

<p><strong>Step 7: Mount ADLS in Databricks</strong></p>
<p>Now, use the following Python code in Databricks to mount the Azure Data Lake Storage:</p>

<code>
configs = {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"fs.azure.account.auth.type": "OAuth",<br>
&nbsp;&nbsp;&nbsp;&nbsp;"fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",<br>
&nbsp;&nbsp;&nbsp;&nbsp;"fs.azure.account.oauth2.client.id": dbutils.secrets.get(scope="my-keyvault-scope", key="app-id"),<br>
&nbsp;&nbsp;&nbsp;&nbsp;"fs.azure.account.oauth2.client.secret": dbutils.secrets.get(scope="my-keyvault-scope", key="client-secret"),<br>
&nbsp;&nbsp;&nbsp;&nbsp;"fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/" + dbutils.secrets.get(scope="my-keyvault-scope", key="directory-id") + "/oauth2/token"<br>
}<br><br>

dbutils.fs.mount(<br>
&nbsp;&nbsp;&nbsp;&nbsp;source = "abfss://&lt;container-name&gt;@&lt;storage-account-name&gt;.dfs.core.windows.net/",<br>
&nbsp;&nbsp;&nbsp;&nbsp;mount_point = "/mnt/&lt;mount-name&gt;",<br>
&nbsp;&nbsp;&nbsp;&nbsp;extra_configs = configs<br>
)
</code>

<p>Replace <code>&lt;container-name&gt;</code>, <code>&lt;storage-account-name&gt;</code>, and <code>&lt;mount-name&gt;</code> with your specific values.</p>

<p><strong>Step 8: Perform File Operations</strong></p>
<p>Once the storage is mounted, you can perform various file operations. Below are some common operations:</p>

<ul>
  <li><strong>List Files:</strong></li>
  <code>display(dbutils.fs.ls("/mnt/&lt;mount-name&gt;"))</code>

  <li><strong>Read a CSV File:</strong></li>
  <code>df = spark.read.csv("/mnt/&lt;mount-name&gt;/path/to/file.csv", header=True)<br>
  display(df)</code>

  <li><strong>Write a CSV File:</strong></li>
  <code>df.write.csv("/mnt/&lt;mount-name&gt;/path/to/output.csv")</code>

  <li><strong>Delete a File:</strong></li>
  <code>dbutils.fs.rm("/mnt/&lt;mount-name&gt;/path/to/file.csv", True)</code>

  <li><strong>Move a File:</strong></li>
  <code>dbutils.fs.mv("/mnt/&lt;mount-name&gt;/path/to/source.csv", "/mnt/&lt;mount-name&gt;/path/to/destination.csv")</code>

  <li><strong>Unmount the Storage:</strong></li>
  <code>dbutils.fs.unmount("/mnt/&lt;mount-name&gt;")</code>
</ul>

<h1>Difference Between Both Methods</h1>
<p><strong>1. Authentication Method</strong></p>
<ul>
  <li><strong>App Registration</strong>:
    <ul>
      <li><strong>Uses OAuth-based authentication</strong>.</li>
      <li>You register an application in Azure Active Directory (AAD) and provide it with the necessary permissions (role assignments).</li>
      <li>The App Registration has its own identity in Azure (i.e., the Application ID and Client Secret).</li>
      <li>You authenticate using this identity (Client ID and Client Secret) via OAuth tokens.</li>
    </ul>
  </li>
  <li><strong>Direct Key Vault Access</strong>:
    <ul>
      <li><strong>Uses access keys or secrets directly</strong>.</li>
      <li>Secrets like storage account access keys or connection strings are stored directly in Azure Key Vault.</li>
      <li>Databricks or other services retrieve these secrets securely from Key Vault using the secret scope.</li>
      <li><strong>Authentication is handled via the Databricks-to-Key Vault secret integration</strong>, but there is no separate identity.</li>
    </ul>
  </li>
</ul>

<p><strong>2. Security and Flexibility</strong></p>
<ul>
  <li><strong>App Registration</strong>:
    <ul>
      <li><strong>More secure and scalable</strong>: It allows fine-grained control with roles and permissions assigned via Azure AD.</li>
      <li><strong>Service Principal Identity</strong>: The App Registration acts as a service principal, which can be granted specific roles like <strong>Storage Blob Data Contributor</strong> or <strong>Reader</strong>.</li>
      <li><strong>Flexible in multi-resource access</strong>: The App Registration can have access to multiple Azure services (like Azure Data Lake, SQL, etc.), which makes it reusable across different services.</li>
    </ul>
  </li>
  <li><strong>Direct Key Vault</strong>:
    <ul>
      <li><strong>Less secure</strong>: You store sensitive connection strings or keys in Key Vault, and these are retrieved when needed. However, if compromised, this exposes the sensitive information directly.</li>
      <li><strong>Limited control over resource access</strong>: You’re primarily managing individual secrets (e.g., storage access keys), so it lacks the flexibility of role-based access control (RBAC) that App Registration provides.</li>
    </ul>
  </li>
</ul>

<p><strong>3. Use Case Scenarios</strong></p>
<ul>
  <li><strong>App Registration</strong>:
    <ul>
      <li>Suitable for <strong>enterprise-level applications</strong> where you need to access multiple Azure resources and services securely using role-based permissions.</li>
      <li>Ideal for cases where you want to centralize identity management and minimize direct handling of secrets like access keys.</li>
    </ul>
  </li>
  <li><strong>Direct Key Vault</strong>:
    <ul>
      <li>Suitable for <strong>simpler applications</strong> where you only need access to specific secrets, such as storage access keys or connection strings.</li>
      <li>Best used when you want to leverage Key Vault to store sensitive data but don’t need the flexibility or security of using an App Registration.</li>
    </ul>
  </li>
</ul>

<p><strong>4. Rotation and Expiry</strong></p>
<ul>
  <li><strong>App Registration</strong>:
    <ul>
      <li><strong>Token-based authentication</strong>: OAuth tokens are temporary and refresh automatically, ensuring more security and less manual intervention.</li>
      <li>The Client Secret has a defined expiration, but tokens are short-lived and thus reduce risk in case of compromise.</li>
    </ul>
  </li>
  <li><strong>Direct Key Vault</strong>:
    <ul>
      <li>When using <strong>keys and secrets</strong> directly, they remain static until manually rotated.</li>
      <li>Keys like storage account access keys need to be managed (rotated) periodically, and manual updates are required in your system.</li>
    </ul>
  </li>
</ul>

<p><strong>Summary:</strong></p>
<ul>
  <li><strong>App Registration</strong> is more secure, scalable, and flexible, offering OAuth-based authentication via Azure AD. It's the recommended approach for enterprise applications where access to multiple resources and fine-grained control is required.</li>
  <li><strong>Direct Key Vault access</strong> is simpler and useful for storing and retrieving specific secrets, but offers less control and flexibility compared to App Registration.</li>
</ul>
