---
title: WINDOW FUNCTIONS
summary: A window function performs a calculation across a set of table rows that are somehow related to the current row.
toc: false
---

Window functions are used to process several items from a query's result set at a time.  They take as input several rows and compute a single output value, just as [aggregate functions][agg] do. You can use the predefined aggregate functions or define your own.

They are called "window" functions because they operate on a subset of the rows selected by a query as if they had a sliding "window" onto the data being processed.

Depending on which function is used, you can access rows from the result that are ahead or behind of the current row (see: `lag()` and `lead()`)

<div id="toc"></div>

## Terms

- _Partition_: in the context of window functions, the partition we are referring to is a partition into groups that the window function operates on - not [`ALTER TABLE ... PARTITION BY`](partition-by.html) or anything to do with network partitions in the distributed systems sense.

- _Virtual table_:

- _Window frame_: 

## How window functions work

## Examples

The examples in this section use the "users" and "rides" tables from the 'movr' database used in the [CockroachDB 2.0 demo][demo].  The table layouts are are shown below.

We are planning to open source the movr data set in the 2.1 release timeframe.  When it's out we will update this page with a link to the repo.

~~~
+-------+-------------------------------------------------------------+    +-------+--------------------------------------------------------------------------+
| Table |                         CreateTable                         |    | Table |                               CreateTable                                |
+-------+-------------------------------------------------------------+    +-------+--------------------------------------------------------------------------+
| users | CREATE TABLE users (                                        |    | rides | CREATE TABLE rides (                                                     |
|       |     id UUID NOT NULL,                                       |    |       |     id UUID NOT NULL,                                                    |
|       |     city STRING NOT NULL,                                   |    |       |     city STRING NOT NULL,                                                |
|       |     name STRING NULL,                                       |    |       |     vehicle_city STRING NULL,                                            |
|       |     address STRING NULL,                                    |    |       |     rider_id UUID NULL,                                                  |
|       |     credit_card STRING NULL,                                |    |       |     vehicle_id UUID NULL,                                                |
|       |     CONSTRAINT "primary" PRIMARY KEY (city ASC, id ASC),    |    |       |     start_address STRING NULL,                                           |
|       |     FAMILY "primary" (id, city, name, address, credit_card) |    |       |     end_address STRING NULL,                                             |
|       | )                                                           |    |       |     start_time TIMESTAMP NULL,                                           |
+-------+-------------------------------------------------------------+    |       |     end_time TIMESTAMP NULL,                                             |
                                                                           |       |     revenue FLOAT NULL,                                                  |
                                                                           |       |     CONSTRAINT "primary" PRIMARY KEY (city ASC, id ASC),                 |
                                                                           |       |     CONSTRAINT fk_city_ref_users FOREIGN KEY (city, rider_id) REFERENCES |
                                                                           |       | users (city, id),                                                        |
                                                                           |       |     INDEX rides_auto_index_fk_city_ref_users (city ASC, rider_id ASC),   |
                                                                           |       |     CONSTRAINT fk_vehicle_city_ref_vehicles FOREIGN KEY (vehicle_city,   |
                                                                           |       | vehicle_id) REFERENCES vehicles (city, id),                              |
                                                                           |       |     INDEX rides_auto_index_fk_vehicle_city_ref_vehicles (vehicle_city    |
                                                                           |       | ASC, vehicle_id ASC),                                                    |
                                                                           |       |     FAMILY "primary" (id, city, vehicle_city, rider_id, vehicle_id,      |
                                                                           |       | start_address, end_address, start_time, end_time, revenue),              |
                                                                           |       |     CONSTRAINT check_vehicle_city_city CHECK (vehicle_city = city)       |
                                                                           |       | )                                                                        |
                                                                           +-------+--------------------------------------------------------------------------+
~~~

To see which customers have generated the most revenue, run:

{% include copy-clipboard.html %}
~~~ sql
SELECT DISTINCT name,
  SUM(revenue) over (partition BY name) AS "total rider revenue"
  FROM users JOIN rides ON users.id = rides.rider_id
  ORDER BY "total rider revenue" DESC
  LIMIT 10;
~~~

~~~
+------------------+---------------------+
|       name       | total rider revenue |
+------------------+---------------------+
| Michael Jones    |  465.25328956751196 |
| James Jones      |   350.0380740792208 |
| Michael Thompson |  329.07722514046185 |
| Steven Lane      |  308.08198141368615 |
| Jennifer Davis   |  297.05709764285797 |
| John Hernandez   |  286.40393951577124 |
| David Smith      |  282.10831776351614 |
| Vanessa Brown    |  281.73030695407414 |
| James Baker      |  280.59061211182495 |
| Robert Hernandez |  264.51810028779965 |
+------------------+---------------------+
(10 rows)
~~~

To see which customers have generated the most revenue and also have taken more than 3 rides, do:

{% include copy-clipboard.html %}
~~~ sql
SELECT * FROM (
  SELECT DISTINCT name,
    COUNT(*)     OVER w AS "number of rides",
    AVG(revenue) OVER w AS "average revenue per ride"
    FROM users JOIN rides ON users.ID = rides.rider_id
    WINDOW w AS (PARTITION BY name)
  )
  WHERE "number of rides" >= 3
  ORDER BY "number of rides",
           "average revenue per ride" DESC
  LIMIT 10;
~~~

~~~
+------------------+-----------------+--------------------+
|       name       | number of rides | revenue per rider  |
+------------------+-----------------+--------------------+
| Michael Jones    |               8 | 465.25328956751196 |
| James Jones      |               5 |  350.0380740792208 |
| Michael Thompson |               5 | 329.07722514046185 |
| Steven Lane      |               4 | 308.08198141368615 |
| Jennifer Davis   |               6 | 297.05709764285797 |
| John Hernandez   |               4 | 286.40393951577124 |
| David Smith      |               4 | 282.10831776351614 |
| Vanessa Brown    |               4 | 281.73030695407414 |
| James Baker      |               4 | 280.59061211182495 |
| Robert Hernandez |               3 | 264.51810028779965 |
+------------------+-----------------+--------------------+
(10 rows)
~~~

To see which customers have the highest average revenue per ride, run:

{% include copy-clipboard.html %}
~~~ sql
SELECT name,
  COUNT(*)     OVER w AS "number of rides",
  AVG(revenue) OVER w AS "average revenue per ride"
  FROM users JOIN rides ON users.ID = rides.rider_id
  WINDOW w AS (PARTITION BY name)
  ORDER BY "average revenue per ride" DESC, "number of rides" ASC
  LIMIT 10;
~~~

~~~
+-------------------+-----------------+--------------------------+
|       name        | number of rides | average revenue per ride |
+-------------------+-----------------+--------------------------+
| Ann Johnson       |               1 |        99.99940404943563 |
| Zachary Terry     |               1 |        99.99713547304452 |
| Michelle Johnson  |               1 |        99.99284158049196 |
| Walter Blake      |               1 |        99.98980862800644 |
| Robert Carr       |               1 |         99.9733715337187 |
| Jeffery Walker    |               1 |        99.95812364841187 |
| Kristin Smith     |               1 |         99.9472029441925 |
| Stephanie Sharp   |               1 |        99.94129781533725 |
| Reginald Schwartz |               1 |        99.92882925427865 |
| Lisa Ross         |               1 |        99.92666994099285 |
+-------------------+-----------------+--------------------------+
(10 rows)
~~~

To see which customers have the highest average revenue per ride, given that they have taken at least 3 rides, run:

{% include copy-clipboard.html %}
~~~ sql
SELECT * FROM (
  SELECT DISTINCT name,
    COUNT(*)     OVER w AS "number of rides",
    AVG(revenue) OVER w AS "average revenue per ride"
    FROM users JOIN rides ON users.ID = rides.rider_id
    WINDOW w AS (PARTITION BY name)
  )
  WHERE "number of rides" >= 3
  ORDER BY "number of rides",
           "average revenue per ride" DESC
  LIMIT 10;
~~~

~~~
+---------------------+-----------------+--------------------------+
|        name         | number of rides | average revenue per ride |
+---------------------+-----------------+--------------------------+
| Robert Hernandez    |               3 |        88.17270009593322 |
| Bonnie Marquez      |               3 |         86.7086579240223 |
| Karen Wright        |               3 |        81.93695939301487 |
| Jessica Woods       |               3 |        79.97061480750095 |
| Joseph Rodriguez    |               3 |         79.7649275479433 |
| James Thomas        |               3 |        78.82526279198679 |
| Angela Lewis        |               3 |        78.36877495433147 |
| Christopher Johnson |               3 |        76.96337251225161 |
| Michael Pena        |               3 |        72.01317327473508 |
| Maria Nelson        |               3 |         70.0469578465542 |
+---------------------+-----------------+--------------------------+
(10 rows)
~~~

We can reuse and edit the query from #XXX to find out the total number of riders and total revenue generated thus far by the app.

{% include copy-clipboard.html %}
~~~ sql

SELECT
  COUNT("name") AS "total # of riders",
  SUM("total rider revenue") AS "total revenue" FROM (
    SELECT name,
           SUM(revenue) over (partition BY name) AS "total rider revenue"
      FROM users JOIN rides ON users.id = rides.rider_id
      ORDER BY "total rider revenue" DESC
      LIMIT (SELECT count(distinct(rider_id)) FROM rides)
);
~~~

~~~
+-------------------+-------------------+
| total # of riders |   total revenue   |
+-------------------+-------------------+
|              9511 | 630401.0617159915 |
+-------------------+-------------------+ 
~~~

## See Also

- [Simple `SELECT` clause][simple-select]
- [Selection Queries][selection-query]
- [Aggregate functions][agg]
- [CockroachDB 2.0 Demo][demo]

<!-- References -->

[agg]: functions-and-operators.html#anyelement-functions
[demo]: https://www.youtube.com/watch?v=v2QK5VgLx6E
[simple-select]: select-clause.html
[selection-query]: selection-queries.html
