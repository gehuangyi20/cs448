# 448PSO Week 9 - Understanding the Query Evaluation Plan

We can see how a query is executed by `EXPLAIN` plan and whether a query is optimized well.

We will use MySQL to show the query evaluation plan.

You can create a MySQL account following this link: https://www.itap.purdue.edu/infrastructure/database/mysql/careeraccount.html . This requires to set a password and please do not use your Purdue career account password for MySQL account.

**You'd better to use MYSQL version 8.0.20 since features like `EXPLAIN ANALYZE` are not available in lower versions.**

## `EXPLAIN` statement
Example tables:
```sql
CREATE TABLE University(
        UnivId INTEGER CHECK (UnivId > 0 AND UnivId <= 9999),
        UnivName VARCHAR(40) NOT NULL,
        PRIMARY KEY (UnivId)
);
```


```sql
CREATE TABLE Employee(
        EmpId INTEGER CHECK (EmpId > 0),
        EmpName VARCHAR(50) NOT NULL,
        Graduate INTEGER,
        PRIMARY KEY (EmpId),
        FOREIGN KEY (Graduate) REFERENCES University (UnivId)
);
```

```sql
INSERT INTO University VALUES (1, 'Purdue');
INSERT INTO University VALUES (2, 'Maryland');
INSERT INTO University VALUES (3, 'UCSB');
```
```sql
INSERT INTO Employee VALUES (1, 'James', 1);
INSERT INTO Employee VALUES (2, 'Michael', 1);
INSERT INTO Employee VALUES (3, 'Lin', 2);
INSERT INTO Employee VALUES (4, 'Rebecca', 3);
```

We can use `Explain` before `SELECT`, `DELETE`, `INSERT`, `REPLACE`, and `UPDATE`.

If we use:
```sql
explain select * from University, Employee where UnivId=Graduate;
```
 id | select_type | table      | type  | possible_keys | key      | key_len | ref                     | rows | Extra    
----|-------------|------------|-------|---------------|----------|---------|-------------------------|------|--------------------------
  1 | SIMPLE      | University | ALL   | PRIMARY       | NULL | NULL    | NULL |    3 |              
  1 | SIMPLE      | Employee   | ALL   | Graduate      | NULL | NULL    | NULL |    4 | Using where; Using join buffer

* select_type: Simple SELECT (not using UNION or subqueries)
* type : The join type.
  * ALL: A full table scan is done for each combination of rows from the previous tables. This is normally not good if the table is the first table not marked const, and usually very bad in all other cases. Normally, you can avoid ALL by adding indexes that enable row retrieval from the table based on constant values or column values from earlier tables.
* possible_keys : The possible_keys column indicates the indexes from which MySQL can choose to find the rows in this table.
* rows: The rows column indicates the number of rows MySQL believes it must examine to execute the query. For `InnoDB` tables, this number is an estimate, and may not always be exact.
* Extra
  * Using where: A WHERE clause is used to restrict which rows to match against the next table or send to the client.
  * Using join buffer: Tables from earlier joins are read in portions into the join buffer, and then their rows are used from the buffer to perform the join with the current table.

If we show the index of `University`:
```sql
show index from University from xingl;
```
 Table      | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment
----------|-----------|---------|-----------|------------|-------|------------|--------|-------|-----|-----------|-------|---------------
 University |          0 | PRIMARY  |            1 | UnivId      | A         |           3 |     NULL | NULL   |      | BTREE      |         |               

If there are no indexes that can be applied to any join condition, or there are one or more indexes that can be used for single-table predicates, then a hash join is used.
```sql
CREATE TABLE t1 (c1 INT, c2 INT);
CREATE TABLE t2 (c1 INT, c2 INT);
CREATE TABLE t3 (c1 INT, c2 INT);
```
```sql
mysql> explain format=tree select count(*) from t1 join t2 on t1.c2 = t2.c2;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: t2
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where; Using join buffer (hash join)
```
Note: With MySQL version > 8.0.20, `format=tree` is no longer needed.

## The INFORMATION_SCHEMA STATISTICS Table
`INFORMATION_SCHEMA` provides access to database metadata, information about the MySQL server such as the name of a database or table, the data type of a column, or access privileges. Other terms that are sometimes used for this information are data dictionary and system catalog.

The `STATISTICS` table provides information about table indexes.

