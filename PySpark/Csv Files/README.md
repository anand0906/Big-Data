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
