
# What is Data Warehouse?

- Is a database, or collection of databases, that centralizes a business's information from multiple sources and applications, and makes it available for analytics and use across the organization.
- Is the place (typically a cloud storage) where a company’s historical data is stored in a structured way, usually in the form of relational databases.
- They can’t be changed, nor deleted.
- We can only retrieve information through aggregation or segmentation and use it for analytical, referential, or reporting purposes.

# Dimensional modeling

- Business Process (event) is the Unit of Design: everything in dimensional modeling starts with **an event that happens in the business**.
- This event becomes the **core of the fact table**.
- A **business process** generates **Measurements** (facts) and **Context** (dimensions)
- Dimensions -> Who, What, Where, When, How, and why (They describe the event)

## Facts

- Container for measurements which used to get KPIs in the organization
- One record = one event  
- Example: each record in Shop Sales = one purchase  
- All rows have the same **grain** 
     - Grain is the lowest level of details
     - choosing grain = choosing how detailed your fact table is.
- Types of measures (additive, semi-additive, non-additive)
- Facts = Event(s) = outcome of a business process
- Record in a fact table must have only one level of grain

## Granularity/Grain

- The lowest level of details for a particular business purpose or for an attribute is called the grain or granularity.
- For example, if we consider the date/time dimension, it could start at the year (topmost), month, quarter, period, week, day, hour, minute, second (lowest) level of granularity.

## Dimensions

- Dimensions are organized into hierarchies. Ex: Time dimension: days → weeks → quarters
- Dimensions have attributes
- Dimensions Show us a good description of what is in the Fact table.
- Dimensions must be wide and descriptive — fully denormalized — because they are the entry points for queries.
- Why denormalize:
     - Faster queries
     - Fewer joins
     - Simpler for analysts
- The Date Dimension is the **most important and most reused dimension** in almost every data warehouse. It is a **conformed dimension** because every fact table shares it.
- Use a **surrogate key** for dimensions

# The Multi-Dimensional Model

- Multi-dimensional Modeling is a technique for structuring data around the business concepts
- ER models describe “ENTITIES” and “RELATIONSHIPS”, Entities grouped into dimensions or attribute in a dimension, Relationships are transformed into Fact tables.
- So, finally Multi-dimensional Models describe “measures” and “dimensions”

![Image](Imgs/Pasted%20image%2020251114201013.png)

# Datawarehouse Keys

## System Keys

- We cannot depend on natural keys because it can be poorly administrated and the same key can be used to represent different entities in different source systems. Ex: A key of churned customer can be given to a new customer.
- We don't have control over it can be changed without notice
- Can be in a poor format
## Surrogate Keys

- Separate Key from the source system natural key
- Sequence start from 1
- Each record inserted into Dimension will have new value
- Under control of DWH (Controlled by ETL Process)
- Integer surrogate keys will enhance performance and help to save space in fact table. (Integer better in indexing)
- Fact tables **do NOT have their own surrogate key**. They only **store the surrogate keys of dimensions**.

## Durable Keys

- A stable, DW-managed business identity key
- Never changes 
- Used to unify all SCD versions of the same entity
- Permanently assigned to a real-life identity
- When source keys change → DW still knows it’s the same person.

# Dimension Hierarchies

- A **hierarchy** in a dimension is simply an ordered set of attributes that roll up into each other.

1. Balanced Hierarchy: A hierarchy where **every branch has the same number of levels**.
     - Type → Subtype, Day → Month → Year, Country → State → City
     - Every path has the same number of levels
     - Easy to index and slice in dimensional models
     - Perfect for OLAP roll-ups
     - There are no “missing” levels
2. Unbalanced (Ragged) Hierarchy: A hierarchy where **different branches have different depths** (levels missing).
     - Some products: Brand → Category → Subcategory → Product Others: Category → Product
     - Need special handling (bridge tables or flattening)

- Balanced hierarchies are the default and easiest to model.
- Ragged hierarchies are often modeled using **bridge tables** to handle many-to-many or variable-depth relations.

# Schema Design

- Most data warehouses use a star schema to represent multi-dimensional model.
- Each dimension is represented by a dimension table that describes it.
- A FACT TABLE connects to all dimension tables with a multiple join.
     - Each tuple in the fact table consists of a pointer to each of the dimension tables that provide its multidimensional coordinates and stores measures for those coordinates.
     - The links between the fact table in the center and the dimension tables in the extremities form a shape like a **STAR**.

## Schema Design Steps

