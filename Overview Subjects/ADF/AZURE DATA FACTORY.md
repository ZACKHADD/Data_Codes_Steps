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

##### 8. HDInsight Activity:

HDinsight is a tool that performs transformations (more complexed ones) just like data flows and gives access to several big data services such as Spark, Kafka, HDbase, Hadoop etc. One thing to bare in mind is that HDinsign datasources (datasets) requiers to write/read data into folders (not directly a specific file). in terms of read a hive engine builds the schema on folder to cover all the partitions of a file.  

First of all, in data factory, a HDinsight activity should be added but before that it requiers creating an HDinsight cluster. **Bare in mind that AZURE charges the HDinsight cluster ever when it is not used, so we should delete it once done.**  

**HDinsignt does not have a direct access to AZURE ressources (like ADLS gen2), so we need to create a managed identity (it is a ressource, like a key)** that will access the AZURE ressource under the IAM section of that ressource (with a specific role : data owner for example) and then assign that managed identity to HDinsight. **The AZURE ressource and the HDinsight must be in the same region to attach them together**   

While creating the HDinsight ressource we are asked to choose whitch cluster to create depending on what we need to performe :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9abe7383-828f-41bc-9a66-5fda18bd4109)  

To interact with HDinsignt cluster, we use AMBARI which is a tool to orchestrate the cluser:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6ebe1d45-c55c-40c3-89b5-c1b9b33ba7d6)  

If we want to query data with schema we can use HIVE.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/730cb829-3789-46fd-afbe-3948276ad688)  

However the interface is not that user friendly. so we can use other tools (that use GDBC connector) such as : **Squirrel or dbvisualizer**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/54d7eeed-f937-4ccc-86da-7015dff2c092)  

all the transformations in HDinsight are done using scripts. Like in this case we use a HIVE script to read data, transform it and then store it. The HDinsight activity then is executed inside ADF pipeline.  

To do so, we create a pipeline and we drag the HDinsight activity we want, in this example HIVE:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e0a72c24-2f02-4f5b-a02f-4a20ae0a2672)  

under HDI Cluster window we create or choose an already created HDI linked service. When creating HDI linked service we can either ask it to bring our HDI we created already (that we manage and we should delete once the transformations are done to not get charged) or to create an on-demand HDI cluster that get destroyed once the transformations are done which is the cost efficient way.  

Under the Script tab, we need to specify where the script is so that the activity can read transformation from it.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a4a65781-32f4-4dad-b81f-525bd4503f4f)  

**Remember to delete the HDI cluster after the end of transformations if we have our own HDI cluster because AZURE charges that even when it's not on run.**  

##### 9. Databricks Activity:

Just like in HDI, we can transform data using Databricks. we need to create an AZURE DATABRICKS ressource which is a Databricks Workspace (every thing is contained in Databricks and linked to AZURE).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9d282e9f-a468-4818-8adf-b3788b08c0ba)  

When we click on our new databricks ressource, it brings us to the Databricks home page : 

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7e01abc3-6f33-497e-a875-55af88bf9e9c)  

We start by creating the cluster we need to do computations :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e0851284-42bb-4b7e-a29c-5870ab335e56)  

Thre are two types of clusters :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/02da0607-5e08-4165-b492-61476d82dd8b)  

###### All purpose / interactive Cluster :  
Used to analyse data and interact with it using notebooks and gives the possibility to collaborate with other team members on the same notebooks. These clusters are manually created, terminated and restart.  

###### Job Cluster :  
Automatically created by Databricks job schedular when we run a job and they are automatically destroyed when the job is done.

###### Create Interactive Cluster:  

in the Databricks envirement, we can go to clusters and create an interactive cluster with all the characteristics we want. Note that it is suitable to select the runtime (scala and spark) that has long term support (LTS).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/49726cf0-f02b-4483-a04e-e65cee806928)  

We can also specify time of inactivity after which we terminate the cluster so we can optimize in terms of expenses. We can also after the creation, clic on the cluster and modify, restart or terminate it.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/069314a3-caab-44a5-84b9-b65ed9d2b24f)  

###### Mount Azure Data Lake Storage containers in databricks : 

