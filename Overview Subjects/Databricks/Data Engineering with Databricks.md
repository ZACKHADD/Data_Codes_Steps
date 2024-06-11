# Data Engineering using Databricks Overview 

Databricks is a powerful data plateforme suitable for big data processing, ingestion and AI workloads. The plateforme is built on top of **Apache Spark** which is a tool for big data processing (using memory).  
Spark has no UI so Databricks offers a great user experience to use Spark and collaborate with lage teams.  

## Apache Spark:

Moving from data processing using single machine to **Cluster** (group of machine that share the workload execution) needed a powerful framework to do the coordination. Spark is a tool for just that, managing and coordinating the execution of tasks on data across a
cluster of computers.  
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

While our executors, for the most part, will always be running Spark code. The driver can be “driven” from a
number of different languages through **Spark’s Language APIs**.  
There is a **SparkSession** available to the user, the SparkSession will be the entrance point to running Spark code. When using Spark from a Python or R, the user never writes explicit JVM instructions, but instead writes Python and R code that Spark will translate into code that Spark can then run on the executor JVMs.  

**Two types of APIs:**
- Structured APIs (High level)
- Unstructured APIs (low level)
### DAG (Directed Acyclic Graph) Scheduler:  
is a fundamental concept that plays a crucial role in the Spark execution model. The DAG is “directed” because the operations are executed in a specific order, and “acyclic” because there are no loops or cycles in the execution plan. This means that each stage depends on the completion of the previous stage, and each task within a stage can run independently of the other.  
It provids the logical and physical plan of executing the job.  

### Spark Job:
A job in Spark refers to a sequence of transformations on data. Whenever an action like count(), first(), collect(), and save() is called on RDD (Resilient Distributed Datasets), a job is created. A job could be thought of as the total work that your Spark application needs to perform, broken down into a series of steps.  
Consider a scenario where you’re executing a Spark program, and you call the action count() to get the number of elements. This will create a Spark job. If further in your program, you call collect(), another job will be created. So, a Spark application could have multiple jobs, depending upon the number of actions.  

### Spark Stage:

A stage in Spark represents a sequence of transformations that can be executed in a single pass, i.e., without any shuffling of data. When a job is divided, it is split into stages. Each stage comprises tasks, and all the tasks within a stage perform the same computation.  
The boundary between two stages is drawn when transformations cause data shuffling across partitions. Transformations in Spark are categorized into two types:  
**narrow and wide**.  
Narrow transformations, like **map()**, **filter()**, and **union()**, can be done within a single partition. But for wide transformations like **groupByKey()**, **reduceByKey()**, or **join()**, **data from all partitions may need to be combined**, thus necessitating **shuffling** and marking the start of a new stage.

### Spark Task:

A task in Spark is the smallest unit of work that can be scheduled. Each stage is divided into tasks. A task is a unit of execution that runs on a single machine. 
 When a stage comprises transformations on an RDD, those transformations are packaged into a task to be executed on a single executor.  
For example, if you have a Spark job that is divided into two stages and you’re running it on a cluster with two executors, each stage could be divided into two tasks. Each executor would then run a task in parallel, performing the transformations defined in that task on its subset of the data.  

More on :  
- https://medium.com/@diehardankush/what-are-job-stage-and-task-in-apache-spark-2fc0d326c15f)

### Data Partitioning:

More On:  
- https://medium.com/@dipayandev/everything-you-need-to-understand-data-partitioning-in-spark-487d4be63b9c

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
- Serverless SQL warehouses: On-demand elastic compute used to run SQL commands on data objects in the SQL editor or interactive notebooks. We can create SQL warehouses using the UI, CLI, or REST API.
- Classic SQL warehouses: Used to run SQL commands on data objects in the SQL editor or interactive notebooks. We can create SQL warehouses using the UI, CLI, or REST API.

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


