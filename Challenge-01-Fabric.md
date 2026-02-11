# Challenge 01 - Data Warehouse Migration to Microsoft Fabric

[< Previous Challenge](./Challenge-00-Fabric.md) - **[Home](./README-Fabric.md)** - [Next Challenge >](./Challenge-02-Fabric.md)

## Introduction

WWI wants to modernize their data warehouse in phases. The first stage will be to migrate their existing data warehouse to the cloud using Microsoft Fabric. The data warehouse migration will be from their on-premise WWI Data Warehouse to a **Fabric Data Warehouse**. They want to reuse their existing ETL logic and leave their source systems as-is (no migration). This will require a hybrid architecture with on-premise OLTP and Microsoft Fabric as the cloud-based analytical target. This exercise showcases how to migrate from a traditional SQL Server (SMP) to Microsoft Fabric's distributed analytics platform.

## Description

The objective of this challenge is to migrate the WWI DW (OLAP) to a **Microsoft Fabric Data Warehouse**. Fabric Data Warehouse is a cloud-native, distributed analytical engine that uses an automatically optimized storage format (Delta Lake with V-Order) and does not require manual distribution key or index configuration like Azure Synapse Dedicated SQL Pool.

There will be four different object types to migrate:

* Database Schemas and Tables
* Database code (Stored Procedures, Functions, etc.)
* ETL code migration (from SSIS to Fabric Dataflows Gen2 or Pipelines)
* Data migration (loading data into Fabric Data Warehouse)

Here are the steps to migrate from SQL Server to Microsoft Fabric:

### Step 1: Create a Fabric Data Warehouse
- In your Fabric workspace, create a new **Data Warehouse** item
- This will be the target for your WWI DW migration

### Step 2: Migrate Database Schemas and Tables
- Migrate all database schemas to the Fabric Data Warehouse
- Create one table per schema to start:
    - `Dimension.City`, `Fact.Order`, and `Integration.Order_Staging`
- **Important Fabric differences from SQL Server:**
    - No `DISTRIBUTION` clause needed (Fabric auto-optimizes distribution)
    - No manual index creation (`CREATE INDEX` is not supported; Fabric uses automatic Clustered Columnstore with V-Order)
    - `IDENTITY` columns are supported but behave differently
    - Some data types are unsupported (e.g., `geography`, `geometry`, `hierarchyid`, `xml`, `sql_variant`)
    - No table constraints like `FOREIGN KEY` or `UNIQUE`
    - Use `CREATE TABLE` syntax adapted for Fabric

### Step 3: Refactor Stored Procedures
- Refactor one Stored Procedure per design pattern:
    - Dimension Tables (`Integration.MigratedCityData`)
    - Fact Table - Appends Only (`Integration.MigratedStagedSaleData`)
    - Fact Table - Merge (`Integration.MigratedStagedMovementData`)
