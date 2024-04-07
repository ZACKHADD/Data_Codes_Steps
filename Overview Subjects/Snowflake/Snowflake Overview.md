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
**The data (the whole file content) are ingested into a table in a single column ==> It is the VARIANT data type which is the key in snowflake of querying non structured data.**  
using a suitable file format to load (visualize before loading also) the data in the table we can either set the **STRIP_OUTER_ARRAY** to **FALSE** to have the data in a single row or to **TRUE** to have a dictionary per row:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/70ef7f45-dedd-47ad-92f1-4a84b69a8d52)  

Ingesting data into Snowflake using the **STRIP_OUTER_ARRAY = TRUE** makes it possible later to query the data using **SQL**.  

For example we can read data using SQL quering conventions to render it like a normalized table:  

```
                                    //returns AUTHOR_UID value from top-level object's attribute
                                    select raw_author:AUTHOR_UID
                                    from author_ingest_json;
                                    
                                    //returns the data in a way that makes it look like a normalized table
                                    SELECT 
                                     raw_author:AUTHOR_UID
                                    ,raw_author:FIRST_NAME::STRING as FIRST_NAME
                                    ,raw_author:MIDDLE_NAME::STRING as MIDDLE_NAME
                                    ,raw_author:LAST_NAME::STRING as LAST_NAME
                                    FROM AUTHOR_INGEST_JSON;
```
**The string here change the type of the data from the original (VARIANT) to STRING (VARCHAR also possible) to remove the quotes. It is simply like if we used CAST() function to change data type.**  

#### Ingesting Nested Semi structered data: 

The nested data are simply more complexed json (for example) format where we will find dictionaries inside others. We can use the same process as we did above to reach each level we want, for example :  

```
                                  select raw_author:BOOKS:YEAR_PUBLISH:AUTHORS
                                  from author_ingest_json;
```
**However if we want to retrieve specific data that is not a level (dictionary) we need either to specify the rank we want to retrieve by specifying it between brakets ([0],[1]..) before we specify the value we want.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/998819fb-70fb-4d7b-a911-91a43978579f)  

**We can not retrieve Fiona and Gian in the same time using the rank specification method.**  

```
                                SELECT RAW_NESTED_BOOK:authors[1].first_name
                                FROM NESTED_INGEST_JSON;
```

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2e2d5b0a-3ad7-451c-b9dd-43061a50d543)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/0488a2c7-4a6f-4ce8-9715-a5733a3c8206)  


**Note that only the first appearances of the level we are at will be displayed**  

**Or we use a function called FLATTEN (in two ways) that will read all the values at once (like if perform a loop on all the values in Python).**  

```
                               //Use these example flatten commands to explore flattening the nested book and author data
                               SELECT value:first_name
                               FROM NESTED_INGEST_JSON
                               ,LATERAL FLATTEN(input => RAW_NESTED_BOOK:authors);
                               
                               SELECT value:first_name
                               FROM NESTED_INGEST_JSON
                               ,table(flatten(RAW_NESTED_BOOK:authors));

```
**Note here that all the appearances of the level we are at will be displayed.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3028778c-0602-4e30-b256-bb1bfe90ec35)  

**We can also retrieve several values field at once, renaming the colums displayed and casting them to a data type different than VARIANT (to remove quotes).**  

```
                               //Add a CAST command to the fields returned
                               SELECT value:first_name::VARCHAR, value:last_name::VARCHAR
                               FROM NESTED_INGEST_JSON
                               ,LATERAL FLATTEN(input => RAW_NESTED_BOOK:authors);
                               
                               //Assign new column  names to the columns using "AS"
                               SELECT value:first_name::VARCHAR AS FIRST_NM
                               , value:last_name::VARCHAR AS LAST_NM
                               FROM NESTED_INGEST_JSON
                               ,LATERAL FLATTEN(input => RAW_NESTED_BOOK:authors);
```

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f2f38b77-a18d-4580-90f1-874fd23af874)  

We can also query data like we normaly do with normalized one using conditions etc:  

```

                                 //select statements as seen in the video
                                 SELECT RAW_STATUS
                                 FROM TWEET_INGEST;
                                 
                                 SELECT RAW_STATUS:entities
                                 FROM TWEET_INGEST;
                                 
                                 SELECT RAW_STATUS:entities:hashtags
                                 FROM TWEET_INGEST;
                                 
                                 //Explore looking at specific hashtags by adding bracketed numbers
                                 //This query returns just the first hashtag in each tweet
                                 SELECT RAW_STATUS:entities:hashtags[0].text
                                 FROM TWEET_INGEST;
                                 
                                 //This version adds a WHERE clause to get rid of any tweet that 
                                 //doesn't include any hashtags
                                 SELECT RAW_STATUS:entities:hashtags[0].text
                                 FROM TWEET_INGEST
                                 WHERE RAW_STATUS:entities:hashtags[0].text is not null;
                                 
                                 //Perform a simple CAST on the created_at key
                                 //Add an ORDER BY clause to sort by the tweet's creation date
                                 SELECT RAW_STATUS:created_at::DATE
                                 FROM TWEET_INGEST
                                 ORDER BY RAW_STATUS:created_at::DATE;
                                 
                                 //Flatten statements that return the whole hashtag entity
                                 SELECT value
                                 FROM TWEET_INGEST
                                 ,LATERAL FLATTEN
                                 (input => RAW_STATUS:entities:hashtags);
                                 
                                 SELECT value
                                 FROM TWEET_INGEST
                                 ,TABLE(FLATTEN(RAW_STATUS:entities:hashtags));
                                 
                                 //Flatten statement that restricts the value to just the TEXT of the hashtag
                                 SELECT value:text
                                 FROM TWEET_INGEST
                                 ,LATERAL FLATTEN
                                 (input => RAW_STATUS:entities:hashtags);
                                 
                                 
                                 //Flatten and return just the hashtag text, CAST the text as VARCHAR
                                 SELECT value:text::VARCHAR
                                 FROM TWEET_INGEST
                                 ,LATERAL FLATTEN
                                 (input => RAW_STATUS:entities:hashtags);
                                 
                                 //Flatten and return just the hashtag text, CAST the text as VARCHAR
                                 // Use the AS command to name the column
                                 SELECT value:text::VARCHAR AS THE_HASHTAG
                                 FROM TWEET_INGEST
                                 ,LATERAL FLATTEN
                                 (input => RAW_STATUS:entities:hashtags);
                                 
                                 //Add the Tweet ID and User ID to the returned table
                                 SELECT RAW_STATUS:user:id AS USER_ID
                                 ,RAW_STATUS:id AS TWEET_ID
                                 ,value:text::VARCHAR AS HASHTAG_TEXT
                                 FROM TWEET_INGEST
                                 ,LATERAL FLATTEN
                                 (input => RAW_STATUS:entities:hashtags);
```
**From these queries we can create a view that we can use to have a normalized presentation of the json data and we can also use it to populate real tables. This ability to process and store non structured data using the VARIANT type of data is a game changer when it comes to normalizing non structured data in warehouses. A thing that was extreamly difficult in the classic tools of data warhousing.**  

```
                                create or replace view SOCIAL_MEDIA_FLOODGATES.PUBLIC.HASHTAGS_NORMALIZED as
                                (SELECT RAW_STATUS:user:id AS USER_ID
                                ,RAW_STATUS:id AS TWEET_ID
                                ,value:text::VARCHAR AS HASHTAG_TEXT
                                FROM TWEET_INGEST
                                ,LATERAL FLATTEN
                                (input => RAW_STATUS:entities:hashtags)
                                );
```

## 7. Collaboration, Marketplace & Cost Estimation:  

in this workshop, we review all the aspects related to collaboration, marketplace and cost estimation in Snowflake.  

#### Share:

Snowflake makes it possible to secure share data with tiers. It already does this with all customers by sharing the **ACCOUNT_USAGE** that provide data to the **SNOWFLAKE** (A db provided by default in the account) database called also **the Account Usage Share.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ce42f16b-ec27-4ce9-8869-fb633fa68411)  

To get the Data from a share, we click on Download button: 

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a2a96e3e-5dd9-4e67-bbe7-f4f0899d1416)  

This will create a database that reads directly from the share. After that we just name the database and the role who can access it.  
**Note that the database will query this shared data. This database will not take up any storage space in your account.**  

We can grant access to databases using the code sample below:  

