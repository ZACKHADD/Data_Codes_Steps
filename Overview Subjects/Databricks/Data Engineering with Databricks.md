# Data Engineering using Databricks Overview 

Databricks is a powerful data plateforme (Paas) suitable for big data processing, ingestion and AI workloads. The plateforme is built on top of **Apache Spark** which is a tool for big data processing (using memory).  
Spark has no UI so Databricks offers a great user experience to use Spark and collaborate with lage teams.  

**What should be understood is that Databricks till now is not made for replacing OLTP (even if they add some features such as ACID support for transactions ..) systems as it is not a RDBMS. Snowflake on the other hand is a Saas Datawarehouse at scale (Cloud based) solution based on SQL (where the storage is seperated from the compute). The data warehousing experience however is supported in databricks using the SQL clusters.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7581505e-194d-4ff4-9c43-8aaf93b383af)  

## Apache Spark:

Moving from data processing using single machine to **Cluster** (group of machine that share the workload execution) needed a powerful framework to do the coordination. Spark is a tool for just that, managing and coordinating the execution of tasks on data across a
cluster of computers.  

### Cluster architecture:

The cluster of machines that Spark will leverage to execute tasks will be managed by a cluster manager like Spark’s Standalone cluster manager, YARN, or Mesos. We then submit Spark Applications to these cluster managers which will
grant resources to our application so that we can complete our work.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ac58011e-1ba8-48d0-8493-15039b4650ba)  

Spark Applications consist of a driver process and a set of executor processes. The driver process runs the **main()** function, sits on a node in the cluster, and is responsible for three things: 
- maintaining information about the Spark Application.
- responding to a user’s program or input; and analyzing.
- distributing, and scheduling work across the executors (defined momentarily).
The driver process is absolutely essential - it’s the heart of a Spark Application and maintains all relevant information during the lifetime of the application.

The executors are responsible for actually executing the work that the driver assigns them. This means, each executor is responsible for only two things: executing code assigned to it by the driver and reporting the state of the computation, on that executor, back to the driver node.  

The cluster manager, on the other hand, controls physical machines and allocates resources to Spark Applications. This can be one of several core cluster managers: Spark’s standalone cluster manager, YARN, or Mesos. This means that there can be multiple Spark Applications running on a cluster at the same time. We will talk more in depth about cluster managers in Part IV: Production Applications of this book.  

While our executors, for the most part, will always be running Spark code. The driver can be “driven” from a number of different languages through **Spark’s Language APIs**.  

**Two types of APIs:**
- Structured APIs (High level)
- Unstructured APIs (low level) RDD API

The term API here means either we deal directly with the RDD which are the core of Spark (then we say it is Low level) or we deal with another abstraction such as Dataframes (built on top of RDDs and Datasets) (then we say it is High level):  
- RDD API (Spark Core): user manipulates directly the RDDs, it is the low level API
- Dataset API (Spark SQL): User manipulates high level typed objects
- DataFrame API (Spark SQL): User manipulates high level untyped objects
- SQL API (Spark SQL): User writes SQL query strings
  
They're called APIs because they're essentially just different interfaces to exactly the same data.  
**For example we can use Spark SQL interface with DataFrame which provides all common SQL functions, but if we decide to use RDDs, we would need to write SQL functions ourselves using RDD transformations.**  
It is recomanded to always use high level APIs (Dataframe and Datasets) because spark then can use optimizers to give the best performance. Otherwise, using RDDs directly will oblige us to handle all the details in terms of transformations and actions that can be easily done with the high level APIs and also handl the physical deployment of the job.  

There is a **SparkSession** available to the user. the SparkSession will be the entrance point to running Spark code. When using Spark from a Python or R, the user never writes explicit JVM instructions, but instead writes Python and R code that Spark will translate into code that Spark can then run on the executor JVMs.  
**Note that the spark context and spark session need to be initialized to be able to connect with the cluster. However, in Databricks this is done automaticaly.**

