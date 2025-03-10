# DataJoint 2.0 Specification
---

# Introduction

DataJoint is a software framework for managing scientific data and computations.
It relies on the relational database model for data organization, transaction processing, and queries.
Computation is incorporated as a first-class citizen in the data model. 

This document provides API specifications, establishing a common language for implementing computational databases and interacting with them.

The reference implementation is for Python, using MySQL or PostgreSQL as backends.
As a result, some definitions may have a Python-oriented flavor.
However, this specification is designed for easy adoption in other programming languages while maintaining full interoperability.

It is important to note that this specification does not cover implementation details for the computational backend, which  orchestrates compute jobs; its scope is limited to the formal definition of such computations.

## Key Objectives

- **Relational database backend** – Built on a rigorous relational model.  
- **Data integrity constraints** – Ensures consistency and validity.  
- **Transaction processing** – Supports atomic, consistent, isolated, and durable (ACID) transactions.  
- **Scientific programming interface** – Enables schema definition, data manipulation, and queries directly from a scientific programming language (e.g., Python).  
- **Management of large data files** – Supports scientific data formats and efficient storage.  
- **Embedded computation** – Integrates computation as a native construct in the data model, using foreign keys to define dependencies.  
- **Extensibility** – Allows for storing complex data structures beyond standard relational types.  

By combining the rigor of the relational data model with support for large scientific datasets and built-in computational workflows, DataJoint empowers scientists to design, implement, and share powerful data pipelines..

## Terminology  

| Term | Definition |
|---|---|
|**Schema**| (a) A collection of table definitions with integrity constraints and (b) a namespace for organizing related tables. |
|**Table**| The single fundamental data structure in the relational data model. A table can be either a named stored table represented as a class or a derived result represented as a query expression. A table consists of named and typed columns (attributes) and unordered rows with values for each attribute. |
|**Attribute** (**Column** or **Field**)| A named attribute  with a specific data type. Identified by name, never by position. |
|**Row** (**Record** or **Tuple**) | A single entry in a table, providing values for each attribute. The order of rows in a table is not significant. Rows are identified and addressed by their primary key.|
|**Query**|A function performed on the stored data on the server side, expressed as a *query expression* and resulting in a new, derived table.|
|**Fetch**|The execution of a query on the server and transferring the result to the client.|
|**Transaction**|A sequence of database operations executed as an atomic, consistent, isolated, and durable (ACID-compliant) unit.|

---

# Schema Definition

## Schema

Tables are organized into schemas. Each schema represents a namespace in the database. 

Schema design is mirrored by the package design of in the scientific language with schemas mapping to modules and tables mapping to classes.

A one-to-one correspondence is strongly recommended between schemas in the databases and separate modules in the programming language.

## Table Definition
Each table definition specifies the table name, table tier, a set of attributes, and a primary key. A table definition may also include foreign keys and seconday indexes. 

## Table Name
Tables are represented as classes in the programming language, whose names follow the CamelCase notation.
The table class `module.ClassName` translates into the corresponding `schema.table_name`, where schema corresponds to module and table name corresponds to class name.

## Table tiers
Each table is designated as `lookup`, `manual`, `imported`, or `computed`.

## Attribute Definition
The table definition defines a number of fields, each on a separate line in the following format:

```
attribute_name: type
attribute_name: type # comment
attribute_name: type = default_value
attribute_name: type = default_value # comment
```

Comments and default values are optional.

A special default value of `null` makes the attribute nullable. There is no other way to make an attribute nullable.

## Primary Key

Each table must have a primary key. The primary separator `---` separates the primary key attributes above from the secondary attributes below. All the attributes above the primary separator, jointly, comprise the primary key.

Primary key attributes cannot have default values.

The primary key separator is required in the table definition. All tables must have a primary key.

The primary key can have one attribute (simple primary key), multiple attributes (composite primary key), or no attributes (singleton table). A singleton table can have at most one row.

## Foreign keys

Foreign keys are defined on separate lines by pointing to the class name representing a parent table.

```
-> ParentClassName
-> [nullable, unique] ParentClassName
-> ParentClassName.proj(new_name=old_name)
```

Cyclical dependencies are not allowed.

Attribute properties can be `nullable` and `unique`.

`.proj` is used to rename primary key attributes to allow changing foreign key attributes from the parent.

A foreign key has the following effects:

1. The primary key attributes of the parent table are included in the child table definition if they are not already included.
2. A referential dependency is established between the child and the parent.

## Indexes

## Lookup tables 
Lookup tables are special tables whose contents is considered part of the schema design rather than project data. 
Therefore, its content is provided as part of the table declaration, although it can evolve over time. 

For example, the following table specifies the days of the 

