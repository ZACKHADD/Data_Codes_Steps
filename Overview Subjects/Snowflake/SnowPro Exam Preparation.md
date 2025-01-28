# This Document contains a recap of the Snowflake paid exam preparation course and other materials

## Data Architecture and cloud features:
Snowflake has three layer architecture :  
- Srorage layer linked to the cloud provider
- The compute layer which is simply the snowflke engine
- The Cloud services layer dealing automaticaly with optimization, gouvernance security and so on

![{7D6D69AB-75B1-42F2-851F-ABB70C07DD38}](https://github.com/user-attachments/assets/7342a7db-7122-43ce-966b-d80bafeec179)  

The data is not siloed so everyone can query everything if the are allowed to and the compute is seperated so you just choose a compute warehouse and data will be loaded there in the memory for compute : 

![{8403962C-73B4-4C14-B551-02C72CD18C30}](https://github.com/user-attachments/assets/f820338e-ac2f-4f21-af09-342b29c4aa29)

The type of computing warehouse is important and depends on the use case and the workload needed:

- For Loading data we can use XS size
- For Transformations we can use S size
- For an app (Data sience models for example) we can use L or XL depending on the need ==> Scaling UP
- Multicluster warehouse (generally Medium size) are essentialy used for reporting and analytics (power bi for example) that needs concurrency. At a certain point if the pressure on a reporting tool is high we can add another cluster to increase the concurrency ! ==> Scaling Out

### Several Editions :

![{90EEAD00-AB29-47C0-9A69-809CE82982AE}](https://github.com/user-attachments/assets/b0dcd6c5-4d88-4eda-8103-f54a8465f3b7)  

### Labs hacks:

tO list all the content of an external stage we can use LIST command.

the complete query to create the file format in order to ingest data from files in an external storage : 

```SQL
          CREATE OR REPLACE FILE FORMAT csv
              TYPE = 'CSV'
              COMPRESSION = 'AUTO'  -- Automatically determines the compression of files
              FIELD_DELIMITER = ','  -- Specifies comma as the field delimiter
              RECORD_DELIMITER = '\n'  -- Specifies newline as the record delimiter
              SKIP_HEADER = 1  -- Skip the first line
              FIELD_OPTIONALLY_ENCLOSED_BY = '\042'  -- Fields are optionally enclosed by double quotes (ASCII code 34)
              TRIM_SPACE = FALSE  -- Spaces are not trimmed from fields
              ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE  -- Does not raise an error if the number of fields in the data file varies
              ESCAPE = 'NONE'  -- No escape character for special character escaping
              ESCAPE_UNENCLOSED_FIELD = '\134'  -- Backslash is the escape character for unenclosed fields
              DATE_FORMAT = 'AUTO'  -- Automatically detects the date format
              TIMESTAMP_FORMAT = 'AUTO'  -- Automatically detects the timestamp format
              NULL_IF = ('')  -- Treats empty strings as NULL values
              COMMENT = 'File format for ingesting data';
```

Using LAG and moving AVG :

```SQL
                SELECT
                    meta.primary_ticker,
                    meta.company_name,
                    ts.date,
                    ts.value AS post_market_close,
                    (ts.value / LAG(ts.value, 1) OVER (PARTITION BY meta.primary_ticker ORDER BY ts.date))::DOUBLE AS daily_return,
                    AVG(ts.value) OVER (PARTITION BY meta.primary_ticker ORDER BY ts.date ROWS BETWEEN 4 PRECEDING AND CURRENT ROW) AS five_day_moving_avg_price
                FROM FINANCE_ECONOMICS.cybersyn.stock_price_timeseries ts
                INNER JOIN company_metadata meta
                ON ts.ticker = meta.primary_ticker
                WHERE ts.variable_name = 'Post-Market Close';
```

