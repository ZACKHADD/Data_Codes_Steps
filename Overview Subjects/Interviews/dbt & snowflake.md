## dbt on Snowflake !

##### This file contains the main subjects to cover for a dbt & snowflake interview !

### dbt :

1- Purpose : 

- Why dbt ? a transformation tool that works with nearly any data platform and the nature of its synthax makes it highly portable from a plateform another easily !
- Facilitate the transformation process and provides a lot of functionalitites such as documentation, lineage, macros, tests and so on !

2- Installation

- dbt works on python, so we need python installed, a virtual environment (localy) and pip install dbt-core and also the connector to the data platform for example dbt-snowflake in this case !
- We create a dbt project by executing in the virtual environment : dbt init project_name !
- This will need some inputs such as the account identifier, user name, role, password or key_pair and so on
- This will create a profile.yaml in the user's directory .dbt/
- This file will not be part of the CICD as it contains sensitive data and it will be created on the fly while running the CICD workflow !
- Once generated we can modify it and add several targets if we want such dev enviroment connection infos, staging and so on !

profile.yaml file example :

```

# This file should NOT be committed to version control
# Create this file locally for development, but use environment variables in CI/CD
# Location: ~/.dbt/profiles.yml (local) or generated dynamically in CI

default:
  target: dev
  outputs:
    
    # Development environment
    dev:
      type: snowflake
      account: "{{ env_var('SNOWFLAKE_ACCOUNT') }}"
      user: "{{ env_var('SNOWFLAKE_USER') }}"
      password: "{{ env_var('SNOWFLAKE_PASSWORD') }}"
      role: "{{ env_var('SNOWFLAKE_ROLE', 'DBT_DEV_ROLE') }}"
      database: "{{ env_var('SNOWFLAKE_DATABASE', 'DEV_DATABASE') }}"
      warehouse: "{{ env_var('SNOWFLAKE_WAREHOUSE', 'DBT_DEV_WH') }}"
      schema: "{{ env_var('SNOWFLAKE_SCHEMA', 'DBT_DEV') }}"
      threads: 4
      client_session_keep_alive: false
      query_tag: dbt_dev
      
    # CI environment - uses ephemeral schemas
    ci:
      type: snowflake
      account: "{{ env_var('SNOWFLAKE_ACCOUNT') }}"
      user: "{{ env_var('SNOWFLAKE_CI_USER') }}"
      password: "{{ env_var('SNOWFLAKE_CI_PASSWORD') }}"
      role: "{{ env_var('SNOWFLAKE_CI_ROLE', 'DBT_CI_ROLE') }}"
      database: "{{ env_var('SNOWFLAKE_CI_DATABASE', 'CI_DATABASE') }}"
      warehouse: "{{ env_var('SNOWFLAKE_CI_WAREHOUSE', 'DBT_CI_WH') }}"
      schema: "{{ var('ci_schema', 'DBT_CI_DEFAULT') }}"
      # threads are used for concurency
      threads: 2
      # Client_session argument is used to auto_suspend the waerehouse once the job done
      client_session_keep_alive: false
      # Query tags are so important for monitoring and see the queries executed by dbt in snowflake UI and query history
      query_tag: dbt_ci
      
    # Production environment
    prod:
      type: snowflake
      account: "{{ env_var('SNOWFLAKE_ACCOUNT') }}"
      user: "{{ env_var('SNOWFLAKE_PROD_USER') }}"
      password: "{{ env_var('SNOWFLAKE_PROD_PASSWORD') }}"
      role: "{{ env_var('SNOWFLAKE_PROD_ROLE', 'DBT_PROD_ROLE') }}"
      database: "{{ env_var('SNOWFLAKE_PROD_DATABASE', 'PROD_DATABASE') }}"
      warehouse: "{{ env_var('SNOWFLAKE_PROD_WAREHOUSE', 'DBT_PROD_WH') }}"
      schema: "{{ env_var('SNOWFLAKE_PROD_SCHEMA', 'DBT_PROD') }}"
      threads: 8
      client_session_keep_alive: false
      query_tag: dbt_prod
```

