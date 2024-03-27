# GridDB TQL Reference


## Table of Contents
* [Introduction](#introduction)
* [Data type](#data-type)
* [TQL Syntax and Calculation Functions](#tql-syntax-and-calculation-functions)
* [Appendix](#appendix)


---
# Introduction

GridDB Community Edition provides two types of application programming interfaces: NoSQL interface for basic data access and TQL execution, and NewSQL interface for executing SQL that conforms to SQL92.

This document describes TQL, one of the NoSQL interfaces of GridDB Community Edition (hereinafter referred to as GridDB CE).

---
# Data Type

This section shows the definitions of types specifying the value constraints for fields and query operations.

## Primitive Types

Here are shown the definitions of primitive types which cannot be represented by a combination of any other types.

### Boolean type (BOOL)

Represents the values: TRUE or FALSE.

### String type (STRING)

Represents a sequence of zero or more Unicode code points (characters) excluding the NULL character (U+0000).
Refer to [GridDB Features Reference](GridDB_FeaturesReference.md) for the upper limit size.

### Integer Type

Represents integer values as follows:
-   BYTE: -2<sup>7</sup> to 2<sup>7</sup>-1 (8-bit)
-   SHORT: -2<sup>15</sup> to 2<sup>15</sup>-1 (16-bit)
-   INTEGER: -2<sup>31</sup> to 2<sup>31</sup>-1 (32-bit)
-   LONG: -2<sup>63</sup> to 2<sup>63</sup>-1 (64-bit)

### Floating point type

Represents the IEEE754 floating point numbers. The following types are available depending on the precision.
-   FLOAT: Single-precision type (32-bit)
-   DOUBLE: Double-precision type (64-bit)

Note. In principle, arithmetic precision conforms to the IEEE754 specifications; however, it might vary depending on the runtime environment.

### Time type (TIMESTAMP)

Represents the combination of a date consisting of year, month and day, and a time consisting of hour, minute and second. Refer to the Annex [Range of values](#label_range_of_values) for the display range.

[Memo]

For the time type, precision can be set by selecting one of the following: millisecond, microsecond, or nanosecond.

For more about how to set precision and the corresponding types, see each API reference.

### Spatial type (GEOMETRY)

Represents the spatial structure. Refer to [GridDB Features Reference](GridDB_FeaturesReference.md) for the upper limit size.
It does not support non-numeric numbers (NAN) and positive and negative infinity (INF, -INF) as the number of coordinates represented by each structure.  In addition, it is capable of storing SRID (Spatial Reference System Identifier) as an integer value, but does not support coordinate range limit by the coordinate system represented by the SRID and the coordinate conversion by the conversion of SRID.

### BLOB type

Represents binary data, such as image and sound. Refer to [GridDB Features Reference](GridDB_FeaturesReference.md) for the upper limit size.

## Composite Types

Here are shown the definitions of types which can be represented by a combination of primitive types.

### Array types

Represent a sequence of values. The following types are available for array values. The length of an array indicates the number of array elements.
Refer to [GridDB Features Reference](GridDB_FeaturesReference.md) for the upper limit size. The element of array cannot be set to NULL.
-   Boolean type
-   String type
-   Integer type
-   Floating point type
-   Time type


# TQL Syntax and Calculation Functions

TQL supports a query corresponding to the SQL SELECT statement which is required to select data to be fetched, deleted or updated. It does not support other than a selection query, such as manipulation of selected data, management of data structure and transaction processing.

## Basic Syntax

All queries are expressed by the syntax below:

``` example
[EXPLAIN [ANALYZE]] SELECT (select expression) [FROM (Collection or TimeSeries name)]  [WHERE (conditional expression)] ORDER BY (Column name) [ASC|DESC] [, (Column name) [ASC|DESC]]* [LIMIT (number) [OFFSET (number)]]
```

A SELECT statement is used to narrow down Rows in a Collection or TimeSeries specified in the FROM clause according to the conditional expression in the WHERE clause and process the result set according to the select expression specifying target Column(s), a calculation formula, etc.

If a target Collection or TimeSeries is already specified, you need to omit the FROM clause or specify the same name as the target in the FROM clause. You should note that the FROM clause is case-insensitive. If the WHERE clause is omitted, all the Rows of a target Collection or TimeSeries are selected.

You can place EXPLAIN or EXPLAIN ANALYZE before a SELECT statement to obtain execution plan information and analysis information on execution results in relation to the SELECT statement. See the later section for more information.

Unlike SQL, you cannot extract only specific Column(s) except for aggregation operations. Additionally, clauses corresponding to the following are not available.
-   GROUP BY
-   HAVING
-   DISTINCT
-   FOR UPDATE (Note. can be specified using the API)
-   JOIN

ASCII characters in the keywords of the basic syntax and the names of functions, operators and enumeration constants described in the later sections can be written in lower case.


## Syntax of Conditional Expressions and Calculation Functions

This section shows the definitions of the syntax of conditional expressions used in the WHERE clause and the operators and functions available in a conditional expression.  When NULL is included in operator or function, it returns NULL unless otherwise noted.

### Values 

A value can be either a constant literal or a field of a specified Column in a Row under operation.

#### Literal values

A literal value can be either of the following:

-   Numeric type: A character string of decimal numbers. The notation of floating point numbers conforms to IEEE754. A double-type non-number, positive infinity and negative infinity are written as NaN, INF and -INF, respectively.
-   String type: A character string enclosed in single quotation marks. In order to represent a single quotation mark within a character string, use two single quotation marks (the first single quotation mark is an escape character for the second). You cannot use a single quotation mark for any other purposes.
-   NULL: It is written as NULL.

#### Field values

To specify a field value held by a row under evaluation, either describe directly the corresponding column name or describe the column name enclosed within quotation marks """. By enclosing within quotation marks, columns with the same name as key words such as "SELECT" and "WHERE" can also be handled. You should note that Column names are case-insensitive. Additionally, you cannot join a Column name and a Collection or TimeSeries name by "." etc.

### Precedence of Operators

Operators are evaluated in the order below: Operators with equal precedence are evaluated in the left-to-right order in which they appear.
1.  -(Unary)
2.  \*, /, %
3.  -(Binary), +
4.  \=, \>=, \>, \<=, \<, \<\>, LIKE, IS, IS NOT
5.  NOT
6.  AND
7.  OR, XOR

Round parentheses can be used to override the order of precedence and force some parts of an expression to be evaluated before others.

### Comparison Operations

#### Type Constraints

A combination of left and right operand types can be only Boolean-Boolean, String-String, numeric-numeric, or time-time. With regard to numeric types, if left and right operand types are different in precision, the type with a lower precision or a narrower representable value range will be converted to the one with a higher precision or a wider range. Magnitude comparison cannot be made between Boolean and Boolean types or String and String types.

#### =, \>=, \>, \<=, \<, \<\>

These operators produce a Boolean value representing the result of comparison; namely, "Equal to," "Greater than or equal to," "Greater than," "Less than or equal to," "Less than" and "Not equal to," respectively.
However if either of operands is NULL, it returns NULL.  In principle, arithmetic precision conforms to the IEEE754 specifications; however, it might vary depending on the runtime environment. GridDB assumes that NaN is equal to NaN and that NaN is greater than any other values.

#### IS, IS NOT

These operators obtain the Boolean results of the comparison operations in which the value on the left side is equal to or not equal to NULL. The right side must be NULL.

### Logical Operations

#### NOT, AND, OR, XOR

These operators produce the result of negation, logical product, logical sum and exclusive logical sum, respectively. Operands of logical operations must be Boolean or NULL.

##### NOT

If an operand is TRUE, it returns FALSE. If an operand is FALSE, it returns TRUE. If an operand is NULL, it returns NULL.

##### AND

If both operands are TRUE, it returns TRUE. If either of the operands is FALSE, it returns FALSE. Otherwise, it returns NULL.

##### OR

If both operands are FALSE, it returns FALSE. If either of the operands is TRUE, it returns TRUE. Otherwise, it returns NULL.

##### XOR

If both operands are TRUE or FALSE, it returns FALSE. If either of the operands is NULL, it returns NULL. Otherwise, it returns TRUE.

In addition, a short evaluation (or minimum evaluation) is carried out in an AND or OR operation. That is, if the evaluation can be confirmed with the formula described first, the evaluation of the formula described later will not be conducted.  For example,

``` example
WHERE A=1 AND B=1
```

In this case, if A is 1, then B is deemed to be equal to 1, and if A is not 1, then the evaluation of B=1 need not be carried out.

### String Operations

#### CHAR\_LENGTH(str)

Returns the length of the specified string.

#### CONCAT(str1, str2, ..@@)

Returns a new string obtained by concatenating all given strings. If NULL is included in the input, NULL will be ignored and the rest will be concatenated.

#### str LIKE pattern \[ESCAPE esc\]

Checks if the whole target string matches the specified pattern. The string matching is case-sensitive. It returns TRUE only if the matching is successful.

The following wildcards can be used for specifying a pattern.
-   %: Matches an arbitrary string, including an empty string.
-   \_: Matches a single arbitrary character.

For example, the expression below returns TRUE for "RDB" and "RDBMS" but FALSE for "ORDB" and "DBMS".

``` example
column LIKE '_DB%'
```

Wildcards can be placed at any arbitrary position(s) within a pattern.  FALSE is always returned if an empty string is specified as a pattern.

In order to search a wildcard character itself, specify an escape character in the ESCAPE clause.  For example, the expression below returns TRUE for "10%" but FALSE for "10$%."

``` example
column LIKE '10$%' ESCAPE '$'
```

The escape character must be a single character.

You can specify either a Column name or a string literal as "str," "pattern" and "esc."

#### SUBSTRING(str, start\[, length\])

Extracts parts of a string.

"start" is the position where to start extraction of characters. The first character is at index 1. "length" is the number of characters to extract (0 or positive number). If "length" is omitted, it extracts all the characters from the "start" position to the end of the string. If "start" or "length" is out of index, it returns an empty string. You should note that "start" must be a positive number and "length" must be 0 or a positive number.

#### UPPER(str), LOWER(str)

Convert ASCII alphabetical characters within a string to upper-case and lower-case characters, respectively.

### Numerical Operations

#### +, -(unary), -(binary), \*, /, %

Perform arithmetic operations; addition, negation, subtraction, multiplication, division and remainder, respectively. You should note that the remainder operator does not support floating point numbers. In floating point division,  if a number other than 0 is divided by 0, the result is INF, and if 0 is divided by 0, the result is NaN.  The operations on INF or NaN conform to IEEE754.

#### ROUND(num), FLOOR(num), CEILING(num)

Round a floating point number "num" off to the closest integer, down to the greatest integer that is less than "num," and up to the least integer that is greater than or equal to "num," respectively. The result is DOUBLE.  They round a number away from 0, toward negative infinity, and toward positive infinity, respectively.  Accordingly, if "num" is negative, the results are equal to -ROUND(abs(num)), -CEIL(abs(num)), and -FLOOR(abs(num)), respectively, where abs(num) is the absolute value of "num."

The table below shows operation examples.

| Value  | ROUND | FLOOR | CEILING |
|-------|-------|-------|---------|
| 1.34   | 1.0   | 1.0   | 2.0     |
| 3.67   | 4.0   | 3.0   | 4.0     |
| -0.23 | 0.0   | -1.0 | -0.0    |
| -3.89 | \-4.0 | \-4.0 | -3.0    |

If you specify +0, -0, an integer, NaN, INF, or -INF as a parameter, they return the same value as the specified value.

### Time-Type Operations

#### NOW()

Returns the current date and time. It returns the constant result during a single query transaction.

Example) Searching for data with the date value of time-type column before the current time
``` example
SELECT * WHERE date < NOW()
```

#### TIMESTAMP(str)

Returns a TIMESTAMP result converted from a timestamp expression.

The following format based on the Western calendar is only supported as a timestamp expression.

Example) Searching for data with the date value of time-type column newer than the time "December 30, 2018 10:15:30 (UTC)"

``` example
SELECT *  WHERE date > TIMESTAMP('2018-12-30T10:15:30.000Z')
```

The following format based on the Western calendar is only supported as a timestamp expression. Time zone strings (Z|±hh:mm|±hhmm) are interpreted.

``` example
YYYY-MM-DDThh:mm:ss.SSSZ
```

".SSS" can be omitted.  In the format above, alphabetical characters stand for decimal integers, as follows:
-   YYYY: Year. Four digits or more.
-   MM: Month. Must be two digits, from 1 to 12.
-   DD: Date. Must be two digits, from 1 to 31. If any other number is specified, it will not be accepted. Acceptable numbers depend on the month and the year.
-   hh: Hours in 24-hour format. Must be two digits, from 0 to 23.
-   mm: Minutes. Must be two digits, from 0 to 59.
-   ss: Seconds. Must be two digits, from 0 to 59.
-   SSS: Milliseconds. Must be three digits, from 0 to 999.

Refer to the Annex *Range of values* for the display range.

#### TIMESTAMPADD(time_unit, timestamp, duration)

Returns the result of adding the specified number ("duration") of designated intervals (first parameter) to the specified "timestamp" value. Specify a numeric value as "duration." If a negative value is specified as "duration," it returns the time earlier than the specified time. The current version uses the UTC timezone for calculation.

For the first argument time_unit, specify one of the following identifiers.
- YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND


Example) Searching for data with the date value of time-type column one hour before the current time

``` example
SELECT * WHERE date < TIMESTAMPADD(HOUR, NOW(), -1)
```


#### TIMESTAMPDIFF(time_unit, timestamp1, timestamp2)

Return the difference between two given timestamps for the designate interval (first parameter). The value returned is a numeric value. The current version uses the UTC timezone for calculation.

For the first argument time_unit, specify one of the following identifiers.
- YEAR | MONTH | DAY | HOUR | MINUTE | SECOND | MILLISECOND

Example) Selecting tickets with three or more days of validity period among a list of tickets (tickets).

``` example
SELECT * WHERE TIMESTAMPDIFF(DAY, expired, issued) >= 3
```

Example) Search for data where the difference between time column end and start (=end-start) is less than 10 hours

``` example
SELECT * WHERE TIMESTAMPDIFF(HOUR, end, start) < 10
```

#### TO_TIMESTAMP_MS(num)

Convert to a TIMESTAMP corresponding to the num in milliseconds of the time 1970-01-01T00:00:00Z. An error occurs if num is a floating-point number. In addition, an error occurs if the conversion results cannot be expressed as time data e.g. negative values or extremely large values, etc. As a result, if a query using this function on a numerical column is issued, an error occurs if the conversion results contain values that cannot be expressed as time data in the column value. For example, an error occurs if the container contains a row with a num=-1 in the query below.

``` example
SELECT * WHERE TO_TIMESTAMP_MS(num) > TIMESTAMP('2011-01-01T00:00:00Z')
```

In such a situation, avoid such errors by using the TO_EPOCHMS function as shown below to evaluate only the value in which the converted numerical data serves as the range of the time series data.

``` example
SELECT * WHERE num < TO_EPOCH_MS(TIMESTAMP('9999-12-31T23:59:59.999Z'))
               AND num >= 0
               AND TO_TIMESTAMP_MS(num) > TIMESTAMP('2011-01-01T00:00:00Z')
```

#### TO_EPOCH_MS(timestamp)

For time series value specified in a timestamp, the time passed in milliseconds starting from the time 1970-01-01T00:00:00Z is converted to a LONG value. An error occurs if a value which is not a time series data is specified. This function is the inverted conversion of the TO_TIMESTAMP_MS function.

Example) Comparing column value num of elapsed time from 1970-01-01T00:00:00Z with the current time

``` example
SELECT * WHERE num > TO_EPOCH_MS(NOW())
```


### Array-Type Operations

#### ARRAY_LENGTH(array)

Returns the length of the specified array.

#### ELEMENT(n, array)

Extracts an array element at the specified position. An array of length 1 or more must be specified.  The parameter "n" is the number indicating the element position, starting from 0.  If "n" is Floating point type or a negative value or longer than the length of "array," or if the length of "array" is 0, an error is returned.  Accordingly, if different lengths of arrays are contained in a Collection, and if a query specifying an array element as below cannot extract the specified array element, an error can be returned.

``` example
SELECT * FROM arrays WHERE ELEMENT(1, array) = 1
```

Rewrite such a query to prevent the ELEMENT function from being evaluated, as below:

``` example
SELECT * FROM arrays WHERE ARRAY_LENGTH(array)>= 1 AND ELEMENT(1, array) = 1
```

### Spatial-Type Operations

Spatial-Type data are widely used in the GIS field, such as OpenGIS.  In TQL, it manages the two or three-dimensional spatial structure, and provides generating function and judgment function.

#### ST_GeomFromText(text)

Generates a spatial-type data from a string of WKT representation.

The WKT is a standard for representing the spatial structure as a string.  In TQL, it supports only the following structure.
-   POINT: Point represented by two or three-dimensional coordinate.
-   LINESTRING: Set of straight lines in two or three-dimensional space represented by two or more points.
-   POLYGON: Closed area in two or three-dimensional space represented by a set of straight lines.
-   POLYHEDRALSURFACE: Area in the three-dimensional space represented by a set of the specified area.
-   QUADRATICSURFACE: two-dimensional curved surface in a three-dimensional space represented by defining equation f(X) = \<AX, X\> + BX + c.

However, it can not include the infinity or non-numeric numbers as a number that make up the coordinates.  In addition, it can not give unsupported spatial structures.

In the case of the rectangle having a diagonal line connecting points (0, 0) and (10, 10) on the two-dimensional space will be expressed as follows.

``` example
POLYGON((0 0,10 0,10 10,0 10,0 0))
```

In addition, it can express the value that does not correspond with a particular spatial range called the empty geometry for each type of spatial structure data type. Express "EMPTY" instead of the coordinate values as the following example.

``` example
LINESTRING(EMPTY)
```

In addition, you can specify the SRID by describing the integer after ";".  In the following example, it indicates the rectangle is in the coordinate system of SRID:4326.

``` example
POLYGON((0 0,10 0,10 10,0 10,0 0);4326)
```

However, it does not correspond to coordinate range limit by the coordinate system represented by this SRID or the coordinate transformation by changing the SRID.  If you do not specify a SRID, it is set to -1 as an invalid SRID.

#### ST_MakeRect(p1, p2)

Generate a rectangle having a diagonal line connecting points p1 and p2 on the two-dimensional space. If p1 is equal to p2 or any of the y-coordinate or x-coordinate of p1 and p2 is equal, the result will be an error because it can not form a rectangle.

#### ST_MakeRect(x1, y1, x2, y2)

Generate a rectangle having a diagonal line connecting points (x1, y1) and (x2, y2) on the two-dimensional space. If x1 is equal to x2 or y1 is equal to y2, the result will be an error because it can not form a rectangle.

#### ST_MakeBox(p1, p2)

Generate a rectangular parallelepiped having a diagonal line connecting points p1 and p2 in the three-dimensional space.  If p1 is equal to p2 or any of the coordinate y, x and z of p1 and p2 is equal, the result will be an error because it can not form a rectangular parallelepiped.

#### ST_MakeBox(x1, y1, z1, x2, y2, z2)

Generate a rectangular parallelepiped having a diagonal line connecting points (x1, y1, z1) and (x2, y2, z2) in the three-dimensional space.  If x1 is equal to x2, y1 is equal to y2 or z1 is equal to z2, the result will be an error because it can not form a rectangle parallelepiped.

#### ST_MakePlane(p0x, p0y, p0z, vx, vy, vz)

Generate a plane from the normal vector v and point p0 in the three-dimensional space.  If the length of v is zero, it generate an undefined plane. This undefined plane does not intersect with any kind of spatial structure.

#### ST_MakeCone(p0x, p0y, p0z, vx, vy, vz, deg)

Generate a cone from the angle deg of the axis and the bus, point p0 and vector v of the axis in the three-dimensional space.  Units of angle deg is a degree. An error occurs, if the length of v is 0.  Even if deg is not in the range from 0 to 90, it generates a cone with the remainder of dividing by 360 deg taking into account the reverse vector of the axis.

#### ST_MakeSphere(p0x, p0y, p0z, r)

Generate a sphere from the point p0 and the radius r in the three-dimensional space.  An error occurs, if r is zero or negative value.

#### ST_MakeCylinder(p0x, p0y, p0z, vx, vy, vz, r)

Generate a cylinder from the point p0, vector v of the axis and the radius r in the three-dimensional space.  An error occurs, if the length of v is zero.  If r is 0, it will be a straight line. If r is a negative value, it will be the same as when specifying -r.

#### ST_MakeQSF(A00, A01, A02, A10, A11, A12, A20, A21, A22, B0, B1, B2, c)

Generate a quadratic surface in the three-dimensional space represented by definition equation f(X) = \<AX, X\> + BX + c.  It does not determine whether the definition formula is completed as a two-dimensional curved surface.

#### ST_MBRIntersects(g1, g2)

Determine whether the "Minimum Bounding Box" of the each spatial ranges intersect.  It returns TRUE only if they intersect.  "intersect" means "there is a common region to the two regions."

Both g1 and g2 cannot be specified quadratic surface (QUADRATICSURFACE).  In addition, for POLYHEDRALSURFACE, the result will be undefined if specified the shapes other than rectangular parallelepipeds, the combination of POLYGON which do not share a side each other and the shape which is not closed as a spatial structure.

It use only x and y coordinates excluding z coordinate as a decision object when one is two-dimensional spatial structure consisting of x and y coordinate and the other is three-dimensional spatial structure consisting of x, y and z coordinates.  For example, the result of intersection determination of POINT (x0 y0) and LINESTRING (x1 y1 z1, x2 y2 z2) is true only if x1 \<= x0 \<= x2 and y1 \<= y0 \<= y2 are completed.

FALSE is always returned if either one or both is an empty geometry.

Circumscribed rectangular parallelepiped is defined according to the type of structure as follows.
-   POINT Rectangular parallelepiped whose all vertices located on the same and length of each side is 0. If the structure is two-dimensional, the range of z coordinates of a rectangular parallelepiped is treated as (-∞, ∞).
-   LINESTRING, POLYGON, POLYHEDRALSURFACE: Rectangular parallelepiped consisting of the minimum and maximum value of x, y and z coordinates of the points which make up the structure. Rectangular parallelepiped whose all vertices located on the same and length of each side is 0. If the structure is two-dimensional, the range of z coordinates of a rectangular parallelepiped is treated as (-∞, ∞).

Example) Selecting a Row such as spatial-type data on the column geom and the specified rectangular range intersect.

``` example
SELECT * WHERE ST_MBRIntersects(geom, ST_GeomFromText('POLYGON((0 0,10 0,10 10,0 10,0 0))'))
```

#### ST_QSFMBRIntersects(q, g)

Determine whether the quadratic curved surface q and the circumscribed rectangular parallelepiped of spatial structure g other than two-dimensional curved intersect. It returns TRUE only if they intersect.  It can not give other than quadratic curved surface to q and a quadratic curved surface to g.  Also, it can not give a two-dimensional spatial structure to g.  The condition of other determination of intersecting are the same as ST_MBRIntersects.

#### ST_GetSrId(g)

Returns the SRID of the spatial structure g. If g does not have a SRID, it returns an invalid SRID (-1).



## Selection Expressions

### Basic Syntax

This section defines the selection expressions to specify target Column(s), a calculation formula, etc. Only the Rows satisfying the conditions specified in the FROM and WHERE clauses are to be processed.

#### \\*

Selects all the Rows satisfying the specified condition(s). By designating a ORDER BY section (to be described later), the data can be sorted by column value.

#### (Operation function)

Any one of operation functions described in the following sections is available for aggregation, selection and other operations.

If an overflow occurs in an internal operation, -INF or INF is returned for floating point operation, and the value "undefined" is returned for integer operation. And if NaN is given as an operand for floating-point operation, NaN is returned.

<a id="aggregation_gen"></a>
### Aggregation Operations - General

Here are described the aggregation operations which can be applied to a set of any Rows.

If no target field is found, the number of results returned is 0, except for the functions COUNT and SUM shown below. Otherwise, the number of results returned is always 1.  If the value of the specified column is NULL, it will not get evaluated unless otherwise noted.

#### MAX(column)

Returns the largest value in the specified Column. Only numerical or time series columns can be specified. The type of a returned value is the same as that of the specified Column.

#### MIN(column)

Returns the smallest value in the specified Column. Only numerical or time series columns can be specified. The type of a returned value is the same as that of the specified Column.

#### COUNT(\*)

Return the number of Rows satisfying a given condition(s). A Column cannot be specified. The type of a returned value is always LONG. If there is not a single row that can serve as a target, the value of the operation result is 0.  It also evaluates rows containing a NULL value.

#### SUM(column)

Return the sum of values in the specified Column. Only a numeric-type Column can be specified. The type of a returned value is LONG if the specified Column is of integer type, and DOUBLE if it is of floating-point type.

#### AVG(column)

Returns the average value of the specified Column. Only a numeric-type Column can be specified. The type of a returned value is always DOUBLE.

#### VARIANCE(column)

Returns the variance of values in the specified Column. Only a numeric-type Column can be specified. The type of a returned value is always DOUBLE.

#### STDDEV(column)

Returns the standard deviation of values in the specified Column. Only a numeric-type Column can be specified. The type of a returned value is always DOUBLE.

<a id="aggregation_ts"></a>
### Aggregation Operations - Time Series

Here are described the aggregation operations which can be applied to a set of Rows in a TimeSeries.

In aggregation operation weighted by a time-type key, for each Row satisfying a condition, a weighted value is obtained by calculating half the time span between the adjacent Rows before and after the Row in terms of seconds. However, if a Row has only one adjacent Row, the time span from the adjacent Row is considered, and if no adjacent Rows exist, 1 (sec.) is used as a weighted value.  If the value of the specified column is NULL, it will not get evaluated unless otherwise noted.

#### TIME_AVG(column)

Returns the average weighted by a time-type key of values in the specified Column. The type of a returned value is always DOUBLE.

The weighted average is calculated by dividing the sum of products of sample values and their respective weighted values by the sum of weighted values. Only a numeric-type Column can be specified. The method for calculating a weighted value is as shown above.

Example) Obtaining the time-weighted average voltage at Point 103, plant1 in July 2011.

``` example
SELECT TIME_AVG(voltage103) FROM plant1
  WHERE TIMESTAMP('2011-07-01T00:00:00Z') <= timestamp AND timestamp < TIMESTAMP('2011-08-01T00:00:00Z')
```

Here, you can see an example of the procedure for calculating a weighted average for the TimeSeries below.

| Key (seconds from 00:00:00 on July 1, 2011) | Column to be aggregated |
|----------------------------------|----------------|
| 0 sec.                                      | 4              |
| 10 sec.                                     | 3              |
| 20 sec.                                     | 2              |
| 40 sec.                                     | 1              |

This TimeSeries is extended as shown in the table below; starting from the left, a midpoint time between adjacent Row values, a weighted value, and a product of a sample value (a Column value) and the weighted value are calculated.

| Key        | Column to be aggregated | Midpoint (seconds)  | Time span from (previous) midpoint | Time span from (following) midpoint | Weighted value | Product of sample and weighted values |
|--------|----------------|------------------|--------------------|--------------------|------------|--------------------|
| 0 sec.     | 4                       | \-                  | \-                                 | 5 (=5-0)                            | 5              | 20 (=4\*5)         |
| (Midpoint) | \-                      | 5 sec.(=(0+10)/2)   | \-                                 | \-                                  | \-             | \-                  |
| 10 sec.    | 3                       | \-                  | 5 (=10-5)                          | 5 (=15-10)                          | 10 (=5+5)      | 30 (=3\*10)        |
| (Midpoint) | \-                      | 15 sec.(=(10+20)/2) | \-                                 | \-                                  | \-             | \-                  |
| 20 sec.    | 2                       | \-                  | 5 (=20-15)                         | 10 (=30-20)                         | 15 (=5+10)     | 30 (=2\*15)        |
| (Midpoint) | \-                      | 30 sec.(=(20+40)/2) | \-                                 | \-                                  | \-             | \-                  |
| 40 sec.    | 1                       | \-                  | 10 (=40-30)                        | \-                                  | 10             | 10 (=1\*10)        |

Finally, all the products of sample values and their respective weighted values and all the weighted values are added up respectively, and then the quotient of both the sums is calculated. In the case of this TimeSeries, it is calculated as: (20+30+30+10)/(5+10+15+10) =90/40=2.25. This procedure is not necessarily the same as that for internal operations in GridDB.

The normal unweighted average is calculated as: (4+3+2+1)/4=10/4=2.5.

<a id="ts_data_selection"></a>
### Selection and Interpolation Operations on Time-Series Data

#### TIME_NEXT(\*, timestamp)

Selects a time-series Row whose timestamp is identical with or just after the specified timestamp.

Example) Obtaining the temperature at plant1 at the beginning of July 2011.

``` example
SELECT TIME_NEXT(*, TIMESTAMP('2011-07-01T00:00:00Z')) FROM plant1
```

#### TIME_NEXT_ONLY(\*, timestamp)

Select a time-series Row whose timestamp is just after the specified timestamp.

#### TIME_PREV(\*, timestamp)

Selects a time-series Row whose timestamp is identical with or just before the specified timestamp.

#### TIME_PREV_ONLY(\*, timestamp)

Selects a time-series Row whose timestamp is just before the specified timestamp.

#### TIME_INTERPOLATED(column, timestamp)

Returns a specified Column value of the time-series Row whose timestamp is identical with the specified timestamp, or a value obtained by linearly interpolating specified Column values of adjacent Rows whose timestamps are just before and after the specified timestamp, respectively. NULL is set if NULL is found in either of the previous or the next timestamp of the specified column value.  If no Row with the same timestamp nor no Row with an earlier or later timestamp is found, an intended Row is not generated and the number of Rows returned is 0. Only a numeric-type Column can be specified. The field values of the Row with the latest timestamp among those with the timestamp identical with or earlier than the specified timestamp are set on the specified Column and the fields other than a key.

#### TIME_SAMPLING(\*|column, timestamp_start, timestamp_end, interval, DAY|HOUR|MINUTE|SECOND|MILLISECOND)

Takes a sampling of Rows in a specific range from a given start time to a given end time.

Each sampling time point is defined by adding a sampling interval multiplied by a non-negative integer to the start time, excluding the time points later than the end time.

If there is a Row whose timestamp is identical with each sampling time point, the values of the Row are used. Otherwise, interpolated values are used. Row field values in the specified Column to be interpolated are obtained by linearly interpolating the values of the Rows just before and after a sampling time point. NULL is set if NULL is found in either of the previous or the next timestamp of the specified column value. For other fields, the values of the Row with the latest timestamp among those with earlier timestamps than a sampling time point are used as interpolated values. Columns to be linearly interpolated must be of numeric type. If "\*" is specified instead of a specific Column name, the latter method of interpolation is applied to all field.

If there is no Rows to be referenced for interpolation at a specific sampling time point, a corresponding Row is not generated, and thus the number of results returned is reduced by the number of such time points. A shorter sampling interval increases the likelihood that identical Row field values will be used for the Columns with no need for linear interpolation even at different sampling time points. The sampling interval parameter "interval" must be a positive value.

Example) Obtaining hourly voltage information at Point 103, plant1 on July 1, 2011.

``` example
SELECT TIME_SAMPLING(
  voltage103, TIMESTAMP('2011-07-01T00:00:00Z'), TIMESTAMP('2011-07-02T00:00:00Z'), 1, HOUR)
FROM plant1
```

In addition, regarding this sampling result, the ORDER BY section (to be described later) can be described and sorted in column sequence.  Example) Determine the hourly voltage at Plant 1, Point 103 on July 1, 2011 and sort the data in voltage sequence.

``` example
SELECT TIME_SAMPLING(
  voltage103, TIMESTAMP('2011-07-01T00:00:00Z'), TIMESTAMP('2011-07-02T00:00:00Z'), 1, HOUR)
FROM plant1 ORDER BY voltage103
```

### Row group selection operation with the maximum value/minimum value

Return any row group that has either the maximum or minimum specified column value.  If the value of the specified column is NULL, it will not get evaluated unless otherwise noted.

#### MAX_ROWS(column)

Find the row group with the maximum specified column value. Only numerical or time series columns can be specified.

#### MIN_ROWS(column)

Find the row group with the minimum specified column value. Only numerical or time series columns can be specified.

  

## Sorting of search results (ORDER BY)

The sorting sequence of the search results can be specified by the description in the ORDER BY section.  The description in the ORDER BY section is defined as follows.

``` example
ORDER BY (Column name) [ASC|DESC] [, (Column name) [ASC|DESC]]*
```

However, an '\*' means that the previous component is repeated 0 or more times.

ASC means to sort in ascending order and DES means to sort in descending order. If the sorting sequence is not specified, it will remain as ascendant.  If multiple sorting conditions are specified, the results will be sorted in order starting from the top-most condition.  NULL is sorted as the max value.

Example) Conduct a search with a in descending order as the first sorting condition, b in ascending order as the second sorting condition, and c in ascending order as the third sorting condition.

``` example
SELECT * ORDER BY a DESC, b ASC, c
```

Unlike SQL, functions and formulas cannot be specified in the sorting conditions.

  

## Number of search results found, relative position specification (LIMIT, OFFSET)

The number of search results found can be limited by the description stated in the LIMIT section.  In addition, the start position for locating the search results can be specified by the OFFSET.  The syntax of the LIMIT and OFFSET specifications are defined as follows.

``` example
LIMIT (number) [OFFSET (number)]
```

OFFSET is always used together with LIMIT. If omitted, the meaning is the same as OFFSET 0.  Negative values and floating point decimals cannot be specified for the LIMIT and OFFSET figures.

Formulas and functions cannot be specified for the LIMIT and OFFSET figures.



## Obtaining Execution Plans and Analyzing Execution Results

You can obtain execution plan information and analysis information on execution results by adding EXPLAIN or EXPLAIN ANALYZE before a SELECT statement.

A obtained result set is composed of an array of entries with the same structure as a Row.

| Name        | Type        | Description                  |
|-------------|-----------|------------------------------------------------------------------------|
| ID          | INTEGER     | An ID indicating the position of an entry in an array of entries.     |
| DEPTH       | INTEGER     | Indicates a depth for representing the relation with other entries. If there is found an entry whose depth is smaller than that of a target entry by one, through checking entries one by one whose IDs are smaller than that of the target entry, it means that the target entry describes the content of the found entry in more detail. |
| TYPE        | String type | Indicates classification of information indicated by an entry; namely, classification of analysis results, such as execution time, classification of components of a query plan, etc.     |
| VALUE_TYPE | STRING      | Indicates the type of a value assigned to the information indicated by an entry. The type of a value assigned to execution time or other analysis results, for example, is returned. Type names are the same as the primitive types defined in TQL. An empty string is set as VALUE if no value is assigned.                       |
| VALUE       | String type | Indicates a character string representing a value assigned to the information indicated by an entry. An empty string is set as VALUE if no value is assigned.         |
| STATEMENT   | String type | Returns a part of a TQL statement corresponding to the information indicated by an entry. An empty string is set if no correspondence is found.            |

### EXPLAIN

EXPLAIN is placed before a SELECT statement to obtain execution plan information about the SELECT statement. The SELECT statement itself will not be executed.

Even a single identical query might return different results depending on the indexing settings or other conditions.

### EXPLAIN ANALYZE

EXPLAIN ANALYZE is placed before a SELECT statement to execute the SELECT statement and obtain analysis information, such as execution time, as well as obtain execution plan information about the SELECT statement.

  

# Appendix

<a id="label_range_of_values"></a>
## Range of values

Describe the range of values such as the upper limit of the value, etc.
Refer to [GridDB Features Reference](GridDB_FeaturesReference.md) for the restriction values of the system.

### Values that may be adopted by basic datatypes
The values that may be adopted by the basic datatypes below are as follows.

| Type           | Representable values |
|----|-----------|
| Boolean (BOOL) | TRUE/FALSE |
| BYTE           | \-2<sup>7</sup> to 2<sup>7</sup>-1 |
| SHORT          | \-2<sup>15</sup> to 2<sup>15</sup>-1 |
| INTEGER        | \-2<sup>31</sup> to 2<sup>31</sup>-1 |
| LONG           | \-2<sup>63</sup> to 2<sup>63</sup>-1 |
| FLOAT          | Conforming to IEEE754 |
| DOUBLE         | Conforming to IEEE754 |
| time type (TIMESTAMP) | from the start of 1/1/1970 to the end of 12/31/9999 (equivalent to UTC). Leap seconds are not supported. Precision can be set by selecting one of the following: millisecond, microsecond, or nanosecond. |


Value that can be used to TQL operation in spatial (GEOMETRY) type is any arbitrary value returned by the ST_GeomFromText function. Among these values, the value that can be stored containers is excluding the QUADRATICSURFACE structure.

The range of values of objects mapped onto the basic types through API may be different from those of the above basic types. The value out of the described range cannot be registered into containers. But the value may be used in the other operations, such as constructing a search condition. For example, a java.util.Date object to be mapped onto TIMESTAMP by Java API can have a value before the year 1970 that cannot be stored in containers, and the value can be used as a RowKey condition of a RowKeyPredicate object or in a sampling query. However, in that case, it is possible that an error occurs when obtaining rows by the condition. For the representation range of the object itself to be mapped onto the above basic types, see the definition of the object type.

Copyright (c) 2017 TOSHIBA Digital Solutions Corporation