## This file is a recap of some of the main things to master in dbt to prepare for the exam

- Configuration at the schema or database level can be override at the object level such as table : eg freshness can be set at the schema level and override at the table level to change the loaded at column for example !
- We can use CODEGEN_PACKAGE to generate the source.yml file if we have hundreds of models
- In dbt studio we can generate the downstreams model as simple select with source refrence using the button generate model.
- We can implement tests for models we create but also for sources we use !
- run tests only on sources : dbt test --select source:*
- If we have a big doc to create we use markdown files that may contain several doc blocks and then we reference that in the description of a filed or table : {{ doc('order_status') }}
- We can also document macros by adding a yml file for that macro
- in dbt we can set a protection on a project to control the access to thoe models if called in a package : PUBLIC, PROTECTED  and PRIVATE (private needs groups to limite access to models to only same group models)

```yml
          models:
            - name: revenue
              config:
                group: finance
                access: private
```

| Access    | Can be used outside group?          | Purpose                       |
| --------- | ----------------------------------- | ----------------------------- |
| public    | ✅ Yes                               | shared models                 |
| protected | ⚠️ limited (package boundary rules) | controlled sharing            |
| private   | ❌ No                                | strict isolation within group |

- indirect selection modes :
  - Eager mode runs all tests regardless of dependencies
  - Buildable mode runs tests with dependencies in selected nodes
  - Empty mode ignores all test dependencies
  - Cautious mode restricts tests to exclusively selected node dependencies
