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

  ##### Popular Testing Libraries:
  
  1. dbt-expectations (most popular)
  
  Extends dbt's testing capabilities
  Provides statistical and data quality tests
  Examples: expect_column_values_to_be_between, expect_table_row_count_to_be_between
  
  2. elementary-data/dbt-data-reliability
  
  Anomaly detection and data monitoring
  Automated data quality monitoring
  Schema change detection
  
  3. dbt-audit-helper
  
  Comparing datasets between environments
  Useful for migration validation
  Model comparison utilities
  
  4. re_data
  
  Data reliability testing
  Time-series monitoring
  Automated alerting
    
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

In dbt, exposures are a way to define and document the downstream uses of your dbt models - essentially the "endpoints" where your data gets consumed. They represent dashboards, reports, ML models, or any other data artifacts that depend on your dbt models.  

Exposures are configured in a yaml file as follows:  

```
    version: 2
    
    exposures:
      - name: executive_sales_dashboard
        label: "Executive Sales Dashboard"
        type: dashboard
        maturity: high
        url: https://bi.company.com/dashboards/exec-sales
        description: |
          Executive-level dashboard showing key sales metrics,
          updated daily for C-suite consumption.
        
        depends_on:
          - ref('mart_sales_summary')
          - ref('dim_sales_territories')
          - source('external_systems', 'market_data')
        
        owner:
          name: Analytics Team
          email: analytics@company.com
        
        tags: ['executive', 'sales', 'daily']
        
        meta:
          refresh_schedule: "daily at 6am"
          business_owner: "VP of Sales"
```


#### 16- Metrics : 

In dbt, metrics are a way to define business logic for key performance indicators (KPIs) and standardize how metrics are calculated across organization. They provide a semantic layer on top of data models. It helps:  

  - Standardize business definitions across teams
  - Centralize metric logic in one place
  - Enable self-service analytics with consistent calculations
  - Document business context alongside technical definitions
  - Generate metric queries programmatically

Metrics are used to generate the semantic layer and, code generation and documentation ! **Note that in dbt the semantic layer requires dbt Cloud enterprise !**  

The code generation part simplifies the model construction by specifiying the complexity of calculation in the metrics yaml then reference it in the model and dbt behind the scene will generate sql code with the complexe calculation for us !  

We can specify dimentions in the metric config to add group by ability and also time intelegence or complexe atomatic model generation :  

```
    -- This automatically includes proper GROUP BY
    {{ metric('total_revenue', dimensions=['product_category', 'region']) }}
    
    -- Generates:
    sum(case when filters... then order_amount end) -- with GROUP BY product_category, region

    -- Instead of complex window functions, just use offset
    {{ metric('total_revenue', offset=-1) }}  -- Previous period
    {{ metric('total_revenue', offset=-12) }} -- Year ago

    -- Build different models for different stakeholders
    {% for department in ['sales', 'marketing', 'finance'] %}
        select 
            '{{ department }}' as department,
            {% for metric in department_metrics[department] %}
                {{ metric(metric) }} as {{ metric }}
                {%- if not loop.last -%},{%- endif -%}
            {% endfor %}
        from {{ ref('base_data') }}
        {% if not loop.last %}union all{% endif %}
    {% endfor %}
  
```

Examples :  

