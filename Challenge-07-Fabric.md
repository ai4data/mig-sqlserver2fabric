# Challenge 07 - Unified Data Governance

[< Previous Challenge](./Challenge-06-Fabric.md) - **[Home](./README-Fabric.md)**

## Introduction

WWI wants a centralized view of all data assets and wants to make data sources discoverable and understandable to enterprise users. Currently, data analysts and engineers spend too much time on manual processes to annotate, catalog, and find trusted data sources. There's no central location to register data sources, and data consumers spend significant time tracing root cause problems from upstream pipelines owned by other teams.

They want to address these questions:
- How do users know what data is available?
- Does the data contain sensitive or personal information (PII, PCI, PHI, etc.)?
- How do administrators manage data when they may not know what types of data exist and where it's stored?
- How can one derive the end-to-end data flow across pipelines and reports?

## Description

The objective of this challenge is to implement data governance in Microsoft Fabric using the **Microsoft Purview Hub**, **OneLake Catalog**, and Fabric's built-in governance features. Unlike the original challenge that required provisioning a standalone Microsoft Purview account, Fabric provides integrated governance capabilities directly within the platform.

### Step 1: Explore the OneLake Catalog

The **OneLake Catalog** is Fabric's centralized discovery and governance interface:

1. Navigate to the OneLake Catalog from the Fabric portal navigation
2. Explore your workspace items -- you should see all Fabric items created in previous challenges:
    - Data Warehouse
    - Lakehouse
    - Pipelines
    - Eventhouse
    - Semantic Models
    - Reports
3. Use the **search and filter** capabilities to discover items:
    - Search by keyword (e.g., "City", "Sale")
    - Filter by item type, workspace, or endorsement status
4. Review the **Govern tab** in the OneLake Catalog:
    - View sensitivity label coverage
    - Check endorsement status across items
    - Review Data Loss Prevention (DLP) evaluation status

### Step 2: Configure Endorsement

Endorsement helps users identify trustworthy, high-quality data:

1. **Promote** items that are ready for broader use:
    - Navigate to an item (e.g., your Data Warehouse or semantic model)
    - Open Settings > Endorsement
    - Set the endorsement to "Promoted"
2. **Certify** items that meet organizational quality standards:
    - Certification must be enabled by a Fabric admin
    - Only authorized users can certify items
3. Verify that endorsed items appear with badges in the OneLake Catalog

### Step 3: Configure Domains

Domains enable **data mesh** organizational structures in Fabric:

1. If you have Fabric admin access, create a domain (e.g., "WWI Analytics")
2. Associate your hack workspace with the domain
3. Domains help visualize which workspaces belong to which business areas

### Step 4: Implement Data Lineage

Fabric automatically captures lineage for data movement and transformations:

1. **View item-level lineage**:
    - Navigate to your workspace
    - Switch to the **Lineage view** (available from the workspace view options)
    - You should see the data flow across items:
      ```
      Lakehouse → Pipeline → Data Warehouse → Semantic Model → Report
      ```
2. **Validate pipeline lineage**:
    - Create a simple **Copy Activity pipeline** that copies `[Fact].[Sale]` to `[Fact].[SaleCopy]` in the Data Warehouse
    - After execution, verify that the lineage view shows this data flow
3. **Review lineage in Microsoft Purview** (if connected):
    - If your organization has Microsoft Purview connected to Fabric, review the end-to-end lineage map that includes external data sources

### Step 5: Apply Sensitivity Labels and Classification

1. **Apply sensitivity labels** to Fabric items:
    - Navigate to each item and apply appropriate labels (e.g., "General", "Confidential", "Highly Confidential")
    - Labels should reflect the data sensitivity (e.g., "Highly Confidential" for items containing PII)
2. **Verify label propagation**:
    - Check that labels appear in the OneLake Catalog
    - Use the Govern tab to see label coverage across your workspace
3. **Scan for sensitive data** (if Microsoft Purview is configured):
    - Register your Fabric workspace as a data source in Microsoft Purview
    - Run a scan to automatically classify columns containing sensitive data (SSN, email, phone numbers, etc.)
    - Verify classification on the `Dimension.Employee` table -- specifically the "WWI Employee ID" column

### Step 6: Configure Data Loss Prevention (DLP)

If Microsoft Purview DLP policies are available in your tenant:

