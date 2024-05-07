# Fabric Overview

The present document gives an overview of Microsoft Fabric and the solutions it proposes in the data field.  

## Whats is Fabric?

It is a SaaS product that allows to expend the capabilities of Power BI to have an end to end analytics solution.  
**It somehow like Snowflake where you have multiple engines working against the same data stored in a data lake (parquet delta files located in OneLake) with a layer of delta lake to support transactions and querying the data.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/00615216-a8ba-44fa-ba1c-146f4efebb01)  

**For data processing, the plateform uses Apache Spark and SQL compute engines.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/69f4bb85-7938-4e60-a6bc-3039c2b2b01e)  

Since we are working on delta tables (on parquet files), the analysis service for tabular models **(Import mode and direct query mode)** will be replaced by **Direct Lake** where we connect to files directly and we will have the same performance as the import mode.   

The **Power BI** part is the developper part that will enable storing files from **Power BI Desktop in files that support version control using Git and CI/CD using Azure Devops**.  

### OneLake:

It is the logical data lake of fabric (**built on top of ADLS Gen2**), very similar to **OneDrive** but data are stored differently (in parquet format and delta tables format). The idea behind it is to have the same copy of data that every works on in the data lake to prevent having multiple copies of the same data.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/03fffa1a-99d5-4b42-b807-6bef3caa9d4e)  

This patern (**normaly a data mesh patern where each department handls it's own data**) in traditionnal data lakes leads generaly to a data swamp. Because when you need data from other departments, well we copy it in our data lake and then we create unecessery dupplicated data and pipelines.  

So to avoid this, Onelake here is simply a **unified ligical layer on top of the data lake (one and only data lake)** where access can be managed using **workspaces**. The same data will be used by multiple **Computes** (Power BI, Data warehouse in synapse, Lakehouse, ADF ...) using **Shortcuts**.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f09ca33f-39f8-4afa-b02d-d9708cd4d8d4)  

The logical layer of OneLake gives the possibility to adopt a **data mesh** structure on top of one single data lake, preventing data lake silos and dupplications. Also it provides powerful data governance capabilities over each domain in the company.  

Data in Onelake, can be accessed just like a normal ADLS Gen2 data using DFS API's for external workloads such as Databricks, AZURE HDI etc.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5901bf48-551d-4342-9473-3f6916a6a8d1)  

Of course we can use a data explorer to navigate and see the data in Onelake just like in ADLS Gen2.  

We can also access external data in AWS S3 buckets or an azure ADLS Gen2 storage using **Shortcuts**.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d28c6b41-708d-4d12-bf83-59933e1ff886)  

Onother important thing in Fabric, and this is where it is similar to Snowflake, we have the compute that is separated from the storage:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6e097529-6539-44c6-b930-1b608750b01f)  

Like in Snowflake when the same data can be processed by several separated and different warehouses, here several computes like T-SQL, Spark, KQL and Analysis e Services can access the same data and are not liked to storage. We say that **The compute can scale independently from the storage**.  

For example we often move the same CSV file into SQL server or into Power BI to perform different operations, will no more of that with this OneLake approach where the same data can be processed in the same time by different compute engines.  

Also, since the data in OneLake is in a **Dela parquet** format, this makes it easy for **data scientists to directly use data with whatever compute engine they want with no need for spark or SQL driver** like it has been the case before.  

So the power here is that once the data is in OneLake, it is in Delta Parquet format, it can be used by any compute engine we prefere with no setup needed.  

**Data Hub provides a single location where you can see all the data you have access to.**  

### What is the difference between Parquet and delta tables?:

Delta Lake has all the benefits of Parquet tables and many other critical features for data practitioners. That’s why using a Delta Lake instead of a Parquet table is almost always advantageous.  
**Parquet tables are OK when data is in a single file but are hard to manage and unnecessarily slow when data is in many files**. Delta Lake makes it easy to manage data in many Parquet files.  

Parquet is an immutable, binary, columnar file format with several advantages compared to a row-based format like CSV. Here are the core advantages of Parquet files compared to CSV:

- The columnar nature of Parquet files allows query engines to cherry-pick individual columns. For row-based file formats, query engines must read all the columns, even those irrelevant to the query.
- Parquet files contain schema information in the metadata, so the query engine doesn’t need to infer the schema / the user doesn’t need to manually specify the schema when reading the data.
- Columnar file formats like Parquet files are more compressible than row-based file formats.
- Parquet files store data in row groups. Each row group has min/max statistics for each column. Parquet allows query engines to skip over entire row groups for specific queries, which can be a huge performance gain when reading data.
- Parquet files are immutable, discouraging the antipattern of manually updating source data.  

