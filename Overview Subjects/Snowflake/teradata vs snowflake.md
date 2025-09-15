## Several axes to look into in terms of migration from teradata to snowflake !


### Security :

| **Category**              | **Teradata**                                                                                                                                                          | **Snowflake**                                                                                                                                                                                                                  |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Authentication**        | Supports password-based login, LDAP, Kerberos, Active Directory.                                                                                                      | Supports username/password, **MFA**, SSO (SAML 2.0), OAuth, Key Pair Authentication, SCIM; strong cloud-native integration with Okta, Azure AD, Ping, etc.                                                                     |
| **RBAC Model**            | Roles exist but are relatively **flat**. Privileges can be granted **directly to users** or to roles (common cause of privilege sprawl).                              | Strict **role-based model**. Privileges always granted to **roles**, never directly to users. Supports **role hierarchy** (roles can inherit other roles). Built-in admin roles (`SYSADMIN`, `SECURITYADMIN`, `ACCOUNTADMIN`). |
| **Object-Level Security** | Privileges can be granted at database, schema, table, column level. Row/column restrictions often implemented with **views or UDFs** (no native row access policies). | Fine-grained: **row access policies** for dynamic filtering, **dynamic data masking** at column-level, **secure views** and **secure UDFs** to protect logic.                                                                  |
| **Data Sharing**          | Sharing is limited to within the same Teradata system. For external sharing, data must be **exported/ETLed** to another system.                                       | Native **secure data sharing** across accounts and even clouds — no need to move or copy data. Can create **reader accounts** for external parties.                                                                            |
| **Auditing & Monitoring** | Relies on **DBQL (Database Query Log)** and **ResUsage** for query and resource usage logs. Security auditing is manual or via external tools.                        | Centralized with **Account Usage** views and **Information Schema**. Tracks **access history** (who queried what). Integrates natively with **SIEM/SOC tools** (Splunk, Datadog, etc.).                                        |
| **Encryption**            | Teradata provides encryption at rest and in transit, but requires **manual setup/config** depending on deployment.                                                    | Always-on **end-to-end encryption** (in transit & at rest), with cloud KMS integration (AWS KMS, Azure Key Vault, GCP KMS). Customer-managed keys possible.                                                                    |
| **Row-Level Security**    | Not natively supported; usually enforced via **views** with WHERE clauses.                                                                                            | **Row Access Policies**: dynamic, policy-driven row-level filtering natively supported.                                                                                                                                        |
| **Column-Level Security** | Implemented via views or UDF-based masking.                                                                                                                           | **Dynamic Data Masking** built-in, supports conditional/unconditional masking, and column-level security policies.                                                                                                             |
| **Separation of Duties**  | Admins often have overlapping privileges; difficult to enforce strict SoD without custom roles.                                                                       | Clear role hierarchy allows strong **separation of duties** between security, account, and system admins.                                                                                                                      |

### Tables & Storage Structures :

| **Feature**                         | **Teradata**                                                            | **Snowflake**                                                                       | **Migration Notes / Recommendations**                                                           |
| ----------------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Primary Index (PI)**              | Determines how rows are distributed across AMPs (performance-critical). | No indexes. Automatic distribution. Optional **clustering keys** for performance. | Analyze current PI usage; define clustering keys on high-volume query columns if needed.        |
| **Secondary Index (SI)**            | Additional access path to speed up queries.                             | Not supported. Use **clustering keys** or **materialized views**.                 | Queries depending on SI may need rewritten or optimized with clustering/materialized views.     |
| **Partitioned Primary Index (PPI)** | Partitions table rows for efficient filtering.                          | Clustering keys or partition pruning.                                             | Map PPIs to clustering keys on commonly filtered columns.                                       |
| **Multiset Table**                  | Allows duplicate rows.                                                  | All Snowflake tables allow duplicates by default.                                 | Usually no migration effort; duplicates are handled natively.                                   |
| **Set Table**                       | Enforces row uniqueness.                                                | PRIMARY KEY / UNIQUE constraints (informational only).                            | Snowflake doesn’t enforce physically — consider adding checks in ETL if uniqueness is critical. |
| **Volatile Table**                  | Session-level temp table, dropped at session end.                       | CREATE TEMPORARY TABLE.                                                           | Straightforward migration; ensure scripts create temp tables with `TEMPORARY` keyword.          |
| **Global Temporary Table**          | Defined permanently but data is session-local.                          | CREATE TEMPORARY or CREATE TRANSIENT table.                                       | Map GTTs to temporary or transient tables depending on lifecycle needs.                         |
| **Fallback Tables**                 | Automatic redundancy (duplicate storage) for resilience.                | Not needed; Snowflake automatically replicates & manages resiliency.              | No action required; Snowflake handles durability and failover automatically.                    |


