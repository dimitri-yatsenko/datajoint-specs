# DataJoint 2.0 Specification
---
## Introduction

DataJoint extends the relational database model to support scientific data, incorporating embedded computational dependencies.

This document provides API specifications, establishing a common language for implementing computational databases and interacting with them.

The current reference implementation is DataJoint for Python, using MySQL or PostgreSQL as backends. As a result, some definitions may have a Python-oriented flavor. However, this specification is designed for easy adoption in other programming languages while maintaining full interoperability.

It is important to note that this specification does not cover the computational backend for orchestrating compute jobs; it solely defines how such computations are specified.

### Key Objectives

- **Relational database backend** – Built on a rigorous relational model.  
- **Data integrity constraints** – Ensures consistency and validity.  
- **Transaction processing** – Supports atomic, consistent, isolated, and durable (ACID) transactions.  
- **Scientific programming interface** – Enables schema definition, data manipulation, and queries directly from a scientific programming language (e.g., Python).  
- **Management of large data files** – Supports scientific data formats and efficient storage.  
- **Embedded computation** – Integrates computation as a native construct in the data model, using foreign keys to define dependencies.  
- **Extensibility** – Allows for storing complex data structures beyond standard relational types.  

By combining the rigor of the relational data model with support for large scientific datasets and built-in computational workflows, DataJoint empowers scientists to design, implement, and share powerful data pipelines..

### Terminology  

| Term | Definition |
|---|---|
|**Schema**| (a) A collection of table definitions with integrity constraints and (b) a namespace for organizing related tables. |
|**Table**| The single fundamental data structure in the relational data model. A table can be either a named stored table represented as a class or a derived result represented as a query expression. A table consists of named and typed columns (attributes) and unordered rows with values for each attribute. |
|**Attribute** (**Column** or **Field**)| A named attribute  with a specific data type. Identified by name, never by position. |
|**Row** (**Record** or **Tuple**) | A single entry in a table, providing values for each attribute. The order of rows in a table is not significant. Rows are identified and addressed by their primary key.|
|**Query**|A function perfored on the stored data on the server side, expressed as a *query expression* and resulting in a new, derived table.|
|**Fetch**|The execution of a query on the server and transfer of the result to the client.|
|**Transaction**|A sequence of database operations executed as an atomic, consistent, isolated, and durable (ACID-compliant) unit.|

## Schema Definition

### Schema

Tables are organized into schemas. Each schema represents a namespace in the database. 

Schema design is mirrored by the package design of in the scientific language with schemas mapping to modules and tables mapping to classes.

A one-to-one correspondence is strongly recommended between schemas in the databases and separate modules in the programming language.

### Table Definition
Each table definition specifies the table name, table tier, a set of attributes, and a primary key. A table definition may also include foreign keys and seconday indexes. 

### Table Name
Tables are represented as classes in the programming language, whose names follow the CamelCase notation.
The table class `module.ClassName` translates into the corresponding `schema.table_name`, where schema corresponds to module and table name corresponds to class name.

### Table tiers
Each table is designated as `lookup`, `manual`, `imported`, or `computed`.

### Attribute Definition
The table definition defines a number of fields, each on a separate line in the following format:

```
attribute_name: type
attribute_name: type # comment
attribute_name: type = default_value
attribute_name: type = default_value # comment
```

Comments and default values are optional.

A special default value of `null` makes the attribute nullable. There is no other way to make an attribute nullable.

### Primary Key

Each table must have a primary key. The primary separator `---` separates the primary key attributes above from the secondary attributes below. All the attributes above the primary separator, jointly, comprise the primary key.

Primary key attributes cannot have default values.

The primary key separator is required in the table definition. All tables must have a primary key.

The primary key can have one attribute (simple primary key), multiple attributes (composite primary key), or no attributes (singleton table). A singleton table can have at most one row.

### Foreign keys

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

### Lookup tables 
Lookup tables are special tables whose contents is considered part of the schema design rather than project data. 
Therefore, its content is provided as part of the table declaration, although it can evolve over time. 

## Attribute Types
Database supports a small set of native types for column attributes.
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

### File Management

### Custom Types

## Manipulations

### Insert

### Update1

### Delete 


## Queries

A query is a function on the data performed on the server side, yieldig a derived table.
The results 

### Fetch

Fetching is the process of executing the qeury transferring query results from the server to the client. The fetch operation retrieves query output in various formats, typically as dictionaries, lists, or NumPy arrays.

- `fetch()`: Returns query results as a dataframe, a numpy recarray, or a sequence of 
- `fetch1()`: Ensures that only a single row is returned and raises an error if multiple rows are present. The result is typically a dictionary.

### Query Operators

#### Restriction `A & cond` and `A - cond`
- by condition
- by sequence
- by AndList
- by a subquery

#### Projection `A.proj(...)`

#### Join `A * B`

#### Union `A + B`

#### Universal Sets `dj.U()`


### Algebraic Closure

Algebraic closure refers to the property that query operations in DataJoint always yield new tables, which can be further queried without any loss of relational integrity. This ensures that any combination of joins, restrictions, projections, and other operations results in a valid table that can be used as input for further operations.

- **Closure under Join (**``**)**: The result of a join between two tables is always a table that can be queried further.
- **Closure under Restriction (**``**)**: Applying a restriction to a table results in a new table that maintains its relational properties.
- **Closure under Projection (**``**)**: Projection creates a new table with a subset of attributes while preserving integrity.
- **Closure under Aggregation**: Aggregation operations such as `group_by` and computed statistics yield tables that can be used in subsequent queries.

This property enables DataJoint to support composable and declarative data queries in a fully relational manner.

#### Semantic Join Rules

In binary operators (join `A * B`, restrict `A & B`, and anti-restrict `A - B`), must relate rows in table `A` to rows in table `B`.

The match is performed as the equality condition on all pairs of attributes that (a) have the same name in both `A` and `B` and trace to the same attribute definition through an uninterrupted chain of foreign keys.

If the tables `A` and `B` have attributes with the same names but do not trace their lineage to the same original definition, the binary operators will be invalid. Users must remove or rename the colliding attributes before performing the binary operation.


## Computation