3- Structure of the project 

- The project generate several subfolders :
  ```
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
| Folder/File                  | Purpose                                                             | Mandatory?                        | Notes / Naming Rules                                                                                                                                                                                                                         |
| ---------------------------- | ------------------------------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dbt_project.yml`            | Core project configuration (paths, materializations, version, etc.) | **Yes**                           | Must exist at project root. Name must be exactly `dbt_project.yml`.                                                                                                                                                                          |
| `profiles.yml`               | Database connection info (credentials, target environments)         | **Yes**                           | Usually located at `~/.dbt/profiles.yml`. Profile name must match `profile` in `dbt_project.yml`.                                                                                                                                            |
| `models/`                    | SQL models (transformations)                                        | **Yes**                           | Models can be organized in subfolders (e.g., `staging`, `marts`). `schema.yml` can be inside a folder or a single file for multiple models. File names are arbitrary but should match `name` in `schema.yml` if using tests or descriptions. |
| `models/<folder>/schema.yml` | Schema tests, column descriptions, documentation                    | **No** (but strongly recommended) | Optional, but needed for tests/documentation. Can be one file per folder or multiple files. Must be YAML and follow dbt version 2 structure.                                                                                                 |
| `analyses/`                  | Ad-hoc SQL queries                                                  | No                                | Optional. File names arbitrary. Models here are not materialized automatically.                                                                                                                                                              |
| `tests/`                     | Custom SQL tests                                                    | No                                | Optional. Naming is arbitrary but recommended to be descriptive. Can also place tests in `schema.yml` instead.                                                                                                                               |
| `macros/`                    | Reusable Jinja macros                                               | No                                | Optional. File names arbitrary, but macro names must be unique across project.                                                                                                                                                               |
| `snapshots/`                 | Track slowly changing dimensions (SCDs)                             | No                                | Optional. Snapshot files must follow `{name}.sql`. Configured inside snapshot using `config()`.                                                                                                                                              |
| `seeds/`                     | Static CSV reference tables                                         | No                                | Optional. File names (without `.csv`) become model names for `ref()`.                                                                                                                                                                        |
| `docs/`                      | Extra documentation                                                 | No                                | Optional. Not required for dbt functionality, used for custom docs.                                                                                                                                                                          |
| `target/`                    | Compiled models and artifacts                                       | No                                | Auto-generated by dbt when you run the project. Do not manually edit.                                                                                                                                                                        |
| `dbt_modules/`               | Installed packages                                                  | No                                | Auto-generated if you use `dbt deps`.                                                                                                                                                                                                        |

4 - dbt_project.yaml :
  This file **should be in the project ROOT folder** and it contains all configurations of the dbt project needed for it to work such as where the models, macros and tests are ! the name of the project and so on. Some configs are mandatory but others are optional !
  - Mandatory :
    | Field            | Purpose                                                      | Notes / Requirements                                                                                         |
    | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
    | `name`           | The name of your dbt project                                 | Must be unique within your environment. Must not contain spaces or special characters. Example: `my_project` |
    | `version`        | Version of your project                                      | Any string, usually semantic versioning like `"1.0"`                                                         |
    | `config-version` | Version of dbt configuration format                          | Currently must be `2` (dbt >= 0.17 uses `config-version: 2`)                                                 |
    | `profile`        | Name of the profile in `profiles.yml` to use for connections | Must match a profile defined in `~/.dbt/profiles.yml`                                                        |
    | `source-paths`   | List of paths where your models are stored                   | Usually `["models"]`. Relative to project root.                                                              |
    | `target-path`    | Where compiled models and artifacts are written              | Default is `target`.                                                                                         |
    | `clean-targets`  | Paths that will be cleaned when running `dbt clean`          | Usually `[“target”, “dbt_modules”]`                                                                          |

  - Optional :
    - analysis-paths: Paths for analyses/ folder
    - macro-paths: Paths for macros/ folder
    - snapshot-paths: Paths for snapshots/ folder
    - data-paths: Paths for seeds/ folder
    - test-paths: Paths for tests/ folder
    - models: section: Default configurations per folder (like materialized: table)

  In this file we can define some chracteristics that are common to several objects in a subfolder such as the materialization, tests, docs colors per models type and so on !
  **But normaly we use a specific yaml file to configure each model separatly.**

  - In the dbt_project.yaml file we declare also project level variable that can be used everywhere in the project !
  - We can also declare variables at the model level in the configs or override them in the command line ! the order of precedence is the following :
  When dbt resolves var("x"), it looks in this order:
    - Command-line (--vars)
    - Model-level config(vars={...})
    - Project-level (dbt_project.yml)
    
