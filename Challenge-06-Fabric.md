# Challenge 06 - Enterprise Security

[< Previous Challenge](./Challenge-05-Fabric.md) - **[Home](./README-Fabric.md)** - [Next Challenge >](./Challenge-07-Fabric.md)

## Introduction

World Wide Importers (WWI) leadership is happy with the progress of the migration thus far but wants to ensure that information security guard rails are implemented appropriately. Additionally, they want to understand the art of the possible when it comes to enterprise data security in Microsoft Fabric. This will be done in the form of a stakeholder update at the end of the exercise.

## Description

The objective of this challenge is to implement enterprise-grade security controls in Microsoft Fabric. Fabric provides a layered security model that spans workspace access, item-level permissions, data-level security, and information protection.

Categories of security enablement are as follows:

- **Workspace Security & Access Control**
- **Row-Level Security (RLS)**
- **Column-Level Security (CLS)**
- **Dynamic Data Masking**
- **Sensitivity Labels (Information Protection)**
- **Managed Identity & Authentication**
- **Auditing & Monitoring**

### Workspace Security & Access Control

Fabric uses **workspace roles** as the primary access control mechanism:

| Role | Permissions |
|---|---|
| **Admin** | Full control, manage membership, delete workspace |
| **Member** | Create, edit, delete items; share items; manage permissions |
| **Contributor** | Create, edit, delete items |
| **Viewer** | View and read items only |

Implement the following:
1. Assign appropriate workspace roles to different user groups
2. Demonstrate the principle of least privilege by assigning Viewer role to consumers and Contributor role to data engineers
3. Use **Microsoft Entra ID security groups** for role assignments rather than individual users

### Row-Level Security (RLS)

Implement RLS on `[Fact].[Sale]` in the Fabric Data Warehouse to restrict data access:

1. Create test users in the Fabric Data Warehouse:
    ```sql
    -- Create users from Entra ID accounts (or use existing accounts)
    CREATE USER [WWIConsultantNJ@yourdomain.com] FROM EXTERNAL PROVIDER;
    CREATE USER [WWIConsultantAL@yourdomain.com] FROM EXTERNAL PROVIDER;
    CREATE USER [sqlauditadmin@yourdomain.com] FROM EXTERNAL PROVIDER;

    -- Grant read access
    GRANT SELECT ON SCHEMA::Fact TO [WWIConsultantNJ@yourdomain.com];
    GRANT SELECT ON SCHEMA::Fact TO [WWIConsultantAL@yourdomain.com];
    GRANT SELECT ON SCHEMA::Dimension TO [WWIConsultantNJ@yourdomain.com];
    GRANT SELECT ON SCHEMA::Dimension TO [WWIConsultantAL@yourdomain.com];
    ```

2. Create a security predicate function and policy:
    ```sql
    -- Create schema for security objects
    CREATE SCHEMA Security;
    GO

    -- Create predicate function
    CREATE FUNCTION Security.fn_securitypredicate(@CityKey AS INT)
    RETURNS TABLE
    WITH SCHEMABINDING
    AS
    RETURN SELECT 1 AS fn_securitypredicate_result
    WHERE
        -- NJ consultant sees only North Beach Haven
        (DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('WWIConsultantNJ@yourdomain.com')
            AND @CityKey IN (SELECT [City Key] FROM [Dimension].[City] WHERE [State Province] = 'New Jersey' AND [City] = 'North Beach Haven'))
        OR
        -- AL consultant sees only Robertsdale
        (DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('WWIConsultantAL@yourdomain.com')
            AND @CityKey IN (SELECT [City Key] FROM [Dimension].[City] WHERE [State Province] = 'Alabama' AND [City] = 'Robertsdale'))
        OR
        -- Admin sees everything
        (DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('sqlauditadmin@yourdomain.com'))
        OR
        -- Workspace admin/owner sees everything
        (IS_MEMBER('db_owner') = 1);

    -- Apply security policy
    CREATE SECURITY POLICY SaleFilter
    ADD FILTER PREDICATE Security.fn_securitypredicate([City Key]) ON [Fact].[Sale]
    WITH (STATE = ON);
    ```

3. **Important:** Any users not accounted for in the security predicate will be unable to see the data. Always include an admin escape clause.

### Column-Level Security (CLS)

Restrict access to the "Profit" column in `[Fact].[Sale]`:

```sql
-- Deny access to Profit column for consultants
DENY SELECT ON [Fact].[Sale] ([Profit]) TO [WWIConsultantNJ@yourdomain.com];
DENY SELECT ON [Fact].[Sale] ([Profit]) TO [WWIConsultantAL@yourdomain.com];
```

### Dynamic Data Masking

Mask tax-related columns so consultants see zeroed-out values:

