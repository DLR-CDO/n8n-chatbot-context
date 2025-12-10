**(Dev) Source Data workspace**

(Dev) Source Data is our **central ingestion workspace** in Microsoft Fabric.  
It is the **first landing point** for all raw and mirrored source systems (Salesforce, Yardi, FNT, Cosmos, App Insights, etc.). All Copy Jobs, Mirroring Databases, and ingestion pipelines land their data here in Lakehouses : **LH_SOURCE_DATA.**

This aligns with Fabric guidance to keep a raw _landing/bronze_ area separate from downstream modeling layers in a medallion-style architecture.

[onelake-medallion-lakehouse-architecture](https://learn.microsoft.com/en-us/fabric/onelake/onelake-medallion-lakehouse-architecture)

**How other workspaces use it**

After ingestion, (Dev) Source Data acts as a **shared "source-data hub"**:

- Downstream workspaces (e.g. GTM, Nexus, DQ, domain-specific workspaces) don't ingest from the external systems directly unless its really specific to that workspace.
- Instead, they **consume tables from (Dev) Source Data via OneLake shortcuts**, treating those tables as their authoritative source.

This matches common Fabric patterns where one workspace owns ingestion and other "satellite" workspaces read the same physical data using OneLake shortcuts, avoiding duplicate copies.

**Why we use this pattern**

- **Single source of truth** - All raw and lightly standardized data is mastered once in (Dev) Source Data, then reused everywhere.
- **No or low duplication** - OneLake shortcuts expose the same Delta tables into other workspaces without copying data, which is exactly what Microsoft recommends to unify data across domains and capacities.
- **Simpler governance** - Security, lineage, and retention rules are defined in one place and inherited by consuming workspaces, in line with Fabric data-mesh / hub-and-spoke guidance.
- **Operational isolation** - Ingestion can be scaled, monitored, and fixed in (Dev) Source Data without impacting downstream modeling/reporting workspaces. Fabric docs explicitly call out using separate workspaces for bronze/silver/gold or ingestion vs. consumption workloads