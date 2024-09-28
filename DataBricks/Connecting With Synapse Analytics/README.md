<h1>Bulk Insertion vs PolyBase in Azure Synapse:</h1>

<p><strong>1. Bulk Insertion:</strong></p>
<ul>
  <li><strong>What is it?</strong> Bulk insertion is a method to quickly move large amounts of data from a local or cloud source into a database. You essentially copy data from one place (like a file or storage) directly into the tables of a Synapse database.</li>
  <li><strong>How it works:</strong> Think of it like manually loading a truck with items and delivering them to a warehouse. You take data files (like CSV or Excel), and use commands to insert all that data into your Synapse tables at once.</li>
  <li><strong>Real-time Example:</strong> Imagine you have an Excel file with customer data. Using bulk insertion, you load the entire file's contents directly into your Synapse database table. You just need the file location and some commands to get the job done.</li>
  <li><strong>When to use it:</strong>
    <ul>
      <li>You have data stored locally or in Azure Blob Storage.</li>
      <li>The data size isn't too massive or repetitive.</li>
      <li>You need fast, simple data uploads without complex processing.</li>
    </ul>
  </li>
</ul>

<p><strong>2. PolyBase:</strong></p>
<ul>
  <li><strong>What is it?</strong> PolyBase is a technology that allows you to query data from external sources (like Azure Blob Storage or data lakes) without actually moving the data into the Synapse database. It virtually connects to the data.</li>
  <li><strong>How it works:</strong> Think of PolyBase like connecting your Synapse database to a massive external data storage, like plugging your computer into a shared drive. You don’t need to move the data into your system, you just query it directly from its original location.</li>
  <li><strong>Real-time Example:</strong> Suppose you have terabytes of customer data stored in Azure Data Lake. Instead of copying this data into Synapse, you use PolyBase to directly query the data from the Data Lake. This saves time and space since you’re not duplicating data, and you can work with huge datasets.</li>
  <li><strong>When to use it:</strong>
    <ul>
      <li>You have massive amounts of data stored externally (like in data lakes).</li>
      <li>You don’t want to move or duplicate the data.</li>
      <li>You want faster querying for large data sources without overloading your database.</li>
    </ul>
  </li>
</ul>

<p><strong>Key Differences:</strong></p>
<ol>
  <li><strong>Data Movement:</strong>
    <ul>
      <li><strong>Bulk Insertion:</strong> Moves data into the Synapse database.</li>
      <li><strong>PolyBase:</strong> Connects to external data without moving it.</li>
    </ul>
  </li>
  <li><strong>Performance:</strong>
    <ul>
      <li><strong>Bulk Insertion:</strong> Suitable for smaller, manageable data loads.</li>
      <li><strong>PolyBase:</strong> Handles large datasets more efficiently by querying externally.</li>
    </ul>
  </li>
  <li><strong>When to Choose:</strong>
    <ul>
      <li>Use <strong>Bulk Insertion</strong> when you need to upload smaller datasets for faster, local querying.</li>
      <li>Use <strong>PolyBase</strong> when you want to query large external datasets without moving them.</li>
    </ul>
  </li>
</ol>



<h1>Step-by-Step Guide to Connect Azure Synapse PolyBase with Databricks Using Azure Data Lake</h1>

<p>This guide will walk you through the process of connecting Azure Synapse to Databricks using Azure Data Lake as the intermediary storage. By following these steps, you'll be able to perform data operations (like reading and writing) between Databricks and Synapse with PolyBase.</p>