- we can define selectors in yml file to run commands in more efficient way:
```yml
      selectors:
      - name: finance_nightly
        definition:
          union:
            - method: tag
              value: finance
            - method: tag
              value: nightly
```
- Data tests (generic tests) report the number of rows that fail their specific conditions (singular or custom test do not! they return success or fail)
- test types : generic (data tests: not_null, unique, accepted_values and relationships FAILS before materializing), custom generic tests (a macro under tests/generic/positive_values.sql NOT ENFORECED/don't fail before materializing) , singular (user defined ones that are just simple sql statements), unit tests (new feature to perform TNRs on fake data we give):

```yml
        unit_tests:
          - name: test_order_total
            model: order_summary
            given:
              - input: ref('orders')
                rows:
                  - {id: 1, amount: 10}
                  - {id: 2, amount: 20}
        
            expect:
              rows:
                - {total_amount: 30}
```
- Implementing a standardized checklist for best practices and verification
- grants can be handeld using macros or in configs (model, schema.yml or dbt_project) :
```yml
        models:
          - name: sales
            config:
              grants:
                select: ["analyst_role", "bi_role"]
```
- Revoke grants simply change the roles specified (dbt each time rebuilds the grants to be sure no grants given manually) or give empty lit or simply give empty dictionnary in the grants:

```yml
        grants:
          select: []
        
        grants: {}
```
- contracts : enforce schema on tables, views and incremental models with schema_on_change config (columns and data types and also the order), some constraints (not_null) :

```yml
        version: 2
        
        models:
          - name: customers
            config:
              contract:
                enforced: true
        
            columns:
              - name: id
                data_type: integer
                constraints:
                  - type: not_null
                  - type: primary_key
        
              - name: email
                data_type: string
        
              - name: created_at
                data_type: timestamp
```

| Feature           | Contracts  | Tests | Docs              |
| ----------------- | ---------- | ----- | ----------------- |
| Columns existence | ✅          | ❌     | ✅ (informational) |
| Data types        | ✅          | ❌     | ✅                 |
| Constraints       | ⚠️ partial | ✅     | ❌                 |
| Data quality      | ❌          | ✅     | ❌                 |

- compilation needs introspective queries to resolve dependencies that is why it fails if no connection to a plateform is established
- in dbt when we set a tag for a column, it propagates to the nodes corresponding to that column including test (only generic tests):
```yml
        columns:
          - name: ssn
            tags: ["pii"]
            tests:  # better use data_tests
              - not_null
              - unique

        dbt test --select tag:pii runs not_null_ssn and unique_ssn
```
- we can store test failures in the data plateform using store_failures config: this will generate a table for each test with the result of failing rows ! we can also set how to materialize the results using store_failures_as : table or view

```yml
          # pattern :
          dbt_test__audit.<test_name>_<model_name>_<column_name>
          # MODEL/COLUMN level
          models:
            - name: customers
              columns:
                - name: id
                  tests:
                    - unique:
                        config:
                          store_failures: true
          
          
          # for all tests in model
          models:
            - name: customers
              config:
                store_failures: true
              columns:
                - name: id
                  tests:
                    - not_null
          
          # for all tests in the project
          
          tests:
            +store_failures: true
```
- pattern of selectors in dbt :

```
      Node type (model/test/source)
      + Filter (tag/path/name/source/etc.)
      + Graph traversal (+ / @)
      + Exclusion (-)
      + State (modified/new)
```

| Method          | Example                     |
| --------------- | --------------------------- |
| `tag`           | `tag:finance`               |
| `path`          | `path:models/staging`       |
| `name`          | `name:customers`            |
| `fqn`           | `fqn:my_project.customers`  |
| `package`       | `package:jaffle_shop`       |
| `config`        | `config.materialized:table` |
| `source`        | `source:raw.orders`         |
| `exposure`      | `exposure:dashboard`        |
| `metric`        | `metric:revenue`            |
| `test_type`     | `test_type:generic`         |
| `resource_type` | `model`, `test`, etc.       |

- we can target a type of tests to run using selectors : test_type
- run all tests of a model or a column : dbt test --select customers | dbt test --select customers.email
- properties are metadata and tests elements while configurations control how dbt materializes objects in the data warehouse
- Environment variable keys in dbt are case-sensitive and must be referenced with exact casing
- The effective way to correlate execution metadata with project structure is: Join run_results.json with manifest.json using unique_id (and optionally invocation_id)
- When using --select : "tag:nightly config.materialized:incremental" means AND condition while "tag:nightly,config.materialized:incremental" means OR condition
- Views are the most cost-effective option for simple, infrequently used models.
- if we have the same model with different versions and we call it without specifying the version dbt resolves to the version marked as 'latest' or with the highest version number
- run 'dbt run' and 'dbt seed' simultaneously may result in potential data integrity problems
- dbt recognizes YAML files for tests and descriptions based on two primary criteria: the file must be located in the appropriate resource directory (such as models/, seeds/, snapshots/, or macros/) and have a .yml file extension.
- Implement post-hooks to automatically assign permission grants during model execution is the most effecient way to handel permissions in dbt
- Block names must consist only of alphanumeric characters and underscores, and cannot start with a digit
- Assign each developer a unique schema using a convention like `dbt_<username>` is powerful to avoid overriding work of other developpers
- dbt Docs requires manual deployment for static sites; Catalog updates automatically after each job run providing dynamic, always-current documentation.
- Table-level 'freshness' settings completely replace source-level configurations for the specific table
- Ephemeral materialization provides reusable logic without persisting data in the warehouse
- In dbt, the runtime does not improve beyond a certain thread count because the Directed Acyclic Graph (DAG) structure inherently constrains parallel model execution. The actual number of models that can be built concurrently is limited by the project's dependency relationships, even if more threads are available.
- in dbt contracts when we set 'alis_types: false (it is true by default)' dbt will pass data types from YAML to the database exactly as specified without converting them to platform-specific types
- Use the --select test_type:unit flag to target specific test types
- To override configs of sources of a package we do that in dbt_project file not in the sources.yml of the pachakge, otherwise it gets override when we run dbt deps :
```yml
                    sources:
                      package_name:
                        source_name:
                          table_name:
                            enabled: false
```
- table level freshness configs completely overrides source level freshness configs :
```yml
                    sources:
                      - name: stripe
                        loaded_at_field: updated_at
                        freshness:
                          warn_after: {count: 24, period: hour}
                          error_after: {count: 48, period: hour}
                    
                        tables:
                          - name: payments
                            freshness:
                              warn_after: {count: 1, period: hour}
                    
                          - name: customers
```
- statement block vs run_query :

```jinja
          {% set results = run_query("select distinct country from {{ ref('customers') }}") %}
          
          {% if execute %}
            {% set countries = results.columns[0].values() %}
          {% else %}
            {% set countries = [] %}
          {% endif %}
          
          # statement block is more flexible and can run multiple queries at once

          {% call statement('create_and_count', fetch_result=True) %}
            create temp table t as select * from {{ ref('orders') }};
            select count(*) from t;
          {% endcall %}
```
- dbt run --empty may be used in dry runs which skip parts of rendering and avoid fully evaluating Jinja for performance.
- To ensure that Jinja expressions like ref() and source() are fully resolved to capture all dependencies correctly, we use  .render() method :
```yml
          {% set _ = ref('payments').render() %}
          
          {% if some_flag %}
            select * from {{ ref('payments') }}
          {% endif %}
```
- if our warehouse supports only 10 concurrent queries, raising the thread will just make it slower : alwyas reduce the thread count to match warehouse concurrency limits to avoid queuing!
- Add '+schema: marketing' under the models:your_project:marketing section in dbt_project.yml to build marketing models in marketing schema.
- 'invalidate_hard_deletes' config in snapshots finds hard deleted records in source, and set dbt_valid_to current time if no longer exists.
- snapshots metadate fields : dbt_valid_from, dbt_valid_to, dbt_scd_id, dbt_updated_at
- Timestamp (updated_at column needed) & Check (list of columns to check unique rows to track in case updated_at column is not reliable) are snapshots strategies
- {{ % docs _overview_ % }} ....... {{ % enddocs % }} block costumizes the landing page of dbt docs web site
- To store test failures results we invok dbt test --store-failures'. the location of which can be configured in the dbt_project.yml file.
- dbt_test__audit is the default schema to store the results of a test failure
- In incremental models : on_schema_change can have 3 values {fail, append_new_columns (only adds new columns), sync_all_columns(remove deleted columns)}
- if we specify a new schema for a model, dbt will concatenate default schema in profiles.yml file with that new schema : <target_schema>_<custom_schema> (unless if we override the generate_schema_name macro)
- for databases if we specify a new database for some models it will override the default one (not concatenate like schemas) ! the macro handeling that is "generate_database_name"
- We define variables in dbt_project file or in the commande line
- To override a variabel in dbt for a run we use --vars command line
- When installing packages in dbt the value of the revision parameter can be : a branch name, a tagged release or a specific commit (full 40-character hash)  or even pull request reference (for dbt hub we use version and not revision):

```yml
          revision: 1.2.0
```
- 
