# GridDB JDBC Driver Manual



## table of contents
* [Overview](#Overview)
* [Specifications](#Specifications)
* [Sample](#Sample)

---
# Introduction

GridDB JDBC (Java interface) is an application interface that conforms to conforms to SQL92 standard specification.

This document describes the handling method and notes of the JDBC driver of GridDB Community Edition (hereinafter referred to as GridDB CE).

---


# Overview

This chapter consists of a description of the specified format and data types that can be used in a program using JDBC parameters, and the points to note during use.

    

## Connection method

### Driver specification

Add the JDBC driver file `/usr/share/java/gridstore-jdbc.jar` to the class path. When added, the driver will be registered automatically.  In addition, import the driver class as follows if necessary (Normally not required).

``` example
Class.forName("com.toshiba.mwcloud.gs.sql.Driver");
```
To secure communication between the GridDB cluster and the client, using the SSL function, additionally specify `/usr/share/java/gridstore-advanced.jar` in the class path.
  

### Connection URL format

The URL has the following forms (A) to (D). If the multicast method is used to compose a cluster, normally it is connected using method (A).  The load will be automatically distributed on the GridDB cluster side and the appropriate nodes will be connected.  Connect using other method only if multicast communication with the GridDB cluster is not possible.

**(A) If connecting automatically to a suitable node in a GridDB cluster using the multicast method**

``` example
jdbc:gs://(multicastAddress):(portNo)/(clusterName)/(databaseName)
```

-   multicastAddress: Multicast address used in connecting with a GridDB cluster. (Default: 239.0.0.1)
-   portNo: Port number used in connecting with a GridDB cluster. (Default: 41999)
-   clusterName: Cluster name of a GridDB cluster
-   databaseName: Database name. Connect to the default database (public) if omitted.

**(B) If connecting directly to a node in a GridDB cluster using the multicast method**

``` example
jdbc:gs://(nodeAddress):(portNo)/(clusterName)/(databaseName)
```

-   nodeAddress: Address of a node
-   portNo: Port number used in connecting with a node (Default: 20001)
-   clusterName: Cluster name of a GridDB cluster that a node belongs to
-   databaseName: Database name. Connect to the default database (public) if omitted.

**(C) If connecting to a GridDB cluster using the fixed list method**

If the fixed list method is used to compose a cluster, use this method to connect.

``` example
jdbc:gs:///(clusterName)/(databaseName)?notificationMember=(notificationMember)
```

-   clusterName: Cluster name of a GridDB cluster
-   databaseName: Database name. Connect to the default database (public) if omitted.
-   NotificationMember: Address list of nodes (URL encoding required). Default port is 20001.
    -   Example: 192.168.0.10:20001,192.168.0.11:20001,192.168.0.12:20001

\* notificationMember can be changed by editing the gs_cluster.json file.  The port in the address list can be changed by editing the gs_node.json file.

**(D) If connecting to a GridDB cluster using the provider method [EE only]**

If the provider method is used to compose a cluster, use this method to connect.

``` example
jdbc:gs:///(clusterName)/(databaseName)?notificationProvider=(notificationProvider)
```

-   clusterName: Cluster name of a GridDB cluster
-   databaseName: Database name. Connect to the default database (public) if omitted.
-   NotificationProvider: URL of the address provider (URL encoding required)

\* notificationProvider can be changed by editing the gs_cluster.json file.

If the user name and password are going to be included in the URL in either one of the cases (A) to (D), add them at the end of the URL as shown below.

``` example
?user=(user name)&password=(password)
```

  

### Connection timeout settings

The connection timeout can be set in either of the following methods (A) or (B). Setting (B) is prioritized if both (A) and (B) are set.  Default value of 300 seconds (5 minutes) is used if neither (1) or (2) has been set, or if there are no settings.

**(A) Specify with the DriverManager\#setLoginTimeout (int seconds)**

The connection timeout is set in one of the following ways depending on the value of seconds. After setting, the connection timeout will be set in the connections to all the GridDB acquired by the DriverManager#getConnection or Driver#connect.

-   If the value is from 1 to the value set in Integer.MAX_VALUE.
    -   The connection timeout is set to the specified number of seconds.
-   If the value is from the value set in Integer.MIN_VALUE to 0:
    -   The connection timeout is not set.

**(B) Specify with DriverManager\#getConnection(String url, Properties info) or Driver\#connect(String url, Properties info)**

Add a property to argument info in the key "loginTimeout". If the value corresponding to the key "loginTimeout" could be converted to a numerical value, the connection timeout will be set in the connection obtained as follows.

-   If the converted value is 1 to Integer.MAX_VALUE:
    -   The connection timeout is set to the specified number of seconds.
-   If the converted value is 0 or greater than Integer.MAX_VALUE:
    -   The connection timeout is not set.

  

### Settings of other information
Along with the settings described above, the following information can be set at the time of connection.
- Authentication method [EE only]
- Setting up SSL communication [EE only]
- External communication path
- Specifying the interface to receive the multicast packets from
- Application name
- Time zone (Z|±HH:MM|±HHMM)

* Since time zone processing burdens GridDB, processing the time zone on the application side is highly recommended.


The information above can be set in either of the following methods (A) or (B). An error occurs when the name is specified using both methods.


**(A) Specify in URL**

For authentication with the GridDB cluster, use internal authentication (INTERNAL) or LDAP authentication (LDAP) according to the cluster settings. To specify the authentication method during connection, add the following to the URL:

``` example
?authentication=(INTERNAL|LDAP)
```

Secure communication with the GridDB cluster through SSL. Three options are available: PREFERRED (follows the cluster settings), VERIFY (SSL is valid and performs server certificate validation), and DISABLED (SSL is invalid),  add the following to the URL:

``` example
?sslMode=(PREFERRED|VERIFY|DISABLED)
```

\*When LDAP authentication is used, disabling SSL communication is not recommended.

\* To specify VERIFY for the settings for SSL communications (sslMode), settings on the GridDB cluster side are also required. For detail, see the section on "Communication encryption" in the [GridDB Features Reference](GridDB_FeaturesReference.md). If a certificate by the Certificate Authority (CA) is not in a truststore, import it using the keytool command. If necessary, specify a truststore (-Djavax.net.ssl.trustStore) and a password (-Djavax.net.ssl.trustStorePassword) as arguments upon launching java. Note that the driver does not support the checking of the expiration date of a CA certificate to ensure it is valid.

To establish a connection using an external communication path, add the following to the URL.  This specification is enabled only when the value is set to "PUBLIC".

``` example
?connectionRoute=PUBLIC
```

Note that establishing a connection using an external communication path also requires settings on the GridDB cluster. For details, see the section on a "client communication path" in the [GridDB Features Reference](GridDB_FeaturesReference.md).


To configure the cluster network in multicast mode when multiple network interfaces are available, the address of the interface to receive the multicast packets from can be specified. Add the following to the URL:

``` example
?notificationNetworkInferfaceAddress=(address of the interface to receive  the multicast packets from)
```

To include the application name in the URL, add it to the end of the URL as follows:

``` example
?applicationName=(application name)
```

To include the time zone in the URL, add it to the end of the URL as follows:

``` example
?timeZone=(time zone)
```


To also include the user name and the password in the URL, use the following method.

``` example
?user=(user name)&password=(password)&applicationName=(application name)&timeZone=(time zone)
```

Encode the time zone symbol ":" and other characters that need to be encoded in the URL format.

**(B) Specify with DriverManager\#getConnection(String url, Properties info) or Driver\#connect(String url, Properties info)**

Add the property with the following key to the argument info.
- authentication method: authentication
    - INTERNAL (internal authentication) | LDAP (LDAP authentication)
- SSL communication: sslMode
    - PREFERRED (follows the cluster settings) | DISABLED (invalid)
- Communication path: ConnnectionRoute
    - PUBLIC (external communication path)
- address of the interface to receive the multicast packets from: notificationNetworkInterfaceAddress 
    - IP address
- Specify the application name: applicationName 
    - Application name
- Time zone: timeZone
    - time zone


## Points to note

- The difference between NoSQL interface/NewSQL interface
  - Containers created using the NoSQL interface client can be referenced and updated using the JDBC driver of NewSQL interface. Besides updating the rows, changes in the schema and index of a container are also included in an update.
  - Tables created using the JDBC driver of NewSQL interface can be referenced and updated by the clients of NoSQL interface.
  - A "container" for NoSQL interface and a "table" for NewSQL interface, both refer to the same object, but with different names.
  - If a time series container created by a client of NoSQL interface is searched with a SQL command from the JDBC driver, the results will not be in chronological order if no ORDER BY phrase is specified for the main key, unlike the search result conducted with a TQL command from a client of NoSQL. Specify an ORDER BY against the main key if a chronological series of the SQL results is required.
- For consistency, regardless of interface differences, READ COMMITTED is supported as a transaction isolation level.
  The search and update results using the NewSQL interface are not necessarily based on a single snapshot when starting a TQL command, as in the case of executing TQL with the partial execution option enabled using the NoSQL interface.
  The results may be based on the snapshot at each execution point divided according to the data range to be executed.
  This characteristic is different from that of the operation for a single unpartitioned table using the NoSQL interface with the default setting.

  

# Specifications

The specifications of the GridDB JDBC driver are shown in this chapter. The chapter explains mainly the support range of the driver as well as the differences with the JDBC standard.  See the JDK API reference for the API specifications that conform to the JDBC standard unless otherwise stated. Please note that the following could be revised in the future versions.

-   Actions not conforming to the JDBC standard
-   Support status of unsupported functions
-   Error messages

  

## Common items

### Supported JDBC version

The following functions corresponding to some of the functions of JDBC4.1 are not supported.

-   Transaction control
-   Stored procedure
-   Batch execution (statement interface)


### Error processing

#### Use of unsupported functions

-   Standard functions
    -   A SQLFeatureNotSupportedException occurs if a function that ought to be but is currently not supported by a driver conforming to the JDBC specifications is used. This action differs from the original SQLFeatureNotSupportedException specifications. The error name (to be described later) is JDBC_NOT_SUPPORTED.
-   Optional functions
    -   If a function not supported by this driver that is positioned as an optional function in the JDBC specifications and for which a SQLFeatureNotSupportedException may occur is used, a SQLFeatureNotSupportedException will occur as per the JDBC specifications. The error name is JDBC_OPTIONAL_FEATURE_NOT_SUPPORTED.

#### Invoke a method against a closed object

As per the JDBC specifications, when a method other than isClosed() is invoked for an object that has a close() method, e.g,. a connection object, etc., a SQLException will occur.  The error name is JDBC_ALREADY_CLOSED.

#### Invalid null argument

If null is specified as the API method argument despite not being permitted, SQLException due to a JDBC_EMPTY_PARAMETER error will occur. Null is not permitted except for arguments which explicitly accepts null in the JDBC specifications or this guide.

#### If there are multiple error causes

If there are multiple error causes, control will be returned to the application at the point either one of the errors is detected.
In particular, if use of an unsupported function is attempted, it will be detected earlier than other errors.
For example, if there is an attempt to create a stored procedure for a closed connection object, an error indicating that the operation is "not supported" instead of "closed" will be returned.

#### Description of exception

A check exception thrown from the driver is made up of a SQLException or a subclass instance of the SQLException.  Use the following method to get the exception details.

- getErrorCode()
  - For errors detected by GridDB in either the server or client, an error number will be returned.
- getSQLState()
  - At least for errors detected in the driver (error code: 14xxxx), non-null values will be returned. In other cases, undefined.
- getMessage()
  - Return an error message containing the error number and error description as a set. The format is as follows.

    ``` example
    [(Error number):( error name)] (error description)
    ```

  - When the error number is not on the list, the following error message format will be used instead:

    ``` example
    (Error Details)
    ```

#### Error list

The list of main errors detected inside the driver is as follows.

| Error no. | Error code name                      | Error description format                                                                              |
|------------|-------------------------------------|-----------------------------------------------------------------------------------------------|
| (See "GridDB error code")     | JDBC_NOT_SUPPORTED                  | Currently not supported                                                                       |
| (See "GridDB error code")     | JDBC_OPTIONAL_FEATURE_NOT_SUPPORTED | Optional feature not supported                                                                |
| (See "GridDB error code")     | JDBC_EMPTY_PARAMETER                | The parameter (argument name) must not be null                                                       |
| (See "GridDB error code")     | JDBC_ALREADY_CLOSED                 | Already closed                                                                                |
| (See "GridDB error code")     | JDBC_COLUMN_INDEX_OUT_OF_RANGE      | Column index out of range                                                                     |
| (See "GridDB error code")     | JDBC_VALUE_TYPE_CONVERSION_FAILED   | Failed to convert value type                                                                  |
| (See "GridDB error code")     | JDBC_UNWRAPPING_NOT_SUPPORTED       | Unwrapping interface not supported                                                            |
| (See "GridDB error code")     | JDBC_ILLEGAL_PARAMETER              | Illegal value: (argument name)                                                                       |
| (See "GridDB error code")     | JDBC_UNSUPPORTED_PARAMETER_VALUE    | Unsupported (parameter name)                                                                    |
| (See "GridDB error code")     | JDBC_ILLEGAL_STATE                  | Protocol error occurred                                                                       |
| (See "GridDB error code")     | JDBC_INVALID_CURSOR_POSITION        | Invalid cursor position                                                                       |
| (See "GridDB error code")     | JDBC_STATEMENT_CATEGORY_UNMATCHED   | Writable query specified for read only request Read only query specified for writable request |
| (See "GridDB error code")     | JDBC_MESSAGE_CORRUPTED              | Protocol error                                                                                |

When there is an error in the source generating the error and so on, additional details may be added to the end of the error description mentioned above.
  


## API detailed specifications

### Connection interface

Describes each method of the connection interface.  Unless otherwise stated, only the description for the case when connection has not been closed is included.

#### Transaction control

As for transaction control, which operates in automatic commitment mode, commit/rollback is not supported.
Note that a request for a commitment or a rollback from applications which use transactions is ignored so that the transaction control may be available even for these applications.
SQLFeatureNotSupportedException does not occur.

Transaction isolation level supports only TRANSACTION_READ_COMMITTED. Other levels cannot be set.

**Methods that have differences with the JDBC specification**

| Method | Description | Difference with JDBC specification |
|-----|----|----|
| void commit()                           | Commit             | Ignore a commit request because the API has only an automatic commitment mode.|
| void rollback()                         | Rollback         | Ignore a rollback request because the API has only an automatic commitment mode.|
| void setAutoCommit(boolean autoCommit)  | Set a commitment mode.  | Mode setting is unavailable because the API has only an automatic commitment mode. Setting autoCommit is ignored and true is always set.|


**Partially unsupported method**

| Method | Description | Unsupported feature |
|-----|----|----|
| Statement createStatement(int resultSetType, int resultSetConcurrency) | Create a statement.  | resultSetType supports ResultSet.TYPE_FORWARD_ONLY only, and resultSetConcurrency supports ResultSet.CONCUR_READ_ONLY only. For other values, SQLFeatureNotSupportedException will occur. |
| Statement createStatement(int resultSetType, int resultSetConcurrency, int resultSetHoldability) | Create a statement.  | resultSetType supports ResultSet.TYPE_FORWARD_ONLY only, resultSetConcurrency supports ResultSet.CONCUR_READ_ONLY only, and resultSetHoldability supports ResultSet.CLOSE_CURSORS_AT_COMMIT only. For other values, SQLFeatureNotSupportedException will occur. |
| PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) | Create a prepared statement.  | resultSetType supports ResultSet.TYPE_FORWARD_ONLY only, and resultSetConcurrency supports ResultSet.CONCUR_READ_ONLY only. For other values, SQLFeatureNotSupportedException will occur.|
| PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency) | Create a prepared statement. | resultSetType supports ResultSet.TYPE_FORWARD_ONLY only, resultSetConcurrency supports ResultSet.CONCUR_READ_ONLY only, and resultSetHoldability supports ResultSet.CLOSE_CURSORS_AT_COMMIT only. For other values, SQLFeatureNotSupportedException will occur.|
| void setTransactionIsolation(int level) | Set a transaction isolation level.  | Argument "level" accepts only Connection.TRANSACTION_READ_COMMITTED. If any other value is set, SQLException will occur.|


**Supported method**

| Method| Description |
|---------|------|
| void close()                   | Close connection.     |
| Statement createStatement()    | Create a statement.      |
| boolean getAutoCommit()        | Get commitment mode.      |
| DatabaseMetaData getMetaData() | Get DatabaseMetaData.  |
| int getTransactionIsolation()  | Get the transaction isolation level. |
| boolean isClosed()             | Get whether the Connection is closed. |
| PreparedStatement prepareStatement(String sql) | Create a prepared statement. |



#### Setting and getting attributes

This section describes methods for setting and getting attributes other than transaction control methods.

**Methods that have differences with the JDBC specification**

| Method | Description | Difference with JDBC specification |
|-----|----|----|
| void setReadOnly(boolean readOnly) | Sets the read-only mode of the Connection object.  | Ignore readOnly and always set false.|


**Partially unsupported method**

| Method | Description | Unsupported feature |
|-----|----|----|
| void setHoldability(int holdability) | Set the holding function of ResultSet object.  | Argument "holdability" accepts only ResultSet.CLOSE_CURSORS_AT_COMMIT. For other values, SQLFeatureNotSupportedException will occur. |


**Supported method**

| Method| Description |
|---------|------|
| int getHoldability()         | Get the holding function of the ResultSet object. |
| boolean isReadOnly()         | Get whether the Connection object is in read-only mode.  |
| boolean isValid(int timeout) | Get the state of connection. |


#### Unsupported function

Unsupported methods in the connection interface are listed below. When these methods are executed, SQLFeatureNotSupportedException will occur.

- Standard functions
  - CallableStatement prepareCall(String sql)

- Optional functions
  - void abort(Executor executor)
  - Array createArrayOf(String typeName, Object[] elements)
  - Blob createBlob()
  - Clob createClob()
  - NClob createNClob()
  - SQLXML createSQLXML()
  - Struct createStruct(String typeName, Object[] attributes)
  - int getNetworkTimeout()
  - String getSchema()
  - Map<String,Class<?>> getTypeMap()
  - CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency)
  - CallableStatement prepareCall(String sql, int resultSetType, int resultSetConcurrency, int resultSetHoldability)
  - PreparedStatement prepareStatement(String sql, int autoGeneratedKeys)
  - PreparedStatement prepareStatement(String sql, int[] columnIndexes)
  - PreparedStatement prepareStatement(String sql, String[] columnNames)
  - void releaseSavepoint(Savepoint savepoint)
  - void rollback(Savepoint savepoint)
  - void setNetworkTimeout(Executor executor, int milliseconds)
  - void setSavepoint()
  - void setSchema(String schema)
  - void setTypeMap(Map<String,Class<?>> map)

  

