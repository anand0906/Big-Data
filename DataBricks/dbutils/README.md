<h1>dbutils</h1>
<p>dbutils is a set of utility functions provided by Databricks to interact with the Databricks File System (DBFS), manage jobs, and work with data and notebooks in the Databricks environment. It allows users to interact with the workspace, notebooks, data, secrets, and more programmatically within a Databricks notebook or job.</p>
<h2>help()</h2>
<p>The dbutils.help() function provides an overview and brief documentation of all available dbutils methods and submodules in the Databricks environment. This is particularly useful when you want to quickly access information about the different utilities provided by dbutils without referring to external documentation.</p>
<ul>
	<li>When you are exploring Databricks utilities for the first time.</li>
	<li>If you want a quick reference without searching for documentation.</li>
	<li>To get detailed descriptions of available methods within a particular submodule.</li>
</ul>

```python
dbutils.help()
```

<p>When you run dbutils.help(), you will see an output similar to this:</p>

<pre>
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
</pre>

<p>This output shows all the major submodules available under dbutils and provides a brief description of each.</p>

<h2>dbutils.fs</h2>

```python
dbutils.fs.help()
```

<pre>
dbutils.fs provides utilities for working with FileSystems. Most methods in this package can take either a DBFS path (e.g., "/foo" or "dbfs:/foo"), or another FileSystem URI. For more info about a method, use dbutils.fs.help("methodName"). In notebooks, you can also use the %fs shorthand to access DBFS. The %fs shorthand maps straightforwardly onto dbutils calls. For example, "%fs head --maxBytes=10000 /file/path" translates into "dbutils.fs.head("/file/path", maxBytes = 10000)".

mount
	mount(source: String, mountPoint: String, encryptionType: String = "", owner: String = null, extraConfigs: Map = Map.empty[String, String]): boolean -> Mounts the given source directory into DBFS at the given mount point
	mounts: Seq -> Displays information about what is mounted within DBFS
	refreshMounts: boolean -> Forces all machines in this cluster to refresh their mount cache, ensuring they receive the most recent information
	unmount(mountPoint: String): boolean -> Deletes a DBFS mount point
	updateMount(source: String, mountPoint: String, encryptionType: String = "", owner: String = null, extraConfigs: Map = Map.empty[String, String]): boolean -> Similar to mount(), but updates an existing mount point (if present) instead of creating a new one

fsutils
	cp(from: String, to: String, recurse: boolean = false): boolean -> Copies a file or directory, possibly across FileSystems
	head(file: String, maxBytes: int = 65536): String -> Returns up to the first 'maxBytes' bytes of the given file as a String encoded in UTF-8
	ls(dir: String): Seq -> Lists the contents of a directory
	mkdirs(dir: String): boolean -> Creates the given directory if it does not exist, also creating any necessary parent directories
	mv(from: String, to: String, recurse: boolean = false): boolean -> Moves a file or directory, possibly across FileSystems
	put(file: String, contents: String, overwrite: boolean = false): boolean -> Writes the given String out to a file, encoded in UTF-8
	rm(dir: String, recurse: boolean = false): boolean -> Removes a file or directory
</pre>

<h3>fsutils</h3>

<p>The <strong>dbutils.fs</strong> module provides file system utilities for interacting with the Databricks File System (DBFS). It's similar to using file handling commands in a Unix/Linux shell but operates within the Databricks environment, allowing you to perform tasks like listing, copying, moving, and deleting files and directories.</p>

<p><strong>Key Functions of dbutils.fs</strong></p>
<p>Here's a detailed explanation of the most commonly used <strong>dbutils.fs</strong> methods, including real-time examples with Python code and expected outputs.</p>

<ol>
    <li><strong>dbutils.fs.ls - Listing Files and Directories</strong>
        <p>This function lists all files and subdirectories in a specified directory on DBFS.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>dbutils.fs.ls(dir: str)</code></p>
        <p><strong>Parameter:</strong></p>
        <ul>
            <li><code>dir</code>: The path to the directory.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <p>Let's list the files in the <code>/mnt/data</code> directory.</p>
        <p>Python Code:</p>

```python
files = dbutils.fs.ls("/mnt/data")
for file in files:
    print(f"Name: {file.name}, Path: {file.path}, Size: {file.size} bytes")

```    
        <p><strong>Expected Output:</strong></p>
        <ul>
            <li>Name: sales_data.csv, Path: dbfs:/mnt/data/sales_data.csv, Size: 2048 bytes</li>
            <li>Name: customer_data.csv, Path: dbfs:/mnt/data/customer_data.csv, Size: 1024 bytes</li>
            <li>Name: reports/, Path: dbfs:/mnt/data/reports/, Size: 0 bytes</li>
        </ul>
    </li>   
    <li><strong>dbutils.fs.cp - Copying Files or Directories</strong>
        <p>This function copies a file or directory from one location to another in DBFS.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>dbutils.fs.cp(source: str, destination: str, recurse: bool = False)</code></p>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><code>source</code>: Path of the source file/directory.</li>
            <li><code>destination</code>: Path of the destination file/directory.</li>
            <li><code>recurse</code>: If True, copies directories recursively.</li>
        </ul>    
        <p><strong>Real-time Example:</strong></p>
        <p>Suppose you want to copy <code>sales_data.csv</code> from <code>/mnt/data</code> to <code>/mnt/archive</code>.</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.cp("/mnt/data/sales_data.csv", "/mnt/archive/sales_data.csv")</code></p>
    </li>
    <li><strong>dbutils.fs.mv - Moving/Renaming Files or Directories</strong>
        <p>This function moves or renames a file or directory in DBFS.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>dbutils.fs.mv(source: str, destination: str, recurse: bool = False)</code></p>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><code>source</code>: Source path.</li>
            <li><code>destination</code>: Destination path.</li>
            <li><code>recurse</code>: If True, moves directories recursively.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <p>Let's move <code>customer_data.csv</code> from <code>/mnt/data</code> to <code>/mnt/processed</code>.</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.mv("/mnt/data/customer_data.csv", "/mnt/processed/customer_data.csv")</code></p>
    </li>  
    <li><strong>dbutils.fs.rm - Removing Files or Directories</strong>
        <p>This function deletes a specified file or directory.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>dbutils.fs.rm(dir_or_file: str, recurse: bool = False)</code></p>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><code>dir_or_file</code>: The path to the file or directory to be deleted.</li>
            <li><code>recurse</code>: If True, deletes directories recursively.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <p>Suppose you want to remove the <code>sales_data.csv</code> file from <code>/mnt/archive</code>.</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.rm("/mnt/archive/sales_data.csv")</code></p>
    </li>
