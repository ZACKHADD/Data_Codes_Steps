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
- 
