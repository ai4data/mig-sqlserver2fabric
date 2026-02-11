# Challenge 02 - OneLake Integration

[< Previous Challenge](./Challenge-01-Fabric.md) - **[Home](./README-Fabric.md)** - [Next Challenge >](./Challenge-03-Fabric.md)

## Introduction

WWI importers realize they need to further modernize their data warehouse and want to proceed to the second stage. They are starting to reach capacity constraints and need to offload data files from the relational database. Likewise, they are receiving more data in JSON and CSV file formats. They've been discussing re-engineering their data architecture to accommodate larger data sets, semi-structured data, and real-time ingestion. They would like to conduct a POC using Microsoft Fabric's **OneLake** and **Lakehouse** to see how best to design it for integration into the Data Warehouse.

For this challenge, WWI wants us to build out the data lake using Fabric's built-in OneLake and show how to load data into it from an on-premise data source.

## Description

The objective of this challenge is to build a **Fabric Lakehouse** backed by **OneLake** as the staging area for all source system data. OneLake is Microsoft Fabric's unified data lake -- think of it as "OneDrive for data." Every item in your Fabric workspace (Lakehouses, Warehouses, etc.) automatically stores its data in OneLake in Delta/Parquet format.

We need to ensure this data lake is well organized and doesn't turn into a data swamp. This challenge will help us organize the folder structure, set up security to prevent unauthorized access, and extract data from the WWI OLTP platform into the Lakehouse.

The OLTP platform is on-premise, so you will need to use an **On-premises Data Gateway** to integrate it into Fabric. The pipeline you build will become the EXTRACT portion of the new E-L-T process.

### Step 1: Create a Fabric Lakehouse
- In your Fabric workspace, create a new **Lakehouse** item (e.g., `wwi_staging`)
- Understand the Lakehouse structure:
    - `/Tables` - Managed Delta Lake tables (refined/gold data)
    - `/Files` - Unmanaged files (raw/bronze data, CSVs, JSONs, etc.)

### Step 2: Organize the Folder Structure
Design a folder structure within your Lakehouse using the `/Files` section:
- **Landing Zone** (`/Files/landing/`) - Incremental extracts from source systems
- **Raw/Historical Zone** (`/Files/raw/`) - Historical snapshots of source data
- **Staged Zone** (`/Files/staged/`) - Cleaned and transformed data ready for loading
- **Curated Zone** - Represented as Delta tables in `/Tables`

### Step 3: Configure Security
- Use **Fabric workspace roles** to control who can access the Lakehouse
- Leverage **OneLake security** features:
    - OneLake exposes POSIX-compatible ACLs via ADLS Gen2 APIs
    - Use **OneLake shortcuts** to share specific data without granting full access
    - Configure appropriate permissions so that:
        - ETL service accounts can write to the landing zone
        - Data scientists can query raw data through the SQL analytics endpoint
        - Unauthorized users cannot access sensitive files

### Step 4: Setup Source Database for Incremental Load
Prior to building the pipeline, ensure there are changes in the City data from the Wide World Importers OLTP Database. Execute the following scripts:

- Execute `generateCityData.sql` from the `/Challenge02/` folder of the `Resources.zip` file to update 10 existing records and insert 1 new record:
    ```sql
    UPDATE T
    SET [LatestRecordedPopulation] = LatestRecordedPopulation + 1000
    FROM (SELECT TOP 10 * from [Application].[Cities]) T

    INSERT INTO [Application].[Cities]
        ([CityName], [StateProvinceID], [Location], [LatestRecordedPopulation], [LastEditedBy])
    VALUES
        ('NewCity' + CONVERT(char(19), getdate(), 121), 1, NULL, 1000, 1);
    ```

- Modify the `[Integration].[GetCityUpdates]` stored procedure in the OLTP database to remove the `Location` field (incompatible data type). Use `GetCityUpdates.sql` from the `/Challenge02/` folder.