Example :  

```
    # dbt_project.yml
    # Main dbt project configuration
    
    name: 'my_dbt_project'
    version: '1.0.0'
    config-version: 2
    
    # This setting configures which "profile" dbt uses for this project.
    profile: 'default'
    
    # These configurations specify where dbt should look for different types of files.
    model-paths: ["models"]
    analysis-paths: ["analysis"]
    test-paths: ["tests"]
    seed-paths: ["data"]
    macro-paths: ["macros"]
    snapshot-paths: ["snapshots"]
    
    target-path: "target"
    clean-targets:
      - "target"
      - "dbt_packages"
    
    # Model configuration
    models:
      my_dbt_project:
        # Staging models - ephemeral for CI efficiency
        staging:
          +materialized: "{{ 'table' if target.name == 'prod' else 'ephemeral' }}"
          +tags: ["staging"]
          +docs:
            node_color: "lightblue"
        
        # Intermediate models
        intermediate:
          +materialized: "{{ 'table' if target.name in ['prod', 'dev'] else 'ephemeral' }}"
          +tags: ["intermediate"]
          +docs:
            node_color: "lightgreen"
        
        # Marts models - always materialized as tables
        marts:
          +materialized: table
          +tags: ["marts"]
          +docs:
            node_color: "gold"
          
          # Business logic marts
          core:
            +tags: ["marts", "core"]
            
          # Analytics marts
          analytics:
            +tags: ["marts", "analytics"]
    
    # Snapshot configuration
    snapshots:
      my_dbt_project:
        +target_schema: "{{ target.schema }}_snapshots"
        +strategy: timestamp
        +updated_at: updated_at
    
    # Seeds configuration
    seeds:
      my_dbt_project:
        +schema: "{{ target.schema }}_seeds"
        +quote_columns: false
    
    # Test configuration
    # Here we persist the failures data in tables or view for monitoring later
    # The schema name will depend on the target (environment) schema name
    tests:
      +store_failures: true
      +schema: "{{ target.schema }}_test_failures"
    
    # Macro configuration : This is used if we create our own macros that overrides standard libraries such as dbt_utils ! generaly we just say to dbt to look for
    # the macro in our project and not use the standars one in dbt_utils
    # So if we’ve written a macro in our project with the same name as one in dbt_utils, our version will override the package version.
    dispatch:
      - macro_namespace: dbt_utils
        search_order: ['my_dbt_project', 'dbt_utils']
    
    # Variables for different environments
    vars:
      # Date variables for incremental models
      start_date: '2020-01-01'
      
      # CI-specific variables
      ci_schema: 'DBT_CI_DEFAULT'
      deployment_type: 'standard'
      
      # Feature flags
      enable_elementary_monitoring: "{{ true if target.name == 'prod' else false }}"
      enable_advanced_tests: "{{ true if target.name in ['prod', 'dev'] else false }}"
    
    # Hooks for environment-specific setup
    on-run-start:
      - "{{ create_schema_if_not_exists() }}"
      - "{{ log_run_start_info() }}"
    
    on-run-end:
      - "{{ log_run_end_info() }}"
      - "{{ cleanup_temp_tables() if target.name == 'ci' else '' }}"
    
    # Quoting configuration for Snowflake
    quoting:
      database: false
      schema: false
      identifier: false
```

