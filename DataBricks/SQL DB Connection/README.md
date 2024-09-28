<h1>Connect With Azure SQL DB</h1>
<p><strong>1. Create SQL Server in Azure Portal</strong></p>
<ol>
    <li>Navigate to the <a href="https://portal.azure.com">Azure Portal</a>.</li>
    <li>Search for SQL Server and select <strong>SQL Servers</strong>.</li>
    <li>Click <strong>Create</strong>, and provide:
        <ul>
            <li><strong>Subscription</strong>: Choose your subscription.</li>
            <li><strong>Resource Group</strong>: Select or create a resource group.</li>
            <li><strong>Server Name</strong>: Provide a globally unique name (e.g., anand-sql-server2).</li>
            <li><strong>Region</strong>: Select a nearby region for performance.</li>
            <li><strong>Admin Login and Password</strong>: Set credentials for the server.</li>
        </ul>
    </li>
    <li>Click <strong>Review + Create</strong> to finalize the SQL server setup.</li>
</ol>

<p><strong>2. Create SQL Database and Load Sample Data</strong></p>
<ol>
    <li>In the Azure Portal, search for <strong>SQL Databases</strong> and click <strong>Create</strong>.</li>
    <li>Select the SQL Server you just created and provide a name for the database (e.g., anand-sql-db).</li>
    <li>Select the <strong>Sample Database</strong> option (AdventureWorks LT) to load sample data.</li>
    <li>Review the settings and click <strong>Create</strong>.</li>
</ol>

<p><strong>3. Connect to the SQL Database with Azure Data Studio</strong></p>
<ol>
    <li>Download and install <a href="https://aka.ms/azuredatastudio">Azure Data Studio</a>.</li>
    <li>Connect to the database using the SQL Server credentials (admin username and password).</li>
    <li>Browse the <strong>SalesLT.Customer</strong> table to view the sample data.</li>
</ol>

<p><strong>4. Create an Azure Key Vault and Store Secrets</strong></p>
<ol>
    <li>In the Azure Portal, search for <strong>Key Vault</strong> and click <strong>Create</strong>.</li>
    <li>Select the same resource group as the SQL Server and provide a unique name for the Key Vault.</li>
    <li>Go to <strong>Secrets</strong> in the Key Vault and create two secrets:
        <ul>
            <li><strong>dbusername</strong>: Store the database username.</li>
            <li><strong>dbuserpassword</strong>: Store the database password.</li>
        </ul>
    </li>
</ol>

<p><strong>5. Set Up Databricks and Create a Secret Scope</strong></p>
<ol>
    <li>In the Azure Portal, search for <strong>Databricks</strong> and create a new Databricks workspace.</li>
    <li>Create a cluster inside the Databricks workspace.</li>
    <li>In the Databricks UI, go to the <strong>Secrets</strong> section under <strong>User Settings</strong> and create a new secret scope that accesses the Key Vault where your SQL credentials are stored.</li>
</ol>

<p><strong>6. Connecting Azure SQL Database to Databricks</strong></p>
<ol>
    <li>In the Azure Portal, go to your SQL Database and find the JDBC URL under <strong>Connection Strings</strong>. It will look like:
        <code>jdbc:sqlserver://&lt;sql-server-name&gt;.database.windows.net:1433;database=&lt;sql-db-name&gt;</code>.
    </li>
    <li>Use the following Python code to connect Databricks to Azure SQL Database:</li>
</ol>

<pre><code>
# Fetch username and password from Key Vault using dbutils
username = dbutils.secrets.get(scope="anand-scope", key="dbusername")
password = dbutils.secrets.get(scope="anand-scope", key="dbuserpassword")

# Define JDBC URL
jdbc_url = "jdbc:sqlserver://anand-sql-server2.database.windows.net:1433;database=anand-sql-db;user=anand@anand-sql-server2"

# Read data from the SQL database
customer_table = (spark.read
  .format("jdbc")
  .option("url", jdbc_url)
  .option("dbtable", "SalesLT.Customer")
  .option("user", username)
  .option("password", password)
  .load()
)

# Show the customer table data
customer_table.show()
</code></pre>

<p><strong>7. CRUD Operations with Python in Databricks</strong></p>

<p><strong>a. Create (Insert) Data</strong></p>
<pre><code>
# Define new data to insert into the Customer table
new_data = [(1001, "John", "Doe", "johndoe@example.com")]

# Convert to DataFrame
new_df = spark.createDataFrame(new_data, ["CustomerID", "FirstName", "LastName", "EmailAddress"])

# Write data to SQL (Append mode)
new_df.write.jdbc(url=jdbc_url, table="SalesLT.Customer", mode="append", properties={"user": username, "password": password})
</code></pre>

<p><strong>b. Read Data</strong></p>
<p>This was demonstrated earlier in Step 6.</p>

<p><strong>c. Update Data</strong></p>
<pre><code>
# Create a SQL query to update a customer record
update_query = "UPDATE SalesLT.Customer SET LastName = 'Smith' WHERE CustomerID = 1001"

# Execute the SQL query using spark.read.jdbc
spark.sql(update_query)
</code></pre>

<p><strong>d. Delete Data</strong></p>
<pre><code>
# Create a SQL query to delete a customer record
delete_query = "DELETE FROM SalesLT.Customer WHERE CustomerID = 1001"

# Execute the SQL query
spark.sql(delete_query)
</code></pre>