### Data Structures:
The APIs we have seen above will give us 3 main data structures in Spark:
- RDDs : which are the core of Spark.
- Dataframes : Came later to solve the problems that RDDs were facing especially with structured data and also to offer a more friendly API to developpers.
- Datasets : The last type to show and it was a bridge to offer the best of the two worlds. **Supported only by Scala**.  
Keys differences ans similarities:

|Feature|RDD|Dataframe|Dataset|
|---|---|---|---|
|Data Representation|Distributed immutable data|Distributed Structured data with schema|extension of dataframes with optional schemas and type safety (obligation to declare the type of the data and gives error if not the good type) feature and object oriented interface (for example a table is a class with columns as properties)|
|Optimization|Optimization plan needs to be writen by the developper|Uses Catalyst optimizer|Uses Catalyst optimizer|
|Projection of schema|No shcema. Needed to be defined manually|Automatically found|Automatically found|
|Type-safety|yes|no|yes|
|Error analysis|Compile time|Rune time|Compile time|
|Type of users|Developpers requiring precise control|Data Engineers, Data Analysts|Data Professionals needing a balance between control and convinience|
|Aggregation Operation|Slow in aggregations|The fastest structure for aggregations|Faster than RDDs|
|Transformations|Lambda based transformations: map(), reduce(), filter()|Expression based transformation:  select(), where(), join()|Uses both|

**Immutability means that once created, a data structure cannot be changed. Instead, any operation on the data structure creates a new one. By embracing immutability, Spark leverages these functional programming features to enhance performance and maintain consistency in its distributed environment.**  

Note that we can transform back and forth from Dataframes and Datasets while if we transform RDD to dataframe we lose the auto optimization of spark engine.  
**As spark evoloved with time, we no longer have Dataframes as a separate data structure. It is simply a Dataset[Rows]. If we use an object other then row inside it becomes Dataset and spark handles this implicite conversion. We talk then about structured API simply.**  
**Dataset, when not containing Rows, can contain Objects of a class that can be manipulated.**  

### DAG (Directed Acyclic Graph) Scheduler:  
DAG is a fundamental concept that plays a crucial role in the Spark execution model. The DAG is “directed” because the operations are executed in a specific order, and “acyclic” because there are no loops or cycles in the execution plan. This means that each stage depends on the completion of the previous stage, and each task within a stage can run independently of the other.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a13238ff-3c22-4327-aa2c-3e5bbdb5951c)

The fact that DAG is **acyclic** allows Spark to optimize and schedule the execution of the operations effectively, as it can determine the dependencies and execute the stages in the most efficient order.  
The  DAG schedular works as follows:
- It computes a DAG of stages for each job, keeps track of which RDDs and stage outputs are materialized, and finds a minimal schedule to run the job.
- It then submits stages as TaskSets to an underlying TaskScheduler implementation that runs them on the cluster.
- It converts a logical execution plan (which consists of the RDD lineage formed through RDD transformations) into a physical execution plan.  

### Spark Job:
A job in Spark refers to a sequence of transformations on data. Whenever an action like count(), first(), collect(), and save() is called on RDD (Resilient Distributed Datasets), a job is created. A job could be thought of as the total work that your Spark application needs to perform, broken down into a series of steps.  
Consider a scenario where you’re executing a Spark program, and you call the action count() to get the number of elements. This will create a Spark job. If further in your program, you call collect(), another job will be created. So, a Spark application could have multiple jobs, depending upon the number of actions.  

### Spark Stage:

A stage in Spark represents a sequence of transformations that can be executed in a single pass, i.e., without any shuffling of data. When a job is divided, it is split into stages. Each stage comprises tasks, and all the tasks within a stage perform the same computation.  
The boundary between two stages is drawn when transformations cause data shuffling across partitions. Transformations in Spark are categorized into two types:  
**narrow and wide**.  
Narrow transformations, like **map()**, **filter()**, and **union()**, can be done within a single partition. But for wide transformations like **groupByKey()**, **reduceByKey()**, or **join()**, **data from all partitions may need to be combined**, thus necessitating **shuffling** and marking the start of a new stage.

