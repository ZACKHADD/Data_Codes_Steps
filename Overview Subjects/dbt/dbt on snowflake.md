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
    unique_key='id'
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

We can also use a hash column that will compare all the rows and only insert the non existing one in the target table