```
                                             grant imported privileges
                                             on database SNOWFLAKE_SAMPLE_DATA
                                             to role SYSADMIN;
```

Note that we mentioned **imported privileges** because privileges for a shared database are pre-defined for maximum data security.  

We can query the shared data by querying the database built on them. We can also export the results of the queries to CSV and so on.  
**The real value of consuming shared data is:**
- Someone else will maintain it over time and keep it fresh.
- Someone else will pay to store it (in the snowflake Data Cloud).
- You will only pay to query it (we pay the data provider andsnowflake for every query we perform).

**Bare inmind that every user has a default role and a default werhouse for computing. So the management of these warehouses and their affectation must be an ADMIN task.**  

In the same manner as granting access to an object, we can grant access to a function but only using code (no button for that):  

```
                                             grant usage 
                                             on function UTIL_DB.PUBLIC.GRADER(VARCHAR, BOOLEAN, NUMBER, NUMBER, VARCHAR) 
                                             to SYSADMIN; -- here we grant usage to a SYSADMIN Role.
```

#### Costs Estimation and Management: 

We can monitor the cost of our data warehouses using the **Cost Management** section that shows during a period how much we spent in terms of **Computing** and **Cloud** (Storage).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/37578331-6722-49d5-ad88-0893f9cff2d4)  

the link here gives an idea regarding the cost conversion in USD per region. **Note that we should pay attention to the region and the cloud provider**
The same way we create databases, we can create Data warehouses:  

```
                                             create warehouse INTL_WH 
                                             with 
                                             warehouse_size = 'XSMALL' 
                                             warehouse_type = 'STANDARD' 
                                             auto_suspend = 600 --600 seconds/10 mins
                                             auto_resume = TRUE;
```

Also the stages can be created either using the wizard or the code:  

```
                                             create stage util_db.public.aws_s3_bucket url = 's3://uni-cmcw';
```

When we use a lot the same select to retrieve data, we can wrap it in a view and query it like a table:  

```
                                             create view intl_db.public.NATIONS_SAMPLE_PLUS_ISO 
                                             ( iso_country_name
                                               ,country_name_official
                                               ,alpha_code_2digit
                                               ,region) AS
                                               <put select statement here>
                                             ;
```

#### Sharing Data with others: 

in opposition to data shared with us that we can find in **Private Sharing** section, we can also share data with others using the section **Provider Studio**.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ec9f0a24-6148-4f8a-8e47-ae3d4e4d178c)  

Once the listing is named and we specify that it is only accessible by specific accounts, we define what this listing should contain:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e74618d3-f188-43ed-ade1-c4618a6ed1f5)  

Note that the objects in the listing should be owned and have access roles as we want since it will be inhereted. If we would like change this setting after creating the listing and adding the object, we should remove it first, set the roles and then add it again.  

After choosing what to include in the listing, we should specify waht are the accounts to share data with:  

 ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b885d7bb-b0ec-40bb-a351-3203387f6b58)  

 ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2ddc6ae9-3469-4770-842f-1b1cafcabf2a)  

 Once the sharing is done, we can access it from the other account in the **Private Sharing** section where we can follow the same steps we did previously to get the data from the sharing to a database:  

 ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1dfc54b0-6279-4555-9c56-ef133934c5b4)  

How it works: 

**Because your trial account (WDE) and your ACME account are on different clouds, in different regions, Snowflake will automatically perform replication to the other cloud/region.  
This will be a cost that WDE/Osiris covers. If Osiris doesn't want to cover that cost, he could insist that Lottie's team get a Snowflake account on the same cloud and in the same region as his primary account. This may become part of their negotiations.**  

**The replication accross different Cloud Provides and regions is done based on a database that was automaticaly created at the data provider account level:**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4522f67b-6cb7-481b-987b-667329887a9d)  

This how the listing looks like at the provider's level and what it contains:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/48b6ef7d-8787-4f88-bd66-543c3d82bc82)  

A listing not only allows for a nicer name, you can also add information into the listing that will make it much easier for consumers to use your data.  
One thing that might make the data easier to use would be adding a Data Dictionary.  
This can be done at the optional information level:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2f78fa4d-94f5-4ab2-a349-e1d040d66ef4)  

We can add the dictionnary to every table if we like, and it gives a detailed description of the columns:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/47a2dd82-a148-43d4-8fee-e345f524abe5)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/000a2448-704f-49b5-8103-2a5892cc0a08)  

We can add also some sample queries:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7060ec2f-14ec-4323-bf7c-860ecb39922c)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2eba0c01-1945-4c96-a2d9-ac7de1f48e1b)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d47b37b2-a133-4361-9910-9f8210e35ecb)  

This is what we see at the customer level once the secure share is replicated:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e203e9ab-7b91-4f4b-83cf-9638dd6255a1)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/849e1cd8-f15c-476f-8c2e-1351537375c2)  

Once we click on **Get**, we specify the roles to grant access to along with ACCOUNTADMIN, we can start querying data:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/72184128-d85b-4034-a904-26629e0c8691)  

This will automaticaly open a new worksheet with the sample query we have.  

We can also check the database created from the share, and check if all the roles we want have usage access to the data:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a8e44118-796c-43fc-8c6f-478220855ea0)  

If we want to add views also to the secure share, we need to alter them and the the secure property:  

```
                                          alter view intl_db.public.NATIONS_SAMPLE_PLUS_ISO
                                          set secure; 
```

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/19f2ee1d-95b7-4b45-a055-0eee3e2f723d)  

**Note however, that the views should refere only to the tables already shared, if a view is a join of data from different databases/tables that are not in the secure share, they will not be added.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/da62cfeb-ae84-48cb-8d7e-ca8ea41a6aee)  

**Note also, that we can not share an already shared database (Share) like for example the SNOWFLAKE share. Simply the shares will not be displyed among choices to add in the share.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/026f8c1a-0757-4ad1-866f-30a0a86f6c9f)  

We can now monitor all the accounts we have to see our comsumption per type of workload:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2308dfb2-9331-4371-8771-37da4bbb5040)  

To see all the metadata regarding the organization accounts and activity, we can query the SNOWFLAKE database that keeps track of everything done in the organisation.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/621e560e-6560-4dc2-b557-ca5d29c12e6a)  

**Note that when the share is done between two accounts with different cloud providers, Snowflake will create a new database called SNOWFLAKE$GDS and use it to automate the replication needed.**  

#### Snowflake Costing:  

To undersand how snowflake is billing it's serveces, here it is a guide : https://www.snowflake.com/pricing/pricing-guide/  
**We have several costs types such as : Storage, Computing, serverless cost(Replication, Clustering ...) and Cloud services (permanent state management and overall coordination of Snowflake.). The sahred data does not cost anything for the end user, only for the one sharing it.**  
**Note that the cost varies depending or regions and the category of cost : https://www.snowflake.com/pricing/  
To estimate the computing cost we can check the the credit used per hour that depends on the size of the warehouse.**  

Thanks to Role-Based Access Control, most corporate workers using Snowflake will not be able to change warehouses capacities and cost the company a lot of money. **The ressource monitors makes it possible to set alerts on costs.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5e516d42-430e-4d95-9629-ef403f3ee319)  

**The monitoring ressource can be set on the Account level or on the Warehouse level.** and we can set the quota.  
In addition to the monitoring part, we can use **Budgets** so that we view the projections of our costs:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3cc11129-d8f7-42e2-aafc-89b80ce45410)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6d52b5e4-d60b-44ac-8edd-dba641bff096)  

**Relevent Query** : we can read the and query the result of query (that is stored in memory) using the following query:  

```
                                 result_scan(last_query_id()) -- This query needs to be run immediatly after the query we want to analyse.
```

#### Snowflake Market Place:  

The snowflake marketplace is like a mall of data and apps where we can purchase data (get an access, a share access, to it without a need to download it).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8285996e-57ea-4a13-80a2-715ba8734418)  

Some data can have a free access:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/44352d41-a29e-43da-a942-80186ea98668)  

For example, the weather data share is free, and once obtained we can query the data using the query sample already existing:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/89d506de-4211-40a0-bc3b-02e78fdf9fb9)  

#### User-Defined (Table) Function:  

**Remember that when we set a variable we should use $ to call it and select it's content, as below :**  

