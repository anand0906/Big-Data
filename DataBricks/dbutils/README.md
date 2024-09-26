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

<p><strong>Overview of dbutils.fs in Databricks</strong></p>
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
        <p><code>files = dbutils.fs.ls("/mnt/data")</code><br>
        <code>for file in files:</code><br>
        <code>&nbsp;&nbsp;&nbsp;print(f"Name: {file.name}, Path: {file.path}, Size: {file.size} bytes")</code></p>       
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
        <p>After this command, running <code>dbutils.fs.ls("/mnt/archive")</code> would show:</p>
        <ul>
            <li>Name: sales_data.csv, Path: dbfs:/mnt/archive/sales_data.csv, Size: 2048 bytes</li>
        </ul>
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
        <p>Now, running <code>dbutils.fs.ls("/mnt/data")</code> would no longer list <code>customer_data.csv</code>, while <code>dbutils.fs.ls("/mnt/processed")</code> would display:</p>
        <ul>
            <li>Name: customer_data.csv, Path: dbfs:/mnt/processed/customer_data.csv, Size: 1024 bytes</li>
        </ul>
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
        <p>After executing this, <code>dbutils.fs.ls("/mnt/archive")</code> would show an empty directory.</p>
    </li>  
    <li><strong>dbutils.fs.put - Writing Data to a File</strong>
        <p>This function writes the specified content into a file.</p>
        <p><strong>Syntax:</strong></p>
        <p><code>dbutils.fs.put(file: str, contents: str, overwrite: bool = False)</code></p>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><code>file</code>: Path of the file to write to.</li>
            <li><code>contents</code>: The content to be written.</li>
            <li><code>overwrite</code>: If True, overwrites existing content.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <p>Let's create a new file called <code>example.txt</code> in the <code>/mnt/data</code> directory and add some content.</p>
        <p>Python Code:</p>
        <p><code>dbutils.fs.put("/mnt/data/example.txt", "This is a sample file created using dbutils.", overwrite=True)</code></p>   
        <p>Running <code>dbutils.fs.head("/mnt/data/example.txt")</code> would output:</p>
        <ul>
            <li>This is a sample file created using dbutils.</li>
        </ul>
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
    <tr>
        <td><code>dbutils.fs.put()</code></td>
        <td>Writes content to a file</td>
        <td>Creating configuration files</td>
    </tr>
    <tr>
        <td><code>dbutils.fs.head()</code></td>
        <td>Reads the start of a file</td>
        <td>Previewing file contents</td>
    </tr>
</table>

<p><strong>Conclusion</strong></p>
<p>The <strong>dbutils.fs</strong> module provides essential tools for managing data files within the Databricks File System (DBFS). By using <strong>dbutils.fs</strong> methods effectively, you can create efficient data handling workflows, enabling seamless integration of data processing and analysis tasks within the Databricks environment.</p>




<h3>mount</h3>
<p><strong>Overview of Mount Operations in DBFS</strong></p>
<p>Mount operations in Databricks allow you to access external storage systems (e.g., Azure Blob Storage, Azure Data Lake, Amazon S3) as if they were part of the Databricks File System (DBFS). The <code>dbutils.fs</code> module provides several methods to manage these mount points efficiently.</p>

<p><strong>Key Mount Methods of dbutils.fs</strong></p>

