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

![image](https://github.com/user-attachments/assets/6d134ef2-12fa-4ca0-9f88-bdbff6da0f67)  
The first level of hierarchy is the organization. An organization can hold multiple accoucnts (up to 25). We can set some administration rules such as replications and accounts creation.  
Then we have the Account level where we will have all the databases and the other objects. If created by snowflake, the structure will contain locator, regon and the cloud provider. If created in an organization using ORGADMIN role it contain the name of the organisation.  

![image](https://github.com/user-attachments/assets/43176a27-dc88-4cfb-9ffb-83c880b2faa6)  

Snowflake has a list of objects in the ecosystem that are listed bellow :
- Databases
- Schemas
- Table Types:

![image](https://github.com/user-attachments/assets/97a090f0-141a-4a50-8440-05eea3cd5752)  

Transient tables are staging ones. They hold temporary data but available accross sessions.  
- View Types:
  
![image](https://github.com/user-attachments/assets/cc2b2745-c083-4882-9ee7-e2b2189343cb)  

Materilized views are said **Precomputed Datasets**

- Data Types
- User-defined Functions (UDFs) and User-defined Table Functions (UDTFs):

![image](https://github.com/user-attachments/assets/1b87cc33-c148-419e-a491-b67f52a47991)  

![image](https://github.com/user-attachments/assets/aafdd2a6-6d78-48dc-a556-bb2579b03fcb)  

![image](https://github.com/user-attachments/assets/888684de-ce22-4158-a99b-d7357bdd8034)  

- Stored Procedures
- Streams :  For CDC and incremental loading of data. A select statement does not consume the stream while using it in a DML does.  
- Tasks : Used to schedule and automate SQL queries or procedures in Snowflake, which can include operations on tables or pipes themselves. A DAG is constructed when we link several tasks together but note that all tasks need to have the same owner and belong to the same schema and database !
- Pipes : Specifically designed for automatically loading data from external stages into Snowflake tables, often used for real-time or continuous data ingestion.
- Shares
- Sequences :  auto incremented values.

### Billing : 
Several types of elements impact the snowflake billing:  

![image](https://github.com/user-attachments/assets/b16e67a8-9bb4-41a8-ad56-2226ab2f96bf)  

The cloud services are all the queries we use and does not use a user managed warehouse. Such as Creating a table !  
The serverless services on the other hand can be snowpipes, database replications and clustering !

![image](https://github.com/user-attachments/assets/1b8794d1-739c-44dc-b1ac-6149b9be1741)  

### External tools :  

Snow SQL, partners and connectors !  

### Snowflake Scripting : 

We can write sql code in a procedural logic in what we call anonymous block (excuted outside a stored procedure) and in SP.  

![image](https://github.com/user-attachments/assets/9ab2aeb7-53b0-45a2-92c9-16a7564054ae)  

Also in SP :  

![image](https://github.com/user-attachments/assets/2137d7d6-c788-4dfa-81d2-d052c7d45fa3)  

Note that declare and exceptions are optionals in a block :  

![image](https://github.com/user-attachments/assets/0a709309-8765-4032-862d-589af6284a62)  

We can use also the looping constructs :  

![image](https://github.com/user-attachments/assets/478af32c-3e23-435d-9447-9561f2aa6875)  

We can also loop on records inside a table. For this we use cursors :  

![image](https://github.com/user-attachments/assets/2d63ac67-3cbf-4aa1-96d2-6faf35b1f809)  

We can also store the result of a SQL query in a Resultset object to be used in a SP or a script and make a SQL query dynamic :  

```SQL
DECLARE res RESULTSET;
DECLARE query TEXT;

SET query = 'SELECT name FROM employees WHERE department = ''Sales'';';
SET res = (EXECUTE IMMEDIATE :query);

SELECT * FROM TABLE(res);
```
In Snowflake, a ResultSet is an object that stores the output of a query execution. It allows you to programmatically fetch and manipulate query results within stored procedures, scripts, or Snowflake connectors. 

### Snowpark :  

![image](https://github.com/user-attachments/assets/86151bd5-627d-4137-8119-f60098ee5543)  

Key Features of Snowpark  
✅ 1. Native Support for Python, Java, and Scala  
Unlike traditional SQL-based transformations, Snowpark lets you use Python, Java, or Scala to process data.  

Example: Instead of writing SELECT * FROM table WHERE ..., you can use Python DataFrames.  

✅ 2. Pushdown Execution in Snowflake  
Code is executed inside Snowflake’s compute engine, reducing data movement.  

No need to export data to external environments (e.g., Spark clusters or pandas in Python).  

✅ 3. Object-Oriented DataFrame API  
Snowpark provides a DataFrame API, similar to pandas (Python) or Spark DataFrames, for manipulating data.  

Example in Python:  

```Python
from snowflake.snowpark import Session

session = Session.builder.configs({...}).create()  # Create Snowflake session
df = session.table("sales").filter("region = 'US'")  # Query data
df.show()  # Display results

```
✅ 4. Supports UDFs (User-Defined Functions) and Stored Procedures  
You can write custom functions in Python, Java, or Scala and run them within Snowflake.  

Example: A Python UDF in Snowflake:  

```python
from snowflake.snowpark.functions import udf

@udf
def square_number(x: int) -> int:
    return x * x
```

This function can be called in Snowflake SQL:  

``` sql
SELECT square_number(10);  -- Returns 100
```  
## Acocunt Access and security:  

### Roles:  

Snowflake uses two access controle frameworks :  

![{539EB27E-6AEB-4E82-9000-331FB7C702B2}](https://github.com/user-attachments/assets/2870497b-da33-4f9f-b5f5-d8cf6d782946)  

We have by system defined roles :  

![{FEF76E61-626C-4598-9559-C6B197C2DC67}](https://github.com/user-attachments/assets/b96afbe7-cb40-4432-8e15-f0d22d6f9e35)  

![{71F50619-982E-4B08-A9E3-70968B54D1EC}](https://github.com/user-attachments/assets/08c7e495-a659-41d4-8e34-27920143bb1e)  


But we can also create custom roles with granular previleges :  

![{BAD95782-05B4-4510-B42F-18716BB803A8}](https://github.com/user-attachments/assets/989061d9-dcfa-4e60-8d8e-4dde8556a9b0)  

All the custom roles need to be affected to a system defined role (in the most cases sysadmin) so that the objects owned by these roles can be tracked by accountadmin for example !  
By default a role to which we affect a another role will inherit all the previleges of this latter !  

### Previleges :  

Each role in snowflake has some previleges. We can create custom roles with specified grants to do a specific job like an analyst for example.  
![{BDB80F4A-3C0A-4FB4-9FDB-7365C8E53D5B}](https://github.com/user-attachments/assets/90629cc9-f8b8-44f5-9b7c-12d69a797278)  

### Authentication:  

There several ways we can authenticate to snowflake:  

✅ 1. Snow UI User authentication:  

Using user name and password :  

![{773DBC07-435C-43F5-92D6-2D1232EB7FD3}](https://github.com/user-attachments/assets/d4c926a9-3c13-46ee-b14f-eb970305ff75)  

To add another layer of security we can add Multifactor authentication using Duo Security:  

![{B156BC7D-683B-42F6-9CF2-AD3568F9981D}](https://github.com/user-attachments/assets/cca8a0bf-fb91-460a-b5c3-6b30d9baa454)  

![{3845C23F-D685-406D-88A9-23A24197A5AF}](https://github.com/user-attachments/assets/a62d7629-6b2f-43f8-84f5-7df01079d248)  

✅ 2. Federated authentication (SSO):  

![{452E8B7E-F3C9-44FB-BDA7-858A307C3416}](https://github.com/user-attachments/assets/8aa7fca0-13fc-4ffe-8e00-8a5e4fc6ad19)  

✅ 3. Key pair authentication:  

It used when we would like to connect to snowflake using a client application such as terraform without using directly password in the code :  

![{201FD6B1-9E3F-48FA-9017-EF3391570631}](https://github.com/user-attachments/assets/0607e77f-46a8-4b8c-8638-31af1fea5f47)  

✅ 4. Oauth and Scim:  

![{A270AF0B-F354-4C15-8A40-101351C899DE}](https://github.com/user-attachments/assets/d1c61194-0a22-46df-9e2d-46c49269a06b)  

### Network policies :   

When we define a network policy in Snowflake, it restricts user authentication to only the allowed IP addresses specified in the policy. 
Meaning that if a user tries to authenticate from an unauthorized IP, Snowflake rejects the connection.  

![{E77D6607-18E4-4FBC-872C-915A92AF2B51}](https://github.com/user-attachments/assets/946f730e-b2d8-46a1-8ed4-7c7244e0e170)  

- Account-Level Policy: Affects all users and roles.  
- User-Level Policy: Overrides account-level policies for specific users.

```SQL
CREATE NETWORK POLICY restrict_office_ips 
    ALLOWED_IP_LIST = ('192.168.1.100', '203.0.113.42') 
    BLOCKED_IP_LIST = ('45.67.89.10');
```
We then use alter Account or User to set the Network_Policy.  

```SQL
ALTER ACCOUNT SET NETWORK_POLICY = restrict_office_ips;
ALTER USER john_doe SET NETWORK_POLICY = restrict_office_ips;
```

### Data Encryption :  

Data in snowflake is enrypted end to end using two encryption mechanisms :  

![image](https://github.com/user-attachments/assets/a612f4ad-99e1-4070-9bcb-37199d3f19e2)  

The flow encryption depends on where data is stored :  

![image](https://github.com/user-attachments/assets/2d89e6d0-8efe-4e33-af1a-4023ee1e8ae2)  

We can set our client side encryption in the external storage but we will need to provide the encryption key to snowflake so it can store data and then encrypt!  

The encryption is so resilient thanks to the hierarchical key model used :  

![image](https://github.com/user-attachments/assets/0dcd20fa-4248-473a-993d-dbc35edc6258)  

This means that if a key is exposed the other components for example files or micro-partitions are  not exposed !  
The mother key is stored seperatly in a AWS cloudHSM service.  

To enhance this encryption process snowflake uses Key rotation that raplaces envery 30 days the account and the table keys while keeping the older keys onlu to decrypte the data previously encrypted !  

![image](https://github.com/user-attachments/assets/26656688-afd5-4dd3-a8ae-cd2a5b8d483c)  

The Rekeying however is another service we can activate (additionnal cost) that changes the keys retired that exceed one year and completely reencrypt its data :  

![image](https://github.com/user-attachments/assets/ecad1894-980c-45a0-a493-149a8b069d6d)  

The last feature where we add another securty layer is the Tri-secret secure and customer managed keys where we simply manage our own keys in a service like KMS or Azure key vaults !  

![image](https://github.com/user-attachments/assets/6ca59327-8b7d-4753-aefe-05aebdeff9f1)  

### Column level security :  

We can set security on a column using data masking :  

![image](https://github.com/user-attachments/assets/dde43eff-38a8-427d-8bfd-7d7d9f1c6771)  

If the user queries data and he is anauthorized to see it it will be masked. For example for emails only the domain name will show !  

To create a masking policy we can do as follows :  

![image](https://github.com/user-attachments/assets/85cc369c-33a9-4b90-ad6a-077a0d1d3ebd)  

Data masking has several properties :  

![image](https://github.com/user-attachments/assets/91486ac1-363f-4617-9301-9c0e5e35dfc4)  

We can also use external functions to mask data inside masking policies. This is called external tokenization:  

![image](https://github.com/user-attachments/assets/85bf9b63-72b7-4138-beb6-260a6afc71f7)  

In this way even snowflake and the cloud provider cannot read the data as it is stored tokenized and needs the function to be detokenized !  

### Row level policies:  

![image](https://github.com/user-attachments/assets/4de08414-628c-4a76-8dd5-8eb94d4d4065)  

We can only set one row access policy per table !  

### Secured Views :  

In standard views, when we define a view it is related to a whole table behind:  

```SQL
CREATE VIEW employees AS
SELECT id, name, department 
FROM employees;
```
The table behind has also the salary column for each employee but we hide it in this view. Now when users query the view, Snowflake rewrite the query and applies query optimizations such as:
    
    1- Predicate Pushdown → Applying WHERE filters as early as possible.
    
    2- Join Reordering → Reordering joins for efficiency.
    
    3- Column Pruning → Removing unused columns to reduce data scanned.
    
    4- Aggregation Pushdown → Performing aggregations earlier in execution.

While standard views mostly restrict direct column access, they can leak metadata and data relationships through: 

- Error messages (revealing hidden columns)
- Query timing (inference attacks)
- Nested data functions (JSON, arrays)
- Statistical anomalies (error handling differences)

All this may give the attacker a way to guess the hidden columns and maybe the values also !  

Secure views explicitly block these side channels:  

![image](https://github.com/user-attachments/assets/8e6885ca-58a2-4ec6-a02f-d179f678451f)  

It does this by bypassing some of the view optimizations performed by the Query Optimizer. But note that these views are slower and for that reason, making views which don't have strict security requirements, secure is discouraged by Snowflake.  

### Account Usage and information schema :
These schemas are so important for metadata analysis of our account and schemas. For the account it holds all what happend to the account objects :  

![image](https://github.com/user-attachments/assets/ef84220b-0082-4ef0-8791-9b614ad52afe)  

The information schema is present in every database and it gives quite similar information but only for the corresponding database. The metadata here are refreshed faster than the account usage :  

![image](https://github.com/user-attachments/assets/15d62cb6-4651-46a5-b9b9-4bef202c430e)  

When to use what ? :  

![image](https://github.com/user-attachments/assets/bb83948f-f7b3-4a45-b442-fdc3c6e54fc0)  







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