### Data Loading & Utilities :

| **Feature / Utility**          | **Teradata**                                             | **Snowflake**                                                         | **Migration Notes / Recommendations**                                                                          |
| ------------------------------ | -------------------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **BTEQ**                       | Scripting for batch SQL, control logic.                  | SQL Scripts, **Tasks**, **Stored Procedures**, **Snowpark Python**. | Replace batch scripts with Snowflake tasks or Snowpark procedures; review procedural logic for compatibility.  |
| **FastLoad**                   | High-speed initial load into empty table.                | `COPY INTO` from stage (cloud storage).                             | Map FastLoad scripts to COPY INTO; ensure staged files are accessible (S3, Azure, GCS).                        |
| **MultiLoad (MLoad)**          | Batch load with updates, inserts, deletes.               | `MERGE` + `COPY INTO`.                                              | Rewrite multi-step load scripts into **staged loads + MERGE statements**.                                      |
| **TPump**                      | Continuous trickle load for near-real-time data.         | **Snowpipe** for auto-ingest from cloud storage.                    | Replace TPump with Snowpipe; adjust event triggers for file arrivals.                                          |
| **TPT (Parallel Transporter)** | Unified ETL framework supporting batch & parallel loads. | Snowflake connectors (Spark, Kafka, Snowpark API).                  | TPT pipelines can be replaced with Snowpark pipelines or external connectors; review parallelism requirements. |

- Batch-oriented loads (FastLoad, MultiLoad, BTEQ) map to COPY INTO + MERGE in Snowflake.

- Continuous / near-real-time loads (TPump) are best implemented with Snowpipe.

- Complex ETL scripts / orchestration (TPT) can leverage Snowpark Python / Spark or Snowflake Tasks for automation.

- The complexity of workloads will define the warehouse sizing !


### Procedural Logic : 

| **Feature**                 | **Teradata**                                               | **Snowflake**                                                        | **Migration Notes / Recommendations**                                                                                             |
| --------------------------- | ---------------------------------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Macros**                  | Predefined SQL statements executed like a function.        | Replace with **Views** or **Stored Procedures**.                   | Simple SQL macros can often be rewritten as **Views**; more complex macros → **Stored Procedures**.                               |
| **Stored Procedures (SPL)** | Procedural logic: loops, cursors, IF/ELSE, error handling. | SQL Stored Procedures, or better with **Snowpark (Python/Scala)**. | Review iterative or row-by-row logic; Snowpark is recommended for complex loops, branching, or algorithmic logic.                 |
| **Triggers**                | Actions fired on DML events.                               | Not supported. Use **Tasks + Streams** to simulate.                | Convert triggers to **stream-based tasks** or schedule via Snowflake Tasks; ensure same behavior for insert/update/delete events. |

- Macros → simple, declarative SQL; prefer Views if only querying.

- Stored Procedures (SPL) → consider Snowpark Python/Scala for row-by-row or complex procedural logic.

- Triggers → Snowflake has no triggers; replace with Tasks + Streams, or incorporate logic into ETL pipelines.

- Iterative logic in Teradata often requires redesign ! Snowflake encourages set-based operations whenever possible.
  
### SQL Features & Functions :

