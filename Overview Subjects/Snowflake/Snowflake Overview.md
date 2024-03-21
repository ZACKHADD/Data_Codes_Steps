# Snowflake Overview (Workshop) :

Snowflake id Data Platform and data warehousing solution (Relational Databases) that enables data storage, processing, and analytic solutions that are faster, easier to use, and far more flexible than traditional offerings. It is a **Cloud datawarehouse with a distributed storage and processing technology**.  

## 1. UI Presentation :

Snowflake UI is very friendly andcoposed of a panel (left) containing several sections. When clicking on a section it opens at the right all the content.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2411e4c6-2602-472d-b3ff-e2c28b42fa75)  

- Projects: A section for coding projects like SQL queries and transformations, python, SPARK jobs ...
- Data: A section related to databases.
- Data Products: A section dedicated to marketplace and data service providers.
- Monitoring: A section where we can monitor all the objectsm jobs, queries etc.
- Admin: Administration section where we can follow up the cost of all the worloads, monitor the data warehouses, change roles etc.  

## 2. Identity and Access :

Identity and access are two different concepts.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5435b89c-b44c-4f1b-be2b-2d618b46f9ff)  

When we managed to prove who we are we get athenticated to use snoflake, but then the authorizer needs to check if we are athorized to do an action or access a specific item inside snowflake.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3eac8f90-e30a-4a14-8d10-edb596163b52)  

The authorizer uses Role Based Access Control (RBAC) to allow the user to perform an action based on his current role.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/67b14209-34ca-4bef-83a6-cecad6bae6fc)  
 
**Snowflake power in this area is that we can switch roles and stay connected with non need to log in again.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/42a6d1d3-789b-4d35-9ad2-d27b8fbaf253)  

The ADMINROLE is the most previleged role. Meaning that having this role gives the all the previleges of the other roles based on the principal of **RBAC Inheritance.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e1736e88-3d7d-4cda-af49-930e38b7ead5)  

Another role was recently added : **ORGADMIN**, very powerful role that can tie multiple Snowflake accounts together and even generate accounts.  
The **SYSADMIN** is widely used to create  databases and warehouses. **Note that it is always a good practice to troubleshoot by checking the role settings if we get some errors when performing an action.**  
In general, we have a default role that gets assigned to us every time we log in and we can switch it if we are allowed. We can also alter the default role if we like.  
**We can also create our own custom.**  
The logic of roles is that for example, to create a database we need to have a SYSADMIN role. However, if we already have an ACCOUNTADMIN role assigned we can switch to a SYSADMIN role to perform the database creation. **So there is a difference between the role assignement and the role impersonated to perform an action.** Note that if we have a SYSADMIN role assigned, we can't impersonate ACCOUNTADMIN one since the process is forward only.  

There also another process behind the scene that is combined to RBAC, which is DAC (Discretionary Access Control) that states : **you create the object, you own it!**. meaning that if SYSADMIN creates a database, they own it and so they can delete it, change the name, and more. **So the ownership of items belongs to roles.**  
Also,every item owned by a role is automaticaly owned by the role above,a principal that we call **custodial oversight**.  

## 3. Database creation :

As every classic SGBDs, **database** is the upper level of data items grouping.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/24747a4a-3dd0-4c3b-8a5a-cd2a21f181e3)  

 Next we find another grouping level which is **schema**: it is a grouping of several tables related to the same scope.  

 ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/668c3854-a7eb-4d05-869e-0a441f97761d)  

By default in snowflake we have **INFORMATION_SCHEMA** schemas containing views of METADATA regarding the all the objects of the database. It cannot be modified.  

We can transfer the ownership of the database to another ROLE if we desire :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6f4286e3-213d-41d0-a1a9-ae3be4c12851)  

**Note that the transfer of Database Ownership does not give access to it's schemas, this should be done also at the schema level.**  
**This can be done also by granting Access which is the best way.**  
The same thing can be done with Datawarhouses:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/725a5326-4ee5-4fcd-8a1e-7e0217290f31)  

## 4. Worksheets Creation :

The worksheet interface has four menus. Two are in the upper right corner and two are close to the first line of code.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/861a96d6-3463-466c-9fe5-6a492659d570)  

We have also the result area withsome metrics at the right.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5c031261-fc24-430a-be47-b2366d1c6505)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9114f357-0de2-403a-bc1e-19f30b34a358)  

We can start creating tables as follows :  

                                          use role sysadmin;
                                          create or replace table GARDEN_PLANTS.VEGGIES.ROOT_DEPTH (
                                             ROOT_DEPTH_ID number(1), 
                                             ROOT_DEPTH_CODE text(1), 
                                             ROOT_DEPTH_NAME text(7), 
                                             UNIT_OF_MEASURE text(2),
                                             RANGE_MIN number(2),
                                             RANGE_MAX number(2)
                                             );  
