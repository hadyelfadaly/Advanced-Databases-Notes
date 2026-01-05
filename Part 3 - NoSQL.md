# Background

- Relational databases are the traditional and most widely used databases for core business applications because they provide strong consistency and transactional guarantees (Relational databases → mainstay of business)
- Web-based applications caused spikes
    - explosion of social media sites (Facebook, Twitter) with large data needs and various data types such as images, videos, and documents.
    - rise of cloud-based solutions such as Amazon S3 (simple storage solution)
    - Hooking RDBMS to web-based application becomes trouble due to big data.
- Current Day Trends
    - **Big Data** → huge volume, variety, velocity
    - **Cloud Computing** → elastic resources, on-demand infrastructure
    - **Solid State Disks (SSD)** → much faster I/O than HDDs 
- And all of these increase system demands: More data, more users, more requests and lower latency expectations
- Solution? Scaling up (**Vertical Scaling**): Increase the power of a single machine. (more RAM, more CPU cores, more SSDs, ...etc)
    - Still  with Big Data + Cloud, scale-up: becomes expensive, has physical limits does not scale infinitely (dataset is just too big).
- Solution? Scaling out (**Horizontal Scaling**): adding more smaller/cheaper servers) is a better choice, And there are Different horizontal scaling approaches (multi-node database).

## Master/Slave

- All writes are written to the master 
- All reads performed against the replicated slave databases
- Critical reads may be incorrect as writes may not have been propagated down
- Large datasets can pose problems as master needs to duplicate data to slaves
![Pasted image 20260103222345.png](./Imgs/Pasted%20image%2020260103222345.png)

### Multi-Master replication

- More than one master node
- Any master can accept writes
- Changes are replicated between masters, then to their replicas
- In multi-master systems, conflicts are dangerous. If two masters update or delete the same record at the same time, conflict resolution becomes complex, To avoid conflicts:
    - Systems prefer append-only (INSERT-only) models
    - Old data is not modified → new versions are added instead
- When is Multi-Master used?
    - If you want High availability  
    -  If you want Global systems (users write from different locations)  
    -  If you want Fault tolerance  
    - don't use it if you want Strong consistency (hard to guarantee)
- No JOINs, thereby reducing query time. Because JOINs require: 
    - Consistent schemas
    - Data across nodes
  In distributed systems:
    - Data is partitioned & replicated
    - JOINs become slow and expensive 
    So:
     - Data is denormalized
     - Related data is stored together
     - Queries are faster and simpler
![Pasted image 20260103223326.png](./Imgs/Pasted%20image%2020260103223326.png)

## Sharding

- **Sharding** (Partitioning) involves partitioning the data across multiple databases based on a key attribute. Example:
    - Users with IDs 1–1M → Shard 1
    - Users with IDs 1M–2M → Shard 2
- Scales well for both reads and writes
    -  Reads: queries hit only one shard instead of the whole database
    - Writes: inserts/updates are distributed across shards
  Load is spread → higher throughput
- Not transparent (The database does not automatically know where data lives), application needs to be partition-aware
    -  The application must: Choose the shard and Route queries to the correct shard -> More logic in application code -> More complexity
- The biggest drawback is it can no longer have relationships/joins across partitions (loss of referential integrity across shards). As Foreign keys cannot span shards. Data is often denormalized to avoid joins.
- In Summary: Sharding improves scalability but sacrifices transparency, joins, and referential integrity across shards.
![Pasted image 20260103224333.png](./Imgs/Pasted%20image%2020260103224333.png)

## Other ways to scale out RDBMS

- **In-memory databases**
    - Primarily relies on main memory for data storage.
    - Faster than disk-optimized databases because disk access is slower than memory access and the internal optimization algorithms are simpler and execute fewer CPU instructions.
    - Accessing data in memory eliminates querying seek time.
- **NewSQL**
    - Is a database that retain main key characteristics of RDBMS but different from the common architecture exhibited by Oracle and SQL.
    - There are two designs
        1. **H-Store:** pure distributed In-Memory database.
        2. **C-Store:** Involves columnar database design


Database technology evolved from rigid centralized systems → relational databases → distributed, scalable, cloud-based data platforms. RDBMS alone is no longer enough, so new models appeared.

## What is the main challenge of the traditional databases?

The main challenge of traditional relational databases is handling semi-structured and unstructured data due to rigid schemas and limited flexibility.

- Relational databases are actually very good at:
    - Structured data
    - Tables, rows, columns
    - SQL, indexes, joins
- They can manage large volumes of structured data, especially with:
    - Indexing
    - Partitioning
    - OLAP systems (with ETL for example)
    So size alone is not the core problem.
- If data could be analyzed “as it comes”, relational DBs could work
- But the real bottleneck is:
    - Memory
    - Flexibility
    - Schema rigidity

- Traditional relational databases struggle with:  **Data variety**, not just volume
    - Semi-structured data (JSON, XML)
    - Unstructured data (logs, text, images, social media data)
    - Frequently changing schemas
    - Data arriving continuously (streams)
- Relational databases require:
    - Fixed schema
    - Schema design upfront
    - Costly schema changes
    This is exactly where **NoSQL** databases shine.
# What is NOSQL?

