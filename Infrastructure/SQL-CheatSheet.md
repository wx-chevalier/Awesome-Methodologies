

# SQL 语法速览与备忘清单

SQL 是 Structrued Query Language 的缩写，即结构化查询语言。它是负责与 ANSI(美国国家标准学会)维护的数据库交互的标准。作为关系数据库的标准语言，它已被众多商用 DBMS 产品所采用，使得它已成为关系数据库领域中一个主流语言，不仅包含数据查询功能，还包括插入、删除、更新和数据定义功能。最为重要的 SQL92 版本的详细标准可以查看[这里](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt)，或者在 [Wiki](https://en.wikipedia.org/wiki/SQL) 上查看 SQL 标准的变化。

# Syntax

## Database and Table Manipulation

| Command                                                                                            | Description                          |
| :------------------------------------------------------------------------------------------------- | :----------------------------------- |
| CREATE DATABASE database_name                                                                      | Create a database                    |
| DROP DATABASE database_name                                                                        | Delete a database                    |
| CREATE TABLE "table_name" ("column_1" "column_1_data_type", "column_2" "column_2_data_type", ... ) | Create a table in a database.        |
| ALTER TABLE table_name ADD column_name column_datatype                                             | Add columns in an existing table.    |
| ALTER TABLE table_name DDROP column_name column_datatype                                           | Delete columns in an existing table. |
| DROP TABLE table_name                                                                              | Delete a table.                      |

## Data Types:

| **Data Type**                      | **Description**                                                                                           |
| :--------------------------------- | :-------------------------------------------------------------------------------------------------------- |
| CHARACTER(n)                       | Character string, fixed length n.                                                                         |
| CHARACTER VARYING(n) or VARCHAR(n) | Variable length character string, maximum length n.                                                       |
| BINARY(n)                          | Fixed-length binary string, maximum length n.                                                             |
| BOOLEAN                            | Stores truth values - either TRUE or FALSE.                                                               |
| BINARY VARYING(n) or VARBINARY(n)  | Variable length binary string, maximum length n.                                                          |
| INTEGER(p)                         | Integer numerical, precision p.                                                                           |
| SMALLINT                           | Integer numerical precision 5.                                                                            |
| INTEGER                            | Integer numerical, precision 10.                                                                          |
| BIGINT                             | Integer numerical, precision 19.                                                                          |
| DECIMAL(p, s)                      | Exact numerical, precision p, scale s.                                                                    |
| NUMERIC(p, s)                      | Exact numerical, precision p, scale s. (Same as DECIMAL ).                                                |
| FLOAT(p)                           | Approximate numerical, mantissa precision p.                                                              |
| REAL                               | Approximate numerical mantissa precision 7.                                                               |
| FLOAT                              | Approximate numerical mantissa precision 16.                                                              |
| DOUBLE PRECISION                   | Approximate numerical mantissa precision 16.                                                              |
| DATE TIME TIMESTAMP                | Composed of a number of integer fields, representing an absolute point in time, depending on sub-type.    |
| INTERVAL                           | Composed of a number of integer fields, representing a period of time, depending on the type of interval. |
| COLLECTION (ARRAY, MULTISET)       | ARRAY(offered in SQL99) is a set-length and ordered the collection of elements.                           |
| XML                                | Stores XML data. It can be used wherever a SQL data type is allowed, such as a column of a table.         |

## Index Manipulation:

| Command                                                                          | Description            |
| :------------------------------------------------------------------------------- | :--------------------- |
| CREATE INDEX index_name ON table_name (column_name_1, column_name_2, ...)        | Create a simple index. |
| CREATE UNIQUE INDEX index_name ON table_name (column_name_1, column_name_2, ...) | Create a unique index. |
| DROP INDEX table_name.index_name                                                 | Drop a index.          |

## SQL Operators:

| Operators               | **Description**                                                                                                                                                                                         |
| :---------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| SQL Arithmetic Operator | Arithmetic operators are addition(+), subtraction(-), multiplication(\*) and division(/). The + and - operators can also be used in date arithmetic.                                                    |
| SQL Comparison Operator | A comparison (or relational) operator is a mathematical symbol which is used to compare two values.                                                                                                     |
| SQL Assignment operator | In SQL the assignment operator ( = ) assigns a value to a variable or of a column or field of a table.                                                                                                  |
| SQL Bitwise Operator    | The bitwise operators are & ( Bitwise AND ), \| ( Bitwise OR ) and ^ ( Bitwise Exclusive OR or XOR ). The valid datatypes for bitwise operators are BINARY, BIT, INT, SMALLINT, TINYINT, and VARBINARY. |
| SQL Logical Operator    | The Logical operators are those that are true or false. The logical operators are AND , OR, NOT, IN, BETWEEN, ANY, ALL, SOME, EXISTS, and LIKE.                                                         |
| SQL Unary Operator      | The SQL Unary operators perform such an operation which contain only one expression of any of the datatypes in the numeric datatype category.                                                           |

## Insert, Update and Delete:

| Command                                                                                                                            | Description                            |
| :--------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------- |
| INSERT INTO table_name VALUES (value_1, value_2,....) INSERT INTO table_name (column1, column2,...) VALUES (value_1, value_2,....) | Insert new rows into a table.          |
| UPDATE table_name SET column_name_1 = new_value_1, column_name_2 = new_value_2 WHERE column_name = some_value                      | Update one or several columns in rows. |
| DELETE FROM table_name WHERE column_name = some_value                                                                              | Delete rows in a table.                |

## Select:

| Command                                                                                                                                   | Description                                                                                                                                                                                                                                                        |
| :---------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SELECT column_name(s) FROM table_name                                                                                                     | Select data from a table.                                                                                                                                                                                                                                          |
| SELECT \* FROM table_name                                                                                                                 | Select all data from a table.                                                                                                                                                                                                                                      |
| SELECT DISTINCT column_name(s) FROM table_name                                                                                            | Select only distinct (different) data from a table.                                                                                                                                                                                                                |
| SELECT column_name(s) FROM table_name WHERE column operator value AND column operator value OR column operator value AND (... OR ...) ... | Select only certain data from a table.                                                                                                                                                                                                                             |
| SELECT column_name(s) FROM table_name WHERE column_name IN (value1, value2, ...)                                                          | The IN operator may be used if you know the exact value you want to return for at least one of the columns.                                                                                                                                                        |
| SELECT column_name(s) FROM table_name ORDER BY row_1, row_2 DESC, row_3 ASC, ...                                                          | Select data from a table with sort the rows.                                                                                                                                                                                                                       |
| SELECT column_1, ..., SUM(group_column_name) FROM table_name GROUP BY group_column_name                                                   | The GROUP BY clause is used with the SELECT statement to make a group of rows based on the values of a specific column or expression. The SQL AGGREGATE function can be used to get summary information for every group and these are applied to individual group. |
| SELECT column_name(s) INTO new_table_name FROM source_table_name WHERE query                                                              | Select data from table(S) and insert it into another table.                                                                                                                                                                                                        |
| SELECT column_name(s) IN external_database_name FROM source_table_name WHERE query                                                        | Select data from table(S) and insert it in another database.                                                                                                                                                                                                       |

## Functions:

| SQL functions       | **Description**                                                                                                                                                                                                                                                  |
| :------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Aggregate Function  | This function can produce a single value for an entire group or table. Some Aggregate functions are - SQL Count functionSQL Sum functionSQL Avg functionSQL Max functionSQL Min function                                                                         |
| Arithmetic Function | A mathematical function executes a mathematical operation usually based on input values that are provided as arguments, and return a numeric value as the result of the operation. Some Arithmetic functions are - abs()ceil()floor()exp()ln()mod()power()sqrt() |
| Character Function  | A character or string function is a function which takes one or more characters or numbers as parameters and returns a character value. Some Character functions are - lower()upper()trim()translate()                                                           |

## Joins:

| Name              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :---------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SQL EQUI JOIN     | The SQL EQUI JOIN is a simple SQL join uses the equal sign(=) as the comparison operator for the condition. It has two types - SQL Outer join and SQL Inner join. SQL INNER JOIN returns all rows from tables where the key record of one table is equal to the key records of another table. SQL OUTER JOIN returns all rows from one table and only those rows from the secondary table where the joined condition is satisfying i.e. the columns are equal in both tables. |
| SQL NON EQUI JOIN | The SQL NON EQUI JOIN is a join uses comparison operator other than the equal sign like >, <, >=, <= with the condition.                                                                                                                                                                                                                                                                                                                                                      |

## Union:

| Command                                   | Description                                                          |
| :---------------------------------------- | :------------------------------------------------------------------- |
| SQL_Statement_1 UNION SQL_Statement_2     | Select all different values from SQL_Statement_1 and SQL_Statement_2 |
| SQL_Statement_1 UNION ALL SQL_Statement_2 | Select all values from SQL_Statement_1 and SQL_Statement_2           |

## View:

| Command                                                                        | Description                                                           |
| :----------------------------------------------------------------------------- | :-------------------------------------------------------------------- |
| CREATE VIEW view_name AS SELECT column_name(s) FROM table_name WHERE condition | Create a virtual table based on the result-set of a SELECT statement. |

# Examples

```sql
-- Comments start with two hyphens. End each command with a semicolon.

-- SQL is not case-sensitive about keywords. The sample commands here
-- follow the convention of spelling them in upper-case because it makes
-- it easier to distinguish them from database, table, and column names.

-- Create and delete a database. Database and table names are case-sensitive.
CREATE DATABASE someDatabase;
DROP DATABASE someDatabase;

-- List available databases.
SHOW DATABASES;

-- Use a particular existing database.
USE employees;

-- Select all rows and columns from the current database's departments table.
-- Default activity is for the interpreter to scroll the results on your screen.
SELECT * FROM departments;

-- Retrieve all rows from the departments table,
-- but only the dept_no and dept_name columns.
-- Splitting up commands across lines is OK.
SELECT dept_no,
       dept_name FROM departments;

-- Retrieve all departments columns, but just 5 rows.
SELECT * FROM departments LIMIT 5;

-- Retrieve dept_name column values from the departments
-- table where the dept_name value has the substring 'en'.
SELECT dept_name FROM departments WHERE dept_name LIKE '%en%';

-- Retrieve all columns from the departments table where the dept_name
-- column starts with an 'S' and has exactly 4 characters after it.
SELECT * FROM departments WHERE dept_name LIKE 'S____';

-- Select title values from the titles table but don't show duplicates.
SELECT DISTINCT title FROM titles;

-- Same as above, but sorted (case-sensitive) by the title values.
SELECT DISTINCT title FROM titles ORDER BY title;

-- Show the number of rows in the departments table.
SELECT COUNT(*) FROM departments;

-- Show the number of rows in the departments table that
-- have 'en' as a substring of the dept_name value.
SELECT COUNT(*) FROM departments WHERE dept_name LIKE '%en%';

-- A JOIN of information from multiple tables: the titles table shows
-- who had what job titles, by their employee numbers, from what
-- date to what date. Retrieve this information, but instead of the
-- employee number, use the employee number as a cross-reference to
-- the employees table to get each employee's first and last name
-- instead. (And only get 10 rows.)

SELECT employees.first_name, employees.last_name,
       titles.title, titles.from_date, titles.to_date
FROM titles INNER JOIN employees ON
       employees.emp_no = titles.emp_no LIMIT 10;

-- List all the tables in all the databases. Implementations typically provide
-- their own shortcut command to do this with the database currently in use.
SELECT * FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE='BASE TABLE';

-- Create a table called tablename1, with the two columns shown, for
-- the database currently in use. Lots of other options are available
-- for how you specify the columns, such as their datatypes.
CREATE TABLE tablename1 (fname VARCHAR(20), lname VARCHAR(20));

-- Insert a row of data into the table tablename1. This assumes that the
-- table has been defined to accept these values as appropriate for it.
INSERT INTO tablename1 VALUES('Richard','Mutt');

-- In tablename1, change the fname value to 'John'
-- for all rows that have an lname value of 'Mutt'.
UPDATE tablename1 SET fname='John' WHERE lname='Mutt';

-- Delete rows from the tablename1 table
-- where the lname value begins with 'M'.
DELETE FROM tablename1 WHERE lname like 'M%';

-- Delete all rows from the tablename1 table, leaving the empty table.
DELETE FROM tablename1;

-- Remove the entire tablename1 table.
DROP TABLE tablename1;
```
