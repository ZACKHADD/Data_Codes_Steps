# AZURE Data Factory Overview
##### The current file gives an overview of AZURE Data Factory to integrate and transform data in cloud solutions.
Similarly to any ETL tool, ADF shares the same paradigm. Several tasks in terms of ingestion and transformation can be done.    

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7cea7281-756f-4476-9919-18f76abf0ffe)  

Some tasks are not executable in stand alone mode and need to be inside a pipeline to be executed. 
**However, ADF is not suitable, alone, for complex data transformations**  
##### Typical Solution architecture :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1a5b368d-5dc8-40cd-9115-b0b12f22d77d)

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

##### 3. Simple Data Copy activity :
A copy activity, is abasic task making it possible to copy data from a source to a destination (sink in ADF language). 
Several components needed for this operation (like SSIS, talend ..) :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/36121bf1-4c66-48d3-bf5b-8ee2a52169c2)  

- Create linked services to the source and to the sink (under the manage data factory tab) : to point simply on the source storage and the destination one.
- Create data sets for the source and the sink : specify exacltly the files, tables etc of the source and destination.
- Create a pipeline to host the activity and all the other components since it is not executable in a stand alone mode.
- Create the stand alone copy activityinside the pipeline.
  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7c1d257c-9f7f-4c2a-ad71-1f4986ed619d)  

As the photo shows we have the first pannel on the left to handle our data factory with four icons for home, edit, monitore and manage.
Under manage tab, we have two pannels : Factory ressources (Datasets, Piplines, Dataflows ..) and the second one gives the possibilities for each factory ressource we are on, for instance activities we can add if we are working on a pipeline.
**In real world scenario** we should not keep the pipeline running all the time as the data in the source may not be available all the time. ==> that's why we add **validation activity** before the copy one as follows : 

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2abe2361-c9af-48a6-bd6c-38246fc66fb8) 
This activity gives the possibility to specify the dataset to validate, the timeout, the sleep and the minimum size. all these parameters will condition the sucess of the validation activity that will tigger the copy one otherwise the copy activity will not be executed.  
Another scenario, being the change in the content of our source. if we expect that the source should respect a certain structure but this latter changed, we should add some conditions to execute the activity only if the content is as expected.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d8f7b1fc-0969-4e63-a3f3-3b565fe4e3b0)  
We can for example verify if the file has the number of column expected by using **Get File Metadate** and the ouptput will be evaluated using **If Condition** activity. we specify the condition expression and under **True** case we put the **Copy activity** while under the **False** case we put **Fail Activity** that throws an error.  