Just like SQL IDE softwares, we can view the code defining the table: 

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2140c0e1-cbc9-4220-94c0-4edb5fd32664)  

We can also alter tables :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/0b03561f-13b5-462d-ab6b-5a1abe70837a)  

### Insert Data :  

We can insert data in many ways such as :  
- Using an **INSERT** statement from the Worksheet. 
- Using the Load Data Wizard:  
  
  ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/36e22a82-7cef-4466-993a-f4bf00e748e0)  

- Using **COPY INTO** statements.

**INSERT** statement gives the possibility to add a row in a time. If we would like to insert multiple rows in a time, we use **insert into**.

## 5. Data Warehouse in Snowflake :

The concept of Data warehouse in Snowflake is different from the one we traditionally know as in Business Intelligence.  
In Snowflake a **Warehouse is simply a computing Workforce** that executes tasks on **seperately stored data**. Meaning that a **warehouse here does not store data**, it only perform tasks on data.  
**Snowflake Warehouses can be Scaled up and down by changing the number of servers working and also Scale out automatically by adding/removing clusters during a peak workload.**  
**Scale up: Increase in compute capacity by increasing the server strength keeping cluster count same. Scale out, handling high concurrency,: Increase in cluster count keeping compute capacity of each cluster same.**  
**The scaling up and down must be done manually (Depending on the queued queries). The scale out and in is automatic (But we should set the Max and Min number of clusters).**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/63636323-30ab-4b75-b852-485141fd1fce)  

We can follow if any queries are queued in the query diagram:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c54a299e-b6b2-4408-8bea-7e93fb80ae74)  

However, when a Snowflake virtual warehouse is resized, only subsequent queries will make use of the new size. Any queries already running will finish running while any queued queries will run on the newly sized virtual ware‐ house. Scaling a virtual warehouse UP will increase the number of servers.  
Snowflake recommends always starting with eXtra-Small warehouses and only scaling up if you find a compelling reason do that. XS warehouses cost less than five dollars to run for an hour. Our biggest warehouse, the 6XL, costs over 500 times that amount, because it's like running 512 XS warehouses at one time.  

In Snowflake also, there no data silos concept in like in traditional datawarehouses where we can create datamarts that are separated (so that the access to it would be more fast).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/643505db-860d-4619-a7f1-faf5982f26d5)  

The problem with datamarts is that they are **silos** and do not communicate with each other. Data engineers can put the subsets of data needed to complete the vision but then we would have several version (figures) of the same info.  
Snowflake create multiple robust computing warehouses that grab data from the same source, so no need for replication, and each department can his owon warehouse (own size and capacity ==> scale up and down) that retrieve the data effeciently.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4577bbd0-8765-40fa-a8d0-fd05bc06e922)  

**Functionally, a Snowflake Warehouse is more like a laptop CPU.**  

**More details:** https://community.snowflake.com/s/article/Snowflake-What-the-Cluster.  

To control the cost of our data warehouses we have a section under Admin that help us in terms of cost management:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8ca80d72-4f78-47ab-974d-5f561b663ab8)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b3e0ea81-881a-4a31-a093-2e328aeab46d)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c70ab488-2092-40e4-bb56-61f294390140)  

We can also get notifications when we reach the limit and set the profile so that we get email alerts about Resource Monitors.

When loading the file we can catch the query generated behind so we can reuse:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b8758bc3-582c-43ff-ad6c-06c74d2a7d4b)  

**Query :**  
```
                           *COPY INTO "GARDEN_PLANTS"."VEGGIES"."VEGETABLE_DETAILS"
                           FROM '@"GARDEN_PLANTS"."VEGGIES"."%VEGETABLE_DETAILS"/__snowflake_temp_import_files__/'
                           FILES = ('veggie_details_k_to_z_pipe.csv')
                           FILE_FORMAT = (
                               TYPE=CSV,
                               SKIP_HEADER=1,
                               FIELD_DELIMITER=',',
                               TRIM_SPACE=FALSE,
                               FIELD_OPTIONALLY_ENCLOSED_BY=NONE,
                               REPLACE_INVALID_CHARACTERS=TRUE,
                               DATE_FORMAT=AUTO,
                               TIME_FORMAT=AUTO,
                               TIMESTAMP_FORMAT=AUTO
                           )
                           ON_ERROR=ABORT_STATEMENT
                           PURGE=TRUE*
```

