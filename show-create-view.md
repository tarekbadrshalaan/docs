---
title: SHOW CREATE VIEW
summary: The SHOW CREATE VIEW statement shows the CREATE VIEW statement that would create a carbon copy of the specified view. 
toc: false
---

The `SHOW CREATE VIEW` [statement](sql-statements.html) shows the `CREATE VIEW` statement that would create a carbon copy of the specified [view](views.html).

<div id="toc"></div>

## Required Privileges

The user must have any [privilege](privileges.html) on the target view as well as the `SELECT` privilege on any table(s) referenced by the view.

## Synopsis

{% include sql/diagrams/show_create_view.html %}

## Parameters

Parameter | Description
----------|------------
`view_name` | The name of the view for which to show the `CREATE VIEW` statement.

## Response

Field | Description
------|------------
`View` | The name of the view.
`CreateView` | The [`CREATE VIEW`](create-view.html) statement for creating a carbon copy of the specified view. 

## Example

~~~ sql
> SHOW CREATE VIEW bank.user_accounts;
~~~

~~~
+--------------------+---------------------------------------------------------------------------+
|        View        |                                CreateView                                 |
+--------------------+---------------------------------------------------------------------------+
| bank.user_accounts | CREATE VIEW "bank.user_accounts" AS SELECT type, email FROM bank.accounts |
+--------------------+---------------------------------------------------------------------------+
(1 row)
~~~

## See Also

- [Views](views.html)
- [`CREATE VIEW`](create-view.html)
- [`ALTER VIEW`](alter-view.html)
- [`DROP VIEW`](drop-view.html)