```

    -- ==============================================
    -- PART 1: METRIC DEFINITIONS (schema.yml)
    -- ==============================================
    
    version: 2
    
    metrics:
      - name: total_revenue
        label: "Total Revenue"
        model: ref('fct_orders')
        calculation_method: sum
        expression: order_amount
        timestamp: order_date
        time_grains: [day, week, month, quarter, year]
        filters:
          - field: order_status
            operator: '='
            value: "'completed'"
          - field: is_refund
            operator: '='
            value: "false"
        dimensions:
          - product_category
          - region
          - customer_segment
    
      - name: unique_customers
        label: "Unique Customers"
        model: ref('fct_orders')
        calculation_method: count_distinct
        expression: customer_id
        timestamp: order_date
        time_grains: [day, week, month, quarter, year]
        filters:
          - field: order_status
            operator: '='
            value: "'completed'"
        dimensions:
          - product_category
          - region
    
      - name: average_order_value
        label: "Average Order Value"
        model: ref('fct_orders')
        calculation_method: average
        expression: order_amount
        timestamp: order_date
        time_grains: [day, week, month, quarter, year]
        filters:
          - field: order_status
            operator: '='
            value: "'completed'"
          - field: is_refund
            operator: '='
            value: "false"
    
      - name: conversion_rate
        label: "Conversion Rate"
        calculation_method: ratio
        numerator:
          name: conversions
          model: ref('fct_events')
          expression: count(*)
          filters:
            - field: event_type
              operator: '='
              value: "'purchase'"
        denominator:
          name: sessions
          model: ref('fct_sessions')
          expression: count(distinct session_id)
        timestamp: event_date
        time_grains: [day, week, month]
    
    -- ==============================================
    -- PART 2: CODE GENERATION IN MODELS
    -- ==============================================
    
    -- models/marts/daily_metrics.sql
    -- Example 1: Basic metric usage
    
    {{ config(materialized='table') }}
    
    select 
        order_date,
        
        -- These metric() calls generate actual SQL aggregations
        {{ metric('total_revenue', grain='day') }} as daily_revenue,
        {{ metric('unique_customers', grain='day') }} as daily_customers,
        {{ metric('average_order_value', grain='day') }} as daily_aov
        
    from {{ ref('fct_orders') }}
    group by 1
    order by 1
    
    -- COMPILED OUTPUT (what dbt actually runs):
    /*
    select 
        order_date,
        
        -- Generated from total_revenue metric
        sum(case when order_status = 'completed' and is_refund = false 
            then order_amount else null end) as daily_revenue,
            
        -- Generated from unique_customers metric  
        count(distinct case when order_status = 'completed' 
            then customer_id else null end) as daily_customers,
            
        -- Generated from average_order_value metric
        avg(case when order_status = 'completed' and is_refund = false 
            then order_amount else null end) as daily_aov
        
    from analytics.fct_orders
    group by 1
    order by 1
    */
    
    -- ==============================================
    -- Example 2: Multi-dimensional metrics
    -- ==============================================
    
    -- models/marts/product_metrics.sql
    
    select 
        date_trunc('month', order_date) as month_date,
        product_category,
        region,
        
        -- Metrics automatically include dimension filtering
        {{ metric('total_revenue', grain='month', dimensions=['product_category', 'region']) }} as monthly_revenue,
        {{ metric('unique_customers', grain='month', dimensions=['product_category', 'region']) }} as monthly_customers
        
    from {{ ref('fct_orders') }}
    group by 1, 2, 3
    
    -- COMPILED OUTPUT:
    /*
    select 
        date_trunc('month', order_date) as month_date,
        product_category,
        region,
        
        sum(case when order_status = 'completed' and is_refund = false 
            then order_amount else null end) as monthly_revenue,
            
        count(distinct case when order_status = 'completed' 
            then customer_id else null end) as monthly_customers
        
    from analytics.fct_orders
    group by 1, 2, 3
    */
    
    -- ==============================================
    -- Example 3: Time-based analysis with offsets
    -- ==============================================
    
    -- models/marts/growth_analysis.sql
    
    select 
        date_trunc('month', order_date) as month_date,
        
        -- Current month metrics
        {{ metric('total_revenue', grain='month') }} as current_month_revenue,
        
        -- Previous month metrics using offset
        {{ metric('total_revenue', grain='month', offset=-1) }} as prev_month_revenue,
        
        -- Year-over-year comparison
        {{ metric('total_revenue', grain='month', offset=-12) }} as yoy_revenue,
        
        -- Calculate growth rates using generated metrics
        case 
            when {{ metric('total_revenue', grain='month', offset=-1) }} > 0 
            then ({{ metric('total_revenue', grain='month') }} - {{ metric('total_revenue', grain='month', offset=-1) }}) 
                 / {{ metric('total_revenue', grain='month', offset=-1) }} * 100
            else null 
        end as mom_growth_percent
    
    from {{ ref('fct_orders') }}
    group by 1
    
    -- COMPILED OUTPUT:
    /*
    select 
        date_trunc('month', order_date) as month_date,
        
        -- Current month
        sum(case when order_status = 'completed' and is_refund = false 
            then order_amount else null end) as current_month_revenue,
        
        -- Previous month with window function
        sum(case when order_status = 'completed' and is_refund = false 
            then order_amount else null end) 
            over (order by date_trunc('month', order_date) 
                  rows between 1 preceding and 1 preceding) as prev_month_revenue,
        
        -- Year-over-year with window function
        sum(case when order_status = 'completed' and is_refund = false 
            then order_amount else null end) 
            over (order by date_trunc('month', order_date) 
                  rows between 12 preceding and 12 preceding) as yoy_revenue,
                  
        -- Growth calculation using the same logic
        case 
            when [previous month calculation] > 0 
            then ([current month] - [previous month]) / [previous month] * 100
            else null 
        end as mom_growth_percent
    
    from analytics.fct_orders
    group by 1
    */
    
    -- ==============================================
    -- Example 4: Ratio metrics
    -- ==============================================
    
    -- models/marts/conversion_analysis.sql
    
    select 
        event_date,
        traffic_source,
        
        -- Ratio metric generates complex SQL automatically
        {{ metric('conversion_rate', grain='day', dimensions=['traffic_source']) }} as daily_conversion_rate
        
    from {{ ref('fct_events') }}
    left join {{ ref('fct_sessions') }} using (session_id)
    group by 1, 2
    
    -- COMPILED OUTPUT:
    /*
    select 
        event_date,
        traffic_source,
        
        -- Complex ratio calculation generated automatically
        case 
            when count(distinct case when s.session_id is not null then s.session_id else null end) > 0
            then count(case when e.event_type = 'purchase' then 1 else null end) * 100.0 
                 / count(distinct case when s.session_id is not null then s.session_id else null end)
            else null 
        end as daily_conversion_rate
        
    from analytics.fct_events e
    left join analytics.fct_sessions s using (session_id)
    group by 1, 2
    */
    
    -- ==============================================
    -- Example 5: Using metrics in macros
    -- ==============================================
    
    -- macros/calculate_metric_targets.sql
    
    {% macro calculate_metric_targets(metric_name, target_increase_pct=10) %}
    
        select 
            current_date as target_date,
            '{{ metric_name }}' as metric_name,
            
            -- Use current metric value
            {{ metric(metric_name, grain='month') }} as current_value,
            
            -- Calculate target based on percentage increase
            {{ metric(metric_name, grain='month') }} * (1 + {{ target_increase_pct }}/100.0) as target_value,
            
            -- Show the gap
            ({{ metric(metric_name, grain='month') }} * (1 + {{ target_increase_pct }}/100.0)) - 
            {{ metric(metric_name, grain='month') }} as gap_to_target
            
        from {{ ref('fct_orders') }}
    
    {% endmacro %}
    
    -- Usage in model:
    -- models/business/revenue_targets.sql
    
    {{ calculate_metric_targets('total_revenue', 15) }}
    union all
    {{ calculate_metric_targets('unique_customers', 20) }}
    
    -- ==============================================
    -- Example 6: Conditional metric usage
    -- ==============================================
    
    -- models/marts/adaptive_metrics.sql
    
    select 
        order_date,
        product_category,
        
        {% if var('include_revenue', true) %}
            {{ metric('total_revenue', grain='day', dimensions=['product_category']) }} as revenue,
        {% endif %}
        
        {% if var('include_customers', true) %}
            {{ metric('unique_customers', grain='day', dimensions=['product_category']) }} as customers,
        {% endif %}
        
        -- Always include this base metric
        {{ metric('average_order_value', grain='day', dimensions=['product_category']) }} as aov
    
    from {{ ref('fct_orders') }}
    group by 1, 2
    
    -- Usage with variables:
    -- dbt run -m adaptive_metrics --vars '{"include_revenue": false}'
    
    -- ==============================================
    -- Example 7: Testing metrics
    -- ==============================================
    
    -- models/tests/metric_validation.sql
    
    with metric_test as (
        select 
            order_date,
            
            -- Generated metric
            {{ metric('total_revenue', grain='day') }} as metric_revenue,
            
            -- Manual calculation for comparison
            sum(case when order_status = 'completed' and is_refund = false 
                then order_amount else null end) as manual_revenue
                
        from {{ ref('fct_orders') }}
        group by 1
    )
    
    select *
    from metric_test
    where abs(metric_revenue - manual_revenue) > 0.01  -- Find discrepancies
    
    -- ==============================================
    -- Example 8: Dynamic metric queries
    -- ==============================================
    
    -- models/marts/executive_dashboard.sql
    
    {% set executive_metrics = [
        'total_revenue',
        'unique_customers', 
        'average_order_value'
    ] %}
    
    select 
        date_trunc('month', current_date) as reporting_month,
        
        {% for metric_name in executive_metrics %}
            {{ metric(metric_name, grain='month') }} as {{ metric_name }}_current_month,
            {{ metric(metric_name, grain='month', offset=-1) }} as {{ metric_name }}_prev_month,
            
            -- Calculate month-over-month change
            case 
                when {{ metric(metric_name, grain='month', offset=-1) }} > 0
                then ({{ metric(metric_name, grain='month') }} - {{ metric(metric_name, grain='month', offset=-1) }})
                     / {{ metric(metric_name, grain='month', offset=-1) }} * 100
                else null 
            end as {{ metric_name }}_mom_change_pct
            
            {%- if not loop.last -%},{%- endif -%}
        {% endfor %}
    
    from {{ ref('fct_orders') }}
    
    -- This generates SQL for all metrics dynamically!
    
    -- ==============================================
    -- ADVANCED: Using metric metadata
    -- ==============================================
    
    -- macros/get_metric_info.sql
    
    {% macro get_metric_description(metric_name) %}
        {% set metric_node = graph.nodes.values() | selectattr('name', 'eq', metric_name) | first %}
        {% if metric_node %}
            {{ return(metric_node.description or 'No description available') }}
        {% else %}
            {{ return('Metric not found') }}
        {% endif %}
    {% endmacro %}
    
    -- Usage in model documentation:
    -- models/marts/documented_metrics.sql
    
    select 
        '{{ get_metric_description("total_revenue") }}' as revenue_description,
        {{ metric('total_revenue', grain='month') }} as total_revenue
    from {{ ref('fct_orders') }}
```

