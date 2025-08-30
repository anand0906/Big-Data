# Databricks Storage System — Managed vs External (A-to-Z Study Guide)

> A practical, step‑by‑step guide with examples and end‑to‑end flows for **Managed** and **External** storage in Databricks (Unity Catalog and legacy Hive Metastore), organized for study and implementation.

---

## 0) Big Picture

**Two ways to persist tables/files in Databricks:**

1. **Managed** – Databricks controls where files live. You just create tables; storage paths are derived from configured managed locations.
2. **External** – You control the cloud path (S3/ADLS/GCS). Tables point to that path; Databricks doesn’t move data.

**Two governance contexts:**

* **Unity Catalog (UC)** – Modern, account-level governance. Preferred for new work. Has *metastore* → *catalog* → *schema* → *table* hierarchy; supports **managed locations**, **external locations**, **volumes**, **UNDROP**, fine-grained permissions.
* **Legacy Hive Metastore (workspace-local)** – The older, workspace-scoped metastore, exposed as catalog `hive_metastore` (schema `default`).

---

## 1) Key Terms (Cheat Sheet)

* **Metastore (UC):** Account-level governance container per region. Holds catalogs, external locations, storage credentials, etc.
* **Catalog (UC):** Top-level namespace under a UC metastore. Example: `prod`, `sandbox`, `marketing`.
* **Schema:** Logical grouping inside a catalog. Example: `prod.analytics`.
* **Managed Location:** A cloud path where **managed** objects in that scope (metastore/catalog/schema) are physically stored.
* **Storage Credential (UC):** Abstraction over a cloud identity (AWS IAM role, Azure SPN/MI, GCP SA) used by UC to access cloud storage.
* **External Location (UC):** A secure, named mapping of a **URL (s3:// / abfss\:// / gs\://)** + **Storage Credential**; used by **external** tables/volumes.
* **Mount Point:** A DBFS mount (e.g., `/mnt/raw`) to a cloud path using credentials configured at the cluster/workspace level. Useful but less governed than UC external locations.

---

## 2) Managed Storage

### 2.1 Legacy Default Metastore (Hive Metastore)

* **What it is:** The older, workspace-local metastore surfaced as catalog `hive_metastore`. Default schema is `default`.
* **Default warehouse path:** Typically `dbfs:/user/hive/warehouse` (can be changed by admin).
* **Managed table behavior:** Creating a table without an explicit `LOCATION` stores data under the warehouse path.

**Example — Managed Table (Legacy Hive Metastore)**

```sql
-- Target: hive_metastore.default
CREATE TABLE hive_metastore.default.sales (
  id INT, amount DOUBLE, ts TIMESTAMP
);

INSERT INTO hive_metastore.default.sales VALUES (1, 100.5, current_timestamp());
```

Files will land under something like:

```
dbfs:/user/hive/warehouse/sales
```

**PySpark write (managed):**

```python
df.write.mode("overwrite").saveAsTable("hive_metastore.default.sales")
```

> **When to use:** For legacy workloads or when UC isn’t enabled yet. Prefer UC for new implementations.

---

### 2.2 Unity Catalog (UC) Managed

In UC, **managed storage** is derived from the nearest managed location:

1. Schema managed location → 2) Catalog managed location → 3) Metastore managed location (default)

#### 2.2.1 Creating a UC Metastore (Account Admin)

High-level steps:

1. In **Account Console → Data**: **Create Metastore** in a region.
2. Provide a **default storage root** (e.g., `s3://dbrx-metastore-root/` or `abfss://ucroot@acct.dfs.core.windows.net/uc/`).
3. Create/attach a **Storage Credential** (IAM role / SPN / SA) with permissions on that storage root.
4. **Assign the metastore** to your workspace(s) and set as default.

> Result: UC is active; you’ll see catalogs like `main` (often created by default). Managed tables now use UC locations unless you explicitly target `hive_metastore`.

#### 2.2.2 Creating a Catalog (with optional managed location)

```sql
-- As a metastore/catal og owner
CREATE CATALOG prod
  MANAGED LOCATION 's3://company-datalake/prod/managed/';

GRANT USAGE ON CATALOG prod TO `data_engineers`;
```

> If `MANAGED LOCATION` is omitted, the catalog inherits from the metastore default root.

#### 2.2.3 Creating a Schema (with optional managed location)

