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


    
