<h1>Direct Connection Spark Session Based</h1>
<p><strong>1. Create a Blob Storage Account in Azure Portal</strong></p>
<ol>
    <li>Go to the <a href="https://portal.azure.com">Azure Portal</a>.</li>
    <li>Navigate to <strong>Storage Accounts</strong> and click <strong>Create</strong>.</li>
    <li>Choose the following options:
        <ul>
            <li><strong>Subscription:</strong> Select your subscription.</li>
            <li><strong>Resource Group:</strong> Create a new one or select an existing one.</li>
            <li><strong>Storage Account Name:</strong> Provide a globally unique name.</li>
            <li><strong>Region:</strong> Choose a nearby region.</li>
            <li><strong>Performance:</strong> Select Standard.</li>
            <li><strong>Replication:</strong> Choose Locally-Redundant Storage (LRS).</li>
        </ul>
    </li>
    <li>Review and create the storage account.</li>
</ol>

<p><strong>2. Create an Azure Key Vault and Provide Access</strong></p>
<ol>
    <li>In the Azure portal, search for <strong>Key Vault</strong> and click <strong>Create</strong>.</li>
    <li>Select the resource group and provide a unique name for the Key Vault.</li>
    <li>Once created, go to the <strong>Access policies</strong> section:
        <ul>
            <li>Click <strong>Add Access Policy</strong>.</li>
            <li>Select <strong>Key Vault Administrator</strong> role and assign it to both the Databricks service principal and your user account.</li>
        </ul>
    </li>
    <li>Save changes.</li>
</ol>

<p><strong>3. Store Storage Account Access Key in Key Vault</strong></p>
<ol>
    <li>In the Key Vault, go to <strong>Secrets</strong> under <strong>Settings</strong>.</li>
    <li>Click <strong>Generate/Import</strong> and create a secret:
        <ul>
            <li><strong>Upload Options:</strong> Manual.</li>
            <li><strong>Name:</strong> Provide a name, e.g., "storage-account-access-key".</li>
            <li><strong>Value:</strong> Paste the <strong>Access Key</strong> of your Blob Storage account, which you can find in the storage account under <strong>Access Keys</strong>.</li>
        </ul>
    </li>
</ol>

<p><strong>4. Set Up Databricks</strong></p>
<ol>
    <li>In the Azure portal, navigate to <strong>Azure Databricks</strong> and create a new Databricks workspace.</li>
    <li>Launch the workspace and create a <strong>Cluster</strong> (this is where your code will run).</li>
    <li>Open the <strong>Databricks UI</strong>, and under <strong>User Settings</strong> → <strong>Secrets</strong> tab, create a new scope:
        <ul>
            <li>Go to <code>#secrets/createScope</code> in your Databricks workspace URL.</li>
            <li>Provide a scope name (e.g., "storage-scope").</li>
            <li>Select the Key Vault you created to link secrets from.</li>
        </ul>
    </li>
</ol>

<p><strong>5. Connecting the Spark Session to Blob Storage</strong></p>
<p>Once you have your Databricks environment ready, you can start a notebook and write Python code to connect to your Blob Storage.</p>

<pre><code>
# Set Spark config to use the secret from Key Vault for accessing Blob storage
spark.conf.set(
  "fs.azure.account.key.&lt;storage-account-name&gt;.blob.core.windows.net",
  dbutils.secrets.get(scope="storage-scope", key="storage-account-access-key")
)

# Define the path to the container in Blob storage
path = "wasbs://&lt;container-name&gt;@&lt;storage-account-name&gt;.blob.core.windows.net/"

# List files in the container
files = dbutils.fs.ls(path)
for file in files:
    print(file)
</code></pre>

<p><strong>6. Connecting to Azure Data Lake</strong></p>
<p>To connect to Azure Data Lake (ADLS), change the endpoint to <code>dfs.core.windows.net</code> (ADLS Gen2).</p>

<pre><code>
# Set Spark config for Data Lake Storage (ADLS Gen2)
spark.conf.set(
  "fs.azure.account.key.&lt;storage-account-name&gt;.dfs.core.windows.net",
  dbutils.secrets.get(scope="storage-scope", key="storage-account-access-key")
)

# Define the path to the Data Lake container
path = "abfss://&lt;container-name&gt;@&lt;storage-account-name&gt;.dfs.core.windows.net/"

# List files in the Data Lake container
files = dbutils.fs.ls(path)
for file in files:
    print(file)
</code></pre>

<p><strong>7. Reuse the Connection in Multiple Notebooks</strong></p>
<p>Instead of setting up the connection in each notebook, you can create a shared notebook that establishes the Spark session connection, and then include it in other notebooks using the <code>%run</code> command.</p>
<ol>
    <li>Create a notebook <code>start_session</code> that includes all connection logic.</li>
    <li>In your other notebooks, use:
        <pre><code>%run /path/to/start_session</code></pre>
    </li>
</ol>

<p><strong>8. Setting Spark Configuration at the Cluster Level</strong></p>
<p>To make the storage connection available across all notebooks in a cluster, you can configure Spark settings directly in the cluster settings:</p>
<ol>
    <li>Go to the <strong>Clusters</strong> page in Databricks.</li>
    <li>Edit the cluster you created.</li>
    <li>Under <strong>Spark Config</strong>, add:
        <pre><code>fs.azure.account.key.&lt;storage-account-name&gt;.blob.core.windows.net &lt;your-secret&gt;</code></pre>
    </li>
</ol>
<p>This way, all notebooks in the cluster will automatically have access to the Blob storage.</p>
