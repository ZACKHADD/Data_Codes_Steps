## Main snowflake axes to cover :  

## 1. Core Snowflake Architecture & Concepts

### Multi-Cluster Shared Data Architecture
Understanding Snowflake's unique architecture is fundamental for making optimal design decisions.

**Key Concepts:**
- **Storage Layer**: Centralized, immutable storage with automatic compression and encryption
- **Compute Layer**: Virtual warehouses that can scale independently 
- **Cloud Services Layer**: Metadata management, query optimization, security

```sql
-- Understanding warehouse sizing and scaling
CREATE WAREHOUSE analytics_wh WITH
  WAREHOUSE_SIZE = 'LARGE'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 10
  SCALING_POLICY = 'STANDARD';

-- Monitor warehouse performance
SELECT 
  warehouse_name,
  avg_running,
  avg_queued_load,
  avg_queued_provisioning,
  avg_blocked
FROM snowflake.account_usage.warehouse_load_history
WHERE start_time >= current_date - 7;
```

### Time Travel and Fail-safe
**Concept**: Time Travel allows querying historical versions of data within the retention period (1-90 days). Fail-safe provides additional 7 days of data recovery by Snowflake support. Understanding the difference between these features is crucial for data recovery strategies and compliance requirements.

**Time Travel Methods**:
1. **AT**: Query data at a specific point in time
2. **BEFORE**: Query data before a specific statement/transaction
3. **OFFSET**: Query data relative to current time

```sql
-- Set retention period (up to 90 days for Enterprise)
ALTER TABLE sales_data SET DATA_RETENTION_TIME_IN_DAYS = 30;

-- Method 1: AT - Query at specific timestamp
SELECT * FROM sales_data AT(TIMESTAMP => '2024-01-01 10:00:00'::timestamp);

-- Method 2: BEFORE - Query before a specific statement
-- Get query ID first
SELECT query_id FROM snowflake.account_usage.query_history 
WHERE query_text ILIKE '%DELETE FROM sales_data%' 
ORDER BY start_time DESC LIMIT 1;

-- Query data before the DELETE statement
SELECT * FROM sales_data BEFORE(STATEMENT => '01a2b3c4-5678-90de-fg12-345678901234');

-- Method 3: OFFSET - Query data from X seconds ago
SELECT * FROM sales_data AT(OFFSET => -3600); -- 1 hour ago

-- Advanced time travel with CHANGES clause
-- Track all changes between two points in time
SELECT * FROM sales_data 
CHANGES(INFORMATION => DEFAULT) 
AT(TIMESTAMP => '2024-01-01'::timestamp)
END(TIMESTAMP => '2024-01-02'::timestamp);

-- Restore dropped table
UNDROP TABLE sales_data;

-- Clone zero-copy for development (preserves time travel)
CREATE TABLE sales_data_dev CLONE sales_data AT(TIMESTAMP => '2024-01-01 10:00:00'::timestamp);

-- Show retention policy for all tables
SHOW TABLES LIKE '%sales%';
SELECT "name", "retention_time" FROM table(result_scan(last_query_id()));
```

## 2. Advanced SQL and Performance Optimization

### Window Functions and Analytics
Essential for complex analytical queries and reporting.

```sql
-- Advanced window functions for business metrics
WITH customer_metrics AS (
  SELECT 
    customer_id,
    order_date,
    order_amount,
    -- Running total
    SUM(order_amount) OVER (
      PARTITION BY customer_id 
      ORDER BY order_date 
      ROWS UNBOUNDED PRECEDING
    ) AS running_total,
    -- Customer lifetime value ranking
    DENSE_RANK() OVER (
      ORDER BY SUM(order_amount) OVER (PARTITION BY customer_id) DESC
    ) AS ltv_rank,
    -- Period-over-period analysis
    LAG(order_amount, 1) OVER (
      PARTITION BY customer_id 
      ORDER BY order_date
    ) AS prev_order_amount
  FROM orders
)
SELECT 
  customer_id,
  running_total,
  ltv_rank,
  CASE 
    WHEN prev_order_amount IS NULL THEN 'First Order'
    WHEN order_amount > prev_order_amount THEN 'Increased'
    ELSE 'Decreased'
  END AS order_trend
FROM customer_metrics;
```

### Query Performance Optimization

