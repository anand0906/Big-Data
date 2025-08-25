# Databricks Jobs - Complete Guide

## 📋 Table of Contents
- [Overview](#overview)
- [dbutils.jobs - Job Utilities](#dbutilsjobs---job-utilities)
- [Job Creation Methods](#job-creation-methods)
- [Job Configuration](#job-configuration)
- [Job Scheduling](#job-scheduling)
- [Job Monitoring & Management](#job-monitoring--management)
- [Job Parameters & Dependencies](#job-parameters--dependencies)
- [Error Handling & Retry Logic](#error-handling--retry-logic)
- [Best Practices](#best-practices)
- [Quick Reference](#quick-reference)

---

## 🎯 Overview

**Databricks Jobs** allow you to run notebooks, JARs, Python scripts, and SQL queries as automated workflows. Jobs can be scheduled, parameterized, monitored, and chained together to create complex data pipelines.

### Key Concepts
- **Job** = A runnable workflow with one or more tasks
- **Task** = Individual unit of work (notebook, script, etc.)
- **Run** = Single execution instance of a job
- **Cluster** = Compute resources where job runs
- **Schedule** = Timing configuration for automatic execution

---

## ⚙️ dbutils.jobs - Job Utilities

### Getting Help
```python
dbutils.jobs.help()
```

**Output:**
```
The jobs module.
taskKey(key: String): String -> Get the unique key for the current task run
taskValues(): TaskValues -> Get the current task run's values
```

### Core Functions

#### 1. `dbutils.jobs.taskKey()` - Get Current Task Key
Returns the unique identifier for the current task within a job.

```python
# Get current task key
task_key = dbutils.jobs.taskKey()
print(f"Current task key: {task_key}")

# Use in logging
print(f"[{task_key}] Processing started...")
```

**Use Cases:**
- Task-specific logging
- Conditional logic based on task
- Dynamic resource allocation

#### 2. `dbutils.jobs.taskValues()` - Get Task Run Values
Retrieves values and metadata from the current task run.

```python
# Get task values
task_values = dbutils.jobs.taskValues()

print(f"Job ID: {task_values.job_id}")
print(f"Run ID: {task_values.run_id}")
print(f"Task Key: {task_values.task_key}")
print(f"Attempt Number: {task_values.attempt_number}")
```

**Available Properties:**
- `job_id` - Unique job identifier
- `run_id` - Current run identifier
- `task_key` - Current task key
- `attempt_number` - Retry attempt number
- `repair_id` - Repair run identifier (if applicable)

---

## 🏗️ Job Creation Methods

### Method 1: Databricks UI (Workflows)
1. **Navigate to Workflows** → Create Job
2. **Configure Job Details:**
   - Job name
   - Description
   - Tags (for organization)

3. **Add Tasks:**
   - Task name and type
   - Notebook/script path
   - Cluster configuration
   - Parameters

### Method 2: Databricks CLI
```bash
# Create job from JSON configuration
databricks jobs create --json-file job-config.json

# Example job-config.json
{
  "name": "Daily ETL Pipeline",
  "tasks": [
    {
      "task_key": "extract_data",
      "notebook_task": {
        "notebook_path": "/jobs/extract_data"
      },
      "new_cluster": {
        "spark_version": "11.3.x-scala2.12",
        "node_type_id": "i3.xlarge",
        "num_workers": 2
      }
    }
  ],
  "schedule": {
    "quartz_cron_expression": "0 0 2 * * ?",
    "timezone_id": "UTC"
  }
}
```

### Method 3: REST API
```python
import requests
import json

# Job configuration
job_config = {
    "name": "API Created Job",
    "tasks": [{
        "task_key": "main_task",
        "notebook_task": {
            "notebook_path": "/jobs/main_notebook",
            "base_parameters": {
                "environment": "production",
                "date": "{{start_date}}"
            }
        },
        "existing_cluster_id": "cluster-id-here"
    }]
}

# Create job via API
response = requests.post(
    f"{workspace_url}/api/2.1/jobs/create",
    headers={"Authorization": f"Bearer {access_token}"},
    json=job_config
)

job_id = response.json()["job_id"]
print(f"Created job with ID: {job_id}")
```

### Method 4: Terraform (Infrastructure as Code)
```hcl
resource "databricks_job" "etl_pipeline" {
  name = "ETL Pipeline"
  
  task {
    task_key = "extract"
    
    notebook_task {
      notebook_path = "/jobs/extract"
      base_parameters = {
        source_table = "raw_data"
        target_path = "/mnt/processed"
      }
    }
    
    new_cluster {
      num_workers   = 2
      spark_version = "11.3.x-scala2.12"
      node_type_id  = "i3.xlarge"
    }
  }
  
  schedule {
    quartz_cron_expression = "0 0 1 * * ?"
    timezone_id           = "America/New_York"
  }
}
```

---

## ⚙️ Job Configuration

### Task Types

#### 1. Notebook Task
```json
{
  "task_key": "notebook_task",
  "notebook_task": {
    "notebook_path": "/jobs/my_notebook",
    "base_parameters": {
      "param1": "value1",
      "param2": "value2"
    },
    "source": "WORKSPACE"
  }
}
```

#### 2. Python Script Task
```json
{
  "task_key": "python_task",
  "python_wheel_task": {
    "package_name": "my_package",
    "entry_point": "main",
    "parameters": ["--arg1", "value1"]
  }
}
```

#### 3. JAR Task
```json
{
  "task_key": "jar_task",
  "spark_jar_task": {
    "main_class_name": "com.company.MainClass",
    "parameters": ["param1", "param2"]
  },
  "libraries": [{
    "jar": "dbfs:/mnt/jars/my-app.jar"
  }]
}
```

#### 4. SQL Task
```json
{
  "task_key": "sql_task",
  "sql_task": {
    "query": {
      "query_id": "12345678-1234-1234-1234-123456789012"
    }
  }
}
```

#### 5. DLT (Delta Live Tables) Task
```json
{
  "task_key": "dlt_task",
  "pipeline_task": {
    "pipeline_id": "pipeline-id-here"
  }
}
```

### Cluster Configuration Options

#### 1. New Cluster
```json
{
  "new_cluster": {
    "spark_version": "11.3.x-scala2.12",
    "node_type_id": "i3.xlarge",
    "num_workers": 2,
    "spark_conf": {
      "spark.sql.adaptive.enabled": "true",
      "spark.sql.adaptive.coalescePartitions.enabled": "true"
    },
    "aws_attributes": {
      "instance_profile_arn": "arn:aws:iam::123456789012:instance-profile/databricks-role"
    }
  }
}
```

#### 2. Existing Cluster
```json
{
  "existing_cluster_id": "1234-567890-abc123"
}
```

#### 3. Job Cluster
```json
{
  "job_cluster_key": "shared_cluster",
  "job_clusters": [{
    "job_cluster_key": "shared_cluster",
    "new_cluster": {
      "spark_version": "11.3.x-scala2.12",
      "node_type_id": "i3.xlarge",
      "num_workers": 2
    }
  }]
}
```

---

## 📅 Job Scheduling

### Schedule Types

#### 1. Cron Scheduling
```json
{
  "schedule": {
    "quartz_cron_expression": "0 30 2 * * ?",
    "timezone_id": "America/New_York",
    "pause_status": "UNPAUSED"
  }
}
```

**Common Cron Patterns:**
- Daily at 2:30 AM: `"0 30 2 * * ?"`
- Every hour: `"0 0 * * * ?"`
- Weekly on Monday: `"0 0 9 ? * MON"`
- Monthly on 1st: `"0 0 9 1 * ?"`
- Every 15 minutes: `"0 */15 * * * ?"`

#### 2. File Arrival Trigger
```json
{
  "trigger": {
    "file_arrival": {
      "url": "s3://my-bucket/data/",
      "min_time_between_triggers_seconds": 60,
      "wait_after_last_change_seconds": 60
    }
  }
}
```

#### 3. Continuous Scheduling
```json
{
  "schedule": {
    "continuous": {
      "pause_status": "UNPAUSED"
    }
  }
}
```

### Schedule Management
```python
# In a notebook to check schedule info
task_values = dbutils.jobs.taskValues()
print(f"This job run started at: {task_values.start_time}")

# Get current date for date-based processing
from datetime import datetime
current_date = datetime.now().strftime("%Y-%m-%d")
print(f"Processing data for: {current_date}")
```

---

## 📊 Job Monitoring & Management

### Job Status Monitoring

#### Using Databricks CLI
```bash
# List all jobs
databricks jobs list

# Get job details
databricks jobs get --job-id 123

# List runs for a job
databricks jobs runs list --job-id 123

# Get run details
databricks jobs runs get --run-id 456

# Cancel a running job
databricks jobs runs cancel --run-id 456
```

#### Using REST API
```python
import requests

def get_job_status(job_id, run_id):
    response = requests.get(
        f"{workspace_url}/api/2.1/jobs/runs/get",
        headers={"Authorization": f"Bearer {access_token}"},
        params={"run_id": run_id}
    )
    
    run_info = response.json()
    return {
        "status": run_info["state"]["life_cycle_state"],
        "result": run_info["state"].get("result_state"),
        "start_time": run_info["start_time"],
        "end_time": run_info.get("end_time")
    }

# Usage
status = get_job_status(123, 456)
print(f"Job Status: {status['status']}")
```

### Job Run States
- **PENDING** - Waiting to start
- **RUNNING** - Currently executing
- **TERMINATING** - Shutting down
- **TERMINATED** - Completed
- **SKIPPED** - Skipped due to conditions
- **INTERNAL_ERROR** - System error occurred

### Result States (for completed jobs)
- **SUCCESS** - Completed successfully
- **FAILED** - Failed due to error
- **TIMEDOUT** - Exceeded timeout limit
- **CANCELED** - Manually canceled

---

## 🔗 Job Parameters & Dependencies

### Task Dependencies

#### Linear Dependencies
```json
{
  "tasks": [
    {
      "task_key": "extract",
      "notebook_task": {"notebook_path": "/jobs/extract"}
    },
    {
      "task_key": "transform",
      "depends_on": [{"task_key": "extract"}],
      "notebook_task": {"notebook_path": "/jobs/transform"}
    },
    {
      "task_key": "load",
      "depends_on": [{"task_key": "transform"}],
      "notebook_task": {"notebook_path": "/jobs/load"}
    }
  ]
}
```

#### Parallel Dependencies
```json
{
  "tasks": [
    {
      "task_key": "extract_source_a",
      "notebook_task": {"notebook_path": "/jobs/extract_a"}
    },
    {
      "task_key": "extract_source_b",
      "notebook_task": {"notebook_path": "/jobs/extract_b"}
    },
    {
      "task_key": "merge_data",
      "depends_on": [
        {"task_key": "extract_source_a"},
        {"task_key": "extract_source_b"}
      ],
      "notebook_task": {"notebook_path": "/jobs/merge"}
    }
  ]
}
```

### Parameter Passing

#### Static Parameters
```json
{
  "base_parameters": {
    "environment": "production",
    "batch_size": "1000",
    "output_path": "/mnt/processed"
  }
}
```

#### Dynamic Parameters
```python
# In the calling notebook/job
from datetime import datetime, timedelta

# Calculate yesterday's date
yesterday = (datetime.now() - timedelta(days=1)).strftime("%Y-%m-%d")

# Run job with dynamic parameter
result = dbutils.notebook.run(
    "/jobs/process_daily_data",
    timeout_seconds=3600,
    arguments={
        "process_date": yesterday,
        "retry_count": "3"
    }
)
```

#### Job-level Parameters (Widgets)
```python
# In the job notebook
dbutils.widgets.text("process_date", "")
dbutils.widgets.text("environment", "dev")
dbutils.widgets.text("batch_size", "1000")

# Get parameter values
process_date = dbutils.widgets.get("process_date")
environment = dbutils.widgets.get("environment")
batch_size = int(dbutils.widgets.get("batch_size"))

print(f"Processing date: {process_date}")
print(f"Environment: {environment}")
print(f"Batch size: {batch_size}")
```

---

## 🔄 Error Handling & Retry Logic

### Task-Level Retry Configuration
```json
{
  "task_key": "reliable_task",
  "notebook_task": {"notebook_path": "/jobs/data_processing"},
  "retry_on_timeout": true,
  "max_retries": 3,
  "min_retry_interval_millis": 300000,
  "timeout_seconds": 3600
}
```

### Custom Error Handling in Notebooks
```python
# In your job notebook
import sys
from datetime import datetime

def log_error(task_key, error_message):
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    print(f"[{timestamp}] [{task_key}] ERROR: {error_message}")

def safe_process_data():
    task_key = dbutils.jobs.taskKey() if 'dbutils' in globals() else "unknown"
    
    try:
        # Your data processing logic here
        print(f"[{task_key}] Starting data processing...")
        
        # Simulate processing
        df = spark.table("source_table")
        processed_df = df.filter(df.status == "active")
        processed_df.write.mode("overwrite").saveAsTable("target_table")
        
        print(f"[{task_key}] Processing completed successfully")
        dbutils.notebook.exit("SUCCESS")
        
    except Exception as e:
        error_msg = str(e)
        log_error(task_key, error_msg)
        
        # Determine if error is retryable
        retryable_errors = ["timeout", "connection", "temporary"]
        if any(err in error_msg.lower() for err in retryable_errors):
            print(f"[{task_key}] Retryable error encountered")
            dbutils.notebook.exit(f"RETRY: {error_msg}")
        else:
            print(f"[{task_key}] Non-retryable error encountered")
            dbutils.notebook.exit(f"FAILED: {error_msg}")

# Execute the safe processing
safe_process_data()
```

### Job-Level Error Handling
```python
# Monitor job execution and handle failures
def monitor_job_run(job_id, run_id, check_interval=60):
    import time
    import requests
    
    while True:
        response = requests.get(
            f"{workspace_url}/api/2.1/jobs/runs/get",
            headers={"Authorization": f"Bearer {access_token}"},
            params={"run_id": run_id}
        )
        
        run_info = response.json()
        state = run_info["state"]["life_cycle_state"]
        
        if state in ["TERMINATED", "SKIPPED", "INTERNAL_ERROR"]:
            result = run_info["state"].get("result_state")
            
            if result == "SUCCESS":
                print("Job completed successfully!")
                return True
            else:
                print(f"Job failed with result: {result}")
                # Handle failure (send alert, trigger retry, etc.)
                return False
        
        print(f"Job status: {state}")
        time.sleep(check_interval)
```

---

## 📖 Best Practices

### 1. Job Design Principles
- **Idempotent Tasks**: Design tasks to be safely re-runnable
- **Atomic Operations**: Each task should be a complete unit of work
- **Fail Fast**: Validate inputs early in the process
- **Logging**: Comprehensive logging for debugging and monitoring

### 2. Resource Management
```python
# Optimize cluster usage
def optimize_cluster_config(data_size_gb):
    if data_size_gb < 10:
        return {
            "node_type_id": "i3.large",
            "num_workers": 1
        }
    elif data_size_gb < 100:
        return {
            "node_type_id": "i3.xlarge", 
            "num_workers": 2
        }
    else:
        return {
            "node_type_id": "i3.2xlarge",
            "num_workers": 4
        }
```

### 3. Parameter Management
```python
# Centralized configuration
class JobConfig:
    def __init__(self):
        self.environment = dbutils.widgets.get("environment") if self.has_widgets() else "dev"
        self.batch_size = int(dbutils.widgets.get("batch_size")) if self.has_widgets() else 1000
        
        # Environment-specific settings
        if self.environment == "production":
            self.retry_count = 3
            self.timeout = 7200
        else:
            self.retry_count = 1
            self.timeout = 3600
    
    def has_widgets(self):
        try:
            dbutils.widgets.getAll()
            return True
        except:
            return False

# Usage
config = JobConfig()
print(f"Running in {config.environment} mode")
```

### 4. Monitoring and Alerting
```python
# Job health check function
def check_job_health(job_id, lookback_days=7):
    import requests
    from datetime import datetime, timedelta
    
    end_time = datetime.now()
    start_time = end_time - timedelta(days=lookback_days)
    
    response = requests.get(
        f"{workspace_url}/api/2.1/jobs/runs/list",
        headers={"Authorization": f"Bearer {access_token}"},
        params={
            "job_id": job_id,
            "start_time_from": int(start_time.timestamp() * 1000),
            "start_time_to": int(end_time.timestamp() * 1000)
        }
    )
    
    runs = response.json().get("runs", [])
    total_runs = len(runs)
    failed_runs = len([r for r in runs if r["state"].get("result_state") == "FAILED"])
    
    success_rate = ((total_runs - failed_runs) / total_runs * 100) if total_runs > 0 else 0
    
    return {
        "total_runs": total_runs,
        "failed_runs": failed_runs,
        "success_rate": success_rate,
        "health_status": "HEALTHY" if success_rate >= 95 else "UNHEALTHY"
    }
```

### 5. Testing Jobs
```python
# Test job locally before deployment
def test_job_logic():
    # Set up test widgets
    dbutils.widgets.text("environment", "test")
    dbutils.widgets.text("batch_size", "10")
    
    # Run the main logic
    try:
        # Your job logic here
        print("Test passed!")
        return True
    except Exception as e:
        print(f"Test failed: {e}")
        return False

# Run tests
if __name__ == "__main__":
    test_job_logic()
```

---

## ⚡ Quick Reference

### Common dbutils.jobs Patterns
```python
# Get current job context
task_values = dbutils.jobs.taskValues()
task_key = task_values.task_key
job_id = task_values.job_id
run_id = task_values.run_id

# Conditional logic based on task
if task_key == "extract":
    # Extract logic
    pass
elif task_key == "transform":
    # Transform logic  
    pass

# Task-specific logging
print(f"[{task_key}] Processing started at {datetime.now()}")
```

### Job Management CLI Commands
```bash
# Create job
databricks jobs create --json-file job-config.json

# Run job now
databricks jobs run-now --job-id 123

# Run with parameters
databricks jobs run-now --job-id 123 --notebook-params '{"param1":"value1"}'

# List recent runs
databricks jobs runs list --job-id 123 --limit 10

# Get run details
databricks jobs runs get --run-id 456

# Cancel run
databricks jobs runs cancel --run-id 456

# Delete job
databricks jobs delete --job-id 123
```

### Job Configuration Template
```json
{
  "name": "Production ETL Pipeline",
  "tags": {
    "team": "data-engineering",
    "environment": "production",
    "cost-center": "analytics"
  },
  "tasks": [
    {
      "task_key": "validate_inputs",
      "notebook_task": {
        "notebook_path": "/jobs/validate_inputs",
        "base_parameters": {
          "source_table": "raw_data",
          "validation_rules": "strict"
        }
      },
      "new_cluster": {
        "spark_version": "11.3.x-scala2.12",
        "node_type_id": "i3.large",
        "num_workers": 1
      },
      "timeout_seconds": 1800,
      "max_retries": 2
    },
    {
      "task_key": "process_data",
      "depends_on": [{"task_key": "validate_inputs"}],
      "notebook_task": {
        "notebook_path": "/jobs/process_data"
      },
      "job_cluster_key": "processing_cluster",
      "timeout_seconds": 7200,
      "max_retries": 3
    }
  ],
  "job_clusters": [{
    "job_cluster_key": "processing_cluster",
    "new_cluster": {
      "spark_version": "11.3.x-scala2.12",
      "node_type_id": "i3.xlarge",
      "num_workers": 4,
      "spark_conf": {
        "spark.sql.adaptive.enabled": "true"
      }
    }
  }],
  "schedule": {
    "quartz_cron_expression": "0 0 2 * * ?",
    "timezone_id": "UTC"
  },
  "max_concurrent_runs": 1,
  "notification_settings": {
    "no_alert_for_skipped_runs": false,
    "no_alert_for_canceled_runs": false
  }
}
```

### Error Handling Patterns
```python
# Standard error handling in job notebooks
def main():
    try:
        # Job logic here
        result = process_data()
        dbutils.notebook.exit(f"SUCCESS: {result}")
    except ValidationError as e:
        dbutils.notebook.exit(f"VALIDATION_FAILED: {e}")
    except TimeoutError as e:
        dbutils.notebook.exit(f"TIMEOUT: {e}")
    except Exception as e:
        dbutils.notebook.exit(f"FAILED: {e}")

if __name__ == "__main__":
    main()
```

This comprehensive guide covers all aspects of working with Databricks jobs, from basic `dbutils.jobs` functions to advanced job management, monitoring, and best practices.
