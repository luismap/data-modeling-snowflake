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

# general notes.
* Snowflake’s scalable consumption-based pricing model requires users to fully understand its revolutionary three-tier cloud architecture and pair it with universal modeling principles to ensure they are unlocking value and not letting money evaporate into the cloud.
