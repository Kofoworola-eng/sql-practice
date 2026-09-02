# SQL Joins: Combining Data from Multiple Tables

Notes from Data With Baraa SQL course, Module: SQL Joins
Status: In progress. Covered so far: NO JOIN, INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL JOIN.
Still to come: LEFT ANTI JOIN, RIGHT ANTI JOIN, FULL ANTI JOIN, CROSS JOIN, choosing the right join, multiple table joins.

## What is Data Combining?

So far, every query has worked with a single table at a time. In real
databases, information is usually split across multiple tables. For
example, customer details might live in a `customers` table, while
their purchases live in a separate `orders` table. To answer a
question like "which customers placed which orders," the data from
both tables needs to be combined into a single result. This is what
`JOIN` does.

A join connects rows from two tables based on a matching column,
usually a shared identifier like an id. In the examples below,
`customers.id` matches `orders.customer_id`, which is what allows SQL
to line up the correct customer with the correct order.

## Table Aliases

Before getting into joins themselves, it helps to understand table
aliases, since every join query uses them.

```sql
FROM customers AS c
INNER JOIN orders AS o
    ON c.id = o.customer_id;
```

`AS c` and `AS o` give the tables short nicknames for the rest of the
query. Instead of writing `customers.id` and `orders.customer_id`
everywhere, you can write `c.id` and `o.customer_id`. This is
especially useful in joins, since both tables might have columns with
the same name (like `id`), and the alias makes it clear which table a
column is coming from.

## No Join (Retrieving Separately)

Before combining data, it is worth seeing what happens without a
join. You can query two tables separately, but you get two separate
result sets, not one combined view.

```sql
-- Retrieve all data from customers & orders in 2 different results
SELECT *
FROM customers;

SELECT *
FROM orders;
```

This does not combine anything. It just shows the full customers
table and the full orders table as two unrelated outputs. This is the
starting point that joins are meant to improve on.

## INNER JOIN

`INNER JOIN` returns only the rows where there is a match in both
tables. If a customer has not placed any order, that customer is left
out of the result entirely, since there is nothing in the orders
table to match them with.

```sql
-- Get all customers along with their orders,
-- but only for customers who have placed an order
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.sales
FROM customers AS c
INNER JOIN orders AS o
    ON c.id = o.customer_id;
```

The `ON` clause defines how the two tables are connected. Here, a row
from `customers` is joined to a row from `orders` only when
`c.id = o.customer_id`. If a customer id does not appear anywhere in
`orders.customer_id`, that customer simply will not show up in the
result at all.

Think of `INNER JOIN` as the intersection: only the overlap between
the two tables is returned.

## LEFT JOIN

`LEFT JOIN` returns all rows from the "left" table (the one listed
first, right after `FROM`), along with any matching rows from the
"right" table. If there is no match on the right side, the columns
from the right table are simply filled with `NULL` instead of the row
being excluded.

```sql
-- Get all customers along with their orders
-- including those without orders
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.sales
FROM customers AS c
LEFT JOIN orders AS o
    ON c.id = o.customer_id;
```

Here, `customers` is the left table because it comes right after
`FROM`. Every customer appears in the result, whether or not they
have placed an order. For a customer with no orders, `order_id` and
`sales` will show as `NULL` in their row, rather than that customer
being dropped from the results.

This is the key difference from `INNER JOIN`: nobody from the left
table gets excluded, even if there is no match on the right.

## RIGHT JOIN

`RIGHT JOIN` works the same way as `LEFT JOIN`, but flipped. It
returns all rows from the "right" table (the one being joined in),
along with any matching rows from the left table. Rows from the left
table with no match are excluded; rows from the right table with no
match still appear, with `NULL` in place of the missing left-side
columns.

```sql
-- Get all customers along with their orders including orders
-- without matching customers
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.sales
FROM orders AS o
LEFT JOIN customers AS c
    ON c.id = o.customer_id;
```

Note that this example actually uses `LEFT JOIN`, but with the tables
swapped around (`orders` is now first, so it acts as the left table).
This produces the same effect as a `RIGHT JOIN` would if `customers`
were kept as the right table. This is a useful thing to notice: a
`RIGHT JOIN` can always be rewritten as a `LEFT JOIN` by swapping
which table is listed first. Because of this, many people prefer to
stick to `LEFT JOIN` everywhere for consistency, and simply reorder
the tables instead of switching keywords.

In this query, every order appears in the result, even if that order
somehow has no matching customer (which would show as `NULL` in the
customer columns). Orders are the priority here; customers are only
included if they match an order.

## FULL JOIN

`FULL JOIN` (also called `FULL OUTER JOIN`) returns all rows from
both tables, matching them up where possible, and filling in `NULL`
on whichever side has no match. Nothing from either table is
excluded.

```sql
-- Get all customers and all orders, even if there's no match
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.sales
FROM orders AS o
FULL JOIN customers AS c
    ON c.id = o.customer_id;
```

This is the most inclusive of the joins covered so far. Every
customer appears, whether or not they have an order. Every order
appears, whether or not it has a matching customer. Where a match
exists, the row is combined normally. Where no match exists on one
side, that side's columns are filled with `NULL`.

## Comparing the Joins So Far

| Join type | Customers with no orders | Orders with no customer |
|---|---|---|
| NO JOIN | shown separately, not combined | shown separately, not combined |
| INNER JOIN | excluded | excluded |
| LEFT JOIN (customers first) | included, NULLs for order columns | excluded |
| RIGHT JOIN / reordered LEFT JOIN | excluded | included, NULLs for customer columns |
| FULL JOIN | included, NULLs for order columns | included, NULLs for customer columns |

The pattern to hold onto: without a join, the tables are not related
to each other at all. `INNER JOIN` keeps only the overlap. `LEFT JOIN`
keeps everything on the left no matter what. `RIGHT JOIN` keeps
everything on the right no matter what. `FULL JOIN` keeps everything
from both sides no matter what.

## Key Takeaways So Far

- Without a join, querying two tables just gives two separate,
  unrelated result sets. A join is what actually connects the data
  from both tables into one combined result.
- A join combines rows from two tables based on a matching column,
  specified in the `ON` clause.
- Table aliases (`AS c`, `AS o`) make queries with multiple tables
  much easier to read, especially when both tables share column
  names like `id`.
- `INNER JOIN` only returns rows that have a match in both tables.
- `LEFT JOIN` returns everything from the left table, with `NULL`
  filled in where there is no match on the right.
- `RIGHT JOIN` is the mirror of `LEFT JOIN`, and can always be
  reproduced using `LEFT JOIN` by swapping which table is listed
  first in the `FROM` clause.
- `FULL JOIN` returns everything from both tables, matching where
  possible and filling `NULL` where there is no match on either side.

## Still To Cover

The next parts of this module will cover anti joins (which return
only the rows that do not have a match, essentially the opposite of
what an inner join keeps), `CROSS JOIN` (which combines every row of
one table with every row of another, regardless of any matching
condition), how to choose the right join for a given situation, and
how to join more than two tables together in a single query. These
will be added to this notes file once completed.