Since Databricks is another provider and it's not a purely AZURE ressource, we will need to create a service principal that will have access to the ADLS and then we attach this service to Databricks workspace.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ea064521-9858-4927-9d63-5545346bdbe7)  

This gives Application ID, tenant ID and the secret.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e9897964-4000-412c-b75f-ecddff5a1859)  

Once this is done, we go to the ADLS and we create a new access role that (Contributor) that we grant to the service principal.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/77ceee33-9b77-402e-8146-193eeaf30118)  

Now, from Databricks we can use a script (Python) that is going to do  access (**mount**) the ADLS using the configs (Application ID, tenant ID and the secret).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e30e92c1-ec40-4d88-95d2-124a2187661c)  

**Note that the best secured way is to put the secret in key vault and use the key vault secret instead.**  

Once all the containers are mounted, we can access their contents :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d6ac1b09-c013-43be-a658-52348b265d50)  

After this we can delete the cluster we used for the mounting since we are going to use the trigger in Azure to run databricks that will create a job cluster to be deleted once the transformations are done.  
All the transformations to do will be in python script (notebook) that will be stored in databricks workspace.  After that we can create at Azure level a Databricks activity that will run the transformations notebook in Databricks.  

###### Create the pipeline for Databricks activity : 

First of all and as always we create a linked service for our databricks workspace. for this we will need an access token that we can generate from our Databricks workspace under user settings.  
Under cluster type, we choose new job cluster to optimize the cost since it is destroyed once transformations finished. **All the details regarding the cluster to be created are specified at the linked service creation level**.  

Inside the pipeline, we create the Databricks activity we want, in our case a Notebook.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9ee5123e-516d-4598-b106-31f8d7a7af3e)  

We specify the path to the notebook to run (our linked service gives us the possibility to connect to the workspace Databricks folders). After the tranformation is done we can access the job cluster to how it was doing the running, but we won't be ableto restart it.  

##### 10. Copy Data To SQL Activity:

After transforming all the data, we can copy it from ADLS to a SQL database. Dataflows allows to do that directly while creating the files in ADLS (byadding copy activity to SQL) but HDI not.  
One thing to bare in mind when we copy data from ADLS to SQL db, if the data are in several files inside the same folder (folder written data) we should wildcard file path and specify * so it can load all the files in that folder.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/65bd7c3b-00a4-4756-b361-9be1145c5ef9)  

In the sink settings we can specify a stored procedure if we have one or we can use a pre-copy script to execute before starting the copy activity. There is also the possibility to let the activity auto create the table if we don't already have it but the risk is that most of the time the columns are **strings and varchar types.** And we can also do a mapping in case the source columns are different (or with no headers) from the sink one.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f9875d19-0153-472c-b645-ae76a2681e47)  


##### 11. Data Orchestration:

Data Orchestration is needed to set the order of execution of the activities inside a pipeline but also between pipelines. Some of the orchestration requirements can be:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/fde28241-144b-441f-9579-df43c8398eed)  

On the other hand, the capabilities of ADF to achieve this are :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/56f724c7-4cb0-44ce-a6a7-bf994d466929)  

We can group pipelines (Datasets and Data flows also) by subject :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/185aab65-3120-48fc-8b64-ceda1a1b6a80)  

###### Pipelines dependency : 

in order to execute pipelines in a precise order we can create a **Parent pipeline** and create inside it an Execute pipeline activity :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/40ee6741-192c-4124-9d7b-f142e1efabb2)  

After that we create a trigger for the parent pipeline that will invoke the first execute pipeline activity in the pipeline. Note that if we had already a trigger for the first pipeline we should delete it. Unfortunatly ADF does not allow renaming triggers so we should delete the first and create another one.  

###### Triggers dependency : 

The dependency between triggers makes it possible to invoke a trigger after that another one was invoked. **Note that this is possible only with Thumbling Window Triggers.**  

##### 12. Monitoring :

The monitoring capability of ADF gives the possibility to follow up all the components in our Data Factory and we create and that they are running as required.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/51f7834f-4a98-40c6-9f70-988d3a165c5c)  

**The monitoring can be done using ADF Monitoring section or also AZURE Monitoring**.  

