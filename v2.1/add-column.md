---
title: ADD COLUMN
summary: Use the ADD COLUMN statement to add columns to tables.
toc: false
---

The `ADD COLUMN` [statement](sql-statements.html) is part of `ALTER TABLE` and adds columns to tables.

<div id="toc"></div>

## Synopsis

{% include sql/{{ page.version.version }}/diagrams/add_column.html %}

## Required privileges

The user must have the `CREATE` [privilege](privileges.html) on the table.

## Parameters

| Parameter | Description |
|-----------|-------------|
| `table_name` | The name of the table to which you want to add the column. |
| `column_name` | The name of the column you want to add. The column name must follow these [identifier rules](keywords-and-identifiers.html#identifiers) and must be unique within the table but can have the same name as indexes or constraints.  |
| `typename` | The [data type](data-types.html) of the new column. |
| `col_qualification` | An optional list of column definitions, which may include [column-level constraints](constraints.html), [collation](collate.html), or [column family assignments](column-families.html).<br><br>Note that it is not possible to add a column with the [`FOREIGN KEY`](foreign-key.html) constraint. As a workaround, you can add the column without the constraint, then use [`CREATE INDEX`](create-index.html) to index the column, and then use [`ADD CONSTRAINT`](add-constraint.html) to add the `FOREIGN KEY` constraint to the column. |

## Viewing schema changes

{% include custom/schema-change-view-job.md %}

## Examples

### Add a single column

{% include copy-clipboard.html %}
~~~ sql
> ALTER TABLE accounts ADD COLUMN names STRING;
~~~

{% include copy-clipboard.html %}
~~~ sql
> SHOW COLUMNS FROM accounts;
~~~

~~~
+-----------+-------------------+-------+---------+-----------+
|   Field   |       Type        | Null  | Default |  Indices  |
+-----------+-------------------+-------+---------+-----------+
| id        | INT               | false | NULL    | {primary} |
| balance   | DECIMAL           | true  | NULL    | {}        |
| names     | STRING            | true  | NULL    | {}        |
+-----------+-------------------+-------+---------+-----------+
~~~

### Add multiple columns

{% include copy-clipboard.html %}
~~~ sql
> ALTER TABLE accounts ADD COLUMN location STRING, ADD COLUMN amount DECIMAL;
~~~

{% include copy-clipboard.html %}
~~~ sql
> SHOW COLUMNS FROM accounts;
~~~

~~~
+-----------+-------------------+-------+---------+-----------+
|   Field   |       Type        | Null  | Default |  Indices  |
+-----------+-------------------+-------+---------+-----------+
| id        | INT               | false | NULL    | {primary} |
| balance   | DECIMAL           | true  | NULL    | {}        |
| names     | STRING            | true  | NULL    | {}        |
| location  | STRING            | true  | NULL    | {}        |
| amount    | DECIMAL           | true  | NULL    | {}        |
+-----------+-------------------+-------+---------+-----------+

~~~

### Add a column with a `NOT NULL` constraint and a `DEFAULT` value

{% include copy-clipboard.html %}
~~~ sql
> ALTER TABLE accounts ADD COLUMN interest DECIMAL NOT NULL DEFAULT (DECIMAL '1.3');
~~~

{% include copy-clipboard.html %}
~~~ sql
> SHOW COLUMNS FROM accounts;
~~~
~~~
+-----------+-------------------+-------+---------------------------+-----------+
|   Field   |       Type        | Null  |          Default          |  Indices  |
+-----------+-------------------+-------+---------------------------+-----------+
| id        | INT               | false | NULL                      | {primary} |
| balance   | DECIMAL           | true  | NULL                      | {}        |
| names     | STRING            | true  | NULL                      | {}        |
| location  | STRING            | true  | NULL                      | {}        |
| amount    | DECIMAL           | true  | NULL                      | {}        |
| interest  | DECIMAL           | false | ('1.3':::STRING::DECIMAL) | {}        |
+-----------+-------------------+-------+---------------------------+-----------+
~~~

### Add a column with `NOT NULL` and `UNIQUE` constraints

{% include copy-clipboard.html %}
~~~ sql
> ALTER TABLE accounts ADD COLUMN cust_number DECIMAL UNIQUE NOT NULL;
~~~

### Add a column with collation

{% include copy-clipboard.html %}
~~~ sql
> ALTER TABLE accounts ADD COLUMN more_names STRING COLLATE en;
~~~

### Add a column and assign it to a column family

#### Add a column and assign it to a new column family

{% include copy-clipboard.html %}
~~~ sql
> ALTER TABLE accounts ADD COLUMN location1 STRING CREATE FAMILY new_family;
~~~

#### Add a column and assign it to an existing column family

{% include copy-clipboard.html %}
~~~ sql
> ALTER TABLE accounts ADD COLUMN location2 STRING FAMILY existing_family;
~~~

#### Add a column and create a new column family if column family does not exist

{% include copy-clipboard.html %}
~~~ sql
> ALTER TABLE accounts ADD COLUMN new_name STRING CREATE IF NOT EXISTS FAMILY f1;
~~~

## See also
- [`ALTER TABLE`](alter-table.html)
- [Column-level Constraints](constraints.html)
- [Collation](collate.html)
- [Column Families](column-families.html)
