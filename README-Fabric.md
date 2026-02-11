# What The Hack - This Old Data Warehouse (Microsoft Fabric Edition)

## Introduction

Modern Data Warehouse is a key upgrade motion for organizations to scale out their on-premise analytical workloads to the cloud. This hack will help data engineers and administrators upgrade their skills to migrate to **Microsoft Fabric**. The hack follows sequential migration steps required to migrate from on-premise to Microsoft Fabric and re-platform attached workloads like ETL and reporting.

The solution will require us to migrate the on-premise data warehouse to Microsoft Fabric. In the first challenge, you will source data from the WWI OLTP sample database to a **Fabric Data Warehouse**. Data will be loaded from source to target through **Fabric Data Factory pipelines and Dataflows Gen2**. Secondly, **OneLake** will be built out as your unified data lake and staging area for future challenges. Third, the SSIS packages from Challenge 1 will be refactored into **Fabric Data Factory pipelines** with **Spark notebooks** to optimize the data loads and leverage OneLake as a staging area. Next, clickstream data will be streamed using **Fabric Eventstream** into an **Eventhouse** for real-time analytics queried via **KQL Querysets**. Lastly, a Power BI data model will leverage **Direct Lake mode** for optimal performance with Fabric's native semantic modeling.

Below is a diagram of the solution architecture you will build in this hack. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

![Solution Architecture](../Coach/images/solution_arch.png)

> **Note:** This is the Microsoft Fabric adaptation of the original Azure Synapse Analytics hack. The original challenges can be found in the [Student](../Student/) folder.

## Learning Objectives

In this hack, data engineers will learn how to migrate their platform to Microsoft Fabric (data, schema, and code). Additionally, they will build out modern data architectures that leverage Fabric's unified analytics platform for large data volumes, different data structures, and real-time ingestion.

1. Microsoft Fabric Architecture and Components
1. Fabric Decision Tree (Warehouse vs. Lakehouse vs. Eventhouse)
1. Refactor T-SQL code to be compatible with Fabric Data Warehouse
1. ELT design patterns using Fabric Data Factory + OneLake
1. Setup a streaming data pipeline with Eventstream and Eventhouse
1. Tune Fabric for analytical workloads and design reports for best performance using Direct Lake
1. Implement Data Governance with Microsoft Purview Hub in Fabric
1. Build Enterprise Security using Fabric's built-in security model

## Challenges

- Challenge 00: **[Setup](./Challenge-00-Fabric.md)**
	 - Prepare your environment to work with Microsoft Fabric
- Challenge 01: **[Data Warehouse Migration](./Challenge-01-Fabric.md)**
	 - Migrate the EDW from SQL Server to Microsoft Fabric Data Warehouse
- Challenge 02: **[OneLake Integration](./Challenge-02-Fabric.md)**
	 - Build out the staging tier in OneLake with Lakehouse architecture
- Challenge 03: **[Data Pipeline Migration](./Challenge-03-Fabric.md)**
	 - Rewrite SSIS ETL jobs to Fabric Data Factory ELT pipelines
- Challenge 04: **[Real-time Data Pipeline](./Challenge-04-Fabric.md)**
	 - Real-time data with Eventstream and Eventhouse
- Challenge 05: **[Analytics Migration](./Challenge-05-Fabric.md)**
	 - Migrate reporting using Direct Lake mode and Fabric semantic models
- Challenge 06: **[Enterprise Security](./Challenge-06-Fabric.md)**
	 - Enterprise Security in Microsoft Fabric
- Challenge 07: **[Unified Data Governance](./Challenge-07-Fabric.md)**
	 - Data Governance with Microsoft Purview Hub in Fabric

## Prerequisites

- A Microsoft Fabric capacity (Trial, F2, or higher)
- A Microsoft Entra ID account with Fabric workspace Admin permissions
- Power BI Desktop (latest version with Fabric support)
- Visual Studio Code
- Azure CLI (for source database setup)
- Download WorldWide Importers Database (OLTP & OLAP)

## Technologies

Microsoft Fabric services and related products:
* Microsoft Fabric Data Warehouse
* Microsoft Fabric Lakehouse (OneLake + Delta Lake)
* Microsoft Fabric Data Factory (Pipelines + Dataflows Gen2)
* Microsoft Fabric Eventhouse (Real-Time Intelligence)
* Microsoft Fabric Spark Notebooks
* Power BI (Direct Lake mode)
* Microsoft Purview Hub

## Learning Path for Microsoft Fabric

- [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)
- [Microsoft Fabric Learning Path](https://learn.microsoft.com/en-us/training/fabric/)
- [Fabric Migration Guides](https://learn.microsoft.com/en-us/fabric/data-warehouse/migration-synapse-dedicated-sql-pool-warehouse)

## Contributors

- Alex Karasek
- Jason Virtue
- Annie Xu
- Chris Mitchell
- Brian Hitney
- Israel Ekpo
- Osamu Hirayama
- Alan Kleinert
- Prashant Atri
- *Fabric Edition adapted with AI assistance*