```
                                 -- Set the variable
                                 
                                 set sample_vin = 'SAJAJ4FX8LCP55916';
                                 
                                 
                                 -- Select the variable
                                 
                                 select $sample_vin;
                                 
                                 -- Use the variable in a complexed query
                                 
                                 select VIN
                                 , manuf_name
                                 , vehicle_type
                                 , make_name
                                 , plant_name
                                 , model_year_name as model_year
                                 , model_name
                                 , desc1
                                 , desc2
                                 , desc3
                                 , desc4
                                 , desc5
                                 , engine
                                 , drive_type
                                 , transmission
                                 , mpg
                                 from
                                   ( SELECT $sample_vin as VIN
                                   , LEFT($sample_vin,3) as WMI
                                   , SUBSTR($sample_vin,4,5) as VDS
                                   , SUBSTR($sample_vin,10,1) as model_year_code
                                   , SUBSTR($sample_vin,11,1) as plant_code
                                   ) vin
                                 JOIN vin.decode.wmi_to_manuf w 
                                     ON vin.wmi = w.wmi
                                 JOIN vin.decode.manuf_to_make m
                                     ON w.manuf_id=m.manuf_id
                                 JOIN vin.decode.manuf_plants p
                                     ON vin.plant_code=p.plant_code
                                     AND m.make_id=p.make_id
                                 JOIN vin.decode.model_year y
                                     ON vin.model_year_code=y.model_year_code
                                 JOIN vin.decode.make_model_vds vds
                                     ON vds.vds=vin.vds 
                                     AND vds.make_id = m.make_id;
```

**To create a function:**  

- Give the function a name.  
- Tell the function what information you will be passing into it. 
- Tell the function what type of information you expect it to pass back to you (Return).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a1871730-a6d8-4377-83fb-5ca1c0229ea3)  

Example:  

the following function accepts a variable and returns a table (could be one row per value of the variable).  

Note that that the function is put to secure since we will share it.  

```
                          create or replace secure function vin.decode.parse_and_enhance_vin(this_vin varchar(25))
                          returns table (
                              VIN varchar(25)
                              , manuf_name varchar(25)
                              , vehicle_type varchar(25)
                              , make_name varchar(25)
                              , plant_name varchar(25)
                              , model_year varchar(25)
                              , model_name varchar(25)
                              , desc1 varchar(25)
                              , desc2 varchar(25)
                              , desc3 varchar(25)
                              , desc4 varchar(25)
                              , desc5 varchar(25)
                              , engine varchar(25)
                              , drive_type varchar(25)
                              , transmission varchar(25)
                              , mpg varchar(25)
                          )
                          as $$
                          
                              select VIN
                          , manuf_name
                          , vehicle_type
                          , make_name
                          , plant_name
                          , model_year_name as model_year
                          , model_name
                          , desc1
                          , desc2
                          , desc3
                          , desc4
                          , desc5
                          , engine
                          , drive_type
                          , transmission
                          , mpg
                          from
                            ( SELECT this_vin as VIN  --here we didn't use a $ since it is not a local variable (it is autodeclared in the function)
                            , LEFT(this_vin,3) as WMI
                            , SUBSTR(this_vin,4,5) as VDS
                            , SUBSTR(this_vin,10,1) as model_year_code
                            , SUBSTR(this_vin,11,1) as plant_code
                            ) vin
                          JOIN vin.decode.wmi_to_manuf w 
                              ON vin.wmi = w.wmi
                          JOIN vin.decode.manuf_to_make m
                              ON w.manuf_id=m.manuf_id
                          JOIN vin.decode.manuf_plants p
                              ON vin.plant_code=p.plant_code
                              AND m.make_id=p.make_id
                          JOIN vin.decode.model_year y
                              ON vin.model_year_code=y.model_year_code
                          JOIN vin.decode.make_model_vds vds
                              ON vds.vds=vin.vds 
                              AND vds.make_id = m.make_id    
                           
                          $$;
```

**What is we want to load a file with 3 columns into a table with more columns?.** We just need to precise in the properties, **of the file format**, that we want to parse the file so Snowflake can find the columns with the matching headers and to set error_on_mismatch to FALSE so that it throw an error for the files it doesn't find:  
**we also don't use the skip header property!!!**  

```
                          CREATE FILE FORMAT util_db.public.CSV_COL_COUNT_DIFF 
                          type = 'CSV' 
                          field_delimiter = ',' 
                          record_delimiter = '\n' 
                          field_optionally_enclosed_by = '"'
                          trim_space = TRUE
                          error_on_column_count_mismatch = FALSE
                          parse_header = TRUE;
```

**This file format is only valid for loading data and not reading it as we did before.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d7f711ad-3617-4229-be45-57bfb3b3e783)  

**it is because another property (match_by_column_name) is needed and it is defined in the copy into query:**  

```
                          copy into stock.unsold.lotstock
                          from @stock.unsold.aws_s3_bucket/Lotties_LotStock_Data.csv
                          file_format = (format_name = util_db.public.csv_col_count_diff)
                          match_by_column_name='CASE_INSENSITIVE';
```

Now, the rest of the empty columns can be populated using the secure function that the ADU account shared with the ACME account!!  
**Note that the function uses data from tables in the data logic of the first account to calculate and give outputs using data of the second account (this ons has access only to the function)!**  
**The function takes the input from account 2 and run it against tables in account 1 and give the result in  account 2.**  
====> this is the power of **Snowflake Collaboration**.  

We can now use the function to select the results joined with the other tables we need:  

'''
                            set my_vin = '5J8YD4H86LL013641'; -- Variable for the value we want
                            select $my_vin;
                            
                            -- the select satatement:
                            
                            select ls.vin, pf.manuf_name, pf.vehicle_type
                                    , pf.make_name, pf.plant_name, pf.model_year
                                    , pf.desc1, pf.desc2, pf.desc3, pf.desc4, pf.desc5
                                    , pf.engine, pf.drive_type, pf.transmission, pf.mpg
                            from stock.unsold.lotstock ls
                            join 
                                (   select 
                                      vin, manuf_name, vehicle_type
                                    , make_name, plant_name, model_year
                                    , desc1, desc2, desc3, desc4, desc5
                                    , engine, drive_type, transmission, mpg
                                    from table(ADU_VIN.DECODE.PARSE_AND_ENHANCE_VIN($my_vin))
                                ) pf
                            on pf.vin = ls.vin;
'''

or we can use it directly to update the data in the target table we have:  

```
                            set my_vin = '5J8YD4H86LL013641'; -- Variable for the value we want
                            select $my_vin;

                          -- We're using "s" for "source." The joined data from the LotStock table and the parsing function will be a source of data for us. 
                          -- We're using "t" for "target." The LotStock table is the target table we want to update.

                          update stock.unsold.lotstock t
                          set manuf_name = s.manuf_name
                          , vehicle_type = s.vehicle_type
                          , make_name = s.make_name
                          , plant_name = s.plant_name
                          , model_year = s.model_year
                          , desc1 = s.desc1
                          , desc2 = s.desc2
                          , desc3 = s.desc3
                          , desc4 = s.desc4
                          , desc5 = s.desc5
                          , engine = s.engine
                          , drive_type = s.drive_type
                          , transmission = s.transmission
                          , mpg = s.mpg
                          from 
                          (
                              select ls.vin, pf.manuf_name, pf.vehicle_type
                                  , pf.make_name, pf.plant_name, pf.model_year
                                  , pf.desc1, pf.desc2, pf.desc3, pf.desc4, pf.desc5
                                  , pf.engine, pf.drive_type, pf.transmission, pf.mpg
                              from stock.unsold.lotstock ls
                              join 
                              (   select 
                                    vin, manuf_name, vehicle_type
                                  , make_name, plant_name, model_year
                                  , desc1, desc2, desc3, desc4, desc5
                                  , engine, drive_type, transmission, mpg
                                  from table(ADU_VIN.DECODE.PARSE_AND_ENHANCE_VIN($my_vin))
                              ) pf
                              on pf.vin = ls.vin
                          ) s
                          where t.vin = s.vin;
```

Now thet we know how it works, we can set a **script (Procedure)** that will automaticaly run the update query above **againt all the VIN values** we have, to store them all in the target table. We will use the **for each** clause in this script.  

The components of stored procedures are like follows :  

**Words like DECLARE, BEGIN, END, FOR are for "control of flow". They allow you to dictate which statements will take place in a certain order and be run one after another.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/cdc7b01d-c7a9-4968-9a52-fe46efeadf46)  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/00e0bd26-5304-45bd-b641-56cf26520972)  

