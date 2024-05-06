# From DWH to Data Lake to Data Lakhouse! Conceptual understanding !

This file gives a conceptual explanation to understand the major elements, and benifits, of each milestone of data analytics and why should we use Lakhouses today!

## The problem !?

traditional databases were working just fine using SGBDs to store and analyse data in the local servers of the company, with a lot of obstacles (a lot of things to maintain, configure, networking, expensive infrastructure cost ...), but it was working just fine.  
Moving to the era of big data (starting 2000s), we confronted a big problem with traditional databases systems not being able to process data quickly enough to respond to the business needs.  

A huge step toward solving this problem was the HADOOP releasing in 2006! the concept of data lake appeared and the ease of querying rapidly massive amounts of data using the distribution and map reduce capabilities, life data users was getting easier ..!  
The problem is that with this huge amount of data and to gain in terms of speed in processing data we sacrificed governance (all the rules of data integrity in the traditional SGBDs)! Later, people started asking what is this data, who created it and why, is it current, accurate, and so on !!  
The data lake became quickly a data swamp where we don't know what the data really is and how to use it in advanced analytics!  
The need of data governance is here again so how can we combine this Data Warehouse characteristic with a benefits of data lakes ? Actually the core of SQL data warehouses should have just been evolved to meet the new needs of big data and not to be thrown off ==> The answer is the **Data Lakehouse.**  

### 1. What are the features of a SQL relational database:  

Relational databases answer a lot of our needs in terms of storing, maintaining and analyzing data!  
The conept is quite simple, in our database we store data in tables (with predefined schemas, not to confuse with the schema which is a logical groupment of tables).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1181d918-974b-4609-9f2b-f1ba43832389)  

Each table is specific to a class of objects (**see the ERM data modeling techniques**) and we store data about these objects in the table using **Transactions: Insert, Update and Delete.** We also interat with the database using **CRUD** operations : **Create, Read, Update and Delete**.  
The SQL language contains a lot of other queries , **called also sublanguages** to handl many objectives in the database refered to as : DDL, DML, DMV, DCL ...  

Note that the transactions can be done by user having access to the database, or users having access to applications connected to the database such as CRM, FRP ...  

To keep data clean and well maintained, it has been agreed on that transaction should respect the **ACID** characteristics:  
- Atomicity
- Consistency
- Isolation
- Durability

It is simply **All or nothing concept** where we apply all the sets of maintanace or nothing at all. For example, lets take the following transaction:  

```
      Insert in sales table
      Insert in costumer table
      Update in sales table the date
      COMMIT
```
This package of operations is called a **transation**. Now, in our data database there is somthing called **referential integrity** where for example the sales table connot contain a sale done by a costumer that does not exist in the costumer table.  
In our transaction, if the insert step of sales is not done, because the costumer is not yet there in costumer then the whole transaction is **Not COMMITED** and will **ROLLBACK.** This will garanty the consistancy.  
What also makes the integrity, data quality, of data leveled up is the **constraints** such as:  
- Domain constraints (Data Types)
- Key Constraints (uniqueness)
- Entity Integrity constraints (primary key cannot be null for example)
- Referential integrity constraints
- Column value constraints (for example, we can put a rule on sales values to be greater than 0)

What supports all this mechanism of transactions is the **Transaction logs.** These are external files maintained when the transactions are made. It keeps track of all the changes made and makes it able to go back to the **last point** (When we set a **COMMIT** statement, it is like a save) of the DB is something wrong occures.  
In parallel with this, DBAs will make backups of databases so that if it is deleted we can recover it again from the last accurate point and we apply the transactions needed using the transaction logs.  
Also, traditional databases handle the security aspects, who can access data, what permissions and so on and also **Triggers** that are SQL rules (queries like checking referential integrity) that runs if an event happens and also can be used to insert auto,atic logs.  

### 2. Database supported workloads:

