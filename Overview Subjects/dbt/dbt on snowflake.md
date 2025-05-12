# This document presents a use case on how to setup and configure DBT on Snowflake to handel data transformations

We will go trough all the steps neede to configure dbt locally and connect it to snowflake then how to automate the deployment using Github Actions !

## dbt Setup : 

First of all we need to create a virtual environnement to isolate our project so it will not be impacted by any dependencies updates (the virtual environnement will have its own dependencies such as python version, packages and so on, that are not shared with other projects).  

![image](https://github.com/user-attachments/assets/43dbc402-859b-4c0d-9bc3-b57dc4d7b965)  

Now let's activate it : 

![image](https://github.com/user-attachments/assets/82793df4-9fa0-44b8-842a-d32f28115128)  

Now we install the adaptor that will link dbt to snowflake :

![image](https://github.com/user-attachments/assets/c6debe7d-20a0-4911-985d-d5ddaa08f079)  

We can see now (on the left of VScode screenshot) that all our dependencies get installed in the virtual environnement :  

![image](https://github.com/user-attachments/assets/6226ef5a-9b53-479f-9144-fb6b5b775c92)  

Then we initiate the dbt project using *dbt init name_project* :  

![image](https://github.com/user-attachments/assets/d2af61e6-0a33-4b12-b3bb-b2bc4b2b769a)  

It will ask for some connection configurations, which will create a yaml profiles file that will hols these configurations.  
 
![image](https://github.com/user-attachments/assets/fb9f709d-7869-46bc-a501-459866ca900e)  

Then we can check the yaml file in its location (user/.dbt path):  

![image](https://github.com/user-attachments/assets/ca40078b-e7e6-45b4-8b6a-4d8c8b545d1e)  