### DatabaseMetaData interface

This section describes DatabaseMetaData interface, which gets the metadata of a table.

#### Attribute that returns ResultSet

Among the methods that return ResultSet as the execution result in DatabaseMetaData interface, the supported methods are as follows.
The methods that return ResultSets other than these are not supported.

| Method| Description |
|--------|----|
| ResultSet getColumns(String catalog, String schemaPattern, String tableNamePattern, String columnNamePattern) | Return the column information of a table. |
| ResultSet getIndexInfo(String catalog, String schema, String table, boolean unique, boolean approximate) |Return the index information of a table. |
| ResultSet getPrimaryKeys(String catalog, String schema, String table) | Return the row key information of a table. |
| ResultSet getTables(String catalog, String schemaPattern, String tableNamePattern, String[] types) | Return the list of tables. |
| ResultSet getTableTypes() | Return the type of a table. |
| ResultSet getTypeInfo() | Return the list of column data types. |


Each of the above methods is explained below.


##### DatabaseMetaData.getColumns
```java
ResultSet getColumns(String catalog, String schemaPattern, String tableNamePattern, String columnNamePattern)
```
- Return the column information of the table that matches the pattern of the specified table name "tableNamePattern". The wildcard "%" specified in the pattern means to match 0 or more characters, and "_" means to match any one character. When null is specified as tableNamePattern, all tables are targeted.
- Other filter conditions catalog, schemaPattern, and columnNamePattern are ignored.
- The column information of a view is not included.
- The columns of execution result "ResultSet" are as follows.

  | Column name            | Data type     | Value                                   |
  |--------------------|--------|--------------------------------------|
  | TABLE_CAT          | String | null                                 |
  | TABLE_SCHEM        | String | null                                 |
  | TABLE_NAME         | String | table name                            |
  | COLUMN_NAME        | String | column  name                              |
  | DATA_TYPE          | int    | Data type value of a column (see the table below)    |
  | TYPE_NAME          | String | Data type name of a column (see the table below)  |
  | COLUMN_SIZE        | int    | 131072                               |
  | BUFFER_LENGTH      | int    | 2000000000                           |
   | DECIMAL_DIGITS     | int    | For TIMESTAMP, specify the number of digits in the fractional seconds to match the target precision  (3 for millisecond precision, 6 for microsecond precision, and 9 for nanosecond precision); for other types, specify null. |
  | NUM_PREC_RADIX     | int    | 10                                   |
  | NULLABLE           | int    | If the column is PRIMARY KEY or has NOT NULL constraint, 0 (the value of a constant "DatabaseMetaData.columnNoNulls"); otherwise, 1 (the value of a constant "DatabaseMetaData.columnNullable") |
  | REMARKS            | String | null                                 |
  | COLUMN_DEF         | String | null                                 |
  | SQL_DATA_TYPE      | int    | 0                                    |
  | SQL_DATETIME_SUB   | int    | 0                                    |
  | CHAR_OCTET_LENGTH  | int    |  For TIMESTAMP, specify the maximum string length to match the target precision  (30 for millisecond precision, 33 for microsecond precision, and 36 for nanosecond precision); for other types, specify 2000000000.                           |
  | ORDINAL_POSITION   | int    | The number of a column (serial number from 1)              |
  | IS_NULLABLE        | String | NOT NULL constraint. If the column is PRIMARY KEY or has NOT NULL constraint, 'NO'; otherwise, 'YES' |
  | SCOPE_CATALOG      | String | null                                 |
  | SCOPE_SCHEMA       | String | null                                 |
  | SCOPE_TABLE        | String | null                                 |
  | SOURCE_DATA_TYPE   | short  | 0                                    |
  | IS_AUTOINCREMENT   | String | 'NO'                                 |
  | IS_GENERATEDCOLUMN | String | 'NO'                                 |

