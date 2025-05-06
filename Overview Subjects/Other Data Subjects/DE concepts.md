all systems store data in files — Snowflake, SQL Server, Databricks, Oracle, etc. Each system uses formats it can understand and process efficiently.

But the real, technical distinction of a database isn’t just about "storing files." It’s about who owns and manages the data lifecycle and how tightly integrated that is with the execution engine and metadata catalog.

✅ The Real Differentiators of a Database
Let’s define what makes something a "database" in a technical and architectural sense — even if all systems use files.

1. Ownership and Opacity of Storage
Aspect	Databases (e.g., Snowflake, SQL Server)	Lakehouses (e.g., Databricks, Iceberg)
Who owns the storage?	The engine (you don’t touch internal files)	You (you manage the Parquet/ORC files yourself)
Is the file format opaque?	Yes — proprietary columnar or page format	No — Parquet/ORC files are open and readable
Can I use other engines?	No — storage is not meant to be reused	Yes — same files can be queried by many engines

✅ Databases "own" your data. You can’t open a Snowflake or SQL Server file and process it with Spark.

❌ Lakehouses give you full control — your data is accessible and queryable across tools.

2. Data Management Responsibility
Feature	Database	Lakehouse
Query engine tightly bound	Yes — engine/storage are one system	No — engines are separate (Spark, Trino, etc.)
Automatic indexing	Yes (e.g., SQL Server B-trees, metadata)	No (you tune manually or use optimizations)
Automatic stats/catalog	Yes — always updated internally	No — sometimes manual or eventually consistent
ACID enforcement	Centralized via transaction log	File-based log (Delta, Iceberg) across systems

✅ A database manages data placement, indexing, stats, concurrency — you don’t need to think about the physical layer.

❌ In lakehouses, you're expected to optimize and manage file layout, compaction, schema evolution, etc.

3. Tightly Coupled Execution + Metadata + Storage
A true database is a vertically integrated stack:

pgsql
Copier
Modifier
┌──────────────┐
│ SQL Engine   │  <-- tightly coupled
├──────────────┤
│ Query Planner│
├──────────────┤
│ Transaction  │
│ Manager      │
├──────────────┤
│ Metadata     │  <-- stored internally
├──────────────┤
│ Storage Layer│  <-- controlled, not user-facing
└──────────────┘
By contrast, a lakehouse (Databricks + Delta) is decoupled:

scss
Copier
Modifier
┌──────────────┐       ┌──────────────┐
│ Query Engine │ <-->  │ Metadata Log │ (Delta)
└──────────────┘       └──────────────┘
         ↕
   Cloud Storage (you manage the files)
This distinction is critical.

✅ So: Why Is Snowflake a Database and Databricks Not (Strictly Speaking)?
Feature	Snowflake (Database)	Databricks with Delta Lake
Owns internal file storage	✅ Yes — not user-accessible	❌ No — user manages files in S3/ADLS
Proprietary storage format	✅ Yes — columnar, compressed, encrypted	❌ No — open formats like Parquet
One engine to read/write data	✅ Yes (Snowflake engine only)	❌ No — Spark, Trino, Presto, etc.
Tightly coupled metadata	✅ Yes — Snowflake catalogs only	❌ No — Delta log + optional Hive metastore
User file access	❌ No	✅ Yes — full access and control
Query portability	❌ Not portable	✅ Portable across engines

💡 Analogy: Think of a Database as an Appliance
You give it data.

It stores, manages, optimizes, indexes, secures, and queries it.

You don’t see or control the parts inside — it's optimized for one purpose.

A lakehouse is more like a toolbox:

You decide where to store the data.

You decide how to organize it.

You can use different tools (Spark, Trino) to access it.

You're responsible for optimizations (Z-ordering, compaction, etc.).

🔄 But Isn’t Databricks Becoming More Database-Like?
Yes, especially with Unity Catalog, transaction management, and Delta optimizations, Databricks is blurring the line. It's turning its lakehouse into a database-like experience, but:

The underlying architecture remains open and decoupled.

You still own and can access your Parquet files directly.

🔚 Final Summary
What makes something a database isn’t just "it uses files" or "you can query with SQL" — it's:

✅ Central ownership of storage
✅ Tightly integrated query, metadata, and transaction layers
✅ No external access to internal files
✅ Automated data management and optimization
