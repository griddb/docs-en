
# GridDB Documentation

This repository contains a revision of the GridDB documentation.

## Overview

GridDB is an open source time-series database with the speed of NoSQL and convenience of SQL.
It is optimized for Industrial IoT and Big Data workload with support for time-series and geospatial data.

GridDB Main Features are:

1. IoT Data Model optimization

GridDB's Key Container data model and Time Series functions are purpose built for IoT applications.

2. NoSQL Performance

GridDB's "Memory first, Storage second" algorithm automatically allocates HOT data to the memory and SEMI-HOT data to disks.

3. SQL Usability

Start right away with the standard SQL, and connect to your favourite applications via JDBC.

## Release Notes

GridDB CE v4.5 combines NoSQL and SQL dual interface capabilities, offering high performance and an improved ease of use.

Changes in v4.5 are as follows:
1. Added SQL Interface and JDBC Driver
    - In addition to the NoSQL interface, it is now possible to access the database using SQL with the JDBC driver.
    - SQL92 compliant such as Group By and Join between different tables.
2. Added Table Partitioning functionality
   - By allocating the data of a huge table in a distributed manner, it is now possible to execute the processors in parallel and speed up access.
3. Multiple nodes clustering is changed to a single node setup.

## Manual List

| Name       | Description                 |
|------------|---------------------|
|[GridDB Quick Start Guide](/manuals/GridDB_QuickStartGuide.md)|This manual describes overview, functions and basic operations of GridDB.  |
|[GridDB Features Reference](/manuals/GridDB_FeaturesReference.md)| This manual describes features and functions of GridDB. |
|[GridDB Java API Reference](http://griddb.github.io/docs-en/manuals/GridDB_Java_API_Reference.html)|This manual describes Java API for the application development of GridDB.|
|[GridDB C API Reference](http://griddb.github.io/docs-en/manuals/GridDB_C_API_Reference.html)|This manual describes C API for the application development of GridDB.|
|[GridDB TQL Reference](/manuals/GridDB_TQL_Reference.md)| This manual describes TQL (query language) for the application development of GridDB.|
|[GridDB JDBC driver guide](/manuals/GridDB_JDBC_Driver_UserGuide.md)| This manual describes JDBC driver for the application development of GridDB. |
|[GridDB SQL Reference](/manuals/GridDB_SQL_Reference.md)| This manual describes SQL (query language) for the application development of GridDB. |








## Community
  * Issues  
    Use the GitHub issue function if you have any requests, questions, or bug reports.
  * PullRequest  
    Use the GitHub pull request function if you want to contribute code.
    You'll need to agree GridDB Contributor License Agreement(CLA_rev1.1.pdf).
    By using the GitHub pull request function, you shall be deemed to have agreed to GridDB Contributor License Agreement.
