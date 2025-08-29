# Multi-Hop Architectures in Databricks

## Table of Contents
1. [Introduction](#introduction)
2. [What is Multi-Hop Architecture?](#what-is-multi-hop-architecture)
3. [Core Components](#core-components)
4. [Architecture Layers](#architecture-layers)
5. [Benefits](#benefits)
6. [Implementation Approach](#implementation-approach)
7. [Simple Example: E-commerce Data Pipeline](#simple-example-e-commerce-data-pipeline)
8. [Code Implementation](#code-implementation)
9. [Best Practices](#best-practices)
10. [Common Use Cases](#common-use-cases)
11. [Troubleshooting](#troubleshooting)

---

## Introduction

Multi-Hop Architecture in Databricks is a data engineering pattern that implements the **Medallion Architecture** (Bronze-Silver-Gold layers) for processing and refining data through multiple stages. Each "hop" represents a transformation stage that progressively cleanses, enriches, and aggregates data.

## What is Multi-Hop Architecture?

Multi-Hop Architecture is a data lakehouse design pattern that:
- Processes data through multiple sequential layers
- Each layer serves a specific purpose in the data refinement process
- Enables incremental data processing and transformation
- Supports both batch and streaming data workloads
- Provides data lineage and quality controls at each stage

### Key Characteristics:
- **Incremental Processing**: Only processes new or changed data
- **Data Quality Gates**: Validation at each hop
- **Scalability**: Handles large volumes efficiently
- **Flexibility**: Supports various data sources and formats

---

## Core Components

### 1. Delta Lake Tables
- **Storage Format**: Parquet files with transaction logs
- **ACID Transactions**: Ensures data consistency
- **Time Travel**: Access historical versions
- **Schema Evolution**: Automatic schema updates

### 2. Delta Live Tables (DLT)
- **Declarative ETL**: Define transformations, not orchestration
- **Automatic Dependency Management**: Handles execution order
- **Data Quality Constraints**: Built-in validation rules
- **Real-time Monitoring**: Pipeline health dashboards

### 3. Structured Streaming
- **Continuous Processing**: Real-time data ingestion
- **Fault Tolerance**: Automatic recovery from failures
- **Exactly-Once Processing**: No duplicate data
- **Watermarking**: Handle late-arriving data

---

## Architecture Layers

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   BRONZE LAYER  │───▶│  SILVER LAYER   │───▶│   GOLD LAYER    │
│   (Raw Data)    │    │ (Cleaned Data)  │    │ (Business Data) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Bronze Layer (Raw Data Ingestion)
- **Purpose**: Ingest raw data from various sources
- **Data Format**: Original format (JSON, CSV, Parquet, etc.)
- **Processing**: Minimal transformation, add metadata
- **Quality**: No data quality enforcement
- **Schema**: Schema-on-read approach

### Silver Layer (Cleaned and Validated)
- **Purpose**: Clean, validate, and standardize data
- **Data Format**: Structured format (Delta tables)
- **Processing**: Data cleansing, deduplication, type conversion
- **Quality**: Data quality rules enforced
- **Schema**: Well-defined schema

### Gold Layer (Business-Ready)
- **Purpose**: Aggregated, business-ready datasets
- **Data Format**: Optimized for analytics (Delta tables)
- **Processing**: Aggregations, joins, business logic
- **Quality**: High-quality, trusted data
- **Schema**: Business-oriented schema

---

## Benefits

### 1. **Data Quality Improvement**
- Progressive data refinement
- Quality gates at each layer
- Error isolation and handling

### 2. **Performance Optimization**
- Incremental processing
- Optimized storage formats
- Efficient query performance

### 3. **Scalability**
- Handles growing data volumes
- Parallel processing capabilities
- Auto-scaling clusters

### 4. **Maintainability**
- Clear separation of concerns
- Easier debugging and monitoring
- Modular pipeline design

### 5. **Cost Efficiency**
- Process only changed data
- Optimized resource usage
- Reduced storage costs

---

## Implementation Approach

### Step 1: Design Data Flow
1. Identify data sources
2. Define transformation requirements
3. Design layer schemas
4. Plan data quality rules

### Step 2: Set Up Infrastructure
1. Create Databricks workspace
2. Configure storage (ADLS, S3, etc.)
3. Set up compute clusters
4. Configure security and access controls

### Step 3: Implement Layers
1. Bronze layer ingestion
2. Silver layer transformations
3. Gold layer aggregations
4. Data quality monitoring

### Step 4: Deploy and Monitor
1. Schedule pipeline execution
2. Set up monitoring and alerting
3. Implement data quality dashboards
4. Configure error handling

---

## Simple Example: E-commerce Data Pipeline

Let's build a simple e-commerce analytics pipeline that processes customer orders through Bronze, Silver, and Gold layers.

### Business Scenario
- **Source**: Customer orders from an e-commerce platform
- **Goal**: Create daily sales analytics for business users
- **Data Flow**: Raw orders → Clean orders → Daily sales summary

### Data Sources
- Orders API (JSON format)
- Customer database (CSV files)
- Product catalog (Parquet files)

---

## Code Implementation

### Bronze Layer: Raw Data Ingestion

```python
# Bronze Layer - Ingest raw order data
import dlt
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

@dlt.table(
    name="bronze_orders",
    comment="Raw order data from e-commerce API"
)
def bronze_orders():
    return (
        spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .option("cloudFiles.schemaLocation", "/mnt/schema/orders")
        .load("/mnt/raw-data/orders/")
        .withColumn("ingestion_timestamp", current_timestamp())
        .withColumn("source_file", input_file_name())
    )

@dlt.table(
    name="bronze_customers",
    comment="Raw customer data"
)
def bronze_customers():
    return (
        spark.read
        .format("csv")
        .option("header", "true")
        .option("inferSchema", "true")
        .load("/mnt/raw-data/customers/")
        .withColumn("ingestion_timestamp", current_timestamp())
    )
```

### Silver Layer: Data Cleaning and Validation

```python
# Silver Layer - Clean and validate data
@dlt.table(
    name="silver_orders",
    comment="Cleaned and validated order data"
)
@dlt.expect_or_fail("valid_order_amount", "order_amount > 0")
@dlt.expect_or_fail("valid_customer_id", "customer_id IS NOT NULL")
@dlt.expect("valid_order_date", "order_date IS NOT NULL")
def silver_orders():
    return (
        dlt.read("bronze_orders")
        .select(
            col("order_id").cast("string"),
            col("customer_id").cast("string"),
            col("order_date").cast("timestamp"),
            col("order_amount").cast("decimal(10,2)"),
            col("product_id").cast("string"),
            col("quantity").cast("integer"),
            col("ingestion_timestamp")
        )
        .filter(col("order_amount") > 0)  # Remove invalid orders
        .dropDuplicates(["order_id"])     # Remove duplicates
        .withColumn("order_year", year("order_date"))
        .withColumn("order_month", month("order_date"))
        .withColumn("order_day", dayofmonth("order_date"))
    )

@dlt.table(
    name="silver_customers",
    comment="Cleaned customer data"
)
@dlt.expect_or_fail("valid_email", "email RLIKE '^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$'")
def silver_customers():
    return (
        dlt.read("bronze_customers")
        .select(
            col("customer_id").cast("string"),
            col("first_name").cast("string"),
            col("last_name").cast("string"),
            col("email").cast("string"),
            col("registration_date").cast("timestamp"),
            col("country").cast("string")
        )
        .filter(col("customer_id").isNotNull())
        .dropDuplicates(["customer_id"])
    )
```

### Gold Layer: Business Aggregations

```python
# Gold Layer - Business-ready analytics
@dlt.table(
    name="gold_daily_sales",
    comment="Daily sales summary for business analytics"
)
def gold_daily_sales():
    orders = dlt.read("silver_orders")
    customers = dlt.read("silver_customers")
    
    return (
        orders.join(customers, "customer_id", "inner")
        .groupBy(
            "order_year",
            "order_month", 
            "order_day",
            "country"
        )
        .agg(
            count("order_id").alias("total_orders"),
            sum("order_amount").alias("total_revenue"),
            countDistinct("customer_id").alias("unique_customers"),
            avg("order_amount").alias("avg_order_value"),
            sum("quantity").alias("total_items_sold")
        )
        .withColumn("sales_date", 
                   to_date(concat_ws("-", "order_year", "order_month", "order_day")))
    )

@dlt.table(
    name="gold_customer_metrics",
    comment="Customer lifetime value and metrics"
)
def gold_customer_metrics():
    orders = dlt.read("silver_orders")
    customers = dlt.read("silver_customers")
    
    return (
        orders.join(customers, "customer_id", "inner")
        .groupBy("customer_id", "first_name", "last_name", "country")
        .agg(
            count("order_id").alias("total_orders"),
            sum("order_amount").alias("lifetime_value"),
            max("order_date").alias("last_order_date"),
            min("order_date").alias("first_order_date"),
            avg("order_amount").alias("avg_order_value")
        )
        .withColumn("customer_tenure_days", 
                   datediff("last_order_date", "first_order_date"))
    )
```

### Pipeline Configuration

```python
# Pipeline configuration file (pipeline.json)
{
    "id": "ecommerce-multihop-pipeline",
    "name": "E-commerce Multi-Hop Pipeline",
    "storage": "/mnt/delta-lake/ecommerce",
    "configuration": {
        "spark.sql.adaptive.enabled": "true",
        "spark.sql.adaptive.coalescePartitions.enabled": "true"
    },
    "clusters": [
        {
            "label": "default",
            "autoscale": {
                "min_workers": 1,
                "max_workers": 5
            }
        }
    ],
    "libraries": [
        {
            "pypi": {
                "package": "delta-spark==2.4.0"
            }
        }
    ],
    "target": "prod"
}
```

---

## Data Flow Example

### Sample Input Data

**Raw Order Data (Bronze)**
```json
{
    "order_id": "ORD-2024-001",
    "customer_id": "CUST-12345",
    "order_date": "2024-08-29T10:30:00Z",
    "order_amount": 149.99,
    "product_id": "PROD-ABC-123",
    "quantity": 2,
    "status": "completed"
}
```

**Processed Data (Silver)**
```
order_id        | customer_id | order_date           | order_amount | product_id   | quantity | order_year | order_month | order_day
ORD-2024-001   | CUST-12345  | 2024-08-29 10:30:00 | 149.99       | PROD-ABC-123 | 2        | 2024       | 8           | 29
```

**Business Metrics (Gold)**
```
sales_date | country | total_orders | total_revenue | unique_customers | avg_order_value | total_items_sold
2024-08-29 | USA     | 1250         | 187,485.75    | 892              | 149.99          | 3,240
```

---

## Best Practices

### 1. **Data Quality Management**
```python
# Implement comprehensive data quality checks
@dlt.expect_or_fail("positive_amount", "amount > 0")
@dlt.expect_or_drop("valid_date", "order_date IS NOT NULL")
@dlt.expect("data_freshness", "ingestion_timestamp > current_timestamp() - interval 1 day")
```

### 2. **Error Handling**
```python
# Handle bad records gracefully
def handle_bad_records(df):
    return (
        df.withColumn("is_valid", 
                     when(col("order_amount") <= 0, False)
                     .when(col("customer_id").isNull(), False)
                     .otherwise(True))
    )
```

### 3. **Performance Optimization**
- Use appropriate partitioning strategies
- Implement Z-ordering for better query performance
- Configure auto-optimize for Delta tables
- Use appropriate cluster sizing

### 4. **Monitoring and Alerting**
- Set up data quality metrics dashboards
- Configure pipeline failure alerts
- Monitor data freshness and volume
- Track processing latencies

---

## Common Use Cases

### 1. **Real-time Analytics**
- Streaming data ingestion
- Real-time dashboard updates
- Alert generation

### 2. **Data Science Workflows**
- Feature engineering pipelines
- Model training datasets
- Experiment tracking

### 3. **Business Intelligence**
- Executive dashboards
- Operational reporting
- Customer analytics

### 4. **Data Governance**
- Data lineage tracking
- Compliance reporting
- Data quality monitoring

---

## Pipeline Deployment Commands

### Using Databricks CLI
```bash
# Install Databricks CLI
pip install databricks-cli

# Configure authentication
databricks configure --token

# Deploy pipeline
databricks pipelines create --settings pipeline.json

# Start pipeline
databricks pipelines start --pipeline-id <pipeline-id>

# Monitor pipeline
databricks pipelines get --pipeline-id <pipeline-id>
```

### Using REST API
```python
import requests

# Create pipeline
def create_pipeline():
    headers = {
        'Authorization': f'Bearer {databricks_token}',
        'Content-Type': 'application/json'
    }
    
    pipeline_config = {
        "name": "ecommerce-multihop-pipeline",
        "storage": "/mnt/delta-lake/ecommerce",
        "clusters": [
            {
                "autoscale": {
                    "min_workers": 1,
                    "max_workers": 5
                }
            }
        ]
    }
    
    response = requests.post(
        f"{databricks_host}/api/2.0/pipelines",
        headers=headers,
        json=pipeline_config
    )
    
    return response.json()
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. **Schema Evolution Errors**
```python
# Solution: Enable schema evolution
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.databricks.delta.schema.autoMerge.enabled", "true")
```

#### 2. **Performance Issues**
```python
# Solution: Optimize table properties
ALTER TABLE silver_orders 
SET TBLPROPERTIES (
    'delta.autoOptimize.optimizeWrite' = 'true',
    'delta.autoOptimize.autoCompact' = 'true'
);

OPTIMIZE silver_orders ZORDER BY (customer_id, order_date);
```

#### 3. **Data Quality Failures**
```python
# Solution: Implement graceful degradation
@dlt.expect_or_drop("valid_amount", "order_amount > 0")
def silver_orders_with_quality():
    return (
        dlt.read("bronze_orders")
        .withColumn("data_quality_flag", 
                   when(col("order_amount") <= 0, "INVALID_AMOUNT")
                   .otherwise("VALID"))
    )
```

---

## Monitoring Dashboard Query Examples

### Pipeline Health Metrics
```sql
-- Check data quality metrics
SELECT 
    table_name,
    expectation_name,
    passed_records,
    failed_records,
    failed_records / (passed_records + failed_records) * 100 as failure_rate
FROM system.dlt.expectations 
WHERE pipeline_id = '<your-pipeline-id>'
ORDER BY failure_rate DESC;
```

### Data Freshness Check
```sql
-- Monitor data freshness
SELECT 
    table_name,
    MAX(ingestion_timestamp) as latest_data,
    current_timestamp() - MAX(ingestion_timestamp) as data_age_hours
FROM silver_orders 
GROUP BY table_name;
```

---

## Advanced Features

### 1. **Change Data Capture (CDC)**
```python
@dlt.table(name="silver_orders_cdc")
def orders_with_cdc():
    return (
        dlt.read_stream("bronze_orders")
        .withColumn("operation_type", lit("INSERT"))
        .withColumn("processed_timestamp", current_timestamp())
    )
```

### 2. **Slowly Changing Dimensions (SCD)**
```python
# Type 2 SCD implementation
@dlt.table(name="gold_customer_scd")
def customer_scd():
    return (
        dlt.read("silver_customers")
        .withColumn("effective_start_date", current_date())
        .withColumn("effective_end_date", lit(None).cast("date"))
        .withColumn("is_current", lit(True))
    )
```

### 3. **Data Lineage Tracking**
```python
# Add lineage information
@dlt.table(name="silver_orders_with_lineage")
def orders_with_lineage():
    return (
        dlt.read("bronze_orders")
        .withColumn("source_table", lit("bronze_orders"))
        .withColumn("transformation_timestamp", current_timestamp())
        .withColumn("pipeline_run_id", lit(spark.conf.get("spark.sql.execution.id")))
    )
```

---

## Conclusion

Multi-Hop Architecture in Databricks provides a robust framework for building scalable, maintainable data pipelines. By implementing the Bronze-Silver-Gold pattern, organizations can:

- Ensure data quality through progressive refinement
- Scale processing to handle growing data volumes
- Maintain clear data lineage and governance
- Enable real-time analytics and machine learning workflows

The key to success is careful planning of each layer's purpose, implementing appropriate data quality controls, and following best practices for performance optimization.

---

## Additional Resources

- [Databricks Delta Live Tables Documentation](https://docs.databricks.com/delta-live-tables/)
- [Medallion Architecture Guide](https://databricks.com/glossary/medallion-architecture)
- [Delta Lake Documentation](https://docs.delta.io/)
- [Structured Streaming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