```sql
-- Clustering for large tables
ALTER TABLE large_fact_table CLUSTER BY (date_column, region_id);

-- Analyze clustering effectiveness
SELECT 
  table_name,
  clustering_key,
  total_partition_count,
  average_overlaps,
  average_depth
FROM snowflake.information_schema.automatic_clustering_history
WHERE table_name = 'LARGE_FACT_TABLE';

-- Use result caching effectively
-- Enable query result caching
ALTER SESSION SET USE_CACHED_RESULT = TRUE;

-- Materialized views for frequently accessed aggregations
CREATE MATERIALIZED VIEW daily_sales_summary AS
SELECT 
  DATE_TRUNC('day', order_timestamp) AS order_date,
  product_category,
  SUM(amount) AS total_sales,
  COUNT(*) AS order_count
FROM orders
GROUP BY order_date, product_category;
```

## 3. Data Modeling and Architecture Design

### Dimensional Modeling in Snowflake

```sql
-- Star schema implementation
CREATE SCHEMA analytics;

-- Dimension tables with SCD Type 2
CREATE OR REPLACE TABLE analytics.dim_customer (
  customer_sk NUMBER AUTOINCREMENT,
  customer_id STRING,
  customer_name STRING,
  email STRING,
  segment STRING,
  effective_date TIMESTAMP,
  expiration_date TIMESTAMP,
  is_current BOOLEAN DEFAULT TRUE,
  PRIMARY KEY (customer_sk)
);

-- Fact table with foreign keys
CREATE OR REPLACE TABLE analytics.fact_sales (
  sale_sk NUMBER AUTOINCREMENT,
  customer_sk NUMBER,
  product_sk NUMBER,
  date_sk NUMBER,
  sale_amount NUMBER(10,2),
  quantity NUMBER,
  discount_amount NUMBER(10,2),
  FOREIGN KEY (customer_sk) REFERENCES analytics.dim_customer(customer_sk)
);
```

### Data Vault 2.0 Pattern

```sql
-- Hub table
CREATE TABLE hub_customer (
  customer_hk STRING PRIMARY KEY,
  customer_id STRING,
  load_date TIMESTAMP,
  record_source STRING
);

-- Satellite table
CREATE TABLE sat_customer_details (
  customer_hk STRING,
  load_date TIMESTAMP,
  customer_name STRING,
  email STRING,
  phone STRING,
  hash_diff STRING,
  PRIMARY KEY (customer_hk, load_date),
  FOREIGN KEY (customer_hk) REFERENCES hub_customer(customer_hk)
);
```

## 4. ETL/ELT Pipeline Development

### Snowpipe for Real-time Ingestion

```sql
-- Create external stage
CREATE STAGE my_s3_stage
  URL = 's3://mybucket/data/'
  CREDENTIALS = (AWS_KEY_ID = 'xxx' AWS_SECRET_KEY = 'yyy')
  FILE_FORMAT = (TYPE = 'JSON');

-- Create Snowpipe
CREATE PIPE customer_pipe
  AUTO_INGEST = TRUE
  AS 
  COPY INTO raw_customers
  FROM @my_s3_stage/customers/
  FILE_FORMAT = (TYPE = 'JSON');

-- Monitor pipe status
SELECT 
  pipe_name,
  avg_file_load_time,
  avg_insert_time,
  last_load_time
FROM snowflake.information_schema.pipe_usage_history
WHERE pipe_name = 'CUSTOMER_PIPE';
```

### Stored Procedures for Complex ETL Logic

```sql
CREATE OR REPLACE PROCEDURE process_daily_sales()
RETURNS STRING
LANGUAGE SQL
AS
$$
DECLARE
  result_msg STRING;
  error_msg STRING;
BEGIN
  -- Start transaction
  BEGIN TRANSACTION;
  
  -- Create staging table
  CREATE OR REPLACE TEMPORARY TABLE stage_daily_sales AS
  SELECT 
    DATE_TRUNC('day', order_timestamp) AS sale_date,
    customer_id,
    SUM(amount) AS daily_total,
    COUNT(*) AS transaction_count
  FROM raw_orders
  WHERE order_timestamp >= CURRENT_DATE - 1
    AND order_timestamp < CURRENT_DATE
  GROUP BY sale_date, customer_id;
  
  -- Merge into target table
  MERGE INTO analytics.daily_customer_sales t
  USING stage_daily_sales s
  ON t.sale_date = s.sale_date AND t.customer_id = s.customer_id
  WHEN MATCHED THEN
    UPDATE SET 
      daily_total = s.daily_total,
      transaction_count = s.transaction_count,
      updated_at = CURRENT_TIMESTAMP
  WHEN NOT MATCHED THEN
    INSERT (sale_date, customer_id, daily_total, transaction_count, created_at)
    VALUES (s.sale_date, s.customer_id, s.daily_total, s.transaction_count, CURRENT_TIMESTAMP);
  
  -- Log success
  result_msg := 'Daily sales processing completed successfully';
  COMMIT;
  
  RETURN result_msg;
  
EXCEPTION
  WHEN OTHER THEN
    ROLLBACK;
    error_msg := 'Error: ' || SQLERRM;
    RETURN error_msg;
END;
$$;

-- Schedule with tasks
CREATE TASK daily_sales_task
  WAREHOUSE = 'ETL_WH'
  SCHEDULE = 'USING CRON 0 2 * * * UTC'
AS
  CALL process_daily_sales();
```

