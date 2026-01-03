# Data Types

- Structured Data
- Semi-Structured Data: Documents like Json, ..etc
- Unstructured Data Like Videos, Images, Mp4, ..etc

# Database Types

## OLTP (Online Transaction Processing)

- Designed for data entry and manipulations (Day to day business operation)
- Normalized data
- Concerned with current data (real time data)
- Flexible schema
- Most common Type

## OLAP (Online Analytical Processing)

- AKA Data Warehouse
- Designed for data collection & analysis (Fast and  Easy access to complex queries)
- Concerned with historical/ statistical information
- Data is summarized
- Rigid or Fixed Schema
- Not usually concerned with individual transactions unlike OLTP

# Why SQL is not enough?

- Today's data can be inconsistent, SQL databases cannot deal with uncertainties, Once you define a DDL, you Have to input data in the same format and cannot keep changing it.
- NoSQL does not need a schema to be defined in the first place to store data. You can dynamically change things on the go to suit your analytical need.

# Data Lakes

- Designed to capture raw data (structured, semi-structured, and unstructured)
- Made for large amounts of data
- Uses ML and AI for analytics and processing
- It is not super-useable: To be useful it is cleansed and organized into databases or data warehouses.

- These advancements in hardware infrastructure (cheaper, larger RAM, and faster processors) directly enable the performance demands for rapid query responses, making IMDB technology vital for modern applications