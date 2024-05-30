# Overview of questions of the DP-600 exam

## Plan, Implement and Manage:

- If we want to be efficient in cost and perform light Fabric workloads with an access to 20 Power BI users to view reports we can purshase an F2 (200 $/month), F4 or F8 and 20 Pro license (10 $/month) but not a F64 (it costs 5000 dollars a month).
- When manipulating models using XMLA (Read & Write) it is recomanded to enable large Model support if the size exeeds 1GB, it operates faster.
- Applying sensitivity labels makes sure that only authorozed people can access Data.
- To be cost effective if we have F64 reserved capacity and avoid being throttled, we need to utilize autoscaling with a preset cost limit.
- Orphaned workspaces are workspaces with no admin users.
- Lineage analysis gives impact on artifacts in the same Workspace while Impact analysis shows it accross workspaces.
- Protocol used in Gateway is TCP.
- Users are managed in the 365 admin center.
- Export to excel can be disabled for some or all security groups at the tenant level or at the report level in the option section.
- In Microsoft INTRA ID Fabric administrator duties can be done if we have Global Administrator, Power Plateform Administrator or Fabric Administrator roles.
- Query Insights in data warehouse track user queries (not internal queries).
- Timeout in Dtaflows Gen 2 with gateway is 60 minutes.
- Label sensitivity can be applied in all artifacts in a workspace (Power BI service/Fabric) (even dataflows).
- Query insights keeps tracking up to 30 days.
- Creating workspaces role is handeled in the tenant level (user must be in the security goupe having this role to create workspaces). We can only update and delete workspace at workspace level if we are admins.
- Connect (or disconnect) workspace to azure repository (GIT) is done by Workspace Admin or Power BI Admin.
- When data leaves the service of Power BI (exported), it is automatically encrypted.
- Sensitivity labels are not supported in PBIP files and we can not create PBIP files from service.
- Pipelines permissions are : Pipeline admin and workspace roles.
- To avoid single point of failure in gateway, we must implement a data gateway cluster.
- Lineage view in power BI is not available for viewer users and in fabric they can't see data sources.
- Workspace retention period (after delete) is configured in Fabric Admin Portal : Default 7 days and Max 90 days.
- To give access to a shared lakehouse to be accessed using Apache Spark we need **ReadAll** permission, ReadData gives only access using SQL.
- You can only connect a workspace to one branch and one folder at a time.
- Deployment Pipeline : Step 1 - Create a deployment pipeline, Step 2 - Assign a workspace, Step 3 - Make a stage public (optional), Step 4 - Deploy to an empty stage, Step 5 - Deploy content from one stage to another, Step 6 - Create deployment rules (optional)
- To avoid 48 hours limit of refresh in power BI, we can set a powershell code to refresh using XMLA endpoint.
- Fabric capacity is created in Azure.
- XMLA read-write access can be configured by the capacity admin or the power bi admin.
- If the user specify colors for data points, it overrides the theme.
- Connot connect to Semantic model in My Workspace or in Excel Workbook using XMLA.
- SSO connection mode is configured in the Fabric Admin Portal.
- To apply sensitivity Labels in PBI : you must have a Power BI Pro or Premium Per User (PPU) license, Sensitivity labels must be enabled for your organization, You must belong to a security group that has permissions to apply sensitivity labels.
- Always use security groups to give access to workspaces and settings.
- Workspaces in Fabric can contain a maximum of 1,000 semantic models, or 1,000 reports per semantic model.
- PBIDS stores data source with all specifications but it might asks for credentials even if they are embeded. And we can only view connection details in the file.
- Power BI Service Admin can : Change capacity size, Add or remove capacities, Add or remove users that have assigned permissions.
- We can access Fabric admin portal only if we are : Global Administrator, Power Plateform Administrator or Fabric Administrator roles otherwise we only see **Capacity Settings.**
- A workspace is assigned to a capacity and we can find that in the admin portal under the workspaces section.
- Data gateway not required for SQL databases in azure.
- Only Admins and Members that can publish & unpublish PBI application. Updating may be given to contributors.

## Prepare and Serve Data:

- To track historical data in a dimension use SCD 2 dimensions.
- Implement SCD2 : Add Valid to and valid from columns, Create new surrogate key to uniquely identify the record, Modify the ETL to insert a new line and not overwriting it and populate the Valit to of the previous row with the date of the change (Date of insert).
- Optimal way to promote code reusibility for complexed queries is User-Defined-Functions (UDFs).
- The most efficient and fast way to ingest data in DW is T-SQL Copy statement.
- You can configure data partitioning and test aggregation and DAX measures using DAX studio and Tabular Editor.
- Partition data for refresh in data models helps in performance (recent partition is processed every night, others monthly maybe and old once a year for example).
- The table maintenance commands **OPTIMIZE** (Reduce the amount of delta parquet files and hence metadata overhead) and **VACUUM** can be used within notebooks and Spark Job Definitions, and then orchestrated using platform capabilities.
- **spark.microsoft.delta.optimizeWrite.binSize** is the property to fix the size of delta parquet files generated when writing.
- Admin, Member and contributor can clone tables.
- To denormalize DB we should first analyze queries that are slow.
- Partitioning large tables based on most used columns in queries is a good startegy to enhance SQL queries performance.
- Nomalization is done primarily for Integrity and consistency for the application.
- Lakehouses offer code-free and code first ingestion options while warehouses primarily rely on built-in data ingestion tools.
- KQL Realtime analytics handle continuous stream of data and makes it possible to store it in a data warehouse.
- Shortcuts are created in lakehouses and KQL databases.
- You can set the 'Degree of copy parallelism' setting in the Settings tab of the Copy activity to indicate the parallelism you want the copy activity to use. Think of this property as the maximum number of threads within the copy activity. The threads operate in parallel. The threads either read from your source or write to your destination data stores.
- Semantic models and Azure SQL DataBases connot be used as sources in in shortcuts.
- Bridge tables when a Many-to-Many relation is needed.
- Start schema supports complexed relationships.
- Stored procedure available in Lakehouses but only for read.
- The highest throughput (performance) possible in ingestion is T-SQL COPY statement.
- Only Admin, Member and contributor roles can create shortcuts.
- Cross-warehouse ingestion sources must be in the same workspace.
- Cross-database queries are not supported between warehouses in different workspaces.
- Table clones create copies from a specific time **Only in the past 7 days** of the existence of the source table (time travel).
- Normalization: 2NF all non-key attributes depend on primary key (or part of composite key), 3NF all non-key attributes depend on one Primary key (non indirect dependency with other primary keys in the table).
- Normalization does not always decrease performance (it might).
- Conversion between numerical data types and data/time is not supported in Fabric. Also conversion from decimal to date.
- Hierarchical partitioning with different levels helps in effecient queries.

## Building and design semantic models:

-