## 5. Security and Governance

### Role-Based Access Control (RBAC)

```sql
-- Create role hierarchy
CREATE ROLE data_engineer;
CREATE ROLE senior_data_engineer;
CREATE ROLE data_lead;

-- Grant role inheritance
GRANT ROLE data_engineer TO ROLE senior_data_engineer;
GRANT ROLE senior_data_engineer TO ROLE data_lead;

-- Database and schema level permissions
GRANT USAGE ON DATABASE analytics TO ROLE data_engineer;
GRANT USAGE ON SCHEMA analytics.staging TO ROLE data_engineer;
GRANT SELECT ON ALL TABLES IN SCHEMA analytics.staging TO ROLE data_engineer;

-- Future grants for new objects
GRANT SELECT ON FUTURE TABLES IN SCHEMA analytics.staging TO ROLE data_engineer;

-- Row-level security
CREATE ROW ACCESS POLICY customer_region_policy AS (region_column STRING) 
RETURNS BOOLEAN ->
  CASE 
    WHEN CURRENT_ROLE() = 'ADMIN' THEN TRUE
    WHEN CURRENT_ROLE() = 'REGIONAL_ANALYST' AND region_column = CURRENT_REGION() THEN TRUE
    ELSE FALSE
  END;

ALTER TABLE customers ADD ROW ACCESS POLICY customer_region_policy ON (region);
```

### Data Masking and Privacy

**Concept**: Data masking protects sensitive information by transforming it based on user roles and context. Snowflake supports dynamic data masking at the column level, conditional masking, and external tokenization. This is essential for compliance with GDPR, HIPAA, and other privacy regulations.

```sql
-- Dynamic data masking with role-based access
CREATE MASKING POLICY email_mask AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('ADMIN', 'DATA_ANALYST') THEN val
    WHEN CURRENT_ROLE() IN ('MARKETING_ANALYST') THEN 
      REGEXP_REPLACE(val, '(.*)@(.*)', 'masked@\\2')
    ELSE '***MASKED***'
  END;

-- Apply masking policy to column
ALTER TABLE customers MODIFY COLUMN email SET MASKING POLICY email_mask;

-- Advanced conditional masking with multiple conditions
CREATE MASKING POLICY sensitive_data_mask AS (val STRING, user_region STRING, data_classification STRING) 
RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN val
    WHEN CURRENT_ROLE() = 'REGIONAL_MANAGER' AND user_region = CURRENT_USER_REGION() THEN val
    WHEN data_classification = 'PUBLIC' THEN val
    WHEN data_classification = 'INTERNAL' AND IS_ROLE_IN_SESSION('EMPLOYEE') THEN 
      LEFT(val, 2) || '***' || RIGHT(val, 2)
    ELSE '***CLASSIFIED***'
  END;

-- Apply multi-parameter masking policy
ALTER TABLE sensitive_data MODIFY COLUMN ssn 
SET MASKING POLICY sensitive_data_mask USING (ssn, region, classification);

-- Unstructured data masking (JSON, VARIANT)
CREATE MASKING POLICY json_pii_mask AS (val VARIANT) RETURNS VARIANT ->
  CASE
    WHEN CURRENT_ROLE() IN ('ADMIN', 'DATA_ENGINEER') THEN val
    ELSE OBJECT_CONSTRUCT(
      'user_id', val:user_id,
      'timestamp', val:timestamp,
      'email', '***MASKED***',
      'phone', '***MASKED***'
    )
  END;

-- External tokenization for highest security
CREATE MASKING POLICY external_token_mask AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN val
    ELSE 'TOKEN_' || SHA2(val || CURRENT_USER(), 256)
  END;

-- Column-level encryption (Enterprise feature)
ALTER TABLE highly_sensitive_data MODIFY COLUMN credit_card_number SET 
  MASKING POLICY external_token_mask FORCE;

-- View masking policies applied to tables
SELECT 
  table_name,
  column_name,
  masking_policy_name,
  policy_kind
FROM snowflake.account_usage.policy_references
WHERE ref_entity_domain = 'TABLE'
  AND ref_database_name = 'SENSITIVE_DB';
```