Full possible configurations:  

```
      version: 2
      
      metrics:
        # Simple Sum Metric with all configurations
        - name: total_revenue
          label: "Total Revenue (USD)"
          model: ref('fct_sales')
          description: |
            Total revenue in USD across all channels and products.
            Includes taxes and shipping fees but excludes refunds.
            Updated hourly from the sales fact table.
          
          calculation_method: sum
          expression: gross_revenue_amount
          timestamp: sale_timestamp
          time_grains: [hour, day, week, month, quarter, year]
          
          # Filtering conditions
          filters:
            - field: is_refund
              operator: '='
              value: "false"
            - field: sale_status
              operator: 'in'
              value: "('completed', 'shipped')"
            - field: gross_revenue_amount
              operator: '>'
              value: "0"
          
          # Available dimensions for grouping
          dimensions:
            - product_category
            - sales_channel
            - customer_segment
            - region
            - salesperson_id
          
          # Rolling window configuration
          window:
            count: 12
            period: month
          
          # Metadata and tags
          meta:
            owner: "Revenue Analytics Team"
            slack_channel: "#revenue-alerts"
            business_definition: "Gross revenue excluding refunds"
            data_source: "Salesforce + Shopify"
            refresh_schedule: "Every hour"
            sla_hours: 2
            dashboard_links:
              - "https://dashboard.company.com/revenue"
              - "https://tableau.company.com/revenue-executive"
            related_kpis: ['net_revenue', 'gross_margin']
            certification_level: "gold"
            last_audit_date: "2024-01-15"
          
          tags: ['revenue', 'kpi', 'executive', 'certified', 'hourly']
          
          # Model-level configurations
          config:
            enabled: true
            materialized: table
            
        # Count Distinct Metric
        - name: unique_customers
          label: "Unique Active Customers"
          model: ref('fct_customer_activity')
          description: "Count of distinct customers who made at least one purchase in the period"
          
          calculation_method: count_distinct
          expression: customer_id
          timestamp: activity_date
          time_grains: [day, week, month, quarter, year]
          
          filters:
            - field: activity_type
              operator: '='
              value: "'purchase'"
            - field: customer_status
              operator: '='
              value: "'active'"
          
          dimensions:
            - customer_tier
            - acquisition_channel
            - geography
          
          meta:
            team: "Customer Analytics"
            business_critical: true
          
          tags: ['customers', 'acquisition', 'retention']
      
        # Average Metric with complex filtering
        - name: average_order_value
          label: "Average Order Value"
          model: ref('fct_orders')
          description: "Average value of orders excluding returns and cancelled orders"
          
          calculation_method: average
          expression: order_total
          timestamp: order_date
          time_grains: [day, week, month, quarter, year]
          
          filters:
            - field: order_status
              operator: 'not in'
              value: "('cancelled', 'returned', 'refunded')"
            - field: order_total
              operator: 'between'
              value: "1 and 50000"
            - field: is_test_order
              operator: '='
              value: "false"
          
          dimensions:
            - product_category
            - customer_type
            - payment_method
            - device_type
          
          window:
            count: 3
            period: month
          
          meta:
            accuracy_threshold: 0.95
            seasonality_adjusted: true
          
          tags: ['orders', 'average', 'commercial']
      
        # Min/Max Metrics
        - name: minimum_order_value
          label: "Minimum Order Value"
          model: ref('fct_orders')
          calculation_method: min
          expression: order_total
          timestamp: order_date
          time_grains: [day, week, month]
          
          filters:
            - field: order_total
              operator: '>'
              value: "0"
          
        - name: maximum_order_value
          label: "Maximum Order Value"
          model: ref('fct_orders')
          calculation_method: max
          expression: order_total
          timestamp: order_date
          time_grains: [day, week, month]
      
        # Ratio Metric
        - name: conversion_rate
          label: "Website Conversion Rate (%)"
          description: "Percentage of website visitors who complete a purchase"
          calculation_method: ratio
          numerator: 
            name: completed_purchases
            model: ref('fct_conversions')
            expression: count(*)
            filters:
              - field: conversion_type
                operator: '='
                value: "'purchase'"
          denominator:
            name: total_website_visits
            model: ref('fct_website_sessions')
            expression: count(distinct session_id)
            filters:
              - field: is_bot
                operator: '='
                value: "false"
          
          timestamp: event_date
          time_grains: [day, week, month, quarter]
          
          dimensions:
            - traffic_source
            - device_category
            - geography
          
          # Ratio-specific configuration
          config:
            treat_nulls_as_zero: false
            
          meta:
            format: "percentage"
            decimal_places: 2
            benchmark_rate: 2.3
          
          tags: ['conversion', 'website', 'ratio', 'percentage']
      
        # Expression Metric (Custom SQL)
        - name: customer_lifetime_value
          label: "Customer Lifetime Value"
          model: ref('fct_customer_metrics')
          description: "Average revenue per customer over their entire relationship"
          
          calculation_method: expression
          expression: |
            sum(total_customer_revenue) / count(distinct customer_id)
          timestamp: calculation_date
          time_grains: [month, quarter, year]
          
          filters:
            - field: customer_status
              operator: '='
              value: "'active'"
            - field: total_customer_revenue
              operator: '>'
              value: "0"
          
          dimensions:
            - acquisition_cohort
            - customer_segment
            - subscription_plan
          
          window:
            count: 24
            period: month
          
          meta:
            calculation_complexity: "high"
            model_version: "2.1"
            confidence_interval: "95%"
          
          tags: ['ltv', 'customers', 'complex', 'predictive']
      
        # Derived Metric (calculated from other metrics)
        - name: revenue_growth_rate
          label: "Month-over-Month Revenue Growth Rate"
          description: "Percentage change in revenue compared to previous month"
          
          calculation_method: derived
          expression: |
            (
              {{ metric('total_revenue', grain='month') }} - 
              {{ metric('total_revenue', grain='month', offset=-1) }}
            ) / {{ metric('total_revenue', grain='month', offset=-1) }} * 100
          
          timestamp: sale_timestamp
          time_grains: [month, quarter, year]
          
          dimensions:
            - product_category
            - region
          
          # Derived metric specific configs
          config:
            require_non_null_base_metrics: true
          
          meta:
            format: "percentage"
            growth_target: 10.0
            alert_threshold_low: 0.0
            alert_threshold_high: 50.0
            seasonality_notes: "Typically higher in Q4"
          
          tags: ['growth', 'mom', 'derived', 'percentage', 'executive']
      
        # Advanced Count Metric with complex conditions
        - name: high_value_customers
          label: "High Value Customer Count"
          model: ref('dim_customers')
          description: "Count of customers with lifetime spend above $10,000"
          
          calculation_method: count
          expression: "*"
          timestamp: last_purchase_date
          time_grains: [day, week, month, quarter]
          
          filters:
            - field: lifetime_spend
              operator: '>='
              value: "10000"
            - field: customer_status
              operator: '='
              value: "'active'"
            - field: last_purchase_date
              operator: '>='
              value: "current_date - interval '365 days'"
          
          dimensions:
            - customer_tier
            - acquisition_channel
            - geographic_region
            - account_manager
          
          window:
            count: 6
            period: month
          
          meta:
            threshold_amount: 10000
            currency: "USD"
            review_frequency: "monthly"
            stakeholders: ['sales', 'customer_success', 'finance']
          
          tags: ['customers', 'high_value', 'segmentation', 'sales']
      
        # Time-based Metric with multiple time grains
        - name: daily_active_users
          label: "Daily Active Users (DAU)"
          model: ref('fct_user_sessions')
          description: "Count of unique users who had at least one session per day"
          
          calculation_method: count_distinct
          expression: user_id
          timestamp: session_date
          time_grains: [day, week, month]
          
          filters:
            - field: session_duration_minutes
              operator: '>='
              value: "1"
            - field: is_test_user
              operator: '='
              value: "false"
          
          dimensions:
            - user_segment
            - platform
            - feature_accessed
            - subscription_tier
          
          # Rolling averages
          window:
            count: 7
            period: day
          
          meta:
            metric_type: "engagement"
            reporting_level: "product"
            benchmark_external: "industry_average"
            data_freshness_sla: "30 minutes"
          
          tags: ['engagement', 'daily', 'users', 'product', 'dau']
      
        # Financial Metric with comprehensive metadata
        - name: gross_margin_percentage
          label: "Gross Margin %"
          model: ref('fct_financial_summary')
          description: |
            Gross margin percentage calculated as (Revenue - COGS) / Revenue.
            Excludes one-time charges and extraordinary items.
          
          calculation_method: expression
          expression: |
            case 
              when sum(gross_revenue) > 0 
              then (sum(gross_revenue) - sum(cost_of_goods_sold)) / sum(gross_revenue) * 100
              else null 
            end
          
          timestamp: financial_period_end_date
          time_grains: [month, quarter, year]
          
          filters:
            - field: is_extraordinary_item
              operator: '='
              value: "false"
            - field: financial_period_type
              operator: '='
              value: "'regular'"
          
          dimensions:
            - product_line
            - business_unit
            - geography
            - sales_channel
          
          config:
            enabled: true
            materialized: table
            post_hook: "{{ log('Gross margin calculated for period: ' ~ var('current_period'), info=true) }}"
          
          meta:
            financial_statement: "P&L"
            gaap_compliant: true
            auditor_reviewed: true
            target_margin: 65.0
            industry_benchmark: 58.2
            calculation_owner: "Finance Team"
            approval_required: true
            sensitivity: "confidential"
            regulatory_reporting: true
            variance_threshold: 5.0
            forecast_accuracy: "±2%"
          
          tags: ['financial', 'margin', 'profitability', 'gaap', 'confidential', 'executive']
      
      # Global metric configurations (optional)
      semantic_model_defaults:
        materialized: table
        
      # Metric groups for organization
      metric_groups:
        - name: revenue_metrics
          label: "Revenue & Financial Metrics"
          description: "Key financial performance indicators"
          metrics:
            - total_revenue
            - gross_margin_percentage
            - revenue_growth_rate
          
        - name: customer_metrics
          label: "Customer Analytics"
          description: "Customer behavior and segmentation metrics"
          metrics:
            - unique_customers
            - customer_lifetime_value
            - high_value_customers
          
        - name: product_metrics
          label: "Product & Engagement"
          description: "Product usage and engagement metrics"
          metrics:
            - daily_active_users
            - conversion_rate
            - average_order_value
```
#### 17- dbt commands : 

