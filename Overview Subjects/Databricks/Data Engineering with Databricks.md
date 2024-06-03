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


