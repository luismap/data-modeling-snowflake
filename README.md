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



# GENERAL NOTES.
* Snowflake’s scalable consumption-based pricing model requires users to fully understand its revolutionary three-tier cloud architecture and pair it with universal modeling principles to ensure they are unlocking value and not letting money evaporate into the cloud.