##### Core Development Commands

#### `dbt run`
Executes SQL models in your dbt project.
```bash
# Run all models
dbt run

# Run specific models
dbt run --models my_model
dbt run --models +my_model  # Include upstream dependencies
dbt run --models my_model+  # Include downstream dependencies
dbt run --models +my_model+ # Include both upstream and downstream

# Run models by tag
dbt run --models tag:daily
dbt run --models tag:finance,tag:marketing

# Run models in specific directory
dbt run --models models/staging
dbt run --models staging.salesforce

# Exclude specific models
dbt run --exclude my_model
dbt run --exclude tag:deprecated

# Run with threads (parallel execution)
dbt run --threads 4
```

#### `dbt test`
Runs tests defined in your project.
```bash
# Run all tests
dbt test

# Run tests for specific models
dbt test --models my_model
dbt test --models staging.users

# Run specific test types
dbt test --select test_type:generic
dbt test --select test_type:singular

# Run tests with store_failures (save failed test results)
dbt test --store-failures
```

#### `dbt build`
Runs models, tests, snapshots, and seeds in dependency order.
```bash
# Build everything
dbt build

# Build specific selection
dbt build --models +my_model+
dbt build --select tag:daily
```

##### Model Selection and Graph Operations

###### Advanced Selection Syntax
```bash
# Multiple model selection
dbt run --models model_a model_b model_c

# Graph operators
dbt run --models @my_model      # @ operator (shorthand for +model+)
dbt run --models 1+my_model     # 1 degree upstream
dbt run --models my_model+2     # 2 degrees downstream
dbt run --models 2+my_model+1   # 2 upstream, 1 downstream

# Resource type selection
dbt build --select resource_type:model
dbt build --select resource_type:test
dbt build --select resource_type:snapshot
dbt build --select resource_type:seed

# Package selection
dbt run --models package:my_package
```