<ol>
    <li><strong>dbutils.fs.mount() - Mounting a Directory</strong>
        <p>The <code>mount</code> method allows you to mount a given source directory from an external storage system into DBFS at a specified mount point.</p>
        <p><strong>Syntax:</strong></p>
        <code>dbutils.fs.mount(source: str, mountPoint: str, encryptionType: str = "", owner: str = None, extraConfigs: dict = {})</code>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><strong>source</strong>: The URI of the source directory you want to mount (e.g., an S3 bucket or Azure Data Lake directory).</li>
            <li><strong>mountPoint</strong>: The path within DBFS where you want to mount the storage.</li>
            <li><strong>encryptionType</strong> (optional): The type of encryption used. Default is an empty string.</li>
            <li><strong>owner</strong> (optional): The owner of the mount point. Default is <code>None</code>.</li>
            <li><strong>extraConfigs</strong> (optional): Additional configurations required for mounting, typically used for authentication.</li>
        </ul> 
        <p><strong>Real-time Example:</strong></p>
        <code>
            dbutils.fs.mount( <br>
            &nbsp;&nbsp;source="wasbs://mycontainer@myaccount.blob.core.windows.net",<br>
            &nbsp;&nbsp;mountPoint="/mnt/mydata",<br>
            &nbsp;&nbsp;extraConfigs={"fs.azure.account.key.myaccount.blob.core.windows.net": "your-access-key"}<br>
            )
        </code>
    </li>
    <li><strong>dbutils.fs.mounts() - Displaying Mount Information</strong>
        <p>The <code>mounts</code> method displays all the current mount points available within DBFS.</p>
        <p><strong>Syntax:</strong></p>
        <code>dbutils.fs.mounts()</code>
        <p><strong>Real-time Example:</strong></p>
        <code>
            mount_points = dbutils.fs.mounts()<br>
            for mount in mount_points:<br>
            &nbsp;&nbsp;print(f"Mount Point: {mount.mountPoint}, Source: {mount.source}")
        </code>
        <p><strong>Expected Output:</strong></p>
        <code>
            Mount Point: /mnt/mydata, Source: wasbs://mycontainer@myaccount.blob.core.windows.net<br>
            Mount Point: /mnt/logs, Source: s3a://my-log-bucket/logs
        </code>
    </li>
    <li><strong>dbutils.fs.refreshMounts() - Refreshing Mount Information</strong>
        <p>The <code>refreshMounts</code> method forces all machines in the cluster to refresh their mount cache, ensuring they have the most recent mount information.</p>
        <p><strong>Syntax:</strong></p>
        <code>dbutils.fs.refreshMounts()</code>
        <p><strong>Real-time Example:</strong></p>
        <code>
            dbutils.fs.refreshMounts()<br>
            print("Mount points have been refreshed.")
        </code>
    </li>
    <li><strong>dbutils.fs.unmount() - Unmounting a Directory</strong>
        <p>The <code>unmount</code> method deletes a specified DBFS mount point.</p>
        <p><strong>Syntax:</strong></p>
        <code>dbutils.fs.unmount(mountPoint: str)</code>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><strong>mountPoint</strong>: The path of the mount point you want to unmount.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <code>
            dbutils.fs.unmount("/mnt/mydata")<br>
            print("The directory has been unmounted successfully.")
        </code>
    </li>
    <li><strong>dbutils.fs.updateMount() - Updating an Existing Mount Point</strong>
        <p>The <code>updateMount</code> method updates an existing mount point's configuration without creating a new one.</p>
        <p><strong>Syntax:</strong></p>
        <code>dbutils.fs.updateMount(source: str, mountPoint: str, encryptionType: str = "", owner: str = None, extraConfigs: dict = {})</code>
        <p><strong>Parameters:</strong></p>
        <ul>
            <li><strong>source</strong>: The URI of the storage you want to update.</li>
            <li><strong>mountPoint</strong>: The existing mount point within DBFS that you want to update.</li>
            <li><strong>encryptionType</strong> (optional): The type of encryption used. Default is an empty string.</li>
            <li><strong>owner</strong> (optional): The owner of the mount point. Default is <code>None</code>.</li>
            <li><strong>extraConfigs</strong> (optional): Additional configurations, typically used for authentication.</li>
        </ul>
        <p><strong>Real-time Example:</strong></p>
        <code>
            dbutils.fs.updateMount(<br>
            &nbsp;&nbsp;source="wasbs://mycontainer@myaccount.blob.core.windows.net",<br>
            &nbsp;&nbsp;mountPoint="/mnt/mydata",<br>
            &nbsp;&nbsp;extraConfigs={"fs.azure.account.key.myaccount.blob.core.windows.net": "new-access-key"}<br>
            )<br>
            print("The mount point has been updated successfully.")
        </code>
    </li>
</ol>

