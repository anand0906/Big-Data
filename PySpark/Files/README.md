<h1>Reading CSV Files Using PySpark</h1>

<p>PySpark, the Python API for Apache Spark, allows easy manipulation and analysis of big data, including reading CSV files. Spark’s <code>DataFrameReader</code> provides several options for reading CSV files.</p>

<p><strong>1. Importing Required Libraries</strong></p>
<p>First, ensure you have the necessary PySpark modules imported:</p>
<code>
from pyspark.sql import SparkSession
</code>

<p><strong>2. Creating a SparkSession</strong></p>
<p>A <code>SparkSession</code> is the entry point to any PySpark functionality:</p>
<code>
spark = SparkSession.builder \
    .appName("CSV Reader Example") \
    .getOrCreate()
</code>

<p><strong>3. Reading a CSV File</strong></p>
<p>To read a CSV file, use the <code>read</code> method from the <code>SparkSession</code>:</p>
<code>
df = spark.read.csv("path_to_file.csv")
</code>

<p><strong>4. Options Available for Reading CSV Files</strong></p>
<p>You can customize how PySpark reads CSV files using different options. Here's a list of the most common options and their purposes:</p>

<ol>
  <li><strong>header</strong>
    <ul>
      <li><strong>Description</strong>: Determines whether the first line of the file contains the header or not.</li>
      <li><strong>Default</strong>: <code>false</code></li>
      <li><strong>Options</strong>: 
        <ul>
          <li><code>True</code>: First row is treated as column headers.</li>
          <li><code>False</code>: No headers, columns are given default names <code>_c0</code>, <code>_c1</code>, etc.</li>
        </ul>
      </li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("header", True).csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>inferSchema</strong>
    <ul>
      <li><strong>Description</strong>: Automatically infers the data types for each column.</li>
      <li><strong>Default</strong>: <code>false</code></li>
      <li><strong>Options</strong>:
        <ul>
          <li><code>True</code>: Infers column types (like int, float).</li>
          <li><code>False</code>: Treats all columns as <code>string</code>.</li>
        </ul>
      </li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("inferSchema", True).csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>delimiter</strong>
    <ul>
      <li><strong>Description</strong>: Specifies the character used as a delimiter between fields (default is a comma <code>,</code>).</li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("delimiter", ";").csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>quote</strong>
    <ul>
      <li><strong>Description</strong>: Specifies the character used to enclose values with special characters such as delimiters.</li>
      <li><strong>Default</strong>: <code>"</code></li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("quote", "'").csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>escape</strong>
    <ul>
      <li><strong>Description</strong>: Character used to escape quotes inside quoted fields. For example, using <code>\\"</code> as an escape character allows quotes inside quoted strings.</li>
      <li><strong>Default</strong>: <code>\</code></li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("escape", "\\\\").csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>nullValue</strong>
    <ul>
      <li><strong>Description</strong>: Specifies the string that should be treated as <code>null</code> values.</li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("nullValue", "NA").csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>mode</strong>
    <ul>
      <li><strong>Description</strong>: Defines how to handle files when there are corrupt records or issues.</li>
      <li><strong>Options</strong>:
        <ul>
          <li><code>PERMISSIVE</code> (default): Puts corrupt records in a separate column called <code>_corrupt_record</code>.</li>
          <li><code>DROPMALFORMED</code>: Discards rows that contain malformed records.</li>
          <li><code>FAILFAST</code>: Fails and throws an exception immediately if corrupted records are found.</li>
        </ul>
      </li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("mode", "DROPMALFORMED").csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>multiline</strong>
    <ul>
      <li><strong>Description</strong>: Allows CSV fields to span multiple lines.</li>
      <li><strong>Default</strong>: <code>False</code></li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("multiline", True).csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>charset</strong>
    <ul>
      <li><strong>Description</strong>: Character set encoding of the CSV file.</li>
      <li><strong>Default</strong>: <code>UTF-8</code></li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("charset", "ISO-8859-1").csv("path_to_file.csv")
    </code>
  </li>

  <li><strong>dateFormat</strong> and <strong>timestampFormat</strong>
    <ul>
      <li><strong>Description</strong>: Specifies custom date and timestamp formats for parsing date/time fields.</li>
    </ul>
    <p><strong>Example</strong>:</p>
    <code>
    df = spark.read.option("dateFormat", "yyyy-MM-dd").csv("path_to_file.csv")
    </code>
  </li>