The problems in parquet tables appears when the datasets are in multiple files. Here are some of the challenges of working with Parquet tables:

- No ACID transactions for Parquet data lakes
- It is not easy to delete rows from Parquet tables
- No DML transactions
- There is no change data feed
- Slow file listing overhead
- Expensive footer reads to gather statistics for file skipping
- There is no way to rename, reorder, or drop columns without rewriting the whole table  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f4b94b73-188c-4963-9651-59588262205a)  


### Direct Lake Mode:

Direct Lake is a dataset storage mode for Power BI that can replace DirectQuery and import modes by grouping the benifits of both of them : No local storage and real-time analysis.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b987b18c-7bf0-408d-9930-3863b4078b2d)  

- Directquery is directly connected to the source, so the **DAX queries for our visuals are translated to SQL queries** to run against our source. This causes latency even if we do some query folding.

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/37f7d3e9-4733-46a5-ae04-a47a7611d7f8)  

- Import Mode is the opposite, data are brought to Power BI **compressed via the Vertipaq engine (Columnar format similar to parquet) and then our DAX are converted to Vertiscan queries (so much faster since based on column storage technology) directly run against the tabular model (The tabular CUBE) in our memory** that is why it is much faster then the Directquery mode. The problem here is the capacity that has a maximum when data is enormous and **we have a lot of data copies since we import the original data into the dataset of Power BI**.

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2d757da4-7ee7-4043-9719-f677f0f32d06)  

**Note that for both of these modes, when we start Power BI we start a local instance of Analysis Services that creates the semantic model in Power BI (Tabular Cube/ Model that pre-aggregates data for fast analysis) locally.**  

**Analysis Services is an analytical data engine (VertiPaq) used in decision support and business analytics. It provides enterprise-grade semantic data model capabilities for business intelligence (BI), data analysis, and reporting applications such as Fabric/Power BI, Excel, Reporting Services, and other data visualization tools.**  

More on Analysis Services : https://learn.microsoft.com/en-us/analysis-services/analysis-services-overview?view=asallproducts-allversions

- Direct Lake addresses the problems of both modes to give a fast and light solution. **Also and in opposition to both previous modes, the tabular model (the CUBE) is no more stored in Power BI (in .idf files) locally but in OneLake in a Delta Parquet files format**.
Because the parquet file is a columnar format, similar to the .idf files, the **vertiscan queries** are sent directly to the Delta tables and the requiredcolumns are loaded in memory. Delta uses different compression than vertipaq so as the data is fetched by Power BI, it is "transcoded" on the fly into a format that Analysis Services engine can understand. In Direct Lake, instead of the native idf files, the Delta parquet files are used which removes the need for duplicating the data or any translation to native queries. Another innovation Microsoft has introduced to make queries faster is the ability to order the data in the parquet files using V-order algorithm which sorts the data similarly to vertipaq engine for higher compression and querying speed. This makes the query execution almost the same as the import mode. Data is cached in memory for subsequent queries. Any changes in the Delta tables are automatically detected and the Direct Lake dataset is refreshed providing the latest changes from the Delta tables. You will create the relationships, measures etc using web modeling or XMLA write (external tools) to turn the tables into a semantic model for further report development.

  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7b5dc127-32fe-4c3a-a38a-aac5e8b9adde)  

**Note that we always need Analysis Services for the Tabular Model/CUBE to create the semantic model that makes users able to query the data sources using DAX/MDX queries.**  

Also, the vertipaq engine querying the delta tables gives the possibility to query data using DAX and Vertiscan but also **SQL** which is a huge step in analytics.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4661912b-b573-451f-9193-b6f238a56f06)  

#### Fallback:

More in : https://fabric.guru/controlling-direct-lake-fallback-behavior  

Direct Lake mode can fallback to DirectQuery if :  

- The number of files per table, row groups per table, number of rows per table, model size on disk are reached

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/08c4e495-e1e4-4c48-abe2-1e9ea00bedd9)  

to check these requirements in our lakehouse/warehouse we can use the below python code:  