<p><strong>Summary of Mount Methods</strong></p>
<table border="1">
    <tr>
        <th>Method</th>
        <th>Description</th>
        <th>Real-time Use Case</th>
    </tr>
    <tr>
        <td><code>dbutils.fs.mount()</code></td>
        <td>Mounts a source directory to DBFS.</td>
        <td>Connecting an Azure Data Lake Storage container to DBFS.</td>
    </tr>
    <tr>
        <td><code>dbutils.fs.mounts()</code></td>
        <td>Displays current mount points in DBFS.</td>
        <td>Listing all connected storage systems.</td>
    </tr>
    <tr>
        <td><code>dbutils.fs.refreshMounts()</code></td>
        <td>Refreshes the mount cache across the cluster.</td>
        <td>Ensuring all nodes have up-to-date mount information.</td>
    </tr>
    <tr>
        <td><code>dbutils.fs.unmount()</code></td>
        <td>Removes a specified mount point from DBFS.</td>
        <td>Detaching an external storage from DBFS.</td>
    </tr>
    <tr>
        <td><code>dbutils.fs.updateMount()</code></td>
        <td>Updates an existing mount point.</td>
        <td>Updating mount credentials after a key rotation.</td>
    </tr>
</table>

<p><strong>Conclusion</strong></p>
<p>The <code>dbutils.fs</code> mount-related methods in Databricks provide powerful tools to connect external storage systems to the Databricks File System (DBFS). These functions facilitate seamless data integration and management, enabling you to interact with data stored outside of your Databricks environment as if it were part of your workspace. By using these methods effectively, you can build flexible and scalable data solutions within your Databricks clusters.</p>

<h2>dbutils.widgets</h2>
<p>The <strong>dbutils.widgets</strong> module in Databricks provides a powerful way to create interactive input controls (widgets) within your notebooks, allowing users to input values dynamically. These widgets are useful for parameterizing notebooks, creating dynamic reports, or enabling data exploration in a more user-friendly way.</p>

<p>Here’s a detailed explanation of the key methods of <code>dbutils.widgets</code> along with real-time examples using Python:</p>

<strong>Overview of dbutils.widgets Methods</strong>

<ol>
    <li><code>combobox()</code> - Creates a combobox widget.</li>
    <li><code>dropdown()</code> - Creates a dropdown widget.</li>
    <li><code>multiselect()</code> - Creates a multi-select widget.</li>
    <li><code>text()</code> - Creates a text input widget.</li>
    <li><code>get()</code> - Retrieves the current value of a widget.</li>
    <li><code>getAll()</code> - Retrieves all widget values as a dictionary.</li>
    <li><code>remove()</code> - Removes a specific widget.</li>
    <li><code>removeAll()</code> - Removes all widgets from the notebook.</li>
</ol>

<p>Let’s dive into each of these methods in detail.</p>

<strong>1. dbutils.widgets.combobox() - Creating a Combobox Widget</strong>
<p>The <code>combobox</code> method creates an input widget with a dropdown menu that allows users to either select from the available choices or type their own input.</p>

<p><strong>Syntax:</strong></p>
<code>dbutils.widgets.combobox(name: str, defaultValue: str, choices: list, label: str)</code>

<p><strong>Real-time Example:</strong></p>
<code>
# Create a combobox widget for selecting a country<br>
dbutils.widgets.combobox("country", "USA", ["USA", "Canada", "Mexico"], "Select a Country")<br>
<br>
# Retrieve the selected value<br>
selected_country = dbutils.widgets.get("country")<br>
print(f"Selected Country: {selected_country}")<br>
</code>

<strong>2. dbutils.widgets.dropdown() - Creating a Dropdown Widget</strong>
<p>The <code>dropdown</code> method creates a dropdown widget where users can select one value from a predefined list of options.</p>

<p><strong>Syntax:</strong></p>
<code>dbutils.widgets.dropdown(name: str, defaultValue: str, choices: list, label: str)</code>

<p><strong>Real-time Example:</strong></p>
<code>
# Create a dropdown widget for selecting a processing mode<br>
dbutils.widgets.dropdown("processing_mode", "fast", ["fast", "medium", "slow"], "Processing Mode")<br>
<br>
# Retrieve the selected value<br>
processing_mode = dbutils.widgets.get("processing_mode")<br>
print(f"Selected Processing Mode: {processing_mode}")<br>
</code>

<strong>3. dbutils.widgets.multiselect() - Creating a Multiselect Widget</strong>
<p>The <code>multiselect</code> method creates a widget that allows users to select multiple values from a predefined list of options.</p>

<p><strong>Syntax:</strong></p>
<code>dbutils.widgets.multiselect(name: str, defaultValue: str, choices: list, label: str)</code>