</ol>

<p><strong>5. Practical Example with Multiple Options</strong></p>
<p>Here’s a comprehensive example where multiple options are used to read a CSV file with customization:</p>
<code>
df = spark.read.options(
    header=True,                # Use the first row as header
    inferSchema=True,           # Infer data types automatically
    delimiter=',',              # Use comma as delimiter
    nullValue='NA',             # Treat "NA" as null values
    quote='"',                  # Use double quotes for values
    escape='\\',                # Escape special characters
    mode='DROPMALFORMED'        # Drop malformed records
).csv("path_to_file.csv")
df.show()  # Display the DataFrame
</code>

<p><strong>6. Saving DataFrames to CSV</strong></p>
<p>Once you've read a CSV file and performed your transformations, you can save the DataFrame back to a CSV file:</p>
<code>
df.write.option("header", True).csv("output_path.csv")
</code>

<p><strong>7. Conclusion</strong></p>
<p>Reading CSV files in PySpark is highly flexible, allowing you to handle headers, delimiters, and schema inference effectively. The available options enable fine-tuning of how CSV files are processed, making PySpark a powerful tool for large-scale data manipulation.</p>


<h1>Using StructType in PySpark</h1>

<p><strong>1. What is StructType?</strong></p>
<p>StructType is used in PySpark to define the structure of your data (called a schema) before reading it into a DataFrame. This schema tells PySpark the name of each column, its data type, and whether the column can have <code>null</code> values.</p>

<p><strong>2. Why Use StructType Instead of inferSchema?</strong></p>
<ul>
  <li><strong>Performance:</strong> Using <code>StructType</code> is faster because PySpark doesn't have to check the data to guess the types (which happens with <code>inferSchema</code>).</li>
  <li><strong>Data Accuracy:</strong> With <code>StructType</code>, you tell PySpark exactly what each column's type is, so there are no mistakes in understanding your data.</li>
  <li><strong>Custom Validation:</strong> Sometimes <code>inferSchema</code> might guess the wrong type (e.g., treating a number as a string). <code>StructType</code> avoids these mistakes by being explicit.</li>
  <li><strong>Consistency:</strong> When reading multiple files, you can ensure they all follow the same structure by defining the schema once with <code>StructType</code>.</li>
</ul>

<p><strong>3. How to Define a Schema with StructType</strong></p>
<p>To define a schema, you use <code>StructType</code> along with <code>StructField</code>, where each field specifies:</p>
<ul>
  <li>The column name</li>
  <li>The data type (like <code>StringType</code> or <code>IntegerType</code>)</li>
  <li>Whether the column can have <code>null</code> values</li>
</ul>

<p>Here’s an example:</p>
<code>
from pyspark.sql.types import StructType, StructField, StringType, IntegerType<br><br>
schema = StructType([<br>
&nbsp;&nbsp;StructField("Name", StringType(), True),<br>
&nbsp;&nbsp;StructField("Age", IntegerType(), True),<br>
&nbsp;&nbsp;StructField("Country", StringType(), True)<br>
])<br><br>
df = spark.read.schema(schema).csv("path_to_file.csv")
</code>

<p>This code defines a schema for a CSV file with three columns: <strong>Name</strong> (as a string), <strong>Age</strong> (as an integer), and <strong>Country</strong> (as a string).</p>

<p><strong>4. Example of Using StructType</strong></p>
<p>If you have a CSV file like this:</p>
<code>
Name,Age,Country<br>
John,30,USA<br>
Jane,25,Canada
</code>

<p>You can read it with a predefined schema like this:</p>
<code>
schema = StructType([<br>
&nbsp;&nbsp;StructField("Name", StringType(), True),<br>
&nbsp;&nbsp;StructField("Age", IntegerType(), True),<br>
&nbsp;&nbsp;StructField("Country", StringType(), True)<br>
])<br><br>
df = spark.read.option("header", True).schema(schema).csv("path_to_file.csv")<br>
df.show()
</code>

<p>This will output:</p>
<code>
+----+---+-------+<br>
|Name|Age|Country|<br>
+----+---+-------+<br>
|John| 30|    USA|<br>
|Jane| 25| Canada|<br>
+----+---+-------+
</code>

