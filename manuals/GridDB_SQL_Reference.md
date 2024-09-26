# GridDB SQL Reference

# Table of Contents
* [Introduction](#introduction)
* [SQL description format](#SQL-description-format)
* [SQL commands supported by griddb](#sql-commands-supported-by-griddb)
* [Metatables](#metatables)
* [Reserved words](#reserved-words)

---
# Introduction

This document describes SQL of NewSQL interface for accessing the database supported by GridDB Community Edition (hereinafter referred to as GridDB CE). Please note that this is a different interface than the NoSQL interface.
The NewSQL interface can be referenced and updated by regarding the container created with the NoSQL interface as a table. Updates include not only row updates, but container schema and index changes. Also, the table created by NewSQL interface can be referenced and updated by NoSQL interface as a container.

---
# SQL description format

This chapter shows the descriptive format of the SQL that can be used in the NewSQL interface.



## Usable operations

Besides the SELECT command, DDL command (Data Definition Language) such as CREATE TABLE, and INSERT/DELETE are also supported. See [SQL commands supported by GridDB](#sql_commands_supported_by_griddb) for details.

  

## Data types

<a id="data_types_used_in_data_storage"></a>
### Data types used in data storage

The data types used for data storage in the NewSQL interface are shown in the table below. These data type names can be specified as a column data type when creating a table.

| Data types | Description                                               |
|-------------|--------------------------------------------------------|
| BOOL | True/False                                             |
| BYTE | Integer value from -2<sup>7</sup> to 2<sup>7</sup>-1 (8 bit)     |
| SHORT | Integer value from -2<sup>15</sup> to 2<sup>15</sup>-1 (16 bit)  |
| INTEGER | Integer value from -2<sup>31</sup> to 2<sup>31</sup>-1 (32 bit)  |
| LONG | Integer value from -2<sup>63</sup> to 2<sup>63</sup>-1 (64 bit)  |
| FLOAT | Single-precision data type (32 bits), floating-point number defined in IEEE754        |
| DOUBLE | Double-precision data type (64 bits), floating-point number defined in IEEE754        |
| TIMESTAMP | Data type representing a date and time pair. Precision can be specified from among millisecond, microsecond, and nanosecond precision. If unspecified, millisecond precision will be used.  |
| STRING | Text that is composed of an arbitrary number of characters using the unicode code point.    |
| BLOB | Data type for binary data such as images and voice, etc.<br>Large objects to be saved directly in the input format.<br>The character x or X can also be added to create a hexadecimal expression such as X'23AB'.   |

A NULL value can be registered to table. The results of operators that is related to NULL value such as "IS NULL" are SQL-compliant.

### How to specify TIMESTAMP precision

A TIMESTAMP type is a type representing a date and time pair.
For time, precision can be specified from among millisecond, microsecond, and nanosecond. 
Precision is specified using the format TIMESTAMP(p). For the precision value p, use one of the following: 3, 6, or 9.
A TIMESTAMP type with this specified value is called a TIMESTAMP type with specified precision.
Specifically, precision can be defined using one of the following type names corresponding to millisecond, microsecond, and nanosecond.

| Type name     |       Description                                         |
|-------------|--------------------------------------------------------|
| TIMESTAMP | Represents time data up to millisecond precision (default precision)  |
| TIMESTAMP(3) | Represents time data up to millisecond precision |
| TIMESTAMP(6) | Represents time data up to microsecond precision |
| TIMESTAMP(9) | Represents time data to up to nanosecond precision.  |

[Memo]
 - The row key for a timeseries container is fixed to millisecond-precision TIMESTAMP; TIMESTAMP with any other precision is not allowed.

### Expression that can be specified as a column data type when creating a table

In the NewSQL interface, for data type names that are described as column data types when the table was created, even if the name does not match the data type name given in [Data types used in data storage](#data_types_used_in_data_storage), follow the rules to interpret and determine the data type to use for data storage.

Check the following rules in sequence starting from the top and determine the data type to use for data storage based on the applicable rule.  The data type name described when checking the rules and the strings to check using the rules are not case sensitive.
If multiple rules apply, the rule ranked higher will be prioritized.
If no rules are applicable, an error will occur and table creation will fail.

| Rule no. | Data type names, that were described as column data types when the table was created     | Column type of the table to be created         |
|-----------|----------------------------------------------------|------------------------------------|
| 1        | Type names listed in [Data types used in data storage](#data_types_used_in_data_storage) | Same as specified type |
| 2        | REAL                                                                                     | DOUBLE                           |
| 3        | TINYINT                                                                                  | BYTE                             |
| 4        | SMALLINT                                                                                 | SHORT                            |
| 5        | BIGINT                                                                                   | LONG                             |
| 6        | Type name including "INT"                                                                | INTEGER                          |
| 7        | Type name including any of "CHAR", "CLOB", "TEXT"                                        | STRING                           |
| 8        | Type name including "BLOB"                                                               | BLOB                             |
| 9        | Type name including any of "REAL", "DOUB"                                                | DOUBLE                           |
| 10       | Type name including "FLOA"                                                               | FLOAT                            |

An example to determine the data type using this rule is shown.
-   Name of specified data type is "BIGINTEGER" -> INTEGER (Rule 6)
-   Name of specified data type is "LONG" -> LONG (Rule 1)
-   Name of specified data type is "TINYINT" -> BYTE (Rule 3)
-   Name of specified data type is "FLOAT" -> FLOAT (Rule 1)
-   Name of specified data type is "VARCHAR" -> STRING (Rule 7)
-   Name of specified data type is "CHARINT" -> INTEGER (Rule 6)
-   Name of specified data type is "BIGBLOB" -> BLOB (Rule 8)
-   Name of specified data type is "FLOATDOUB" -> DOUBLE (Rule 9)
-   Name of specified data type is "INTREAL" -> INTEGER (Rule 6)
-   Name of specified data type is "FLOATINGPOINT" -> INTEGER (Rule 6)
-   Name of specified data type is "DECIMAL" -> error

Describe the data type as follows in the NewSQL interface when using the data type equivalent to the one used in the clients of the NoSQL interface. However, some data types may not be used as the equivalent type do not exist.

| Data type in the NoSQL interface in the client | Equivalent column data type descriptions in NewSQL interface |
|--------------------------------------|----------------------------------------------------|
| STRING (string data type)                      | STRING or "Expression to be STRING"               |
| BOOL (Boolean)                                 | BOOL                                         |
| BYTE (8-bit integer)                           | BYTE or "Expression to be BYTE"                   |
| SHORT (16-bit integer)                         | SHORT or "Expression to be SHORT"                 |
| INTEGER (32-bit integer)                       | INTEGER or "Expression to be INTEGER"             |
| LONG (64-bit integer)                          | LONG or "Expression to be LONG"                   |
| FLOAT (32 bitwise floating point number)       | FLOAT or "Expression to be FLOAT"                  |
| DOUBLE (64 bitwise floating point number)      | DOUBLE or "Expression to be DOUBLE"                |
| TIMESTAMP (time data type)                     | TIMESTAMP                                    |
| GEOMETRY (spatial data type)                   | Cannot be specified as a data type of the column when creating a table       |
| BLOB                                           | BLOB or "Expression to be BLOB"                  |
| ARRAY                                          | Cannot be specified as a data type of the column when creating a table       |

[Memo]
   - For TIMESTAMP, the precision value (i.e., p in TIMESTAMP(p)) of TIMESTAMP with NewSQL precision needs to be defined by referring to precision information using the NoSQL interface. For how to refer to precision information, see "GridDB Java API Reference" ([GridDB_Java_API_Reference.html](GridDB_Java_API_Reference.html)) or "GridDB C API Reference" ([GridDB_C_API_Reference.html](GridDB_C_API_Reference.html)).

### Data type when accessing a container as a table and the treatment of the values

The container created with the NoSQL interface client is handled as follows using the container's column type and value when accessing it with the NewSQL interface:

| Column type of container | Data type mapped in NewSQL                      | Value             |
|--------------------|----------------------------------|----------------|
| STRING                   | STRING                                          | Same as original value   |
| BOOL                     | BOOL                                            | Same as original value   |
| BYTE                     | BYTE                                            | Same as original value   |
| SHORT                    | SHORT                                           | Same as original value   |
| INTEGER                  | INTEGER                                         | Same as original value   |
| LONG                     | LONG                                            | Same as original value   |
| FLOAT                    | FLOAT                                           | Same as original value   |
| DOUBLE                   | DOUBLE                                          | Same as original value   |
| TIMESTAMP                | TIMESTAMP                                       | Same as original value   |
| GEOMETRY                 | Same data type as NULL constant (Types.UNKNOWN) | All the values are NULL |
| BLOB                     | BLOB                                            | Same as original value   |
| ARRAY                    | Same data type as NULL constant (Types.UNKNOWN) | All the values are NULL |


[memo]
   - For TIMESTAMP, precision information for NoSQL TIMESTAMP needs to be set by checking the precision value (i.e., p in TIMESTAMP(p)) of TIMESTAMP with SQL precision. For how to set the precision information, see "GridDB Java API Reference" ([GridDB_Java_API_Reference.html](GridDB_Java_API_Reference.html)) or "GridDB C API Reference" ([GridDB_C_API_Reference.html](GridDB_C_API_Reference.html)).

### Treatment of the data type not supported by SQL

The data types which are supported by the NoSQL interface, but not by the NewSQL interface are as follows.

- GEOMETRY
- ARRAY

This section explains how to handle the data of these data types when accessed using the NewSQL interface.

- Creating a table using CREATE TABLE
  - These data types cannot be specified as a data type of the column when creating a table. An error occurs.

- Deleting a table using DROP TABLE
  - The table, which has any columns of these data types, can be deleted.

- Registration/updating/deleting using INSERT/UPDATE/DELETE
  - For a table with the column of these data types, INSERT/UPDATE/DELETE causes an error.
  - Rows can not be registered or updated even by specifying only the column values of the supported data types, without specifying any column values of these data types.

    ```
    // The table created using the NoSQL interface
      name: sample1
      Column: id INTEGER
              value DOUBLE
              geometry GEOMETRY

    // Register rows by specifying only INTEGER and DOUBLE columns. -> An error occurs because the table has a GEOMETRY type column.
    INSERT INTO sample1 (id, value) VALUES (1, 192.3)
    ```

- Searching using SELECT
  - Whenever a table with the column of these data types are searched, NULL returns from these columns.


- Creating/deleting an index using CREATE INDEX/DROP INDEX
  - Creating/deleting an index on a GEOMETRY type column is possible.
  - Creating/deleting an index on an array type column is not allowed. An error occurs. (In the NoSQL interface, creating/deleting an index on an array type column is not allowed.)

  

## User and database

There are 2 types of GridDB user, an administrator user and a general user, which differ in terms of the functions which can be used.  In addition, access can be separated on a user basis by creating a database.
See [GridDB Features Reference](GridDB_FeaturesReference.md) for the details of users and a database.



## Naming rules

The naming rules are as follows:

-   A database name, table name, view name, column name, index name and general user name is a string composed of one or more ASCII alphanumeric characters, the underscore "_" , the hyphen "-" , the dot "." , the slash "/" and the equal "=".
-   For table name, the "@" character can also be specified for the node affinity function.

See [GridDB Features Reference](GridDB_FeaturesReference.md) for the details about the node affinity function, and the rules and the restrictions of naming.


[Notice]
- If the name of a table or a column contains characters other than ASCII alphanumeric characters and underscore, or if the first character of the name is a number in a SQL statement, enclose the name with double quotation marks.

  ```
  SELECT "column.a1" FROM "Table-5"
  ```


<a id="sql_commands_supported_by_griddb"></a>
# SQL commands supported by GridDB

Supported SQL commands are in the table as follows.


| Command                             | Overview                                                |
|-------------------------------------|----------------------------------------------------|
| [CREATE DATABASE](#create-database) | Create a database.                             |
| [CREATE TABLE](#create-table)       | Create a table.                                 |
| [CREATE INDEX](#create-index)       | Create an index.                                     |
| [CREATE VIEW](#create-view)         | Create a view.                                   |
| [CREATE USER](#create-user)         | Create a general user.                               |
| [CREATE ROLE](#create-role)     |        Create a role.                               |
| [DROP DATABASE](#drop-database)     | Delete a database.                             |
| [DROP TABLE](#drop-table)           | Delete a table.                                 |
| [DROP INDEX](#drop-index)           | Delete an index.                                     |
| [DROP VIEW](#drop-view)             | Delete a view.                                   |
| [DROP USER](#drop-user)             | Delete a general user.                               |
| [DROP ROLE](#drop-role)       | Delete a role.                               |
| [ALTER TABLE](#alter-table)         | Change the structure of a table.                         |
| [GRANT](#grant)                     | Assign database access rights to a general user.     |
| [REVOKE](#revoke)                   | Revoke database access rights from a general user.   |
| [SET PASSWORD](#set-password)       | Change the password of a general user.                   |
| [SELECT](#select)                   | Select data.                                   |
| [INSERT](#insert)                   | Insert rows into a table.                             |
| [DELETE](#delete)                   | Delete rows from a table.                           |
| [UPDATE](#update)                   | Update rows in a table.                         |
| [Comment](#comment)                 | Add a comment.                                 |
| [Hints](#hint)                      | Control an execution plan.                                 |

An explanation for each category of SQL command is given in this chapter.


## Data definition language (DDL)

### CREATE DATABASE

Create a database.

**Syntax**

|                                   |
|-----------------------------------|
| CREATE DATABASE database_name; |

**Specifications**

-   This command can only be executed by the administrative user.
-   Databases with the same name as "public", "information_schema" cannot be created as these are reserved for internal use in GridDB.
-   Nothing will be changed if a database with the same name already exists.
-   See [GridDB Features Reference](GridDB_FeaturesReference.md) for the rules of a database name.

### CREATE TABLE

#### Creating a table

Create a table.

**Syntax**

- Table (collection)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] table name (column definition \[, column definition ...\] \[, PRIMARY KEY (column name \[, ...\])\])<br>\[WITH (property key = property value)\]; |

- Timeseries table (timeseries container)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] table_name ( column_name TIMESTAMP PRIMARY KEY \[, column definition ...\] )<br>USING TIMESERIES \[WITH (property_key=property_value \[, property_key=property_value ...\])\]; |

- **column definition**
  - column_name data_type \[ column_constraint \]

- **column_constraint**
  - PRIMARY KEY (only the 1st column can be specified)
  - NULL
  - NOT NULL

**Specifications**

- See [GridDB Features Reference](GridDB_FeaturesReference.md) for the rule of a table name and a column name.
- If "IF NOT EXISTS" is specified, the specified table can be created only if another table with the same name does not exist.
- The column name and data type name need to be specified in column definition. See [Data types used in data storage](#data_types_used_in_data_storage) for the data types that can be specified.
- Composite primary key can be set to a table (collection) by setting the primary key after describing the column definition. The composite primary key must be set to the columns which are continuous from the first column and can be set up to 16 columns. It cannot be set together with the PRIMARY KEY as a column constraint, cannot be set to a time series table (time series container).
- See [GridDB Features Reference](GridDB_FeaturesReference.md) for details of time series table (Time series container).
- Options related to data affinity can be specified in the format "WITH (property key = property value, ...)".


<a id="label_data_affinity_property"></a>

  | Function | Item | Property key | Property value type                             |
  |---------------------|------------|--------------------------|----------------------------------------------|
  | Data affinity | hint<br>(Character string indicating similarity between containers) | data_affinity | STRING                                     |

- See [GridDB Features Reference](GridDB_FeaturesReference.md) for the details of each item.

**Examples**

- Creating a table

  ``` example
  CREATE TABLE myTable (
    key INTEGER PRIMARY KEY,
    value1 DOUBLE NOT NULL,
    value2 DOUBLE NOT NULL
  );
  ```

#### Creating a partitioned table

Creating a partitioned table

See [GridDB Features Reference](GridDB_FeaturesReference.md) for details of each partitioning function.

**(1) Creating a hash partitioned table**

**Syntax**

- Table (collection)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] table_name ( column definition \[, column definition ...\] \[, PRIMARY KEY(column name \[, ...\])\] )<br>\[WITH (property_key=property_value)\]<br>PARTITION BY HASH (column_name_of_partitioning_key) PARTITIONS division_count; |

- Timeseries table (timeseries container)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] table_name ( column definition \[, column definition ...\] )<br>USING TIMESERIES \[WITH (property_key=property_value, ...)\]\]<br>PARTITION BY HASH (column_name_of_partitioning_key) PARTITIONS division_count ; |

**Specifications**

- Create a hash partitioned table usng the column name of the partitioning key and the value of division count.
- Specify the value from 1 to 1024 for "division_count".
- The partitioning key requires the primary key. To set a key other than the primary key, the restriction in the configuration file need to be removed. For details, refer to the cluster definition file settings in [GridDB Features Reference](GridDB_FeaturesReference.md).
- The column specified as partitioning key cannot be updated.

***Option specifications***

****Data affinity****
- Options related to data affinity can be specified in the format "WITH (property key = property value, ...)". The options that can be specified are same as [normal table](#label_data_affinity_property).

**Examples**

- Creating a hash partitioned table

  ``` example
  CREATE TABLE myHashPartition (
    id INTEGER PRIMARY KEY,
    value STRING
  ) PARTITION BY HASH (id) PARTITIONS 128;
  ```

**(2) Creating an interval partitioned table**

**Syntax**

- Table (collection)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] table_name ( column definition \[, column definition ...\] \[, PRIMARY KEY(column name \[, ...\])\])<br>\[WITH (property_key=property_value, ...)\]<br>PARTITION BY RANGE(column_name_of_partitioning_key) EVERY(interval_value \[, interval_unit \]); |

- Timeseries table (timeseries container)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] table_name ( column definition \[, column definition ...\] )<br>USING TIMESERIES \[WITH (property_key=property_value, ...)\]<br>PARTITION BY RANGE(column_name_of_partitioning_key) EVERY(interval_value \[, interval_unit \]) ; |

**Specifications**

- Specify the column which type is BYTE, SHORT, INTEGER, LONG or TIMESTAMP for "column_name_of_partitioning_key".
- The partitioning key requires the primary key. To set a key other than the primary key, the restriction in the configuration file need to be removed. For details, refer to the cluster definition file settings in [GridDB Features Reference](GridDB_FeaturesReference.md).
- The column specified as partitioning key cannot be updated.
- The following values can be specified as the "interval_value".

  | Partitioning key type | Possible interval value    |
  |----------------------------|---------------------------|
  | BYTE                  | from 1 to 2<sup>7</sup>-1                    |
  | SHORT                 | from 1 to 2<sup>15</sup>-1                  |
  | INTEGER               | from 1 to 2<sup>31</sup>-1             |
  | LONG                  | from 1000 to 2<sup>63</sup>-1 |
  | TIMESTAMP             | 1 or more                     |

- If the column of TIMESTAMP (including TIMESTAMP with specified precision) is specified, the interval unit should also be specified. DAY and HOUR are the values that can be specified as the interval unit.
- The interval unit cannot be specified for columns of any types other than above.

[Memo]

- When a new interval is created, a data partition (container) that corresponds to that particular interval will also be generated. The partitioning feature has the benefit of properly reducing the data size that one data partition manages and leveraging parallelism, thereby improving performance. At the same time, however, it has also the disadvantage in that an increase in the number of data partitions incurs an overhead in merging them, which in turn leads to memory increase and performance degrade.
- By default, GridDB results in an error at the point when 10,000 data partitions are generated in a table.  The configuration file value for the upper limit on the number of data partitions to be generated can be modified. For details, see the [GridDB Features Reference](GridDB_FeaturesReference.md).

***Option specifications***

****Data affinity****
- Options related to data affinity can be specified in the format "WITH (property key = property value, ...)".
  The options that can be specified are same as [normal table](#label_data_affinity_property).

****Expiry release****
- The options about expiry release can be specified by the format " WITH (property_key=property_value, ...)".

  | Function | Item | Property key | Property value type | Required or optional when setting expiry release |
  |--------------|----------|-------------------------|---------------------------------------------------|-------------------------|
  | Expiry release function | Type            | expiration_type | STRING<br>(Only the following type can be specified. <br>PARTITION: Partition expiry release) | Optional (default: PARTITION) |
  | | Expiration time | expiration_time | INTEGER              | Required |
  | | Expiration time unit | expiration_time_unit | STRING <br>(The following five types can be specified. <br>DAY / HOUR / MINUTE / SECOND / MILLISECOND ) | Optional (default: DAY) |
  | | Expiration division count | expiration_division_count | INTEGER           | Optional (default: 8) |

- The partition expiry release can only be specified for followings:
  - Timeseries table (timeseries container)
  - Table (collection) whose partitioning key is TIMESTAMP (including TIMESTAMP with specified precision) .
- See [GridDB Features Reference](GridDB_FeaturesReference.md) for the details of each item.

[memo]
 - The row key for a timeseries container is fixed to millisecond-precision TIMESTAMP; TIMESTAMP with any other precision is not allowed.

***Data partition placement***
- The option to determine where to place data partitions corresponding to each date can be specified in the format "WITH (property key = property value, ...)".

  | Function                | Item                                         | Property key        | Property value type   | Required or optional when setting data partition placement |
  |---------------------|----------------------------------------------|---------------------|-------------------|------------|
  |   Interval group number  | The number that identifies where a data partition is placed. Two tables that are created by specifying different interval group numbers from each other are allocated in such a way that they do not cause conflicts among processing threads on the same day.| interval_worker_group            | INTEGER  | Required |
  | Interval group node correction value  | A value that corrects the node on which the processing threads that are determined by an interval group number are performed. This value can be specified by the user, but it is also possible to allow the server to determine it. | interval_worker_group_position | INTEGER  | Optional (zero by default) |

[Memo]
 - Data partition placement can be applied only when the partition key for interval partitioning is "TIMESTAMP".
 - There are several conditions, both functional and operational, that must be met before applying this feature. To learn more about these conditions, see the [GridDB Features Reference](GridDB_FeaturesReference.md).

**Examples**

- Creating an interval partitioned table.

  ``` example
  CREATE TABLE myIntervalPartition (
    date TIMESTAMP PRIMARY KEY,
    value STRING
  ) PARTITION BY RANGE (date) EVERY (30, DAY);
  ```

- Creating an interval partitioned table (timeseries table) using the partition expiry release function.

  ``` example
  CREATE TABLE myIntervalPartition2 (
    date TIMESTAMP PRIMARY KEY,
    value STRING
  ) USING TIMESERIES WITH (
    expiration_type='PARTITION',
    expiration_time=90,
    expiration_time_unit='DAY'
  ) PARTITION BY RANGE (date) EVERY (30, DAY);
  ```

- Creating an interval partitioned table by specifying where to place the data partition.

  ``` example
  CREATE TABLE myIntervalPartition2 (
    date TIMESTAMP PRIMARY KEY,
    value STRING
  ) WITH (
    interval_worker_group=1
  ) PARTITION BY RANGE (date) EVERY (1, DAY);
  ```

**(3) Creating an interval hash partitioned table**

**Syntax**

- Table (collection)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] table_name ( column definition \[, column definition ...\] \[, PRIMARY KEY((column_name \[, ...\])\] )<br>\[WITH (property_key=property_value, ...) \]<br>PARTITION BY RANGE(column_name_of_interval_partitioning_key) EVERY(interval_value \[, interval_unit \])<br>SUBPARTITION BY HASH(column_name_of_hash_partitioning_key) SUBPARTITIONS division_count; |

- Timeseries table (timeseries container)

  |                                   |
  |-----------------------------------|
  | CREATE TABLE \[IF NOT EXISTS\] table_name ( column definition \[, column definition ...\] ) USING TIMESERIES \[WITH (property_key=property_value, ...)\] PARTITION BY RANGE(column_name_of_partitioning_key) EVERY(interval_value \[, interval_unit \]) SUBPARTITION BY HASH(column_name_of_hashpartitioning_key) SUBPARTITIONS division_count ; |

**Specifications**

- Specify the column which type is BYTE, SHORT, INTEGER, LONG or TIMESTAMP for "column_name_of_interval_partitioning_key".
- The following values can be specified as the "interval_value".

  | Partitioning key type | Possible interval value                     |
  |----------------------------|--------------------------------------------|
  | BYTE                  | from 1 to 2<sup>7</sup>-1                                     |
  | SHORT                 | from 1 to 2<sup>15</sup>-1                                   |
  | INTEGER               | from 1 to 2<sup>31</sup>-1                              |
  | LONG                  | from 1000 \* division_count to -2<sup>63</sup>-1 |
  | TIMESTAMP             | 1 or more                                      |

  - If the column of TIMESTAMP is specified, it is also required to specify the interval unit.If the column of TIMESTAMP is specified, it is also required to specify the interval unit. DAY and HOUR are the values that can be specified as the interval unit.
  - The interval unit cannot be specified for any types other than TIMESTAMP.

- Specify the value from 1 to 1024 for "division_count".
- The partitioning key requires the primary key. To set a key other than the primary key, the restriction in the configuration file need to be removed. For details, refer to the cluster definition file settings in [GridDB Features Reference](GridDB_FeaturesReference.md).
- The column specified as partitioning key cannot be updated.

[Memo]
- Like interval partitions, the number of data partitions of one table that are generated has an upper limit. To change the number, see the section on the "setting of the cluster definition file" in the [GridDB Features Reference](GridDB_FeaturesReference.md).
- Note that unlike interval partitions, for interval hash partitions, the same number of data partitions are generated for a single interval as the number of hash partitions.

***Option specifications***

****Data affinity****
- Options related to data affinity can be specified in the format "WITH (property key = property value, ...)".
  The options that can be specified are same as [normal table](#label_data_affinity_property).

****Expiry release****
- The options about expiry release can be specified by the format " WITH (property_key=property_value, ...)".


  | Function | Item | Property key | Property value type | Required or optional when setting expiry release |
  |--------------|----------|-------------------------|---------------------------------------------------|-------------------------|
  | Expiry release function | Type | expiration_type | STRING<br>(Any of the followings. If omitted, PARTITION. <br>PARTITION: Partition expiry release<br>ROW: Row expiry release) | Optional |
  | | Expiration time | expiration_time | INTEGER              | Required |
  | | Expiration time unit | expiration_time_unit | STRING <br>(The following five types can be specified. <br>DAY / HOUR / MINUTE / SECOND / MILLISECOND ) | Optional (default: DAY) |
  | | Expiration division count | expiration_division_count | INTEGER           | Optional (default: 8) |


**Examples**

- Creating an interval-hash partitioned table

  ``` example
  CREATE TABLE myIntervalHashPartition (
    date TIMESTAMP,
    value STRING,
    PRIMARY KEY (date, value)
  ) PARTITION BY RANGE (date) EVERY (60, DAY)
  SUBPARTITION BY HASH (value) SUBPARTITIONS 64;
  ```

- Creating an interval-hash partitioned table (timeseries table) using the partition expiry release function.

  ``` example
  CREATE TABLE myIntervalHashPartition2 (
    date TIMESTAMP PRIMARY KEY,
    value STRING
  ) USING TIMESERIES WITH (
    expiration_type='PARTITION',
    expiration_time=90,
    expiration_time_unit='DAY'
  ) PARTITION BY RANGE (date) EVERY (60, DAY)
  SUBPARTITION BY HASH (date) SUBPARTITIONS 64;
  ```

### CREATE INDEX

Create an index.

**Syntax**

|                                   |
|-----------------------------------|
| CREATE INDEX \[IF NOT EXISTS\] index_name ON table_name ( column_name_to_be_indexed ); |

**Specifications**

- See [GridDB Features Reference](GridDB_FeaturesReference.md) for the rules of an index name.
- For a table, an index with the same name as an existing index in the table cannot be created.
- If a transaction under execution exists in a table subject to processing, the system will wait for these to be completed before creating the data.
- An index cannot be created on a column of BLOB type and ARRAY type.
- An index with multiple columns can be created. This is called a composite index.
- The maximum number of columns that can be specified in one composite index is 16, and the same column cannot be specified more than one time.
- Time series table does not allow a primary key in a composite index.

### CREATE VIEW

Create a view.

**Syntax**

|                                   |
|-----------------------------------|
| CREATE \[FORCE\] VIEW view_name AS SELECT statement; |

**Specifications**

- See [GridDB Features Reference](GridDB_FeaturesReference.md) for the rules of a view name.
- Whether the result from the SELECT statement is available or not is checked. If the result is not available, a view cannot be created.
- When FORCE is specified, the result from the SELECT statement is not checked, while a syntactic check is done.
- The SELECT statement can include other view names. If other view names in the SELECT statement cause circular reference, the view cannot be created even if FORCE is specified.

### CREATE USER

Create a general user.

**Syntax**

|                                   |
|-----------------------------------|
| CREATE USER user_name IDENTIFIED BY 'password_string' ; |

**Specifications**

- See [GridDB Features Reference](GridDB_FeaturesReference.md) for the rules of a user name.
- Can be executed by an administrator user only.
- A user with the same name as an administrator user (admin and system) registered during installation cannot be created.
- The password must consist of ASCII characters. It is case-sensitive.

### CREATE ROLE

Create a role required for LDAP authentication.

**Syntax**

|                                   |
|-----------------------------------|
| CREATE ROLE *role name* ; |

**Specifications**

- For the argument role name, specify either an LDAP user name or an LDAP group name. For details, see the [GridDB Features Reference](GridDB_FeaturesReference.md).
- This command can only be executed by the administrative user.
- A user with the same name as an administrator user (admin and system) registered during installation cannot be created.
- The password must consist of ASCII characters. It is case-sensitive.


### DROP DATABASE

Delete a database.

**Syntax**

|                                   |
|-----------------------------------|
| DROP DATABASE database_name;   |

**Specifications**

- This command can only be executed by the administrative user.
- The database with the following names are reserved for internal use and thus cannot be deleted: "public", "information_schema", and any names starting with "gs\#".
- A database containing tables created by a user cannot be deleted.

### DROP TABLE

Delete a table.

**Syntax**

|                                   |
|-----------------------------------|
| DROP TABLE \[IF EXISTS\] table_name; |

**Specifications**

- If "IF EXISTS" is specified, nothing will change if no table with the specified name exists.
- If there is an active transaction involving the table, the table will be deleted only after the transaction is completed.

### DROP INDEX

Delete the specified index.

**Syntax**

|                                   |
|-----------------------------------|
| DROP INDEX \[IF EXISTS\] index_name ON table_name; |

**Specifications**

- If "IF EXISTS" is specified, nothing will change if no index with the specified name exists.
- If there is an active transaction involving the table, the table will be deleted only after the transaction is completed.
- The unnamed index creating through NoSQL I/F can not be deleted by "DROP INDEX".

### DROP VIEW

Delete a view.

**Syntax**

|                                   |
|-----------------------------------|
| DROP VIEW \[IF EXISTS\] view name ; |

**Specifications**

- If "IF EXISTS" is specified, nothing will be changed if a view with the specified name does not exist.

### DROP USER

Delete a general user.

**Syntax**

|                                   |
|-----------------------------------|
| DROP USER user_name;             |

**Specifications**

- This command can only be executed by the administrative user.

### DROP ROLE

Delete a role required for LDAP authentication.

**Syntax**

|                                   |
|-----------------------------------|
| DROP ROLE *role name* ; |

**Specifications**

- This command can only be executed by the administrative user.



### ALTER TABLE

Change the structure of a table.

#### Adding columns to a table

Add columns to the end of the table.

**Syntax**

|                                   |
|-----------------------------------|
| ALTER TABLE table_name ADD \[COLUMN\] column definition \[,ADD \[COLUMN\] column definition ...\]; |

- **column definition**
  - column_name data_type \[ column_constraint \]

- **column_constraint**
  - NULL
  - NOT NULL

**Specifications**

- The added column is located in the end of the table. If multiple columns are specified, they are located in their order.
- PRIMARY KEY can not be specified to the column constraint.
- If the same name column exists, an error occurs.

**Examples**

- Adding multiple columns to the table

  ``` example
  ALTER TABLE myTable1
  ADD COLUMN col111 STRING NOT NULL,
  ADD COLUMN col112 INTEGER;
  ```

#### Deleting data partitions

Delete data partitions created by table partitioning.

**Syntax**

|                                   |
|-----------------------------------|
| ALTER TABLE table_name DROP PARTITION FOR ( value_included_in_the_data_partition ); |

**Specifications**

- Data partitions can be deleted only for interval and interval-hash partitioning.
- Specify the value included in the data partition to be deleted.
- Data in the range of the once deleted data partition (from the lower limit value to the upper limit value of the data partition) cannot be registered.
- The lower limit value of a data partition can be checked by metatable. In many cases, the upper limit of a data partition is the lower limit value plus division width value.
- For interval-hash partitioned tables, there are multiple data partitions which have the same lower limit value, and the maximum number of those partitions is equal to the hash division count.
  Those data partitions are deleted simultaneously. Deleted partitions are checked by metatable.

See [Metatables](#label_metatables) for the details on the metatable.

**Examples**

**Interval partitioned table**

- Check the lower limit value of the interval partitioned table "myIntervalPartition1" (partitioning key type: TIMESTAMP, interval: 30 DAY)

  ``` example
  SELECT PARTITION_BOUNDARY_VALUE FROM "#table_partitions"
  WHERE TABLE_NAME='myIntervalPartition1' ORDER BY PARTITION_BOUNDARY_VALUE;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
   2017-01-10T13:00:00.000Z
   2017-02-09T13:00:00.000Z
   2017-03-11T13:00:00.000Z
         :
  ```

- Delete unnecessary data partitions

  ``` example
  ALTER TABLE myIntervalPartition1 DROP PARTITION FOR ('2017-01-10T13:00:00Z');
  ```

**Interval hash partitioned table**

- Check the lower limit value of each data partitions on the interval hash partitioned table "myIntervalHashPartition" (partitioning key type: TIMESTAMP, interval value: 90 DAY, division count 3)

  ``` example
  SELECT PARTITION_BOUNDARY_VALUE FROM "#table_partitions"
  WHERE TABLE_NAME='myIntervalHashPartition' ORDER BY PARTITION_BOUNDARY_VALUE;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
  2016-08-01T10:00:00.000Z    The data of the same lower limit is hashed and
  2016-08-01T10:00:00.000Z    is divided into three data partitions.
  2016-08-01T10:00:00.000Z
  2016-10-30T10:00:00.000Z
  2016-10-30T10:00:00.000Z
  2016-10-30T10:00:00.000Z
  2017-01-29T10:00:00.000Z
         :
  ```

- Delete unnecessary data partitions

  ``` example
  ALTER TABLE myIntervalHashPartition DROP PARTITION FOR ('2016-09-15T10:00:00Z');
  ```

- Data partitions that have same boundary value will be deleted

  ``` example
  SELECT PARTITION_BOUNDARY_VALUE FROM "#table_partitions"
  WHERE TABLE_NAME='myIntervalHashPartition' ORDER BY PARTITION_BOUNDARY_VALUE;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
  2016-10-30T10:00:00.000Z    For the section (lower limit '2016-08-01T10: 00: 00Z') including '2016-09-15T10: 00: 00Z'
  2016-10-30T10:00:00.000Z    three data partitions are deleted.
  2016-10-30T10:00:00.000Z
  2017-01-29T10:00:00.000Z
         :
  ```

#### RENAME COLUMN

Change an existing specified column.

**Syntax**

|                                   |
|-----------------------------------|
| ALTER TABLE *table name* RENAME COLUMN *column name before renaming* TO *column name after renaming*;|


**Specifications**

- An error will occur if the column name before renaming is not in the specified table.
- An error will occur if the column name after renaming is already in the specified table.

**Examples**

- Renaming a column

  ``` example
  ALTER TABLE myTable1 RENAME COLUMN col112 TO col121;
  ```

  

## Data control language (DCL)

### GRANT

Assign database access rights to a general user or a role.

**Syntax**

|                                   |
|-----------------------------------|
| GRANT {SELECT\|ALL} ON database_name TO {user_name\|role name}; |

**Specifications**

- This command can only be executed by the administrative user.
- SELECT indicates reference authority and ALL indicates reference authority and update authority.

### REVOKE

Revoke database access rights from a general user or a role.

**Syntax**

|                                   |
|-----------------------------------|
| REVOKE {SELECT\|ALL} ON database_name FROM {user_name\|role name}; |

**Specifications**

- This command can only be executed by the administrative user.
- SELECT indicates reference authority and ALL indicates reference authority and update authority.

### SET PASSWORD

Change the password of a general user.

**Syntax**

|                                   |
|-----------------------------------|
| SET PASSWORD \[FOR user_name \] = 'password_string'; |

**Specifications**

- An administrator user can change the passwords of all general users.
- A general user can change its own password only.

    

## Data management language (DML)

### SELECT

Select data. Made up of a variety of [Clauses](#label_clauses) such as FROM, WHERE, etc.

**Syntax**

|                                   |
|-----------------------------------|
| SELECT \[{ALL\|DISTINCT}\] * \| column_name_1 \[, column_name_2 ...\]<br>\[FROM clause\]<br>\[WHERE clause\]<br>\[GROUP BY clause \[HAVING clause\]\]<br>\[{UNION \[ALL\] \|INTERSECT\|EXCEPT} SELECT statement\]<br>\[ORDER BY clause\]<br>\[LIMIT clause \[OFFSET clause\]\] ; |


### INSERT

Register rows in a table. INSERT only registers rows, while INSERT OR REPLACE and REPLACE overwrite the existing data, when the data with the same primary key as that of the existing data is given. REPLACE is an alias of INSERT OR REPLACE and they are the same in their functions.

**Syntax**

|                                   |
|-----------------------------------|
| {INSERT\|INSERT OR REPLACE\|REPLACE} INTO table_name<br>{VALUES ( { number_1 \| string_<em>1</em> } \[, { number_2 \| string_<em>2</em> } ...\] )\|SELECT statement} ; |

**Specifications**

- If a SELECT statement is specified instead of VALUES, the execution result will be registered. Except that the inquiry result including UNION/INTERSECT/EXCEPT cannot be registered.

``` example
INSERT INTO myTable1 VALUES(1, 100);

REPLACE INTO myTable1 VALUES(1, 200);

INSERT INTO myTable1 SELECT * FROM myTable2;
```

### DELETE

Delete rows from a table.

**Syntax**

|                                   |
|-----------------------------------|
| DELETE FROM table_name \[ WHERE clause \]; |

### UPDATE

Update the rows existing in a table.

**Syntax**

|                                   |
|-----------------------------------|
| UPDATE table_name SET column_name_1 = expression_1 \[, column_name_2 = expression_2 ...\] \[ WHERE clause \]; |

**Specifications**

- The value of the PRIMARY KEY column can not be updated.
- For a partitioned table, a column , set as a partitioning key, can not be updated to a different value using the UPDATE statement. In such a case, INSERT after DELETE.
  - Example:
    ``` example
    CREATE TABLE tab (a INTEGER, b STRING) PARTITION BY HASH a PARTITIONS 5;

    -- NG
    UPDATE tab SET a = a * 2;
    [240016:SQL_COMPILE_PARTITIONING_KEY_NOT_UPDATABLE] Partitioning column='a' is not updatable

    -- OK
    UPDATE tab SET b = 'XXX';
    ```

- A column name specified with SET cannot be qualified with a table name.
  - Example:
    ``` example
    CREATE TABLE myTable1 (key INTEGER, value INTEGER);

    -- NG
    UPDATE myTable1 SET myTable1.value = 999 WHERE myTable1.key = 8;

    -- OK
    UPDATE myTable1 SET value = 999 WHERE myTable1.key = 8;
    ```

- Subqueries cannot be used for update values, while it can be used for conditional statements such as WHERE.
  - Example:
    ``` example
    CREATE TABLE myTable1 (key INTEGER, value INTEGER);
    
    -- NG
    UPDATE myTable1 SET value = (SELECT 999) WHERE key = 8;

    -- OK
    UPDATE myTable1 SET value = 999 WHERE key = (SELECT 8);
    ```



<a id="label_clauses"></a>
## Clauses

### FROM

Specify the table name, view name, and subquery on which to execute data operations.

**Syntax**

|                                   |
|-----------------------------------|
| FROM table_name_1 \[, table_name_2 ... \] |
| FROM (sub_query) \[AS\] Alias \[, ...\] |

**Specifications**

- A sub query must be enclosed with () and requires an alias.

Example:
``` example
SELECT a.ID, b.ID FROM mytable a, (SELECT ID FROM mytable2) b;

ID     ID
---+-----
 1    100
 1    200
 2    100
 2    200
   :
```

### GROUP BY

Among the results of the clauses specified earlier, rows having the same value in the specified column will be grouped together.

**Syntax**

|                                   |
|-----------------------------------|
| GROUP BY column_name_1 \[, column_name_2 ...\] |

### HAVING

Perform filtering using the search condition on data grouped by the GROUP BY clause. GROUP BY clause cannot be omitted.

**Syntax**

|                                   |
|-----------------------------------|
| HAVING search_conditions                    |

### ORDER BY

Sort search results.

|                                   |
|-----------------------------------|
| ORDER BY column_name_1 \[{ASC\|DESC}\] \[, column_name_2 \[{ASC\|DESC}\] ...\] |

### WHERE

Apply a search condition on the result of the preceding FROM clause.

**Syntax**

|                                   |
|-----------------------------------|
| WHERE search_conditions                     |

**Specifications**

- Search conditions can be described using an expression, a function, a subquery, etc.

### LIMIT/OFFSET

Extract the specified number of data from the specified location.

**Syntax**

|                                   |
|-----------------------------------|
| LIMIT value_1 \[OFFSET value_2 \]      |

**Specifications**

- Value_1 represents the number of data to extract while value_2 represents the position of the data to extract.

  

### JOIN

Join a table.

**Syntax**

| Type of join    | Syntax                 |
|-------------|-----------------------|
| Inner join      | Table 1 \[INNER\] JOIN table 2 \[ON condition \| USING(Column name \[, column name ...\])\]  |
| Left outer join | Table 1 LEFT \[OUTER\] JOIN table 2 \[ON type \| USING (column name \[, column name ...\])\]   |
| Cross join      | Table 1 CROSS JOIN table 2 \[ON condition \| USING (column name \[, column name ...\])\] |

- Inner join returns records that have matching values in both tables in the specified row.
- Left outer join returns records that have matching values in both tables in the specified row, as well as the records only exist in the table 1.
- A cross join is equivalent to an inner join (INNER JOIN).

Specify join conditions with ON or USING.

Example:
```example
name: employees

 id   first_name   department_id
----+------------+----------------
  0   John         0
  1   William      1
  2   Richard      0
  3   Mary         4
  4   Lisa         3
  5   James        1

name: departments

 department_id   department   
---------------+------------
  0              Sales
  1              Development
  2              Research
  3              Marketing

○Inner join
SELECT * FROM employees e INNER JOIN departments d ON e.department_id=d.department_id;

 id    first_name  department_id  department_id  department
------+-----------+--------------+--------------+-----------
  0    John         0              0             Sales
  1    William      1              1             Development
  2    Richard      0              0             Sales
  4    Lisa         3              3             Marketing
  5    James        1              1             Development


○Left outer join
SELECT * FROM employees e LEFT JOIN departments d  ON e.department_id=d.department_id;

 id    first_name  department_id  department_id  department
------+-----------+--------------+--------------+-----------
  0    John         0              0             Sales
  1    William      1              1             Development
  2    Richard      0              0             Sales
  3    Mary         4              (NULL)        (NULL)
  4    Lisa         3              3             Marketing
  5    James        1              1             Development
```

  

Natural join (NATURAL JOIN) joins tables that have matching values in the rows under the same name.

| Type of join    | Syntax                                              |
|-------------|----------------------------------------------------|
| Inner join      | Table 1 NATURAL \[INNER\] JOIN table 2        |
| Left outer join | Table 1 NATURAL LEFT \[OUTER\] JOIN table 2   |
| Cross join      | Table 1 NATURAL CROSS JOIN table 2          |

``` example
SELECT * FROM employees NATURAL INNER JOIN departments;

 department_id   id    first_name     department
---------------+-----+--------------+--------------
  0              0     John           Sales
  1              1     William        Development
  0              2     Richard        Sales
  3              4     Lisa           Marketing
  1              5     James          Development
```

### UNION/INTERSECT/EXCEPT

Calculate on a set of two query results.

**Syntax**

|                              |                                                   |
|------------------------------|---------------------------------------------------|
| Inquiry 1 UNION inquiry 2 | Returns all the results of two queries. (duplication is not included)  |
| Query 1 UNION ALL query 2 | Returns all the results of two queries. (duplication is included)      |
| Query 1 INTERSECT query 2 | Returns the results common to the results of two queries.                    |
| Query 1 EXCEPT query 2    | Returns the difference of two queries (result included in the query 1, not in the query 2). |


### OVER

Split and sort query results. Use with a WINDOW function.

**Syntax**

|                                   |
|-----------------------------------|
| Function OVER (\[PARTITION BY expression 1\] \[ORDER BY expression 2\])     |

**Specifications**


- Can be used in the SELECT clause.
- The supported functions are as follows:
  - ROW_NUMBER()
  - LAG()
  - LEAD()
  - AVG()
  - COUNT()
  - MAX()
  - MIN()
  - SUM()/TOTAL()
  - STDDEV_SAMP()
  - STDDEV()/STDDEV0()
  - STDDEV_POP()
  - VAR_SAMP()
  - VARIANCE()/VARIANCE0()
  - VAR_POP()
- DISTINCT cannot be specified as the first argument of the function.
- Split the query result with PARTITION BY clause. Sort rows by ORDER BY clause.
- Multiple use of the WINDOW function/OVER clause in the same SELECT clause, and simultaneous use of the WINDOW function/OVER clause and the MEDIAN function are not allowed.
- The following expressions cannot be specified in the PARTITION BY clause.
  - Expression containing OVER clause
  - Expression containing aggregate function
  - Expression containing column aliases
  - Subquery
- The following expressions cannot be specified in the ORDER BY clause.
  - Expression containing OVER clause
  - Expression containing aggregate function
  - Expression containing column aliases
  - Subquery


### GROUP BY RANGE

Create result collections in which results are split for each given time span and perform aggregation operations. If interpolation operation is specified, value interpolation is performed for those collections where values are not included in results.

**Syntax**

|                                   |
|-----------------------------------|
| GROUP BY RANGE(*date column name*) EVERY( *time interval*, *unit* \[ ,*offset* \] ) \[ FILL(*interpolation method*) \]    |

**Specifications**

SELECT statements containing the GROUP BY RANGE clause have the following constraints:

- For a *list of expressions* in a SELECT statement, column expressions and functions other than analytic functions can be specified, but not analytic functions and subqueries.
- For the search conditions specified in the WHERE clause, the conditions on the date range regarding *date column names* must be specified. More specifically, one of the following conditions is required. In either case, comparison with a constant value must be made.
  + range conditions using the conditions for comparison with *date column names*
  + range conditions using the BETWEEN expression with *date column names*

The GROUP BY RANGE clause has the following specifications:

- The start point time of result collections performing aggregation and interpolation is calculated as a value of the column with *date column name*. The only column that can be specified here is a TIMESTAMP column, including a column of the TIMESTAMP type with specified precision.
- The start point time of result collections is determined based on the "first time" and the "time interval". More specifically, the date and time equivalent to N times the sum of the "first time" and the "time interval" is used for the start point time for each collection result.
- The "first time" and the "time interval" are set as follows:
  - The "first time" is specified using the *offset*. The following two methods are provided to explicitly specify the *offset*:  
    * Specifying an integer value (constant value): Specify an integer value (LONG). To specify the unit of the value, use the *unit*.
    * Specifying the timezone using strings: Specify the timezone in Z|±HH:MM|±HHMM format.
  - If the *offset* is unspecified, the offset of the timezone determined at the time of connecting to databases is used as an *offset* value. For how to determine the timezone at the time of connecting to databases, see the [GridDB JDBC Driver User Guide](GridDB_JDBC_Driver_UserGuide.md) and other relevant documents.
  - The "time interval" that is used when calculating the start point time of a result collection is specified using the *time interval* and the *unit* as follows:
    + For the *time interval*, specify a positive integer value. Only a constant value is allowed.
    + For the *unit*, one of the following can be specified.
      * DAY | HOUR | MINUTE | SECOND | MILLISECOND
- If the obtained result collection contains no values in the assumed time interval, the missing values can be obtained by using interpolation as specified by the "interpolation method" below.
  + For the *interpolation method*, one of the following can specified.
    * LINEAR: Missing values are replaced by a linear interpolation using a column value at the immediately preceding and following time to be output as the resulting values. 
      + For columns of the type in which numeric operations are not possible, the column value at the immediately preceding time is output as PREVIOUS below.
    * NONE: The entire row is not output.
    * NULL: The column value is set to NULL.
    * PREVIOUS: The column value at the immediately preceding time is output.

Example
```example
name: trend_data1

  ts                     value
-----------------------+-------
  2023-01-01T00:00:00       10
  2023-01-01T00:00:10       30
  2023-01-01T00:00:20       30
  2023-01-01T00:00:30       50
  2023-01-01T00:00:40       50
  2023-01-01T00:00:50       70


aggregation operation
SELECT ts,avg(value) FROM trend_data1
  WHERE ts BETWEEN TIMESTAMP('2023-01-01T00:00:00Z') AND TIMESTAMP('2023-01-01T00:01:00Z')
  GROUP BY RANGE ts EVERY (20,SECOND)

  ts                     value
-----------------------+-------
  2023-01-01T00:00:00       20
  2023-01-01T00:00:20       40
  2023-01-01T00:00:40       60
```

```example
name: trend_data2

  ts                     value
-----------------------+-------
  2023-01-01T00:00:00        5
  2023-01-01T00:00:10       10
  2023-01-01T00:00:20       15
  (time the data is missing)
  2023-01-01T00:00:40       25
  

  interpolation operation
SELECT * FROM trend_data2
  WHERE ts BETWEEN TIMESTAMP('2023-01-01T00:00:00Z') AND TIMESTAMP('2023-01-01T00:01:00Z')
  GROUP BY RANGE ts EVERY (10,SECOND) FILL (LINEAR)

  ts                     value
-----------------------+-------
  2023-01-01T00:00:00        5
  2023-01-01T00:00:10       10
  2023-01-01T00:00:20       15
  2023-01-01T00:00:30       20
  2023-01-01T00:00:40       25

```


## Operator

This section explains the operators used in SQL statements.

### List of Operators

The list of operators is as follows.

| Class | Operator | Description |
|------|-------|------|
| Arithmetic | + | Add  |
|       | - | Subtract  |
|       | * | Multiply  |
|       | / | Divide  |
|       | % | Modulo |
| Character | \|\| | Connect the value of arbitrary types as a character string. <br>If any one of the values is NULL, NULL is returned. |
| Compare | =, == | Compare whether both sides are equal. |
|       | !=, \<\> | Compare whether both sides are not equal. |
|       | \> | Compare whether the left side is larger than the right side. |
|       | \>= | Compare whether the left side is larger than or equal to the right one. |
|       | \< | Compare whether the left side is smaller than the right side. |
|       | \<= | Compare whether the left side is smaller than or equal to the right side. |
|       | IS | Compare whether both sides are equal. <br>Return true, when both sides are NULLs. <br>Return false, when either side is NULL. |
|       | IS NOT | Compare whether both sides are not equal. <br>Return false, when both sides are NULLs. <br>Return true, when either side is NULL. |
|       | ISNULL | Determine whether the left side is NULL. |
|       | NOTNULL | Determine whether the left side is not NULL. |
|       | [LIKE](#op_like) | Search the character string on the right. |
|       | [GLOB](#op_glob) | Search the character string on the right. |
|       | [BETWEEN](#op_between) | Extract values of the specified range.|
|       | [IN](#op_in) | Return whether the specified value is included in the set of values. |
| Bit | & | A & B : Bitwise AND of A and B |
|       | \| | A \| B : Bitwise OR of A and B |
|       | ~ | ~A : Bitwise NOT of A |
|       | \<\< | A \<\< B : Shift A to the left by B bit. |
|       | \>\> | A \>\> B : Shift A to the right by B bit. |
| Logic | AND | Return true, when both sides are true. <br>Return false, when either side is false. <br>Otherwise return NULL. |
|       | OR | Return true, when the expression on either side is true. <br>Return false, when the expressions on both sides are false. <br>Otherwise return NULL. |
|       | NOT | Return false, when the expression on the right is true. <br>Return true, when the expression on the right is false. <br>Otherwise return NULL. |

<a id="op_like"></a>
### LIKE

Search the character string on the right.

**Syntax**

|                                   |
|-----------------------------------|
| *str* \[NOT\] LIKE *pattern_str* \[ESCAPE *escape_str* \] |

**Specifications**

- See [LIKE function](#like-1).

<a id="op_glob"></a>
### GLOB

**Syntax**

Search the character string on the right.

|      |
|-------|
| *str* GLOB *pattern_str* |

**Specifications**

- See [GLOB function](#glob-1).

<a id="op_between"></a>
### BETWEEN

Extract values of the specified range.

**Syntax**

|                                   |
|-----------------------------------|
| expression_1 \[NOT\] BETWEEN expression_2 AND expression_3 |

**Specifications**

- Return true if the following conditions are met

  ``` example
  expression_2 <= expression_1 <= expression_3
  ```

- Return true if the following conditions are not met when NOT is specified.

<a id="op_in"></a>
### IN

Return whether the specified value is included in the set of values.

**Syntax**

|                                   |
|-----------------------------------|
| expression_1 \[NOT\] IN ( expression_2 \[, expression_3 ...\] ) |

**Specifications**

- Return true when the value of expression_1 is included in the result of expression_N.
- IN can be used in a [sub query](#in_sub_query).


<a id="label_function"></a>
## Functions

This section explains the functions used in SQL statements.

### List of Functions

The following functions are available for SQL statements.

| Class                       | Function name                          | Description |
|------|-----------------|------|
| [Aggregation](#aggregation) | [AVG](#avg) | Return the average value.    |
|      | [COUNT](#count)                        | Return the number of rows.     |
|      | [MAX](#Aggregate_MAX)                  | Return the maximum.     |
|      | [MIN](#Aggregate_MIN)                  | Return the minimum.     |
|      | [SUM](#sumtotal)                       | Return a sum of values.     |
|      | [TOTAL](#sumtotal)                     | Return a sum of values.     |
|      | [GROUP_CONCAT](#group_concat)         | Connect values.     |
|      | [STDDEV_SAMP](#stddev_samp)           | Return the sample standard deviation.     |
|      | [STDDEV](#stddevstddev0)               | Return the sample standard deviation.     |
|      | [STDDEV0](#stddevstddev0)              | Return the sample standard deviation.     |
|      | [STDDEV_POP](#stddev_pop)             | Return the population standard deviation.     |
|      | [VAR_SAMP](#var_samp)                 | Return the sample variance.     |
|      | [VARIANCE](#variancevariance0)         | Return the sample variance.     |
|      | [VARIANCE0](#variancevariance0)        | Return the sample variance.     |
|      | [VAR_POP](#var_pop)                   | Return the population variance.     |
|      | [MEDIAN](#MEDIAN)                      | Return the median.     |
|      | [PERCENTILE_CONT](#PERCENTILE_CONT)  | Return the percentile value.     |
| [Mathematical](#mathematical-functions) | [ABS](#abs)                            | Return an absolute value.     |
|      | [ROUND](#round)                        | Round off.     |
|      | [RANDOM](#random)                      | Return a random number.     |
|      | [MAX](#Arithmetic_MAX)                 | Return the maximum.     |
|      | [MIN](#Arithmetic_MIN)                 | Return the minimum.     |
|      | [LOG](#Arithmetic_LOG)                 | Return the logarithm.     |
|      | [SQRT](#Arithmetic_SQRT)               | Return the square root. |
|      | [TRUNC](#Arithmetic_TRUNC)             | Round down numbers. |
|      | [HEX_TO_DEC](#Arithmetic_HEX_TO_DEC) | Converts a hexadecimal string to a decimal number |
| [Character](#character-functions) | [LENGTH](#length)                      | Return the length of a character string.     |
|      | [LOWER](#lower)                        | Convert a character string to a lowercase.     |
|      | [UPPER](#upper)                        | Convert a character string to an uppercase.     |
|      | [SUBSTR](#substr)                      | Cut out part of a character string.     |
|      | [REPLACE](#replace)                    | Replace a character string.     |
|      | [INSTR](#instr)                        | Return the position of a specified character string in a character string.     |
|      | [LIKE](#like-1)                        | Search a character string.     |
|      | [GLOB](#glob-1)                        | Search a character string.     |
|      | [TRIM](#trim)                          | Remove a specified character(s) from the both ends of a character string.     |
|      | [LTRIM](#ltrim)                        | Remove a specified character(s) from the left end of a character string.     |
|      | [RTRIM](#rtrim)                        | Remove a specified character(s) from the right end of a character string.     |
|      | [QUOTE](#quote)                        | Enclose a character string with single quotes.     |
|      | [UNICODE](#unicode)                    | Return the Unicode code point of a character.     |
|      | [CHAR](#char)                          | A Unicode code point is converted to characters and connected.     |
|      | [PRINTF](#printf)                      | Return the converted character string     |
|      | [TRANSLATE](#translate)                | Replace a character string.     |
| [Time](#time_function) | [NOW](#now)                            | Return the present time.       |
| |[TIMESTAMP](#timestamp)|	Convert the string representation of time to millisecond-precision TIMESTAMP. |
| |[TIMESTAMP_MS](#timestamp_ms)|	Convert the string representation of time to millisecond-precision TIMESTAMP (TIMESTAMP(3)). |
| |[TIMESTAMP_US](#timestamp_us)|	Convert the string representation of time to microsecond-precision TIMESTAMP (TIMESTAMP(6)). |
| |[TIMESTAMP_NS](#timestamp_ns)|	Convert the string representation of time to microsecond-precision TIMESTAMP (TIMESTAMP(9)). |
|      | [TIMESTAMP_ADD](#timestamp_add)       | Add a duration to a time.        |
|      | [TIMESTAMP_DIFF](#timestamp_diff)     | Return the difference of times.     |
|      | [TO_TIMESTAMP_MS](#to_timestamp_ms)  | Add lapsed time to the time point '1970-01-01T00:00:00.000Z'.    |
|      | [TO_EPOCH_MS](#to_epoch_ms)          | Return the lapsed time from the time point '1970-01-01T00:00:00.000Z'.     |
|      | [EXTRACT](#extract)                    | Take out the value of the specific field from time.    |
|      | [STRFTIME](#strftime)                  | Return a character string with the time converted.    |
|      | [MAKE_TIMESTAMP](#make_timestamp)     | Generate time.    |
|      | [TIMESTAMP_TRUNC](#timestamp_trunc)   | Truncate time.    |
| [WINDOW](#window_function)  | [ROW_NUMBER](#row_number)             | Assign a unique sequential value to the resulting Row       |
| [Other](#other_function)    | [COALESCE](#coalesce)                  | Return the first argument that is not NULL.     |
|      | [IFNULL](#ifnull)                      | Return the first argument that is not NULL.     |
|      | [NULLIF](#nullif)                      | Return NULL when two arguments are the same, return the first argument when the arguments are different.     |
|      | [RANDOMBLOB](#randomblob)              | Return a BLOB type value (random number).     |
|      | [ZEROBLOB](#zeroblob)                  | Return a BLOB type value (0x00).     |
|      | [HEX](#hex)                            | Convert a BLOB type value to a hexadecimal type.     |
|      | [TYPEOF](#typeof)                      | Return the data type of a value.     |


These functions are described using the data in the following table as an example.

```example
table: employees

 id   first_name   last_name   age     department    enrollment_period
----+------------+-----------+-------+-------------+-------------------
  0   John         Smith       43      Sales         15.5
  1   William      Jones       59      Development   23.2
  2   Richard      Brown       (NULL)  Sales          7.0
  3   Mary         Taylor      31      Research      (NULL)
  4   Lisa         (NULL)      29      (NULL)         4.9
  5   James        Smith       43      Development   10.3

table: departments

 id   department   
----+------------
  0   Sales
  1   Development
  2   Research

table: travelexpenses

 id    date         empId   amount
-----+------------+-------+--------
  101  2020/02/01   0       200
  102  2020/02/03   2       2500
  103  2020/02/03   3       60
  104  2020/02/04   0       200
  105  2020/02/05   0       150
  106  2020/02/06   3       80
```

[Notice]
- NULL value is expressed as (NULL).


<a id="aggregation"></a>
### Aggregate functions

Functions to aggregate values
DISTINCT or ALL can be specified as the argument of an aggregate function.

|      |      |
|------|-------|
| Format | function( \[DISTINCT \| ALL\] *argument*) |

| Point    | Meaning  |
|---------|-------|
| DISTINCT | Rows of duplicate values are excluded and aggregated |
| ALL      | All the rows including the duplicate values are aggregated. |

When no argument is specified, the resut will be the same as ALL is specified.

[Notice]
- An aggregate function can be used only for a SELECT phrase.
- If there are no rows to be calculated, the result of COUNT is 0. Other aggregate functions result in NULL.


Aggregate functions are also available with the OVER clause as analytic functions. See OVER clause for details.   

Example) Example using aggregate function SUM and OVER clause
```example
SELECT id, date, empId, amount, SUM(amount) OVER(PARTITION BY empID ORDER BY id) as accumulated FROM travelexpenses;
Result:
 id    date         empId   amount   accumulated
-----+------------+-------+--------+-------------
  101  2020/02/01   0       200      200
  104  2020/02/04   0       200      400
  105  2020/02/05   0       150      550
  102  2020/02/03   2       2500     2500
  103  2020/02/03   3       60       60
  106  2020/02/06   3       80       140
```
[Points to note]
- GROUP_CONCAT and MEDIAN cannot be used with the OVER clause.



#### AVG

|      |      |
|------|-------|
| Format | AVG( \[DISTINCT \| ALL\] *n*) |

Return the average value of n.

- Specify a numeric value as the argument n.
- Rows with n of NULL value are excluded from the calculation.
- The result is of a DOUBLE type.

Example:
```example
SELECT AVG(age) FROM employees;
Result: 41.0

SELECT AVG(DISTINCT age) FROM employees;
Result: 40.5

SELECT department, AVG(age) avg FROM employees GROUP BY department;
Result:
  department   avg
  ------------+-----
  Development  51.0
  Research     31.0
  Sales        43.0
  (NULL)       29.0
  ```



#### COUNT

|      |      |
|------|-------|
| Format | COUNT(\* \| \[DISTINCT \| ALL\] *x*) |

Return the number of rows.

- Rows with x of NULL value are excluded from the calculation. They are not included in the number of rows.
- The result is of a LONG type.

Example:
```example
SELECT COUNT(*) FROM employees;
Result: 6

// Count the rows ignoring the ones with NULL value.
SELECT COUNT(department) FROM employees;
Result: 5

SELECT COUNT(DISTINCT department) FROM employees;
Result: 3
```

<a id="Aggregate_MAX"></a>
#### MAX

|      |      |
|------|-------|
| Format | MAX( \[DISTINCT \| ALL\] *x*) |

Return the maximum.

- Specify the value of arbitrary types as the argument x.
  - For the argument of character string type, the character string started with the largest character code is returned.
  - For the argument of TIMESTAMP type, return the newest time.
- Rows with x of NULL value are excluded from the calculation.
- The type of the result is the same as that of the argument x.

Example:
```example
SELECT MAX(age) FROM employees;
Result: 59

SELECT MAX(first_name) FROM employees;
Result: William
```


<a id="Aggregate_MIN"></a>
#### MIN

|      |      |
|------|-------|
| Format | MIN( \[DISTINCT \| ALL\] *x*) |

Return the minimum.

- Specify the value of arbitrary types as the argument x.
  - For the argument of character string type, the character string started with the smallest character code is returned.
  - For the argument of TIMESTAMP type, return the oldest time.
- Rows with x of NULL value are excluded from the calculation.
- The type of the result is the same as that of the argument x.


Example:
```example
SELECT MIN(age) FROM employees;
Result: 29

SELECT MIN(first_name) FROM employees;
Result: James
```



#### SUM/TOTAL

|      |      |
|------|-------|
| Format | SUM( \[DISTINCT \| ALL\] *n*) |
| Format | TOTAL( \[DISTINCT \| ALL\] *n*) |

Return a sum of values.

- Specify a numeric value as the argument n.
- Rows with n of NULL value are excluded from the calculation.

- The difference between SUM and TOTAL is as follows.
  - When n includes integer type values only, SUM returns a value of integer (LONG) type, while TOTAL returns a value of floating point number (DOUBLE).
  - When n includes a floating point number type value, both of them return a value of floating point number (DOUBLE).
  - When n includes NULL only, SUM returns NULL, while TOTAL returns 0. while TOTAL returns 0.

Example:
```example
SELECT SUM(age) FROM employees;
Result: 205

SELECT TOTAL(age) FROM employees;
Result: 205.0

SELECT department, SUM(age) sum FROM employees GROUP BY department;
Result: 
  department   sum
  ------------+-----
  Development  102
  Research      31
  Sales         43
  (NULL)        29
```



#### GROUP_CONCAT

|      |      |
|------|-------|
| Format | GROUP_CONCAT( \[DISTINCT \| ALL\] *x* \[, *separator*\] ) |

Return the character string in which the values of x are concatenated.
Specify the separator to be concatenated as "separator". When not specified, ", " is used.

- Specify the value of arbitrary types as the argument x.
  - A TIMESTAMP value (including a value of the TIMESTAMP type with specified precision) is converted to the string representation of time in 'YYYY-MM-DDThh:mm:ss.SSS(Z|±hh:mm)' format and connected. The number of digits in fractional parts of time is determined according to the precision of the TIMESTAMP type argument. For details, see the explanation of the corresponding precision in the sections on [TIMESTAMP_MS function](#timestamp_ms) and other relevant sections. 
- Rows with x of NULL value are excluded from the calculation.
- The result is of a STRING type.

Example:
```example
// Concatenate the name last_name with '/'
SELECT GROUP_CONCAT(last_name, '/') from employees;
Result:  Smith/Jones/Brown/Taylor/Smith

// Concatenate the name "first_name" for each department "department"
SELECT department, GROUP_CONCAT(first_name) group_concat from employees GROUP BY(department);
Result: 
   department    group_concat
  -------------+--------------
   Development  William,James
   Research     Mary
   Sales        John,Richard
   (NULL)       Lisa

SELECT GROUP_CONCAT(age, ' + ') FROM employees;
Result: 43 + 59 + 31 + 29 + 43
```



#### STDDEV_SAMP

|      |      |
|------|-------|
| Format | STDDEV_SAMP( \[DISTINCT \| ALL\] *x*) |

Returns the sample standard deviation.

- Specify a numeric value for the argument x.
  - Expressions cannot contain aggregate functions or WINDOW functions/OVER clauses.
- Rows with x of NULL value are excluded from the calculation.
- If x is one, returns NULL.
- The result is of a DOUBLE type.

Example:
```example
SELECT department, STDDEV_SAMP(enrollment_period) enrollment_period_stddev from employees GROUP BY department;
Result:
   department    enrollment_period_stddev
  -------------+--------------------------
   Development  9.121677477306465
   Research     (NULL)
   Sales        6.010407640085654
   (NULL)       (NULL)

```



#### STDDEV/STDDEV0

|      |      |
|------|-------|
| Format | STDDEV( \[DISTINCT \| ALL\] *x*) |
| Format | STDDEV0( \[DISTINCT \| ALL\] *x*) |

Returns the sample standard deviation. Returns the sample standard deviation. STDDEV is an alias of the STDDEV_SAMP function.

- Specify a numeric value for the argument x.
  - Expressions cannot contain aggregate functions or WINDOW functions/OVER clauses.
- Rows with x of NULL value are excluded from the calculation.
- The result is of a DOUBLE type.
- The differences between STDDEV and STDDEV0 are as follows:
  - STDDEV returns NULL if x is 1,
  - while STDDEV0 returns 0 when x is 1.

Example:
```example
SELECT department, STDDEV(enrollment_period) enrollment_period_stddev from employees GROUP BY department;
Result:
   department    enrollment_period_stddev
  -------------+--------------------------
   Development  9.121677477306465
   Research     (NULL)
   Sales        6.010407640085654
   (NULL)       (NULL)

SELECT department, STDDEV0(enrollment_period) enrollment_period_stddev from employees GROUP BY department;
Result:
   department    enrollment_period_stddev
  -------------+--------------------------
   Development  9.121677477306465
   Research     (NULL)
   Sales        6.010407640085654
   (NULL)       0.0

SELECT STDDEV(enrollment_period) enrollment_period_stddev from employees WHERE age >= 55;
Result:
   enrollment_period_stddev
  --------------------------
   (NULL)

SELECT STDDEV0(enrollment_period) enrollment_period_stddev from employees WHERE age >= 55;
Result:
   enrollment_period_stddev
  --------------------------
   0.0
```



#### STDDEV_POP

|      |      |
|------|-------|
| Format | STDDEV_POP( \[DISTINCT \| ALL\] *x*) |

Returns the population standard deviation.

- Specify a numeric value for the argument x.
  - Expressions cannot contain aggregate functions or WINDOW functions/OVER clauses.
- Rows with x of NULL value are excluded from the calculation.
- The result is of a DOUBLE type.

Example:
```example
SELECT department, STDDEV_POP(enrollment_period) enrollment_period_stddev from employees GROUP BY department;
Result:
   department    enrollment_period_stddev
  -------------+--------------------------
   Development  6.450000000000002
   Research     (NULL)
   Sales        4.25
   (NULL)       0.0

```



#### VAR_SAMP

|      |      |
|------|-------|
| Format | VAR_SAMP( \[DISTINCT \| ALL\] *x*) |

Returns the sample variance.

- Specify a numeric value for the argument x.
  - Expressions cannot contain aggregate functions or WINDOW functions/OVER clauses.
- Rows with x of NULL value are excluded from the calculation.
- If x is one, returns NULL.
- The result is of a DOUBLE type.

Example:
```example
SELECT department, VAR_SAMP(enrollment_period) enrollment_period_variance from employees GROUP BY department;
Result:
   department    enrollment_period_variance
  -------------+----------------------------
   Development  83.20500000000004
   Research     (NULL)
   Sales        36.125
   (NULL)       (NULL)

```



#### VARIANCE/VARIANCE0

|      |      |
|------|-------|
| Format | VARIANCE( \[DISTINCT \| ALL\] *x*) |
| Format | VARIANCE0( \[DISTINCT \| ALL\] *x*) |

Returns the sample variance. Returns the sample variance. VARIANCE is an alias of the VAR_SAMP function.

- Specify a numeric value for the argument x.
  - Expressions cannot contain aggregate functions or WINDOW functions/OVER clauses.
- Rows with x of NULL value are excluded from the calculation.
- The result is of a DOUBLE type.
- The differences between VARIANCE and VARIANCE0 are as follows:
  - VARIANCE returns NULL if x is 1,
  - while VARIANCE0 returns 0 if x is 1.

Example:
```example
SELECT department, VARIANCE(enrollment_period) enrollment_period_variance from employees GROUP BY department;
Result:
   department    enrollment_period_variance
  -------------+----------------------------
   Development  83.20500000000004
   Research     (NULL)
   Sales        36.125
   (NULL)       (NULL)

SELECT department, VARIANCE0(enrollment_period) enrollment_period_variance from employees GROUP BY department;
Result:
   department    enrollment_period_variance
  -------------+----------------------------
   Development  83.20500000000004
   Research     (NULL)
   Sales        36.125
   (NULL)       0.0

SELECT VARIANCE(enrollment_period) enrollment_period_variance from employees WHERE age >= 55;
Result:
   enrollment_period_variance
  ----------------------------
   (NULL)

SELECT VARIANCE0(enrollment_period) enrollment_period_variance from employees WHERE age >= 55;
Result:
   enrollment_period_variance
  ----------------------------
   0.0
```


#### VAR_POP

|      |      |
|------|-------|
| Format | VAR_POP( \[DISTINCT \| ALL\] *x*) |

Returns the population variance.

- Specify a numeric value for the argument x.
  - Expressions cannot contain aggregate functions or WINDOW functions/OVER clauses.
- Rows with x of NULL value are excluded from the calculation.
- The result is of a DOUBLE type.

Example:
```example
SELECT department, VAR_POP(enrollment_period) enrollment_period_variance from employees GROUP BY department;
Result:
   department    enrollment_period_variance
  -------------+----------------------------
   Development  41.60250000000002
   Research     (NULL)
   Sales        18.0625
   (NULL)       0.0

```



#### MEDIAN

|      |      |
|------|-------|
| Format | MEDIAN(*n*) |

Returns the median of n. If the number of rows to be calculated is even, returns the average value of the two rows near the center.

- Specify a numeric value as the argument n.
  - Subqueries cannot be specified.
- Rows with n of NULL value are excluded from the calculation.
- The type of the result is a LONG type when n includes only integers, a DOUBLE type when n includes a floating point number.
- Multiple use of the WINDOW function/OVER clause in the same SELECT clause, and simultaneous use of the WINDOW function/OVER clause and the MEDIAN function are not allowed.

Example:
```example
SELECT MEDIAN(age) FROM employees;
Result: 43

SELECT department, MEDIAN(age) mn FROM employees GROUP BY department ORDER BY mn DESC;
Result:
  department   mn
  ------------+-----
  Development  51
  Sales        43
  Research     31
  (NULL)       29
```


#### PERCENTILE_CONT

|      |      |
|------|-------|
| Format | PERCENTILE_CONT(*percentile*) WITHIN GROUP ( ORDER BY *sort_key* ) |

Return a value that corresponds to the percentile specified by the *percentile* which is based on a continuous distribution model, given the sort order specified by *sort_key*.

- For the *percentile*, specify a DOUBLE constant greater than or equal to zero and less than or equal to 1. 
- NULL values that exist in *sort_key* are not aggregated.
- NULL is returned only when there is nothing to be aggregated.
- PERCENTILE_CONT cannot be used at the same time with the WINDOW function and OVER clause within the same SELECT clause, the MEDIAN function, and other PERCENTILE_CONT's.

Example:
```example
SELECT PERCENTILE_CONT(0.25) WITHIN GROUP( ORDER BY age ) FROM employees;
Result: 18

```


### Mathematical functions

#### ABS

|      |      |
|------|-------|
| Format | ABS(*n*) |

Return the absolute value of n. For a positive number, the value as it is is returned and for a negative number, the value multiplied by -1 is returned.

- Specify a numeric value as the argument n.
- Return NULL, when the result value is NULL.
- Cause an overflow error, when the value is an integer of -2<sup>63</sup>.
- The type of the result is a LONG type when n includes only integers, a DOUBLE type when n includes a floating point number.

Example:
```example
SELECT first_name, ABS(age) abs FROM employees;
Result:
  first_name    abs
  ------------+-------
  John          43
  William       59
  Richard       (NULL)
  Mary          31
  Lisa          29
  James         43
```



#### ROUND

|      |      |
|------|-------|
| Format | ROUND(*n* \[, *m*\]) |

Round off. Returns the value of n rounded to m decimal places.

- Specify a row of a numeric type as the argument n.
- Specify an integer greater than or equal to 0 as the argument m. When no value is specified for m, the default value 0 is specified.
- Return NULL, when the result value is NULL.
- The type of the result is a LONG type when n includes only integers, a DOUBLE type when n includes a floating point number.

Example:
```example
SELECT first_name, ROUND(enrollment_period, 0) round FROM employees;
Result:
  first_name    round
  ------------+-------
  John          16.0
  William       23.0
  Richard        7.0
  Mary          (NULL)
  Lisa           5.0
  James         10.0
```



#### RANDOM

|      |      |
|------|-------|
| Format | RANDOM() |

Return a random number. A random number is an integer of the range from -2<sup>63</sup> to 2<sup>63</sup>-1.

- The result is of a LONG type.

Example:
```example
SELECT first_name, RANDOM() random FROM employees;
Result：
  first_name    random
  ------------+----------------------
  John          -3382931580741820003
  William       -7362300487836647182
  Richard        8834368641333737477
  Mary          -8544493602797564288
  Lisa          -7727163797274657674
  James          6751560427268247384
```


<a id="Arithmetic_MAX"></a>
<a id="Arithmetic_MIN"></a>
#### MAX/MIN

|      |      |
|------|-------|
| Format | MAX(*x1*, *x2* \[,...\]) |

Return the greatest value among the values xN.

|      |      |
|------|-------|
| Format | MIN(*x1*, *x2* \[,...\]) |

Return the smallest value among the values xN.


Example:
```example
SELECT first_name, age, enrollment_period, MAX(age, enrollment_period) max FROM employees;
Result:
  first_name    age    enrollment_period   max
  ------------+-------+------------------+--------
  John          43      15.5               43.0
  William       59      23.2               59.0
  Richard       (NULL)   7.0               (NULL)
  Mary          31      (NULL)             (NULL)
  Lisa          29       4.9               29.0
  James         43      10.3               43.0
```

<a id="Arithmetic_LOG"></a>
#### LOG

|     |               |
| --- | ------------- |
| Format | LOG(*n*, *m*) |

Returns the logarithm of m with base n.

- For the argument n, specify a numeric value greater than 0 and other than 1.
- For the argument m, specify a numeric value greater than 0.
- Return NULL, when the result value is NULL.
- The result is of a DOUBLE type.

Example:

```example
SELECT LOG(2, 8);
Result: 3.0

SELECT LOG(0.5, 2.0);
Result: -1.0
```

<a id="Arithmetic_SQRT"></a>
#### SQRT

|     |           |
| --- | --------- |
| Format | SQRT(*n*) |

Returns the positive square root of n.

- Specify a numeric value of 0 or greater as the argument n.
- Return NULL, when the result value is NULL.
- The result is of a DOUBLE type.

Example:

```example
SELECT SQRT(4);
Result: 2.0

SELECT SQRT(16.0);
Result: 4.0
```

<a id="Arithmetic_TRUNC"></a>
#### TRUNC

|     |                                                    |
| --- | -------------------------------------------------- |
| Format | TRUNC(*n* \[,*m*\]) |

In the case of m\>=0, return the value of n, rounded down to the nearest m digits.

In the case of m\< 0, return the value of n, rounded down to the nearest -m digits.

- Specify a numeric value as the argument n.
- Specify an integer as the argument m. When no value is specified for m, the default value 0 is specified. A value greater than 309 or less than -308 cannot be specified.
- Return NULL, when the result value is NULL.
- The result type is LONG if an integer is specified for the argument n and DOUBLE if a decimal is specified.

Example:

```example
SELECT TRUNC(123.4567);
Result: 123.0

SELECT TRUNC(123.4567, 2);
Result: 123.45

SELECT TRUNC(123.4567, -1);
Result: 120.0

SELECT TRUNC(123.4567, -3);
Result: 0.0

SELECT TRUNC(1234567, -2);
Result: 1234500
```

<a id="Arithmetic_HEX_TO_DEC"></a>
#### HEX_TO_DEC

|     |               |
| --- | ------------- |
| Format | HEX_TO_DEC(*str*) |

Converts hexadecimal string str to decimal number type.

- Specify a character string type value (0-9, a-f, A-F) that can be converted to hexadecimal for the argument str.
- Return NULL, when the result value is NULL.
- The result is of a LONG type.

Example:
```example
SELECT HEX_TO_DEC('FF');
Result: 255

SELECT HEX_TO_DEC('10');
Result: 16
```

### Character functions


#### LENGTH

|      |      |
|------|-------|
| Format | LENGTH(*str*) |

Return the length of the character string str.

- Specify character string type values for the argument str.
  - Unicode code point of a character string is used.
- Return NULL, when the result value is NULL.
- The result is of a LONG type.
- A BLOB type can also be specified for an argument.

Example:
```example
SELECT last_name, LENGTH(last_name) length FROM employees;
Result:
  last_name     length
  ------------+----------------------
  Smith         5
  Jones         5
  Brown         5
  Taylor        6
  (NULL)        (NULL)
  Smith         5
```



#### LOWER

|      |      |
|------|-------|
| Format | LOWER(*str*) |

Convert all the alphabet of the character string str to lowercases.

- Specify character string type values for the argument str.
- Return NULL, when the result value is NULL.
- The result is of a character string type.
- Unicode characters other than ASCII alphabetic characters are not converted.

Example:
```example
SELECT last_name, LOWER(last_name) lower FROM employees;
Result:
  last_name     lower
  ------------+----------------------
  Smith         smith
  Jones         jones
  Brown         brown
  Taylor        taylor
  (NULL)        (NULL)
  Smith         smith
```



#### UPPER

|      |      |
|------|-------|
| Format | UPPER(*str*) |

Convert all the alphabet of the character string str to uppercases.

- Specify character string type values for the argument str.
- Return NULL, when the result value is NULL.
- The result is of a character string type.
- Unicode characters, such as Cyrille characters, other than ASCII alphabetic characters are not converted.

Example:
```example
SELECT last_name, UPPER(last_name) upper FROM employees;
Result:
  last_name     upper
  ------------+----------------------
  Smith         SMITH
  Jones         JONES
  Brown         BROWN
  Taylor        TAYLOR
  (NULL)        (NULL)
  Smith         SMITH
```



#### SUBSTR

|      |      |
|------|-------|
| Format | SUBSTR(*str*, *index* \[, *length*\]) |

Cut out a part of a character string. from the character on the starting position, indicated by "index" up to the length specified by "length".

- Specify character string type values for the argument str.
- Specify an integer, 1 or larger, as the argument index. The starting position at the beginning of a character string is 1.
- When the argument length is not specified, the character strings up to the end of str is cut out.
- Return NULL, when the str value is NULL.
- The result is of a character string type.
- A BLOB type can also be specified for an argument.

Example:
```example
SELECT SUBSTR('abcdefg', 3);
Result: cdefg

SELECT SUBSTR('abcdefg', 3, 2);
Result: cd
```



#### REPLACE

|      |      |
|------|-------|
| Format | REPLACE(*str*, *search_str*, *replacement_str*) |

Replace a character string.
In the character string str, replace all the parts matching the character string search_str with replacement_str.

- Specify character string type values for the argument search_str, replacement_str.
- Return NULL, when the str value is NULL.
- The result is of a character string type.

Example:
```example
SELECT REPLACE('abcdefabc', 'abc', '123');
Result: 123def123
```



#### INSTR

|      |      |
|------|-------|
| Format | INSTR(*str*, *search_str* \[, offset\] \[, occurrence\]) |

Search for character string search_str in the character string str, and return its starting position. Return 0, when not found. Return 0, when not found.

- Specify a string type or BLOB type value for the arguments str and search_str. The values of the same data type must be specified for str and search_str. For the offset and occurrence arguments, specify a LONG value.
- For string type, it is calculated in Unicode code point unit, and for BLOB type, it is calculated in byte unit.
- offset indicates the position where the search starts: for a positive value, the search starts from the front; for a negative value, the search starts from the rear end; when 0 is specified, 0 is returned meaning no match.
- occurrence indicates the number of matches: the search is repeated the specified number of times and the last matched position is returned. when 0 is specified, 0 is returned meaning no match.
- Return NULL, when either of the value of the arguments is NULL.
- The result is of a LONG type.


Example:
```example
SELECT INSTR('abcdef', 'cd');
Result: 3

SELECT INSTR('abcdef', 'gh');
Result: 0

SELECT INSTR('abcabcabcde', 'ab', 2, 2);
Result: 7

SELECT INSTR('abcabcabcde', 'ab', -1, 2);
Result: 4

```



#### LIKE

|      |      |
|------|-------|
| Format | LIKE(*pattern_str*, *str* \[, *escape_str*\]) |

Search the character string on the right.
Return true, when the character string str matches the match pattern pattern_str. Return false, when no match was found.
The following two wild cards are available for a match pattern.

| Wild card | Meaning |
|------|-----|
| _        | Any one character |
| %         | Any character with zero or more character strings |

Specify the escape character escape_str when searching for the character _ or % in str containing the wildcard character _ or %.
If a escape character is specified before the wild card character, it will no longer be interpreted as a wild card.

- Specify character string type values for the argument str, pattern_str ,escape_str.
- Return NULL, when either of the value of the arguments is NULL.
- Uppercase and lowercase characters are not distinguished.
- The result is of a BOOL type.

Example:
```example
SELECT last_name, LIKE('%mi%', last_name) like_name FROM employees;
Result:
  last_name     like_name
  ------------+----------------------
  Smith         true
  Jones         false
  Brown         false
  Taylor        false
  (NULL)        (NULL)
  Smith         true


SELECT LIKE('%C%E%',  'ABC%DEF');
Result: true

SELECT LIKE('%C@%E%', 'ABC%DEF', '@');
Result: false

SELECT LIKE('%C@%D%', 'ABC%DEF', '@');
Result: true
```



#### GLOB

|      |      |
|------|-------|
| Format | GLOB(*pattern_str*, *str*) |

Search the character string on the right.
Return true, when the character string str matches the match pattern pattern_str. Return false, when no match was found.
The following wild cards are available for a match pattern.

| Wild card | Meaning |
|------|-----|
| ?         | Any one character |
| \*        | Any character with zero or more character strings |
| [abc]   | Match any of the letters a, b or c |
| [a-e]   | Match any of the letters from a to e |

- Specify character string type values for the argument str, pattern_str.
- Return NULL, when either of the value of the arguments is NULL.
- Uppercase and lowercase characters are distinguished.
- The result is of a BOOL type.

Example:
```example
SELECT GLOB('*[BA]AB?D', 'AABCD');
Result: true
```



#### TRIM

|      |      |
|------|-------|
| Format | TRIM(*str* \[, *trim_str*\]) |

Delete all the characters of character string trim_str from both ends of the character string str.

- Specify character string type values for the argument str and trim_str.
- Delete all the characters contained in the argument trim_str. When no value is specified, spaces are deleted from both ends of str.
- The result is of a character string type.

Example:
```example
SELECT TRIM(' ABC ');
Result: ABC (no space at both ends)

SELECT TRIM('ABCAA', 'BA');
Result: C
```



#### LTRIM

|      |      |
|------|-------|
| Format | LTRIM(*str* \[, *trim_str*\]) |

Delete all the characters of character string trim_str from the left end of the character string str.

- Specify character string type values for the argument str and trim_str.
- Delete all the characters contained in the argument trim_str. When no value is specified, spaces are deleted from the left end of str.
- The result is of a character string type.

Example:
```example
SELECT TRIM(' ABC ');
Result: ABC (no space at the left end)

SELECT TRIM('ABCAA', 'BA');
Result: BCAA
```



#### RTRIM

|      |      |
|------|-------|
| Format | RTRIM(*str* \[, *trim_str*\]) |

Delete all the characters of character string trim_str from the right end of the character string str.

- Specify character string type values for the argument str and trim_str.
- Delete all the characters contained in the argument trim_str. When no value is specified, spaces are deleted from the right end of str.
- The result is of a character string type.

Example:
```example
SELECT RTRIM(' ABC ');
Result: ABC (no space at the right end)

SELECT RTRIM('ABCAA', 'A');
Result: ABC
```



#### QUOTE

|      |      |
|------|-------|
| Format | QUOTE(*x*) |

Returns a character string containing the value of x enclosed in single quotes.

- For the argument x, specify a value of a character string type, a numeric type, a TIMESTAMP type, and a BLOB type value.
  - For the string type, single quotes contained in the string are escaped into two single quotes ''.
  - For the numeric type, a numeric value is returns as it is. It is not enclosed in single quotes.
  - A TIMESTAMP value (including a value of the TIMESTAMP type with specified precision) is converted to the string representation of time in 'YYYY-MM-DDThh:mm:ss.SSS(Z|±hh:mm)' format and connected. It is not enclosed in single quotes. The number of digits in fractional parts of time is determined according to the precision of the TIMESTAMP type argument. For details, see the explanation of the corresponding precision in the sections on [TIMESTAMP_MS function](#timestamp_ms) and other relevant sections. 
  - For the BLOB type, return the character string X'BLOB type value'.
- The result is of a character string type.

Example:
```example
SELECT QUOTE(last_name) last_name, QUOTE(age) age FROM employees;
Result:
  last_name     age
  ------------+-------
  'Smith'       43
  'Jones'       59
  'Brown'       (NULL)
  'Taylor'      31
  (NULL)        29
  'Smith'       43

SELECT QUOTE(RANDOMBLOB(4));
Result: X'A45EA28D'

// The value of column "value" is a character string "Today's news."
SELECT value, QUOTE(value) FROM testcontainer;
Result:
   value            QUOTE(value)
  ---------------+-------------------
   Today's news     'Today''s news'
```



#### UNICODE

|      |      |
|------|-------|
| Format | UNICODE(*str*) |

Returns the UNICODE code point of the first character of the string str.

- Specify character string type values for the argument str.
- The result is of a LONG type.

Example:
```example
SELECT last_name, UNICODE(last_name) unicode FROM employees;
Result:
  last_name     unicode
  ------------+----------------------
  Smith         83
  Jones         74
  Brown         66
  Taylor        84
  (NULL)        (NULL)
  Smith         83
```



#### CHAR

|      |      |
|------|-------|
| Format | CHAR(*x1* \[, *x2*, ... , *xn*\]) |

Returns a concatenated character string of characters with Unicode code point value xn.

- Specify a Unicode code point value for an argument xn.
- The result is of a STRING type.

Example:
```example
SELECT CHAR(83, 84, 85);
Result: STU
```



#### PRINTF

|      |      |
|------|-------|
| Format | PRINTF(*format* \[, *x1*, *x2*, ..., *xn*\]) |

Return the converted character string according to the specified format "format".
A format equivalent to the printf function of the standard C libraries can be used.
There are two other formats as below.

| Format | Description |
|----|-----------------------|
| %q | A single quote in a character string is escaped to two single quotes ''.  |
| %Q | A single quote in a character string is escaped to two single quotes ''. <br>Enclose both ends of the character string by single quotes.   |

Example:
```example
SELECT enrollment_period, PRINTF('%.2f', enrollment_period) printf FROM employees;
Result:
  enrollment_period   printf
  ------------------+-----------
  15.5                15.50
  23.2                23.20
   7.0                 7.00
  (NULL)               0.00
   4.9                 4.90
  10.3                10.30
```

#### TRANSLATE
|     |                                                   |
| --- | ------------------------------------------------- |
| Format | TRANSLATE(*str*, *search_str*, *replacement_str*) |

Replace a character string. Among the character string str, the characters matched the character string search_str is replaced by the characters of character string replacement_str in the same position as search_str. When replacement_str is shorter than search_str, thus having no characters to substitute in the part longer than replacement_str, the characters to be replaced will be deleted.

- Specify character string type values for the argument search_str, replacement_str.
- Return NULL, when the result value is NULL.
- The result is of a character string type.

Example:

```example
SELECT TRANSLATE('abcde', 'ace', '123');
Result:1b2d3

SELECT TRANSLATE('abcdeca', 'ace', '123');
Result: 1b2d321

SELECT TRANSLATE('abcde', 'ac', '123');
Result: 1b2de

SELECT TRANSLATE('abcde', 'ace', '12');
Result: 1b2d

SELECT TRANSLATE('abcde', 'AB', '123');
Result: abcde

SELECT TRANSLATE('abcde', 'abc', '');
Result: de
```

<a id="time_function"></a>
### Time functions

#### NOW

|      |      |
|------|-------|
| Format | NOW() |

Returns the current time value.

- If the time zone is specified at the time of connection, the offset calculated value is returned.
- The type of the result is millisecond-precision TIMESTAMP.

Example:
```example
SELECT NOW();
Result: 2019-09-17T04:07:31.825Z

SELECT NOW();
Result: 2019-09-17T13:09:20.918+09:00
```

#### TIMESTAMP

|      |      |
|------|-------|
| Format | TIMESTAMP(*timestamp_string* \[, timezone\]) |

Convert values of the string representation of time *timestamp_string* to millisecond-precision TIMESTAMP.
This function is equivalent to TIMESTAMP_MS below. For details, see the section on TIMESTAMP_MS.

#### TIMESTAMP_MS

|      |      |
|------|-------|
| Format | TIMESTAMP_MS(*timestamp_string* [, timezone]) |

Convert values of the string representation of time *timestamp_string* to millisecond-precision TIMESTAMP(3).

- In the argument timestamp_string, specify a character string in the following format as a character string representation of time.
  - YYYY-MM-DDThh:mm:ssZ
  - YYYY-MM-DDThh:mm:ss.SSSZ
  - YYYY-MM-DD
  - hh:mm:ss

  | Notation | Item                    | The range of value   |
  |------|----------|------------|
  | YYYY     | Year (A.D.)             | 1970- |
  | MM       | Month                   | 1 to 12 |
  | DD       | Day                     | 1 to 31 |
  | hh       | Time (24-hour notation) | 0 to 23 |
  | mm       | Minute                  | 0 to 59 |
  | ss       | Second                  | 0 to 59 |
  | SSS      | Millisecond             | 0 to 999 |
  | Z        | Time zone               | Z|±hh:mm|±hhmm |

- For the timezone argument, specify the time zone (Z|±hh:mm|±hhmm), not required when time zone information is included in timestamp_string. An error is returned if the specified values are inconsistent.
- If the time zone is specified at the time of connection, the offset calculated value is returned.
- The type of the result is millisecond-precision TIMESTAMP(3).
- For the reverse conversion of the TIMESTAMP_MS function, i.e., conversion of a millisecond-precision TIMESTAMP(3) type to a string type, use [CAST](#CAST).
  - CAST(*timestamp* AS STRING)

Example:
```example
// Search for a row with the value of column date (TIMESTAMP type) newer than time '2018-12-01T10: 30: 00Z'
SELECT * FROM timeseries WHERE date > TIMESTAMP('2018-12-01T10:30:00Z');
```


#### TIMESTAMP_US

|      |      |
|------|-------|
| Format | TIMESTAMP_US(*timestamp_string* [, timezone]) |

Convert values of the string representation of time *timestamp_string* to microsecond-precision TIMESTAMP(6).

- For the argument *timestamp_string*, specify strings in the following format as a string representation of time.。
  - YYYY-MM-DDThh:mm:ssZ
  - YYYY-MM-DDThh:mm:ss.SSSZ
  - YYYY-MM-DDThh:mm:ss.SSSSSSZ
  - YYYY-MM-DD
  - hh:mm:ss
    
  | notation | meaning      | value range   |
  |------|----------|------------|
  | YYYY | year  | 1970 and onwards |
  | MM   | month　　　 | 1 to 12 |
  | DD   | day　　 | 1 to 31 |
  | hh   |  hour (24-hour format)　　　 | 0 to 23 |
  | mm   | minute　　　 | 0 to 59 |
  | ss   | second　　　 | 0 to 59 |
  | SSSSSS  | microsecond　 | 0 to 999999 |
  | Z    | timezone | Z\|±hh:mm\|±hhmm |

- For the argument *timezone*, specify the timezone (Z|±hh:mm|±hhmm). If *timestamp_string* contains timezone information, specification is not necessary. But if the timezone is specified and there is an inconsistency between timezone information and the specified timezone, an error will be returned.
- If the timezone is specified at the time of connection, a value obtained by calculating the offset is returned.
- The type of the result is microsecond-precision TIMESTAMP(6).
- For the reverse conversion of the TIMESTAMP_US function, i.e., conversion of a microsecond-precision TIMESTAMP(6) type to a string type, use [CAST](#CAST).
  + CAST(*timestamp* AS STRING)


#### TIMESTAMP_NS

|      |      |
|------|-------|
| Format | TIMESTAMP_NS(*timestamp_string* [, timezone]) |

Convert values of the string representation of time *timestamp_string* to nanosecond-precision TIMESTAMP(9).

- For the argument *timestamp_string*, specify strings in the following format as a string representation of time.
  - YYYY-MM-DDThh:mm:ssZ
  - YYYY-MM-DDThh:mm:ss.SSSZ
  - YYYY-MM-DDThh:mm:ss.SSSSSSZ
  - YYYY-MM-DDThh:mm:ss.SSSSSSSSSZ
  - YYYY-MM-DD
  - hh:mm:ss

  
  | notation | meaning      | value range   |
  |------|----------|------------|
  | YYYY | year  | 1970 and onwards |
  | MM   | month　　　 | 1 to 12 |
  | DD   | day　　 | 1 to 31 |
  | hh   |  hour (24-hour format)　　　 | 0 to 23 |
  | mm   | minute　　　 | 0 to 59 |
  | ss   | second　　　 | 0 to 59 |
  | SSSSSSSSS  | nanosecond　 | 0 to 99999999 |
  | Z    | timezone | Z\|±hh:mm\|±hhmm |

- For the argument *timezone*, specify the timezone (Z|±hh:mm|±hhmm). If *timestamp_string* contains timezone information, specification is not necessary. But if the timezone is specified and there is an inconsistency between timezone information and the specified timezone, an error will be returned.
- If the timezone is specified at the time of connection, a value obtained by calculating the offset is returned.
- The type of the result is nanosecond-precision TIMESTAMP(9).
- For the reverse conversion of the TIMESTAMP_NS function, i.e., conversion of a nanosecond-precision TIMESTAMP(9) type to a string type, use [CAST](#CAST).
  + CAST(*timestamp* AS STRING)


#### TIMESTAMP_ADD

|      |      |
|------|-------|
| Format | TIMESTAMP_ADD(*time_unit*, *timestamp*, *duration* \[, timezone\])

The value obtained by adding the period "duration" (unit: time_umit) to time period "timestamp" is returned.

- For the argument *timestamp*, specify TIMESTAMP values. Values for the TIMESTAMP type with specified precision values can also be specified.
- Specify an integer for an argument duration. Subtract from the time point, when a negative number is specified.
- Specify one of the following identifiers for the argument time_unit:
  - YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND
- For the timezone argument, specify the time zone (Z|±hh:mm|±hhmm),
- If the calculated day of the month does not exist as a result of adding a year or a month, the day is rounded to the last day of the month. For example, if one month is added to May 31, the result will be rounded to June 30 because June 31 does not exist.
- If the time zone is specified at the time of connection, the offset calculated value is returned.
- The result is of a TIMESTAMP type.
- TIMESTAMPADD can also be used as a function alias.

Example:
```example
Add ten days to time period '2018-12-01T11:22:33.444Z'.
SELECT TIMESTAMP_ADD(DAY, TIMESTAMP('2018-12-01T11:22:33.444Z'), 10);
Result: 2018-12-11T11:22:33.444Z

SELECT TIMESTAMP_ADD(MONTH, TIMESTAMP('2019-05-31T01:23:45.678Z'), 1);
Result: 2019-06-30T01:23:45.678Z

SELECT TIMESTAMP_ADD(MONTH, TIMESTAMP('2019-05-31T01:23:45.678Z'), 1, '-02:00');
Result: 2019-07-01T01:23:45.678Z

```

 - The time unit *time_unit* can be specified up to MILLISECOND.


#### TIMESTAMP_DIFF

|      |      |
|------|-------|
| Format | TIMESTAMP_DIFF(*time_unit*, *timestamp1*, *timestamp2* \[, timezone\])

Returns the difference of timestamp1 and timestamp2 (timestamp1-timestamp2) as a value expressed in the time unit "time_unit".
When a time difference is represented in time units, the decimal places are rounded off.

- For the arguments *timestamp1* and *timestamp2*, specify TIMESTAMP values. Values for the TIMESTAMP type with specified precision values can also be specified.
- Specify one of the following identifiers for the argument time_unit: Instead of calculating the difference only in the unit specified by the identifier, the unit less than the identifier is also used in the calculation. For example, if MONTH is specified and 2019/09/30 is compared with 2019/10/02, the output will be 0 instead of 1 because 2 days of 0 months will be the difference.
  - YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND
- For the timezone argument, specify the time zone (Z|±hh:mm|±hhmm),
- If the time zone is specified at the time of connection, the offset calculated value is used for the calculation of the difference.
- The result is of a LONG type.
- TIMESTAMPDIFF can also be used as a function alias.

Example:
```example

// Time unit: Month
SELECT TIMESTAMP_DIFF(MONTH, TIMESTAMP('2018-12-11T10:30:15.555Z'), TIMESTAMP('2018-12-01T10:00:00.000Z'));
Result: 0

// Time unit: Day
SELECT TIMESTAMP_DIFF(DAY,   TIMESTAMP('2018-12-11T10:30:15.555Z'), TIMESTAMP('2018-12-01T10:00:00.000Z'));
Result: 10
SELECT TIMESTAMP_DIFF(DAY,   TIMESTAMP('2018-12-01T11:00:00.000Z'), TIMESTAMP('2018-12-11T10:30:15.555Z'));
Result:-9

// Time unit: Time point
SELECT TIMESTAMP_DIFF(HOUR,  TIMESTAMP('2018-12-11T10:30:15.555Z'), TIMESTAMP('2018-12-01T10:00:00.000Z'));
Result: 240

// Time unit: Minute
SELECT TIMESTAMP_DIFF(MINUTE, TIMESTAMP('2018-12-11T10:30:15.555Z'), TIMESTAMP('2018-12-01T10:00:00.000Z'));
Result: 14430

// Here is an example where the result changes depending on the time zone. 
SELECT TIMESTAMP_DIFF(MONTH, MAKE_TIMESTAMP(2019, 8, 1), MAKE_TIMESTAMP(2019, 6, 30), 'Z');
Result: 2

SELECT TIMESTAMP_DIFF(MONTH, MAKE_TIMESTAMP(2019, 8, 1), MAKE_TIMESTAMP(2019, 6, 30), '-01:00');
Result: 1

```

[memo]
- The time unit *time_unit* can be specified up to MILLISECOND.


#### TO_TIMESTAMP_MS

|      |      |
|------|-------|
| Format | TO_TIMESTAMP_MS(*milliseconds*) |

Return the time point obtained by adding the value of argument "milliseconds" as millisecond, to the time point'1970-01-01T00:00:00.000Z'.

This function is an inverse conversion of TO_EPOCH_MS function.

- Specify an integer for the argument "milliseconds".
- If the time zone is specified at the time of connection, the offset calculated value is returned.
- The result is of a TIMESTAMP type.

Example:
```example
SELECT TO_TIMESTAMP_MS(1609459199999);
Result: 2020-12-31T23:59:59.999Z
```



#### TO_EPOCH_MS

|      |      |
|------|-------|
| Format | TO_EPOCH_MS(*timestamp*) |

Return the lapsed time (in milliseconds) from the time '1970-01-01T00:00:00.000Z' to the time "timestamp".

This function is an inverse conversion of TO_EPOCH_MS function.

- For the argument *timestamp*, specify TIMESTAMP values. Values for the TIMESTAMP type with specified precision values can also be specified.
- The result is of a LONG type.

Example:
```example
SELECT TO_EPOCH_MS(TIMESTAMP('2020-12-31T23:59:59.999Z'));
Result: 1609459199999

SELECT TO_EPOCH_MS(TIMESTAMP('2020-12-31T23:59:59.999+09:00'));
Result: 1609426799999
```

[memo]
- Even when a microsecond or nanosecond-precision TIMESTAMP value is specified, the value returned has a millisecond precision.


#### EXTRACT

|      |      |
|------|-------|
| Format | EXTRACT(*time_field*, *timestamp* \[, timezone\]) |

Retrieve the value of time field "time_field" from the time "timestamp". The time will be the value of UTC.

- For the argument *timestamp*, specify TIMESTAMP values. Values for the TIMESTAMP type with specified precision values can also be specified.
- Specify one of the following identifiers for the argument time_field:
  - YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND | MICROSECOND | NANOSECOND | DAY_OF_WEEK | DAY_OF_YEAR
    - DAY_OF_WEEK is from Sunday, as 0, to Saturday, as 6.
    - DAY_OF_YEAR is from January first, as 1, to December 31th, as 365 or 366.
    - If MILLISECOND, MICROSECOND, or NANOSECOND is specified, the time value will include all decimal digits.
- For the *timezone* argument, specify the time zone (Z|±hh:mm|±hhmm).
- If the time zone is specified at the time of connection, the offset calculated value is returned. If it is also specified in the argument timezone, the one specified in the argument will be used.
- The result is of a LONG type.


Example:
```example
// Calculate the value of the year, the day, and the millisecond of time point '2018-12-01T10:30:02.392Z'.

// The value of the year
SELECT EXTRACT(YEAR, TIMESTAMP('2018-12-01T10:30:02.392Z'));
Result: 2018

SELECT EXTRACT(DAY, TIMESTAMP('2018-12-01T10:30:02.392Z'));
// The value of the day
Result: 1

// The value of the millisecond
SELECT EXTRACT(MILLISECOND, TIMESTAMP('2018-12-01T10:30:02.392Z'));
Result: 392


// Consider the time zone. 
SELECT EXTRACT(HOUR, TIMESTAMP('2018-12-01T10:30:02.392Z'), '+09:00');
Result: 19
```

#### STRFTIME

|      |      |
|------|-------|
| Format | STRFTIME(*format*, *timestamp* \[, *modifier*,...\])

Return a time converted to a string according to the specified format.

- Specify the following in the format argument to extract time information.

| Format | Description |
|----|-----------------------|
| %Y     | Extract the year in YYYY format.  |
| %m     | Extract the month in MM format. |
| %d     | Extract the day in DD format. |
| %H     | Extract the time in hh format. |
| %M     | Extract the minute in mm format. |
| %S     | Extract the second in ss format. |
| %3f	| Extract the millisecond-precision fractional parts in SSS format. |
| %6f	| Extract the microsecond-precision fractional parts in SSSSSS format.|
| %9f	| Extract the nanosecond-precision fractional parts in SSSSSSSSS format.|
| %z     | Extract the time zone in ± hh:mm format. |
| %w     | Extracts the day of the week in D format (0 to 6): from Sunday, as 0, to Saturday, as 6. |
| %W     | Extracts the number of the week of the year in DD format (from 00 to 53). The first Monday is considered to be in the first week, and days before that are considered to be in the 0th week. |
| %j     | Extract the number of days from January first in DDD format (001 to 366). |
| %c     | Extract the time in the format YYYY-MM-DDThh:mm:ss\[.SSS\](Z|±hh:mm|±hhmm). |
| %%     | Output % as a character. |

- Specify a TIMESTAMP type value for the argument timestamp.
- For the timezone argument, specify the time zone (Z|±hh:mm|±hhmm),
- The result is of a STRING type.

Example:
```example

SELECT STRFTIME('%c', TIMESTAMP('2019-06-19T14:15:01.123Z'));
Result: 2019-06-19T14:15:01.123Z

SELECT STRFTIME('%H:%M:%S%z', TIMESTAMP('2019-06-19T14:15:01.123Z'), '+09:00');
Result: 23:15:01+09:00

SELECT STRFTIME('%W', TIMESTAMP('2019-01-19T14:15:01.123Z'));
Result: 02
```

#### MAKE_TIMESTAMP

|      |      |
|------|-------|
| Format | MAKE_TIMESTAMP(year, month, day \[, timezone\])<br>MAKE_TIMESTAMP(year, month, day, hour, min, sec \[, timezone\])|

Generate and return a TIMESTAMP type value.

- If the hour, min, and sec arguments are not specified, it is assumed that all 0 have been specified.
- The argument sec can be specified in milliseconds. The value less than a millisecond is rounded, possibly causing a floating point calculation error.
- For the timezone argument, specify the time zone (Z|±hh:mm|±hhmm),
- The type of the result is millisecond-precision TIMESTAMP.

Example:
```example

SELECT MAKE_TIMESTAMP(2019, 9, 19);
Result: 2019-09-19T00:00:00.000Z

SELECT MAKE_TIMESTAMP(2019, 9, 19, 10, 30, 15.123, '+09:00');
Result: 2019-09-19T01:30:15.123Z

```

#### TIMESTAMP_TRUNC

|      |      |
|------|-------|
| Format | TIMESTAMP_TRUNC(field, timestamp \[, timezone\])|

Truncates the time information.

- Specify one of the following identifiers for the argument field:
  - YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND | MICROSECOND | NANOSECOND
- For the argument *timestamp*, specify TIMESTAMP values. Values for the TIMESTAMP type with specified precision values can also be specified.
- For the timezone argument, specify the time zone (Z|±hh:mm|±hhmm),

Example:
```example

SELECT TIMESTAMP_TRUNC(HOUR, MAKE_TIMESTAMP(2019, 9, 19, 10, 30, 15.123));
Result: 2019-09-19T10:00:00.000Z

SELECT TIMESTAMP_TRUNC(DAY, MAKE_TIMESTAMP(2019, 5, 15), '-01:00');
Result: 2019-05-14T01:00:00.000Z

```


<a id="window_function"></a>
### WINDOW function

- Use the WINDOW function with the OVER clause. See [OVER clause](#over) for details.

#### ROW_NUMBER

|      |      |
|------|-------|
| Format | ROW_NUMBER()  |

Assign a unique serial number to the resulting rows.

- Use the function ROW_NUMBER with the clause OVER. For details, see [OVER clause](#over).

Example:
```example
SELECT ROW_NUMBER() OVER(PARTITION BY department ORDER BY age) no, first_name, age, department FROM employees;
Result:
  no   first_name   age      department
  ----+------------+--------+-------------
  1    James        43       Development
  2    William      59       Development
  1    Mary         31       Research
  1    John         43       Sales
  2    Richard      (NULL)   Sales
  1    Lisa         29       (NULL)
```

#### LAG

|      |       |
|------|-------|
| Format | Format LAG( x \[, offset \[, default \] \] ) |

Returns x ahead of the current row by offset rows.

- Specify the value of arbitrary types as the argument x.
- For the argument offset, specify a LONG type value:   
  0 for the current position, a positive number for the forward position, a negative number for the backward position.   
  When no value is specified, offset will be 1.
- Specify the value of arbitrary types as the argument x.   
  If the expected row does not exist, default will be returned.   
  When no value is specified, default will be NULL.
- The type of the result is the same as that of the argument x.
- Specify the value of the same type for the arguments x and default. There are some different types that can be specified. Refer to [CASE](#case) for the allowed combination of types.

Example:
```example
SELECT id, date, empId, amount, LAG(amount) OVER(PARTITION BY empID ORDER BY id) as lag_amount FROM travelexpenses;
Result：
 id    date         empId   amount   lag_amount   
-----+------------+-------+--------+------------
  101  2020/02/01   0       200      (NULL)     
  104  2020/02/04   0       200      200        
  105  2020/02/05   0       150      200        
  102  2020/02/03   2       2500     (NULL)     
  103  2020/02/03   3       60       (NULL)     
  106  2020/02/06   3       80       60         
```

#### LEAD

|      |       |
|------|-------|
| Format | Format LEAD( x \[, offset \[, default \] \] ) |

Returns x behind the current row by offset rows.

- Specify the value of arbitrary types as the argument x.
- For the argument offset, specify a LONG type value:   
  0 for the current position, a positive number for the forward position, a negative number for the backward position.   
  When no value is specified, offset will be 1.
- Specify the value of arbitrary types as the argument x.   
  If the expected row does not exist, default will be returned.   
  When no value is specified, default will be NULL.
- The type of the result is the same as that of the argument x.
- Specify the value of the same type for the arguments x and default. There are some different types that can be specified. Refer to [CASE](#case) for the allowed combination of types.

Example:
```example
SELECT id, date, empId, amount, LEAD(amount) OVER(PARTITION BY empID ORDER BY id) as lead_amount FROM travelexpenses;
Result：
 id    date         empId   amount   lead_amount 
-----+------------+-------+--------+-------------
  101  2020/02/01   0       200      200         
  104  2020/02/04   0       200      150         
  105  2020/02/05   0       150      (NULL)      
  102  2020/02/03   2       2500     (NULL)      
  103  2020/02/03   3       60       80          
  106  2020/02/06   3       80       (NULL)      
```


<a id="other_function"></a>
### Other functions

#### COALESCE

|      |      |
|------|-------|
| Format | COALESCE(*x1*, *x2* \[,..., *xn*\]) |

Return the value of the first argument that is not NULL in the specified argument xn.

- Specify the same type value for the argument xn. There are some different types that can be specified. Refer to [CASE](#case) for the allowed combination of types.


- Return NULL, when all argument values are NULL.


Example:
```example
SELECT last_name, COALESCE(last_name, 'XXX') coalesce FROM employees;
Result:
  last_name     coalesce
  ------------+----------------------
  Smith         Smith
  Jones         Jones
  Brown         Brown
  Taylor        Taylor
  (NULL)        XXX
  Smith         Smith

SELECT age, COALESCE(age, -1) coalesce FROM employees;
Result:
  age       coalesce
  --------+-----------
  43         43
  59         59
  (NULL)     -1
  31         31
  29         29
  43         43
```



#### IFNULL

|      |      |
|------|-------|
| Format | IFNULL(*x*, *y*) |

Return the value of the first argument that is not NULL among the specified arguments x and y. The IFNULL function is equivalent to the COALESCE function with two arguments.

- Specify the value of the same type for the arguments x and y. There are some different types that can be specified. Refer to [CASE](#case) for the allowed combination of types.
- Return NULL, when all argument values are NULL.

Example:
```example
SELECT last_name, IFNULL(last_name, 'XXX') ifnull FROM employees;
Result：
  last_name     ifnull
  ------------+----------------------
  Smith         Smith
  Jones         Jones
  Brown         Brown
  Taylor        Taylor
  (NULL)        XXX
  Smith         Smith

SELECT age, IFNULL(age, -1) ifnull FROM employees;
Result：
  age       coalesce
  --------+-----------
  43         43
  59         59
  (NULL)     -1
  31         31
  29         29
  43         43
```



#### NULLIF

|      |      |
|------|-------|
| Format | NULLIF(*x*, *y*) |

Return NULL when two arguments are the same, return the first argument when the arguments are different.

- Specify the value of the same type for the arguments x and y. There are some different types that can be specified. Refer to [CASE](#case) for the allowed combination of types.

Example:
```example
// Execute NULLIF with the value of value1 and value2.
SELECT value1, value2, NULLIF(value1, value2) nullif FROM container_sample;
Result: 
   value1   value2   nullif
  --------+--------+--------
      10       10    (NULL)
       5        0      5
   (NULL)       4    (NULL)
       3    (NULL)     3
   (NULL)   (NULL)   (NULL)


// Convert 0 to NULL to prevent division by zero errors in the calculation of value1 / value2
SELECT value1, value2, value1/NULLIF(value2, 0) division FROM container_sample;
Result: 
   value1   value2   division
  --------+--------+--------
      10       10      1
       5        0    (NULL)
   (NULL)       4    (NULL)
       3    (NULL)   (NULL)
   (NULL)   (NULL)   (NULL)
```



#### RANDOMBLOB

|      |      |
|------|-------|
| Format | RANDOMBLOB(*size*) |

Return a BLOB type value (random number).

- Specify the size (number of bytes) of a BLOB type value as an integer for the argument size.
- The result is of a BLOB type.

Example:  
```example
// Generate a 10-byte blob value (random number)
SELECT HEX(RANDOMBLOB(10));
Result: 7C8C893C8087F07883AF
```



#### ZEROBLOB

|      |      |
|------|-------|
| Format | ZEROBLOB(*size*) |

Return a BLOB type value (0x00).

- Specify the size (number of bytes) of a BLOB type value as an integer for the argument size.
- The result is of a BLOB type.

Example:  
```example
// Generate a 10-byte blob value (0x00).
SELECT HEX(ZEROBLOB(10));
Result: 00000000000000000000
```



#### HEX

|      |      |
|------|-------|
| Format | HEX(*x*) |

Convert a BLOB type value to a hexadecimal type.
Interpret the argument x as a BLOB type value, and return the character string (uppercase) converted into the hexadecimal.

- Specify a BLOB type and a character string type for the argument x.
  - For a character string type argument, return the character string in which the Unicode code point of all the characters converted to hexadecimal.
- The result is of a character string type.

Example:
```example
SELECT HEX(RANDOMBLOB(2));
Result: E18D

SELECT first_name, HEX(first_name) hex FROM employees;
Result:
  first_name    hex
  ------------+----------------------
  John          4A6F686E
  William       57696C6C69616D
  Richard       52696368617264
  Mary          4D617279
  Lisa          4C697361
  James         4A616D6573
```



#### TYPEOF

|      |      |
|------|-------|
| Format | TYPEOF(*x*) |

Return the character string indicating the data type of the value of x.

- The correspondence between the data type and the string returned by the TYPEOF function is shown below.

  | Data types | Character string which TYPEOF function returns |
  |-------------------|-----------------------|
  | BOOL       | BOOL |
  | STRING     | STRING |
  | BYTE       | BYTE |
  | SHORT      | SHORT |
  | INTEGER    | INTEGER |
  | LONG       | LONG |
  | FLOAT      | FLOAT |
  | DOUBLE     | DOUBLE |
  | TIMESTAMP(3)       | TIMESTAMP |
  | TIMESTAMP(6)       | TIMESTAMP(6) |
  | TIMESTAMP(9)       | TIMESTAMP(9) |
  | GEOMETRY   | NULL |
  | BLOB       | BLOB |
  | ARRAY      | NULL |

- The result is of a character string type.
- When a NULL value is specified, 'NULL' is returned.

Example:
```example
SELECT TYPEOF(ABS(-10)) abs, TYPEOF(RANDOMBLOB(10)) randomblob,
    TYPEOF(TIMESTAMP('2018-12-01T10:30:02.392Z')) timestamp;
Result：
   abs    randomblob   timestamp
  ------+------------+-----------
   LONG   BLOB         TIMESTAMP
```

  

## Other syntaxes

### CAST

|      |      |
|------|-------|
| Format | CAST(*x* AS *data_type*) |

Convert the value x into the data type "data_type".

- Specify the following values for argument "data_type" according to the converted data type.

  | Converted data type | Value for data_type |
  |-------------------|-----------------------|
  | BOOL       | BOOL |
  | STRING     | STRING |
  | BYTE       | BYTE |
  | SHORT      | SHORT |
  | INTEGER    | INTEGER |
  | LONG       | LONG |
  | FLOAT      | FLOAT |
  | DOUBLE     | DOUBLE |
  | TIMESTAMP(3)       | TIMESTAMP or TIMESTAMP(3) |
  | TIMESTAMP(6)       | TIMESTAMP(6) |
  | TIMESTAMP(9)       | TIMESTAMP(9) |
  | BLOB       | BLOB |


#### Convert to string type

|      |      |
|------|-------|
| Format | CAST(*x* AS STRING) |

Convert the argument x to a character string type.

The data types of the value which can be specified for x, and the converted values are as follows.

| Data type of x | Value converted to character string type |
|-------------------|--------------------|
| BOOL | 'true' if true, 'false' if false |
| STRING | Original value   |
| BYTE<br>SHORT<br>INTEGER<br>LONG<br>FLOAT<br>DOUBLE | Value converted from a number to a character string   |
| TIMESTAMP(3)|String notation of millisecond-precision time 'YYYY-MM-DDThh:mm:ss.SSS(Z| ±hh:mm)'<br>The time zone setting at the time of connection is used.|
| TIMESTAMP(6)|String notation of microsecond-precision time 'YYYY-MM-DDThh:mm:ss.SSSSSS(Z| ±hh:mm)'<br>The time zone setting at the time of connection is used.|
| TIMESTAMP(9)| String notation of nanosecond-precision time 'YYYY-MM-DDThh:mm:ss.SSSSSSSSS(Z| ±hh:mm)'<br>The time zone setting at the time of connection is used.|
| BLOB | A character string equivalent to the converted character string using [HEX function](#hex)   |


#### Convert to numeric type

|      |      |
|------|-------|
| Format | CAST(*x* AS BYTE\|SHORT\|INTEGER\|LONG\|FLOAT\|DOUBLE) |

Convert the argument x into a numeric type.

The data types of the value which can be specified for x, and the converted values are as follows.

| Data type of x | Value converted to numeric type |
|-------------------|--------------------|
| BOOL | 1 if true, 0 if false |
| STRING | The value converted from the character string to numerical value  |
| BYTE<br>SHORT<br>INTEGER<br>LONG<br>FLOAT<br>DOUBLE | The numerical value converted to the specified numeric type   |

- An error will occur if the converted number exceeds the range of numeric values specified in data_type.

```example
// An error occurs if exceeding BYTE type range (-128 to 127)
SELECT CAST(128 AS BYTE);
Result: error

// An error occurs if exceeding INTEGER type range (-2147483648-2147483647).
SELECT CAST('2147483648' AS INTEGER);
Result: error
```

- When converted from floating-point type (FLOAT, DOUBLE) to integer type (BYTE, SHORT, INTEGER, LONG), the number of significant digits in the result may be reduced.

```example
SELECT CAST(10.5 AS INTEGER);
Result: 10
```

- The following character strings can be specified in the conversion from a character string type to a numeric type (case insensitive). An error will occur when character strings other than these are specified.
  - The character string containing a number, a sign (". "," - ", "+"), or "E"
  - "Inf" (signed data acceptable)
  - "Infinity" (signed data acceptable)
  - "NaN"

```example
SELECT CAST('abc' AS INTEGER);
Result: error

SELECT CAST('-1.09E+10' AS DOUBLE);
Result: -1.09E10
```


#### Convert to time type

|      |      |
|------|-------|
| Format | CAST(*x* AS TIMESTAMP) |

Convert the argument x to a time type. If the time zone is specified at the time of connection, that value is used for offset calculation.
TIMESTAMP with specified precision can also be specified.

The data types of the value which can be specified for x, and the converted values are as follows.

| Data type of x                                                        | Value converted to time type                             |
|----------------------------------------------------|-----------------------------------------------|
| STRING (string notation of millisecond-precision time 'YYYY-MM-DDThh:mm:ss.SSS(Z\|±hh:mm))| Equivalent to the value converted using the [TIMESTAMP_MS function](#timestamp_ms)　(can be converted regardless of which precision is specified) |
| STRING (string notation of microsecond-precision time 'YYYY-MM-DDThh:mm:ss.SSSSSS(Z\|±hh:mm))| Equivalent to the value converted using the [TIMESTAMP_US function](#timestamp_us)　(Conversion is only possible when microsecond or nanosecond precision is specified.)) |
| STRING (string notation of nanosecond-precision time 'YYYY-MM-DDThh:mm:ss.SSSSSSSSS(Z\|±hh:mm))| Equivalent to the value converted using the [TIMESTAMP_NS function](#timestamp_ns)　(Conversion is only possible when nanosecond precision is specified.)) |

```example
SELECT CAST('2018-12-01T10:30:00Z' AS TIMESTAMP);
Result: 2018-12-01T10:30:00.000Z

SELECT CAST('2018-12-01T10:30:00+09:00' AS TIMESTAMP);
Result: 2018-12-01T01:30:00.000Z
```


#### Convert to BOOL type

|      |      |
|------|-------|
| Format | CAST(*x* AS BOOL) |

Convert the argument x to a BOOL type.

The data types of the value which can be specified for x, and the converted values are as follows.

| Data type of x | Value converted to time type |
|-------------------|--------------------|
| STRING | True if 'true', false if 'false' (case insensitive)   |
| BYTE<br>SHORT<br>INTEGER<br>LONG | False if 0, otherwise true |


#### Convert to BLOB type

|      |      |
|------|-------|
| Format | CAST(*x* AS BLOB) |

Convert the argument x to a BLOB type.

The data types of the value which can be specified for x, and the converted values are as follows.

| Data type of x | Value converted to BLOB type |
|-------------------|--------------------|
| STRING         | The value converted from character string as hexadecimal data to BLOB type   |

  

### CASE

|      |      |
|------|-------|
| Format | CASE <br>WHEN *condition1* THEN *result1* <br>[WHEN *condition2* THEN *result2*]<br>... <br>[ELSE *resultElse*] <br>END |

When the conditional expression conditionN is true, the value of corresponding resultN is returned.
When all the conditional expressions are false or NULL, and if ELSE is specified, the value of resultElse will be returned. When ELSE is not specified, NULL is returned.

  

|      |      |
|------|-------|
| Format | CASE x <br>WHEN *value1* THEN *result1* <br>[WHEN *value2* THEN *result2*]<br>... <br>[ELSE *resultElse*] <br>END |

When the value of x is valueN, the value of corresponding resultN is returned.
When the value of x is not equal to all values, and if ELSE is specified, the value of resultElse will be returned. When ELSE is not specified, NULL is returned.

  

Specify the same type value for resultN. There are some different types that can be specified.
- If the arguments are of different types, only the combination of the following types can be calculated. Any other combinations will result in an error.

  | Type of argument | Type of argument                  | Type of argument when calculating the two arguments |
  |- | - | -|
  | SHORT            | BYTE                              | LONG |
  | INTEGER          | BYTE, SHORT                       | LONG |
  | LONG             | BYTE, SHORT, INTEGER              | LONG |
  | FLOAT            | BYTE, SHORT, INTEGER, LONG        | DOUBLE |
  | DOUBLE           | BYTE, SHORT, INTEGER, LONG, FLOAT | DOUBLE |


Example:
```example
// Display the employee's age (30's, 40's, 50's, other than these)
SELECT id, first_name, age,
  CASE
    WHEN age > 50 THEN '50s'
    WHEN age > 40 THEN '40s'
    WHEN age > 30 THEN '30s'
    ELSE 'other'
  END AS period
FROM employees;

Result: 
  id   first_name   age     period
  ----+------------+-------+--------
  0    John          43      40s
  1    William       59      50s
  2    Richard      (NULL)   other
  3    Mary          31      30s
  4    Lisa          29      other
  5    James         43      40s


// Display a location according to their departments.
SELECT id, first_name, department,
  CASE department
    WHEN 'Sales' THEN 'Tokyo'
    WHEN 'Development' THEN 'Osaka'
    ELSE 'Nagoya'
  END AS location
FROM employees;

Result: 
  id   first_name   department    location
  ----+------------+-------------+---------
  0    John         Sales         Tokyo
  1    William      Development   Osaka
  2    Richard      Sales         Tokyo
  3    Mary         Research      Nagoya
  4    Lisa         (NULL)        Nagoya
  5    James        Development   Osaka
```

<a id="sub_query"></a>
### Subquery

Subqueries can be specified in various parts of an SQL statemnt other than FROM and WHERE clauses.
Some operation types for subqueries are also provided, which are explained in this section.


<a id="in_sub_query"></a>
#### IN

Return whether the specified value is included in the sub query execution result.

**Syntax**

|                                   |
|-----------------------------------|
| Expression 1 \[NOT\] IN ( sub_query ) |

**Specifications**

- Return true when the value of expression_1 is included in the result of the sub query.
- The result of a sub query must be data of one row.

Example:
```example
// Display the information of the employee who belongs to the department of id=1 in the departments table from the employees table.
SELECT * FROM employees
WHERE department IN(
  SELECT department FROM departments
  WHERE id = 1
);
Result: 
  id   first_name   last_name   age     department    enrollment_period
  ----+------------+-----------+-------+-------------+-------------------
   1   William      Jones       59      Development   23.2
   5   James        Smith       43      Development   10.3
```

#### EXISTS

Return whether the execution result of the sub query exists.

**Syntax**

|      |
|-------|
| \[NOT\] EXISTS( *sub_query* ) |


**Specifications**

- Check whether the execution result of the sub query exists. Return true if the number of execution result is 1 or more, false if it is 0.

- The result is of a BOOL type.

Example:
```example
// Display the information of the employee who belongs to the department of id=1 in the departments table from the employees table.
SELECT * FROM employees
WHERE EXISTS(
   SELECT * FROM departments
   WHERE employees.department=departments.department AND departments.id=1
);
結果：
  id   first_name   last_name   age     department    enrollment_period
  ----+------------+-----------+-------+-------------+-------------------
   1   William      Jones       59      Development   23.2
   5   James        Smith       43      Development   10.3
```

#### Scalar sub query

Subquery which returns one result, which can be used for the result of a SELECT statement or for an expression.

Example:
```example
SELECT id, first_name,
       (SELECT department FROM departments WHERE department_id=employees.department_id)
FROM employees;

Result: 
  id  first_name  department
  ---+-----------+-------------
   0  John        Sales
   1  William     Development
   2  Richard     Sales
   3  Mary        (NULL)
   4  Lisa        Marketing
   5  James       Development
```

### Placeholder

A prepared statement can describe a placeholder in SQL statements.
A placeholder indicates the position of the parameter to be substituted when the statement is executed.
The parameter number starts from 1.

The placeholder can use several forms for compatibility with other databases.
However, the parameter number will be the already assigned parameter number + 1, regardless of which format is specified.


| Format | Description                        | Example of description |
|--------|--------|--------|
| ?      | Format of a standard placeholder   | ? |
| ?NNN   | NNN indicates a number.            | ?56 |
| :AAAA  | AAAA indicates a character string. | :name |
| @AAAA  | AAAA indicates a character string. | @name |

The placeholder must not start with $.

Example:
```java
String sql = "SELECT * FROM users WHERE id > ? AND id != :exclude_id;";
PreparedStatement pstmt = con.prepareStatement(sql);
pstmt.setInt(1, 100);  // 1: ?
pstmt.setInt(2, 253);  // 2: :exclude_id
ResultSet rs = pstmt.executeQuery();
```

<a id="comment"></a>
## Comment

Comments can be written in a SQL command.  Format: Description at the back of -- (2 hyphens) or enclose with /\* \*/.
A new line needs to be returned at the end of the comment.

```example
SELECT * -- comment
FROM employees;

SELECT *
/*
  comment
*/
FROM employees;
```



<a id="hint"></a>
## Hints

In GridDB, specifying the hints indicating the execution plan in the query makes it possible to control the execution plan without changing the SQL statement.

See [GridDB SQL Tuning Guide](GridDB_SQL_TuningGuide.md) to tune with hint clauses.



### Error handling

In the following cases, a syntax error occurs.
- Multiple block comments for hints are described
- The hint is described in the wrong position
- There is a syntax error in the description of the hint phrase
- Duplicate hint of the same class are specified for the same table

In the following case, a table specification error occurs:
- The table specification of the hint phrase is incorrect

[Memo]
- When a table specification error occurs, ignore the error hint phrase and execute the query using the others.
- When a syntax error and a table specification error occur at the same time, a syntax error occurs.


<a id="label_metatables"></a>
# Metatables


## About metatables

The metatables are tables that are used for checking metadata of data management in GridDB.

[Memo]
- Metatables can only be referred. It is not allowed to register or delete data in the metatables.
- When [SELECT](#select) data from the metatables, it is necessary to enclose the table name with double quotation marks.
- In the case of typical metatables, information on the database that the user is connecting to is the only information that can be obtained, regardless of the user's authority.
- However, for some metatables (statistical metatables), a user with an authority as a database administrator is allowed to obtain information on all the databases to which the user is not connecting as well.

[Points to note]
- The schema of metatables may be changed in future version.


## Table information

Table information can be obtained.

**Table name**

\#tables

**Schema**

| Column name | Item | Type      |
|----------------------------|-----------------------------------------------------|---------|
| DATABASE_NAME | Database name | STRING  |
| TABLE_NAME | Table name | STRING  |
| TABLE_OPTIONAL_TYPE | Table type<br>COLLECTION / TIMESERIES | STRING  |
| DATA_AFFINITY | Data affinity | STRING  |
| EXPIRATION_TIME | Expiry release elapsed time | INTEGER |
| EXPIRATION_TIME_UNIT | Expiry release elapsed time unit | STRING  |
| EXPIRATION_DIVISION_COUNT | Expiry release division count | INTEGER  |
| PARTITION_TYPE | Partitioning type | STRING  |
| PARTITION_COLUMN | Partitioning key | STRING  |
| PARTITION_INTERVAL_VALUE | Interval value (For interval or interval hash) | STRING |
| PARTITION_INTERVAL_UNIT | Interval unit (For interval of interval hash) | STRING  |
| PARTITION_DIVISION_COUNT | Division count (For hash) | INTEGER |
| SUBPARTITION_TYPE | Partitioning type<br>("Hash" for interval hash) | STRING  |
| SUBPARTITION_COLUMN | Partitioning key<br>(for interval hash) | STRING  |
| SUBPARTITION_INTERVAL_VALUE | Interval value | STRING |
| SUBPARTITION_INTERVAL_UNIT | Interval unit | STRING  |
| SUBPARTITION_DIVISION_COUNT | Division count<br>(For interval hash) | INTEGER |
| EXPIRATION_TYPE | Expiration type <br>PARTITION | STRING  |


## Index information

Index information can be obtained.

**Table name**

\#index_info

**Schema**

| Column name | Item | Type     |
|------------------|------------------------------|--------|
| DATABASE_NAME | Database name | STRING |
| TABLE_NAME | Table name | STRING |
| INDEX_NAME | Index name | STRING |
| INDEX_TYPE | Index type <br>TREE / SPATIAL | STRING |
| ORDINAL_POSITION | Column order in index (sequential number from 1) | SHORT  |
| COLUMN_NAME | Column name | STRING |


## Partitioning information

Data about partitioned tables can be obtained from this metatable.

**Table name**

\#table_partitions

**Schema**

| Column name | Item | Type     |
|-------------------------|---------------------------------------|--------|
| DATABASE_NAME | Database name | STRING |
| TABLE_NAME                | Partitioned table name                       | STRING |
| PARTITION_BOUNDARY_VALUE | The lower limit value of each data partition | STRING |
| CLUSTER_PARTITION_INDEX | Cluster partition number            | INTEGER |
| CLUSTER_NODE_ADDRESS | Node address:port number            | STRING |
| WORKER_INDEX | Processing thread number            | INTEGER |

**Specifications**

- Each row represents the information of a data partition.
  - For example, when searching rows of a hash partitioned table in which the division count is 128, the number of rows displayed will be 128.
- In the metatable "#table_partitions", the other columns may be displayed besides the above columns.
- It is required to cast the lower limit value to the partitioning key type for sorting by the lower limit value.
- Information as to where each interval is placed can be obtained from "CLUSTER_PARTITION_INDEX", "CLUSTER_NODE_ADDRESS", and "WORKER_INDEX".

**Examples**

- Check the number of data partitions

  ``` example
  SELECT COUNT(*) FROM "#table_partitions" WHERE TABLE_NAME='myIntervalPartition';

  COUNT(*)
  -----------------------------------
   8703
  ```

- Check the lower limit value of each data partition

  ``` example
  SELECT PARTITION_BOUNDARY_VALUE FROM "#table_partitions" WHERE TABLE_NAME='myIntervalPartition'
  ORDER BY PARTITION_BOUNDARY_VALUE;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
  2016-10-30T10:00:00Z
  2017-01-29T10:00:00Z
          :
  ```

- Check the lower limit value of each data partitions on the interval partitioned table "myIntervalPartition2" (partitioning key type: INTEGER, interval value: 20000)

  ``` example
  SELECT CAST(PARTITION_BOUNDARY_VALUE AS INTEGER) V FROM "#table_partitions"
  WHERE TABLE_NAME='myIntervalPartition2' ORDER BY V;

  PARTITION_BOUNDARY_VALUE
  -----------------------------------
  -5000
  15000
  35000
  55000
    :
  ```

## View information

View information can be obtained.

**Table name**

\#views

**Schema**

| Column name | Item | Type      |
|----------------------------|-----------------------------------------------------|---------|
| DATABASE_NAME | Database name | STRING  |
| VIEW_NAME       | View name                      | STRING  |
| VIEW_DEFINITION | View defining character string | STRING  |


## Information about a running SQL

Statistics about SQLs (queries or jobs) that are running can be obtained.

**Table name**

\#sqls

**Schema**

| Column name | Item | Type       |
|----------------------------|-----------------------------------------------------|----------|
| DATABASE_NAME | Database name | STRING   |
| NODE_ADDRESS     | address of the node being processed (system)  | STRING   |
| NODE_PORT        | The port of the node being processed (system) | INTEGER  |
| START_TIME       | Processing start time                         | TIMESTAMP|
| APPLICATION_NAME | Application name                              | STRING   |
| SQL               | Query character string                        | STRING   |
| QUERY_ID         | Query ID                                      | STRING   |
| JOB_ID           | Job ID                                        | STRING   |
| USER_NAME                  | User name                                            | STRING   |

[Memo]
  - Database administrators can obtain information across all databases.

## Information about a running execution

Statistics about events that are running can be obtained.

**Table name**

\#events

**Schema**

| Column name | Item | Type       |
|----------------------------|-----------------------------------------------------|----------|
| NODE_ADDRESS     | address of the node being processed (system)  | STRING   |
| NODE_PORT        | The port of the node being processed (system) | INTEGER  |
| START_TIME       | Processing start time                         | TIMESTAMP|
| APPLICATION_NAME | Application name                              | STRING   |
| SERVICE_TYPE             | Service type (SQL/TRANSACTION/CHECKPOINT/SYNC) | STRING   |
| EVENT_TYPE               | Event types (PUT/CP_START/SYNC_START etc.)   | STRING   |
| WORKER_INDEX               | Processing thread number                              | INTEGER  |
| CLUSTER_PARTITION_INDEX | Cluster partition number                       | INTEGER  |
| DATABASE_ID    | Database ID                          | LONG  |


## Connection information

Statistics about the connected connection can be obtained.

**Table name**

\#sockets

**Schema**

| Column name | Item | Type       |
|----------------------------|-----------------------------------------------------|----------|
| SERVICE_TYPE             | Service type (SQL/TRANSACTION)                            | STRING   |
| SOCKET_TYPE              | Socket type                                               | STRING   |
| NODE_ADDRESS             | Connection source node address (viewed from a node)       | STRING   |
| NODE_PORT                | Connection source node port (viewed from a node)          | INTEGER  |
| REMOTE_ADDRESS           | Connection destination node address (viewed from a node)  | STRING   |
| REMOTE_PORT              | Connection destination node port (viewed from a node)     | INTEGER  |
| APPLICATION_NAME | Application name                              | STRING   |
| CREATION_TIME              | Connection time                                   | TIMESTAMP|
| DISPATCHING_EVENT_COUNT | Total number of times to start request for event handling | LONG     |
| SENDING_EVENT_COUNT     | Total number of times to start event transmission         | LONG     |
| DATABASE_NAME        | Database name                        | LONG     |

The following three are available for SOCKET_TYPE (socket type) above:

|Value     | Description|
|- | -|
|SERVER    | TCP connection between servers       |
|CLIENT    | TCP connection with a client     |
|MULTICAST | Multicasting socket      |
|NULL      | In case currently unidentified during the cases such as connection attempt|

CREATION_TIME (connection time) in the table above is defined as follows according to each case:
- If SERVICE_TYPE is SQL, CREATION_TIME refers to the time a JDBC connection has started.
- If SERVICE_TYPE is TRANSACTION, CREATION_TIME refers to the time a NoSQL connection has started.


**Examples**

Only in case of TCP connection with a client (socket type: CLIENT), it can be determined whether the connection is waiting for execution.


Specifically, if DISPATCHING_EVENT_COUNT is larger than SENDING_EVENT_COUNT, it can be determine that the possibility is relatively high that the time waiting for execution existed.


``` example
SELECT CREATION_TIME, NODE_ADDRESS, NODE_PORT, APPLICATION_NAME FROM "#sockets"
WHERE SOCKET_TYPE='CLIENT' AND DISPATCHING_EVENT_COUNT > SENDING_EVENT_COUNT;

CREATION_TIME             NODE_ADDRESS   NODE_PORT  APPLICATION_NAME
--------------------------------------------------------------------
2019-03-27T11:30:57.147Z  192.168.56.71  20001      myapp
2019-03-27T11:36:37.352Z  192.168.56.71  20001      myapp
          :
```

## List of databases

A list of database names and the corresponding database IDs can be obtained.

**Table name**

\#databases

**Schema**

| Column name                       | Item                                                | Type      |
|----------------------------|-----------------------------------------------------|---------|
| DATABASE_NAME              | Database name                                      | STRING  |
| DATABASE_ID                  | Database ID                                            | INTEGER  |INTEGER  |

 ## Database statistics

Statistics aggregated for each database can be obtained.

**Table name**

\#database_stats

**Schema**

| Column name | Item | Type |
|------|-----------------------------------------------------|---------|
| DATABASE_ID | Database ID | LONG |
| NODE_ADDRESS | Node address | STRING |
| NODE_PORT | Node port number | INTEGER |
| TRANSACTION_CONNECTION_COUNT | Number of transaction connections| LONG |
| TRANSACTION_REQUEST_COUNT | Number of transaction requests | LONG |
| SQL_CONNECTION_COUNT | Number of SQL connections | LONG |
| SQL_REQUEST_COUNT | Number of SQL requests | LONG |
| STORE_BLOCK_SIZE | Size of data files used  | LONG |
| STORE_MEMORY_SIZE | Size of database buffers used  | LONG |
| STORE_SWAP_READ_SIZE | Size of data files swapped and read | LONG |
| STORE_SWAP_WRITE_SIZE | Size of data files swapped and written  | LONG |
| SQL_WORK_MEMORY_SIZE | Size of an SQL work memory buffer | LONG |
| SQL_STORE_USE_SIZE | Size of a buffer used for storing intermediate SQL results | LONG |
| SQL_STORE_SWAP_READ_SIZE | Size of swap files for intermediate SQL results swapped and read | LONG |
| SQL_STORE_SWAP_WRITE_SIZE | Size of swap files for intermediate SQL results swapped and written  | LONG |
| SQL_TASK_COUNT | Total number of running SQL tasks | LONG |
| SQL_PENDING_JOB_COUNT | Number of SQL jobs that are stopped because sending is suppressed | LONG |
| SQL_SEND_MESSAGE_SIZE | SQL sent message size | LONG |

[Memo]
 - Database administrators can obtain information across all databases.
 - The items in the above table are the same as the ones that are aggregated by the entire server, but the rest may vary.
 - To display the exact value for the item with an asterisk, additional settings are required in the node definition file.
 - For details about each statistical item and how to set it, see the [GridDB Features Reference](GridDB_FeaturesReference.md)


# Reserved words

The following terms are defined as keywords in the SQL of GridDB.

ABORT ACTION AFTER ALL ANALYZE AND AS ASC BEGIN BETWEEN BY CASE CAST COLLATE COLUMN COMMIT CONFLICT CREATE CROSS DATABASE DAY DELETE DESC DISTINCT DROP ELSE END ESCAPE EXCEPT EXCLUSIVE EXISTS EXPLAIN EXTRACT FALSE FOR FROM GLOB GRANT GROUP HASH HAVING HOUR IDENTIFIED IF IN INDEX INITIALLY INNER INSERT INSTEAD INTERSECT INTO IS ISNULL JOIN KEY LEFT LIKE LIMIT MATCH MILLISECOND MINUTE MONTH NATURAL NO NOT NOTNULL NULL OF OFFSET ON OR ORDER OUTER PARTITION PARTITIONS PASSWORD PLAN PRAGMA PRIMARY QUERY RAISE REGEXP RELEASE REPLACE RESTRICT REVOKE RIGHT ROLLBACK ROW SECOND SELECT SET TABLE THEN TIMESTAMPADD TIMESTAMPDIFF TO TRANSACTION TRUE UNION UPDATE USER USING VALUES VIEW VIRTUAL WHEN WHERE WITHOUT XOR YEAR

Copyright (c) 2017 TOSHIBA Digital Solutions Corporation