- Return the combination of TYPE_NAME and DATA_TYPE values according to the data type of each column.

  | Data type of a column    | Value of TYPE_NAME       | Value of DATA_TYPE        |
  |--------------------|---------------------|----------------------|
  | BOOL            | 'BOOL'              | -7 (Types.BIT)       |
  | STRING           | 'STRING'            | 12 (Types.VARCHAR)   |
  | BYTE             | 'BYTE'              | -6 (Types.TINYINT)   |
  | SHORT            | 'SHORT'             | 5 (Types.SMALLINT)   |
  | INTEGER          | 'INTEGER'           | 4 (Types.INTEGER)    |
  | LONG             | 'LONG'              | -5 (Types.BIGINT)    |
  | FLOAT            | 'FLOAT'             | 6 (Types.FLOAT)      |
  | DOUBLE           | 'DOUBLE'            | 8 (Types.DOUBLE)     |
  | TIMESTAMP        | 'TIMESTAMP'         | 93 (Types.TIMESTAMP) |
  | BLOB             | 'BLOB'              | 2004 (Types.BLOB)    |
  | GEOMETRY         | 'GEOMETRY'          | 1111 (Types.OTHER)   |
  | BOOL ARRAY      | 'BOOL_ARRAY'        | 1111 (Types.OTHER)   |
  | STRING ARRAY    | 'STRING_ARRAY'      | 1111 (Types.OTHER)   |
  | BYTE ARRAY      | 'BYTE_ARRAY'        | 1111 (Types.OTHER)   |
  | SHORT ARRAY     | 'SHORT_ARRAY'       | 1111 (Types.OTHER)   |
  | INTEGER ARRAY   | 'INTEGER_ARRAY'     | 1111 (Types.OTHER)   |
  | LONG ARRAY      | 'LONG_ARRAY'        | 1111 (Types.OTHER)   |
  | FLOAT ARRAY     | 'FLOAT_ARRAY'       | 1111 (Types.OTHER)   |
  | DOUBLE ARRAY    | 'DOUBLE_ARRAY'      | 1111 (Types.OTHER)   |
  | TIMESTAMP ARRAY | 'TIMESTAMP_ARRAY'   | 1111 (Types.OTHER)   |