**Note that in the synthax of configs, we always use + ! that is because in dbt_project we generaly override configs that are normaly declared in models or in dbt profile.yaml file**  
**whenever we override some configs we use +**  

Two places where we can set configs :  

| File / context        | Can set `materialized`? | Can set `schema`/others? | `+` required?          |
| --------------------- | ----------------------- | ------------------------ | ---------------------- |
| `schema.yml`          | no                      | no                       | n/a                    |
| Inline SQL `config()` | yes                     | yes (for overrides)      | `+` for overrides only |
| `dbt_project.yml`     | yes                     | yes                      | yes                    |

**The schema.yaml file or model_name.yaml specifies only description, documention and tests for a specific model !**  


5- Dependencies :
In dbt we can use some packages that are prebuilt to perform some operations like tests, functions and so on !  
  - dbt_utils → utility macros
  - dbt_expectations → test macros
  - elementary → monitoring and observability macros ! This persists the results of tests for monitoring later !
  
**These ones are different from the dependencies we install for python and that we may use in the CI part for example**.  
We specifie the dbt packages in a packages.yaml file : **Note that this is a root file!**

```
  packages:
  - package: dbt-labs/dbt_utils
    version: 1.3.0
  - package: metaplane/dbt_expectations
    version: 0.10.8
```

To install these packages we run `dbt deps` ! These packages are just macros ! we can override them if we like, we scan create a macro with the same name and make some configs in the dbt_project.file to tell dbt to search first in our project then if it does not find it, it will point to the standard packages directory! 

```
    dispatch:
      - macro_namespace: dbt_utils
        search_order: ['my_dbt_project', 'dbt_utils']
```

6- Models, Sources and schemas.yaml :

This is the heart of dbt, as models are the definition of the query that will create the tables, views and so on !  

  - Module Example :

    ```
    WITH raw_hosts AS (
    SELECT
        *
    FROM
       {{ source('airbnb', 'hosts') }}
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

    This behind the scene will create an object in snowflake (table, view depending on the materialization strategy !)

  - Sources :
    Sources in dbt are basically a way of telling dbt: “these upstream tables/views already exist in my warehouse, dbt didn’t build them, but I want to use them in my models.”  
    we can create sources either in a separate sources.yaml file that groups all the sources used in the project or we can create a source file per staging model !
    dbt will after all combine all the yaml files, but it is recommanded for organization puposes to separate files !

    Why use sources (vs hardcoding schema.table)?

      - Portability → you don’t have to hardcode warehouse-specific schema names.
      - Documentation → dbt Docs will show your sources in the DAG.
      - Testing → you can add freshness tests and column tests on sources.
      - Consistency → {{ source() }} resolves correctly across environments (dev, prod, CI).
        

    Source example :

    ```
    version: 2 # Mendatory in every yaml file since it tells dbt what format of yaml files are we using (legacy version 1 or moderne version 2)

    sources:
      - name: airbnb
        database : AIRBNB
        schema: RAW
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
  - Schema.yaml or model_name.yaml :
    
    This is a file that is useful to add descriptions, docs and tests to models !
    Two ways of doing that just like sources : One single file for all models or a file per model !
    These arguments can also be used in the dbt_project.yaml file but it is not recommanded as for big projects it affects readability !

    Example :

    ```
    version: 2

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
    
           - name: minimum_nights
             description: '{{ doc("desc_dim_listing_min_nights")}}'
             test:
               - null_column_test *
    
       - name: dim_listings_with_hosts
         tests:
          - dbt_expectations.expect_table_row_count_to_equal_other_table:
              compare_model: source('airbnb','listings')

    ```
