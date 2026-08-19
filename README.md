# ADF Pipelines

## Overview

This branch contains Azure Data Factory pipelines for **client onboarding and data loading**.

## Pipelines

| Pipeline                  | Purpose                                                       |
| ------------------------- | ------------------------------------------------------------- |
| `Onboarding_pipeline`     | Creates and initializes the client environment.               |
| `Masterpipeline`          | Discovers, validates, and routes source files for processing. |
| `LoadMultipleTables_csv`  | Loads CSV files into the client Azure SQL database.           |
| `LoadMultipleTables_xlsx` | Loads Excel sheets into the client Azure SQL database.        |

## Onboarding

The `Onboarding_pipeline` performs the initial client setup:

* Creates the client-specific Blob Storage container and required folders.
* Creates the client Azure SQL database.
* Creates required schemas.
* Retrieves and executes database/DDL scripts from GitHub.
* Initializes the environment required for data loading.

## Data Load

The `Masterpipeline` acts as the main orchestration pipeline. It:

* Detects files in the client Blob Storage container.
* Validates files against `dbo.etl_lookupsource`.
* Routes CSV and XLSX files to their respective pipelines.
* Handles inactive and unsupported files.
* Archives successfully processed files and moves rejected files to the `rejected` folder.

The child pipelines load data into the **`prebronze` schema**. Target tables are determined dynamically using the ETL lookup configuration.

## Logging

The pipelines maintain:

* `dbo.etl_auditlog` – records successful data loads, including source/target row counts.
* `dbo.etl_errorlog` – records file validation, inactive file, and rejected file scenarios.

## End-to-End Flow

```text
Onboarding_pipeline
        |
        v
Client Environment Setup
        |
        v
Masterpipeline
        |
   +----+----+
   |         |
  CSV       XLSX
   |         |
   v         v
CSV Load   XLSX Load
   |         |
   +----+----+
        |
        v
   prebronze
        |
        v
Audit / Archive
```
