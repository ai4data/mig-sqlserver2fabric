# Challenge 00 - Setup (Microsoft Fabric Edition)

**[Home](./README-Fabric.md)** - [Next Challenge >](./Challenge-01-Fabric.md)

## Introduction

The objective of this challenge is to setup your on-premise source databases and your Microsoft Fabric environment for the hack. This will be your starting point for the migration.

## Common Prerequisites

- A **Microsoft Fabric capacity** (Trial, F2, or higher). You can start a [free Fabric trial](https://learn.microsoft.com/en-us/fabric/get-started/fabric-trial) if needed.
- A **Microsoft Entra ID** account with permissions to create Fabric workspaces
- An **Azure Subscription** (for deploying the source SQL Server databases)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- [Azure Cloud Shell](https://shell.azure.com)

## Student Resources

Your coach will provide you with a `Resources.zip` file that contains resource files you will use to complete some of the challenges for this hack.

You will use some of the files in this package on your local workstation. Other files will be used in the Azure Cloud Shell. We recommend you unpack a copy of the `Resources.zip` file in both locations.

The rest of the challenges will refer to the relative paths inside the `Resources.zip` file where you can find the various resources to complete the challenges.

## Description

### Setup your Development Environment on your Laptop

For your local PC, ensure the following tools are installed:

1. [SQL Server Management Studio (Version 18.x or higher)](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) or [Azure Data Studio](https://learn.microsoft.com/en-us/azure-data-studio/download-azure-data-studio)
2. [Visual Studio Code](https://code.visualstudio.com/Download)
3. [Power BI Desktop](https://www.microsoft.com/en-us/download/details.aspx?id=58494) (latest version with Fabric support)

### Deploy Source Databases

WWI runs their existing database platforms on-premise with SQL Server 2019. There are two database samples for WWI. The first one is for their Line of Business application (OLTP) and the second is for their data warehouse (OLAP). You will need to setup both environments as our starting point in the migration.

For this challenge, you will deploy the WWI source databases using the provided deployment script and ARM Template. We **STRONGLY** recommend you complete this step using the Azure Cloud Shell.

You will find the provided deployment script (`hacksetup.sh`), ARM Template (`deployHack.json`), and parameters file (`deployHackParameters.json`) in the `/Challenge0/` folder of the `Resources.zip` file provided by your coach.

Navigate to wherever you have unpacked the `/Challenge0/` folder in your [Azure Cloud Shell](https://shell.azure.com) and complete the following steps:

1. Run the deployment script:
    ```bash
    # Make the file executable
    chmod +x hacksetup.sh
    # Run the script
    ./hacksetup.sh
    ```
    The script will prompt you for a resource name prefix, an Azure region, and a password for the Azure SQL Database.

    The script will deploy the following resources into Azure:
    - An Azure Container Instance with a SQL Server instance that has the WideWorldImporters and WideWorldImportersDW databases
    - Azure Data Factory (used only for SSIS runtime in Challenge 1 if following the lift-and-shift approach)

### Setup Microsoft Fabric Environment

1. **Enable Fabric Capacity**: Ensure your tenant has Microsoft Fabric enabled. If using a trial, activate it from [Microsoft Fabric Trial](https://learn.microsoft.com/en-us/fabric/get-started/fabric-trial).

2. **Create a Fabric Workspace**: Navigate to [app.fabric.microsoft.com](https://app.fabric.microsoft.com) and create a new workspace for this hack.
    - Name it something like `WWI-DataWarehouse-Hack`
    - Assign it to your Fabric capacity (Trial or paid)
    - Configure workspace settings as needed

3. **Review Fabric Components**: Familiarize yourself with the key Fabric items you will create during this hack:
    - **Data Warehouse** - SQL-based analytical data warehouse
    - **Lakehouse** - Open data lake with Delta Lake tables
    - **Data Factory Pipelines** - Orchestration and data movement
    - **Dataflows Gen2** - Low-code data transformations
    - **Eventhouse** - Real-time analytics engine
    - **Spark Notebooks** - Code-first data engineering

4. Review the database catalog on the source data warehouse for familiarity of the schema: [Reference document](https://docs.microsoft.com/en-us/sql/samples/wide-world-importers-dw-database-catalog?view=sql-server-ver15)

5. Review the ETL workflow to understand the data flow and architecture: [Reference document](https://docs.microsoft.com/en-us/sql/samples/wide-world-importers-perform-etl?view=sql-server-ver15)

## Success Criteria

- Source SQL Server databases are deployed and accessible
- Microsoft Fabric workspace is created and configured
- You can access the Fabric portal and create items in your workspace
- You understand the WWI database schema and ETL workflow

## Learning Resources

- [Get Started with Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/get-started/fabric-trial)
- [Create a Fabric Workspace](https://learn.microsoft.com/en-us/fabric/get-started/create-workspaces)
- [Microsoft Fabric Concepts and Licenses](https://learn.microsoft.com/en-us/fabric/enterprise/licenses)
- [Fabric Capacity and SKUs](https://learn.microsoft.com/en-us/fabric/enterprise/plan-capacity)
- [Decision Guide: Warehouse vs. Lakehouse](https://learn.microsoft.com/en-us/fabric/get-started/decision-guide-warehouse-lakehouse)
- [WWI Database Catalog](https://docs.microsoft.com/en-us/sql/samples/wide-world-importers-dw-database-catalog?view=sql-server-ver15)