1. Create a DLP policy that detects sensitive information in Fabric items
2. Apply the policy to your Fabric workspace
3. DLP now extends to **Lakehouse items** -- upload a test file with simulated PII to verify detection
4. Review DLP alerts and policy matches in the compliance portal

### Step 7: Leverage the Microsoft Purview Hub

The **Purview Hub** in Fabric provides a centralized gateway to governance insights:

1. Navigate to the Purview Hub from the Fabric portal
2. Review the available reports and insights:
    - Sensitive data distribution across your workspace
    - Endorsement coverage
    - Domain associations
3. Use the Purview Hub to navigate to advanced Microsoft Purview capabilities (Data Catalog, Information Protection, Audit)

## Success Criteria

- **OneLake Catalog**: Demonstrate searching and discovering data items across your workspace using keywords and filters
- **Endorsement**: At least one item is Promoted and visible with its endorsement badge in the catalog
- **Lineage**: Show the end-to-end lineage view from Lakehouse through Pipeline to Data Warehouse to Semantic Model to Report
- **Sensitivity Labels**: At least two items have sensitivity labels applied, and labels are visible in the OneLake Catalog Govern tab
- **Classification** (if Purview is configured): Show classified columns in the `Dimension.Employee` table
- **DLP** (if available): Demonstrate a DLP policy match for sensitive data in the Lakehouse

## Learning Resources

- [Governance and Compliance in Fabric](https://learn.microsoft.com/en-us/fabric/governance/governance-compliance-overview)
- [OneLake Catalog](https://learn.microsoft.com/en-us/fabric/governance/onelake-catalog)
- [Microsoft Purview Hub in Fabric](https://learn.microsoft.com/en-us/fabric/governance/use-microsoft-purview-hub)
- [Endorsement in Fabric](https://learn.microsoft.com/en-us/fabric/governance/endorsement-overview)
- [Domains in Fabric](https://learn.microsoft.com/en-us/fabric/governance/domains)
- [Lineage in Fabric](https://learn.microsoft.com/en-us/fabric/governance/lineage)
- [Sensitivity Labels in Fabric](https://learn.microsoft.com/en-us/fabric/governance/information-protection)
- [DLP Policies for Fabric](https://learn.microsoft.com/en-us/fabric/governance/data-loss-prevention-configure)
- [Data Classification in Microsoft Purview](https://learn.microsoft.com/en-us/purview/data-classification-overview)

## Tips

- The **OneLake Catalog** replaced the earlier "Data Hub" -- it is the primary discovery interface in Fabric
- The **Govern tab** in the OneLake Catalog provides a dashboard view of governance health -- use this to demonstrate compliance posture to leadership
- **Lineage view** is available at the workspace level -- switch from the default "List" view to "Lineage" view
- Sensitivity labels require **Microsoft Purview Information Protection** to be enabled in your Microsoft 365 tenant
- DLP policies for Fabric are configured in the **Microsoft Purview compliance portal**, not within Fabric itself
- For custom classification of the Employee ID column, create a **custom sensitive information type** in the Microsoft Purview compliance portal with a regex pattern matching the employee ID format. Use `wwiemployee.csv` from the `/Challenge07/` folder of the `Resources.zip` for testing the regex pattern
- Endorsement is a Fabric-native feature that does not require Microsoft Purview -- it's the easiest governance feature to implement
- Use the **Fabric REST APIs** to programmatically manage governance:
    ```
    GET https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/items
    ```
- If Microsoft Purview is not fully configured in your tenant, focus on the Fabric-native governance features: OneLake Catalog, endorsement, lineage, and sensitivity labels

## Advanced Challenges (Optional)

Too comfortable? Eager to do more? Try these additional challenges!

- Integrate **Microsoft Purview Data Catalog** with your Fabric workspace for enterprise-wide data discovery
- Create a **custom classification rule** in Microsoft Purview and verify it detects the employee ID pattern in your Fabric Data Warehouse
- Use the **Fabric REST APIs** to programmatically apply endorsement and retrieve governance metadata
- Set up a **DLP policy** that blocks export of Highly Confidential labeled items from Fabric
- Configure **audit log** queries to track who accessed which data items and when
- Create a governance dashboard in Power BI that visualizes classification coverage, endorsement status, and DLP alerts across your Fabric estate