##### Development & Debugging Commands

#### `dbt compile`
Compiles dbt models to raw SQL without executing.
```bash
dbt compile
dbt compile --models my_model
```

#### `dbt parse`
Parses dbt project and updates manifest.json.
```bash
dbt parse
```

#### `dbt ls` (list)
Lists resources in your dbt project.
```bash
# List all models
dbt ls --resource-type model

# List models with selection
dbt ls --models staging.*
dbt ls --select tag:daily

# Output formats
dbt ls --output json
dbt ls --output name
dbt ls --output path
```

#### `dbt show`
Preview SQL results without materializing.
```bash
# Show compiled SQL and sample results
dbt show --models my_model

# Limit number of rows
dbt show --models my_model --limit 10
```

#### Data Management Commands

#### `dbt seed`
Loads CSV files from data/ directory into your warehouse.
```bash
# Load all seeds
dbt seed

# Load specific seed
dbt seed --select my_seed_file

# Full refresh (drop and recreate)
dbt seed --full-refresh
```

#### `dbt snapshot`
Executes snapshot models for slowly changing dimensions.
```bash
# Run all snapshots
dbt snapshot

# Run specific snapshot
dbt snapshot --select my_snapshot
```

#### `dbt run-operation`
Executes macros directly.
```bash
# Run a macro
dbt run-operation my_macro

# Run macro with arguments
dbt run-operation grant_select --args '{table: "my_table", role: "reporting"}'
```

#### Documentation & Lineage

#### `dbt docs`
```bash
# Generate documentation
dbt docs generate

# Serve documentation locally
dbt docs serve --port 8080

# Generate and serve
dbt docs generate && dbt docs serve
```

#### Environment & Configuration

#### `dbt debug`
Validates your dbt installation and project configuration.
```bash
dbt debug
```

#### `dbt deps`
Downloads dependencies specified in packages.yml.
```bash
dbt deps
```

#### `dbt clean`
Removes dbt artifacts (target/, logs/, dbt_packages/).
```bash
dbt clean
```

##### State-based Commands (Slim CI)

#### `dbt run --state`
```bash
# Run modified models only
dbt run --models state:modified --state path/to/manifest

# Run new models
dbt run --models state:new --state ./prod-manifest

# Run modified and downstream
dbt run --models state:modified+ --state ./target-prod
```

#### 17- Slim CI :  

Slim CI is a dbt feature that enables running only the modified models and their dependencies, dramatically reducing CI/CD execution time and cost. It compares the current state of project against a previous state (usually production) to determine what has changed.  

#### Prerequisites for Slim CI

1. **State artifacts**: You need manifest.json from your production environment
2. **dbt version**: 0.18.0 or higher
3. **Git workflow**: Changes detected through file modifications

#### State Selection Operators :  

```bash
# Modified models only
dbt run --select state:modified --state ./prod-artifacts

# New models only  
dbt run --select state:new --state ./prod-artifacts

# Modified + downstream dependencies
dbt run --select state:modified+ --state ./prod-artifacts

# Modified + upstream dependencies
dbt run --select +state:modified --state ./prod-artifacts

# Modified + upstream and downstream
dbt run --select +state:modified+ --state ./prod-artifacts

# Combine with other selectors
dbt run --select state:modified,tag:daily --state ./prod-artifacts
```

#### Defer Configuration

This is so important especially in CI part where schemas created are ephemeral for test purposes : 

The `--defer` flag tells dbt to use production relations for unchanged models. Normaly we would tell dbt to run only modified objects and the downstreams that depend on them.  

**But there’s a problem !**  If our branch doesn’t include all models (for example, we only changed one staging model), dbt doesn’t know how to resolve references to the rest of the DAG unless it builds everything, which defeats the purpose of Slim CI.  

Enter The --defer flag that tells dbt:  

“If a model isn’t present (or selected) in this run, assume it already exists in another environment (the one you pointed --state at).”  

dbt will defer building unchanged models and instead resolve references to them against the manifest/state from our environment.  

:

```bash
# Use production relations for unselected models
dbt run --select state:modified+ --defer --state ./prod-artifacts
```


#### GitLab CI Slim CI Configuration Example :  

```yaml
# .gitlab-ci.yml
stages:
  - test
  - deploy

variables:
  DBT_PROFILES_DIR: "./"
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

cache:
  paths:
    - .cache/pip
    - dbt_packages/

before_script:
  - python --version
  - pip install dbt-postgres>=1.0.0
  - dbt deps

dbt_test:
  stage: test
  image: python:3.9
  script:
    - dbt debug
    # Download production state
    - aws s3 cp s3://my-dbt-artifacts/prod/manifest.json ./prod-manifest.json || echo "No prod manifest found"
    # Run slim CI
    - |
      if [ -f "prod-manifest.json" ]; then
        dbt build --select state:modified+ --defer --state ./prod-manifest.json --target ci
      else
        echo "No production manifest found, running full build"
        dbt build --target ci
      fi
  only:
    - merge_requests
  environment:
    name: ci

dbt_deploy:
  stage: deploy
  image: python:3.9
  script:
    - dbt debug
    - dbt build --target prod
    # Upload new artifacts
    - aws s3 cp ./target/manifest.json s3://my-dbt-artifacts/prod/manifest.json
  only:
    - main
  environment:
    name: production
```