- Update the load control date in your Fabric Data Warehouse. Use `UpdateLoadControl.sql` from the `/Challenge02/` folder:
    ```sql
    UPDATE INTEGRATION.LOAD_CONTROL
    SET LOAD_DATE = getdate()
    ```

### Step 5: Build the Data Pipeline
Create a **Fabric Data Factory Pipeline** to extract incremental City data:

1. **Lookup Activity** - Query `[Integration].[ETL Cutoff]` table in your Fabric Data Warehouse to get the `@LastCutoff` date
2. **Lookup Activity** - Query `[Integration].[Load_Control]` table in your Fabric Data Warehouse to get the `@NewCutoff` date
3. **Copy Activity** - Use `[Integration].[GetCityUpdates]` stored procedure in the on-premise WideWorldImporters OLTP database as source (via On-premises Data Gateway), and write to the `/Files/landing/city/` directory in your Lakehouse as the sink

## Success Criteria

- Lakehouse is created with a well-organized folder structure visible in the Fabric portal
- 11 updated records from the City table exist in the landing zone of your Lakehouse
- An unauthorized user (someone without workspace access) cannot view the city data files
- Pipeline executes successfully and shows data flow from on-premise to OneLake

## Learning Resources

- [OneLake Overview](https://learn.microsoft.com/en-us/fabric/onelake/onelake-overview)
- [Lakehouse Overview](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-overview)
- [Delta Lake in Fabric](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-and-delta-tables)
- [OneLake File Explorer](https://learn.microsoft.com/en-us/fabric/onelake/onelake-file-explorer)
- [OneLake Security](https://learn.microsoft.com/en-us/fabric/onelake/security/get-started-security)
- [OneLake Shortcuts](https://learn.microsoft.com/en-us/fabric/onelake/onelake-shortcuts)
- [Fabric Data Factory Pipelines](https://learn.microsoft.com/en-us/fabric/data-factory/activity-overview)
- [Copy Activity in Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/copy-data-activity)
- [On-premises Data Gateway](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-install)
- [Incremental Copy Pattern](https://learn.microsoft.com/en-us/azure/data-factory/tutorial-incremental-copy-multiple-tables-portal)

## Tips

- Things to consider when designing your Lakehouse:
    - OneLake is hierarchical: Tenant > Workspaces > Items (Lakehouses) > `/Tables` and `/Files`
    - Files in `/Tables` are automatically in Delta format and queryable via the SQL analytics endpoint
    - Files in `/Files` are unmanaged -- use this for landing raw data in CSV/Parquet/JSON formats
    - Use **Lakehouse schemas** to organize Delta tables into logical groupings (e.g., `staging`, `dimension`, `fact`)
- For on-premise connectivity:
    - Install the **On-premises Data Gateway** on a machine with network access to your SQL Server
    - Register the gateway in your Fabric tenant
    - Create a **connection** in Fabric that uses the gateway
- You can browse your OneLake data using:
    - **OneLake File Explorer** (Windows desktop app -- mounts OneLake as a drive)
    - **Azure Storage Explorer** (using the OneLake DFS endpoint)
    - The Fabric portal Lakehouse explorer
- Be sure to review the `[Integration].[ETL Cutoff]` and `[Integration].[Load_Control]` tables in your Fabric Data Warehouse before executing the pipeline. If dates are not set correctly, the source stored procedure will not return any data.
- You can use **Fabric notebook** (PySpark) as an alternative to the Copy Activity for more complex extraction scenarios

## Advanced Challenges (Optional)

Too comfortable? Eager to do more? Try these additional challenges!

- Parameterize the source and sink properties in your pipeline so you can reuse it for all tables
- Develop an incremental load pattern for each copy activity to avoid full loads
- Create an **OneLake shortcut** from your Lakehouse to an external ADLS Gen2 account to demonstrate Fabric's multi-cloud data virtualization
- Use a **Fabric Spark notebook** to explore and profile the raw data in the landing zone
- Use the **Fabric REST API** to programmatically create the Lakehouse and configure shortcuts
