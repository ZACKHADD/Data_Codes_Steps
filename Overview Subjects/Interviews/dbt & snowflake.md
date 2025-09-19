## dbt on Snowflake !

##### This file contains the main subjects to cover for a dbt & snowflake interview !

### dbt :

#### 1- Purpose : 

- Why dbt ? a transformation tool that works with nearly any data platform and the nature of its synthax makes it highly portable from a plateform another easily !
- Facilitate the transformation process and provides a lot of functionalitites such as documentation, lineage, macros, tests and so on !

#### 2- Installation

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

#### 3- Structure of the project 

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


#### 5- Dependencies :
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

#### 6- Models, Sources and schemas.yaml :

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
#### 7- Materialization and strategies :  

  Materialization is the heart of models since it defines what schema objects will be created in snowflake from the model !  
  
  - table - Full refresh every run
  - view - Creates a view, no data stored
  - incremental - Uses strategies above
  - ephemeral - CTE, not materialized -- Used for several models for example since it can be referenced just like normal models and it appears in the lineage !

  Ephemeral materializations should be used only for simple logics not complexed ones, since the complexe logic is expensive to compute (e.g., large joins, aggregations),   and repeating it as an inline CTE in multiple downstream models can hurt performance.  

  for the incremental materializations, there are several strategies :  
  
| Strategy           | Updates Existing Records     | Inserts New Records | Requires `unique_key` | Best For                                  |
| ------------------ | ---------------------------- | ------------------- | --------------------- | ------------------------------------------| 
| `append`           | No                           | Yes                 |  No                   | Simple appends without updates            |
| `merge`            | Yes                          | Yes                 |  Yes                  | Upserts where `MERGE` is supported        |
| `delete+insert`    | Yes                          | Yes                 |  Yes                  | Warehouses without `MERGE` support        |
| `insert_overwrite` | No (overwrites partitions)   | Yes                 |  No                   | Partitioned tables in BigQuery, Snowflake |
| `microbatch`       | Yes                          | Yes                 |  Yes (recommended)    | Large time-series datasets                |  

Each of these strategies can have some configs ! for example the merge one has : merge_update_columns=['valid_to', 'is_current'] to specify that we want only to updat these two columns ! Otherwise the default behaviour is to update all columns !  

#### DBT CONFIGURATION REFERENCE : 