## 6. Network Policies and Security

### Network Policies
**Concept**: Network policies control access to Snowflake based on IP addresses, ensuring that users can only connect from approved networks. This is crucial for enterprise security, especially when combined with SSO and RBAC. Network policies can be applied at account, user, or role level.

```sql
-- Create network policy allowing specific IP ranges
CREATE NETWORK POLICY corporate_network_policy
  ALLOWED_IP_LIST = (
    '192.168.1.0/24',      -- Corporate network
    '10.0.0.0/8',          -- VPN range
    '203.0.113.0/24'       -- Branch office
  )
  BLOCKED_IP_LIST = (
    '192.168.1.100'        -- Specific blocked IP
  )
  COMMENT = 'Corporate network access policy';

-- Apply network policy to account (affects all users)
ALTER ACCOUNT SET NETWORK_POLICY = corporate_network_policy;

-- Apply network policy to specific user
ALTER USER john_doe SET NETWORK_POLICY = corporate_network_policy;

-- Apply network policy to role
ALTER ROLE data_analyst SET NETWORK_POLICY = corporate_network_policy;

-- Create stricter policy for admin access
CREATE NETWORK POLICY admin_network_policy
  ALLOWED_IP_LIST = ('192.168.1.10', '192.168.1.11')  -- Only admin workstations
  COMMENT = 'Restricted access for admin users';

-- Apply to admin users
ALTER ROLE accountadmin SET NETWORK_POLICY = admin_network_policy;

-- View network policy assignments
SHOW NETWORK POLICIES;

-- Monitor network policy violations
SELECT 
  user_name,
  client_ip,
  reported_client_type,
  error_message,
  first_authentication_factor,
  event_timestamp
FROM snowflake.account_usage.login_history
WHERE is_success = 'NO' 
  AND error_message ILIKE '%network policy%'
  AND event_timestamp >= DATEADD(day, -7, CURRENT_TIMESTAMP())
ORDER BY event_timestamp DESC;

-- Temporarily bypass network policy for emergency access
ALTER ACCOUNT SET NETWORK_POLICY = NULL;  -- Remove account-level policy
-- Remember to reapply after emergency: ALTER ACCOUNT SET NETWORK_POLICY = corporate_network_policy;
```

## 7. Tags and Data Classification

### Object Tagging
**Concept**: Tags in Snowflake are key-value pairs that can be applied to databases, schemas, tables, columns, warehouses, and other objects. They enable data governance, cost allocation, policy enforcement, and metadata management. Tags are essential for large-scale data platform management.

```sql
-- Create tag taxonomy for data classification
CREATE TAG data_classification
  ALLOWED_VALUES ('PUBLIC', 'INTERNAL', 'CONFIDENTIAL', 'RESTRICTED')
  COMMENT = 'Data sensitivity classification';

CREATE TAG data_subject
  ALLOWED_VALUES ('CUSTOMER', 'EMPLOYEE', 'FINANCIAL', 'OPERATIONAL')
  COMMENT = 'Type of data subject';

CREATE TAG cost_center
  COMMENT = 'Business unit for cost allocation';

CREATE TAG data_owner
  COMMENT = 'Data steward responsible for this data';

-- Apply tags at different object levels

-- Database level tagging
ALTER DATABASE customer_db SET TAG (
  data_classification = 'CONFIDENTIAL',
  cost_center = 'MARKETING',
  data_owner = 'john.doe@company.com'
);

-- Schema level tagging
ALTER SCHEMA customer_db.pii SET TAG (
  data_classification = 'RESTRICTED',
  data_subject = 'CUSTOMER'
);

-- Table level tagging
ALTER TABLE customers SET TAG (
  data_classification = 'CONFIDENTIAL',
  data_subject = 'CUSTOMER',
  data_owner = 'data-governance@company.com'
);

-- Column level tagging (most granular)
ALTER TABLE customers MODIFY COLUMN email SET TAG (
  data_classification = 'RESTRICTED',
  data_subject = 'CUSTOMER'
);

ALTER TABLE customers MODIFY COLUMN phone SET TAG (
  data_classification = 'RESTRICTED',
  data_subject = 'CUSTOMER'
);

-- Warehouse tagging for cost allocation
ALTER WAREHOUSE analytics_wh SET TAG (
  cost_center = 'ANALYTICS',
  data_owner = 'analytics-team@company.com'
);

-- Query objects by tag values
SELECT 
  object_name,
  object_type,
  tag_name,
  tag_value,
  domain
FROM snowflake.account_usage.tag_references
WHERE tag_name = 'DATA_CLASSIFICATION'
  AND tag_value = 'RESTRICTED';

-- Find all objects owned by specific team
SELECT 
  object_name,
  object_type,
  tag_value as owner
FROM snowflake.account_usage.tag_references
WHERE tag_name = 'DATA_OWNER'
  AND tag_value = 'data-governance@company.com';

-- Cost allocation by tags
SELECT 
  warehouse_name,
  tag_value as cost_center,
  SUM(credits_used) as total_credits,
  SUM(credits_used * 2.5) as estimated_cost_usd  -- Assuming $2.5 per credit
FROM snowflake.account_usage.warehouse_metering_history w
JOIN snowflake.account_usage.tag_references t
  ON w.warehouse_name = t.object_name
  AND t.object_type = 'WAREHOUSE'
  AND t.tag_name = 'COST_CENTER'
WHERE start_time >= DATEADD(month, -1, CURRENT_TIMESTAMP())
GROUP BY warehouse_name, cost_center
ORDER BY total_credits DESC;
```

