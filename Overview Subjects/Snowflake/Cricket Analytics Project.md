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

Now that we explored our data we need to load in our land schema. We can do this usinf **snowsight (the UI) but it doesn't support bulk loading** so we will need the **SNOWSQL CLI**  

After installing it we run our terminal and we verify that it recognises the CLI :

![image](https://github.com/user-attachments/assets/8309075a-5958-4794-a4ed-6f644d46cb44)  

In order to connect to our snowflake account we will need to update the SnowSql CLI config file :  

![image](https://github.com/user-attachments/assets/5247f021-ab37-41a1-9771-ec3f74e116a3)  

Here we will specify in **plain text** all the parameters such as password, user name, account and so on.  

**Note that if the account is located in the west region (oregon), the account_name could be the locator and no need to specify the region.**  

NOw when we run **snowsql** we can start writing queries :  

![image](https://github.com/user-attachments/assets/3b487e08-b6ec-45fc-8d93-a67fd9287f49)  