| **Feature**                                     | **Teradata**                                     | **Snowflake**                                                           | **Migration Notes / Recommendations**                                                                |
| ----------------------------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **QUALIFY**                                     | Filters rows after window functions.             | Supported natively in Snowflake.                                      | Queries using `QUALIFY` can be migrated directly without change.                                     |
| **TOP n**                                       | Returns first n rows.                            | Use `LIMIT n`.                                                        | Rewrite `TOP` clauses to `LIMIT`.                                                                    |
| **SAMPLE**                                      | Return a random sample of rows.                  | `TABLESAMPLE` supported.                                              | Use `TABLESAMPLE` syntax; adjust sampling method if needed.                                          |
| **HASH Functions**                              | Used for distribution & joins.                   | Standard `HASH()` functions supported; not required for distribution. | Remove hash-based distribution logic; Snowflake handles data partitioning automatically.             |
| **OLAP / Window Functions**                     | RANK, ROW\_NUMBER, SUM OVER, etc.                | Fully supported.                                                      | Most window function queries migrate directly; check performance on very large datasets.             |
| **Recursive Queries**                           | Recursive CTEs for hierarchical queries.         | Supported via `WITH RECURSIVE`.                                       | Ensure Teradata syntax is adapted to standard ANSI SQL recursive syntax.                             |
| **TOP PERCENT / SAMPLE PERCENT**                | Return a percentage of rows.                     | Use `TABLESAMPLE BERNOULLI (n)` or `SYSTEM (n)`.                      | Adjust syntax; behavior may slightly differ, verify sampling results.                                |
| **QUALIFY with Aggregates**                     | Filter after aggregation & windowing.            | Fully supported.                                                      | Queries using combined `QUALIFY` and `OVER()` functions usually migrate directly.                    |
| **Derived Tables / Volatile Tables in Queries** | Inline views or temp tables.                     | Subqueries / CTEs.                                                    | Replace Teradata volatile tables with **CTEs** (`WITH`) or Snowflake temporary tables if persistent. |
| **Date & Interval Functions**                   | ADD\_MONTHS, CURRENT\_DATE, INTERVAL arithmetic. | Snowflake equivalents: `DATEADD`, `CURRENT_DATE`, `INTERVAL`.         | Rewrite function calls with Snowflake-compatible syntax.                                             |
| **String Functions**                            | SUBSTRING, TRIM, INDEX, OTRANSLATE, etc.         | Snowflake supports standard string functions, some minor differences. | Review and adjust uncommon string functions.                                                         |
| **Conditional Expressions**                     | CASE, DECODE, COALESCE.                          | Fully supported.                                                      | Direct mapping, minor syntax changes if using DECODE → CASE recommended.                             |

- Use CTEs to replace volatile tables where needed, keeping queries set-based.
  
### Data Types :

| **Feature / Data Type**             | **Teradata**                                     | **Snowflake**                                              | **Migration Notes / Recommendations**                                    |
| ----------------------------------- | ------------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------------------ |
| **BYTEINT**                         | 1-byte integer (-128 to 127)                     | `NUMBER(3,0)` or `SMALLINT`                              | Direct mapping; numeric range is compatible.                             |
| **SMALLINT**                        | 2-byte integer (-32,768 to 32,767)               | `SMALLINT`                                               | Direct mapping.                                                          |
| **INTEGER / INT**                   | 4-byte integer (-2,147,483,648 to 2,147,483,647) | `INTEGER`                                                | Direct mapping.                                                          |
| **BIGINT**                          | 8-byte integer                                   | `BIGINT`                                                 | Direct mapping.                                                          |
| **DECIMAL(n,m) / NUMERIC(n,m)**     | Fixed precision numeric                          | `NUMBER(n,m)`                                            | Direct mapping; check precision/scale.                                   |
| **FLOAT / REAL / DOUBLE PRECISION** | Approximate numeric types                        | `FLOAT`                                                  | Direct mapping; slight differences in precision representation possible. |
| **CHAR(n)**                         | Fixed-length string                              | `CHAR(n)`                                                | Direct mapping. Pad with spaces in Snowflake.                            |
| **VARCHAR(n)**                      | Variable-length string                           | `VARCHAR(n)`                                             | Direct mapping; Snowflake max length 16 MB.                              |
| **CLOB**                            | Character large object                           | `STRING`                                                 | Direct mapping. Handles large text.                                      |
| **BLOB**                            | Binary large object                              | `BINARY`                                                 | Direct mapping.                                                          |
| **VARIANT**                         | Not in Teradata                                  | Supports semi-structured data (JSON, XML, AVRO, Parquet) | Ideal for JSON/XML migration from Teradata.                              |
| **INTERVAL**                        | Time intervals (DAY TO SECOND, YEAR TO MONTH)    | `INTERVAL`                                               | Supported with standard Snowflake syntax.                                |
| **DATE**                            | Calendar date                                    | `DATE`                                                   | Direct mapping.                                                          |
| **TIMESTAMP**                       | Date + time                                      | `TIMESTAMP_NTZ` (no timezone)                            | Choose appropriate type based on TZ requirement.                         |
| **TIMESTAMP WITH TIME ZONE**        | Date + time with timezone                        | `TIMESTAMP_TZ`                                           | Direct mapping; ensures timezone awareness.                              |
| **BOOLEAN**                         | Not native in Teradata                           | `BOOLEAN`                                                | Replace integer flags (0/1) with Boolean if needed.                      |