We can also create a **file format**  so that we can load the data into tables using this format (it is like a file type definition, cvs, tsv ...):  

                        *create file format garden_plants.veggies.PIPECOLSEP_ONEHEADROW 
                            TYPE = 'CSV'--csv is used for any flat file (tsv, pipe-separated, etc)
                            FIELD_DELIMITER = '|' --pipes as column separators
                            SKIP_HEADER = 1 --one header row to skip
                            ;*
                        
                        We can specify only the agrguments that we requier:  
                        
                        *create file format garden_plants.veggies.COMMASEP_DBLQUOT_ONEHEADROW 
                            TYPE = 'CSV'--csv for comma separated files
                            SKIP_HEADER = 1 --one header row  
                            FIELD_OPTIONALLY_ENCLOSED_BY = '"' --this means that some values will be wrapped in double-quotes bc they have commas in them
                            ;*

Once created, we can see the file formats inside our schema under a new section (after the table section) called **File Formats** :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f4f66538-f831-497f-8b3a-9edda449feda)  

**This ability distenguishes Snowflak from the other SGBDs.**  

## 6. Staging :

Staging, with the same logic of a real world staging, is a place to put data in to be stored in a convinient way later. It is simply a middle stop between OLTP and the OLAP.  
Snowflake stores data in the staging area in a form of files to be used later in the databases similiraly to a FTP process. We can have either internal staging (local in snowflake) or external using one of the 3 cloud providers.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ab155a49-fc60-4f7c-bc87-6503f5965f2c)  

To create an external cloud staging area we need 3 main things:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8397deb6-1aae-4ed9-aed6-36ae399ae09c)  

the stage definition is simply a Snowflake object that contains the references to the cloud location and the cloud access credentials.  

**Note that snoflake is case insensitive (Unless you use quotes when creating things, and then you'll have to use quotes forever after that to deal with the object.)**  
Except for S3 storages where we have to be precise regarding the names of the files.  

To load data from staging area to snowflake tables we run a **COPY INTO** command which is not a SQL command:  
```
                                        COPY INTO WEIGTH_INGEST -- The table to load files to
                                        FROM @MY_S3_BUCKET/load/
                                        FILES = ('WEIGTH.txt')
                                        FILE_FORMAT = (FORMAT_NAME = USDA_FILE_FORMAT);
```
The file format is the format from which the data are coming from in the staging.  

**Note that the snowflake Account can be based on AWS but can load data from external storage such as Azure and GCP.**  

We can view the data in the source files before loading it into the Snowflake tables:  
```
                                        select $1, $2, $3
                                        from @util_db.public.like_a_window_into_an_s3_bucket/LU_SOIL_TYPE.tsv
                                        (file_format => garden_plants.veggies.COMMASEP_DBLQUOT_ONEHEADROW);
```
The select with numbers here referes to how many column we want the data to be shown in (since we don't have a schema yet). we need also to s[ecify the file format (the data devider).  

#### Sequencers:  

**Sequencers** can be created to incremental an id column for example when loading.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6c44597b-2359-477f-b8b7-93ff06075ae3)  

Once created, we can use **.nextval** 
```
                                     INSERT INTO AUTHOR(AUTHOR_UID,FIRST_NAME,MIDDLE_NAME, LAST_NAME) 
                                     Values
                                     (SEQ_AUTHOR_UID.nextval, 'Laura', 'K','Egendorf')
                                     ,(SEQ_AUTHOR_UID.nextval, 'Jan', '','Grover')
                                     ,(SEQ_AUTHOR_UID.nextval, 'Jennifer', '','Clapp')
                                     ,(SEQ_AUTHOR_UID.nextval, 'Kathleen', '','Petelinsek');
```

It can also be used as a default value when creating a new table so that the id will be added in an incremental way.  

```
                                     CREATE OR REPLACE TABLE BOOK
                                    ( BOOK_UID NUMBER DEFAULT SEQ_BOOK_UID.nextval
                                     ,TITLE VARCHAR(50)
                                     ,YEAR_PUBLISHED NUMBER(4,0)
                                    );
```

#### Ingesting semi and unstructured data:  

For Data Science purposes we can ingest non structured data such json files as the are for an later use.  
**The data (the whole file content) are ingested into a table in a single column.**  
using a suitable file format to load (visualize before loading also) the data in the table we can either set the **STRIP_OUTER_ARRAY** to **FALSE** to have the data in a single row or to **TRUE** to have a dictionary per row:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/70ef7f45-dedd-47ad-92f1-4a84b69a8d52)  

Ingesting data into Snowflake using the **STRIP_OUTER_ARRAY = TRUE** makes it possible later to query the data using **SQL**.  