The stored procedure:  

```
                          DECLARE
                              update_stmt varchar(2000);
                              res RESULTSET;
                              cur CURSOR FOR select vin from stock.unsold.lotstock where manuf_name is null;
                          BEGIN
                              OPEN cur;
                              FOR each_row IN cur DO
                                  update_stmt := 'update stock.unsold.lotstock t '||
                                      'set manuf_name = s.manuf_name ' ||
                                      ', vehicle_type = s.vehicle_type ' ||
                                      ', make_name = s.make_name ' ||
                                      ', plant_name = s.plant_name ' ||
                                      ', model_year = s.model_year ' ||
                                      ', desc1 = s.desc1 ' ||
                                      ', desc2 = s.desc2 ' ||
                                      ', desc3 = s.desc3 ' ||
                                      ', desc4 = s.desc4 ' ||
                                      ', desc5 = s.desc5 ' ||
                                      ', engine = s.engine ' ||
                                      ', drive_type = s.drive_type ' ||
                                      ', transmission = s.transmission ' ||
                                      ', mpg = s.mpg ' ||
                                      'from ' ||
                                      '(       select ls.vin, pf.manuf_name, pf.vehicle_type ' ||
                                              ', pf.make_name, pf.plant_name, pf.model_year ' ||
                                              ', pf.desc1, pf.desc2, pf.desc3, pf.desc4, pf.desc5 ' ||
                                              ', pf.engine, pf.drive_type, pf.transmission, pf.mpg ' ||
                                          'from stock.unsold.lotstock ls ' ||
                                          'join ' ||
                                          '(   select' || 
                                          '     vin, manuf_name, vehicle_type' ||
                                          '    , make_name, plant_name, model_year ' ||
                                          '    , desc1, desc2, desc3, desc4, desc5 ' ||
                                          '    , engine, drive_type, transmission, mpg ' ||
                                          '    from table(ADU_VIN.DECODE.PARSE_AND_ENHANCE_VIN(\'' ||
                                            each_row.vin || '\')) ' ||
                                          ') pf ' ||
                                          'on pf.vin = ls.vin ' ||
                                      ') s ' ||
                                      'where t.vin = s.vin;';
                                  res := (EXECUTE IMMEDIATE :update_stmt);
                              END FOR;
                              CLOSE cur;   
                          END;
```

**Note that : this operation will be slow. Snowflake was originally optimized for bulk loading and bulk updating. it was designed for loading and updating large record sets with a single statement, not for updating one row at a time, using a FOR LOOP. There are more efficient ways to achieve the result we achieved above, but this lesson's example allowed you to see how each part became a building block for the next.**  
**In the future Snowflake will offer a new table type that allows for these individual row update operations to run more quickly. For now, don't focus on the speed, just focus on understanding how the flow works.**  

## 8. Data Application Builders Workshop: 

Working with data in production is so complex especially when we have a lot of senarios we want to test and visualize. This is true in data science field whre changing the parameters we work with is so frequent, and we will need to change the variables in the script and run it again and again.  
The solution for this, is to create a web application that will do all of this for us with a front layer making it user friendly. However, using the classic, powerful tools would take too much time. We need a simple tool that will do this for us in a quick way.  
**Stremlit** is a framework offering this solution for data projects. https://www.youtube.com/watch?v=R2nr1uZ8ffc&ab_channel=Streamlit    

**Snowflake bought Streamlit (2022) and now there are two versions of Streamlit.**   
One version of Streamlit (the original) is a standalone and you don't have to have a Snowflake Account to use it.  
The other is a highly integrated tool called Streamlit in Snowflake (around here we call it "SiS"). **Easy to use and configure**  

#### Create a Streamlit App in Snowflake:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a92b4506-72c7-46bb-a133-f51f9bab98ee)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6b01fc14-52b6-42b7-a550-a915b0cc0d51)  

This automatically creates an app example in the database (a stage holding the streamlit files of the app) we choose and we can see the code generated behind which is a **Python** code.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/64144407-203e-42eb-97f5-0012fed8b969)  

We can now edit the application as we wish by refering to the streamlit doc (snippet to use). for example to add a select box : https://docs.streamlit.io/library/api-reference/widgets/st.selectbox  

We need now to render data from a database. First we create one and we load data in it.  
**We can create an internal stage (like the external ones we used before) to hold our data files and call them from there instead of our laptop.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1183c4e5-f418-4da3-b3da-8abe03ec5343)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5a38a541-6aff-47b5-92a8-009a9dc4330d)  

Sometimes the order of columns in our source files is not the same as  our target table. We can reorder the columns during the COPY INTO loading phase:  
https://docs.snowflake.com/en/user-guide/data-load-transform#reorder-csv-columns-during-a-load  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ab3d52cf-ec46-49e8-8b08-d42696debf5b)  

The hack is to load from a select statement rather than the original file itself. we use the select $1, $2 mothod to reorder the columns and rename them as per the tagret table schema.  

Once loaded into the database, we can now call the data to be rendered in our Streamlit App:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8eed0a20-ee12-4ca5-b4a5-f800fb7f423e)  

We simply get the credentials to access the database using active_session and we create a dataframe from the table we choose.  

We can then create a simple form that stores data in a table as shown bellow:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/22786af6-54d1-4b4a-8475-72304faf870e)  

in the stage that was automatically created, we can find the code file (py) of our application:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/00ea38d8-3039-422c-a8f9-63552531a925)  

**Note that streamlit apps uses a warehouse to keep the pages runing, so we need to leave the app page whenever we don't use it. We should also create a ressource monitoring alert.**  

Since Snowpark is built on top of Python, many libraries are supported : https://repo.anaconda.com/pkgs/snowflake/  
But, beware. Not all packages in the Anaconda channel for Snowpark can be used for SiS. Because of this, use the list as a starting point and then test the packages you want to use in your SiS app by typing in the import statement for the library you want to use.  
If you don't see an error message about the package you just imported, you can use that package. 

#### App development process:  

In the 1980s and 1990s, teams used to spend months writing up an official Requirements Document and only when all stakeholders had "signed off" was the document passed to the developers. This was called a "Waterfall Method" -- you can research **SDLC Waterfall** Method and read more about it if you are curious. **SDLC stands for Systems Development Life Cycle** and just means "process or method for developing software."

Then, Rapid Prototyping (RAD, Agile, and others) became more popular. In our exampl we are using an ITERATIVE SDLC, and it's based on RAPID PROTOTYPING.  
Many times a customer will "gold plate" requirements when they are expressing how they want an app to look or behave. A good developer or requirements analyst will be able to track the different requirements according to whether they are "must haves" or "nice to haves."   

In rapid prototyping, we deliver progressivly parts of the project in short timeframes called **"Sprints"**. In old waterfall development projects, teams would set a date months or years in the future as a deadline, and then work backward setting "Milestone" dates for when each phase of the project HAD to be finished.  

in the next examples we are going to add some functionnalities to our app such as giving the customer the ability to add a name for the order.  

If we want to add a unique column ID to an existing table we need to truncate the table and then add the column:  

```
                         alter table SMOOTHIES.PUBLIC.ORDERS 
                         add column order_uid integer --adds the column
                         default smoothies.public.order_seq.nextval  --sets the value of the column to sequence
                         constraint order_uid unique enforced; --makes sure there is always a unique value in the column;
```

#### Interfaces :

We have two types of user interfaces: 
 - GUI: Graphical user interface with buttons.
 - CLI:Command line interface using only commande lines.  
Most of modern applications are using both interfaces.

#### Functions in Snowflake:  

As we can create a function python, we can do the same in snowflake, for example:  

```
                         create or replace function NEUTRALIZE_WHINING (statement VARCHAR(100))
                         returns VARCHAR as 'select INITCAP(statement)';
```
No need to use $ to call the variables in a function, not like local variables in the worksheet. Also, we can call the function using a select clause:  

```
                         select NEUTRALIZE_WHINING('ZaKaRia'); -- this will return : Zakaria
```

The final scripts for our twon apps we created in snowflake are the following:  

