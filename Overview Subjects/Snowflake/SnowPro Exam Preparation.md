# This Document contains a recap of the Snowflake paid exam preparation course and other materials

## Data Architecture and cloud features:

Snowflake is SaaS data plateform that has a hybrid architecture using a shared disk for the storage and a shared nothing for computing. This separates computing from storing data leveringin high scalabitily !  
Generally the cloud data systems one of the two either Shared disk or shared nothing architecture !  

![image](https://github.com/user-attachments/assets/2d778e8e-514d-4db7-80d9-b498d500cf7d)  

![image](https://github.com/user-attachments/assets/a7dc74d2-16ad-4559-a40a-73c06fddbc2b)  

![image](https://github.com/user-attachments/assets/1f08eddc-f1db-4ae7-b1ac-d1be0e49a395)  

Snowflake is a hybrid of these two :  

![image](https://github.com/user-attachments/assets/5e016d97-d9c7-4bd7-9ae5-eec12ad044a4) 

Snowflake can only run on cloud and all the infrastructure used in it is provided from one of the 3 famous cloud providers : AWS, AZURE and GCP. (Originally designed to run on AWS).  

### Snowflake has three layer architecture : 

1- Storage layer linked to the cloud provider :
  
![image](https://github.com/user-attachments/assets/4b01cb99-1a96-4de9-8d42-132227e84ed4)

2- The compute layer which is simply the snowflke engine : if we are on AWS it is simply a cluster of EC2 instances.

![image](https://github.com/user-attachments/assets/28acd1e2-d9c2-4f29-9653-62b8192dae23)  

Data cache in snowflake is so important as it avoids averheads and unessesary costs. Here how it works when a query is executed :  
- Step-by-step execution:
  - Query Planning: Snowflake checks the metadata cache to identify which micro-partitions are needed (each column has medatada regarding min/max, distinct values .. and if we have a where clause in the query snowflake skips the partitions where the min/max does not contain the where clause valeue). If partition pruning is possible, only relevant partitions are considered.
  - Checking Result Cache: If the exact query was executed before and **data hasn’t changed**, Snowflake instantly returns results from the result cache (No compute cost ✅).
  - Checking Virtual Warehouse Cache: If this warehouse has executed a similar query recently, it reads from its local cache (avoiding storage access, making it much faster). If the warehouse cache is empty, it fetches data from storage.
  - Fetching from Storage (If Needed): If no cache is available, the warehouse reads micro-partitions from storage (slowest step) and loads them into the memory. This data is cached for future queries.
  - Returning Results: Snowflake compresses results and stores them in the result cache for reuse.

**Note that to trigger data prunning when using where clause, we should not use functions with the condition such as : UPPER(region) = 'US'; this will force snowflake to scan all partitions and apply the upper transformation row by row before filtering since the metadata is not precomputed with upper function**

along side with the micro-partitionning that is automatic following the order of insertion, we can **Cluster** data which is the same thing only using specific keys we explicitly define rather than the order of insertion !  

3- The Cloud services layer dealing automaticaly with optimization, gouvernance security and so on:

![image](https://github.com/user-attachments/assets/6be0ae41-ae85-4d35-ade8-6f62e6b01ee9)  

![{7D6D69AB-75B1-42F2-851F-ABB70C07DD38}](https://github.com/user-attachments/assets/7342a7db-7122-43ce-966b-d80bafeec179)  

The data is not siloed so everyone can query everything if the are allowed to and the compute is seperated so you just choose a compute warehouse and data will be loaded there in the memory for compute : 

![{8403962C-73B4-4C14-B551-02C72CD18C30}](https://github.com/user-attachments/assets/f820338e-ac2f-4f21-af09-342b29c4aa29)

The type of computing warehouse is important and depends on the use case and the workload needed:

- For Loading data we can use XS size
- For Transformations we can use S size
- For an app (Data sience models for example) we can use L or XL depending on the need ==> Scaling UP
- Multicluster warehouse (generally Medium size) are essentialy used for reporting and analytics (power bi for example) that needs concurrency. At a certain point if the pressure on a reporting tool is high we can add another cluster to increase the concurrency ! ==> Scaling Out

### Several Editions :

![image](https://github.com/user-attachments/assets/bbf29850-ef51-413c-a51e-6507506edf4d)  

The Business Critical edition offers higher levels of data protection and enhanced security to support organizations with very sensitive data, for example, data that must comply with regulations. With this edition, we can enable private connectivity to our Snowflake account, or use our own customer-managed key to encrypt data in Snowflake. Business Critical includes all the features and services of the Enterprise and Standard editions.  

Virtual Private Snowflake. This offers the highest level of security for organizations that have strict requirements around data protection, such as governmental bodies or financial institutions.  

The services layer is shared between accounts. If you were to create a VPS edition account, this wouldn't be the case. Services like the metadata store wouldn't be shared.  

### Snowflake Catalog and objects : 

![image](https://github.com/user-attachments/assets/6d134ef2-12fa-4ca0-9f88-bdbff6da0f67)  
The first level of hierarchy is the organization. An organization can hold multiple accoucnts (up to 25). We can set some administration rules such as replications and accounts creation.  
Then we have the Account level where we will have all the databases and the other objects. If created by snowflake, the structure will contain locator, regon and the cloud provider. If created in an organization using ORGADMIN role it contain the name of the organisation.  

![image](https://github.com/user-attachments/assets/43176a27-dc88-4cfb-9ffb-83c880b2faa6)  

Snowflake has a list of objects in the ecosystem that are listed bellow :
- Databases
- Schemas
- Table Types:

![image](https://github.com/user-attachments/assets/97a090f0-141a-4a50-8440-05eea3cd5752)  

Transient tables are staging ones. They hold temporary data but available accross sessions.  
- View Types:
  
![image](https://github.com/user-attachments/assets/cc2b2745-c083-4882-9ee7-e2b2189343cb)  

Materilized views are said **Precomputed Datasets**

- Data Types
- User-defined Functions (UDFs) and User-defined Table Functions (UDTFs):

![image](https://github.com/user-attachments/assets/1b87cc33-c148-419e-a491-b67f52a47991)  

![image](https://github.com/user-attachments/assets/aafdd2a6-6d78-48dc-a556-bb2579b03fcb)  

![image](https://github.com/user-attachments/assets/888684de-ce22-4158-a99b-d7357bdd8034)  

- Stored Procedures
- Streams :  For CDC and incremental loading of data. A select statement does not consume the stream while using it in a DML does.  
- Tasks : Used to schedule and automate SQL queries or procedures in Snowflake, which can include operations on tables or pipes themselves. A DAG is constructed when we link several tasks together but note that all tasks need to have the same owner and belong to the same schema and database !
- Pipes : Specifically designed for automatically loading data from external stages into Snowflake tables, often used for real-time or continuous data ingestion.
- Shares
- Sequences :  auto incremented values.

### Billing : 
Several types of elements impact the snowflake billing:  

![image](https://github.com/user-attachments/assets/b16e67a8-9bb4-41a8-ad56-2226ab2f96bf)  

The cloud services are all the queries we use and does not use a user managed warehouse. Such as Creating a table !  
The serverless services on the other hand can be snowpipes, database replications and clustering !

![image](https://github.com/user-attachments/assets/1b8794d1-739c-44dc-b1ac-6149b9be1741)  

### External tools :  

Snow SQL, partners and connectors !  

### Snowflake Scripting : 

We can write sql code in a procedural logic in what we call anonymous block (excuted outside a stored procedure) and in SP.  

![image](https://github.com/user-attachments/assets/9ab2aeb7-53b0-45a2-92c9-16a7564054ae)  

Also in SP :  

![image](https://github.com/user-attachments/assets/2137d7d6-c788-4dfa-81d2-d052c7d45fa3)  

Note that declare and exceptions are optionals in a block :  

![image](https://github.com/user-attachments/assets/0a709309-8765-4032-862d-589af6284a62)  

We can use also the looping constructs :  

![image](https://github.com/user-attachments/assets/478af32c-3e23-435d-9447-9561f2aa6875)  

We can also loop on records inside a table. For this we use cursors :  

![image](https://github.com/user-attachments/assets/2d63ac67-3cbf-4aa1-96d2-6faf35b1f809)  

We can also store the result of a SQL query in a Resultset object to be used in a SP or a script and make a SQL query dynamic :  

```SQL
DECLARE res RESULTSET;
DECLARE query TEXT;

SET query = 'SELECT name FROM employees WHERE department = ''Sales'';';
SET res = (EXECUTE IMMEDIATE :query);

SELECT * FROM TABLE(res);
```
In Snowflake, a ResultSet is an object that stores the output of a query execution. It allows you to programmatically fetch and manipulate query results within stored procedures, scripts, or Snowflake connectors. 

### Snowpark :  

![image](https://github.com/user-attachments/assets/86151bd5-627d-4137-8119-f60098ee5543)  

Key Features of Snowpark  
✅ 1. Native Support for Python, Java, and Scala  
Unlike traditional SQL-based transformations, Snowpark lets you use Python, Java, or Scala to process data.  

Example: Instead of writing SELECT * FROM table WHERE ..., you can use Python DataFrames.  

✅ 2. Pushdown Execution in Snowflake  
Code is executed inside Snowflake’s compute engine, reducing data movement.  

No need to export data to external environments (e.g., Spark clusters or pandas in Python).  

✅ 3. Object-Oriented DataFrame API  
Snowpark provides a DataFrame API, similar to pandas (Python) or Spark DataFrames, for manipulating data.  

Example in Python:  

```Python
from snowflake.snowpark import Session

session = Session.builder.configs({...}).create()  # Create Snowflake session
df = session.table("sales").filter("region = 'US'")  # Query data
df.show()  # Display results

```
✅ 4. Supports UDFs (User-Defined Functions) and Stored Procedures  
You can write custom functions in Python, Java, or Scala and run them within Snowflake.  

Example: A Python UDF in Snowflake:  

```python
from snowflake.snowpark.functions import udf

@udf
def square_number(x: int) -> int:
    return x * x
```

This function can be called in Snowflake SQL:  

``` sql
SELECT square_number(10);  -- Returns 100
```  
## Acocunt Access and security:  

### Roles:  

Snowflake uses two access controle frameworks :  

![{539EB27E-6AEB-4E82-9000-331FB7C702B2}](https://github.com/user-attachments/assets/2870497b-da33-4f9f-b5f5-d8cf6d782946)  

We have by system defined roles :  

![{FEF76E61-626C-4598-9559-C6B197C2DC67}](https://github.com/user-attachments/assets/b96afbe7-cb40-4432-8e15-f0d22d6f9e35)  

![{71F50619-982E-4B08-A9E3-70968B54D1EC}](https://github.com/user-attachments/assets/08c7e495-a659-41d4-8e34-27920143bb1e)  


But we can also create custom roles with granular previleges :  

![{BAD95782-05B4-4510-B42F-18716BB803A8}](https://github.com/user-attachments/assets/989061d9-dcfa-4e60-8d8e-4dde8556a9b0)  

All the custom roles need to be affected to a system defined role (in the most cases sysadmin) so that the objects owned by these roles can be tracked by accountadmin for example !  
By default a role to which we affect a another role will inherit all the previleges of this latter !  

### Previleges :  

Each role in snowflake has some previleges. We can create custom roles with specified grants to do a specific job like an analyst for example.  
![{BDB80F4A-3C0A-4FB4-9FDB-7365C8E53D5B}](https://github.com/user-attachments/assets/90629cc9-f8b8-44f5-9b7c-12d69a797278)  

### Authentication:  

There several ways we can authenticate to snowflake:  

✅ 1. Snow UI User authentication:  

Using user name and password :  

![{773DBC07-435C-43F5-92D6-2D1232EB7FD3}](https://github.com/user-attachments/assets/d4c926a9-3c13-46ee-b14f-eb970305ff75)  

To add another layer of security we can add Multifactor authentication using Duo Security:  

![{B156BC7D-683B-42F6-9CF2-AD3568F9981D}](https://github.com/user-attachments/assets/cca8a0bf-fb91-460a-b5c3-6b30d9baa454)  

![{3845C23F-D685-406D-88A9-23A24197A5AF}](https://github.com/user-attachments/assets/a62d7629-6b2f-43f8-84f5-7df01079d248)  

✅ 2. Federated authentication (SSO):  

![{452E8B7E-F3C9-44FB-BDA7-858A307C3416}](https://github.com/user-attachments/assets/8aa7fca0-13fc-4ffe-8e00-8a5e4fc6ad19)  

✅ 3. Key pair authentication:  

It used when we would like to connect to snowflake using a client application such as terraform without using directly password in the code :  

![{201FD6B1-9E3F-48FA-9017-EF3391570631}](https://github.com/user-attachments/assets/0607e77f-46a8-4b8c-8638-31af1fea5f47)  

✅ 4. Oauth and Scim:  

![{A270AF0B-F354-4C15-8A40-101351C899DE}](https://github.com/user-attachments/assets/d1c61194-0a22-46df-9e2d-46c49269a06b)  

### Network policies :   

When we define a network policy in Snowflake, it restricts user authentication to only the allowed IP addresses specified in the policy. 
Meaning that if a user tries to authenticate from an unauthorized IP, Snowflake rejects the connection.  

![{E77D6607-18E4-4FBC-872C-915A92AF2B51}](https://github.com/user-attachments/assets/946f730e-b2d8-46a1-8ed4-7c7244e0e170)  

- Account-Level Policy: Affects all users and roles.  
- User-Level Policy: Overrides account-level policies for specific users.

```SQL
CREATE NETWORK POLICY restrict_office_ips 
    ALLOWED_IP_LIST = ('192.168.1.100', '203.0.113.42') 
    BLOCKED_IP_LIST = ('45.67.89.10');
```
We then use alter Account or User to set the Network_Policy.  

```SQL
ALTER ACCOUNT SET NETWORK_POLICY = restrict_office_ips;
ALTER USER john_doe SET NETWORK_POLICY = restrict_office_ips;
```

### Data Encryption :  

Data in snowflake is enrypted end to end using two encryption mechanisms :  

![image](https://github.com/user-attachments/assets/a612f4ad-99e1-4070-9bcb-37199d3f19e2)  

The flow encryption depends on where data is stored :  

![image](https://github.com/user-attachments/assets/2d89e6d0-8efe-4e33-af1a-4023ee1e8ae2)  

We can set our client side encryption in the external storage but we will need to provide the encryption key to snowflake so it can store data and then encrypt!  

The encryption is so resilient thanks to the hierarchical key model used :  

![image](https://github.com/user-attachments/assets/0dcd20fa-4248-473a-993d-dbc35edc6258)  

This means that if a key is exposed the other components for example files or micro-partitions are  not exposed !  
The mother key is stored seperatly in a AWS cloudHSM service.  

To enhance this encryption process snowflake uses Key rotation that raplaces envery 30 days the account and the table keys while keeping the older keys onlu to decrypte the data previously encrypted !  

![image](https://github.com/user-attachments/assets/26656688-afd5-4dd3-a8ae-cd2a5b8d483c)  

The Rekeying however is another service we can activate (additionnal cost) that changes the keys retired that exceed one year and completely reencrypt its data :  

![image](https://github.com/user-attachments/assets/ecad1894-980c-45a0-a493-149a8b069d6d)  

The last feature where we add another securty layer is the Tri-secret secure and customer managed keys where we simply manage our own keys in a service like KMS or Azure key vaults !  

![image](https://github.com/user-attachments/assets/6ca59327-8b7d-4753-aefe-05aebdeff9f1)  

### Column level security :  

We can set security on a column using data masking :  

![image](https://github.com/user-attachments/assets/dde43eff-38a8-427d-8bfd-7d7d9f1c6771)  

If the user queries data and he is anauthorized to see it it will be masked. For example for emails only the domain name will show !  

To create a masking policy we can do as follows :  

![image](https://github.com/user-attachments/assets/85cc369c-33a9-4b90-ad6a-077a0d1d3ebd)  

Data masking has several properties :  

![image](https://github.com/user-attachments/assets/91486ac1-363f-4617-9301-9c0e5e35dfc4)  

We can also use external functions to mask data inside masking policies. This is called external tokenization:  

![image](https://github.com/user-attachments/assets/85bf9b63-72b7-4138-beb6-260a6afc71f7)  

In this way even snowflake and the cloud provider cannot read the data as it is stored tokenized and needs the function to be detokenized !  

### Row level policies:  

![image](https://github.com/user-attachments/assets/4de08414-628c-4a76-8dd5-8eb94d4d4065)  

We can only set one row access policy per table !  

### Secured Views :  

In standard views, when we define a view it is related to a whole table behind:  

```SQL
CREATE VIEW employees AS
SELECT id, name, department 
FROM employees;
```
The table behind has also the salary column for each employee but we hide it in this view. Now when users query the view, Snowflake rewrite the query and applies query optimizations such as:
    
    1- Predicate Pushdown → Applying WHERE filters as early as possible.
    
    2- Join Reordering → Reordering joins for efficiency.
    
    3- Column Pruning → Removing unused columns to reduce data scanned.
    
    4- Aggregation Pushdown → Performing aggregations earlier in execution.

While standard views mostly restrict direct column access, they can leak metadata and data relationships through: 

- Error messages (revealing hidden columns)
- Query timing (inference attacks)
- Nested data functions (JSON, arrays)
- Statistical anomalies (error handling differences)

All this may give the attacker a way to guess the hidden columns and maybe the values also !  

Secure views explicitly block these side channels:  

![image](https://github.com/user-attachments/assets/8e6885ca-58a2-4ec6-a02f-d179f678451f)  

It does this by bypassing some of the view optimizations performed by the Query Optimizer. But note that these views are slower and for that reason, making views which don't have strict security requirements, secure is discouraged by Snowflake.  

### Account Usage and information schema :
These schemas are so important for metadata analysis of our account and schemas. For the account it holds all what happend to the account objects :  

![image](https://github.com/user-attachments/assets/ef84220b-0082-4ef0-8791-9b614ad52afe)  

The information schema is present in every database and it gives quite similar information but only for the corresponding database. The metadata here are refreshed faster than the account usage :  

![image](https://github.com/user-attachments/assets/15d62cb6-4651-46a5-b9b9-4bef202c430e)  

When to use what ? :  

![image](https://github.com/user-attachments/assets/bb83948f-f7b3-4a45-b442-fdc3c6e54fc0)  

### Object tagging :  

they can be very helpful to group objects of a buisiness for example.  

![image](https://github.com/user-attachments/assets/81538055-2d72-46f3-a326-1f325559b284)  

object tags are schema level objects, so we'll need to have a database and schema set to create one. This will store all the tags and only the data stewarts can have access to that and later maybe give access to others to apply tags on account objects !  example :  

```SQL
ALTER WAREHOUSE COMPUTE_WH SET TAG business_unit = "Finance"

# we can also when creating the tag (or altering it) add a list of allowed values

CREATE OR REPLACE TAG business_unit ALLOWED_VALUES  'sales', 'finance', 'hr'
```
**Note that a child object will inherit the tag of the parent object !. Also an object can have up to 50 unique tag !**  

Tags can be used also to enhance the data governance and security. For example we can use tags on columns to automatically enforce masking policies.  

![image](https://github.com/user-attachments/assets/bf285afe-4dc2-4299-8682-faa3138ec76a)  

This data governance using tags is handeled by a data stewart or a team of data stewarts in two aproaches centrelized when they create and apply tags and decentrelized when they only create tags and give the possibility to others to apply these tags :  

![image](https://github.com/user-attachments/assets/1e972bad-7c78-4e3c-96af-4fb119a22693)  

Snowflake recommend creating a custom role like a tag admin, which can be granted to a data steward user.  

## Performance Concepts: Virtual Warehouses 

### How VW work:  

A virtual warehouse is simply a cluster of insrances that performes computations:  

![{D4770917-404A-41A2-BE03-9A25C60B9973}](https://github.com/user-attachments/assets/715ed352-949f-46e0-9715-fd711250daa6)  

They can be created and modified on the fly :  

![{132C5E2A-E55C-4230-A71B-34096EA82FF7}](https://github.com/user-attachments/assets/f2d18f0a-f884-4594-8183-f6a9d5e603b1)  

The are completely flexible which makes them suits a lot of worloads types :  

![{6842F704-D3FB-4295-BF97-2BB9ADD7D893}](https://github.com/user-attachments/assets/34963c5a-b2a2-4d5e-b2b8-8299ba4b5f55)  

the warehouse has 3 properties :  

![{5694D3DC-E600-4D6F-88B1-D74C80AA2CE8}](https://github.com/user-attachments/assets/8f29bf09-1131-46f4-8a6e-036b435238ec)  

Note that if for billing optimizations reasons we decide to set the auto suspend to it's lowest it may results to more query execution time as the warehouse needs to be resumed each time and also we get charged the minimmum cost each time which is 60 min.  
Also the cache gets deleted so we don't benifit from its performance enhancement.  

Billing is based on warehouse running time, not per query. Snowflake charges per second (with a minimum of 60 seconds) when a warehouse is active (running).  
If a warehouse is running but no queries are executed, it still incurs charges because it is consuming compute resources.  

### Ressource monitors

The ressource monitors helps optimizing and track our costs :  

![{907BB773-10F8-4F50-BD87-0C917AA47BD2}](https://github.com/user-attachments/assets/f6932e7c-d868-40c5-893b-878de02b4998)  

we can create ressource monitors with several properties and attach them to warehouses :  

![{48104337-977B-4B8B-94BD-3444E0A89D97}](https://github.com/user-attachments/assets/18b0adff-866a-42d3-9a7d-c9c54246635c)  

This will notify us whenever we reach the a percentage of cost. We can set the ressource monitors on account also which will track the cost of all the warehouses in the account.  

### Scaling up and scaling out:  

The scaling up has a relation with improving the performance of queries when it exceed the capacity of the warehouse so we resize the warehouse adding new nodes to the same cluster :  

![{CF2C8AC1-D401-4DAB-879C-E4EF8DCFDE9D}](https://github.com/user-attachments/assets/b65d6f6e-0d8f-4f58-a431-3553e8863b1f)  

This operation is manual !  

Whereas the scaling out has a relation with respnding to queries coming from multiple users in the same time to enhance the concurency :  

![{14B04A7F-8E78-4581-95AD-E6724E769E86}](https://github.com/user-attachments/assets/4662b1ae-2aaa-4acc-a8fd-389699c6995b)  

By default and starting from the enterprise edition, all warehouses have multi clustring set with a min and max set to 1.  

We can also set some scaling out policies :  

![{AFB0F24E-C1B1-4390-81A5-835B8CF91335}](https://github.com/user-attachments/assets/cc368442-561a-457e-bcfe-37521a1fd9f1)  

![{82D8370E-89DC-41B0-B8E1-78EED0167D28}](https://github.com/user-attachments/assets/e96ab651-1a40-4174-9850-4d9948c23f80)  

Some properties are important in the warehouse creation that set for example the timeout :  

![{583910CB-B918-4F56-B160-D0E47BF8D3FB}](https://github.com/user-attachments/assets/95c32083-d8bd-4fdf-ac91-e1db643a089b)  

The default max queries before queuing is 8.  

### Query acceleration service :  

The Query Acceleration Service, or QAS, is a feature we can enable on a virtual warehouse. Its purpose is to dynamically add serverless compute power to a warehouse, only when a sufficiently complex query needs it. Since it is not set by us as we do with normal warehouses we call it serverless as it is handeled by snowflake ! In this case we pay for only the part of the query that was handeled by the QAS !    

![{E0907E01-C3C4-4E44-92E2-69ED5095401C}](https://github.com/user-attachments/assets/40f92b98-d97a-4e31-9d60-754ec8fc41ed)  

Example :  

![{D014DDD3-077A-4E3D-9AFE-A13C265B824A}](https://github.com/user-attachments/assets/7189b36c-de6e-4aa4-ae2c-476e1956e587)  

For a single peak if wa scale up we would be pating for a larger warehouse ! scaling out would work but it will depend on the scaling policy defined ! so the perfect thing would be the QSA that would be cheeper and would accelarate the execusion !  

However not all the queries can be accelerated !! Snowflake decides that !  

![{69D62693-D41E-49FB-A1E2-999AAFF0431F}](https://github.com/user-attachments/assets/9a6a7c1d-f9ba-4477-908b-55e5954b2841)  

![{83E32A21-1AEC-4E4D-8C87-E51540CDEC2C}](https://github.com/user-attachments/assets/920b2383-30bc-4920-9814-cc602c4019cc)  

![{AABFD250-254A-4A50-A523-3F659EB273A1}](https://github.com/user-attachments/assets/5a8745dc-bb30-4175-bd99-0f610198300c)  

![{0CF3E5A3-BA40-4D17-8A05-13973ED0B576}](https://github.com/user-attachments/assets/44cde67c-9077-4f8e-86dc-0f081c62db1f)  

## Performance concepts : Query optimization

Optimizing the queries we run in snowflake is an important thing along side with the warehouse optimizations we already highlited above ! There are several tools to identify when the query optimization is needed :  

![{36337975-AEB2-43ED-87A9-04004E2EF1FA}](https://github.com/user-attachments/assets/43597a57-c56c-4e58-b58c-f4dc3057a71e)  

Query history is available for everyone regardless of the role they have but no one can see the results of the queries of other users.  

The query profile gives a graphical representation of how snowflake executes the query just like and EXPLAIN query would !  

![{AB78125E-2F00-4F89-A02C-2F617EADC28A}](https://github.com/user-attachments/assets/38eb433f-4c97-4674-8cd1-93aa07deb8c0)  

![{66CD3EBC-1811-462E-A1AC-8C5397363AB7}](https://github.com/user-attachments/assets/16b64ba8-deab-4261-906b-16e82a09bf12)  

The UI query history is limited to 14 days. If we want more we can use the account usage schema. Only the accountADMIN can query this schema and it has a latency of 45 min compared to the information schema in each database that has 0 latency but only goes up to 7 days of history.  

### SQL tuning :  

If we want to optimize queries we need first to know the order of execution of the queries !  

![{B2DFBFFE-6D5B-49A4-B627-314E5E6B7F3B}](https://github.com/user-attachments/assets/0c93c528-fa2e-40ad-bc6b-ccb8d9947bbe)  

This order is important since it first filter data before applying more expensive operations. **Always use filrer operations as early as possible !**  

Using limit when ordering data can significantly improve the performance as it only order the filtered data :  

![{B2C408EA-DEEC-4DBC-8F0B-EDA7A7456940}](https://github.com/user-attachments/assets/8b6c3dac-c8e1-4da2-b52c-e78ade05a4bc)  

This is to avoid spilling to local disk. This occurs when Snowflake runs out of memory (RAM) while processing a query and needs to temporarily store intermediate results on disk instead. This can significantly slow down query performance since disk I/O is much slower than memory access.  

Several reasons can cause spilling to local disk : 

- Large Joins or Aggregations: Queries require more memory than available in the warehouse.
- Sorting or Window Functions: These operations may generate large intermediate result sets.
- High Concurrency: Multiple queries compete for the same memory resources.
- Suboptimal Warehouse Size: The warehouse is too small for the workload.

The spilling can also happen to remore storage which is even worst !  

So as we see optimizing joins is so important here to reduce the intermediate operations that generates heavy intermediate datasets !  

![{130BE077-B2AE-472D-8D73-D5D9651EB1E8}](https://github.com/user-attachments/assets/ceb90988-f576-4e1f-80b7-a2452bef831a)  

Order by is an expensive operation so we need to handel its position very carefully and use it only when needed and preferably with LIMIT:  

![{96108407-5C31-4314-A09F-6A0CA1CE0B16}](https://github.com/user-attachments/assets/14faad2f-ae94-4612-909b-b24008ae25c4)  

The group by also is an important operation that we need to use carefully and most of time with low to medium cardinality columns :  

![{DA4C43DE-9F62-4E20-ABD7-A5AD5E1DC952}](https://github.com/user-attachments/assets/5e8e41a6-01ec-4549-a7d4-4048007a7c58)  

Lets wrap up all of this : 

1️⃣ Snowflake’s Storage & Compute Layers

Snowflake follows a decoupled storage and compute model:

- Storage Layer (e.g., AWS S3, Azure Blob, Google Cloud Storage)
- Stores data as compressed, immutable micro-partitions.
- This layer is separate from compute.
- Data remains here until queried.

Virtual Warehouse (Compute Layer)

- Retrieves only the required micro-partitions from storage.
- Loads them into local SSD storage (warehouse cache).
- Uses RAM for intermediate computations.
- No persistent data is stored here—only cached data for optimization.

2️⃣ How Query Execution Works

When you run a query, Snowflake:

- Prunes the micro-partitions → Only relevant partitions are fetched.
- Loads them into warehouse SSD cache → To avoid direct storage access.
- Processes data in RAM → Joins, aggregations, sorting, etc., are done here.
- Returns results → If everything fits in RAM, the query runs optimally.

3️⃣ When Does Spilling to Disk Happen?

If RAM isn’t enough to hold intermediate results, Snowflake moves them to local SSD (warehouse disk) for temporary storage.

- First Level of Spilling → Moves data from RAM to local SSD (warehouse disk).
✅ Still relatively fast since SSDs are used.

- Second Level of Spilling → If local SSD is also full, data spills to remote storage (AWS S3, Azure Blob, etc.).
❌ This is very slow because it requires network I/O.

4️⃣ Why is Spilling a Performance Issue?

- RAM → SSD is slow (compared to keeping everything in memory).
- SSD → Remote Storage is even slower (adds network overhead).
- Queries that spill extensively can take much longer to complete.

### Caches in snowflake :  

Several caches at each layer : 

- Metadata Cache:  

![{E669FD9E-A195-4B30-8AC8-80E73A2EA385}](https://github.com/user-attachments/assets/079812bf-0b7b-4d39-9b1b-409a72f89e00)  

Note that Queries that only access metadata in Snowflake are free because they do not use a virtual warehouse. Instead, they fetch precomputed information from Snowflake’s metadata cache.  

- Result Cache :

This is where the results of queries are hold up to 24h and it extends each time the same query is used up to 31 days after which it is purged. This is very helpful to reduce costs especially for BI puposes :  

![{8BA080FC-FA80-4A88-9355-9555D08675E9}](https://github.com/user-attachments/assets/f68f0e81-04cd-4550-b833-5255d2ebbfe5)  

🚀 This means that if Power BI for example runs the same queries repeatedly, Snowflake won’t charge extra compute costs!  

We can verify this in the query profile :  

![{D9924443-4331-42D1-A433-BC2511B3840C}](https://github.com/user-attachments/assets/5039183b-f7a0-4faa-ad6d-a0675e54cb40)

✅ Result Cache applies if:

- The same query is executed (no formatting or structure changes).
- The underlying data hasn't changed (no INSERT, UPDATE, DELETE, or TRUNCATE).
- The query is executed within 24 hours (or refreshed within 31 days in higher editions).
- The same role/user runs the query (some cache optimizations depend on the role).

**Remember that we only pay for storage and computing !**  

### Warehouse cache (generates costs):  

![{8E779625-47B8-4D12-BD8D-D973CD70202F}](https://github.com/user-attachments/assets/330e5005-0252-4c0e-8d38-4c91e9fbd506)  

If we rerun a queries (different in the syntax) that will use the same partitions the warehouse cached is used. And we can see in the query profile that it uses 100% of data processed from the warehouse cache and not the remore cache:  

![{8EEACA52-FF46-4E0B-AD2D-FC246E252E19}](https://github.com/user-attachments/assets/67d5203e-44d5-42f0-ab93-3153185b2b92)  

If we use queries that will add other data, only the new data will be processed from the remote storage and the rest is already in the warehouse cache and we can see that in the query profile with a processing % in the remore storage lower than 100% :  

![{85427661-5928-4AD6-8FDD-A61B0F26CED3}](https://github.com/user-attachments/assets/2cba4c58-0456-452e-9c13-56694d6ed8fc)  


How the warehouse caching is used? :  

1️⃣ First Query Execution:

- The virtual warehouse reads micro-partitions from remote cloud storage.
- Data is stored in memory (RAM) and cached on local SSDs of compute nodes.
- The query runs and returns results.

2️⃣ Subsequent Queries (Within Active Warehouse Session):

- If another query needs the same micro-partitions, it is fetched directly from the Local Disk Cache, avoiding slow cloud storage reads.
- Much faster execution compared to the first run.

3️⃣ When the Warehouse is Suspended or Reassigned:

- Local Disk Cache is cleared when the warehouse stops.
- Restarting the warehouse means queries must re-fetch data from remote storage unless cached in Result Cache (Service Layer).

### Materialized views:  
Materialized views improve the performance, especially if used on top of external tables.  

![image](https://github.com/user-attachments/assets/98459ed9-15f6-4381-9f78-8d994e1a9352)  

However there are some limitations such as using only one table, no joins and some SQL functions are not allowed :  

![{F5917CAE-DED9-4E57-AAA0-A1CBC027DEBF}](https://github.com/user-attachments/assets/1d2e318a-76d2-410a-bc5a-0adc4d31591d)  

Note that a materialized view is essentially a precomputed result set that is physically stored in the database. The goal of a materialized view is to optimize performance by saving the result of an expensive query and allowing the database to retrieve that result instead of recomputing it each time a query is run. LIMIT, on the other hand, restricts the result set to only a specific number of rows. If you use LIMIT in the definition of a materialized view, the database would store only a subset of the data, which defeats the purpose of a materialized view.  

Instead of using LIMIT, most systems recommend using techniques like window functions or ROW_NUMBER() to simulate the effect of limiting data in a controlled and predictable manner.  

### Clustring :  

All data in Snowflake is stored in database tables, logically structured as collections of columns and rows. To best utilize Snowflake tables, particularly large tables, it is helpful to have an understanding of the physical structure behind the logical structure.  
**Micro-partitions and data clustering**, two of the principal concepts utilized in Snowflake physical table structures.  
Snowflake automatically divides tables into micro-partitions, which are immutable internal storage units (each typically 50–500 MB compressed).  
Micro-partitionning is automatic and does not need to be maintained !  
This is powerful as it gives the possibility to prune columns to be scaned when we use filter predicates !  

How it works : 
- When data is inserted, Snowflake automatically divides it into micro-partitions.
- Each micro-partition stores metadata like min/max values, column stats, and cardinality.
- These micro-partitions are immutable and stored in columnar format.
- Snowflake prunes unnecessary micro-partitions at query time, reducing scan costs.

By default micro partitionning groups data based on the insert order ! **so if this order is randomly inserting data then the prunning will not be that efficient and we may need to cluster data**!  

![image](https://github.com/user-attachments/assets/958576e7-0892-45ff-9aa7-02d24f7f6258)  

How Does Clustering Work?
- Snowflake checks how well data is ordered within micro-partitions.
- If needed, it automatically reclusters data in the background (for large tables).
- We can manually trigger reclustering with ALTER TABLE RECLUSTER.

This will optimize the queries using frequently some columns like date, id and so on !  
We can use this command to cluster based on a columns or multiple columns : 

```SQL
          CLUSTER BY {Column1, Column2 ...}
```

### Clustering Depth
The clustering depth for a populated table measures the average depth (1 or greater) of the overlapping micro-partitions for specified columns in a table. The smaller the average depth, the better clustered the table is with regards to the specified columns.  

![image](https://github.com/user-attachments/assets/446c99c0-37a1-4e6e-b09f-e0fe43aecebb)  

### Automatic clustering :  

We can manually set the clustering on a columns or several columns that are used in most queries to improve the performance. Once the key or keys are set, snowflake would run periodically a recluster operation to rearange data (by creating new micro partitions clustered as needed).  

![image](https://github.com/user-attachments/assets/35b32fbc-b0c7-4a58-b7f5-2e45be8a313a)  

the true indication of whether a clustering key is required is whether queries are taking unreasonably long to execute, especially when filtering and sorting on frequently used columns. It's important to decide carefully as there is a cost associated with clustering, which should be compared with the benefits gained. Larger tables benefit from clustering simply because even if small tables are poorly clustered, their performance is still good.  

Clustering is most effective for tables that are **frequently queried and change infrequently**. The more frequently a table is queried, the more benefit you'll get from clustering. **However, the more frequently a table changes, the higher the cost will be to maintain the clustering. The cost of maintaining a key might outweigh the performance benefit given by clustering.**  

The cardinality when choosing clustering keys is so important. high-cardinality columns result in randomly distributed values across partitions. This means related rows are scattered across multiple partitions.  
In this case Snowflake cannot prune partitions efficiently because Transaction_ID for example is scattered. It will likely have to scan many partitions to find the row (the metadata of max and min won't work since data is randomly partitioned).  

![image](https://github.com/user-attachments/assets/6d04e856-ad73-4993-8a7e-8e32be1213cd)  

Also, if the cardinality is so low the partitions would overlap (since the size of the partition has a limit) with high clustering depth and snowflake would scan too much micro partitions to find tha wanted values.  

it's suggested that when using multi keys clustering that the lowest cardinality is selected first, followed by the higher cardinality. 

![image](https://github.com/user-attachments/assets/651bbfae-d8ce-433f-a085-7d96a3c3b99e)  

Example of a well clustered key :  

![image](https://github.com/user-attachments/assets/36c2305f-66b8-4f6a-95cf-a13a59c24bb1)  

The higher total constant partitions is the better clustering is. Average ovelaps and depth sould be low and the partition depth histogram must be skewed on the left meaning most of partitions have a low partition depth!  

Example of bad clustered key :  

![image](https://github.com/user-attachments/assets/b4bc8456-b961-4587-a645-7e1a16ed986a)  

We can clearly see here that to retrieve a value we would scan all the micro partitions !  

in a query we can see in the profile if it did benifit from the good clustering or not by checking the pruning information :  

![image](https://github.com/user-attachments/assets/4b818651-d203-4028-974d-53ca1a2d37c3)  

Finaly, the automatic clustering uses serverless compute (since is does it for a first time and maintains it periodicaly) and uses also the storage and we can use the AUTOMATIC_CLUSTERING_HISTORY to see how much credits does the clustering comsume.  

### Search Optimization Service:  

In some scenarios when a user might need to look up an individual value. On a very large table, say terabytes in size, this operation can be costly and time consuming.  

The search optimization service is a table level property aimed at improving the performance of these types of queries called selective point lookup queries.  

![image](https://github.com/user-attachments/assets/61087cf5-371f-4c32-97f1-9e66923264af)  

Once set as a property on a table, it creates an auxiliary index structure on selected columns to quickly locate values without scanning entire partitions.  
**It is very Useful for high-cardinality columns, point lookups, and sparse queries (e.g., searching by email, UUID).** in opposition to clustering on high cardinality keys. Snowflake automatically maintains the index, but there’s a cost associated with it.  

Search Optimization Service supports searching the following data types: fixed point numbers, so INTEGER and NUMERIC, DATE, TIME, and TIMESTAMP, VARCHAR and BINARY.  
- Without Search Optimization:  
❌ Snowflake scans all partitions (because email is high-cardinality, pruning is ineffective).

- With Search Optimization Enabled:  
✅ Snowflake instantly finds the relevant partition(s) and retrieves the row → Faster query.

📌 When to Use Search Optimization:  
✅ Point Lookups (WHERE id = 123456)  
✅ High-Cardinality Filters (WHERE email = 'x@example.com')  
✅ Sparse Queries (queries returning very few rows)  

📌 When NOT to Use Search Optimization:  
❌ Range Queries (WHERE order_date > '2024-01-01') → Pruning works better!  
❌ Full Table Scans → No benefit from search optimization.  
❌ Small Tables → The overhead is unnecessary.  

![image](https://github.com/user-attachments/assets/d8d6d214-8479-4388-bd22-3997514bf49e)  

And lastly here, we can run a SHOW TABLES command to check the status of search optimization and its progress. This shows the percentage of the table that has been optimized so far.  

#### Types of tables : 

![{E267A5DC-3989-4108-A69B-CE768B92C9AA}](https://github.com/user-attachments/assets/39934e0a-b80e-44f8-be59-b25af6a7302b)  

**Note that :**
Since micro-partitions cannot be changed, Snowflake does NOT physically move data between existing partitions. Instead, it:

- Reads the existing micro-partitions.
- Sorts the data based on the clustering key.
- Creates new micro-partitions that follow the new order.
- Marks the old micro-partitions as obsolete (logical deletion).

**Snowflake suggests sometimes clustering improvements (via SHOW CLUSTERING INFORMATION).**  

https://docs.snowflake.com/en/user-guide/tables-micro-partitions

## Data Loading and unloading :  

There are various methods of data ingress and egress in Snowflake, we have bulk data loading via the copy to table statement, continuous data loading with the serverless feature, Snowpipe, and data unloading with the copy to location command.  

![image](https://github.com/user-attachments/assets/9244dad7-f297-4de9-929f-75da5f3a42a3)  

### Stages: 
Stages are so important to hold row data before loading it into tables. There are two types of stages in snowflake. External and internal storages.  

![image](https://github.com/user-attachments/assets/1c8886ef-aa39-4d22-a37c-cf79b56793d9)  

#### Internal stages: 
Internal stages are a storage handeled by snowflake where we can load data temporarily before loading it into a table. We have 3 types of internal stages, two of them are by default available and the third is created manually:  

![image](https://github.com/user-attachments/assets/365d9fa0-a2ff-4a68-802d-e7e0c572d5cb)  

**Note that Uncompressed files are automatically compressed using GZIP when loaded into an internal stage, unless explicitly set not to.**  

1️⃣ User Stage:  
🔹 What is it?
- A personal staging area for each Snowflake user.
- Automatically available—each user has their own private storage space.
- Can only be accessed by the specific user.

🔹 Use Case: 
✅ Storing temporary files before loading them into tables.  
✅ Uploading small datasets for personal testing.  

🔹 How to Use It:  
📤 Upload a file to your user stage:  

```sql
PUT file://path/to/data.csv @~;
```

📥 Check files in your user stage:  

``` sql
LIST @~;
```

📤 Load data from user stage into a table:

```sql
COPY INTO my_table FROM @~ FILE_FORMAT = (TYPE = 'CSV');
```

📌 **The @~ refers to the current user's stage.**

2️⃣ Table Stage  

🔹 What is it?  

- A staging area tied to a specific table.
- Each table in Snowflake automatically has its own internal stage.
- Only accessible by users with privileges on the table.

🔹 Use Case:  

✅ Storing files specific to a table before inserting data.  
✅ Automating data loads for a table without needing external storage.  

🔹 How to Use It: 

📤 Upload a file to the table stage: 

```sql
PUT file://path/to/data.csv @%my_table;
```
📥 Check files in the table stage:

```sql
LIST @%my_table;
```
📤 Load data from the table stage:

```sql
COPY INTO my_table FROM @%my_table FILE_FORMAT = (TYPE = 'CSV');
```

📌 **The @%my_table refers to the stage of the table my_table.**

3️⃣ Named Stage  

🔹 What is it?  

- A manually created internal stage that can be shared across multiple users and tables.
- Unlike user or table stages, it is explicitly created.
- Provides more flexibility and control over data storage.

🔹 Use Case:  
✅ Centralized storage for multiple tables or shared data.  
✅ Organizing files before processing in Snowflake.  
✅ Better access control—you can grant privileges on the stage.  

🔹 How to Create & Use It:  
📌 Create a named stage:  

```sql
CREATE STAGE my_stage;
```

📤 Upload a file to the named stage:

```sql
PUT file://path/to/data.csv @my_stage;
```

📥 Check files in the named stage:

```sql
LIST @my_stage;
```

📤 Load data from the named stage into a table:

```sql
COPY INTO my_table FROM @my_stage FILE_FORMAT = (TYPE = 'CSV');
```

📌 **Named stages must be explicitly created, unlike user and table stages.**  

Note that PUT is not supported in the snowsight UI worksheets but in the SnoSQL CLI :  

![image](https://github.com/user-attachments/assets/64dd401d-c3a9-45e2-91d4-39234ccaa126)  

![image](https://github.com/user-attachments/assets/f94620fa-4e03-4049-9026-53292beef050)  

#### Exteranal Stages: 
External stages are of named stage type only !:  

![image](https://github.com/user-attachments/assets/46791e0d-4c62-41aa-8e14-82fd56c78074)  

Similarly to synapse, when connecting ro external data we need create external data source or in this case external stage !  but to connect, we need also to create a **Storage integration** that will authenticate to the source like ADLS gen2 or we can use directly SAS token : 

![{D8B3BEB9-0989-476A-928C-D96009A78328}](https://github.com/user-attachments/assets/4098ac76-a196-4761-83b0-13d32145f160)  

![{3D381A18-9A18-4E0D-BA16-AA88F14E4AC9}](https://github.com/user-attachments/assets/e0957fa8-32a6-4fc2-99c2-c4fb46f4eff5)  

Once created we will need to consent for a first time using the consent url : 

![{AFF3D708-D8F4-447C-80E0-1D74F62CFD55}](https://github.com/user-attachments/assets/198ac549-c2a0-4f54-bce6-5cdc952e58f8)  

to create the storage integration : 

```SQL
                    create storage integration <storage_integration_name>
                        type = external_stage
                        storage_provider = azure
                        azure_tenant_id = '<azure_tenant_id>'
                        enabled = true
                        storage_allowed_locations = ( 'azure://<location1>', 'azure://<location2>' )
                        -- storage_blocked_locations = ( 'azure://<location1>', 'azure://<location2>' )
                        -- comment = '<comment>';
```

![image](https://github.com/user-attachments/assets/45a890a3-4211-43f0-8f07-d16e832764e2)  

### Data Loading :  

Data can be loaded using several ways:  
- Insert Method (using select,value, overwrite ..)
- Copy into using the UI and code
- PUT

#### Copy Into :

Copy into is one of the important functions to load data into tables from stages.  

![image](https://github.com/user-attachments/assets/e9cd4988-0c35-41b8-9866-4bae7c52077e)  

We can specify what exactly to copy using some parameters :  

![image](https://github.com/user-attachments/assets/d8334e06-5baa-4bfb-8a1f-0418127953d4)  

We can also perform some transormations while loading :  

![image](https://github.com/user-attachments/assets/c1a7548c-abcb-4080-8c26-665108f3e685)  

![image](https://github.com/user-attachments/assets/4fd3ee78-a742-4772-a08f-8676ae9de770)  

We have also some options with the copy into we can use :  

![image](https://github.com/user-attachments/assets/5a100795-c8cd-4a74-9b86-dada0fc8330f)  

Also a good practice would be to set some validation parameters to be sure that the copy into will go well **Before loading** ==> Dry Run:  

![image](https://github.com/user-attachments/assets/9dbf25e6-7d0c-4846-a60d-b44abd1d4d16)  

If we encounter an error while testing the loading we can get more details about that using the VALIDATE function :  

```SQL
SELECT * FROM VALIDATE(table, job_id=>'id_of_the_dry_run_query') 
```
This is a great way to debug errors that we would encounter if we try to load data !  

These methods don't support COPY INTO <table> commands

#### File Formats:  

File formats are useful to tell snowflake how to parse the files used in the copy into statement :  

![image](https://github.com/user-attachments/assets/76cc7277-36a6-4dff-9e85-c5c5fd62db0e)  

We can set the file format at the COPY INTO statement, at the TABLE level using ALTER TABLE *** SET STAGE_FILE_FORMAT=(FORMAT_NAME='CSV_FILE_FORMAT') or at the stage level using :  ALTER STAGE *** SET FILE_FORMAT = CSV_FILE_FORMAT. For the latter two ways we can use COPY INTO directly with no file format to specify as it is already attached to the table or the stage.  

Snowflake recommend compressing the file stored in a stage, whether that be an external or internal stage.  

![image](https://github.com/user-attachments/assets/b353e244-0563-43ee-8002-23691ff1c5c1)  


#### SnowPipe:  

Snowpipe is a serverless feature that allows to manually loand data into a table whenever a new file arrives at the stage level especially if we deal with streaming data :  

![image](https://github.com/user-attachments/assets/d4015303-e8b6-4d31-8ef1-3d49c61aa7d6) 

AUTO_INGEST = TRUE : 
this method involves configuring cloud blob storage like an S3 bucket to send Snowflake a notification telling it that a new file has been uploaded. This will act as a trigger for Snowflake to go ahead and execute the copy into statement defined in the pipe. This method only works with external stages.  

![image](https://github.com/user-attachments/assets/65255a95-3ee3-4998-bbbe-e5acfc16ef91)  

AUTO_INGEST = FALSE : 

If on the other hand you set auto ingest a false, you're telling Snowflake that you'll let them know yourself when a new file is uploaded via a call to a Snowflake resting point. This method works both with internal and external stages.  

In this scenario we make use of a Snowflake rest endpoint to notify a pipe that a file has been uploaded to a stage. This applies to both internal and external stages. Client applications for which there are Java and Pythons SDKs provided can call Snowflake via a public insert files rest endpoint providing a list of file names that were uploaded to the stage along with a reference to a pipe.  
For the files that matched the list provided in the API call, Snowflake provided compute resources will execute the copy into table command in the pipe definition to populate the target table.  

![image](https://github.com/user-attachments/assets/fdf26333-968c-492f-8cc2-4a1c64b86c4b)  

Some key differences between Snowpipes and Bulk Loading :  

![image](https://github.com/user-attachments/assets/fc2a0414-b0f7-4840-8606-0896c9c622da)  

best practices for data loading to keep costs down and ensure good performance. These recommendations apply to both bulk loading and continuous data loading with SnowPipe. We should split large files into multiple files around a hundred to 250 megabytes of compressed data. This is because a single server in a warehouse can process up to eight files in parallel. So providing one large file will not utilize the whole warehouse's capacity. Files exceeding a hundred gigabytes are not recommended to be loaded and should be split.  

The reverse is also true if we have many files which are very small, for example, messages of 10 kilobytes, it would be a good idea to bundle these up into larger files to avoid including too much processing overhead.  

![image](https://github.com/user-attachments/assets/18075501-07e4-4ff4-acac-e82b7bfe4f18)  

Organizing stage data has the benefit of creating a natural order of data, which after copying into Snowflake, translates into micro partitions that are more evenly distributed and therefore have the potential for improved pruning.  

As SnowPipe aims to load a file within one minute, it's recommended not to upload files at a shorter frequency than that. If we were to load very small files more often than once per minute, SnowPipe would not be able to keep up with the processing overheard and the files would back up in the queue in current costs.  

### Data Unloading:  

Data unloading is the process of getting data out of snowflake.  

![image](https://github.com/user-attachments/assets/473acbdb-c21e-473a-ba20-23c99091e879)  

We can use for that two methods:  COPY INTO LOCATION and GET method.  

![image](https://github.com/user-attachments/assets/4bae4029-da71-48cf-bddb-9bfb8dd9d463)  

The COPY INTO will export data to the external stage and then we can use GET (works only in the CLI) to download the files to the local file system.  

![image](https://github.com/user-attachments/assets/1f3d2427-ed11-4274-9593-384706d069e7)  

Data files unloaded to cloud storage can be encrypted if a security key is provided to Snowflake.  

![image](https://github.com/user-attachments/assets/65baa6bc-8264-4cad-98b7-69bacbb5a618)  

Some options can be used with the COPY INTO LOCATION :  

![image](https://github.com/user-attachments/assets/57b072b9-a4da-4690-ae33-c26abf1ac180)  

The GET command is essentially the reverse of PUT, you specify a source stage in a target local directory to download the file to. Typically, this command is executed after using the COPT INTO location command. GET cannot be used for external stages. You would access those via their own utilities like the AWS console or CLI. GET can only be used with internal stages.  

![image](https://github.com/user-attachments/assets/98d2a5d4-28d5-4c43-94e1-9df3b26e4b86)  

### Loading semi-structured data:  

Snowflake has extended the standard SQL language to include purpose-built, semi-structured data types. These allow us to store semi-structured data inside a relational table.  

![image](https://github.com/user-attachments/assets/1637582f-c135-477b-82aa-2efb87dadd5d)  

#### Data types 

Semi structured data have several types :  

- Array:

![image](https://github.com/user-attachments/assets/934b9707-096b-4b75-9919-cbd74b178d67)  

- Object :

![image](https://github.com/user-attachments/assets/3e7c5fde-2827-4dbe-8ec4-29aa1a7fd2eb)  

- Variant:

The basic idea with the variant is that it can hold any other data type, objects, arrays, numbers, anything. This is why you might see it referred to as a universal semi-structured data type in the documentation. Objects and arrays are really just restrictions of the variant type.  
It's primarily used for storing whole, semi-structured data files.  

![image](https://github.com/user-attachments/assets/3f5173d1-b87c-4a14-b69f-2a0cb9a4a721)  

Variant data type can hold up to 16 megabytes of compressed data per row, so if we're loading a large file into a variant column that exceeds that threshold, an error will be thrown. we'll either have to split that JSON file prior to loading it or use a file format option to separate the JSON objects in a file so they're loaded as separate rows.  

#### Semi strcutured file formats :  

Snowflake have taken a few of the most popular semi-structured file formats and figured out a way to load these into their storage, intelligently extracting columns from their complex structures and transforming them into Snowflake's column and file format.  

This is what we mean by natively supporting semi-structured data.  

![image](https://github.com/user-attachments/assets/7e1f71eb-f7ae-48b4-9981-4696d0b20a30)  

![image](https://github.com/user-attachments/assets/629dde2d-8477-4757-997b-57ab5051351f)  

![image](https://github.com/user-attachments/assets/fd29579f-b1cb-4b1f-b570-b14a941f18a0)  

Strip outer array is quite an important option, as it's a very convenient way to bypass the 16 megabyte limit on a variant column during data loading, The top level elements in a JSON file are generally listed in an array. so by setting this option to true the array is removed and each JSON object is stored as a separate row.  

Loading semi structured data can be done using 3 ways :  

![image](https://github.com/user-attachments/assets/604121d4-e7d8-4259-a207-713b51637d41)  

The unloading is similar to the structured data :  

![image](https://github.com/user-attachments/assets/4ee2a579-81e2-4ba9-9098-7138ae7e01aa)  

#### Accessing semi structured data :  

Several ways to access semi structured data :  

![image](https://github.com/user-attachments/assets/9210bbb0-f572-4bd5-9136-e167c037a9b3)  

#### Casting semi structured data :  

Casting can be done using 3 methods :  

![image](https://github.com/user-attachments/assets/fdd5e096-e37a-43bd-9fbe-9fb1cdc8ae91)  

The AS datatype function is the same in terms of functionality to the TO datatype function. However, it's only used for converting values from a variant column.

#### Semi structured data options :  

![image](https://github.com/user-attachments/assets/747c6c8a-77db-4718-81d8-98530bc1d3a2)  

- Flatten function :

FLATTEN() is a table function in Snowflake that explodes semi-structured data (arrays, objects, or variants) into multiple rows.  
👉 It takes nested data (JSON, XML, ARRAY, OBJECT, VARIANT, etc.) and returns each element as a separate row.

FLATTEN() expects a single input, but when using a subquery without LATERAL, it can return multiple rows, which Snowflake doesn’t allow in this context.  

LATERAL on the other hand ensures FLATTEN() processes each row individually. It's like performing a join to apply flatten to each row in the table !  

![image](https://github.com/user-attachments/assets/d0635ec5-5b06-482e-b2fe-cddf1b18199b)  

![image](https://github.com/user-attachments/assets/1fb0d95a-a40e-4f5e-93fd-6b40fa0da399)  

![image](https://github.com/user-attachments/assets/1d9c0850-f2ed-4237-824b-b5f1e6e7b7ee)

```SQL
SELECT jt.id, f.value AS hobby
FROM json_table jt,
LATERAL FLATTEN(input => jt.data:hobbies) f;
```

### Data Transformations :  

Several function are supported in snowflake :  

![image](https://github.com/user-attachments/assets/bb20af61-bcf9-44ee-8e89-58b9484ada81)  

#### Scalar functions :  

![image](https://github.com/user-attachments/assets/1b0eb107-4469-4b2e-9423-bdf9e001cb84)  

Example :  

![image](https://github.com/user-attachments/assets/7ebd3bb0-635d-49c1-afc8-640a5bab61ba)  

#### Aggregate functions :  

![image](https://github.com/user-attachments/assets/2e1dac6e-a76e-4467-b2ee-d12abeb7b226)  

- Estimation Functions :

![image](https://github.com/user-attachments/assets/34e6b571-fb25-4ec9-8a67-cf6d24151574)  

why use an estimation function if we already have the distinct keyword, which when used in conjunction with count, will give us an accurate and exact output of the number of distinct values in a column? Executing a count distinct operation requires an amount of memory proportional to the cardinality, which could be quite costly for very large tables. So Snowflake have implemented something called the HyperLogLog cardinality estimation algorithm. This returns an approximation of the distinct number of values of a column. The main benefit being that it consumes significantly less memory and is therefore suitable for when input is quite large.  

![image](https://github.com/user-attachments/assets/0338561f-2102-4f26-b118-02097fb935b6)  

![image](https://github.com/user-attachments/assets/268943d6-f51b-40de-b1c7-83fab8af0d25)  

Comparision between distinct(count(*)) and HHL():  

![image](https://github.com/user-attachments/assets/991006fd-8578-4680-86f5-110a9d5dd6ca)  

![image](https://github.com/user-attachments/assets/f43e7e9a-f640-4df4-9959-2489b246aa30)  

![image](https://github.com/user-attachments/assets/2ac58e12-aad8-434d-8e35-2b87045f5b8e)  

MINHASH two input parameters, K and an expression.

K determines how many hash values we like to be calculated on the input rows. The higher this number, the more accurate an estimation can be. But bear in mind, increasing this number also increases the computation time.The max value you can set is 1024.  

![image](https://github.com/user-attachments/assets/dbae0efb-445d-4d51-8a5a-2d40d06bb1fc)  

- Frequency estimation :

![image](https://github.com/user-attachments/assets/cdef6e79-22d7-4883-ba1a-7f09a4eda238)  

![image](https://github.com/user-attachments/assets/93ca43a8-3ba7-4806-a6bd-b35c9341c814)  


This function has three input parameters. The column we like to calculate the frequency of values for, the number of values we'd like to be approximated, and the maximum number of distinct values that can be tracked at a time during the estimation process. Increasing this max number makes the estimation more accurate, and in theory uses more memory. Although setting it to the max still has good performance.  

If we want exact value :  

![image](https://github.com/user-attachments/assets/665e909a-0d38-40b2-abc4-814df07d69d5)  

- Percentile estimation :

![image](https://github.com/user-attachments/assets/19a3222a-440c-414f-9581-dfa22ec8a9b5)  

Example :  

![image](https://github.com/user-attachments/assets/b193c54c-2823-4540-9d3b-ec1e3d2dfb93)  

what score we'd have to achieve to get as good or better than 80% of other students,we'd plug in 0.8. For this group of 10 students, we'd have to achieve at least 74 to be at the 80th percentile.  

#### Windows functions :  

![image](https://github.com/user-attachments/assets/f24d2c79-fa64-4f1c-976f-52f628b64d35)  

#### Table functions : 

![image](https://github.com/user-attachments/assets/6c2303e5-1e98-48cf-a69a-6711d1ee345b)  

Example :  

![image](https://github.com/user-attachments/assets/d3d88c13-fd1a-4636-bffc-e22c041124de)  

Note that to query all table functions, we wrape them in TABLE().  

#### System functions :  

![image](https://github.com/user-attachments/assets/7ce1945a-a75d-4f2f-aeff-dc140b864b30)  

![image](https://github.com/user-attachments/assets/79a13b05-5549-49a4-8d72-0c772b5f3381)  

![image](https://github.com/user-attachments/assets/16b6d5f7-b94a-4936-92f0-c7806f698407)  

#### Table Sampling :  

These function are useful to choose the sample of data to work with by giving probabilities to each row or block of data to apear in the final sample:  

![image](https://github.com/user-attachments/assets/b8f1fe43-b57c-4421-b38d-2d25e612fa2e)  

The above methods generate random size but the bellow can generate a fixed size:  

![image](https://github.com/user-attachments/assets/94eb51c0-902e-4de6-8836-e2ae22d18e7d)  

### Unstructured data :  

How snowflake handels unstructured data ?  

![image](https://github.com/user-attachments/assets/2e46c044-23eb-49c6-aee6-463174ea426d)  

6 functions for this type of data :  

![image](https://github.com/user-attachments/assets/995f1710-780a-4272-96f1-903e1f1af25e)  

From an unstructured data we use these functions to generate URLS:  

![image](https://github.com/user-attachments/assets/a47fa0eb-af67-4d82-a2b3-9cca3353ce37)  

Then querying it like the following can give  access to the file :  

![image](https://github.com/user-attachments/assets/660e52a3-e74b-449d-8317-2bf0e17992dc)  

Note that this method generates a URL with a time limit of 24 hours.  

If we want to give access to other users to these URLS we can use them in a view :  

![image](https://github.com/user-attachments/assets/e5d4ef56-473b-425f-9d91-605a317b0a5c)  


We can produce permenant URLS using another function :  

![image](https://github.com/user-attachments/assets/3c927c2d-8021-477b-94e9-bd7ed385c62b)  

The encryption part needed if the access to the URLs gives an error like corrupted files meaning that the files behined are encrypted !  

- BUILD_SCOPED_FILE_URL :
  
  - Purpose: Generates a URL for accessing a file stored in a Snowflake internal stage.  
  - Use Case: This function is used when you want to create a URL that provides direct access to files stored in an internal stage within Snowflake. It's more commonly used when dealing with Snowflake's internal file storage and doesn't work with external cloud storage services (such as Amazon S3 or Azure Blob).  

- BUILD_STAGE_FILE_URL :  

  - Purpose: Generates a URL for accessing a file stored in an external stage (like Amazon S3, Azure Blob Storage, or Google Cloud Storage).

  - Use Case: This function is typically used for external stages and creates a URL for accessing the file from the cloud service (such as an S3 bucket). Unlike GET_PRESIGNED_URL, which generates a temporary presigned URL, BUILD_STAGE_FILE_URL provides a regular URL (not presigned) to the external file.

- GET_PRESIGNED_URL : 
  - Unlike the above methods that needs to be signd in to snowflake to use the URLs, the GET_PRESIGNED_URL method generate a presigned URL that allows temporary, secure access to a file stored in an external stage (e.g., Amazon S3, Microsoft Azure Blob Storage, or Google Cloud Storage) without requiring direct Snowflake credentials.  

![image](https://github.com/user-attachments/assets/42903c71-6df2-4edf-b171-a30da592adfe)  
 

Example :  

![image](https://github.com/user-attachments/assets/95d3e12f-696f-4e47-b753-66a890611c4d)  

![image](https://github.com/user-attachments/assets/6d18c82a-88e1-4938-9cd7-0a2a198ccf07)  

![image](https://github.com/user-attachments/assets/2b7cabdb-1e50-420d-b0db-b7c85025d469)  

#### Directory Table :

There's simply an interface we can query to retrieve metadata about the files residing in a stage. Their output is somewhat similar to the list function we reviewed when looking at stages. And here is the syntax for querying a stage with a directory table enabled, Select star from directory, parentheses, and stage name. And unlike the list command, it includes an additional field relevant to unstructured data access called FILE_URL. This is a Snowflake hosted file URL identical to the URL you'd get if you executed the build stage file URL function. We also have additional information like the file size, the file hash, and the last modified time of a stage file.  

![image](https://github.com/user-attachments/assets/84e118b5-870d-4b53-8ce1-536e547819d4)  

Manual refresh of Directory Tables :  

![image](https://github.com/user-attachments/assets/ae048eae-9f70-4de1-9c0b-a103421b5b91)  

Automated refresh of Directory Tables :

Relevant for external stages only, is to set up automated refreshers. This is done by setting up event notifications in the cloud provider in which your stage is hosted.  

![image](https://github.com/user-attachments/assets/7a301db5-8d6a-4360-8605-11ef8279cbc6)  

#### File support REST API :  

File support REST API allows us to download data files from either an internal stage or an external stage programmatically, ideal for building custom applications, like a custom dashboard that renders product images, for example.  

![image](https://github.com/user-attachments/assets/882c1f05-a9c6-4d18-af39-c32c11ff5d2d)  

USAGE Role needed for External stage and READ Role for internal stage.  

Example :  

![image](https://github.com/user-attachments/assets/ccbcc7d8-c034-4e2a-a0d9-b2e338756206)  

Here we hardcoded the value, but we could do something like use the SQL REST API to make a call to a stored procedure that generates the URL for us.  


## Storage, Data Protection and data Sharing :  

The data storage part is so important and its understanding makes it much easier for us to manage the optimizations.  

Data are stored behind the scene in the cloud provider storage servive we choosed initially when we created the snowflake account. Once the data are loaded we can no longer access it ! only using the SQL commands.  

When loading data into snowflake, this latter creates micropartitions in a propriotary format similar to parquet files.  

By default data are partitioned based on the order of insertion.  

![{AED5469A-8E1F-4DF2-9685-8882EF5A896A}](https://github.com/user-attachments/assets/c99dea94-34ec-4e59-9ca2-c5c94a136769)  

This column storage is best for reading data faster thanks to data prunning capabilities ! The prunning is the fact that snowflake uses the micropartitions metadata (min, max ..) to filter data !  

![{37526445-731C-4739-9E2B-F354B8F525C2}](https://github.com/user-attachments/assets/31de619a-6668-4a76-a801-c7b78b98459d)  

That's why also the min, max and row count do not need warehouses to be computed !  

![{B1A7761C-0A7A-4E84-B2C8-166215BFAC5F}](https://github.com/user-attachments/assets/bc7d584c-dd24-474a-abfb-2e650dbe9f96)  


### Data lifecycle : 

Snowflake cares alot about the data loss situations, hence it added some features to prevent that !  

![{BFDBD494-5F81-4B7E-9012-AA26453D4D3A}](https://github.com/user-attachments/assets/ed56d9a2-cf43-4f09-b748-54aef301daed)  

#### Time travel :  

Time travel is a feature that keeps track of objects such as schemas, tables and databases so that the user can restore them id needed afer removal.  

![{FF6080B7-6E9E-40A2-877B-33C29B249D5F}](https://github.com/user-attachments/assets/31761313-4beb-490d-bccf-acc7fad76a17)  

Time travel has a retention period to be set :  


this follows the objects hierarchy in snowflake meaning that if we set 90 days retention period on the database it would be the same for the schemas and tables under that database unless we specify a different retention period in the elementary object !  

![{733865BA-AFDC-4929-BA47-965E936BC122}](https://github.com/user-attachments/assets/77ba1d98-6ee4-447a-890b-c772a80c4a7c)  

To use time travel we need to follow 3 sql queries : AT, BEFORE and UNDROP  

![{1C093DA9-29EA-4391-B562-F71CE2FF8798}](https://github.com/user-attachments/assets/31d7af36-1889-4dc4-9942-1c96898a5c5f)  


#### Fail Safe :  

There is another layer that keeps the removed data even if we exceed the time travel ! but this feature is completely managed by snowflake :  

![{95B7E1EE-9BD6-49D4-A031-AF188A9569B6}](https://github.com/user-attachments/assets/9a60c42e-99a9-4b47-aa5f-31e0399cdf2e)  

#### Cloning :  

Cloning is feature that creates duplicates of snowflake objects without the need to duplicate data. The two objects have seperate but similar architecture and the share the same data (Metadata files pointing to the data). Only new date or updated ones are specific to each copy meaning the if we update data using the cloned table for example new partitions will be created and will only be linked to this table :  

![{4CE4D411-5C51-49E6-A9BB-18B12BBFA230}](https://github.com/user-attachments/assets/1103a20e-061e-4eb7-87bd-426f9534424f)  

Note that cloning databases and schemas is recursive and clones subobjects such as tables, views ... even if we can't directly clone views.  

![{B3E1C772-D2BE-49B1-88FC-EF936F7E9554}](https://github.com/user-attachments/assets/d780a76d-e245-4854-86ac-73891107abd7)  

The are however some limitations of cloning :  

![{6383C85A-C6F9-4DE1-8307-AC0E8442E0FD}](https://github.com/user-attachments/assets/046feb91-92f0-4324-ab31-889b7c87e231)  

With this said, cloned tables do not keep track of loaded files and can lead to loading the same file twice and having dupplicated data !  
We can use alos cloning with the time travel feature :  

![{CC3C0DD1-5E0F-4D4F-B81F-ED711BE2CCA6}](https://github.com/user-attachments/assets/3f6dc90f-b059-489d-a843-8af8958d8798)  

#### Replication :  

It happens between accounts and it moves data from an account to another :  

![{02770500-F7C9-4E60-974E-9BE7DFE6F420}](https://github.com/user-attachments/assets/ba748beb-7a48-4c51-a72a-9dc556f30edc)  


![{B4D880A6-B111-4A72-AA5B-06DDC1E90A7B}](https://github.com/user-attachments/assets/a5d3d72a-8f8e-47e4-a258-1bc0d9f664a2)  


#### Billing storage :  

Snowflake calculate a daily average storage and by the end of the month another monthly average to estimate the billing :  

![{9DFD4379-1267-4CFC-8ADA-D76DEDE155F5}](https://github.com/user-attachments/assets/de271a0c-03f9-4e5a-bc78-df49eca6c007)  

#### Secure Data sharing :  

This is a powerful feature that makes it safer to share data inside or outside the company :  

![{203D115E-CB5D-4305-8FDA-445D5BD03D68}](https://github.com/user-attachments/assets/89c2ae3b-30b0-4663-965b-ec242f6b5dd6)  

The consumer will only pay for the computing resources.  

What can we share :  

![{76176EF8-1BEF-4EEA-8BD5-34AA47FCE29E}](https://github.com/user-attachments/assets/1c9031aa-adee-4299-b4df-f804f482262f)  

![{E2E26B16-E22E-4D46-B264-0A6BE543D541}](https://github.com/user-attachments/assets/fad2730f-fd24-4cbc-880b-16012986955f)  

Any new object created within a shared database will not automatically be added to a share. We have to explicitly add the grant to a share to make it accessible to consumers. Future grants cannot be used in shares.  

![{9834A3DE-1968-4DFE-AD28-87E69EDBFB86}](https://github.com/user-attachments/assets/0acff8a5-e7ef-4bdf-b05f-752affdb3cc2)  

![{A1DD30CF-D281-445A-AA8A-D05566010AF8}](https://github.com/user-attachments/assets/36d4f3c2-418d-4322-a8de-397bbcb8582d)  

If a provider would like to share with an account outside of its region, for example, it would involve replicating the database we'd like to share into an account within the same region as the consumer account and then creating a share from the replicated database.  

![{99B25657-6C3D-4336-9410-6C434A5A7252}](https://github.com/user-attachments/assets/86538aa1-0792-4afb-b338-390fdb99effb)  

![{E03DCD0B-6141-4FFB-8757-FEED5215A3CD}](https://github.com/user-attachments/assets/220ae99a-8bfb-4d71-b038-68512b9fff60)  

We can also create reader accounts. This type of account is like a limited version of a standard account whose sole purpose is to provide access to database objects from the data provider to a non Snowflake user. They wouldn't have to enter an agreement with Snowflake. They're simply provided an account URL, username and password and can query the shared data as if they did have a full account.  

![{E1093C0B-FFC8-4D77-9576-4FA9D77F9543}](https://github.com/user-attachments/assets/07954862-56c3-49ed-b48c-d27b862b064f)  

#### Data Exchange :  

So what if we'd like to share our data publicly but only to a specific group of accounts? nA data exchange effectively allows us to create a private version of a data marketplace on the Snowsight UI, and invite other accounts to provide and consume data within it. That could be other departments in a company, suppliers, any trusted party that wants to produce or consume data.  

We can set up a data exchange by contacting Snowflake Support and providing a name and description of the exchange. The data exchange feature is currently not available for VPS editions of Snowflake.  


![{9D5C6FC7-9912-434B-852C-5F0C229C4EAD}](https://github.com/user-attachments/assets/3e1d7def-2933-452e-a34e-1f3f935afa34)  

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

### Query Cashing :

Snowflake has a result cache that holds the results of every query executed in the past 24 hours. These are available across warehouses, so query results returned to one user are available to any other user on the system who executes the same query, provided the underlying data has not changed. Not only do these repeated queries return extremely fast, but they also use no compute credits.

### Clonning:

When we create a cloned table in Snowflake:
- The clone initially shares the same metadata and storage as the original table. No additional storage is consumed until changes are made to either the clone or the original.
- Changes (e.g., deletes, inserts, updates) made to the clone are independent of the original table, and vice versa.
This behavior is achieved through Snowflake's zero-copy cloning:
- Snowflake uses metadata pointers to the original table's data.
- When you modify data in the clone, only the modified data is stored separately (copy-on-write mechanism).

```SQL
                    -- Create the original table
                    CREATE TABLE original_table (
                        id INT,
                        name STRING
                    );
                    
                    -- Insert some data
                    INSERT INTO original_table VALUES (1, 'Alice'), (2, 'Bob');
                    
                    -- Create a clone of the original table
                    CREATE TABLE cloned_table CLONE original_table;
                    
                    -- Delete data from the clone
                    DELETE FROM cloned_table WHERE id = 1;
                    
                    -- Check data in the clone
                    SELECT * FROM cloned_table;
                    -- Output: Only the record with id = 2 remains
                    
                    -- Check data in the original table
                    SELECT * FROM original_table;
                    -- Output: Both records (1, 'Alice' and 2, 'Bob') are still present
```

### Time Travel:

Snowflake's powerful Time Travel feature enables accessing historical data, as well as the objects storing the data, at any point within a period of time. The default window is 24 hours and, if you are using Snowflake Enterprise Edition, can be increased up to 90 days.  
Some useful applications include:
- Restoring data-related objects such as tables, schemas, and databases that may have been deleted.
- Duplicating and backing up data from key points in the past.
- Analyzing data usage and manipulation over specified periods of time.

We should pay attention to the settings of the timezone !! we can check the current timezone using :

```SQL
                    show parameters like '%timezone%';
                    ALTER SESSION SET TIMEZONE = 'Europe/Paris';
```

```SQL
                    -- Set the session variable for the query_id
                    SET query_id = (
                      SELECT query_id
                      FROM TABLE(information_schema.query_history_by_session(result_limit=>5))
                      WHERE query_text LIKE 'UPDATE%'
                      ORDER BY start_time DESC
                      LIMIT 1
                    );
                    
                    -- Use the session variable with the identifier syntax (e.g., $query_id)
                    CREATE OR REPLACE TABLE company_metadata AS
                    SELECT *
                    FROM company_metadata
                    BEFORE (STATEMENT => $query_id);
                    
                    -- Verify the company names have been restored
                    SELECT *
                    FROM company_metadata;
```

