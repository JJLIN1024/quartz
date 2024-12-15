# Chapter 2: Data Models and Query Languages

<aside>
💡 If a workman wishes to do a good job, he must first sharpen his tools.

</aside>

https://youtu.be/W2Z7fbCLSTw?si=JoPebsQY-YRH9Uff
## SQL(Relational Model)

<aside>
📖 The relational model organizes data into tables with rows and columns, where relationships between data elements are established through keys.

</aside>

Popular SQL database engines include MySQL, PostgreSQL, SQLite, and MariaDB.

Here’s a simple example:

```markdown
Table: Customers
+------------+----------+
| CustomerID | Name     |
+------------+----------+
| 1          | John Doe |
| 2          | Jane Doe |
+------------+----------+
  ^
  |
  | Foreign Key
  |
Table: Orders
+--------+------------+-------------+
| OrderID | CustomerID | OrderAmount |
+--------+------------+-------------+
| 101     | 1          | 500.00      |
| 102     | 2          | 750.00      |
| 103     | 1          | 250.00      |
+--------+------------+-------------+
```

> CustomerID & OrderID are primary keys, while CustomerID serves as a foreign key in Orders Table
> 

Here’s the MySQL code to generate the  Customers and Orders Tables:

```sql
-- Create the Customers table
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    Name VARCHAR(100) NOT NULL
);

-- Create the Orders table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderAmount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- Insert sample data into Customers table
INSERT INTO Customers (CustomerID, Name) VALUES
(1, 'John Doe'),
(2, 'Jane Doe');

-- Insert sample data into Orders table
INSERT INTO Orders (OrderID, CustomerID, OrderAmount) VALUES
(101, 1, 500.00),
(102, 2, 750.00),
(103, 1, 250.00);
```

You can perform CRUD Operations on these tables

- CREATE: Add new data
- READ: Retrieve data
- UPDATE: Modify existing data
- DELETE: Remove data

You can also Join two different tables, to join tables, they must share some common key:

```markdown
Table: Employees
+----+------+----------+-------------+
| ID | Name | Position | DepartmentID |
+----+------+----------+-------------+
| 1  | Alice| Manager  | 101         |
| 2  | Bob  | Engineer | 102         |
| 3  | Carol| Designer | 101         |
+----+------+----------+-------------+
                |
                | JOIN ON
                | Employees.DepartmentID = Departments.ID
                v
Table: Departments
+-----+----------+
| ID  | Name     |
+-----+----------+
| 101 | Marketing|
| 102 | R&D      |
| 103 | Sales    |
+-----+----------+

Result of INNER JOIN:
+----+------+----------+-------------+----------+
| ID | Name | Position | DepartmentID| Dept_Name|
+----+------+----------+-------------+----------+
| 1  | Alice| Manager  | 101         | Marketing|
| 2  | Bob  | Engineer | 102         | R&D      |
| 3  | Carol| Designer | 101         | Marketing|
+----+------+----------+-------------+----------+
```

## NoSQL(Document Model)

<aside>
📖 Unlike SQL, where data stored are structured(with a schema), NoSQL stores unstructured or semi-structured data, and is schema-less.

</aside>

Popular NoSQL database engines include MongoDB, Apache Cassandra, Redis, Amazon DynamoDB, and Neo4j.

There are many types of NoSQL databases, first, look at Document Stores.

### Document Stores

Common Document Stores database engines include MongoDB and CouchDB.

Document stores save data in flexible, JSON-like documents. Each document can have a different structure.

```json
Collection: Users
+------------------------------------------+
| Document 1                               |
| {                                        |
|   "_id": 1,                              |
|   "name": "Alice",                       |
|   "age": 30,                             |
|   "skills": ["Python", "JavaScript"]     |
| }                                        |
+------------------------------------------+
| Document 2                               |
| {                                        |
|   "_id": 2,                              |
|   "name": "Bob",                         |
|   "company": "Acme Inc",                 |
|   "address": {                           |
|     "city": "New York",                  |
|     "zip": "10001"                       |
|   }                                      |
| }                                        |
+------------------------------------------+
```

A collection is a grouping of documents. Documents within a collection can have different fields. A collection is the equivalent of a table in a relational database system. A collection exists within a single database.

```markdown
MongoDB                                 SQL
+----------------------------------+    +---------------------------+
| Database                         |    | Database                  |
|                                  |    |                           |
| +------------------------------+ |    | +-------------------------+
| | Collection (Users)           | |    | | Table (Users)           |
| |                              | |    | |                         |
| | +----------------------------+ |    | | +-----+------+----------+
| | | Document 1                 | |    | | | ID  | Name | Age      |
| | | {                          | |    | | +-----+------+----------+
| | |   "name": "Alice",         | |    | | | 1   | Alice| 30       |
| | |   "age": 30,               | |    | | +-----+------+----------+
| | |   "email": "a@example.com" | |    | | | 2   | Bob  | 25       |
| | | }                          | |    | | +-----+------+----------+
| | +----------------------------+ |    | |                         |
| |                              | |    | |                         |
| | +----------------------------+ |    | |                         |
| | | Document 2                 | |    | |                         |
| | | {                          | |    | |                         |
| | |   "name": "Bob",           | |    | |                         |
| | |   "age": 25,               | |    | |                         |
| | |   "skills": ["Java", "C++"]| |    | |                         |
| | | }                          | |    | |                         |
| | +----------------------------+ |    | |                         |
| |                              | |    | |                         |
| +------------------------------+ |    | +-------------------------+
+----------------------------------+    +---------------------------+
```