```
# Import python packages
import streamlit as st
from snowflake.snowpark.context import get_active_session
from snowflake.snowpark.functions import col

# Write directly to the app
st.title(":cup_with_straw: Example Streamlit App :cup_with_straw:")
st.write(
    """Choose the fruits you want in your custom Smoothie!
    """
)

name_on_order = st.text_input('Name on Smoothie:')
st.write('The name on your Smoothie will be:', name_on_order)

session = get_active_session()
my_dataframe = session.table("smoothies.public.fruit_options").select(col('FRUIT_NAME'))
#st.dataframe(data=my_dataframe, use_container_width=True)
ingredients = st.multiselect('Choose up to 5 ingredients:',my_dataframe,max_selections=5)

if ingredients:
    ingredients_string = ''
    for fruit in ingredients:
        ingredients_string +=fruit + ' '
    #ingredients_string.strip(' ')
    st.write(ingredients_string)

    my_insert_stmt = """ insert into smoothies.public.orders(ingredients,name_on_order)values ('""" + ingredients_string + """','"""+ name_on_order + """')"""
    
    time_to_insert = st.button('Submit Order')
    
    if time_to_insert:
        session.sql(my_insert_stmt).collect()
        
        st.success('Your Smoothie is ordered, '+name_on_order+'!', icon="✅")
```

The order validation app:  

```
# Import python packages
import streamlit as st
from snowflake.snowpark.context import get_active_session
from snowflake.snowpark.functions import col, when_matched

# Write directly to the app
st.title(":cup_with_straw: Pending Smoothie Orders :cup_with_straw:")
st.write(
    """Orders that need to filled.
    """
)

session = get_active_session()
my_dataframe = session.table("smoothies.public.orders").filter(col("ORDER_FILLED")==0).collect()

if my_dataframe:
    editable_df = st.experimental_data_editor(my_dataframe)
    submitted = st.button('Submit')
    if submitted:
        
        og_dataset = session.table("smoothies.public.orders")
        edited_dataset = session.create_dataframe(editable_df)
        
        try:
            
            og_dataset.merge(edited_dataset
                             , (og_dataset['order_uid'] == edited_dataset['order_uid'])
                             , [when_matched().update({'ORDER_FILLED': edited_dataset['ORDER_FILLED']})]
                            )
            st.success("Someone clicked the button.", icon= "👍")
        except:
            st.write('Somthing went wrong.')
else:
    st.success('There are no pending orders right now', icon= "👍")
```

**Now that we finiched tuning our app, we can deploy it in Streamlit rather than leaving it inside Snowflake. We can copy all our files in GITHUB account that will serve as a backend for the app, and give access to streamlit so it can read the files from the GITHUB repository.**  

The biggest differences between SiS and SniS are:  

1) How users connect to the app.   
2) How we connect our app to Snowflake.  

With SniS, users will be able to connect to the app more easily. We can set up our SniS app in a way that doesn't require them to log in or have a USER in our Snowflake account. In fact, Streamlit will host our app for free if we make it open to the public.  

Few changes are need to be done to migrate from SiS to Streamlit:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/83aca756-8e16-4331-b3fe-2ac7eb5a360b)  

Also we need a requirement file containing the version of snowflake and python and also the connector.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/061304b6-ce3e-40ae-9977-0e8408a8429a)  

Once all of this is set, we can now create a streamlit account and link it to our repository:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/177fdf09-74ca-40bd-b897-90ca762c99f1)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3ba716c4-6671-47f7-8c7e-43213eab6428)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/bb506f6a-d972-46ea-9fe5-a7f80f95adfa)  

Once our workspace is created we can deploy a new app:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1a570077-6826-4463-be63-3177780f1cbe)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/16e03e20-db2c-4465-a7ee-42ac8c9e1b7c)  

An error will appear regarding the connection mode:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/910535c8-90fd-4b35-95f0-db0998e7e1ea)  

Till now we still didn't specify to Streamlit how it will connect to our Snoflake database:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/34d0e756-a4a8-4ab8-9416-9ce3a5fa743f)  

To do so we need to edit a file called **secrets.toml**:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/a7302ab0-9f89-4241-b2d2-b7a46f824f4b)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/618a3780-01c3-4db9-ab75-c3ee94f6cdda)  

To do this modifications we can grab the paranmeters to set from the doc: https://docs.streamlit.io/knowledge-base/tutorials/databases/snowflake#add-connection-parameters-to-your-local-app-secrets  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d738d400-57cc-4e26-a194-b4499669f8bd)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/02b627dd-3429-4a81-add1-0e9b164f7203)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/cb63b18a-2c22-4e5b-b392-6b174eb5df09)  

**Note that streamlit may not process the password and user name if they contain accents. It is better to modify them.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/63bcee04-f592-46f6-bfcd-a8723bf16d46)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/24e3f3a5-bd92-472d-aea8-46321b47dbff)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/74fc239d-377f-4642-a5f2-59f5dc03e896)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8a8205d1-36fc-4a28-9433-547cc9f7e379)  

#### Using API (external source of data to use in the app) in the Streamlit App:  

Using API in our app will need a python package called **Request.**  
To add any package to a SniS app, we need two steps:  
1) Add the library to the requirements.txt file so that Streamlit knows to install it when starting up the project.
2) Add the import statement to the body of the streamlit_app.py file so it is ready to be used in the code.

**Note that:**  
- Anytime we change the streamlit_app.py file, and commit the changes in GitHub, the app will automatically start using the changes.
- However, anytime we make changes to the requirements.txt file, we will need to reboot the app.

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d31acc03-52c5-4cb6-9852-30e02744e087)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c23d8b21-f35d-4247-a669-80f754ed7825)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5d0a9154-d455-4dd6-a1a0-d9841461e05f)  

**The custom smoothies app with nutrition information API integrated:**  

```
# Import python packages
import streamlit as st
from snowflake.snowpark.functions import col
import requests
import pandas as pd

# Write directly to the app
st.title(":cup_with_straw: Customize Your Smoothie!  :cup_with_straw:")
st.write(
    """Choose the fruits you want in your custom Smoothie!
    """
)

name_on_order = st.text_input('Name on Smoothie:')
st.write('The name on your Smoothie will be:', name_on_order)

cnx = st.connection("snowflake")
session = cnx.session()

my_dataframe = session.table("smoothies.public.fruit_options").select(col('FRUIT_NAME'),col('SEARCH_ON'))
#st.dataframe(data=my_dataframe, use_container_width=True)
#convert the snowpark Dataframe to python dataframe so we can use LOC function
pd_df = my_dataframe.to_pandas()
ingredients = st.multiselect('Choose up to 5 ingredients:',my_dataframe,max_selections=5)

if ingredients:
    ingredients_string = ''
    
    for fruit in ingredients:
        ingredients_string +=fruit + ' '
        
        search_on=pd_df.loc[pd_df['FRUIT_NAME'] == fruit, 'SEARCH_ON'].iloc[0]
        st.write('The search value for ', fruit,' is ', search_on, '.')
        
        st.subheader(fruit+' Nutrition information')
        #API part
        fruityvice_response = requests.get("https://fruityvice.com/api/fruit/" + search_on)
        fv_df = st.dataframe(data=fruityvice_response.json(), use_container_width=True)
    #ingredients_string.strip(' ')
    st.write(ingredients_string)

    my_insert_stmt = """ insert into smoothies.public.orders(ingredients,name_on_order)values ('""" + ingredients_string + """','"""+ name_on_order + """')"""
    
    time_to_insert = st.button('Submit Order')
    
    if time_to_insert:
        session.sql(my_insert_stmt).collect()
        
        st.success('Your Smoothie is ordered, '+name_on_order+'!', icon="✅")
```


## 9. Data Lake Workshop:  

When we talk about Data Types - we usually mean the column type settings for Structured Data stored in Tables, like NUMERIC, VARCHAR, VARIANT and more. But we can also talk about the structure of the files that hold the data. In this workshop we will always say Data Structure Types when we mean the file storage structures for data that can be referred to as Structured, Semi-Structured, and Unstructured.  

Structured data in a file (like a .txt or .csv) is arranged as a series of rows (often each line in the file is a row). Each row in the file is separated into columns. The value used to separate each row of data into columns can vary. Sometimes a comma is used to separate the values in each row into columns. Sometimes a tab is used. You can have pipes (sometimes called "vertical bars") as column delimiters, or many other symbols.  Regardless of which separator is used, any data file with data arranged in rows and columns (and no nesting) is called "structured data" and we often load each value into a column and each line into a row in our Snowflake tables.  

