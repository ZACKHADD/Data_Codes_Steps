# From DWH to Data Lake to Data Lakhouse! Conceptual understanding !

This file gives a conceptual explanation to understand the major elements, and benifits, of each milestone of data analytics and why should we use Lakhouses today!

## The problem !?

Traditionnal databases were working just fine using SGBDs to store and analyse data in the local servers of the company, with a lot of obstacles (a lot of things to maintain, configure, networking, expensive infrastructure cost ...), but it was working just fine.  
Moving to the era of big data (starting 2000s), we confronted a big problem with traditionnal databases systems not being able to process data quickly enough to respond to the business needs.  

A huge step toward solving this problem was the HADOOP releasing in 2006! the concept of data lake appeared and the ease of querying rapidly massive amounts of data using the distribution and map reduce capabilities, life data users was getting easier ..!  
The problem is that with this huge amount of data and to gain in terms of speed in processing data we sacrificed governance (all the rules of data integrity in the traditionnal SGBDs)! Later, people started asking what is this data, who created it and why, is it current, accurate, and so on !!  
The data lake became quickly a data swamp where we don't know what the data really is and how to use it in advanced analytics!  
The need of data governance is here again so how can we combine this Data Warehouse characteristic with a benefits of data lakes ? Actually the core of SQL data warehouses should have just been evolved to meet the new needs of big data and not to be thrown off ==> The answer is the **Data Lakehouse.**  

### 1. What are the features of a SQL relational database:  

Relational databases answer a lot of our needs in terms of storing, maintaining and analyzing data!  
The conept is quite simple, in our database we store data in tables (with predefined schemas, not to confuse with the schema which is a logical groupment of tables).  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/1181d918-974b-4609-9f2b-f1ba43832389)  

Each table is specific to a class of objects (**see the ERM data modeling techniques**) and we store data about these objects in the table using **Transactions: Insert, Update and Delete.** We also interat with the database using **CRUD** operations : **Create, Read, Update and Delete**.  
The SQL language contains a lot of other queries , **called also sublanguages** to handl many objectives in the database refered to as : DDL, DML, DMV, DCL ...  

Note that the transactions can be done by user having access to the database, or users having access to applications connected to the database such as CRM, FRP ...  

To keep data clean and well maintained, it has been agreed on that transaction should respect the **ACID** characteristics:  
- Atomicity
- Consistency
- Isolation
- Durability

It is simply **All or nothing concept** where we apply all the sets of maintanace or nothing at all. For example, lets take the following transaction:  

```
      Insert in sales table
      Insert in costumer table
      Update in sales table the date
      COMMIT
```
This package of operations is called a **transation**. Now, in our data database there is somthing called **referential integrity** where for example the sales table connot contain a sale done by a costumer that does not exist in the costumer table.  
In our transaction, if the insert step of sales is not done, because the costumer is not yet there in costumer then the whole transaction is **Not COMMITED** and will **ROLLBACK.** This will garanty the consistancy.  
What also makes the integrity, data quality, of data leveled up is the **constraints** such as:  
- Domain constraints (Data Types)
- Key Constraints (uniqueness)
- Entity Integrity constraints (primary key cannot be null for example)
- Referential integrity constraints
- Column value constraints (for example, we can put a rule on sales values to be greater than 0)

What supports all this mechanism of transactions is the **Transaction logs.** These are external files maintained when the transactions are made. It keeps track of all the changes made and makes it able to go back to the **last point** (When we set a **COMMIT** statement, it is like a save) of the DB is something wrong occures.  
In parallel with this, DBAs will make backups of databases so that if it is deleted we can recover it again from the last accurate point and we apply the transactions needed using the transaction logs.  
Also, traditionnal databases handle the security aspects, who can access data, what permissions and so on and also **Triggers** that are SQL rules (queries like checking referential integrity) that runs if an event happens and also can be used to insert auto,atic logs.  
