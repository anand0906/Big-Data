# ETL with Spark SQL - Simple Guide

## What is ETL?

ETL is like a data factory process with three steps:
- **E**xtract = Get data from somewhere (like downloading files)
- **T**ransform = Clean and organize the data (like sorting your photos)
- **L**oad = Put the clean data somewhere useful (like uploading to a database)

## What is Spark SQL?

Think of Spark SQL as a super-powered Excel that can:
- Handle millions of rows of data
- Work with data stored anywhere (files, databases, cloud)
- Use familiar SQL commands
- Run really fast by using multiple computers at once

## Why Use Spark SQL for ETL?

**Old Way (Traditional ETL)**
- Processes one thing at a time (like washing dishes one by one)
- Slow with large amounts of data
- Limited to specific tools

**New Way (Spark SQL ETL)**
- Processes many things simultaneously (like having 10 people wash dishes together)
- Fast with any amount of data
- Uses simple SQL commands

## Getting Started in Databricks

### Step 1: Set Up Your Workspace

```sql
-- Create a workspace for your data projects
CREATE DATABASE IF NOT EXISTS my_etl_project;
USE my_etl_project;

-- Make processing faster
SET spark.databricks.delta.autoOptimize.autoCompact = true;
```

Think of this like creating a new folder on your computer for a project.

### Step 2: Install What You Need

```python
# Install useful tools (like downloading apps on your phone)
%pip install delta-lake
%pip install pandas
```

