# CS448 PSO1 SQLITE3 demonstration

SQLite is a C-language library that implements a small, fast, self-contained, high-reliability, full-featured, SQL database engine. In this PSO1, we will demonstrate how to install and manage a sqlite database. In addition, we will cover the basic usage of SQL language which is a standard language for managing relational databases such as MYSQL, PostgresSQL, ORACLE, SQLite, and etc.

The document is created based on [tutorialspoint](https://www.tutorialspoint.com/sqlite/index.htm)
## 1. SQLite Installation

- Window
  - Step 1 − Go to SQLite [download page](https://www.sqlite.org/download.html), and download pre-compiled binaries from Windows section.

  - Step 2 − Download `sqlite-shell-win32-\*.zip` and `sqlite-dll-win32-\*.zip` zipped files.

  - Step 3 − Create a folder `C:\>sqlite` and unzip above two zipped files in this folder, which will give you `sqlite3.def`, `sqlite3.dll` and `sqlite3.exe` files.

  - Step 4 − Add `C:\>sqlite` in your PATH environment variable and finally go to the command prompt and issue sqlite3 command, which should display the following result.

- Linux and Mac are pre-installed.

- You can visit (https://www.tutorialspoint.com/sqlite/sqlite_installation.htm) for installation details.

- GUI: DB Browser for SQLite is a cross-platform SQLite GUI which allows you to manage a SQLite database much easier. You are able to perform all database management operation through graphics user interface. Check it out on https://sqlitebrowser.org/

![screenshot](screenshot.png)

## 2. Database Management
- Create Database in SQLite
```bash
sqlite3 [dbname]
# sqlite3 /tmp/testDB.db
```
- List Database in `sqlite` prompt
```
sqlite> .databases
main: /tmp/testDB.db
```

- Open or Re-Open an existing database
```bash
sqlite> .open [dbname]
# [.open /tmp/testDB.db]
```

- Quit SQLite
```bash
sqlite> .quit
```

## 3. Tables Management

- Syntax

Following is the basic syntax of `CREATE TABLE` statement.
```sql
CREATE TABLE database_name.table_name (
   column1 datatype PRIMARY KEY(one or more columns),
   column2 datatype,
   column3 datatype,
   .....
   columnN datatype
);
```

datatype can be text, numeric, integer, real, and none. For details, please read this article https://www.tutorialspoint.com/sqlite/sqlite_data_types.htm.

- Table creation

Following is an example which creates a `classroom` table with `building` and `room_number` as the primary key.

| building  | room_number | capacity |
| --------- | ----------- | -------- |
| Lamberton | 134         | 10       |
| Chandler  | 375         | 10       |
| Fairchild | 145         | 27       |
| Nassau    | 45          | 92       |
| Grace     | 40          | 34       |

```sql
create table classroom (
    building        varchar(15),
    room_number     varchar(7),
    capacity        numeric(4,0),
    primary key (building, room_number)
);
```

- List Tables

You can verify if your table has been created successfully using SQLite command `.tables` command, which will be used to list down all the tables in an attached database.
```
sqlite> .tables
classroom
```

- Table Info

You can get complete information about a table using the following SQLite `.schema` command.
```
sqlite> .schema classroom
CREATE TABLE classroom (
    building        varchar(15),
    room_number     varchar(7),
    capacity        numeric(4,0),
    primary key (building, room_number)
);

```

- Drop Table

Let us first verify `classroom` table and then we will delete it from the database.
```
sqlite> .tables
classroom
```

This means `classroom` table is available in the database, so let us drop it as follows −
```
sqlite> DROP TABLE classroom;
```

Now, if you try `.tables` command, then you will not find `classroom` table anymore.
```
sqlite> .tables
```

## 4. Import and Dump database

- Import a database from a `.sql` file
```
sqlite> .read [SQL File]
```

- Dump a database into a `.sql` file
```
sqlite> .output [SQL file]
sqlite> .dump
```

You need to specific the output file first, otherwise the `.dump` command will output the sql in the shell

## 5. Database Operations

To manage the data records in a database, we can perform four types of operations: `SELECT`, `INSERT`, `UPDATE`, and `DELETE`, whereas `SELECT` is read operation, and `INSERT`, `UPDATE`, and `DELETE` are write operations, which are summarized in the following
  - Read
    - Select
  - Write
    - Insert, Update, Delete

### Import data
```
sqlite> .read DDL.sql
sqlite> .read smallRelationsInsertFile.sql
```

### INSERT Query

- Syntax
```
INSERT INTO TABLE_NAME [(column1, column2, column3,...columnN)]  
VALUES (value1, value2, value3,...valueN);
```

- Recall the definition of `classroom` table
```
sqlite> .schema classroom
CREATE TABLE classroom (
    building        varchar(15),
    room_number     varchar(7),
    capacity        numeric(4,0),
    primary key (building, room_number)
);
```

- Insert a row into `classroom` table
```
insert into classroom values ('Lawson', '10', '500');
```

### SELECT Query
- Syntax
```
SELECT column1, column2, columnN FROM table_name;
```
Here, column1, column2 ... are the fields of a table, whose values you want to fetch. If you want to fetch all the fields available in the field, then you can use the following syntax
```
SELECT * FROM table_name;
```

- Example

Lookup the data in `classroom` table
```
# Setup a properly output format
sqlite> .header on
sqlite> .mode column
# Lookup the data
sqlite> SELECT * FROM classroom;
building    room_number  capacity  
----------  -----------  ----------
Packard     101          500       
Painter     514          10        
Taylor      3128         70        
Watson      100          30        
Watson      120          50        
Lawson      10           500     
```

### WHERE Clause
SQLite `WHERE` clause is used to specify a condition while fetching the data from one table or multiple tables.

If the given condition is satisfied, means true, then it returns the specific value from the table. You will have to use `WHERE` clause to filter the records and fetching only necessary records.

The WHERE clause not only is used in `SELECT` statement, but it is also used in `UPDATE`, `DELETE` statement, etc., which will be covered later.

- Syntax
```
SELECT column1, column2, columnN
FROM table_name
WHERE [condition]
```

- Example

Consider `classroom` table with the following records.

```
building    room_number  capacity  
----------  -----------  ----------
Packard     101          500       
Painter     514          10        
Taylor      3128         70        
Watson      100          30        
Watson      120          50        
Lawson      10           500     
```

We will find records which the capacity is smaller than 100.
```
sqlite> SELECT * FROM classroom WHERE capacity < 100;
building    room_number  capacity  
----------  -----------  ----------
Painter     514          10        
Taylor      3128         70        
Watson      100          30        
Watson      120          50   
```

You can use other logical operators such as ` >, <, =, LIKE, NOT`, etc

### AND & OR Clause
SQLite **AND** & **OR** operators are used to compile multiple conditions to narrow down the selected data in an SQLite statement. These two operators are called **conjunctive** operators.

- Syntax
```
SELECT column1, column2, columnN
FROM table_name
WHERE [condition1] [AND or OR] [condition2]...[AND or OR] [conditionN];
```

- Example

Consider `classroom` table with the following records.

```
building    room_number  capacity  
----------  -----------  ----------
Packard     101          500       
Painter     514          10        
Taylor      3128         70        
Watson      100          30        
Watson      120          50        
Lawson      10           500     
```

We will find records which the capacity is smaller than 100 and larger than 20.

```
sqlite> SELECT * FROM classroom WHERE capacity < 100 AND capacity > 20;
building    room_number  capacity  
----------  -----------  ----------
Taylor      3128         70        
Watson      100          30        
Watson      120          50    
```

### UPDATE Query
SQLite `UPDATE` Query is used to modify the existing records in a table. You can use `WHERE` clause with UPDATE query to update selected rows, otherwise all the rows would be updated.

```
UPDATE table_name
SET column1 = value1, column2 = value2...., columnN = valueN
WHERE [condition];
```

- Example

Consider `classroom` table with the following records.

```
building    room_number  capacity  
----------  -----------  ----------
Packard     101          500       
Painter     514          10        
Taylor      3128         70        
Watson      100          30        
Watson      120          50        
Lawson      10           500     
```

Following is an example, which will update `classroom` which building is Watson and room_number is 100.
```
sqlite> UPDATE classroom SET capacity = 1000 WHERE building = 'Watson' AND room_number = 100;
```

Now classroom table will have following records:
```
building    room_number  capacity  
----------  -----------  ----------
Packard     101          500       
Painter     514          10        
Taylor      3128         70        
Watson      100          1000      
Watson      120          50        
Lawson      10           500    
```

### DELETE Query
SQLite DELETE Query is used to delete the existing records from a table. You can use WHERE clause with DELETE query to delete the selected rows, otherwise all the records would be deleted.

- Syntax
```
DELETE FROM table_name
WHERE [condition];
```

- Example

Consider `classroom` table with the following records.

```
building    room_number  capacity  
----------  -----------  ----------
Packard     101          500       
Painter     514          10        
Taylor      3128         70        
Watson      100          30        
Watson      120          50        
Lawson      10           500     
```

We will delete the classroom which building equals to Lawson.

```
sqlite> DELETE FROM classroom WHERE building = 'Lawson';
```
Now classroom table will have following records:
```
building    room_number  capacity  
----------  -----------  ----------
Packard     101          500       
Painter     514          10        
Taylor      3128         70        
Watson      100          1000      
Watson      120          50     
```

If you want to DELETE all the records from COMPANY table, you do not need to use WHERE clause with DELETE query, which will be as follows

```
sqlite> DELETE FROM classroom;
```

## 6. Advanced SELECT

In SQLite, we can perform more complicated SELECT operations such as JOIN, LIKE, LIMIT, ORDER BY, GROUP BY, HAVING, AGGREGATION, DISTINCT
