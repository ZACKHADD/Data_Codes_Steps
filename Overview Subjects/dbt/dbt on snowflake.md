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