Semi-Structured data (in a file like .json, or .xml)  is data that has unpredictable levels of nesting and often each "row" of data can vary widely in the information it contains. Semi-structured data stored in a flat file may have markup using angle brackets (like XML) or curly brackets (like JSON). Snowflake can load and process common nested data types like JSON, XML, Avro, Parquet, and ORC. Each of the semi-structured data types is loaded into Snowflake tables using a column data type called VARIANT.  A VARIANT column might contain many key-value pairs and multiple nested records and it will also keep the markup symbols like curly braces and angle brackets.  NOTE: A file can be semi-structured, even if the file's name ends in .txt. If the data inside a file is formatted using semi-structured layout and markup, the file is considered semi-structured.  

There is a third type of data file called Unstructured data. (File names will have extensions like .mp3, .mp4, .png, etc.)  Snowflake added support for Unstructured Data in August of 2021. Snowflake has ways to help you store, access, process, manage, govern, and share unstructured data. Videos, images, audio files, pdfs and other file types are considered unstructured data.  

Snowflake's goal is to make it very easy for us to combine all three types of data so that we can analyze and process it together rather than needing to use multiple tools to do our data work.  

We can view the stages we create to access data in external storages differently now that we will use the data lake aspects. As it turns out, a Snowflake Stage Object can be used to connect to and access files and data you never intend to load!!!  
We can create a stage that points to an external bucket. Then, instead of pulling data through the stage into a table, we can reach through the stage to access and analyze the data where it is already sitting.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2eb9e0d4-23d0-4725-aa03-2be9bea47d26)  

**We can access it AND if we use a File Format, we can make it appear almost as if it is actually loaded!**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9cf663b1-dd74-4f22-aef3-df6d93edc657)  

We will need just to specify the **File Format** so we can read the data as it should.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f117fdb3-a69d-425d-a877-1a1a19c96d7c)  

In this example, snowflake hasn't been told anything about how the data in these files is structured so it's just making assumptions.  Snowflake is presuming that the files are CSVs because CSVs are a very popular file-formatting choice. It's also presuming each row ends with CRLF (Carriage Return Line Feed) because CRLF is also very common as a row delimiter.  
Example of typical csv file (with a semicolumn instead of comma):  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/eaa30691-37c6-4adc-8c1a-1da5b801e2c5)  

We can also fix some problems in the data like the presence of spaces at the end and the beginning using **TRIM_SPACE = TRUE**. **But it should be real spaces.**  
![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/99a8f23c-fc9f-4f26-aca7-66bcc40b9498)  

In fact, many data files use CRLF (Carriage Return Line Feed) as the record delimiter, so if a different record delimiter is used, the **CRLF** can end up displayed or loaded! When strange characters appear in our data, we can refine your select statement to deal with them.  
In SQL we can use ASCII references to deal with these characters:  
 -13 is the ASCII for Carriage return
 -10 is the ASCII for Line Feed  
 
SQL has a function, CHR() that will allow you to reference ASCII characters by their numbers. So, chr(13) is the same as the Carriage Return character and chr(10) is the same as the Line Feed character.  

In Snowflake, we can CONCATENATE two values by putting || between them (a double pipe). So we can look for CRLF by telling Snowflake to look for: **chr(13)||chr(10).**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/fd7792e3-e1bd-4399-9071-14a5ad5ec76d)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/d85c22e3-6617-4f34-83a3-ea2a87a3db03)  

Also to **Remove empty rows from the select we use:**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/857aeb33-c441-4ebe-b45f-25a7b3429f15)  

Now that we fixes the data, we can create a view from the select so that we can query it like a normal table:  

```
                          create view zenas_athleisure_db.products.sweatsuit_sizes as 
                          select REPLACE($1,chr(13)||chr(10)) as sizes_available
                          from @uni_klaus_zmd/sweatsuit_sizes.txt
                          (file_format => zmd_file_format_1 )
                          Where sizes_available <> '';
                          
                          select * from zenas_athleisure_db.products.sweatsuit_sizes;
```

**Thie is the power of snowflake when dealing with big data in general and flat files. We can create a view to read the data as a table and query this view.**  

#### Query Unstructured data:  

Querying images for example is possible in snowflake. But what can we query exactlly ?  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2cfaf4da-9f28-4b92-805a-5f5d2eacf6af)  

we can query the meta data:  

```
                          select metadata$filename, metadata$file_row_number
                          from @uni_klaus_clothing/90s_tracksuit.png; -- here we read metadata of a single file we can remove it and read all files
```
We can query all the files metadatas and count the number per file:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1a12b18b-70ed-4b71-95cd-e58cbe394a09)  

#### Directory tables:  

We can use the directory tables to read the unstructured data (such as images).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/62e3132a-94ee-43be-aa62-4c40105d2f28)  

Some few rules of directroy tables:  
- They are attached to a Stage (internal or external).  
- You have to enable them. 
- You have to refresh them.

```
                          --Directory Tables
                          select * from directory(@uni_klaus_clothing);
                          
                          -- Oh Yeah! We have to turn them on, first
                          alter stage uni_klaus_clothing 
                          set directory = (enable = true);
                          
                          --Now?
                          select * from directory(@uni_klaus_clothing);
                          
                          --Oh Yeah! Then we have to refresh the directory table!
                          alter stage uni_klaus_clothing refresh;
                          
                          --Now?
                          select * from directory(@uni_klaus_clothing);
```

Now we can see some new columns including the **FILE_URL:**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1a1663e5-0907-461f-b1ac-23bb749c5d33)  

**Now from this directory table results, we can run some queries to modify data and create a view on top of it to be used as a table!**  

Example of some changes to make is to standerdize the file name:  

```
--Nested functions to trqnsform data
                          select REPLACE(REPLACE(REPLACE(UPPER(RELATIVE_PATH),'/'),'_',' '),'.PNG') as PRODUCT_NAME
                          from directory(@uni_klaus_clothing);
```

We can also use them in joins:  

```
                         select * from
                         (select REPLACE(REPLACE(REPLACE(UPPER(RELATIVE_PATH),'/'),'_',' '),'.PNG') as PRODUCT_NAME, SIZE, LAST_MODIFIED, MD5, ETAG, FILE_URL, RELATIVE_PATH
                         from directory(@uni_klaus_clothing)) dt 
                         join
                         ZENAS_ATHLEISURE_DB.PRODUCTS.SWEATSUITS sw
                         on dt.RELATIVE_PATH = SUBSTR(sw.DIRECT_URL,54,50);
```
This is the power of combining structured data and unstructured data, as we can join loaded ones and non loaded ones.  When some data is loaded and some is left in a non-loaded state the two types can be joined and queried together, this is sometimes referred to as a **Data Lakehouse**.  

The Data Lake metaphor was introduced to the world in 2011 by James Dixon, who was the Chief Technology Officer for a company called Pentaho, at that time.  

Dixon said:  

**If you think of a data mart as a store of bottled water -- cleansed and packaged and structured for easy consumption -- the data lake is a large body of water in a more natural state. The contents of the data lake stream in from a source to fill the lake, and various users of the lake can come to examine, dive in, or take samples.**  

When we talk about Data Lakes at Snowflake, we tend to mean data that has not been loaded into traditional Snowflake tables. We might also call these traditional tables "native" Snowflake tables, or "regular" tables.  

**Create a view from parquet files:**  

```
                              create view CHERRY_CREEK_TRAIL as 
                              select 
                               $1:sequence_1 as point_id,
                               $1:trail_name::varchar as trail_name,
                               $1:latitude::number(11,8) as lng, --remember we did a gut check on this data
                               $1:longitude::number(11,8) as lat
                              from @trails_parquet
                              (file_format => ff_parquet)
                              order by point_id;



                              --To add a column, we have to replace the entire view
                              --changes to the original are shown in red
                              create or replace view cherry_creek_trail as
                              select 
                               $1:sequence_1 as point_id,
                               $1:trail_name::varchar as trail_name,
                               $1:latitude::number(11,8) as lng,
                               $1:longitude::number(11,8) as lat,
                               lng||' '||lat as coord_pair
                              from @trails_parquet
                              (file_format => ff_parquet)
                              order by point_id;


                             select 
                             'LINESTRING('||
                             listagg(coord_pair, ',')  -- window function
                             within group (order by point_id)
                             ||')' as my_linestring
                             from cherry_creek_trail
                             where point_id <= 10
                             group by trail_name;


                            -- create a view that queries data from json without loading it
                            create view DENVER_AREA_TRAILS as
                            select
                            $1:features[0]:properties:Name::string as feature_name
                            ,$1:features[0]:geometry:coordinates::string as feature_coordinates
                            ,$1:features[0]:geometry::string as geometry
                            ,$1:features[0]:properties::string as feature_properties
                            ,$1:crs:properties:name::string as specs
                            ,$1 as whole_object
                            from @trails_geojson (file_format => ff_json);
```

