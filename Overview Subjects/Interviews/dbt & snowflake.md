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
      threads: 2
      client_session_keep_alive: false
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
