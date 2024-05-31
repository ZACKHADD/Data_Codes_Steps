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

- Security Roles in SQL server are not RLS. And RLS are not inhereted when import mode is used, only when direct query is used.
- Use large semantic model storage format when we deal with large datasets (It is a feature in power bi Premium).
- Composite models are best suited for various data sources when we search for best performance in PBI.
- OLS testing is only supported in Power BI Descktop or using Tabular Editor.
- OLS and RLS are only applied to Viewer roles.
- Semantic models using incremental refrech connot be downlowed from the service.
- Incremental refresh policices are defined in Power BI Descktop (Start and end date, filter data and set the incremental refresh points).
- Dimension tables are set to dual **(Data will be retrieved by the engine either from import mode or directquery depending on the query and how often data is queried)** mode storage while fact tables to Direct query and aggregations to Import.
- Enable Composite models in POwer BI we must: Allow XMLA Endpoints and Analyze in Excel with on-premises semantic models. Users can work with Power BI semantic models in Excel using a live connection. Allow DirectQuery connection to Power BI semantic models.
- Calculation groups can be used to calculate dynamicaly conversion rate.
- RLS can be defined using DAX while OLS uses only Roles and can be defined only using Tabular Editor.
- All Power BI Import and non-multidimensional DirectQuery data sources can work with aggregations.
- Tabular editor supports parallele development using GIT, And tabular Editor 3 can query data using DAX.
- Tabular Editor is not used for creating relationships between tables.
- Example of XMLA endpoint of a workspace : powerbi://api.powerbi.com/v1.0/myorg/Test%20Workspace
- To use RLS with aggregation tables it should be set on both detailed and aggregations.
- Large semantic models in Power BI are only available in Azure regions that support Azure Premium Files Storage.
- Dynamic format strings (apply to measures and not visuals) are not the same as conditional formatting (colors).
- We should whenever possible create RLS in the data warehouse in Fabric.
- Under DirectLake, the semantic model is refreshed but only the metadata and the cashed data (most used in queries, the engine handels this automaticaly) and the refresh can be triggered each time the underlying data changes.
- letting Power BI perform Automatic aggregations optimize query performance.

## Explore and Analyze data:

- Decision tree analysis for diagnostic analysis (why something happend).
- Visual Query editor supports execution of Data Query Language or Read-Only Select statement (DDL, DML are not supported), it includes intellisense, code completion .. and it has data preview features for data preview with a limitation of 1000 row.
- Even Excel can connect to parquet files.
- Visual query generates optimized SQL queries.
- Visual and SQL Query editor can be saved for future used.

 ## Miscellaneous Questions:

 - V-Order and Optimize Writes helps in making reads more efficient.
 - Partitionby in notebooks performs a split of data accross multiple folders. Partitioning is a way to improve the performance of query when working with a large semantic model. Avoid choosing a column that generates too small or too large partitions. Define a partition based on a set of columns with a good cardinality and split the data into files of optimal size.
 - Unmanaged tables are external tables (metadata handeled in spark but storage is external). Managed tables are normal tables.
 - Primary keys in fact tables are not to use (not easily compressed especialy in large tables).
 - For direct query models we assume referential integrity so that the engine perform inner joins instead of outer joins which more efficient.
 - F64 is the least Premium license (suitable to 1500 viewers with no pro licenses).
 - DAX studio gives statistics regarding model size and columns.

## Assessment exam:

- To use Lakehouse explorer, users need at least contributor role (viwers cannot).
- Limitation of XMLA endpoint in POwer BI : PBIX file cannot be downloaded from the Power BI service.
- Renaming and other query logics can be done using Views.
- While you can add the SQL endpoint of a lakehouse to a warehouse for cross database querying, that is not the simplest method. The simplest method is to use a shortcut.
- Dataflow Gen2 is a great tool for transformations; however, for large data ingestions without any transformations, the Copy data activity performs best.
- Only notebooks and SQL stored procedures provide a possibility to define parameters in the data pipeline UI.
- SCD2 keeps tracks of history without adding columns for previous values like SCD3 does.
- A staging dataflow copies raw data “as-is” from the data source and can then be used as a data source for further downstream transformations. This is especially useful when access to a data source is restricted to narrow time windows and/or to a few users.
- OPTIMIZE has a parameter to enable V-ORDER to increase direct lake speed.
- The high concurrency mode for Fabric notebooks is set at the workspace level.
- Table statistics need to be updated because the SQL query optimizer tries to use it to enumerate all possible plans and choose the most efficient candidate.
- When manually creating/updating statistics for optimizing query performance, you should focus on columns used in JOIN, ORDER BY, and GROUP BY clauses.
- Some Microsoft Power Query transformations break query folding, which causes all the data to be pulled into Power Query before it is filtered. Splitting a column by position (or delimiter) breaks query folding and causes all the data to load before the current year filter is applied.
- Always in power query filter data before transforming (spliting) to not break the query folding when the source is SQL database.
- Dynamic measure formatting is the simplest and most effective way to add logic-based formatting to a single measure.
- The large semantic model storage format can be enabled from the Power BI service from the semantic model settings. It will allow data to grow beyond the 10-GB limit for Power BI premium capacities or Fabric capacities of F64 or higher.
- Declaring the measures as VAR will cache the measure and only load it once. This reduces the amount of processing and data loading that the measure must do and increases its speed.
- The ALM Toolkit is a schema diff tool for Power BI models and can be used to perform the deployment of metadata only. We can use it to update measures without refreshing the semantic model.
- Filtering on the Calendar Dimension table will almost always perform faster than filtering directly on any fact table, as that requires more processing by both the DAX formula and the storage engine.
- Only the distinct values displayed under Column distribution will show the true number of rows of values that are distinct (one row per value).
- df.groupBy("ProductKey").count().sort("count", descending=True).show()
- display(df.limit(100))
- describe is used to generate descriptive statistics of the DataFrame. For numeric data, results include COUNT, MEAN, STD, MIN, and MAX, while for object data it will also include TOP, UNIQUE, and FREQ.
- You can profile your dataframe by clicking on Inspect button.
- plt.bar(x=data['SalesTerritory'], height=data['CityCount']) plt.xlabel('SalesTerritory') plt.ylabel('Cities') plt.show().
- DENSE_RANK() function returns the rank of each row within the result set partition, with no gaps in the ranking values. The RANK() function includes gaps in the ranking.

## Exam topics:

- Only Azure Devops Repo is supported for now in Fabric, Github is not.
- In Fabric, a domain is a way to logically group together services in the organization,we must use Fabric Admin portal to do the groupings.
- Data flows in ADF are not the same as DataflowsGen2 in Fabric.
- Incremental refresh in lakehouse can be done using Dtaflows Gen2 in fabric by adding a query that retrieves the maximum Order ID to filter data to ingest from the source.
- Read data fromlakhouse in notebook: A. spark.read.format(“delta”).load(“Tables/ResearchProduct”) B. spark.sql(“SELECT * FROM Lakehouse1.ResearchProduct ”).
- The SQL analytics endpoint is a read-only warehouse.
- Quickvisualisation creates a powerBI report inside notebooks.
- external tables created using spark are not visible when using SQL endpoint.
- to make the data available in the Chart view in a notebook we use display() function.
- XMLA is set to Read-Only first, you must go to the capacity settings to enable read-write.
- 