```
        -- ============================================================================
        -- COMPLETE DBT CONFIGURATION REFERENCE
        -- ============================================================================
        
        {{ config(
            -- ========================================================================
            -- MATERIALIZATION STRATEGIES
            -- ========================================================================
            materialized='incremental',           -- Options: table, view, incremental, ephemeral, snapshot
            
            
            -- ========================================================================
            -- INCREMENTAL-SPECIFIC CONFIGS
            -- ========================================================================
            
            -- Incremental Strategy - HOW to process incremental data
            incremental_strategy='merge',          -- Options: merge (default), append, delete+insert, microbatch
            
            -- Unique Key - Critical for merge/delete+insert strategies
            unique_key='id',                       -- Single column
            -- unique_key=['customer_id', 'date'], -- Composite key (uncomment if needed)
            
            -- Merge Update Columns - WHICH columns to update (only works with merge strategy)
            merge_update_columns=['valid_to', 'is_current'],  -- Only update specified columns during merge
            
            -- Merge Exclude Columns - Exclude columns from merge updates
            merge_exclude_columns=['created_at', 'id'],       -- These columns won't be updated during merge
            
            -- Schema Change Handling - Handle schema evolution
            on_schema_change='append_new_columns', -- Options: fail (default), ignore, append_new_columns, sync_all_columns
            
            -- Incremental Predicates - Add custom WHERE conditions to incremental logic
            incremental_predicates=["dbt_valid_to is null", "status = 'active'"],
            
            
            -- ========================================================================
            -- SNOWFLAKE-SPECIFIC CONFIGS
            -- ========================================================================
            
            -- Table Clustering - For table clustering performance
            cluster_by=['date_column', 'category'],
            
            -- Automatic Clustering - Enable auto-clustering
            automatic_clustering=true,
            
            -- Secure Views - Create secure views (hides definition)
            secure=true,                           -- Only works with materialized='view'
            
            -- Copy Grants - Preserve grants during refreshes
            copy_grants=true,
            
            -- Query Tag - Add query tags for monitoring and cost tracking
            query_tag='dbt_marts_layer',
            
            -- Transient Tables - Use transient tables (no Fail-safe, lower cost)
            transient=true,
            
            -- Warehouse - Specify warehouse for model execution
            snowflake_warehouse='TRANSFORM_WH',
            
            
            -- ========================================================================
            -- PERFORMANCE & RESOURCE CONFIGS
            -- ========================================================================
            
            -- Pre/Post Hooks - Run SQL before/after model execution
            pre_hook="ALTER WAREHOUSE {{ target.warehouse }} SET WAREHOUSE_SIZE = 'LARGE'",
            post_hook=[
                "ALTER WAREHOUSE {{ target.warehouse }} SET WAREHOUSE_SIZE = 'MEDIUM'",
                "GRANT SELECT ON {{ this }} TO ROLE ANALYST_ROLE"
            ],
            
            -- Grant Access - Grant permissions to roles
            grant_access_to=[
                {'database_role': 'ANALYST_ROLE'},
                {'account_role': 'MARKETING_TEAM'}
            ],
            
            -- database_role - Database-Level Roles (Newer Feature). This is a newer Snowflake feature that allows roles to be scoped to specific databases:
            -- Create a database role (scoped to a specific database)
                  CREATE DATABASE ROLE ANALYTICS_DB.READER;
                  CREATE DATABASE ROLE ANALYTICS_DB.WRITER;

            -- ========================================================================
            -- DOCUMENTATION & METADATA CONFIGS
            -- ========================================================================
            
            -- Persist Docs like descripotionns - Include documentation in warehouse metadata as comnents tabel level and column level
            persist_docs={"relation": true, "columns": true}, 

            -- "relation": true → Persist description for the database object (table/view/etc.)

            -- Example :
            # models/marts/dim_customers.yml
                version: 2
                
                models:
                  - name: dim_customers
                    description: "Customer dimension with SCD Type 2 history"
                    config:
                      persist_docs: {"relation": true, "columns": true}
                    columns:
                      - name: customer_id
                        description: "Unique customer identifier"
            -- ========================================================================
            -- DATA QUALITY CONFIGS (CONTRACTS)
            -- ========================================================================
            
            -- Contract Enforcement - Define and enforce explicit data contracts
            contract={
                'enforced': true                   -- Validates schema and constraints before materialization
            }
            
            
            -- ========================================================================
            -- SNAPSHOT-SPECIFIC CONFIGS (use in snapshot files only)
            -- ========================================================================
            
            -- Note: These configs are only used in {% snapshot %} blocks, not regular models
            
            -- target_database='analytics',       -- Where to store snapshot
            -- target_schema='snapshots',         -- Schema location for snapshot
            -- target_table='dim_customers_hist', -- Custom table name for snapshot
            -- unique_key='customer_id',          -- Primary key for change detection
            -- strategy='timestamp',              -- Options: timestamp, check
            -- updated_at='last_modified',        -- Column for timestamp strategy
            -- check_cols=['status', 'email'],    -- Columns to check for changes (check strategy)
            -- check_cols='all',                  -- Check all columns for changes
            -- invalidate_hard_deletes=true,      -- Handle deleted source records
            
        ) }}
        
        -- ============================================================================
        -- EXAMPLE USAGE PATTERNS
        -- ============================================================================
        
        -- PATTERN 1: High-performance incremental model with clustering
        /*
        {{ config(
            materialized='incremental',
            incremental_strategy='merge',
            unique_key='id',
            cluster_by=['date_created'],
            transient=true,
            copy_grants=true
        ) }}
        */
        
        -- PATTERN 2: SCD Type 2 dimension from snapshot
        /*
        {{ config(
            materialized='incremental',
            incremental_strategy='merge',
            unique_key=['customer_id', 'valid_from'],
            merge_update_columns=['valid_to', 'is_current'],
            on_schema_change='append_new_columns'
        ) }}
        */
        
        -- PATTERN 3: Append-only fact table
        /*
        {{ config(
            materialized='incremental',
            incremental_strategy='append',
            cluster_by=['transaction_date'],
            query_tag='fact_table_load'
        ) }}
        */
        
        -- PATTERN 4: Secure view with documentation
        /*
        {{ config(
            materialized='view',
            secure=true,
            persist_docs={"relation": true, "columns": true},
            copy_grants=true
        ) }}
        */
        
          -- ============================================================================
          -- ADDITIONAL IMPORTANT DBT CONFIGS
          -- ============================================================================
          
          -- ========================================================================
          -- PROJECT-LEVEL CONFIGS (dbt_project.yml)
          -- ========================================================================
          
          -- Global model configurations that apply to all models unless overridden
          -- Example dbt_project.yml structure:
          /*
          name: 'my_project'
          version: '1.0.0'
          config-version: 2
          
          model-paths: ["models"]
          analysis-paths: ["analyses"] 
          test-paths: ["tests"]
          seed-paths: ["seeds"]
          macro-paths: ["macros"]
          snapshot-paths: ["snapshots"]
          
          target-path: "target"
          clean-targets: ["target", "dbt_packages"]
          
          models:
            my_project:
              +materialized: view              # Default materialization for all models
              staging:
                +materialized: view            # All staging models as views
              marts:
                +materialized: table           # All mart models as tables
                +transient: true               # All mart tables as transient
          */
          
          -- ========================================================================
          -- SOURCE CONFIGURATIONS
          -- ========================================================================
          
          -- Source Freshness - Monitor data freshness
          /*
          sources:
            - name: raw_data
              description: "Raw source data"
              freshness:
                warn_after: {count: 12, period: hour}    # Warn if data older than 12 hours
                error_after: {count: 24, period: hour}   # Error if data older than 24 hours
              loaded_at_field: _loaded_at                # Column that tracks load time
              
              tables:
                - name: customers
                  description: "Customer data from CRM"
                  freshness:
                    warn_after: {count: 6, period: hour}  # Override for this table
                  columns:
                    - name: id
                      description: "Primary key"
                      tests:
                        - unique
                        - not_null
          */
          
          -- ========================================================================
          -- SEED CONFIGURATIONS
          -- ========================================================================
          
          -- Seed files (CSV) configurations
          /*
          seeds:
            my_project:
              +column_types:                    # Define column types for CSV files
                id: varchar(50)
                name: varchar(100)
                is_active: boolean
              +quote_columns: false             # Don't quote column names
              +delimiter: ','                   # CSV delimiter
          */
          
          -- ========================================================================
          -- MACRO CONFIGURATIONS
          -- ========================================================================
          
          -- Custom macro with configuration
          /*
          {% macro generate_schema_name(custom_schema_name, node) -%}
              {%- set default_schema = target.schema -%}
              {%- if custom_schema_name is none -%}
                  {{ default_schema }}
              {%- else -%}
                  {{ default_schema }}_{{ custom_schema_name | trim }}
              {%- endif -%}
          {%- endmacro %}
          */
          
          -- ========================================================================
          -- EXPOSURE CONFIGURATIONS
          -- ========================================================================
          
          -- Define downstream dependencies (dashboards, reports, etc.)
          /*
          exposures:
            - name: customer_dashboard
              type: dashboard
              maturity: high
              url: https://tableau.company.com/customer_dashboard
              description: Executive customer metrics dashboard
              depends_on:
                - ref('dim_customers')
                - ref('fct_orders')
              owner:
                name: Marketing Team
                email: marketing@company.com
          */
          
          -- ========================================================================
          -- METRIC CONFIGURATIONS (dbt v1.6+)
          -- ========================================================================
          
          -- Define business metrics
          /*
          metrics:
            - name: total_revenue
              label: Total Revenue
              model: ref('fct_orders')
              description: "Sum of all order values"
              calculation_method: sum
              expression: order_amount
              timestamp: order_date
              time_grains: [day, week, month, quarter, year]
              dimensions:
                - customer_id
                - product_category
              filters:
                - field: status
                  operator: '='
                  value: "'completed'"
          */
          
          -- ========================================================================
          -- TESTING CONFIGURATIONS
          -- ========================================================================
          
          -- Advanced test configurations
          /*
          tests:
            - name: assert_positive_order_amounts
              description: "Ensure all order amounts are positive"
              config:
                severity: error              # Options: warn, error
                error_if: ">= 1"            # Fail if >= 1 records returned
                warn_if: ">= 1"             # Warn if >= 1 records returned
                limit: 100                  # Only show first 100 failing records
                store_failures: true       # Store failing records in table
          */
          
          -- ========================================================================
          -- ENVIRONMENT & TARGET CONFIGURATIONS
          -- ========================================================================
          
          -- Conditional configurations based on environment
          {{ config(
              materialized='incremental' if target.name == 'prod' else 'view',
              transient=true if target.name == 'prod' else false,
              tags=['finance'] if target.name == 'prod' else ['dev'],
          ) }}
          
          -- ========================================================================
          -- ADVANCED MODEL CONFIGURATIONS
          -- ========================================================================
          
          {{ config(
              -- File Format Options (Snowflake)
              file_format='parquet',                    -- For external table materialization
              
              -- Tagging for organization
              tags=['pii', 'customer_data', 'daily'],  -- Organize models by tags
              
              -- Model alias (different name in warehouse)
              alias='customer_dimension',               -- Table name different from file name
              
              -- Custom schema (override default schema)
              schema='customer_mart',                   -- Override default schema
              
              -- Full refresh control
              full_refresh=false,                       -- Prevent accidental full refresh
              
              -- SQL Header (run before model SQL)
              sql_header="SET QUERY_TAG = 'dbt_customer_processing';",
              
              -- Bind variables for parameterized queries  
              bind=false,                               -- Disable parameter binding
              
              -- Hours to live (for transient tables)
              hours_to_expiration=24,                   -- Auto-drop after 24 hours
              
          ) }}
```

