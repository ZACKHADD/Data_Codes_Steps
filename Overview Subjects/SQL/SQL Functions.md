# SQL : Functions, Optimization & Code

The present file contain several key notes regarding SQL and how to use its functions in different areas going from data analysis to data engineering. It also contains some optimization aspects and codes to reuse.  

## Functions:

SQL has a lot of functiond of diffrent types depending on the objectif of the operation we are performing. The following diagram shows **the main** functions (their are many others) and their use.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/bfbaa8e0-40b2-4d46-98ef-c1f2763dfed2)  

### Data Definition Language (DDL):

- CREATE: Commands to create databases, tables, and views.  
- ALTER: Modify existing database objects, such as tables. This includes adding or removing columns and constraints.  
- DROP: Delete tables and databases.  
- TRUNCATE: Remove all records from a table while keeping its structure.  

### Data Manipulation Language (DML):

- SELECT: Retrieve data from the database.  
- INSERT: Add new data to a table.  
- UPDATE: Modify existing data in a table.  
- DELETE: Remove data from a table.  

### Data Control Language (DCL):

- GRANT: Give users access privileges.  
- REVOKE: Remove users' access privileges.  

### Transaction Control Language (TCL):

- COMMIT: Save changes made during a transaction.  
- ROLLBACK: Undo changes made during a transaction.  
- SAVEPOINT: Set a point within a transaction to which you can roll back.  

### Functions:

- Aggregate Functions: Perform calculations on a set of values and return a single value (AVG, MAX, MIN, SUM, COUNT).  
- Window Functions: Perform calculations across a set of table rows related to the current row (ROW_NUMBER, RANK, DENSE_RANK, PERCENT_RANK, NTILE, LAG, LEAD, FIRST_VALUE, LAST_VALUE, NTH_VALUE). They will partition the table and performe the needed action.
*Note that window functions need OVER and PARTITION BY clause tu run. Also we can add ORDER BY clause that will simply order data inside each partition.*
Example:
```SQL
              SELECT start_terminal,
                   duration_seconds,
                   SUM(duration_seconds) OVER
                     (PARTITION BY start_terminal ORDER BY start_time)
                     AS running_total
              FROM tutorial.dc_bikeshare_q1_2012
             WHERE start_time < '2012-01-08'
  ```

### JOIN Operations:

- INNER JOIN: Select records that have matching values in both tables.  
- LEFT JOIN: Select all records from the left table and the matched records from the right table.  
- RIGHT JOIN: Select all records from the right table and the matched records from the left table.  
- FULL JOIN: Select all records when there is a match in either left or right table.  

### WHERE Clause:

- Filter records based on specified conditions (AND, OR, NOT, BETWEEN, LIKE, IN, ANY, ALL, EXISTS).

### Qualify:

In a SELECT statement, the QUALIFY clause filters the results of window functions.  
QUALIFY does with window functions what HAVING does with aggregate functions and GROUP BY clauses.  

```SQL
                SELECT * 
                    FROM (
                         SELECT i, p, o, 
                                ROW_NUMBER() OVER (PARTITION BY p ORDER BY o) AS row_num
                            FROM qt
                        )
                    WHERE row_num = 1
                    ;
                +---+---+---+---------+
                | I | P | O | ROW_NUM |
                |---+---+---+---------|
                | 1 | A | 1 |       1 |
                | 3 | B | 1 |       1 |
                +---+---+---+---------+

                 -------------------------------------------------------------
              --With Qualify clause

              SELECT i, p, o
                        FROM qt
                        QUALIFY ROW_NUMBER() OVER (PARTITION BY p ORDER BY o) = 1
                        ;
                    +---+---+---+
                    | I | P | O |
                    |---+---+---|
                    | 1 | A | 1 |
                    | 3 | B | 1 |
                    +---+---+---+
```
More on : https://docs.snowflake.com/fr/sql-reference/constructs/qualify

## Optimization:

SQL is all about writing data properly (ACID) and reading it as fast as possible. For these reasons, optimization is a must.  
It can be done at several levels : Queries, tables definition, database ...  

### Query Optimization:
When runing queries against our data, we should focus more on how the engine would perform what we are asking it. This is called **"The query execution plan"**.  
It represents the most optimized path that the SQL engine found and will follow. Of course, this is so affected by how we write our queries.  
To help making the queries more optimized, several properties are needed in our query definition:  
- SARGability: SARG means **Search Argument**, Which simply indicates whether or not the engine can retrieve data directly from an index without any transformation needed.
```SQL
        SELECT COUNT(id)
        FROM dbo.Posts
        WHERE CONVERT(CHAR(10),CreationDate,121) = '2023-05-10'
        
        -------------------------------------------------------------
        
        SELECT COUNT(id)
        FROM dbo.Posts
        WHERE CreationDate = '2023-05-10'
```

### Stored Procedures:

A stored procedure is a group of SQL statements that are created and stored in a database management system, allowing multiple users and programs to share and reuse the procedure. A stored procedure can accept input parameters, perform the defined operations, and return multiple output values. This enables users to provide different inputs to manipulate or retrieve specific portions of the data. Then, when one user modifies the stored procedure, all users will receive that update.  
**The query execution plan** of a stored procedure is cached in the server memory, making the code execution much faster than dynamic SQL because the compilation is not done at run time which is a factor making the dynamic SQL performance slower.  

When a stored procedure is executed it is optimized and compiled and the query plan is placed in procedure cache (In memory) for other users, as long as there is space. However, they are removed using the least recently used (LRU) algorithm.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/39582225-5ac8-4b6c-9b76-2a38602e7a56)  

Stored procedures are also a great tool to establish security on data by giving access only to  the the result of the stored procedure and not a table or a view.  

### SQL Order of execution: 

![image](https://github.com/user-attachments/assets/a896a93d-5a58-4820-971e-4ec2bfb73a6d)
