# This document gives a recap of some questions in the DP-203 exam

- When we create a database in Spark pool using a notebook using parquet for example, we can later on query the data in serverless pool.
- Dealing with partitions in synapse is faster than row per row operations (because of logs mainly). If we want to delete a partition, we can create an empty copy of the table, swith the corresponding partition from the original table to the copy table and finaly drop the table. The SWITCH PARTITION operation is a metadata-only that is faster then the deletion operation since It avoids the overhead of row-by-row operations, making it highly efficient. Why Is This Faster? No Data Movement: The SWITCH PARTITION operation is purely a metadata update and avoids moving rows physically. Also Efficient Partition Management: Instead of scanning the table to locate and delete rows (DELETE), it simply updates the metadata about which table owns the partition.

```SQL
    -- Create the empty table
        CREATE TABLE [schema].[staging_table]
        WITH
        (
            DISTRIBUTION = HASH(partition_column), -- Match the distribution of the original table
            PARTITION (partition_column RANGE RIGHT FOR VALUES (...)) -- Match the partition scheme
        )
        AS
        SELECT *
        FROM [schema].[partitioned_table]
        WHERE 1 = 0; -- Ensures the table is empty
    -- Switch the partition where partition_number is the ID of the partition
        ALTER TABLE [schema].[partitioned_table]
        SWITCH PARTITION partition_number TO [schema].[staging_table];
    -- Drop the new table
      DROP TABLE [schema].[staging_table];

    -- Update the stats and reclaim the unused space with vaccum
      UPDATE STATISTICS [schema].[partitioned_table];
      VACUUM [schema].[partitioned_table];
```
- Accessing files when creating external tables is different depending on which sql pool we use: The Serverless one needs the wildcard to be specified if we want all the files to be included even in the subfolders but the dedicated one scan the whole folder and assumes that all the files are part of the table (even the subfolders).
- Iwe want to fastly read columns from files in ADLS GEN 2 we need PARQUET format since it's column oriented but if we want fast read of rows with timestamp we need AVRO (It uses a json format with timestamps).
- Replicated tables for Dimensions and Hashed distributed ones for Facts.
- Steps to give access to SQL pool to an ADLS GEN2 accessible through virtual network and by a specific group : Create a managed identity (mandatory), add it to the AAD group and then use thr managed identity as credentials in SQL pool (avoid SAS).
- What will a user not authorized to see masked data will have as output when querying this data :
  
|Data Type|	Masked Value (Default Masking)|
|----|-----|
|String (e.g., NVARCHAR)	|Replaced with XXXX (four X characters).|
|Numeric (e.g., INT, BIGINT)|	Replaced with 0.|
|Datetime	|Replaced with 1900-01-01 00:00:00.000.|
|Binary	|Replaced with a single byte value 0x00.|

- External tables use metadata to poiint to file in a data lake, that is why we can not alter them diectly. If want to do that we need to drop the external table and recreate it.
- Using Parquet for both source and sink in ADF without transformations ensures the fastest and most efficient copy operations, leveraging the format's optimized storage, compression, and native support in ADF. However Binary can be faster than Parquet When :  
    - No Parsing Overhead: Since ADF does not parse or deserialize the file content, copying binary data eliminates the processing overhead associated with formats like Parquet.
    - Direct File Transfer: Binary mode is effectively a raw file copy, meaning the data is moved directly from source to sink without processing columnar structures or schemas.
    - Compression Compatibility: Binary datasets allow compressed files (e.g., .gzip, .zip) to be transferred as-is without decompression.
 
- Choosing the Right Redundancy :
  - Locally Redundant Storage (LRS):
    - Data is replicated three times within a single data center in the same Azure region.
    - Ensures high durability and protects against server or drive failures within the same location.
  - Zone-Redundant Storage (ZRS):
    - Data is replicated synchronously across three availability zones within the same Azure region.
    - Offers higher availability and fault tolerance compared to LRS.
  - Geo-Redundant Storage (GRS) : 
    - Data is replicated three times in the primary region (LRS) and then asynchronously replicated to a secondary region hundreds of miles away.
    - The secondary region serves as a disaster recovery option in the event of a primary region outage.
  - Read-Access Geo-Redundant Storage (RA-GRS):
    - Builds on GRS by providing read access to the secondary region.
    - Data is replicated both locally in the primary region (LRS) and asynchronously to a secondary region. The secondary region can be used for read-only operations during normal operations or outages.