#### 8- Snapshots : 
  Snapshots are an object that use CDC to ingest data into a target table which is a SCD2 dim table ! Snapshots capture and preserve historical versions of records by tracking changes over time.  
  
  How it works:
  - Initial snapshot: Captures current state of source data
  - Subsequent runs: Compares new data with existing snapshot
  - Change detection: Identifies what changed using a strategy
  - Version creation: Creates new records for changes while preserving old ones
  
  Snpashots are stored as tables in data warehouse (Snowflake, BigQuery, etc.):  
  - Data never vanishes - that's the whole point!
  - Each record gets additional metadata columns: 
      - dbt_valid_from: When this version became active
      - dbt_valid_to: When this version became inactive (NULL for current)
      - dbt_updated_at: When dbt processed this change
      - dbt_scd_id: Unique identifier for each version

  Example :  

  ```
    {% snapshot customers_snapshot %}
        {{
            config(
              target_database='analytics',        # Where to store
              target_schema='snapshots',          # Schema location
              target_table='dim_customers_hist',  # Custom table name otherwise it uses the snapshot name
              unique_key='customer_id',
              strategy='timestamp',
              updated_at='last_modified',
              invalidate_hard_deletes=true,       # Handle deleted records
            )
        }}
        SELECT * FROM {{ source('raw', 'customers') }}
    {% endsnapshot %}
  ```

  Once the snpashot is populated, we use it to populate the SCD2 dimension :  
  
    ```
          -- models/marts/dim_customers.sql
          {{ config(
              materialized='incremental', # we can also do a full refresh if the dimension is not do big !
              unique_key='customer_id || valid_from',  -- Composite key for SCD Type 2
              merge_update_columns=['valid_to', 'is_current']  # in the case of incremental it updates the old records columns
              on_schema_change='fail'
          ) }}
          
          SELECT 
              customer_id,
              customer_name,
              email,
              status,
              -- Add business logic
              CASE 
                  WHEN status = 'premium' THEN 'High Value'
                  ELSE 'Standard'
              END as customer_tier,
              
              -- SCD fields from snapshot
              dbt_valid_from as valid_from,
              dbt_valid_to as valid_to,
              
              -- Current record indicator
              CASE WHEN dbt_valid_to IS NULL THEN TRUE ELSE FALSE END as is_current
              
          FROM {{ ref('customers_snapshot') }}
          
          {% if is_incremental() %}
            -- Only process records that changed since last run
            WHERE dbt_updated_at > (SELECT MAX(dbt_updated_at) FROM {{ this }})
          {% endif %}
    ```
  Why This Separation Makes Sense:  
  Snapshots = Raw Historical Capture
  - Pure data capture with minimal transformation
  - Handles the complex SCD Type 2 logic
  - Reusable across multiple downstream models

  Dimension Models = Business Logic + Presentation
  - Add calculated fields, business rules
  - Apply data quality checks
  - Format for specific use cases
  - Can combine multiple snapshots
    
  Snapshots have several strategies to insure the SCD2 process (how to check for new data ?):
  
    - Timestamp Strategy:
    
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
    
