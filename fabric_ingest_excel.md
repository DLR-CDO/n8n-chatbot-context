# Microsoft Fabric — Ingest Excel (Multi‑Sheet) with Spark → Delta (Lakehouse)

## Overview

**Goal:** Read an Excel workbook with multiple sheets (`Global_Superstore_2016.xlsx`) using `pandas`, then write one Delta table per sheet (bronze schema) in a Fabric Lakehouse so the data is queryable via the SQL Endpoint and Power BI.

---

## A) Open Fabric & Lakehouse

1. Sign in to Fabric: https://app.fabric.microsoft.com/
2. Open your Workspace and select the Lakehouse where you will ingest data.

**Fabric Home / Workspace**

![Fabric_Ingest_Excel_image1](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image1.png)

**Lakehouse opened**

![Fabric_Ingest_Excel_image2](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image2.png)
![Fabric_Ingest_Excel_image3](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image3.png)

- Navigate: **(Dev) CDO → Notebook → bronze**  
- As we follow medallion architecture all source ingestion happens in the **bronze** layer.

When starting a new project, create a folder with a similar name to organize notebooks.

![Fabric_Ingest_Excel_image4](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image4.png)

Click **+ New Item**, search for *notebook* and select a notebook option.

![Fabric_Ingest_Excel_image5](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image5.png)

A notebook must be attached to a data source (Lakehouse/Warehouse/EventHouse). Click **Add data items → Choose your existing lakehouse** (e.g., `LH_CDO`).

![Fabric_Ingest_Excel_image6](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image6.png)
![Fabric_Ingest_Excel_image7](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image7.png)
![Fabric_Ingest_Excel_image8](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image8.png)

---

## B) Upload the Excel workbook to Files

- In the Lakehouse **Files** pane, create a folder, click **Upload** and select `Global_Superstore_2016.xlsx`.

![Fabric_Ingest_Excel_image9](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image9.png)
![Fabric_Ingest_Excel_image10](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image10.png)
![Fabric_Ingest_Excel_image11](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image11.png)

- Click **Refresh** after upload, then navigate to your folder to confirm the file is present.
- Hover the file, click the three dots → **Copy ABFS path**. This copies the absolute path to use in code.

![Fabric_Ingest_Excel_image12](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image12.png)

---

## Some quick introductions to notebooks

**What is a Fabric notebook?** A browser-based, cell-by-cell workspace where you run code, SQL, and markdown against data in your Lakehouse/Warehouse/KQL DB using the Fabric Spark runtime.

**Key concepts**

- **Cells:** Each cell has a type (`Python/PySpark`, `SQL`, `Markdown`). Run individually or run all (Shift + Enter).
- **Runtime (compute):** Fabric auto-manages Spark. Sessions start on first run; jobs scale as needed.
- **Environments:** Optional per‑notebook package sets (e.g., `openpyxl`, `pandas`).
- **Outputs & Tables:** Writes to **Delta** become queryable in the **Tables** pane and via the **SQL Endpoint**.

Shortcuts: [53 Shortcuts for Jupyter Notebook](https://shortcutworld.com/Jupyter-Notebook/win/Jupyter-Notebook_Shortcuts)

### Common operations

**Read a file from Files (Lakehouse attached)**

```python
import pandas as pd
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

# Local Lakehouse path (example)
XLSX = "/lakehouse/default/Files/landing/excel/Global_Superstore_2016.xlsx"

# Single sheet
pdf = pd.read_excel(XLSX, sheet_name="Orders")  # or 0 for first sheet
df = spark.createDataFrame(pdf)

# Multi-sheet
all_sheets = pd.read_excel(XLSX, sheet_name=None)  # dict {sheet_name: pandas_df}
dfs = {name: spark.createDataFrame(pdf) for name, pdf in all_sheets.items()}
```

**Write a Delta table** (appears in Tables pane):

```python
df.write.mode("overwrite").format("delta").saveAsTable("bronze.sample")
```

**Query with SQL**

```sql
%%sql
SELECT COUNT(*) AS rows FROM bronze.sample;
```

---

## C) Paste and run the minimal ingestion code (single cell)

Paste the code below into a single notebook cell and run it. Edit only the variables under **CONFIG**.

```python
# CONFIG - edit these values
import pandas as pd, re
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

XLSX_PATH = "abfss://ffb370c1-c77b-4e94-a18e-32682e8da19b@onelake.dfs.fabric.microsoft.com/6d4ca419-89d0-4e59-996f-8d6c1ad0e56e/Files/adhoc/global_superstore_2016.xlsx"
TARGET_DB = "bronze"
BASE_NAME = "superstore"
MODE = "overwrite"
SHEET_LIST = None  # or list like ['Orders','People']

_name_rx = re.compile(r"[^a-zA-Z0-9\_]+")

def _to_local_lakehouse_path(path: str) -> str:
    if path.startswith("/lakehouse/default"):
        return path
    marker = "/Files/"
    i = path.find(marker)
    return "/lakehouse/default" + path[i:] if i != -1 else path

def sanitize(s: str) -> str:
    return _name_rx.sub("_", (s or "sheet1")).strip("_").lower()

def normalize_columns(pdf: pd.DataFrame) -> pd.DataFrame:
    pdf = pdf.copy()
    pdf.columns = [_name_rx.sub("_", str(c).strip().lower().replace(" ", "_")) for c in pdf.columns]
    return pdf

local_path = _to_local_lakehouse_path(XLSX_PATH)
xls = pd.ExcelFile(local_path)
sheets = SHEET_LIST or xls.sheet_names
if not sheets:
    raise ValueError("No sheets found.")

created = {}

for sh in sheets:
    pdf = pd.read_excel(xls, sheet_name=sh)
    pdf = normalize_columns(pdf)
    sdf = spark.createDataFrame(pdf)
    tbl = f"{TARGET_DB}.{BASE_NAME}_{sanitize(sh)}"
    sdf.write.format("delta").mode(MODE).saveAsTable(tbl)
    cnt = spark.table(tbl).count()
    print(f"{tbl}: {cnt} rows")
    created[tbl] = cnt

print("Ingestion complete:", created)
```

---

## D) Verify created tables and run quick SQL

- In the Lakehouse **Tables** pane, confirm tables exist (e.g., `bronze.superstore_orders`, `bronze.superstore_people`, `bronze.superstore_returns`).

Run quick checks in a SQL cell:

```sql
SELECT COUNT(*) AS rows FROM bronze.superstore_orders;
SELECT COUNT(*) AS rows FROM bronze.superstore_people;
SELECT COUNT(*) AS rows FROM bronze.superstore_returns;
```

**Example output (table counts)**

![Fabric_Ingest_Excel_image16](https://cdostorageacct.blob.core.windows.net/fabric-chatbot/fabric_images/Fabric_Ingest_Excel_image16.png)

---

### Notes & Tips
- Use `SHEET_LIST` to limit which sheets are ingested.  
- The notebook attaches to the Lakehouse, so `saveAsTable` writes into the selected Lakehouse DB (here `bronze`).  
- Normalize column names to `snake_case` to follow Lakehouse standards.  
- If a sheet is very large, consider chunked ingestion or using Spark's native Excel readers in production.
