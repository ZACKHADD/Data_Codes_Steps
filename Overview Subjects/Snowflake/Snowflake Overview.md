# Snowflake Overview (Workshop) :

Snowflake id Data Platform and data warehousing solution (Relational Databases) that enables data storage, processing, and analytic solutions that are faster, easier to use, and far more flexible than traditional offerings. It is a **Cloud datawarehouse with a distributed storage and processing technology**.  

## 1. UI Presentation :

Snowflake UI is very friendly andcoposed of a panel (left) containing several sections. When clicking on a section it opens at the right all the content.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2411e4c6-2602-472d-b3ff-e2c28b42fa75)  

- Projects: A section for coding projects like SQL queries and transformations, python, SPARK jobs ...
- Data: A section related to databases.
- Data Products: A section dedicated to marketplace and data service providers.
- Monitoring: A section where we can monitor all the objectsm jobs, queries etc.
- Admin: Administration section where we can follow up the cost of all the worloads, monitor the data warehouses, change roles etc.  

## 2. Identity and Access :

Identity and access are two different concepts.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5435b89c-b44c-4f1b-be2b-2d618b46f9ff)  

When we managed to prove who we are we get athenticated to use snoflake, but then the authorizer needs to check if we are athorized to do an action or access a specific item inside snowflake.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/3eac8f90-e30a-4a14-8d10-edb596163b52)  

The authorizer uses Role Based Access Control (RBAC) to allow the user to perform an action based on his current role.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/67b14209-34ca-4bef-83a6-cecad6bae6fc)  
 
**Snowflake power in this area is that we can switch roles and stay connected with non need to log in again.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/42a6d1d3-789b-4d35-9ad2-d27b8fbaf253)  

The ADMINROLE is the most previleged role. Meaning that having this role gives the all the previleges of the other roles based on the principal of **RBAC Inheritance.**  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/e1736e88-3d7d-4cda-af49-930e38b7ead5)  

Another role was recently added : **ORGADMIN**, very powerful role that can tie multiple Snowflake accounts together and even generate accounts.  
The **SYSADMIN** is widely used to create  databases and warehouses. **Note that it is always a good practice to troubleshoot by checking the role settings if we get some errors when performing an action.**  
In general, we have a default role that gets assigned to us every time we log in and we can switch it if we are allowed. We can also alter the default role if we like.  
**We can also create our own custom.**  
The logic of roles is that for example, to create a database we need to have a SYSADMIN role. However, if we already have an ACCOUNTADMIN role assigned we can switch to a SYSADMIN role to perform the database creation. **So there is a difference between the role assignement and the role impersonated to perform an action.** Note that if we have a SYSADMIN role assigned, we can't impersonate ACCOUNTADMIN one since the process is forward only.  

There also another process behind the scene that is combined to RBAC, which is DAC (Discretionary Access Control) that states : **you create the object, you own it!**. meaning that if SYSADMIN creates a database, they own it and so they can delete it, change the name, and more. **So the ownership of items belongs to roles.**  
Also,every item owned by a role is automaticaly owned by the role above,a principal that we call **custodial oversight**.  

## 3. Database creation :

As every classic SGBDs, **database** is the upper level of data items grouping.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/24747a4a-3dd0-4c3b-8a5a-cd2a21f181e3)  

 Next we find another grouping level which is **schema**: it is a grouping of several tables related to the same scope.  

 ![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/668c3854-a7eb-4d05-869e-0a441f97761d)  

By default in snowflake we have **INFORMATION_SCHEMA** schemas containing views of METADATA regarding the all the objects of the database. It cannot be modified.  

We can transfer the ownership of the database to another ROLE if we desire :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/6f4286e3-213d-41d0-a1a9-ae3be4c12851)  

**Note that the transfer of Database Ownership does not give access to it's schemas, this should be done also at the schema level.**  
**This can be done also by granting Access which is the best way.**  
The same thing can be done with Datawarhouses:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/725a5326-4ee5-4fcd-8a1e-7e0217290f31)  

## 4. Worksheets Creation :

The worksheet interface has four menus. Two are in the upper right corner and two are close to the first line of code.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/861a96d6-3463-466c-9fe5-6a492659d570)  

We have also the result area withsome metrics at the right.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/5c031261-fc24-430a-be47-b2366d1c6505)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/9114f357-0de2-403a-bc1e-19f30b34a358)  

We can start creating tables as follows :  

                                          use role sysadmin;
                                          create or replace table GARDEN_PLANTS.VEGGIES.ROOT_DEPTH (
                                             ROOT_DEPTH_ID number(1), 
                                             ROOT_DEPTH_CODE text(1), 
                                             ROOT_DEPTH_NAME text(7), 
                                             UNIT_OF_MEASURE text(2),
                                             RANGE_MIN number(2),
                                             RANGE_MAX number(2)
                                             );  
Just like SQL IDE softwares, we can view the code defining the table: 

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/2140c0e1-cbc9-4220-94c0-4edb5fd32664)  

We can also alter tables :  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/0b03561f-13b5-462d-ab6b-5a1abe70837a)  




