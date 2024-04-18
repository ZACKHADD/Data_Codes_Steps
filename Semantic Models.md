## The present document gives a definition of the semantic model and it's benefits:  

credit: https://www.dimodelo.com/blog/2023/what-is-a-semantic-layer-what-why-how-and-more/#:~:text=A%20semantic%20layer%20is%20typically,of%20information%20across%20the%20business.

#### Datawarehouse vs semantic model:

A semantic layer is typically the “top” layer of a data warehouse/lakehouse. It is accessible to end users and report developers, who use it as the source for reports, dashboards, and ad hoc analysis.  
A semantic layer is important because it fosters a shared understanding of information across the business. It supports the autonomous development of consistent reports, analysis and dashboards by end users.  

It is also:  

- First, a business glossary. The basis of a semantic layer is a glossary of related commonly understood business entities, terms and metrics, e.g., The concepts of a Customer, Product, Sale and the relationship between Sales and Products, and Sales and Customers.  
- Represented as a “business view” of data. Physically, the “business glossary” exists as a “business view” of enterprise data. The “business view” is implemented in the semantic layer. The semantic layer is typically the highest/final layer of a data warehouse or data lakehouse implementation. The raw enterprise data undergoes a series of transformations to arrive at the semantic layer’s “business view” format. The most common “business view” format is a dimensional model.  
- Accessible to BI developers and end users. The semantic layer is accessible to end users, data analysts and BI developers. It’s the source of data for reports, ad-hoc analysis and dashboards.  
- Supports Autonomy. A semantic layer supports autonomous access and navigation of the data. It supports self-service BI and ad-hoc (drag-drop) analysis.  
- Fosters a shared understanding. A semantic layer fosters a common understanding of enterprise data amongst all business users.

#### The different Types of Semantic Layers

The semantic layer tends to fall into four categories:

##### 1. Data Store or fat semantic layer

The Datastore flavour contains a complete additional copy of the Data. The data is stored in a proprietary, highly compressed and optimised format (usually column store). This flavour comes with an internally optimised vector-based query engine. The Datastore flavour tends to perform better than the Virtual flavour because these data stores are structured to support high-performance aggregated queries. The emphasis is on aggregated. The data store is usually transient and re-loaded periodically from the Data Warehouse. Example technologies include Microsoft Azure Analysis Services, PowerBI data sets, Kyligence, GoodData, Apache Druid and Apache Pinot.

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/ae93bbea-5bbc-4f3d-9d8d-c9b5b455d284)  

**Advantages:**
- High performance for aggregated queries
- Easier to define security.
- It contains functional query languages that make it possible to define complex metrics.
  
**Disadvantages:**  

- Increased complexity and cost to load an additional datastore.

##### 2. Virtual or thin semantic layer

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/832d1ed5-fa26-4383-945c-aff9b0d01ccd)  

The virtual semantic layer doesn’t store a separate copy of the data (although it might cache data for performance). Instead, the virtual semantic layer contains the logic that defines the semantic model. When a query is executed, the virtual semantic layer acts as a proxy and generates and runs an SQL query against the underlying data source(s). Examples of technologies include Cube, Malloy, LookML, Metriql, AtScale, MetricFlow, Metlo, and Denodo.  

**Advantages:**  

- Avoids data movement to an additional data store.Lower complexity.Lower cost due to less data storage.
  
**Disadvantages:**  

- Tend to be slower than data store-based semantic layers optimised for aggregated queries. Caching and pre-computation can elevate some of this disparity, but not all; frankly, it moves this solution toward the hybrid option anyway.

##### 3. Hybrid semantic layer 

Hybrid flavours support both datastore and virtual modes. Developers can generally define which tables within a semantic model are stored vs virtual. This can help with the trade-offs of performance vs complexity/capacity. In some cases, the data may be so large it is beyond the capacity of the semantic platform to store it. It could be argued that virtual caching is one way to achieve a hybrid model. A better example is Power BI datasets that allow a single semantic model to have a mix of stored vs virtual (direct query) tables.  

##### 4. Meta Semantic layer