```sql
CREATE SCHEMA prod.analytics
  MANAGED LOCATION 's3://company-datalake/prod/analytics/';

GRANT USAGE ON SCHEMA prod.analytics TO `analysts`;
```

#### 2.2.4 Creating Managed Tables (no LOCATION clause)

```sql
USE CATALOG prod;
USE SCHEMA analytics;

CREATE TABLE orders (
  order_id BIGINT,
  customer_id BIGINT,
  amount DECIMAL(12,2),
  order_ts TIMESTAMP
);
```

Physical layout (example):

```
s3://company-datalake/prod/analytics/tables/orders
```

*(Exact folder naming may vary; Databricks manages placement under the configured managed location.)*

**PySpark (managed):**

```python
df.write.format("delta").mode("append").saveAsTable("prod.analytics.orders")
```

#### 2.2.5 How the **managed location resolution** works (flow)

```
When you CREATE TABLE without LOCATION:
  if schema has MANAGED LOCATION → use it
  else if catalog has MANAGED LOCATION → use it
  else → use metastore default root
```

---

## 3) Assigning Custom Locations (UC)

You can control storage placement at three levels. Use **managed** for fully-governed tables and **external** for bring-your-own path.

### 3.1 Managed Location at Catalog Level

```sql
ALTER CATALOG prod SET MANAGED LOCATION 'abfss://lake@acct.dfs.core.windows.net/prod/managed/';
```

### 3.2 Managed Location at Schema Level

```sql
ALTER SCHEMA prod.analytics SET MANAGED LOCATION 'gs://lake-prod/analytics/managed/';
```

### 3.3 Table-Level (External) Explicit Location

If you specify `LOCATION`, you’re creating an **external table** (not managed) even inside UC.

```sql
CREATE TABLE prod.analytics.clickstream (
  user_id BIGINT,
  page STRING,
  ts TIMESTAMP
)
USING DELTA
LOCATION 's3://company-raw/clickstream/delta/';
```

---

## 4) Flow of Storing Data (Managed Scenarios)

### 4.1 UC Managed Table (SQL)

```
Actor: Data Engineer
Context: UC metastore initialized; catalog/schema managed locations set
Steps:
1) USE CATALOG prod; USE SCHEMA analytics;
2) CREATE TABLE orders (...); -- no LOCATION
3) INSERT INTO orders ...; or df.write.saveAsTable('prod.analytics.orders')
Outcome: Files stored under schema managed location (or inherited), ACLs enforced by UC.
```

### 4.2 Legacy Hive Managed Table (SQL)

```
Actor: Data Engineer
Context: Using hive_metastore
Steps:
1) CREATE TABLE hive_metastore.default.events (...);
2) INSERT INTO ...
Outcome: Files stored under dbfs:/user/hive/warehouse/... (or configured warehouse path)
```

---

## 5) Dropping and Undropping in Unity Catalog

### 5.1 DROP

```sql
DROP TABLE prod.analytics.orders;     -- moves to UC trash (soft delete)
DROP VIEW  prod.analytics.orders_vw;
DROP SCHEMA prod.analytics;           -- must be empty or use CASCADE
DROP CATALOG prod;                    -- must be empty or use CASCADE
```

### 5.2 UNDROP (Restore recently dropped objects)

```sql
UNDROP TABLE prod.analytics.orders;   -- restores last dropped version of the object
UNDROP SCHEMA prod.analytics;
UNDROP CATALOG prod;
```

**Notes:**

* UNDROP is supported for UC objects (tables, views, schemas, catalogs) as long as they’re still in UC’s trash retention window (set by admins). It restores **metadata** links; underlying Delta data is unchanged unless it was physically removed.
* For data-level rollback, use Delta Lake time travel:

```sql
RESTORE TABLE prod.analytics.orders TO VERSION AS OF 123;  -- or TO TIMESTAMP AS OF '2025-08-01T12:00:00Z'
```

---

## 6) External Storage

There are two main ways in Databricks:

1. **UC External Locations (Recommended)** – governed, auditable, permissionable.
2. **Mount Points (DBFS mounts)** – convenient, but not governed by UC policies; use with care in UC environments.

### 6.1 UC External Locations (End‑to‑End)

**Step A — Create Storage Credential**

