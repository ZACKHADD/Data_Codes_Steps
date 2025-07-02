## This file gives all the concepts needed to be known about azure functions


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