- Consider data precision (DECIMAL/NUMBER) for financial applications.
  
### Workload & Query Mgmt :

| **Feature**                                  | **Teradata**                                                                           | **Snowflake**                                                                                                    | **Migration Notes / Recommendations**                                                                     |
| -------------------------------------------- | -------------------------------------------------------------------------------------- | -----------------------------------------------------------------------------------------------------------------| --------------------------------------------------------------------------------------------------------- |
| **TASM (Teradata Active System Management)** | Controls resources by workload type, sets priorities, manages concurrency.             | **Virtual Warehouses** (compute clusters) with auto-scaling & workload isolation.                                | Map workloads to separate warehouses or multi-cluster warehouses for concurrency & priority management.   |
| **AMPs (Access Module Processors)**          | Teradata’s parallel compute units responsible for data distribution & query execution. | Not applicable — Snowflake auto-distributes compute across nodes.                                                | No need to manage AMPs; ensure queries leverage Snowflake’s **micro-partitioning** for performance.       |
| **Collect Stats**                            | Manual statistics for the optimizer; used to improve query plans.                      | Not needed — Snowflake uses **automatic query optimization** and adaptive caching.                               | Remove manual stats collection; rely on Snowflake’s automatic optimization and clustering keys if needed. |
| **Query Prioritization / Throttling**        | TASM rules set query priorities for different workloads.                               | Virtual warehouses and **resource monitors** control compute usage & limit resource consumption.                 | Implement resource monitors to prevent runaway queries; adjust warehouse sizes per workload.              |
| **Concurrency / Multi-User Control**         | TASM handles concurrent query scheduling.                                              | Multi-cluster warehouses allow **high concurrency**; Snowflake queues queries if warehouse capacity is exceeded. | Design warehouses for expected concurrency; consider multi-cluster auto-scale for spikes.                 |
| **Workload Isolation**                       | Separate rules in TASM for ETL vs analytics vs ad-hoc queries.                         | Separate warehouses or separate roles + warehouses provide isolation.                                            | Assign workloads to different warehouses to prevent interference.                                         |

- Use multi-cluster warehouses and resource monitors to replicate TASM-like workload control.

### Advanced / Analytical :

