# **Nexus Analytics – Fabric Orchestration & Lakehouse Documentation (Dev)**

## **1. High-level Overview**

Nexus Analytics is a Fabric-based analytics which provides insights into usage,quality,cost and performance which impact how good the bot is being recieved and its future success. The analytics system built on a **medallion architecture**:

* **Bronze** – raw ingestion stored physically in **`LH_SOURCE_DATA`** which are then used in **`LH_NEXUS`** as shortcuts
* **Silver** – cleansed and standardized entities inside **`LH_NEXUS`**
* **Gold** – analytics-ready dimensions + fact tables powering the Nexus Semantic Model

Fabric orchestration is handled using three pipelines in **(Dev) Nexus**:

* **PL_NEXUS_MASTER** – Master controller (logging, monitoring, alerts)
* **PL_NEXUS_LOAD** – Child orchestrator for Bronze → Silver → Gold ETL
* **PL_NEXUS_DEPT2GROUP_LOAD** - Used to load mapping of detailed department names to standardized department groups for reporting and analytics.

---

## **2. Lakehouses & Shortcuts**

### **2.1 Lakehouses**

#### **`LH_SOURCE_DATA` – (Dev) Source Data**

Stores Following tables :

* Worker Dimension - gold.dim_worker
* Cosmos History Logs and feeback - bronze.nexus_cosmos_history_logs

Above are present as shortcuts within `LH_NEXUS`


#### **`LH_NEXUS` – (Dev) Nexus**

* Contains `bronze.translation_data` which gets loaded from translation_department.xlsx
* Hosts **Silver** + **Gold** layers
* Contains the **Nexus Semantic Model (Power BI)**

---

### **2.2 Schemas / Naming Conventions**

* `bronze.*` – raw ingest from source systems
* `silver.*` – cleansed, standardized, conformed entities
* `gold.*` – dimensional model (facts + dimensions)
* `audit.*` – ETL logging & watermark tables (e.g., `audit.watermark_log`)

---

## **3. Pipelines**

---

## **3.1 `PL_NEXUS_MASTER`** – Top-Level Controller

**Purpose:** Reliable orchestration, logging, and alerting.

### **Pipeline Flow**

1. **LOG_PIPELINE_START**
   Records pipeline start-time, job-id, workspace, and trigger info.

2. **INVOKE_NEXUS_DATA_REFRESH (Execute Pipeline)**

   * Calls `PL_NEXUS_LOAD`
   * `waitOnCompletion = true` to ensure synchronous execution

3. **On Success → LOG_PIPELINE_SUCCESS**
   Writes completion record and optional metrics.

4. **On Failure → LOG_PIPELINE_FAILURE + SEND_FAILURE_EMAIL**

   * Logs failure details (activity, message, duration).
   * Sends email via Outlook with:

     * Failed activity name
     * Pipeline name
     * Error message from:

       ```
       @activity('INVOKE_NEXUS_DATA_REFRESH').error.Message
       ```
     * Run ID + time window

---

## **3.2 `PL_NEXUS_LOAD`** – Child ETL Pipeline

**Purpose:** Executes all ingestion + transformation notebooks.

### **Execution Steps**

1. **DEPT_TO_GROUP**

   * Department → group mapping reference → `bronze.translation_data`.

2. **BRONZE_RAW_HR_WORKER_INGEST**

   * Loads HR workers → `gold.dim_worker`.

3. **BRONZE_NEXUS_COSMOS_HISTORY_LOGS_INGEST**

   * Flattens mirrored Cosmos logs and Nexus feedback  → `bronze.nexus_cosmos_history_logs`.

4. **SILVER_USERS_CLEANSE**

   * Combines HR + AD → `silver.users`.
   * Produces `user_name_nk` natural key.

5. **SILVER_NEXUS_EVENT_LOGS_COSMOS**

   * Creates row-level events → `silver.nexus_event_logs`.
   * Incremental load driven by watermark.

6. **SILVER_DATE_POPULATE**

   * Creates date + time seeds:

     * `silver.date`
     * `silver.time_by_hour` *(new)*

7. **GOLD_DIM_USER_MODEL**

   * Builds `gold.dim_user` with surrogate `user_sk`.

