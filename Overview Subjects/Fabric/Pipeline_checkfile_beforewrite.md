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

### 5. Json configuration file:

```json
    {
    "name": "Check_data_before_load",
    "objectId": "84cbcca8-75c2-45b9-a27a-99cd5ff4aa24",
    "properties": {
        "activities": [
            {
                "name": "Get Metadata1",
                "type": "GetMetadata",
                "dependsOn": [],
                "policy": {
                    "timeout": "0.12:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "typeProperties": {
                    "fieldList": [
                        "exists"
                    ],
                    "datasetSettings": {
                        "annotations": [],
                        "linkedService": {
                            "name": "3eb306a1_6b8d_4c18_b41c_1c24e8ecb386",
                            "properties": {
                                "annotations": [],
                                "type": "Lakehouse",
                                "typeProperties": {
                                    "workspaceId": "90f00dcd-3d7d-4cc5-8fdd-84669dbe468e",
                                    "artifactId": "@pipeline().parameters.Lakehouse_Name",
                                    "rootFolder": "Files"
                                }
                            }
                        },
                        "type": "DelimitedText",
                        "typeProperties": {
                            "location": {
                                "type": "LakehouseLocation",
                                "fileName": {
                                    "value": "@pipeline().parameters.File_",
                                    "type": "Expression"
                                },
                                "folderPath": "Baseball"
                            },
                            "columnDelimiter": ",",
                            "escapeChar": "\\",
                            "firstRowAsHeader": true,
                            "quoteChar": "\""
                        },
                        "schema": []
                    },
                    "storeSettings": {
                        "type": "LakehouseReadSettings",
                        "enablePartitionDiscovery": false
                    },
                    "formatSettings": {
                        "type": "DelimitedTextReadSettings"
                    }
                }
            },
            {
                "name": "Check if file empty or not",
                "type": "Switch",
                "dependsOn": [
                    {
                        "activity": "Get Metadata1",
                        "dependencyConditions": [
                            "Succeeded"
                        ]
                    }
                ],
                "typeProperties": {
                    "on": {
                        "value": "@string(activity('Get Metadata1').output.exists)",
                        "type": "Expression"
                    },
                    "cases": [
                        {
                            "value": "False",
                            "activities": [
                                {
                                    "name": "File Eroor",
                                    "type": "Fail",
                                    "dependsOn": [],
                                    "typeProperties": {
                                        "message": "File is empty",
                                        "errorCode": "400"
                                    }
                                }
                            ]
                        },
                        {
                            "value": "True",
                            "activities": [
                                {
                                    "name": "Flter_Data",
                                    "type": "TridentNotebook",
                                    "dependsOn": [],
                                    "policy": {
                                        "timeout": "0.12:00:00",
                                        "retry": 0,
                                        "retryIntervalInSeconds": 30,
                                        "secureOutput": false,
                                        "secureInput": false
                                    },
                                    "typeProperties": {
                                        "notebookId": "6e960c0e-1c53-45e2-bb23-c9d64dd0ba24",
                                        "workspaceId": "90f00dcd-3d7d-4cc5-8fdd-84669dbe468e",
                                        "parameters": {
                                            "Lakehouse": {
                                                "value": {
                                                    "value": "@pipeline().parameters.Lakehouse_Name",
                                                    "type": "Expression"
                                                },
                                                "type": "string"
                                            },
                                            "file_name": {
                                                "value": {
                                                    "value": "@pipeline().parameters.File_",
                                                    "type": "Expression"
                                                },
                                                "type": "string"
                                            }
                                        }
                                    }
                                }
                            ]
                        }
                    ],
                    "defaultActivities": []
                }
            }
        ],
        "parameters": {
            "Lakehouse_Name": {
                "type": "string",
                "defaultValue": "63a3faba-fe05-4b94-9b7f-a23dd0e3bb03"
            },
            "File_": {
                "type": "string",
                "defaultValue": "Players.csv"
            }
        },
        "variables": {
            "var_output": {
                "type": "String"
            }
        },
        "lastModifiedByObjectId": "c294ad3a-c426-4d72-a466-c706708b94ca",
        "lastPublishTime": "2024-06-08T16:45:29Z"
    }
}
```
