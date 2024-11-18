# Database Guideline

Databases in inDrive are subject to high loads, which imposes certain requirements on the development of queries and data structure migrations. This guideline provides the main recommendations for naming entities, making changes, and performance considerations that engineers should pay attention to during the development process.

Following this guideline will help pass SQL reviews faster. Prepared migrations should always be run through SQLLint.


## Naming Rules for Entities

The general recommendation for naming all entities is to use common words that do not allow ambiguous interpretations. For example, use the term `gender` instead of `sex` for storing gender data. Avoid naming conflicts with reserved words from the SQL dialect being used.


### Table Names

Tables should be named with a noun in English indicating the main entity in the singular. For compound names, words are separated by underlining.

Good:

* `order`;
* `vehicle_details` (since each row in the table stores a set of attributes and options of the car, plural is allowed).

Bad:

* `orders`;
* `rides_orders`;
* `VehicleDetails`;
* `vehicle_detail`.


### Column Names

Table columns should be named with a singular English noun. For compound names, words are separated by underscores. For storing unstructured information, use the reserved field name - `jdoc`. Don't use [reserved words](https://dev.mysql.com/doc/refman/8.4/en/keywords.html).

When naming table fields, use standard prefixes and suffixes:

* `is_` for logical fields;
* `_by` for event sources;
* `_at` for timestamps.

Good:

* `id`;
* `uuid`;
* `order_id` (if it's a link to the order in another table);
* `document_text`;
* `created_at`;
* `updated_by`.

Bad:

* `order_ids`;
* `text`;
* `creation_date`;
* `updated_by_user`.


### Index Names

It is recommended to make up the index name from the table name, field(s), and the `_idx` suffix. If the name is too long, abbreviations are allowed that do not lead to loss of information about the index’s purpose.


### Key and Constraint Names

It is recommended to make up the name from the field(s) name and the `_key` suffix. If the name is too long, abbreviations are allowed that do not lead to loss of information about the purpose of this index.


### Foreign Keys

Despite certain advantages and data integrity control, the use of foreign keys in DB schemas is prohibited because they introduce a large number of restrictions on online DDL execution. More information can be found [here](https://code.openark.org/blog/mysql/the-problem-with-mysql-foreign-key-constraints-in-online-schema-changes).


## Indices

The table must always have a `PRIMARY (UNIQUE) key`. This is critically important for replication. The primary key must be compact. It is recommended to avoid zero (`NULL`) values in key columns, as nullable fields require more space and additional calculations inside `innodb`.

It is important to note that the standard query `create index ...` does not perform an online migration but rather acts directly on the table. When creating an index on a large table, this can lead to blocking. It is safer to use the following approach:

```sql
ALTER TABLE table_name ADD INDEX ...
```

### Cardinality and Selectivity

The index will work well only if the column it is created on has a sufficient number of distinct values (high cardinality), which helps reduce the volume of data selected. Cardinality can be checked with the following command:

```sql
SHOW INDEX FROM tbl_name;
```

In rare cases, the cardinality value can be disregarded if there is a clear understanding that the column has a strong data skew (e.g., 95/5 distribution) and only the smaller values are needed in queries. In such cases, creating an index on the column may yield high selectivity.

For more details, refer to this [article](https://webmonkeyuk.wordpress.com/2010/09/27/what-makes-a-good-mysql-index-part-2-cardinality/).


### B-tree Indices

* well suited for most search operations (=, >, <, >=, <=, between);
* works for `LIKE` searches if searching from the beginning (e.g., `LIKE 'word%'`, but not `LIKE '%word'`).


### Hash Indices

* work only for equality conditions (=, <>), but are very fast;
* cannot be used in `ORDER BY`.


### Composite Keys

Prefer one composite key to several simple ones:

* the composite key is used from left to right;
* place the most selective fields of the composite key at the beginning;
* selectivity: keys with a selection of +50% of the table are **useless**. MySQL does not have partial keys.

Avoid duplicates: when one index contains another index entirely.

Example — the first key is redundant because the second contains all necessary fields:

```sql
KEY `city_id` (`city_id`),
KEY `city_id_type_status` (`city_id`,`type`,`status`)
```

When using `OR`, ensure each group by `OR` uses an indexed field in the same index.

Suppose there is a composite index on columns (`index_part1`, `index_part2`, `index_part3`).

These queries will use the index:

```sql
-- index_part1, index_part2 will be used
... WHERE index_part1=1 AND index_part2=2 AND other_column=3

-- index_part1 = 1 on the left and index_part2 = 2 on the right will be used
... WHERE (index_part1=1 OR other_column=10) AND index_part2=2

-- only "index_part1='hello'" but not index_part3 will be used
... WHERE index_part1='hello' AND index_part3=5

-- only index_part1 will be used, as it is in both parts of OR
... WHERE (index_part1=1 AND index_part2=2) OR (index_part1=3 AND index_part3=3);
```

And here the index will not be used:

```sql
-- index_part1 is not used, so the index does not work
... WHERE index_part2=1 AND index_part3=2

-- there is no index for other_column=10, so the entire expression will not use the index
... WHERE index_part1=1 OR other_column=10

-- there is no condition for index_part1 on the right
... WHERE index_part1=1 OR index_part2=10
```

Always use `EXPLAIN` to check index performance. In small selections, `EXPLAIN` may work incorrectly.


### Key Selections

* equality is the best choice:
  * `WHERE column = value`;
  * `WHERE column IN (value1, value2)`.

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


### Choosing the Identifier Format: Number vs UUID

It is recommended to use [UUIDv7](https://dev.indriver.io/components/architecture-docs/main/documentation/adrs/0003-ARCH-205-uuidv7-primary-key-selection) or [UUIDv8](https://dev.indriver.io/components/architecture-docs/main/documentation/adrs/0009-ARCH-383-universal-entity-identifier) in `BINARY(16)` format for the primary key in the table. The identifiers of both versions already contain the record creation time, so for data consistency, create a virtual `created_at` column whose values are derived from the identifier. For example:

```sql
`created_at` DATETIME GENERATED ALWAYS AS (from_unixtime(conv(hex(`user_uuid` >> 80),16,10) / 1000)) VIRTUAL
```

The primary key in `NUMBER` format may be used for reference tables with slow-changing, region-neutral data, avoiding data merging and identifier conflicts.