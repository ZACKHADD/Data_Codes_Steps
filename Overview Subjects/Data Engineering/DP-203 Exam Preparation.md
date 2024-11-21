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
- 