```python

                    import pandas as pd
                    import pyarrow.parquet as pq
                    import numpy as np
                    
                    def gather_table_details():
                        """
                        Sandeep Pawar  |  fabric.guru  |  Nov 25, 2023
                        Collects details of Delta tables including number of files, rowgroups, rows, size, and last OPTIMIZE and VACUUM timestamps.
                        This can be used to optimize Direct Lake performance and perform maintenance operations to avoid fallback.
                        The default Lakehouse mounted in the notebook is used as the database.
                    
                        Returns:
                        DataFrame containing the details of each table, or a message indicating no lakehouse is mounted.
                        """
                        # Check if a lakehouse is mounted
                        lakehouse_name = spark.conf.get("trident.lakehouse.name")
                        if lakehouse_name == "<no-lakehouse-specified>":
                            return "Add a lakehouse"
                    
                        def table_details(table_name):
                            detail_df = spark.sql(f"DESCRIBE DETAIL `{table_name}`").collect()[0]
                            num_files = detail_df.numFiles
                            size_in_bytes = detail_df.sizeInBytes
                            size_in_mb = size_in_bytes / (1024 * 1024)
                    
                            # Optional, set to False to avoid counting rows as it can be expensive
                            countrows = True
                            num_rows = spark.table(table_name).count() if countrows else "Skipped"
                    
                            delta_table_path = f"Tables/{table_name}"
                            latest_files = spark.read.format("delta").load(delta_table_path).inputFiles()
                            file_paths = [f.split("/")[-1] for f in latest_files]
                    
                            # Handle FileNotFoundError
                            num_rowgroups = 0
                            for filename in file_paths:
                                try:
                                    num_rowgroups += pq.ParquetFile(f"/lakehouse/default/{delta_table_path}/{filename}").num_row_groups
                                except FileNotFoundError:
                                    continue
                    
                            history_df = spark.sql(f"DESCRIBE HISTORY `{table_name}`")
                            optimize_history = history_df.filter(history_df.operation == 'OPTIMIZE').select('timestamp').collect()
                            last_optimize = optimize_history[0].timestamp if optimize_history else None
                            vacuum_history = history_df.filter(history_df.operation == 'VACUUM').select('timestamp').collect()
                            last_vacuum = vacuum_history[0].timestamp if vacuum_history else None
                    
                            return lakehouse_name, table_name, num_files, num_rowgroups, num_rows, int(round(size_in_mb, 0)), last_optimize, last_vacuum
                    
                        tables = spark.catalog.listTables()
                        table_list = [table.name for table in tables]
                        details = [table_details(t) for t in table_list]
                        details_df = pd.DataFrame(details, columns=['Lakehouse Name', 'Table Name', 'Num_Files', 'Num_Rowgroups', 'Num_Rows', 'Delta_Size_MB', 'Last OPTIMIZE Timestamp', 'Last VACUUM Timestamp'])
                        return details_df.sort_values("Delta_Size_MB", ascending=False).reset_index(drop=True)
                    
                    details_df = gather_table_details()
                    details_df

```

- Semantic model uses data warehouse views
- If the model size on disk exceeds the max size per SKU, the model (not the query) will fall back to DQ
- RLS/OLS are defined in the data warehouse
- Semantic model is published via XMLA endpoint and has not been reframed. This is not a fallback criteria rather a known issue/limitation. If you create a Direct Lake model and it is unprocessed, it will fall back to Direct Query.

To use the Fabric mode, we need to provision a **Lakehouse first since it will lacate our data to be used by the engines.**  

### Creating Lakehouse and Data Warehouse:

The lakehouse in Fabric is the pointer to our data in onelake (in delta perquet formats). Every lakehouse has an endpoint to query it using SQL.  
**By default when we create a Lakhouse, a data warehouse is created with it and also a default Power BI semantic model associated with it.**   

**Another big benefit of Lakehouses, is that if we have a  lot of teams experts in different languages (python, sql, scala, DAX PBI, R, .NET ...) they all can query the same data using their prefered language while if we have a data warehouse we only can use SQL.**  
Imagining we are analyzing data from social media (text, audios, videos ...) we an store this in warehouses but it would be so difficult to handl (not in terms of quantity but the performance and the maintainance needed, ETL, ELT ...). With Lakehouses the data is in Delta Parquet format and we can choose which engine and language to use that will best suite our needs.  

**Note however that SQL queries to write data are not supported in Lakehouse, only in Data warehouse.**  
Also for structured data, we may skip a lot of ETL and ELT processes using Lakehouse since the data is in Delta Parquet format ready to be queried.  

**All of this is made possible thanks to Parquet files** That are:  
- Highly compressed just like Vertipaq in power BI
- Having Columnar storag giving fast read capabilities
- Language agnostic meaning they can be quieried by what ever language we want
- Open source so no vendor and maitainance cost
- Support complexed data types