- Key T-SQL adaptations for Fabric:
    - `MERGE` is supported in Fabric Data Warehouse
    - CTEs (Common Table Expressions) are supported
    - Review [T-SQL surface area](https://learn.microsoft.com/en-us/fabric/data-warehouse/tsql-surface-area) for supported features

### Step 4: Create Remaining Objects
- Execute the T-SQL script `Master Create.sql` from the `/Challenge01/` folder of the student `Resources.zip` package
    - **Note:** You will need to adapt this script for Fabric's T-SQL surface area (remove unsupported data types, distribution clauses, and index definitions)
    - This will create all remaining fact, dimension, integration tables and stored procedures

### Step 5: Load Data into Fabric Data Warehouse
- **Option A: Use Fabric Data Factory Pipeline** (Recommended)
    - Create a pipeline with a **Copy Activity** to move data from the on-premise SQL Server to the Fabric Data Warehouse
    - Use an **On-premises Data Gateway** to connect to the source SQL Server
    - Configure the Copy Activity source as the on-premise database and the sink as your Fabric Data Warehouse tables
- **Option B: Use Fabric Migration Assistant**
    - Export a DACPAC from the source SQL Server data warehouse
    - Use the [Fabric Migration Assistant](https://learn.microsoft.com/en-us/fabric/data-warehouse/migration-assistant) to import the schema and data
    - The Migration Assistant will automatically convert DDL to Fabric-compatible syntax
- **Option C: Use Dataflows Gen2**
    - Create Dataflows Gen2 in your Fabric workspace to replicate the SSIS ETL functionality
    - Use Power Query Online to connect to the source and transform data
    - Load data directly into Fabric Data Warehouse tables
- Review the data setup instructions before executing the data load
- **For the first time setup only**, execute the adapted `Master Create.sql` script to populate all control tables before loading data. After initial setup, execute the Reseed ETL Stored Procedure for subsequent runs.

### Step 6: Validate with Power BI
- Connect Power BI to your Fabric Data Warehouse
    - Fabric provides a native SQL connection endpoint for your Data Warehouse
    - You can also create a **semantic model** directly from the warehouse
- Open the `WWI_Sales.pbit` file from the `/Challenge01/` folder of the student `Resources.zip` package
- Update the connection to point to your Fabric Data Warehouse

## Success Criteria

- All schemas and tables have been created in the Fabric Data Warehouse
- Stored procedures have been refactored and execute successfully in Fabric
- Data has been loaded from the source SQL Server into the Fabric Data Warehouse
- Compare your Power BI report results with the coach's screenshot to confirm the migration was successful

## Learning Resources

### Overall Migration
- [Migration Strategy: Synapse to Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/migration-synapse-dedicated-sql-pool-warehouse)
- [Migration Methods](https://learn.microsoft.com/en-us/fabric/data-warehouse/migration-synapse-dedicated-sql-pool-methods)
- [Fabric Migration Assistant](https://learn.microsoft.com/en-us/fabric/data-warehouse/migration-assistant)

### Database Schema Migration
- [Fabric Data Warehouse Overview](https://learn.microsoft.com/en-us/fabric/data-warehouse/data-warehousing)
- [Tables in Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/tables)
- [T-SQL Surface Area in Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/tsql-surface-area)
- [Create Table Syntax](https://learn.microsoft.com/en-us/fabric/data-warehouse/create-table)
- [Data Types in Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/data-types)

### Database Code Rewrite (T-SQL)
- [T-SQL Surface Area Reference](https://learn.microsoft.com/en-us/fabric/data-warehouse/tsql-surface-area)
- [Stored Procedures in Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/query-activity#stored-procedures)
- [MERGE Statement in Fabric](https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=fabric)

### Data Loading
- [Ingest Data into Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/ingest-data)
- [COPY INTO Command](https://learn.microsoft.com/en-us/fabric/data-warehouse/ingest-data-copy)
- [Data Factory Pipelines in Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/activity-overview)
- [Dataflows Gen2 Overview](https://learn.microsoft.com/en-us/fabric/data-factory/dataflows-gen2-overview)
- [On-premises Data Gateway](https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-install)

## Tips

- Connect to the source SQL Server 2019 databases using Azure Data Studio or SSMS
- Run this query on the source to identify columns with unsupported data types in Fabric:
    ```sql
    SELECT  t.[name], c.[name], c.[system_type_id], c.[user_type_id], y.[is_user_defined], y.[name]
    FROM sys.tables  t
    JOIN sys.columns c on t.[object_id]    = c.[object_id]
    JOIN sys.types   y on c.[user_type_id] = y.[user_type_id]
    WHERE y.[name] IN ('geography','geometry','hierarchyid','image','text','ntext','sql_variant','timestamp','xml')
    OR  y.[is_user_defined] = 1;
    ```
- When adapting `CREATE TABLE` statements for Fabric, remove:
    - `DISTRIBUTION = HASH(column)` / `REPLICATE` / `ROUND_ROBIN`
    - `CLUSTERED COLUMNSTORE INDEX` / `CLUSTERED INDEX`
    - `WITH (HEAP)` or other index hints
    - Fabric automatically handles distribution and uses V-Order Clustered Columnstore
- The Fabric Migration Assistant can automate much of the schema conversion from a DACPAC file
- For on-premise connectivity, install and configure an **On-premises Data Gateway** on a machine that has network access to your SQL Server
- Use the **Fabric REST APIs** to programmatically create workspace items if preferred:
    ```
    POST https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/items
    ```

## Advanced Challenges (Optional)

Too comfortable? Eager to do more? Try these additional challenges!

- Use the [Fabric REST APIs](https://learn.microsoft.com/en-us/rest/api/fabric/articles/) to programmatically create the Data Warehouse and load data
- Automate the schema migration using the [Fabric Migration Assistant](https://learn.microsoft.com/en-us/fabric/data-warehouse/migrate-with-migration-assistant) with a DACPAC export
- Set up [Git integration](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/intro-to-git-integration) for version control of your Fabric artifacts
- Compare query performance between the source SQL Server and Fabric Data Warehouse