1. Define dimensions (or the way that analyzes information)
2. Identify the dimensions’ attributes
3. Organize attributes hierarchically within a dimension
4. Assign unique identifiers to dimensions
5. Assign description fields in the attributes
6. Identify and create fact tables
7. Assign unique identifiers to Fact tables

## The “Classic” Star Schema (Ralph Kimball modeling)

- A relational model with a one-to-many relationship between dimension table and fact table.
- A single fact table, with detail and summary data
- Fact table primary key has only one key column per dimension
- Each dimension is a single table, highly denormalized
- Star schemas are “business-friendly.”
- Benefits:
     - Easy to understand
     - intuitive mapping between the business entities (the tables directly match how business users think)
     - easy to define hierarchies
     - reduces number of physical joins
     - low maintenance
     - very simple metadata
- Drawbacks:
     - Summary data in the fact table yields poorer performance for summary levels (Summaries require aggregation over huge detail fact tables → slower performance unless using summary tables)
     - huge dimension tables a problem
- To solve the Summary problem:
     - Aggregating Fact Tables: Aggregate fact tables are summaries of the most granular data at higher levels along the dimension hierarchies.

### The “Fact Constellation” Schema

- A **Fact Constellation** (also called **Galaxy Schema**) is a data warehouse schema that contains: Multiple fact tables and Multiple shared dimension tables. This creates a “constellation” (group) of related stars.
- Described as Families of Stars
- When to use: 
     - When the business has **many business processes**. Ex: Ordering, shipping, payments, ...etc. Each becomes a **separate fact table** But have shared conformed dimensions.
 - Why is a Fact Constellation useful?
     - Scalability — supports many business processes
     - Conformed dimensions allow cross-fact reporting
     - Reduces redundancy (one Product dimension used everywhere)

## Snowflake Schema

- Snowflake schema is a type of star schema but a more complex model.
- "Snowflaking” is a method of normalizing the dimension tables in a star schema.
- The normalization eliminates redundancy.
- The result is more complex queries and reduced query performance.
- The attributes with low cardinality in each original dimension table are removed to form separate tables (lookups)
- These new tables are linked back to the original dimension table through artificial keys.
- Snowflaking helps tools understand the relationships more clearly.
- Snowflaking makes the data structure cleaner and more machine-readable for advanced analytical tools, even if it's not ideal for human querying.
![Image](Imgs/Pasted%20image%2020251114213515.png)
- Advantages:
     - Small saving in storage space
     - Normalized structures are easier to update and maintain
- Disadvantages:
     - Schema gets more complex (complicates data loading)
     - Ability to browse through the contents difficult
     - Degrade query performance because of additional joins
- Why to use snowflake schema?
     - To satisfy data gathering functionality for more advanced DW / mining tools. Because:
         - Data mining algorithms often expect **entities to be clearly separated** (e.g., Customer, Geography, Product Category).
         - Tools that automatically detect dependency patterns work better when the structure is normalized.
         - Some advanced tools cannot handle “wide, denormalized tables” with hundreds of attributes mixed in one large dimension.
     - To logically separate large dimensions to make them human readable.
     - To more naturally separate dimensional data. (Know customers vs anonymous customers)
- Do not use it unless you have to or need to use it. It is not generally recommended in a data warehouse environment.

## What is the Best Design?

- Performance benchmarking can be used to determine what is the best design.
- Snowflake schema: easier to maintain dimension tables when dimension tables are very large (reduce overall space).
- Star schema: more effective for data cube browsing (less joins): can affect performance.
- Benchmarks are all about making choices:
    - What kind of data will I use? How much? What kind of queries?
    - How you make these choices matters a lot: Change the shape of your data or the structure of your queries and the fastest warehouse can become the slowest.



# Aggregates

- ROLLUP (moving UP the hierarchy) -> Less Detail Ex: When you go from:
     - p1 on day 1 = 62” and “p2 on day 1 = 19” to 
     - “Day 1 total = 81”
- DRILL-DOWN (going deeper into detail) -> More Detail Ex: When you go from:
     - “Day 1 total = 81”  to 
     - “p1 on day 1 = 62” and “p2 on day 1 = 19”
# Data Cubes