```sql
SELECT * FROM INFORMATION_SCHEMA.STATISTICS WHERE table_name = 'University' AND table_schema = 'xingl';
```
TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | NON_UNIQUE | INDEX_SCHEMA | INDEX_NAME | SEQ_IN_INDEX | COLUMN_NAME | COLLATION | CARDINALITY | SUB_PART | PACKED | NULLABLE | INDEX_TYPE | COMMENT | INDEX_COMMENT
---------------|--------------|------------|----------|--------------|-----------|-----------|-----------|---------|------------|--------|-------|--------|----------|--------|---------------
def           | xingl        | University |          0 | xingl        | PRIMARY    |            1 | UnivId      | A         |           3 |     NULL | NULL   |          | BTREE      |         |               

* TABLE_CATALOG: The name of the catalog to which the table containing the index belongs. This value is always def.
* TABLE_SCHEMA: The name of the schema (database) to which the table containing the index belongs.
* INDEX_NAME: The name of the index. If the index is the primary key, the name is always PRIMARY.
* SEQ_IN_INDEX: The column sequence number in the index, starting with 1.
* COLLATION: How the column is sorted in the index. This can have values A (ascending), D (descending), or NULL (not sorted).
* CARDINALITY: An estimate of the number of unique values in the index. To update this number, run `ANALYZE TABLE` or (for MyISAM tables) `myisamchk -a`.
* INDEX_TYPE: The index method used (BTREE, FULLTEXT, HASH, RTREE).

### Extended Explain
The `EXPLAIN` statement produces extra (“extended”) information that is not part of `EXPLAIN` output but can be viewed by issuing a `SHOW WARNINGS` statement following `EXPLAIN`.

```sql
explain extended select UnivId, UnivId in (select Graduate from Employee ) from University;
```
 id | select_type        | table      | type           | possible_keys | key      | key_len | ref  | rows | filtered | Extra       
----|--------------------|------------|----------------|---------------|----------|---------|------|------|----------|-------------
  1 | PRIMARY            | University | index          | NULL          | PRIMARY  | 4       | NULL |    3 |   100.00 | Using index
  2 | DEPENDENT SUBQUERY | Employee   | index_subquery | Graduate      | Graduate | 5       | func |    2 |   100.00 | Using index

* filtered: This column indicates the estimated percentage of table rows that are filtered by the table condition.
* Extra
  * Using index: The column information is retrieved from the table using only information in the index tree without having to do an additional seek to read the actual row. This strategy can be used when the query uses only columns that are part of a single index.

You can get a good indication of how good a join is by taking the product of the values in the rows column of the EXPLAIN output. This should tell you roughly how many rows MySQL must examine to execute the query.

```sql
mysql> show warnings;
```
 Level | Code | Message  
-------|------|----------
| Note  | 1003 | select 'xingl'.'University'.'UnivId' AS 'UnivId',<in_optimizer>('xingl'.'University'.'UnivId',<exists>(<index_lookup>(<cache>('xingl'.'University'.'UnivId') in Employee on Graduate checking NULL having <is_not_null_test>('xingl'.'Employee'.'Graduate')))) AS 'UnivId in (select Graduate from Employee )' from 'xingl'.'University'



### EXPLAIN ANALYZE
It produces `EXPLAIN` output along with timing and additional, iterator-based, information about how the optimizer's expectations matched the actual execution.

Supported in MySQL 8.0.18.

```sql
CREATE TABLE t3 (
    pk INTEGER NOT NULL PRIMARY KEY,
    i INTEGER DEFAULT NULL
);
```
```sql
mysql> EXPLAIN ANALYZE SELECT * FROM t3 WHERE pk > 17\G
*************************** 1. row ***************************
EXPLAIN: -> Filter: (t3.pk > 17)  (cost=1.26 rows=5)
(actual time=0.013..0.016 rows=5 loops=1)
    -> Index range scan on t3 using PRIMARY  (cost=1.26 rows=5)
(actual time=0.012..0.014 rows=5 loops=1)
```


## Example: how a multi-table join can be optimized progressively based on the information provided by `EXPLAIN`.

```sql
EXPLAIN SELECT tt.TicketNumber, tt.TimeIn,
               tt.ProjectReference, tt.EstimatedShipDate,
               tt.ActualShipDate, tt.ClientID,
               tt.ServiceCodes, tt.RepetitiveID,
               tt.CurrentProcess, tt.CurrentDPPerson,
               tt.RecordVolume, tt.DPPrinted, et.COUNTRY,
               et_1.COUNTRY, do.CUSTNAME
        FROM tt, et, et AS et_1, do
        WHERE tt.SubmitTime IS NULL
          AND tt.ActualPC = et.EMPLOYID
          AND tt.AssignedPC = et_1.EMPLOYID
          AND tt.ClientID = do.CUSTNMBR;
```
The `tt.ActualPC` values are not evenly distributed.