#### 9- Analysis: Reusable queries 

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

#### 10- Tests :  

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
    
#### 11- dbt contract (*similar to tests but before materialization, and if violated, materialization fails !*):
  
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
    
#### 12 - Hooks:
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

#### 13- Seeds:  

Seeds are csv files that contains **reference data** that is essentially "lookup" or "master" data that provides context, categorization, or mapping information for your business processes. It's typically small, changes infrequently, and is used to enrich or categorize your main transactional data.  

Common examples of reference data: 
- Geographic mappings:
    - Country codes (US = United States, CA = Canada, etc.)
    - State/province abbreviations
    - Currency codes (USD, EUR, GBP)
    - Time zone mappings

- Business categorizations:
    - Product categories or hierarchies
    - Department codes and names
    - Customer segments or tiers
    - Sales territories and regions

Configuration example :  

```
    seeds:
      my_project:
        +column_types:                    # Define column types for CSV files
          id: varchar(50)
          name: varchar(100)
          is_active: boolean
        +quote_columns: false             # Don't quote column names
        +delimiter: ','                   # CSV delimiter
```

#### 14- Groups : 

An advanced feature that turns implicit relationships into an explicit grouping ! a group is simply a bunch of objects such as models, tests, macors and so on that belong to a domain for example and once we define a group we need to define responsibilities of teams and so assign a team to the group !  

It allows :  

  - Team Ownership & Accountability
  - Data Governance & Access Control: When a resource is grouped, dbt will allow it to reference private models within the same group.
  - Clear Boundaries

When we define groups we get to set also Access Levels:  

  - access: private - Only models within the same group can reference this model
  - access: public - Any model can reference this, regardless of group
  - access: protected - Can be referenced by models in the same project/package

Examples :  
    
  ```
      # dbt_project.yml
      groups:
        - name: finance
          owner:
            name: Finance Team
            email: finance@company.com
        - name: marketing
          owner:
            name: Marketing Team
            email: marketing@company.com
        - name: product
          owner:
            name: Product Team
            email: product@company.com
  ```
  ```
    -- models/finance/staging/stg_transactions.sql
      {{ config(
          group='finance',
          access='private'
      ) }}
      
      -- This is a staging model that only finance team should use
      -- Other groups can't reference this model
      SELECT * FROM {{ source('raw', 'transactions') }}
  ```

#### 15- Exposures : 

#### 16- Metrics : 

#### 17- dbt commands and Slim CI: 
