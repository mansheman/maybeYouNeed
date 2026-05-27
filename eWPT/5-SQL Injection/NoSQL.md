2025-08-15 16:17

Status:

Tags: [[eWPT]] [[SQL Injection]]
###### Prerequisites: 

# Introduction to NoSQL

- NoSQL databases, also known as **"Not Only SQL"** databases, are a class of database management systems that provide a non-relational approach for storing and retrieving data.
    
- Unlike traditional relational databases, which organize data into tables with predefined schemas, NoSQL databases offer more flexible data models that can handle unstructured, semi-structured, and rapidly evolving data.
    
- NoSQL databases emerged as a response to the need for scalability, performance, and agility in handling modern data types and workloads.
    

---

## Types of NoSQL Databases

### Key-Value Stores

- Store data as a collection of key-value pairs.
    
- The value can be text, JSON, or binary objects.
    
- **Examples:** Redis, Riak, Amazon DynamoDB.
    

### Document Databases

- Store and retrieve data in JSON-like documents.
    
- Documents can vary in structure.
    
- Provide features for querying and indexing based on document content.
    
- **Examples:** MongoDB, Couchbase Server.
    

### Columnar Databases

- Organize data into columns rather than rows.
    
- Efficient for analytical workloads and handling large volumes of data.
    
- **Examples:** Apache Cassandra, Apache HBase.
    

---

## NoSQL vs SQL Databases

|Feature|SQL|NoSQL|
|---|---|---|
|Type|Relational|Non-Relational|
|Data Storage Model|Tables with fixed rows and columns|Unstructured; JSON files; key-value pairs; tables with dynamic columns|
|Schema|Static / Rigid|Dynamic / Flexible|
|Scalability|Vertical|Horizontal|
|Language|Structured Query Language (SQL)|Unstructured Query Languages|
|Schema Bound|Rigid, relationship-based|Flexible, non-rigid|
|Query Complexity|Supports complex queries|Limited complex query support|

---

## Popular NoSQL Databases

### MongoDB

- Document database storing data in flexible, JSON-like documents.
    
- Provides high scalability, automatic sharding, and a powerful query language.
    

### Cassandra

- Distributed columnar database for large-scale data across multiple servers.
    
- Offers high availability, fault tolerance, and linear scalability.
    

### Redis

- In-memory key-value store used as a database, cache, or message broker.
    
- Supports a wide range of data structures.
    
- Known for high performance and low latency.
    

---

## NoSQL Database Query Languages

- NoSQL databases often have their own query languages or interfaces.
    

**MongoDB:**

- Uses MongoDB Query Language (MQL).
    
- Offers rich operators and functions for querying and manipulating documents.
    

**Redis:**

- Operates primarily via commands on data structures (strings, lists, sets, hashes).
    
- No traditional query language; commands handle data reading, writing, and expiration.