# Managing Permissions in Databricks

## Table of Contents
- [What are Permissions?](#what-are-permissions)
- [Permission Model Overview](#permission-model-overview)
- [Types of Permissions](#types-of-permissions)
- [User Management](#user-management)
- [Group Management](#group-management)
- [Workspace Permissions](#workspace-permissions)
- [Data Permissions](#data-permissions)
- [Compute Permissions](#compute-permissions)
- [Job Permissions](#job-permissions)
- [Simple Examples](#simple-examples)
- [Advanced Scenarios](#advanced-scenarios)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## What are Permissions?

Permissions in Databricks control **who can do what** with your data and resources. Think of it like having different keys for different rooms in a building - some people can enter all rooms, others only specific ones.

**Why are permissions important?**
- **Security**: Protect sensitive data from unauthorized access
- **Compliance**: Meet regulatory requirements (GDPR, HIPAA, etc.)
- **Organization**: Keep different teams' work separate
- **Cost Control**: Prevent unauthorized resource usage

## Permission Model Overview

Databricks uses a **hierarchical permission model** with three main levels:

```
Account Level (Highest)
    ↓
Workspace Level (Middle)  
    ↓
Object Level (Most Specific)
```

**How it works**: Permissions flow down from higher levels, but you can make them more restrictive at lower levels (but not more permissive).

## Types of Permissions

### 1. **Account-Level Permissions**
Control access to the entire Databricks account
- Account Admin
- Workspace Creator
- User Management

### 2. **Workspace-Level Permissions**
Control what users can do within a workspace
- Workspace Admin
- User
- Contributor

### 3. **Object-Level Permissions**
Control access to specific items like notebooks, clusters, jobs
- Owner
- Can Edit
- Can View
- Can Run

## User Management

### Adding Users

#### Method 1: Through UI
1. Go to **Settings** → **Identity and Access**
2. Click **Users** tab
3. Click **Add User**
4. Enter email and assign role
5. Click **Send Invitation**

#### Method 2: Through API
```python
from databricks.sdk import AccountClient

# Initialize account client
account_client = AccountClient()

# Add new user
user = account_client.users.create(
    user_name="john.doe@company.com",
    display_name="John Doe", 
    active=True
)

print(f"User created with ID: {user.id}")
```

### User Roles Explained

| Role | What They Can Do | Best For |
|------|------------------|----------|
| **Account Admin** | Everything in the account | IT administrators |
| **Workspace Admin** | Manage workspace settings | Team leads |
| **User** | Create notebooks, clusters | Data scientists |
| **Viewer** | View shared content only | Business stakeholders |

## Group Management

Groups make permission management much easier by allowing you to assign permissions to groups instead of individual users.

### Creating Groups

#### Through UI
1. Go to **Settings** → **Identity and Access**
2. Click **Groups** tab
3. Click **Add Group**
4. Name your group (e.g., "data_engineers", "analysts")
5. Add members

#### Through Code
```python
# Create a new group
group = account_client.groups.create(
    display_name="Data Engineering Team",
    members=[
        {"value": "user1@company.com"},
        {"value": "user2@company.com"}
    ]
)
```

### Common Group Structure

```
📁 Company Groups
├── 👥 data_engineers (Full access to data processing)
├── 👥 data_scientists (Access to ML and analytics)  
├── 👥 business_analysts (Read access to gold tables)
├── 👥 finance_team (Access to financial data only)
└── 👥 external_contractors (Limited temporary access)
```

## Workspace Permissions

### Permission Levels

#### **Workspace Admin**
- Manage all workspace settings
- Add/remove users
- Control billing
- Manage security settings

```python
# Grant workspace admin access
workspace_client.permissions.update(
    request_object_type="workspace",
    request_object_id="workspace_id",
    access_control_list=[
        {
            "user_name": "admin@company.com",
            "permission_level": "IS_OWNER"
        }
    ]
)
```

#### **User Access**
- Create notebooks and clusters
- Run jobs
- Access shared resources

#### **No Access**
- Cannot see or access the workspace

### Workspace Features Control

```python
# Control specific workspace features
workspace_settings = {
    "enableNotebookTableClipboard": False,  # Disable copy/paste from tables
    "enableWebTerminal": False,             # Disable terminal access
    "enableDbfsFileBrowser": True,          # Allow file browser
    "enableExperimentalFeatures": False     # Disable beta features
}
```

## Data Permissions

### Table Access Control

Data permissions in Databricks work at multiple levels:

#### **Catalog Level** (Highest)
Controls access to entire data catalogs

#### **Schema Level** (Middle) 
Controls access to groups of tables

#### **Table Level** (Most Specific)
Controls access to individual tables

### Setting Up Data Permissions

#### 1. **Enable Unity Catalog** (Recommended)
Unity Catalog provides fine-grained data governance.

```sql
-- Grant permissions using Unity Catalog
GRANT SELECT ON CATALOG sales_data TO `data_analysts`;
GRANT ALL PRIVILEGES ON SCHEMA sales_data.bronze TO `data_engineers`;
GRANT SELECT ON TABLE sales_data.gold.customer_summary TO `business_users`;
```

#### 2. **Legacy Table ACLs** (Older method)
```sql
-- Grant table permissions (legacy method)
GRANT SELECT ON DATABASE sales TO GROUP analysts;
GRANT ALL PRIVILEGES ON TABLE sales.customers TO GROUP data_engineers;
DENY SELECT ON TABLE sales.sensitive_data TO GROUP contractors;
```

### Data Classification Example

```sql
-- Example of setting up data access by classification
-- Public data - everyone can read
GRANT SELECT ON SCHEMA company.public TO `all_users`;

-- Internal data - only employees
GRANT SELECT ON SCHEMA company.internal TO `employees`;

-- Confidential data - only specific teams
GRANT SELECT ON SCHEMA company.confidential TO `senior_analysts`;
GRANT SELECT ON SCHEMA company.confidential TO `data_engineers`;

-- Restricted data - very limited access
GRANT SELECT ON TABLE company.confidential.salary_data TO `hr_team`;
GRANT SELECT ON TABLE company.confidential.salary_data TO `finance_executives`;
```

## Compute Permissions

### Cluster Permissions

#### **Cluster Access Levels**
- **Can Attach To**: Can run notebooks on the cluster
- **Can Restart**: Can start/stop the cluster  
- **Can Manage**: Full control over cluster settings

#### Setting Cluster Permissions
```python
# Set cluster permissions via API
cluster_permissions = {
    "access_control_list": [
        {
            "group_name": "data_engineers",
            "permission_level": "CAN_MANAGE"
        },
        {
            "group_name": "data_analysts", 
            "permission_level": "CAN_ATTACH_TO"
        },
        {
            "user_name": "contractor@external.com",
            "permission_level": "CAN_ATTACH_TO"
        }
    ]
}

workspace_client.permissions.update(
    request_object_type="clusters",
    request_object_id=cluster_id,
    **cluster_permissions
)
```

### Cluster Policies

Cluster policies control what types of clusters users can create.

```json
{
  "name": "Standard Data Science Policy",
  "definition": {
    "node_type_id": {
      "type": "allowlist",
      "values": ["i3.xlarge", "i3.2xlarge", "r5.xlarge"]
    },
    "num_workers": {
      "type": "range", 
      "min": 1,
      "max": 10
    },
    "autotermination_minutes": {
      "type": "fixed",
      "value": 120
    },
    "spark_conf.spark.databricks.cluster.profile": {
      "type": "fixed", 
      "value": "singleNode"
    }
  }
}
```

## Job Permissions

### Job Access Levels
- **Is Owner**: Full control over the job
- **Can Manage**: Edit job configuration and permissions
- **Can View**: See job details and run history
- **Can Run**: Trigger job execution only

### Example: Setting Up Job Permissions

```python
# Job permission setup for a data pipeline
job_permissions = {
    "access_control_list": [
        {
            "group_name": "data_engineers",
            "permission_level": "IS_OWNER"  # Can modify job
        },
        {
            "group_name": "data_analysts",
            "permission_level": "CAN_VIEW"  # Can see results
        },
        {
            "group_name": "business_users",
            "permission_level": "CAN_RUN"   # Can trigger runs
        }
    ]
}

workspace_client.permissions.update(
    request_object_type="jobs",
    request_object_id=job_id,
    **job_permissions
)
```

## Simple Examples

### Example 1: Setting Up a New Data Team

#### Scenario
You're setting up permissions for a new data science team with 3 roles:
- **Team Lead** (manages everything)
- **Data Scientists** (create models, access clean data)
- **Interns** (limited access, supervised work)

#### Step-by-Step Setup

```python
# Step 1: Create groups
groups_to_create = [
    "ds_team_leads",
    "ds_scientists", 
    "ds_interns"
]

for group_name in groups_to_create:
    account_client.groups.create(display_name=group_name)

# Step 2: Add users to groups
# Team Lead
account_client.groups.patch(
    id="ds_team_leads",
    operations=[{
        "op": "add",
        "path": "members",
        "value": [{"value": "team.lead@company.com"}]
    }]
)

# Data Scientists
scientists = ["alice@company.com", "bob@company.com", "charlie@company.com"]
for scientist in scientists:
    account_client.groups.patch(
        id="ds_scientists",
        operations=[{
            "op": "add", 
            "path": "members",
            "value": [{"value": scientist}]
        }]
    )
```

```sql
-- Step 3: Set up data access
-- Team leads get full access
GRANT ALL PRIVILEGES ON CATALOG ml_data TO `ds_team_leads`;

-- Scientists get read/write to specific schemas
GRANT SELECT, MODIFY ON SCHEMA ml_data.features TO `ds_scientists`;
GRANT SELECT, MODIFY ON SCHEMA ml_data.models TO `ds_scientists`;
GRANT SELECT ON SCHEMA ml_data.gold TO `ds_scientists`;

-- Interns get read-only access to training data
GRANT SELECT ON SCHEMA ml_data.training TO `ds_interns`;
GRANT USE SCHEMA ON SCHEMA ml_data.training TO `ds_interns`;
```

### Example 2: Project-Based Access Control

#### Scenario
Multiple teams working on different projects need isolated access.

```sql
-- Project Alpha (Marketing Analytics)
CREATE SCHEMA IF NOT EXISTS company.project_alpha;
GRANT ALL PRIVILEGES ON SCHEMA company.project_alpha TO `marketing_team`;
GRANT ALL PRIVILEGES ON SCHEMA company.project_alpha TO `alpha_contractors`;

-- Project Beta (Financial Reporting)  
CREATE SCHEMA IF NOT EXISTS company.project_beta;
GRANT ALL PRIVILEGES ON SCHEMA company.project_beta TO `finance_team`;
GRANT SELECT ON SCHEMA company.project_beta TO `finance_auditors`;

-- Shared reference data
GRANT SELECT ON SCHEMA company.reference TO `marketing_team`;
GRANT SELECT ON SCHEMA company.reference TO `finance_team`;
```

### Example 3: Environment-Based Permissions

```python
# Different permissions for different environments
environments = {
    "dev": {
        "groups": ["developers", "data_engineers"],
        "cluster_policy": "dev_policy",
        "permissions": "CAN_MANAGE"
    },
    "staging": {
        "groups": ["qa_team", "data_engineers"],
        "cluster_policy": "staging_policy", 
        "permissions": "CAN_ATTACH_TO"
    },
    "prod": {
        "groups": ["production_support"],
        "cluster_policy": "prod_policy",
        "permissions": "CAN_VIEW"
    }
}

# Apply permissions for each environment
for env, config in environments.items():
    for group in config["groups"]:
        # Set workspace permissions
        workspace_client.permissions.update(
            request_object_type="workspace",
            access_control_list=[{
                "group_name": group,
                "permission_level": config["permissions"]
            }]
        )
```

## Advanced Scenarios

### 1. **Dynamic Row-Level Security**

```sql
-- Create a view with row-level security
CREATE OR REPLACE VIEW secure_customer_data AS
SELECT 
    customer_id,
    customer_name,
    order_amount,
    region
FROM customers 
WHERE 
    -- Users can only see data from their region
    region = (
        SELECT region 
        FROM user_regions 
        WHERE user_email = current_user()
    )
    OR
    -- Admins can see everything
    is_account_group_member('admin_group');
```

### 2. **Time-Based Access Control**

```python
# Grant temporary access (e.g., for contractors)
from datetime import datetime, timedelta

def grant_temporary_access(user_email, days=30):
    # Grant access
    workspace_client.permissions.update(
        request_object_type="workspace",
        access_control_list=[{
            "user_name": user_email,
            "permission_level": "CAN_USE"
        }]
    )
    
    # Schedule removal (would need external scheduler)
    removal_date = datetime.now() + timedelta(days=days)
    print(f"Access granted until {removal_date}")
    
    # In practice, you'd schedule this with a job
    return removal_date

# Usage
grant_temporary_access("contractor@external.com", days=60)
```

### 3. **Conditional Permissions Based on Data Classification**

```sql
-- Different access based on data sensitivity
CREATE OR REPLACE FUNCTION can_access_sensitive_data(user_email STRING)
RETURNS BOOLEAN
RETURN (
    is_account_group_member('senior_analysts') OR
    is_account_group_member('data_protection_officers') OR
    user_email IN ('cto@company.com', 'legal@company.com')
);

-- Apply in views
CREATE OR REPLACE VIEW customer_data_view AS
SELECT 
    customer_id,
    customer_name,
    CASE 
        WHEN can_access_sensitive_data(current_user()) 
        THEN email_address 
        ELSE 'REDACTED' 
    END as email_address,
    CASE 
        WHEN can_access_sensitive_data(current_user()) 
        THEN phone_number 
        ELSE 'REDACTED' 
    END as phone_number,
    order_history
FROM customers;
```

## Permission Setup Walkthrough

### Setting Up a Complete Project

Let's set up permissions for a "Customer Analytics" project:

#### Step 1: Create Project Groups
```python
# Create project-specific groups
project_groups = {
    "customer_analytics_admins": "Full project control",
    "customer_analytics_engineers": "Data pipeline development", 
    "customer_analytics_analysts": "Data analysis and reporting",
    "customer_analytics_viewers": "View reports only"
}

for group_name, description in project_groups.items():
    account_client.groups.create(
        display_name=group_name,
        meta={"description": description}
    )
```

#### Step 2: Set Up Data Access
```sql
-- Create project schema
CREATE SCHEMA IF NOT EXISTS analytics.customer_project;

-- Set permissions by role
-- Admins: Full control
GRANT ALL PRIVILEGES ON SCHEMA analytics.customer_project 
TO `customer_analytics_admins`;

-- Engineers: Read/write to staging, full control of bronze/silver
GRANT ALL PRIVILEGES ON SCHEMA analytics.customer_project 
TO `customer_analytics_engineers`;

-- Analysts: Read access to silver/gold, write to sandbox
GRANT SELECT ON SCHEMA analytics.customer_project 
TO `customer_analytics_analysts`;
GRANT CREATE TABLE ON SCHEMA analytics.customer_project 
TO `customer_analytics_analysts`;

-- Viewers: Read-only access to gold layer only
CREATE VIEW analytics.customer_project.customer_summary_view AS
SELECT customer_segment, COUNT(*) as customer_count, AVG(total_value) as avg_value
FROM analytics.customer_project.gold_customer_summary
GROUP BY customer_segment;

GRANT SELECT ON VIEW analytics.customer_project.customer_summary_view 
TO `customer_analytics_viewers`;
```

#### Step 3: Configure Compute Access
```python
# Create cluster policy for the project
cluster_policy = {
    "name": "Customer Analytics Policy",
    "definition": {
        "node_type_id": {
            "type": "allowlist",
            "values": ["i3.large", "i3.xlarge", "i3.2xlarge"]
        },
        "autotermination_minutes": {
            "type": "range",
            "min": 10,
            "max": 180
        },
        "custom_tags.project": {
            "type": "fixed",
            "value": "customer_analytics"
        },
        "custom_tags.cost_center": {
            "type": "fixed", 
            "value": "marketing_dept"
        }
    }
}

policy = workspace_client.cluster_policies.create(**cluster_policy)

# Assign policy to groups
workspace_client.permissions.update(
    request_object_type="cluster-policies",
    request_object_id=policy.policy_id,
    access_control_list=[
        {
            "group_name": "customer_analytics_engineers",
            "permission_level": "CAN_USE"
        },
        {
            "group_name": "customer_analytics_analysts",
            "permission_level": "CAN_USE"
        }
    ]
)
```

#### Step 4: Set Up Job Permissions
```python
# Job permission matrix for the project
job_permissions = {
    "daily_customer_etl": {
        "customer_analytics_admins": "IS_OWNER",
        "customer_analytics_engineers": "CAN_MANAGE", 
        "customer_analytics_analysts": "CAN_VIEW"
    },
    "weekly_customer_reports": {
        "customer_analytics_analysts": "IS_OWNER",
        "customer_analytics_viewers": "CAN_RUN"
    }
}

for job_name, permissions in job_permissions.items():
    # Find job by name (you'd need to implement this)
    job_id = find_job_by_name(job_name)
    
    # Apply permissions
    acl = []
    for group, permission in permissions.items():
        acl.append({
            "group_name": group,
            "permission_level": permission
        })
    
    workspace_client.permissions.update(
        request_object_type="jobs",
        request_object_id=job_id,
        access_control_list=acl
    )
```

## Common Permission Patterns

### 1. **Data Lake Zones Pattern**
```sql
-- Bronze Zone (Raw Data) - Data Engineers only
GRANT ALL PRIVILEGES ON SCHEMA datalake.bronze TO `data_engineers`;

-- Silver Zone (Cleaned Data) - Engineers and Scientists  
GRANT ALL PRIVILEGES ON SCHEMA datalake.silver TO `data_engineers`;
GRANT SELECT, CREATE TABLE ON SCHEMA datalake.silver TO `data_scientists`;

-- Gold Zone (Business Data) - Broader access
GRANT SELECT ON SCHEMA datalake.gold TO `business_analysts`;
GRANT SELECT ON SCHEMA datalake.gold TO `data_scientists`;
GRANT ALL PRIVILEGES ON SCHEMA datalake.gold TO `data_engineers`;

-- Sandbox - Individual user spaces
GRANT CREATE TABLE ON SCHEMA datalake.sandbox TO `all_users`;
```

### 2. **Department-Based Access**
```sql
-- Finance data access
GRANT ALL PRIVILEGES ON SCHEMA finance.* TO `finance_team`;
GRANT SELECT ON SCHEMA finance.reporting TO `executives`;

-- HR data access (most restrictive)
GRANT ALL PRIVILEGES ON SCHEMA hr.* TO `hr_team`;
GRANT SELECT ON SCHEMA hr.public_metrics TO `managers`;

-- Marketing data access
GRANT ALL PRIVILEGES ON SCHEMA marketing.* TO `marketing_team`;
GRANT SELECT ON SCHEMA marketing.campaigns TO `sales_team`;
```

### 3. **Environment Isolation**
```python
# Separate permissions by environment
environments = ["dev", "test", "staging", "prod"]

for env in environments:
    # Dev: Everyone can experiment
    if env == "dev":
        groups = ["all_developers", "data_teams"]
        permission = "CAN_MANAGE"
    
    # Test: QA and developers
    elif env == "test":
        groups = ["qa_team", "senior_developers"]
        permission = "CAN_MANAGE"
    
    # Staging: Limited access
    elif env == "staging":
        groups = ["release_managers", "qa_leads"]
        permission = "CAN_VIEW"
    
    # Prod: Very restricted
    elif env == "prod":
        groups = ["production_support"]
        permission = "CAN_VIEW"
    
    # Apply permissions for this environment
    for group in groups:
        # Set workspace permissions (simplified)
        set_workspace_permissions(f"{env}_workspace", group, permission)
```

## Security Best Practices

### 🔐 **Principle of Least Privilege**
Give users the minimum access they need to do their job.

```python
# Good: Specific access
GRANT SELECT ON TABLE sales.daily_summary TO `sales_analysts`;

# Bad: Too broad access  
GRANT ALL PRIVILEGES ON CATALOG * TO `sales_analysts`;
```

### 👥 **Use Groups, Not Individual Users**
```python
# Good: Group-based permissions
groups = {
    "marketing_analysts": ["alice@company.com", "bob@company.com"],
    "finance_team": ["charlie@company.com", "diana@company.com"]
}

# Bad: Individual user permissions (hard to maintain)
individual_users = ["alice@company.com", "bob@company.com", ...]
```

### 🔄 **Regular Access Reviews**
```python
# Script to review permissions quarterly
def quarterly_access_review():
    # Get all users and their permissions
    users = account_client.users.list()
    
    report = []
    for user in users:
        user_permissions = get_user_permissions(user.user_name)
        last_login = get_last_login(user.user_name)
        
        # Flag users who haven't logged in for 90+ days
        if last_login and (datetime.now() - last_login).days > 90:
            report.append({
                "user": user.user_name,
                "last_login": last_login,
                "permissions": user_permissions,
                "action": "REVIEW_FOR_REMOVAL"
            })
    
    return report

# Run quarterly
review_results = quarterly_access_review()
```

### 🔍 **Audit Logging**
```python
# Monitor permission changes
def setup_permission_monitoring():
    # Enable audit logs
    workspace_client.workspace_conf.set_status(
        name="enableAuditLog",
        value="true"
    )
    
    # Set up log analysis job
    audit_job = {
        "name": "Permission Change Monitoring",
        "tasks": [{
            "task_key": "analyze_permission_changes",
            "notebook_task": {
                "notebook_path": "/Security/audit_permission_changes"
            }
        }],
        "schedule": {
            "quartz_cron_expression": "0 0 9 * * ?",  # Daily at 9 AM
            "timezone_id": "UTC"
        }
    }
```

### 🛡️ **Data Classification and Labeling**
```sql
-- Tag tables with sensitivity levels
ALTER TABLE customer_data SET TBLPROPERTIES (
    'classification' = 'PII',
    'retention_period' = '7_years',
    'access_level' = 'restricted'
);

ALTER TABLE product_catalog SET TBLPROPERTIES (
    'classification' = 'public',
    'access_level' = 'open'
);

-- Use tags in permission grants
GRANT SELECT ON TABLE customer_data TO `pii_authorized_users`;
GRANT SELECT ON TABLE product_catalog TO `all_users`;
```

## Monitoring Permissions

### 1. **Permission Reports**
```python
# Generate permission summary report
def generate_permission_report():
    report = {
        "users": [],
        "groups": [], 
        "objects": []
    }
    
    # Get all users and their access
    users = account_client.users.list()
    for user in users:
        user_info = {
            "email": user.user_name,
            "active": user.active,
            "groups": get_user_groups(user.id),
            "last_login": get_last_login(user.user_name),
            "workspace_access": get_workspace_permissions(user.user_name)
        }
        report["users"].append(user_info)
    
    return report
```

### 2. **Automated Compliance Checks**
```python
# Check for compliance violations
def compliance_check():
    violations = []
    
    # Check 1: No individual user should have admin access
    admins = get_workspace_admins()
    for admin in admins:
        if not is_in_admin_group(admin):
            violations.append({
                "type": "INDIVIDUAL_ADMIN_ACCESS",
                "user": admin,
                "recommendation": "Move to admin group"
            })
    
    # Check 2: Contractors should not have permanent access
    contractors = get_users_by_domain("contractor.com")
    for contractor in contractors:
        access_duration = get_access_duration(contractor)
        if access_duration > 90:  # 90 days
            violations.append({
                "type": "LONG_TERM_CONTRACTOR_ACCESS", 
                "user": contractor,
                "recommendation": "Review and potentially revoke"
            })
    
    return violations
```

## Integration Examples

### 1. **SAML/SSO Integration**
```python
# Configure SAML groups mapping
saml_config = {
    "group_mapping": {
        "AD_DataEngineers": "data_engineers",
        "AD_DataScientists": "data_scientists", 
        "AD_BusinessAnalysts": "business_analysts"
    },
    "user_attributes": {
        "department": "http://schemas.company.com/department",
        "cost_center": "http://schemas.company.com/cost_center"
    }
}
```

### 2. **API-Based Permission Management**
```python
# Automate user onboarding
def onboard_new_user(email, department, role):
    # Create user
    user = account_client.users.create(
        user_name=email,
        active=True
    )
    
    # Determine groups based on department and role
    groups = determine_groups(department, role)
    
    # Add to appropriate groups
    for group in groups:
        account_client.groups.patch(
            id=group,
            operations=[{
                "op": "add",
                "path": "members", 
                "value": [{"value": email}]
            }]
        )
    
    # Send welcome email with access details
    send_welcome_email(email, groups)
    
    return user

# Example usage
onboard_new_user(
    email="new.hire@company.com",
    department="marketing", 
    role="analyst"
)
```

## Troubleshooting Common Issues

### ❌ **"Access Denied" Errors**

#### Problem: User can't access a notebook
**Solution Steps:**
1. Check if user has workspace access
2. Verify notebook permissions
3. Check if notebook is in accessible folder
4. Ensure user is in correct groups

```python
# Debugging script
def debug_access_issue(user_email, object_path):
    print(f"Debugging access for {user_email} to {object_path}")
    
    # Check workspace access
    workspace_perms = get_workspace_permissions(user_email)
    print(f"Workspace access: {workspace_perms}")
    
    # Check object permissions  
    object_perms = get_object_permissions(object_path, user_email)
    print(f"Object access: {object_perms}")
    
    # Check group memberships
    groups = get_user_groups(user_email)
    print(f"User groups: {groups}")
    
    # Recommendations
    if not workspace_perms:
        print("→ Grant workspace access first")
    elif not object_perms:
        print("→ Grant object-level permissions")
```

#### Problem: User can see table but can't query it
**Solution:**
```sql
-- Check current permissions
SHOW GRANTS ON TABLE my_table;

-- Grant necessary permissions
GRANT SELECT ON TABLE my_table TO `user_group`;
GRANT USE SCHEMA ON SCHEMA my_schema TO `user_group`;
GRANT USE CATALOG ON CATALOG my_catalog TO `user_group`;
```

### ❌ **Performance Issues with Permissions**

#### Problem: Permission checks are slow
**Solutions:**
1. Use groups instead of individual permissions
2. Optimize permission inheritance
3. Cache permission results

```python
# Optimize permission structure
# Bad: Many individual permissions
for user in users:
    grant_permission(table, user, "SELECT")

# Good: Group-based permissions  
create_group("table_readers", users)
grant_permission(table, "table_readers", "SELECT")
```

## Permission Automation

### 1. **Auto-Remove Inactive Users**
```python
# Job to clean up inactive users
def cleanup_inactive_users():
    inactive_threshold = 90  # days
    
    users = account_client.users.list()
    for user in users:
        last_login = get_last_login(user.user_name)
        
        if last_login:
            days_inactive = (datetime.now() - last_login).days
            
            if days_inactive > inactive_threshold:
                # Disable user instead of deleting
                account_client.users.patch(
                    id=user.id,
                    operations=[{
                        "op": "replace",
                        "path": "active",
                        "value": False
                    }]
                )
                
                print(f"Disabled inactive user: {user.user_name}")
```

### 2. **Dynamic Group Management**
```python
# Auto-assign groups based on user attributes
def auto_assign_groups(user_email):
    # Get user details from HR system (example)
    user_info = get_hr_info(user_email)
    
    groups_to_assign = []
    
    # Assign based on department
    if user_info["department"] == "Engineering":
        groups_to_assign.append("engineers")
        
        if user_info["team"] == "Data":
            groups_to_assign.append("data_engineers")
    
    elif user_info["department"] == "Marketing":
        groups_to_assign.append("marketing_team")
        
        if user_info["seniority"] == "Senior":