<p><strong>8. Reusing Code Across Notebooks with %run</strong></p>
<ol>
    <li>Create a common notebook (e.g., <code>db_connection</code>) that contains the connection logic:</li>
</ol>

<pre><code>
# db_connection notebook
username = dbutils.secrets.get(scope="anand-scope", key="dbusername")
password = dbutils.secrets.get(scope="anand-scope", key="dbuserpassword")
jdbc_url = "jdbc:sqlserver://anand-sql-server2.database.windows.net:1433;database=anand-sql-db;user=anand@anand-sql-server2"
</code></pre>

<ol>
    <li>In other notebooks, use <code>%run</code> to include the connection setup:</li>
</ol>

<pre><code>
# Include the connection notebook
%run /path/to/db_connection

# Now use the established connection to perform operations
customer_table = (spark.read
  .format("jdbc")
  .option("url", jdbc_url)
  .option("dbtable", "SalesLT.Customer")
  .option("user", username)
  .option("password", password)
  .load()
)

customer_table.show()
</code></pre>

<p>This approach makes your code more modular and reusable, allowing you to focus on operations rather than connection setup.</p>


<h1>Connecting Azure SQL DB With Azure Studio</h1>
<p><strong>Steps to Connect Azure SQL DB with Azure Data Studio</strong></p>

<p><strong>1. Download and Install Azure Data Studio</strong></p>
<ol>
    <li>Visit the official Microsoft download page for <a href="https://learn.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio" target="_blank">Azure Data Studio</a>.</li>
    <li>Choose the appropriate version for your operating system (Windows, macOS, or Linux).</li>
    <li>Follow the installation instructions to complete the setup.</li>
</ol>

<p><strong>2. Get Connection Details from Azure SQL Database</strong></p>
<p>Before connecting to your Azure SQL Database in Azure Data Studio, you need the connection details. These details can be found in the Azure portal.</p>
<ol>
    <li>Navigate to the <a href="https://portal.azure.com" target="_blank">Azure Portal</a>.</li>
    <li>Search for <strong>SQL Databases</strong> in the search bar and select your SQL Database.</li>
    <li>Go to the <strong>Overview</strong> section of your SQL Database resource. You'll need the following information:
        <ul>
            <li><strong>Server name</strong>: The fully qualified name of your SQL Server (e.g., <code>anand-sql-server2.database.windows.net</code>).</li>
            <li><strong>Database name</strong>: The name of your database (e.g., <code>anand-sql-db</code>).</li>
            <li><strong>Login credentials</strong>: The username and password you provided when you created the SQL Server.</li>
            <li><strong>Connection string</strong>: Available under the <strong>Connection strings</strong> tab. This will be used later.</li>
        </ul>
    </li>
</ol>

<p><strong>3. Configure Firewall Rules for SQL Server</strong></p>
<p>Azure SQL Database uses firewall rules to prevent unauthorized access.</p>
<ol>
    <li>In the SQL Database's <strong>Overview</strong> section, find the <strong>Set server firewall</strong> option.</li>
    <li>Add a rule to allow your current IP address (or range) to connect to the database.</li>
    <li>Save the firewall rule so that your machine is allowed to access the SQL Server.</li>
</ol>

<p><strong>4. Open Azure Data Studio and Set Up a New Connection</strong></p>
<p>Now, it's time to connect to the Azure SQL Database using Azure Data Studio.</p>
<ol>
    <li>Open <strong>Azure Data Studio</strong>.</li>
    <li>Click on <strong>New Connection</strong> in the top-left corner of the window.</li>
    <li>In the connection window, fill in the following details:
        <ul>
            <li><strong>Server</strong>: Enter the SQL Server name you got from the Azure portal (e.g., <code>anand-sql-server2.database.windows.net</code>).</li>
            <li><strong>Database</strong>: Choose the database name, or leave it blank to connect to the default database.</li>
            <li><strong>Authentication Type</strong>: Choose <strong>SQL Login</strong>.</li>
            <li><strong>Username</strong>: Enter the SQL Server admin username (created while setting up the SQL Server).</li>
            <li><strong>Password</strong>: Enter the corresponding password.</li>
        </ul>
    </li>
    <li>Optionally, check <strong>Remember Password</strong> to save the credentials for future connections.</li>
    <li>Click <strong>Connect</strong>.</li>
</ol>

<p><strong>5. Explore the Database in Azure Data Studio</strong></p>
<p>Once the connection is successful, you can start interacting with your Azure SQL Database.</p>
<ol>
    <li>After connecting, you will see the <strong>Server</strong> and <strong>Database</strong> listed in the <strong>Connections</strong> pane on the left.</li>
    <li>Expand the database node to see the available <strong>Tables</strong>, <strong>Views</strong>, and <strong>Stored Procedures</strong>.</li>
    <li>You can right-click any table (e.g., <code>SalesLT.Customer</code> from the sample AdventureWorks database) to perform operations like viewing data, editing, or running queries.</li>
</ol>

<p><strong>6. Run SQL Queries</strong></p>
<p>You can now run SQL queries directly from Azure Data Studio to interact with your Azure SQL Database.</p>
<ol>
    <li>Click <strong>New Query</strong> from the toolbar.</li>
    <li>Write a SQL query to interact with your database. For example, to view all data from a table:</li>
    <pre><code>SELECT * FROM SalesLT.Customer;</code></pre>
    <li>Click <strong>Run</strong> to execute the query and see the results in the bottom pane.</li>
</ol>
