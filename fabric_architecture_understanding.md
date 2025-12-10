# Microsoft Fabric Enterprise Architecture Diagram

![Fabric Architecture Diagram](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Architecture.png)

## Detailed Notes for Each Component

### Data Sources

1. **SQL Database**
   - **Why needed**: Enterprise systems (ERP, CRM, HR, Finance) primarily store transactional data in SQL.
   - **Purpose**: Enables historical + operational reporting once ingested.

2. **API**
   - **Why needed**: Many SaaS platforms (Salesforce, Marketo, ServiceNow) expose REST/GraphQL APIs.
   - **Purpose**: Needed to integrate external or partner data into Fabric.

3. **IoT / Event Hub**
   - **Why needed**: Provides **real-time telemetry and events** from devices, sensors, logs.
   - **Purpose**: Needed for streaming use cases like predictive maintenance, clickstream analytics.

4. **ADLS Gen2**
   - **Why needed**: Many organizations already invested in Azure Data Lake.
   - **Purpose**: Fabric shortcuts avoid data duplication by reading ADLS data **in place**.

### Ingestion & Processing

1. **Data Pipeline**
   - **Why needed**: Low-code ETL/ELT orchestration.
   - **Purpose**: Needed for scheduled, repeatable batch ingestion.
   - **Example**: Load ERP daily sales data into Lakehouse Bronze.

2. **Notebook (PySpark)**
   - **Why needed**: **Complex transformations, ML prep, schema alignment**.
   - **Purpose**: Cleansing JSON logs, applying NLP before storing in Silver/Gold.

3. **Event Stream**
   - **Why needed**: **Real-time ingestion** from IoT/Event Hub.
   - **Purpose**: Lets you define outputs → Lakehouse, KQL DB, or push to APIs.
   - **Example**: Streaming IoT temperature data into Bronze Lakehouse.

### OneLake & Lakehouse Layers

1. **OneLake**
   - **Why needed**: **Single source of truth** across the organization.
   - **Purpose**: Eliminates siloed storage (instead of each team having their own data lake).

2. **Lakehouse**
   - **Why needed**: Unifies **data lake + warehouse style access**.
   - **Purpose**: Stores Delta tables with Spark + SQL support.

3. **Landing → Bronze → Silver → Gold**
   - **Landing**: Needed for auditability (raw data retained for reprocessing).
   - **Bronze**: Needed to convert into standard format (Delta).
   - **Silver**: Needed for cleansed, standardized enterprise-wide data.
   - **Gold**: Needed for **business-ready KPIs/fact-dim models**.
   - **Purpose**: These layers enforce **data quality, trust, and separation of concerns**.

4. **Shortcuts**
   - **Why needed**: **Virtualize external data** without moving it.
   - **Example**: Use ADLS Gen2 logs directly in Fabric.

### Other Fabric Items

1. **SQL Endpoint**
   - **Why needed**: **SQL-based analysts** and Power BI.
   - **Purpose**: Provides an easy relational view of Lakehouse tables.

2. **Warehouse**
   - **Why needed**: **Governed, relational analytics** (financial reporting, RLS/OLS, constraints).
   - **Purpose**: Provides dedicated **T-SQL performance** and schema enforcement.
   - **Example**: CFO dashboard should source from Warehouse, not flexible Gold tables.

3. **Dataflow Gen2**
   - **Why needed**: **Citizen developers & analysts** who aren't Spark/SQL experts.
   - **Purpose**: Provides **low-code/no-code transformations** (Power Query-like).
   - **Benefits**: Reusable across datasets, semantic models, reports.
   - **Example**: Business user creating a quick "Customer Segmentation" data prep without waiting on data engineers.
   - **Difference vs Semantic Model**:
     - **Dataflow Gen2** = pre-processing & shaping raw data.
     - **Semantic Model** = business semantic layer (facts/dims, measures, relationships) optimized for Power BI.
     - **Both needed** → Dataflow Gen2 feeds curated data into Semantic Models.

4. **Mirrored DB**
   - **Why needed**: Business wants **near real-time replication** of operational DBs into Fabric.
   - **Example**: SQL Server orders mirrored in Fabric within minutes for analytics.

### Consumption

1. **Power BI**
   - **Why needed**: **Visual storytelling**.
   - **Example**: Executive dashboards, self-service reporting.

2. **Data Analyst**
   - **Why needed**: Ad-hoc exploration, SQL queries, hypothesis testing.

3. **API / AI Apps**
   - **Why needed**: Modern apps and AI agents consume structured data.
   - **Example**: AI Copilot fetching customer churn predictions from Fabric tables.

### Governance, Security, Monitoring

1. **Microsoft Purview**
   - **Why needed**: **Discoverability, compliance, lineage, glossary**.
   - **Example**: Which reports consume PII data? Purview answers this.

2. **Microsoft Entra ID**
   - **Why needed**: **Identity & access control**.
   - **Example**: Only Finance group can access Gold finance tables.

3. **Azure Key Vault**
   - **Why needed**: **Securing credentials, keys, API secrets** used in pipelines.

4. **Azure Monitor**
   - **Why needed**: **Operational monitoring & alerts**.
   - **Example**: Alert if pipeline fails or refresh latency > SLA.

5. **Azure DevOps & GitHub**
   - **Why needed**: **CI/CD of Fabric assets**.
   - **Example**: Promote tested semantic models from Dev → Test → Prod.

6. **Azure Policy**
   - **Why needed**: **Compliance enforcement**.
   - **Example**: Prevent data from being stored outside specific regions.

### Why Dataflow Gen2 vs Semantic Model Both Are Needed

- **Dataflow Gen2** = Self-service **data prep tool** (cleaning, merging, shaping raw data).
- **Semantic Model** = Enterprise-wide **semantic/business layer** (measures, relationships, RLS).
- Dataflows can feed semantic models, but they **don't replace them**.
- **Example**:
  - A marketing analyst uses **Dataflow Gen2** to merge Salesforce + Marketo data.
  - That output feeds a **Semantic Model** that defines business KPIs like "Customer Lifetime Value."