- ADF Monitor :

  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d7b9d35d-25a7-430b-8582-cfb232e0232b)

- AZURE Monitor :

  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4f1999e4-ed93-4c26-b563-c057a14c3ceb)

###### ADF Monitor : 
At the ADF Monitor, we have a dashboard so we can see the components executed, the one scceeded and those who failed so we can rerun them. We can also see the history of triggers, pipelines, integration runtimes and data flows up to 45 days.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5edf4d9a-1f7f-4d12-a363-14fda4e780ec)  

We can also create Alerts so in case of a fail we get notified.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/20d286d6-5615-46cc-9699-59bc581dd0d1)  

We can specify for example to get alerted when a failure accurs, and set up the criteria that condition the alert (check for failure every 1 hour and give failures of the last houre ...)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/45a0a818-7db5-4039-9ce5-e2125a4c4a93)  

We can then set up the notification type : SMS, email ... Once created, we can enable and disable the alert as we wish.  
**In case of a failure in a pipeline, we can either rerun the whole pipeline or only the failed activities. Note however that reruning the activity is concidered as manual so the trigger will still show failure (this is a quite a bug in ADF),we should rerun the trigger again. This causes a problem when we have triggers dependencies because the next one won't get invoked unless the previous has suceeded**  

If we need more metrics to follow, we can use AZURE monitor and specify that we want to see the ADF we created.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/589eba36-53a2-4b70-bbcc-5053771bfa2d)  

We can then create charts and pin them to the dashboard.

###### AZURE Monitor : 
We can monitor our ADF using AZURE monitor.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/70f57ac9-d635-4a1f-a190-6ddc00366352)  

Once we choose the ADF we can add some diagnostic settings:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e8b60c40-724d-4def-8dea-72d5a71acb3b)  

This gets the logs we specify and the metrics we choose. We can send these logs and metrics to Logs analytics, storage account or event hub, and also specify the retantion policy (how many days to keep these logs, 0 means forever)  

We can also use **Log Analytics** to store the logs and metrics and query the database. We can specify that all the logs and metrics get stored in a single table **azure diagnostics** or inseperate tables **resource specific**.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e2adaed0-ab39-43e4-b2cf-5fc4bb465bb2)  

In the logs analytics workspace, we cam query the logs using **KUSTO query language.** We can also create visuals and pin them to the azure project dashboard.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/289d1ba6-baca-45c8-b908-a9a222bdf2cd)  


##### 13. CI/CD :
In this section, we get to dive in the process of CI/CD (one of the Devops practices) in ADF. The overall process is as follows:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/872c017b-d679-4e98-ba83-4ec33e2df35c)  

Traditionally, companies seperated the Development and Operations teams. The dev team completed the development **(writing code and testing it)** first then the Operations (Deployment and code maintenance at run) team makes the release to production ==> this generated a lot of problems : Lack of coordination, delay in reaching the market, toomuch bugs etc.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e3aad20d-ef36-40b6-9f09-b45efd39b76d)  

To solve these issues, companies started bringing the two teams together and the intersection between the two is called Devops. This needs :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b965faaf-d48d-4ee8-9b88-edf64c441564)

Generally in an IT project we have several steps where we clearly can see the role of CI and CD. In an ADF project we have the same logic.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/38e44885-1950-42d4-a953-a52236512c22)  

The coding files are json files (maintained using GIT) and the build files in ADF are called ARM files (Azure Ressource Management).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/effb47c9-f801-4260-8567-ee21a2859146)

Test automation in ADF is difficult so most of the tests are manual.  

The build can be done following 3 ways :  

- ADF Publish Button (still used) : Using the ADF Publish button to deploy.
- Automated deployment (the most used in large projects) : Using GIT and build files.
- ADF third party tools.

###### ADF Publish Button : 

Developing in ADF directly (live mode connection to ADF production repository) is suitable for small projects and when we don't have more than one developer.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1448ab2e-8009-4d4b-a88d-20a5088c0f60)  

For large projects with more than one developer we need to have code version control suchas GIT.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d33ea030-8552-4eb0-9c3e-973125070bc6)  

The publish here is still a manual process but the CD one is automated.  

###### AZURE DEVOPS :

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/74a95396-021f-4bf5-be82-bda4df267b66)  

