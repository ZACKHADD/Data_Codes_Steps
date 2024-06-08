# The present document gives an example of a Fabrci data pipline to check the existence of file before transformation and writing to the destination

## Action: The pipeline checks if the CSV source file exist or not before running a notebook that filters the data and write it down to a delta table in onelake.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/01e11196-636d-4757-8e3d-64f4fb2ca333)  

The pipeline has two activities: Get Metadata activity to capture the **exists** property and Switch that runs activiies depending on an expression returned value.  

**Note that at the pipeline level (click outside of activities), we specify two parameters : the lakehouese GUI (identifier and not the name) and the file name with the extension.**  

### 1. Get Metadata activity:

The Get Metadata activity capture the **exists** value (true or false) that sepcifies if the file evaluated exists or not before performing any action.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/78ac2709-fe3b-4d39-b44c-98b3b20dd6a1)  

under the settings section we specify the parameters to use and the output of the activity under the **Field List.**  

### 2. Switch activity:

The switch activity is like the switch function. It evaluates the output of an **expression** and return back not values but activities to be performed:  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/8aee2b94-b8cc-4768-9dae-9c34bfe8496d)  

The expression evaluation is compared with the **name of the Case**. The comparision is case sensitive. If the output of the expression evaluation matches any of the cases the coreresponding activity is run.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/925c7971-2331-4b5e-af41-29fed76b40ba)  

The expression is simpily the output of the Get Metadata activity. Since this latter is an object (dictionnary) we need to tranform it to a string since the case name is a string.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/4dc93852-46d0-474c-99b5-27c25f94fb4a)  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/24f183ea-5a53-4b17-9f98-46421263f934)  

If the output is True the filter activity is run. However if the output of the expression is False an error activity with customized message is run.  

### 3. Notebook Activity:

The activity is pointing to a notebook that filters the data and write to a delta table in onelake.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/19d2a1e8-a3cd-4876-a32a-8a94b0608f73)  

The first cell is parametrized, meaning it accepts parameters from the pipeline run to dynamically run the notebook against other files if we like.  

### 4. Error activity:

A simple fail activity that displays a custom message.  

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/b924f0fe-4899-43ee-9eb4-f9e680bdcc75)  