<p><strong>Real-time Example:</strong></p>
<code>
# Create a multiselect widget for selecting data columns<br>
dbutils.widgets.multiselect("columns", "age", ["age", "salary", "department"], "Select Columns")<br>
<br>
# Retrieve the selected values (comma-separated string)<br>
selected_columns = dbutils.widgets.get("columns")<br>
print(f"Selected Columns: {selected_columns}")<br>
</code>

<strong>4. dbutils.widgets.text() - Creating a Text Input Widget</strong>
<p>The <code>text</code> method creates a text input widget where users can enter any string value.</p>

<p><strong>Syntax:</strong></p>
<code>dbutils.widgets.text(name: str, defaultValue: str, label: str)</code>

<p><strong>Real-time Example:</strong></p>
<code>
# Create a text widget for entering a file path<br>
dbutils.widgets.text("file_path", "/mnt/data/input.csv", "Enter File Path")<br>
<br>
# Retrieve the entered value<br>
file_path = dbutils.widgets.get("file_path")<br>
print(f"Entered File Path: {file_path}")<br>
</code>

<strong>5. dbutils.widgets.get() - Retrieving Widget Values</strong>
<p>The <code>get</code> method retrieves the current value of a specified widget.</p>

<p><strong>Syntax:</strong></p>
<code>dbutils.widgets.get(name: str) -> str</code>

<p><strong>Real-time Example:</strong></p>
<code>
# Assuming we have a widget named "processing_mode"<br>
processing_mode = dbutils.widgets.get("processing_mode")<br>
print(f"Current Processing Mode: {processing_mode}")<br>
</code>

<strong>6. dbutils.widgets.getAll() - Retrieving All Widget Values</strong>
<p>The <code>getAll</code> method returns a dictionary containing all current widget values.</p>

<p><strong>Syntax:</strong></p>
<code>dbutils.widgets.getAll() -> dict</code>

<p><strong>Real-time Example:</strong></p>
<code>
# Retrieve all widget values<br>
all_widget_values = dbutils.widgets.getAll()<br>
print("All Widget Values:", all_widget_values)<br>
</code>

<strong>7. dbutils.widgets.remove() - Removing a Specific Widget</strong>
<p>The <code>remove</code> method removes a specified widget from the notebook.</p>

<p><strong>Syntax:</strong></p>
<code>dbutils.widgets.remove(name: str)</code>

<p><strong>Real-time Example:</strong></p>
<code>
# Remove the "processing_mode" widget<br>
dbutils.widgets.remove("processing_mode")<br>
</code>

<strong>8. dbutils.widgets.removeAll() - Removing All Widgets</strong>
<p>The <code>removeAll</code> method removes all widgets from the notebook, useful for cleaning up after execution.</p>

<p><strong>Syntax:</strong></p>
<code>dbutils.widgets.removeAll()</code>

<p><strong>Real-time Example:</strong></p>
<code>
# Remove all widgets from the notebook<br>
dbutils.widgets.removeAll()<br>
</code>

<strong>Summary of dbutils.widgets Methods</strong>
<table border="1">
    <tr>
        <th>Method</th>
        <th>Description</th>
        <th>Real-time Use Case</th>
    </tr>
    <tr>
        <td><code>dbutils.widgets.combobox()</code></td>
        <td>Creates a combobox widget with options and a default value.</td>
        <td>Allowing users to select or enter a custom category.</td>
    </tr>
    <tr>
        <td><code>dbutils.widgets.dropdown()</code></td>
        <td>Creates a dropdown widget with pre-defined choices.</td>
        <td>Selecting a processing mode (e.g., "fast", "medium", "slow").</td>
    </tr>
    <tr>
        <td><code>dbutils.widgets.multiselect()</code></td>
        <td>Creates a widget for selecting multiple options.</td>
        <td>Choosing multiple columns for data analysis.</td>
    </tr>
    <tr>
        <td><code>dbutils.widgets.text()</code></td>
        <td>Creates a text input widget for user-defined values.</td>
        <td>Entering file paths or custom parameter values.</td>
    </tr>
    <tr>
        <td><code>dbutils.widgets.get()</code></td>
        <td>Retrieves the current value of a widget.</td>
        <td>Fetching user input for further processing.</td>
    </tr>
    <tr>
        <td><code>dbutils.widgets.getAll()</code></td>
        <td>Retrieves a dictionary of all current widget values.</td>
        <td>Accessing all input parameters in a single step.</td>
    </tr>
    <tr>
        <td><code>dbutils.widgets.remove()</code></td>
        <td>Removes a specific widget from the notebook interface.</td>
        <td>Removing irrelevant widgets after use.</td>
    </tr>
    <tr>
        <td><code>dbutils.widgets.removeAll()</code></td>
        <td>Removes all widgets, cleaning up the notebook interface.</td>
        <td>Resetting the notebook to its original state.</td>
    </tr>