</ol>

<p><strong>Summary Table of dbutils.fs Methods</strong></p>
<table border="1">
    <tr>
        <th>Method</th>
        <th>Description</th>
        <th>Real-time Use Case</th>
    </tr>
    <tr>
        <td><code>dbutils.fs.ls()</code></td>
        <td>Lists files in a directory</td>
        <td>Viewing data files available in DBFS</td>
    </tr>
    <tr>
        <td><code>dbutils.fs.cp()</code></td>
        <td>Copies files/directories</td>
        <td>Copying raw data for backup</td>
    </tr>
    <tr>
        <td><code>dbutils.fs.mv()</code></td>
        <td>Moves/renames files/directories</td>
        <td>Moving data to a processed folder</td>
    </tr>
    <tr>
        <td><code>dbutils.fs.rm()</code></td>
        <td>Removes files/directories</td>
        <td>Cleaning up old files</td>
    </tr>
</table>

<p><strong>Conclusion</strong></p>
<p>The <strong>dbutils.fs</strong> module provides essential tools for managing data files within the Databricks File System (DBFS). By using <strong>dbutils.fs</strong> methods effectively, you can create efficient data handling workflows, enabling seamless integration of data processing and analysis tasks within the Databricks environment.</p>



<h3>mount</h3>
<p><strong>Mount Operations in DBFS</strong></p>
<p>The following methods allow you to manage mount points in the Databricks File System (DBFS). A mount point connects external storage (e.g., Azure Data Lake, Amazon S3, etc.) to DBFS, making it easier to access data stored outside of Databricks.</p>

<ol>
    <li><strong>mount</strong>
        <p>This method mounts the given source directory into DBFS at the specified mount point.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>mount(source: String, mountPoint: String, encryptionType: String = "", owner: String = null, extraConfigs: Map = Map.empty[String, String]): boolean</code></p>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><strong>source</strong>: The URI of the storage you want to mount (e.g., an S3 bucket or Azure Data Lake directory).</li>
            <li><strong>mountPoint</strong>: The path within DBFS where you want to mount the storage.</li>
            <li><strong>encryptionType</strong> (optional): The type of encryption (if any) used.</li>
            <li><strong>owner</strong> (optional): The owner of the mount point.</li>
            <li><strong>extraConfigs</strong> (optional): Additional configurations required for mounting, typically used for authentication details like keys.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <p>Mount an Azure Data Lake directory:</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.mount(source="wasbs://mycontainer@myaccount.blob.core.windows.net",</code><br>
           <code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mountPoint="/mnt/mydata",</code><br>
           <code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;extraConfigs={"fs.azure.account.key.myaccount.blob.core.windows.net": "your-access-key"})</code></p>
    </li>
    <li><strong>mounts</strong>
        <p>This method displays information about what is currently mounted within DBFS.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>mounts: Seq</code></p>
        <p><strong>Real-time Example:</strong></p>
        <p>Display all mount points:</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.mounts()</code></p>
    </li>
    <li><strong>refreshMounts</strong>
        <p>This method forces all machines in the cluster to refresh their mount cache, ensuring they receive the most recent information about the mounts.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>refreshMounts: boolean</code></p>
        <p><strong>Real-time Example:</strong></p>
        <p>Refresh all mount points:</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.refreshMounts()</code></p>
    </li>
    <li><strong>unmount</strong>
        <p>This method deletes a DBFS mount point.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>unmount(mountPoint: String): boolean</code></p>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><strong>mountPoint</strong>: The path of the mount point you want to unmount.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <p>Unmount a previously mounted directory:</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.unmount("/mnt/mydata")</code></p>
    </li>
    <li><strong>updateMount</strong>
        <p>This method updates an existing mount point instead of creating a new one.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>updateMount(source: String, mountPoint: String, encryptionType: String = "", owner: String = null, extraConfigs: Map = Map.empty[String, String]): boolean</code></p>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><strong>source</strong>: The URI of the storage you want to update the mount to.</li>
            <li><strong>mountPoint</strong>: The existing mount point within DBFS that you want to update.</li>
            <li><strong>encryptionType</strong> (optional): The type of encryption (if any) used.</li>
            <li><strong>owner</strong> (optional): The owner of the mount point.</li>
            <li><strong>extraConfigs</strong> (optional): Additional configurations required for updating the mount, typically used for authentication details.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <p>Update an existing mount point with new credentials:</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.updateMount(source="wasbs://mycontainer@myaccount.blob.core.windows.net",</code><br>
           <code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mountPoint="/mnt/mydata",</code><br>
           <code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;extraConfigs={"fs.azure.account.key.myaccount.blob.core.windows.net": "new-access-key"})</code></p>
    </li>
</ol>