<p><strong>5. Benefits of StructType over inferSchema</strong></p>
<ol>
  <li><strong>Faster Performance:</strong> StructType skips the step of scanning the data to guess the types, which is faster for large datasets.</li>
  <li><strong>Better Accuracy:</strong> It avoids mistakes that can happen when inferSchema guesses the wrong data types.</li>
  <li><strong>Handles Complex Data:</strong> With StructType, you can control exactly how to handle special cases like <code>null</code> values.</li>
  <li><strong>Reusable Schema:</strong> Once you define a schema with StructType, you can reuse it for multiple files or datasets to ensure consistency.</li>
</ol>

<p><strong>6. Conclusion</strong></p>
<p>Using <code>StructType</code> in PySpark is a more efficient and reliable way to define the structure of your data compared to <code>inferSchema</code>. It provides better control over how the data is processed, avoids errors, and ensures consistency when handling large datasets.</p>

<p><strong>Parquet, Avro, and ORC</strong> are three popular file formats used in big data processing. They are optimized for performance, storage efficiency, and scalability, especially in distributed computing environments like Hadoop and Spark. Let's explore each of them:</p>

<ol>
  <li>
    <strong>Parquet</strong>
    <ul>
      <li><strong>File Type</strong>: Columnar</li>
      <li><strong>Compression</strong>: Supports multiple compression algorithms (Snappy, Gzip, LZO, etc.)</li>
      <li><strong>Schema</strong>: Self-describing schema (schema is stored with the file)</li>
      <li><strong>Best Use Case</strong>: Analytical workloads with complex queries</li>
    </ul>
    <p><strong>Key Features</strong>:</p>
    <ul>
      <li><strong>Columnar Storage</strong>: Parquet stores data in a column-wise format. This means data is grouped by columns rather than rows, which makes it highly efficient for read-heavy analytical queries where you only need to access certain columns.</li>
      <li><strong>Efficient Compression</strong>: Parquet applies compression at the column level, which improves space savings, especially when columns contain similar types of data.</li>
      <li><strong>Splittable</strong>: Files can be split for parallel processing, which is useful in distributed computing environments like Hadoop.</li>
    </ul>
    <p><strong>Example Usage</strong>:</p>
    <p>Parquet is commonly used in data warehouse scenarios where frequent querying is required on large datasets, such as in tools like Hive, Spark, or AWS Athena.</p>
  </li>

  <li>
    <strong>Avro</strong>
    <ul>
      <li><strong>File Type</strong>: Row-based</li>
      <li><strong>Compression</strong>: Supports different compression codecs (Snappy, Deflate, etc.)</li>
      <li><strong>Schema</strong>: Schema is required to read and write data</li>
      <li><strong>Best Use Case</strong>: Data serialization and streaming data</li>
    </ul>
    <p><strong>Key Features</strong>:</p>
    <ul>
      <li><strong>Row-Oriented Storage</strong>: Avro stores data row-wise, meaning all fields in a row are written together. This format is ideal for scenarios where entire rows of data are frequently accessed.</li>
      <li><strong>Schema Evolution</strong>: Avro stores the schema alongside the data, making it easy to handle changes in schema over time without breaking compatibility.</li>
      <li><strong>Compact Binary Format</strong>: Avro uses a compact binary encoding format, making it lightweight and fast for data serialization.</li>
      <li><strong>RPC Support</strong>: It provides built-in support for remote procedure calls (RPC), making it useful in distributed systems for communication between services.</li>
    </ul>
    <p><strong>Example Usage</strong>:</p>
    <p>Avro is often used for log files, messaging queues, and data pipelines (e.g., Kafka), where schema evolution is important.</p>
  </li>

  <li>
    <strong>ORC (Optimized Row Columnar)</strong>
    <ul>
      <li><strong>File Type</strong>: Columnar</li>
      <li><strong>Compression</strong>: Supports multiple compression codecs (Zlib, Snappy, etc.)</li>
      <li><strong>Schema</strong>: Self-describing schema</li>
      <li><strong>Best Use Case</strong>: Optimized for Hadoop and HDFS environments</li>
    </ul>
    <p><strong>Key Features</strong>:</p>
    <ul>
      <li><strong>Columnar Storage</strong>: Like Parquet, ORC also stores data in a columnar format, optimizing it for analytical queries that need access to specific columns.</li>
      <li><strong>Lightweight Indexes</strong>: ORC stores lightweight indexes within the file, including min/max values for each column, making query execution faster.</li>
      <li><strong>Efficient Compression</strong>: ORC applies efficient compression at the block level and stores metadata for columns, which minimizes disk usage and speeds up query processing.</li>
      <li><strong>Optimized for Hadoop</strong>: ORC was specifically designed to improve upon the limitations of earlier formats in the Hadoop ecosystem, making it highly optimized for Hadoop workloads.</li>
    </ul>
    <p><strong>Example Usage</strong>:</p>
    <p>ORC is widely used in Hive and other Hadoop-based tools where large-scale batch processing is performed. It’s especially good for queries that scan large datasets.</p>
  </li>