</table>

<p><strong>Conclusion</strong></p>
<p>The <code>dbutils.widgets</code> module provides a set of powerful methods to create interactive and dynamic notebooks in Databricks. By using these widgets, you can enhance user experience, parameterize workflows, and create more engaging data analysis and reporting environments. This makes your notebooks flexible, reusable, and adaptable for different scenarios and users.</p>


<h2>dbutils.notebook</h2>
<p>The dbutils.notebook module in Databricks provides a set of utility functions to work with notebooks. It allows you to run, exit, manage parameters, and interact with different notebooks within a Databricks environment. These utilities are particularly useful for orchestrating workflows where multiple notebooks are involved in a data pipeline.</p>

<p>Here’s a detailed explanation of the key methods of dbutils.notebook along with real-time examples using Python:</p>

```python
dbutils.notebook.help()

#output
The notebook module.
exit(value: String): void -> This method lets you exit a notebook with a value
run(path: String, timeoutSeconds: int, arguments: Map): String -> This method runs a notebook and returns its exit value
```
<ol>
    <li><strong>dbutils.notebook.run() - Running Another Notebook</strong></li>
    <p>The run method allows you to call another notebook from the current notebook. This is useful when you want to modularize your workflows by breaking them into smaller, reusable notebooks.</p>
    <p><strong>Syntax : </strong>dbutils.notebook.run(path: str, timeout_seconds: int, arguments: dict = None)</p>
    <p><strong>Parameters</strong></p>
    <ul>
        <li><strong>path (String)</strong>: The path to the notebook you want to run (relative or absolute).</li>
        <li><strong>timeout_seconds (Integer)</strong>: The maximum time to wait for the notebook to run (in seconds). A value of 0 means no timeout.</li>
        <li><strong>arguments (Dictionary, optional)</strong>: Key-value pairs to pass parameters to the notebook.</li>
    </ul>
    <p><strong>Real time example : Let's say you have a main notebook that needs to run a data processing notebook named DataProcessing.</strong></p>

```python
# Running the DataProcessing notebook with parameters
result = dbutils.notebook.run("/Users/example@databricks.com/DataProcessing", 300, {"inputPath": "/mnt/data/input", "outputPath": "/mnt/data/output"})
print("Result from DataProcessing notebook:", result)

```
    <p><strong>Explnation : </strong>In this example, the notebook /Users/example@databricks.com/DataProcessing is executed with a timeout of 300 seconds. The inputPath and outputPath parameters are passed to the target notebook.</p>
    <li><strong>dbutils.notebook.exit() - Exiting a Notebook</strong></li>
    <p>The exit method allows you to exit a notebook and return a string value to the calling notebook or job. This method is typically used when you want to return a status, message, or result from the notebook.</p>
    <p><strong>Syntax</strong> : dbutils.notebook.exit(value: str)</p>
    <p><strong>Parameters</strong></p>
    <ul>
        <li><strong>value (String)</strong>: The value to return when exiting the notebook.</li>
    </ul>
    <p><strong>Real-time Example:</strong>Imagine that at the end of a notebook's execution, you want to return a success message.</p>

```python
# Performing some data processing
print("Data processing completed successfully.")

# Exiting the notebook with a success message
dbutils.notebook.exit("Data processing completed successfully.")
```
    <p><strong>Explanation : </strong>The string "Data processing completed successfully." will be returned to any notebook or job that executed this notebook using dbutils.notebook.run().</p>
</ol>


<h2>%run</h2>
<p><strong>%run Command in Databricks</strong></p>

<p>The <code>%run</code> command in Databricks is used to execute a notebook from another notebook. It's similar to how functions or libraries are imported and reused in programming. In Databricks, you often divide large notebooks into smaller ones to keep the code clean, modular, and reusable. Using <code>%run</code>, you can integrate or reuse logic from other notebooks efficiently.</p>

<p><strong>1. Basic Usage of %run</strong></p>