- For GEOMETRY type and array type, the value will be returned if information of the table with these data types created by NoSQL interface is acquired. JDBC cannot create tables with these data types.

  

##### DatabaseMetaData.getIndexInfo

```java
ResultSet getIndexInfo(String catalog, String schema, String table, boolean unique, boolean approximate)
```

- Returns index information for the table that matches the specified table name "table". When the table of the specified name does not exist, null will be returned for the execution result "ResultSet".
- Specify false for "unique". When the value for unique is other than false, null will be returned for the execution result "ResultSet".
- Other filter conditions catalog, schema, and a parameter "approximate" are ignored.
- The columns of execution result "ResultSet" are as follows.

  | Column name          | Data type      | Value               |
  |------------------|---------|------------------|
  | TABLE_CAT          | String | null             |
  | TABLE_SCHEM        | String | null             |
  | TABLE_NAME         | String | table name        |
  | NON_UNIQUE       | boolean | true             |
  | INDEX_QUALIFIER  | String  | null             |
  | INDEX_NAME       | String  | index name            |
  | TYPE             | short   | 2 (value of a constant "DatabaseMetaData.tableIndexHashed" representing a hash index) or 3 (value of a constant "DatabaseMetaData.tableIndexOther" representing an index other than hash) |
  | ORDINAL_POSITION | short   | Start from 1.         |
  | COLUMN_NAME      | String  | Column name          |
  | ASC_OR_DESC      | String  | null             |
  | CARDINALITY      | long    | 0                |
  | PAGES            | long    | 0                |
  | FILTER_CONDITION | String  | null             |

  

##### DatabaseMetaData.getPrimaryKeys

```java
ResultSet getPrimaryKeys(String catalog, String schema, String table)
```
- Returns row key information for the table that matches the specified table name "table". When the table of the specified name does not exist, null will be returned for the execution result "ResultSet". Provided that, when null is specified for table, information of all the tables for which the row key is set is returned.
- Other filter conditions catalog and schema are ignored.
- The columns of execution result "ResultSet" are as follows.

  | Column name          | Data type      | Value           |
  |------------------|---------|--------------|
  | TABLE_CAT          | String | null         |
  | TABLE_SCHEM        | String | null         |
  | TABLE_NAME         | String | table name    |
  | COLUMN_NAME        | String | column  name      |
  | KEY_SEQ          | short   | 1            |
  | PK_NAME          | String  | null         |

  

##### DatabaseMetaData.getTables

```java
ResultSet getTables(String catalog, String schemaPattern, String tableNamePattern, String[] types)
```
- Return the information of the table that matches the pattern of the specified table name "tableNamePattern". The wildcard "%" specified in the pattern means to match 0 or more characters, and "_" means to match any one character. When null is specified as tableNamePattern, all tables are targeted.
- Specify null or an array of character strings for types. Specify "TABLE" or "VIEW" for the character string element. If types have no element that matches "TABLE" or "VIEW", null is always returned. Character string elements inside types are not case sensitive. (Types is not a value that represents a collection, type of a table, or a time series container.)
- Other filter conditions catalog and schemaPattern are ignored.
- The columns of execution result "ResultSet" are as follows.

  | Column name                    | Data type      | Value               |
  |----------------------------|---------|------------------|
  | TABLE_CAT                  | String  | null             |
  | TABLE_SCHEM                | String  | null             |
  | TABLE_NAME         | String | table name        |
  | TABLE_TYPE                 | String  | 'TABLE' or 'VIEW'          |
  | REMARKS                    | String  | null             |
  | TYPE_CAT                   | String  | null             |
  | TYPE_SCHEM                 | String  | null             |
  | TYPE_NAME                  | String  | null             |
  | SELF_REFERENCING_COL_NAME  | String  | null             |
  | REF_GENERATION             | String  | null             |

  

##### DatabaseMetaData.getTableTypes

```java
ResultSet getTableTypes()
```
- Return the type of a table. The returned result has only one column 'TABLE_TYPE with the value 'TABLE' or 'VIEW' stored '.
- The columns of execution result "ResultSet" are as follows.

  | Column name          | Data type      | Value          |
  |------------------|---------|-------------|
  | TABLE_TYPE       | String  | 'TABLE' or 'VIEW'     |

  

##### DatabaseMetaData.getTypeInfo()

