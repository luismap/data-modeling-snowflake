Notes base on the book Data Modeling with Snowflake 2nd edition
buy the book at [amazon](https://www.amazon.com/Data-Modeling-Snowflake-accelerating-development/dp/1837028036)

support the author.

# modeling toolkit components
* natural language semantics - words
* technical semantics - sql
* visual semantics - diagrams

# ***
* common diff between transactional db and dw ![alt text](images/db_dw.png)
* business layer and where modeling happens ![alt text](images/modeling.png)
* principles when modeling ![alt text](images/modeling_principles.png) 

# four types of modeling
* conceptual
* logical
* physical
* transformational

# conceptual modeling
Process of identifying and visually mapping the moving pieces, or **entities**, of a business operation

**entity** is a person, object, place, event, or concept relevant to the business for which an organization wants to maintain information. 

Quoting: "**Conceptual modeling is** a collaborative process of discovery between the business and the data
teams that produce visual project artifacts. Before data about entities can be collected and stored,
the entities themselves must be uncovered and agreed upon. The goal of conceptual modeling is
the synthesis of business and technical understanding between operational and data teams—the
diagram is the by-product. The conversations that take place in the conceptual modeling service
move the organization forward and add value in further stages of design and analysis"

It is **easier to define** a conceptual model by what it is not expected to contain than what it does.
Unlike a physical model, which must have strict database-specific precision to be deployable,
conceptual models include as much or as little detail as needed to view and understand the
details of the model at a high level

## two examples of doing conceptual modeling
* show only entities and relatioship ![alt text](images/cer.png)
* same model augmented with descriptions ![alt text](images/cer_desc.png)
* enhancing with some attributes and cardinality ![alt text](images/cer_enh.png)


# logical modeling
**note**: some literatures condense logical and conceptual into one design.

**Logical modeling** is the bridge between the business’s conceptual operating model and the
physical structure of the database.

Main diff between logical and physical is that the technical details are not specific to any database type.

Logical model includes details such as data types, unique identifiers, and relation-
ships. Contextual notation for subtypes and supertypes, as well as many-to-many relationships,
are also included

* logical design example ![alt text](images/log_ex.png)

the above ER design shows a subtype relationship that is both complete (double parallel lines)
and exclusive (letter X in the center)

**note** some times, to avoid confusion, we can use the chen notation to label an M:M relationship
![alt text](images/log_ex_chen.png)


# physical modeling
contains all the information necessary to construct and deploy the design to a specific database

* example of physical desing ![alt text](images/pm.png)

# transformational modeling
creates new objects by selecting data from existing sources. This is done through views using the CTAS command, or other DML statements. At its core, transformational logic is SQL.
* ex. transformational logic expressed as code ![alt text](images/tm.png)


# snowflake architecture
* A three tier approach (storage, compute, cloud services)
![alt text](images/sf_arch.png)

* compute layer
* storage layer
* service layer

**storage layer physically** "stores data on disks in the cloud provider hosting the Snowflake
account. As data is loaded into Snowflake, it is compressed, encrypted, and logically organized
into tables, schemas, and databases. The users define the logical hierarchy of the database and
its objects, while Snowflake takes care of the underlying partitioning and storage. The customer
is billed only for the data they store, with no provisioning or sizing required"

**compute layer** A virtual warehouse that provides CPU, RAM and t-shirt sizes. Vertical and horizontal scaling available.

**services layer** "coordinates all account activity in this hybrid-layer model and manages
everything from security to encryption to metadata. The services layer handles operations such as
query parsing and optimization, data sharing, and caching. From login, the services layer is there
for the user through every operation they perform in the warehouse. The amount of time-saving
work and automation that the services layer provides is reflected in Snowflake’s marketing language: promising the user near-zero maintenance, along with low cost and exceptional performance."

## snowflake features
* zero copy cloning
* time travel
* hybrid tables (aka.HTAP:hybrid transactional analytical processing)

# cost to consider when using snowflake p.84
* storage cost
* vw compute cost
* serverless cost (snowpipe and query acceleration)
* service layer compute cost
* data transfer cost (ingress no, egress base on the cloud provider)

# save on cost by using cache feature of snowflake
* service layer (use metadata and query result cache)
* vw cache (uses vw disk to cache data, will be purge when vw stops)


**note** on query result cache to activate, the queries needs to:
* be syntactically equivalent
* Dynamic functions such as current_date() are not used
* The data in the underlying tables has not changed
* Users have the required permissions to access the underlying sources

**note** on vw cache
* balance cost of keeping vw alive to reuse cache VS just reading data

Query result cache inner working.
* query -> cache with a TTL of 24 hours
* new query -> cache hit (under the firs 24 hours) -> set TTL of query cache to 31 days

**note**: Snowflake runs entirely on virtually provisioned resources from cloud platforms (Amazon, Micro-
soft, and Google Cloud). Snowflake handles all interactions with the cloud provider transparently,
abstracting the underlying virtual resources and letting the customer manage their data through
a unified three-layer architecture

# snowflake objects
**stages:** logical objects that abstract cloud filesystems so they can be used in a standard manner to
load data into Snowflake. ![alt text](images/external_internal.png)

    - can be internal or external
    - interal uses the hosting provider storage
    - external uses (s3, gcp buckets, azure containers)

**tables:** physical tables logically grouped by schemas. Consist of columns with names, data types, and 
optional constraints and properties.

    - can be permanent/transient/temporary
retention period for tables ![alt text](images/retention_period.png)

**Stage metadata tables:** External and directory tables exist in Snowflake to allow users to access data in staged files as though selecting from regular tables.

    - external: enable users to access staged file contents as if they were stored in regular tables.
    - directory: External tables permit users to access data stored in external cloud storage using the same conventions as regular physical tables, but directory tables only show file metadata.

**views:** store a SELECT statement over physical objects as an object in a schema.

**materialized views:** physical tables that store the results of a view.
When to use materialized views:

    - The query results do not change often
    - The view results are used significantly more frequently than data changes occur
    - The query consumes a lot of compute resources

**streams:** logical objects that capture data changes in underlying sources, including (physical tables, views, and external and directory tables). Whenever a DML operation occurs in the source object, a stream tracks the changes (inserts, deletions, and the before/after images of updates)

When a stream is created, metadata columns are tacked onto the source object and begin tracking
changes. ![alt text](images/stream_metadata_columns.png)

Streams properties:

    - consuming from stream, clears buffers
    - failed consumption does not clear the buffer
    - default retention period 14 days

![alt text](images/stream_work.png)

Type of Streams

    - Standard (or delta): Records inserts, deletes, and updates. This type of stream is supported for tables, directory tables, and views
    - Append/insert-only: Only tracks inserted records and ignores any updates or deletions. This stream type is supported for physical tables, directory tables, and views as append-only and, for external tables, as insert-only

**change tracking**: hange tracking is enabled directly on tables, allowing Snowflake users to query CDC metadata. Change tracking uses the same metadata fields found in streams but appends them directly to a table. Unlike streams, the changes are not eliminated if they are used to update downstream objects; instead, the change tracking persists for the data retention time of the table.

**tasks**: schedule and automate data loading and transformation. Tasks automate data pipelines by executing SQL in serial or parallel steps. Tasks can be combined with streams for continuous ETL workflows to process recently changed table rows. This can be done serverlessly (using auto-scalable Snowflake-managed compute clusters that do not require an active warehouse) or using a dedicated user-defined warehouse.

* task tree with serial and parallel dependencies ![alt text](images/parallel_ser_task.png)

# Logical concepts to snowflake objects

* how snowflake stores data

    - using immutable micropartitions
    - size of 50-500 mb
    - micropartitions metadata allows optimizer to prune
    - micropartitions creation depend on the order data is loaded
    - 

ex. snowflake data logical view ![alt text](images/sf_data_log_view.png)
ex. micropartion view (micro-partitions are created, 
and the data is compressed using a columnar format)![alt text](images/micropartition_view.png)
ex. partition pruning ![alt text](images/part_pruning.png)

* snowflake service stores clustering statistics like:

    - The total number of micro-partitions in each table
    - The number of micro-partitions containing values that overlap with each other (in a specified subset of table columns)
    - The depth of the overlapping micro-partitions

* benefits of using a clustering key

    - Improved query performance through pruning by skipping data that does not match query filters
    - Better column compression and reduced storage costs
    - Maintenance-free automated reclustering performed by Snowflake

**IMPORTANT** only consider setting a clustering key if performance is prioritized over cost or the clustering performance offsets the credits required to maintain it.

* snowflake semi-structure data types

    - variant: hierarchical semi-structure data like (json, avro, parquet, orc. can also store all other data types but variant)
    - object: key[varchar], value[variant]
    - array

* constraints an enforcement:

    - constraints: primary key, unique, foreign key, not null
    - hybrid tables enforce all contraints.
    - standard table enforce only not null

* keys taxonomi

    - business key: hold info relevant to business
    - surrogate key: mainly holds process meaning
    - sequential surrogate key: auto sequence.

```sql
--sequence example
create or replace sequence seq1;
create or replace table foo (k number default seq1.nextval, v number);

-- insert rows with unique keys (generated by seq1) and explicit values
insert into foo (v) values (100);
insert into foo values (default, 101);
-- insert rows with unique keys (generated by seq1) and reused values.
-- new keys are distinct from preexisting keys.
insert into foo (v) select v from foo;
-- insert row with explicit values for both columns
insert into foo values (200, 202);
select * from foo;
+------+------+
| K | V |
|------+------|
| 1 | 100 |
| 2 | 101 |
| 3 | 100 |
| 4 | 101 |
| 200 | 202 |
+------+------+

--if you want to guarantee contigous sequence use autoincrement or identity
CREATE TABLE BAR
(
k number NOT NULL AUTOINCREMENT START 10 INCREMENT 5,
v number NOT NULL
);

--creating fk
CREATE OR REPLACE TABLE employee
(
employee_skey integer NOT NULL AUTOINCREMENT START 1
INCREMENT 1,
employee_bkey varchar(10) NOT NULL,
name varchar NOT NULL,
social_security_id number(8,0) NOT NULL,
healthcare_id integer NOT NULL,
birth_date date NOT NULL,
CONSTRAINT pk_employee_skey PRIMARY KEY ( employee_skey ),
CONSTRAINT ak_employee_bkey UNIQUE ( employee_bkey ),
CONSTRAINT ak_healthcare_id UNIQUE ( healthcare_id ),
CONSTRAINT ak_ss_id UNIQUE ( social_security_id )
);
CREATE OR REPLACE TABLE employee_of_the_month
(
month date NOT NULL,
employee_bkey varchar(10) NOT NULL,
awarded_for varchar NOT NULL,
comments varchar NOT NULL,
CONSTRAINT pk_employee_of_the_month_month PRIMARY KEY ( month ),
CONSTRAINT fk_ref_employee FOREIGN KEY ( employee_bkey ) REFERENCES employee ( employee_bkey )
);

```

# advance snowflake objects
* iceberg tables. 

    - can be managed(catalog inside snowflake) or unmanaged (catalog outside snowflake services, currently only read mode when accessing from snowflake)
    - if manage, make sure only write through snowflake

* **Snowflake managed Iceberg**: The organization operates in a multi-cloud and multi-team environment and wants interoperability with file formats along with DML operations, schema evolution, time travel, and the ability to report on data stored in Iceberg in Snowflake with performance demands on par with native tables.

* **Externally managed Iceberg**: The organization operates in a multi-cloud and multi-team environment, and another cloud provider or catalog is already established as the owner of the datasets being stored in Iceberg. The data team wants to report on data stored in Iceberg in Snowflake with performance demands that exceed those of external tables.

* **External tables**: The data team wants to provide read-only access to data stored in exter- nal stages in various supported formats, as if the data were actually stored in Snowflake.

* **Standard tables**: This is the best practice recommendation when none of the above conditions are met, which offers the greatest flexibility and performance when working with data in the Snowflake platform.

## Dynamic tables: 
integrate transformational logic with data pipelines into a single, self-orchestrating object.
[dynamic table info](https://docs.snowflake.com/en/user-guide/dynamic-tables-about)

* refresh types

    - incremental, full, auto.

* controller pattern using dynamic tables.

    - ensure all tables are set to refresh target_lag=dowstream
    - create dummy controller table, target_lag=dowstream as well.
    - create task to update dynamic table

dynamic pipeline controller pattern![alt text](images/dynamic_pipeline.png)

```sql
CREATE DYNAMIC TABLE leaf1
...
;
...
CREATE DYNAMIC TABLE leafN
...
;

CREATE DYNAMIC TABLE controller
TARGET_LAG = DOWNSTREAM
WAREHOUSE = <my dedicated warehouse>
AS
SELECT 1 AS dummy FROM <leaf1>, …, <leafN> LIMIT 0;

CREATE OR REPLACE TASK daily_refresh
WAREHOUSE = <same as dynamic table warehouse>
SCHEDULE = <required schedule> --ex. 'USING CRON 0 6 * * * UTC'
AS
ALTER DYNAMIC TABLE controller REFRESH;
```

* comparison of dynamic tables, streams and tasks ![alt text](images/dt_s_task_comp.png)

indexes

    - primary key and unique create their own indexes
    - use include when creating and index to include a subindex
    - subindex is more efficient than creating a new index

```sql
CREATE OR REPLACE INDEX indexName ON tableName (col1, [col2]);

CREATE HYBRID TABLE books (
book_id INT PRIMARY KEY,
book_title STRING,
book_category STRING,
INDEX idx_category (book_category) INCLUDE (book_title)
);
SELECT book_title FROM books WHERE book_category = 'Fiction';
```

quick thoughts about indexes

    - Indexes increase storage costs by saving additional copies of a subset of the data.
    - While they accelerate data retrieval, indexes add overhead to DML operations because they need to be updated synchronously as changes occur.
    - Understanding the criticality and frequency of commonly run queries is important to determine whether an index justifies the added costs and DML overhead.

when will an index be use

    - A primary index is used: TableScan with Scan Mode = ROW_BASED is visible in the query profile.
    - A secondary index is used: IndexScan with Scan Mode = ROW_BASED is visible in the query profile. The index name and access predicate details are also available.
    - Column store is used: The query reads from a column store copy of the dataset. Index scans can also access object storage for Time Travel queries.

## hybrid tables

hybrid tables load types

    - Bulk optimized load: Can only be performed the first time a hybrid table is created
    - Incremental batch load: All subsequent loads use the non-optimized batch method, typically allowing 1 million records per minute.

* hybrid vs standard tables ![alt text](images/hybrid_standard.png)

## event tables
solution to collecting and managing telemetry data

    - By default, all Snowflake accounts include an event table called SNOWFLAKE.TELEMETRY.EVENTS
    - Telemetry data comes from: Stored procedures/ Streamlit apps/ User-defined functions/ User-defined table functions


# snowflake arch in modeling notation
* example of a digram in chen notation ![alt text](images/chen_diag.png)
* example of physical model using crows's foot notation ![alt text](images/crows_foot.png)

note: the modeling conventions presented here will be a collage of multiple standards, the aim
is to make is easily synchronizable between all phase of modelling (CLPT)

## notations that will be used
entities

    - perpendicular corners in entities: strong entity
    - rounded corners: weak entity

ex. ![alt text](images/weak_strong.png)

relationships

* relationships crow's notation ![alt text](images/crows_notation.png)
* reading crow notation ![alt text](images/crow_not_read.png)
* ex sync logical and physical model ![alt text](images/sync_log_phy.png)

subtypes and supertypes

subtypes realationships can be 

    - Complete/incomplete: Indicates whether all the subtypes are known or whether more can be added in the future, respectively.
    - Inclusive/exclusive: Indicates whether the supertype can exist as multiple subtypes.
ex. table depicting all posible subtypes relationship ![alt text](images/subtypes_rel.png)

* example of a model in CLP diagram ![alt text](images/clp.png)

# conceptual modeling into practice
main point is that it focuses on capturing business entities and their relationships.

quick reminder on kimballs multidimensional modeling process

    - Define the business process
    - Declare the grain
    - Identify the dimensions
    - Identify the facts

* A bus matrix example ![alt text](images/bus_matrix.png)
* conceptual model showing only dimensions ![alt text](images/concep_dim.png)
* full conceptual model ![alt text](images/full_concept.png)

Reverse engineering a model. Usually will give you a physical model.
start by working in reverse of the kimball modeling process.

    - classify each table into facts and dimensions
    - find primary keys
    - find fk an stablish relationships
    - establish cardinality
    - draw the physical model

# logical modeling into practice
when going from conceptual to logical. key questions

    - how company identifies entity X (do this for all entities)
    - assure by talking to business experts that facts grains are correct
    - find out the measures of the fact tables

![alt text](images/concept_to_logical.png)
![alt text](images/fact_measures.png)

In a many-to-many relationship, if the fact entity needs to be audited by
its own. Then a new strong entity can be created.
ex.
* m-to-m orders, order need to be tracked alone ![alt text](images/orders.png)
* create new weak entity to cover the usecase. line entity created ![alt text](images/line_entity.png)

* to highlight m-to-m relationship in logical model, use diamond notation ![alt text](images/diamond.png)

## Inheritance
* a customer can be (loyal, or just customer) ![alt text](images/customer.png)
* expanding diagram with inheritance ![alt text](images/cust_inherit.png)

* a complete logical model ![alt text](images/complete_log.png)


# GENERAL NOTES.
* Snowflake’s scalable consumption-based pricing model requires users to fully understand its revolutionary three-tier cloud architecture and pair it with universal modeling principles to ensure they are unlocking value and not letting money evaporate into the cloud.