</ol>

<p><strong>Comparison of Parquet, Avro, and ORC</strong></p>

<table>
  <tr>
    <th>Feature</th>
    <th>Parquet</th>
    <th>Avro</th>
    <th>ORC</th>
  </tr>
  <tr>
    <td><strong>Storage Format</strong></td>
    <td>Columnar</td>
    <td>Row-based</td>
    <td>Columnar</td>
  </tr>
  <tr>
    <td><strong>Compression</strong></td>
    <td>Column-wise</td>
    <td>Block-level</td>
    <td>Column-wise</td>
  </tr>
  <tr>
    <td><strong>Schema Evolution</strong></td>
    <td>Supported</td>
    <td>Supported</td>
    <td>Supported</td>
  </tr>
  <tr>
    <td><strong>Read/Write Pattern</strong></td>
    <td>Optimized for read-heavy</td>
    <td>Optimized for write-heavy</td>
    <td>Optimized for read-heavy</td>
  </tr>
  <tr>
    <td><strong>Use Case</strong></td>
    <td>Analytical queries, data lake</td>
    <td>Data serialization, streaming</td>
    <td>Large-scale batch processing</td>
  </tr>
</table>

<p><strong>When to Use Each Format</strong>:</p>
<ul>
  <li><strong>Parquet</strong>: When performing complex analytical queries over large datasets, where only specific columns are needed.</li>
  <li><strong>Avro</strong>: For data serialization and when schema evolution is important, such as in data pipelines or streaming data.</li>
  <li><strong>ORC</strong>: When working in Hadoop-based environments and optimizing for large-scale, read-heavy workloads with analytical queries.</li>
</ul>

<h1>Readinf Parquet,Avro,Orc Files</h1>
<p><strong>1. Parquet File Format in Spark</strong></p>

<p><strong>Reading Parquet Files</strong></p>
<p>You can use the built-in <code>spark.read.parquet()</code> method to read Parquet files.</p>

<code>
# Read Parquet file into DataFrame
df = spark.read.parquet("path_to_parquet_file")

# Display contents
df.show()

# Show schema of the DataFrame
df.printSchema()
</code>

<p><strong>Writing Parquet Files</strong></p>
<p>You can write a Spark DataFrame to a Parquet file using the <code>write</code> method.</p>

<code>
# Write DataFrame to Parquet file
df.write.parquet("path_to_output_parquet_file")
</code>

<p><strong>Additional Options</strong></p>
<ul>
    <li><strong>Compression</strong>: Use <code>.option()</code> to specify the compression algorithm (snappy, gzip, lzo, etc.).</li>
    <li><strong>Partitioning</strong>: Partition the file by certain columns for better query performance.</li>
</ul>

<code>
# Writing Parquet with Snappy compression and partitioning by column
df.write.option("compression", "snappy").partitionBy("column_name").parquet("path_to_output_parquet_file")
</code>

<p><strong>Parquet File Options</strong></p>
<ul>
    <li><strong>mergeSchema</strong>: Merges schemas when reading multiple Parquet files.</li>
    <li><strong>compression</strong>: Specifies the compression algorithm (Snappy, Gzip, etc.).</li>
</ul>

<p><strong>2. Avro File Format in Spark</strong></p>

<p><strong>Setting Up Avro Support</strong></p>
<p>Avro is not bundled with Spark, so you need to include the <code>spark-avro</code> package when starting Spark.</p>