* **AWS:** An IAM role with trust to Databricks; grant `s3:*` as needed on target buckets/prefixes.
* **Azure:** A Service Principal (or Managed Identity) with ACLs/role assignments (e.g., `Storage Blob Data Contributor`) on the storage account/container.
* **GCP:** A Service Account with `Storage Object Admin`/appropriate roles on the bucket.

```sql
-- Example (abstract):
CREATE STORAGE CREDENTIAL sc_data
  WITH ROLE 'arn:aws:iam::123456789012:role/databricks-uc-access';
-- or
CREATE STORAGE CREDENTIAL sc_data
  WITH AZURE_SERVICE_PRINCIPAL (CLIENT_ID 'xxx', CLIENT_SECRET '***')
  TENANT_ID 'yyy';
```

**Step B — Create External Location**

```sql
CREATE EXTERNAL LOCATION ext_raw
  URL 's3://company-raw/'
  WITH STORAGE CREDENTIAL sc_data
  COMMENT 'Raw zone for ingestion';

GRANT USAGE ON EXTERNAL LOCATION ext_raw TO `ingestion_team`;
```

**Step C — Create External Tables pointing at paths under that location**

```sql
-- Explicit LOCATION under ext_raw
CREATE TABLE prod.raw.customers
USING DELTA
LOCATION 's3://company-raw/customers/delta/';

-- Or create first by writing files, then register:
CREATE TABLE prod.raw.clicks
USING DELTA
LOCATION 's3://company-raw/clicks/';
```

**PySpark example (external):**

```python
(df
 .write
 .format("delta")
 .mode("overwrite")
 .option("overwriteSchema", "true")
 .save("s3://company-raw/customers/delta/")  # write files
)

spark.sql("""
  CREATE TABLE prod.raw.customers
  USING DELTA
  LOCATION 's3://company-raw/customers/delta/'
""")
```

**Flow (UC External)**

```
Actor: Metastore Admin + Data Engineer
1) Admin: CREATE STORAGE CREDENTIAL → CREATE EXTERNAL LOCATION → GRANT USAGE
2) Engineer: CREATE TABLE ... USING DELTA LOCATION '<path-under-external-location>'
Outcome: UC enforces access; files remain at your specified path.
```

**Pros:** Governance, fine-grained permissions, multi-workspace, audit.
**Cons:** Initial setup requires admin work (credentials, permissions).

---

### 6.2 Mount Points (DBFS) — Alternative

* **What:** Map a cloud path to DBFS (e.g., `/mnt/raw`) using `dbutils.fs.mount`.
* **When:** Quick access for notebooks/jobs; legacy patterns; reading from tools that expect local-like paths.
* **Caution with UC:** Mounts are **not** represented as UC external locations. Access is governed by workspace/cluster credentials, not UC. For UC-governed tables, prefer **External Locations** instead of mounts.

**Mount Examples**

*Azure ADLS Gen2*

```python
configs = {
  "fs.azure.account.auth.type": "OAuth",
  "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
  "fs.azure.account.oauth2.client.id": "<app-id>",
  "fs.azure.account.oauth2.client.secret": "<secret>",
  "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<tenant-id>/oauth2/token"
}

dbutils.fs.mount(
  source = "abfss://raw@acct.dfs.core.windows.net/",
  mount_point = "/mnt/raw",
  extra_configs = configs
)
```

*AWS S3*

```python
dbutils.fs.mount(
  source = "s3a://company-raw/",
  mount_point = "/mnt/raw",
  extra_configs = {"fs.s3a.aws.credentials.provider": "com.amazonaws.auth.InstanceProfileCredentialsProvider"}
)
```

**Use mounted paths**

```python
spark.read.format("delta").load("/mnt/raw/customers/delta/")
```

**Register external table from a mount (works, but not UC-recommended):**

```sql
CREATE TABLE hive_metastore.default.customers
USING DELTA
LOCATION '/mnt/raw/customers/delta/';
```

**Flow (Mounts)**

```
Actor: Workspace Admin + Data Engineer
1) Admin/Engineer: Configure secrets/credentials → dbutils.fs.mount
2) Engineer: Read/write using /mnt/... paths → Optional: CREATE TABLE ... LOCATION '/mnt/...'
Outcome: Convenient access; governance via workspace/cluster, not UC external location policies.
```

---

## 7) Managed vs External — Quick Comparison