Five main services are provided by Azure Devops :  
- Boards: supports all the tools of project management and agile methodology to follow the project.
- Repos: versioning tools such as GIT.
- Pipelines: which handel the build and release phases for automated CI/CD.
- Test plans: Browse based test management testing to set up tests and so on.
- Artifact: a library to store packages and developed artifacts that can be shared or used inside a CI/CD pipeline for example.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/786d5eb7-b485-44d0-b795-02d2a4de1518)  

The structure of Azure Devops, figure bellow, has two levels : the organization (container of several projects) and the project.  
An organization is simply a department, the whole company or a business unit etc. This means we can have several organizations. Inside of the organization we can create multiple projects. We can also create teams that can work on several projects.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/fd7fb4c3-693c-40dc-affd-2d3b4d7f0ee6)  

To use azure devops we should sign in to dev.azure.com with the same account we have under AZURE. Then we should **specify the Azure active directory** (we can find it bellow the profile picture in azure portal) we are working on (that will contain the organization) which needs to be the same as the directory holding our azure ressources.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/67b1208c-f16b-4c39-b56a-ccc88f5e2087)  

This step creates the organization inside which we are going to create our projects. Under the organization settings we can set all the options and billings strategy we want.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7d34bbe5-a452-4c6d-90f5-50fc300cd596)  

We can specify also which version control to use GIT or Team version control and also which Project management process to use Agile, Basic, Scrum etc. this will condition the boards part of the project. Once created we can start working on our project.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5f49d92a-c330-42f5-9053-1e5ee06a469f)  

If we take ADF as an example, generaly we set 3 envirements (meaning 3 ADF ressources) for Dev, test and production. The test part can have two or more ressources like for integration tests and user acceptance tests. **GIT is enabled only for Dev envirement.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/91b8a05e-a344-454e-a153-e19c5847129b)  

One we create all the ADF ressources of the envirements we need, we can create a GIT repository in AZURE Devops (It can also be done from ADF Dev ressource directly) that we will link to the Dev envirement. This can be done under the Repos rubric.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/cce50ab3-6444-47f7-a972-b90600e25586)  

###### CI :

The default bransh is **main** (master if we create the git from ADF) and since this bransh is the one that get published to the build and release it should not accept any direct coding and we should set a bransh policy that will prevent direct coding and accept only merges after a pull request.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d02a7468-8f3a-4332-a1af-5e4ddab244de)

A lot of choices are available, but the main thing is to require a minimum number of reviewers before the code changes can be accepted.  

We can then in ADF dev ressource, configure the GIT repository (**here we use Azure Devops GIT but we can use Github also**): 

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5e14f14a-2e8a-4e74-b562-6dc715ec29ae)  

**Note that the Publish bransh is the one that will contain the ARM files for the build and release, while the collaborative bransh is the one holding the code that we collaborate on using pull requests**.  
 Now our ADF project is connected to our Azure Devops and we can see on the top of the ADF screen the Bransh (we click on it to create a new bransh or a pull request) of GIT we are working on. If we still don't have any feature branshs by default we will have the main one, but we can't develop directly in this bransh since we set a policy on it. We should create feature branshs to make our developments and then merge to the main one via a pull request.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d6930af1-95d9-40e8-9346-1480f14a9770)  

**Note that we can organise the branshs using folders by simply adding /.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8b0339a2-df5c-4260-b784-0a844112f7da)  

**Note that beafore linking ADF to GIT we were saving changes directly by using the Publish button, while nowwe can use Save or Save All since the publish button is only available in the main bransh which will create the ARM files for the build and release.**  

Once we save the pipeline for example in ADF dev, a json file is generated (or updated if already exist) containing all the details of the objects in Azure Devops. It is basicaly a commit command behind the scene.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2cbd49c4-79cb-4a3f-b573-c544e9d240ce)  

We can create a pull request from the azure devops or directly from the ADF studio. Both options give a form to validate the creation of the pull request in Azure Devops. The form specify the name and description of the pull request and the reviewers that must validate the code.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/fe586204-cbbe-468a-9e9a-e2daffaf58b4)  

