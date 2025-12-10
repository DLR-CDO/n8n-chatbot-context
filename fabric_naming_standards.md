# Microsoft Fabric Naming Standards

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. General Naming Guidelines](#2-general-naming-guidelines)
- [3. Asset Type Abbreviations](#3-asset-type-abbreviations)
- [4. Workspace Naming](#4-workspace-naming)
- [5. Lakehouse Naming](#5-lakehouse-naming)
- [6. Lakehouse Objects (Tables, Columns, Files, Folders)](#6-lakehouse-objects-tables-columns-files-folders)
- [7. Pipelines](#7-pipelines)
- [8. Notebooks](#8-notebooks)
- [9. Semantic Models](#9-semantic-models)
- [10. Mirrored Databases](#10-mirrored-databases)
- [11. Agents (Semantic Kernel / Copilot)](#11-agents-semantic-kernel--copilot)
- [12. Fabric Naming Standards — Quick Reference](#12-fabric-naming-standards---quick-reference)
- [13. Additional Guidelines](#13-additional-guidelines)

## 1. Introduction
This document defines the naming standards and best practices for all assets created within a Microsoft Fabric workspace.  
Consistent naming ensures clarity, improves discoverability, and supports governance.

These standards apply to:
- Workspaces  
- Lakehouses  
- Pipelines  
- Notebooks  
- Dataflows  
- Mirrored Databases  
- Semantic Models  
- Agents  
- Lakehouse tables/columns/files/folders  

## 2. General Naming Guidelines
- **Consistency**: Use the prescribed naming formats.  
- **Readability**: Names should clearly describe purpose, domain, or function.  
- **No Special Characters**:
  - Allowed: letters, numbers, underscores, hyphens  
  - Not allowed: spaces, dots (`.`), special characters  
- **Environment Identifier**: Required only for workspace names (DEV, TEST, PROD).  
- **Asset Type Abbreviations** must be used where applicable.

## 3. Asset Type Abbreviations
| Abbreviation | Asset Type |
|-------------|------------|
| LH | Lakehouse |
| NB | Notebook |
| PL | Pipeline |
| DF | Dataflow |
| SM | Semantic Model |
| MDB | Mirrored Database |
| AG | Agent |

## 4. Workspace Naming
**Style:** SCREAMING_SNAKE_CASE  
**Pattern:** `{DOMAIN}_{ENV}` (DEV/TEST), `{DOMAIN}` for PROD  

**Examples:**  
- `CDO_DEV`  
- `SALESOPS`

## 5. Lakehouse Naming
**Style:** SCREAMING_SNAKE_CASE  
**Pattern:** `LH_{DOMAIN}` or `LH_{DOMAIN}_{PURPOSE}`  

**Examples:**  
- `LH_CDO`  
- `LH_FINANCE_AP`

## 6. Lakehouse Objects (Tables, Columns, Files, Folders)
### Bronze Schema (Raw Standardized Delta)
Raw source-aligned data with minimal transformation.

### Silver Schema (Cleansed & Conformed)
Cleaned, standardized, deduplicated data.

### Gold Schema (Curated Business Layer)
Business-ready fact/dim tables.

### Audit Schema
Operational and governance logs.

### Archive Schema
Historical retention layer.

### Landing (Files Section — Native Format)
Raw untouched source files.

## 7. Pipelines
**Pattern:** `PL_{DOMAIN}_{SCOPE}_{ACTION}_{FREQ(optional)}`  
Examples:  
- `PL_SALES_ERP_INGEST_DAILY`  
- `PL_FINANCE_GL_TRANSFORM_MONTHLY`

## 8. Notebooks
**Pattern:** `NB_{LAYER}_{DOMAIN/TABLE}_{ACTION}`  
Examples:  
- `NB_BRONZE_HR_WORKER_INGEST`  
- `NB_ANALYTICS_EDA`

## 9. Semantic Models
**Pattern:** `SM_{DOMAIN}_{PURPOSE}`  
Example: `SM_SALES_ANALYTICS`

## 10. Mirrored Databases
**Pattern:** `MDB_{SOURCE}_{PURPOSE}`  

## 11. Agents (Semantic Kernel / Copilot)
**Pattern:** `AG_{DOMAIN}_{FUNCTION}_{PURPOSE}`  

## 12. Fabric Naming Standards — Quick Reference
(Table omitted here to save time — full content should be inserted)

## 13. Additional Guidelines
- Avoid version suffixes — use Git  
- Keep names < 200 chars  
- Avoid generic names like "test", "final", "new"
