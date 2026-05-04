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
- Avoid unpinned packages : this can avoid to dbt installing diffrent version each time dbt deps is run which may break the project ! dbt throw a warning if unpinned packges are detected !

```yml
          packages:
            - package: dbt-labs/dbt_utils
              version: ">=1.0.0"
```
- Removing a package from packages.yml will not remove the package from the project ! it still exists in your dbt_packages/ directory. If you want to completely uninstall a package, you should either: delete the package directory in dbt_packages/ or run dbt clean to delete all packages (and any compiled models), followed by dbt deps.
- A metric is a timeseries aggregation over a table that supports zero or more dimensions. They appear in pink
- To define a metric we use :
```yml
          metrics:
            - name: total_revenue
              label: Total Revenue
              model: ref('orders')
              calculation_method: sum
              expression: amount
              timestamp: order_date
              time_grains: [day, week, month]
              filter:
                - field: status
                  operator: "="
                  value: completed

          # we can also reuse the logic for several metrics :
          
          - name: conversion_rate
            calculation_method: derived
            expression: orders / visitors
            metrics:
              - name: orders
              - name: visitors

          # newer approche (semantic layer):
          
          Name,
          Model (not required for expression metrics),
          Type,
          Sql,
          Timestamp,
          time_grains

          # For expressions No model needed because it uses other metrics
          metrics:
            - name: conversion_rate
              type: expression
              sql: orders / visitors

```
- Expression metrics : This metric type is defined as any non-aggregating calculation of 1 or more metrics : we can create with that ratios, subtractions, any arbitrary calculation
- Available types are count, count_distinct, sum, average, min, max, expression
- --select, --exclude --selector are the only arguments used with dbt snapshot command (no --defer allowed)
- dbt run --select 3+my_model+4 : select the model and un to its 3 parent and down to its 4 child
- graph selectors : +, n+n, @, * (@ to be used in complex selections using tags states ..)
- dbt run -s @my_model writes the command to select 'my_model, it's children and the parents of it's children
- 3 ways to run models in a sub folder :
```bash
          dbt run -s staging.harvest.* or
          dbt run -s models/staging/harvest or
          dbt run -s path:models/staging/harvest
```
- set operators in dbt commands : union (,) and intersection (space)
- select all models of a source: dbt run --select source:snowplow+
- we can select models also based on configs (depends on warehouses) for example clustering column in snowflake : dbt run -s config.cluster_by:client_id
- Deferral requires both --defer and --state flags to be set.
- states in dbt selector :

| Selector           | Meaning                                                                     |
| ------------------ | --------------------------------------------------------------------------- |
| `state:modified`   | Models that changed vs reference state                                      |
| `state:unmodified` | Models that did NOT change                                                  |
| `state:new`        | Models that exist in current project but not in state                       |
| `state:deleted`    | Models that existed in state but no longer exist in current project         |
| `state:old`        | (less common alias in some docs/tools) same idea as deleted/orphaned models |