#### 18- Gitlab CICD : 

The CICD logic Gitlab for dbt is cruacial to design ! Normaly we need to create stages that will point to the enivronments desired and deploy the changes !  

A typical workflow would be :

1. Featue branch with changes made
2. PR request that triggers the CICD pipeline
3. lint stage verifies the quality of the code using sqlfluff lint models/ command. **Note that we need to add a sqlfluff configuration file .sqlfluff so that the library will know what we are using as tools !**
4. compile stage using dbt compile which validates all the artifacts aznd references (just like dbt parse) and also generates the sql for check
5. CI stage where we will create ephemeral schemas that will contain only the objects changes in dbt (dbt run --select state:modified+ --defer --state ./dev-manifest) ! **Note that defer here is so important especially if the modified models have relations with other models that were not modified (hense not selected) ! without defer, some tests referencing other models not changed will not work !!**. defer tells dbt “If a model isn’t being built in this run, resolve its ref() to an already-built version in another environment.” The CI Stage contains a final script that removes all the ephemeral generated objects.
6. Dev stage : after CI sucess it deploys to dev
7. Test stage : after dev success it deploys to test so that business can test
8. Prod : after manual approaval (Continuous delivery), deployment happens in PROD

**sqlfluff** :  

The sqlfluff-templater-dbt plugin is critical — without it, sqlfluff won’t understand Jinja, ref(), source(), macros, etc.  
```
  pip install sqlfluff sqlfluff-templater-dbt

```

We add the .sqlfluff config file to your repo, Example:  

```
    # .sqlfluff
    # SQLFluff configuration for SQL linting
    
    [sqlfluff]
    # Supported dialects https://docs.sqlfluff.com/en/stable/dialects.html
    dialect = snowflake
    templater = dbt
    large_file_skip_byte_limit = 40000
    
    # Rules to exclude
    exclude_rules = L034,L036,L044
    
    [sqlfluff:indentation]
    # Indentation configuration
    tab_space_size = 2
    indent_unit = space
    
    [sqlfluff:templater]
    unwrap_wrapped_queries = true
    
    [sqlfluff:templater:dbt]
    # Configure dbt templater
    project_dir = ./
    profiles_dir = ./profiles
    profile = default
    
    [sqlfluff:rules]
    # Rule-specific configuration
    max_line_length = 120
    single_table_references = consistent
    unquoted_identifiers_policy = all
    
    [sqlfluff:rules:L010]
    # Keywords should be consistently upper case
    capitalisation_policy = upper
    
    [sqlfluff:rules:L030]
    # Function names should be consistently upper case
    extended_capitalisation_policy = upper
    
    [sqlfluff:rules:L040]
    # Null & Boolean Literals should be consistently upper case
    capitalisation_policy = upper
    
    [sqlfluff:rules:L047]
    # Trailing commas within select clause
    select_clause_trailing_comma = forbid
    
    [sqlfluff:rules:L052]
    # Semi-colon formatting approach
    multiline_newline = true
    require_final_semicolon = false
    
    [sqlfluff:rules:L054]
    # GROUP BY/ORDER BY column references
    group_by_and_order_by_style = consistent
    
    [sqlfluff:rules:L063]
    # Data Types should be consistently upper case
    extended_capitalisation_policy = upper
```

This tells sqlfluff:

  - Use Snowflake SQL dialect.
  - Parse SQL with dbt (so Jinja + macros are handled).
  - Which rules to enforce (warn/error/ignore).
  
CICD Workflow Example :  

```
      stages:
        - lint
        - parse
        - ci-test
        - deploy-dev
        - deploy-test
        - deploy-prod
      
      default:
        image: python:3.10
        before_script:
          - pip install --upgrade pip
          - pip install sqlfluff sqlfluff-templater-dbt dbt-snowflake
          # Define dbt profiles dynamically (no hardcoded creds)
          - mkdir -p ~/.dbt
          - |
            cat > ~/.dbt/profiles.yml <<EOL
            my_project:
              target: $DBT_TARGET
              outputs:
                ci:
                  type: snowflake
                  account: $SNOWFLAKE_ACCOUNT
                  user: $SNOWFLAKE_USER
                  password: $SNOWFLAKE_PASSWORD
                  role: $SNOWFLAKE_ROLE
                  warehouse: $SNOWFLAKE_WAREHOUSE
                  database: $SNOWFLAKE_DATABASE
                  schema: "CI_${CI_COMMIT_REF_SLUG}_${CI_PIPELINE_ID}"
                dev:
                  type: snowflake
                  account: $SNOWFLAKE_ACCOUNT
                  user: $SNOWFLAKE_USER
                  password: $SNOWFLAKE_PASSWORD
                  role: $SNOWFLAKE_ROLE
                  warehouse: $SNOWFLAKE_WAREHOUSE
                  database: $SNOWFLAKE_DATABASE
                  schema: DEV
                test:
                  type: snowflake
                  account: $SNOWFLAKE_ACCOUNT
                  user: $SNOWFLAKE_USER
                  password: $SNOWFLAKE_PASSWORD
                  role: $SNOWFLAKE_ROLE
                  warehouse: $SNOWFLAKE_WAREHOUSE
                  database: $SNOWFLAKE_DATABASE
                  schema: TEST
                prod:
                  type: snowflake
                  account: $SNOWFLAKE_ACCOUNT
                  user: $SNOWFLAKE_USER
                  password: $SNOWFLAKE_PASSWORD
                  role: $SNOWFLAKE_ROLE
                  warehouse: $SNOWFLAKE_WAREHOUSE
                  database: $SNOWFLAKE_DATABASE
                  schema: PROD
            EOL
      
      variables:
        DBT_TARGET: "ci"
      
      # -------------------
      # Stage 1: Lint
      # -------------------
      lint:
        stage: lint
        script:
          - sqlfluff lint models/
      
      # -------------------
      # Stage 2: Parse
      # -------------------
      parse:
        stage: parse
        script:
          - dbt parse
          - mkdir -p artifacts
          - cp target/manifest.json artifacts/
        artifacts:
          paths:
            - artifacts/manifest.json
          expire_in: 1 week
      
      # -------------------
      # Stage 3: CI Tests (Ephemeral Schema + Defer)
      # -------------------
      ci-test:
        stage: ci-test
        script:
          - dbt run --target ci --select state:modified --defer --state artifacts
          - dbt test --target ci --select state:modified --defer --state artifacts
        after_script:
          - |
            echo "Cleaning up ephemeral schema..."
            snowsql -a $SNOWFLAKE_ACCOUNT -u $SNOWFLAKE_USER -r $SNOWFLAKE_ROLE \
              -q "drop schema if exists CI_${CI_COMMIT_REF_SLUG}_${CI_PIPELINE_ID} cascade;" || true
      
      # -------------------
      # Stage 4: Deploy to DEV
      # -------------------
      deploy-dev:
        stage: deploy-dev
        script:
          - dbt run --target dev
          - dbt test --target dev
        rules:
          - if: $CI_COMMIT_BRANCH == "develop"
      
      # -------------------
      # Stage 5: Deploy to TEST (manual approval)
      # -------------------
      deploy-test:
        stage: deploy-test
        when: manual
        script:
          - dbt run --target test
          - dbt test --target test
        rules:
          - if: $CI_COMMIT_BRANCH == "main"
      
      # -------------------
      # Stage 6: Deploy to PROD (manual approval)
      # -------------------
      deploy-prod:
        stage: deploy-prod
        when: manual
        script:
          - dbt run --target prod
          - dbt test --target prod
        rules:
          - if: $CI_COMMIT_BRANCH == "main"
```


