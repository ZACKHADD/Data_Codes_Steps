## Relevent Remarks :
- Power Query CASE sensitive vs Power BI case insensitive ==> clean, trim and use uppercase to put the columns to one format.
- RELATED function is used in one direction : Many to 1 ==> get in the many side the column of the one side.

## DAX queries generated behind the scene:  
Bare in mind that whatever mode, calculation, graphs or envirement we work on, the DAX queries generated will have the same basis which is **Using SUMMARIZECOLUMNS function.**  

![image](https://github.com/user-attachments/assets/e7647650-5911-4502-acf0-2a75586196b6)  

Understanding the function along with calculate is fundamental key to understand how DAX function work.  

## Auto-Exist behaviour:
Auto-Exist is a DAX technology that optimizes filtering in order to reduce the amount of processing required for certain DAX queries. Auto-exist is triggered by default when we **we filter on two or several columns from the same table in the same time**. The engine will not scan all the table to give us the result (by doing a crossjoin of the two columns) but will generate only the existing combinations in the **filter context (such as slicers)** the calculation is made in.  
Let's take an example using the following table:  

![image](https://github.com/user-attachments/assets/ed2890bb-f379-4d2a-b4be-a76dcb0b847c)  

Now generating a matrix will give this:  

![image](https://github.com/user-attachments/assets/033324d2-adfd-40e4-9ef9-44b505a346bb)  

Adding two measures to calculate the absolute number of projects and the simple number of projects would be done as follows:  

```DAX

				# Projects = COUNTROWS ( Projects )
			 
				# Projects All Time = CALCULATE (
				    [# Projects],
				    ALL ( Projects[Year] )
)
```
We expect add this point that the **ALL function would remove any filter on the Year!**  

If we add some silicers to filter the context using the year and the language, we get the following page:  

![image](https://github.com/user-attachments/assets/02283f91-2a8b-4019-a74e-4d3e3aa22ffb)  

Now we can see that by filtering on 2017 we have the numbers we want, but what if we filter on 2018?:  

![image](https://github.com/user-attachments/assets/5af2c5f6-7281-4160-9187-05cfaa2fc5af)  

We can see that the **# Projects All Time** figure is wrong! So the ALL() function is not working as it should do!  
This is causeed by the auto-exist behaviour that filters behind the scene the table using only the existing cominations in the filter context of **Year = 2018 and Langauages = DAX || Python.** So first, the auto-exist will filters the table on the existing combinations of the filter context and then runs the ALL() function which will be simillar to adding the two filters together and not in a seprarate way!  

It is simillar to using the following query (Group by using summarize function):  

```DAX
		EVALUATE
		SUMMARIZECOLUMNS (
		-- The table is prefiltered using the filter context (slicers)
		    CALCULATETABLE (
		        SUMMARIZE ( Projects, 'Projects'[Language], 'Projects'[Year] ),
		        TREATAS ( { "DAX", "Python" }, 'Projects'[Language] ),
		        TREATAS ( { 2018 }, 'Projects'[Year] )
		    ),
		    "Result", [# Projects All Time]
		)
```

Since there is non combination of Python and 2018, the filtered table will not include **Python** data and so the **# Projects All Time** measure will not take them into consideration.  
The solution here is to use the column language from another table (such as a Language table). By default, when filtering on columns from different table **a crossjoin** is used instead of **Auto-Exist**. This is why it is so recommanded to use a star schema as a modeling technique in Power BI.  

More on this subject here:  

https://www.sqlbi.com/articles/understanding-dax-auto-exist/


## Row Context
The row context is simply the current row, in a table, for which a calculation is made. This latter is filtered based on the values, in the current row, of all the othe columns of the table.  
ALL() function used with a table **Removes entiraly the row context.** While used with a single column in a row context tiggers the **auto-exist** behaviour that calculate the expression only for the existing combinations of values of the columns of the table.  

## Filter Context
## Calculate evaluation

## Table Expansion:  

### What is that?
Table expansion is very important concept in Power BI and in DAX calculations in general. It is simply left outer join operation done behinde the scenes between the many side table and all the 1 side tables. For example, let's take the following model:  

![image](https://github.com/user-attachments/assets/efef10f9-e2f9-4350-a78c-2a1a2eb54037)  

All the tables on the many side can be expended using a left ouster join behind the scene when the expanded tables behavious is triggered. The left outer join will keep all the columns of the many side and add to them all the columns of the 1 side. For example the expanded table of "Winesales" will have all the columns of "Costumers", "Wines", "Salespoeple", "DateTable", and "Regions" **but also "Region Groups"** since "Region" is the many side of the relation between "Regions" and "Region Groups" **(Also if it is a 1 to 1 relation and it happens using a full outer join)**.  
So in this case the expanded table of "Winesales" is the whole data model.  
Another example would be the following model:  

![image](https://github.com/user-attachments/assets/7908a53e-5cbf-460e-998e-8c065b763427)  

The expanded tables for each base table would be:  

![image](https://github.com/user-attachments/assets/3b52688b-3fc3-4540-83fe-11c39f4782dc)  

### Why using it?

**For the most part, the reason we use table expansion in our expressions is to “reach” dimensions to perform aggregations on the filtered data within them.**  
This behaviour saves us time of creating crossjoins between tables since it is done behind the scenes in memory.  

### When it is triggered?

Table expansion is triggered whenever a whole table (In the many side of a relationship) is filtered. For example, consider the following model:  

![image](https://github.com/user-attachments/assets/66933e68-8f59-4b4a-99c3-c4f8850e1b27)  

Let's do some aggregation on the customer table with a filter on the sales table.  

```DAX
    EVALUATE
      	ROW(
      		"Filtered Customer Count on column", {
      			CALCULATE(
      				COUNTROWS(DISTINCT(Customer[CustomerKey])),
      				Sales[Quantity] > 2
      			)
      		},
      		"Customer Count", {
      			COUNTROWS(DISTINCT(Customer[CustomerKey]))
      		}
      	)
```

![image](https://github.com/user-attachments/assets/70dbbfa5-5427-471b-b11e-6fd839dfdad7)  

The results as we can see are the same with a filter using a the column **Sales[Quantity] > 50** and without filter at all. This is because the filter direction is one way from **Customer to Sales** and not the other way around. But what if we use all the Sales table as a filter?  

```DAX
    EVALUATE
    	ROW(
    		"Filtered Customer Count on column", {
    			CALCULATE(
    				COUNTROWS(DISTINCT(Customer[CustomerKey])),
    				Sales[Quantity] > 2
    			)
    		},
    		"Filtered Customer Count on sales table", {
    			CALCULATE(
    				COUNTROWS(DISTINCT(Customer[CustomerKey])),
    				FILTER(
    					Sales,
    					Sales[Quantity] > 2
    				)
    			)
    		}
    	)
```

![image](https://github.com/user-attachments/assets/3f1ed124-0dcb-4048-934d-6b7f61c285d3)  

Now we can see that the result of th count on the customer table is filtered based on sales table using the quantity column.  
This is working thanks to the **Table expansion** behaviour triggered when we filter using a whole table statement such as we have done above: **FILTER( Sales, Sales[Quantity] > 2)**. The expanded table of the base table **Sales** has all the columns of customer's table and all the other dimensions also.  
Now the calculation of the **COUNTROWS(DISTINCT(Customer[CustomerKey]))** is done based on the expanded table, and filtering on the column "Sales[Quantity]" will filter the Customer part and give the correct value.  

Another example of when the **Table Expansion** behaviour is triggered is when we have a **Row Context with a whole table** combined with a **Calculate function** as follows:  

```DAX
        EVALUATE
	ADDCOLUMNS(
		Sales,
		"Filtered Count Customer", CALCULATE(COUNTROWS(DISTINCT(Customer[CustomerKey])))) -- Calculate here, combined with the row context of sales tables, triggers a context transition
	)
```

The calculate function performs a **Context transition** by turning each row in the sales table (Row context) to filters on all the columns of the sales table where parameters equal the values of the row.  

![image](https://github.com/user-attachments/assets/88ceb390-7fa0-41b4-8481-7d450e8d83dd)  

For example for the first row the filters would be written as follows:  

```DAX
    FILTER(Sales,Sales[Unit Price]=77.68 && Sales[Line Number]=0 && Sales[Unit Cost] = 35.72 .... ) -- and so on using all the columns of the sales table
```
It is like using the sales table (Which is a many side table) as a filter. This behaviour of the calculate function on the row context generates a **table expansion** that makes it possible to do aggregations on costumer columns filtered in the context of the current row as the [Filtered Count Customer] calculated column shows:  

![image](https://github.com/user-attachments/assets/da9f399c-c408-4779-a0c6-cc6167ff0e3f)  

Note that if we limite the **row context** to only a part of the sales table (using SELECTCOLUMNS for example), the table expansion is not triggered and the calculation will be the same whatever the row context is:  

![image](https://github.com/user-attachments/assets/543851fe-05a4-4810-a534-237b4eace88f)  