**We see that we can do every operation we want on data lake files just as the regular tables with loaded data.**   

#### Geospatial Functions:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5724076e-d13a-4d8a-9af2-028f72c41fec)  

When working with geospatial data, we need to convert them to GeaoSpatial, otherwise the spatial type functions won't work:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/98931947-a98e-4ef2-beaf-88e0a4b3e885)  

```
                           select 
                           'LINESTRING('||
                           listagg(coord_pair, ',') -- window function
                           within group (order by point_id)
                           ||')' as my_linestring
                           ,st_length(TO_GEOGRAPHY(my_linestring) ) as length_of_trail --this line is new! but it won't work!
                           from cherry_creek_trail
                           group by trail_name;
```

**to get the DDL code of a view or an object we can use the following statement:**  

```
                           select get_ddl('view', 'DENVER_AREA_TRAILS');
```

```
                          create or replace view DENVER_AREA_TRAILS(
                          	FEATURE_NAME,
                          	FEATURE_COORDINATES,
                          	GEOMETRY,
                              TRAIL_LENGTH,
                          	FEATURE_PROPERTIES,
                          	SPECS,
                          	WHOLE_OBJECT
                          ) as
                          select
                          $1:features[0]:properties:Name::string as feature_name
                          ,$1:features[0]:geometry:coordinates::string as feature_coordinates
                          ,$1:features[0]:geometry::string as geometry
                          ,st_length(to_geography(geometry)) as trail_length
                          ,$1:features[0]:properties::string as feature_properties
                          ,$1:crs:properties:name::string as specs
                          ,$1 as whole_object
                          from @trails_geojson (file_format => ff_json);
```

We have layered structure into our queries using file formats and views. We're not saying this is a great way to engineer data, we're just dicovering all the tools in the leave-it-where-it-lands toolbox. But, Depending on the project team, and the setting, and the project deadline - these no-loading tools might save you from spending critical time on the wrong tasks.  

#### Search for a location or trail:  

Use one of the following tools with geaospatial data:  
```
                           GOOGLE MAPS: 39.76471253574085, -104.97300245114094
                           
                           WKT PLAYGROUND: POINT(-104.9730024511  39.76471253574)
                           
                           GEOJSON.IO: Paste between the square brackets. 
                           
                           {
                                 "type": "Feature",
                                "properties": {
                                   "marker-color": "#ee9bdc",
                                  "marker-size": "medium",
                                   "marker-symbol": "cafe",
                                   "name": "Melanie's Cafe"
                                },
                                "geometry": {
                                   "type": "Point",
                                  "coordinates": [
                                     -104.97300870716572,
                                     39.76469906695095
                                   ]
                                 }
                               }
```

We can also use snowflake functions to generate coordinates from longitude and latitude:  

```
                          -- Melanie's Location into a 2 Variables (mc for melanies cafe)
                          set mc_lat='-104.97300245114094';
                          set mc_lng='39.76471253574085';
                          
                          --Confluence Park into a Variable (loc for location)
                          set loc_lat='-105.00840763333615'; 
                          set loc_lng='39.754141917497826';
                          
                          --Test your variables to see if they work with the Makepoint function
                          select st_makepoint($mc_lat,$mc_lng) as melanies_cafe_point;
                          select st_makepoint($loc_lat,$loc_lng) as confluent_park_point;
                          
                          --use the variables to calculate the distance from 
                          --Melanie's Cafe to Confluent Park
                          select st_distance(
                                  st_makepoint($mc_lat,$mc_lng)
                                  ,st_makepoint($loc_lat,$loc_lng)
                                  ) as mc_to_cp;
```

**Make a select of all columns and modify few of them:**  

```
                          SELECT
                           name
                           ,cuisine
                           ,distance_to_mc(coordinates) AS distance_from_melanies
                           ,*    --refere to all the columns, so for example name and cuisine will be returned twice
                          FROM  competition
                          ORDER by distance_from_melanies;
```

#### Function overloading:  

First we had a function called DISTANCE_TO_MC and it had two arguments. Then, we ran a CREATE OR REPLACE statement that defined the DISTANCE_TO_MC UDF so that it had just one argument. Maybe we expected only one function called DISTANCE_TO_MC would exist after that. But we look in our LOCATIONS Schema under FUNCTIONS and we you find that there are two!  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c7d9f569-18c4-483a-99ff-8cab4f1970d1)  

**it is what we call function overloading. it means that we can have different ways of running the same function and Snowflake will figure out which way to run the UDF, based on what we send it. So if we send the UDF two numbers it will run our first version of the function and if we pass it one geography point, it will run the second version.**  
**This means we can run the function several different ways and they will all result in the same answer.  When speaking about a FUNCTION plus its ARGUMENTS we can refer to it as the FUNCTION SIGNATURE.**  

#### Materialized views:  

A Materialized View is like a view that is frozen in place (more or less looks and acts like a table).  
**The big difference is that, for regular views, if some part of the underlying data changes,  Snowflake recognizes the need to refresh it, automatically.**  
**People often choose to create a materialized view if they have a view with intensive logic that they query often but that does NOT change often.  We can't use a Materialized view on any of our trails data because you can't put a materialized view directly on top of staged data.**  

#### External Tables:  

An External Table is a table put over the top of non-loaded data (just like the views we put on top a select statements from external stages).  
**External tables allow you to easily load data into Snowflake from various external data sources without the need to first stage the data within Snowflake. Data Integration: Snowflake supports seamless integration with other data processing systems and data lakes.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/7d299981-ca4e-46db-8416-77e6ee3b6f5d)  

There are other parts that are somewhat new, but that don't seem complicated. In our views we define the PATH and CAST first and then assign a name by saying AS <column name>. For the external table we just flip the order. State the column name first, then AS, then the PATH and CAST column definition.  
Also, there's a property called AUTO_REFRESH -- which seems self-explanatory!  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f21317f4-8e5f-4576-a02e-8407345b466e)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/58bacc08-624d-4b86-8040-090aefea1260)  

**Building an external table will load data in snowflake and we will be able to create a materialized view on top of it.**  
**In other words, you CAN put a Materialized View over staged data, as long as you put an External Table in between them, first!**  

```
                              create or replace external table T_CHERRY_CREEK_TRAIL(
                              	my_filename varchar(50) as (metadata$filename::varchar(50))
                              ) 
                              location= @trails_parquet
                              auto_refresh = true
                              file_format = (type = parquet);
```

We can use get ddl to have all the properties needed in create external table statement.  

Difference in syntax between view and external table:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/0abb0298-d085-4037-9da8-7b7abc4dd280)  

#### Iceberg tables:  
Iceberg tables are based on the iceberg technologie, which is an apache technology.  

Iceberg is a high-performance format for huge analytic tables. Iceberg brings the reliability and simplicity of SQL tables to big data, while making it possible for engines like Spark, Trino, Flink, Presto, Hive and Impala to safely work with the same tables, at the same time.  