| Aspect           | Managed (UC)                            | External (UC External Location)          | Legacy Hive Managed   | Mount-Based External |
| ---------------- | --------------------------------------- | ---------------------------------------- | --------------------- | -------------------- |
| Who decides path | UC (managed location)                   | You (LOCATION under External Location)   | Hive warehouse path   | You (mount source)   |
| Governance       | Full UC (best)                          | Full UC (best)                           | Workspace-level       | Weak UC alignment    |
| Portability      | High across workspaces (same metastore) | High                                     | Low (workspace-local) | Medium               |
| Setup effort     | Low/Medium                              | Medium (needs credential + ext location) | Low                   | Medium (mount setup) |
| Recommended      | ✅                                       | ✅                                        | Legacy only           | For special cases    |

---

## 8) End‑to‑End Implementation Playbooks

### 8.1 UC Managed Tables — New Project

```
Roles: Account Admin, Metastore Admin, Data Engineer

Admin:
1) Create UC Metastore with default storage root
2) Create catalog prod (optional: MANAGED LOCATION)
3) Create schema prod.analytics (optional: MANAGED LOCATION)
4) Grant privileges to teams

Engineer:
5) USE CATALOG prod; USE SCHEMA analytics;
6) CREATE TABLE prod.analytics.orders (...)  -- no LOCATION
7) Load data (INSERT / COPY INTO / df.write.saveAsTable)
```

**Example**

```sql
CREATE CATALOG prod MANAGED LOCATION 's3://lake/prod/managed/';
CREATE SCHEMA prod.analytics MANAGED LOCATION 's3://lake/prod/analytics/';
GRANT USAGE ON CATALOG prod TO `data_engineers`;
GRANT USAGE, CREATE ON SCHEMA prod.analytics TO `data_engineers`;

CREATE TABLE prod.analytics.orders (
  order_id BIGINT, amount DECIMAL(12,2), order_ts TIMESTAMP
);
```

### 8.2 UC External Tables — Curated Zone on BYO Path

```
Roles: Metastore Admin, Data Engineer

Admin:
1) CREATE STORAGE CREDENTIAL sc_curated
2) CREATE EXTERNAL LOCATION ext_curated URL 'abfss://curated@acct.dfs.core.windows.net/' WITH STORAGE CREDENTIAL sc_curated
3) GRANT USAGE ON EXTERNAL LOCATION ext_curated TO `curation_team`

Engineer:
4) Write Delta files to 'abfss://curated/.../delta/'
5) CREATE TABLE catalog.schema.table USING DELTA LOCATION 'abfss://curated/.../delta/'
```

**Example**

```sql
CREATE EXTERNAL LOCATION ext_curated
  URL 'abfss://curated@acct.dfs.core.windows.net/'
  WITH STORAGE CREDENTIAL sc_curated;

CREATE TABLE prod.curated.sales USING DELTA
LOCATION 'abfss://curated@acct.dfs.core.windows.net/sales/delta/';
```

### 8.3 Legacy Hive Managed — Quick Prototype

```sql
CREATE TABLE hive_metastore.default.tmp_orders (
  id BIGINT, amt DOUBLE
);
INSERT INTO hive_metastore.default.tmp_orders VALUES (1, 29.99);
```

### 8.4 Mounts — Read Raw Files and Register as External Table

```python
# After mounting /mnt/raw → abfss://raw@acct.dfs.core.windows.net/
df = (spark.read.format("csv").option("header", True)
      .load("/mnt/raw/ingest/2025-08-30/*.csv"))

df.write.format("delta").mode("overwrite").save("/mnt/raw/customers/delta/")

spark.sql("""
  CREATE TABLE hive_metastore.default.customers
  USING DELTA
  LOCATION '/mnt/raw/customers/delta/'
""")
```

---

## 9) Operational Notes & Best Practices

* **Prefer UC** for new work. Use **managed** for ease, **external** for BYO paths/data zones.
* **Name clearly:** `{env}.{domain}` catalogs (e.g., `prod.finance`), schemas like `{team}` or `{subject}`.
* **Set managed locations** at **catalog/schema** to keep data organized by domain.
* **Use External Locations** instead of mounts for governed external tables.
* **Permissions:**

  * `GRANT USAGE ON CATALOG/SCHEMA` to allow discovery.
  * `GRANT SELECT, MODIFY` on tables to control read/write.
  * `GRANT READ FILES/WRITE FILES` on external locations as needed.
