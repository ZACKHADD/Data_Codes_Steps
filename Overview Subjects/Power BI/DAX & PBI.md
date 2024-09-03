## Relevent Remarks :
- Power Query CASE sensitive vs Power BI case insensitive ==> clean, trim and use uppercase to put the columns to one format.
- RELATED function is used in one direction : Many to 1 ==> get in the many side the column of the one side.

## Row Context
## Filter Context
## Calculate evaluation

## Table Expansion:  

### What is that?
Table expansion is very important concept in Power BI and in DAX calculations in general. It is simply left outer join operation done behinde the scenes between the many side table and all the 1 side tables. For example, let's take the following model:  

![image](https://github.com/user-attachments/assets/efef10f9-e2f9-4350-a78c-2a1a2eb54037)  

All the tables on the many side can be expended using a left ouster join behind the scene when the expanded tables behavious is triggered. The left outer join will keep all the columns of the many side and add to them all the columns of the 1 side. For example the expanded table of "Winesales" will have all the columns of "Costumers", "Wines", "Salespoeple", "DateTable", and "Regions" **but also "Region Groups"** since "Region" is the many side of the relation between "Regions" and "Region Groups" **(Also if it is a 1 to 1 relation)**.  
So in this case the expanded table of "Winesales" is the whole data model.  
### Why using it?

**For the most part, the reason we use table expansion in our expressions is to “reach” dimensions to perform aggregations on the filtered data within them.**  
This behaviour saves us time of creating crossjoins between tables since it is done behind the scenes in memory.  

### When it is triggered:

Table expansion is triggered whenever a whole table (In the many side of a relationship) is filtered.  


