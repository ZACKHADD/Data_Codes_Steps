# Overview of questions of the DP-600 exam

## Plan, Implement and Manage:

- If we want to be efficient in cost and perform light Fabric workloads with an access to 20 Power BI users to view reports we can purshase an F2 (200 $/month), F4 or F8 and 20 Pro license (10 $/month) but not a F64 (it costs 5000 dollars a month).
- When manipulating models using XMLA (Read & Write) it is recomanded to enable large Model support if the size exeeds 1GB, it operates faster.
- Applying sensitivity labels makes sure that only authorozed people can access Data.
- To be cost effective if we have F64 reserved capacity and avoid being throttled, we need to utilize autoscaling with a preset cost limit.
- Orphaned workspaces are workspaces with no admin users.
- Lineage analysis gives impact on artifacts in the same Workspace while Impact analysis shows it accross workspaces.
- Protocol used in Gateway is TCP.
- Users are managed in the 365 admin center.
- Export to excel can be disabled for some or all security groups at the tenant level or at the report level in the option section.
- In Microsoft INTRA ID Fabric administrator duties can be done if we have Global Administrator, Power Plateform Administrator or Fabric Administrator roles.
- Query Insights in data warehouse track user queries (not internal queries).
- Timeout in Dtaflows Gen 2 with gateway is 60 minutes.
- Label sensitivity can be applied in all artifacts in a workspace (Power BI service/Fabric) (even dataflows).
- Query insights keeps tracking up to 30 days.
- Creating workspaces role is handeled in the tenant level (user must be in the security goupe having this role to create workspaces). We can only update and delete workspace at workspace level if we are admins.
- Connect (or disconnect) workspace to azure repository (GIT) is done by Workspace Admin or Power BI Admin.
- When data leaves the service of Power BI (exported), it is automatically encrypted.
- Sensitivity labels are not supported in PBIP files and we can not create PBIP files from service.
- Pipelines permissions are : Pipeline admin and workspace roles.
- To avoid single point of failure in gateway, we must implement a data gateway cluster.
- Lineage view in power BI is not available for viewer users and in fabric they can't see data sources.
- Workspace retention period (after delete) is configured in Fabric Admin Portal : Default 7 days and Max 90 days.
- To give access to a shared lakehouse to be accessed using Apache Spark we need **ReadAll** permission, ReadData gives only access using SQL.
- You can only connect a workspace to one branch and one folder at a time.
- Deployment Pipeline : Step 1 - Create a deployment pipeline, Step 2 - Assign a workspace, Step 3 - Make a stage public (optional), Step 4 - Deploy to an empty stage, Step 5 - Deploy content from one stage to another, Step 6 - Create deployment rules (optional)
- To avoid 48 hours limit of refresh in power BI, we can set a powershell code to refresh using XMLA endpoint.
- Fabric capacity is created in Azure.
- XMLA read-write access can be configured by the capacity admin or the power bi admin.
- If the user specify colors for data points, it overrides the theme.
- Connot connect to Semantic model in My Workspace or in Excel Workbook using XMLA.
- SSO connection mode is configured in the Fabric Admin Portal.
- To apply sensitivity Labels in PBI : you must have a Power BI Pro or Premium Per User (PPU) license, Sensitivity labels must be enabled for your organization, You must belong to a security group that has permissions to apply sensitivity labels.
- Always use security groups to give access to workspaces and settings.
- Workspaces in Fabric can contain a maximum of 1,000 semantic models, or 1,000 reports per semantic model.
- PBIDS stores data source with all specifications but it might asks for credentials even if they are embeded. And we can only view connection details in the file.
- Power BI Service Admin can : Change capacity size, Add or remove capacities, Add or remove users that have assigned permissions.
- We can access Fabric admin portal only if we are : Global Administrator, Power Plateform Administrator or Fabric Administrator roles otherwise we only see **Capacity Settings.**
- A workspace is assigned to a capacity and we can find that in the admin portal under the workspaces section.
- Data gateway not required for SQL databases in azure.
- Only Admins and Members that can publish & unpublish PBI application. Updating may be given to contributors.

## Prepare and Serve Data:

- To track historical data in a dimension use SCD 2 dimensions.
- 