<p>The basic form of <code>%run</code> lets you run another notebook as if it were part of the current notebook. This is useful when the other notebook contains reusable code, functions, or classes.</p>

<p><strong>Example:</strong></p>
<code>%run /path/to/notebook</code>

<p><ul>
<li><code>/path/to/notebook</code>: This is the path to the notebook you want to execute. The code from this notebook is executed as part of the current notebook.</li>
<li>All the variables, functions, and classes defined in the executed notebook become available in the current one.</li>
</ul></p>

<p><strong>Key Points:</strong></p>
<ul>
<li>You cannot run <code>%run</code> on files like .py. It is specifically for running other Databricks notebooks.</li>
<li>Once you execute a notebook using <code>%run</code>, all its contents, including functions and variables, are loaded into the current notebook context.</li>
</ul>

<p><strong>2. Using Parameters with %run</strong></p>

<p>Databricks allows passing parameters from one notebook to another using the <code>$</code> syntax. The parameters are key-value pairs that are passed into the executed notebook.</p>

<p><strong>Example: Without parameters</strong></p>
<code>%run /path/to/child_notebook</code>

<p><strong>Example: Passing parameters using $</strong></p>
<code>%run /path/to/child_notebook $param1="value1" $param2="value2"</code>

<ul>
<li><strong>$param1</strong> and <strong>$param2</strong> are parameters being passed to the child notebook.</li>
<li>In the <strong>child notebook</strong>, you can access these parameters using <code>dbutils.widgets.get</code>.</li>
</ul>

<p><strong>In the child notebook (receiving parameters):</strong></p>
<code>
dbutils.widgets.text("param1", "default_value1")<br>
dbutils.widgets.text("param2", "default_value2")<br>
param1 = dbutils.widgets.get("param1")<br>
param2 = dbutils.widgets.get("param2")<br>
print("Param1:", param1)<br>
print("Param2:", param2)
</code>

<p><strong>Key Points:</strong></p>
<ul>
<li>Parameters should be declared using <code>dbutils.widgets.text()</code> in the child notebook.</li>
<li>You can retrieve the passed parameters using <code>dbutils.widgets.get()</code>.</li>
<li>If a parameter is not passed, the widget's default value will be used.</li>
</ul>

<p><strong>3. Dynamic Path to Notebook</strong></p>

<p>You can also use Python variables to dynamically specify the path of the notebook being run.</p>

<p><strong>Example:</strong></p>
<code>notebook_path = "/path/to/notebook"<br>
%run $notebook_path</code>

<p><ul>
<li><strong>notebook_path</strong> is a variable that holds the path of the notebook to be executed.</li>
<li>This is useful when you want to execute different notebooks dynamically based on certain conditions.</li>
</ul></p>

<p><strong>4. Error Handling in %run</strong></p>

<p>If the notebook being run by <code>%run</code> fails, the current notebook will stop executing as well, similar to how an exception would propagate in Python.</p>

<p><strong>Example:</strong></p>
<code>
try:<br>
&nbsp;&nbsp;&nbsp;&nbsp;%run /path/to/failing_notebook<br>
except Exception as e:<br>
&nbsp;&nbsp;&nbsp;&nbsp;print(f"Failed to run the notebook: {e}")<br>
</code>

<p><strong>5. Reusability and Modularity with %run</strong></p>

<p>One of the key benefits of using <code>%run</code> is reusability. For instance, you can define common utility functions or configurations in one notebook and reuse them across multiple notebooks.</p>

<p><strong>Example: Reusing a utility notebook</strong></p>
<code>
# Common utility notebook<br>
def add_numbers(a, b):<br>
&nbsp;&nbsp;&nbsp;&nbsp;return a + b
</code>

<p>You can now run this notebook in another notebook:</p>
<code>
%run /path/to/common_utility_notebook<br>
result = add_numbers(5, 10)<br>
print(result)  # Output will be 15
</code>

<p><strong>6. Global Scope with %run</strong></p>

<p>When you run a notebook using <code>%run</code>, all the functions, variables, and classes defined in the child notebook become part of the global scope of the parent notebook. This allows you to seamlessly access and use them as if they were defined in the parent notebook.</p>

<p><strong>Example:</strong></p>
<code>
# child_notebook.py<br>
x = 10<br>
y = 20<br>
def multiply(a, b):<br>
&nbsp;&nbsp;&nbsp;&nbsp;return a * b
</code>