--------------------------------------------------------------------
### Complete dbt Functions & Macros Reference Guide
---------------------------------------------------------------------


#### Core dbt Jinja Functions

### 1. `ref()`
**Purpose**: Reference other models in your dbt project and establish lineage
```sql
-- Basic usage
select * from {{ ref('my_model') }}

-- With package
select * from {{ ref('my_package', 'my_model') }}

-- In a macro
{% macro get_payment_methods() %}
    select distinct payment_method 
    from {{ ref('fct_orders') }}
{% endmacro %}
```

### 2. `source()`
**Purpose**: Reference raw data sources defined in schema.yml
```sql
-- Basic usage
select * from {{ source('raw_data', 'customers') }}

-- In a macro
{% macro get_latest_orders(days_back=7) %}
    select * from {{ source('raw_data', 'orders') }}
    where order_date >= current_date - {{ days_back }}
{% endmacro %}
```

### 3. `var()`
**Purpose**: Use variables for configuration and dynamic behavior
```sql
-- In dbt_project.yml
vars:
  start_date: '2023-01-01'
  include_test_data: false

-- In model
select * from orders 
where order_date >= '{{ var("start_date") }}'
{% if var("include_test_data") %}
    -- include test records
{% else %}
    and is_test = false
{% endif %}

-- With default value
where status = '{{ var("order_status", "completed") }}'
```

### 4. `target`
**Purpose**: Access information about the current target (environment)
```sql
-- Environment-specific logic
{% if target.name == 'prod' %}
    select * from {{ ref('dim_customers') }}
{% else %}
    select * from {{ ref('dim_customers') }} limit 1000
{% endif %}

-- Different schemas per environment
{% if target.name == 'dev' %}
    {{ config(schema='dev_' + target.user) }}
{% endif %}

-- Available target properties:
-- target.name, target.schema, target.type, target.user, target.warehouse
```

### 5. `this`
**Purpose**: Reference the current model being built
```sql
-- In pre/post hooks
{{ config(
    pre_hook="delete from {{ this }} where updated_at < current_date - 30"
) }}

-- In macros
{% macro log_model_info() %}
    {% do log("Building model: " ~ this, info=true) %}
{% endmacro %}
```

### 6. `graph`
**Purpose**: Access the dbt project graph information
```sql
-- Get all models that reference current model
{% for node in graph.nodes.values() %}
    {% if this in node.depends_on.nodes %}
        -- {{ node.name }} depends on this model
    {% endif %}
{% endfor %}
```

## Jinja Control Structures

### 1. Conditionals
```sql
-- Basic if/else
{% if var('include_deleted') %}
    select * from customers
{% else %}
    select * from customers where deleted_at is null
{% endif %}

-- Multiple conditions
{% if target.name == 'prod' and var('full_refresh', false) %}
    truncate table {{ this }}
{% elif target.name == 'dev' %}
    -- dev logic
{% endif %}
```

### 2. Loops
```sql
-- For loop with list
{% set payment_methods = ['card', 'cash', 'bank_transfer'] %}
select 
  order_id,
  {% for method in payment_methods %}
  sum(case when payment_method = '{{ method }}' then amount else 0 end) as {{ method }}_total
  {%- if not loop.last -%},{%- endif %}
  {% endfor %}
from payments group by 1

-- For loop with dictionary
{% set status_mapping = {'P': 'Pending', 'C': 'Complete', 'F': 'Failed'} %}
case 
  {% for key, value in status_mapping.items() %}
  when status = '{{ key }}' then '{{ value }}'
  {% endfor %}
end as status_description
```

### 3. Macros
```sql
-- Basic macro
{% macro cents_to_dollars(column_name, precision=2) %}
  ({{ column_name }} / 100)::numeric(16, {{ precision }})
{% endmacro %}

-- Usage: {{ cents_to_dollars('price_in_cents') }}

-- Macro with multiple parameters
{% macro generate_alias_name(custom_alias_name=none, node=none) %}
    {%- if custom_alias_name is none -%}
        {{ node.name }}
    {%- else -%}
        {{ custom_alias_name | trim }}
    {%- endif -%}
{% endmacro %}

-- Macro that returns SQL
{% macro get_date_dimension(start_date, end_date) %}
  {% set query %}
    select 
      date_day,
      extract(year from date_day) as year,
      extract(month from date_day) as month,
      extract(dayofweek from date_day) as day_of_week
    from (
      select dateadd(day, seq4(), '{{ start_date }}') as date_day
      from table(generator(rowcount => datediff(day, '{{ start_date }}', '{{ end_date }}') + 1))
    )
  {% endset %}
  {{ return(query) }}
{% endmacro %}
```

## dbt-utils Functions

### 1. Key Generation
```sql
-- surrogate_key: Generate surrogate keys
select 
  {{ dbt_utils.surrogate_key(['customer_id', 'order_date']) }} as order_key,
  customer_id,
  order_date
from orders

-- generate_surrogate_key (newer version)
select 
  {{ dbt_utils.generate_surrogate_key(['customer_id', 'order_date']) }} as order_key
from orders
```

