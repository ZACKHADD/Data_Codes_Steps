# This Document contains a recap of the Snowflake paid exam preparation course and other materials

## Data Architecture and cloud features:

Snowflake is SaaS data plateform that has a hybrid architecture using a shared disk for the storage and a shared nothing for computing. This separates computing from storing data leveringin high scalabitily !  
Generally the cloud data systems one of the two either Shared disk or shared nothing architecture !  

![image](https://github.com/user-attachments/assets/2d778e8e-514d-4db7-80d9-b498d500cf7d)  

![image](https://github.com/user-attachments/assets/a7dc74d2-16ad-4559-a40a-73c06fddbc2b)  

![image](https://github.com/user-attachments/assets/1f08eddc-f1db-4ae7-b1ac-d1be0e49a395)  

Snowflake is a hybrid of these two :  

![image](https://github.com/user-attachments/assets/5e016d97-d9c7-4bd7-9ae5-eec12ad044a4) 

Snowflake can only run on cloud and all the infrastructure used in it is provided from one of the 3 famous cloud providers : AWS, AZURE and GCP. (Originally designed to run on AWS).  

### Snowflake has three layer architecture : 

1- Storage layer linked to the cloud provider :
  
![image](https://github.com/user-attachments/assets/4b01cb99-1a96-4de9-8d42-132227e84ed4)

2- The compute layer which is simply the snowflke engine : if we are on AWS it is simply a cluster of EC2 instances.

![image](https://github.com/user-attachments/assets/28acd1e2-d9c2-4f29-9653-62b8192dae23)  

Data cache in snowflake is so important as it avoids averheads and unessesary costs. Here how it works when a query is executed :  
- Step-by-step execution:
  - Query Planning: Snowflake checks the metadata cache to identify which micro-partitions are needed (each column has medatada regarding min/max, distinct values .. and if we have a where clause in the query snowflake skips the partitions where the min/max does not contain the where clause valeue). If partition pruning is possible, only relevant partitions are considered.
  - Checking Result Cache: If the exact query was executed before and **data hasn’t changed**, Snowflake instantly returns results from the result cache (No compute cost ✅).
  - Checking Virtual Warehouse Cache: If this warehouse has executed a similar query recently, it reads from its local cache (avoiding storage access, making it much faster). If the warehouse cache is empty, it fetches data from storage.
  - Fetching from Storage (If Needed): If no cache is available, the warehouse reads micro-partitions from storage (slowest step) and loads them into the memory. This data is cached for future queries.
  - Returning Results: Snowflake compresses results and stores them in the result cache for reuse.

**Note that to trigger data prunning when using where clause, we should not use functions with the condition such as : UPPER(region) = 'US'; this will force snowflake to scan all partitions and apply the upper transformation row by row before filtering since the metadata is not precomputed with upper function**

along side with the micro-partitionning that is automatic following the order of insertion, we can **Cluster** data which is the same thing only using specific keys we explicitly define rather than the order of insertion !  

3- The Cloud services layer dealing automaticaly with optimization, gouvernance security and so on:

![image](https://github.com/user-attachments/assets/6be0ae41-ae85-4d35-ade8-6f62e6b01ee9)  

![{7D6D69AB-75B1-42F2-851F-ABB70C07DD38}](https://github.com/user-attachments/assets/7342a7db-7122-43ce-966b-d80bafeec179)  

The data is not siloed so everyone can query everything if the are allowed to and the compute is seperated so you just choose a compute warehouse and data will be loaded there in the memory for compute : 

![{8403962C-73B4-4C14-B551-02C72CD18C30}](https://github.com/user-attachments/assets/f820338e-ac2f-4f21-af09-342b29c4aa29)

The type of computing warehouse is important and depends on the use case and the workload needed:

- For Loading data we can use XS size
- For Transformations we can use S size
- For an app (Data sience models for example) we can use L or XL depending on the need ==> Scaling UP
- Multicluster warehouse (generally Medium size) are essentialy used for reporting and analytics (power bi for example) that needs concurrency. At a certain point if the pressure on a reporting tool is high we can add another cluster to increase the concurrency ! ==> Scaling Out

### Several Editions :

![image](https://github.com/user-attachments/assets/bbf29850-ef51-413c-a51e-6507506edf4d)  

The Business Critical edition offers higher levels of data protection and enhanced security to support organizations with very sensitive data, for example, data that must comply with regulations. With this edition, we can enable private connectivity to our Snowflake account, or use our own customer-managed key to encrypt data in Snowflake. Business Critical includes all the features and services of the Enterprise and Standard editions.  

Virtual Private Snowflake. This offers the highest level of security for organizations that have strict requirements around data protection, such as governmental bodies or financial institutions.  

The services layer is shared between accounts. If you were to create a VPS edition account, this wouldn't be the case. Services like the metadata store wouldn't be shared.  

### Snowflake Catalog and objects : 

Snowflake has a list of objects in the ecosystem that are listed bellow :
- Databases
- Schemas
- Table Types
- View Types
- Data Types
- User-defined Functions (UDFs) and User-defined Table Functions (UDTFs)
- Stored Procedures
- Streams :  For CDC and incremental loading of data
- Tasks : Used to schedule and automate SQL queries or procedures in Snowflake, which can include operations on tables or pipes themselves.
- Pipes : Specifically designed for automatically loading data from external stages into Snowflake tables, often used for real-time or continuous data ingestion.
- Shares
- Sequences :  auto incremented values.

All data in Snowflake is stored in database tables, logically structured as collections of columns and rows. To best utilize Snowflake tables, particularly large tables, it is helpful to have an understanding of the physical structure behind the logical structure.  
**Micro-partitions and data clustering**, two of the principal concepts utilized in Snowflake physical table structures.  
Snowflake automatically divides tables into micro-partitions, which are immutable internal storage units (each typically 50–500 MB compressed).  
Micro-partitionning is automatic and does not need to be maintained !  
This is powerful as it gives the possibility to prune columns to be scaned when we use filter predicates !  

How it works : 
- When data is inserted, Snowflake automatically divides it into micro-partitions.
- Each micro-partition stores metadata like min/max values, column stats, and cardinality.
- These micro-partitions are immutable and stored in columnar format.
- Snowflake prunes unnecessary micro-partitions at query time, reducing scan costs.

By default micro partitionning groups data based on the insert order ! **so if this order is randomly inserting data then the prunning will not be that efficient and we may need to cluster data**!  
How Does Clustering Work?
- Snowflake checks how well data is ordered within micro-partitions.
- If needed, it automatically reclusters data in the background (for large tables).
- We can manually trigger reclustering with ALTER TABLE RECLUSTER.

This will optimize the queries using frequently some columns like date, id and so on !  
We can use this command to cluster based on a columns or multiple columns : 

```SQL
          CLUSTER BY {Column1, Column2 ...}
```
#### Clustering Depth
The clustering depth for a populated table measures the average depth (1 or greater) of the overlapping micro-partitions for specified columns in a table. The smaller the average depth, the better clustered the table is with regards to the specified columns.  

#### Types of tables : 

![{E267A5DC-3989-4108-A69B-CE768B92C9AA}](https://github.com/user-attachments/assets/39934e0a-b80e-44f8-be59-b25af6a7302b)  

**Note that :**
Since micro-partitions cannot be changed, Snowflake does NOT physically move data between existing partitions. Instead, it:

- Reads the existing micro-partitions.
- Sorts the data based on the clustering key.
- Creates new micro-partitions that follow the new order.
- Marks the old micro-partitions as obsolete (logical deletion).

**Snowflake suggests sometimes clustering improvements (via SHOW CLUSTERING INFORMATION).**  

https://docs.snowflake.com/en/user-guide/tables-micro-partitions

#### Connecting snowflake to external data:

Similarly to synapse, when connecting ro external data we need create external data source or in this case external stage !  but to connect, we need also to create a **Storage integration** that will authenticate to the source like ADLS gen2 or we can use directly SAS token : 

![{D8B3BEB9-0989-476A-928C-D96009A78328}](https://github.com/user-attachments/assets/4098ac76-a196-4761-83b0-13d32145f160)  

![{3D381A18-9A18-4E0D-BA16-AA88F14E4AC9}](https://github.com/user-attachments/assets/e0957fa8-32a6-4fc2-99c2-c4fb46f4eff5)  

Once created we will need to consent for a first time using the consent url : 

![{AFF3D708-D8F4-447C-80E0-1D74F62CFD55}](https://github.com/user-attachments/assets/198ac549-c2a0-4f54-bce6-5cdc952e58f8)  



to create the storage integration : 

```SQL
                    create storage integration <storage_integration_name>
                        type = external_stage
                        storage_provider = azure
                        azure_tenant_id = '<azure_tenant_id>'
                        enabled = true
                        storage_allowed_locations = ( 'azure://<location1>', 'azure://<location2>' )
                        -- storage_blocked_locations = ( 'azure://<location1>', 'azure://<location2>' )
                        -- comment = '<comment>';
```
### Labs hacks:

tO list all the content of an external stage we can use LIST command.

the complete query to create the file format in order to ingest data from files in an external storage : 

```SQL
          CREATE OR REPLACE FILE FORMAT csv
              TYPE = 'CSV'
              COMPRESSION = 'AUTO'  -- Automatically determines the compression of files
              FIELD_DELIMITER = ','  -- Specifies comma as the field delimiter
              RECORD_DELIMITER = '\n'  -- Specifies newline as the record delimiter
              SKIP_HEADER = 1  -- Skip the first line
              FIELD_OPTIONALLY_ENCLOSED_BY = '\042'  -- Fields are optionally enclosed by double quotes (ASCII code 34)
              TRIM_SPACE = FALSE  -- Spaces are not trimmed from fields
              ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE  -- Does not raise an error if the number of fields in the data file varies
              ESCAPE = 'NONE'  -- No escape character for special character escaping
              ESCAPE_UNENCLOSED_FIELD = '\134'  -- Backslash is the escape character for unenclosed fields
              DATE_FORMAT = 'AUTO'  -- Automatically detects the date format
              TIMESTAMP_FORMAT = 'AUTO'  -- Automatically detects the timestamp format
              NULL_IF = ('')  -- Treats empty strings as NULL values
              COMMENT = 'File format for ingesting data';
```

Using LAG and moving AVG :

```SQL
                SELECT
                    meta.primary_ticker,
                    meta.company_name,
                    ts.date,
                    ts.value AS post_market_close,
                    (ts.value / LAG(ts.value, 1) OVER (PARTITION BY meta.primary_ticker ORDER BY ts.date))::DOUBLE AS daily_return,
                    AVG(ts.value) OVER (PARTITION BY meta.primary_ticker ORDER BY ts.date ROWS BETWEEN 4 PRECEDING AND CURRENT ROW) AS five_day_moving_avg_price
                FROM FINANCE_ECONOMICS.cybersyn.stock_price_timeseries ts
                INNER JOIN company_metadata meta
                ON ts.ticker = meta.primary_ticker
                WHERE ts.variable_name = 'Post-Market Close';
```

### Query Cashing :

Snowflake has a result cache that holds the results of every query executed in the past 24 hours. These are available across warehouses, so query results returned to one user are available to any other user on the system who executes the same query, provided the underlying data has not changed. Not only do these repeated queries return extremely fast, but they also use no compute credits.

### Clonning:

When we create a cloned table in Snowflake:
- The clone initially shares the same metadata and storage as the original table. No additional storage is consumed until changes are made to either the clone or the original.
- Changes (e.g., deletes, inserts, updates) made to the clone are independent of the original table, and vice versa.
This behavior is achieved through Snowflake's zero-copy cloning:
- Snowflake uses metadata pointers to the original table's data.
- When you modify data in the clone, only the modified data is stored separately (copy-on-write mechanism).

```SQL
                    -- Create the original table
                    CREATE TABLE original_table (
                        id INT,
                        name STRING
                    );
                    
                    -- Insert some data
                    INSERT INTO original_table VALUES (1, 'Alice'), (2, 'Bob');
                    
                    -- Create a clone of the original table
                    CREATE TABLE cloned_table CLONE original_table;
                    
                    -- Delete data from the clone
                    DELETE FROM cloned_table WHERE id = 1;
                    
                    -- Check data in the clone
                    SELECT * FROM cloned_table;
                    -- Output: Only the record with id = 2 remains
                    
                    -- Check data in the original table
                    SELECT * FROM original_table;
                    -- Output: Both records (1, 'Alice' and 2, 'Bob') are still present
```

### Time Travel:

Snowflake's powerful Time Travel feature enables accessing historical data, as well as the objects storing the data, at any point within a period of time. The default window is 24 hours and, if you are using Snowflake Enterprise Edition, can be increased up to 90 days.  
Some useful applications include:
- Restoring data-related objects such as tables, schemas, and databases that may have been deleted.
- Duplicating and backing up data from key points in the past.
- Analyzing data usage and manipulation over specified periods of time.

We should pay attention to the settings of the timezone !! we can check the current timezone using :

```SQL
                    show parameters like '%timezone%';
                    ALTER SESSION SET TIMEZONE = 'Europe/Paris';
```

```SQL
                    -- Set the session variable for the query_id
                    SET query_id = (
                      SELECT query_id
                      FROM TABLE(information_schema.query_history_by_session(result_limit=>5))
                      WHERE query_text LIKE 'UPDATE%'
                      ORDER BY start_time DESC
                      LIMIT 1
                    );
                    
                    -- Use the session variable with the identifier syntax (e.g., $query_id)
                    CREATE OR REPLACE TABLE company_metadata AS
                    SELECT *
                    FROM company_metadata
                    BEFORE (STATEMENT => $query_id);
                    
                    -- Verify the company names have been restored
                    SELECT *
                    FROM company_metadata;
```