If we were assigned to be reviewers we can see the code changes under Files(Old Code and New Code) tab as follows :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b87b6002-d90b-4eb4-b260-67e52f8702fb)  


Once the changes are validated we can merge the feature bransh with the main one by clicking on Complete pull request.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5cd1028a-4d4e-4ba6-82e7-56a50f9e6ba6)  

We have several options such as the type of merge, complete the associated work (if we assigned a task in the board pane) and also if we want to delete the feature bransh once the merge is done. Now we can see the pipeline in the main bransh.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/cb1a725a-6a08-4a8a-86e3-380d6cee22ad)  

One it is ok for us we can publish the changes to the live Mode, and this will create a new Bransh called **Publish_Bransh** that will contain the ARM files for Build.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7186857c-3941-48d9-b29c-ade8a519ab92)
  
We can see that the ARM files are in a folder and we have two files inside one for the template factory and the other for parameters. The role of parameter is simply to change the name of the ADF ressource we want to deploy the files to. for example change the [factoryName] of the ressource from dev to test and we can build the project in it.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/411e4868-e678-4d49-9e12-e72e151336cb)  

###### CD :

After generating the build files (ARM in our case), we can start the release phase. The general process of release can be automated (Netflix does that) but most of the companies use a manual one by adding an Approve step before releasing to production.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/78f0da5d-afdf-402e-8152-8d9b9ab400c0)  

For ADF, the deployment to the test and prod envirements is done via **ARM deployment**. Unfortunatly, this tool has many limitations such not being able to delete an artifact (pipeline for example) and it doesn't allow the update of triggers in run. So microsoft team provided two powershell scripts to handl these limitations : Pre Deployment and Post Deployment Scripts (for the triggers, it simply stops thems and rerun them once the update is done).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/760d6a5a-311d-444f-800a-851df9db72a2)  

These deployment steps are grouped in a stage and can be reused to deploy in the production envirement. The only difference is that the production deployment needs an approval from the user and waits for the test to complete.  

To build the release pipeline we do that in Azure Devops:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/726ff27f-bf71-4afb-ad5f-091f796ceb82)  

This gives the possibility to create a release pipeline either from scratch (empty pipeline) or using a template.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b4dc42f0-234b-480b-a0fc-47790af6b12b)  

In the empty job, we have to specify two components, the artifacts and the stages (test and prod). Artifacts are simply the build files of the project we created (in this case ARM files). In the artifacts we select the git repo option since the build we have is manual,and we select the git repo and the publish bransh containing the ARM files.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7c02d42e-3ac9-42ef-825b-d23f285a8f43)  

Inside the stages we can create tasks. In our case the ARM template deployment and the pre and post deployment scripts.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1872d085-2447-4f91-a4f1-9675ccfd5316)  


Once created, we can set the ARM to connect to our ressource, since it will acces it. we don't want to authorize the ARM to access all the subscription so we click on advance to choose the specific ressource to deploy to (In our case it's the test ADF ressource). This will simply create a service principle for ARM with a contributor role.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b643caed-b1fd-4226-b00b-7520d661b623)  

At the Azure IAM ressource level this role looks as follows :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b38dc501-bdb2-45c6-858a-266aa6d32e98)  

After that we specify several information such as the action to perform (create and update or delete), the template and the template parameters which are the ARM files. Also we have override parameters section where we can change the name of the ressource destination of our release : in our example replace dev ressource with test.  

We can also, instead of hard coding the options for the release pipeline, use variables so that we can reuse the same pipeline for several ressources (test, test integration , Prod ..).  

**Note that the deployment mode has 3 values and that we need to specify the incremental one which update the ressource group with the ARM files. The complete mode however deletes all the ressources and build the ressource group using the ARM files and we don't want to do that !!!**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9e3e032b-f57e-4bb3-97bb-977ca87ab0d7)  

Now we can trigger the release pipeline using the trigger button on the artifact part and enable the trigger and also filter on the publish bransh to include in the release and we save the whole pipeline. you will notice under the enabled toggle it states that every time a GIT push occurs in the selected repository, a realease is automaticaly created.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a2c19154-fdc7-403e-8432-b56d980e1046)  

We can also create the releases manually using the **Create Release** button.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f82bbf71-5974-4f9e-8192-c0f3a716d2aa)  