## Basic Attribute Types
Database supports a small set of basic types for column attributes for storing them 
However, they include *binary large objects* (blobs) and files for storing large scientific data.
Type adaptors allow defining custom types stored into the native attribute types.
The spec sides with names that are more convenient for data scientists (e.g. `uint8` rather than SQL's `tinyint unsigned`)

Attributes can be declared with the following types:

| Category | Identifies  |  Comment |
| --- | --- | --- |
| **UUID** | `uuid` | universally unique identifiers as defined in RFC 4122. Defaults values are not supported. |
| **Integers:** | `int8`, `uint8`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64` ||
| **Scientific** | `float32`, `float64` | `nan` is not supported by MySQL backend |
| **Decimal** | `decimal(M,N)` ||
| **Character strings** | `char(N)`, `varchar(N)` ||
| **Enumeration** | `enum('value1', 'value2', 'value3')` ||
| **Date** | `date` | in ISO 8601 standard. Special value `NOW` can be used as default |
| **Time** | `timestamp`  |  Microsecond precision in UTC in ISO 8601 standard. Special value `NOW` can be used for default. |
| **Binary large object** | `blob` ||
| **File or folder** | `file`, `file@store` | where `store` is a named storage backend.
| **Custom type** | `<adaptor_name>` | see Type Adaptors |

## File Management

## Custom Types

-------
# Data Manipulation
Data manipulations are operations that atler the state of the data stored in the database. 

Records (rows) in a stored table are considered immutable: each is inserted or deleted as a whole using `insert` and `delete` operators.

However, to allow correcting erros in existing data, the `update1` operator is provided to perform deliberate corrections in the data by changing the values of individual attributes.

Insert comes in two flavors: `insert1` for individual records and `insert` for batches of records.

## Insert1

The method `Table.insert1(rec)`inserts one record `rec` into one table. 

The record to be inserted must be fully formed and comply with all data integrity constaints: 
* All attributes must be of the right data type, i.e. must meet attribute domain constraints specified in the table definition.
* The record must provide values for all required fields.
* Unique constraints must be satisfied (no duplicate value)
* Referential integrity must be uphead (foreign keys must reference existing records).

If the record violates any constraints, ensuring atomicity—either the record is successfully inserted, or nothing is changed.

A record value may be specified as an ordered tuple, in which case all fields must be present or as a mapping (dict), in which case optional fields may be omitted.

Example:
```python
# insert a student record
Student.insert1({
    'student_id': 1002,
    'first_name': 'Alice',
    'last_name': 'Johnson',
    'sex': 'F',
    'date_of_birth': '1998-03-12',
    'home_address': '1234 Elm Street',
    'home_city': 'Springfield',
    'home_state': 'IL',
    'home_zipcode': '62704',
    'home_phone': '(217)555-1234'
})
```


## Batch Insert
The method `Table.insert(seq)` inserts a sequence of records. 
In this case, entire sequence of records is inserted in a single transacation: if a single records fails to insert, the entire sequence fails. 
This makes the `insert` operator atomic. 

Example:
```python
Student.insert([
    {'student_id': 1000, 'first_name': 'Rebecca', 'last_name': 'Sanchez',
     'sex': 'F', 'date_of_birth': '1997-09-13', 'home_address': '6604 Gentry Turnpike Suite 513',
     'home_city': 'Andreaport', 'home_state': 'MN', 'home_zipcode': '29376',
     'home_phone': '(250)428-1836'},
    {'student_id': 1001, 'first_name': 'Matthew', 'last_name': 'Gonzales',
     'sex': 'M', 'date_of_birth': '1997-05-17', 'home_address': '1432 Jessica Freeway Apt. 545',
     'home_city': 'Frazierberg', 'home_state': 'NE', 'home_zipcode': '60485',
     'home_phone': '(699)755-6306x996'}
])
```


## Query Insert 
The method `Table.insert(query_expression)` to insert the result of a [query expression](#query-expression).
In this case, the query expression is treated similarly to the inserted sequence in batch insert and must meet the same requirements.

The remarkable property of query insert is that no data is fetched to the client.
Both the source query expression and the subsequent insert are performed server-side without sendig data to the client.
Both the query and the insert are performed as an atomic transaction. 

Example:
```python
# insert all possible majors for all students
StudentMajor.insert(Department.proj() * Student.proj()) 
```

## Delete 

## Update1


------------
# Queries

A query is a function on the data performed on the server side, yieldig a derived table.
The results 

## Fetch

Fetching is the process of executing the qeury transferring query results from the server to the client. The fetch operation retrieves query output in various formats, typically as dictionaries, lists, or NumPy arrays.

- `fetch()`: Returns query results as a dataframe, a numpy recarray, or a sequence of 
- `fetch1()`: Ensures that only a single row is returned and raises an error if multiple rows are present. The result is typically a dictionary.

## <a name="qeury-expression"></a>Query Expressions


## Query Operators

### Restriction `A & cond` and `A - cond`
- by condition
- by sequence
- by AndList
- by a subquery

### Projection `A.proj(...)`

### Join `A * B`

### Union `A + B`

### Universal Sets `dj.U()`


## Algebraic Closure

Algebraic closure refers to the property that query operations in DataJoint always yield new tables, which can be further queried without any loss of relational integrity. This ensures that any combination of joins, restrictions, projections, and other operations results in a valid table that can be used as input for further operations.

- **Closure under Join (**``**)**: The result of a join between two tables is always a table that can be queried further.
- **Closure under Restriction (**``**)**: Applying a restriction to a table results in a new table that maintains its relational properties.
- **Closure under Projection (**``**)**: Projection creates a new table with a subset of attributes while preserving integrity.
- **Closure under Aggregation**: Aggregation operations such as `group_by` and computed statistics yield tables that can be used in subsequent queries.

This property enables DataJoint to support composable and declarative data queries in a fully relational manner.

## Semantic Join Rules

In binary operators (join `A * B`, restrict `A & B`, and anti-restrict `A - B`), must relate rows in table `A` to rows in table `B`.

The match is performed as the equality condition on all pairs of attributes that (a) have the same name in both `A` and `B` and trace to the same attribute definition through an uninterrupted chain of foreign keys.

If the tables `A` and `B` have attributes with the same names but do not trace their lineage to the same original definition, the binary operators will be invalid. Users must remove or rename the colliding attributes before performing the binary operation.


-----------
# Computation


