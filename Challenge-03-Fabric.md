# Challenge 03 - Data Pipeline Migration

[< Previous Challenge](./Challenge-02-Fabric.md) - **[Home](./README-Fabric.md)** - [Next Challenge >](./Challenge-04-Fabric.md)

## Introduction

WW Importers keep missing the SLAs for their nightly data load process. The loads take six hours to complete and start each evening at 1:00AM. They must complete by 8:00AM but frequently these jobs are taking longer than planned. In addition, critical stakeholders are asking for data more frequently. Since these business units are key stakeholders, they have the funding to help replatform the data pipelines.

WW Importers realizes they need to leverage **OneLake** to scale and load data into their **Fabric Data Warehouse** for Stage 3. These data pipelines must be ELT (Extract, Load & Transform) so they can quickly write the data to the cloud and scale out the compute to transform the data.

## Description

The objective of this challenge is to modernize the ETL pipeline that was originally built in SSIS. We need to rebuild this pipeline in Microsoft Fabric leveraging scale-out architecture to transform this data. The data flow will include steps to extract the data from the OLTP platform, store it in the **Fabric Lakehouse** (OneLake), and bulk ingest it into the **Fabric Data Warehouse**. This will be run on a nightly basis using **Fabric Data Factory Pipelines** for orchestration and scheduling.

![Current SSIS Workflow](../Coach/images/SSISFlow.png)

**Below is a summary of each task in the existing SSIS package. We will rebuild these steps using Fabric capabilities:**

1. **Retrieve ETL Cutoff Date** - Query `[Integration].[Load_Control]` in Fabric Data Warehouse (should already exist from Challenge 2)
2. **Populate Date Dimension** - Execute `[Integration].[PopulateDateDimensionForYear]` stored procedure in Fabric Data Warehouse
3. **Create Lineage Key** - Execute `[Integration].[GetLineageKey]` to create a record for each activity in `[Integration].[Lineage Key]`
4. **Truncate Staging Tables** - Truncate `[Integration].[[Table]_Staging]` tables
5. **Get ETL Cutoff Dates** - Retrieve cutoff dates for last successful load of each table from `[Integration].[ETL Cutoffs]`
6. **Extract and Load Data** - Read new data from the OLTP source and load into staging tables in the Fabric Data Warehouse via OneLake
7. **Merge into Target Tables** - Execute `[Integration].[MigrateStaged[Table]]` stored procedures to merge staged data into `[Dimension]` and `[Fact]` tables
    - **NOTE:** Dimension tables must be loaded before Fact tables to maintain referential integrity for surrogate keys

**NOTE:** This challenge builds upon the previous 2 challenges -- reuse content wherever possible.

### Build a Data Pipeline for `[Dimension].[City]`

Create a **Fabric Data Factory Pipeline** that orchestrates the following activities:

#### Activity 1: Lookup - Get Last Cutoff Date
- Query `[Integration].[ETL Cutoff]` in the Fabric Data Warehouse
- Output: `@LastCutoff` parameter

#### Activity 2: Lookup - Get New Cutoff Date
- Query `[Integration].[Load_Control]` in the Fabric Data Warehouse
- Output: `@NewCutoff` parameter

#### Activity 3: Create Lineage Key
- Execute `[Integration].[GetLineageKey]` stored procedure
- Use the T-SQL script `proc_Integration.CreateLineageKey.sql` from the `/Challenge03/` folder of the `Resources.zip`

#### Activity 4: Extract to OneLake (Lakehouse)
- **Copy Activity**: Extract data from the on-premise OLTP database using `[Integration].[GetCityUpdates]` stored procedure
- Sink: Write to the `/Files/landing/city/` directory in your Lakehouse (Parquet or CSV format)
- This replaces Step 6 in the original SSIS package

#### Activity 5: Load from OneLake to Fabric Data Warehouse
Load the staged files from OneLake into `[Integration].[City_Staging]` table in the Fabric Data Warehouse. You have several options:

- **Option A: COPY INTO command** (Recommended)
    ```sql
    COPY INTO [Integration].[City_Staging]
    FROM 'https://onelake.dfs.fabric.microsoft.com/<workspace>/<lakehouse>.Lakehouse/Files/landing/city/'
    WITH (FILE_TYPE = 'PARQUET');
    ```
- **Option B: Copy Activity** - Use a Copy Activity with Lakehouse as source and Warehouse as sink
- **Option C: Spark Notebook** - Use PySpark to read from the Lakehouse and write to the Warehouse

Review the T-SQL scripts `[proc_Integration.Ingest@@@Data]` from the `/Challenge03/` folder for reference. For this challenge, only run the City script.

#### Activity 6: Merge into Dimension Table
- Execute `[Integration].[MigratedStagedCityData]` stored procedure to merge data from `[Integration].[City_Staging]` into `[Dimension].[City]`

#### Activity 7: Archive to Raw Zone
- Move or copy the landing files to `/Files/raw/WWIDB/City/{YYYY}/{MM}/{DD}/` in your Lakehouse for historical retention

## Success Criteria

- Pipeline executes successfully end-to-end
- 11 new records from the City data file have been loaded into `[Dimension].[City]` in the Fabric Data Warehouse
- Data flow follows the ELT pattern: Extract to OneLake, Load into staging, Transform/Merge into target

## Learning Resources

- [Fabric Data Factory Pipelines](https://learn.microsoft.com/en-us/fabric/data-factory/activity-overview)
- [Copy Activity in Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/copy-data-activity)
- [COPY INTO in Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/ingest-data-copy)
- [Stored Procedure Activity](https://learn.microsoft.com/en-us/fabric/data-factory/stored-procedure-activity)
- [Notebook Activity in Pipelines](https://learn.microsoft.com/en-us/fabric/data-factory/notebook-activity)
- [Lookup Activity in Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/lookup-activity)
- [Pipeline Scheduling and Triggers](https://learn.microsoft.com/en-us/fabric/data-factory/pipeline-runs)

## Tips

- Optimize where possible by using dynamic content, parameters, and executing tasks in parallel
- Use **expressions** in pipeline activities to build dynamic file paths:
    ```
    @concat('landing/city/', formatDateTime(utcnow(), 'yyyy'), '/', formatDateTime(utcnow(), 'MM'), '/', formatDateTime(utcnow(), 'dd'), '/')
    ```
- The `COPY INTO` command in Fabric Data Warehouse is the fastest way to bulk load data from OneLake files
- Ensure the `[Integration].[City_Staging]` table exists and is empty before loading
- Use the **Debug** button in the pipeline designer to test your pipeline before scheduling
- Fabric pipelines support **ForEach** loops -- useful for iterating over multiple tables in the advanced challenge
- You can monitor pipeline runs from the **Monitor Hub** in the Fabric portal
- Consider using a **Spark notebook** for complex transformations that go beyond what T-SQL stored procedures can handle

## Advanced Challenges (Optional)

Too comfortable? Eager to do more? Try these additional challenges!

- Enhance the pipeline to load **all tables** using a ForEach loop:
    - Use expressions and parameters for dynamic table names
    - Load Dimensions before Facts using dependency configuration
    - Review `PopulateETLCutoff.sql` from `/Challenge03/` for load dependency order
- Use a **Fabric Spark notebook** (PySpark/Scala) to replace the T-SQL ingestion scripts
- Build the pipeline using **Fabric REST APIs** for fully programmatic deployment:
    ```
    POST https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/items
    ```
- Implement **CI/CD** using Fabric's Git integration and deployment pipelines
- Create a **Dataflow Gen2** as an alternative to the Copy + Stored Procedure pattern for simpler table loads