7- Snapshots : 
  Snapshots are an object that use CDC to ingest data into a target table which is a SCD2 dim table ! Snapshots capture and preserve historical versions of records by tracking changes over time.  
  
  How it works:
  - Initial snapshot: Captures current state of source data
  - Subsequent runs: Compares new data with existing snapshot
  - Change detection: Identifies what changed using a strategy
  - Version creation: Creates new records for changes while preserving old ones
  
  Snpashots are stored as tables in your data warehouse (Snowflake, BigQuery, etc.):  
  - Data never vanishes - that's the whole point!
  - Each record gets additional metadata columns: 
      - dbt_valid_from: When this version became active
      - dbt_valid_to: When this version became inactive (NULL for current)
      - dbt_updated_at: When dbt processed this change
      - dbt_scd_id: Unique identifier for each version
  Snapshots have several strategies to insure the SCD2 process (how to check for new data ?):
    - Timestamp Strategy
      ```
          strategy='timestamp',
          updated_at='last_modified_date'
      ```
    - Check Strategy :
      ```
          strategy='check',
          check_cols=['status', 'email', 'phone']
      ```
    - Check All Columns :
      ```
          strategy='check',
          check_cols='all'
      ```      
    
Example :  

  ```
    {% snapshot customers_snapshot %}
        {{
            config(
              target_database='analytics',        # Where to store
              target_schema='snapshots',          # Schema location
              target_table='dim_customers_hist',  # Custom table name
              unique_key='customer_id',
              strategy='timestamp',
              updated_at='last_modified',
              invalidate_hard_deletes=true,       # Handle deleted records
            )
        }}
        SELECT * FROM {{ source('raw', 'customers') }}
    {% endsnapshot %}
  ```

7- Analysis: Reusable queries 

An analysis is just a .sql file stored under the /analyses directory in our dbt project. Unlike models, dbt does not build them into tables or views in our warehouse. Instead, they are compiled SQL queries that we can run manually (ad hoc analysis, investigations, debugging).  
Analysis need to run manualy and they are not automaticaly run when we use `dbt run` !  

Example :  

```
    -- analyses/user_activity.sql
    select
        user_id,
        count(*) as order_count
    from {{ ref('orders') }}
    group by 1
    having count(*) > 100

```

We compile the analysis like this :  

```
dbt compile --select analyses/
```
**Then this will generate compiled sql queries that we can copy past to run in snowflake for example !**  

8- Tests : 
Tests are operations that will check if a hypothesis is true and will true error if not !  
We have several types of tests (generic or macros, data tests and built-in tests)  
Tests (generic and built-in) can be run at the column level or table level to apply it to all columns at once !  

dbt has a few different ways to define tests:  
  - In the model yaml file. By default dbt provides some standard tests such as : unique, not_null, accepted_values, relationships.
    ```
    models:
      - name: orders
        columns:
          - name: order_id
            tests:
              - not_null
              - unique
    ```
  - test macros (generic tests): it is simply a macro that perfomrs a test on columns for example !
    
    ```
    {% macro test_null_column_test(model) %}
      SELECT *
      FROM {{ model }}
      WHERE some_complex_logic_across_multiple_columns
    {% endmacro %}
    
    ```
    Then we can call the test macro in the model yaml :

    ```
      models:
        - name: users
          tests:
            - null_column_test  # This runs against the entire table # note that we didn't call the whole name with test_ as dbt generates it by default !
          columns:
            - name: email
              tests:
                - unique  # This runs against just the email column
    ```
    
  - In the test folder (single data test) : here we define more sofisticated tests that are more complexed and dbt runs them as part of the dbt test command or dbt run command !
    
    ```
    select o.*
      from {{ ref('orders') }} o
      left join {{ ref('users') }} u
        on o.user_id = u.id
      where u.id is null
    ```
    **We cannot directly call singular tests from model YAML the same way we call generic tests**
    
