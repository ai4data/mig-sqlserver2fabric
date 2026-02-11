# Challenge 05 - Analytics Migration

[< Previous Challenge](./Challenge-04-Fabric.md) - **[Home](./README-Fabric.md)** - [Next Challenge >](./Challenge-06-Fabric.md)

## Introduction

WWI leadership wants to leverage Power BI to create rich semantic models that promote self-service BI using Microsoft Fabric. They want to empower analysts from different organizations to use Power BI reporting capabilities and build analytics reports and dashboards from those data models. The final solution needs to consider both report design and optimal response times. Dashboards need to return in less than 5 seconds.

## Description

The objective of this challenge is to have the Power BI report "WWI_Sales.pbix" return in less than 5 seconds. This will require you to optimize the Power BI data model using Fabric's native capabilities -- particularly **Direct Lake mode** -- and tune your Fabric Data Warehouse. After completion, review your results with the coach to determine optimal design.

### Understanding Direct Lake Mode

Direct Lake is a Fabric-specific storage mode that reads data directly from Delta tables in OneLake into the Power BI engine's memory -- combining the performance of Import mode with the freshness of DirectQuery. This is the key differentiator for analytics performance in Fabric.

| Storage Mode | Data Freshness | Performance | Data Volume |
|---|---|---|---|
| Import | Stale (needs refresh) | Fastest | Limited by model size |
| DirectQuery | Real-time | Slowest | Unlimited |
| **Direct Lake** | **Near real-time** | **Near-Import speed** | **Very large** |

### Step 1: Review Table Distribution in Fabric Data Warehouse

Unlike Synapse Dedicated SQL Pool, Fabric Data Warehouse **automatically manages data distribution** -- you do not manually set distribution types (hash, round-robin, replicated). However, you should still:

1. Check for **data skew** by examining row counts and data distribution across tables
2. Use the original `skew.pbix` from `/Challenge05/` as a reference to understand your data profile
3. Consider **table partitioning** for large fact tables (e.g., partition `[Fact].[Sale]` by date) to enable partition elimination during queries

### Step 2: Review Query Performance

1. Open the Power BI report and capture baseline runtime for the "Total Sales by State Province" visual in the "High Level Dashboard"
2. Use **Performance Analyzer** in Power BI Desktop to record the response time
3. Copy the DAX query from Performance Analyzer and run it against the Fabric Data Warehouse SQL endpoint
4. Review the query execution plan:
    ```sql
    EXPLAIN
    SELECT [City].[State Province], SUM([Sale].[Total Including Tax])
    FROM [Fact].[Sale]
    JOIN [Dimension].[City] ON [Sale].[City Key] = [City].[City Key]
    JOIN [Dimension].[Date] ON [Sale].[Invoice Date Key] = [Date].[Date]
    GROUP BY [City].[State Province]
    ```

### Step 3: Run Statistics and Optimize

Fabric Data Warehouse supports **automatic statistics** that are proactively maintained. However, you can also:

1. **Verify automatic statistics** are enabled and up-to-date
2. **Create manual statistics** on key columns if needed:
    ```sql
    CREATE STATISTICS stats_city_key ON [Fact].[Sale] ([City Key]);
    CREATE STATISTICS stats_date_key ON [Fact].[Sale] ([Invoice Date Key]);
    ```
3. **Enable Result-set Caching** for frequently executed queries:
    ```sql
    ALTER DATABASE CURRENT SET RESULT_SET_CACHING ON;
    ```
4. Rerun the Performance Analyzer and compare with your baseline

### Step 4: Create a Direct Lake Semantic Model

1. Navigate to your Fabric Data Warehouse or Lakehouse in the portal
2. Create a new **semantic model** (previously called "dataset") from the warehouse
3. Select the relevant tables (`Fact.Sale`, `Dimension.City`, `Dimension.Date`, etc.)
4. The semantic model will automatically use **Direct Lake mode** when created from a Fabric item
5. Define relationships between tables in the semantic model editor
6. Publish the semantic model to your workspace

### Step 5: Optimize the Power BI Report

1. Connect the `WWI_Sales.pbit` report to your new Direct Lake semantic model
2. Use **Performance Analyzer** to capture runtimes for all visuals in the "High Level Dashboard"
3. Experiment with **Composite models** if needed:
    - Keep large fact tables in Direct Lake mode
    - Consider Import or Dual mode for small dimension tables
4. Compare response times between:
    - DirectQuery to Fabric Data Warehouse
    - Direct Lake mode (semantic model on warehouse)
    - Direct Lake mode (semantic model on Lakehouse Delta tables)
5. Identify which approach achieves sub-5-second response times

## Success Criteria

- Power BI report refresh times are under 5 seconds for all visuals on the "High Level Dashboard"
- A Direct Lake semantic model is created and connected to the Fabric Data Warehouse or Lakehouse
- Demonstrate the performance improvement from baseline (DirectQuery) to optimized (Direct Lake)

## Learning Resources

- [Direct Lake Overview](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview)
- [Direct Lake in Power BI Desktop](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-power-bi-desktop)
- [Semantic Models in Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/semantic-models)
- [Direct Lake Web Modeling](https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-web-modeling)
- [Composite Models with Direct Lake](https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-composite-models)
- [Performance Tuning in Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/guidelines-warehouse-performance)
- [Statistics in Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/statistics)
- [Result-set Caching](https://learn.microsoft.com/en-us/fabric/data-warehouse/result-set-caching)
- [V-Order Optimization](https://learn.microsoft.com/en-us/fabric/data-engineering/delta-optimization-and-v-order)

## Tips

- **Direct Lake is the recommended approach** for Power BI reports on Fabric data -- it eliminates the need for data import while maintaining near-import performance
- Direct Lake falls back to DirectQuery if the data exceeds memory guardrails -- monitor for fallback events
- Use the **DAX query view** in Power BI Desktop to test DAX queries directly against the semantic model
- Fabric Data Warehouse uses **V-Order optimization** on Clustered Columnstore storage -- this provides 3-4x better compression and up to 10x faster reads than standard Parquet
- You do NOT need to manually choose distribution types in Fabric -- the engine auto-optimizes
- Create direct query mode data model first (if experimenting) because you can convert DirectQuery to Import but not vice versa
- The **SQL analytics endpoint** on a Lakehouse provides read-only T-SQL access to Delta tables -- another option for Direct Lake semantic models

## Advanced Challenges (Optional)

Too comfortable? Eager to do more? Try these additional challenges!

- Build **Aggregate tables** in the semantic model to accelerate common query patterns
- Create a **Composite semantic model** that combines Direct Lake tables from the Lakehouse with Import tables from external sources
- Use the **Fabric REST API** to programmatically create and configure semantic models
- Set up **deployment pipelines** in Fabric for dev/test/prod promotion of your semantic models and reports
- Compare performance metrics between a semantic model on the Data Warehouse vs. one on the Lakehouse
- Enable **automatic page refresh** in Power BI for near-real-time dashboard updates