### Transformations and Actions:

- **Transformation** : Spark Transformation is a function that produces new RDD (Or any data structure) from the existing RDDs. It takes RDD as input and produces one or more RDD as output. Each time it creates new RDD when we apply any transformation. Thus, the so input RDDs, cannot be changed since RDD are immutable in nature. Transformations are the core of how you will be expressing your business logic using Spark.  

The two kinds of transformations are : 
**Narrow** that are performed with the **pipelining** meaning that if we specify multiple filters on DataFrames they’ll all be performed **in-memory**. **Wide** on the other hand are transformations that involve **Shuffle** and will write the results to disk. Data is typically first spilled to disk and then read back into memory as needed. This is because shuffling can involve moving large amounts of data between nodes, and **memory is often limited in distributed systems**.  
Transformation are said **Lazy**. **Lazy evaulation** means that Spark will wait until the very last moment to execute the graph of computation instructions. In Spark, **instead of modifying the data immediately when we express some operation, we build up a plan of transformations that we would like to apply to our source data**. Spark, by waiting until the last minute to execute the code, will compile this plan from your raw, DataFrame transformations, to an **efficient physical plan** that will run as efficiently as possible across the cluster.  

More on : https://stackoverflow.com/questions/49753298/transformation-vs-action-in-the-context-of-laziness  

- **Actions**: An action instructs Spark to compute a result from a series of transformations. For example **saveAsTextFile()** or **count()** or **.show()**. One specified, tha action load the data into the memory to perform the computation and return the result.  

### Note on Suffling: 

Shuffling is an expensive operation that needs a great attention to reduce the compute cost of the organisation.  
It is an operation that moves the data across the network causing a network trafic that impacts the performance and the cost. Suffle is done when we apply a wide transformation (Groupby(), join() ..).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/0133a958-e8bb-4596-a09e-133380efd55c)  


### Spark Task:

A task in Spark is the smallest unit of work that can be scheduled. Each stage is divided into tasks. A task is a unit of execution that runs on a single machine. 
 When a stage comprises transformations on an RDD, those transformations are packaged into a task to be executed on a single executor.  
For example, if you have a Spark job that is divided into two stages and you’re running it on a cluster with two executors, each stage could be divided into two tasks. Each executor would then run a task in parallel, performing the transformations defined in that task on its subset of the data.  

More on :  
- https://medium.com/@diehardankush/what-are-job-stage-and-task-in-apache-spark-2fc0d326c15f)


### Data Partitioning:

In order to allow every executor to perform work in parallel, Spark breaks up the data into chunks, called partitions. A partition is a collection of rows that sit on one physical machine in our cluster. A DataFrame’s partitions represent how the data is physically distributed across your cluster of machines during execution. If you have one partition, Spark will only have a parallelism of one even if you have thousands of executors. If you have many partitions, but only one executor Spark will still only have a parallelism of one because there is only one computation resource.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/0304bd35-da51-413a-9b07-bb38fc67fdcc)  

We always need to avoid having too big or small files. Having bigger partitioned data will lead to some of the executor doing the heavy load work, while others are just sitting idle. We need to ensure that no executors in the cluster is sitting idle due to the skewed workload distribution across the executors. This will lead to increased data processing time because of weak utilisation of the cluster.  
On the other hand, having too many small files may require lots of shuffling data on disk space, taking a lot of your network compute and Driver memory.  

|Too Small|Too Larg|
|------|------|
|Slow read time downstream|Long computation time|
|Large task creation overhead|Slow write times|
|Driver OOM (Out Of Memory Error)| Executor OOM|

The default file size in Spark is 128MB but the recommendation is to keep your partition file size ranging 256MB to 1GB.  

More On:  
- https://medium.com/@dipayandev/everything-you-need-to-understand-data-partitioning-in-spark-487d4be63b9c
- https://www.youtube.com/watch?v=hvF7tY2-L3U&ab_channel=PalantirDevelopers  