### Tags with Data Masking Policies
**Concept**: Combining tags with masking policies creates a powerful, scalable approach to data protection. Instead of applying masking policies to individual columns, you can apply them to tags, and any column with that tag will automatically inherit the policy. This is essential for enterprise-scale data governance.

```sql
-- Create masking policies that work with tags
CREATE MASKING POLICY tag_based_pii_mask AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('ADMIN', 'DATA_PROTECTION_OFFICER') THEN val
    WHEN CURRENT_ROLE() IN ('ANALYST', 'DATA_SCIENTIST') THEN 
      REGEXP_REPLACE(val, '(.{2})(.*)(.{2})', '\\1***\\3')
    WHEN CURRENT_ROLE() IN ('REPORT_VIEWER') THEN 
      '***PROTECTED***'
    ELSE '***UNAUTHORIZED***'
  END;

-- Create more specific masking policies
CREATE MASKING POLICY financial_data_mask AS (val NUMBER) RETURNS NUMBER ->
  CASE
    WHEN CURRENT_ROLE() IN ('ADMIN', 'FINANCE_MANAGER') THEN val
    WHEN CURRENT_ROLE() IN ('FINANCE_ANALYST') THEN ROUND(val, -2)  -- Round to hundreds
    WHEN CURRENT_ROLE() IN ('BUSINESS_ANALYST') THEN 
      CASE WHEN val > 1000 THEN 1000 ELSE val END  -- Cap at 1000
    ELSE NULL
  END;

-- Apply masking policy to a tag (affects all columns with this tag)
ALTER TAG data_classification SET MASKING POLICY tag_based_pii_mask;

-- Create specialized tags for different data types
CREATE TAG pii_email
  COMMENT = 'Email addresses requiring protection';
  
CREATE TAG pii_phone
  COMMENT = 'Phone numbers requiring protection';
  
CREATE TAG financial_amount
  COMMENT = 'Financial amounts requiring protection';

-- Apply different masking policies to different tags
ALTER TAG pii_email SET MASKING POLICY email_specific_mask;
ALTER TAG pii_phone SET MASKING POLICY phone_specific_mask;
ALTER TAG financial_amount SET MASKING POLICY financial_data_mask;

-- Tag columns with appropriate classification
ALTER TABLE customers MODIFY COLUMN email SET TAG (pii_email = 'true');
ALTER TABLE customers MODIFY COLUMN phone SET TAG (pii_phone = 'true');
ALTER TABLE orders MODIFY COLUMN order_total SET TAG (financial_amount = 'true');
ALTER TABLE orders MODIFY COLUMN customer_lifetime_value SET TAG (financial_amount = 'true');

-- Create conditional masking based on multiple tag values
CREATE MASKING POLICY multi_tag_conditional_mask AS (
  val STRING, 
  sensitivity STRING, 
  data_type STRING
) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN val
    WHEN sensitivity = 'PUBLIC' THEN val
    WHEN sensitivity = 'INTERNAL' AND IS_ROLE_IN_SESSION('EMPLOYEE') THEN val
    WHEN sensitivity = 'CONFIDENTIAL' AND data_type = 'EMAIL' THEN
      REGEXP_REPLACE(val, '(.*)@(.*)', '***@\\2')
    WHEN sensitivity = 'CONFIDENTIAL' AND data_type = 'PHONE' THEN
      REGEXP_REPLACE(val, '(...)(.*)(....)', '\\1-***-\\3')
    WHEN sensitivity = 'RESTRICTED' THEN '***CLASSIFIED***'
    ELSE '***UNAUTHORIZED***'
  END;

-- Apply multi-parameter masking using tag values
ALTER TABLE customer_details MODIFY COLUMN personal_email 
SET MASKING POLICY multi_tag_conditional_mask 
USING (personal_email, GET_TAG('DATA_CLASSIFICATION'), GET_TAG('DATA_TYPE'));

-- Bulk tagging operations using procedures
CREATE OR REPLACE PROCEDURE bulk_tag_pii_columns()
RETURNS STRING
LANGUAGE SQL
AS
$
DECLARE
  table_cursor CURSOR FOR 
    SELECT table_schema, table_name, column_name
    FROM information_schema.columns
    WHERE column_name IN ('email', 'phone', 'ssn', 'credit_card')
      AND table_schema NOT IN ('INFORMATION_SCHEMA', 'SNOWFLAKE');
      
  sql_command STRING;
BEGIN
  FOR table_record IN table_cursor DO
    CASE table_record.column_name
      WHEN 'email' THEN
        sql_command := 'ALTER TABLE ' || table_record.table_schema || '.' || 
                      table_record.table_name || ' MODIFY COLUMN ' || 
                      table_record.column_name || ' SET TAG (pii_email = ''true'', data_classification = ''RESTRICTED'')';
      WHEN 'phone' THEN
        sql_command := 'ALTER TABLE ' || table_record.table_schema || '.' || 
                      table_record.table_name || ' MODIFY COLUMN ' || 
                      table_record.column_name || ' SET TAG (pii_phone = ''true'', data_classification = ''RESTRICTED'')';
      WHEN 'ssn' THEN
        sql_command := 'ALTER TABLE ' || table_record.table_schema || '.' || 
                      table_record.table_name || ' MODIFY COLUMN ' || 
                      table_record.column_name || ' SET TAG (data_classification = ''RESTRICTED'')';
      ELSE
        sql_command := 'ALTER TABLE ' || table_record.table_schema || '.' || 
                      table_record.table_name || ' MODIFY COLUMN ' || 
                      table_record.column_name || ' SET TAG (data_classification = ''CONFIDENTIAL'')';
    END CASE;
    
    EXECUTE IMMEDIATE sql_command;
  END FOR;
  
  RETURN 'Bulk tagging completed successfully';
EXCEPTION
  WHEN OTHER THEN
    RETURN 'Error in bulk tagging: ' || SQLERRM;
END;
$;

-- Monitor tag-based masking policy effectiveness
SELECT 
  tr.object_name AS table_name,
  tr.column_name,
  tr.tag_name,
  tr.tag_value,
  pr.policy_name,
  pr.policy_kind
FROM snowflake.account_usage.tag_references tr
LEFT JOIN snowflake.account_usage.policy_references pr
  ON tr.tag_name = pr.ref_entity_name
  AND pr.ref_entity_domain = 'TAG'
WHERE tr.domain = 'COLUMN'
  AND tr.tag_name IN ('DATA_CLASSIFICATION', 'PII_EMAIL', 'PII_PHONE', 'FINANCIAL_AMOUNT')
ORDER BY tr.object_name, tr.column_name;

-- Data governance reporting with tags
CREATE VIEW data_governance_dashboard AS
SELECT 
  t.object_database,
  t.object_schema,
  t.object_name,
  t.column_name,
  t.tag_name,
  t.tag_value,
  CASE WHEN p.policy_name IS NOT NULL THEN 'PROTECTED' ELSE 'UNPROTECTED' END AS protection_status,
  p.policy_name,
  CURRENT_TIMESTAMP() AS report_timestamp
FROM snowflake.account_usage.tag_references t
LEFT JOIN snowflake.account_usage.policy_references p
  ON (t.object_name = p.ref_entity_name AND t.column_name = p.ref_column_name)
  OR (t.tag_name = p.ref_entity_name AND p.ref_entity_domain = 'TAG')
WHERE t.domain = 'COLUMN'
  AND t.object_database NOT IN ('SNOWFLAKE', 'INFORMATION_SCHEMA')
ORDER BY t.object_database, t.object_schema, t.object_name, t.column_name;
```

