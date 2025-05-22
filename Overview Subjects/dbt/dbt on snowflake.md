# This document presents a use case on how to setup and configure DBT on Snowflake to handel data transformations

We will go trough all the steps neede to configure dbt locally and connect it to snowflake then how to automate the deployment using Github Actions !

## Well first of all what is dbt ?

dbt is a solution that sets on the warehouse and handels **transformations** (Only transformations not loading or unloading !) in a flexible and especially dynamic way ! It gives the possibility to outpass limits of using SQL logics directly in the warehouse solution for instance using variables for dynamic sources, tests, documentation, CDC, **lineage** (super helpful) and so on !  

![image](https://github.com/user-attachments/assets/0c1642ab-864c-472f-b4b1-6b5d84071080)  

It fully supports version control and using the cloud version or combining the core one with a CI/CD tool makes great for automatically the deployments trough all the environnements.  

We simply connect dbt to our data warehouse and write all the transformations there then it will compile the SQL and execute it in the warehouse compute layer !  

## dbt Setup : 

First of all we need to create a virtual environnement to isolate our project so it will not be impacted by any dependencies updates (the virtual environnement will have its own dependencies such as python version, packages and so on, that are not shared with other projects).  

![image](https://github.com/user-attachments/assets/43dbc402-859b-4c0d-9bc3-b57dc4d7b965)  

Now let's activate it : 

![image](https://github.com/user-attachments/assets/82793df4-9fa0-44b8-842a-d32f28115128)  

Now we install the adaptor that will link dbt to snowflake :

![image](https://github.com/user-attachments/assets/c6debe7d-20a0-4911-985d-d5ddaa08f079)  

We can see now (on the left of VScode screenshot) that all our dependencies get installed in the virtual environnement :  

![image](https://github.com/user-attachments/assets/6226ef5a-9b53-479f-9144-fb6b5b775c92)  

Then we initiate the dbt project using *dbt init name_project* :  

![image](https://github.com/user-attachments/assets/d2af61e6-0a33-4b12-b3bb-b2bc4b2b769a)  

It will ask for some connection configurations, which will create a yaml profiles file that will hols these configurations.  
 
![image](https://github.com/user-attachments/assets/fb9f709d-7869-46bc-a501-459866ca900e)  

Then we can check the yaml file in its location (user/.dbt path):  

![image](https://github.com/user-attachments/assets/ca40078b-e7e6-45b4-8b6a-4d8c8b545d1e)  

We can modify the profiles yaml as we like and then test the connection using : dbt debug  

![image](https://github.com/user-attachments/assets/9c08455f-b913-4961-afc8-350f318159d6)  

**Note that we used the key-pair authentication method (details in the terraform part folder !)  **

**Note also that the profiles yaml file database and schema are the default database we work on (read from it and create new items in it !). But we can override this in the models section to read from different sources and write to different targets !**  

By default dbt gives a project structure with the following elements :  

``` cpp
my_dbt_project/  
├── dbt_project.yml  
├── models/  
│   ├── staging/  
│   ├── marts/  
├── snapshots/  
├── seeds/  
├── macros/  
├── tests/  
├── analyses/  
└── target/  (auto-generated)  
```

| Component         | Purpose                                      | Connected To             |
| ----------------- | --------------------------------             | ------------------------ |
| `dbt_project.yml` | Project config                               | All components           |
| `models/`         | SQL logic (core transformations)             | `ref()`, macros, tests   |
| `snapshots/`      | Historical change tracking (for SCD 2)       | models                   |
| `seeds/`          | Static data from CSVs                        | models                   |
| `macros/`         | SQL functions / templates                    | models, tests, snapshots |
| `tests/`          | Data quality checks                          | models, seeds, snapshots |
| `analyses/`       | Non-materialized SQL                         | None (for dev/debug use) |
| `target/`         | Compiled output (auto-generated)             | Debugging, runtime logs  |  


### 1. dbt_project.yml — 🧠 The Project Brain  

#### Purpose:
This YAML file defines global settings for your project, like model directories, naming conventions, and materialization defaults.

#### Key fields:

```yaml
name: my_dbt_project
version: '1.0'
profile: my_profile
model-paths: ["models"]
target-path: "target"
How it connects to others:
```

- Tells dbt where to find models (model-paths)

- Connects to profiles.yml via the profile key

- Affects how dbt compiles and runs all components

### 2. models/ — 📊 Your Data Transformations:

#### Purpose:
This is where you write your SQL models. These are select statements turned into views or tables in your warehouse.

#### Structure (recommended):

```kotlin
models/
├── staging/   ← Raw → Cleaned data
├── marts/     ← Staging → Business logic
we can add also other things such as intermediate
```
#### How it connects to others:

- Models use ref('another_model') to refer to each other

- Controlled by dbt_project.yml and profiles.yml

- Can reference macros, seeds, and snapshots

### 3. snapshots/ — 🕰️ Track Slowly Changing Data

#### Purpose:
Snapshots let you track changes over time in tables that change slowly (like customer records).

Example:

```sql

{% snapshot customer_snapshot %}
...
{% endsnapshot %}

```
#### How it connects:

- Snapshot results are stored in your data warehouse

- They can be used as inputs to models (via ref())

### 4. seeds/ — 🌱 Static CSV Data
#### Purpose:
CSV files that dbt loads into your data warehouse as static tables (great for lookup tables or testing data).

Example:

```cpp
seeds/
├── country_codes.csv
```
#### How it connects:

- Seeded tables can be referenced in models using ref('country_codes')

- Controlled by dbt_project.yml

### 5. macros/ — 🧩 Reusable SQL Functions
#### Purpose:
Macros are templated SQL functions using Jinja, to avoid repetition and enforce standards.

Example:

``` sql
{% macro is_not_null(column) %}
    {{ column }} IS NOT NULL
{% endmacro %}
```

#### How it connects:

- Used inside models, tests, or other macros
- Can be included in conditionals or loops in SQL files

### 6. tests/ — 🧪 Data Quality Checks
#### Purpose:
You define tests to validate assumptions (e.g. no nulls, unique IDs). dbt also supports custom tests.

Example (in model file):

``` yaml
models:
  - name: customers
    columns:
      - name: id
        tests:
          - unique
          - not_null
```
#### How it connects:

- Applied to models or sources
- Use ref() to test downstream or upstream tables
- Can be custom-built using macros

### 7. analyses/ — 📑 Ad Hoc SQL Reports
#### Purpose:
Write ad hoc SQL queries or research analysis that doesn't get materialized like models.

Example:

``` sql
-- analyses/customer_growth.sql
SELECT ...
```
#### How it connects:

- Useful during development or for debugging
- Doesn’t get run via dbt run, but can be compiled with dbt compile

### 8. target/ — ⚙️ Build Output (Auto-Generated)
#### Purpose:
This folder is created by dbt and holds compiled SQL, logs, and artifacts from your latest run.

#### How it connects:

- Helpful for debugging: see exactly what SQL dbt sends to your warehouse
- Not meant to be edited

### 🔁 How It All Ties Together (Flow Overview)
- Raw data is loaded into your warehouse (outside dbt).
- Staging models clean and standardize the raw data.
- Marts models apply business logic to create curated datasets.
- Snapshots capture historical changes to tables.
- Seeds provide static reference data.
- Tests ensure quality of all models.
- Macros enable reusability and cleaner logic.
- dbt run compiles it all into SQL, respecting dependency order (via ref()).

The model folder can be developed to suite more complex real world use cases :  

```
dbt_project/
│
├── models/
│   ├── staging/
│   │   ├── source_1/
│   │   │   ├── stg_source_1_table_a.sql
│   │   │   └── stg_source_1_table_b.sql
│   │   └── source_2/
│   │       └── stg_source_2_table_x.sql
│   │
│   ├── intermediate/
│   │   └── combine_or_enrich_data.sql
│   │
│   ├── marts/
│   │   ├── core/
│   │   │   └── dim_customers.sql
│   │   ├── finance/
│   │   │   └── fct_revenue.sql
│   │   └── marketing/
│   │       └── fct_campaigns.sql
│   │
│   └── schema.yml  ← docs/tests grouped here or per folder
│
├── snapshots/
│   └── dim_customers_snapshot.sql
│
├── seeds/
│   └── country_codes.csv
│
├── macros/
│   └── custom_macros.sql
│
├── analyses/
│   └── ad_hoc_analysis.sql
│
├── tests/
│   └── custom_tests.sql
│
├── dbt_project.yml
└── packages.yml
```
We can set diffirent steps like staging, intermediate and marts (marts are the gold part dim and facts or data warehouse more close to the business). In the **schema.yml** (we can name it as we want) we can specify several sources to be used in the models (databases, schemas and tables) :  
#### Sources : 

```yaml
version: 2

sources:
  - name: salesforce
    database: raw_data
    schema: salesforce
    tables:
      - name: contacts

  - name: shopify
    database: external_data
    schema: shopify
    tables:
      - name: orders
```
The user and warehouse need to have permissions to use the objects mentioned here !  

#### Documentation & basic tests : 

We can also add in the schema yaml file documentation and basic tests (constraints to verify not complexe tests. These latter can be set in the test folder using SQL logic) of each model :  

```yaml
models:
  - name: dim_customers
    description: "Dimension table for customer info"
    columns:
      - name: customer_id
        description: "Primary key"
        tests:
          - unique
          - not_null
```
#### Exposures : 
Another type of documentation that list all the items that depends on the objects created using dbt (very helpful for impact analysis and lineage).  

```yaml
exposures:
  - name: customer_dashboard
    type: dashboard
    description: "Exec dashboard showing customer KPIs"
    depends_on:
      - ref('dim_customers')
      - ref('fct_orders')
    owner:
      name: Jane Smith
      email: jane@company.com
```

#### Metrics :

Metrics are reusable business logic defined centrally, to standardize KPIs across tools. Only supported in dbt Cloud (not Core) with the semantic layer.  

```yaml
metrics:
  - name: total_revenue
    label: "Total Revenue"
    model: ref('fct_orders')
    calculation_method: sum
    expression: revenue
    time_grain: month
    description: "Sum of revenue by month"
```

#### Groups :  

define logical or permission-based ownership of models.  
Used with:
- Access control in dbt Cloud
- access: private/public/protected in models

```yaml
groups:
  - name: finance_team
    owner:
      name: Finance Owner
      email: finance@company.com
```

In the model : 

```yaml
models:
  - name: fct_revenue
    access: protected
    group: finance_team
```
This gives ownership and controls who can use or depend on that model.  

Full example :  

```yaml
version: 2

# 1. Define a source table from your raw database
sources:
  - name: salesforce
    database: raw_data
    schema: salesforce
    tables:
      - name: contacts
        description: "Raw contact data from Salesforce"
        columns:
          - name: id
            description: "Unique contact ID"
            tests:
              - not_null
              - unique
          - name: email
            description: "Email address of the contact"

# 2. Document your model (transformed table)
models:
  - name: dim_customers
    description: "Cleaned and deduplicated customer dimension"
    access: protected
    group: marketing_team
    columns:
      - name: customer_id
        description: "Primary key for customer"
        tests:
          - not_null
          - unique
      - name: email
        description: "Customer's email address"
        tests:
          - not_null
      - name: signup_date
        description: "Date when customer signed up"
    meta:
      owner: "marketing@company.com"
      pii: true

# 3. Define a metric (for dbt Cloud Semantic Layer)
metrics:
  - name: total_customers
    label: "Total Customers"
    model: ref('dim_customers')
    calculation_method: count
    expression: customer_id
    time_grain: day
    description: "Total number of customers per day"
    type: simple
    timestamp: signup_date

# 4. Define an exposure (e.g., a dashboard)
exposures:
  - name: customer_dashboard
    type: dashboard
    description: "Business-facing dashboard tracking customer metrics"
    maturity: high
    url: https://tableau.company.com/dashboard/customers
    depends_on:
      - ref('dim_customers')
    owner:
      name: Jane Smith
      email: jane.smith@company.com

# 5. Define a group (for access control)
groups:
  - name: marketing_team
    owner:
      name: Marketing Analytics Lead
      email: marketing@company.com
```

#### Freshness : 

Another useful feature is the freshness that can tell us about how fresh our data is. For example we can set a warnning if data hasn't been loaded or refreshed after 2 days or so !  

```yaml
version: 2

sources:
  - name: salesforce
    database: raw_data
    schema: salesforce
    freshness:
      warn_after: {count: 24, period: hour}
      error_after: {count: 48, period: hour}
    loaded_at_field: _loaded_at  # column in the source table that shows when it was loaded
    tables:
      - name: contacts
        description: "Raw contact data from Salesforce"
        columns:
          - name: id
            description: "Unique contact ID"
            tests:
              - not_null
              - unique
          - name: email
            description: "Email address of the contact"
```
- loaded_at_field: The timestamp column in your raw source that tracks when the record was loaded.
- warn_after: dbt will warn if the data is older than this.
- error_after: dbt will fail the check if the data is older than this.

We can check the freshness using **dbt source freshness** or when we run dbt : dbt run. This will fail or warn us during run !  

## Airbnb data example :

Now that the full setup is done we can use a real example using data from the Airbnb site : https://insideairbnb.com/fr/get-the-data/

![image](https://github.com/user-attachments/assets/ef2faa15-93e5-4c5e-9091-d7d9552e3d82)  

### Loading data : 

We can either use the compressed csv files and load them into an internal storage in snowflake or use the public S3 bucket available for berlin data : 

Reviews table loading example :  

```SQL
COPY INTO raw_reviews (
        listing_id,
        date,
        reviewer_name,
        comments,
        sentiment
    )
from
    's3://dbtlearn/reviews.csv' FILE_FORMAT = (
        type = 'CSV' skip_header = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    );
```

Note that we can create an external stage that points directely to the files in the S3 bucket and explore data before lading it ! :  

```SQL
CREATE OR REPLACE FILE FORMAT airbnbs3_file_format
    TYPE = 'csv'
    skip_header = 1 
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
; # This will make it easy to parse the files! no need to hard code each time the parsing method and options

CREATE OR REPLACE STAGE AIRBNB_S3
  URL = 's3://dbtlearn'  #No need to specify any integration since the S3 is public
  FILE_FORMAT = airbnbs3_file_format; 
```

![image](https://github.com/user-attachments/assets/67b1d2fb-705f-4d95-b0e2-11e043f1194d)  

Full code to load and create tables on : [https://github.com/dlt-hub/dlt  ](https://github.com/nordquant/complete-dbt-bootcamp-zero-to-hero/blob/main/_course_resources/course-resources.md)  

### Project structure :  
The structure will be as follows :  

![image](https://github.com/user-attachments/assets/8a95d859-81f7-408b-ae7d-34da9bc08b7d)  

The models folder will contain subfolders per source (if we have multiple sources) and for each source we will have models files in sql extension.  
We should also averride the default database and schema to adapt that for each model if we have seperate sources sources we work wich. We can do this using a **dbt_project.yml** file where we configure what materialization for each model and what **target** schema/database to be used and also **schema.yml** file where we specify all our **sources** that we will mention dynamically in our models logic using a function called source('schema','table') !  

In our case we have one source and target database which is the airbnb database (meaning we read from tables in airbnb and create tables and views in the same database) :  

![image](https://github.com/user-attachments/assets/24a26486-e268-4f89-abe5-c7bb4032a5b4)  

So for the staging models we set the materialization to views (create views), the target database AIRBNB and the schema RAW. **Note that we can add another subfolder for another schema and add it the same way we dod for src_airbnb and change RAW by the one we want!**  
This means that dbt will create a view using the SQL we will define in our model in the RAW schema of the AIRBNB database.  

**We can also use a more granular mode changing the config at the model's file level using :**  
```SQL
#At the top level of the file
{{ config(materialized='table') }}
```

**Note that commenting jinja code is special and not like sql or other types:**  

```SQL
#{{ config(materialized='table') }} -- dbt still reads this !!
{# {{ config(materialized='table') }} #} -- this is the right way to comment jinja code !
```

**⚠️ Note that by default, all models, seeds, and snapshots are materialized as transient tables (cheeper) in Snowflake unless configured otherwise.**  

### Creating Models in dbt : 

Now comes the part of transforming data ! the transformation we will create will be materialized as views before loading them later into the final tables : marts (facts and dimensions).  

Let's start by doing some renaming to the columns of our 3 tables : listings, hosts and reviews :  

#### Listings : 

We test the transformations in snowflake (only selects) to check the result before running it in dbt or if we use dbt cloud we can use the preview section :  

```SQL
WITH raw_listings AS (
    SELECT
        *
    FROM
        AIRBNB.RAW.RAW_LISTINGS
)
SELECT
    id AS listing_id,
    name AS listing_name,
    listing_url,
    room_type,
    minimum_nights,
    host_id,
    price AS price_str,
    created_at,
    updated_at
FROM
    raw_listings 
```

![image](https://github.com/user-attachments/assets/bd34a84c-fb12-4bde-a105-cfbe5eecd90d)  

**We can also use dbt Power User that gives so much features including visualize the result of a query directly in VScode along with other features:**  

![image](https://github.com/user-attachments/assets/02e8e273-3137-4264-b9ea-91d3f8c93f6e)  

Once we are sure of the query we can run : dbt run to deploy  

![image](https://github.com/user-attachments/assets/9504d827-ee84-44a3-814f-2ff8b21520b0)  

**Note that by default, if the schema in the profiles.yml file is different than the schema used in dbt_project.yml file, dbt does not override. It prefexes the second one with the first. Behind the scene it's a macro :**  

```jinja
{{ target.schema }}_{{ custom_schema_name }}
```
It will create a new schema :  

![image](https://github.com/user-attachments/assets/4e30dac6-c7ef-4077-a2f7-f8dc9ea42d9c)  

So we need to change this behaviour before running dbt by changing the built-in macor that dbt uses to generate schemas names:  

```jinja
{% macro generate_schema_name(custom_schema_name, node) %}
    {{ custom_schema_name }}
{% endmacro %}
```

**This macro is to stored as .sql file under macros folder !  **  
Now we can see in the logs info that it created a view in the target schema RAW that replaces the default one in the profiles config file.  

![image](https://github.com/user-attachments/assets/665dae68-59b3-4c46-b47c-2b255ed52b26)  

In snowflake we can check the deployment :  

![image](https://github.com/user-attachments/assets/147e653a-dd19-41f7-944b-b30da48850bd)  

#### Reviews : 

We so the same thing with the reviews raw data :  

```SQL
WITH raw_reviews AS (
    SELECT
        *
    FROM
        AIRBNB.RAW.RAW_REVIEWS
)
SELECT
    listing_id,
    date AS review_date,
    reviewer_name,
    comments AS review_text,
    sentiment AS review_sentiment
FROM
    raw_reviews
```

![image](https://github.com/user-attachments/assets/1f3f0dec-d6a8-4758-acb1-936f46555f46)  

We check in snowflake the deployment :  

![image](https://github.com/user-attachments/assets/5ea6ad5d-24f1-4801-8918-cf1dc5462866)  

We can see the also the DDL generated at snowflake warehouse ! dbt translate the code in the model file to CREATE OR REPLACE VIEW.  

#### Reviews : 

```SQL
WITH raw_hosts AS (
    SELECT
        *
    FROM
       AIRBNB.RAW.RAW_HOSTS
)
SELECT
    id AS host_id,
    NAME AS host_name,
    is_superhost,
    created_at,
    updated_at
FROM
    raw_hosts
```
When we run the dbt run it runs again all the models so it recreates everything !  

![image](https://github.com/user-attachments/assets/8beec23b-102a-4d1f-b6a3-0e8f28908b50)  

if we want to run only specific models we use :  

```cmd
dbt run --select "file_name.sql"
```

![image](https://github.com/user-attachments/assets/c4a66bbd-e025-44d8-9ad8-c417131fe0c7)  

**dbt doc:**  
https://docs.getdbt.com/reference/commands/run  
https://docs.getdbt.com/reference/node-selection/syntax  

Now all our **SILVER** views are constructed from the RAW tables (BRONZ).  

#### Listings dimension: 

Here we will build the GOLD layer using dimension tables and the source of these models will be the silver layer.  
To do so, we will create a new folder under **Marts** that we will call **aibnb_gold** (in case we want to build different gold layers in separate schemas) and we create the transformation for the the dimension (replacing some values and adding CASE operations):  

![image](https://github.com/user-attachments/assets/a542e2f8-fa89-4751-b96c-fb971adc65cf)  

Since this is a new folder, we need to update the dbt_project.yml file to point to this new folder and specify the materialization type.  

![image](https://github.com/user-attachments/assets/8dc34634-1fc6-46d4-87bb-95d3e241e9e1)  

Then we can check in snowflake :  

![image](https://github.com/user-attachments/assets/f85f7495-459e-4cb5-a0ac-2fb998cb33dc)

⚠️**Note that i updated the code to add the proper schema for the stging files that are normaly in this case SILVER objects!**  

![image](https://github.com/user-attachments/assets/31e4881b-7dce-4fb5-a1d7-db454cb41c80)  

![image](https://github.com/user-attachments/assets/67b50418-4b3a-430c-9590-18ab55e38736)  

#### Hosts dimension: 

We do the same thing for the host dimension:  

![image](https://github.com/user-attachments/assets/2e7b1e20-9882-47b0-8d56-a7ea79435c1e)  

#### Join the two dimensions to single dimension :  

To be consistent with the star schema model, we need to join the two dimensions to a single one that will be used to filter the fact table later on. However, since this table will be queried frequently, we will materialize it as a table :  

```sql
{{
  config(
    materialized = 'table',
    )
}} -- file level configuration, as the one in dbt_project.yml is view materialization 

WITH h AS (
    SELECT * FROM 
    {{ ref('dim_hosts') }}
)
, 

l AS (
    SELECT * FROM
    {{ ref('dim_listings') }}
)

SELECT
l.LISTING_ID	
,l.LISTING_NAME	
,l.ROOM_TYPE	
,l.MINIMUM_NIGHTS	
,l.HOST_ID	
,l.PRICE	
,l.CREATED_AT AS LISTING_UPDATED_AT
,l.UPDATED_AT AS LISTING_CREATED_AT
,h.HOST_NAME
,IS_SUPERHOST
,h.CREATED_AT AS HOST_CREATED_AT
,h.UPDATED_AT AS HOST_UPDATED_AT
FROM l 
LEFT JOIN h ON l.HOST_ID = h.HOST_ID
```

![image](https://github.com/user-attachments/assets/fa6d2da4-1f8f-45d8-a53e-a84eb2288403)  

![image](https://github.com/user-attachments/assets/ba91e7a4-c483-488a-b11f-cf328c20ec32)  


#### Incremental materialization :

We have seen that in dbt we have several materializations : view, table and ephemeral (CTEs) but also we have incremental materialization (a table).  

In dbt, when using incremental models (materialized='incremental'), there are different incremental_strategy options that determine how dbt handles new data compared to what's already in the target table.  

#### 1. insert_overwrite

This strategy overwrites partitions of data rather than updating or inserting individual rows.  

When to use :  
- We use this strategy when working with partitioned tables (especially in BigQuery or Snowflake).
- You want to fully replace partitions (e.g., a day, month) instead of appending or merging.

``` sql
{{ config(
    materialized='incremental',
    incremental_strategy='insert_overwrite',
    partition_by={'field': 'event_date', 'data_type': 'date'}
) }}

SELECT *
FROM {{ source('app', 'events') }}
```

#### 2. merge (Default in Snowflake, Databricks)

with this strategy, dbt generates a MERGE statement that updates matching rows and inserts new ones based on a unique_key.  
The equivalent sql command is : 

``` sql
MERGE INTO analytics.customers AS target
USING (
    SELECT
        customer_id,
        first_name,
        last_name,
        updated_at
    FROM raw.app_customers
) AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN UPDATE SET
    customer_id = source.customer_id,
    first_name = source.first_name,
    last_name = source.last_name,
    updated_at = source.updated_at

WHEN NOT MATCHED THEN INSERT (
    customer_id,
    first_name,
    last_name,
    updated_at
) VALUES (
    source.customer_id,
    source.first_name,
    source.last_name,
    source.updated_at
);
```

**Note that this is not the same thing as SCD2 where we need other columns to be updated in a costum way such as valid_from, valid_to and is_current or current_flag**  

When to use : 
- We want dbt to handle deduplication and updates.
- The warehouse supports MERGE (e.g., Snowflake, BigQuery, Databricks).


```sql

{{ config(
    materialized='incremental',
    incremental_strategy='merge',
    unique_key='id' -- We can specify multiple keys if we want
) }}

SELECT id, name, email
FROM {{ source('crm', 'customers') }}

```

#### 3. delete+insert (Default in Postgres & Redshift)

dbt here deletes existing rows based on unique_key, then inserts the new records.  

When to use : 
- When using Postgres or Redshift.
- MERGE is not available.
- We want a basic upsert behavior.


```sql

{{ config(
    materialized='incremental',
    incremental_strategy='delete+insert',
    unique_key='user_id'
) }}

SELECT user_id, user_name
FROM {{ source('app', 'users') }}

```

#### 4. microbatch Strategy
The microbatch strategy is designed for efficiently processing large time-series datasets by dividing the workload into smaller, manageable batches based on a specified time column. This approach enhances performance and resilience, especially when dealing with substantial volumes of data.

**Key Features**:
- Time-Based Batching: Processes data in discrete time intervals (e.g., daily, hourly) defined by an event_time column.
- Automatic Filtering: dbt automatically applies filters based on the event_time to process only the relevant data for each batch.
- Parallel Execution: Supports parallel processing of batches, improving efficiency.
- Resilience: If a batch fails, it can be retried independently without affecting other batches.

To implement the microbatch strategy, we can configure our model as follows:

```sql

{{ config(
    materialized='incremental',
    incremental_strategy='microbatch',
    event_time='event_timestamp',
    batch_size='day',
    lookback_period=3
) }}

SELECT
    id,
    event_type,
    event_timestamp,
    user_id
FROM {{ ref('stg_events') }}
```

**Explanation**:

- event_time: Specifies the timestamp column used for batching.
- batch_size: Defines the granularity of each batch (e.g., 'day', 'hour').
- lookback_period: Determines how many past batches to reprocess, useful for handling late-arriving data.

We need to ensure that upstream models also have the event_time configured to enable automatic filtering and efficient processing. 

**Considerations:**  

- Adapter Support: The microbatch strategy is supported in dbt Core v1.9 and later. Ensure your data warehouse adapter supports this strategy.
- Batch Granularity: Currently, the default granularity is daily. Adjust batch_size as needed, but be aware of potential limitations in granularity support.
- Resource Management: Processing a large number of small batches can lead to increased overhead. Monitor and adjust batch_size and lookback_period to balance performance and resource utilization.

#### 5. Custom Incremental Logic (Using is_incremental())

in this approach we manually write the logic to filter data during incremental runs using is_incremental().

When to use :
- When having a time-based column (like created_at) to filter new rows.
- We don't need to update existing rows.
- We have SCD 2 dimensions
- We want full control over incremental behavior.

```sql
{{ config(materialized='incremental') }}

SELECT *
FROM {{ source('raw', 'transactions') }}
WHERE 1=1
{% if is_incremental() %}
  AND transaction_date > (SELECT max(transaction_date) FROM {{ this }})
{% endif %}
```
**This approach does not require unique_key or any strategy setting.**  

**Strategies overview**:  

| Strategy           | Updates Existing Records     | Inserts New Records | Requires `unique_key` | Best For                                  |                                                                                        |
| ------------------ | ---------------------------- | ------------------- | --------------------- | ----------------------------------------- | -------------------------------------------------------------------------------------- |
| `append`           | ❌ No                         | ✅ Yes               | ❌ No                  | Simple appends without updates            |                                                                             |
| `merge`            | ✅ Yes                        | ✅ Yes               | ✅ Yes                 | Upserts where `MERGE` is supported        |                                                                                        |
| `delete+insert`    | ✅ Yes                        | ✅ Yes               | ✅ Yes                 | Warehouses without `MERGE` support        |                                                                                        |
| `insert_overwrite` | ❌ No (overwrites partitions) | ✅ Yes               | ❌ No                  | Partitioned tables in BigQuery, Snowflake |                                                                                        |
| `microbatch`       | ✅ Yes                        | ✅ Yes               | ✅ Yes (recommended)   | Large time-series datasets

#### Reviews fact: 

For the facts, the materialization will be different. Normaly facts contains a huge number of rows compared to dimensions and recreating the table each time would exessive. That is why we need to do incremental load, meaning that we only append new data.  

**Note that here we talk only about appending data since it's a fact table. Not like a dimension of type SCD2 where we need to UPSERT**  

We can, for example; use the review_date and insert only the data in the source where the date of review is > Max(review_date) in the target.  

We can also use a hash column that will compare all the rows and only insert the non existing ones in the target table.  

```SQL
WITH src_reviews AS (
  SELECT * FROM {{ ref('stg_reviews') }}
)
SELECT * FROM src_reviews
WHERE review_text is not null -- We load only the non null rows

-- here we need to add the logic of incrementation
-- we append data so we can for example use the review_date and insert only the data in the source where the date of review is > Max(review_date) in the target

{% if is_incremental() %}
  AND review_date > (select max(review_date) from {{ this }})
{% endif %}
```

Here we used a custom incremental materialization.  

Now if we insert a new row (test row with a date before the max date) in the the RAW_REVIEWS table which is the source table of our fact table and we rerun the fact_reviews model, the new row should be inserted only if the date is after the max date :  

![image](https://github.com/user-attachments/assets/c742d909-29a1-4daf-b0a5-80edce7e068e)  

The we run the model :  

![image](https://github.com/user-attachments/assets/5664a247-56fc-42a6-9377-c9d6ce1124ad)  

Now we check in snowflake :   

![image](https://github.com/user-attachments/assets/41b03cce-9fb6-4112-b204-c5c21f8f5ec2)  

We can see that the row was not inserted because the review_date is not after the max date review of the table before merge.  

If we insert another row with today's date we can see that it will be inserted :  

![image](https://github.com/user-attachments/assets/957a4ce2-0bc5-42d2-a955-c8365236d4b8)  

The logic of incrementation can be as complexed as we want depending on our use case.  

Also if we want to rebuild the whole table even if we are in incremental mode, we can use : **dbt run --full-refresh**.  

#### Ephemeral materialization: 

This type of materialization can be used for intermediate results that we want to reuse without creating real tables or views in our target data warehouse:  
- A dbt model that is not materialized as a table or view in your database.
- Instead, its SQL is inlined (embedded) into models that reference it via {{ ref() }}.
- It is like a reusable SQL CTE (Common Table Expression).

In the compiled SQL , dbt will inline the SQL of ephemeral_model, like this:  

```
ephemeral_model.sql   --> reusable filtering logic
final_model.sql       --> selects from {{ ref('ephemeral_model') }}
```

```sql
WITH ephemeral_model AS (
    SELECT ...
)
SELECT * FROM ephemeral_model
```

#### Sources and seeds:
**Seeds :**  
In the data warehouse, data can be ingested using two different ways: Using what we call sources in dbt which are the applications and other databases or seeds. Seeds are just local files in dbt that we can use to populate our data warehouse.  

To add seeds we can either use the url to the files we want and add them in the seeds file or drag it there manually:  

![image](https://github.com/user-attachments/assets/70657041-6656-4f4c-b9a3-c3b8f0c529c6)  

```cmd
curl "https://dbtlearn.s3.us-east-2.amazonaws.com/seed_full_moon_dates.csv" -O seeds/seed_full_moon_dates.csv
```

Then if we want to load the files in snowflake we would just run : *dbt seed* 
This will use the default configs in the **profiles.yml** file unless we specify other configs in the **dbt_project.yml** file :  

```yaml
seeds: -- we use the seeds category like we used models before
  my_project_name:
    customers.csv:
      +schema: customer_data
      +quote_columns: true
```

We can also add other options such as :  

```yaml
seeds:
  my_project_name:
    +column_types:
      id: integer
      signup_date: date
    +quote_columns: false
    +header: true
```

**Note that for now, dbt supports only csv files.**  

![image](https://github.com/user-attachments/assets/94e9cef4-8cbd-4d64-bbfd-b2d78572c513)  

![image](https://github.com/user-attachments/assets/4cd68473-06fb-4540-b9ed-7c099ad91849)  

**Note that we can later reference seeds csv files in the models just like we reference other models.**  

**Sources :**  

Sources are just an entity we create so that we can structure more the project and make more dynamic.  
When we reference in our queries the *FROM database.schema.table* part, we hard code these elements. But what if we this source changes ? we would be obliged to do so in every model !  
The best appraoch would be to define a sources object in yaml file that we will call by name in models and if it changes, we will only change it in one place !  

we can define everything in the schema.yml file, or we can create another yaml file under the models folder.  

```yaml
version: 2

sources:
  - name: airbnb
    schema: raw
    tables:
      - name: listings
        identifier: raw_listings

      - name: hosts
        identifier: raw_hosts

      - name: reviews
        identifier: raw_reviews
```

![image](https://github.com/user-attachments/assets/db9c001f-d04a-479f-acc2-3c9d40df0e41)  

Then we modify the models to point to the sources:  

![image](https://github.com/user-attachments/assets/ad6e14c2-e958-4247-90a6-f8943c6fafbf)  

This will structure more the project and make it more dynamic.  

We can also implement a **freshness** feature to notify us during run if data in a source is not up to date !  


```yaml
version: 2

sources:
  - name: airbnb
    schema: raw
    tables:
      - name: listings
        identifier: raw_listings

      - name: hosts
        identifier: raw_hosts

      - name: reviews
        identifier: raw_reviews
        loaded_at_field: date  #Which column that tells us about the freshness of data
        freshness:
          warn_after: {count: 1, period: hour} # warning after how much time
          error_after: {count: 24, period: hour} # error after how much time
     
```

Using **dbt source freshness** we can check the freshness of the sources:  

![image](https://github.com/user-attachments/assets/7b21662f-a89a-4c48-9507-35aff26e3caa)  

#### Snapshots (for automatique SCD2):  

Snapdhots are an important feature in dbt that makes it easy to deal with SCD2 dimensions. It handels automatically the SCD2 tables by inserting only new or updated rows and adding the *valide_from* and *valid_to* comparing values of **unique_id** which can be a single or multiple columns. Same as we did with models, we can create sql files with the select statement of the table to create and dbt will handel the process behind the scene!  

We define the structure of snapshot as follows:  

```jinja

{% snapshot my_snapshot %} # name of the snapshot

# the config can be specified in the dbt_project.yml file (general ones like schema) or here (specific like columns)

{{  
  config(
    target_schema='snapshots',
    unique_key='id',
    strategy='timestamp',
    updated_at='last_updated_at'
  )
}}

SELECT * FROM {{ source('my_source', 'my_table') }}

{% endsnapshot %}

```

This will tell dbt to check each time the source and compare it with the SCD it creates and only insert new or updeted rows!  

In our case we can try that by creating a SCD2 from the raw listings table:  

```sql
{% snapshot scd_raw_listings %}

--the confings bellow can also be defined in the dbt_project.yml level

{{ 
   config(
       database= 'AIRBNB', 
       target_schema='GOLD',
       unique_key='id',
       strategy='timestamp', -- we can change this to check if we want
       updated_at='updated_at',
       invalidate_hard_deletes=True
   )
}}

select * FROM {{ source('airbnb', 'listings') }}

{% endsnapshot %}

```

![image](https://github.com/user-attachments/assets/c656a4d8-e8a7-434a-b98c-e1b18875466f)  

In snowflake we can check the created table :  

![image](https://github.com/user-attachments/assets/76fcb49d-d9ea-4411-b0e4-026d48058bce)  

We can notice that by default, the table created is transient (default behaviour in snowflake). We can change that by adding in the confg : 

**permanent=true** 

Now we can check the behaviour of the snapshot by changing values in a row in the source table and re-run dbt snapshot:  

```SQL
UPDATE AIRBNB.RAW.RAW_LISTINGS SET MINIMUM_NIGHTS=30,
    updated_at=CURRENT_TIMESTAMP() WHERE ID=3176;
```

![image](https://github.com/user-attachments/assets/4397a327-04f6-4b5c-988c-6e43cf46ee08)  

After running dbt snapshot:  

![image](https://github.com/user-attachments/assets/4179fc7f-91a8-487e-9d30-b0c5ea19fb58)  

We can see that dbt added the changed row and updated the valid_to column!  

**Again, this is the automatique behaviour handeled by dbt on snowflake, it may be different for other tools like databricks. If we want custom logic, we will need to implement custom materialization!**  

Also, we can change the strategy if we don't have a date filed that indicates when the row was updated. We can use the **check** strategy rather than the **timstamp**. In this case, dbt will check values in the specified columns (hash them) and if they change it inserts data!  

```jinja
config(
  strategy='check',
  unique_key='id',
  check_cols=['col1', 'col2'] -- Alternatively, we can use check_cols='all'
)
```
We can add another option to handel deleted data : **invalidate_hard_deletes=True**. If the row no longer exists in source, dbt will mark it as invalid (sets dbt_valid_to).  

#### Tests implementation:  

In dbt, the most helpful part would be test implementation. This feature helps building tests to run againt all the objects to make sure the are compliant with the rules we set for our data warehouse.  

If a test fails, dbt will show it clearly in the run output and optionally fail the pipeline.   

There are two types of tests:  
- 1. Generic Tests (Pre-built by dbt)
These are reusable, built-in tests that check common conditions:

We can define them in your schema.yml file like this:  

```yaml
models:
  - name: dim_customers
    columns:
      - name: customer_id
        tests:
          - unique
          - not_null
      - name: email
        tests:
          - not_null
```
Built-in tests:  
| Test                 | What it checks                          |
| -------------------- | --------------------------------------- |
| `not_null`           | No nulls in the column                  |
| `unique`             | All values in the column are unique     |
| `accepted_values`    | Values must be from a specific list     |
| `relationships`      | Foreign key relationship between tables |
| `expression_is_true` | Custom SQL boolean expression           |

- 2. Custom Tests (You create them):  
These are dbt models that return rows when the test fails.

Example:  

```sql
-- to be created in the test folder : tests/no_future_dates.sql

SELECT *
FROM {{ ref('orders') }}
WHERE order_date > current_date
In schema.yml:
```
then in the schema.yml file we call it in the test  

```yaml
models:
  - name: orders
    tests:
      - no_future_dates
```

We can also build a more dynamic test using macros and call it in the schema.yml test part :  

```yaml
models:
  - name: orders
    columns:
      - name: order_date
        tests:
          - my_custom_test:
              column_name: order_date
```

Or if not per column:  

```yaml
models:
  - name: orders
    tests:
      - my_custom_test:
          column_name: order_date
```

Then we create a marco in the macros folder:  

```jijna
-- macros/my_custom_test.sql

{% test my_custom_test(model, column_name) %}
SELECT *
FROM {{ model }}
WHERE {{ column_name }} > current_date
{% endtest %}
```

The macro has two arguments: model (where we call the test), and the column (the one we want to test).  

Then we can run **dbt test** to see the results of test and we can include it after dbt run inside the CICD workflow!  

In our example we can set some built in tests on columns :  

![image](https://github.com/user-attachments/assets/4c36bf48-5bf3-47b9-8bad-f55e45bb75b8)  

We can also add other tests :  

```yaml
models:
   - name: dim_listings
     description: "A view that changes the names of the listings raw table columns"
     columns:
       - name: listing_id
         description: "The primary key for this table"
         tests:
           - unique
           - not_null

       - name: host_id
         tests:
           - not_null
           - relationships:
              to: ref('dim_hosts')
              field: HOST_ID

       - name: room_type
         tests:
           - accepted_values:
              values: ['Entire home/apt',
                        'Private room',
                        'Shared room',
                        'Hotel room']
```
Let's change values in the test to break it and create an error so we can debug it :  

![image](https://github.com/user-attachments/assets/816a2ba5-69a1-4918-ac49-0c0f14346767)  

We can check the file generated to see what is exactly the error generated using *type link_generated_for_file*:  

![image](https://github.com/user-attachments/assets/0686874f-062a-4522-84c1-32713bb2e41b)  

This gives the compiled sql used against snowflake to run the tests.  

We can also create a test file to check if the minimum nights for example is not less than 1:  

![image](https://github.com/user-attachments/assets/54a226cd-f540-491a-8b31-8357ecde0001)  

The query simply should not return any row for the test to pass.  

If we inverse the logic, it will return rows for that specific query and give an error :  

![image](https://github.com/user-attachments/assets/d4bf0ba7-514f-4664-89f8-23b409df2a67)  

#### Macros :  

Macros in dbt are reusable snippets of logic written in Jinja (a templating language). They let us parameterize and dynamically generate SQL code.  

We have built in macros in dbt and we can create custom ones for our needs (for test puposes for example).  

an example of a custom macro could be :  

```sql
{% macro no_nulls_in_columns(model) %}
    SELECT * FROM {{ model }} WHERE
    {% for col in adapter.get_columns_in_relation(model) -%}
        {{ col.column }} IS NULL OR
    {% endfor %}
    FALSE
{% endmacro %}
```

This macro is a loop that checks if once the columns of a model is null !  

Now we can use this macro in a test file under the tests folder and run dbt test --select test_name.sql:  

![image](https://github.com/user-attachments/assets/758ace33-9309-4f6b-9d46-692a5ba5a886)  

We can do the same thing by using a macro as generic test. It will be like a function to call on a model and we will pass column argument to test the a certain condition :  

```sql
{% test positive_value(model, column_name) %}
SELECT
    *
FROM
    {{ model }}
WHERE
    {{ column_name}} < 1
{% endtest %}
```
This is the same test we did before on minimum nights column !  

![image](https://github.com/user-attachments/assets/eb9f8c02-b592-451e-8704-7474f060b367)  

now we can set it in the schema.yml to be used as a test in a model for a specific column:  

![image](https://github.com/user-attachments/assets/eb017425-37ce-47a4-bc37-b64e89c072dc)  

Here it takes the model from the current model we are at and the column name from the column where we call the macro null_column_test which is minimum_nights.  

#### packages:

We can also import third party packages if we like using : https://hub.getdbt.com/  

![image](https://github.com/user-attachments/assets/4919ff67-59a9-4cbb-8efa-054809038c84)  

For example if we need to use hash functions that will generate surrogate keys for us in dbt we can use the dbt_utils package:  

![image](https://github.com/user-attachments/assets/cb2257d0-f154-4a50-a2c0-9e3d18175c56)  

![image](https://github.com/user-attachments/assets/afe2458f-6aa9-454e-9a26-c53c5a2debb6)  

![image](https://github.com/user-attachments/assets/7edbb57e-2722-406c-b2b1-6343673c495c)  

Now to call some packages, we need just to create a root yaml file called packages.yml inside which we will specify all the packages we want to use and dbt will call them :  

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.3.0
```
Then we need to install these packages using : **dbt deps**  

This tells dbt to install the packages we specified in our packages.yml file.  

![image](https://github.com/user-attachments/assets/0462dec3-f130-44eb-87e4-2ee965b2f852)  

Now we can use the functions that come with dbt utils for example to add a surrogate key to the reviews fact table :  

![image](https://github.com/user-attachments/assets/d4086926-3900-4a6b-b82a-1e1a55f90356)  

Once we do the modification by adding the surrogate key, and since the schema will change we need to regenarate the table using : *dbt run --select fact_reviews.sql --full-refresh*  

Now we can see in snowflake that the surrogate key was added successfully :  

![image](https://github.com/user-attachments/assets/c98991dc-791f-4366-949e-5488d3d4a115)  

⚠️ since dbt install the packages in dbt_packages folder in our project by making an https request to the github of the dbt package such as : https://github.com/dbt-labs/dbt-utils/tree/1.3.0/#generate_surrogate_key-source ! sometimes if we are on production and depending on the company proxy and rules, it may change the certificate that handels the downloading of packages using the https call !  In this case we can just manually download them and put them in the dbt_packages folder !  

#### Documentation :  

Documentation is at the heart of what dbt offers, and it makes it possible to describe every object in our data warehouse. We do that simply by adding the *decription* tag in the schema.yml in the models section (remember that we have sources and models there) :  

![image](https://github.com/user-attachments/assets/1aeac0c8-f701-4aa2-9c67-1119f754eb5d)  

Once we are done we can generate the documentation by running : *dbt docs generate*  
This will compile all the documents and generate files (json and html index) that will serve to visualize the documentation.  

Then we run :  *dbt docs serve*  

![image](https://github.com/user-attachments/assets/68d36a99-5b60-4f92-948c-a433a62bc152)  

This will create a local web app using localhost so we can check the docmentation properely :  

![image](https://github.com/user-attachments/assets/850df81e-d4d3-4aa0-a02a-c5eee110ca8d)  

The doc gives details about the databases, projects and groupes. It also gives the lineage of a table :  

![image](https://github.com/user-attachments/assets/4376710a-4ad9-4e10-b083-cc353035fc4d)  

And at the project level (without selecting any table) we can see the project global lineage:  

![image](https://github.com/user-attachments/assets/4227a807-f525-47b8-9228-b2c71807bf9f)  

It gives also the tests and all the objects that dbt supports and their lineage. It also specifies tags also which is so useful !  

**Note that we can filter the DAG base on what we want to see in select and exclude: object+ will show only the object and all the other objects that depends on it and the opposite thing also true:**  

![image](https://github.com/user-attachments/assets/e914cfa1-f155-438f-ac18-b2a84c75a989)  

**The same logic can be applied when we run dbt commands on specific models.**  

We can also add more sophisticated doc using markdowns and images. Rather than just specifiying a simple description in the description tag, we can call a more detailed one. To do that we create a file in models folder that we will call docs and in it we will add detailed docs for every table/column:  

![image](https://github.com/user-attachments/assets/66a9b64d-e224-448c-843a-218814cc2812)  

Here we created the following doc using markdown style :  

```md
{% docs desc_dim_listing_min_nights %}

### This column gives the minimum nights required to rent the property

:w note that for old listings this column might have a value of 0 so in the transformation process we change this to 1

{% enddocs %}
```

We then call this doc in the description option of the corresponding column in the schema.yml file ! for example :  

![image](https://github.com/user-attachments/assets/54e4b278-219f-41b9-8c96-c768efa3c65c)  

Note how we call the description here : '{{ doc("desc_dim_listing_min_nights")}}'  

Then we regenerate the doc and serve to access it in web app style :  

![image](https://github.com/user-attachments/assets/e761e2e1-245b-4fad-8d7f-5eb41bd84f00)  

We can see now that for that column we are having the full description in markdown style !  

We can also add images  in the overview page of the project. To do so we need to create an asset folder that will hold our images and assets in general and we need to point to it using asset-paths in dbt_project.yml file:  

![image](https://github.com/user-attachments/assets/be17a033-1eca-4284-bd09-af85bc8a7c89)  

Now we create an overview.md file where we will put the new project overview content wit images:  

```md

{% docs __overview__ %}

# Airbnb pipeline

Hey, welcome to the Airbnb project documentation!

Here is the schema of our input data:  

![input schema](assets/input_schema.png) -- here we point to the image we want to use !

{% enddocs %}

```

![image](https://github.com/user-attachments/assets/f659b59b-6212-4dfc-9a36-f745d0c26e57)  

Then we regenerate the doc and serve and we can see that the project overview page has been replaced by our overview content:  

![image](https://github.com/user-attachments/assets/0ca2f91d-4c5c-4b35-b25d-4d80e9b23eb9)  

**Note also that the dbt power user extension gives also the possibility to seen lineage and also generate table and column documention**  


#### Analysis :

Analysis is a way to run ad-hoc queries that aren’t part of your normal model pipeline but are still version-controlled and benefit from dbt’s structure. They're typically used for exploratory analysis, report generation, or any SQL queries you want to run consistently without materializing them into tables or views.  

They live in the folder of analysis 

``` bash

project/
│
├── analyses/
│   └── my_analysis.sql

```

And we can run them using :  

``` bash
dbt compile
dbt run-operation run_analysis --args '{"analysis": "my_analysis"}'
```

These queries are compiled and we can use the compiled code later and run it in snowflake for example.  

#### Hooks:  
Hooks on the other hand Hooks are SQL commands that are automatically executed before or after a model, seed, snapshot, or test runs.  
They enable automation of tasks like granting permissions, logging actions, or setting environment-specific configurations.  

We have 4 types of hooks:  

| Hook Type      | Location                     | Runs When                            | Common Uses                   |
| -------------- | ---------------------------- | ------------------------------------ | ----------------------------- |
| `+pre-hook`    | Model file or project config | Before each model/test/seed/snapshot | Setup, logging, temp cleanup  |
| `+post-hook`   | Model file or project config | After each model/test/seed/snapshot  | Permissions, cleanup, logging |
| `on-run-start` | `dbt_project.yml`            | Once at beginning of `dbt run`       | Audit logs, init configs      |
| `on-run-end`   | `dbt_project.yml`            | Once at end of `dbt run`             | Audit logs, email triggers    |

Several benefits we can have using hooks:  

| Benefit                      | Explanation                                                                                       |
| ---------------------------- | ------------------------------------------------------------------------------------------------- |
| **1. Automation**            | Hooks run automatically with `dbt run`, no manual intervention required.                          |
| **2. Consistency**           | Ensures uniform actions (e.g., permissions, cleanup) across all models every time they are built. |
| **3. Version Control**       | Defined in your codebase → tracked in Git → changes reviewed, audited, and deployed safely.       |
| **4. Idempotency**           | Handles operations (like reapplying permissions) that would be lost on `CREATE OR REPLACE`.       |
| **5. Environment Awareness** | With Jinja, you can make behavior dynamic by environment, model, or context.                      |
| **6. Centralized Logic**     | Keeps everything (transforms, security, logging) in one place — your dbt project.                 |

Hooks (pre/post) run automatically as part of your dbt run and ensures that every time a model is rebuilt, the same grants or configurations are applied.  

It also prevents "it worked on dev but not in prod" scenarios !  

We can use an example of hooks in our scenario by creating an analyst role in snowflake and handeling the permissions on tables (modules) in dbt !  

![image](https://github.com/user-attachments/assets/b7869b2b-033d-4a5a-8f3a-380ccb101c33)  

Here we created a new user we called PBI and a new ROLE called ANALYST.  

We grant ANALYST preveleges to be able to use AIRBNB database and GOLD schema at snowflake level. The access to tables however can be specified as hooks dynamically in dbt.  

```SQL
CREATE ROLE IF NOT EXISTS ANALYST;

GRANT ALL ON WAREHOUSE COMPUTE_WH TO ROLE ANALYST;

GRANT USAGE ON DATABASE AIRBNB TO ROLE ANALYST;

GRANT USAGE ON SCHEMA AIRBNB.GOLD TO ROLE ANALYST;

CREATE USER IF NOT EXISTS PBI
     PASSWORD = 'Pbi123@'
     LOGIN_NAME = 'PBI'
     MUST_CHANGE_PASSWORD=FALSE
     DEFAULT_WAREHOUSE = 'COMPUTE_WH'
     DEFAULT_ROLE='ANALYST'
     DEFAULT_NAMESPACE='AIRBNB.GOLD';



GRANT ROLE ANALYST TO USER PBI;

GRANT ROLE ANALYST TO USER ZACKHADD;

```
In the dbt_project.yml file we can add the hook at the level of fact tables if we want that or at the level of the schema GOLD globally which will iterate on all the tables in the GOLD schema using {{this}} operator to grant the role ANALYST select previlege on all its tables !  

![image](https://github.com/user-attachments/assets/27d4148a-83a4-443e-a362-89a8329e1672)  

Now we can check in snowflake :  

![image](https://github.com/user-attachments/assets/4e5c2616-dd6e-49a7-8853-4695e182c4c9)  

Now the role ANALYST can see the tables in the GOLD schema.  

We will use this role we asigned also to user PBI to connect to snowflake using POWER BI and create a visual that will be based on our GOLD tables.  

Once we create the report we can publish it and use the link to create an exposure in dbt that will show us in the documentation : the report, its lineage and the link to follow if we want to see it !  

#### Exposures:  

Exposures are a way to document and track how downstream tools (like dashboards, reports, or applications) use your data models. They help answer the question:  *What BI dashboards or external tools depend on this dbt model?*  

We can create exposures in the schema.yml file or in a seperate file in the models folder:  

```yaml
exposures:
  - name: revenue_dashboard
    label: Revenue Dashboard
    type: dashboard
    maturity: high
    url: https://my.bi.tool/revenue_dashboard
    description: |
      Dashboard showing daily and monthly revenue KPIs.

    depends_on:
      - ref('fct_orders')
      - ref('dim_customers')

    owner:
      name: Data Analyst Team
      email: data-team@example.com

```

In our case we can create a new file called dashboards.yml that will contain all the dashboards or reports related to the project :  

![image](https://github.com/user-attachments/assets/3d0be77a-f525-4eb5-9b3a-2bca2ca8d2e5)  

We regenerate the docs again and we serve to check the documentation of the project :  
![image](https://github.com/user-attachments/assets/e2fb9f57-2fe5-4164-84ad-65effd9e8d7c)  

We can see now that a dashboard exposure was added with the description and all the details with it even the link to the report online using the button *view this exposure*. We can check the lineage of the dashboard since we already specified the depends on option :  

![image](https://github.com/user-attachments/assets/24d3b0e8-9fdc-4a5b-9f01-80c15e39e54e)  

**This can be so helpful for the impact analysis !**  