- if we want to compare what was deleted compared to a state (prod for example) we use dbt ls --state prod --select state:deleted (even run and build will not materialize since we no longer the code)
- the defer and state flages can be passed in the cli or we can set two environment variable : DBT_DEFER_TO_STATE and DBT_ARTIFACT_STATE_PATH (dbt checks for these variables and if set it uses them in simple commandes like : dbt build ( it adds --defer --state path/to/prod)
- if tests of parent models fail the build of the child models is skipped
- manifest.json for DAG graphs, catalog.json for metadata from datawarehouse (freshness ...) and perf_info.json for dbt performance debug, sources.json for sources freshness details(dbt source freshness), semantic_manifest/json (Semantic layer / metrics artifacts), partial_parse.msgpack (for fast startup and partial parse)

| Artifact                 | Purpose                  |
| ------------------------ | ------------------------ |
| `manifest.json`          | DAG + metadata           |
| `catalog.json`           | warehouse schema         |
| `run_results.json`       | execution results        |
| `sources.json`           | freshness results        |
| `perf_info.json`         | performance profiling    |
| `semantic_manifest.json` | metrics & semantic layer |
| `partial_parse.msgpack`  | parsing cache            |
| `graph_summary.json`     | DAG summary              |
| `compiled/`              | compiled SQL             |
| `run/`                   | executed SQL             |
| `index.html`             | docs UI                  |


- dbt debug --config-dir will show where .dbt configuration directory is located.
- parse in dbt creates a perf_info.json where it writes time of each operation : it makes it easy to debug why dbt is slow
```json
          {
            "parse_project_elapsed": 1.23,
            "load_macros_elapsed": 0.45,
            "compile_elapsed": 2.10,
            "execute_elapsed": 15.67,
            "adapter_timings": [
              {
                "connection_open": 0.2,
                "execute": 14.9
              }
            ]
          }
```
- the -o (or --output) flag is used to override the file destination in target folder i.e. dbt source freshness --output target/file_name.json to override sources.json
- dbt ls -s config.materialized:view --output json this creates a file .json that lists all the models materialized as views
- -x and --fail-fast flags makes dbt exit immediately if a single resource fails to build
- dbt --version and dbt -r or --record-timing-info are cli supported dbt commands
- global config can be set in CLI, Environment Variables or Yaml configs (usually profiles.yml)
- to cache schemas related to selected resources for the current run we need cache_selected_only: true (to be set in the dbt_project.yml file or profiles.yml) ! it only only cache schemas that are relevant to the selected resources in the current run which improve performance since dbt will not scan all the schemas (default behaviour) !
- dbt --no-version-check run disable the --version-check config (faster CI/CD) especially if we controle dbt version
- we use 'quiet' config to show only error logs in stdout
- By disabling write_json config we stop dbt from writing json artifacts (eg. manifest.json, run_results.json) to the target/ directory (this can be useful to speed up the CI in ephemeral envs)
- exit code 0 means the dbt invocation completed without error
- full re-parsing is trigegred if one of these changes :
```yml
          --vars
          profiles.yml content (or env_var values used within)
          dbt_project.yml content (or env_var values used within)
          installed packages
          dbt version
          certain widely-used macros, e.g. builtins overrides or generate_x_name for database/schema/alias
```
- ways to optimize dbt performance :

```yml
- LibYAML bindings for PyYAML (uses C (compiled code) instead of pure python to parse yml files which is faster)
- Partial parsing, which avoids re-parsing unchanged files between invocations
- An experimental parser, which extracts information from simple models much more quickly (it falls back to full parsing if we have macros, loops or conditionnal models and operations)
- RPC server, which keeps a manifest in memory, and re-parses the project at server startup/hangup : dbt stays alive instead of restarting each time
```
- After parsing the project, dbt stores an internal manifest in partial_parse.msgpack
- we can override the seeds folder using seed-paths: ["cust_seeds", "other_seeds"]
- By default, dbt will search in all resource paths for docs blocks (i.e. the combined list of model-paths, seed-paths, analysis-paths, macro-paths and snapshot-paths). If this option is configured, dbt will only look in the specified directory for docs blocks. Example: docs-paths: ["docs"]
- dispatch to override packages :
```yml
          dispatch:
            - macro_namespace: dbt_utils
              search_order: ['my_project', 'dbt_utils']
```
- the network representation of the dbt resource DAG is stored in graph.gpickle
- git branch -m feature_products renames the current branch
- git branch -d feature_a to delete branchs but only merged ones otherwise it throws an error and ask to use : git branch -D feature_a
- git branch -r : list all remote branchs
- 'git branch <branchname>' creates the branch but does not selects it so we need to checkout
- git reset dim_orders unstage dim_orders file
- using git commit -am "commit message" or git commit -a -m "commit message" makes it possible to add changes and commit them in the same time
- git reset HEAD~2 (Removes the last 2 commits from project history)
- persist_docs:
```yml
          models:
            my_project:
              +persist_docs:
                relation: true
                columns: true
```
- source supports only one config wich is enabled (other things are properties)
- quote_columns and column_types are seeds configurations
- sql_header (sql queries that run before for example sql_header="set timezone = 'UTC';" ) ( and materialized are two configs in models
- configs for tests : where, severity, warn_if, error_if, limit, fail_calc, store_failures.
- configs for snapshots : target_schema, target_database, unique_key, strategy, updated_at, check_cols, invalidate_hard_deletes
- if {{ config( full_refresh = false) }} if set the --full-refresh command will not work when invoked ! The config takes precedence over the flag.
- the leading + is in fact only required when you need to disambiguate between resource paths and configs.
- just like sql models, we can configure python models configs in the config, the dbt_project.yml and the schema.yml files :

```yml
          # Model level
          def model(dbt, session):
              dbt.config(
                  materialized="table",
                  tags=["finance", "daily"],
                  schema="analytics"
              )
          
              df = dbt.ref("stg_orders")
              return df
          # dbt_project.yml
          models:
            my_project:
              python_models:
                +materialized: table
                +tags:
                  - finance
                finance:
                  +schema: analytics
                  +tags:
                    - daily
          # schema.yml
          models:
            - name: my_python_model
              description: "A Python model for finance data"
              config:
                materialized: table
                tags:
                  - finance
                  - daily
                schema: analytics
              columns:
                - name: order_id
                  description: "Primary key"
                  tests:
                    - unique
                    - not_null
```
- It is recommended to include as many columns as possible in the snapshot, even if they do not seem useful at the moment, as snapshots cannot be recreated
- by default dbt does not quote name but we can change that using the quote config
- When running a dbt project with dbt run --select python_model, dbt will prepare and pass in both arguments (dbt and session) to the model() function:
          - dbt: A class compiled by abt Core, unique to each model, enables you to run your Python code in the context of your dbt project and DAG.
          - session: A class representing your data platform's connection to the Python backend. The session is needed to read in tables as DataFrames, and to write DataFrames back to tables. In PySpark, by convention, the SparkSession is named spark, and available globally. For consistency across platforms, we always pass it into the model function as an explicit argument called session.
- log outputs : we can controle that :
```bash
          dbt run --log-level warn     # only show warnings and errors
          dbt run --debug              # show all debug logs
          dbt run --log-format json    # structured JSON logs for ingestion into log tools
```
Here's everything consolidated into one table:The visualizer seems to be timing out. Here's the full consolidated table in Markdown instead:

| Command | Severity | Log / Message | Cause |
|---|---|---|---|
| `dbt run` | ERROR | `Database Error` | SQL compilation or execution failure in the warehouse |
| `dbt run` | ERROR | `Relation already exists` | Model conflicts with an existing warehouse object |
| `dbt run` | ERROR | `Runtime Error: Got X results, expected Y` | `unique_key` returning multiple rows during incremental run |
| `dbt run` | ERROR | `Node not found` | A `ref()` or `source()` points to a non-existent model |
| `dbt run` | ERROR | `Compilation Error` | Jinja syntax error in a `.sql` or `.py` model |
| `dbt run` | ERROR | `Permission denied` | Warehouse user lacks CREATE/INSERT rights on target schema |
| `dbt run` | WARN | `On-run-start/end hook failed` | A hook ran but did not block execution |
| `dbt run` | WARN | `Deprecation Warning: config X is deprecated` | Using an old config key that will be removed in a future dbt version |
| `dbt run` | WARN | `Model is disabled` | A model is referenced but has `enabled: false` in config |
| `dbt test` | ERROR | `Test failed: X failures` | Data test found rows that violated the assertion |
| `dbt test` | ERROR | `Database Error in test` | The test SQL itself failed to execute (not a data failure) |
| `dbt test` | ERROR | `Compilation Error in test` | Jinja error in a custom or generic test definition |
| `dbt test` | ERROR | `Node not found` | Test references a model or source that doesn't exist |
| `dbt test` | WARN | `Test is disabled` | Test has `enabled: false` |
| `dbt test` | WARN | `No tests defined for model X` | Model exists but has no tests in `.yml` |
| `dbt test` | WARN | `warn_if / error_if threshold met` | Test uses `warn_if: ">0"` and the threshold is reached |
| `dbt compile` | ERROR | `Compilation Error` | Invalid Jinja, bad `ref()`, or malformed SQL |
| `dbt compile` | ERROR | `Node not found` | A `ref()` or `source()` target doesn't exist in the project |
| `dbt compile` | ERROR | `Ambiguous ref: X` | Multiple models share the same name across packages |
| `dbt compile` | ERROR | `Invalid config value` | A config key has the wrong type or invalid value |
| `dbt compile` | WARN | `Unused variable in Jinja` | A `set` variable is defined but never used |
| `dbt compile` | WARN | `Deprecation Warning` | Using deprecated Jinja functions or config keys |
| `dbt build` | ERROR | `Build failed for node X` | Any node (model, test, seed, snapshot) failed, halting downstream nodes |
| `dbt build` | ERROR | `Depends on a node that failed` | A downstream model was skipped because an upstream node errored |
| `dbt build` | WARN | `Skipping X because of earlier failure` | Upstream failure caused this node to be skipped |
| `dbt source freshness` | ERROR | `Source X is past error threshold` | Latest record is older than the `error_after` config |
| `dbt source freshness` | ERROR | `Database Error` | Query against the source table failed (missing table, bad permissions) |
| `dbt source freshness` | ERROR | `Column X not found` | The `loaded_at_field` column doesn't exist in the source table |
| `dbt source freshness` | WARN | `Source X is past warn threshold` | Latest record is older than `warn_after` but within `error_after` |
| `dbt source freshness` | WARN | `No freshness config for source X` | Source is defined but has no `freshness` block — check is skipped |
| `dbt deps` | ERROR | `Version conflict` | Two packages require incompatible versions of the same dependency |
| `dbt deps` | ERROR | `Could not find package X` | Package doesn't exist on dbt Hub or the given git repo |
| `dbt deps` | ERROR | `Authentication error` | Private git repo requires credentials that aren't configured |
| `dbt deps` | ERROR | `Invalid packages.yml` | Malformed YAML in `packages.yml` |
| `dbt deps` | WARN | `Package X is deprecated` | The package author has flagged it as deprecated |
| `dbt deps` | WARN | `Unpinned package` | A package is listed without a version — reproducibility risk |
| `dbt deps` | WARN | `Newer version available` | A pinned package has a newer release available |
| `dbt seed` | ERROR | `Database Error` | Failed to create or insert into the seed table |
| `dbt seed` | ERROR | `Column type mismatch` | CSV data doesn't match the explicitly declared `column_types` config |
| `dbt seed` | ERROR | `File not found` | A seed file is referenced but missing from the `seeds/` directory |
| `dbt seed` | ERROR | `Duplicate column name` | CSV has two columns with the same header |
| `dbt seed` | WARN | `Seed X is too large` | File exceeds recommended size — seeds aren't meant for large datasets |
| `dbt seed` | WARN | `Seed quoting differs from project config` | Column quoting inconsistency between CSV headers and project settings |
| All commands | ERROR | `Profile not found` | `profiles.yml` is missing or profile name doesn't match `dbt_project.yml` |
| All commands | ERROR | `Target schema does not exist` | The target schema hasn't been created in the warehouse |
| All commands | ERROR | `Cycle detected in DAG` | Two models reference each other, creating a circular dependency |
| All commands | ERROR | `dbt version mismatch` | Project's `require-dbt-version` doesn't match the installed dbt version |
| All commands | ERROR | `Dispatch could not find macro X` | An `adapter.dispatch()` call found no matching macro for the current adapter |
| All commands | WARN | `Environment variable X is not set` | An `env_var()` call has no default and the variable is missing |
| All commands | WARN | `Partial parse warning` | Cached parsing state is stale — dbt falls back to full re-parse |

- Python model in dbt has the capability to incorporate additional functions either through importing external functions or by defining its own. This allows for the creation of non-dbt functions within the same Python model file for use in the model. However, it's currently not possible to import and reuse Python functions defined in one dbt model in other models
- 