- A **Data Cube** is a multi-dimensional data structure used in OLAP systems to help users analyze data quickly. (A data cube is a multi-dimensional view built from a fact table joined to its dimensions, pre-aggregated to support fast OLAP queries along multiple dimensions.)
- Cubes are logical representations of dimensional models for fast analytics
- They are mainly used for fast retrieval of aggregated data.
- The key elements of a data cube are dimensions, attributes, facts and measures.
- Examples of dimensions could be Products, Store Location, or Date.
- A fact is the smallest part of a data cube, and it comes with a certain measure. Using the dimensions listed above, a fact could be the number of jeans sold in a given store on a certain date.
- Why Data Cubes Matter:
     - Fast summarization
     - Multi-dimensional analysis
     - Efficient slicing/dicing
     - Business intelligence dashboards
     - Trend analysis

![Image](Imgs/Pasted%20image%2020251114220854.png)
## Data cube aggregation

- Data cube aggregation summarizes records belonging to the same attribute, be it finding their sum, minimum value, maximum value, standard deviation, count, etc.
- Aggregating values in a data cube reduces its dimensionality. Ex:
    - Imagine the revenue of a company is split by months. One can therefore aggregate the data by year, summing the revenue from all 12 months for each year.
- One way to think about aggregation is like zooming out on a data cube.
![Image](Imgs/Pasted%20image%2020251114221223.png)

# Types of Facts/Measures

- Additive measures can be aggregated across any (or combination) of the dimensions associated with the fact table.
- The most flexible and useful facts are fully additive.
## Additive

- Summarize across all dimensions
- Can be summed and aggregated over all dimensions
- For example, sales revenue. It can be summed up across all dimensions such as time, product, location, and customer.

## Semi-Additive

- In Semi-additive measures is a limited additive measure. It can be summed or aggregated (Min, Max, Avg) over a set of dimensions but not all.
- For example, inventory levels can be summed up across time and location dimensions, but not across product or customer dimensions. (Some products are in other are out)

## Non-Additive

- Can not be summed and aggregated (Min, Max, Avg) over all dimensions.
- Like we cant some a percentage or a ratio but ofc there are other ways to do analytics
- For example, average temperature or percentage of market share are nonadditive measures. 

## Derived Measures

- These are measures that are calculated based on the existing measures in a data cube.
- For example, profit margin can be calculated by dividing the sales revenue by the cost of goods sold.

# Types of Fact Tables

- A Fact table is a collection of measurement
- A DW will have many fact tables. Each table store measure about specific business area.

## Transactional Fact Table

- Popular Fact Table
- Simple Structure.
- Keeps most detailed level (Lowest Grain of data)
- Easy to aggregate.
- Mostly ADDITIVE Facts.
- One record in a fact is one event or set of events happened

## Fact-less Fact Tables

- It is a fact table with no measures.
- For example associate entity between two tables with binary Many to Many relationship.
- Ex: A fact table which has only product key and date key is a Fact-less fact. There are no measures in this table. But still you can get the number products sold over a period of time.
- Records mean a certain event happened (There is no measures)
- Grain -> One record represent one event

## Periodic Snapshot Fact Table

- Periodic snapshot tables record the cumulative performance of the business at predefined periods of time.
- A predetermined interval for taking snapshots is the key: daily, weekly, monthly, etc.
- The **grain** is the **time period** (not a single transaction).
- If the periodic snapshot only cares about a certain grain (e.g., monthly), then all other grains (daily, hourly) are NULL.  Or you can create separate tables for each grain if needed. Explained below.
- There are **two choices** when dealing with periodic snapshot tables:
     - Use the same fact table for all time grains but for the “non-interesting grains” put **NULL**. Example:
        - If month is the grain: Month column: value, Quarter column: NULL and Year column: NULL
    - Use separate fact tables Based on: volume, reporting needs and performance
         - Daily Snapshot Fact
         - Monthly Snapshot Fact
         - Quarterly Snapshot Fact
- One record represents summary of measurements over a period of time.
- Grain -> period of time is the grain.
- Just as transaction fact tables have many dimensions, the snapshot usually has fewer dimensions which leads it not to have the lower grain of data. (**Periodic snapshot fact table** → fewer dimensions → summary only (high grain))
- Two use cases:
     - When we need to avoid many calculations,
     - When need to follow up the STATUSES of very essential object. Ex: Bed Statuses in hospital every half an hour, every two hours, ...
- Stores current state of an essential object in the DW at a regular interval.
- Good to work with not Fully Additive Measures.

## Accumulating Snapshot Fact Table

- It is suitable for accumulating any business process that requires following up the process (step by step).

## Summary of Fact Table Types