<code>
# Starting Spark with Avro package
spark-shell --packages org.apache.spark:spark-avro_2.12:3.0.0
</code>

<p><strong>Reading Avro Files</strong></p>
<p>To read an Avro file, you need to use the <code>format("avro")</code> method.</p>

<code>
# Read Avro file into DataFrame
df = spark.read.format("avro").load("path_to_avro_file")

# Show contents of the DataFrame
df.show()

# Print schema
df.printSchema()
</code>

<p><strong>Writing Avro Files</strong></p>
<p>To write a DataFrame to an Avro file, use the <code>write.format("avro")</code> method.</p>

<code>
# Write DataFrame to Avro file
df.write.format("avro").save("path_to_output_avro_file")
</code>

<p><strong>Additional Options</strong></p>
<code>
# Writing Avro file with Snappy compression
df.write.format("avro").option("compression", "snappy").save("path_to_output_avro_file")
</code>

<p><strong>Avro File Options</strong></p>
<ul>
    <li><strong>compression</strong>: Specifies the compression algorithm (Snappy, Deflate, etc.).</li>
    <li><strong>avroSchema</strong>: Explicitly sets the Avro schema.</li>
</ul>

<p><strong>3. ORC File Format in Spark</strong></p>

<p><strong>Reading ORC Files</strong></p>
<p>ORC is another columnar format, and Spark provides built-in support for it.</p>

<code>
# Read ORC file into DataFrame
df = spark.read.orc("path_to_orc_file")

# Show contents of the DataFrame
df.show()

# Print schema of the ORC file
df.printSchema()
</code>

<p><strong>Writing ORC Files</strong></p>
<p>You can write a DataFrame to an ORC file with the <code>write.orc()</code> method.</p>

<code>
# Write DataFrame to ORC file
df.write.orc("path_to_output_orc_file")
</code>

<p><strong>Additional Options</strong></p>
<code>
# Writing ORC file with Zlib compression and partitioning by a column
df.write.option("compression", "zlib").partitionBy("column_name").orc("path_to_output_orc_file")
</code>

<p><strong>ORC File Options</strong></p>
<ul>
    <li><strong>compression</strong>: Specifies the compression algorithm (Zlib, Snappy, etc.).</li>
    <li><strong>orcSchema</strong>: Specifies an explicit ORC schema.</li>
</ul>

<p><strong>General Options for Reading and Writing Files</strong></p>

<ul>
    <li><strong>path</strong>: The location of the file you want to read or write.</li>
    <li><strong>compression</strong>: You can set compression algorithms (Snappy, Gzip, Deflate, etc.).</li>
    <li><strong>mode</strong>: Determines how existing data is handled when writing files:
        <ul>
            <li><strong>overwrite</strong>: Overwrites the existing file or directory.</li>
            <li><strong>append</strong>: Appends data to an existing file or directory.</li>
            <li><strong>ignore</strong>: Ignores the operation if the data already exists.</li>
            <li><strong>error</strong>: Throws an error if the data exists (default behavior).</li>
        </ul>
    </li>
    <li><strong>partitionBy</strong>: Partitions the output by specified columns, improving query performance.</li>
    <li><strong>schema</strong>: Specifies an explicit schema when reading the file.</li>
</ul>

<p><strong>Reading Multiple File Formats Example</strong></p>

<code>
# Reading multiple formats
parquet_df = spark.read.parquet("path_to_parquet_file")
avro_df = spark.read.format("avro").load("path_to_avro_file")
orc_df = spark.read.orc("path_to_orc_file")

# Display the first few rows of each DataFrame
parquet_df.show()
avro_df.show()
orc_df.show()
</code>

<p><strong>Performance Tips</strong></p>
<ul>
    <li><strong>Use Compression</strong>: Always use compression when writing large datasets to save space and speed up I/O.</li>
    <li><strong>Schema Inference</strong>: If the schema of your data is known, explicitly define it when reading files to avoid the overhead of inferring the schema.</li>
</ul>

<code>
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Define schema
schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True)
])

# Read file with defined schema
df = spark.read.schema(schema).parquet("path_to_parquet_file")
</code>

<ul>
    <li><strong>Partitioning</strong>: Use partitioning wisely to optimize read performance for large datasets.</li>
</ul>

<code>
# Writing Parquet file partitioned by "year" and "month"
df.write.partitionBy("year", "month").parquet("path_to_output_parquet_file")
</code>