## Cluster design and configuration:
The choice of the cluster type and number of nodes depends on the type of work we want to perform:
- For ETL/ELT normal jobs: Memory Optimized cluster
- For Normal Dev and interactive jobs: General purpose cluster
- For heavy jobs needing data shuffeling: Storage optimized cluster (caching enhanced)

More on:
- https://medium.com/technology-and-trends/estimating-the-size-of-spark-cluster-1cb4d59c5a03

## Databricks on top of Spark:

Databricks resides on top of Spark engin giving a great GUI to use and interact with spark using notebooks. It creates a Lakehouse logic that combines the benefits of data warehouses and data lakes.  

The architecture of Databricks is the following:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/69c57df2-92d6-43ed-acbf-c2237c3a5fa7)  

- Data Plane is where the data is processed like in the workspace clusters or SQL warehouse and so on.
- Control Plane is the backend of databricks where we precise what ressources to create and how to be created and manages the deployment and also leverage data governance as well as orchestration of jobs.

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/af7e1999-d7d7-4f6c-99f1-fc41ff63e0da)  

### Data Storage :
Data is physicaly stored in the data lake of the cloud provider (Azure, AWS or GCP). However, databricks has a DBFS (Databricks File storage) that is a layer on top of the original data to view it as if it resides in databricks.  

**Explanation Stackoverflow:** *DBFS is an abstraction layer on top of S3 (or Blob in Azure or GCP) that lets us access data as if it were a local file system. By default when We deploy Databricks We create a bucket that is used for storage and can be accessed via DBFS. When We mount to DBFS, We are essentially mounting a S3 bucket to a path on DBFS.*  

### Start working with data :

To start working with data, we need first to create a cluster that will do the processing for us.  
#### Cluster types:
These are several types of clusters available in Databricks depending on the objectif:
- Serverless compute for notebooks (Public Preview): On-demand, scalable compute used to execute SQL and Python code in notebooks.
- Serverless compute for workflows (Public Preview): On-demand, scalable compute used to run your Databricks jobs without configuring and deploying infrastructure.
- **All-Purpose compute**: Provisioned compute used to analyze data in notebooks. We can create, terminate, and restart this compute using the UI, CLI, or REST API.
- **Job compute**: Provisioned compute used to run automated jobs. The Databricks job scheduler automatically creates a job compute whenever a job is configured to run on new compute. The compute terminates when the job is complete. We cannot restart a job compute.
- Instance pools: Compute with idle, ready-to-use instances, used to reduce start and autoscaling times. We can create this compute using the UI, CLI, or REST API.
- **Serverless SQL warehouses**: On-demand elastic compute used to run SQL commands only on data objects in the SQL editor or interactive notebooks. We can create SQL warehouses using the UI, CLI, or REST API.
- **Classic SQL warehouses**: Used to run SQL commands only on data objects in the SQL editor or interactive notebooks. We can create SQL warehouses using the UI, CLI, or REST API.

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/512d5c40-075c-4a70-9a16-64d2850b88fe)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b5a29998-98cb-4480-a8b4-0b9a2bbc7826)  
*Note that this screen is from the community edition*  

Depending on the objectif and the workload, we decise how the cluster should be configured. Two main types of implementation:
- Single Node: for lightweight exploratory analysis.
- Standard (Muli Node) : For production workloads (At least 2 VMs).

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/58db7077-2a70-47df-bcf3-f2f91c4285d4)  

#### Security features:
To Manage the access to clusters and the ability to create them, we do that in the workspace. The workspace admin gives the roles to users to set what they can and cannot do with the clusters.  
**Account admins** handle general account management, and **workspace admins** manage the settings and features of individual workspaces in the account. Both account and workspace admins manage Databricks users, service principals, and groups, as well as authentication settings and access control.  

This security features regarding who can access what are handeled using **Access Control systems:**

|Securable object|Access control system|
|---|---|
|Workspace-level securable objects|Access control lists|
|Account-level securable objects|Account role based access control|
|Data securable objects|Unity Catalog|