The columns being compared have been declared as:
Table|Column|Data Type
-|-|-
tt|ActualPC|CHAR(10)
tt|AssignedPC|CHAR(10)
tt|ClientID|CHAR(10)
et|EMPLOYID|CHAR(15)
do|CUSTNMBR|CHAR(15)

And the tables have the following indexes:
Table|Index
-|-
tt|ActualPC
tt|AssignedPC
tt|ClientID
et|EMPLOYID (primary key)
do|CUSTNMBR (primary key)

Before optimization, `EXPLAIN` statement produces the following information:
table|type|possible_keys|key|key_len|ref|rows|Extra
-|-|-|-|-|-|-|-
et|ALL|PRIMARY|NULL|NULL|NULL|74
do|ALL|PRIMARY|NULL|NULL|NULL|2135
et_1|ALL|PRIMARY|NULL|NULL|NULL|74
tt|ALL|AssignedPC,ClientID,ActualPC Range checked for each record (index map: 0x23)|NULL|NULL|NULL|3872

Because `type` is `ALL` for each table, this output indicates that MySQL is generating a Cartesian product of all the tables; that is, every combination of rows. This takes quite a long time, because the product of the number of rows in each table must be examined. For the case at hand, this product is 74 × 2135 × 74 × 3872 = 45,268,558,720 rows.

One problem here is that MySQL can use indexes on columns more efficiently if they are declared as the same type and size. In this context, `VARCHAR` and `CHAR` are considered the same if they are declared as the same size. `tt.ActualPC` is declared as `CHAR(10)` and `et.EMPLOYID` is `CHAR(15)`, so there is a length mismatch in the `WHERE` clause: `tt.ActualPC = et.EMPLOYID`.

To fix this disparity between column lengths, use `ALTER TABLE` to lengthen `ActualPC` from 10 characters to 15 characters:

```sql
ALTER TABLE tt MODIFY ActualPC VARCHAR(15);
```
Executing the EXPLAIN statement again produces this result:
table|type|possible_keys|key|key_len|ref|rows|Extra
-|-|-|-|-|-|-|-
et|eq_ref|PRIMARY|PRIMARY|15|tt.ActualPC|1
do|ALL|PRIMARY, Range checked for each record (index map: 0x1)|NULL|NULL|NULL|2135
et_1|ALL|PRIMARY, Range checked for each record (index map: 0x1)|NULL|NULL|NULL|74
tt|ALL|AssignedPC,ClientID,ActualPC |NULL|NULL|NULL|3872|Using where

Much better: The product of the rows values is less by a factor of 74. This version executes in a couple of seconds.

A second alteration can be made to eliminate the column length mismatches for the tt.AssignedPC = et_1.EMPLOYID and tt.ClientID = do.CUSTNMBR comparisons:

```sql
ALTER TABLE tt MODIFY AssignedPC VARCHAR(15),
               MODIFY ClientID   VARCHAR(15);
```
After that modification, `EXPLAIN` produces the output shown here:
table|type|possible_keys|key|key_len|ref|rows|Extra
-|-|-|-|-|-|-|-
et|ALL|PRIMARY|NULL|NULL|NULL|74
do|eq_ref|PRIMARY|PRIMARY|15|tt.ClientID|1
et_1|eq_ref|PRIMARY|PRIMARY|15|tt.AssignedPC|1
tt|ref|AssignedPC,ClientID,ActualPC |ActualPC|15|et.EMPLOYID|52|Using where

At this point, the query is optimized almost as well as possible. The remaining problem is that, by default, MySQL assumes that values in the `tt.ActualPC` column are evenly distributed, and that is not the case for the `tt` table. We can use `ANALYZE TABLE` to perform a key distribution analysis and store the distribution for the named table or tables.
```sql
ANALYZE TABLE tt;
```

If we execute EXPLAIN AGAIN:
table|type|possible_keys|key|key_len|ref|rows|Extra
-|-|-|-|-|-|-|-
et|eq_ref|PRIMARY|PRIMARY|15|tt.ActualPC |1
do|eq_ref|PRIMARY|PRIMARY|15|tt.ClientID|1
et_1|eq_ref|PRIMARY|PRIMARY|15|tt.AssignedPC|1
tt|ref|AssignedPC,ClientID,ActualPC |NULL|NULL|NULL|3872|Using where

## References
* MySQL documentation (https://dev.mysql.com/doc/refman/8.0/en/information-schema-statistics-table.html)