A clarificatio regarding data storage types : 

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7816093f-d5eb-41d4-923e-aedcca63b812)  

**Row-store fast for writing data while columnar store is fast when reading data. Adding the Delta layer on top of parquet files gives the possibility to Read and Write data faster.**  

The Warehouse in Fabric is now a SaaS warehouse with no need to provision ressources such as dedicated SQL pools and so on (It may be done behind the scene). It is now fully serverless so you pay as you go.  
The data of our data warehouse will be in SQL server format but stored in open Delta-Parquet in OneLake. Which gives the interoperability between all tyes of workloads.
**It has also the auto scale of ressources (up and down) and auto-optimization (no need to handl indexation, database statistics ...).**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e57b7f29-a6cd-4a08-8a2f-183a10f5d23c)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ad37b0f9-2520-4761-bd1e-a931fa1f0a79)  

### Data Activator:

It is a service that gives the ability to trigger actions when an event occurs such as a change in the data source. For example sending a notification when the Power BI and SQL DWH are refreshed or generate a power automate flow and so on:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b215ff1e-d64d-428b-9e87-509140cb3834)  

It can be used to notify managers if the inventory is lower than a certain level or in sales ...  

### Fabric Pricing:

To understand the pricing we need to understand the structure of Fabric licencing.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/837420cc-e1e7-4306-8390-f75efd09ae9a)  

We have 3 levels:  
- The Tenant : which is the top level of the licence.
- The Capacity : A pool of ressources (SQL, SPARK, PBI ..) that can be used underneeth the tenant level with different CPU and memory levels (CUs units of measurement of capacities) 
- The workspace : the level where we collaborate with other developpers and users and where we create all the data objects
- The domain: it is simply a logical groupment of workspaces to organize data and ressources and the access to it

**Capacities can be so helpful if we want to assign costs to different departments just like virtual warehouses in Snowflake.**  

The SKU table gives the types of Fabric licencing and their prices. It is only compute, the storage is not included in the price.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2f0e27fb-287e-4925-83b9-020a6fcdd762)  

Premium capacity is F64 and more while shared capacity is less than F64. The shared capacity does not support Power BI, licence per user must be purshased.  

More on : https://learn.microsoft.com/en-us/fabric/enterprise/licenses  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b94e8291-23a3-4188-864a-59309f383dbe)  

**Shortcuts**:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6a545372-bff7-4da4-b2a5-c091808f0509)  

### Loading data:

We start first by creating a lakehouse so that we can load data into onelake.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4eb04c63-6342-48ec-b93f-e8022dfde417)  

Once created, we are going to land on the Lakehouse explorer:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/03772ab3-0e8b-4b42-b8bf-89dc8fa286c7)  

In the notifications we can see that the SQL endpoint got created also so we can use external SQL tools to query data:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/52b4fe2b-c590-4c4c-bf0d-e3f16870246a)  

We can find the SQL endpoint to connect using SSMS or Power BI for example:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7d8d41b7-413d-442b-a8b4-28cbea3f1a60)  

We have two "folders" in our Lakehouse: Tables and Files.  

Files contain the files in their raw format (CSV, text ...) but we will not be able to query them using SQL or any other tool until they are moved to the table folder in delta parquet format.  

Now lets upload the files in the Files folder first then transforme them to Delta Parquet in the Tables folder:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4a3da891-2af9-4ae5-a2b6-559aed00cbf6)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/73662b01-03ef-42e2-8a14-cee586e0f5dc)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f3f4df4b-0033-44c7-a702-77bd9e6d38bb)  

The preview will give this :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ab589ff1-eb37-46d3-a800-95a03f1c3535)  

Now we are going to move this file to the tables folder by doing a right click on the file and click on "Load to table":  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/68e3c1dd-1b5a-4edc-b89f-21020148564d)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/eb9f32e7-5497-44b9-aaee-be751e74407d)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b6776dc1-1249-439b-9a0c-4204f4e25b4f)  

This process just transformed a CSV file to a Delta Table Parquet file that now can be quieried by every engine we have inside Fabric.  

This is how the workspace should look like after creating the lakehouse:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/02b4a91e-87cd-4c28-89c9-937ccc08bfbf)  

We have the dafault semantic model and the SQL endpoint created with it which makes it possible to query data using SQL language.  

Inside the lakehouse we can take a look on our newly created table and by doing a right click we can explore the menu:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/044baa4f-108c-441f-92de-417d5c842aea)  

