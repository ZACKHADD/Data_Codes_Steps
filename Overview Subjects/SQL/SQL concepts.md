How NULL values are treated in SQL engines : 

As scalar : NULL=NULL is not supported ! it produces NULL and not TRUE or FALSE we cannot compare two **undefined** values !  
This behaviour garantees safe joins and represents the reality and logic : UNKNOWN does not equal UNKNOWN ! This means that in conditions, joins and filters we cannot compare nulls !  

But we can check if a column has an UNKNOWN value at a position using : IS NULL. This will check or retrieve all the positions (rows) where a column contains NULL values !  

**The big difference is in how we ask the question : does NULL=NULL ? we don't know ! Does a column contain NULL values in some positions (IS NULL) ? yes we can check that USING METADATA or BITMAP (There is a flag for null values for a column in each row) that stores behind the scenes positions of NULL values**  

In an ARRAY object (in some databases such as SNOWFLAKE) [NULL]=[NULL] is TRUE because we compare the structure type not individual values ! **it is like saying : are the two object arrays and do thay have NULL values in the first position** !  
**It’s a deterministic structural check, not a semantic “unknown” evaluation.**  

Most of aggregate operations ignore NULL values since it's mathematical and involves comparing scalar values ! **But other functions do not ignore NULLs such as DISTINCT**  
COUNT for example ignores NULL values if columns are specified inside it : COUNT(COL1, COL2 ...). But if we use **Row constructure** (*) :  
- COUNT(*) does not ignore NULL values as it considers the row constructure as a tuple and so it compare the structure not values ! Just like ARRAY does.
  
DISTINCT function treats NULLS as same NOT by comparing but by using hash function (most of the time when possible or sort-based and eliminates successive dupplicates values or NULL ones) that gives the same value to NULLs to tell us that for example several rows have the NULL values for a column ! and as a **structure** these rows are considered dupplicates !  

![image](https://github.com/user-attachments/assets/81eab8e8-6e77-4163-8140-fc689f9df6ac)  