In Databricks, we can use **access control lists (ACLs)** to configure permission to access workspace objects such as notebooks and SQL Warehouses. All workspace admin users can manage access control lists, as can users who have been given delegated permissions to manage access control lists.  

Account role based access control are used to configure permission to use account-level objects such as service principals and groups. Account roles are defined once, in the account, and apply across all workspaces. All account admin users can manage account roles, as can users who have been given delegated permissions to manage them, such as group managers and service principal managers.  

In addition to access control on securable objects, there are built-in roles on the Databricks platform. Users, service principals, and groups can be assigned roles. There are two main levels of admin privileges available on the Databricks platform:

- Account admins: Manage the Databricks account, including workspace creation, user management, cloud resources, and account usage monitoring.
- Workspace admins: Manage workspace identities, access control, settings, and features for individual workspaces in the account.

Additionally, users can be assigned these feature-specific admin roles, which have narrower sets of privileges:

- Marketplace admins: Manage their account’s Databricks Marketplace provider profile, including creating and managing Marketplace listings.
- Metastore admins: Manage privileges and ownership for all securable objects within a Unity Catalog metastore, such as who can create catalogs or query a table.

Users can also be assigned to be workspace users. A workspace user has the ability to log in to a workspace, where they can be granted workspace-level permissions.  

**Workspaces Policies:**
A policy is a tool workspace admins can use to limit a user or group’s compute creation permissions based on a set of policy rules.  
Policies provide the following benefits:
- Limit users to creating clusters with prescribed settings.
- Limit users to creating a certain number of clusters.
- Simplify the user interface and enable more users to create their own clusters (by fixing and hiding some values).
- Control cost by limiting per cluster maximum cost (by setting limits on attributes whose values contribute to hourly price).
- Enforce cluster-scoped library installations.

We can also add **libraries** to a policy so libraries are automatically installed on compute resources. We can add a maximum of 500 libraries to a policy.  

#### Create a cluster:

Under the compute section, we can create a cluster and specify the detailed configuration depending on the objectif of the compute as explained before.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/871da968-e795-4e91-9b6e-3860a270b717)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3ca62028-1a5f-4e8a-8079-caa8b719616e)  

We can specify details such as the runtime to use, the terminate after duration, Min and Max workers for the scalability purpose etc.  
Also we can grant permissions on the cluster to users:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8179cda2-ea52-44f7-888f-cfc37707ccb7)  

#### Start Working with notebooks:

Just like jupyter notebooks, we use databricks notebooks to code and interact with spark.  
Magic commands gives the possibility to set some key envirement variables like the language to code with (we can mix as we want).  
Examples of magic commands:  

- %fs, which is the same as making dbutils.fs calls. See Mix languages.
- %sh, which runs a command by using the cell magic %%script on the local machine. This does not run the command in the remote Databricks workspace.
- %md and %md-sandbox, which runs the cell magic %%markdown.
- %sql, which runs spark.sql.
- %pip, which runs pip install on the **local machine. This does not run pip install in the remote Databricks workspace**.
- %run, which runs another notebook. 

Limitations include:
- The notebooks magics %r and %scala are not supported and display an error if called.
- The notebook magic %sql does not support some DML commands, such as **Show Tables**.

**Databricks Utilities (dbutils)** is also a key tool that works in Python, R and Scala to:
- Work with files and object storage efficiently.
- Work with secrets.

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3f5305e8-1a01-462d-99e6-c7b70f07d821)  

#### In memory computing (cashing or persesting):

One of the most important capabilities in Spark is **persisting** (or caching) a dataset in memory across operations. When you persist an RDD, each node stores any partitions of it that it computes in memory and reuses them in other actions on that dataset.  
Also, one can provide Storage Level as **MEMORY_AND_DISK**.  
MEMORY_AND_DISK : Store RDD as **deserialized Java objects in the JVM**. If the **RDD does not fit in memory, store the partitions that don't fit on disk**, and read them from there when they're needed.  

#### SQL (DBMS) vs Spark SQL:

Relational database management and querying are done using SQL. Its main applications are in the retrieval, updating, insertion, and deletion of data from databases.  DBMS are great for relational databases where the consistancy is a must and constraints are to be conserved.  
Spark SQL however: is a part of the open-source distributed computing system Apache Spark. By extending the SQL language, Spark SQL makes it possible to query structured data in Spark programmes. With it, users can run SQL queries in addition to Spark programmes.  
**SparkSQL is a pure SQL interface that uses spark as execution engine**.
Relational Databases have features **(referential integrity for example)** that Spark (distributed computing file systems) does not have. However, Databricks is trying to add the **Relational Database functionalities** to have the same abilities of the distributed DBMS such as : Snowflake, Amazon REDSHIFT, BigQuery and Synapse.  

**Note that when we use High level APIs, Dataframes and datasets, we are using Spark SQL as the Python or R code is translated to SQL and executed in Spark engine.**  

Spark SQL query Execution:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/42f7bad8-1120-4846-b73e-363bda2bb9fb)  

More on :  
https://github.com/ZACKHADD/Data_Codes_Steps/blob/main/Overview%20Subjects/Data%20Engineering/DWH%2C%20Data%20lake%2C%20Data%20Lakhouse.md  
https://medium.com/@rganesh0203/sql-vs-spark-sql-15dd385a7b40  
Comparison DBMS vs Spark : https://db-engines.com/en/system/MySQL%3BSpark+SQL  
https://www.youtube.com/watch?v=Kz-oYfYEsC0&list=PL7_h0bRfL52qWoCcS18nXcT1s-5rSa1yp&index=7&ab_channel=BryanCafferky  
https://fr.slideshare.net/slideshow/a-deep-dive-into-query-execution-engine-of-spark-sql/144699027  
https://dataninjago.com/2022/02/14/spark-sql-query-engine-deep-dive-19-adaptive-query-execution-part-1/

### Data warehousing in Databricks:

In this section we will try to use SQL to create some data objects to simulate a datawarehouse in Databricks.  
We start by creating a cluster that will attach to our notebook and the we start creating our DW.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/468d5367-55bc-4a63-b8f3-41f1ffcec7b5)  

#### Create a Database:

A database in databricks is not the same as in traditionnal relational databases as we don't have all the options and constraints (primary keys, unique idetifiers ...). There are just files stored in format that makes it possible to query them using SQL.  
A database in this context would be like folder with a hierarchy that contains many files having some kind of relation between them. Note that now Databricks adds a lot of features that gets closer and closer to the classic SQL Datawarehouses capabilities.  

