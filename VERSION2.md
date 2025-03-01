# DataJoint 2.0 Specification

DataJoint extends the relational database model to support scientific data embedding computational dependencies.

This document specifies the API, serving as a *common language* for defining computational databases for various host programming languages, database backends, and file storage backends.

## Key features:

1. Relational database integration
2. Data integrity constraints
3. Transaction processing
4. Direct support in scientific programming languages (e.g. Python)
5. Management of large data files
6. Computations are embedded as a native construct of the data model, relying on foreign keys to define computational dependencies.
7. Extensions allow storing complex data structures in the database or in

# Terminology

- **Schema:** (a) a set of table definitions and integrity constraints and (b) a namespace for a collection of related tables.
- **Table**: the principal data structure in the relational data model. A table can be a named table stored in a database schema or the result derived by a query.
- **Attribute**=**Column**=**Field**: Tables have named columns (attributes) with specified data types.
- **Row:** Tables have rows providing values for each attribute. The order of rows in a table is not significant.
- **Query:** a function on stored data performed by the server, yielding a new derived table.
- **Fetch:** transfer of query output from server to client.
- **Transaction:** a series of data manipulations that are performed as an atomic, serializable operation, with ACID compliance.

# Table definition

## Schema

Tables are organized into schemas. Each schema represents a namespace in the database. Each schema is defined as a collection of classes in a dedicated module in the host scientific programming language.

## Table name

Tables are represented as classes in the programming language, whose names follow the CamelCase notation. The table class `module.ClassName` translates into the corresponding `schema.table_name`, where schema corresponds to module and table name corresponds to class name.

## Table tier

Each table is designated as `lookup`, `manual`, `imported`, or `computed`.

## Attribute definition

The table definition defines a number of fields, each on a separate line in the following format:

```
attribute_name: type
attribute_name: type # comment
attribute_name: type = default_value
attribute_name: type = default_value # comment
```

Comments and default values are optional.

A special default value of `null` makes the attribute nullable. There is no other way to make an attribute nullable.

## Primary separator

The primary separator `---` separates the primary key attributes above from the secondary attributes below. All the attributes above the primary separator, jointly, comprise the primary key.

Primary key attributes cannot have default values.

The primary key separator is required in the table definition. All tables must have a primary key.

The primary key can have one attribute (simple primary key), multiple attributes (composite primary key), or no attributes (singleton table). A singleton table can have at most one row.

## Foreign keys

Foreign keys are defined on separate lines by pointing to the class name representing a parent table.

```
-> ClassName
-> [nullable, unique] ClassName
-> ClassName.proj(new_name=old_name)
```

Cyclical dependencies are not allowed.

Properties can be `nullable` and `unique`.

`.proj` renames primary key attributes.

A foreign key has the following effects:

1. The primary key attributes of the parent table are included in the child table definition if they are not already included.
2. A referential dependency is established between the child and the parent.

# Attribute Types

Attributes can use the following data types:

- **UUID**: `uuid` — follows RFC 4122. Defaults are not supported.
- **Integers:** `int8`, `uint8`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64`
- **Scientific**: `float32`, `float64` (`nan` is not supported by MySQL backend)
- **Decimal:** `decimal(M,N)`
- **Character strings**: `char(N)`, `varchar(N)`
- **Enumeration:** `enum('choice1', 'choice2', 'choice3')`
- **Date:** `date` — in ISO 8601 standard. Special value `NOW` can be used for default.
- **Time:** `timestamp`   Microsecond precision in UTC in ISO 8601 standard.
  Special value `NOW` can be used for default.
- **Binary large object:** `blob`
- **File or folder:** `file` or `file@store` where `store` is a named storage backend.
- **Custom type**: `<adaptor_name>` — see Type Adaptors.

# Queries

A query is a function on the data performed on the server side, yieldig a derived table.
The results 

## Fetch

Fetching is the process of executing the qeury transferring query results from the server to the client. The fetch operation retrieves query output in various formats, typically as dictionaries, lists, or NumPy arrays.

- `fetch()`: Returns query results as a dataframe, a numpy recarray, or a sequence of 
- `fetch1()`: Ensures that only a single row is returned and raises an error if multiple rows are present. The result is typically a dictionary.

## Query Operators

## Algebraic Closure

Algebraic closure refers to the property that query operations in DataJoint always yield new tables, which can be further queried without any loss of relational integrity. This ensures that any combination of joins, restrictions, projections, and other operations results in a valid table that can be used as input for further operations.

- **Closure under Join (**``**)**: The result of a join between two tables is always a table that can be queried further.
- **Closure under Restriction (**``**)**: Applying a restriction to a table results in a new table that maintains its relational properties.
- **Closure under Projection (**``**)**: Projection creates a new table with a subset of attributes while preserving integrity.
- **Closure under Aggregation**: Aggregation operations such as `group_by` and computed statistics yield tables that can be used in subsequent queries.

This property enables DataJoint to support composable and declarative data queries in a fully relational manner.

## Semantic Join

In binary operators (join `A * B`, restrict `A & B`, and anti-restrict `A - B`), must relate rows in table `A` to rows in table `B`.

The match is performed as the equality condition on all pairs of attributes that (a) have the same name in both `A` and `B` and trace to the same attribute definition through an uninterrupted chain of foreign keys.

If the tables `A` and `B` have attributes with the same names but do not trace their lineage to the same original definition, the binary operators will be invalid. Users must remove or rename the colliding attributes before performing the binary operation.