One can perform CRUD operations (ex: [**`db.collection.find()`](https://www.mongodb.com/docs/manual/reference/method/db.collection.find/), [`db.collection.find()`](https://www.mongodb.com/docs/manual/reference/method/db.collection.find/))** in a Document Store DB, but native *join* operation is not supported, that is to say, the application code has to emulate the join operation with multiple queries.

### Key-Value Stores

The most popular Key-Value Store is Redis.

Key-value stores are simple databases that store data as a collection of key-value pairs.

This type of DB serves as a cache in RAM to other persistence DB, while the data capacity is limited, the speed is ultra-fast.

```markdown
+----------+-----------------------------+
| Key      | Value                       |
+----------+-----------------------------+
| user:1   | {"name": "Alice", "age": 30}|
+----------+-----------------------------+
| user:2   | {"name": "Bob", "age": 25}  |
+----------+-----------------------------+
| session:1| "logged_in"                 |
+----------+-----------------------------+
```

### Wide-Column Stores

Popular options include Cassandra, HBase.

Wide-Column Stores is just like adding another layer on top of Key-Value Stores, essentially it is a 2-dimensional key-value store, which makes it more efficient to query data compared to the full table scan required in SQL.

```markdown
Wide-Column Store
+-----------------------------------+
|            Column Keys            |
|   +---------------------------+   |
|   |    Key-Value Store        |   |
| R |  +-----+---------------+  |   |
| o |  | Key | Value         |  |   |
| w |  +-----+---------------+  |   |
|   |  | ... | ...           |  |   |
| K |  +-----+---------------+  |   |
| e |                           |   |
| y |                           |   |
| s |                           |   |
+-----------------------------------+
```

```markdown

Keyspace: SocialNetwork
+------------------------------------------------------------------+
| Table: Users                                                     |
|                                                                  |
| +---------+-------------------+------------------+---------------+
| | Row Key | Column Family: Profile               | CF: Activity  |
| |         +----------+--------+----------+-------+---------------+
| |         | name     | email  | location | age   | last_login    |
+-+---------+----------+--------+----------+-------+---------------+
| | user:1  | "Alice"  | "a@..." | "NY"    | 30    | "2023-08-10"  |
+-+---------+----------+--------+----------+-------+---------------+
| | user:2  | "Bob"    | "b@..." | "SF"    | 28    | "2023-08-11"  |
+-+---------+----------+--------+----------+-------+---------------+
|                                                                  |
| Table: Posts                                                     |
|                                                                  |
| +---------+-------------------+------------------+---------------+
| | Row Key | Column Family: Content              | CF: Metadata   |
| |         +----------+----------------------+---+---------------+
| |         | title    | body                 | timestamp | likes |
+-+---------+----------+----------------------+-----------+-------+
| | post:1  | "Hello..." | "First post..."    | 1628640000 | 5     |
+-+---------+----------+----------------------+-----------+-------+
| | post:2  | "Cassandra"| "Cassandra is..." | 1628726400 | 10    |
+-+---------+----------+----------------------+-----------+-------+
+------------------------------------------------------------------+
```

> Here’s an example of data stored in Cassandra.
> 

Cassandra provides Cassandra Query Language (CQL) for user to query data, it looks like SQL, but does not support Join.

### Graph Databases

Popular options include Neo4j, Amazon Neptune.

A graph database is a systematic collection of data that emphasizes the relationships between the different data entities.

When to use this kind of DB? When many-to-many relationships are very common in your data.

## SQL v.s. NoSQL

| **Aspect** | **SQL Databases** | **NoSQL Databases** |
| --- | --- | --- |
| **Data Model** | Relational (tables, rows, columns) | Non-relational (key-value, document, column-family, graph) |
| **Schema** | Fixed schema; requires predefined structure | Flexible schema; schema-less or dynamic structure |
| **Query Language** | SQL (Structured Query Language) | Varies by type (e.g., MongoDB uses BSON, Cassandra uses CQL) |
| **Data Integrity** | Strong ACID compliance (Atomicity, Consistency, Isolation, Durability) | Eventual consistency or BASE (Basically Available, Soft state, Eventually consistent) |
| **Scalability** | Vertical scaling (scale-up by increasing hardware capacity) | Horizontal scaling (scale-out by adding more servers) |
| **Flexibility** | Less flexible; changes to schema can be complex and disruptive | Highly flexible; easy to add new types of data or change schema |
| **Data Relationships** | Well-suited for complex relationships using joins | Less efficient for complex relationships; better suited for hierarchical or flat data |
| **Examples** | MySQL, PostgreSQL, Oracle, SQL Server | MongoDB, Cassandra, Redis, Neo4j (graph database) |

> The reason that it is more difficult to scale a SQL DB is because they typically promise strong ACID.
> 
- SQL(declarative query) has an internal optimizer, which is usually better than imperative query APIs.
    - The fact that SQL is more limited in functionality gives the database much more room for automatic optimizations.
    - Imperative code is very hard to parallelize across multiple cores and multiple machines because it specifies instructions that must be performed in a particular order.
- Join operation is more costly in NoSQL
    - Does Join matters? Data has a tendency of becoming more interconnected as features are added to applications, and at that point, Join does matter.
- MapReduce
    - Google’s MapReduce lies somewhere between Declarative Query and Imperative Query. Its “map” and “reduce”, where both have to be pure functions, which enable “map” and “reduce” to be run in any order and in anywhere.

## TODO

Denormalization and Normalization of a DB

NoSQL v.s. SQL usecase(when and why)