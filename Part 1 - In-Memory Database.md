# What is In-Memory Database (IMDB)?
 
- An IMDB also called Main memory Database (MMDB) is a database whose primary data store is main memory, in contrast to databases that store data on disk or SSDs.

- IMDBs are designed to enable minimal response times by eliminating the need to access disks.
- IMDB risks losing data upon a process or server failure.
- IMDB can persist data on disks by storing each operation in a log or by taking snapshots.
- IMDBs are ideal for applications that require microsecond response times or have large spikes in traffic such as gaming leaderboards, session stores, and real-time analytics.

# Why now so important?

1.  **Lowering Costs and Growing Size of RAM:** The cost of Random Access Memory (RAM) has decreased significantly while its size capacity has increased. This makes large-scale adoption of memory-intensive systems economically feasible
2. **Multicore Processors:** The rise of multicore processors supports **parallel and faster computation**, optimizing the processing capabilities required for high-speed in-memory operations.
3. **64-bit Computing:** The shift to **64-bit computing** enables the use of multiple gigabytes (GB) of main memory, allowing IMDBs to handle larger datasets entirely within RAM.
4. **Faster Responses to Queries**  

- These advancements in hardware infrastructure (cheaper, larger RAM, and faster processors) directly enable the performance demands for rapid query responses, making IMDB technology vital for modern applications

# In-memory Databases Architecture

- **Primary Storage of Data Base:** This refers to the main memory, which holds the primary copy of the data.
- **Application SQL Engine:** This component processes SQL requests coming from applications
- **Query Optimizer:** This unit is responsible for optimizing queries. In an IMDB, query optimization factors include the cardinality of the table, the presence of an index, and the existence of any `ORDER BY` clause
- **Data Store:** The data is held here in memory addresses
- **Index and Data Manager:** This component manages the storage structures for data and indices, such as the lock and log mechanisms. In IMDBs, data access methods utilize structures explicitly designed for memory, such as the **T-tree index structure**
- **Logs/Redo/Ckpt (Checkpoints) and Secondary Storage (Recovery Purpose):** Although the data resides primarily in main memory, IMDBs must incorporate mechanisms to manage the risk of losing data upon a process or server failure. They persist data on secondary storage (like disks) for recovery purposes by storing each operation in a **log** or by taking **snapshots** (checkpoints). These logging and checkpoint mechanisms are essential for achieving transactional **durability**, as IMDBs can otherwise be said to lack support for this portion of the ACID properties

# IMDB Practical Applications (use cases)

1. Real-time bidding: refers to the buying and selling of online ad impressions.  
     - The bid has to be made while the user is loading a webpage, in 100-120 milli-sec and sometimes as little as 50 milli-sec.
     - During this time period, real-time bidding applications request bids from all buyers for the ad spot, select a winning bid based on multiple criteria, display the bid, and collect post ad-display information.
     - In-memory databases are ideal choices for processing, and analyzing real-time data with sub-millisecond latency.
2. Gaming leaderboards: A gaming leaderboard shows a gamer's position relative to other players of a similar rank.
     - It helps to build engagement among players and meanwhile keep gamers from becoming demotivated when compared only to top players.
     - For a game with millions of players, in-memory databases can deliver sorting results quickly and keep the leaderboard updated in real time.

# DRDB VS IMDB

- DRDB is Disk Resident Data Base

| DRDB                               | IMDB                            |
| ---------------------------------- | ------------------------------- |
| Carries File I/O burden            | No file I/O burden              |
| Extra memory For Cache             | No extra memory                 |
| Algorithm optimized for disk       | Algorithms optimized for memory |
| More CPU cycles                    | Less CPU cycles                 |
| Assumes Memory is abundant (وفيرة) | Uses memory more efficiently    |
# Myths about IMDB’s

- Given the same amount of RAM, disk DBs can perform at the same speed as IMDBs (by using caching technology). Because:
     - IMDBs are engineered specifically to enable **minimal response times by eliminating the need to access disks**. This architectural advantage means that even optimal caching within a DRDB cannot fully replicate the inherent speed and efficiency of a true IMDB
- If a RAM disk is created and a traditional disk DB is deployed on it, it delivers the same performance as an in-memory database because:
    - even with the physical latency benefits of a RAM disk, the software overhead and the disk-centric logic of the traditional database prevent it from achieving the speed and efficiency of a purpose-built IMDB
- In-memory database the same as an embedded database.
- Since RAM size is limited, sizes of IMDBs are also limited. Is a fact.]

# Impact of IMDB’s
 
 1.  Data Representation: In Disk Resident DB, we use flat files and sequential access While In IMDB, Relational tuples with direct pointers: Space Efficient
 2. Concurrency Control (lock based)
     - Locking technique: A lock enables transactions to acquire read or write access to data, ensuring conflict prevention and isolation enforcement.
     - In DRDB, locking granules are low level -> To reduce contention
     - In IMDB, due to fast processing it create coarser locks -> locking granules like a relation or entire database.
3. Data Access Methods
     - In DRDB, B-tree index structure is used. (Ranges , exact match queries, Lies in hard disk)
     - In IMDB, T-tree index structure is explicitly designed (Reduces the CPU processing, Eliminates index value compression and expansion)
4. Query Processing
     - In DRDB, main focus is on processing costs, and attempt to minimize disk access.
     - In IMDB, main factors are: Cardinality of table, Presence of index, Any ORDER BY clause, Predicate (condition) evaluation.
5. ACID Properties: IMDBs can be said to lack support for the durability portion of the ACID
     - Many MMDBs have added durability via the following mechanisms: Checkpoints, Transaction logging and NVRAM(Non-Volatile RAM).
 6. Recovery: transactional durability is kept, by keeping two separate but synchronized copies of the database at all times as well as storing log files on-disk.

# IMDB Challenges

1. Durability
2. Query optimization
3. Size of Database

# Limitations of IMDB

1. Limited scalability: Due to the limited amount of available memory, they may not be able to handle large amounts of data.
2. Limited durability: It does not provide the same level of durability as traditional disk-based databases (if system crashes or there is a power failure, data may be lost).
3. Higher cost: in terms of both hardware costs and licensing fees.
4. Limited compatibility: Some IMDB may not be compatible with certain types of data or certain data structures, which limits the usefulness in certain situations.