|                | Transaction                                                                                    | Periodic Snapshot                                                     | Accumulating Snapshot                                                                       |
| -------------- | ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Definition     | Stores lowest grain of data                                                                    | Stores the current state of data at a Regular interval of time.       | Stores intermediate steps that has happened over a period of time.                          |
| Grain          | Captures one row per transaction.                                                              | Captures one row per time period.                                     | Captures one row for entire lifetime of event.                                              |
| Date Dimension | Date dimension at the lowest grain (lowest level)                                              | Date/Time is regular interval of snapshot frequency.                  | Multiple date time dimensions for intermediate steps in business processes.                 |
| Aggregation    | Very Easy                                                                                      | Either No or Very Minimal Aggregation is Needed. (Already aggregated) | No Need. But if needed it will be very complicated process.                                 |
| DB Size        | Very Large                                                                                     | Smaller than transactional Fact table                                 | Smallest                                                                                    |
| Update         | No updates                                                                                     | No Updates                                                            | Update happens after each stage or milestone. Updates happen regularly and very frequently. |
| Best Use Cases | Most of the business requirements are achieved using Transaction FT. Used for aggregated data. | Whenever business performance is reviewed at regular interval.        | When business process has multiple stages or phases.                                        |

# Types of Dimensions

## Slowly changing dimensions (SCD)

- Used to track changes
- Usage driven by the business

### Type 0: No History

- Dimension attributes never changes, for example Data Dimension attributes

### Type 1: New Override Old Value (Overwrite)

- New attribute values override the old values\

### Type 2: Keep All History

- Add new dimension record for every change in dimension, every DWH batch process we track what had changed for this dimension and we insert new record with the new attributes.
- In this type we either use flag to know the last record or use time interval (date from - to) or using both.
- we use very far date in the future 01-01-9999 to indicate that this is the last record.

### Type 3: Keep One Value of History

- Keep the previous value for required columns only in a separate column for each attribute

### Type 6: Combine Type 1, 2, 3

- Use this type if you want access to full history of changes and also a quick access to current and previous values
- Steps:
     1. Update Value
     2. Insert new record for every change, track with validity dates and flags
     3. Add previous historical value column

![Image](Imgs/Pasted%20image%2020251114234753.png)

## Conformed dimension (Distorted)

- A **Conformed Dimension** is a dimension that is **shared across multiple fact tables** and means the **same thing** in all of them.
- consistent
- standardized
- Identical across the whole organization
- Example: Date dimension
- Called distorted because the meaning of the dimension in each fact table changes based on the business process. Example: 
    - In **Sales**, PRODUCT_ID refers to the **product that was sold**
    - In **Complaints**, PRODUCT_ID refers to the **product being complained about**
## Junk Dimension

- A **junk dimension** collects a group of small, low-cardinality, miscellaneous attributes that **don’t belong to any main dimension**. Examples: TRUE/FALSE flags, Yes/No indicators, Status codes (10, 20, 30, 40…), etc.
- WHY do we use a Junk Dimension?
     - These little flags would be stored directly in the **fact table**
     - Fact table becomes **wider** (more columns)
     - Fact table becomes **sparser** (lots of NULLs)
     - Fact table becomes **slower**
     - Harder to maintain
     - Harder for OLAP tools to read

## Degenerate Dimension

- Sometimes a dimension is defined that has no content except for its PRIMARY KEY.
- A dimension which has ONLY primary key attribute and this attribute can be stored in a fact table and has no associated dimension table is called DEGENERATE DIMENSION.
- Degenerate dimensions are most common with Transaction and Accumulating Snapshot Fact Tables.
- A fact table can contain more than one degenerate dimension.
- Useful For:
     - Grouping line items
     - Getting average sales
     - Tracking the history of order

## Role Playing Dimension

- Role Playing Dimension is the dimension which may has MULTIPLE VALID relationships with the Fact Table.
- As an example, the date dimension can be used to fetch Transaction, Invoice and Shipment Date.
- In this case we do not have to create 3 Date Dimensions.
- All the THREE Date References refer to the same Date Dimension.
- The Date Dimension is stored only once, processed once and accessed THREE times. This saves time and memory space.

## Static Dimension

- Static Dimensions are those dimensions which are not extracted from the original data source but are created within the context of the data warehouse.
- Static Dimensions can be:
     - Loaded Manually - for example with status codes.
     - Generated By a Procedure, such as a date or time dimension
 - As an example, the Date, Geography or Location Dimensions.

## Shrunken Dimension