Two Workloads are supported in SQL Databases:  

| Transactional Databases | Data Warehouses |
| --- | --- |
|Store the business operational systems such as sales, banking, finance ..| Created for data analysis and reporting purpose to suport decision making| 
|Focus on maintaining data (insert, update and delete)|Focus on querying and aggregating data to get quick answers for analysis|
|Little amount of transactions per user (but could have million users around the world)|A batch insert of data, usualy at night|
|It is the source of data and it is crucial and needs to be reliable, secure, fault tolerant, recoverable ... to not lose operational data|Needs to give fast query results, reliable, aggreation of large datasets, needs data integration (Pull data from sources and aggregate it)|
|Uses Entity Relationship Modeling ==> **Normalization** by eliminating the data redundancy|**Denormalization** using dimentional modeling (Star Schema) to make queries faster|

Now that we have seen all these features in the traditional Databases and warehouse, we should know that the **Lakehouse** is trying to emulate all these features that are important for organizations in the world of big data (large file systems, data lakes ...).  

Data **Lake** + Data Ware**house** = Data **Lakehouse**  

Actually, adding those features into a data lake is not that easy for several reasons: 

|Old Warehouse|Data Lake|
|---|---|
|Single box of data, which is the database, everything is local. E.g you can run a query to check the uniqueness of row before inserting it and it will be done fast against the table, simple architecture, single **monolithique interface**, no network needed|Complexe architecture of **files**, stored in large network of machines (nodes), (Scaledout data plateforme: Databricks, Snowflake or synapse), so checking the existence of a value for example will need a big work of shuffling to and search in all the nodes of the cluster|
|Data stored as one database|Data stored as separate files|
|Data maintenance is easy|Not that easy since data is in a form of files, someone can delete a file while it is a part of your database|

The new data plateforms have made a great progress emulating the features of traditional databases in the lakehouse architecture.  

- **Spark** which is a so powerful processing engine that supports SQL queries against files. Basicly, it does what we call **Schema On Read** by defining a SQL schema layer called **Hive** to read the data but **Not Maintaining it!.**  

- Then, transaction has been implemented in data lakes, and we started talking about **Delta Lakes** that are data lakes supporting transactions by storing data in a form of **parquet file** in a **Delta Format**.
- Delta adds transaction logs in the data lake for all the files to keep track of changes.
- That brought the ACID support, enabling the COMMIT or ROLLBACK functionality.
- Constraints were also added, such as identity column to use primary keys, but still evolving.
- Databricks added also the referential integrity constraints.
- Security, in opposition to traditional databases (Oracle, SQL server ...) is not a real issue here as long as the cloud security is insured. We can however do some grant and envok commands but not like we do in traditional systems.
- Data Backups (Using extracting tools to export data in a backup file) are less relevent here because we are talking about distributed files not like one database (data as a whole). So it is up to the user to manage the data and make sure it is well stored (Making copies of data in different storages ...). Governance tools are however added each time in the scaled out data plateforms.
- **Schema evolution** : A huge add in the world of scaled out data plateforms. Traditionaly, the schema evolves by adding columns sundenly which breaks the whole process of data integration, transformation etc. Databricks for example, added t a feature to specify in code that we allow the schema to evolve by adding new columns if the source changes.

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/43a0ebea-ea3e-4c54-b261-0f49dea5ee84)

**Another big benefit of Lakehouses, is that if we have a  lot of teams experts in different languages (python, sql, scala ...) they all can query the same data using their prefered language while if we have a data warehouse we only can use SQL.**  
Imagining we are analyzing data from social media (text, audios, videos ...) we an store this in warehouses but it would be so difficult to handl (not in terms of quantity but the performance and the maintainance needed, ETL, ELT ...). With Lakehouses the data is in Delta Parquet format and we can choose which engine and language to use that will best suite our needs.  
 
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c3e65f1b-e20f-4891-9035-fa821d61c001)  