<ul>
    <li><strong>Parallelism</strong>: You can control the number of partitions to use while reading or writing large datasets for better parallelism.</li>
</ul>

<p><strong>Complete Example: Reading and Writing Files in Spark</strong></p>

<code>
from pyspark.sql import SparkSession

# Create SparkSession
spark = SparkSession.builder.appName("FileFormatExample").getOrCreate()

# Reading Parquet file
parquet_df = spark.read.parquet("path_to_parquet_file")
parquet_df.show()

# Writing DataFrame as Avro
parquet_df.write.format("avro").save("path_to_output_avro_file")

# Reading Avro file
avro_df = spark.read.format("avro").load("path_to_avro_file")
avro_df.show()

# Writing DataFrame as ORC with Zlib compression
avro_df.write.option("compression", "zlib").orc("path_to_output_orc_file")

# Stopping Spark session
spark.stop()
</code>

<h1>Common DataFrame Operations</h1>
<p><strong>Common Spark DataFrame Operations</strong></p>

<p>Here is an explanation of common Spark DataFrame operations used in PySpark for data manipulation, transformation, and analysis.</p>

<p><strong>1. Creating DataFrames</strong></p>

<p><strong>Creating a DataFrame from a List</strong></p>

<code>
from pyspark.sql import SparkSession<br>
# Create Spark session<br>
spark = SparkSession.builder.appName("DataFrameOperations").getOrCreate()<br><br>

# Example data<br>
data = [("John", 25), ("Anna", 28), ("Mike", 23)]<br><br>

# Define DataFrame schema<br>
columns = ["Name", "Age"]<br><br>

# Create DataFrame from list<br>
df = spark.createDataFrame(data, schema=columns)<br>
df.show()
</code>

<p><strong>Creating a DataFrame from CSV</strong></p>

<code>
# Reading a CSV file into DataFrame<br>
df = spark.read.csv("path_to_file.csv", header=True, inferSchema=True)<br>
df.show()
</code>

<p><strong>2. Basic Operations</strong></p>

<p><strong>show()</strong><br>
Displays the top rows of the DataFrame.</p>

<code>
df.show(5)  # Show top 5 rows
</code>

<p><strong>printSchema()</strong><br>
Prints the schema of the DataFrame.</p>

<code>
df.printSchema()
</code>

<p><strong>select()</strong><br>
Selects specific columns from the DataFrame.</p>

<code>
df.select("Name", "Age").show()
</code>

<p><strong>filter() or where()</strong><br>
Filters rows based on a condition.</p>

<code>
# Filter rows where Age > 25<br>
df.filter(df.Age > 25).show()<br><br>

# Another way to filter using where()<br>
df.where(df.Age > 25).show()
</code>

<p><strong>distinct()</strong><br>
Returns only distinct (unique) rows.</p>

<code>
df.select("Name").distinct().show()
</code>

<p><strong>3. Transformations</strong></p>

<p><strong>withColumn()</strong><br>
Adds a new column or modifies an existing column.</p>

<code>
# Add a new column "AgeInMonths"<br>
df = df.withColumn("AgeInMonths", df.Age * 12)<br>
df.show()
</code>

<p><strong>drop()</strong><br>
Drops a specific column from the DataFrame.</p>

<code>
# Drop the column "Age"<br>
df = df.drop("Age")<br>
df.show()
</code>

<p><strong>alias()</strong><br>
Provides an alias (temporary name) for a DataFrame or column.</p>

<code>
df.select(df.Name.alias("FullName")).show()
</code>

<p><strong>groupBy()</strong><br>
Groups the DataFrame by one or more columns.</p>

<code>
# Group by "Age" and count occurrences<br>
df.groupBy("Age").count().show()
</code>

<p><strong>orderBy() or sort()</strong><br>
Sorts the DataFrame based on one or more columns.</p>

<code>
# Order by Age in ascending order<br>
df.orderBy("Age").show()<br><br>

# Sort by Name in descending order<br>
df.sort(df.Name.desc()).show()
</code>

<p><strong>4. Joins</strong></p>

<p><strong>Inner Join</strong></p>