If we open the files of the tables we see all the delta parquet files generated for our table. We can have several files since the table may be partitionned.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c291121c-39ed-4d8b-9def-e030c2534e9d)  

We can see here that we have one file (6MB much lower than the first CSV file since it is compressed) that ends with **.snappy.parquet** with snappy being the compression algorithm.  
We have also the **Delta Log** folder that contains all the transactions made on our table in json formats (this is what adds  the ACID characteristics to the parquet files).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d791989c-78d9-4655-9661-77e828ddc64a)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8479a272-3578-4824-94a8-7790136d9bee)  

The log file contains metadata, the operations made and so on.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/479b0416-6570-4089-bee9-382101644df1)  

We habe also more information regarding the engine used which is in our case the **Apache Spark engine** and we can also see that the Vertipaq-ORDERING technique is set to true meaning that when creating our table it was ordered using the V-ORDER technology to make the querying faster.  

### *Note on Vertipaq-Order (V-ORDER) vs Z-ORDER*:

Both V-Ordering and Z-Ordering are data organization techniques used in Microsoft’s data platform, but they serve different purposes and have distinct functionalities:  

**V-Ordering (VertiPaq Ordering):**

- Timing: V-Ordering happens during write time. It’s applied when data is written to Parquet files, a popular data format for analytics.   
- Purpose: V-Ordering focuses on compression and general read performance. It employs a combination of techniques like sorting, row group distribution, dictionary encoding, and compression on the Parquet files. This compressed, organized format allows data engines to read and process the data faster.  
- Compatibility: V-Ordering is universally compatible. Any engine that can read Parquet files can benefit from the performance improvements offered by V-Ordering.  

**Z-Ordering (Delta Lake Z-Ordering):**

- Timing: Z-Ordering happens during read time (or table optimization). It’s a feature of Delta Lake, a storage layer for big data workloads on Azure Databricks.
- Purpose: Z-Ordering focuses on co-locating frequently accessed data together based on specific columns or predicates (conditions) in your queries. This physical co-location allows data engines to scan and process relevant data chunks faster, improving query performance for workloads with specific access patterns.
- Compatibility: Z-Ordering is specifically designed for Delta Lake tables. It requires tools like Delta Lake to function.

Here’s an analogy to understand the difference:

- V-Ordering: Imagine organizing a library by genre (sorting) and then placing all the books within a genre on the same shelf (row group distribution). This makes browsing for any book within a genre faster (general read performance).
- Z-Ordering: Imagine further organizing the books within a genre by the first letter of the author’s last name (Z-Ordering based on a specific column). This makes finding books by a particular author even faster (optimized read performance for specific queries).

Key Differences Summary:

|Feature	|V-Ordering	|Z-Ordering|
|---|---|---|
|Timing	|During write time	|During read time (or table optimization)|
|Purpose|	Compression & General Read Performance|	Co-locate data for specific queries|
|Compatibility	|Universally compatible	|Requires tools like Delta Lake|

Using Together: V-Ordering and Z-Ordering can be complementary techniques. You can leverage V-Ordering for general compression and performance benefits, and then use Z-Ordering on Delta Lake tables for further optimization based on specific query patterns.  

Now since our Lakehouse has an SQL endpoint, we can switch to it to use SQL to interact with our lakehouse like a data warehoue:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5a715380-880b-48bf-b406-3c1bdc3cf29a)  

### *Note on SQL endpoint*:  

**Note that this is not a full data warehouse, even if it looks like one, it is just an endpoint to query data using SQL and only in read mode.**  
**If we want to modify data we need to switch to lakehouse and use Apache Spark.**  

The SQL Endpoint warehouse is an auto-generated artifact which is created when you create a Lakehouse artifact.  
It is a read-only view of your data, any modification to your data still needs to be made through notebooks here. **This endpoint can be used to query data as well as define views and permissions.**

**The Synapse Data Warehouse, on the other hand, is a SQL engine which is used to query and transform data in our Data Lake (OneLake) and has full transactional, DDL and DML query support.** Data here also uses the Delta format in the same way the Lakehouse artifact does but an important difference is that that you would need to be using **structured data.** Working with Data Warehouse data happens in SQL which gives us transactional support and compatibility with existing T-SQL tools.  

The explorer in the SQL endpoint is similar to a SQL tool where we can see schemas, tables, stored procs queries and so on.  
Now we can run some queries against our data:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b82ade6b-cd7c-4c62-8529-dfaa96c2a65e)  

*The query tool here has intellisense*

