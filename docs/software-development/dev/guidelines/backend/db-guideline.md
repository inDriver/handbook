# Database Guideline

Databases in inDrive are subject to high loads, which imposes certain requirements on the development of queries and
data structure migrations. This guideline provides the main recommendations for naming entities, making changes, and
performance considerations that engineers should pay attention to during the development process.

Following this guideline will help pass SQL reviews faster. Prepared migrations should always be run through SQLLint.


## Naming Rules for Entities

The general recommendation for naming all entities is to use common words that do not allow ambiguous interpretations.
For example, use the term `gender` for storing gender data. Avoid naming conflicts with reserved words
from the SQL dialect being used.


### Table Names

Tables should be named with a noun in English indicating the main entity in the singular. For compound names, words are
separated by underlining.

Good:

* `order`;
* `vehiсle_details` (since each row in the table stores a set of attributes and options of the car, plural is allowed).

Bad:

* `orders`;
* `rides_orders`;
* `VehicleDetails`;
* `vehicle_detail`.


### Column Names

Table columns should be named with a singular English noun. For compound names, words are separated by underscores. For
storing unstructured information, use the reserved field name - `jdoc`.  Don't use
[reserved words](https://dev.mysql.com/doc/refman/8.4/en/keywords.html).

When naming table fields, use standard prefixes and suffixes:

* `is_` for logical fields;
* `_by` for event sources;
* `_at` for timestamps.

Good:

* `id`;
* `uuid`;
* `order_id` (if it's a link to the order in other table);
* `document_text`;
* `created_at`;
* `updated_by`.

Bad:

* `order_ids`;
* `text`;
* `creation_date`;
* `updated_by_user`.


### Index Names

It is recommended to make up the index name from the table name, field/fields and the `_idx` suffix. If the name is too
long, abbreviations are allowed that do not lead to the loss of information about the purpose of this index


### Key and Constraint Names

It is recommended to make up the name from the field/fields name and the _key suffix. If the name is too long,
abbreviations are allowed that do not lead to the loss of information about the purpose of this index


### Foreign Keys

Despite certain advantages and data integrity control, we prohibit the use of foreign keys in DB schemas because they
introduce a large number of restrictions on the execution of online DDL. You can read more
[here](https://code.openark.org/blog/mysql/the-problem-with-mysql-foreign-key-constraints-in-online-schema-changes).


## Indices

The table must always have a `PRIMARY(UNIQUE) key`. This is critically important for replication. The primary key must
be compact. It is recommended to avoid zero (`NULL`) values in columns with keys, since a nullable field requires more
space and additional calculations inside `innodb`.

It should be noted that a standard query of the `create index ...` is not performed by online migration, but directly
on the table. When creating an index on a large table, this can lead to its blocking. It is safer to use a construction
of the following type:

```sql
ALTER TABLE table_name ADD INDEX ...
```


### Cardinality and Selectivity

The index being created will work well only if the column by which it is created has a sufficient number of different
values (high cardinality). Then this index will effectively reduce the volume of data being selected. You could check
cardinality with the following command:

```sql
SHOW INDEX FROM tbl_name;
```

There is a rare case when the cardinality value can be neglected if there is an understanding that the column by which
the index is created has a strong skew in the data. For example, the distribution is 95/5, and in the samples you need
to get only the values that are smaller, then in this case you can create an index on such a column in order to get
high selectivity.

More details in this [article](https://webmonkeyuk.wordpress.com/2010/09/27/what-makes-a-good-mysql-index-part-2-cardinality/).


### B-tree Indices

* well suited for most search operations (=, >, <, >=, <=, between);
* works for LIKE searches if you need to start from the beginning (LIKE 'word%' but not LIKE '%word').


### Hash Indices

* work only for equality conditions (=, <>), but very fast;
* cannot be used in ORDER BY.


### Composite Keys

Prefer one composite key to several simple ones:

* the composite key is used from left to right;
* place the most [selective](#cardinality-and-selectivity) fields of the composite key at the beginning;
* selectivity: keys with a selection of +50% of the table are **useless**. MySQL does not have partial keys.

Avoid duplicates: when one index contains another index entirely.

Example — the first key is useless because the second contains everything needed:

```sql
KEY `city_id` (`city_id`),
KEY `city_id_type_status` (`city_id`,`type`,`status`)
```

When using `OR`, make sure that each group by `OR` uses an indexed field in the same index.

Suppose there is a composite index on columns (`index_part1`, `index_part2`, `index_part3`).

These queries will use the index.

```sql
-- index_part1, index_part2 will be used
... WHERE index_part1=1 AND index_part2=2 AND other_column=3

    /* index_part1 = 1 on the left and index_part2 = 2 on the right will be used */
... WHERE (index_part1=1 OR other_column=10) AND index_part2=2

    /* only "index_part1='hello'" but not index_part3 will be used */
... WHERE index_part1='hello' AND index_part3=5

    /* Only index_part1 will be used, as it is in both parts of OR */
... WHERE (index_part1=1 AND index_part2=2) OR (index_part1=3 AND index_part3=3);
```

And here the index will not be used:

```sql
    /* index_part1 is not used, so the index does not work */
... WHERE index_part2=1 AND index_part3=2

    /* there is no index for other_column=10, so the entire expression will not use the index */
... WHERE index_part1=1 OR other_column=10

    /* there is no condition for index_part1 on the right */
... WHERE index_part1=1 OR index_part2=10
```

Always use `EXPLAIN` to check index performance. In small selections, `EXPLAIN` will work incorrectly.

Key selections:

* equality is the best choice:
  * `WHERE` column = value;
  * `WHERE` column IN (value1, value2).

Do not use functions on indexed columns — the index will not work:

Bad:

```sql
SELECT COUNT(*) FROM tbl_modercheck 
WHERE 
  TIMESTAMPDIFF(HOUR, created, NOW()) < 12
```

Good:

```sql
SELECT COUNT(*) FROM tbl_modercheck 
WHERE 
 (created > NOW() - interval 12 hour) 
```


### Identifier Format Number vs UUID

In general, it is recommended to use [UUIDv7](https://dev.indriver.io/components/architecture-docs/main/documentation/adrs/0003-ARCH-205-uuidv7-primary-key-selection)
or [UUIDv8](https://dev.indriver.io/components/architecture-docs/main/documentation/adrs/0009-ARCH-383-universal-entity-identifier)
in `BINARY(16)` format for the Primary Key in the table. The identifiers of both versions already contain the record
creation time, so for data consistency, we create a virtual created_at column - its values will be formed from the
identifier. For example:

```sql
`created_at` DATETIME GENERATED ALWAYS AS (from_unixtime(conv(hex(`user_uuid` >> 80),16,10) / 1000)) VIRTUAL
```

You can use the Primary Key in `NUMBER` format in case of creating any reference books in which the data changes slowly,
at one point and is common to all regions, which eliminates the need for further data merging and record identifier
conflicts.


## Partitioning

We use partitioning in large tables to simplify data management: deleting old data, moving data to cold storage, etc.
Partitions can also have a positive effect on the speed of some queries by reducing the amount of data being processed.

Since the most common way for us to partition data is by record creation date, and date and time are part of the UUIDv7
and UUIDv8 identifier, the correct way is to partition the table by ID. This method will simplify working with old
records and allow you to find a record by its identifier as quickly as possible.

Other ways of partitioning tables are also possible depending on the data usage scenario.

Partitioning would be applied by DBA, developers should make this possible:

* use UUID v7 or v8 as PK;
* implement code that will work with partitions efficiently;
* be careful with JOINs.

Example of partitioning by UUID:

```sql
PARTITION BY RANGE COLUMNS(uuid) (
    PARTITION p01 VALUES LESS THAN (_binary 0x018D61F7180000000000000000000000) ENGINE = InnoDB, 
    PARTITION p_max VALUES LESS THAN (MAXVALUE) ENGINE = InnoDB
)
```

## Using String Types for Enumerated Values

To store string values with a limited number of options (about ten), it is recommended to use the `ENUM` or `SET` type. If
you need to use a larger number of options, you should put them in a separate reference table.


## Limitation of using BLOB / JSON fields

It is recommended to store data in MySQL in normalized form. However, in some cases, when it is necessary to save a
dynamic structure, or it is impossible to normalize data at the current level of knowledge about the subject area, it is
allowed to use JSON fields in tables. It should be remembered that such fields tend to grow, collecting more and more
attributes over time. When setting up data replication between DB instances, such fields create a large data flow,
which is fraught with lagging replicas and other difficulties.

When adding JSON fields to the table structure, it is necessary to limit the set of stored attributes at least at the
level of the API contract of the service being developed.


## Using Fake Values of External Entity Identifiers in the DB

Sometimes a situation may arise when a field in the DB structure is declared as `NOT NULL`, but at the same time, when
inserting data, we do not yet have information about the related entity and its identifier. Due to the lack of
verification by Foreign Key for such tables, it becomes possible to pass some special “fake” identifier to the DB,
which would act as a missing value. For example, make the value 0 or generate a “zero” UUID, which will be safely saved
in the table. Using this approach complicates the processing of such data at the level of analysts and the Data
Department, and also worsens the readability and support of the code.

Using fake identifiers is prohibited. In this situation, it is recommended to `ALTER` such a field and make it nullable
with a default value of `NULL`, which will correctly describe such a relationship and natively allow you to perform a
`JOIN`.


## Comments

To make it easier for other engineers to understand the purpose of tables and fields in the database, all migrations
should include comments on the entities being created, except for obvious ones like the id field. Comments should be
in English, begin with a capital letter, and briefly describe the purpose of the given entity. For example:

```sql
CREATE TABLE IF NOT EXISTS `user_vehicle_vertical_status`
(
`uuid`              BINARY(16)                  NOT NULL PRIMARY KEY,
`user_vehicle_uuid` BINARY(16)                  NOT NULL COMMENT 'UUID from user_vehicle table',
`status`            ENUM ('denied', 'approved') NOT NULL COMMENT 'WD1 application status on vertical',
`service`           VARCHAR(20)                 NOT NULL COMMENT 'Name of service where user is authorized to work on vehicle',
`created_at`        DATETIME                    NOT NULL DEFAULT NOW(),
`modified_at`       DATETIME                    NOT NULL DEFAULT NOW() ON UPDATE CURRENT_TIMESTAMP,
UNIQUE KEY `user_vehicle_uuid` (`user_vehicle_uuid`, `service`)
) COMMENT = 'This table normalizes data with WD1 by containing additional info on vertical authorization';
```

If the column values use different numeric codes (magic numbers), the `ENUM` type with a list of possible values should
be used for such a column. In such a case, the comment should provide brief descriptions of these values.

To simplify data governance tasks, comments can contain special tags. See the [Sensitive Data](#sensitive-data) section.


## Estimating the Number of Records in Tables

The `select count(*) from ...` queries are prohibited on the production environment, as this query creates an increased
load and can lead to a significant drop in DB performance. To estimate the table size in records, we recommend using
MySQL internal statistics, which will allow you to roughly estimate the table size without risks to performance:

```sql
select table_rows
  from information_schema.tables
 where table_schema = 'database_name'
   and table_name = 'table_name' ;
```

Another option for indirectly estimating the table size can be the `EXPLAIN` query. In this case, you need to pay
attention to the rows value - the number of rows that the query will have to process.


## Data Selection (`SELECT`)

* never use an expression like `select * from ...` - always explicitly specify the fields to select, even if all are
  needed;
* the selection must always be limited by the `WHERE` condition.
* queries must use limits and pagination to protect against unexpectedly large selections.


## Data Modification (`UPDATE`)

When performing data updates, you need to follow a few simple rules to avoid problems in a production environment. Large
updates of a large number of records should be performed in parts by the DBA. It is recommended to perform any updates
in a transaction with a manual `COMMIT/ROLLBACK`, controlling the number of changed records.

Performing update operations without `WHERE` is prohibited. Also, if you know in advance the number of records to be
changed, it is a good practice to use the `LIMIT` construct specifying this number to avoid massive data corruption in
case of an error.


### Modify Only Necessary Fields

For example, we need to update only `first_name`.

Bad:

```sql
update user set last_name = ?, first_name = ?, balance = ? where id = ?;
```

Good:

```sql
update user set first_name = ? where id = ?;
```

There may be a situation where someone else changes the user's balance in parallel, and you then overwrite the old
balance value, resulting in incorrect data in the DB.


### Change Incremental Values Via N = N + 1

Bad:

```sql
-- calculate the balance in the code
update user set balance = ? where id = ?;
```

Good:

```sql
-- increment balance in query
update user set balance = balance + ? where id = ?;
```

As in the previous case, it is necessary to take into account that the balance can change in parallel.


## Transactions

A transaction should take as little time as possible and affect only as much data as it needs. No blocking operations
are allowed while the method has an open transaction. This is especially true for synchronous network requests in a
transaction - this behavior is **strictly prohibited**! It is necessary to design the code in such a way that the
transaction works as quickly as possible.

To avoid lock wait time, please use `NOWAIT` clause: query will return error `couldn't obtain lock` in this case. Don't
forget to properly handle this error: retry it automatically or return an error to the user.

```sql
SELECT id, status FROM order WHERE id = ? FOR UPDATE NOWAIT
```


## Preparing Migrations

The general approach to changing the data structure can be described by the following points:

* migration should not break backward compatibility;
* migration should be idempotent if possible;
* migration should not lead to database degradation and should not affect the operation of the service.

You should always be aware that migration may lead to blocking or decreased service performance and take all necessary
efforts to minimize possible consequences: use online migration tools, have an understanding of INSTANT/INPLACE
operations and their applicability to our version of MySQL/Aurora. If in doubt, it is always worth consulting
[SQL Advocates](https://indriver.atlassian.net/wiki/spaces/~63bea50b2341bff4fff68770/pages/1680081111/SQL+Advocates).


### Blocking Operations

When planning migrations, you need to consider how the planned changes are performed: do they make changes only to
metadata or lead to a complete rebuild of the table; do they require locking operations or can they be performed
concurrently.

Also, you need to consider the amount of data in the DB table that needs to be changed, since even if the operation
does not lead to locks and the service continues to work normally, a long migration can lead to a Liquibase timeout.

More [details](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-operations.html).


### Backward Compatibility

Changes to the database must be made in such a way as to not break backward compatibility. If backward compatibility
is not possible, consider using a multiphase migration. Examples of backward compatible changes:

* adding a table;
* adding a `NULLABLE` column;
* adding a `NOT NULL` column if there is a `DEFAULT` value;
* creating an index;
* changing the column dimension to increase;
* adding partitioning to a table;
* adding a Set or Enum to the list of possible values.

Examples of operations that break backward compatibility:

* drop a table or a column;
* rename a table or a column;


### Deleting a Column or Table

It is recommended not to delete an object from the database directly, as this may result in the service not working and
the inability to roll back to the previous version due to incompatibility of the data structure.

It is recommended to perform such operations in two phases:

1. Renaming a column/table and launching a new version of the service to assess the functionality of the build. If
   problems were detected at this step, the original column name can be instantly returned back, and the data will not be
   lost.
2. If no problems were detected at step 1, you can perform the DROP operation.


### DML in Migrations

In general, it is better to refrain from placing DML queries in Liquibase migrations. Migrations can only be used to
fill static reference books. `INSERT` operations are safe for migrations, but you should carefully check the number of
records being changed if you need to perform an `UPDATE` or `DELETE` - this can lead to blocking.

It is better to perform large DML queries in batches using a DBA.


## Sensitive Data

It is `PROHIBITED` to accumulate and process personal data in the service database without prior approval. The process of
managing, protecting and coordinating the need to work with PD in all cases without exception is described in
[personal information handling](../../../../software-architecture/personal-information-handling.md)
“Personal Information Handling”.

If the creation/addition of a column containing PD has been approved, these objects must be marked with a special #PPI
tag in the comment. Similarly, the creation/addition of a column containing geo data must be marked with a special `#GEO`
tag in the comment. Open Metadata Platform will automatically collect the markup from the comments, which will allow us
to track the path of PD through the inDrive infrastructure.


## Integration with DWH

If change capture is configured for the objects being changed using CDC (Debezium), you should pay attention to whether
the changes being made break the compatibility of the data structure with the DWH. Pay special attention and alert the
data engineering team if:

1. DROP of tables is performed
2. DROP of columns with data is performed
3. Change of the column data type is performed without backward compatibility (for example, varchar to int)


## Data Normalization

[Data normalization](https://en.wikipedia.org/wiki/Database_normalization) works poorly in the case of a loaded database.

Bad:

* table `region` has columns (`id`, `name`);
* table `city` has columns (`id`, `name`, `region_id`);
* table `user` stores (`id`, `user_name`, `city_id`);
* to find out region we have to use `join city`.

Good:

* table `region` has columns (`id`, `name`);
* table `city` has columns (`id`, `name`, `region_id`);
* table `user` stores (`id`, `user_name`, `city_id`, `region_id`);
* user region is already known.

So if it is known that there will be many typical requests, it is better to follow the principle **if the data changes
simultaneously (in the example, when `city_id` changes, `region_id` changes and vice versa), it is better to store
them together**.


## Data Separation

If the data belongs to one entity and will be used only within this entity, there is no need to allocate this data to
a separate table.

Example, there is an entity user:

| `user`       |
|--------------|
| `id`         |
| `first_name` |
| `last_name`  |
| `city_id`    |

You need to add payment methods. You can add two tables:

| **`payment_provider`**  |
|-------------------------|
| `id`                    |
| `name`                  |

Or you can add the link to the `user` table if you know that the payment methods will be used only with `user`.

| **`column`**  | **`type`**  | **`example`**           |
|---------------|-------------|-------------------------|
| `id`          | `int`       |                         |
| `first_name`  | `varchar`   |                         |
| `last_name`   | `varchar`   |                         |
| `city_id`     | `int`       |                         |
| `jdoc`        | `json`      | `{"providers": [1, 4]}` |


As a result, there is no need to do a `join` to get the payment methods


## New Columns

If the number of columns starts to exceed a certain critical threshold consider two options:

* add a json field;
* still split the table into different entities.


## Virtual Columns

MySQL allows to create virtual columns and create indices over them. For example, if you have json field with a
`customer_id` inside, you could create a virtual column and create an index over it.

```sql
`contractor_id`      bigint GENERATED ALWAYS AS (json_unquote(json_extract(`contractor`, _utf8mb4'$.id'))) VIRTUAL,
…
KEY `idx_contractor_id` (`contractor_id`)
```

## Read and Write Load

It's wrong when you write to master and read from replicas due to possible issues with replication lag.
So you could read from replica only information that could be outdated and that doesn't affect the business logic.
When you need account balance or order status, you should read from the master.


## Smaller Table - Faster Query

Table should be separated for Hot and Cold data. Hot data is data that is often used and Cold data is data that is
rarely used. Hot data should be in a separate table, so the query will be faster. For example, order stores in Hot table
since it created and moves to Cold storage when it is fulfilled or canceled.


## Analytical Tables

Don't use RDBMS for analytical tasks - it's very expensive solution for it. Use ClickHouse, Kafka or similar tools.


## Don't Use ORM

ORM is a universal tool that is poorly compatible with high loads. It helps to quickly write code, but it slows down
everything else. It is better to write simple queries and logic in the application. The only exception is the use of
ORM for internal admin panel with low load.


## Triggers and Stored Procedures

Don't use triggers and Stored procedures, because they:

* spread the logic between app and database;
* hard to support.