8. **GOLD_DIM_DATE_MODEL**

* Builds `gold.dim_date` with surrogate `date_sk`.

9. **GOLD_DIM_TIME_MODEL** *(new)*

* Creates `gold.dim_time` (0–23 hour rows).
* Includes default row with `time_sk = -1`.

10. **GOLD_FACT_USER_EVENTS**

* Creates `gold.fact_agent_user_events` by joining:

  * `silver.tfm_nexus_event_logs`
  * `gold.dim_user`
  * `gold.dim_date`
  * `gold.dim_time`
* Implements MERGE on `(event_nk, event_type)`
* Updates watermark in `audit.watermark_log`

12. **Semantic Model Refresh**
    *`Nexus_Analytics` - semantic model
    *`Digital Nexus Analytics` - report

---

## **4. Notebooks by Layer**

---

## **4.1 Bronze Layer (Physical Storage: `LH_SOURCE_DATA`)**

### **NB_BRONZE_NEXUS_COSMOS_HISTORY_LOGS_INGEST**

* Mirrored Cosmos → flattened → `bronze.nexus_cosmos_history_logs`.
* Handles schema drift and stores ETL timestamps.
---

## **4.2 Silver Layer (`LH_NEXUS`)**

### **NB_SILVER_USERS_CLEANSE**

* Inputs:  gold.dim_worker + bronze.translation_data
* Outputs: `silver.tfm_users`
* Standardizes:

  * Emails, names, dept, groups
  * Generates `user_name_nk`

### **NB_SILVER_NEXUS_EVENT_LOGS_COSMOS**

* Input: Bronze Cosmos logs
* Outputs: `silver.tfm_nexus_event_logs`
* Logic:

  * Watermark-based incremental ingest
  * Generates `event_nk`
  * Normalizes all attributes (timestamp, names, event type, etc.)
  * Preserves token telemetry and feedback fields

### **NB_SILVER_DATE_POPULATE**

* Generates:

  * `silver.date` → date spine
  * `silver.time_by_hour` → time spine (0–23)

---

## **4.3 Gold Layer (`LH_NEXUS`)**

### **NB_GOLD_DIM_DATE_MODEL**

* Builds `gold.dim_date`
* Assigns `date_sk`
* Adds default row (`date_sk = -1`)

### **NB_GOLD_DIM_TIME_MODEL** *(new)*

* Builds `gold.dim_time`
* Assigns `time_sk = HourNumber`
* Adds default row (`time_sk = -1`)

### **NB_GOLD_DIM_USER_MODEL**

* Builds `gold.dim_user`
* Assigns `user_sk`

### **NB_GOLD_FACT_USER_EVENTS_MODEL**

* Joins Silver event logs to dimensions:

  * `dim_date`
  * `dim_time`
  * `dim_user`
* Writes `gold.fact_agent_user_events` using MERGE
* Updates watermark

### **NB_BRONZE_HR_WORKER_INGEST**

* Input: DWH.DIM_HR_WORKER 
* Source : Synapse Prod EDA
* Outputs: `gold.dim_worker` present as shortcuts in gold schema of **LH_NEXUS**

---

## **5. Watermarks, Logging & Error Handling**

### **Watermark Mechanism**

Stored in:

```
audit.watermark_log
```

Pattern:

1. Read `MAX(watermark)` for dataset
2. Filter new rows from Bronze/Silver
3. After MERGE → write new watermark

### **Logging**

* Start → LOG_PIPELINE_START
* Success → LOG_PIPELINE_SUCCESS
* Failure → LOG_PIPELINE_FAILURE

### **Email Alerts**

Triggered using:

```
@activity('INVOKE_NEXUS_DATA_REFRESH').error.Message
```

---

## **6. Semantic Model / Power BI**

### **Tables Used**

* `gold.dim_user`
* `gold.dim_date`
* `gold.dim_time`
* `gold.fact_agent_user_events`

### **Fact → Dimension Joins**

```sql
fact_agent_user_events.user_sk = dim_user.user_sk
fact_agent_user_events.date_sk = dim_date.date_sk
fact_agent_user_events.time_sk = dim_time.time_sk
```

### **Purpose**

analyze conversation volumes, user and agent activity, channels, and trends over time
---
