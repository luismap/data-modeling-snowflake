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

# GENERAL NOTES.
* Snowflake’s scalable consumption-based pricing model requires users to fully understand its revolutionary three-tier cloud architecture and pair it with universal modeling principles to ensure they are unlocking value and not letting money evaporate into the cloud.