We can also see the details and logs of the release created.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/cd89e0dd-057d-451c-9cb7-f4bcd631bcc6)  

Now we have the continous deployment built.  

###### Pre an Post deployment tasks :

Now we need to rectify our pipeline since as we said before the ARM deployment does not support : Deleting objects (Does not generate an error in logs deployment but no deleting happens in the destination ressource) or Updating active triggers (does generate an error in the logs and the release fails).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a2b370f0-5ead-4d3d-b413-7c488eeedaef)  

**Link to the scripts : https://github.com/Azure/Azure-DataFactory/tree/main/SamplesV2/ContinuousIntegrationAndDelivery**  

There are two versions, 1 and 2. the second is more efficient since it only deactivate the triggers to be updated and not all the triggers.  
We should download the file and upload it to the Devops GIT repository so we can use it in the release pipeline. For that we create a feature bransh (since we can't modify directlyin the main bransh only after a pull request) in devops and we upload the file and make a pull request.  

Now we add the scripts to the release pipeline artifacts by clicking on edit.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/053164a8-3bc1-4537-87d6-7dd59bd3a4c3)  

Note that the two artifacts we have are from different sources, the first is Publish bransh since it's ARM files getting updated each time, but the scripts are in the main bransh since it does not change and it's not part of the publish process in ADF. No trigger to set the second artifact since we don't have the release to be triggered when w change occurs in the main bransh but only in the publish bransh.  
To use the scripts we need to add an **Azure Powershell Task** in the stage area.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b40bdf51-e683-4d92-8322-770907d5f4bf)  

Some settings need to be done such as using inline script of a file path (which the case here).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/97c48eee-a443-4517-9a81-0718f1037b0b)  

**We need also to specify the Script arguments that we can copy from : https://learn.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery-sample-script**,and we need to replace the values with the parameters we have in our pipeline (we can also use variables).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/993d1664-ab10-4ef8-b2b9-0a9322876956)  

Now that the Pre deployment is ready, we need to drag it up before the ARM deployment.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4998fede-6d3a-4ba1-bded-27665e3c7447)  

To create the Post deployment task we can right click on the pre depolyment task and clone it, place it after the ARM deployment and make the changes in the settings including the Script Arguments. Then we go back to the release pipeline view and save the work.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4988dd05-4fea-4847-a5af-ca410fb5f48a)  


Now if we change the ADF project, commit (via a pull request) the changes to the main bransh, publish the changes to the build phase we are going to see the release triggered as it should with no errors if am active trigger was modified and if an object was deleted, we can see it in the test envirement.  

###### Variables :

To extend the pipeline we created to the production stage without creating a new one just by reusing the test one, we need to create variables to replace the hard coded parameters such as the data factory name and so on. To do so, under the variables tab in the release pipeline section in Azure Devops we add variables:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2a07f671-c70d-4482-82e3-500da014e5d1)  

Note that we need to specify the scopes of the variables to the right stage, since the values are only applicable to the Test scope (the two stages prod and test can have different locations for example). After that we change the Pre and Post scripts and the ARM to replace the hard coded values with the variables using the syntax : **$(variablename)**

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/aefd7058-90e3-4404-9fc8-1217f824ec48)  

Now we can create a production stage, by cloning the test one and changing the variables. The only difference between the two stages is that the production will need to wait for an approval before being triggered.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/466b7605-d5cb-41c1-bc9b-73144a42b801)  

The clone creates a new stage with the same tasks.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7c5c0619-5d07-452e-ac43-cd6498e789e2)  

Then we can changes the variables inside the tasks to the production ones (The variables are created automaticaly when cloning the stage).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9ae7fdbe-86c4-420b-8289-8d869c2234da)  

We should just change the values of the variables newly created after the cloning.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/df116489-af15-4e5b-85ba-af405e647457)  

**One important thing needs to be done since we cloned the Stage, is to give the ARM connection (the principal service) access as contributor role to the production ADF ressource so it can build the release in it just like in the test one.** We do that at the ADF studio under IAM of the Production ressource.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/fcdd580e-0c9f-4d3c-a9da-6d6d6fcbf6c3)  