<ol>
  <li>
    <strong>Create a Storage Account</strong>
    <ul>
      <li>Go to the Azure Portal and create a new <strong>Azure Storage Account</strong>.</li>
      <li>Store files and intermediate data that will be accessed by Databricks and Synapse through this storage account.</li>
    </ul>
  </li>

  <li>
    <strong>Create Azure Synapse and a SQL Pool (Data Warehouse)</strong>
    <ul>
      <li>In the Azure Portal, create an <strong>Azure Synapse Analytics</strong> workspace.</li>
      <li>Inside the Synapse workspace, create a <strong>SQL Pool</strong> (formerly called a Data Warehouse).</li>
      <li>This pool is where your Synapse will store and retrieve data.</li>
    </ul>
  </li>

  <li>
    <strong>Use SQL Data Warehouse with Synapse Studio or Azure Data Studio</strong>
    <ul>
      <li>You can use <strong>Synapse Studio</strong> (the built-in tool in Synapse) or <strong>Azure Data Studio</strong> to connect to and manage the SQL Pool.</li>
      <li>Use either of these tools to run queries, create tables, and manage your data.</li>
    </ul>
  </li>

  <li>
    <strong>Create Demo Tables and Set Up Security in Synapse</strong>
    <ul>
      <li>Inside your SQL Pool, create a <strong>master key</strong> using the following SQL:</li>
      <code>CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'yourPassword';</code>
      <li>Create some demo tables using SQL queries. For example, you could create a <strong>cust</strong> table:</li>
      <code>
        CREATE TABLE dbo.cust (id INT PRIMARY KEY, name NVARCHAR(100), email NVARCHAR(100));
      </code>
    </ul>
  </li>

  <li>
    <strong>Create an Azure Key Vault</strong>
    <ul>
      <li>In the Azure Portal, create a new <strong>Azure Key Vault</strong>.</li>
      <li>This Key Vault will securely store your sensitive data like the storage account access key and SQL Pool connection string.</li>
      <li>Assign the <strong>Key Vault Administrator role</strong> to both <strong>Databricks</strong> and your user account to access secrets.</li>
    </ul>
  </li>

  <li>
    <strong>Store Secrets in Key Vault</strong>
    <ul>
      <li>Store the <strong>Storage Account Key</strong> in the Key Vault as a secret (e.g., <strong>sqldwdfs</strong>).</li>
      <li>Store the <strong>SQL Pool JDBC Connection String</strong> as another secret (e.g., <strong>sqldwjdbc</strong>).</li>
    </ul>
  </li>

  <li>
    <strong>Create Databricks Workspace and Cluster</strong>
    <ul>
      <li>Create a <strong>Databricks workspace</strong> in the Azure Portal.</li>
      <li>Inside Databricks, create a <strong>cluster</strong> that will be used to run your notebooks and perform the data operations.</li>
    </ul>
  </li>

  <li>
    <strong>Create Secret Scope in Databricks</strong>
    <ul>
      <li>In Databricks, create a <strong>secret scope</strong> to securely access the Azure Key Vault.</li>
      <li>Use the URL <strong>#secrets/createScope</strong> to create a new secret scope.</li>
    </ul>
  </li>

  <li>
    <strong>Connect Databricks with Azure Synapse via Data Lake</strong>
    <p>Now, let's connect Databricks to Azure Synapse using PolyBase, with Azure Data Lake as an intermediary.</p>
    <code>
    # Fetch the secret values from the Key Vault<br>
    sqldwblob = dbutils.secrets.get(scope="anand-scope", key="sqldwdfs")<br>
    sqldwjdbc = dbutils.secrets.get(scope="anand-scope", key="sqldwjdbc")<br><br>
    # Set up the Azure Data Lake connection using the storage account key<br>
    spark.conf.set("fs.azure.account.key.ananddatalake3.dfs.core.windows.net", sqldwblob)<br><br>
    # Read data from Synapse via Data Lake using PolyBase<br>
    df = spark.read.format("com.databricks.spark.sqldw")<br>
    .option("url", sqldwjdbc)<br>
    .option("tempDir", "abfss://output@ananddatalake3.dfs.core.windows.net/myfolder")<br>
    .option("forwardSparkAzureStorageCredentials", "true")<br>
    .option("dbTable", "dbo.cust")<br>
    .load()<br><br>
    # Display the data<br>
    display(df)
    </code>
  </li>

  <li>
    <strong>Perform CRUD Operations in Databricks Using Synapse</strong>
    <p>After connecting, you can perform <strong>CRUD (Create, Read, Update, Delete)</strong> operations on your data. Below are examples of each operation:</p>
    <ul>
      <li><strong>Insert Data (Create)</strong></li>
      <code>
        new_data = [(3, 'John Doe', 'johndoe@example.com')]<br>
        columns = ['id', 'name', 'email']<br><br>
        new_df = spark.createDataFrame(new_data, columns)<br><br>
        new_df.write.format("com.databricks.spark.sqldw")<br>
        .option("url", sqldwjdbc)<br>
        .option("dbTable", "dbo.cust")<br>
        .mode("append")<br>
        .option("forwardSparkAzureStorageCredentials", "true")<br>
        .option("tempDir", "abfss://output@ananddatalake3.dfs.core.windows.net/myfolder")<br>
        .save()
      </code>
      <li><strong>Read Data</strong></li>
      <code>
        df = spark.read.format("com.databricks.spark.sqldw")<br>
        .option("url", sqldwjdbc)<br>
        .option("dbTable", "dbo.cust")<br>
        .option("forwardSparkAzureStorageCredentials", "true")<br>
        .option("tempDir", "abfss://output@ananddatalake3.dfs.core.windows.net/myfolder")<br>
        .load()<br><br>
        display(df)
      </code>
      <li><strong>Update Data</strong></li>
      <code>
        df.createOrReplaceTempView("cust_temp")<br>
        spark.sql("UPDATE cust_temp SET name = 'Jane Doe' WHERE id = 3")
      </code>
      <li><strong>Delete Data</strong></li>
      <code>
        spark.sql("DELETE FROM dbo.cust WHERE id = 3")
      </code>
    </ul>
  </li>
</ol>

<p>This method allows for efficient interaction between Databricks and Synapse, especially when handling large datasets via Azure Data Lake.</p>
