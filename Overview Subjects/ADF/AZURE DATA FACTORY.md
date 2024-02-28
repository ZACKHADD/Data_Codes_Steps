# AZURE Data Factory Overview
##### The current file gives an overview of AZURE Data Factory to integrate and transform data in cloud solutions.
Similarly to any ETL tool, ADF shares the same paradigm. Several tasks are tasks in terms of ingestion and transformation can be done.  
Some tasks are not executable in stand alone mode and need to be inside a pipeline to be executed.  
##### 1. ADF Main Functionnalities :
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7aade850-1a54-4bf4-93e7-7cd294c6ada0) 
##### 2. ADF Key components :
- **Pipelines** : it is a logical grouping of activities that performs a unit of work. Together, the activities in a pipeline perform a task.The activities in a pipeline can be chained together to operate sequentially, or they can operate independently in parallel.
- **Activities** : Activities represent a processing step in a pipeline. For example, we might use a copy activity to copy data from one data store to another data store. Data Factory supports three types of activities: data movement activities, data transformation activities, and control activities.
- **Datasets** : represent data structures within the data stores, which simply point to or reference the data we want to use in activities as inputs or outputs.
- **Linked services** : Linked services are much like connection strings, which define the connection information that's needed for Data Factory to connect to external resources.
- **Data Flows** : Create and manage graphs of data transformation logic that we can use to transform any-sized data. We can build-up a reusable library of data transformation routines and execute those processes in a scaled-out manner from our ADF pipelines.
- **Integration Runtimes** : provides the compute environment where the activity either runs on or gets dispatched from. This way, the activity can be performed in the region closest possible to the target data store or compute service in the most performant way while meeting security and compliance needs. There are 3 types of IR :
    - Azure : for cloud ressources managed fully by Azure.
    - Self-hosted : used when dealing with on-promise data or private network (cloud but maitained in a private way).
    - Azure-SSIS : when wanting to excute on promise SSIS packages in the cloud.
- Triggers : represent the unit of processing that determines when a pipeline execution needs to be kicked off. There are different types of triggers for different types of events.
- **Parameters** : key-value pairs of read-only configuration.  Parameters are defined in the pipeline. The arguments for the defined parameters are passed during execution from the run context that was created by a trigger or a pipeline that was executed manually. Activities within the pipeline consume the parameter values.
- **Control flow** : it is an orchestration of pipeline activities that includes chaining activities in a sequence, branching, defining parameters at the pipeline level, and passing arguments while invoking the pipeline on-demand or from a trigger. It also includes custom-state passing and looping containers, that is, For-each iterators.
- **Variables** : they can be used inside of pipelines to store temporary values and can also be used in conjunction with parameters to enable passing values between pipelines, data flows, and other activities.