<code>
# Example DataFrames<br>
df1 = spark.createDataFrame([("John", 25), ("Anna", 28)], ["Name", "Age"])<br>
df2 = spark.createDataFrame([("John", "USA"), ("Anna", "UK")], ["Name", "Country"])<br><br>

# Inner join on "Name"<br>
df1.join(df2, on="Name", how="inner").show()
</code>

<p><strong>Left, Right, and Outer Joins</strong></p>

<code>
# Left join<br>
df1.join(df2, on="Name", how="left").show()<br><br>

# Right join<br>
df1.join(df2, on="Name", how="right").show()<br><br>

# Outer join<br>
df1.join(df2, on="Name", how="outer").show()
</code>

<p><strong>5. Aggregations</strong></p>

<p><strong>agg()</strong><br>
Used for aggregation on multiple columns.</p>

<code>
from pyspark.sql import functions as F<br><br>

# Aggregate data: get average age<br>
df.agg(F.avg("Age")).show()<br><br>

# Multiple aggregations: min, max, and average age<br>
df.agg(F.min("Age"), F.max("Age"), F.avg("Age")).show()
</code>

<p><strong>count()</strong><br>
Counts the number of rows in the DataFrame.</p>

<code>
# Count total number of rows<br>
df.count()
</code>

<p><strong>6. Working with Nulls</strong></p>

<p><strong>fillna()</strong><br>
Fills missing values with a specified value.</p>

<code>
# Fill all null values in the "Age" column with 0<br>
df.fillna(0, subset=["Age"]).show()
</code>

<p><strong>dropna()</strong><br>
Drops rows with null values.</p>

<code>
# Drop rows where any column has null values<br>
df.dropna().show()<br><br>

# Drop rows where "Age" is null<br>
df.dropna(subset=["Age"]).show()
</code>

<p><strong>7. Actions</strong></p>

<p><strong>collect()</strong><br>
Collects all the rows as a list of Row objects.</p>

<code>
rows = df.collect()<br>
for row in rows:<br>
&nbsp;&nbsp;&nbsp;&nbsp;print(row)
</code>

<p><strong>take()</strong><br>
Returns the first N rows as a list of Row objects.</p>

<code>
rows = df.take(3)<br>
for row in rows:<br>
&nbsp;&nbsp;&nbsp;&nbsp;print(row)
</code>

<p><strong>first()</strong><br>
Returns the first row of the DataFrame.</p>

<code>
row = df.first()<br>
print(row)
</code>

<p><strong>count()</strong><br>
Returns the total number of rows in the DataFrame.</p>

<code>
df.count()
</code>

<p><strong>describe()</strong><br>
Provides summary statistics (e.g., count, mean, stddev) for numeric columns.</p>

<code>
df.describe().show()
</code>

<p><strong>8. Saving DataFrames</strong></p>

<p><strong>Saving as CSV</strong></p>

<code>
# Save DataFrame as CSV<br>
df.write.csv("path_to_output_csv", header=True)
</code>

<p><strong>Saving as Parquet</strong></p>

<code>
# Save DataFrame as Parquet<br>
df.write.parquet("path_to_output_parquet")
</code>

<p><strong>9. Caching and Persistence</strong></p>

<p><strong>cache()</strong><br>
Caches the DataFrame in memory.</p>

<code>
df.cache()  # Cache the DataFrame<br>
df.count()  # Perform an action to cache the data
</code>

<p><strong>persist()</strong><br>
Persists the DataFrame with a specified storage level (e.g., MEMORY_AND_DISK).</p>

<code>
from pyspark import StorageLevel<br><br>

df.persist(StorageLevel.MEMORY_AND_DISK)
</code>

<p><strong>10. SQL Queries</strong></p>

<p>You can run SQL queries on DataFrames by registering them as a temporary view.</p>

<code>
# Register DataFrame as a SQL temporary view<br>
df.createOrReplaceTempView("people")<br><br>

# Run SQL query<br>
result = spark.sql("SELECT Name, Age FROM people WHERE Age > 25")<br>
result.show()
</code>


<h1>Reading Excel,XML,Json</h1>
<p><strong>1. Excel Files Reading and Writing Using PySpark</strong></p>

<p>PySpark does not provide direct support for reading and writing Excel files. However, we can use external libraries such as <strong>com.crealytics.spark.excel</strong> to work with Excel files. This requires the Spark Excel package.</p>