### Query Performance Monitoring

```sql
-- Identify expensive queries
CREATE OR REPLACE VIEW query_performance_monitor AS
SELECT 
  query_id,
  query_text,
  database_name,
  schema_name,
  user_name,
  warehouse_name,
  warehouse_size,
  execution_status,
  total_elapsed_time/1000 AS execution_seconds,
  bytes_scanned,
  rows_produced,
  credits_used_cloud_services,
  start_time,
  end_time
FROM snowflake.account_usage.query_history
WHERE start_time >= DATEADD(hour, -24, CURRENT_TIMESTAMP())
  AND total_elapsed_time > 60000  -- Queries taking more than 60 seconds
ORDER BY total_elapsed_time DESC;

-- Warehouse utilization analysis
SELECT 
  warehouse_name,
  DATE_TRUNC('hour', start_time) AS hour,
  AVG(avg_running) AS avg_running_queries,
  AVG(avg_queued_load) AS avg_queue_depth,
  SUM(credits_used) AS total_credits
FROM snowflake.account_usage.warehouse_load_history
WHERE start_time >= DATEADD(day, -7, CURRENT_TIMESTAMP())
GROUP BY warehouse_name, hour
ORDER BY warehouse_name, hour;
```

### Data Quality Monitoring

```sql
-- Data freshness checks
CREATE OR REPLACE PROCEDURE check_data_freshness()
RETURNS TABLE (table_name STRING, hours_since_update NUMBER, status STRING)
LANGUAGE SQL
AS
$$
DECLARE
  c1 CURSOR FOR 
    SELECT table_name, 
           DATEDIFF(hour, MAX(created_at), CURRENT_TIMESTAMP()) AS hours_old
    FROM (
      SELECT 'orders' AS table_name, MAX(created_at) AS created_at FROM orders
      UNION ALL
      SELECT 'customers', MAX(updated_at) FROM customers
    ) t
    GROUP BY table_name;
BEGIN
  FOR record IN c1 DO
    IF (record.hours_old > 24) THEN
      INSERT INTO results VALUES (record.table_name, record.hours_old, 'STALE');
    ELSE
      INSERT INTO results VALUES (record.table_name, record.hours_old, 'FRESH');
    END IF;
  END FOR;
  RETURN TABLE(results);
END;
$$;
```