**We can also connect using the SQL endpoint in Azure Studio, SSMS and other tools.**  

Also note that the queries are saved automatically for us (Same as snowflake).  

We have also in the same view the **Model tab** where we can see our semantic model where create new one, we can create measures and reports and visuals etc.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4858676b-1735-4c61-8288-1a65cb0e7455)  

We can also in the SQL endpoint create visual queries which is a low-code no-code experience (quite similar to Power Query):  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/0b2b0a68-1711-4951-8d43-9f6123e47e3a)  

Also we can save the queries **as Views**:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/074d793e-bd4c-455d-9f83-5072c3bc6e57)  

But note that the query should be selected:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8ac78d0d-abb8-4f92-ba66-154c9e873583)  


#### Create and modify the PBI semantic model:

We can create and modify the semantic model of our data lakehouse just like we would do in Power BI :

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7391e651-f470-42af-8f0e-a919c169e5fb)  

**Saves are done automatically, no rollback !! ==> That is why we need to use Deschtop for now.**  

Then we can generate Reports in the Browser version of PBI or the Descktop if we want:  

To use PBI descktop we need to copy the SQL endpoint connection string:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e2e15f7a-10d5-4ffc-a7be-89b85a9a82f0)  

In PBI Descktop we Get Data like we are connecting to SQL Database and we paste the connection string we copied:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c6a752f9-9189-4189-a3f7-3a9b956ba674)  

We connect using SSO mode then we can access our lakehouse SQL endpoint (Read mode warehouse):  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4c533f27-f6f7-46a9-a46f-d5e5170b2528)  

We can now choose the tables we want to load to create the semantic model:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6c4bd02e-f596-4d9a-af17-cf1e7ec59d6c)  

**Note that here when we use the endpoint string directly we have only two Options : Direct Query and Import Modes. To connect using DirectLake we need to connect to the lakehouse directly and not the SQL Endpoint using the get data section and searching for lakehouse.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/205e104b-217f-401c-bc14-5f287d0108ec)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/bb958e85-2f50-456f-b881-b210129841bf)  

Now we can access the data and create our Model, and all the measures and so on:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3d90081e-9751-406f-af30-6ab6f44b00fe)  

### Warning !!! :  
We can't access the Lakehouse if we don't activate the **Manage Default Semantic Model**:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/80abe093-bfef-428d-889a-efdd66350dd0)  

### Loading Data Using Dataflows Gen2:  

Just like Dataflows Gen1 in Power BI, Gen2 gives the same experience but compatible with Lakehouses and with other benefits too (20 times faster):  

More on : https://learn.microsoft.com/en-us/fabric/data-factory/dataflows-gen2-overview  

**Bare in mind however that some features we used to have in Gen1 are not yet available:**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7c01cfbe-4189-4500-8160-fde2badedc8e)  

The most valuable element in Dataflows Gen2 is the **Enhanced Compute Engine** that makes it faster by 20 times. This is because it uses an internal SQL cash where it loads data to accelerate the performance of transformation operations on data.  

Also while creating Dataflows Gen2, the work is Saved automatically so no progress lost if the browser is down.  

And the new thing added is that we can set a **destination** for our Dataflow.    

Now lets create a DataFlow using ODATA as source of data:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c7c7efa6-fed5-4f35-a0fb-df50ad548dba)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/202d5dd8-5af6-4d0b-b495-d07b8376ce16)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8c66af8e-1a39-460d-9fb0-1d4efa7efc60)  

Then like we see, in the transformations step, we have PowerQuery experience:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/170ea83f-e5f3-4648-a145-caf12247d5e0)  

Once done, we can create a destination like Lakehouse, warehouse, KQL and Azure DB:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/46f099ae-8811-48b7-b15f-0f4cf3908f9f)  

**Note that we need to check the compatibility of the type of our columns with th destination (for exapmple Datetime are not supported in lakehouses).**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/347b12fd-252b-43f1-a895-33c2f45bc063)  

Once the destination is choosed we can set our Lakehouse:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/936dbdcb-880f-476d-9e7f-762409b05cab)  

**This should be done for each query we have in the Dataflow**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8eaa8abd-fc26-4d70-9357-4e8b940c034a)  

Once the Maping is done, we publish the Dataflow and it will be refreshed a first time and we can set the automatic refresh later. In the lakehouse we can see that Delta Parquet files were created using the Dataflow and now we can query them with every engine we want including SQL.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/74b8fe99-96d3-4239-b3a5-eb523b5f0c86)  