**We can find the name of the service principale by clicking on the manage button next to azure ressource manager connection**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9bceef3e-5958-4464-80d3-52c1ce476289)  

In the ressource Group (or ressource depending on what we created) under IAM section we add new role assignement and we search for the service principal we want.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/767f5248-7a13-40b7-8529-136ee2707d52)  

One thing left to do is to add an approval step before triggering the production stage.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/08984b3e-9180-49ba-b9a6-2505421153d5)  

We select the persons to approve the triggering and also a timeout after which if it is not approved the production stage will not be triggered.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2591e06f-effb-4c50-b6b5-117498c665a8)  

Save the work and add comments if we like. If some changes are commited, this is how the whole release pipeline will be:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5519f50e-c238-4fc1-8628-368dcf713a55)  

Note the the production stage states that we have a pending approval and who can approve.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ea39f80a-dd9a-4334-95e3-0b88f6b18033)  

An email will be sent to the approvers so they can approve the production phase.  

 ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/fbeab586-62f8-44b5-93bb-4550f3fb6f54)

Once we approve the production is triggered.  

###### Automate the build phase (Replace the publish button) :

Till now, we used the publish button to do the the build phase, but we can also automate that using **npm package in ADF Utilities**.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e44c138b-2042-4566-8021-c40d1c3c2089)  

We can build a devops pipeline that will invoke the ADF utilities npm package and on succes the npm will create the ARM files inside the ARM artifacts in the devops pipeline.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/11599031-9653-4e7f-8895-1c048cca385b)  

This means that after a pull request the whole pipeline gets triggered automaticaly and the only thing to do manually is the production approval.  

**The npm package and it's documentation is available here: https://learn.microsoft.com/en-us/azure/data-factory/continuous-integration-delivery-improvements**  

To create the pipeline in azure devops, either we use interface as usual or we can use **YAML configuration files (just like json)**. **The npm package should be a json file (simply containing the instruction to install the ADF utilities npm package) in the azure devops repository.**  

**the yaml configuration file needs to be modified to point on the dev ressource group using variable we can add inside it to replace the hard coded values inside the code. The YAML file is provided in the ADF folder**   

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b24b64f5-066a-4f8e-bf8d-a07b95c31962)  

Once the files are modified we need to upload them in Azure Devops main bransh, as always since the main bransh needs a pull request to be modified, we need to add a new feature bransh, add the files in it and then make a pull request to merge with the main branch.  
**Note that the folder containing the files should be named the same as the yaml configuration file.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/343883d9-2748-4cc7-b792-d0ac3776f448)  

Now we need to create the build pipeline to replace the Publish button. Note that beafore we were creating release pipelines under the release section, now we need a build pipeline that we can create under build section.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5fac978e-5eb0-48e9-9434-c83d87a52f2d)  

Now we specify where the build code is (YAML file):  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3b163898-bd91-4385-a900-ff9d0e9be08a)  

Note also in the bottom, we have the possibility to use the classic editor to create the build pipelines just like we did with the release one. Herewe use the configuration files instead.  

By clicking on the GIT repository option, we have possibility to use prebuilt Node.js (since pipeline in Devops are Node.js projects) templates (with angular, react, Vue ...), start a new one from scratch or use the YAML file and that is our option here.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3c31abb6-6c7c-4da7-b442-66dc95da0b45)  

We need then to specify the bransh and path for our YAML file.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/02f87729-cab4-4283-8692-e5549429412e)  

We can review the code and save it (by clicking on the arrow next to the run button and choosing save).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/506544f6-ebf4-4417-a841-bf7362c4a760)  

Once saved, we can see the pipeline. we can rename it and so on.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8baa9e7e-ef69-404a-ab7d-bc5a05a381dc)  

To test it, we can either run it manually or make some changes in the ADF and commit these changes to the main bransh and the build pipeline get released automaticaly (that is beacause in the YAML file we specified the **trigger : -main** meaning upon every change in the main bransh). on run we can see the logs of each step.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/60265f43-43ca-453e-8b83-29ce8095da29)  

This publishs the artifacts that our realease pipeline can consume.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7742ac86-998d-44e6-9a51-8446b4989c9d)  