```sql
ALTER TABLE [Fact].[Sale] ALTER COLUMN [Tax Rate] ADD MASKED WITH (FUNCTION = 'default()');
ALTER TABLE [Fact].[Sale] ALTER COLUMN [Tax Amount] ADD MASKED WITH (FUNCTION = 'default()');
ALTER TABLE [Fact].[Sale] ALTER COLUMN [Total Excluding Tax] ADD MASKED WITH (FUNCTION = 'default()');
ALTER TABLE [Fact].[Sale] ALTER COLUMN [Total Including Tax] ADD MASKED WITH (FUNCTION = 'default()');
```

### Sensitivity Labels (Information Protection)

1. Apply **Microsoft Purview Information Protection sensitivity labels** to Fabric items:
    - Navigate to your Data Warehouse or semantic model in the Fabric portal
    - Apply a sensitivity label (e.g., "Confidential" or "Highly Confidential")
2. Sensitivity labels **persist when data leaves Fabric** (exports to Excel, PDF, PowerPoint)
3. Labels can be applied manually or through automated policies

### Managed Identity & Authentication

1. Enable **Workspace Identity** for your Fabric workspace:
    - Fabric creates a managed service principal for the workspace
    - This identity can be used for secure data access without storing credentials
2. Demonstrate using the Workspace Identity in a pipeline:
    - Create a pipeline that copies `[Fact].[Sale]` to `[Fact].[SaleCopy]` using the workspace's managed identity for authentication
3. All authentication in Fabric is through **Microsoft Entra ID** -- there are no SQL Server authentication logins

### Auditing & Monitoring

1. Fabric provides built-in audit logging through the **Microsoft 365 audit log** and **Fabric admin monitoring**:
    - Navigate to the Fabric Admin Portal > Audit Logs
    - Review user activities: who queried which tables, when, and from where
2. Set up **alerts** using Azure Monitor or Microsoft 365 compliance center:
    - Create an alert for failed authentication attempts
    - Configure email/SMS notifications

## Success Criteria

**Workspace Security**
- Demonstrate different access levels by testing with users assigned to different workspace roles

**Row-Level Security**
- Validate that `WWIConsultantNJ` can only see rows for North Beach Haven, NJ
- Validate that `WWIConsultantAL` can only see rows for Robertsdale, AL
- Validate that `sqlauditadmin` can see all rows

**Column-Level Security**
- Validate that consultants cannot see the "Profit" column
- Validate that admin users can see all columns

**Dynamic Data Masking**
- Validate that tax columns show masked/zeroed values for consultants
- Validate that admin users see the actual values

**Sensitivity Labels**
- Demonstrate a sensitivity label applied to at least one Fabric item
- Show that the label persists when exporting data

**Managed Identity**
- Validate that a pipeline can access data using the workspace managed identity
- Describe the security benefits of managed identity over stored credentials

**Auditing**
- Show audit log entries for recent data access activities

## Learning Resources

- [Fabric Data Warehouse Security Overview](https://learn.microsoft.com/en-us/fabric/data-warehouse/security)
- [Workspace Roles](https://learn.microsoft.com/en-us/fabric/get-started/roles-workspaces)
- [Row-Level Security in Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/row-level-security)
- [Column-Level Security in Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/column-level-security)
- [Dynamic Data Masking in Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/dynamic-data-masking)
- [Sensitivity Labels in Fabric](https://learn.microsoft.com/en-us/fabric/governance/information-protection)
- [Workspace Identity](https://learn.microsoft.com/en-us/fabric/security/workspace-identity)
- [Fabric Admin Monitoring](https://learn.microsoft.com/en-us/fabric/admin/monitoring-workspace)
- [Microsoft Entra ID Authentication](https://learn.microsoft.com/en-us/fabric/security/security-overview)

## Tips

- Fabric Data Warehouse uses **Microsoft Entra ID exclusively** for authentication -- there are no SQL logins or passwords to manage
- When testing RLS, use `EXECUTE AS USER = 'username@domain.com'` to impersonate users:
    ```sql
    EXECUTE AS USER = 'WWIConsultantNJ@yourdomain.com';
    SELECT TOP 10 * FROM [Fact].[Sale];
    REVERT;
    ```
- Sensitivity labels require Microsoft Purview Information Protection to be configured in your tenant
- Workspace Identity is automatically managed by Fabric -- no need to create or rotate credentials manually
- For programmatic security configuration, use the **Fabric REST APIs**:
    ```
    PATCH https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/roleAssignments
    ```
- Dynamic Data Masking applies to query results -- the underlying data is not modified
- RLS and CLS in Fabric Data Warehouse work the same way as in Azure SQL Database

## Advanced Challenges (Optional)

Too comfortable? Eager to do more? Try these additional challenges!

- Configure **OneLake data access roles** for fine-grained file-level security in the Lakehouse
- Set up **Data Loss Prevention (DLP) policies** using Microsoft Purview to detect sensitive data in your Lakehouse
- Automate security role assignments using **PowerShell** or the **Fabric REST API**
- Implement RLS in the **Power BI semantic model** in addition to the Data Warehouse for defense-in-depth
- Configure **Conditional Access policies** in Microsoft Entra ID for your Fabric workspace
