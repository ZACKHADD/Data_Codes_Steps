# AZURE Data Factory Overview
##### The current file gives an overview of AZURE Data Factory to integrate and transform data in cloud solutions.
Similarly to any ETL tool, ADF shares the same paradigm. Several tasks in terms of ingestion and transformation can be done.    

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7cea7281-756f-4476-9919-18f76abf0ffe)  

Some tasks are not executable in stand alone mode and need to be inside a pipeline to be executed. 
**However, ADF is not suitable, alone, for complex data transformations**  
##### Typical Solution architecture :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1a5b368d-5dc8-40cd-9115-b0b12f22d77d)

Two main tools to use with ADF : 
- Azure Storage Explorer : to visualize more easily all the storages we have on azure.
- Azure Data Studio : like SSMS makes it possible to write and run queries on SQL databases, for example, in azure.  
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

##### 4. Delete activity :
In some scenarios we are required to delete for example the file in the source after that the copy activity is done.  
**we should note that till now ADF has not the move activity that's why it is handy to use the delete activity after the copy one**  
In our last scenario we simply drag the delete activity after the copy one inside the True clause in IF condition activity.  

##### 5. Triggers :
ADF gives the possibility to run pipelines when a trigger is activated.  
3 main types of triggers are available :  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3ed3c182-2957-464e-b3d6-9a5b0a13b7ac)  

- Schedule trigger : runs on calendar/clock and we can have many pipelines attached to the same trigger and vice versa.
- Tumbling window trigger : gives an alternative of schedule trigger if we want to execute a pipeline that has failed earlier for the data created in that periode of time in the past. the relation with pipelines is one to one.
- Event trigger : runs pipelines in response to an event (creation or deletion of a blob that contains a file). Many to many relation with pipelines.
Envent triggers are created under the Managed section (same as linked services).

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5f0e40c5-c3da-4391-8454-f13e0f769130)  

**Note that the ressource provider of triggers should be registered in the subscription so that the trigger can work**
  Once done, we go to the pipeline **we want to attach the trigger** to, and under the **add trigger** button we choose the trigger we created and we pulish all to save the ADF project.
  Under the monitor section, we can see all the componnents on run including triggers.

##### 6. Parameters and Variables :

Parameters are used, like in any coding exercice, to reuse the same code over different specified values. The same can be done in ADF to reuse the same pipeline with different components as parameter (datasets, linked services ...).  
For example, imagine that we have a dozen of datasets that need to be copied following the same pipeline as below :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/94a81c68-29f4-40e6-8e0c-8446ca994d7c)  
by dupplicating the same pipeline for each dataset we will have an exessive amount of components, while we can simply create parameters that one pipeline will use the run for all the datasets we specify in those parameters.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3465f4e2-e2aa-43c7-adcb-954f896ed648)  
Variables have a similar approach but inside the pipeline to set values to be reused between activities.  

The following figure shows the section where we can parametrize a pipeline :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8a726320-1c32-471c-9378-0e16b510ef33)  
Now the pipeline is generic and can be reused for several datasources (files), and the values of the parameters can be specified at run time either **manually** or when a **trigger is launched.**Now what if we want to reuse the same pipeline for several parameters without creating too much triggers? that is where Control Flow activities come to the rescue.  
We can also parametrize the linked services by creating parameters inside them to be sat before starting the pipeline. this parameters will passe the values to the datasets source and sink parameters (should be created) which will passe these values to pipeline parameters also (should be created).  

##### 7. Control Flow Activities :
We can create a json file for example that has all the sources names we want the pipeline to iterate over and we can use the **Lookup** activity to read inside the file:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/38e5f8ac-158e-434f-a160-6b3fa9b0fa65)  

and then use the **Foreach** activity to loop over these values and copy data for example **(the copy data should be inside the Foreach activity    )**:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/24ad86c6-f9b8-44f0-acdf-e8f2b116b1af)  
the foreach activity can loop over the items sequentialy or in parallel. We can also add inside the foreach activity a **set variable** one so we can see the output being used inside the foreachactivity :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b9ed041b-0e92-4820-9fa4-26afc80124c8)  

### Data Transformation:

The data transformation can be done in ADF using several tools including DataFlows, HDinsights and Databricks.  

##### 8. Data Flows :  

Data flows are a type of activities we can use to transform data **(Not recomanded for very complexed transformation, for that we code transformations in spark notebook for example)**.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/0652a860-b33e-4e92-8f2e-ffb322e3a5e6)  

Two types of dataflows are available, one for the stable data with a schema and the other for data wrangling generaly for data science purposes.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9bd1625e-00d4-44d8-b57d-e3f4e01e3754)  