- Stands for "Not Only SQL"
- The term NOSQL was introduced by Carl Strozzi in 1998 to name his file-based database, It was again re-introduced by Eric Evans when an event was organized to discuss open source distributed databases. Eric states that “… but the whole point of seeking alternatives is that you need to solve a problem that relational databases are a bad fit for. …”
- Key Features (Advantages):
    - Non-relational and doesn’t require schema. (Perfect for unstructured / semi-structured data (JSON, logs, social data).)
    - Data are replicated to multiple nodes (fault-tolerant): down nodes easily replaced and no single point of failure (Handles huge growth without redesigning the database)
    - Horizontal scalable 
    - cheap, easy to implement
    - Massive write performance
    - Fast key-value access
- Disadvantages::
    - Don’t fully support relational features (no join, group by, order by operations (except within partitions) and no referential integrity constraints across partitions
    - No declarative query language like SQL so requires more programming
    - No easy integration with other applications that support SQL
    - Relaxed ACID (see CAP theorem) → fewer guarantees

## Brewer’s CAP Theorem:

- **CAP theorem** says for any system sharing data, it is “impossible” to guarantee simultaneously all of these three properties:
    - Consistency (C): All client always have the same view of the data
    - Availability (A): Each client always can read and write
    - Partition tolerance (P): A system can continue to operate in the presence of a network partitions
- You can have at most two of these three properties for any shared-data system.
- Very large systems will “partition” at some point That leaves either C or A to choose from (traditional DBMS prefers C over A and P)
- NoSQL systems usually choose: **Availability + Partition tolerance (AP)** or sometimes **Consistency + Partition tolerance (CP)** 
    Example: Apache Cassandra focus on AP, while MongoDB and Kafka focus on CP
- Compared to relational databases:
    - Data may be temporarily inconsistent
    - Reads may return slightly stale data
    - Transactions may be eventually consistent, not immediately consistent
- Example: Client `A` writes `X` to `Node 1`, Client `B` reads `X` from `Node 3`, Will it be the last value of `X`? - Maybe, however eventual consistency occurs.
- **Eventual consistency** allows temporary inconsistency but guarantees that all replicas will become consistent over time.
    - Updates do not appear immediately everywhere
    - Different nodes may show different values for a short time
    - After some time → all nodes agree
    Why it exists
    -  Used in distributed / NoSQL systems
    - Helps achieve high availability and scalability
    - Follows the CAP theorem (AP systems)
    Example: You update your profile picture, One server shows the new picture, Another still shows the old one, After a while → all show the new picture.
![Pasted image 20260103234727.png](./Imgs/Pasted%20image%2020260103234727.png)

## NO-SQL categories

1. Key-value. Example: DynamoDB, Voldermort, Scalaris
2. Document-based. Example: MongoDB, CouchDB
3. Column-based. Example: BigTable, Cassandra, Hbased
4. Graph-based. Example: Neo4J, InfoGrid

## Are NoSQL databases better than relational databases?

NoSQL databases are not “better” than relational databases in general. They are better for different use cases. The choice depends on the application requirements, not superiority.

- **Relational Databases (RDBMS)**
    - Best for structured data
    - Strong ACID guarantees
    - Complex joins and transactions
    - Examples: banking systems, ERP, accounting

- **NoSQL Databases**
    - Best for large-scale, distributed, fast-changing data
    - Horizontal scalability
    - Relaxed consistency (eventual consistency)
    - Schema-flexible
    - Examples: social media, IoT, big data analytics

Relational databases remain the backbone of traditional business applications, while NoSQL databases complement them by addressing scalability, distribution, and big data challenges.
# Key-value database 

**key-value database** is a system that stores values indexed by keys. It can store structured and unstructured data.

- Focus on scaling to huge amounts of data designed to handle massive data loads
- Data model: (global) collection of Key-value pairs.
- Example: (DynamoDB)
    - Items (rows) having one or more attributes (name, value)
    - An attribute can be single-valued or multi-valued like set.
    - items are combined into a table
![Pasted image 20260104201411.png](./Imgs/Pasted%20image%2020260104201411.png)

- DynamoDB is **schema-flexible**: items in the same table can have different attributes, and attributes can be multi-valued.
    - { "UserID": "1", "Name": "Ali" }
    - { "UserID": "2", "Email": "ali@mail.com" }
    Both are valid in the same table
- Pros:
     - very fast
     - very scalable (horizontally distributed to nodes based on key)
         - UserID = 101  → Node A
         - UserID = 205  → Node B
         - UserID = 999  → Node C
     - simple data model
     - eventual consistency
     - fault-tolerance
- Cons:
     - Can’t model more complex data structure such as objects

## Key-value Database API Functions

- `Get(key)`: extract the value given a key
- `Put(key, value)`: create or update the value given its key
- `Delete(key)`: remove the key and its associated value
- `Update(key, value)`: create or update the value given its key
- `Execute(key, operation, parameters)`: uses the key to locate a value that is a complex data structure (e.g. List, Set, Map .... etc), then applies an operation directly on that structure.
    - Assume: `key = "user123" value = List of friends`
    - Call: `Execute("user123", add, "Ali")` 

## Key-value Platforms

All key–value platforms prioritize fast access by key, but differ in how complex the value is and what operations they allow on it.

| **Name**                         | **Data Model**                                                                                                                                                                           | **Querying**                                                                                                  |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **SimpleDB (Amazon)**            | Key → set of attributes, Each attribute = `(name, value)`, So the value is not a single thing, but a small record.<br>Example:<br>User123 → (age, 22)<br>                  (city, Cairo) | Very limited SQL-like commands, Mostly: SELECT, DELETE, GETATTRIBUTES AND PUT ATTRIBUTES<br><br><br>          |
| **Redis (Salvatore Sanfilippo)** | set of couples `(key, value)`, where value is simple typed value (String, Int), list, ordered (according to ranking) or unordered set, hash value                                        | No SQL, Each data type has its own operations.<br>Redis is data-structure–oriented, very fast, in-memory.     |
| **Dynamo (Amazon)**              | like SimpleDB                                                                                                                                                                            | simple get operation and put in a context (partition, node, etc.).<br>Built for massive scale + availability. |
| **Voldemort (LinkedIn)**         | like SimpleDB                                                                                                                                                                            | Similar to Dynamo                                                                                             |

# Big Data 

## Google

![Pasted image 20260104202212.png](./Imgs/Pasted%20image%2020260104202212.png)

- Explosive growth of data, The graph shows the number of websites from 1990 to 2008.
    - Early years → very slow growth
    - Around late 1990s → growth accelerates
    - After 2000 → exponential increase
    This means: The amount of web data Google has to crawl, index, and search grew insanely fast.
- Google needs to:
    - Store billions of web pages
    - Index them
    - Search them in milliseconds
    - Continuously update them
- Traditional relational databases:
    - Expect fixed schemas
    - Rely on centralized or limited scaling
    - Don’t scale easily to this magnitude
    They don’t fit Google’s scale.
- This growth led Google to:
    - Use distributed systems
    - Store data across thousands of machines
    - Prefer scale-out architectures
    - Inspire the Google Stack Software

### Google Stack Software

- Google developed major software layers as foundation for google platform:

1. **Google File System** (GFS): a distributed cluster file system that allows all of the disks within the Google data center to be accessed as one massive, distributed, redundant file system.
2. **MapReduce**: a distributed processing framework that executes algorithms in parallel across many nodes, provides fault tolerance, and efficiently processes massive datasets.
    - **Map phase**
        - Each node processes a chunk of data
        - Produces `(key, value)` pairs
    - **Reduce phase**
        - Groups same keys together
        - Aggregates results
3. **BigTable**: a nonrelational database system that uses the Google File System for storage

![Pasted image 20260104203629.png](./Imgs/Pasted%20image%2020260104203629.png)
![Pasted image 20260104204129.png](./Imgs/Pasted%20image%2020260104204129.png)
![Pasted image 20260104202917.png](./Imgs/Pasted%20image%2020260104202917.png)

## Hadoop and Hive

Hive provides an SQL-like interface that translates queries into MapReduce jobs executed by Hadoop to process large datasets.

1. User writes SQL
    - The user does NOT write Java or MapReduce instead they write **HiveQL** (very similar to SQL)
2. Hive translates SQL into MapReduce
    - Hive is NOT a database, It is query layer / abstraction, It converts SQL queries into MapReduce jobs.
    - These jobs are written/executed in Java
3. Hadoop MapReduce executes the job
    - Hadoop:
        - Processes massive data
        - Runs MapReduce in parallel
        - Uses HDFS (Hadoop File Storage System) for storage. 
    - Actual computation happens here, not in Hive
4. Results are returned to the user
    - MapReduce finishes execution
    - Results go back through Hive
    - User sees the final output (tables / results)

- **Hive** → query layer (SQL-like)
- **Hadoop / MapReduce** → processing
- **HDFS** → storage

# Apache Cassandra

- Is a free and open-source distributed NoSQL database management.
- Handles large amounts of data across many commodity servers, providing high availability with no single point of failure. (Uses peer-to-peer)
    1. Client connects to any node (called a coordinator)
    2. That node:
        - Figures out which nodes should store the data
        - Forwards the request if needed
    3. Data is written to multiple nodes (replicas)
    4. Even if one node fails → system still works
- It was started by Facebook and it is an open source Apache project written in Java.
- Cassandra is called a **column-family database** because data is stored as **rows identified by a primary key**, where each row contains a **set of columns grouped into column families**

| **Property**                  | **Cassandra**                               | **RDBMS**                               |
| ----------------------------- | ------------------------------------------- | --------------------------------------- |
| **Core Architecture**         | Masterless (no single point of failure)     | Master-slave (single points of failure) |
| **High Availability**         | Always-on continuous availability           | General replication with master-slave   |
| **Data Model**                | Dynamic; structured and unstructured data   | Legacy RDBMS; Structured data           |
| **Scalability Model**         | Big data/Linear scale performance           | Oracle RAC or Exadata                   |
| **Multi-Data Center Support** | Multi-directional, multi-cloud availability | Nothing specific                        |
| **Enterprise Search**         | Integrated search on Cassandra data.        | Handled via Oracle search               |
| **In-Memory Database Option** | Built-in in-memory option                   | Columnar in-memory option               |
- Advantages:
    1. Cassandra is developed to be a distributed server, but it can also be run as a simple node.
    2. Horizontal scalability (Distributed storage.).
    3. Quick answers even if demand grows.
    4. High write speeds to manage incremental data volumes
    5. Ability to change the data structure.
    6. A simple API for your favorite programming language.
    7. Automatic fault detection and fault tolerant.
    8. There is no single point of failure which means that each node knows about the others.
    9. Decentralized.
    10. Allows the use of Hadoop to use Map Reduce.
- Disadvantages:
     1. **Ad-hoc queries**: Cassandra does not support ad-hoc querying; data must be modeled according to predefined access patterns.
         -  Example If you need:
            - Query by `user_id` → table must be partitioned by `user_id`
            - Query by `email` → you need another table 
            You can’t suddenly ask a new question if the table wasn’t designed for it.
     2. **No-Aggregations**: Cassandra does not efficiently support aggregation operations because data is distributed and not designed for global scans, doing functions like Sum, Min, Max, and Average are incredibly resource intensive if even possible to accomplish.
     3. **Unpredictable performance**: Because Cassandra has many different Asynchronous Jobs in the background(Compaction, repair, replication, Garbage collection, ...etc).

- Alternatives:
     - **PostgreSQL** → Rich queries, joins, ACID
     - **MongoDB** → Flexible schema, easier querying
     - **Redis** → Ultra-fast in-memory key-value
     - **Riak** → Availability & fault tolerance
     - **Cassandra** → Massive scale, high writes, no single point of failure

## Cassandra Database Elements

1. **Clusters**: Container for Keyspaces. It is the outermost structure in Cassandra.
     - Sometimes called the ring, because Cassandra assigns data to nodes in the cluster by arranging them in a ring.
     - A node holds a replica for different range of data
     - **Cassandra Gossip Protocol**: Gossip is the message system that Cassandra nodes, virtual nodes uses to make their data consistent with each other.
        - A node has a data replica. If something goes wrong, a replica can respond. The `replication_factor` parameter in the creation of a Keyspace indicates how many machines in the cluster will receive copies of the same data.
2. **Keyspaces**: The outermost container for data in Cassandra, it corresponds to Database in RDMS but without interrelations (stores data).
     - Like a relational database, a keyspace has a name and a set of attributes that define keyspace-wide behavior.  
     - Keyspace is used to group column families together.
     - The keyspaces require that some attributes be defined, such as user-defined names, replication strategies and others.
     - KeySpaces require configuration according to consistency that are:
        1.  The **replication facto**r which indicates how much do you want to pay performance in favor of consistency.
        2. The **replica placement strategy**, which indicates how the replicas are placed in the ring such as SimpleStrategy, OldNetwork TopologyStrategy, and NetworkTopologyStrategy.
            -  **Simple Strategy**: Cassandra arranges nodes in a ring, One node is chosen as the primary replica Remaining replicas are placed on the next nodes clockwise. It ignores data centers and racks and assumes all nodes are equal. Easy but not safe for production. Used when single data center only. Not Multiple data centers (risk of correlated failures)
            - **Network Topology Strategy**: Aware of data centers and racks. Distributes replicas across different nodes, different racks and different data centers.
                -  one node fails → other nodes have data
                - one rack fails → other racks have data
                - one data center fails → other DC still serves requests
                Used in Production Systems and multi-data center setups.
        - **Coordinator role**: Client sends requests to any node, this nodes becomes the coordinator and forwards write/read to all replica nodes.
3. **Column Family**: Column family is a container of an ordered collection of rows containing an ordered collection of columns.
    -  Cassandra manages columns and family of columns.
    - We can freely add any column to any column family at any time.
    - Table schema must be specified, yet can be modified later on
4. **CQL Table**: It is a column family.
    - Provide two-dimensional view of column family, which contains potentially multi-dimensional data.
    - CQL Table and column family are largely interchangeable terms.
![Pasted image 20260105211311.png](./Imgs/Pasted%20image%2020260105211311.png)

- Instance → keyspaces → tables → rows → columns
- Rows in a table do not need to have the same columns
- Each row is uniquely identified by a primary key

![Pasted image 20260105211859.png](./Imgs/Pasted%20image%2020260105211859.png)

- 1st Table - Static: 
    - Most rows share similar columns
    - Looks similar to a relational table
    - But still not enforced by schema 
    - Even “static-looking” data in Cassandra is still flexible
    - Cassandra does NOT require every row to have all columns
    - Missing columns are simply not stored (no NULLs)
- 2nd Table - Dynamic:
    -  **Row key** = blog owner
    - **Column names** = subscribers
    - **Column values** = usually empty / timestamp / marker
    - Column names are not predefined
    - Each row can have completely different columns
    - Columns are created at write time

### Columns

- Column is the smallest increment of data
    - Name + value + timestamp
    - Value can be empty
- Can be indexed on their name (secondary index), primary index = Row Key
- Types:
    1.  **Expiring** – with optional expiration date called **TTL** (Can be queried)
    2. **Counter** – to store a number that incrementally counts the occurrences of a particular event or process
        -  Operation increment/decrement with a specified value
        - Internally ensures consistency across all replicas
- Column Values:
    -  **Empty value** -> NULL
    - **Atomic value**
        -  **Native data types**
            -  **tInyint , smallint, int, bigint** -> Signed integers (1B, 2B, 4B, 8B)
            - **varint** -> Arbitrary-precision integer
            - **decimal** -> Variable-precision decimal
            - **float , double** -> Floating point numbers (4B, 8B)
            - **boolean** -> Boolean values true and false
            - **text, varchar** -> UTF8 encoded string, Enclosed in single quotes not double qoutes
            - **ascii** -> ASCII encoded string
            - **date, time, timestamp** -> Dates, times and timestamps
            - **counter** -> 8B signed integer
                - Only 2 operations supported: incrementing and decrementing
                - Value of a counter cannot be set to a particular number
                - Counters cannot be a part of a primary key
                - Either all table columns (outside the primary key) are counters, or none of them
                - TTL is not supported
            - **blob** -> arbitrary bytes
            - **inet** -> IP address (both IPv4 and IPv6)
        - **Tuples** -> Tuple of anonymous fields, each of any type (even different)
        - **User defined types** (UDT) -> Set of named fields of any type
    - **Collections** -> Lists, sets, and maps
        -  Nested tuples, UDTs, or collections are also allowed, but currently only in a frozen mode
        - **List** -> Ordered collection of non-unique values (order based on index value). 
            - Certain limitations and performance issues unfortunately exist (Internal read-before-write operations must be executed)
        - **Set** -> Ordered collection of unique values (Order based on values)
        - **Map** -> Ordered collection of key-value pairs (Order based on keys and they must be unique) Each element is internally stored as one Cassandra column and Each element can have an individual time-to-live
- **Time-to-live** (TTL) -> After a certain period of time (number of seconds) a given column / element is automatically deleted
- **Timestamp** (write time) -> Timestamp of the last modification, Assigned automatically or manually as well
- Joining column families at query time is not supported We need to pre-compute the query / use a secondary index
- TTL is added after the insertion of the row that you want to specify the TTL for and it can be queried
```CQL
INSERT INTO excelClicks (userid, url, date, name)VALUES
(3715e600-2eb0-11e2-81c1-0800200c9a66, 'http://apache.org', '2013-10-09', 'Mary') USING TTL 86400; --HERE its specified
```
```CQL
SELECT TTL (name) from excelClicks --HERE its queired
WHERE url = 'http://apache.org' ALLOW FILTERING;
```

## When Not to Use

- Systems that Require **ACID** Transactions 
    -  Column-family stores are not just a special kind of RDBMSs with variable set of columns.
- Aggregation of the Data Using Queries
    - Have to be done on the client side
- For Early Prototypes
    - We are not sure how the query patterns may change
    - As the query patterns change, we have to change the column family design

## Cassandra API

-  **CQLSH**:
    - Interactive command line shell
    - bin/cqlsh
    - Uses CQL (Cassandra Query Language)
- **Client drivers**
    - Provided by the community
    - Available for various languages -> Java, Python, Ruby, PHP, C++, Scala, Erlang, …

## CQL (Cassandra Query Language)

CQL offers a more than close to SQL to create schema and manipulate data.

- Declarative query language inspired by SQL

Some of the features CQL has are:
-  Data types
- Security
- Data definition
- Functions
- Data manipulation
- Arithmetic operations
- Secondary indexes
- JSON support
- Materialized views 
- Triggers 

### DDL Statements

#### KEYSPACES

##### CREATE
![Pasted image 20260105214540.png](./Imgs/Pasted%20image%2020260105214540.png)
- Example: 
```CQL
CREATE KEYSPACE moviedb
WITH repl icat ion = { 'class' : 'SimpleStrategy' , ' replication_ factor': 3}
```

- Replication option is mandatory
    - SimpleStrategy (only one replication factor)
    - NetworkTopologyStrategy (individual replication factor for each data center)

##### USE

Set a keyspace as the current working keyspace

- Syntax:
```CQL
USE KEYSPACE KEYSPACENAME;
```

##### DROP

Removes a keyspace, all its tables, data etc.

- Syntax:
```CQL
DROP KEYSPACE KEYSPACENAME IF EXISTS; --IF EXISTS optional
```

##### ALTER

Modifies options of an existing keyspace

- Example Syntax:
```CQL
ALTER KEYSPACE Excel
WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 4};
```

#### TABLES

##### CREATE

Creates a new table within the current keyspace

- Each table must have exactly one primary key specified
![Pasted image 20260105215234.png](./Imgs/Pasted%20image%2020260105215234.png)

- Example:
```CQL
CREATE TABLE movies (
id TEXT,
title TEXT,
director TUPLE<TEXT, TEXT>,
year SMALLINT,
actors MAP<TEXT, TEXT>,
genres LIST<TEXT>,
countries SET<TEXT>,
PRIMARY KEY ( id)
)
CREATE TABLE actors (
id TEXT PRIMARY KEY,
name TUPLE<TEXT, TEXT>,
year SMALLINT,
movies SET<TEXT>
)
```

- Primary key has two parts:
    -  **Compulsory partition key** 
        - Single column or multiple columns
        - Determines how rows are distributed in a cluster
    - **Optional clustering columns** -> Defines the clustering order, i.e. how table rows are locally stored within a given shard

##### DROP

Removes a table together with all data it contains

```CQL
DROP TABLE TABLENAME IF EXISTS; --IF EXISTS OPTIONAL
```

##### TRUNCATE

Preserves a table but removes all data it contains

```CQL
TRUNCATE TABLE TABLENAME; --TABLE OPTIONAL
```

##### ALTER

Allows to alter, add or drop table columns

##### CREATE INDEX

Create a (secondary) index

- Allow efficient querying of other columns than key
- Example Syntax:
```CQL
CREATE INDEX userIndex ON timeline (posted_by);
```

##### DROP INDEX

Drop an index

- Syntax:
```CQL
DROP INDEX userIndex;
```

### DML Statements
#### SELECT
Selects matching rows from a single table

![Pasted image 20260105221459.png](./Imgs/Pasted%20image%2020260105221459.png)

- Example:
```CQL
SELECT i d , t i t l e , actors
FROM movies --KEYSPACENAME.movies OPTIONAL
WHERE year = 2000 AND genres CONTAINS 'comedy'
```
- joining of multiple tables is not possible

- WRITETIME and TTL:
    -  Selects modification timestamp / remaining time-to-live of a given column
    - Cannot be used on collections and their elements
    - Cannot be used in other clauses (e.g. WHERE)

##### WHERE

- One or more conditions a row must satisfy in order to be included in the query result
- Only simple conditions can be expressed and not all conditions are allowed, e.g.:
     -  only primary key columns can be involved unless secondary index structures exist
     - non-equal relations on partition keys are not supported
     - ....
![Pasted image 20260105222816.png](./Imgs/Pasted%20image%2020260105222816.png)

- Relations:
    -  **Comparisons** -> =, !=, <, <=, =>, >
    - **IN** -> Returns true when the actual value is one of the enumerated
    - **CONTAINS** -> May only be used on collections (lists, sets, and maps) and Returns true when a collection contains a given element
    - **CONTAINS KEY** -> May only be used on maps and Returns true when a map contains a given key

##### ORDER BY

Defines the order of rows returned in the query result 

- Cassandra does not support arbitrary sorting. It only supports ordering that already exists due to clustering columns.

##### GROUP BY

Groups rows of a table according to certain columns

- In Cassandra, GROUP BY is restricted to primary key columns because data is physically partitioned and ordered only by the primary key, and Cassandra does not perform runtime grouping or data reshuffling across partitions.
- In Cassandra, when a non-grouping column is selected without aggregation, the database returns the first value encountered because grouping is based solely on the physical primary key order and no runtime aggregation or reshuffling is performed. Cassandra allows this for efficiency, not correctness.

##### ALLOW FILTERING

- By default, only non-filtering queries are allowed I.e. queries where the number of rows read ∼ the number of rows returned
- Example:
    - Assume: `PRIMARY KEY ((user_id), time)` 
    - Allowed: `SELECT * FROM posts WHERE user_id = 5;` and `SELECT * FROM posts WHERE user_id = 5 AND time > 100;
    - Forbidden: `SELECT * FROM posts WHERE time > 100;`
        - Cassandra would need to:
            - scan all partitions
            - check each row
            - discard most rows
        - Rows read ≫ rows returned -> Unpredictable and dangerous
- ALLOW FILTERING enables (some) filtering queries means Cassandra lets the query run but give no performance gurantees
    - This would run: `SELECT * FROM posts WHERE time > 100 ALLOW FILTERING;` 
    - This may:
        - Scan many nodes
        - Read huge amounts of data
        - Kill performance in production
    -  `ALLOW FILTERING` = “I know this might be expensive, do it anyway”

#### INSERT

Inserts a new row into a given table, When a row with a given primary key already exists,
it is updated.

- Values of at least primary key columns must be set
- Names of columns must always be explicitly enumerated
![Pasted image 20260105225319.png](./Imgs/Pasted%20image%2020260105225319.png)

- Example:
```CQL
INSERT INTO movies ( id , title , director , year, actors , genres)
VALUES ('stesti', 'Štěstí' , ('Bohdan' , 'Sláma' ), 2005,
{ 'vi lhelmova' : 'Monika' , ' liska ' : 'Toník' }, [ 'comedy', 'drama' ]
)
USING TTL 86400
```

#### UPDATE

Updates existing rows within a given table, When a row with a given primary key does not yet exist,
it is inserted. 

- At least all primary key columns must be specified in the WHERE clause
![Pasted image 20260105225623.png](./Imgs/Pasted%20image%2020260105225623.png)

- Example:
```CQL
UPDATE movies SET
year = 2006,
director = ( 'Jan' , 'Svěrák' ) , M--TUPLE
actors = { 'machacek': 'Robert Landa' , 'sverak' : 'Josef' } , --MAP
genres = [ 'comedy' ] , -- L i s t
countries = { 'CZ' } --Set
WHERE id = 'vratnelahve'

UPDATE movies SET
actors[ 'vilhelmova' ] = 'Helenka' ,
genres[1] = 'comedy' WHERE id = 'vratnelahve'

UPDATE movies
SET
actors = actors + { 'vilhelmova' : 'Helenka' } ,
genres = [ 'drama' ] + genres,
countries = countries + { 'SK' }
WHERE id = 'vratnelahve'

UPDATE movies
SET
actors = actors - { 'vilhelmova' , ' landovsky' } ,
genres = genres - [ 'drama' , ' sci - fi ' ] ,
countries = countries - { 'SK' }
WHERE id = 'vratnelahve'
```
##### Assignments

Describe modifications to be applied

- Allowed assignments:
    - Value of a whole column is replaced
     Value of a list or map element is replaced
     Value of an UDT field is replaced

# Columnar Database Systems

- Stores content by columns rather than row, great for analytics (SUM, AVG, COUNT).
- 2-D data at conceptual level → 1-D at physical level
     -  **Conceptual level**: You still see a normal table (rows × columns)
     - **Physical level**: Data is actually stored as separate 1-D arrays, one per column
     - Example: 
        - Table: (id, age, salary)
        - Physical storage: id → [1,2,3,4] age  → [20,25,30,35] salary → [3000,4000,5000,6000]
- Row-by-row approach keeps all info about one entity together, While Column-by-column approach keeps all attribute information together
    - Row approach best for OLTP, and Inserts, updates, point lookups
    - Column approach best for aggregations and scans over million rows
- Column oriented databases handle fixed length data. as they usually same type values which allows better compression and faster processing
- In Row approach since Each row is stored together on disk its good for OLTP and Transactional loads but bad for analytics. 
    - Query like: `SELECT SUM(salary) FROM salesperson;` The DB must read every row and Load DOB, Name, Sales, Expenses even though you only need `salary` -> Lots of unnecessary I/O
 - In column Approach Each column is stored separately and values of the same attribute are stored together so the DB Reads only salary column and ignores Name, DOB, Sales, Expenses -> Much less disk I/O and much better cache usage

## how data is stored on disk

- Row-oriented (traditional RDBMS)
    - Each row is stored together
    - `[id | name | salary | dept]`
     `[id | name | salary | dept]`
 - Column-oriented
    - Each column is stored together
    - `[id, id, id, id]
    - `[name, name, name]
    - `[salary, salary, salary]`
- This is physical storage, not logical view.
![Pasted image 20260105165605.png](./Imgs/Pasted%20image%2020260105165605.png)
## Query Execution

```SQL
SELECT COUNT(E.id) FROM Employee E;
```

- Row-Oriented 
    - DB reads entire rows even if we only need one column, it still reads all columns
    - Large I/O cost and poor cache efficiency
    - Good for transactional queries but Wasteful for analytics (Reading unnecessary data slows analytics.)
- Columnar Database
    -  DB reads only the `id` column and Other columns are completely ignored
    - Each column lives in its own storage blocks
    - Queries touch only required columns Enabling:
        - Faster Scans
        - Better Compression
        - Vectorized execution
    - Extremely efficient and minimal disk I/O (This is why column stores are perfect for aggregations)
![Pasted image 20260105170154.png](./Imgs/Pasted%20image%2020260105170154.png)

## Why Column Oriented Database?

- Most data warehousing applications make more number of reads and lesser number of writes.
- They mostly retrieve and analyze lesser number of columns compare to the several number of columns that actually exist.
- Row oriented databases have the overhead of seeking through all columns but Row oriented data warehouses still persistent.
- Two Main Advantages:
    1. Queries that seek to aggregate values of specific columns are optimized.
    2. Compression
- Disadvantages: 
    -  Columnar databases are poor choice to OLTP databases due to read and write overheads as retrieving a single row involves assembling the row from each of the column stores for that table.
        - Read overhead can be partly reduced by caching and multicolumn projections (storing multiple columns together on disk). However, DML operations—particularly inserts—there is virtually no way to avoid having to modify all the columns for each row.
![Pasted image 20260105193331.png](./Imgs/Pasted%20image%2020260105193331.png)
### Compression

**Compression** algorithms work by removing redundancy

- If data repeats, we don’t store it again and again, We store it once + some small info to reconstruct it. Example: AAAABBBCC → A4B3C2
- Highly repetitive data achieves higher compression ratios
    - If values repeat a lot → **excellent compression** 
    - If values are random → **poor compression**
- Compression works best on localized subsets of data because compression is applied block by block, If similar values are close together, compression is faster and cheaper. (CPU cost is lower when compressing small isolated blocks, not entire datasets)
- Why columnar databases achieve very high compression ratios?
    -  because each column stores similar and often sorted values, allowing techniques like delta encoding to efficiently remove redundancy. 
- **Delta Encoding**: Representing each value as a “delta” from the previous value
    - Instead of storing: 5000, 5000, 5000, 5100, 5100, 5200. 
    - Store: 5000, 0, 0, +100, 0, +100
    - Deltas are small numbers and small numbers compress extremely well
- In Columnar Databases:
    -  One column = same attribute
    - Same attribute = similar values
    - Similar values = high redundancy
    - High redundancy = high compression
    Row-based DBs mix unrelated values → bad compression.

## Rise of Columnar Databases

- In 1990, Sybase launched the first commercial column-oriented database (Sybase IQ).
- In 2005, Mike StoneBreaker and colleagues published paper “C-Store” that presents new innovation to write techniques.
- In 2011, Breaker established “Vertica” company to implement C-Store.

## Columnar Database Architecture

- Since Insert and update are key weaknesses. Solution is Bulk load jobs or classic overnight ETL.
- What about Real-time or near real-time data? Solution is Write-Optimized Delta Store.
    -  The **delta store**: Stores recent inserts & updates
         - Uncompressed
         - Often row-oriented
         - Memory-Resident
         - Designed for high-frequency writes
    - New data → **delta store**, Old stable data → **column store**

| **Store**                 | **Purpose**                                  |
| --------------------- | ---------------------------------------- |
| **Main Column Store** | Read-optimized, compressed, disk-based   |
| **Delta Store**       | Write-optimized, fast, usually in memory |

### Query execution

- When you run a query the DB does:
    1. Read from column store
    2. Read from delta store
    3. Merge results
    4. Return Final Answer

### Tuple Mover

- The **tuple mover** is a background process that Periodically:
    - Takes data from delta store
    - Converts it to column format
    - Compresses it
    - Moves it into main column store
- This Happens: Asynchronously Without blocking queries or inserts that is why it is called (Asynchronous Tuple Mover))
- **Traditional Writes**:
    - Every write goes to many partitions
    - Many random disk I/Os
    - Bad performance
- **Optimized Writes**: 
    - Writes go only to delta store
    - Simple, fast appends
    - Column store remains untouched
![Pasted image 20260105195124.png](./Imgs/Pasted%20image%2020260105195124.png)

# Document Database

**Document DBs** are non-relational DB store documents in the value part of the key-value.

- Documents usually self describing and hierarchal.
    - A document contains its own schema inside it.
    - Each document carries its field names and structure with it
    - Documents can contain nested structures (documents inside documents, arrays inside documents).
- Documents are indexed using a Btree and queried using a JavaScript query engine
- Usually formatted as XML, JSON, BSON.
![Pasted image 20260105201830.png](./Imgs/Pasted%20image%2020260105201830.png)

- Documents in the same collection do not have to share the same fields
- Document databases support **schema flexibility**, allowing different documents to have different structures.
- In **relational databases**:
    - Table schema is fixed
    - Every row must follow the same columns
    - If a value doesn’t exist → store NULL
    Causes many NULL Value -> Wasted Space and scheme changes are expensive

| **Aspect**         | **Document DB**             | **Relational DB**             |
| ------------------ | ----------------------- | ------------------------- |
| **Schema**         | Flexible                | Fixed                     |
| **Attributes**     | Can differ per document | Same columns for all rows |
| **Missing data**   | Field omitted           | NULL stored               |
| **Data evolution** | Easy                    | Expensive                 |

## JSON is the evolution of XML

The first document DB built around the **XML** (Extensible Markup Language) technology.

- XML is supported by variety of tools and standards:
    1. **XPath**: A syntax for retrieving elements from an XML doc.
    2. **XQuery**: A query language for interrogating XML docs.
    3. **XSLT**: A language for transforming XML documents into alternative formats.
    4. **DOM** (Document Object Model): An object-oriented API that programs can use to interact with XML, XHTML, and similarly structured documents.
- Also Supported in many DBMS such as Oracle and Microsoft SQL server.
- XML tags are repetitious typically increasing the amount of storage required by several factors. As a result, XML format is relatively expensive to parse.
- JSON is a lightweight form of XML that mostly support web-based operational workloads—storing and modifying the dynamic content and transactional data of web-based applications.
- JSON and XML are similar formats. However, XML document databases excel for content management systems, while JSON document databases generally aim to support operational web applications.

## JSON Databases Terminology

- A **document** is the basic unit of storage, corresponding approximately to a row in an RDBMS.
- A document comprises one or more key-value pairs, and may contain nested documents and arrays.
- A **collection** or **data bucket** is a set of documents sharing some common purpose; this is equivalent to a relational table.
- The documents in a collection don’t have to be of the same type, though it is typical for documents in a collection to represent a common category of information.

## Document Embedding

![Pasted image 20260105203953.png](./Imgs/Pasted%20image%2020260105203953.png)
![Pasted image 20260105204026.png](./Imgs/Pasted%20image%2020260105204026.png)

- There is a pattern in which “actors” are nested as an array within the “films” documents.
- Advantages:
    -  Allowing a film and all its actors to be retrieved in a single operation (avoids joins within the application code).
- Disadvantages:
    -  Results in “actors” duplication across multiple documents, leading to inconsistencies upon “actor” updating.
    - Problem occurs if the number of embedded documents (Actors) increase without limit (the size of a single JSON doc. is usually capped—64MB in MongoDB, for instance).

## Data Models in Document Databases

- **Document linking**: A database designer can link multiple documents using document identifiers, much in the way a relational database relates rows via foreign keys.
- Example: we embed below an array of actor IDs into the “films” document.
![Pasted image 20260105204413.png](./Imgs/Pasted%20image%2020260105204413.png)

- Another example of document linking. Although this is somewhat an unnatural design for a document database, for some workloads it may provide the best balance between performance and maintainability. (So linking here feels more like relational thinking)
![Pasted image 20260105205127.png](./Imgs/Pasted%20image%2020260105205127.png)

- Referencing is preferred when data is shared frequently and updated independently

## Commercial Tools

- MongoDB
- CouchDB
- RethinkDB
- RavenDB
- Cassandra
- DataStax Astra DB

## DataStax Database connection

- NoSQL Database requests and manipulations are performed through:
     1. CQL (Cassandra Query Language)
     2. APIs (REST, Document API Such as Swagger, and GraphQL).
     3. Drivers (Python, Java, C++, ..etc): using software packages.


### Document API: REST API

**REST** (Representational State Transfer) used to communicate between client and server.

- By using a REST interface, different clients hit the same REST endpoints, perform the same actions, and receive the same responses.
- REST is simple, standardized, and stateless.
- REST requires that a client make a request to the server in order to retrieve or modify data on the server.
- A request generally consists of (an HTTP verb, header, a path to a resource, an optional message body containing data).
- **HTTP Verbs**: There are 4 basic HTTP verbs we use in requests to interact with resources in a REST system:
    1. **GET** — retrieve a specific resource (by id) or a collection of resources
    2. **POST** — create a new resource
    3. **PUT** — update a specific resource (by id)
    4. **DELETE** — remove a specific resource by id