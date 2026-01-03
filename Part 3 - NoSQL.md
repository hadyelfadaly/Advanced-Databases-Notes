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