| **Feature**                                | **Teradata**                                                | **Snowflake**                                                                                              | **Migration Notes / Recommendations**                                                             |
| ------------------------------------------ | ----------------------------------------------------------- | -----------------------------------------------------------------------------------------------------------| ------------------------------------------------------------------------------------------------- |
| **Teradata ML Engine**                     | Built-in advanced analytics and machine learning functions. | **Snowpark ML (Python / PySpark)**; integrates with scikit-learn, TensorFlow, XGBoost, etc.                | Rewrite ML workflows in Snowpark; leverage Snowflake for data storage and processing.             |
| **Geospatial Types**                       | Spatial data types and functions (ST\_Geometry, etc.)       | `GEOGRAPHY` type and spatial functions in Snowflake.                                                       | Convert Teradata spatial columns to GEOGRAPHY; rewrite spatial queries with Snowflake functions.  |
| **JSON / XML Storage**                     | Shredded into relational form or stored as CLOBs.           | `VARIANT` type for semi-structured data; supports JSON, XML, Avro, Parquet, ORC.                           | Replace shredded relational model with VARIANT columns; leverage native semi-structured querying. |
| **Advanced Analytics / Custom Algorithms** | Procedural or UDF-based analytics in Teradata.              | Snowpark (Python/Scala/Java) UDFs and DataFrame operations.                                                | Rewrite procedural analytics in Snowpark for set-based, parallel execution.                       |
| **Time Series / Forecasting**              | Limited built-in functions or custom SPL procedures.        | Snowpark ML and external Python libraries (Prophet, scikit-learn).                                         | Move forecasting logic to Snowpark; integrate with Snowflake storage.                             |
| **Data Visualization / BI Integration**    | External tools only; no native engine.                      | Fully integrates with BI tools (Tableau, Power BI, Looker) and supports Snowflake SQL or Snowpark queries. | Update dashboards to query Snowflake directly; leverage semi-structured support.                  |

### dbt vs teradata :

| Teradata Concept                         | dbt / Snowflake Equivalent        | Notes / Considerations                                                                                 |
| ---------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Macro                                    | dbt Macro (Jinja + SQL)           | Can replicate parameterized SQL logic; dbt macros are more modular and reusable across models.         |
| Transformation (via macro/procedure)     | dbt Model (SQL select statements) | Set-based logic translates directly; loops/iterative logic may need **Snowpark** or set-based rewrite. |
| Multi-step ETL logic                     | dbt Models + DAG                  | Each macro/step can become a separate dbt model; dependency handled automatically.                     |
| Conditional/parameterized operations     | dbt Macro + config variables      | Use dbt variables or Jinja templates to replace Teradata macro parameters.                             |
| Procedural logic / row-by-row processing | Snowpark (Python/Scala)           | Iterative logic cannot run efficiently as-is; rewrite in Snowpark for performance.                     |

- Teradata often relies on row-by-row loops inside macros or procedures.

- Snowflake + dbt favors set-based SQL — process all rows in one statement.

- Iterative macros may need to be rewritten using dbt + Snowflake SQL or Snowpark for performance.

### Orchestration : 

- Teradata does have internal scheduling/orchestration capabilities:

  - Teradata Parallel Transporter (TPT): Can orchestrate complex ETL workflows internally.

  - BTEQ scripts + batch scheduling: Many shops rely on cron or enterprise schedulers (like Control-M) to run scripts/macros/procedures in sequence.

- Limitations:

  - Orchestration is mostly script-driven.

  - Scheduling, dependencies, retries, and notifications often require external schedulers.

  - Complex DAGs are hard to maintain internally.

Consider dbt cloud, dbt core with an orchestrator like dasgter or airflow !

### Snowflake tasks vs external orchestrators : 

| Capability                        | Snowflake Tasks               | External Orchestrator                    |
| --------------------------------- | ------------------------------| -----------------------------------------|
| Conditional execution / branching | Not natively supported        | Yes, full dynamic DAGs                   |
| Cross-system orchestration        | Only Snowflake SQL/procedures | Yes, e.g., Snowflake + Kafka + S3 + APIs |
| Retry/backoff policies            | Minimal                       | Full support with logging & alerts       |
| Monitoring dashboards             | Limited                       | Centralized UI & alerting                |
| Parameterization across tasks     | Limited                       | Supported with templating & variables    |


### In general : 

- Set-based SQL → Rewrite in Snowflake SQL.

- Procedural / Iterative Logic (SPL, macros) → Rewrite in Snowpark (Python/Scala/Java).

- Heavy ETL Scripts (BTEQ, FastLoad, TPT) → Replace with Snowflake COPY, MERGE, Tasks, Snowpipe, or Snowpark orchestration.

- Advanced Analytics / ML → Move to Snowpark ML (Python/PySpark).
