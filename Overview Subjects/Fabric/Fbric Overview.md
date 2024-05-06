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
By default when we create a Lakhouse, a data warehouse is created with it.   

**Another big benefit of Lakehouses, is that if we have a  lot of teams experts in different languages (python, sql, scala ...) they all can query the same data using their prefered language while if we have a data warehouse we only can use SQL.**  
Imagining we are analyzing data from social media (text, audios, videos ...) we an store this in warehouses but it would be so difficult to handl (not in terms of quantity but the performance and the maintainance needed, ETL, ELT ...). With Lakehouses the data is in Delta Parquet format and we can choose which engine and language to use that will best suite our needs.  

Also for structured data, we may skip a lot of ETL and ELT processes using Lakehouse since the data is in Delta Parquet format ready to be queried.  