9 - dbt contract (*similar to tests but before materialization, and if violated, materialization fails !*):
  
  Contracts are a feature that allows you to define and enforce explicit agreements about our data models' structure and behavior.
  - Model contracts - These define the expected schema (columns, data types) for your models. When you enable a contract on a model, dbt will enforce that the model's output matches exactly what you've specified in the contract.
  - Column-level specifications : we can define data types, constraints, and documentation for each column in our model. If the actual output doesn't match these specifications, dbt will fail the run.
  - Enforcement mechanism : When a model has a contract enabled, dbt validates the model's output against the contract during compilation and execution.
    
  How it works :
    - dbt runs your model SQL (SELECT statement)
    - dbt examines the result set in memory/staging
    - dbt checks if the result violates the contract
    - If contract passes: dbt materializes to the final table
    - If contract fails: dbt aborts and doesn't write to the final table
      
  So dbt essentially does a "dry run" validation before committing the results
  
  Example :

  ```
  models:
  - name: my_model
    config:
      contract:
        enforced: true # All what we specify after contract will be enforced and will fail the materialization if there is a violation
    columns:
      - name: user_id
        data_type: integer
        constraints:
          - type: not_null
      - name: email
        data_type: varchar(255)
        constraints:
          - type: not_null
          - type: unique
  ```
  **Keep in mind that dbt contracts are more expensive in terms of performance as they scan all data before materializing then it materialize it !**

   Many teams use a hybrid strategy:
    - Contracts for small, critical tables (dimentions) where integrity > performance
    - Tests for large tables where speed > immediate validation
    - Sampling strategies for tests on huge datasets
    
10 - Hooks:
A hook is a piece of code that runs automatically at a certain point in a process. In dbt, hooks let you run arbitrary SQL before or after certain operations (like building a model, snapshot, or test).

| Hook type     | When it runs                      | Syntax example                                                         |
| ------------- | --------------------------------- | ---------------------------------------------------------------------- |
| **pre-hook**  | Right **before a model is built** | Runs SQL just before `select ...` is executed or table is materialized |
| **post-hook** | Right **after a model is built**  | Runs SQL after the model is finished building                          |

Hooks are used for :  

  - Audit logging: insert a row into an audit table every time a model runs
  - Permissions: grant access to a table after it’s created
  - Data prep / cleanup: truncate a staging table before loading new data
  - Also after or before dbt run process ! we can call some macros !

| Hook type        | Runs when                                   | Scope              | Example use case                                    |
| ---------------- | ------------------------------------------- | ------------------ | --------------------------------------------------- |
| **pre-hook**     | Before a specific model is built            | Only that model    | Insert audit row before a table is created          |
| **post-hook**    | After a specific model is built             | Only that model    | Grant permissions after model creation              |
| **on-run-start** | Once, **before the entire dbt run starts**  | Global for the run | Create schemas, log run start, initialize variables |
| **on-run-end**   | Once, **after the entire dbt run finishes** | Global for the run | Log run status, cleanup temporary tables            |


Example of inline config :
```
{{ config(
    materialized='table',
    pre_hook="insert into audit_log(table_name, run_time) values ('users', current_timestamp)",
    post_hook="grant select on {{ this }} to analyst_role"
) }}

select * from raw.users
```

Example of dbt_project.yaml config :  

```
models:
  my_project:
    +post-hook:
      - "grant select on {{ this }} to analyst_role"
      - "insert into audit_log(table_name, run_time) values ('{{ this.name }}', current_timestamp)"
```

Example of on-run-start and on-rub-end :  

```
    # Hooks for environment-specific setup
    on-run-start:
      - "{{ create_schema_if_not_exists() }}"
      - "{{ log_run_start_info() }}"
    
    on-run-end:
      - "{{ log_run_end_info() }}"
      - "{{ cleanup_temp_tables() if target.name == 'ci' else '' }}"
```