<p><code>
%run /path/to/child_notebook<br>
print(x)  # Output: 10<br>
print(y)  # Output: 20<br>
print(multiply(2, 3))  # Output: 6
</code></p>

<p><strong>Summary of %run in Databricks:</strong></p>
<ul>
<li><strong>Basic Execution</strong>: Use <code>%run</code> to execute code from another notebook.</li>
<li><strong>Passing Parameters</strong>: Parameters can be passed using <code>$param=value</code> and accessed using <code>dbutils.widgets.get</code> in the child notebook.</li>
<li><strong>Dynamic Paths</strong>: You can dynamically specify notebook paths using variables.</li>
<li><strong>Reusability</strong>: It's ideal for reusing utility functions or configuration across multiple notebooks.</li>
<li><strong>Error Handling</strong>: If a child notebook fails, the current notebook will also stop execution unless handled in a <code>try-except</code> block.</li>
<li><strong>Global Scope</strong>: All variables, functions, and classes defined in the child notebook are accessible in the parent notebook.</li>
</ul>

<h2>dbutils.secrets</h2>
<p><strong>Detailed Explanation of dbutils.secrets in Databricks</strong></p>

<p>In Databricks, managing sensitive data such as passwords, API keys, or tokens securely is crucial. The <code>dbutils.secrets</code> module provides utilities for securely storing and retrieving secrets within Databricks notebooks. Secrets allow you to avoid hardcoding sensitive information in your notebooks, enabling safe collaboration and deployment.</p>

<p><strong>1. What is dbutils.secrets?</strong></p>

<p><code>dbutils.secrets</code> is a Databricks utility that helps manage secrets securely by allowing you to:</p>

<ul>
    <li>Store sensitive information in a dedicated, encrypted space called a <strong>secret scope</strong>.</li>
    <li>Retrieve secrets securely using the scope and key.</li>
    <li>Manage secret scopes, and view metadata associated with stored secrets.</li>
</ul>

<p><strong>2. Key Components of dbutils.secrets</strong></p>

<p>There are four main methods provided by <code>dbutils.secrets</code>:</p>

<ol>
    <li><code>get(scope: String, key: String): String</code> - Retrieves the string value of a secret from a specified scope and key.</li>
    <li><code>getBytes(scope: String, key: String): byte[]</code> - Retrieves the byte array of a secret value from a specified scope and key.</li>
    <li><code>list(scope: String): Seq</code> - Lists metadata for all secrets within the specified secret scope (does not return actual values).</li>
    <li><code>listScopes(): Seq</code> - Lists all available secret scopes in the Databricks workspace.</li>
</ol>

<p><strong>3. How dbutils.secrets Works</strong></p>

<p>To manage secrets using <code>dbutils.secrets</code>, you need to follow these steps:</p>

<ol>
    <li><strong>Create a Secret Scope</strong>: Secret scopes act as containers for your secrets.</li>
    <li><strong>Store Secrets</strong>: Store sensitive data inside it using a key-value pair (where key is the name, and value is the secret itself).</li>
    <li><strong>Access Secrets</strong>: Access these secrets securely in your notebooks.</li>
</ol>

<p><strong>Step 1: Create a Secret Scope</strong></p>
<code>databricks secrets create-scope --scope my-scope</code>

<p>This command creates a new secret scope called <strong>my-scope</strong>.</p>

<p><strong>Step 2: Store a Secret</strong></p>
<code>databricks secrets put --scope my-scope --key my-api-key</code>

<p>This command stores a secret (<code>my-api-key</code>) under the scope <strong>my-scope</strong>.</p>

<p><strong>Step 3: Access Secrets Using dbutils.secrets.get</strong></p>
<code>api_key = dbutils.secrets.get(scope="my-scope", key="my-api-key")</code>

<p>In this case, the API key is retrieved and stored in the variable <code>api_key</code> without exposing it in the notebook.</p>

<p><strong>4. Real-World Use Cases of dbutils.secrets</strong></p>

<p><strong>Example 1: Accessing API Keys for Data Ingestion</strong></p>
<code>
# Fetching the API key from the secret scope<br>
gcp_api_key = dbutils.secrets.get(scope="cloud-secrets", key="gcp-api-key")<br><br>

# Using the API key to make a request to Google Cloud API<br>
import requests<br><br>

