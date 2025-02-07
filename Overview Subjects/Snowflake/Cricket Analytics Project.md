# Cricket analytics project

## This project was a POC created as a demo to be presented to clients willing to migrate their old data warehouses to snowflake lakehouse architecture !

We describe in this file all the steps as well as the production best practices when working on similar projects !  

### Stack of the project:

- AZURE Storage account
- Snowflake
- VS Studio
  
### About the data
 The data correspond to the men's cricket world cup results of the season 2023/2024.  
 Data for each match is in a json file, stored in an azure blob storage.  

### Architecture of the project :
Here is a simple diagram that shows the structure of the project :  

![{AB56B11B-C8B0-4C54-A6FE-FE465B9089FE}](https://github.com/user-attachments/assets/52843ab8-c360-4bf0-9a1a-1cf0bc95607b)  

**Note that the API part is not covered in the project. This could be done using a python code inside ADF and then ADF will copy the json files into the Blob Storage.**  
After loading files in the blob storage (in production case it is recommanded to use ADLS Gen 2 to benifit from the hierarchy if needed), we connect snowflake to the azure storage using external stage.  
With a files format, we will be able to parse the json data and then load it into Snowflake.  

#### Medallion achitecture: 

Following the medallion architecture of lakehouses, we created 4 schemas (the first one is optional as we could directly read from the external stage) : 
- Landing area : it is simply a schema that will hold a table containing all the json files as rows. This area is aptional!
- Raw area (Bronz) : this area is all about parsing the json files and flatten them with no additional transformations. This area is important as it keeps records of the original data in case needed.
- Clean area (Silver) : an area containing cleaned data that could be used by other data engineers or data scientists and apply on it more transformations.
- Consumption area (Gold) : this is the real data warehouse as we classicaly now it with dimentions and fact tables.
- Business area :  we can add another layer as views that the business teams could use to build their dashboards. This layer will contain the business names of columns and tables and will be linked to the gold area that has technical names that are harder to use and understand by the business teams.

First of all we create our Database that will hold all the objects :

``` SQL
          create database if not exists cricket;
          create or replace schema land;
          create or replace schema raw;
          create or replace schema clean;
          create or replace schema consumption;
```
By default when we create a database, two schemas are created : Public anf Information schema that holds all the metadata of our database. Generaly, we use public schema to hold all the objects that are not specifc to any of the other medallion architecture schemas:  

![{7BAFE518-29DE-4048-91EF-EF8502EA7891}](https://github.com/user-attachments/assets/136a9da8-cc7f-46fb-8aa5-d7d121f30e00)  

Now we need to connect to the data stored in the Azure Blob Storage.  

### Data Loading :
Now we need to create other objects that will help loading the data into snowflake: 

#### Connecting snowflake to external data:

Similarly to synapse, when connecting to external data we need create external data source or in this case external stage !  but to connect, we need also to create a **Storage integration** that will authenticate to the source like ADLS gen2 (This is what should be done in the production scenario) or we can use directly SAS token (for test purposes) : 

![{D8B3BEB9-0989-476A-928C-D96009A78328}](https://github.com/user-attachments/assets/4098ac76-a196-4761-83b0-13d32145f160)  

![{5C9BF266-23C3-4E44-9D65-C262BB0F7373}](https://github.com/user-attachments/assets/76ed495d-a729-4276-9cf7-f891c11e8e75)  

to create the storage integration : 

```SQL
                    create storage integration <storage_integration_name>
                        type = external_stage
                        storage_provider = azure
                        azure_tenant_id = '<azure_tenant_id>'
                        enabled = true
                        storage_allowed_locations = ( 'azure://<location1>', 'azure://<location2>' )
                        -- storage_blocked_locations = ( 'azure://<location1>', 'azure://<location2>' )
                        -- comment = '<comment>';
```

**The storage integration will need an app registration at azure level that will create a principal service for snowflake to allo it to read data from the blob storage.**  

In our case and since it is a POC we used SAS token that we generated at the blob container level in AZURE:  

![{98A91B57-2778-4F0B-B9FD-78C1056DDE57}](https://github.com/user-attachments/assets/84e86dcb-3ed5-46ba-b72d-88ab0dd3191c)  

**Note that the SAS token has to have the not only the READ permission but also the LIST permission as snowflake requests it to list the files in the container !**  

#### Read data from the external storage : 

Now that we are connected to the data in azure using the extarnal storage, we will need a **File Format** to parse the data. In our case the file format has a json type:

![{41E2DFBA-7B99-4B07-AE98-408CB697C225}](https://github.com/user-attachments/assets/84b4b301-a50f-4c88-b600-217a92b9592a)  

```SQL
                    USE SCHEMA PUBLIC;
                    CREATE OR REPLACE FILE FORMAT cricket.land.json_ff
                        type = json
                        null_if = ('\\n','null','')
                        strip_outer_array = true 
                        comment = 'json file format'; 
                    
                    -- let's try it out with a select !
                    
                    select $1 from @CRICKET_JSON_FILES_CONTAINER_ONLY
                        ( FILE_FORMAT => cricket.land.json_ff);
```

Now we can explore our data in snowflake even before laoding it. Before doing that we can see the structure of the json file in VSCode to understand it's structure : 

![{60DE73B2-4CD3-433D-B63D-91A83B1685C3}](https://github.com/user-attachments/assets/5b048b4c-b2eb-4aea-9761-32b00f9db84a)  

We can see that the file starts with some metadata regarding the file.  

![{7D453BA5-1827-44B7-BFAF-F36D93566851}](https://github.com/user-attachments/assets/9b0d8ad0-e956-455b-890e-7f1cd5c8c559)  

Then we have "info" that groups all the qualitative data such as the event name, categorie, teams players ...  

Then we have the "innings" which groups all the quantitative data regarding number of runs, scoring ...  

![{63890039-4C9A-47DB-B9EB-84B6B4C3634F}](https://github.com/user-attachments/assets/b44a4921-f637-44c4-a7a7-2219ec4f6e94)  

We can query the file and see the columns in a propert wat using the file format in snowflake as follows :  

```SQL
                  select 
                        t.$1:meta::variant as meta, 
                        t.$1:info::variant as info, 
                        t.$1:innings::array as innings, 
                        metadata$filename as file_name,
                        metadata$file_row_number int,
                        metadata$file_content_key text,
                        metadata$file_last_modified stg_modified_ts
                     from  @CRICKET_JSON_FILES_CONTAINER_ONLY/1384412.json
                     (file_format => 'cricket.land.json_ff') t;    
```

![{017E7732-D127-4650-A468-795CADCE4BAB}](https://github.com/user-attachments/assets/8e6170f2-e59a-42e6-a8a6-c55618e44a56)  

Now that we explored our data we need to load in our land schema. In our case data is already an external storage so we will use **COPY INTO** directly. But imagine another scenario where data is in a local storage such as a on premise server ?!

#### Scenario of local data (on premise server):  

We can do this using **snowsight (the UI) but it doesn't support bulk loading** so we will need the **SNOWSQL CLI**  

After installing it we run our terminal and we verify that it recognises the CLI :

![image](https://github.com/user-attachments/assets/8309075a-5958-4794-a4ed-6f644d46cb44)  

In order to connect to our snowflake account we will need to update the SnowSql CLI config file :  

![image](https://github.com/user-attachments/assets/5247f021-ab37-41a1-9771-ec3f74e116a3)  

Here we will specify in **plain text** all the parameters such as password, user name, account and so on.  

**Note that if the account is located in the west region (oregon), the account_name could be the locator and no need to specify the region.**  

NOw when we run **snowsql** we can start writing queries :  

![image](https://github.com/user-attachments/assets/3b487e08-b6ec-45fc-8d93-a67fd9287f49)  

We make sure that we are using the right settings for our queries :  

![image](https://github.com/user-attachments/assets/9fc0c5fc-18e1-43c3-acb5-08c1341de0cf)  

To do a bulk load we need to use a special command in snowsql CLI called : **PULL**. Documentation [here](https://docs.snowflake.com/en/sql-reference/sql/put)  

**Note that SnowSql is very useful for automating scripts !**  

#### Loading data into raw layer : 

Our data is well connected to our snowflake using the external stage, we can start populating our medallion architecture with data.  

First of all we will create a table in the raw layer taht will hold all the json files with some metadata columns. This is very useful when tracking data to the source in case of an error !  

```sql

              USE SCHEMA raw;
              USE ROLE SYSADMIN;
              USE WAREHOUSE COMPUTE_WH;
              
              CREATE OR REPLACE transient TABLE match_raw_table (
                  meta object not null,
                  info variant not null,
                  innings array not null,
                  file_name text not null,
                  file_row_number int not null,
                  file_hashkey text not null,
                  modified_ts timestamp not null
              )
              comment = 'this table holds the original data files with all its metadata';

```

To have a clear vision of the json structure we can use json cracker in VSCode to visualize it as a digram which will help deciding on the columns types :  

![{77FDDE5C-08E2-48A7-A19A-1AD2C09A876E}](https://github.com/user-attachments/assets/4370c897-dc9e-4b1c-90c7-bfe34205dd7a)  

We can see that "Meta" is a dictionary which can correspond to an OBJECT type in snowflake !  
The "INFO" however contains a hierarchy so we can assing the VARIANT type to this column !  
For the "INNINGS" it is clearly an array so we can assing the type ARRAY to it !  

![{BB9E6A40-A98B-4111-90A9-DA26AF49DB4D}](https://github.com/user-attachments/assets/4503f15c-bf72-4b3b-8b1c-a57d7a9ff459)  

Let's copy data into our table :  

```SQL
                    COPY INTO CRICKET.RAW.MATCH_RAW_TABLE
                    FROM (
                        SELECT 
                            t.$1:meta::object AS meta, 
                            t.$1:info::variant AS info, 
                            t.$1:innings::array AS innings, 
                            --
                            metadata$filename,
                            metadata$file_row_number,
                            metadata$file_content_key,
                            metadata$file_last_modified
                        FROM @CRICKET_JSON_FILES_CONTAINER_ONLY
                        (FILE_FORMAT => 'cricket.land.json_ff') t
                        )
                    ON_ERROR = continue
                    ;
```
Note that the **"metadata$--"** is a built in function that retrievse metadata of the objects in the stage !  

![{0CCBF0C4-4184-457A-BDED-D5B367CD5B46}](https://github.com/user-attachments/assets/93344ca6-e038-4abf-bfb0-d937cbbf2050)  

We can also quickly check if we have errors on some of the files using the UI :  

![{CEE8E566-C83B-4E99-94FD-EA2646384B10}](https://github.com/user-attachments/assets/4bcab1e8-bd37-4693-b1ff-efafe1c74663)  

Everything is green ! means all is good !  

If we select the content of the table :  

![{596BE2C9-D63C-4741-8BB5-BBB2F4FFF86E}](https://github.com/user-attachments/assets/edd1992d-f75e-40fd-afa4-df12e4d80316)  

![{88888EC5-D8CE-4F8E-8FB8-74C17ECFAC45}](https://github.com/user-attachments/assets/d3751918-64df-44eb-b58b-41d5fbda10d4)  

Now we are ready to clean data and especialy flatten it !  

#### Silver layer :  

In the silver layer we need to flatten data in the columns containing several objectes so that we can have a column for each object. For example from the meta column of the raw table we can extract 3 columns :  

##### Matchs details :
```SQL
            SELECT 
                META:data_version::text AS data_version,
                META:created::date AS created,
                META:revision::number as revision
            FROM
                CRICKET.RAW.MATCH_RAW_TABLE;

```
The :: operator is similar to CAST function !  

![{E276050E-7B27-4C46-B315-9757AFF686D6}](https://github.com/user-attachments/assets/eb59fc4c-5427-4fe2-8504-fb28909e37a0)  

For Meta things are simple, but for the info column in the raw table we need to understand the pattern as some things need to be dynamicaly retrieved !  

Let's tale a look on the json sample :  

![{B63C5AAA-F476-4596-B58F-30AB222782BF}](https://github.com/user-attachments/assets/32ec27ca-7f43-4544-9aec-3eabb9d96600)  

We can see that info has 10 elements including some data regarding the match, the dates, the events and so on but if we pay attention to the "players" element we can see that we have two teams :  

![{90A5CF81-372F-4A6F-A58B-80FEC16D5649}](https://github.com/user-attachments/assets/a02a8612-a3fc-404f-8a52-30bd2ed62927)  

Now, this will need a dynamic way to retrieve first the names of the two teams so that we can use that to retrieve the players using for example : {Root}.info.players.England[6] to retrieve a player !  
Luckly we have the "teams" element that list the two teams so we can use that in oue dynamic logic !  
We can query the first elements of INFO to see how data looks in all the files :  

```SQL
              USE SCHEMA RAW;
              USE ROLE SYSADMIN;
              USE WAREHOUSE COMPUTE_WH;
              
              SELECT
                  INFO:match_type_number::int as match_type_number,
                  INFO:match_type::text as match_type,
                  INFO:season::text as season,
                  INFO:team_type::text as team_type,
                  INFO:overs::int as overs,
                  INFO:city::text as city,
                  INFO:venue::text as venue
              FROM
                  CRICKET.RAW.MATCH_RAW_TABLE;
```

**We can see that these data can be later useful to create some dimensions for our datawarehouse !!**  

![{104F86E7-FC30-483B-8655-5F11D17E7380}](https://github.com/user-attachments/assets/07c24a44-aed8-4b51-b478-e8e85eeb2e99)  

This will facilitate the data exploring for us to take some decisions on how we will transform data : for example id we have null values what to do !  

After anlysing the data we can adopt the following transformations to create a silver table holding the details of the matchs :  

```SQL
                USE SCHEMA CLEAN;
                USE ROLE SYSADMIN;
                USE WAREHOUSE COMPUTE_WH;
                
                create or replace transient table cricket.clean.match_detail_clean as
                select
                    info:match_type_number::int as match_type_number, 
                    info:event.name::text as event_name,
                    case
                    when 
                        info:event.match_number::text is not null then info:event.match_number::text
                    when 
                        info:event.stage::text is not null then info:event.stage::text
                    else
                        'NA'
                    end as match_stage,   
                    info:dates[0]::date as event_date,
                    date_part('year',info:dates[0]::date) as event_year,
                    date_part('month',info:dates[0]::date) as event_month,
                    date_part('day',info:dates[0]::date) as event_day,
                    info:match_type::text as match_type,
                    info:season::text as season,
                    info:team_type::text as team_type,
                    info:overs::text as overs,
                    info:city::text as city,
                    info:venue::text as venue, 
                    info:gender::text as gender,
                    info:teams[0]::text as first_team,
                    info:teams[1]::text as second_team,
                    case 
                        when info:outcome.winner is not null then 'Result Declared'
                        when info:outcome.result = 'tie' then 'Tie'
                        when info:outcome.result = 'no result' then 'No Result'
                        else info:outcome.result
                    end as matach_result,
                    case 
                        when info:outcome.winner is not null then info:outcome.winner
                        else 'NA'
                    end as winner,   
                
                    info:toss.winner::text as toss_winner,
                    initcap(info:toss.decision::text) as toss_decision,
                    --
                    file_name ,
                    file_row_number,
                    file_hashkey,
                    modified_ts
                    from 
                    CRICKET.RAW.MATCH_RAW_TABLE;
```

**Note that here we used CTAS to create the table since the data is not huge, but in production, we need to first create the table with the schema desired then load the data usinf COPY INTO**  

##### Players data :

Let's try to retrieve players data (this will be a dimention later) from the raw table :  





