- A Shrunken Dimension is a subset of another dimension.
- For example, the orders fact table may include a foreign key for product, but the target fact table may include a foreign key only for Product Category, which is in the product table, but much less granular. 
- Creating a smaller dimension table, with Product Category as its primary key, is one way of dealing with this situation of heterogeneous grain.
- If the product dimension is snow flaked, there is probably already a separate table for Product Category, which can serve as the shrunken dimension

## Stacked Dimension

- A **Stacked Dimension** is a single dimension that combines **multiple small dimensions**   because their attributes **belong to the FACT**, not to a real business entity
- It is used when you have several **type** or **status** fields like: Product Type, Customer Status, Store Type, etc.
- Why do we need it?
     - You’ll end up with **many useless tiny dimensions** (ProductTypeDim, CustomerStatusDim, PaymentTypeDim, StoreTypeDim…)
     - Fact table will have **many foreign keys** pointing to tiny lookup tables
     - Querying becomes messy
     - BI tools hate tons of tiny dimensions
- Benefits:
     - one dimension instead of many
     - simplifies joins
     - keeps fact table cleaner
     - these attributes belong to the **fact**, not the core business entities

## Deferred (Inferred) Dimension

- When loading a fact record, a dimension record may not be ready yet. What to do?
     -  One solution Is to generate a surrogate key with null for all the other attributes.
     - This should technically be called an inferred member but is often called an inferred dimension.
- A fact table that references dimension members that are not yet loaded contains inferred members.
- Data for inferred members can be updated rather than created when it is loaded, so you do not need to create a new record.
- A late-arriving dimension (sometimes called early-arriving facts) occurs when the dimension data arrives in the data warehouse later than the fact data that references it.
- There is 4 Solutions:
     1. Never Process the Fact (Ignore records NULL value for the FKs).
     2. Park the record and retry.
     3. Insert a dummy value for the missing assignment. (90% of the DW folks apply this choice).
     4. Insert a record in the dimension and update (fill in the missing values) later.

## Rapidly changing Dimension

- A dimension is a fast changing or rapidly changing dimension if one or more of its attributes in the table changes very fast and in many rows.
- Handling rapidly changing dimension in data warehouse is very difficult because of many performance implications.
- slowly changing dimension type 2 is used to preserve the history for the changes. But the problem with type 2 is, with each change in the dimension attribute, it adds new row to the table. Due to changing a lot, table becomes larger and may cause serious performance issues. Hence, use of the SCD type 2 may not be a wise decision.
- Consider the fact table, in which not all the attributes of the table changes rapidly. There may be some attribute that may be changing rapidly and other not. The idea here is to separate the rapidly changing attribute from the slowly changing ones and move those attribute to another table called junk dimension and maintain the slowly changing attribute in same table. In this way, we can handle situation of increasing table size.
- Example: Patient table with Patient junk dimension
     - We cannot simply refer the junk dimension table by adding its primary key to patient table as foreign key. Because any changes made to junk dimension will have to reflect in the patient table, this obviously increases the data in patient dimension.
     - Instead, we create one more table called mini dimension that acts as a bridge between Patient and Junk dimension. We can also add the columns such as start and end date to track the change history.

# Components of a DW

## Data Sources (Operational Source Systems)

- These are the operational systems of record that capture the transactions of the business.
- The main priorities of the sources systems are performance and availability.
- The sources maintain little historical data. But IF you have a good data warehouse; then no need for the data sources to hold history or at least to hold long time interval of historical data.

## Data staging Area (DSA) / Operational Data Store (ODS)

- It is both a storage area and an operational area for a set of processes commonly referred as ETL/ ETCL. Extract -> Transform -> Cleanse -> Load
- The DSA/ODS is everything between the Data Marts/DW and the data presentation area / layer.

### Extraction

- Extraction is the first step in the process of getting data into the data warehouse environment.
- Extraction means reading and understanding the source data and copying the data needed for the data warehouse into the staging area for further data manipulation.
- Used methods for extraction:
    -  Get the whole table every time.
    - Get the table incrementally (ONLY new additions / Changes).
    - Using a fixed time periods.
- There are different Considerations on which method to use
    -  The less time it requires to do the data extraction the better it is. (DW will be updated more frequently)
    - The second consideration is practically. i.e., What are possible and what are not.
        - We need to extract data incrementally, but this may not be possible then we should search for and use another method.  
        - We need to extract the whole table but if the table has 100 millions records, then we may have to do it incrementally (if possible)