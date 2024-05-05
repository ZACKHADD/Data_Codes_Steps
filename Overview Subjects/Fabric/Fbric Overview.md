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

It is the data lake of fabric, very similar to **OneDrive** but data are stored differently (in parquet format and delta tables format).  

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