- Iceberg is an open-source table type, which means a private company does not own the technology. Iceberg Table technology is not proprietary.  
- Iceberg Tables are a layer of functionality you can lay on top of parquet files (just like the Cherry Creek Trails file we've been using) that will make files behave more like loaded data. In this way, it's like a file format, but also MUCH more.   
- Iceberg Table data will be editable via Snowflake! Read that again. Not just the tables are editable (like the table name), but the data they make available (like the data values in columns and rows). So, you will be able to create an Iceberg Table in Snowflake, on top of a set of parquet files that have NOT BEEN LOADED into Snowflake, and then run INSERT and UPDATE statements on the data using SQL 🤯.   
Iceberg Tables will make Snowflake's Data Lake options incredibly powerful!!  

https://www.snowflake.com/blog/5-reasons-apache-iceberg/  

**In all the above, we have seen that using non loaded data is of a great advantage in prototyping and descovering data before before normalizing it and loading it in a real data warehouse.**  


## 10. Data Engineering:  

In this part, we discover all the functionalities related to data engineering.  

Note that when we want to use **COPY INTO** to load several files (having the same structure) we don't specify the file's name in **FROM** clause.  

```
                                  copy into AGS_GAME_AUDIENCE.RAW.GAME_LOGS
                                  from @uni_kishore/kickoff     -- we specify only the folder,  then all the files in it will be loaded
                                  file_format = (format_name = FF_JSON_LOGS)
                                  ;
```

#### TimeZone:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/bfb8e69a-3a19-419e-a5c1-442092ca53f2)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4a73bd27-7272-4432-a10e-24e51c459d45)  

Even while our trial account has a default time zone of "America/Los_Angeles" (UTC-7), we can change the time zone of each worksheet, independently.  
A worksheet can be referred to as a "session". For the purposes of this lesson, just consider a "worksheet" and a "session" to mean the same thing*.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4a0a6e47-e5e5-440f-9911-8ed70b58d7c0)  

To alter the timezone of a session we can use the following queries:  

```
                             --what time zone is your account(and/or session) currently set to? Is it -0700?
                             select current_timestamp();
                             
                             --worksheets are sometimes called sessions -- we'll be changing the worksheet time zone
                             alter session set timezone = 'UTC';
                             select current_timestamp();
                             
                             --how did the time differ after changing the time zone for the worksheet?
                             alter session set timezone = 'Africa/Nairobi';
                             select current_timestamp();
                             
                             alter session set timezone = 'Pacific/Funafuti';
                             select current_timestamp();
                             
                             alter session set timezone = 'Asia/Shanghai';
                             select current_timestamp();
                             
                             --show the account parameter called timezone
                             show parameters like 'timezone';
```

Snowflake uses the IANA list. We can see it here: https://data.iana.org/time-zones/tzdb-2021a/zone1970.tab  

Know snowflake stores and display datetime data differently:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/92e64bbf-89f1-4d6a-a651-4cd5dfc050d3)  

The Z represents Zulu...as in Greenwich Mean Time...as in UTC 0.  But it could just mean "time zone unknown."  In other words, the Z either tells you a lot, or very little.  
For example, it could be telling you that when game players logged in and out:  
Their datetime info was captured in the players' local time, but the time zone info was lost along the way.  
Their datetime info was captured in the players' local time, but the data was converted to UTC before being made available to Agnie.  
The game captured the datetime info in the server's default time, but the server's time zone information was lost along the way.  
The game captured the datetime info in the server's default time, but the data was converted to UTC before being made available to Agnie.  
Some other capture and convert/loss scenario.   
In short, "Z" can mean we know the time zone and it is in a standardized, zero-offset form OR it can mean we don't know what the original time zone was.  
It can also sometimes mean that the time zone is stored in a separate column and you are expected to combine the two values when you compare two different timestamps.  

**The info about how the format of the date data is captured is crucial to do time comparaisions.**  

The **epoch** format is also a well know format of storing date data: The Unix epoch (or Unix time or POSIX time or Unix timestamp) is the number of seconds that have elapsed since January 1, 1970 (midnight UTC/GMT), not counting leap seconds (in ISO 8601: 1970-01-01T00:00:00Z).  

#### Schema on read:

We have been using this from the begining using the FILE_FORMAT to read data in VARIANT type. We store it in VARIANT column and on the read we define a schema like follows:  

```
                         create or replace view AGS_GAME_AUDIENCE.RAW.LOGS(
                             IP_ADDRESS,
                         	DATETIME_ISO8601,
                         	USER_EVENT,
                         	USER_LOGIN,
                         	RAW_LOG
                         ) as (
                         select
                         RAW_LOG:ip_address::VARCHAR as IP_ADDRESS
                         ,RAW_LOG:datetime_iso8601::TIMESTAMP_NTZ as datetime_iso8601
                         , RAW_LOG:user_event::VARCHAR as USER_EVENT
                         , RAW_LOG:user_login::VARCHAR as USER_LOGIN
                         ,*
                         from GAME_LOGS
                         where RAW_LOG:agent::text is null
                         );
```

#### ETL & ELT:

In many organizations, a Data Engineer is given access to extracted data, and told what the end goals are (the final, transformed state). Then, it is within their power and discretion to decide what steps they will follow to get there.  
We have seen above an example of ELT when we used the view to parse the variant column of the original table.  

These choices are called **design** and the structures and processes that result are called the **architecture**.  
Data Engineers often perform a series of ETL steps and so they have different "layers" where data is expected to have reached certain levels of refinement or transformation. In this workshop we'll have named our layers: 
    - RAW
    - ENHANCED
    - CURATED

If we have IP address and we want to get all the infos regarding it we can use a snowflake function **parse_ip**  

```
                       select parse_ip('107.217.231.17','inet'):ipv4;

                      -- Result :
                      {
                       "family": 4,
                       "host": "107.217.231.17",
                       "ip_fields": [
                         1809442577,
                         0,
                         0,
                         0
                       ],
                       "ip_type": "inet",
                       "ipv4": 1809442577,
                       "netmask_prefix_length": null,
                       "snowflake$type": "ip_address"
                     }
```

#### Query profiling:

**Query profiling is so important in order to evalaute if the query is optimized or not as this will impact the computing cost we pay in snowflake.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/c1af9a7e-c2ad-47ce-b71f-e71956dc9122)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9b329daa-bd05-4719-b40a-75e78df61e82)  

If we forget to look at the Query Profile while the results are still on our screen, we just use the side menu to find and explore previous queries.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/59b556c4-c1b0-40fe-be24-ff7262fd6271)  

**Bare in mind that joins on integers are more efficient then the strings.**  

```
                            --Use two functions supplied by IPShare to help with an efficient IP Lookup Process!
                            SELECT logs.ip_address
                            , logs.user_login
                            , logs.user_event
                            , logs.datetime_iso8601
                            , city
                            , region
                            , country
                            , timezone 
                            from AGS_GAME_AUDIENCE.RAW.LOGS logs
                            JOIN IPINFO_GEOLOC.demo.location loc 
                            ON IPINFO_GEOLOC.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key
                            AND IPINFO_GEOLOC.public.TO_INT(logs.ip_address) 
                            BETWEEN start_ip_int AND end_ip_int;
```

Wecanuse a lot of functions in snowflake to convert to local time zones:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/40ea9f8b-b047-412a-9ba7-bbf4f6aa4543)  

**Combining list aggregate function and group by makes it possible to return all the values of the same observation in the group by column:**  

```
                           select tod_name, listagg(hour,',') 
                           from time_of_day_lu
                           group by tod_name;
```

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/f8a1e322-061d-4ce3-bed0-acca01ae9ff3)  

As we do with views, if we want to create a table from a selection we can do that using **Create Table As Select:**  

```
                            create table ags_game_audience.enhanced.logs_enhanced as(
                            SELECT logs.ip_address
                            , logs.user_login as GAMER_NAME
                            , logs.user_event as GAME_EVENT_NAME
                            , logs.datetime_iso8601 as GAME_EVENT_UTC
                            , city
                            , region
                            , country
                            , timezone as GAMER_LTZ_NAME
                            , CONVERT_TIMEZONE('UTC',timezone,logs.datetime_iso8601) as GAME_EVENT_LTZ
                            , DAYNAME(GAME_EVENT_LTZ) as DOW_NAME
                            , TOD_NAME
                            from AGS_GAME_AUDIENCE.RAW.LOGS logs
                            JOIN IPINFO_GEOLOC.demo.location loc 
                            ON IPINFO_GEOLOC.public.TO_JOIN_KEY(logs.ip_address) = loc.join_key
                            AND IPINFO_GEOLOC.public.TO_INT(logs.ip_address) 
                            BETWEEN start_ip_int AND end_ip_int
                            JOIN AGS_GAME_AUDIENCE.RAW.TIME_OF_DAY_LU
                            ON AGS_GAME_AUDIENCE.RAW.TIME_OF_DAY_LU.HOUR = date_part(HOUR,logs.datetime_iso8601)
                            );
```

#### Productionizing the work:

This part is simply the automatization of the ETL/ELT process using tasks, jobs, pipelines and stored procedures ...   
**Create Task:**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4c322598-1535-4690-9a19-a6a771db318c)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4446fe99-26ac-44d5-8a68-ee2339c4bb2e)  