url = f"https://cloud.googleapis.com/compute/v1/projects/my-project/zones/us-central1-a"<br>
headers = {"Authorization": f"Bearer {gcp_api_key}"}<br><br>

response = requests.get(url, headers=headers)<br>
print(response.json())
</code>

<p>In this example:</p>
<ul>
    <li>The API key for Google Cloud is stored securely in the secret scope <code>cloud-secrets</code> under the key <code>gcp-api-key</code>.</li>
    <li>The key is retrieved using <code>dbutils.secrets.get()</code>, and then passed as a Bearer token in the HTTP request header.</li>
</ul>

<p><strong>Example 2: Connecting to a Database Securely</strong></p>
<code>
# Fetching database credentials securely<br>
db_username = dbutils.secrets.get(scope="db-secrets", key="db-username")<br>
db_password = dbutils.secrets.get(scope="db-secrets", key="db-password")<br><br>

# Connecting to the database using the credentials<br>
import pymysql<br><br>

connection = pymysql.connect(<br>
&nbsp;&nbsp;&nbsp;&nbsp;host="your-database-host",<br>
&nbsp;&nbsp;&nbsp;&nbsp;user=db_username,<br>
&nbsp;&nbsp;&nbsp;&nbsp;password=db_password,<br>
&nbsp;&nbsp;&nbsp;&nbsp;db="your-database-name"<br>
)<br><br>

# Executing a query securely<br>
cursor = connection.cursor()<br>
cursor.execute("SELECT * FROM customers")<br>
result = cursor.fetchall()<br><br>

# Printing results<br>
for row in result:<br>
&nbsp;&nbsp;&nbsp;&nbsp;print(row)
</code>

<p>In this example:</p>
<ul>
    <li>The database username and password are stored securely in <code>db-secrets</code>.</li>
    <li>These credentials are retrieved using <code>dbutils.secrets.get()</code> and used to connect to the database without exposing sensitive information in the notebook.</li>
</ul>

<p><strong>5. Other Methods in dbutils.secrets</strong></p>

<p><strong>Listing Secrets in a Scope</strong></p>
<code>
# List all secrets in the "cloud-secrets" scope<br>
secret_metadata = dbutils.secrets.list(scope="cloud-secrets")<br>
for secret in secret_metadata:<br>
&nbsp;&nbsp;&nbsp;&nbsp;print(secret)
</code>

<p>This will list metadata like secret names, but not the actual values.</p>

<p><strong>Listing Available Secret Scopes</strong></p>
<code>
# List all secret scopes in the workspace<br>
scopes = dbutils.secrets.listScopes()<br>
for scope in scopes:<br>
&nbsp;&nbsp;&nbsp;&nbsp;print(scope.name)
</code>

<p><strong>6. Security Best Practices</strong></p>

<ul>
    <li><strong>Avoid Hardcoding</strong>: Always use <code>dbutils.secrets.get()</code> to retrieve sensitive information, avoiding hardcoding secrets directly into notebooks.</li>
    <li><strong>Use Secret Scopes</strong>: Store all secrets in secret scopes to ensure they are encrypted and managed securely.</li>
    <li><strong>Access Controls</strong>: Use Databricks’ Access Control Lists (ACLs) to manage access to secret scopes.</li>
</ul>

<p><strong>7. Error Handling with Secrets</strong></p>
<code>
try:<br>
&nbsp;&nbsp;&nbsp;&nbsp;secret_value = dbutils.secrets.get(scope="invalid-scope", key="nonexistent-key")<br>
except Exception as e:<br>
&nbsp;&nbsp;&nbsp;&nbsp;print(f"Failed to retrieve secret: {e}")
</code>

<p>This ensures that your code does not fail silently, and you get clear feedback if something goes wrong.</p>

<p><strong>8. Summary of dbutils.secrets</strong></p>

<ul>
    <li><strong>Security</strong>: <code>dbutils.secrets</code> helps manage secrets securely in Databricks.</li>
    <li><strong>Functions</strong>: Key functions include <code>get()</code>, <code>getBytes()</code>, <code>list()</code>, and <code>listScopes()</code>.</li>
    <li><strong>Best Practices</strong>: Always retrieve secrets using secure scopes, and never hardcode sensitive information.</li>
    <li><strong>Real-World Use Cases</strong>: API keys for external services, database credentials, or any sensitive configurations.</li>
</ul>
