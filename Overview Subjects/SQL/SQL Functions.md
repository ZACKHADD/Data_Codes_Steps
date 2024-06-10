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


