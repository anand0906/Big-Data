# Databricks Auto Loader - Incremental Data Processing Guide

## Table of Contents
1. [What is Auto Loader?](#what-is-auto-loader)
2. [Why Use Auto Loader?](#why-use-auto-loader)
3. [How Auto Loader Works](#how-auto-loader-works)
4. [Basic Setup](#basic-setup)
5. [Simple Examples](#simple-examples)
6. [Advanced Features](#advanced-features)
7. [Best Practices](#best-practices)
8. [Common Use Cases](#common-use-cases)
9. [Troubleshooting](#troubleshooting)

---

## What is Auto Loader?

Auto Loader is Databricks' **smart file watcher** that automatically processes new files as they arrive in cloud storage. Think of it as a **digital mailman** who instantly knows when new mail (files) arrives and delivers it to the right place.

### Key Features
- **Automatic Detection**: Finds new files without you checking manually
- **Incremental Processing**: Only processes new/changed files
- **Schema Evolution**: Automatically adapts to changes in data structure
- **Fault Tolerance**: Recovers from failures and continues processing

---

## Why Use Auto Loader?

### Traditional Approach Problems
```python
# Old way - you had to manually check for new files
files = dbutils.fs.ls("/path/to/data/")
for file in files:
    if file.name not in processed_files:
        process_file(file)  # Process each file manually
```

### Auto Loader Benefits
- **Real-time Processing**: Files processed as soon as they arrive
- **No Manual Monitoring**: Automatically detects new files
- **Cost Effective**: Only processes what's new
- **Reliable**: Built-in error handling and recovery

---

## How Auto Loader Works

### Architecture Overview
```
Cloud Storage → Auto Loader → Delta Table
     ↓              ↓            ↓
New Files    File Detection   Processed Data
```

### Two Detection Modes

**1. File Notification Mode** (Recommended)
- Uses cloud events (like AWS S3 events)
- Instant detection when files arrive
- More cost-effective for high-volume data

**2. Directory Listing Mode**
- Periodically scans directories
- Works when cloud events aren't available
- Good for smaller datasets

---

## Basic Setup

### Prerequisites
```python
# Import required libraries
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *
```

### Create Spark Session
```python
spark = SparkSession.builder \
    .appName("AutoLoaderExample") \
    .getOrCreate()
```

---

## Simple Examples

### Example 1: Basic CSV Auto Loader

**Scenario**: Customer data files arrive daily in CSV format

```python
# Setup paths
source_path = "/mnt/raw-data/customers/"
checkpoint_path = "/mnt/checkpoints/customers/"
target_table = "bronze.customers"

# Create Auto Loader stream
df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "csv") \
    .option("cloudFiles.schemaLocation", checkpoint_path + "/schema") \
    .option("header", "true") \
    .load(source_path)

# Write to Delta table
query = df.writeStream \
    .option("checkpointLocation", checkpoint_path) \
    .trigger(processingTime="1 minute") \
    .toTable(target_table)

# Start the stream
query.start().awaitTermination()
```

**What happens:**
1. Auto Loader watches `/mnt/raw-data/customers/` folder
2. When new CSV files arrive, it automatically reads them
3. Data gets written to `bronze.customers` Delta table
4. Process repeats every minute

### Example 2: JSON Files with Schema Evolution

**Scenario**: API logs in JSON format with changing structure

```python
# Define initial schema (optional)
initial_schema = StructType([
    StructField("timestamp", TimestampType(), True),
    StructField("user_id", StringType(), True),
    StructField("action", StringType(), True)
])

# Auto Loader with schema evolution
df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .option("cloudFiles.schemaLocation", "/mnt/checkpoints/api_logs/schema") \
    .option("cloudFiles.schemaEvolutionMode", "addNewColumns") \
    .schema(initial_schema) \
    .load("/mnt/raw-data/api-logs/")

# Add processing timestamp
df_processed = df.withColumn("processed_at", current_timestamp())

# Write to Delta table
query = df_processed.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/mnt/checkpoints/api_logs/") \
    .outputMode("append") \
    .table("bronze.api_logs")

query.start()
```

### Example 3: Parquet Files with Data Transformation

**Scenario**: Sales data in Parquet format needs cleaning

```python
def clean_sales_data(df):
    """Simple data cleaning function"""
    return df \
        .filter(col("amount") > 0) \
        .filter(col("customer_id").isNotNull()) \
        .withColumn("amount_rounded", round(col("amount"), 2))

# Auto Loader for Parquet files
source_df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "parquet") \
    .option("cloudFiles.schemaLocation", "/mnt/checkpoints/sales/schema") \
    .load("/mnt/raw-data/sales/")

# Apply transformations
cleaned_df = clean_sales_data(source_df)

# Write to target table
query = cleaned_df.writeStream \
    .option("checkpointLocation", "/mnt/checkpoints/sales/") \
    .trigger(once=True)  # Process once then stop \
    .toTable("silver.sales_clean")

query.start().awaitTermination()
```

---

## Advanced Features

### 1. Schema Evolution Modes

```python
# Different schema evolution strategies
options = {
    "cloudFiles.schemaEvolutionMode": "addNewColumns",  # Add new columns only
    # "cloudFiles.schemaEvolutionMode": "failOnNewColumns",  # Fail if new columns
    # "cloudFiles.schemaEvolutionMode": "rescue"  # Put unexpected data in _rescued_data
}
```

### 2. File Notification Setup

```python
# For AWS S3
df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .option("cloudFiles.useNotifications", "true") \
    .option("cloudFiles.includeExistingFiles", "false") \
    .load("s3://my-bucket/data/")
```

### 3. Schema Inference Control

```python
# Control how many files to sample for schema
df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "csv") \
    .option("cloudFiles.schemaHints", "id int, name string, age int") \
    .option("cloudFiles.inferColumnTypes", "true") \
    .option("cloudFiles.sampleSize", "1000") \
    .load(source_path)
```

---

## Best Practices

### 1. Organize Your Data Pipeline

```python
# Good folder structure
/mnt/data/
├── raw/           # Auto Loader source
├── bronze/        # Raw data in Delta format  
├── silver/        # Cleaned data
└── gold/          # Business-ready data

# Corresponding checkpoint structure  
/mnt/checkpoints/
├── bronze/
├── silver/
└── gold/
```

### 2. Handle Different File Formats

```python
def create_autoloader_stream(source_path, file_format, checkpoint_path):
    """Reusable Auto Loader function"""
    
    return spark.readStream \
        .format("cloudFiles") \
        .option("cloudFiles.format", file_format) \
        .option("cloudFiles.schemaLocation", f"{checkpoint_path}/schema") \
        .option("cloudFiles.schemaEvolutionMode", "addNewColumns") \
        .load(source_path)

# Use for different formats
csv_stream = create_autoloader_stream("/data/csv/", "csv", "/checkpoints/csv")
json_stream = create_autoloader_stream("/data/json/", "json", "/checkpoints/json")
```

### 3. Error Handling

```python
# Add metadata columns for debugging
df_with_metadata = df \
    .withColumn("input_file_name", input_file_name()) \
    .withColumn("processing_time", current_timestamp())

# Rescue bad data
df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .option("cloudFiles.schemaEvolutionMode", "rescue") \
    .load(source_path)

# Bad data goes to _rescued_data column
```

---

## Common Use Cases

### Use Case 1: Real-time Log Processing

```python
# Process web server logs as they arrive
log_stream = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "text") \
    .load("/logs/webserver/")

# Parse log lines
parsed_logs = log_stream \
    .select(
        regexp_extract(col("value"), r"(\d+\.\d+\.\d+\.\d+)", 1).alias("ip"),
        regexp_extract(col("value"), r"\[([^\]]+)\]", 1).alias("timestamp"),
        regexp_extract(col("value"), r'"([^"]+)"', 1).alias("request")
    )

# Write to table
parsed_logs.writeStream \
    .option("checkpointLocation", "/checkpoints/logs/") \
    .toTable("bronze.web_logs") \
    .start()
```

### Use Case 2: IoT Sensor Data

```python
# Process IoT sensor data (JSON)
sensor_stream = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .option("cloudFiles.schemaLocation", "/checkpoints/sensors/schema") \
    .load("/data/iot-sensors/")

# Calculate averages per hour
hourly_avg = sensor_stream \
    .withColumn("hour", date_trunc("hour", col("timestamp"))) \
    .groupBy("sensor_id", "hour") \
    .agg(avg("temperature").alias("avg_temp"),
         avg("humidity").alias("avg_humidity"))

# Write aggregated data
hourly_avg.writeStream \
    .option("checkpointLocation", "/checkpoints/sensors/") \
    .outputMode("complete") \
    .toTable("silver.sensor_hourly")
```

### Use Case 3: File Format Conversion

```python
# Convert CSV to Delta automatically
csv_to_delta = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "csv") \
    .option("header", "true") \
    .option("cloudFiles.schemaLocation", "/checkpoints/conversion/schema") \
    .load("/raw-csv/")

csv_to_delta.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/checkpoints/conversion/") \
    .option("mergeSchema", "true") \
    .toTable("bronze.converted_data") \
    .start()
```

---

## Monitoring Your Auto Loader

### Check Stream Status
```python
# Get active streams
active_streams = spark.streams.active
for stream in active_streams:
    print(f"Stream: {stream.name}, Status: {stream.status}")
```

### Monitor Progress
```python
# Stream metrics
query = df.writeStream.toTable("my_table").start()
print(query.lastProgress)  # See processing statistics
```

---

## Troubleshooting

### Common Issues

**Problem**: Stream stops unexpectedly
```python
# Solution: Check stream status and restart
if not query.isActive:
    print("Stream stopped. Check logs for errors.")
    query = df.writeStream.toTable("my_table").start()
```

**Problem**: Schema conflicts
```python
# Solution: Use rescue mode
df = spark.readStream \
    .option("cloudFiles.schemaEvolutionMode", "rescue") \
    .load(source_path)
```

**Problem**: Too many small files
```python
# Solution: Use trigger intervals
.trigger(processingTime="5 minutes")  # Batch files together
```

---

## Quick Start Checklist

- [ ] Set up cloud storage paths
- [ ] Choose file format (CSV, JSON, Parquet)
- [ ] Create checkpoint directory
- [ ] Set up Auto Loader stream
- [ ] Define target Delta table
- [ ] Start the stream
- [ ] Monitor progress

---

## Sample Complete Pipeline

```python
# Complete example: CSV to clean Delta table
def create_sales_pipeline():
    # 1. Source configuration
    source_path = "/mnt/raw/sales/"
    checkpoint_path = "/mnt/checkpoints/sales/"
    target_table = "analytics.clean_sales"
    
    # 2. Auto Loader stream
    raw_sales = spark.readStream \
        .format("cloudFiles") \
        .option("cloudFiles.format", "csv") \
        .option("cloudFiles.schemaLocation", f"{checkpoint_path}/schema") \
        .option("header", "true") \
        .load(source_path)
    
    # 3. Data cleaning
    clean_sales = raw_sales \
        .filter(col("amount").isNotNull()) \
        .filter(col("amount") > 0) \
        .withColumn("processed_date", current_date()) \
        .dropDuplicates(["transaction_id"])
    
    # 4. Write stream
    query = clean_sales.writeStream \
        .format("delta") \
        .option("checkpointLocation", checkpoint_path) \
        .option("mergeSchema", "true") \
        .trigger(processingTime="2 minutes") \
        .toTable(target_table)
    
    return query.start()

# Run the pipeline
sales_stream = create_sales_pipeline()
print("Sales pipeline started successfully!")
```

This pipeline will:
1. **Watch** for new CSV files in the raw folder
2. **Clean** the data by removing invalid records
3. **Store** processed data in a Delta table
4. **Run continuously** processing new files every 2 minutes

---

*Auto Loader makes incremental data processing simple and reliable. Start with basic examples and gradually add complexity as your needs grow!*