![image](https://github.com/user-attachments/assets/03e5d0c5-17e6-4213-851b-dffeb388aa23)  

Once we create a database, we can add several properties to tell more about it such as if it is PROD database or DEV and also add comments.  


# Complete other concepts before going to the next chapters

## Data ingestion with Delta lake:

### Delta lake and data objects:

Delta lake is a type of storage of data in a data lake using parquet formats. It is optimizes and built for scalable metadata handling. This latter is actually the big advantage added to data lakes : the ability to query and write big data using some key features of SQL databases such as ACID. 
Delta Lake adds a lot of best practices inhereted from traditional databases engines to data lakes such as : 
  - Dealing with modifications as a whole transation that rollback if it fails.
  - Data integrity garanteed with jobs failure : if a job fails data is not written.
  - historical data version management (time traveling) using logs.
  - It adds also the ability to skipp scanning unnecessary data (files) using files statistics to optimize queries.
The delta lake architecture (lakehouse) is mainly as follows:  

![{C7EF915A-8B35-485C-AE70-07E02E7FA650}](https://github.com/user-attachments/assets/c8fab1b3-61c2-4d65-9f27-ce51a59c1ea8)  

The structure of data objects in databricks is the following : 

![{40244094-9621-4482-A943-0EA4B9E483F0}](https://github.com/user-attachments/assets/b5c7f6bb-92bb-457d-98c1-6acc717c7b5b)  

- The Metastore is built by the administrator and it is assigned to a workspace (or several workspaces) and it has a default storage location for managed tables.
- Catalogs are groups of database (schemas in the traditionnal SGBDS)
- Schema is like a database if we work in relationnal systems. It is a logical groupment of files.

![{10B3E4DC-590A-489F-947E-98AD1C6A9217}](https://github.com/user-attachments/assets/8ed659a2-a17b-4e60-8ae0-d20efb8a9f12)  
![{D4C2A3B7-B1FE-47D9-906C-82F714498E41}](https://github.com/user-attachments/assets/72c2d612-75ed-46a2-bc5c-d964264bbda8)  

#### Liquid Clustring 
To optimize querying data in delta lake, we generaly partition tables (Z-Order) and decide on the frequency to run some optimization commands suh as **OPTIMIZE** and **VACCUM**. Onother way is presented recently by databricks, is **Liquid Clustring**.  
Liquid Clustering is a data organization technique that ensures data within a Delta Lake table is clustered dynamically and adaptively without requiring explicit user-defined partitioning. Unlike static partitioning, which relies on rigid boundaries, liquid clustering adjusts clustering behavior automatically based on the size and distribution of the data.  
It makes of partitionning and Z-order of data automatic and also runs automaticaly the **OPTIMIZE** and **VACCUM** operations behind the scene.  

#### Deletion Vectors

Deletion vectors are a storage optimization feature you can enable on Delta Lake tables. By default, when a single row in a data file is deleted, the entire Parquet file containing the record must be rewritten. With deletion vectors enabled for the table, DELETE, UPDATE, and MERGE operations use deletion vectors to mark existing rows as removed or changed without rewriting the Parquet file. Subsequent reads on the table resolve the current table state by applying the deletions indicated by deletion vectors to the most recent table version.  

- It is a logical Deletion first : Instead of physically removing rows from files, a deletion vector records the identifiers (e.g., row numbers or offsets) of rows that are considered deleted.
  - The underlying data files remain unchanged, and deletions are tracked separately.
- Compact Representation: Deletion vectors use a compact structure, like bitmaps or lists, to represent deleted rows. For example:
      - A bitmap might use 1 for active rows and 0 for deleted rows.
      - A list might store the row indices of deleted rows (e.g., [2, 7, 15]).
- Query Execution: During query execution, the deletion vector is applied dynamically to filter out deleted rows. For example:
      - A query scanning a file with a deletion vector will read only the rows not marked as deleted in the vector.
- Deferred Deletion: **Physical deletion** happens later, often during maintenance tasks like **compaction** or **OPTIMIZE** operations, when files are rewritten to remove deleted rows permanently.

The VACCUM and OPTIMIZE operations are now done trough Predictive I/O which is a machine learning model that decides automtically when to trigger these operations and hence accelerate the compute (does not these operations while data are heavely queried).  

### Set up and load delta tables:

We can read data directly from a parquet files in data lake (such as azure) as follows:  

```SQL

  # Note that what is between brackets is a path variable for dbfs:/mnt/dbacademy-datasets/data-engineer-learning-path/v04
       SELECT items FROM PARQUET.`${DA.paths.datasets}/ecommerce/raw/sales-historical` LIMIT 10;
  # We can also create this as a temporary view
      CREATE OR REPLACE TEMP VIEW sales_unparsed AS
          SELECT items FROM PARQUET.`${DA.paths.datasets}/ecommerce/raw/sales-historical` LIMIT 10;
      
```  

we can do the same thing using a csv file. We replace PARQUET with CSV. However we need to specify other details such as the delimiter, headers and so on so that the data can be rendered correctly.  

We can use two approaches : USING & OPTIONS or READ_FILES() table valued function :

```SQL
        CREATE OR REPLACE TEMP VIEW sales_tmp_vw
          (order_id LONG, email STRING, transactions_timestamp LONG, total_item_quantity INTEGER, purchase_revenue_in_usd DOUBLE, unique_items INTEGER, items STRING)
        USING CSV
        OPTIONS (
          path = "${da.paths.datasets}/ecommerce/raw/sales-csv",
          header = "true",
          delimiter = "|"
        );

        SELECT * FROM read_files(csv.`${da.paths.datasets}/ecommerce/raw/sales-csv`,
              format => cvs,
              sep => "|",
              header => true,
              mode => "FAILFAST");
        
```

**Note that it is never a good practice to store data in DBFS !! it is recommanded to use the object storage systems such as S3, ADLS gen2 and GCP. Mainly because DBFS is more expensive and has limited data management features. Actually if we try to create an external table using DBFS we will get an error**  

#### Incremental load using COPY INTO:

A great thing about COPY INTO when storing data in delta format is that it keeps track of the files metadata we are ingesting data from and if the metadata (size, name, modification time ..) didn't change then it will not reaload the same file twice !!

```SQL
            COPY INTO users_bronz
            FROM "${da.paths.datasets}/ecommerce/raw/users-30m"
            FILEFORMAT = parquet
            COPY_OPTIONS ('mergeSchema'='true');
```

#### Some built-in functions:

```SQL

            COPY INTO users_bronze FROM
              (SELECT *,
                 cast(cast(user_first_touch_timestamp/1e6 AS TIMESTAMP) AS DATE) first_touch_date,
                 current_timestamp() updated,
                 input_file_name() source_file
                 FROM "${da.paths.datasets}/ecommerce/raw/users-historical/")
                 FILEFORMAT = PARQUET
                 COPY_OPTIONS ('mergeSchema'='true');

```

#### Clonning tables:

Two types of clonning : DEEP and SHALLAW.  

A deep clone in Databricks creates a completely independent copy of a Delta table, including both its data files and transaction log history. Unlike a shallow clone, which only references the source table's data files, a deep clone physically copies the data to a new location. However, note that in a shallow clone, changes to the data in the original table will impact the cloned table, because the shallow clone shares the same underlying data files as the source table. The shallow clone does not create a physical copy of the data; it merely creates a new table that references the same data files and schema.  

#### Data overwriting:

When we do data overwriting we can trak changes in the table and perform time travel.  

```SQL
            CREATE OR REPLACE TABLE events AS 
                SELECT * FROM PARQUET.`${da.paths.datasets}/ecommerce/raw/users-historical/`;
            
                DESCRIBE HISTORY events;
```

![{9F83FFD1-C173-40B3-95B0-BE9BE7399BC3}](https://github.com/user-attachments/assets/e97963af-d8a6-4d53-bd9d-4ce3c76b5b7e)  

The DESCRIBE HISTORY gives us the delta log of our delta table.  
We can also use MERGE INTO : 

```SQL
              MERGE INTO users_bronze a
              USING users_update b
              ON a.user_id = b.user_id
              WHEN MATCHED AND a.email IS NULL AND b.email IS NOT NULL THEN
                UPDATE SET email = b.email, updated = b.updated
              WHEN NOT MATCHED THEN
                INSERT (user_id, email, updated)
                VALUES (b.user_id, b.email, b.updated)
```

#### GENERATED columns:

We can also use generated columns which are just calculated ones depending on the values of other columns.  

```SQL
            CREATE OR REPLACE TABLE purchase_dates (
              id STRING,
              transaction_timestamp STRING,
              price STRING,
              date DATE GENERATED ALWAYS AS (
                cast(cast(transaction_timestamp/1e6 AS TIMESTAMP) AS DATE)
              ) COMMENT "generated baed on 'transaction_timestamp' column"
            )
```

Two types of generated columns exist : Virtual (which is calculated in query time) and stored that are phisicaly stored. Note that databricks supports only Stored columns.  

We need to SET spark.databricks.delta.schema.autoMerge.enabled=true variable so that the generated column can be added.  

#### Constraints:

Databricks only support natively **Two types of constraints** : NOT NULL & CHECK. Wen can see if our table has contrains using DESCRIBE EXTENDED.  

**It is a good practice to use Working Tables as intermediate ones before loading to silver or gold layer.**  

#### UDFS (user defined functions):

 