<p><strong>Reading Excel Files</strong></p>

<ol>
    <li>You need to add the required package to your Spark session to read Excel files.</li>
</ol>

<ul>
    <li><strong>Example:</strong></li>
</ul>

<p>
<code>
from pyspark.sql import SparkSession<br>
spark = SparkSession.builder.config("spark.jars.packages", "com.crealytics:spark-excel_2.12:0.13.5").getOrCreate()<br>
df = spark.read.format("com.crealytics.spark.excel").option("header", "true").option("inferSchema", "true").load("path_to_excel_file.xlsx")<br>
df.show()
</code>
</p>

<ul>
    <li><strong>Options for Reading Excel Files:</strong></li>
</ul>

<ol>
    <li><strong>header:</strong> Specifies whether the first row in the Excel file is treated as a header.</li>
    <li><strong>inferSchema:</strong> Automatically infers the schema of the data.</li>
    <li><strong>sheetName:</strong> You can specify the sheet name if the Excel file has multiple sheets.</li>
</ol>

<p><strong>Writing Excel Files</strong></p>

<ul>
    <li><strong>Example:</strong></li>
</ul>

<p>
<code>
df.write.format("com.crealytics.spark.excel").option("header", "true").mode("overwrite").save("path_to_output_excel_file.xlsx")
</code>
</p>

<p><strong>2. XML Files Reading and Writing Using PySpark</strong></p>

<p>For reading and writing XML files, PySpark uses the <strong>spark-xml</strong> library. You need to install the package <strong>com.databricks.spark.xml</strong>.</p>

<p><strong>Reading XML Files</strong></p>

<ul>
    <li><strong>Example:</strong></li>
</ul>

<p>
<code>
spark = SparkSession.builder.config("spark.jars.packages", "com.databricks:spark-xml_2.12:0.11.0").getOrCreate()<br>
df = spark.read.format("xml").option("rowTag", "record").load("path_to_xml_file.xml")<br>
df.show()
</code>
</p>

<ul>
    <li><strong>Options for Reading XML Files:</strong></li>
</ul>

<ol>
    <li><strong>rowTag:</strong> Defines the tag that is used as the root for each row.</li>
    <li><strong>inferSchema:</strong> Automatically infers the schema of the XML file.</li>
    <li><strong>attributePrefix:</strong> Prefix to use for attributes in the XML.</li>
</ol>

<p><strong>Writing XML Files</strong></p>

<ul>
    <li><strong>Example:</strong></li>
</ul>

<p>
<code>
df.write.format("xml").option("rootTag", "records").option("rowTag", "record").save("path_to_output_xml_file.xml")
</code>
</p>

<p><strong>3. JSON Files Reading and Writing Using PySpark</strong></p>

<p>PySpark provides built-in support for reading and writing JSON files.</p>

<p><strong>Reading JSON Files</strong></p>

<ul>
    <li><strong>Example:</strong></li>
</ul>

<p>
<code>
spark = SparkSession.builder.appName("JSONExample").getOrCreate()<br>
df = spark.read.json("path_to_json_file.json")<br>
df.show()
</code>
</p>

<ul>
    <li><strong>Options for Reading JSON Files:</strong></li>
</ul>

<ol>
    <li><strong>multiline:</strong> Set to true if the JSON data spans multiple lines (i.e., pretty-printed JSON).</li>
</ol>

<p><strong>Writing JSON Files</strong></p>

<ul>
    <li><strong>Example:</strong></li>
</ul>

<p>
<code>
df.write.json("path_to_output_json_file.json")
</code>
</p>

<ul>
    <li><strong>Options for Writing JSON Files:</strong></li>
</ul>

<ol>
    <li><strong>mode:</strong> Specifies the write mode, such as overwrite, append, ignore, or error.</li>
    <li><strong>compression:</strong> Allows compression of the output files (e.g., gzip, snappy).</li>
</ol>

<p><strong>Summary</strong></p>

<ul>
    <li><strong>Excel:</strong> You need to use the <strong>com.crealytics.spark.excel</strong> package for reading and writing Excel files.</li>
    <li><strong>XML:</strong> PySpark requires the <strong>com.databricks.spark.xml</strong> library to work with XML files.</li>
    <li><strong>JSON:</strong> PySpark has built-in support for JSON files.</li>
</ul>