## 8. Monitoring and Observability

### External Data Sources

```sql
-- External tables for data lake integration
CREATE EXTERNAL TABLE external_logs (
  log_timestamp TIMESTAMP AS (value:timestamp::timestamp),
  user_id STRING AS (value:user_id::string),
  event_type STRING AS (value:event_type::string),
  properties VARIANT AS (value:properties)
)
LOCATION = @s3_stage/logs/
REFRESH_ON_CREATE = TRUE
AUTO_REFRESH = TRUE
FILE_FORMAT = (TYPE = 'JSON');
```

### Snowpark Integration

```python
# Python connector and Snowpark
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col, when_matched, when_not_matched

# Create session
session = Session.builder.configs({
    "account": "your_account",
    "user": "your_user", 
    "password": "your_password",
    "role": "DATA_ENGINEER",
    "warehouse": "COMPUTE_WH",
    "database": "ANALYTICS",
    "schema": "STAGING"
}).create()

# DataFrame operations
df = session.table("raw_sales")
transformed_df = df.select(
    col("sale_id"),
    col("customer_id"), 
    col("amount"),
    when(col("amount") > 1000, "High Value").otherwise("Standard").alias("sale_category")
).filter(col("sale_date") >= "2024-01-01")

# Write back to Snowflake
transformed_df.write.mode("overwrite").save_as_table("processed_sales")
```

## 9. Integration and Ecosystem

### Code Review Standards

```sql
-- Example of well-documented, maintainable SQL
/*
Purpose: Calculate customer lifetime value with proper error handling
Author: Data Engineering Team
Created: 2024-01-15
Dependencies: dim_customer, fact_orders
*/

CREATE OR REPLACE VIEW customer_lifetime_value AS
WITH customer_orders AS (
  SELECT 
    c.customer_id,
    c.customer_name,
    c.segment,
    o.order_date,
    o.order_amount,
    -- Data quality check
    CASE 
      WHEN o.order_amount IS NULL OR o.order_amount < 0 
      THEN 0 
      ELSE o.order_amount 
    END AS clean_order_amount
  FROM dim_customer c
  JOIN fact_orders o ON c.customer_sk = o.customer_sk
  WHERE c.is_current = TRUE
    AND o.order_date >= '2020-01-01'  -- Business rule: Only recent orders
),
customer_metrics AS (
  SELECT 
    customer_id,
    customer_name,
    segment,
    COUNT(DISTINCT order_date) AS total_orders,
    SUM(clean_order_amount) AS total_spent,
    AVG(clean_order_amount) AS avg_order_value,
    MAX(order_date) AS last_order_date,
    DATEDIFF(day, MIN(order_date), MAX(order_date)) AS customer_lifespan_days
  FROM customer_orders
  GROUP BY customer_id, customer_name, segment
)
SELECT 
  customer_id,
  customer_name,
  segment,
  total_orders,
  total_spent,
  avg_order_value,
  last_order_date,
  customer_lifespan_days,
  -- LTV calculation with business logic
  CASE 
    WHEN customer_lifespan_days = 0 THEN total_spent
    ELSE (total_spent / NULLIF(customer_lifespan_days, 0)) * 365 * 2  -- 2-year projection
  END AS projected_ltv,
  CURRENT_TIMESTAMP() AS calculated_at
FROM customer_metrics;
```

