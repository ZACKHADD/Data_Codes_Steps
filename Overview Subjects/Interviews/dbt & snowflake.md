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
- sd