```java
ResultSet getTypeInfo()
```
- Return the list of column data types.
- The information common to all data types and the information of each data type are as follows.

  | Column name            | Data type      | Value                |
  |--------------------|---------|-------------------------|
  | TYPE_NAME          | String  | Name of data type (see the table below)     |
  | DATA_TYPE          | int     | Value of data type (see the table below)     |
  | PRECISION          | int     | For TIMESTAMP, specify 9, which is the number of digits in the fractional seconds for nanosecond as the maximum precision; for other types, specify 0.         |
  | LITERAL_PREFIX     | String  | null      |
  | LITERAL_SUFFIX     | String  | null      |
  | CREATE_PARAMS      | String  | null      |
  | NULLABLE           | short   | 1 (Value of a constant DatabaseMetaData.typeNullable representing that a null value is allowed for this data type)    |
  | CASE_SENSITIVE     | boolean | true      |
  | SEARCHABLE         | short   | 3 (Value of a constant DatabaseMetaData.typeSearchable indicating that this data type can be used in the WHERE clause    |
  | UNSIGNED_ATTRIBUTE | boolean | false     |
  | FIXED_PREC_SCALE   | boolean | false     |
  | AUTO_INCREMENT     | boolean | false     |
  | LOCAL_TYPE_NAME    | String  | null      |
  | MINIMUM_SCALE      | short   | 0         |
  | MAXIMUM_SCALE      | short   | 0         |
  | SQL_DATA_TYPE      | int     | 0         |
  | SQL_DATETIME_SUB   | int     | 0         |
  | NUM_PREC_RADIX     | int     | 10        |


- For columns TYPE_NAME and DATA_TYPE, all values of the following combinations are returned

  | Value of TYPE_NAME       | Value of DATA_TYPE         |
  |---------------------|----------------------|
  | 'BOOL'              | -7 (Types.BIT)       |
  | 'STRING'            | 12 (Types.VARCHAR)   |
  | 'BYTE'              | -6 (Types.TINYINT)   |
  | 'SHORT'             | 5 (Types.SMALLINT)   |
  | 'INTEGER'           | 4 (Types.INTEGER)    |
  | 'LONG'              | -5 (Types.BIGINT)    |
  | 'FLOAT'             | 6 (Types.FLOAT)      |
  | 'DOUBLE'            | 8 (Types.DOUBLE)     |
  | 'TIMESTAMP'         | 93 (Types.TIMESTAMP) |
  | 'BLOB'              | 2004 (Types.BLOB)    |
  | 'UNKNOWN'           | 0 (Types.NULL)       |

  

#### Method that returns a value

Among the methods of DatabaseMetaData interface, the execution results are listed about the method that returns simple value such as int type and String type as execution result.

| Method                                               | Result                                                                               |
|---------------------------------------------------------|------------------------------------------------------------------------------------|
| allProceduresAreCallable()                              | false                                                                              |
| allTablesAreSelectable()                                | true                                                                               |
| autoCommitFailureClosesAllResultSets()                  | false                                                                              |
| dataDefinitionCausesTransactionCommit()                 | false                                                                              |
| dataDefinitionIgnoredInTransactions()                   | true                                                                               |
| deletesAreDetected(type)                                | false                                                                              |
| doesMaxRowSizeIncludeBlobs()                            | false                                                                              |
| generatedKeyAlwaysReturned()                            | false                                                                              |
| getCatalogSeparator()                                   | "."                                                                                |
| getCatalogTerm()                                        | "catalog"                                                                          |
| getDefaultTransactionIsolation()                        | TRANSACTION_READ_COMMITTED                                                |
| getExtraNameCharacters()                                | . - / = (unordered)                                                                   |
| getIdentifierQuoteString()                              | "                                                                                  |
| getMaxBinaryLiteralLength()                             | 0                                                                                  |
| getMaxCatalogNameLength()                               | 0                                                                                  |
| getMaxCharLiteralLength()                               | 0                                                                                  |
| getMaxColumnNameLength()                                | 0                                                                                  |
| getMaxColumnsInGroupBy()                                | 0                                                                                  |
| getMaxColumnsInIndex()                                  | 0                                                                                  |
| getMaxColumnsInOrderBy()                                | 0                                                                                  |
| getMaxColumnsInSelect()                                 | 0                                                                                  |
| getMaxColumnsInTable()                                  | 0                                                                                  |
| getMaxConnections()                                     | 0                                                                                  |
| getMaxCursorNameLength()                                | 0                                                                                  |
| getMaxIndexLength()                                     | 0                                                                                  |
| getMaxSchemaNameLength()                                | 0                                                                                  |
| getMaxProcedureNameLength()                             | 0                                                                                  |
| getMaxRowSize()                                         | 0                                                                                  |
| getMaxStatementLength()                                 | 0                                                                                  |
| getMaxStatements()                                      | 0                                                                                  |
| getMaxTableNameLength()                                 | 0                                                                                  |
| getMaxTablesInSelect()                                  | 0                                                                                  |
| getMaxUserNameLength()                                  | 0                                                                                  |
| getProcedureTerm()                                      | "procedure"                                                                        |
| getResultSetHoldability()                               | CLOSE_CURSORS_AT_COMMIT                                                    |
| getRowIdLifetime()                                      | true                                                                               |
| getSchemaTerm()                                         | "schema"                                                                           |
| getSearchStringEscape()                                 | "|                                                                                 |
| getSQLKeywords()                                        | ""                                                                                 |
| getSQLStateType()                                       | sqlStateSQL99                                                                      |
| getStringFunctions()                                    | ""                                                                                 |
| getSystemFunctions()                                    | ""                                                                                 |
| getURL()                                                | null                                                                               |
| getUserName()                                           | (user name)                                                                         |
| insertsAreDetected(type)                                | false                                                                              |
| isCatalogAtStart()                                      | true                                                                               |
| isReadOnly()                                            | false                                                                              |
| locatorsUpdateCopy()                                    | false                                                                              |
| nullPlusNonNullIsNull()                                 | true                                                                               |
| nullsAreSortedAtEnd()                                   | false                                                                              |
| nullsAreSortedAtStart()                                 | false                                                                              |
| nullsAreSortedHigh()                                    | true                                                                               |
| nullsAreSortedLow()                                     | false                                                                              |
| othersDeletesAreVisible(type)                           | false                                                                              |
| othersInsertsAreVisible(type)                           | false                                                                              |
| othersUpdatesAreVisible(type)                           | false                                                                              |
| ownDeletesAreVisible(type)                              | false                                                                              |
| ownInsertsAreVisible(type)                              | false                                                                              |
| ownUpdatesAreVisible(type)                              | false                                                                              |
| storesLowerCaseIdentifiers()                            | false                                                                              |
| storesLowerCaseQuotedIdentifiers()                      | false                                                                              |
| storesMixedCaseIdentifiers()                            | true                                                                               |
| storesMixedCaseQuotedIdentifiers()                      | false                                                                              |
| storesUpperCaseIdentifiers()                            | false                                                                              |
| storesUpperCaseQuotedIdentifiers()                      | false                                                                              |
| supportsAlterTableWithAddColumn()                       | false                                                                              |
| supportsAlterTableWithDropColumn()                      | false                                                                              |
| supportsANSI92EntryLevelSQL()                           | false                                                                              |
| supportsANSI92FullSQL()                                 | false                                                                              |
| supportsANSI92IntermediateSQL()                         | false                                                                              |
| supportsBatchUpdates()                                  | false                                                                              |
| supportsCatalogsInDataManipulation()                    | false                                                                              |
| supportsCatalogsInIndexDefinitions()                    | false                                                                              |
| supportsCatalogsInPrivilegeDefinitions()                | false                                                                              |
| supportsCatalogsInProcedureCalls()                      | false                                                                              |
| supportsCatalogsInTableDefinitions()                    | false                                                                              |
| supportsColumnAliasing()                                | true                                                                               |
| supportsConvert()                                       | false                                                                              |
| supportsConvert(fromType, toType)                       | false                                                                              |
| supportsCoreSQLGrammar()                                | true                                                                               |
| supportsCorrelatedSubqueries()                          | true                                                                               |
| supportsDataDefinitionAndDataManipulationTransactions() | false                                                                              |
| supportsDataManipulationTransactionsOnly()              | false                                                                              |
| supportsDifferentTableCorrelationNames()                | false                                                                              |
| supportsExpressionsInOrderBy()                          | true                                                                               |
| supportsExtendedSQLGrammar()                            | false                                                                              |
| supportsFullOuterJoins()                                | false                                                                              |
| supportsGetGeneratedKeys()                              | false                                                                              |
| supportsGroupBy()                                       | true                                                                               |
| supportsGroupByBeyondSelect()                           | true                                                                               |
| supportsGroupByUnrelated()                              | true                                                                               |
| supportsIntegrityEnhancementFacility()                  | false                                                                              |
| supportsLikeEscapeClause()                              | true                                                                               |
| supportsLimitedOuterJoins()                             | true                                                                               |
| supportsMinimumSQLGrammar()                             | true                                                                               |
| supportsMixedCaseIdentifiers()                          | false                                                                              |
| supportsMixedCaseQuotedIdentifiers()                    | true                                                                               |
| supportsMultipleOpenResults()                           | false                                                                              |
| supportsMultipleResultSets()                            | false                                                                              |
| supportsMultipleTransactions()                          | false                                                                              |
| supportsNamedParameters()                               | false                                                                              |
| supportsNonNullableColumns()                            | true                                                                               |
| supportsOpenCursorsAcrossCommit()                       | false                                                                              |
| supportsOpenCursorsAcrossRollback()                     | false                                                                              |
| supportsOpenStatementsAcrossCommit()                    | false                                                                              |
| supportsOpenStatementsAcrossRollback()                  | false                                                                              |
| supportsOrderByUnrelated()                              | true                                                                               |
| supportsOuterJoins()                                    | true                                                                               |
| supportsPositionedDelete()                              | false                                                                              |
| supportsPositionedUpdate()                              | false                                                                              |
| supportsResultSetConcurrency(type, concurrency)         | Only in the case where type is TYPE_FORWARD_ONLY, and concurrency is CONCUR_READ_ONLY |
| supportsResultSetHoldability(holdability)               | Only in case of CLOSE_CURSORS_AT_COMMIT                                          |
| supportsResultSetType()                                 | Only in case of TYPE_FORWARD_ONLY                                               |
| supportsSavepoints()                                    | false                                                                              |
| supportsSchemasInDataManipulation()                     | false                                                                              |
| supportsSchemasInIndexDefinitions()                     | false                                                                              |
| supportsSchemasInPrivilegeDefinitions()                 | false                                                                              |
| supportsSchemasInProcedureCalls()                       | false                                                                              |
| supportsSchemasInTableDefinitions()                     | false                                                                              |
| supportsSelectForUpdate()                               | false                                                                              |
| supportsStatementPooling()                              | false                                                                              |
| supportsStoredFunctionsUsingCallSyntax()                | false                                                                              |
| supportsStoredProcedures()                              | false                                                                              |
| supportsSubqueriesInComparisons()                       | false                                                                              |
| supportsSubqueriesInExists()                            | true                                                                               |
| supportsSubqueriesInIns()                               | true                                                                               |
| supportsSubqueriesInQuantifieds()                       | false                                                                              |
| supportsTableCorrelationNames()                         | false                                                                              |
| supportsTransactionIsolationLevel(level)                | Only in case of TRANSACTION_READ_COMMITTED                                      |
| supportsTransactions()                                  | true                                                                               |
| supportsUnion()                                         | true                                                                               |
| supportsUnionAll()                                      | true                                                                               |
| updatesAreDetected(type)                                | false                                                                              |
| usesLocalFilePerTable()                                 | false                                                                              |
| usesLocalFiles()                                        | false                                                                              |


#### Unsupported methods

Among the methods of DatabaseMetaData interface, the unsupported methods are listed below.
When these methods are executed, SQLFeatureNotSupportedException will not occur and the following results will be returned.

| Method                           | Result          |
|------------------------------------|---------------|
| ResultSet getAttributes(String catalog, String schemaPattern, String typeNamePattern, String attributeNamePattern) | Empty ResultSet |
| ResultSet getBestRowIdentifier(String catalog, String schema, String table, int scope, boolean nullable) | Empty ResultSet |
| ResultSet getCatalogs()                        | Empty ResultSet |
| ResultSet getClientInfoProperties()            | Empty ResultSet |
| ResultSet getColumnPrivileges(String catalog, String schema, String table, String columnNamePattern) | Empty ResultSet |
| ResultSet getCrossReference(String parentCatalog, String parentSchema, String parentTable, String foreignCatalog, String foreignSchema, String foreignTable) | Empty ResultSet |
| ResultSet getExportedKeys(String catalog, String schema, String table)  | Empty ResultSet |
| ResultSet getFunctionColumns(String catalog, String schemaPattern, String functionNamePattern, String columnNamePattern) | Empty ResultSet |
| ResultSet getFunctions(String catalog, String schemaPattern, String functionNamePattern)   | Empty ResultSet |
| ResultSet getImportedKeys(String catalog, String schema, String table)  | Empty ResultSet |
| ResultSet getProcedureColumns(String catalog, String schemaPattern, String procedureNamePattern, String columnNamePattern)  | Empty ResultSet |
| ResultSet getProcedures(String catalog, String schemaPattern, String procedureNamePattern)  | Empty ResultSet |
| ResultSet getPseudoColumns(String catalog, String schemaPattern, String tableNamePattern, String columnNamePattern) | Empty ResultSet |
| ResultSet getSchemas()                       | Empty ResultSet |
| ResultSet getSchemas(String catalog, String schemaPattern)  | Empty ResultSet |
| ResultSet getSuperTables(String catalog, String schemaPattern, String tableNamePattern)  | Empty ResultSet |
| ResultSet getSuperTypes(String catalog, String schemaPattern, String typeNamePattern)    | Empty ResultSet |
| ResultSet getTablePrivileges(String catalog, String schemaPattern, String tableNamePattern)  | Empty ResultSet |
| ResultSet getUDTs(String catalog, String schemaPattern, String typeNamePattern, int[] types) | Empty ResultSet |
| ResultSet getVersionColumns(String catalog, String schema, String table) | Empty ResultSet |

  

### Statement interface

#### Set/get fetch size

Only check the specified value.

When checking this value, check that the number of rows obtained by getMaxRows() of the statement is not exceeded as well.  Limits related to this value are stated only in the JDBC specifications from JDBC4.0 or earlier.  However, unlike the previous JDBC specifications, this excludes the case in which the result of getMaxRows() has been set to the default value 0.

#### Set/get fetch direction

Only FETCH_FORWARD is supported for the fetch direction. A SQLException occurs if FETCH_FORWARD is not specified.

#### Unsupported function

-   Batch update
    -   Batch updates for the Statement interface are not supported. If the following functions are used, the same error as the one that occurs when unsupported standard functions are used.
        -   addBatch()
        -   clearBatch()
        -   executeBatch()
-   Standard functions
    -   The following methods are ignored when invoked. This behavior is different from the JDBC specifications.
        -   setEscapeProcessing(enable)
-   Optional functions
    -   A SQLFeatureNotSupportedException occurs when the method below is invoked.
        -   closeOnCompletion()
        -   execute(sql, autoGeneratedKeys)
        -   execute(sql, columnIndexes)
        -   execute(sql, columnNames)
        -   executeUpdate(sql, autoGeneratedKeys)
        -   executeUpdate(sql, columnIndexes)
        -   executeUpdate(sql, columnNames)
        -   getGeneratedKeys()
        -   getMoreResults(current)
        -   isCloseOnCompletion()

  

### PreparedStatement interface

#### Set/get parameter

The following methods are supported. A SQLException occurs when invoking the query execution API like executeQuery without setting all parameters.

-   addBatch()
-   clearBatch()
-   executeBatch()
    -   Batch updates are only possible for SQL statements that do not contain ResultSet. If they do, an error will be returned. This error is returned not when addBatch() and clearBatch() are used but when executeBatch() is used. An error also occurs when the target SQL statements are complex statements.
-   clearParameters()
-   getMetaData()
-   getParameterMetaData()
-   setBinaryStream(int parameterIndex, InputStream x)
    -    setBinaryStream (including overload), on the client side, may require memory that is several times that of the input data. If the memory becomes insufficient during execution, adjust the heap memory size when starting the JVM.
-   setBinaryStream(int parameterIndex, InputStream x, int length)
-   setBinaryStream(int parameterIndex, InputStream x, long length)
-   setBlob(int parameterIndex, Blob x)
    -    setBinaryStream (including overload), on the client side, may require memory that is several times that of the input data. If the memory becomes insufficient during execution, adjust the heap memory size when starting the JVM.
-   setBlob(int parameterIndex, InputStream inputStream)
-   setBlob(int parameterIndex, InputStream inputStream, long length)
-   setBoolean(int parameterIndex, boolean x)
-   setByte(int parameterIndex, byte x)
-   setDate(int parameterIndex, Date x)
-   setDouble(int parameterIndex, double x)
-   setFloat(int parameterIndex, float x)
-   setInt(int parameterIndex, int x)
-   setLong(int parameterIndex, long x)
-   setObject(int parameterIndex, Object x)
    +   The value set for TIMESTAMP accepts an object of the java.sql.Timestamp or java.util.Date subclass as follows:
    +   for java.sql.Timestamp, TIMESTAMP with nanosecond precision.
    +   for java.util.Date, TIMESTAMP with millisecond precision.
-   setShort(int parameterIndex, short x)
-   setString(int parameterIndex, String x)
-   setTime(int parameterIndex, Time x)
-   setTimestamp(int parameterIndex, Timestamp x)
    +   The value set for TIMESTAMP accepts an object of the java.sql.Timestamp or java.util.Date subclass as follows:
    +   for java.sql.Timestamp, TIMESTAMP with nanosecond precision.
    +   for java.util.Date, TIMESTAMP with millisecond precision.

#### SQL execution

The following methods are supported.

-   execute()
-   executeQuery()
-   executeUpdate()

#### Unsupported function

-   Standard functions
    -   A SQLFeatureNotSupportedException occurs when the method below is invoked. This behavior is different from the JDBC specifications.
        -   setBigDecimal(int parameterIndex, BigDecimal x)
        -   setDate(int parameterIndex, Date x, Calendar cal)
        -   setTime(int parameterIndex, Time x, Calendar cal)
        -   setTimestamp(int parameterIndex, Timestamp x, Calendar cal)
-   Optional functions
    -   A SQLFeatureNotSupportedException occurs when the method below is invoked. All overloads for which the argument has been omitted are not supported.
        -   setArray
        -   setAsciiStream
        -   setBytes
        -   setCharacterStream
        -   setClob
        -   setNCharacterStream
        -   setNClob
        -   setNString
        -   setNull
        -   setObject(int parameterIndex, Object x, int targetSqlType)
        -   setObject(int parameterIndex, Object x, int targetSqlType, int scaleOrLength)
        -   setRef
        -   setRowId
        -   setSQLXML
        -   setUnicodeStream
        -   setURL

  

### ParameterMetaData interface

This section describes the ParameterMetaData interface which gets PreparedStatement metadata.

All methods in the JDBC specification are supported but the methods below will always return a fixed value regardless of the value of argument param.

| Method                                 | Result        |
|-----------------------------------------|-------------|
| int getParameterType(int param)         | Types.OTHER |
| String getParameterTypeName(int param)  | "UNKNOWN"   |
| int getPrecision(int param)             | 0           |
| int getScale(int param)                 | 0           |
| boolean isSigned(int param)             | false       |

  

### ResultSet interface

#### Set/get fetch size

Only the specified value is checked and configuration changes will not affect the actual fetch process.  When checking the value, check that the number of rows obtained by getMaxRows() of the statement in the source generating the target ResultSet is not exceeded as well.  This limit is stated only in the JDBC specifications from JDBC4.0 or earlier. However, unlike the previous JDBC specifications, this excludes the case in which the result of getMaxRows() has been set to the default value 0. Actual fetch process is not affected but the revised setting can be acquired.

#### Set/get fetch direction

Only FETCH_FORWARD is supported for the fetch direction. A SQLException occurs if FETCH_FORWARD is not specified. This behavior is different from the JDBC specifications.

#### Get cursor data

The following cursor-related methods are supported.
-   isAfterLast()
-   isBeforeFirst()
-   isFirst()
-   isLast()
-   next()

Since the only fetch direction supported is FETCH_FORWARD, when the following method is invoked, a SQLException caused by a command being invoked against a FETCH_FORWARD type ResultSet will occur.
-   absolute(row)
-   afterLast()
-   beforeFirst()
-   first()
-   last()
-   previous()
-   relative(rows)

#### Management of warnings

As warnings will not be recorded, the actions to manage warnings are therefore as follows.

| Method         | Behavior                 |
|-----------------|----------------------|
| getWarnings()   | null           |
| clearWarnings() | Warnings are not cleared. |

#### Attribute that returns fixed value

The support status of a method to return a fixed value all the time while the ResultSet remains open is as follows.

| Method         | Result                             |
|------------------|----------------------------------|
| getCursorName()  | null                       |
| getType()        | TYPE_FORWARD_ONLY           |
| getConcurrency() | CONCUR_READ_ONLY            |
| getMetaData()    | (JDBC-compliant)                       |
| getStatement()   | (JDBC-compliant)                       |

#### Data type conversion

When getting the value of a specified column, if the data type maintained by the ResultSet differs from the requested data type, data type conversion will be attempted for the following combinations only.

| The Java type of the destination | BOOL | INTEGRAL *1 | FLOATING *2 | TIMESTAMP | STRING | BLOB |
|----------------|------|-------------|-------------|-----------|--------|------|
| boolean        | ✔    | ✔ *3        |             |           | ✔ *4   |      |
| byte           | ✔    | ✔           | ✔           |           | ✔      |      |
| short          | ✔    | ✔           | ✔           |           | ✔      |      |
| int            | ✔    | ✔           | ✔           |           | ✔      |      |
| long           | ✔    | ✔           | ✔           |           | ✔      |      |
| float          |      | ✔           | ✔           |           | ✔      |      |
| double         |      | ✔           | ✔           |           | ✔      |      |
| byte\[\]       | ✔    | ✔           | ✔           |           | ✔      |      |
| java.sql.Date  |      |             |             | ✔         | ✔ *5  |      |
| Time           |      |             |             | ✔         | ✔ *5  |      |
| Timestamp      |      |             |             | ○ ※8     | ○ ※5  |      |
| String         | ✔    | ✔           | ✔           | ✔ *6     | ✔      | ✔ *7 |
| Blob           |      |             |             |           | ✔ *7   | ✔    |
| Object         | ✔    | ✔           | ✔           | ✔         | ✔      | ✔    |

-   (*1). INTEGRAL: Indicates one of BYTE/SHORT/INTEGER/LONG
-   (*2). FLOATING: Indicates one of FLOAT/DOUBLEFLOATING
-   (*3). Convert to false if 0 and to true if not 0.
-   (*4). Convert to false if "false" and to true if "true". ASCII letters are case insensitive. Other letters cannot be converted and cause an error.
-   (*5). Convert the time of a string expression according to the following rules.
    -   Acceptable expressions expressed in a pattern string like java.command.SimpleDateFormat are as follows, except for the time zone.
        -   yyyy-MM-dd'T'HH:mm:ss.SSS
        -   yyyy-MM-dd'T'HH:mm:ss
        -   yyyy-MM-dd HH:mm:ss.SSS
        -   yyyy-MM-dd HH:mm:ss
        -   yyyy-MM-dd
        -   HH:mm:ss.SSS
        -   HH:mm:ss
    -   If a setting value for the time zone is contained in the string, the value is adopted. Otherwise, if the time zone is specified in the argument java.util.Calendar of an API such as ResultSet#getTimeStamp(), the content is referred to. Still otherwise, the time zone specified at the time of connection is checked, and if none is specified, UTC is adopted. Besides expressions that can be interpreted by a java.command.SimpleDateFormat "z" or "Z" pattern, "Z" expressions indicating that the time is UTC will be accepted as a timezone string.
-   (*6). The time zone information specified at the time of connection will be used. If not specified, UTC will be used.
-   (*7). Convert strings and BLOB to each other by treating it as hexadecimal binary representation. ASCII letters are case insensitive. Other letters cannot be converted and cause an error.
-   *8 Conversion between TIMESTAMP types with different precision is implicitly performed.

#### Get column value

Column values can be obtained using the method corresponding to supported type conversion destination types. As to how to specify columns, both column labels and column indexes are supported.

In obtaining column values, the following objects are obtained for TIMESTAMP columns according to their precision: 
- For TIMESTAMP with microsecond and nanosecond precision, the java.sql.Timestamp object
- For TIMESTAMP with millisecond precision,  the java.util.Date object

Besides these, the following functions can be used.

-   getBinaryStream
    -   Equivalent to the data type conversion result in byte[]
-   wasNull
    -   JDBC-compliant

#### Error processing

-   Invalid column index
    -   If an invalid column index is specified in an attempt to get the value, an SQLException due to a JDBC_COLUMN_INDEX_OUT_OF_RANGE error will occur.
-   Data type conversion error
    -   If data type conversion fails, an SQLException due to a JDBC_VALUE_TYPE_CONVERSION_FAILED error will occur.

#### Unsupported function

The following optional functions are not supported. All overloads for which the argument has been omitted are not supported.

-   cancelRowUpdates()
-   getArray
-   getAsciiStream
-   getBigDecimal
-   getClob
-   getNClob
-   getNCharacterStream
-   getNString
-   getObject(columnIndex, map)
-   getObject(columnLabel, map)
-   getObject(columnIndex, type)
-   getObject(columnLabel, type)
-   getRef
-   getRow()
-   getRowId
-   getSQLXML
-   getUnicodeStream
-   getURL
-   moveToInsertRow()
-   moveToCurrentRow()
-   refreshRow()
-   rowInserted()
-   rowDeleted()
-   rowUpdated()
-   All methods starting with insert
-   All methods starting with update
-   All methods starting with delete

  

### ResultSetMetaData interface

This section describes the ResultSetMetaData interface which gets the search result ResultSet metadata.

For all the methods in the JDBC specification of the ResultSetMetaData interface, the contents and the execution results of each method are described under the following classification.

- Methods which return the data type of a column
- Methods other than the above
- Unsupported methods

#### Data type of column

ResultSetMetaData interface has a method that returns the column data type of the search result ResultSet.

| Method| Description |
|----------|------|
| String getColumnClassName(int column)  | Returns the class name of the specified column data type. |
| int getColumnType(int column)          | Return the value of the specified column data type.      |
| String getColumnTypeName(int column)   | Return the name of the specified column data type.     |

The correspondence between the column data type and the value of the execution result of each method is shown below.

| Data type of a column   | getColumnClassName  | getColumnType   | getColumnTypeName |
|--------------------|---------------------|-----------------|-------------------|
| BOOL             | "java.lang.Boolean" | Types.BIT       | "BOOL"            |
| STRING           | "java.lang.String"  | Types.VARCHAR   | "STRING"          |
| BYTE             | "java.lang.Byte"    | Types.TINYINT   | "BYTE"            |
| SHORT            | "java.lang.Short"   | Types.SMALLINT  | "SHORT"           |
| INTEGER          | "java.lang.Integer" | Types.INTEGER   | "INTEGER"         |
| LONG             | "java.lang.Long"    | Types.BIGINT    | "LONG"            |
| FLOAT            | "java.lang.Float"   | Types.FLOAT     | "FLOAT"           |
| DOUBLE           | "java.lang.Double"  | Types.DOUBLE    | "DOUBLE"          |
| TIMESTAMP        | "java.util.Date"    | Types.TIMESTAMP | "TIMESTAMP"       |
| TIMESTAMP (3)     | "java.util.Date"    | Types.TIMESTAMP | "TIMESTAMP"       |
| TIMESTAMP (6)     | "java.sql.Timestamp" | Types.TIMESTAMP | "TIMESTAMP"      |
| TIMESTAMP (9)     | "java.sql.Timestamp" | Types.TIMESTAMP | "TIMESTAMP"      |
| BLOB             | "java.sql.Blob"     | Types.BLOB      | "BLOB"            |
| GEOMETRY         | "java.lang.Object"  | Types.OTHER     | "UNKNOWN"         |
| ARRAY             | "java.lang.Object"  | Types.OTHER     | "UNKNOWN"         |
| When the data type of a column cannot be identified (*1) | "java.lang.Object"  | Types.OTHER     | "UNKNOWN"         |

[Memo]

- (*1) For example, in the case of ResultSet obtained by executing "SELECT NULL"
- For GEOMETRY type and array type, the value will be returned if the table with these data types created by NoSQL interface is searched. JDBC cannot create tables with these data types.


#### Attribute that returns a value

The result of executing the method other than returning the data type of the column in the ResultSetMetaData interface is shown below.

| Method                                   | Result                         |
|-------------------------------------------|------------------------------|
| String getCatalogName(int column)         | ""                           |
| int getColumnCount()                      | The number of columns                    |
| int getColumnDisplaySize(int column)      | 131072                       |
| String getColumnLabel(int column)         | The label name of a column              |
| String getColumnName(int column)          | The name of a column                  |
| int getPrecision (int column)              | For TIMESTAMP, specify 9, which is the number of digits in the fractional seconds for nanosecond, as the maximum precision; for other types, specify 0.          |
| int getScale(int column)                  | 0                            |
| String getSchemaName(int column)          | ""                           |
| String getTableName(int column)           | ""                           |
| boolean isAutoIncrement(int column)       | false                        |
| boolean isCaseSensitive(int column)       | true                         |
| boolean isCurrency(int column)            | false                        |
| boolean isDefinitelyWritable(int column)  | true                         |
| int isNullable(int column)                | The constant ResultSetMetaData.columnNullable(=1) that allows NULL values for the column, or the constant columnNoNulls(=0) that does not allow NULL values for the column  |
| boolean isReadOnly(int column)            | false                        |
| boolean isSearchable(int column)          | true                         |
| boolean isSigned(int column)              | false                        |
| boolean  isWritable(int column)           | true                         |


#### Unsupported function

There are no unsupported methods (methods that cause SQLFeatureNotSupportedException) in the ResultSetMetaData interface.

  


# Sample

A JDBC sample programs is given below.

``` java
// Execute Sample 2 before running this program. 
package test;

import java.sql.*;

public class SampleJDBC {
	public static void main(String[] args) throws SQLException {
		if (args.length != 5) {
			System.err.println(
				"usage: java SampleJDBC (multicastAddress) (port) (clusterName) (user) (password)");
			System.exit(1);
		}

		// url format "jdbc:gs://(multicastAddress):(portNo)/(clusterName)"
		String url = "jdbc:gs://" + args[0] + ":" + args[1] + "/" + args[2];
		String user = args[3];
		String password = args[4];

		System.out.println("DB Connection Start");

		// Connection to a GridDB cluster
		Connection con = DriverManager.getConnection(url, user, password);
		try {
			System.out.println("Start");
			Statement st = con.createStatement();
			ResultSet rs = st.executeQuery("SELECT * FROM point01");
			ResultSetMetaData md = rs.getMetaData();
			while (rs.next()) {
				for (int i = 0; i < md.getColumnCount(); i++) {
					System.out.print(rs.getString(i + 1) + "|");
				}
				System.out.println("");
			}
			rs.close();
			System.out.println("End");
			st.close();
		} finally {
			System.out.println("DB Connection Close");
			con.close();
		}
	}
}
```

Copyright (c) 2017 TOSHIBA Digital Solutions Corporation
