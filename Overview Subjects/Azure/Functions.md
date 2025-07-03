## This file gives all the concepts needed to be known about azure functions

As per the official documentation, developping azure function would require the following folder structure :  

```
<project_root>/
│
├── .venv/                    # Local Python virtual environment (optional but recommended)
├── .vscode/                 # VS Code config files (launch.json, tasks.json, etc.)
│
├── function_app.py          # ✅ Main function app file (holds decorated Azure functions)
├── additional_functions.py  # Other modules with decorated Azure functions (also scanned by runtime)
│
├── tests/                   # Unit/integration tests
│   └── test_my_function.py
│
├── .funcignore              # Files/folders to ignore when publishing
├── host.json                # Global function host config (required)
├── local.settings.json      # Local settings (e.g., secrets, environment variables)
├── requirements.txt         # Python dependencies
├── Dockerfile               # (Optional) For containerizing the function app
```

function_app.py is the main entry that contains the app identifier : 

func.FunctionApp()  
**Note that we cannot change this name otherwise it will break**  
In the new Azure Functions Python programming model (v2), we use decorators to define and control the types of functions we are using.  

There are two mandatory decorators required for every function, one to register the function so that it can be recognised by azure and the other one is the trigger type used for this function.  

✅ 1. @app.function_name(...) :

- Purpose: Registers the function with Azure Functions.  
- Applies to: All function types (HTTP, Blob, Timer, Event Grid, etc.)
- Without it: Azure won’t recognize or run the function.

```Python
@app.function_name(name="MyFunction")

```

✅ 2. A trigger-specific decorator — Required
This tells Azure Functions what kind of trigger is used (HTTP request, blob upload, timer schedule, etc.).  

Types of triggers :  

| Trigger Type            | Decorator                                | Description                                                 |
| ----------------------- | ---------------------------------------- | ----------------------------------------------------------- |
| **HTTP Trigger**        | `@app.route(...)`                        | Triggered by an HTTP request                                |
| **Timer Trigger**       | `@app.schedule(...)`                     | Triggered on a CRON schedule                                |
| **Blob Trigger**        | `@app.blob_trigger(...)`                 | Triggered when a file is added/modified in Blob Storage     |
| **Queue Trigger**       | `@app.queue_trigger(...)`                | Triggered when a new message appears in a Storage Queue     |
| **Event Grid Trigger**  | `@app.event_grid_trigger(...)`           | Triggered by Event Grid events (e.g., Blob uploaded)        |
| **Service Bus Trigger** | `@app.service_bus_queue_trigger(...)`    | Triggered when a message is received on a Service Bus queue |
| **Cosmos DB Trigger**   | Not yet supported in Python (as of 2025) |                                                             |
| **SignalR Trigger**     | Limited support in Python                | (Mostly for WebSockets scenarios in JS/.NET)                |


We can define for example a function that will receive an event from the eventgrid to check if the file size for example exceeds 1 mb then senf an email to the user saying that a file was creatred in the blob storage !  

### Create and test the function locally:  

First of all we will create the function in VSCode and test it before creating the eventgrid and link it to the function !  

Since we are creating an azure function and we need to test it locally we need like a simulator of azure which is AZURITE ! This creates an envirement like if we are lunching the function in Azure.  

We can create an azure function project using : ctrl  + Shift + P. This will open a window where we can choose to deploy, create a azure function project then we give the name, choose the type of the function we say **event grid trigger function** and if it is the first time it will ask to connect first ! Once done it will create the default folder structure (with the virtual environnement also that we will need to activate using : source activate) with files containing the minimum configs for an event grid function !  

![image](https://github.com/user-attachments/assets/57222988-38b3-4c32-b8dc-2ec1aad9e4ad)  

Now we can start creating our function :  

```Python
import logging
import azure.functions as func
from azure.functions import EventGridEvent

app = func.FunctionApp()

@app.function_name(name="tests_func")
@app.event_grid_trigger(arg_name="event")
def send_email_if_large_blob(event: EventGridEvent):
    event_data = event.get_json()
    data = event_data.get("data", {})
    content_length = data.get("contentLength", 0)
    logging.info(f"file size: {content_length}")
```

- logging library will handel the logs our function will show.
- azure.function is the main function that will create the entry point that the runtime will check to search for functions that are registered
- than we can have other libraries or functions such as EventGridEvent that will be useful for us to catch the evengrid notifications

Note that azure-functions need to be installed : pip install azure-function  

We then use the decorators to declare the name of the function and the envent-grid-trigger that means that the function should be triggered by an Event Grid event. The name of the event to receive is "event" (we can give another name if we like). After that we can start defining our function that will take as an argument the event we receive and we declare it as EventGridEvent type.  

then we defice event_data which weill cantain the payload sent by the eventgrid the we retrieve the elements we want such as content_lenght. To understand this we need to see what is the structure of the schema sent by event grid ! Typically a event grid schema would look like follows :  


```json
[
  {
    "id": "1234",
    "eventType": "Microsoft.Storage.BlobCreated",
    "subject": "/blobServices/default/containers/testcontainer/blobs/test.txt",
    "eventTime": "2025-07-02T15:00:00Z",
    "data": {
      "api": "PutBlob",
      "clientRequestId": "abc123",
      "requestId": "req123",
      "eTag": "0x8D4BCC2E4835CD0",
      "contentType": "text/plain",
      "contentLength": 2000000,
      "blobType": "BlockBlob",
      "url": "https://example.blob.core.windows.net/file.txt"
    },
    "dataVersion": "1",
    "metadataVersion": "1"
  }
]
```

So we can see that following the strcuture we can retrieve the elements we want to handel using .get() function or ["data"] for example.  