- For fast loading in staging tables : Choose Round-Robin distribution, Heap tables (no index) and no partitionning (especially if the staging table gets truncated).
- Use column store index : Suitable for production **fact tables** after the data is loaded and when query performance is a priority. Also Use for incremental loads where query efficiency outweighs loading speed.
- For transactional worloads, Clustered index is preferble.
- Indexes :
  - Column store index (Default) ==> Stores data in columnar format (best for analytical workloads).
  - Clustered index ==> Stores data in row format (transactional worloads).
![image](https://github.com/user-attachments/assets/7d8a9dd2-896f-41ae-bc72-77f2d1a1fbdf)  
- When creating the table, the distribution of type Hash should not be done on a date column or columns that are used in where clause!
- PolyBase can't load rows that have more than 1,000,000 bytes of data. When you put data into the text files in Azure Blob storage or Azure Data Lake Store, they must have fewer than 1,000,000 bytes of data. This byte limitation is true regardless of the table schema. All file formats have different performance characteristics. For the fastest load, use compressed delimited text files. The difference between UTF-8 and UTF-16 performance is minimal. We shoul also split large compressed files into smaller compressed files.
- If a complexe query will be frequently used we can create a materialized view and ADF, Synapse Pipelines or Automation account to schedule the refresh.
- For each Spark external table based on Parquet or CSV and located in Azure Storage, an external table is created in a serverless SQL pool database. As such, you can shut down your Spark pools and still query Spark external tables from serverless SQL pool.
- If a processing needs to be donce using Java or scala Databricks is the best choice.
- Hierarchy : Organize your data using logical folder structures to improve discoverability and reduce unnecessary scanning.
- Merging csv files is suiatable for large processing.
- If the tables are using row-level security , the query results don't get cached even if wa turn on result-set caching for datapool.
- Run the DBCC PDW_SHOWSPACEUSED command against the table to view the data skew.
- Minimizing storage costs when implementing batch datasets in the Parquet format using Azure Data Factory (ADF) use Snappy Compression (Default).
- number of partition ranges that provides optimal compression and performance for the clustered columnstore index :
  - Row Group Size and Compression Efficiency:
    - Clustered columnstore indexes organize data into row groups, each containing up to 1,048,576 rows (1 million rows) for optimal compression.
    - If partitions are too small (fewer rows per partition), row groups may be incomplete, leading to poor compression and increased storage overhead.
  - However excessive partitioning can increase metadata overhead and slow query performance.
  - Aim to have at least 1 million rows per partition per distribution to maximize row group efficiency and compression, meaning at least 60 million rows per partition (1 million rows × 60 distributions).
- In Azure Synapse Dedicated SQL Pool, data distribution is applied before partitioning. Order of Operations:
    - Distribution:
        - When data is loaded into a table, the first step is distributing it across the 60 underlying distributions. This step ensures that data is spread across the nodes in the pool based on the table's distribution type (Hash, Round-Robin, or Replicated).
        - Hash-distributed tables: Data is assigned to distributions based on the hash value of the distribution column.
        - Round-robin distributed tables: Data is assigned randomly and evenly across all distributions.
        - Replicated tables: Entire table is copied to all distributions.
    - Partitioning (within distributions):
        - Once data is distributed to a distribution, partitioning is applied locally within each distribution if the table is partitioned.
        - Each distribution manages its own partitions based on the partition column and the defined partition scheme.
        - Partitioning organizes the data within a single distribution into smaller subsets.
        - Note that unlike physical partitioning in some systems, in synapse, partitions are logical and are simply metadata constructs that help optimize how data is queried and processed. These metadata records the physical data location.
- Differences Between Distribution and Partitioning :
  ![image](https://github.com/user-attachments/assets/3700f1a5-07a2-48d0-a46a-1862e81cb78a)
  - Choosing the Right Column Types for distribution : choose ==> High Cardinality, Join Columns. Avoid ==> Low Cardinality, Null-heavy Columns and Round-Robin Distribution (use only when no candidat column or when staging).
  - Partition Columns : Date/Time Columns (even if the type is integer), Range-based Filters and High Cardinality. Avoid : Low Cardinality, Frequently Updated Columns (Updating partitioning columns can lead to expensive data movement)
- Denormalizing from 3NF to 2NF is a good practice in datawarehousing.
- Streaming in Azure : Azure event hubs & Azure stream analytics :
  - ![image](https://github.com/user-attachments/assets/5716aba2-d25d-4316-b1e1-3866fb200576)
  - How They Work Together:
    - Event Hubs often serves as the data source for Stream Analytics. For instance:
      - Event Hub receives raw telemetry data from IoT devices.
      - Stream Analytics processes this data in real-time (e.g., detecting anomalies, aggregating statistics).
      - The processed data is then sent to a destination such as a dashboard in Power BI or a storage account.
- We can get information to monitor blocked queries fired against a dedicated SQL by firing a query against the system view - sys.dm_pdw_waits.
- The runs in Azure Data Factory are only maintained for 45 days. If we need data beyond this, we will need to stream the pipeline run data into a Log Analytics workspace.
    ![image](https://github.com/user-attachments/assets/adbc609b-7e3f-4ad8-9947-e0130867d28c)
    ![image](https://github.com/user-attachments/assets/90958237-8187-4120-a41c-44a124c88e01)
- If the Backlogged events in your Azure Stream Analytics job show a non-zero value consistently, it indicates that the job cannot keep up with the incoming data volume ==> Increase the Streaming Units (SUs) for the job.
- Avro is a compressed row based file format. And here the format is optimized for retrieving multiple rows of records in its entirety.
- We can use LOOKUP activity in ADF to call the result of a stored procedure.
- Use CSV as file type to query json files in azure serverless pool using OPENROSET.
- if we want to keep the configuration of the cluster even after it is terminated, we need to pin the cluster.
- Ensure to output events only for points in time when the content of the window actually changes : Sliding window.
- Ensure to group events that arrive at similar times : Session window.
- The WHERE clause in the SQL statements would make use of the partitions.
- Access tiers policy in ADLS GEN 2:
  - Hot tier : An online tier optimized for storing data that is accessed or modified frequently. The hot tier has the highest storage costs, but the lowest access costs.
  - Cool tier : An online tier optimized for storing data that is infrequently accessed or modified. Data in the cool tier should be stored for a minimum of 30 days. The cool tier has lower storage costs and higher access costs compared to the hot tier.
  - Cold tier : An online tier optimized for storing data that is rarely accessed or modified, but still requires fast retrieval. Data in the cold tier should be stored for a minimum of 90 days. The cold tier has lower storage costs and higher access costs compared to the cool tier.
  - Archive tier : An offline tier optimized for storing data that is rarely accessed, and that has flexible latency requirements, on the order of hours. Data in the archive tier should be stored for a minimum of 180 days.
- Polybase does not support JSON file format.
- Polybase is designed for loading data effeciently from external sources:
  ![image](https://github.com/user-attachments/assets/2602764d-821c-4cbc-b642-a8c7844b20b2)
![image](https://github.com/user-attachments/assets/43c7ce7f-bb64-4baa-b24f-43d87880f42f)

- When to Consider Other loading methods :
    - COPY Command: If the dataset is moderately large (up to terabytes), and you want simpler syntax with high performance.
    Recommended for JSON, Parquet, or CSV data formats.
    - Bulk Insert: For smaller datasets or quick ad-hoc loading tasks.
    Suitable for data already stored in files accessible by the Synapse compute.
    - Azure Data Factory (ADF): For orchestrated ETL pipelines, data transformations, and loading data from multiple heterogeneous sources.
    - Databricks/Synapse Pipelines: If the data needs preprocessing, complex transformations, or comes from streaming sources.
- Allways use switch partition to orverwrite or delete a partition.
- In databricks, to implement the hourely incremental refresh we use : df.write.partitionBy("ID","Year","Month","Day","Houre").mode(SaveMode.Append).parquet("path + table name")
- The Invoke-AzDataFactoryV2Pipeline cmdlet is used to start a Data Factory pipeline.
- The **OPENJSON** command converts a JSON document into table format.
- If we want to **optimize time of stopping the compute resources used by Azure Synapse Analytics during periods of zero activity e need to check the sys.dm_operation_status dynamic management view until no transactions are active in the database** before stopping the compute resources ensures that any running transaction will finish before stopping the computer nodes. If you stop the node while a transaction is running, the transaction will be rolled back, which can take time to occur.
- Databricks shared storage, which all the nodes of the cluster can access, is built and formatted by using DBFS.
- To increase the number of records per batch, we need to increase the **writeBatchSize**. The default value for this parameter is **10,000**, so to increase this we need to use a value that is higher than the default.
- Hopping window : **"Events Window" HoppingWindow(Duration(minute, 5), "updated each" Duration(minute, 1))** for example A 5-minute average updated every 1 minute. In the hopingwindow, events can overlap if the update duration is smaller than the window duration. ideal for trend detection.
- Tumbling window : **TumblingWindow(Duration(minute, 1))**. window period and update period are the same. it is essentially a specific case of a hopping window where the time period and the event aggregation period are the same.
- Sliding Window : A dynamically-sized window that is triggered by events. The window starts with the first event and keeps sliding until no new events occur within a specified duration. Useful when Monitoring event inactivity or detecting patterns in irregular event streams. For example : we want to detect errors that occur within 2 minutes of any given error to identify clusters of related issues ==> all errors that are in overlapped windows are in the same cluster.
- Session Window : A dynamic, event-driven window that groups events based on a timeout period of inactivity. Each session window starts when an event arrives and continues until no events arrive within the specified timeout. Useful when analyzing user interactions or tracking usage patterns like **user interactions with websites**.
- **Each six SUs is one node**,therefore three nodes will require a minimum of 18 SUs.
- Errors may occur when sending events and dependinf on the type of error we can set parmaeters to keep these events : for late arrival events we need to use the **lateness tolerance** setting to specify how long to wait for late-arriving events.
**Default tolerance is 0 seconds**, meaning no late arrivals are accepted after the window closes. For **Out-of-Order Events** use the **out-of-order tolerance** setting to define how far out of order events can be while still being processed. Default tolerance is **5 seconds**.
- To trigger ADF pipelines using events other than timer, event hub, and storage triggers (that can be triggered using ADF designer) we need to use a logic apps (similar to power automate) for example when receiving an email run the ADF pipeline.
- Types of Event-Based Triggers:
  - Blob Events: Trigger pipelines based on blob creation or deletion in an Azure Blob Storage container.
  - Custom Events: Trigger pipelines using custom events sent to Azure Event Grid (e.g., events generated by applications or services).
- Enabling sampling in source1 of a dataflow in ADF pipeline during debugging allows to specify how many rows to retrieve.
- If we want to send emails (trigger emails) if a pipeline fails we need to create an alert in the Data Factory resource.
- Change data capture captures every change to the data and presents the new values as a new row in the tables. This feature needs to be enabled in the datawarehouse.
```
EXEC sys.sp_cdc_enable_db;
# or on a specific table 
EXEC sys.sp_cdc_enable_table  
    @source_schema = N'dbo',  
    @source_name = N'YourTableName',  
    @role_name = NULL;
# check if it is enabled on a table
SELECT is_cdc_enabled FROM sys.databases WHERE name = 'YourDatabaseName';
# check all the tables here it is applied
SELECT * FROM sys.tables WHERE is_tracked_by_cdc = 1;
```
- To reduce the amount of state data to improve latency during a long-running steaming operation we need to use watermarks.
- RocksDB state management helps debug job slowness.
- Parallelization in Stream analytics : we can leverage partitionnning to get parallelization. For a job to be parallel, **partition keys need to be aligned between all inputs, all query logic steps, and all outputs**.
- Data paritionning strategy in ADLS GEN2: we should consider the data residency. Meaning that if a partition of data is legally needed to be stored in a specific regeion. If all data is stored in an ADLS account in Europe, but some data needs to be logically isolated by country (e.g., Germany or France), partitions can enforce that separation:
```bash
/data/country=Germany/...
/data/country=France/...
```
- Sharding the source data into multiple files will increase the amount of bandwidth available to the import process when inserting data using Polybase.
- **Horizontal partitioning** in the context of Azure data sources refers to dividing a large dataset into smaller, more manageable subsets (partitions) based on rows. Each partition contains a subset of the dataset, often defined by a range or category, such as dates, IDs, or other attributes.
- A new column of type date or int can be used to track lineage in a source table and be used for filtering during an incremental load.
- If you are granting permissions by using only ACLs (Access control lists) (not Azure RBAC), then to grant a security principal read or write access to a file, you will need to grant the security principal Execute permissions to the root folder of the container and to each folder in the hierarchy of folders that lead to the file.
- To authenticate to the Azure Databricks REST API, a user can create a personal access token (PAT) and use it in their REST API request. Databricks recommends you use OAuth access tokens instead of PATs for greater security and convenience.
- To revoke a user’s token, you need to use Token Management API 2.0.
- We should open the Monitor page and review the Pipeline runs tab as the status information is displayed on this tab.
- To review the performance of a SQL query we should open the Monitor page and review the SQL request tab where we will find all the queries running on the dedicated SQL pools and we can also query the sys.dm_pdw_exec_requests dynamic management view, as it contains information about the queries, including their duration.
- To monitor bottlenecks related to the SQL Server OS state on each node of the dedicated SQL pools we need to use the **sys.dm_pdw_wait_stats** as it holds information related to the SQL Server OS state related to instances running on the different nodes.
- The sys.dm_pdw_waits view holds information about all wait stats encountered during the execution of a request or query, including locks and waits on a transmission queue.
More on SQL and Parallel Data Warehouse Dynamic Management Views : [here](https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sql-and-parallel-data-warehouse-dynamic-management-views?view=azure-sqldw-latest)
- The runtimes of existing pipelines is available in the Azure portal, on the Data Factory blade, under the Monitor & Manage tile.
- Data Factory only stores pipeline runtimes for 45 days. To view the data for a longer period, that data must be sent to Azure Monitor, where the information can then be retrieved and viewed.
- The Gantt view of the pipeline runs shows you a graphical view of the runtime data so that you can see which pipelines are running at the same time, and which runs are running at different times.
- When Data Factory is configured to send logging data to Log Analytics and is in Azure-Diagnostics mode, the data will be sent to the AzureDiagnostics table in Log Analytics.
- The ADFActivityRun, ADFPipelineRun, ADFSSISIntegrationRuntimeLogs, and ADFSSISPackageExecutableStatistics tables are used when the Data Factory is in Resource-Specific mode.
- A Data Factory pipeline stores monitoring data for 45 days. To keep data for longer, you need to create a Log Analytical workspace and send the data to an Azure Storage account.
- All the DMVs in synapse start with **sys.dm_pdw_** not like SQL server that start with **sys.dm_**.
- sys.dm_pdw_exec_sessions shows the status of the sessions, not the running requests.
- sys.dm_pdw_exec_requests shows the requests that are in process, completed, failed, or closed.
- Increasing the concurrency on a database requires scaling the database up by using the **Set-AzSqlDatabase** cmdlet.
- **Default, Number, and Custom String** are valid options to implement dynamic data masking at the SQL Server database layer. Row-level security (RLS) is an access restriction capability not a data obfuscation capability.
- There are two types of security policies supported by row-level security:
![image](https://github.com/user-attachments/assets/d00ebe6d-ec5a-4cf6-9269-00590b291341)  
- We should increase the number of SUs because the job is running out of resources. When the Backlogged Input Events metric is greater than zero, the job is not able to process all incoming events.
- How to read a json file to a flat table in databricks using pyspark :

```json
{
    "persons": [
        {
            "name":"Keith",
            "age":30,
            "dogs":["Fido","Fluffy"]
        },
        {
            "name":"Donna",
            "age":46,
            "dogs":["Spot"]
        }
    ]
}
```  

![image](https://github.com/user-attachments/assets/2a833cfe-6f33-4151-b664-5e2dc731ded2)  

```pyspark
            dbutils.fs.put("/tmp/source.json", source_json, True)

            #spark.read.csv: This is the Spark SQL function to read CSV files and create a DataFrame.

            source_df = spark.read.option("multiline", "true").json("/tmp/source.json")

            persons = source_df.select(explode("persons").alias("persons"))

            persons_dogs = persons.select(col("persons.name").alias("owner"), col("persons.age").alias("age"), explode(col("persons.dog")).alias("dog"))

            persons_dogs.display()
```

- Explode function in databricks pyspark : Returns a new row for each element in the given array or map. Uses the default column name col for elements in the array and key and value for elements in the map unless specified otherwise.
- ![image](https://github.com/user-attachments/assets/a9d9e301-20fb-4dc0-9762-083e98c23ba7)
- Using PySpark in an Apache Spark pool within Azure Synapse Analytics is the most flexible and powerful way to handle JSON files with varying structures and data types. PySpark can infer schema and handle complex data transformations, making it well-suited for loading heterogeneous JSON data into tables while preserving the original data types.
- To reduce the time needed for a Databricks cluster to start and scale up, using **instance pools** is the most cost-effective option while still providing significant performance benefits. Instance pools keep a set of virtual machines ready for allocation to clusters. When not in use, the VMs in the pool are in a stopped state, incurring minimal costs.
- In general, six SUs are needed for each partition. If we have 10 partitions for step 1 and 10 for step 2 (2 streams), it should be 120 and not 60 because each stream needs 6 SUs and if it is partitionned we multiply by the number of partitions.
- To create external data source for Data Lake Storage Gen2:
        - abfs[s] <container>@<storage_account>.dfs.core.windows.net
        - http[s] <storage_account>.dfs.core.windows.net/<container>/subfolders
        - wasb[s] <container>@<storage_account>.blob.core.windows.net
- Round robin distribution hqs no column to specify.
- Data redundancy in ADLS Gen 2 :
  - Locally redundant storage :
    ![image](https://github.com/user-attachments/assets/e8b3ee71-edf4-4a43-b7b9-ecd3d010c2e2)
  - Zone-redundant storage:
    ![image](https://github.com/user-attachments/assets/4b129c1d-aa28-43de-bf2a-44d74b9ac071)
  - Geo-redundant storage:
    ![image](https://github.com/user-attachments/assets/0d231700-0a83-4426-adec-4d224a9df6bb)
  - Read access Geo-zone-redundant storage:
    ![image](https://github.com/user-attachments/assets/c9a21150-30e4-4adc-8b2a-7edc7b95d891)
- A Type 3 SCD supports storing two versions of a dimension member as separate columns. The table includes a column for the current value of a member plus either the original or previous value of the member. So Type 3 uses additional columns to track one key instance of history, rather than storing additional rows to track each change like in a Type 2 SCD.
- hash distribution on a culumn that is frequently queried using aggregations is faster since for example all the rows of the same product will fall inthe same node minimizing the shuffling operations.
- It is generally faster to load data into smaller or empty partitions compared to large partitions in Azure Synapse Analytics.
- In Azure Synapse Analytics, the maximum row size for a table is 1 MB. This includes all fixed-length and variable-length data stored in a single row. If a row exceeds 1 MB, the system will throw an error during data insertion or loading.
- **ALTER INDEX REBUILD** statement can be used to rebuild your indexes.
- In databricks and in order to incremently process new files in a ADLS gen2 source we use **AUTO LOADER** as it minimize implementation and maintenance effort and supports schema evolution.
- To have a good performance in synapse dedicated sql pool we need at least 1 million per distribution and per **partition** (If we have only one partition per distibution).
- **Openrowset** uses always **BULK**.
- Query acceleration supports CSV and JSON formatted data as input to each request.
- Azure Synapse Analytics allows Apache Spark pools in the same workspace to share a managed HMS (Hive Metastore) compatible metastore as their catalog. When customers want to persist the Hive catalog metadata outside of the workspace, and share catalog objects with other computational engines outside of the workspace, such as HDInsight and Azure Databricks, they can connect to an external Hive Metastore. see the settings here :https://learn.microsoft.com/en-us/azure/synapse-analytics/spark/apache-spark-external-metastore
- SCD : Type 0 No changes, Type 1 replace with new values, type 2 history, type 3 one column for the last value and another for the previous or original value.
- We can use a Private endpoint to connect the virtual machines to workspace1. Service endpoints are unavailable for Azure Synapse Analytics.
- Sharding pattern in azure apps : the hashing partitionning strategy provides the ideal balance.
- By using vertical partitioning, different parts of the database can be isolated from each other to improve cache use.
- You need a master key in the Azure SQL database for lineage to work.
- For hive tables we use wasb, not http.
- There is a limit of simultaneous pipelines in an integration runtime. You need to split the pipeline to run into multiple runtimes.
- if we need to identify only the rows that have changer without the data we can use Change tracking that captures the fact that a row was changed without tracking the data that was changed. Change tracking requires less server resources than change data capture.
- Watermarks for performance and rocksDB to debug slowliness.
- Data residency must be considered to identify whether different datasets can only exist in specific regions.
- By using larger files when importing data, transaction costs can be reduced. This is because the reading of files is billed with a 4-MB operation, even if the file is less than 4 MB. To reduce costs, the entire 4 MB should be used per read.
- You need to create a certificated that is protected by the master key. Having this certificate, you can then create a database encryption key. Creating a database encryption key can be done once there is a certificate created in the master database. You can start the database encryption only when you have a database encryption key.
- Data Factory only stores pipeline runtimes for 45 days. To view the data for a longer period, that data must be sent to Azure Monitor, where the information can then be retrieved and viewed.
- You should use **delta.autoOptimize.autoCompact = auto** because it compacts the files to the size that is appropriate to the use case. **delta.autoOptimize.autoCompact = true** and delta.autoOptimize.autoCompact = legacy compact the files to 128 MB. **delta.autoOptimize.autoCompact = false** disables automated file compaction.
- 





