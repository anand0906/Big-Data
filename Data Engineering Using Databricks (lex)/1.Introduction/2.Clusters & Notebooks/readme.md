# Databricks Clusters & Notebooks Complete Guide 🚀

## Table of Contents
- [Clusters Overview](#clusters-overview)
- [Creating Clusters](#creating-clusters)
- [Managing Clusters](#managing-clusters)
- [Cluster Types Deep Dive](#cluster-types-deep-dive)
- [Notebooks Basics](#notebooks-basics)
- [Working with Notebooks](#working-with-notebooks)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Clusters Overview

### What is a Cluster? 🖥️
**Simple Definition:** A cluster is like a team of virtual computers working together to process your data.

**Real-World Analogy:** 
- **Single Computer** = One person doing math problems
- **Cluster** = A team of mathematicians solving problems together, much faster

### Why Do We Need Clusters?
```
Small Data (Excel file):     Your laptop can handle it
Medium Data (1GB CSV):       Might be slow on laptop  
Big Data (100GB+ files):     Need cluster power!
```

### Cluster Components
```
Cluster = Driver Node + Worker Nodes

Driver Node:    "The Manager"
├── Coordinates all work
├── Distributes tasks to workers  
├── Collects results
└── Communicates with Databricks UI

Worker Nodes:   "The Team Members"
├── Actually process the data
├── Store data in memory
├── Report back to driver
└── Can scale up/down automatically
```

---

## Creating Clusters

### Step-by-Step Cluster Creation 🛠️

#### Step 1: Access Cluster Page
```
Databricks Workspace → Compute → Create Cluster
```

#### Step 2: Basic Configuration
```yaml
Cluster Name: "my-analysis-cluster"
Cluster Mode: 
  - Standard (most common)
  - High Concurrency (for SQL)
  - Single Node (for learning/testing)
```

#### Step 3: Choose Databricks Runtime
```
Runtime Options:
├── Standard Runtime (7.3.x LTS)      # General purpose
├── ML Runtime (7.3.x ML)             # Machine Learning
├── GPU Runtime (7.3.x GPU ML)        # Deep Learning  
└── Photon Runtime (Latest)           # Ultra-fast SQL
```

#### Step 4: Worker Configuration
```yaml
Worker Type: 
  - i3.xlarge   (4 cores, 30.5 GB RAM)    # Balanced
  - r5.xlarge   (4 cores, 32 GB RAM)      # Memory optimized  
  - c5.xlarge   (4 cores, 8 GB RAM)       # Compute optimized

Workers:
  Min: 1    # Minimum number always running
  Max: 8    # Maximum for auto-scaling
```

#### Step 5: Driver Configuration  
```yaml
Driver Type: Same as worker (recommended)
             or larger for memory-intensive jobs
```

#### Step 6: Auto Scaling Settings
```yaml
Enable Autoscaling: ✅ Yes
Terminate after: 120 minutes of inactivity
```

### Real-World Example: E-commerce Analytics Cluster

**Business Need:** Analyze daily sales data (50GB) and customer behavior

**Cluster Configuration:**
```yaml
Cluster Name: "ecommerce-daily-analytics"
Runtime: "11.3 LTS (Scala 2.12, Spark 3.3.0)"
Worker Type: "r5.large" (2 cores, 16 GB RAM)
Workers: Min=2, Max=6
Driver: "r5.large" 
Auto Termination: 60 minutes
Auto Scaling: Enabled

Estimated Cost: $2-12/hour depending on load
```

**Why This Configuration:**
- **r5.large** - Good memory for data caching
- **Min=2** - Always have some capacity ready
- **Max=6** - Can handle peak loads
- **60 min timeout** - Saves cost when not in use

---

## Managing Clusters

### Cluster Lifecycle 🔄

#### Cluster States
```
PENDING    → Starting up (2-5 minutes)
RUNNING    → Ready for work
RESTARTING → Applying configuration changes  
RESIZING   → Adding/removing workers
TERMINATING→ Shutting down
TERMINATED → Stopped (not using resources)
```

### Common Management Tasks

#### 1. Starting a Cluster
```
Actions:
- Click "Start" button
- Wait 2-5 minutes for initialization
- Green "Running" status means ready

What happens behind scenes:
1. Databricks requests VMs from cloud provider
2. Installs runtime software
3. Downloads libraries and dependencies
4. Establishes network connections
5. Reports "Ready" to control plane
```

#### 2. Monitoring Cluster Performance
```
Metrics Tab shows:
├── CPU Usage (should be 60-80% for good efficiency)
├── Memory Usage (watch for out-of-memory errors)  
├── Network I/O (data transfer rates)
├── Disk Usage (temporary storage)
└── Task Execution (success/failure rates)
```

#### 3. Scaling Operations
```
Manual Scaling:
- Edit cluster → Change min/max workers → Confirm
- Takes 2-3 minutes to add workers
- Immediate to remove workers

Auto Scaling:
- Monitors task queue length
- Adds workers when queue grows
- Removes workers when idle
```

#### 4. Installing Libraries
```
Libraries Tab:
├── PyPI packages:     pip install pandas==1.3.3
├── Maven (Scala/Java): groupId:artifactId:version
├── CRAN (R packages):  install.packages("ggplot2")
└── Upload JAR/Wheel:   Custom packages
```

### Cost Management 💰

#### Cost Optimization Strategies
```yaml
# Development Cluster (Cost-Optimized)
Worker Type: "t3.medium"     # Cheap, good for learning
Workers: Min=0, Max=2        # Scale to zero when unused
Auto Termination: 30 min    # Aggressive termination

# Production Cluster (Performance-Optimized) 
Worker Type: "r5.xlarge"     # More power for real work
Workers: Min=2, Max=20       # Always ready, can scale high
Auto Termination: 120 min   # Keep running longer
```

#### Monitoring Costs
```
Cost Tracking:
├── Account Console → Usage Dashboard
├── Shows DBU (Databricks Unit) consumption  
├── Filter by cluster, user, job
└── Set up budget alerts
```

---

## Cluster Types Deep Dive

### 1. All-Purpose Clusters 🔧

**Best For:** Interactive data exploration and development

**Characteristics:**
- Can be shared by multiple users
- Stay running between sessions
- Good for notebooks and ad-hoc analysis

**Example Use Case:**
```python
# Data scientist exploring customer data
import pandas as pd
import matplotlib.pyplot as plt

# Load data
df = spark.read.table("sales.customer_transactions")

# Explore interactively
df.describe()
df.groupBy("product_category").count().show()
```

### 2. Job Clusters ⚡

**Best For:** Automated, scheduled jobs

**Characteristics:**
- Start automatically when job runs
- Terminate when job completes  
- More cost-effective for production workflows

**Example Configuration:**
```json
{
  "job_id": 123,
  "cluster_spec": {
    "spark_version": "11.3.x-scala2.12",
    "node_type_id": "r5.large",
    "num_workers": 4,
    "runtime_engine": "STANDARD"
  },
  "schedule": {
    "cron_expression": "0 2 * * *",  // Daily at 2 AM
    "timezone": "UTC"
  }
}
```

### 3. SQL Warehouses 📊

**Best For:** SQL queries and BI dashboards

**Characteristics:**  
- Optimized for SQL performance
- Auto-scaling based on query load
- Easy for business analysts to use

**Example Usage:**
```sql
-- Business analyst creating weekly report
SELECT 
    DATE_TRUNC('week', transaction_date) as week,
    product_category,
    SUM(revenue) as weekly_revenue,
    COUNT(DISTINCT customer_id) as unique_customers
FROM sales.transactions 
WHERE transaction_date >= CURRENT_DATE - INTERVAL 30 DAYS
GROUP BY week, product_category
ORDER BY week DESC, weekly_revenue DESC
```

---

## Notebooks Basics

### What is a Notebook? 📓

**Simple Definition:** A notebook is like a digital lab notebook where you can write code, see results, and add explanations all in one place.

**Components:**
```
Notebook = Cells + Code + Results + Documentation

Cell Types:
├── Code Cell:     Write and execute code
├── Markdown Cell: Add explanations, headers  
├── SQL Cell:      Database queries
└── Command Cell:  Special Databricks commands
```

### Creating Your First Notebook 🆕

#### Step 1: Create Notebook
```
Workspace → Create → Notebook

Settings:
├── Name: "Customer Analysis"
├── Default Language: Python
├── Cluster: Select existing cluster
└── Path: /Users/your.email@company.com/
```

#### Step 2: Basic Notebook Structure
```python
# Cell 1: Setup and Imports
import pandas as pd
import matplotlib.pyplot as plt
from pyspark.sql import functions as F

# Cell 2: Load Data  
df = spark.read.table("sales.customer_data")
print(f"Loaded {df.count()} customer records")

# Cell 3: Data Exploration
display(df.describe())
```

### Notebook Interface Elements 🖥️

#### Top Menu Bar
```
File    Edit    View    Insert    Runtime    Tools    Help
└── Save, Export, Import, Settings, etc.
```

#### Cell Controls
```
[▶ Run]  [↓ Add Cell]  [🗑️ Delete]  [↑↓ Move]  [⋯ More]
```

#### Right Sidebar
```
├── Table of Contents (auto-generated from headers)
├── Variable Explorer (see current variables)  
├── Comments (collaborate with team)
└── Revision History (track changes)
```

---

## Working with Notebooks

### Multi-Language Support 🌐

#### Default Language Setting
```python
# Notebook default is Python
df = spark.read.table("sales.data")
df.show(5)
```

#### Language Magic Commands
```python
# Switch to SQL for this cell
%sql
SELECT * FROM sales.data LIMIT 5

# Switch to Scala
%scala  
val df = spark.read.table("sales.data")
df.show(5)

# Switch to R
%r
library(SparkR)
df <- sql("SELECT * FROM sales.data LIMIT 5")
```

### Data Visualization 📈

#### Built-in Visualization
```python
# Create DataFrame
df = spark.sql("""
    SELECT product_category, SUM(revenue) as total_revenue
    FROM sales.transactions  
    GROUP BY product_category
""")

# Display with built-in charts
display(df)
# Click on chart icon below result → Choose bar chart
```

#### Custom Plots with Matplotlib
```python
import matplotlib.pyplot as plt

# Convert to Pandas for plotting
pandas_df = df.toPandas()

# Create custom plot
plt.figure(figsize=(10, 6))
plt.bar(pandas_df['product_category'], pandas_df['total_revenue'])
plt.title('Revenue by Product Category')
plt.xticks(rotation=45)
plt.show()
```

### Databricks Utilities (dbutils) 🛠️

#### File System Operations
```python
# List files in storage
dbutils.fs.ls("dbfs:/mnt/datalake/sales/")

# Copy files
dbutils.fs.cp("source/path", "destination/path")

# Remove files  
dbutils.fs.rm("path/to/file", True)  # True for recursive
```

#### Secrets Management
```python
# Get secret from key vault (secure way to store passwords)
api_key = dbutils.secrets.get(scope="my-secrets", key="api-key")

# Use in connection
response = requests.get(
    "https://api.example.com/data",
    headers={"Authorization": f"Bearer {api_key}"}
)
```

#### Notebook Workflows
```python
# Run another notebook and get result
result = dbutils.notebook.run("./data-preprocessing", 600, {"date": "2024-01-15"})

# Exit notebook with return value
dbutils.notebook.exit("processing-complete")
```

### Real-World Example: Customer Analysis Notebook

```python
# === CELL 1: Setup ===
# %md
# # Daily Customer Analysis Report
# **Date:** 2024-01-15  
# **Purpose:** Analyze customer behavior and identify trends

# === CELL 2: Configuration ===
# Parameters
analysis_date = "2024-01-15"
min_purchase_amount = 50

# Imports
from pyspark.sql import functions as F
import pandas as pd
import matplotlib.pyplot as plt

# === CELL 3: Data Loading ===
# Load customer transactions
transactions = spark.read.table("sales.transactions") \
    .filter(F.col("transaction_date") == analysis_date) \
    .filter(F.col("amount") >= min_purchase_amount)

print(f"Found {transactions.count()} transactions above ${min_purchase_amount}")

# === CELL 4: Analysis ===
# Customer segmentation
customer_stats = transactions.groupBy("customer_id") \
    .agg(
        F.sum("amount").alias("total_spent"),
        F.count("*").alias("transaction_count"),
        F.avg("amount").alias("avg_transaction")
    )

# Show sample
display(customer_stats.limit(10))

# === CELL 5: Insights ===
# Top customers by spending  
top_customers = customer_stats.orderBy(F.desc("total_spent")).limit(20)
display(top_customers)

# === CELL 6: Visualization ===
# Distribution of customer spending
spending_dist = customer_stats.select("total_spent").toPandas()

plt.figure(figsize=(12, 6))
plt.hist(spending_dist['total_spent'], bins=50, alpha=0.7)
plt.title('Customer Spending Distribution')
plt.xlabel('Total Spent ($)')
plt.ylabel('Number of Customers')
plt.show()

# === CELL 7: Export Results ===
# Save results for later use
top_customers.write.mode("overwrite").saveAsTable("analytics.top_customers_daily")
print("Results saved to analytics.top_customers_daily table")
```

---

## Advanced Features

### Collaborative Features 👥

#### Real-time Collaboration
```
Multiple users can:
├── Edit same notebook simultaneously
├── See each other's cursors and changes
├── Add comments on specific cells
└── Chat via comment threads
```

#### Version Control
```
Integration Options:
├── Git Integration (connect to GitHub/GitLab)
├── Revision History (automatic saves)
├── Export/Import (.dbc files)
└── Workspace file versioning
```

### Notebook Automation 🤖

#### Scheduled Notebooks
```python
# Create job from notebook
{
  "name": "Daily Customer Analysis",
  "notebook_task": {
    "notebook_path": "/Users/analyst/customer-analysis",
    "parameters": {
      "analysis_date": "{{DS}}"  # Today's date
    }
  },
  "schedule": {
    "cron_expression": "0 9 * * *"  # Daily at 9 AM
  },
  "email_notifications": {
    "on_success": ["team@company.com"],
    "on_failure": ["admin@company.com"]
  }
}
```

#### Parameterized Notebooks
```python
# Cell 1: Get parameters
dbutils.widgets.text("start_date", "2024-01-01", "Analysis Start Date")
dbutils.widgets.dropdown("region", "US", ["US", "EU", "APAC"], "Region")

# Cell 2: Use parameters
start_date = dbutils.widgets.get("start_date")
region = dbutils.widgets.get("region")

df = spark.read.table("sales.data") \
    .filter(F.col("date") >= start_date) \
    .filter(F.col("region") == region)
```

---

## Best Practices

### Cluster Best Practices 🎯

#### 1. Right-Sizing Clusters
```yaml
# Small Data (< 1GB)
Cluster: Single Node
Worker Type: i3.large
Cost: ~$0.30/hour

# Medium Data (1-100GB)  
Workers: 2-8 nodes
Worker Type: r5.xlarge  
Cost: ~$2-8/hour

# Large Data (100GB+)
Workers: 8-50 nodes
Worker Type: r5.2xlarge+
Cost: ~$16-200/hour
```

#### 2. Cost Optimization
```python
# Use spot instances for development
spot_config = {
    "use_spot_instances_for_driver": False,  # Driver should be stable
    "use_spot_instances_for_workers": True,  # Workers can handle interruptions
    "spot_bid_price_percent": 50            # Bid 50% of on-demand price
}

# Set aggressive auto-termination for dev
auto_termination_minutes = 15  # For experimentation
auto_termination_minutes = 120 # For production jobs
```

#### 3. Performance Tuning
```python
# Spark Configuration for better performance
spark_config = {
    "spark.sql.adaptive.enabled": "true",
    "spark.sql.adaptive.coalescePartitions.enabled": "true", 
    "spark.sql.adaptive.skewJoin.enabled": "true",
    "spark.serializer": "org.apache.spark.serializer.KryoSerializer"
}
```

### Notebook Best Practices 📝

#### 1. Structure and Organization
```python
# ✅ GOOD: Clear structure
# === SETUP ===
# imports and configuration

# === DATA LOADING === 
# load all required datasets

# === ANALYSIS ===
# main analysis logic

# === RESULTS ===
# save outputs and generate reports

# ❌ BAD: Mixed structure
# imports scattered throughout
# unclear cell purposes
```

#### 2. Documentation
```markdown
# ✅ GOOD: Well documented
# %md
# ## Customer Segmentation Analysis
# 
# **Objective:** Identify high-value customer segments
# **Data Sources:** 
# - `sales.transactions` - Purchase history
# - `customers.profiles` - Demographics
# 
# **Methodology:**
# 1. RFM Analysis (Recency, Frequency, Monetary)
# 2. K-means clustering
# 3. Segment profiling
```

#### 3. Code Quality
```python
# ✅ GOOD: Clean, readable code
def calculate_customer_ltv(transactions_df, months=12):
    """
    Calculate Customer Lifetime Value
    
    Args:
        transactions_df: DataFrame with customer transactions
        months: Number of months to project (default 12)
    
    Returns:
        DataFrame with customer_id and projected_ltv
    """
    monthly_avg = transactions_df.groupBy("customer_id") \
        .agg(F.avg("monthly_revenue").alias("avg_monthly_revenue"))
    
    return monthly_avg.withColumn(
        "projected_ltv", 
        F.col("avg_monthly_revenue") * months
    )

# ❌ BAD: Unclear, undocumented code  
df2 = df.groupBy("c").agg(F.avg("r").alias("ar"))
df3 = df2.withColumn("ltv", F.col("ar") * 12)
```

---

## Troubleshooting

### Common Cluster Issues 🔧

#### Issue 1: Cluster Won't Start
```
Symptoms: Stuck in "PENDING" state for >10 minutes

Possible Causes:
├── No available capacity in cloud region
├── Invalid cluster configuration  
├── IAM/permission issues
└── Network connectivity problems

Solutions:
├── Try different node types
├── Switch to different availability zone
├── Check cloud account limits
└── Verify network and security settings
```

#### Issue 2: Out of Memory Errors
```
Error: "java.lang.OutOfMemoryError"

Solutions:
├── Increase worker memory (upgrade instance type)
├── Reduce data partition size
├── Use more efficient data types
├── Add more workers to cluster
└── Implement data caching strategies

Example Fix:
# Before: Loading all data at once
df = spark.read.table("huge_table")  # May crash

# After: Process in smaller chunks  
df = spark.read.table("huge_table") \
    .filter(F.col("date") >= "2024-01-01") \  # Filter early
    .repartition(50)  # Increase partitions
```

#### Issue 3: Slow Performance
```
Symptoms: Queries taking much longer than expected

Diagnostics:
├── Check Spark UI (cluster → Spark UI)
├── Look for data skew in partitions
├── Monitor cluster resource usage
└── Review query execution plans

Common Fixes:
├── Repartition skewed data
├── Broadcast small lookup tables  
├── Use appropriate file formats (Delta, Parquet)
└── Optimize JOIN operations
```

### Common Notebook Issues 📓

#### Issue 1: Notebook Not Responding
```
Symptoms: Cells not executing, spinning indefinitely

Solutions:
├── Restart cluster (Compute → Restart)
├── Clear notebook state (Run → Clear State)  
├── Detach and reattach cluster
└── Check cluster logs for errors
```

#### Issue 2: Memory Issues in Notebooks
```
Symptoms: "Driver out of memory" errors

Solutions:
# Avoid collecting large datasets to driver
# ❌ BAD
large_df = spark.read.table("big_table")
pandas_df = large_df.toPandas()  # Crashes with large data

# ✅ GOOD  
large_df = spark.read.table("big_table")
sample_df = large_df.sample(0.01)  # Sample first
pandas_df = sample_df.toPandas()   # Safe to convert
```

#### Issue 3: Library Installation Issues
```
Error: "ModuleNotFoundError: No module named 'xyz'"

Solutions:
├── Install on cluster (Libraries tab)
├── Use %pip install in notebook cell
├── Check library compatibility with runtime
└── Restart cluster after installation

Example:
# Install library in notebook
%pip install plotly==5.0.0

# Restart Python process  
dbutils.library.restartPython()

# Now can import
import plotly.express as px
```

---

## Summary

### Clusters Quick Reference 🎯
```yaml
Development:
  Type: All-Purpose, Single Node
  Size: Small (i3.large)
  Auto-terminate: 30 minutes
  
Production Jobs:  
  Type: Job Cluster
  Size: Medium-Large (r5.xlarge+)
  Auto-scaling: Enabled
  
SQL Analytics:
  Type: SQL Warehouse  
  Size: Auto-scaling
  Optimized: For query performance
```

### Notebooks Quick Reference 📋
```python
# Essential notebook patterns
# 1. Setup cell
import libraries, set configuration

# 2. Data loading cell  
df = spark.read.table("source.table")

# 3. Analysis cells
results = df.groupBy().agg().orderBy()

# 4. Visualization cell
display(results)  # Built-in charts
# or matplotlib for custom plots

# 5. Save results cell
results.write.saveAsTable("output.table")
```

### Key Takeaways 💡
1. **Start Small** - Use single-node clusters for learning
2. **Right-size** - Match cluster size to data size  
3. **Auto-terminate** - Save costs with automatic shutdown
4. **Document Everything** - Use markdown cells liberally
5. **Version Control** - Connect notebooks to Git
6. **Monitor Costs** - Set up alerts and track usage
7. **Collaborate** - Share notebooks and work together

This foundation will help you effectively create, manage, and use Databricks clusters and notebooks for your data projects!
