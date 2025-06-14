# Cricket analytics project

## This project was a POC created as a demo to be presented to clients willing to migrate their old data warehouses to snowflake lakehouse architecture !

We describe in this file all the steps as well as the production best practices when working on similar projects !  

### 1. Stack of the project:

- AZURE Storage account
- Snowflake
- VS Studio
  
### 2. About the data
 The data correspond to the men's cricket world cup results of the season 2023/2024.  
 Data for each match is in a json file, stored in an azure blob storage.  

### 3. Architecture of the project :
Here is a simple diagram that shows the structure of the project :  

![{AB56B11B-C8B0-4C54-A6FE-FE465B9089FE}](https://github.com/user-attachments/assets/52843ab8-c360-4bf0-9a1a-1cf0bc95607b)  

**Note that the API part is not covered in the project. This could be done using a python code inside ADF and then ADF will copy the json files into the Blob Storage.**  
After loading files in the blob storage (in production case it is recommanded to use ADLS Gen 2 to benifit from the hierarchy if needed), we connect snowflake to the azure storage using external stage.  
With a files format, we will be able to parse the json data and then load it into Snowflake.  

#### 3.1 Medallion achitecture: 

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

### 4. Data Loading :
Now we need to create other objects that will help loading the data into snowflake: 

#### 4.1 Connecting snowflake to external data:

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

#### 4.2 Read data from the external storage : 

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

#### 4.3 Scenario of local data (on premise server):  

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

#### 4.5 Loading data into raw layer : 

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

#### 4.6 Silver layer :  

In the silver layer we need to flatten data in the columns containing several objectes so that we can have a column for each object. For example from the meta column of the raw table we can extract 3 columns :  

##### 4.6.1 Matchs details :
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

##### 4.6.2 Players data :

Let's try to retrieve players data (this will be a dimention later) from the raw table.  

Two columns from the info side will interest us : Teams and players !  

```SQL
SELECT
    raw.INFO:teams,
    raw.INFO:players

FROM CRICKET.RAW.MATCH_RAW_TABLE raw; 
```

![{A19984FC-D91A-4104-9F4C-76F2D1A55178}](https://github.com/user-attachments/assets/0aac366d-8190-413c-aa34-d6e0874853c8)  

We can see that we can either use values in the teams array as keys to retrieve players from the players dictionnary or simply use only the players column and flatten it to retrieve the key and values (array) into rows. But we will need basicaly two columns after this flatten operation, one for the key (which is the country) and the other for the player name (which is the value in the array) ! ==> This is where the **LATERAL** function comes handy!  

```SQL
SELECT
    raw.INFO:match_type_number,
    raw.INFO:players,
    p.key as team,
    p.value as players

FROM CRICKET.RAW.MATCH_RAW_TABLE raw,
LATERAL FLATTEN(input => raw.INFO:players) p 
WHERE raw.INFO:match_type_number='4683'
;
```

The code above will return the following table :  

![{BA923D20-DF4A-4E66-897A-F3EFF06F3456}](https://github.com/user-attachments/assets/57532eb9-6e6d-47d3-97a3-dca536e531ad)  

It simply return for a single match the players lists for both teams  as well as two new columns : team and the the array of players of that team !  

**Note that the flatten on INFO:players gave two rows one per team since we have only two elements in the dictionary !**:  

![{EB63C0F3-24DD-4F80-BD1C-67FB9F14656E}](https://github.com/user-attachments/assets/1f004ea6-4b94-465d-a084-c63edad385eb)  

The lateral however made it possible to add the new column for teams and players !  

Now we can do the same to flatten the array of players and have one row per player !  

![{858AA460-F09A-4F64-903E-424C547D5002}](https://github.com/user-attachments/assets/255ebeb3-5a0d-4e12-8e53-f9d24a66e5c5)  

```SQL
SELECT
    raw.INFO:match_type_number,
    p.key as team,
    players.value as players

FROM CRICKET.RAW.MATCH_RAW_TABLE raw,
LATERAL FLATTEN(input => raw.INFO:players) p,
LATERAL FLATTEN(input => p.value) players
WHERE raw.INFO:match_type_number='4683'
;
```

using this we can create a table that will hold this structured data in the silver layer with some additional audit columns !  

![{884D01CF-D862-4E31-8658-45ED12B6D2D9}](https://github.com/user-attachments/assets/9e4d43e0-4b27-4258-b873-3edf5f6c819d)  

```SQL
SELECT
    raw.INFO:match_type_number,
    p.key as team,
    players.value as players,
    file_name ,
    file_row_number,
    file_hashkey,
    modified_ts

FROM CRICKET.RAW.MATCH_RAW_TABLE raw,
LATERAL FLATTEN(input => raw.INFO:players) p,
LATERAL FLATTEN(input => p.value) players
;
```

The final code :  

```SQL
CREATE OR REPLACE TABLE player_clean_tb AS
SELECT
    raw.INFO:match_type_number::int as match_type_number,
    p.key::text as team,
    players.value::text as player_name,
    file_name ,
    file_row_number,
    file_hashkey,
    modified_ts

FROM CRICKET.RAW.MATCH_RAW_TABLE raw,
LATERAL FLATTEN(input => raw.INFO:players) p,
LATERAL FLATTEN(input => p.value) players;
```

Let's add also a "Not null" constraint on the columns : 

```SQL
ALTER TABLE player_clean_tb
MODIFY COLUMN team set not null;

ALTER TABLE player_clean_tb
MODIFY COLUMN player_name set not null;

ALTER TABLE player_clean_tb
MODIFY COLUMN match_type_number set not null;
```

**Note that not null and check are the only constraints enforced for now! the others act like comments !**  

We can add other constraints for documentation puposes (since they are not enforced) such as the primary key !  

**Note that if want to maitain referential integrity we need to create a special script for that!**  

```SQL
USE SCHEMA clean;

ALTER TABLE clean.match_detail_clean
add constraint pk_match_type_number primary key (match_type_number);

ALTER TABLE clean.player_clean_tb
add constraint fk_match_id FOREIGN KEY  (match_type_number)
REFERENCES clean.match_detail_clean (match_type_number);

select get_ddl('table','clean.match_detail_clean');
```

![{087546FA-1745-4F5D-9B92-18F3EF72C3DA}](https://github.com/user-attachments/assets/f09e3bc6-18e5-4249-8f55-73c269d4a366)  

##### 4.6.3 Deliveries data: 

After cleaning data of what will be our dimensions later on, we can do the same for the deliveries inside the innings part of our json so that we can have out table of events of our fact table :  

![{CE63F58A-B5D3-40A8-B31C-ECD65A2292F7}](https://github.com/user-attachments/assets/994a141c-aeb1-4dd9-9cc3-db6011bc8e27)  

Inside the innings, we have for each team the details of the game and especialy the overs and deliveries that are the numeric values the iterest us. We will proceed with the same way we did for the previous data to flatten the deliveries ones.  

![{C6B78355-99A4-4970-B1BE-61E59ADA5435}](https://github.com/user-attachments/assets/47509590-a1ad-4ba3-9eee-66c1a930eef2)  

Let's see what the lateral flatten table of innings fot a single match contains : 

![{688FE344-61FC-4ADF-A69B-6F320585A82F}](https://github.com/user-attachments/assets/6f5830af-ff8a-4a5d-b5b2-8bd1e65df5fb)  

This shows the subcolumns we can retrieve from the flatten table. In our case we will use the value column to extract the data we want, for example the team : 

![{8C8BF2A5-8123-4C2D-8151-994B06D5B951}](https://github.com/user-attachments/assets/56b50a8d-5d24-44ca-a653-7c96892d9e15)  

We can also explode the overs to have an over per row :  

![{87085F57-5D74-4573-82CC-3851B14EAA4B}](https://github.com/user-attachments/assets/9582ad34-c317-4268-9d15-b2eb877105e3)  

![{409186D0-9CFD-40E0-BE84-0BC88BB8A924}](https://github.com/user-attachments/assets/4e61e98c-58b3-4e2f-bfba-b3b507d5a246)  

```SQL
select
    INFO:match_type_number::int as match,
    i.value:team::text as team,
    o.value overs,
from raw.match_raw_table m,
lateral flatten(input=> innings) i,
lateral flatten (input=>i.value:overs) o,
WHERE match = 4673;
```

We can see that 85 overs were played, 50 for New Zealand and 35 for Afghanistan. Now for each over we can explode data to have a row per ball played with some additional columns such as the batter, the bowler and so on.  

![{A06DD827-14A6-467D-AA2D-112E1011CB5D}](https://github.com/user-attachments/assets/4555ecda-c48d-42d2-98e9-c85774a94b72)  

We can see in the example that the example that for the first over, for the 6 balls playes 5 of them the batter was "DP Conway". So far the explode precess is going good. We can add now the other columns we need:  

```SQL
select
    INFO:match_type_number::int as match,
    i.value:team::text as team,
    o.value:over::int over,
    d.value:bowler::text bowler,
    d.value:batter::text batter,
    d.value:non_striker::text non_striker,
    d.value:runs::text runs,
    d.value:runs:extras::text extras,
    d.value:runs:total::text total,
from raw.match_raw_table m,
lateral flatten(input=> innings) i,
lateral flatten (input=>i.value:overs) o,
lateral flatten (input=>o.value:deliveries) d
WHERE match = 4673;
```

![{FCBCFB8E-F8D7-485F-A7C1-A3202D629E24}](https://github.com/user-attachments/assets/27e55e7d-5782-45ca-b8fa-9dddfa717c9c)  

We have also another case to take into concideration which is when we have extras. In that case we need to have the type and the value of that extra:  

![{58F7AAB7-30F8-4B4B-96AD-368331EAF44E}](https://github.com/user-attachments/assets/db4f843a-b735-41e6-80d5-8f89c362529c)  

```SQL
select
    INFO:match_type_number::int as match,
    i.value:team::text as team,
    o.value:over::int over,
    d.value:bowler::text bowler,
    d.value:batter::text batter,
    d.value:non_striker::text non_striker,
    d.value:runs::text runs,
    d.value:runs:extras::text extras,
    d.value:runs:total::text total,
    e.key::text extra_type,
    e.value::number extra_runs
from raw.match_raw_table m,
lateral flatten(input=> innings) i,
lateral flatten (input=>i.value:overs) o,
lateral flatten (input=>o.value:deliveries) d,
lateral flatten (input=>d.value:extras, outer=> TRUE) e
;
```

For elements that may not have the extras, by default the flatten function ommits them. So to keep them we need to specify the outer to true.  

we can do the same thing for the "Wickets".  

![{C2DA76A4-3C40-4536-96C9-32E401DE67FC}](https://github.com/user-attachments/assets/29a6819e-029f-4e8a-901c-d42122df055e)  

the final select statement that we can use inside a CTAS to create the table is the following :  

```SQL
USE SCHEMA CLEAN;
CREATE OR REPLACE TRANSIENT TABLE  delivery_match_clean_tb AS
    select
        INFO:match_type_number::int as match,
        i.value:team::text as team,
        o.value:over+1::int over,
        d.value:bowler::text bowler,
        d.value:batter::text batter,
        d.value:non_striker::text non_striker,
        d.value:runs:batter::text runs,
        d.value:runs:extras::text extras,
        d.value:runs:total::text total,
        e.key::text extra_type,
        e.value::number extra_runs,
        w.value:player_out::text player_out,
        w.value:kind::text player_out_kind,
        w.value:player_out_filders::variant player_out_filders,
        file_name ,
        file_row_number,
        file_hashkey,
        modified_ts
    from raw.match_raw_table m,
    lateral flatten(input=> innings) i,
    lateral flatten (input=>i.value:overs) o,
    lateral flatten (input=>o.value:deliveries) d,
    lateral flatten (input=>d.value:extras, outer=> TRUE) e,
    lateral flatten (input=>d.value:wickets, outer=> TRUE) w
;
```

We added the usual audit columns and also added 1 to the over column to start the idex at 1 rather than 0. Let's add some constraints same way as we did for the other tables.  

```SQL
alter table cricket.clean.delivery_match_clean_tb
modify column match_type_number set not null;

alter table cricket.clean.delivery_match_clean_tb
modify column team set not null;

alter table cricket.clean.delivery_match_clean_tb
modify column over set not null;

alter table cricket.clean.delivery_match_clean_tb
modify column bowler set not null;

alter table cricket.clean.delivery_match_clean_tb
modify column batter set not null;

alter table cricket.clean.delivery_match_clean_tb
modify column non_striker set not null;

-- fk relationship
alter table cricket.clean.delivery_match_clean_tb
add constraint fk_delivery_match_id
foreign key (match_type_number)
references cricket.clean.match_detail_clean (match_type_number);
```

#### 4.7 Gold layer :  

In this area we will create our data warehouse from the silver layer. This step depends on the business needs ans the answers to be provided to the questions asked by the business. For example when a certain match was palyed, who one, what was the score and so on ...  

we can think of a list of dimensions that will be needed such as :  
  - Date Dim : useful for time intelligence analysis.
  - MatchType Dim : useful to analyse data per type of match.
  - Geography Dim : to analyse data based on the location of the match.
  - Player Dim : essentiel to analyse data based on the details of players. For example the youngest player to score.
  - Team Dim : gives details regarding the teams so that we can analyse statistics of a single team for example.
  - Match Fact : it is the table holding details and scores about the matchs to analyse using the other dimensions. 

Next we will first create dimensions then fact table since these latters will hold the FKs of the dimensions.  

#### 4.7.1 Creating the dimentions : 

```SQL
USE DATABASE CRICKET;
USE SCHEMA CONSUMPTION;
USE ROLE SYSADMIN;

--date
create or replace table date_dim (
    date_id int primary key autoincrement,
    full_dt date,
    day int,
    month int,
    year int,
    quarter int,
    dayofweek int,
    dayofmonth int,
    dayofyear int,
    dayofweekname varchar(3), -- to store day names (e.g., "Mon")
    isweekend boolean -- to indicate if it's a weekend (True/False Sat/Sun both falls under weekend)
);

-- referee
create or replace table referee_dim (
    referee_id int primary key autoincrement,
    referee_name text not null,
    referee_type text not null
);

-- team
create or replace table team_dim (
    team_id int primary key autoincrement,
    team_name text not null
);

-- player..
create or replace table player_dim (
    player_id int primary key autoincrement,
    team_id int not null,
    player_name text not null
);

alter table cricket.consumption.player_dim
add constraint fk_team_player_id
foreign key (team_id)
references cricket.consumption.team_dim (team_id);


-- geography
create or replace table geography_dim (
    venue_id int primary key autoincrement,
    venue_name text not null,
    city text not null,
    state text,
    country text,
    continent text,
    end_Names text,
    capacity number,
    pitch text,
    flood_light boolean,
    established_dt date,
    playing_area text,
    other_sports text,
    curator text,
    lattitude number(10,6),
    longitude number(10,6)
);

create or replace table match_type_dim (
    match_type_id int primary key autoincrement,
    match_type text not null
);

```

**Note that the player dimension is a sub dimension of the team dimension. So the model is actualy a snowflake rather than a star schema.**  


#### 4.7.2 Creating the fact table : 

In the match fact table we can create some calculated columns that will hold the metrics of each match that can be used in a Cube or a PoweBI model to visualize the data.  


```SQL
-- Math fact table

CREATE or replace TABLE match_fact (
    match_id INT PRIMARY KEY,
    date_id INT NOT NULL,
    referee_id INT NOT NULL,
    team_a_id INT NOT NULL,
    team_b_id INT NOT NULL,
    match_type_id INT NOT NULL,
    venue_id INT NOT NULL,
    total_overs number(3),
    balls_per_over number(1),

    overs_played_by_team_a number(2),
    bowls_played_by_team_a number(3),
    extra_bowls_played_by_team_a number(3),
    extra_runs_scored_by_team_a number(3),
    fours_by_team_a number(3),
    sixes_by_team_a number(3),
    total_score_by_team_a number(3),
    wicket_lost_by_team_a number(2),

    overs_played_by_team_b number(2),
    bowls_played_by_team_b number(3),
    extra_bowls_played_by_team_b number(3),
    extra_runs_scored_by_team_b number(3),
    fours_by_team_b number(3),
    sixes_by_team_b number(3),
    total_score_by_team_b number(3),
    wicket_lost_by_team_b number(2),

    toss_winner_team_id int not null, 
    toss_decision text not null, 
    match_result text not null, 
    winner_team_id int not null,

    CONSTRAINT fk_date FOREIGN KEY (date_id) REFERENCES date_dim (date_id),
    CONSTRAINT fk_referee FOREIGN KEY (referee_id) REFERENCES referee_dim (referee_id),
    CONSTRAINT fk_team1 FOREIGN KEY (team_a_id) REFERENCES team_dim (team_id),
    CONSTRAINT fk_team2 FOREIGN KEY (team_b_id) REFERENCES team_dim (team_id),
    CONSTRAINT fk_match_type FOREIGN KEY (match_type_id) REFERENCES match_type_dim (match_type_id),
    CONSTRAINT fk_venue FOREIGN KEY (venue_id) REFERENCES geography_dim (venue_id),
    CONSTRAINT fk_toss_winner_team FOREIGN KEY (toss_winner_team_id) REFERENCES team_dim (team_id),
    CONSTRAINT fk_winner_team FOREIGN KEY (winner_team_id) REFERENCES team_dim (team_id)
);
```

Using database tool such as SSMS or Dbeaver we can view the ER diagram of our consumption layer :  

![{A995DCAD-4549-4FEA-993F-49010B2AE7DA}](https://github.com/user-attachments/assets/4bbcdbcf-ae7b-4285-8199-0545c81bb5db)  

We can also add another fact table containing the detailed deliveries of each match like the following : 

```SQL
  USE DATABASE CRICKET;
  USE SCHEMA CONSUMPTION;
  USE ROLE SYSADMIN;
CREATE or replace TABLE delivery_fact (
    match_id INT ,
    team_id INT,
    bowler_id INT,
    batter_id INT,
    non_striker_id INT,
    over INT,
    runs INT,
    extra_runs INT,
    extra_type VARCHAR(255),
    player_out VARCHAR(255),
    player_out_kind VARCHAR(255),

    CONSTRAINT fk_del_match_id FOREIGN KEY (match_id) REFERENCES match_fact (match_id),
    CONSTRAINT fk_del_team FOREIGN KEY (team_id) REFERENCES team_dim (team_id),
    CONSTRAINT fk_bowler FOREIGN KEY (bowler_id) REFERENCES player_dim (player_id),
    CONSTRAINT fk_batter FOREIGN KEY (batter_id) REFERENCES player_dim (player_id),
    CONSTRAINT fk_stricker FOREIGN KEY (non_striker_id) REFERENCES player_dim (player_id)
);
```

#### 4.7.4 Populating the fact and dimension tables : 

##### - Populating Team dimension table : 

We can start with the simple one which is the team dimension. We can simply make a union of "First_team" and "Second_team" and perform a select distinct on the result :  

```SQL
SELECT DISTINCT team_name
FROM
    (
    SELECT 
        first_team team_name
    FROM CLEAN.match_detail_clean
    UNION ALL
    SELECT 
        second_team team_name
    FROM CLEAN.match_detail_clean
    )
;
```

Let's insert this data in our team table :  

```SQL
INSERT INTO consumption.team_dim (team_name)
SELECT DISTINCT team_name
FROM
    (
    SELECT 
        first_team team_name
    FROM CLEAN.match_detail_clean
    UNION ALL
    SELECT 
        second_team team_name
    FROM CLEAN.match_detail_clean
    )
ORDER BY team_name
;
```
##### - Populating Player dimension table : 

The player dimension is also a simple one and we can follow the same logic :  

```SQL
SELECT t.team_id, p.team, p.player_name 
FROM
(
    SELECT team, player_name 
    FROM clean.player_clean_tb
    GROUP BY team, player_name
) p
JOIN consumption.team_dim t ON t.team_name = p.team;
```

![{801DF6D3-F88C-42C0-AD11-D5F02574B948}](https://github.com/user-attachments/assets/5f13d03b-f1a6-4f2d-adf1-75217fa0f458)


Then we can insert data in the player table in the consumption layer :  

```SQL
INSERT INTO CRICKET.CONSUMPTION.PLAYER_DIM (team_id, player_name)
SELECT t.team_id, p.player_name 
FROM
(
    SELECT team, player_name 
    FROM clean.player_clean_tb
    GROUP BY team, player_name
) p
JOIN consumption.team_dim t ON t.team_name = p.team;
```

##### - Populating Geography dimension table : 

For the geography dimension the logic is the same :  

```SQL
INSERT INTO CONSUMPTION.GEOGRAPHY_DIM (city, venue_name)
SELECT city, venue FROM CLEAN.MATCH_DETAIL_CLEAN
GROUP BY city, venue;
```

The other information regarding the stadium and coordinates can be populated using a GIS API.  

##### - Populating Player dimension table : 

Populating the match type the same way :  

```SQL
INSERT INTO CONSUMPTION.MATCH_TYPE_DIM (MATCH_TYPE)
SELECT  match_type FROM clean.match_detail_clean GROUP BY match_type;
```

##### - Populating Date dimension table : 

Now for the date dimension we will build using stored procedure based on the max and min dates in the match clean table :  

![{9059E0EB-42A5-42A9-9959-66A2E29BC694}](https://github.com/user-attachments/assets/e2275a1e-78f1-41d4-a975-63bfe961cf89)  

The stored procedure will insert date in the date dimension dynamicaly using variables from a selection that will generate date based on the min and max date:  

```SQL
DECLARE 
        min_date date;
        max_date date;
        row_count INT;
        sql_stat text;
        res RESULTSET;
BEGIN
        min_date := (SELECT min(event_date) FROM clean.match_detail_clean);
        max_date := (SELECT max(event_date) FROM clean.match_detail_clean); 
        row_count := (SELECT DATEDIFF(DAY, :min_date ,  :max_date));
        sql_stat := 'SELECT DATEADD(DAY, ROW_NUMBER() OVER (ORDER BY seq4()) - 1,''' || :min_date || ''')::date AS date_value   
                    FROM TABLE(GENERATOR(ROWCOUNT => ' || :row_count || '))'
```

The functions that will generate the range of dates between the max and min date variables are GENERATOR(ROWCOUNT => integer) and seq4() combined together.  

Let's create a sequence of 30 numbers starting from 0 :  

```SQL
SELECT SEQ4() FROM TABLE(GENERATOR(ROWCOUNT => 30));

```
Note that the table function is used to transform the list generated.  

![{E80E2EAF-2416-476A-9634-DF0DD7B77E74}](https://github.com/user-attachments/assets/9439f064-7066-4139-942f-b14948096ae8)  

now we can add a column that will use these values inside a DATEADD function to generate the dates starting from the min date : 

![{62FFD201-22D2-4094-AEA8-C2228EFF5C01}](https://github.com/user-attachments/assets/7d7e1fd5-accd-478e-bbe2-5672d183f137)  

The SEQ function may generate some gaps. So to insure that we don't have any gaps we can add ROW_NUMBER function -1 to start with index 0.  

```SQL
SELECT SEQ4(), DATEADD(DAY, ROW_NUMBER() OVER (ORDER BY seq4()) - 1,$min_date) FROM TABLE(GENERATOR(ROWCOUNT => 30));
```

Now from this date column generated we can give it a name and add other detailed columns based on it such as the day name, weekend or not and so on ...  

![{FFAD35EC-935B-4129-823A-4F3DD1CB015C}](https://github.com/user-attachments/assets/fd05ec81-3b48-48e8-912e-83a199ea1a2a)  

```SQL
SET min_date = (SELECT min(event_date) FROM clean.match_detail_clean);
SET max_date = (SELECT max(event_date) FROM clean.match_detail_clean); 

SELECT
    date_value AS Full_Dt,
    EXTRACT(DAY FROM date_value) AS Day,
    EXTRACT(MONTH FROM date_value) AS Month,
    EXTRACT(YEAR FROM date_value) AS Year,
    CASE WHEN EXTRACT(QUARTER FROM date_value) IN (1, 2, 3, 4) THEN EXTRACT(QUARTER FROM date_value) END AS Quarter,
    DAYOFWEEKISO(date_value) AS DayOfWeek,
    EXTRACT(DAY FROM date_value) AS DayOfMonth,
    DAYOFYEAR(date_value) AS DayOfYear,
    DAYNAME(date_value) AS DayOfWeekName,
    CASE When DAYNAME(date_value) IN ('Sat', 'Sun') THEN 1 ELSE 0 END AS IsWeekend
FROM
( SELECT SEQ4(), DATEADD(DAY, ROW_NUMBER() OVER (ORDER BY seq4()) - 1,$min_date) date_value FROM TABLE(GENERATOR(ROWCOUNT => 30)) );
```

We can also create a variable that will calculate the rowcount value using the differene between the max and the min date value + 1 (to include the max date also) and replcae the integer 30. Hewever, the generate function does not allow the use of variables inside it so we need to wrap the whole sql query inside an EXECUTE IMMEDIATE query that will hardcode all the variables in the query. Note that EXECUTE IMMEDIATE needs the query to be a string !  
The full stored procedure is the following :  

```SQL
CREATE OR REPLACE PROCEDURE generate_dates()
RETURNS TABLE()
LANGUAGE SQL
AS

DECLARE 
        min_date date;
        max_date date;
        row_count INT;
        sql_stat text;
        res RESULTSET;
BEGIN
        min_date := (SELECT min(event_date) FROM clean.match_detail_clean);
        max_date := (SELECT max(event_date) FROM clean.match_detail_clean); 
        row_count := (SELECT DATEDIFF(DAY, :min_date ,  :max_date) + 1);
        TRUNCATE TABLE CRICKET.CONSUMPTION.DATE_DIM;
        sql_stat := 'INSERT INTO cricket.consumption.date_dim (Full_Dt, Day, Month, Year, Quarter, DayOfWeek,      
                     DayOfMonth, DayOfYear, DayOfWeekName, IsWeekend)
                        SELECT
                            date_value AS Full_Dt,
                            EXTRACT(DAY FROM date_value) AS Day,
                            EXTRACT(MONTH FROM date_value) AS Month,
                            EXTRACT(YEAR FROM date_value) AS Year,
                            CASE WHEN EXTRACT(QUARTER FROM date_value) IN (1, 2, 3, 4) THEN EXTRACT(QUARTER FROM date_value) END AS Quarter,
                            DAYOFWEEKISO(date_value) AS DayOfWeek,
                            EXTRACT(DAY FROM date_value) AS DayOfMonth,
                            DAYOFYEAR(date_value) AS DayOfYear,
                            DAYNAME(date_value) AS DayOfWeekName,
                            CASE When DAYNAME(date_value) IN (''Sat'', ''Sun'') THEN 1 ELSE 0 END AS IsWeekend
                        FROM
                        (
                        SELECT DATEADD(DAY, ROW_NUMBER() OVER (ORDER BY seq4()) - 1,''' || :min_date || ''')::date AS date_value   
                        FROM TABLE(GENERATOR(ROWCOUNT => ' || :row_count || '))
                        )';
        res := (EXECUTE IMMEDIATE :sql_stat);
        RETURN TABLE(res)

        ;
END;

CALL generate_dates();
```

We insure that we truncate the table each time we need to call the SP since it is a simple table.  

##### - Populating Match fact table : 

For the Match fact table we will need to populate the foreign keys first which will need joins between the MATCH_DETAIL_CLEAN (at the silver layer) and  all the dimension tables already populated. Also some calculations will be retrieved from the delivery clean table.    

For example we can retrieve the date ID, the first team ID and the second team ID using the following query.  

```SQL
SELECT 
    m.match_type_number AS match_id,
    dd.date_id AS date_id,
    ftd.team_id AS first_team_id,
    std.team_id AS second_team_id,
FROM 
    cricket.clean.match_detail_clean m
    JOIN date_dim dd ON m.event_date = dd.full_dt
    JOIN team_dim ftd ON m.first_team = ftd.team_name 
    JOIN team_dim std ON m.second_team = std.team_name;
```

![{3FD5826E-B513-484E-B0D4-4C8CAF403789}](https://github.com/user-attachments/assets/dae7f752-c6df-4a4f-b60b-5e84120ff4d8)  

We can follow the samz logic for the rest of the columns and then add the calculations and finish the query with the group by clause :  

```SQL
SELECT 
    m.match_type_number AS match_id,
    dd.date_id AS date_id,
    0 AS referee_id,
    ftd.team_id AS first_team_id,
    std.team_id AS second_team_id,
    mtd.match_type_id AS match_type_id,
    vd.venue_id AS venue_id,
    50 AS total_overs,
    6 AS balls_per_overs,
    max(CASE WHEN d.team = m.first_team THEN  d.over ELSE 0 END ) AS OVERS_PLAYED_BY_TEAM_A,
    sum(CASE WHEN d.team = m.first_team THEN  1 ELSE 0 END ) AS balls_PLAYED_BY_TEAM_A,
    sum(CASE WHEN d.team = m.first_team THEN  d.extras ELSE 0 END ) AS extra_balls_PLAYED_BY_TEAM_A,
    sum(CASE WHEN d.team = m.first_team THEN  d.extra_runs ELSE 0 END ) AS extra_runs_scored_BY_TEAM_A,
    0 fours_by_team_a,
    0 sixes_by_team_a,
    (sum(CASE WHEN d.team = m.first_team THEN  d.runs ELSE 0 END ) + sum(CASE WHEN d.team = m.first_team THEN  d.extra_runs ELSE 0 END ) ) AS total_runs_scored_BY_TEAM_A,
    sum(CASE WHEN d.team = m.first_team AND player_out IS NOT null THEN  1 ELSE 0 END ) AS wicket_lost_by_team_a,    
    
    max(CASE WHEN d.team = m.second_team THEN  d.over ELSE 0 END ) AS OVERS_PLAYED_BY_TEAM_B,
    sum(CASE WHEN d.team = m.second_team THEN  1 ELSE 0 END ) AS balls_PLAYED_BY_TEAM_B,
    sum(CASE WHEN d.team = m.second_team THEN  d.extras ELSE 0 END ) AS extra_balls_PLAYED_BY_TEAM_B,
    sum(CASE WHEN d.team = m.second_team THEN  d.extra_runs ELSE 0 END ) AS extra_runs_scored_BY_TEAM_B,
    0 fours_by_team_b,
    0 sixes_by_team_b,
    (sum(CASE WHEN d.team = m.second_team THEN  d.runs ELSE 0 END ) + sum(CASE WHEN d.team = m.second_team THEN  d.extra_runs ELSE 0 END ) ) AS total_runs_scored_BY_TEAM_B,
    sum(CASE WHEN d.team = m.second_team AND player_out IS NOT null THEN  1 ELSE 0 END ) AS wicket_lost_by_team_b,
    tw.team_id AS toss_winner_team_id,
    m.toss_decision AS toss_decision,
    m.matach_result AS matach_result,
    mw.team_id AS winner_team_id
     
from 
    cricket.clean.match_detail_clean m
    JOIN date_dim dd ON m.event_date = dd.full_dt
    JOIN team_dim ftd ON m.first_team = ftd.team_name 
    JOIN team_dim std ON m.second_team = std.team_name 
    JOIN match_type_dim mtd ON m.match_type = mtd.match_type
    JOIN geography_dim vd ON m.venue = vd.venue_name AND m.city = vd.city
    JOIN cricket.clean.delivery_match_clean_tb d  ON d.match_type_number = m.match_type_number 
    JOIN team_dim tw ON m.toss_winner = tw.team_name 
    JOIN team_dim mw ON m.winner= mw.team_name 
    
GROUP BY
    m.match_type_number,
    date_id,
    referee_id,
    first_team_id,
    second_team_id,
    match_type_id,
    venue_id,
    total_overs,
    toss_winner_team_id,
    toss_decision,
    matach_result,
    winner_team_id
        ;
```

![{48975168-6E61-418D-ABDA-9C90EFCE8EE0}](https://github.com/user-attachments/assets/fccef7f2-17e0-4f27-a914-e96ae76b4062)  

Now we can wrap it inside an insert claue :  

```SQL
INSERT INTO CONSUMPTION.MATCH_FACT

SELECT 
    m.match_type_number AS match_id,
    dd.date_id AS date_id,
    0 AS referee_id,
    ftd.team_id AS first_team_id,
    std.team_id AS second_team_id,
    mtd.match_type_id AS match_type_id,
    vd.venue_id AS venue_id,
    50 AS total_overs,
    6 AS balls_per_overs,
    max(CASE WHEN d.team = m.first_team THEN  d.over ELSE 0 END ) AS OVERS_PLAYED_BY_TEAM_A,
    sum(CASE WHEN d.team = m.first_team THEN  1 ELSE 0 END ) AS balls_PLAYED_BY_TEAM_A,
    sum(CASE WHEN d.team = m.first_team THEN  d.extras ELSE 0 END ) AS extra_balls_PLAYED_BY_TEAM_A,
    sum(CASE WHEN d.team = m.first_team THEN  d.extra_runs ELSE 0 END ) AS extra_runs_scored_BY_TEAM_A,
    0 fours_by_team_a,
    0 sixes_by_team_a,
    (sum(CASE WHEN d.team = m.first_team THEN  d.runs ELSE 0 END ) + sum(CASE WHEN d.team = m.first_team THEN  d.extra_runs ELSE 0 END ) ) AS total_runs_scored_BY_TEAM_A,
    sum(CASE WHEN d.team = m.first_team AND player_out IS NOT null THEN  1 ELSE 0 END ) AS wicket_lost_by_team_a,    
    
    max(CASE WHEN d.team = m.second_team THEN  d.over ELSE 0 END ) AS OVERS_PLAYED_BY_TEAM_B,
    sum(CASE WHEN d.team = m.second_team THEN  1 ELSE 0 END ) AS balls_PLAYED_BY_TEAM_B,
    sum(CASE WHEN d.team = m.second_team THEN  d.extras ELSE 0 END ) AS extra_balls_PLAYED_BY_TEAM_B,
    sum(CASE WHEN d.team = m.second_team THEN  d.extra_runs ELSE 0 END ) AS extra_runs_scored_BY_TEAM_B,
    0 fours_by_team_b,
    0 sixes_by_team_b,
    (sum(CASE WHEN d.team = m.second_team THEN  d.runs ELSE 0 END ) + sum(CASE WHEN d.team = m.second_team THEN  d.extra_runs ELSE 0 END ) ) AS total_runs_scored_BY_TEAM_B,
    sum(CASE WHEN d.team = m.second_team AND player_out IS NOT null THEN  1 ELSE 0 END ) AS wicket_lost_by_team_b,
    tw.team_id AS toss_winner_team_id,
    m.toss_decision AS toss_decision,
    m.matach_result AS matach_result,
    mw.team_id AS winner_team_id
     
from 
    cricket.clean.match_detail_clean m
    JOIN date_dim dd ON m.event_date = dd.full_dt
    JOIN team_dim ftd ON m.first_team = ftd.team_name 
    JOIN team_dim std ON m.second_team = std.team_name 
    JOIN match_type_dim mtd ON m.match_type = mtd.match_type
    JOIN geography_dim vd ON m.venue = vd.venue_name AND m.city = vd.city
    JOIN cricket.clean.delivery_match_clean_tb d  ON d.match_type_number = m.match_type_number 
    JOIN team_dim tw ON m.toss_winner = tw.team_name 
    JOIN team_dim mw ON m.winner= mw.team_name 
    
GROUP BY
    m.match_type_number,
    date_id,
    referee_id,
    first_team_id,
    second_team_id,
    match_type_id,
    venue_id,
    total_overs,
    toss_winner_team_id,
    toss_decision,
    matach_result,
    winner_team_id
        ;
```

##### - Populating Deliveries fact table :

The same logic applies to the deliveries fact table :  

```SQL
INSERT INTO delivery_fact
SELECT 
    d.match_type_number AS match_id,
    td.team_id,
    bpd.player_id AS bower_id, 
    spd.player_id batter_id, 
    nspd.player_id AS non_stricker_id,
    d.over,
    d.runs,
    CASE WHEN d.extra_runs IS NULL THEN 0 ELSE d.extra_runs END AS extra_runs,
    CASE WHEN d.extra_type IS NULL THEN 'None' ELSE d.extra_type END AS extra_type,
    CASE WHEN d.player_out IS NULL THEN 'None' ELSE d.player_out END AS player_out,
    CASE WHEN d.player_out_kind IS NULL THEN 'None' ELSE d.player_out_kind END AS player_out_kind
FROM 
    cricket.clean.DELIVERY_MATCH_CLEAN_TB d
    JOIN team_dim td ON d.team = td.team_name
    JOIN player_dim bpd ON d.bowler = bpd.player_name
    JOIN player_dim spd ON d.batter = spd.player_name
    JOIN player_dim nspd ON d.non_striker = nspd.player_name;
```

![{2D346410-4683-4DE2-A46C-584060EA6F23}](https://github.com/user-attachments/assets/30a0dfc7-268e-46d4-84df-2d3c802eaeeb)  

### 5. Data Viz :

After building our Gold layer which is nothing else than a datawarehouse, we can connect to it using a BI tool such as PowerBI and view the modele as well as create some graphs !  

![{2B1C99FF-A5A7-40C3-9900-BAC5714C4D43}](https://github.com/user-attachments/assets/883ce9c0-1acd-4e77-8e96-485a6baca9ff)  

![{9DBE859B-7F99-400F-9713-6F6B192968B7}](https://github.com/user-attachments/assets/c54ef59f-a25a-4aa2-9ea8-851dc816127d)  

![{1A9DA1EB-BD62-484C-BE6C-670EB0FDF2E4}](https://github.com/user-attachments/assets/3cf84b4e-ff2a-4412-8155-dca880407055)  

![{5128121A-E170-4A06-B353-E47E042F3EC7}](https://github.com/user-attachments/assets/3f87983d-25be-4d54-bfe1-b1a21e34ee86)  

![{2E14C48B-3CF2-40D8-85FE-C8239F211EDA}](https://github.com/user-attachments/assets/76665443-2de9-4da8-a646-09cd10dc7663)  


### 6. Automate the data flow :

We managed so far to build our lakehouse with all the needed components, but what if new data (a new json file in our case) arrives? should we repeat the whole process manually ? This where the automation is needed !  
Several tools can do this but in our project we will use mainly the built-in functionnalities of snowflake to automate the process of data loading.  

![image](https://github.com/user-attachments/assets/b8e0d300-7dcf-42ae-8e38-b72e17d9737e)  

3 snowflake objects are needed :  

- Snowpipe : is Snowflake’s continuous data ingestion service, designed to automatically load data from external storage (AWS S3, Azure Blob, GCS) into Snowflake as soon as new files arrive.
- Snowflake stream :  captures row-level changes (INSERTs, UPDATEs, DELETEs) on a table. Streams are used to track new data loaded by Snowpipe before further processing.
- Snowflake task : automates execution of SQL or Snowpark scripts at scheduled intervals. Tasks can process data from streams and transform it into final tables.

![{D21B87A0-9B7D-40FF-B970-20CDF91D910F}](https://github.com/user-attachments/assets/45cf18a0-9c61-439d-bb33-6e23135e99dc)  

#### 6.1 Use of snowpipe to automate data loading from stage:

To use a snowpipe we need to configure azure to store events of the blob storage and send notifications to the snowflake to be captured to trigger the snowpipe. This will need a configuration of an **Event subscription**:  

![{CF6BC2ED-EE5D-45EB-89B7-ED71238D8097}](https://github.com/user-attachments/assets/4f9d77fe-1aaa-47bf-baeb-3b3573c592ba)  

![{51A9F905-B200-40D0-A285-71EB7653994A}](https://github.com/user-attachments/assets/459e1f2c-f4df-4655-b660-8461e4c2dae1)  

We specify where to store the events. In this case we will use the storage Queue:  

![{D17F600B-3F28-45FD-8244-E083BE8D1AD7}](https://github.com/user-attachments/assets/5ab7d177-bfed-4df0-be37-de9898fd81ea)  

We choose one or create a new Queue:  

![{E6170701-A240-4A19-A82B-3DFA0197BE04}](https://github.com/user-attachments/assets/73b11648-6d20-480d-8e7f-7d43856b70c5)  

In the filter section we can specify that we want this events subscription to listen to only one container:  

![{643C187D-2387-46F7-B8E9-17998E72F5B1}](https://github.com/user-attachments/assets/f9e41799-1712-4f5b-9846-c284c3100d9c)  

The result sent to the queue is a JSON file and similar to the following :  

![{559C4FFB-C4B8-44F7-B672-63C7BE8E87B0}](https://github.com/user-attachments/assets/0892ad45-4d88-4e9d-9f10-bb7961560cf4)  

![{78AD3C01-3CFB-494A-94E1-CD91ABD6D5CF}](https://github.com/user-attachments/assets/9fe66e60-fa9e-4488-a8bb-531cf634c3ae)  

The notifications are captured in snowflake using **notification integration** and will trigger the pipe each time a file is added in the external storage (Azure in our case).  

```SQL
CREATE  NOTIFICATION INTEGRATION EventFiles
ENABLED =TRUE
TYPE=QUEUE
NOTIFICATION_PROVIDER=AZURE_STORAGE_QUEUE
AZURE_STORAGE_QUEUE_PRIMARY_URI='*****'
AZURE_TENANT_ID='****';
```
**Note that this will need a first authentification (we can find the link to consent in the result of the DESC query on the NOTIFICATION INTEGRATION) as it uses an app registration to connect to AZURE events (We also need to grant this app the Queue Blob Contributor Role.)**  

Then we create the pipe as follows: 

```SQL

CREATE OR REPLACE pipe "load_to_raw"
  auto_ingest = true
  integration = 'EventFiles'
  AS
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

ALTER PIPE BANK_TRANSACTIONS_PIPE REFRESH;
```

Now the snowpipe is running and will capture notifications and trigger the Copy Into query !  

We can do this in another way using only tasks. A task to run each 5 min for example to Copy data into the table !  


#### 6.2 Use of tasks to automate data loading from stage and table populating :

We have one big raw table which is the raw area that feeds 3 tables in the clean area :  

![{C278128C-88D9-449F-884B-2D9E9D8E95E4}](https://github.com/user-attachments/assets/e9b691c1-fcf3-4e60-92f4-9b77572743bc)  

So we will need to build 3 streams on the raw table to feed the 3 tables since a stream is only consumed once !  

```SQL
CREATE OR REPLACE STREAM CRICKET.RAW.match_stream ON TABLE CRICKET.RAW.MATCH_RAW_TABLE APPEND_ONLY = TRUE;

CREATE OR REPLACE STREAM CRICKET.RAW.player_stream ON TABLE CRICKET.RAW.MATCH_RAW_TABLE APPEND_ONLY = TRUE;

CREATE OR REPLACE STREAM CRICKET.RAW.delivery_stream ON TABLE CRICKET.RAW.MATCH_RAW_TABLE APPEND_ONLY = TRUE;
```

 Now these streams will  capture all the changes in the tables so we can use them in the insert query later on. **Note that the streams above will track only the inserts since we specified the APPEND_ONLY to true. If Wa are dealing with SCD type 2 we will need to track inserts and updates so we should not set the APPEND_ONLY to true. By default the stream tracks inserts, updates and deletes. We can filer on the result to choose only inserts an updates like follows**:  


```SQL
SELECT * 
FROM my_stream
WHERE METADATA$ACTION != 'DELETE';
```

We need to create now a task that will run on a schedule to copy data in the first big table in the raw area !  

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_json_files
WAREHOUSE = 'COMPUTE_WH'
SCHEDULE = '5 minute'
AS 
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
    FROM @CRICKET.PUBLIC.CRICKET_JSON_FILES_CONTAINER_ONLY
    (FILE_FORMAT => 'cricket.land.json_ff') t
    )
ON_ERROR = continue
;
```

Now we need a task that will depend on the first task and take the content of the raw stream and insert it in the clean table :  

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_to_clean_match_table
WAREHOUSE = 'COMPUTE_WH'
AFTER CRICKET.RAW.load_json_files
WHEN SYSTEM$STREAM_HAS_DATA('CRICKET.RAW.match_stream')
AS
INSERT INTO cricket.clean.match_detail_clean 
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
from CRICKET.RAW.match_stream;
```

The same thing for the player clean table : 

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_to_clean_player_table
WAREHOUSE = 'COMPUTE_WH'
AFTER CRICKET.RAW.load_to_clean_match_table
WHEN SYSTEM$STREAM_HAS_DATA('CRICKET.RAW.player_stream')
AS
INSERT INTO CRICKET.CLEAN.PLAYER_CLEAN_TB
SELECT
    raw.INFO:match_type_number::int as match_type_number,
    p.key::text as team,
    players.value::text as player_name,
    file_name ,
    file_row_number,
    file_hashkey,
    modified_ts

FROM CRICKET.RAW.player_stream raw,
LATERAL FLATTEN(input => raw.INFO:players) p,
LATERAL FLATTEN(input => p.value) players;
```

Also the delivery clean table :  

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_to_clean_delivery_table
WAREHOUSE = 'COMPUTE_WH'
AFTER CRICKET.RAW.load_to_clean_player_table
WHEN SYSTEM$STREAM_HAS_DATA('CRICKET.RAW.delivery_stream')
AS
INSERT INTO CRICKET.CLEAN.DELIVERY_MATCH_CLEAN_TB
select
INFO:match_type_number::int as match,
i.value:team::text as team,
o.value:over+1::int over,
d.value:bowler::text bowler,
d.value:batter::text batter,
d.value:non_striker::text non_striker,
d.value:runs:batter::text runs,
d.value:runs:extras::text extras,
d.value:runs:total::text total,
e.key::text extra_type,
e.value::number extra_runs,
w.value:player_out::text player_out,
w.value:kind::text player_out_kind,
w.value:player_out_filders::variant player_out_filders,
file_name ,
file_row_number,
file_hashkey,
modified_ts
from CRICKET.RAW.delivery_stream m,
lateral flatten(input=> innings) i,
lateral flatten (input=>i.value:overs) o,
lateral flatten (input=>o.value:deliveries) d,
lateral flatten (input=>d.value:extras, outer=> TRUE) e,
lateral flatten (input=>d.value:wickets, outer=> TRUE) w
;
```

Now that we built the first tasks to populate the landing layer, we can view them using the DAG view in the UI : 

![{160C649E-E8D9-433F-9DA6-9AB5468021AF}](https://github.com/user-attachments/assets/40d8097b-6f44-462c-9ced-f39fc79690f2)  

We need other tasks to populate the consumption layer. We can either use the streams as we did above or use the "Merge Into" to insert the new elements and the updated ones.  
```SQL
MERGE INTO consumption.team_dim AS tgt

USING     
(
    SELECT 
        first_team team_name
    FROM CLEAN.match_detail_clean
    UNION ALL
    SELECT 
        second_team team_name
    FROM CLEAN.match_detail_clean
    ) AS src
    
ON tgt.team = src.team

ORDER BY team_name
)
 WHEN NOT MATCHED THEN 
    INSERT (team)
    VALUES (src.team)

 WHEN MATCHED THEN 
    UPDATE SET team = src.team;
```
In our case we have only inserts so no need to use a merge but we use another approch based on "Except" or "Minus". If we apply that to the team dimension we will have :  

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_to_dim_team
WAREHOUSE = 'COMPUTE_WH'
AFTER CRICKET.RAW.load_to_clean_delivery_table
AS
INSERT INTO consumption.team_dim (team_name)
    SELECT * FROM (
        SELECT DISTINCT team_name
        FROM
            (
            SELECT 
                first_team team_name
            FROM CLEAN.match_detail_clean
            UNION ALL
            SELECT 
                second_team team_name
            FROM CLEAN.match_detail_clean
            )
        ORDER BY team_name
        )
    MINUS
    SELECT team_name FROM CRICKET.CONSUMPTION.TEAM_DIM
;
```
The minus will simply perform a difference between the team dimension and the select that retrieve all the data of the team table including the old and new data. And since the difference is only the new rows that don't exist in the team dimension the minus returns only these new rows.  

Same logic for the GEOGRAPHY dimension. We will just add a CASE clause to set the city to "NA" if it is null in the clean table:  

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_to_dim_venue
WAREHOUSE = 'COMPUTE_WH'
AFTER CRICKET.RAW.load_to_clean_delivery_table
AS
INSERT INTO CRICKET.CONSUMPTION.GEOGRAPHY_DIM (city, venue_name)
  SELECT city, venue FROM 
   (SELECT 
       CASE WHEN city is null then 'NA' ELSE city END city, 
       venue
       FROM CRICKET.CLEAN.MATCH_DETAIL_CLEAN
       GROUP BY city, venue
   )
    MINUS
    SELECT city , venue_name FROM CRICKET.CONSUMPTION.GEOGRAPHY_DIM
;
```

We create also a task for the date dimension :  

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_to_dim_date
WAREHOUSE = 'COMPUTE_WH'
AFTER CRICKET.RAW.load_to_clean_delivery_table
AS
CALL generate_dates();
```
For the fact tables since it is generally larger than the dimension tables, it is not recommanded to use the MINUS or EXCEPT approach since it scans all the columns to perform the difference while the left join will only scan the necessery columns. We will use a left join : 

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_to_match_fact
WAREHOUSE = 'COMPUTE_WH'
AFTER CRICKET.RAW.load_to_dim_team, CRICKET.RAW.load_to_dim_player ,CRICKET.RAW.load_to_dim_venue ,CRICKET.RAW.load_to_dim_date
AS
INSERT INTO CRICKET.CONSUMPTION.MATCH_FACT
    SELECT a.* FROM
    (
    SELECT 
        m.match_type_number AS match_id,
        dd.date_id AS date_id,
        0 AS referee_id,
        ftd.team_id AS first_team_id,
        std.team_id AS second_team_id,
        mtd.match_type_id AS match_type_id,
        vd.venue_id AS venue_id,
        50 AS total_overs,
        6 AS balls_per_overs,
        max(CASE WHEN d.team = m.first_team THEN  d.over ELSE 0 END ) AS OVERS_PLAYED_BY_TEAM_A,
        sum(CASE WHEN d.team = m.first_team THEN  1 ELSE 0 END ) AS balls_PLAYED_BY_TEAM_A,
        sum(CASE WHEN d.team = m.first_team THEN  d.extras ELSE 0 END ) AS extra_balls_PLAYED_BY_TEAM_A,
        sum(CASE WHEN d.team = m.first_team THEN  d.extra_runs ELSE 0 END ) AS extra_runs_scored_BY_TEAM_A,
        0 fours_by_team_a,
        0 sixes_by_team_a,
        (sum(CASE WHEN d.team = m.first_team THEN  d.runs ELSE 0 END ) + sum(CASE WHEN d.team = m.first_team THEN  d.extra_runs ELSE 0 END ) ) AS total_runs_scored_BY_TEAM_A,
        sum(CASE WHEN d.team = m.first_team AND player_out IS NOT null THEN  1 ELSE 0 END ) AS wicket_lost_by_team_a,    
        
        max(CASE WHEN d.team = m.second_team THEN  d.over ELSE 0 END ) AS OVERS_PLAYED_BY_TEAM_B,
        sum(CASE WHEN d.team = m.second_team THEN  1 ELSE 0 END ) AS balls_PLAYED_BY_TEAM_B,
        sum(CASE WHEN d.team = m.second_team THEN  d.extras ELSE 0 END ) AS extra_balls_PLAYED_BY_TEAM_B,
        sum(CASE WHEN d.team = m.second_team THEN  d.extra_runs ELSE 0 END ) AS extra_runs_scored_BY_TEAM_B,
        0 fours_by_team_b,
        0 sixes_by_team_b,
        (sum(CASE WHEN d.team = m.second_team THEN  d.runs ELSE 0 END ) + sum(CASE WHEN d.team = m.second_team THEN  d.extra_runs ELSE 0 END ) ) AS total_runs_scored_BY_TEAM_B,
        sum(CASE WHEN d.team = m.second_team AND player_out IS NOT null THEN  1 ELSE 0 END ) AS wicket_lost_by_team_b,
        tw.team_id AS toss_winner_team_id,
        m.toss_decision AS toss_decision,
        m.matach_result AS matach_result,
        mw.team_id AS winner_team_id
         
    from 
        cricket.clean.match_detail_clean m
        JOIN CRICKET.CONSUMPTION.date_dim dd ON m.event_date = dd.full_dt
        JOIN CRICKET.CONSUMPTION.team_dim ftd ON m.first_team = ftd.team_name 
        JOIN CRICKET.CONSUMPTION.team_dim std ON m.second_team = std.team_name 
        JOIN CRICKET.CONSUMPTION.match_type_dim mtd ON m.match_type = mtd.match_type
        JOIN CRICKET.CONSUMPTION.geography_dim vd ON m.venue = vd.venue_name AND m.city = vd.city
        JOIN cricket.clean.delivery_match_clean_tb d  ON d.match_type_number = m.match_type_number 
        JOIN CRICKET.CONSUMPTION.team_dim tw ON m.toss_winner = tw.team_name 
        JOIN CRICKET.CONSUMPTION.team_dim mw ON m.winner= mw.team_name 
        
    GROUP BY
        m.match_type_number,
        date_id,
        referee_id,
        first_team_id,
        second_team_id,
        match_type_id,
        venue_id,
        total_overs,
        toss_winner_team_id,
        toss_decision,
        matach_result,
        winner_team_id
        ) AS a
        
    LEFT JOIN  (SELECT match_id  FROM CRICKET.CONSUMPTION.MATCH_FACT) b ON a.match_id = b.match_id
    WHERE b.match_id is null;
```

We can also optimize the query by only joining the fact table with the selection of the match_id as the right table.  
we  apply the same logic for the second fact table :  

```SQL
CREATE OR REPLACE TASK CRICKET.RAW.load_to_delivery_fact
WAREHOUSE = 'COMPUTE_WH'
AFTER CRICKET.RAW.load_to_match_fact
AS
INSERT INTO CRICKET.CONSUMPTION.delivery_fact
    SELECT a.* FROM 
    (SELECT
        d.match_type_number AS match_id,
        td.team_id,
        bpd.player_id AS bower_id, 
        spd.player_id batter_id, 
        nspd.player_id AS non_stricker_id,
        d.over,
        d.runs,
        CASE WHEN d.extra_runs IS NULL THEN 0 ELSE d.extra_runs END AS extra_runs,
        CASE WHEN d.extra_type IS NULL THEN 'None' ELSE d.extra_type END AS extra_type,
        CASE WHEN d.player_out IS NULL THEN 'None' ELSE d.player_out END AS player_out,
        CASE WHEN d.player_out_kind IS NULL THEN 'None' ELSE d.player_out_kind END AS player_out_kind
    FROM 
        cricket.clean.DELIVERY_MATCH_CLEAN_TB d
        JOIN CRICKET.CONSUMPTION.team_dim td ON d.team = td.team_name
        JOIN CRICKET.CONSUMPTION.player_dim bpd ON d.bowler = bpd.player_name
        JOIN CRICKET.CONSUMPTION.player_dim spd ON d.batter = spd.player_name
        JOIN CRICKET.CONSUMPTION.player_dim nspd ON d.non_striker = nspd.player_name) a
    LEFT JOIN (SELECT match_id  FROM CRICKET.CONSUMPTION.DELIVERY_FACT) b ON a.match_id = b.match_id 
    WHERE b.match_id is null;
```

The final DAG looks as follows :  

![{0ADCC56B-8CB2-4905-B429-D98406325984}](https://github.com/user-attachments/assets/10a51537-c1c4-430b-893f-17b3c9cfe2fa)  

By default all the tasks are suspended. We need to resume that, but to do so we need first to grant the sysadmin role this ability :  

```SQL
USE ROLE ACCOUNTADMIN;
GRANT EXECUTE TASK, EXECUTE MANAGED TASK ON ACCOUNT TO ROLE SYSADMIN;
USE ROLE SYSADMIN;
```

Then we resume tasks in the reverse order, starting with the latest then the oldest :  

```SQL
ALTER TASK CRICKET.RAW.load_to_delivery_fact RESUME;
ALTER TASK CRICKET.RAW.load_to_match_fact RESUME;
ALTER TASK CRICKET.RAW.load_to_dim_date RESUME;
ALTER TASK CRICKET.RAW.load_to_dim_venue RESUME;
ALTER TASK CRICKET.RAW.load_to_dim_player RESUME;
ALTER TASK CRICKET.RAW.load_to_dim_team RESUME;
ALTER TASK CRICKET.RAW.load_to_clean_delivery_table RESUME;
ALTER TASK CRICKET.RAW.load_to_clean_player_table RESUME;
ALTER TASK CRICKET.RAW.load_to_clean_match_table RESUME;
ALTER TASK CRICKET.RAW.load_json_files RESUME;
```

![{35B8A576-3A57-49C4-B518-CCF62D83168E}](https://github.com/user-attachments/assets/48d5d21a-2bcb-4329-a783-de98b954ac44)  

We can see that the first task runs correctly since it is schedueled every 5 min. The next one loading to the clean match table will be skipped since there is no data in the stream.  

![{C142B7B9-DB78-49ED-B4B0-D8EBF01C3FDA}](https://github.com/user-attachments/assets/be2248d9-9bc3-4c18-962e-1b2a25cc17ae)  

Now let's add a new file with some outlier data so we can identify them in process :  

![{A78D708D-429C-4059-922C-A0437B927E07}](https://github.com/user-attachments/assets/7a699fca-05a8-4a40-bffe-c90590a6e87f)  

We set the date to 2025-02-19, the city to paris and we change one palyer's name to "ZAKARIA H" and we set venue to blank.  

![{DE8A7FC4-1B9E-4834-8B90-2435F910E513}](https://github.com/user-attachments/assets/b3915d14-2185-43ea-ac81-84152fd38d22)  

Now let's load the new file in the blob storage :  

![{57098EED-5914-4A63-9134-127C4FF77920}](https://github.com/user-attachments/assets/589a9cd1-818b-485e-9351-f56b8db50d67)  

We need to manually refresh the external stage :  

![{43A9018C-97F1-40A0-817A-ADF2FEB1B5B3}](https://github.com/user-attachments/assets/6a60a454-49b5-4d5c-9624-91c4914b784b)  

It is refreshed :  

![{5FC96CD4-A908-4F85-8C97-6B1AAB664EB5}](https://github.com/user-attachments/assets/35ceeda0-d83d-4d27-a86f-812316cd625f)  

Now we wait for the flow to start. Than we can check the status and run history of every task in the graph to see is it succeeded or not :  

![{0710CC08-BFC7-4BCD-9BAC-8272311C1432}](https://github.com/user-attachments/assets/ad71c10b-9e42-4ab0-8307-fea8941bed52)  

Or we can use the DVM of tasks to list all the executed tasks:  

```SQL
SELECT NAME, STATE, ERROR_MESSAGE, SCHEDULED_TIME
FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY())
WHERE QUERY_START_TIME  > '2025-02-19 05:47:38.498 -0800';  
```

![{15B66959-B016-4C0A-8FA1-9D13ABBA4D06}](https://github.com/user-attachments/assets/adc28e37-8a5d-48e0-9c24-64847cfb9b5b)  

We can see that all the tasks have suceeded !  

Now we can check the data we changed in the file in the tables:  

The new file was inserted.  

![{67807A1D-C1EA-45AA-AB34-CBE9AF5B85E3}](https://github.com/user-attachments/assets/fc5f970f-f7d0-45f8-9d54-efc415895502)  

The data of the new match were inserted in the match clean table and we can see the match_number = 10000 :  

![{A697FBD7-F7DC-4761-A77B-AE6A62469DB9}](https://github.com/user-attachments/assets/33295315-ce55-4aed-8305-06d28bab6794)  

The data of players of the new match were inserted also and we can see "ZAKARIA H"

![{5E887A00-961F-4BB0-93D9-F94F685FF5D8}](https://github.com/user-attachments/assets/c8d28404-043e-4fb2-a1e2-5e66fe060e2a)  

The same ting for the delivery clean table :  

![{7FCF1ADD-E742-4898-B6C3-DA870F88504F}](https://github.com/user-attachments/assets/5ad0730a-28c0-4c9a-b260-a26bfe24a1aa)  

Here we can see really the power of using metadata columns that links the raws to the file they come from. In case tasks failed for a reason we can come back to the concerned data liked to the concerned file and do our changes and we never lose data !  
This same logic can be so beneficial in the consumption layer also.  

In the date dimension we can see that data untill the new date "2025-02-20":  

![{DE46899F-8318-4102-8451-47BC63373625}](https://github.com/user-attachments/assets/ddf51abc-8d48-4a57-a646-71832c8325b5)  

The same thing for the player dimension :  

![{DB02F870-BFAE-4328-94B5-640E140CE4A1}](https://github.com/user-attachments/assets/191f4e0c-66a2-47e6-a12b-1bb0ba889e9e)  

And also for the match fact table :  

![{9BD6D5AD-5B3F-4F83-B3F6-FDD42C3D6B5C}](https://github.com/user-attachments/assets/37f82da5-c15b-4a6e-8980-b8976b599237)  


Now that we built the whola lakehouse with the automated process of lading data, we will need one last brick in this project which is the CICD implementation.  

### 7. CICD implementation using GitHub Actions :


![{24456CE9-930B-417F-8FC6-7D1EB35E59E0}](https://github.com/user-attachments/assets/7764f2eb-f13e-460c-98d5-7032ae99c8e9)  

Most of tools are imperative.  

Some imprerative tools:  

![{74546D84-2017-43ED-877B-89C260A8756B}](https://github.com/user-attachments/assets/228682be-d81f-443a-a019-fb6dc7389e6a)  

Some declarative tools:  

![{B020CBDB-1993-45D2-9AFC-F3B2D35DDB2A}](https://github.com/user-attachments/assets/303894d3-5900-4bb4-931e-c194a71bbfb4)  

some tools are more powerfull dealing with some objects than others. For example **terraform** id best for managing the high level objects such as databases, roles, warehouses but for the other objects at the schema level other tools such as **Schemachange** come very handy !  

We will use in our POC terraform for databases, warehouses and roles management and schemachange for the schema level objects management.  

#### 7.1 Terraform :  

Terraform is a great tool that lets us write code to manage cloud stuff like servers, databases, networks, and more—just like you'd write code for an app.

🔧 Key Features:

- Infrastructure as Code (IaC): Declare what you want (e.g., “I need 3 servers in AWS”), and Terraform builds it.

- Multi-cloud Support: Works with AWS, Azure, GCP, and many others.

- Declarative Syntax: You describe the desired state, not the steps to get there.

- Plan & Apply Workflow:
    - terraform plan: Shows what changes it will make.
    - terraform apply: Actually makes those changes.

- State Management: Keeps track of what resources exist and their configurations using a state file.

A simple example :  

```hcl
provider "aws" {
  region = "us-west-1"
}

resource "aws_instance" "my_server" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

This simple code generates an AWS server that simple (of course we need to setup the authentication to AWS first)  

#### Structure :  

Terraform can work using one single file where we declare all the needed configurations and ressources to create ! **But this will get sooo messy !**. Hence we use a conventional structure that helps organising our project and reuse some common elements !  

A classic structure would be :  

```bash
project-root/
│
├── main.tf              # Main resources and configurations
├── variables.tf         # Input variable definitions
├── outputs.tf           # Output values (for chaining modules or debugging)
├── terraform.tfvars     # Actual values for variables (optional)
├── provider.tf          # Cloud provider config (e.g., AWS, Azure, etc.)
├── versions.tf          # Terraform & provider version constraints
├── modules/             # Reusable resource modules
│   └── my_module/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│
└── environments/        # Separate configs for prod, dev, etc. (optional)
    ├── dev/
    │   └── terraform.tfvars
    └── prod/
        └── terraform.tfvars
```

🧱 File Breakdown
🔹 main.tf
Core infrastructure logic.  

E.g., creating VPCs, EC2 instances, S3 buckets.  

🔹 variables.tf  
Defines all input variables:  

```hcl
Copier
Modifier
variable "region" {
  type    = string
  default = "us-east-1"
}
```
🔹 terraform.tfvars
Assigns values to variables (can be separate per environment).

```hcl
Copier
Modifier
region = "us-west-2"
```
🔹 outputs.tf
Outputs values after provisioning (useful in multi-stage setups):

```hcl
Copier
Modifier
output "instance_id" {
  value = aws_instance.my_instance.id
}
```
🔹 provider.tf
Cloud provider setup:

```hcl
Copier
Modifier
provider "aws" {
  region = var.region
}
```
🔹 versions.tf
Lock Terraform version & provider versions:

```hcl
Copier
Modifier
terraform {
  required_version = ">= 1.4.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

📦 Using Modules (Optional but Recommended)
Modules help break infrastructure into reusable, logical chunks. You can call them like this:

```hcl
Copier
Modifier
module "vpc" {
  source = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
}
```

We will follow a quite similar structure for our project !  

```bash
Snowflake_TF_CC_CICD/
├── environments/              # Separate env configurations (like dev, prod)
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   ├── backend.tf         # we use remote state file backend for each envirement (the file is stored in AZURE blob storage)
│   │   ├── outputs.tf
│   ├── prod/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   ├── backend.tf         # we use remote state file backend for each envirement (the file is stored in AZURE blob storage)
│   │   ├── outputs.tf
├── modules/                   # Reusable modules
│   ├── database/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   ├── roles/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   ├── warehouse/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
├── .github/
│   └── workflows/
│       └── terraform.yml      # GitHub Actions CI/CD workflow
├── .gitignore
└── backend.tf                 # If using remote state backend (optional)
```

the snippet above shows the global structure combining both environnements DEV and PROD. In real deployment we will have two branchs each with corresponding structure and not the combined one ! For example dev would be :  

```bash
Snowflake_TF_CC_CICD/
├── environment/              # Separate env configurations (like dev, prod)
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   ├── backend.tf         # we use remote state file backend for each envirement (the file is stored in AZURE blob storage)
│   │   ├── outputs.tf
├── modules/                   # Reusable modules
│   ├── database/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   ├── roles/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   ├── warehouse/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
├── .github/
│   └── workflows/
│       └── terraform.yml      # GitHub Actions CI/CD workflow
├── .gitignore
└── backend.tf                 # If using remote state backend (optional)
```

We have used a cmd script to produce automaticaly this file structure.  

#### Implementation :  

The files of terraform are two types: common files that are not specific to the environnement are others that are environnement specific such as the backend and terraform.tfvars ! the environnement specific files need to be mentionned in the **.gitignore** files so that they don't get replaced when merging branchs:  

![image](https://github.com/user-attachments/assets/e15ffa9d-7b6f-4e3b-b5b2-32345bc84f86)  

#### - Backend
In our case we use terraform to manage snowflake infrastructure so the provider would be Snowflake. We can check terraform providers documentation that gives all the details regarding how to implement fot each provider !  

To follow the best practices we will keep the state file that allows snowflake to track all the changes in a remote location : Azure blob storage !  

This will need from us to setup the connection to this blob storage in the backend file :  

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "Eco_demos"         # The name of the resource group containing the storage account
    storage_account_name = "ecodemostorage"    # The name of the storage account
    container_name       = "terraformcicd"     # The container in the blob storage where the state file will be stored
    key                  = "terraform_SRV_dev.tfstate" # The name of the state file (it can be any name, usually terraform.tfstate or per environment)
  }
}
```

To connect terraform will need an access key depending on the authentication configuration that we choose. The best practice is to use a service principal with a secret key and this needs to create a middle application (azure creates it for us but we need to be admin to do that) and this one will authenticate securely for terraform.  

Since this is just a demo and not a real world scenario we will use a simpler method to authenticate : SAS - shared access signature !  
This method generates a key that has a validation period with some permissions on the blob storage so that terraform can access it !  

![image](https://github.com/user-attachments/assets/05950056-c7f3-475d-a017-a2129ebab39f)  

The required permissions are read, write and list (behind the scene terraform will need to list all the containers to check if the one containing the state file is there before proceeding ).  
Once the SAS is generated we can store it in an environment variable (locally and in the github environment secrets) to be used by terraform.  

This is the corresponding backend file for the SAS authentication :  

![image](https://github.com/user-attachments/assets/8dd4d89d-6784-47ac-8f05-d43e361a2101)  


The SAS key environnement variable need to be named ARM_SAS_TOKEN so that terraform can read it automaticaly :  

![image](https://github.com/user-attachments/assets/1f05f68a-a6cb-4a01-909b-49dbd0df6619)  

if we have a service principle the secrets to connect to azure backend are the following : 

``` bash
export ARM_CLIENT_ID="your-client-id"
export ARM_CLIENT_SECRET="your-client-secret"
export ARM_SUBSCRIPTION_ID="your-subscription-id"
export ARM_TENANT_ID="your-tenant-id"
```

These are variables to declare in our environment (local or in github also) and terraform will retrieve that automaticaly.  

**Pay attention to how you set your environment variables locally !! Terraform looks for environment variables before falling back on the values in .tfvars files or variables.tf. Specifically, when an environment variable follows the pattern TF_VAR_<variable_name>, or a variable named the same way as the provider parameters, Terraform automatically uses it to populate the corresponding variable. For example if we have both SNOWFLAKE_PRIVATE_KEY_PATH and TF_VAR_SNOWFLAKE_PRIVATE_KEY, Terraform might be confused about which one to use. the best thing is to use environment variables only for things we can't store in the .tfvars or variables.tf files such as the private key.**  

#### - Snowflake configuration

The main.tf file is conventionaly used to specify the changes we want to make :  

![image](https://github.com/user-attachments/assets/256a1d2d-b25c-468f-b137-5e34973ae753)  

Here we specify the provider config and the objects to track. In our example the provider is snowflake and the objects are database, warehouse and roles.  

The provider section is for snowflake parameters and authentication :  

```hcl
provider "snowflake" {
  organization_name  = var.SNOWFLAKE_ORGANIZATION_NAME
  account_name  = var.SNOWFLAKE_ACCOUNT_NAME
  user          = var.SNOWFLAKE_USER
  private_key   = base64decode(var.SNOWFLAKE_PRIVATE_KEY)
  authenticator = "SNOWFLAKE_JWT"
  role          = "ACCOUNTADMIN"
}
```

the private key is to be generated using the SSH key pair authentication : a public key to be set in the user parameters in snowflake and a private key that terraform will use to authenricate !  

To create the ssh key we first create a .ssh file that will hold the keys then we follow the instructions:  

```cmd
$ cd ~/.ssh
$ openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out snowflake_tf_snow_key.p8 -nocrypt
$ openssl rsa -in snowflake_tf_snow_key.p8 -pubout -out snowflake_tf_snow_key.pub
```

we open the public key file and we copy the key value (all of it): 
example :  

``` bash
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArW7MkZcU9XhKkOQz5iOT
hfStGeBGHn2j3q+U8u7BoS0O4DZohT0Gy8n3CgZ9xzL8NmYMTtFf8yR7dghA4QOI
pjwnhNdqjB6XxlA7NffzUGrJGLeME1rP2TYOn8aK1aPx8WnX+6m9oChZ2pXmU5np
hGJbbkyp+5MiUE0uXeBFL9U9KMAYAkYI8U8HQIDAQAB
-----END PUBLIC KEY-----
```
The private key is in the snowflake_tf_snow_key.p8 file. We can base encode it (best p)ractice when using cicd tools) and decode it in the provider section before using it !  ==>  hence the base64decode(var.SNOWFLAKE_PRIVATE_KEY) !  

The authentication method to use by terraform to connect to snowflake is JWT !  

**Pay attention to the synchronisation of the machine used since false time zone can generate a token that is already expired !**  ==> we can generate the same token that terraform generate using JWT library in python :  

![image](https://github.com/user-attachments/assets/c533bf77-0ebb-4a1f-9111-0dd195eaa73e)  

The code simply reads the private key and generate the token same as terraform does. Then we can check the token validity using the JWT debugger : **https://jwt.io/**  

Then in snowflake we alter the users (or create them is not yet created) to affect keys to them (two users in our case dev user and prod user) :  

```SQL
USE ROLE ACCOUNTADMIN;

CREATE USER TERRAFORM_SRV
    TYPE = SERVICE
    COMMENT = "Service user for Terraforming Snowflake"
    RSA_PUBLIC_KEY = "<RSA_PUBLIC_KEY_HERE>";

GRANT ROLE SYSADMIN TO USER TERRAFORM_SRV;
GRANT ROLE SECURITYADMIN TO USER TERRAFORM_SRV; # needed to create roles

ALTER USER ZACKHADD SET RSA_PUBLIC_KEY = "<RSA_PUBLIC_KEY_HERE>"; # to give access also to my user profile if i want to lanch terraform on my machine
```

**Note that we will use some variables to be read from the environnements either in local or github actions environnements (dev and prod each has its own variables) such as the SNOWFLAKE_USER (one for dev and the other for prod), the SNOWFLAKE_PRIVATE_KEY and ARM_SAS_TOKEN**  

Locally we have only once environnement variable declared which is the SAS key. The snowflake user and other variables are only specified in the .tfvars file. However we declare in the github actions the needed variables per environnement to override the default value and use the correponding one (this is to be specified in the github actions cicd workflow to tell terraform which environnements to use) !  

![image](https://github.com/user-attachments/assets/0478d989-f459-45b3-9e23-2ed7b17ce491)  

In our example i use the default value of "ZACKHADD" as user but in the github actions it will be overwritten since we set environnement variables containing other values depending on the environnement :  

![image](https://github.com/user-attachments/assets/57b44482-7405-4251-ae9f-08ce05b72424)  

**Note that the variables declared with names that terraform knows override the one set in the default section of variables files or in the .tfvars file but the one set inline when we excute the terraform commandes override everything else !**  

Now once all the setup to connect to snowflake is done we can test that locally usin **terraform init** command :  

![image](https://github.com/user-attachments/assets/6c43df08-1f88-4ff2-9d22-212619c93a2d)  

this will generate a sucess message and create the state file as well as some terrform files !  
We can chek the Azure blob storage ro see the knewly create state file :  

![image](https://github.com/user-attachments/assets/75595ae2-dcca-4d36-97b2-1e7756369236)  

Now all is good for the connection part and terraform initialization !  

Now let's see the other files. In our case we use the modules that makes it easy to make changes if the module is to be used in several locations ! we change only the module and all the locations will read that !  

**Note whenever we use variables, these need to be declared ! either in the main file or in a seperate variables file. This only the declaration, we can specify the value in it also but better to use .tfcvars file that will hold the values.  **

The modules also have a similar structure as the root : main, outputs and variables file since we use them in the main file of the module. This means that the variables are to be declared twice in the variables file of the module and the root one ! simply because each main file will check for the variables declaration in its level (root or module) and then search for the values int the .tfvars file.  

the variables file is configured as follows :  

![image](https://github.com/user-attachments/assets/62be6d76-b638-474e-8117-6ff77a2fc137)  

It contains all the variables used in the main file such as connection credentials !  

The modual one is similar but only for variables called in the module main file :  

![image](https://github.com/user-attachments/assets/e3dcb61b-df71-4b18-b56b-0bc803a24a37)  

Main file : databse example :  

![image](https://github.com/user-attachments/assets/47c32edd-e3d0-468a-9ad9-fb8711218aa6)  

Here we can see that the module creates a database with a name that will be retrieved from the .tfvars file in the root since there where we specified the database name value. **This file is environnement specific meaning we have one for dev and another one for prod !**. example of dev :  

![image](https://github.com/user-attachments/assets/7c2526a5-0c1b-4ae6-a045-1862845da4c3)  

Now we can test the locally; using **terraform plan**; what would terraform build if we would to apply the current configuration :  

![image](https://github.com/user-attachments/assets/6fbb3e67-14d6-49de-be31-e3f8126dc78e)  

Here we can see that all went well and terraform would create a database called DEV_DB, a role called DEV_ROLE and a warehouse called DEV_WH :  

![image](https://github.com/user-attachments/assets/d6db3bcf-f797-4523-b3ef-00f78637afaa)  

Here i run the locally the same configuration as the dev environnement !  

The outputs files are for debuging puposes :  

![image](https://github.com/user-attachments/assets/48e60204-2293-4e20-9aaa-9a27299b7dfe)  

That is we see in the result of the plan command the outputs that will be created.  

#### - CICD in github

In this step we need to create a github workflow that will simulate the same work we did locally with all the commands to run and specifying also the variables to use depending on the environnement.  

**Our goal is once we make change to a feature branch locally, we push it to the repo and create a PR that will merge the feature with the dev branch, which will trigger the workflow and run all the terraform commands in the dev environnement. Then another PR will merge the dev with prod and trigger the workflow to run all the terraform commands to create prod environnement objects**:  


        +----------------------+
        | 1. Local Development |
        |    (feature branch)  |
        +----------------------+
                  |
                  v
        +-----------------------------+
        | 2. Push & Create Pull Request|
        |    feature -> dev            |
        +-----------------------------+
                  |
                  v
        +------------------------------+
        | 4. Merge PR to dev branch    |
        +------------------------------+
                  |
                  v                  
        +------------------------------+
        | 3. GitHub Actions Workflow   |
        |    (Triggered on PR to dev)  |
        | - terraform init             |
        | - terraform fmt              |
        | - terraform validate         |
        | - terraform plan (dev env)   |
        | - terraform apply (dev env)* |
        +------------------------------+
                  |
                  v
        +------------------------------+
        | 5. Create PR: dev -> prod    |
        +------------------------------+
                  |
                  v
          +------------------------------+
        | 4. Merge PR to dev branch    |
        +------------------------------+
                  |
                  v                    
        +------------------------------+
        | 6. GitHub Actions Workflow   |
        |    (Triggered on PR to prod) |
        | - terraform init             |
        | - terraform plan (prod env)  |
        | - terraform apply (prod env) |
        +------------------------------+

To achieve this, we will create two branchs : one for Dev and the other for Prod (the main one). This means we will have exclude the environnement specific files when merging branchs like : backend (since it point to the environnement state file) and .tfvars files.  
We can do this by specifying these files in the .gitattributes file :  

![image](https://github.com/user-attachments/assets/024a5570-e764-431a-9d2a-398eccefc5cc)  

This means that each environnement will keep its own files unmodified by the merge process !  

Now we can create the Github workflow that will run the terraform commands in each environnement. This workflow will be parametrized so it can work the same in both environnements.  

Since the terraform commands need to be executed in each **target** branch after the merge is completed to take the appropriate secrets of the corresponding environnement we will devide the workflow into two parts : 

- The first will be triggered by the PR after its completion and will trigger the second workflow in the target branch environnement (to retrieve the appropriate secrets, otherwise it will be triggerd in the base branch and use the wrong secrets)
- The second one contains all the terraform commands

The github workflows are yaml files containing steps to run !  

#### 1st workflow :

```yaml
name: Trigger Dev/Prod Workflow

on:
  pull_request:
    branches:
      - dev
      - prod
    types:
      - closed 
jobs:
  trigger-workflow:
    name: 'Trigger Dev/Prod Workflow'
    if: github.event.pull_request.merged == true  # Only run if the PR was merged
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Print Target Branch
        run: |
          echo "Target Branch: ${{ github.event.pull_request.base.ref }}"
      - name: Trigger Dev/Prod Workflow
        uses: actions/github-script@v6
        with:
#          github-token: ${{ secrets.REPO_PAT }}  # Use the PAT for authentication if needed
          script: |
            try {
              const response = await github.rest.actions.createWorkflowDispatch({
                owner: context.repo.owner,
                repo: context.repo.repo,
                workflow_id: 'terraform_snowflake.yml',  // Name of the second workflow
                ref: '${{ github.event.pull_request.base.ref }}',  // Target branch (dev or prod)
              });
              console.log(`Workflow dispatch successful! Status: ${response.status}`);
            } catch (error) {
              console.error(`Error triggering workflow: ${error.message}`);
              console.error(`Error details: ${JSON.stringify(error, null, 2)}`);
              throw error;
            }

```
The workflow is simple as it runs if a pull request is created to Dev or Prod and only after it's closed. The job trigger-workflow runs if the merge is done and then it creates the VM image "ubuntu-latest" then it performs the commands in the steps :  

- Print the target branch (for debugging)
- triggers the workflow in the target branch called "terraform_snowflake.yml"
- uses try function to throw error if any

#### 2nd workflow :

```yaml
name: Terraform Dynamic Deployment

on:
  workflow_dispatch:
jobs:
  terraform:
    name: 'Terraform Apply'
    runs-on: ubuntu-latest
    environment: ${{github.ref_name }}
    env:
      SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
      SNOWFLAKE_PRIVATE_KEY: ${{ secrets.SNOWFLAKE_PRIVATE_KEY }} 
      ARM_SAS_TOKEN: ${{ secrets.ARM_SAS_TOKEN }} 
      
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0

      - name: Initialize Terraform
        run: terraform init
        working-directory: environment

      - name: Check Terraform Formatting
        run: terraform fmt -recursive # to do it for all the files even in subfolders
        working-directory: environment

      - name: Validate Terraform Configuration
        run: terraform validate
        working-directory: environment
        
      - name: Plan Terraform
        run: |
          terraform plan -var="SNOWFLAKE_USER=$SNOWFLAKE_USER" -var="SNOWFLAKE_PRIVATE_KEY=$SNOWFLAKE_PRIVATE_KEY" -var-file=terraform.tfvars -out=tfplan.out      
        working-directory: environment
        
      - name: Apply Terraform
        run: |
          terraform apply -auto-approve -var="SNOWFLAKE_USER=$SNOWFLAKE_USER" -var="SNOWFLAKE_PRIVATE_KEY=$SNOWFLAKE_PRIVATE_KEY" -var-file=terraform.tfvars
        working-directory: environment
```

The workflow is triggerd by the first one and it runs the terraform commands:  

- it first provisions a ubuntu image then specifies the environnement and its secrests to be used then perfom the terraform steps
- it checkouts first to the root the sets up terraform
- it initializes terraform in the working directory of the environnement
- then it formats all the files to avoid any errors
- then it validates all the files and the configurations
- then it plans the tasks to do in snowflake. Note that we specified the variables to be used inline to overwrite the default ones in the variables and tfvars files (used for local testing). We also specify -out=tfplan.out to tell terraform to save the plan steps and use it in the apply step
- the it applys the plan to create all the ressources needed in the snowflake corresponding environnement (Dev or Prod)

Now let's try this out! First we will push the changes made locally in the feature branch to the remore repository :  

![image](https://github.com/user-attachments/assets/858729d5-2e43-467a-81d1-383e050abce5)  

Now in github we will create the pull request from the feature to the Dev branch then from the Dev to the Prod:  

![image](https://github.com/user-attachments/assets/d0e464ae-8d36-488b-b0c4-5c2a1dbe5a6c)  

![image](https://github.com/user-attachments/assets/1c16b574-6847-4621-918c-a47f59b112a5)  

![image](https://github.com/user-attachments/assets/fde39f6b-3df2-49bf-8e3b-106fb6d55c8a)  

Under the Github actions workflows section we can see that both workflows were triggerd :  

![image](https://github.com/user-attachments/assets/ee7e8090-9b88-413d-990b-7edd9adbf4e2)  

The first one runs in the feature branch and the seconf in the Dev branch (target) and both of them have suceeded !  

we can see the details of the job executed in the same section :  

![image](https://github.com/user-attachments/assets/4e4a62ac-f87e-4682-b9c4-1105811ca3cc)  

![image](https://github.com/user-attachments/assets/dc7de32d-469a-4d84-adc3-691092b3e393)  

and we can see that the apply step have succeeded as expected :  

![image](https://github.com/user-attachments/assets/d1e032e6-af20-4476-bb9b-866f233676a0)  

Now we can check in snowflake to see if the ressources were correctly created :  

The data base DEV_DB was created as expected : 

![image](https://github.com/user-attachments/assets/d054bdac-b8af-4663-94ce-630ff978088b)  

We can also see that in its attributes it says that it was created using terraform :  

![image](https://github.com/user-attachments/assets/2a17d339-653e-46a9-bced-631691f14b62)  

The role also DEV_ROLE:  

![image](https://github.com/user-attachments/assets/63473f76-a25c-4266-9ddb-f7ed11584635)  

And finaly the warehouse DEV_WH also :  

![image](https://github.com/user-attachments/assets/49d8eb3d-97be-4df1-8b69-7c7e8d2b9510)  

The same process will be followed when we create a PR from Dev to Prod !  












