# Task Orchestration with Databricks Jobs

## Table of Contents
- [What is Task Orchestration?](#what-is-task-orchestration)
- [Databricks Jobs Overview](#databricks-jobs-overview)
- [Key Components](#key-components)
- [Types of Tasks](#types-of-tasks)
- [Creating Your First Job](#creating-your-first-job)
- [Simple Example: Data Pipeline](#simple-example-data-pipeline)
- [Advanced Orchestration](#advanced-orchestration)
- [Job Configuration](#job-configuration)
- [Monitoring and Alerts](#monitoring-and-alerts)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## What is Task Orchestration?

Task orchestration is like being a conductor of an orchestra. Instead of musicians, you're coordinating different data tasks (notebooks, scripts, models) to work together in the right order and timing.

**Why do we need it?**
- Some tasks must run before others (dependencies)
- Tasks might fail and need to retry
- Different tasks may need different resources
- We want to schedule when things run automatically

## Databricks Jobs Overview

Databricks Jobs is a service that helps you:
- **Schedule** tasks to run automatically
- **Coordinate** multiple tasks in the right order
- **Monitor** task execution and handle failures
- **Scale** resources based on workload needs

Think of it as your personal assistant that runs your data work while you sleep!

## Key Components

### 1. **Job**
The main container that holds all your tasks and settings.

### 2. **Tasks**
Individual pieces of work (like running a notebook or script).

### 3. **Clusters**
Computing resources where your tasks actually run.

### 4. **Triggers**
Rules that decide when your job should start running.

### 5. **Dependencies**
Rules about which tasks must finish before others can start.

## Types of Tasks

### 1. **Notebook Task**
Runs a Databricks notebook
```json
{
  "task_key": "data_processing",
  "notebook_task": {
    "notebook_path": "/Workspace/data_pipeline/process_data",
    "base_parameters": {
      "input_date": "2024-01-15"
    }
  }
}
```

### 2. **Python Script Task**
Runs a Python file
```json
{
  "task_key": "python_analysis",
  "python_wheel_task": {
    "entry_point": "main",
    "package_name": "my_analysis_package"
  }
}
```

### 3. **SQL Task**
Executes SQL queries
```json
{
  "task_key": "sql_aggregation",
  "sql_task": {
    "query": {
      "query_id": "abc123-def456"
    }
  }
}
```

### 4. **Delta Live Tables Task**
Runs DLT pipelines
```json
{
  "task_key": "dlt_pipeline",
  "pipeline_task": {
    "pipeline_id": "xyz789-abc123"
  }
}
```

### 5. **MLflow Model Task**
Deploys or runs ML models
```json
{
  "task_key": "model_inference",
  "run_job_task": {
    "job_id": 12345
  }
}
```

## Creating Your First Job

### Step 1: Access Jobs UI
1. Open Databricks workspace
2. Click "Workflows" in sidebar
3. Select "Jobs"
4. Click "Create Job"

### Step 2: Basic Configuration
- **Job Name**: Give your job a clear name
- **Description**: Explain what the job does
- **Tags**: Add labels for organization

### Step 3: Add Tasks
- Click "Add Task"
- Choose task type
- Configure task settings
- Set up dependencies if needed

### Step 4: Configure Triggers
- **Manual**: Run when you click "Run Now"
- **Scheduled**: Run at specific times
- **File Arrival**: Run when new files appear
- **Continuous**: Keep running continuously

## Simple Example: Data Pipeline

Let's create a simple data processing pipeline that:
1. Downloads raw sales data
2. Cleans and validates the data
3. Creates daily summary reports
4. Sends notification when complete

### Example Job Configuration

```python
# Task 1: Data Ingestion Notebook
# File: /Workspace/pipelines/01_ingest_data

# Read raw data from source
df = spark.read.format("csv") \
    .option("header", "true") \
    .load("/mnt/raw-data/sales/")

# Save to bronze layer
df.write.format("delta") \
    .mode("overwrite") \
    .save("/mnt/bronze/sales_raw")

print("Data ingestion completed!")
```

```python
# Task 2: Data Cleaning Notebook  
# File: /Workspace/pipelines/02_clean_data

# Read bronze data
df = spark.read.format("delta").load("/mnt/bronze/sales_raw")

# Clean and validate
cleaned_df = df \
    .filter(col("amount") > 0) \
    .filter(col("date").isNotNull()) \
    .withColumn("amount", col("amount").cast("decimal(10,2)")) \
    .dropDuplicates(["transaction_id"])

# Save to silver layer
cleaned_df.write.format("delta") \
    .mode("overwrite") \
    .save("/mnt/silver/sales_clean")

print("Data cleaning completed!")
```

```python
# Task 3: Create Summary Notebook
# File: /Workspace/pipelines/03_create_summary

# Read cleaned data
df = spark.read.format("delta").load("/mnt/silver/sales_clean")

# Create daily summary
summary = df \
    .groupBy("date") \
    .agg(
        sum("amount").alias("total_sales"),
        count("transaction_id").alias("total_transactions"),
        avg("amount").alias("avg_transaction")
    ) \
    .orderBy("date")

# Save to gold layer
summary.write.format("delta") \
    .mode("overwrite") \
    .save("/mnt/gold/daily_sales_summary")

print("Summary creation completed!")
```

### Job Definition (JSON/API)

```json
{
  "name": "Daily Sales Processing Pipeline",
  "description": "Process daily sales data and create summaries",
  "tasks": [
    {
      "task_key": "ingest_data",
      "description": "Download and store raw sales data",
      "notebook_task": {
        "notebook_path": "/Workspace/pipelines/01_ingest_data"
      },
      "new_cluster": {
        "spark_version": "12.2.x-scala2.12",
        "node_type_id": "i3.xlarge",
        "num_workers": 2
      }
    },
    {
      "task_key": "clean_data", 
      "description": "Clean and validate sales data",
      "depends_on": [{"task_key": "ingest_data"}],
      "notebook_task": {
        "notebook_path": "/Workspace/pipelines/02_clean_data"
      },
      "new_cluster": {
        "spark_version": "12.2.x-scala2.12", 
        "node_type_id": "i3.xlarge",
        "num_workers": 2
      }
    },
    {
      "task_key": "create_summary",
      "description": "Generate daily sales summary",
      "depends_on": [{"task_key": "clean_data"}],
      "notebook_task": {
        "notebook_path": "/Workspace/pipelines/03_create_summary"
      },
      "new_cluster": {
        "spark_version": "12.2.x-scala2.12",
        "node_type_id": "i3.xlarge", 
        "num_workers": 1
      }
    }
  ],
  "schedule": {
    "quartz_cron_expression": "0 0 2 * * ?",
    "timezone_id": "UTC"
  },
  "email_notifications": {
    "on_success": ["team@company.com"],
    "on_failure": ["alerts@company.com"]
  }
}
```

### Workflow Visualization
```
[Raw Data Files] 
       ↓
[Task 1: Ingest Data] → Bronze Layer
       ↓
[Task 2: Clean Data] → Silver Layer  
       ↓
[Task 3: Create Summary] → Gold Layer
       ↓
[Email Notification]
```

## Advanced Orchestration

### 1. **Conditional Logic**
```python
# Task that runs only if previous task found new data
{
  "task_key": "process_if_new_data",
  "condition_task": {
    "op": "EQUAL_TO",
    "left": "{{tasks.check_data.values.has_new_data}}",
    "right": "true"
  }
}
```

### 2. **Parallel Execution**
```json
{
  "tasks": [
    {
      "task_key": "process_sales",
      "depends_on": [{"task_key": "ingest"}]
    },
    {
      "task_key": "process_inventory", 
      "depends_on": [{"task_key": "ingest"}]
    },
    {
      "task_key": "create_report",
      "depends_on": [
        {"task_key": "process_sales"},
        {"task_key": "process_inventory"}
      ]
    }
  ]
}
```

### 3. **Dynamic Parameters**
```python
# Pass runtime parameters between tasks
dbutils.jobs.taskValues.set("processed_records", record_count)

# Use in next task
record_count = dbutils.jobs.taskValues.get("previous_task", "processed_records")
```

### 4. **Error Handling**
```json
{
  "task_key": "resilient_task",
  "max_retries": 3,
  "min_retry_interval_millis": 10000,
  "retry_on_timeout": true,
  "timeout_seconds": 3600
}
```

## Job Configuration

### Cluster Configuration
```json
{
  "new_cluster": {
    "spark_version": "12.2.x-scala2.12",
    "node_type_id": "i3.xlarge",
    "num_workers": 2,
    "spark_conf": {
      "spark.databricks.delta.preview.enabled": "true"
    },
    "init_scripts": [
      {
        "dbfs": {
          "destination": "dbfs:/databricks/scripts/setup.sh"
        }
      }
    ]
  }
}
```

### Schedule Examples
```json
{
  "schedule": {
    "quartz_cron_expression": "0 0 8 * * ?",    // Daily at 8 AM
    "timezone_id": "America/New_York"
  }
}
```

Common Cron Patterns:
- `0 0 8 * * ?` - Daily at 8 AM
- `0 0 8 * * MON` - Every Monday at 8 AM  
- `0 0 8 1 * ?` - First day of every month at 8 AM
- `0 */15 * * * ?` - Every 15 minutes

## Monitoring and Alerts

### 1. **Job Monitoring**
- View run history and status
- Check task execution times
- Monitor resource usage
- Track success/failure rates

### 2. **Alert Configuration**
```json
{
  "email_notifications": {
    "on_start": ["team@company.com"],
    "on_success": ["reports@company.com"], 
    "on_failure": ["alerts@company.com"],
    "on_duration_warning_threshold_exceeded": ["performance@company.com"]
  },
  "webhook_notifications": {
    "on_failure": [
      {
        "id": "slack-webhook-id"
      }
    ]
  }
}
```

### 3. **Custom Metrics**
```python
# In your notebook task
import time
start_time = time.time()

# Your data processing logic here
process_data()

# Track execution time
execution_time = time.time() - start_time
dbutils.jobs.taskValues.set("execution_time_seconds", execution_time)

# Log custom metrics
print(f"Processed {record_count} records in {execution_time:.2f} seconds")
```

## Best Practices

### 🎯 Design Principles

1. **Keep Tasks Focused**
   - One clear responsibility per task
   - Avoid overly complex tasks
   - Make tasks reusable where possible

2. **Handle Failures Gracefully**
   - Use appropriate retry policies
   - Implement proper error logging
   - Have fallback procedures

3. **Optimize Resource Usage**
   - Right-size your clusters
   - Use job clusters for cost efficiency
   - Consider auto-scaling options

4. **Maintain Clear Documentation**
   - Document what each task does
   - Explain dependencies
   - Keep runbooks updated

### ⚡ Performance Tips

1. **Cluster Management**
   - Use job clusters for production
   - Share clusters for development
   - Consider serverless for simple tasks

2. **Task Dependencies**
   - Minimize unnecessary dependencies
   - Use parallel execution when possible
   - Avoid circular dependencies

3. **Resource Optimization**
   - Monitor cluster utilization
   - Adjust cluster size based on data volume
   - Use spot instances for cost savings

### 🔧 Development Workflow

1. **Development Phase**
   - Test individual notebooks first
   - Use small data samples
   - Debug in interactive mode

2. **Testing Phase**
   - Create test job with dev clusters
   - Use realistic data volumes
   - Test error scenarios

3. **Production Phase**
   - Use production clusters
   - Set up proper monitoring
   - Configure appropriate alerts

## Real-World Example: E-commerce Analytics Pipeline

### Scenario
An e-commerce company needs to process daily data to update their analytics dashboard.

### Pipeline Flow
```
Daily Files → Data Validation → Customer Analysis → Product Analysis → Dashboard Update → Reports
```

### Job Configuration

```json
{
  "name": "E-commerce Daily Analytics Pipeline",
  "description": "Process daily e-commerce data for analytics dashboard",
  "tasks": [
    {
      "task_key": "validate_data",
      "description": "Validate incoming data files",
      "notebook_task": {
        "notebook_path": "/Workspace/ecommerce/01_validate_data",
        "base_parameters": {
          "data_date": "{{job.start_time.date}}"
        }
      },
      "job_cluster_key": "small_cluster"
    },
    {
      "task_key": "process_customers",
      "description": "Analyze customer behavior",
      "depends_on": [{"task_key": "validate_data"}],
      "notebook_task": {
        "notebook_path": "/Workspace/ecommerce/02_customer_analysis"
      },
      "job_cluster_key": "medium_cluster"
    },
    {
      "task_key": "process_products", 
      "description": "Analyze product performance",
      "depends_on": [{"task_key": "validate_data"}],
      "notebook_task": {
        "notebook_path": "/Workspace/ecommerce/03_product_analysis"
      },
      "job_cluster_key": "medium_cluster"
    },
    {
      "task_key": "update_dashboard",
      "description": "Update analytics dashboard",
      "depends_on": [
        {"task_key": "process_customers"},
        {"task_key": "process_products"}
      ],
      "notebook_task": {
        "notebook_path": "/Workspace/ecommerce/04_update_dashboard"
      },
      "job_cluster_key": "small_cluster"
    },
    {
      "task_key": "generate_reports",
      "description": "Generate and email daily reports", 
      "depends_on": [{"task_key": "update_dashboard"}],
      "notebook_task": {
        "notebook_path": "/Workspace/ecommerce/05_generate_reports"
      },
      "job_cluster_key": "small_cluster"
    }
  ],
  "job_clusters": [
    {
      "job_cluster_key": "small_cluster",
      "new_cluster": {
        "spark_version": "12.2.x-scala2.12",
        "node_type_id": "i3.large", 
        "num_workers": 1
      }
    },
    {
      "job_cluster_key": "medium_cluster",
      "new_cluster": {
        "spark_version": "12.2.x-scala2.12",
        "node_type_id": "i3.xlarge",
        "num_workers": 3
      }
    }
  ],
  "schedule": {
    "quartz_cron_expression": "0 0 6 * * ?",
    "timezone_id": "UTC"
  },
  "max_concurrent_runs": 1,
  "timeout_seconds": 7200
}
```

### Sample Task Implementation

```python
# Notebook: /Workspace/ecommerce/02_customer_analysis
# Task: process_customers

from pyspark.sql.functions import *
from datetime import datetime

# Get parameters
data_date = dbutils.widgets.get("data_date")
print(f"Processing customer data for {data_date}")

try:
    # Read validated data
    orders_df = spark.read.format("delta") \
        .load("/mnt/silver/orders") \
        .filter(col("order_date") == data_date)
    
    customers_df = spark.read.format("delta") \
        .load("/mnt/silver/customers")
    
    # Customer analysis
    customer_metrics = orders_df \
        .join(customers_df, "customer_id") \
        .groupBy("customer_id", "customer_segment") \
        .agg(
            sum("order_amount").alias("total_spent"),
            count("order_id").alias("total_orders"),
            avg("order_amount").alias("avg_order_value")
        )
    
    # Save results
    customer_metrics.write.format("delta") \
        .mode("append") \
        .save("/mnt/gold/customer_daily_metrics")
    
    # Set task values for next task
    total_customers = customer_metrics.count()
    dbutils.jobs.taskValues.set("customers_processed", total_customers)
    
    print(f"Successfully processed {total_customers} customers")
    
except Exception as e:
    print(f"Error in customer processing: {str(e)}")
    raise e
```

## Advanced Orchestration

### 1. **Multi-Environment Setup**
```json
{
  "name": "Multi-Env Data Pipeline",
  "tasks": [
    {
      "task_key": "dev_processing",
      "condition_task": {
        "op": "EQUAL_TO",
        "left": "{{job.parameters.environment}}",
        "right": "dev"
      },
      "notebook_task": {
        "notebook_path": "/Workspace/dev/process_data"
      }
    },
    {
      "task_key": "prod_processing", 
      "condition_task": {
        "op": "EQUAL_TO",
        "left": "{{job.parameters.environment}}", 
        "right": "prod"
      },
      "notebook_task": {
        "notebook_path": "/Workspace/prod/process_data"
      }
    }
  ]
}
```

### 2. **Dynamic Task Generation**
```python
# Create tasks programmatically using Databricks API
from databricks.sdk import WorkspaceClient

w = WorkspaceClient()

# Define base task template
base_task = {
    "notebook_task": {
        "notebook_path": "/Workspace/processing/process_region"
    },
    "new_cluster": {
        "spark_version": "12.2.x-scala2.12",
        "node_type_id": "i3.xlarge",
        "num_workers": 2
    }
}

# Create tasks for each region
regions = ["us-east", "us-west", "europe", "asia"]
tasks = []

for region in regions:
    task = base_task.copy()
    task["task_key"] = f"process_{region}"
    task["notebook_task"]["base_parameters"] = {"region": region}
    tasks.append(task)

# Create the job
job_spec = {
    "name": "Multi-Region Processing",
    "tasks": tasks
}

job = w.jobs.create(**job_spec)
```

### 3. **Integration with External Systems**
```python
# Task that calls external API
import requests

# Call external system
response = requests.post("https://api.external-system.com/trigger", 
                        json={"pipeline_id": "daily_sales"})

if response.status_code == 200:
    dbutils.jobs.taskValues.set("external_job_id", response.json()["job_id"])
    print("External system triggered successfully")
else:
    raise Exception(f"Failed to trigger external system: {response.text}")
```

## Monitoring and Debugging

### 1. **Job Run Monitoring**
```python
# In your notebook tasks - add logging
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def process_data():
    logger.info("Starting data processing")
    
    # Your processing logic
    record_count = df.count()
    
    logger.info(f"Processed {record_count} records")
    dbutils.jobs.taskValues.set("record_count", record_count)
    
    return record_count
```

### 2. **Custom Metrics Tracking**
```python
# Track custom business metrics
from datetime import datetime

metrics = {
    "pipeline_name": "sales_processing",
    "run_date": datetime.now().isoformat(),
    "records_processed": record_count,
    "processing_time_minutes": execution_time / 60,
    "data_quality_score": quality_score
}

# Log metrics (could send to external monitoring system)
print(f"METRICS: {json.dumps(metrics)}")
```

### 3. **Health Checks**
```python
# Add health checks in your tasks
def health_check():
    # Check if data is recent
    latest_data = spark.sql("SELECT MAX(update_date) FROM my_table").collect()[0][0]
    hours_old = (datetime.now() - latest_data).total_seconds() / 3600
    
    if hours_old > 25:  # Data should be updated daily
        raise Exception(f"Data is {hours_old:.1f} hours old - too stale!")
    
    # Check record counts
    current_count = spark.sql("SELECT COUNT(*) FROM my_table").collect()[0][0]
    if current_count < 1000:  # Expect at least 1000 records
        raise Exception(f"Only {current_count} records found - too few!")
    
    print("Health check passed!")

health_check()
```

## Common Patterns

### 1. **Data Lake ETL Pattern**
```
Raw Data → Bronze (Raw) → Silver (Clean) → Gold (Aggregated) → Analytics
```

### 2. **ML Pipeline Pattern**
```
Data Prep → Feature Engineering → Model Training → Model Validation → Model Deployment
```

### 3. **Real-time + Batch Pattern**
```
Streaming Data → Real-time Processing → Batch Aggregation → Reports
```

## Troubleshooting Guide

### Common Issues and Solutions

#### ❌ **Job Fails to Start**
- Check cluster permissions
- Verify notebook paths exist
- Ensure parameters are valid
- Check cluster configuration

#### ❌ **Tasks Run Out of Order**
- Review dependency configuration
- Check for circular dependencies
- Verify task keys are unique

#### ❌ **Performance Issues**
- Monitor cluster utilization
- Check for data skew
- Optimize Spark configurations
- Consider larger clusters

#### ❌ **Intermittent Failures**
- Increase retry attempts
- Add longer timeout values
- Check external system availability
- Implement better error handling

### Debugging Tips

1. **Start Small**: Test with sample data first
2. **Check Logs**: Always review task logs for errors
3. **Use Job Parameters**: Make jobs flexible with parameters
4. **Test Dependencies**: Verify each task works independently
5. **Monitor Resources**: Watch CPU, memory, and disk usage

## Quick Start Checklist

- [ ] Create simple single-task job
- [ ] Add scheduling trigger
- [ ] Test job execution
- [ ] Add second dependent task
- [ ] Configure error handling
- [ ] Set up monitoring alerts
- [ ] Add documentation
- [ ] Deploy to production

## Pro Tips

1. **Use meaningful task names** - makes debugging easier
2. **Group related tasks** - organize by business function
3. **Version your notebooks** - track changes over time
4. **Test thoroughly** - especially dependency chains
5. **Monitor costs** - jobs can consume resources quickly
6. **Plan for failures** - they will happen!

---

**Remember**: Start simple with basic task chains, then gradually add more complex orchestration as you become comfortable with the platform. Databricks Jobs handles the hard infrastructure parts, so you can focus on your business logic!
