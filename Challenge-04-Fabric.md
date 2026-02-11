# Challenge 04 - Real-time Data Pipeline

[< Previous Challenge](./Challenge-03-Fabric.md) - **[Home](./README-Fabric.md)** - [Next Challenge >](./Challenge-05-Fabric.md)

## Introduction

Worldwide Importers wants to build out their data platform to include clickstream data. There are a number of online stores that the marketing department wants to track for campaign and online ads. These marketing users want to monitor the clickstream data and have the ability to run exploratory data analysis to support ad-hoc research. This data needs to be in real-time so the campaigns and ads are timely based on user activity in the online stores.

## Description

Build a streaming pipeline to ingest simulated clickstream data into Microsoft Fabric using **Real-Time Intelligence** -- Fabric's integrated real-time analytics solution. This replaces the original architecture of Azure Event Hubs + Azure Databricks with Fabric's native **Eventstream** and **Eventhouse** components.

### Architecture Overview

The real-time pipeline will flow as follows:

```
Stream Generator → Azure Event Hub → Fabric Eventstream → Eventhouse (KQL Database) → KQL Queryset
```

### Step 1: Deploy Azure Event Hub (Data Source)

Deploy an Azure Event Hub to receive the simulated clickstream data:

- Create an Event Hub namespace and Event Hub in Azure
- Generate a Shared Access Policy for producing and consuming events
- Note the connection string and Event Hub name

### Step 2: Start the Stream Generator

Use the provided .Net application ([Stream Generator](https://github.com/alexkarasek/ClickStreamGenerator)) to generate simulated clickstream data.

**Start the stream using Azure Cloud Shell:**
```bash
az container create -g [Resource Group Name] \
    --name [container name] \
    --image whatthehackmsft/wwiclickstreamgenerator:1 \
    --environment-variables \
        'hostName'='[EH Namespace].servicebus.windows.net' \
        'sasKeyName'='RootManageSharedAccessKey' \
        'sasKeyValue'='[SAS Key]' \
        'eventHubName'='[Event Hub Name]'
```

### Step 3: Create an Eventhouse

In your Fabric workspace:
1. Create a new **Eventhouse** item
2. This automatically creates a **KQL Database** inside it
3. The Eventhouse is optimized for high-throughput ingestion and fast analytical queries using KQL (Kusto Query Language)

### Step 4: Create an Eventstream

Create a **Fabric Eventstream** to connect the Azure Event Hub to your Eventhouse:

1. In your Fabric workspace, create a new **Eventstream**
2. **Add Source**: Select "Azure Event Hub" and configure the connection:
    - Event Hub namespace, Event Hub name, SAS key
3. **Add Destination**: Select your KQL Database in the Eventhouse
    - Configure the target table (e.g., `ClickstreamRaw`)
    - Map the incoming fields to table columns
4. Optionally, add **transformations** in the Eventstream to filter or enrich data before it reaches the Eventhouse

### Step 5: Query Real-time Data with KQL

Create a **KQL Queryset** in your workspace to interactively query the streaming data:

```kql
// View the latest 100 clickstream events
ClickstreamRaw
| take 100

// Count events per product in the last 5 minutes
ClickstreamRaw
| where ingestion_time() > ago(5m)
| summarize EventCount = count() by ProductName
| order by EventCount desc

// Time series chart of events per minute
ClickstreamRaw
| where ingestion_time() > ago(1h)
| summarize EventCount = count() by bin(ingestion_time(), 1m)
| render timechart
```

### Step 6: Create a Real-Time Dashboard (Optional)

Create a **Real-Time Dashboard** in Fabric to visualize the streaming data:
1. In your workspace, create a new Real-Time Dashboard
2. Add tiles that query the KQL Database
3. Set auto-refresh intervals for near-real-time updates

## Success Criteria

- Eventstream is connected and actively receiving data from the Event Hub
- Data is being ingested into the Eventhouse KQL Database in real-time
- KQL queries return live clickstream data
- Demonstrate a KQL query that shows event counts over a recent time window

## Learning Resources

- [Eventhouse Overview](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/eventhouse)
- [Create an Eventstream](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/create-manage-an-eventstream)
- [Add Azure Event Hub Source to Eventstream](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/add-source-azure-event-hubs)
- [Add Eventhouse Destination to Eventstream](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/event-streams/add-destination-kql-database)
- [KQL Quick Reference](https://learn.microsoft.com/en-us/kusto/query/kql-quick-reference)
- [Real-Time Dashboards in Fabric](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/dashboard-real-time-create)
- [Stream Generator Source Code](https://github.com/alexkarasek/ClickStreamGenerator)

## Tips

- The Eventhouse ingests data in near-real-time (seconds of latency) -- much faster than batch loading
- KQL (Kusto Query Language) is optimized for log and telemetry analytics. Key operators:
    - `where` - filter rows
    - `summarize` - aggregate data
    - `render` - create visualizations (timechart, barchart, piechart)
    - `extend` - add calculated columns
    - `project` - select and rename columns
- The Eventstream provides a visual designer for connecting sources, transformations, and destinations
- You can use **Direct ingestion mode** in the Eventstream for raw data, or **Event processing mode** to apply transformations before ingestion
- Make sure the Event Hub SAS policy has both **Send** and **Listen** permissions
- You can monitor Eventstream metrics (events/sec, latency) from the Eventstream monitoring view

## Advanced Challenges (Optional)

Too comfortable? Eager to do more? Try these additional challenges!

- Create an **OneLake shortcut** from your Eventhouse to make the real-time data accessible from the Lakehouse
- Build a **Fabric Spark notebook** that reads from the Eventhouse using the Kusto Spark connector and joins clickstream data with warehouse dimension data
- Create a **Power BI report** that uses the KQL Database as a data source for real-time campaign analytics
- Configure **data retention policies** on the KQL Database to manage storage costs
- Use the **Fabric REST APIs** to programmatically configure the Eventstream and Eventhouse