This is a relatively new development in semantic modelling. Effectively, companies like dbt allow developers to define metrics in a platform-agnostic language. The idea is that dbt can generate the code for a given metric for any supported platform. That’s the idea, at least. Think potentially generating a Power BI dataset or another language for a different platform. This provides portability. It’s very early days, and so far, dbt’s implementation is more focused on supporting its own dbt cloud semantic server. However, the original open-source and open-platform idea persists. Time will tell. dbt Semantic Layer | dbt Developer Hub (getdbt.com)  

#### How are semantic layers implemented?

From a physical point of view, a semantic layer is implemented using specialized software. There are several flavours, which we will expand on in this section.  

##### 1. Semantic layer implemented within a BI tool

Modern BI reporting and analysis tools like Power BI, Tableau and Qlik allow data analysts to model a semantic layer directly within a dashboard or report. Some tools will enable you to deploy the semantic model independently from a report and have it act as the semantic layer that many dashboards and reports share.  

Ultimately, this approach has issues. An undisciplined proliferation of separate semantic models within dashboards and reports defeats the purpose of a shared semantic layer. What is needed is a disciplined development approach that enforces shared data models.  

Power BI has the most comprehensive use case as a semantic layer among the most popular BI tools. While it can support a semantic layer coupled with a report, it can also support a scenario as a stand-alone semantic layer. First, a data set can be published independently from a report and shared by many reports. Power BI premium capacity supports large data sets beyond the typical 10GB up to the capacity size. Power BI has an XMLA endpoint and a REST API that allows external applications to make queries. It enables users to create composite models to supplement data, e.g., a budget Excel spreadsheet combined with actuals from a data warehouse. Power BI supports stored, virtual and hybrid query models. People have even used Power BI as the semantic layer and Tableau as the client via XLMA endpoints. It also has a highly capable functional language, DAX, an internal Vertipaq/tabular data store, and a query engine.  

In contrast, Tableau’s strength is its visual aesthetics. It does support some semantic modelling concepts in its data model. It also supports high-performance queries with its Hyper query engine. However, it lacks the concept of publishing a data model as a stand-alone semantic layer. It also lacks the rich analytics language like DAX.  

#### Semantic Layer implemented in a Data Warehouse/Data Lakehouse Architecture

![image](https://github.com/ZACKHADD/Data_Codes_Steps/assets/59281379/658e6e58-f0eb-4bd4-b6fb-2b4f22b14721)

The semantic layer is the final layer of a data warehouse/data lakehouse architecture.  

The semantic layer sits between the presentation layer (the gold layer in Data Lakehouse) and the reports/analysis/dashboards. End users can view, navigate, and query the semantic model through a BI tool.  

The best practice is to model the presentation/gold layer as a dimensional model. A dimensional model star schema lends itself nicely to semantic layer entities. The core of a dimensional model is business processes, which are modelled as facts with their measures/metrics. The business entities involved in business processes are modelled as dimensions with their descriptive attributes.  

Most semantic modelling work and transformation occurs in the presentation/gold layers. The semantic layer should map directly to the Facts and Dimensions in that layer.  

#### So, if the semantic layer looks like the presentation layer, why need a separate semantic layer?

As discussed before, the semantic layer software offers:  

- High-performance aggregated queries.
- Augments and Enhanced Information. Enhances and simplifies the information in the underlying Data Warehouse or Data Lakehouse. This includes:
- Hiding tables, columns, and relationships irrelevant to the business, including surrogate keys and management columns.
- Adding Hierarchies. Hierarchies enable hierarchical reports, drill-downs and more straightforward navigation. Adding reusable context-aware calculated metrics.
- Renaming tables and columns if necessary (although your data warehouse, if modelled correctly, should already use the correct naming standard).
- Unified Data. Semantic models can combine data from multiple sources. Indeed, an end user could enhance the data warehouse with their own data source at the semantic layer.
- Supports BI tools. Many BI tools natively support connecting to and querying semantic layer software.
- Context-Aware Security. Restrict data access based on tables, rows, columns and formulas.
- Supports ad hoc analysis. Provides an interface that BI tools use to enable ad-hoc, drag-and-drop, pivotable style data analysis. The data store and query engine behind semantic layer software makes this possible.