### 2. Data Type Utilities
```sql
-- safe_cast: Cast with error handling
select 
  {{ dbt_utils.safe_cast("'2023-13-45'", api.Column.translate_type("date")) }} as invalid_date

-- cast_bool_to_text: Convert boolean to text
select 
  {{ dbt_utils.cast_bool_to_text("is_active") }} as is_active_text
```

### 3. Date/Time Functions
```sql
-- date_trunc: Cross-database date truncation
select 
  {{ dbt_utils.date_trunc('month', 'order_date') }} as order_month,
  count(*) as order_count
from orders
group by 1

-- dateadd: Cross-database date addition
select 
  order_date,
  {{ dbt_utils.dateadd('day', 30, 'order_date') }} as due_date
from orders

-- datediff: Cross-database date difference
select 
  {{ dbt_utils.datediff('order_date', 'shipped_date', 'day') }} as days_to_ship
from orders

-- last_day: Get last day of period
select 
  {{ dbt_utils.last_day('order_date', 'month') }} as month_end
from orders
```

### 4. String Functions
```sql
-- split_part: Cross-database string splitting
select 
  {{ dbt_utils.split_part('full_name', "' '", 1) }} as first_name,
  {{ dbt_utils.split_part('full_name', "' '", 2) }} as last_name
from customers

-- right: Get rightmost characters
select 
  {{ dbt_utils.right('phone_number', 4) }} as last_four_digits

-- replace: Cross-database string replacement
select 
  {{ dbt_utils.replace('phone_number', "'-'", "''") }} as clean_phone
```

### 5. Cross-Database Functions
```sql
-- listagg: Cross-database string aggregation
select 
  customer_id,
  {{ dbt_utils.listagg('product_name', "', '", "order by product_name") }} as products_ordered
from order_items
group by 1

-- bool_or: Cross-database boolean OR aggregation
select 
  customer_id,
  {{ dbt_utils.bool_or('is_returned') }} as has_any_returns
from orders
group by 1
```

### 6. Pivot/Unpivot
```sql
-- pivot: Dynamic pivot tables
select *
from (
  select 
    customer_id,
    product_category,
    order_count
  from customer_product_summary
) 
{{ dbt_utils.pivot(
    'product_category',
    dbt_utils.get_column_values(ref('customer_product_summary'), 'product_category'),
    agg='sum',
    then_value='order_count'
) }}

-- unpivot: Convert columns to rows
{{ dbt_utils.unpivot(
  relation=ref('monthly_sales'),
  cast_to='numeric',
  exclude=['customer_id', 'year'],
  remove=['total'],
  field_name='month',
  value_name='sales_amount'
) }}
```

### 7. Data Generation
```sql
-- generate_series: Create series of numbers/dates
select * from (
  {{ dbt_utils.generate_series(1, 100) }}
) as series(generated_number)

-- date_spine: Generate date range
{{ dbt_utils.date_spine(
    datepart="day",
    start_date="cast('2023-01-01' as date)",
    end_date="cast('2023-12-31' as date)"
) }}
```

### 8. Data Quality
```sql
-- get_column_values: Extract unique values
{% set payment_methods = dbt_utils.get_column_values(ref('orders'), 'payment_method') %}

-- equal_rowcount: Test for equal row counts
{{ dbt_utils.equal_rowcount(ref('orders'), ref('order_items_summary')) }}

-- fewer_rows_than: Test row count relationship
{{ dbt_utils.fewer_rows_than(ref('returned_orders'), ref('orders')) }}
```

### 9. Table Operations
```sql
-- union_relations: Union multiple tables
{{ dbt_utils.union_relations(
    relations=[ref('orders_2022'), ref('orders_2023')],
    column_override={"order_date": "date", "order_id": "varchar(50)"}
) }}

-- deduplicate: Remove duplicates
select * from (
  {{ dbt_utils.deduplicate(
      relation=ref('raw_customers'),
      partition_by='customer_id',
      order_by='updated_at desc'
  ) }}
)
```

### 10. URL/Web Functions
```sql
-- get_url_host: Extract host from URL
select 
  {{ dbt_utils.get_url_host('page_url') }} as website_host

-- get_url_path: Extract path from URL
select 
  {{ dbt_utils.get_url_path('page_url') }} as page_path
```

## Advanced Macro Patterns

### 1. Dynamic Table Creation
```sql
{% macro create_monthly_partitions(table_name, start_month, end_month) %}
  {% for month in range(start_month, end_month + 1) %}
    create table {{ table_name }}_{{ month }} as
    select * from {{ ref(table_name) }}
    where extract(month from order_date) = {{ month }};
  {% endfor %}
{% endmacro %}
```

### 2. Environment-Specific Logic
```sql
{% macro get_warehouse_size() %}
  {% if target.name == 'prod' %}
    {{ return('LARGE') }}
  {% elif target.name == 'dev' %}
    {{ return('XSMALL') }}
  {% else %}
    {{ return('SMALL') }}
  {% endif %}
{% endmacro %}

-- Usage in model config
{{ config(
    snowflake_warehouse=get_warehouse_size()
) }}
```

### 3. Custom Test Macros
```sql
-- Custom test for checking business rules
{% test expect_column_values_to_be_in_set(model, column_name, value_set) %}
  select *
  from {{ model }}
  where {{ column_name }} not in (
    {% for value in value_set %}
    '{{ value }}'{% if not loop.last %},{% endif %}
    {% endfor %}
  )
{% endtest %}
```

### 4. Data Quality Macros
```sql
{% macro test_not_null_proportion(model, column_name, at_least=0.95) %}
  select 
    count(*) as total_rows,
    count({{ column_name }}) as non_null_rows,
    count({{ column_name }}) * 1.0 / count(*) as non_null_proportion
  from {{ model }}
  having non_null_proportion < {{ at_least }}
{% endmacro %}
```

### 5. Performance Optimization Macros
```sql
{% macro optimize_table(table_name) %}
  {% if target.type == 'snowflake' %}
    alter table {{ table_name }} cluster by (order_date);
  {% elif target.type == 'bigquery' %}
    -- BigQuery optimization logic
  {% endif %}
{% endmacro %}
```

## Best Practices

1. **Use meaningful variable names** in your macros
2. **Add documentation** to complex macros using comments
3. **Handle edge cases** with proper conditionals
4. **Test macros thoroughly** in different environments
5. **Keep macros focused** on single responsibilities
6. **Use proper indentation** for readability
7. **Leverage dbt-utils** for cross-database compatibility
8. **Version control your macros** and document changes