### Performance Testing Framework

```sql
-- Create performance baseline
CREATE OR REPLACE PROCEDURE benchmark_query_performance(query_name STRING, query_sql STRING)
RETURNS TABLE (query_name STRING, execution_time_ms NUMBER, rows_processed NUMBER, credits_used NUMBER)
LANGUAGE SQL
AS
$$
DECLARE
  start_time TIMESTAMP;
  end_time TIMESTAMP;
  execution_ms NUMBER;
  query_id STRING;
BEGIN
  -- Record start time
  start_time := CURRENT_TIMESTAMP();
  
  -- Execute the query
  EXECUTE IMMEDIATE query_sql;
  
  -- Get the last query ID
  query_id := LAST_QUERY_ID();
  
  -- Get performance metrics
  SELECT 
    total_elapsed_time,
    rows_produced,
    credits_used_cloud_services
  INTO execution_ms, rows_processed, credits_used
  FROM snowflake.account_usage.query_history
  WHERE query_id = query_id;
  
  -- Return results
  INSERT INTO results VALUES (query_name, execution_ms, rows_processed, credits_used);
  RETURN TABLE(results);
END;
$$;
```

## 10. Leadership and Team Management

### Warehouse Management

```sql
-- Automated warehouse scaling based on queue depth
CREATE OR REPLACE PROCEDURE optimize_warehouse_scaling()
RETURNS STRING
LANGUAGE SQL
AS
$$
DECLARE
  queue_depth NUMBER;
  current_size STRING;
BEGIN
  -- Check current queue depth
  SELECT AVG(avg_queued_load)
  INTO queue_depth
  FROM snowflake.account_usage.warehouse_load_history
  WHERE warehouse_name = 'ANALYTICS_WH'
    AND start_time >= DATEADD(minute, -10, CURRENT_TIMESTAMP());
  
  -- Get current warehouse size
  SELECT warehouse_size 
  INTO current_size
  FROM snowflake.information_schema.warehouses
  WHERE warehouse_name = 'ANALYTICS_WH';
  
  -- Scale up if queue is building
  IF (queue_depth > 10 AND current_size = 'MEDIUM') THEN
    ALTER WAREHOUSE analytics_wh SET warehouse_size = 'LARGE';
    RETURN 'Scaled up to LARGE due to high queue depth';
  -- Scale down if underutilized  
  ELSEIF (queue_depth < 1 AND current_size = 'LARGE') THEN
    ALTER WAREHOUSE analytics_wh SET warehouse_size = 'MEDIUM';
    RETURN 'Scaled down to MEDIUM due to low utilization';
  END IF;
  
  RETURN 'No scaling needed';
END;
$$;
```

## 11. Cost Optimization Strategies

### Database Replication

```sql
-- Enable replication for critical databases
ALTER DATABASE analytics ENABLE REPLICATION TO ACCOUNTS ('backup_account');

-- Failover procedures
CREATE OR REPLACE PROCEDURE failover_to_secondary()
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
  -- Promote secondary database to primary
  ALTER DATABASE analytics_replica REFRESH;
  
  -- Update connection strings (would be done at application level)
  -- Redirect traffic to backup account
  
  RETURN 'Failover completed successfully';
EXCEPTION
  WHEN OTHER THEN
    RETURN 'Failover failed: ' || SQLERRM;
END;
$$;
```

## 12. Disaster Recovery and Business Continuity

1. **Architecture Decision Making**: Ability to evaluate trade-offs between performance, cost, and maintainability
2. **Team Development**: Mentoring junior engineers and establishing best practices
3. **Stakeholder Communication**: Translating technical concepts to business users
4. **Project Management**: Balancing technical debt with feature delivery
5. **Performance Culture**: Establishing monitoring and optimization as core practices

## Continuous Learning Resources

- Snowflake certification paths (SnowPro Core, Advanced)
- Stay updated with Snowflake's quarterly releases
- Practice with real-world scenarios using Snowflake's free trial
- Engage with the Snowflake community and forums
- Hands-on experience with complementary tools (dbt, Fivetran, etc.)

This comprehensive guide covers the technical depth and leadership breadth needed to excel as a Snowflake tech lead data engineer.