* **Data lifecycle:** For external tables, you own the path. Dropping the table **does not** delete files. For managed tables, Databricks manages files with table lifecycle (subject to retention/trash).
* **Delta safety:** Use `VACUUM` conservatively and keep retention policies compliant with your recovery needs. Combine **UNDROP** (metadata) with **RESTORE** (data) for robust recovery.
* **Migrations:** To move from legacy Hive to UC: create UC catalog/schema, set permissions, then `CREATE TABLE ... USING DELTA LOCATION '<legacy-path>'` under UC, or `CLONE` tables where appropriate.

---

## 10) Frequently Used Snippets

**Create table from files (managed target)**

```sql
USE CATALOG prod; USE SCHEMA analytics;
CREATE TABLE orders USING DELTA AS
SELECT * FROM delta.`s3://company-landing/orders/`;
```

**Ingest files into managed table**

```python
spark.sql("USE CATALOG prod; USE SCHEMA analytics;")
(spark.read.format("json").load("s3://landing/orders/2025-08-30/")
     .write.format("delta").mode("append")
     .saveAsTable("prod.analytics.orders"))
```

**Register existing Delta folder as external table (UC)**

```sql
CREATE TABLE prod.curated.inventory
USING DELTA
LOCATION 'gs://lake-curated/inventory/delta/';
```

**Change managed location (catalog)**

```sql
ALTER CATALOG prod SET MANAGED LOCATION 's3://new-root/prod/';
```

**Drop & Undrop**

```sql
DROP TABLE prod.analytics.orders;
UNDROP TABLE prod.analytics.orders;
```

---

## 11) Troubleshooting & Pitfalls

* **Permission denied on external location:** Ensure the storage credential has cloud‑level access and you’ve **GRANTED USAGE** on the external location to the user/group.
* **Table created as external unintentionally:** If you specified `LOCATION`, it’s external. Remove `LOCATION` for managed tables.
* **Mounts + UC confusion:** Tables created over `/mnt/...` paths are not governed by UC external location policies. Prefer UC external locations.
* **Path collisions:** For managed locations, avoid manually writing files into the same folder where UC is managing tables.
* **Time travel vs undrop:** `UNDROP` restores metadata; `RESTORE` reverts table contents; both may be needed after accidental changes.

---

## 12) Study Flow (Recommended Reading Order)

1. **Big Picture & Key Terms** (Sections 0–1)
2. **Managed Storage (UC)** (Sections 2.2–3–4)
3. **Drop/Undrop & Recovery** (Section 5)
4. **External Storage via External Locations** (Section 6.1)
5. **Mount Points and Caveats** (Section 6.2)
6. **Compare & Choose** (Section 7)
7. **Playbooks & Snippets** (Sections 8–10)

---

### Quick Visual Flows (ASCII)

**UC Managed Create Table**

```
CREATE TABLE (no LOCATION)
   └─→ UC resolves managed location (schema → catalog → metastore)
        └─→ Files written under resolved path
             └─→ Permissions via UC
```

**UC External Table**

```
CREATE STORAGE CREDENTIAL → CREATE EXTERNAL LOCATION → GRANT USAGE
   └─→ CREATE TABLE ... LOCATION 's3|abfss|gs://path-under-external-location'
        └─→ Files remain at your path; UC governs access
```

**Mount-based External (not UC‑governed)**

```
dbutils.fs.mount('/mnt/raw' → cloud path)
   └─→ Read/Write /mnt/raw/... in code
        └─→ Optional: CREATE TABLE ... LOCATION '/mnt/raw/...'
```

---

> You now have a complete, implementation-ready reference for **Managed vs External** storage in Databricks, covering default/UC metastores, catalogs, managed/external locations, mounts, and drop/undrop flows.


```
Prompt
----

Databricks Storage System.
-------------------------
Types : Managed and External
Managed :
--------
defualt Metastore
Creating Custom Metastore
Unity Catalog
Creating Catalog
Assigning custom location to Catalog, schema, table
Flow of storing data in each scenario
Drop and Undrop tables in unity catalog

External :
---------
Ways to store 
external location
mount point



Explain in detailed manner with examples and flow for each case to implement.
Give it in very organzed manner to study in best flow

```
