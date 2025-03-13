# DataJoint 2.0 Specifications
---
# Introduction

DataJoint extends the **relational database model** into a **computational database**, where some tables store inputs or source data, while others define computations and store computed results. This approach enables **structured, scalable, and reproducible** scientific data processing.

A **computational database** can serve as a **scientific data pipeline**, explicitly defining dependencies between data acquisition, transformation, and analysis. DataJoint ensures **data integrity**, supports **ACID-compliant transactions**, and integrates seamlessly with scientific programming languages such as **Python**. It also extends relational databases to handle **complex scientific data types** (e.g., multidimensional arrays) and embeds computations using **foreign-key dependencies**.

This document defines the **DataJoint 2.0 API**, a major upgrade focused on **scalability, extensibility, and interoperability**. A **new Python implementation** is planned for **2025**, incorporating modern best practices for usability, performance, and compatibility with scientific computing frameworks. The current [reference implementation](https://github.com/datajoint/datajoint-python) supports **MySQL** and **PostgreSQL** backends. While the API reflects Python conventions, it is designed for **adoption in other languages** with full interoperability.

DataJoint serves as a formal common language for defining **how data and computations are structured**, but does not dictate how computational jobs are executed. It can be combined with external **workflow management systems** (e.g., Apache Airflow, Nextflow) for scheduling computations. Likewise, **graphical user interfaces and dashboards** can be integrated for interactive data exploration—though such integrations are outside the scope of this specification.

By combining the rigor of relational databases with built-in support for scientific data and computations, **DataJoint empowers researchers to design, implement, and share reliable, scalable data workflows**.

## Open-Source Development

This document specifies the **DataJoint open-source framework**, a **free, Python-based library** that empowers scientists to design, manage, and query relational data pipelines. It provides tools for **defining schemas, tracking dependencies, and integrating computations**, but users must manage their own **database, compute, and storage infrastructure**.

For research teams seeking a fully managed solution, [DataJoint Inc.](https://datajoint.com) offers the **Data Operations Platform for Scientific Research**—a **turnkey infrastructure** built on the open-source framework.
The platform provides **hosted databases, scalable object storage, automated computation, and web-based tools** for seamless data exploration and collaboration.

The DataJoint Platform is **flexible** and can be deployed **on-premise, in the cloud, or as a hybrid infrastructure**, ensuring **scalability, reliability, and security** for large-scale scientific workflows.
While the platform’s design is proprietary, researchers maintain full **ownership and control** over their data and pipeline code, meeting **data residency and licensing requirements**—whether running their own infrastructure or leveraging the DataJoint Platform.

## Key Objectives
- **Relational foundation** — Built upon a rigorous relational database model.
- **Data integrity** — Enforces constraints ensuring consistency, correctness, and validity.
- **ACID transactions** — Supports atomic, consistent, isolated, and durable database operations.
- **Scientific programming interface** — Allows schema definitions, data manipulations, and queries directly from languages like Python.
- **Scalability** — Efficiently manages large scientific datasets and complex data types.
- **Embedded computation** — Explicitly integrates computations within the database structure.
- **Extensibility** — Supports complex and custom data structures beyond standard relational types.

## Terminology

DataJoint adopts familiar terms from relational database theory and clearly defines their usage within this specification:

| Term | Definition |
|---|---|
| **Data Pipeline** | Also called a *DataJoint project*, *computational database*, or *workflow*. A structured organization of data and computations that includes a relational database (MySQL or PostgreSQL), a dedicated code repository (e.g., `git`), and a file or object store. A pipeline serves as the **system of record** aggregating primary and computed data for collaborative scientific projects. |
| **Schema** | (a) A collection of related table definitions and integrity constraints, and (b) a namespace organizing related tables. |
| **Table** | The core relational data structure, either stored permanently (base table) or derived temporarily (query result). Tables have named and typed **columns (attributes)** and unordered **rows**. |
| **Attribute** (**Column**/**Field**) | A named, typed element of a table. Always referenced by name, never by position. |
| **Row** (**Record**/**Tuple**) | A single entry in a table with values corresponding to each attribute. Rows are uniquely identified by their **primary key**. |
| **Primary key**| A set of attributes in a table that are designated to uniquely identify any row in that table.  |
| **Query** | A function on stored data, expressed as a [**query expression**](#query-expressions), resulting in a new derived table. |
| **Query Expression** | A formal definition of a query expressed with [**query operators**](#query-operators)  acting on input tables to define a new output table.
| **Fetch** | The execution of a query and transfer of results from server to client. |
| **Transaction** | A sequence of database operations executed as an atomic, consistent, isolated, durable (ACID) unit. All operations succeed or fail together, with partial results invisible externally. |

## Frequently Asked Questions

### Is DataJoint an ORM?

**Object-Relational Mapping (ORM)** is a technique allowing developers to interact with relational databases through object-oriented programming, abstracting direct SQL queries. Popular Python ORMs include **SQLAlchemy** and **Django ORM**, often used in web development.

Although DataJoint shares certain ORM characteristics, it is primarily a **computational database framework** designed explicitly for scientific workflows. Unlike traditional ORMs, which focus mainly on simplifying database interactions for web applications, DataJoint explicitly defines data dependencies and computational relationships, ensuring data integrity, traceability, and reproducibility.

Thus, DataJoint can be considered an **ORM specialized for scientific databases**, purpose-built for structured experimental data and computational workflows.

### Is DataJoint a Workflow Management System?

Not exactly. DataJoint provides robust capabilities for embedding computations within a relational database structure, managing derived data, and tracking explicit data dependencies. However, DataJoint itself does not handle scheduling, distributed execution, or orchestration of parallel computational tasks, which are typical roles of workflow management systems such as Apache Airflow or Nextflow. Instead, DataJoint complements these systems, formalizing data dependencies so that external workflow schedulers can effectively manage computational tasks.

### Is DataJoint a Lakehouse?

DataJoint and lakehouses share some similar goals—such as integrating structured data management with scalable storage and computational capabilities. However, a **lakehouse** typically merges the flexibility of **data lakes** (handling raw, semi-structured data at scale) with the structured schemas and transactional guarantees of traditional databases.

In contrast, **DataJoint** focuses specifically on scientific data workflows, emphasizing rigorous **schema definitions**, explicit **computational dependencies**, and robust **data integrity**. While lakehouses offer flexible analytics on structured and unstructured data, DataJoint prioritizes precise data modeling, reproducibility, and computational traceability within structured scientific datasets.

Therefore, DataJoint complements lakehouse architectures but is tailored specifically for managing structured experimental data and computational pipelines in science.

### Does DataJoint require SQL knowledge?

No, **DataJoint does not require SQL knowledge**, but understanding relational concepts can be helpful.

DataJoint provides a **Python-based API** that abstracts SQL, allowing users to define schemas, insert data, and query tables without writing SQL. Instead of composing SQL queries, users interact with the database using **intuitive Python methods**.

Examples comparing **SQL vs. DataJoint**
| SQL | DataJoint-Python |
|---|---|
| `CREATE TABLE` | Define tables as Python classes |
| `INSERT INTO` | `.insert()` method |
| `SELECT * FROM ...` | `.fetch()`, `.proj()`, `.aggr()` |
| `JOIN` | `table1 * table2` |
| `WHERE ...` | `table & condition` |

Since DataJoint uses relational database backends, all data can be accessed through SQL as well.

---

# Pipeline Design

A **data pipeline** supporting a scientific study is  a structured system for managing **scientific data, dependencies, computations, and execution workflows**.
It organizes **structured data, metadata, and large data objects**, ensuring **data integrity, traceability, and automated processing*
. In addition to handling **computational dependencies and job execution**, a pipeline may also include **graphical interfaces for data navigation, analysis, and collaboration**.

## **Operational Components**

A fully functional DataJoint pipeline consists of the following core components:

### **1. Dedicated Code Repository**
A **shared version-controlled code repository** (e.g., **GitHub, GitLab, Bitbucket**, or a self-hosted `git` instance) serves as the **central hub** for managing pipeline development and execution. It stores:
- **Pipeline definitions** – schemas, table structures, and computational dependencies.
- **Configuration settings** – database, storage, and execution parameters.
- **Access control policies** – permissions and security settings.
- **Containerized environments** – configurations for executing computations reproducibly.
- **Quality Assurance** - unit tests, integration tests
- **Automations** – deployment scripts, CI/CD pipelines, and operational workflows.

Version control ensures **collaborative development, reproducibility, and historical tracking** of pipeline changes. Research teams are responsible for code management, including **reviews, merges, and updates**.

### **2. Relational Database for Structured Data**
A **relational database** (e.g., **MySQL, PostgreSQL**) serves as the **core metadata store** for the pipeline, ensuring:
- **Structured tabular storage** – for experiment data, metadata, and results.
- **Foreign-key relationships** – enforcing **data integrity and traceability**.
- **Transactional support (ACID compliance)** – ensuring consistency in data updates.

The **database acts as the authoritative source of truth**, defining dependencies, data provenance, and analysis results.

Database administrators handle **access control**, ensuring data security.
The **DataJoint Platform** implements **role-based access control (RBAC)** to manage permissions for research teams efficiently.

### **3. Object Storage for Large Data Objects**
A **scalable storage backend** is required for managing **large scientific datasets** that are referenced in the relational database but stored externally.

Supported storage backends include:
- **Local storage** – POSIX-compliant file systems (e.g., NFS, SMB).
- **Cloud-based object storage** – (e.g., Amazon S3, Google Cloud Storage, Azure Blob, MinIO).
- **Hybrid solutions** – Combining local and cloud storage for flexibility.

Object storage enables **efficient handling of unstructured data** (e.g., images, neural recordings, time series) while maintaining **structured metadata** in the relational database.

**Access control for stored objects is synchronized with database permissions**, ensuring parallel security enforcement. The **DataJoint Platform seamlessly integrates databases with object storage** to provide a unified experience.

### **4. Job Orchestration for Automated Computations (Optional)**
For smaller, self-hosted projects, **computations may be executed manually** or through custom scripts. However, for **automated execution**, a **job orchestration system** can:
- **Schedule and execute computations** based on pipeline dependencies.
- **Ensure proper execution order** of dependent tasks.
- **Support distributed or parallel processing** for computational scalability.

#### **Orchestration Options**:
- **DataJoint Compute Service** – A fully integrated job execution system on the DataJoint Platform.
- **Self-managed custom solutions** – User-defined execution workflows.
- **Workflow automation tools** – Apache Airflow, Prefect, Dagster.
- **Distributed job schedulers** – Kubernetes, SLURM, AWS Batch.

Automated job execution improves **auditability, observability, and traceability**, ensuring reproducible computational workflows.

### **5. Web Interfaces for Interactive Data Exploration (Optional)**
Many self-hosted projects **do not require graphical interfaces** beyond basic scripting environments. However, for interactive data exploration, **web-based tools can enhance usability** by providing:
- **Data entry, visualization, and exploration** (e.g., dashboards).
- **Structured metadata querying and filtering**.
- **Collaboration tools** for managing shared datasets.
- **Pipeline monitoring and job execution status tracking**.

**Available Interfaces**:
- **Custom-built dashboards** – Using frameworks like **Streamlit, Dash, or Panel**.
- **Jupyter Notebooks** – Interactive scripting environments.
- **DataJoint Platform Web Tools** – Plotly Dash-based dashboards for data entry and visualization, and Jupyter Notebooks with configurable compute instances.

## **Pipeline ≡ Python Package**

A **DataJoint pipeline** is typically implemented as a **dedicated Python package**, where **modules correspond to database schemas**. This structure ensures that data, computations, and dependencies remain **organized, traceable, and reproducible**.

A **DataJoint pipeline follows a directed acyclic graph (DAG) structure**, where:

- **Nodes** represent **Python modules** (database schemas).
- **Edges** represent **dependencies**, including:
  - **Python import dependencies** between modules.
  - **Referential dependencies** (foreign keys) between tables, defining data flow.

![Pipeline Design](figures/pipeline-illustration.png)

> **Cyclical dependencies are not allowed** – All referential constraints within a schema must also form a **DAG**, meaning that foreign keys cannot create circular dependencies.

## **Database Schema ≡ Python Module**

Each **database schema** in DataJoint corresponds to a **Python module**, serving as a **namespace** for organizing related tables. Maintaining a **one-to-one mapping** between database schemas and Python modules ensures modular, maintainable code.

![Schema Design](figures/schema-illustration.png)

- **Schemas in the database** group related tables with shared dependencies.
- **Schemas in Python** are structured as **separate modules**, keeping the code modular and scalable.
- **Foreign keys within a schema must also form a DAG**, ensuring **a unidirectional flow of data dependencies**.

Maintaining this structured mapping ensures that the **database and code remain in sync**, facilitating reproducibility and collaboration.

## **Database Table ≡ Python Class**

In DataJoint, **each table is represented as a Python class**, following a consistent naming convention:

| Python | Database |
|---|---|
| **Class Names** | Written in **CamelCase** |
| **Table Names** | Written in **snake_case** |
| **Fully Qualified Python Class Name** | `<module>.<ClassName>` |
| **Fully Qualified Database Table Name** | `<schema_name>.<table_name>` |

**Example:**
- **Python Class:** `scan.ScanLocation` ≡ **Database Table:** `scan.scan_location`

This **naming convention** ensures clarity, consistency, and seamless alignment between the **Python implementation** and the **underlying database schema**.

## Table Tiers

Each table is assigned to one of **four tiers**, defining how data is populated and maintained. In diagrams and visualizations, **color codes** are used to distinguish these tiers:

| Tier | Description | Color Code (in Diagrams) |
|---|---|---|
| `lookup` | Static reference data that is part of the schema definition (e.g., parameters, controlled vocabularies). | **Gray** |
| `manual` | Data manually entered from external sources, typically by users. | **Green** |
| `imported` | Data automatically ingested from external sources (e.g., raw data files, external databases). | **Blue** |
| `computed` | Data generated from upstream tables through **automated computations**. | **Red** |

> **Note:**
> GitHub Markdown does not support colored text formatting, but these color codes are **used in diagrams** and can be applied in external documentation formats such as HTML, LaTeX, or GUI-based schema visualization tools.

These tiers ensure **clear separation** between manually curated, automatically imported, and computed data, preserving **data integrity and provenance** across the pipeline.


# Table Definition

Each table definition consists of:

- **Table name** – Defined as a class in Python and translated to a database table name.
- **Table tier** – Specifies the table's role in the pipeline.
- **Primary key** – Defines the unique identifier for each row.
- **Attributes** – Columns specifying the table’s data structure.
- **Foreign keys** – Define dependencies on upstream tables.
- **Indexes** – Optimize queries and enforce constraints.


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

## Indexes
Besides the primary key, a table may have a number of secondary indexes on combinations of fields.
An additional index may be used to speed up searched by those fields.
This is done in the format `index (attr1, attr2)` or `unique index (attr1, ..., attrn)`.

Example table with additional unique indexes:
```python
@schema
class Person(dj.Manual):
    definition = """
    # Table representing a person with unique identifiers
    person_id        : uint32  # Unique person identifier
    ---
    first_name       : varchar(50)
    last_name        : varchar(50)
    drivers_license  : varchar(20) = null
    cell_phone       : varchar(15) = null
    email            : varchar(100) = null

    index (last_name, first_name)
    unique index (drivers_license)
    unique index (cell_phone)
    unique index (email)
    """
```
This definition allows fast searches by `last_name`, `(last_name, first_name)`, as well as by other attributes with indices.
Note that a unique index on a nullable field does not prevent multiple rows with `null` in the unique field.


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
3. An implicit index is created in the child table on the foreign key to speed up matches on foreign key attributes.

## Lookup tables
Lookup tables are special tables whose contents is considered part of the schema design rather than project data.
Therefore, its content is provided as part of the table declaration, although it can evolve over time.

For example, the following table specifies the periodic table of elements:
```python
import datajoint as dj

schema = dj.Schema('chemistry')

@schema
class ChemicalElement(dj.Lookup):
    definition = """
    # Lookup table for chemical elements
    atomic_number : uint8       # Atomic number
    ---
    symbol        : char(2)          # Chemical symbol
    name          : varchar(20)      # Element name
    atomic_weight : decimal(7, 4)    # Standard atomic weight
    """
    contents = [
        {'atomic_number': 1, 'symbol': 'H',  'name': 'Hydrogen',  'atomic_weight': 1.008},
        {'atomic_number': 2, 'symbol': 'He', 'name': 'Helium',    'atomic_weight': 4.0026},
        {'atomic_number': 3, 'symbol': 'Li', 'name': 'Lithium',   'atomic_weight': 6.94},
        {'atomic_number': 4, 'symbol': 'Be', 'name': 'Beryllium', 'atomic_weight': 9.0122}
    ]

```

## Attribute Types

DataJoint supports a **small set of fundamental types** for storing attributes in database columns.
These types **abstract and simplify** the underlying SQL data types provided by relational backends ([PostgreSQL](https://www.postgresql.org/docs/current/datatype.html) and [MySQL](https://dev.mysql.com/doc/refman/8.4/en/data-types.html)) to align with the needs of **data science and experimental workflows**.

This specification **prioritizes intuitive type names** (e.g., `uint8` instead of SQL’s `tinyint unsigned`) to improve usability for **scientific applications**.

### **Supported Attribute Types**

| Category | Type | Description |
| --- | --- | --- |
| **UUID** | `uuid` | Universally unique identifier (RFC 4122). Default values are not supported. |
| **Integers** | `int8`, `uint8`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64` | Standard integer types. |
| **Scientific** | `float32`, `float64` | Floating-point numbers. `NaN` is not supported in the MySQL backend. |
| **Decimal** | `decimal(M,N)` | Fixed-point decimal with precision `M` and scale `N`. |
| **Character Strings** | `char(N)`, `varchar(N)` | Fixed or variable-length character data. |
| **Enumeration** | `enum('value1', 'value2', 'value3')` | A predefined set of allowed values. |
| **Date** | `date` | ISO 8601 format. Special value `NOW` can be used as a default. |
| **Time** | `timestamp` | Microsecond precision in UTC (ISO 8601). Special value `NOW` can be used as a default. |
| **Binary Large Object (BLOB)** | `blob` | Stores large binary data inside the database. |
| **External Object Reference** | `object` | A reference to an **external object** managed by DataJoint (e.g., a file or dataset stored in an object store or file system). |
| **Custom Type** | `<adaptor_name>` | User-defined [type adaptors](#type-adaptors) for specialized data handling. |

### **Blob vs. Object**
| Type | Best For | Where Data is Stored |
| --- | --- | --- |
| `blob` | **Raw binary data stored inside the database** | **Database storage (e.g., MySQL, PostgreSQL BLOB columns)** |
| `object` | **Externally stored files, datasets, or objects** | **File systems, object stores (S3, GCS, Azure Blob), or network storage (NFS, SMB)** |


## Type Adaptors

---
# Object Storage
DataJoint combines **relational databases** for structured metadata and **object stores** for large scientific datasets.
The relational database ensures **data integrity, fast querying, and transactional consistency**, while object storage efficiently handles **high-volume, unstructured data** (e.g., images, arrays).
This hybrid approach **scales seamlessly**, keeping metadata relational while allowing **flexible, distributed storage** for large data objects.


## Object Management
In DataJoint tables, the `object` datatype enables **object-augmented schemas**, where structured metadata resides in a relational database, while large scientific data objects are stored externally.
This integration allows **structured tabular data** to reference **large, complex data structures** (e.g., multidimensional arrays, images, time series).

Objects are managed similarly to other database attributes but are stored in a **configurable storage backend**, using a structured key-naming convention.
They are created and accessed through standard **DataJoint operations**: `insert`, `delete`, and `fetch`.
Operations on stored objects maintain the **same consistency and integrity guarantees** as data stored in the database.


A DataJoint client is configured to access the **storage backend** associated with each database.
For each **insert** and **fetch** operation, the client **constructs a structured key** for object fields.
The corresponding **database entry** stores metadata such as:
- **Object key (structured reference to the stored object)**
- **File format/extension**
- **Size**
- **Checksum (for integrity verification)**
- **Tags (for metadata indexing and lookup)**

The **organizational structure of stored objects is configurable**, allowing **logical partitioning** based on primary key attributes.


## Storage Backend Configuration
A DataJoint client is configured to access the storage backend associated with each database.
For each insert and fetch operation, the client constructs the relative path for the file fields.
The database entry for the file type stores metadata such as file extension, size, checksum, and tags.
The structure is configurable as part of the storage backend configuration.

### **Example Structured Object Storage Pattern**

A typical storage structure follows a **structured key-naming convention**, maintaining logical organization:

```bash
📁 project_name/
├── datajoint.toml
├──📁 schema_name1/
├──📁 schema_name2/
├──📁 schema_name3/
│  ├── schema.py
│  ├── 📁 tables
│  │   ├── table1/key1-value1.parquet
│  │   ├── table2/key2-value2.parquet
│  │   ...
│  ├── 📁 objects
│  │   ├── table1-field1/key3-value3.zarr
│  │   ├── table1-field2/key3-value3.gif
...  ...
```
When using object storage, the corresponding object keys might be:
```
s3://project_name/schema_name3/objects/table1/key1-value1.parquet
s3://project_name/schema_name3/fields/table1-field1/key3-value3.zarr
```

This structured naming approach allows:
* Scalable organization (even in flat object storage systems).
* Efficient indexing and retrieval using key-based lookups.
* Cross-platform compatibility across file systems and cloud object stores.

This file hierarchy serves to store a complete copy of the tabular relational data in the `tables` subfolder and the contents of the file fields in the fields subfolder.
The table name, field name, and primary key attributes form the path using a configurable pattern.
Several options are available for partitioning this file repository based on the values of a specific subset of primary key attributes.```

The `datajoint.toml` configuration file is essential for setting up a DataJoint store to support a project.
It defines the repository's organization, and key information within this file should remain unchanged after data population to avoid issues with existing data.

## Partitioning
The storage backend configuration supports logical partitioning, allowing DataJoint to organize stored objects using a structured key pattern.
For example, configuring partitioning in datajoint.toml:
```
# Configuration for object storage
[object_storage]
partition_pattern = "subject{subject_id}/session{session_id}"
```

Here:
* `[object_storage]` defines settings for storage backend configuration.
* `partition_pattern` dynamically replaces placeholders `({subject_id}, {session_id})` with actual values during object storage operations.

```
s3://my-bucket/subject123/session45/image1.tiff
s3://my-bucket/subject124/session46/image2.tiff
```
This logical organization improves data navigation and retrieval while maintaining compatibility with object storage paradigms.

## File Operations
The DataJoint client uses protocols such as [`fsspec`](https://filesystem-spec.readthedocs.io/en/latest/), supporting various storage backends.
During insertion, a field of type `file` expects a bytestream or a callback function that accepts a path as its argument and writes its contents to that path.
The DataJoint client constructs the file structure and hierarchy according to the specied configuration.

## Export, Sharing, and Backup/Restore
The structured project data folder serves as a comprehensive, publishable copy of the data, accessible and querying  by DataJoint and  other tools.
It is designed for both human navigation and tool accessibility.
The Schema Diagram object provides methods for selecting portions of the data for export, including specific tables and data slices.

The export method recreates the data repository in full at another location.
DataJoint offers tools to restore the database from the file repository using the tabular data, supporting data publishing and pipeline cloning alongside the data.

-------
# Data Operations
Data operations atler the state of the data stored in the database.

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
The method `Table.insert(query_expression)` to insert the result of a [query expression](#query-expressions).
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
The delete operation removes records from a stored tables and cascades to all dependent records.
Delete is often used in combination with the restriction operator to specify records to delete.

Example:
```python
# delete all students
Student.delete()

# delete specific students
(Student & "student_id IN (500, 501, 503)").delete()
```

## Update
Update is used to change the values of secondary attributes in one record only.
The `Table.update1(rec)` operator takes a mapping, `rec`, which must provide the complete value of the primary key for the updated record and the new values for all updated attributes.

Example:
```python
# Update email and cellphone for student id 1001
Student.update1({
    'student_id': 1001,
    'email': 'new_email@example.com',
    'cell_phone': '(555)123-4567'
})
```

Note that this formulation is incapable of updating primary key attributes.
The record must already exist and only secondary attributes will be updated.
Modifying the primary key requires deleting and reinserting the record.


## Effect of Foreign Key Constraints
Foreign keys enforce referential integrity, which is enforced by restricting how insert, delete, and update1 operations behave.

* **Insert**: When using `insert` or `insert1`, foreign key constraints require that referenced records exist in the parent table before dependent records can be added.
This ensures that no orphaned entries are created.
If a foreign key references a non-existent record, the insertion will fail.
This rule helps maintain consistency by preventing invalid references in dependent tables.

* **Delete**: On deletion (`delete`), foreign keys enforce cascading behavior, meaning that deleting a referenced record automatically removes all dependent records downstream.
This prevents data inconsistencies by ensuring that no dependent records exist without their required parent records.

* **Update**: For updates, `update1` modifies secondary attributes of a single record. An update can fail if the secondary attribute is part of a foreign key and no matching record is found in the parent table.

* **Drop**: When a table is dropped from the schema, all dependent tables will be dropped as well. DataJoint client provides a full list of tables to be dropped before executing the schema update.

------------
# Queries

A query is a function on the data performed on the server side, yieldig a derived table.
The results

## Fetch

Fetching is the process of executing the qeury transferring query results from the server to the client. The fetch operation retrieves query output in various formats, typically as dictionaries, lists, or NumPy arrays.

- `fetch()`: Returns query results as a dataframe, a numpy recarray, or a sequence of dictionaries.
- `fetch1()`: Ensures that only a single row is returned and raises an error if multiple rows are present. The result is typically a dictionary.

## Query Expressions


## Query Operators

## Restriction
`A & cond` and `A - cond`

### Restriction by a Condition

### Restriction by a Sequence
- by AndList

### Restriction by a Subquery

## Projection
`A.proj(...)`

## Join `A * B`

## Union `A + B`

## Universal Sets `dj.U()`


## Algebraic Closure

In DataJoint, the operands and the output of any query operator are well-formed relational tables having named columns of known data types and a well-defined primary key.
This property is described as as *algebraic closure*, which is essential for constructing complex queries from simple ones.


from a query expression is  produce tables that  in DataJoint always yield new tables, which can be further queried without any loss of relational integrity. This ensures that any combination of joins, restrictions, projections, and other operations results in a valid table that can be used as input for further operations.

- **Closure under Join (**``**)**: The result of a join between two tables is always a table that can be queried further.
- **Closure under Restriction (**``**)**: Applying a restriction to a table results in a new table that maintains its relational properties.
- **Closure under Projection (**``**)**: Projection creates a new table with a subset of attributes while preserving integrity.
- **Closure under Aggregation**: Aggregation operations such as `group_by` and computed statistics yield tables that can be used in subsequent queries.

This property enables DataJoint to support composable and declarative data queries in a fully relational manner.

## Semantic Match
For binary operators in becomes necessary to match rows in one

In binary operators (join `A * B`, restrict `A & B`, and anti-restrict `A - B`), must relate rows in table `A` to rows in table `B`.

The match is performed as the equality condition on all pairs of attributes that (a) have the same name in both `A` and `B` and trace to the same attribute definition through an uninterrupted chain of foreign keys.

If the tables `A` and `B` have attributes with the same names but do not trace their lineage to the same original definition, the binary operators will be invalid. Users must remove or rename the colliding attributes before performing the binary operation.


-----------
# Computation