Data transformation debug in data flow requiers an integration runtime which is a spark one by default runs with 4 cores. We can however create an integration runtime that is suitable to our case.**Note that the spark engine for debugging is a paied service depending on the runinng duration**. It can be activated under data flow debug toggle button.  

###### Source Transformation :  
First thing needed to be created is source transformation. **we can either use the datasets already created at datafactory level as sources or create them inside the data flow transformation (what is called an INLINE) but they will not be reusable elsewhere**.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/07b9f136-03c0-4ebf-a9b8-417746e83ce1)  

Under source settings we can specify the data source and what to respect as schema for the transformation to succeed (is schema isn't the same we accepte or reject transformation). For debugging purpose we can enable sa;pling to preview the result or we can specify a sample file under debug settings.  
Under source options we can decide what to do with the file source once the transformation is done or which file to process depending on the last modified date.  
Under projection we specify the type of columns or detect data type automatically, or import a projection.  
Optimize makes it possible to modify the spark cluster configuration to execute the transformations (partitionning).  
Data preview will give us a preview of data based on the default sample or the sample file.  
###### Filter Transformation : 
This is basically a filter task that makes it possible to filter data using expressions.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/fdd4a69c-3116-4eea-8df0-0f8175e99f1c)  

when we clic on Filter on we get a window (visual expression builder) where we can write our expression (based on scala) that uses intellesense to makes it easy to autocomplete the code,it's a little similar to power query.also we can preview the result of the filter.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/534cc7bc-6dde-4db7-bc94-a68269223131)

###### Select Transformation : 
This is a transformation where we can choose the columns to keep and if we would like to change names of the columns. we can either use fixed mapping or rule based mapping.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/168791a5-b906-4d44-ad0e-299fa6eb89f3)

###### Derived column Transformation : 
This transformation makes it possible to update or modify existing columns or create new ones.

###### Aggregate Transformation : 
Performs an aggregation on data using a group by clause and it is also possible to do in it the same as we can do in derived column transformation.

###### Pivot Transformation : 
A little bit similar to power query, we create a pivot transformation by specifying the coloums to group data by and the remaining one will be pivoted.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/efc3a748-f815-4e67-827a-74034bc0ef8d)  

Then we specify the values based on which the new columns will be added (if we don't specify anything ADF will do that automatically using distinct values).   

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6963182b-4327-4089-9b94-fbe35103c5e8)  

Finally under Pivoted colums we must specify an expression that uses an agregation to be calculated inside the new pivoted columns.  

###### Lookup Transformation : 
This is a trnasformation that makes it possible to performe a lookup value (left outer join) between two datasets.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1b7396cf-3763-4962-a3d1-55696508f818)  

The lookup transformation retrieve all the column from the second dataset so to leave only the column wanted we should add another select transformation to remove the columns dupplicated or that we don't want keep. There is an option in Optimize that makes it possible to **broadcast** (save data in memory) data in the spark clusters so that the lookup (or joins) can be faster. We can either put it to auto and ADF decides what to broadcast or we can specify the fixed mode and choose what to broadcast.

###### Join Transformation :
Classic joins that we can perform on two datasources. Same possibility as lookup transformationif we want to broadcast.   

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/aa9d9bb8-a2db-4e95-a933-542077fba5dc)

###### Sort Transformation :
This transformation is suitable for sorting data.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/078d1db1-0c1c-41c5-9c0e-e234272711d6)  

###### Split Transformation : 
We can also split our streams if from the same file or source we want to create two files or sinks.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/762d5b04-0768-4a5c-ab03-2663e2f9ca30)


###### Sink Transformation : 
This is the last step of transformation that makes it possible to write into a source destination the result of all the transformations done. Only folders are specified as for the destination where to write the file for example.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d8a976c3-a8c7-4255-97b5-37782f02e633)  

Since the transformation run on spark cluster (distributed) generally the file created at the sink level is processed in several parts but we can specify under settings to create only one single file **(slow performance however)**. we can also set a mapping to choose what columns to keep for the writing part and wether to respect a schema (if wew set one at the sink level) or accept drifts if there are any.  

Once the data flow is done, it can't run in stand alone mode, it should be done inside a pipeline.  
Once the pipeline is created and debuged, we should put off the Debug flow debug to save money, and create a trigger for the pipeline. ADF then create the spark cluster behind 9depending on what we specified in the DF settings) and destroy it once it's done so we won't be charged when the DF is not used.  

##### 8. HDInsight :

HDinsight is a tool that performs transformations (more complexed ones) just like data flows and gives access to several big data services such as Spark, Kafka, HDbase, Hadoop etc. One thing to bare in mind is that HDinsign datasources (datasets) requiers to write/read data into folders (not directly a specific file). in terms of read a hive engine builds the schema on folder to cover all the partitions of a file.

