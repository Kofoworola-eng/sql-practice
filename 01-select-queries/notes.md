# SQL Basics: SELECT Queries

Notes from Data With Baraa SQL course, Module: SELECT Queries

## What is a SQL Query?

A SQL query is a set of instructions given to a database asking it to
retrieve, filter, sort, or summarize data. A query doesn't change the
data by default. It just asks the database to return a result set
based on the conditions you specify.

## Components of SQL

A typical SQL statement is built from clauses, each with a specific job:

- `SELECT`: which columns to return
- `FROM`: which table to pull data from
- `WHERE`: which rows to include, based on a condition
- `GROUP BY`: how to group rows together for aggregation
- `HAVING`: which groups to include, after grouping
- `ORDER BY`: how to sort the final result

Not every query needs every clause, but when they're used together,
order matters (see Execution Order below).

## SELECT & FROM

`SELECT` specifies the columns you want to see. `FROM` specifies the
table those columns come from.

```sql
SELECT first_name, country, score
FROM customers;
```

Using `SELECT *` returns all columns in the table, useful for quick
exploration, but in real projects it's better to name only the
columns you need. It's more readable and less wasteful on large
tables.

## WHERE

`WHERE` filters rows based on a condition. Only rows where the
condition is true are returned.

```sql
SELECT *
FROM customers
WHERE country = 'Germany';
```

Common operators used in `WHERE`:
- `=` equal to
- `!=` or `<>` not equal to
- `>`, `<`, `>=`, `<=` comparison
- `AND`, `OR` to combine multiple conditions

Example, filtering out a specific value:

```sql
SELECT *
FROM customers
WHERE score != 0;
```

## ORDER BY

`ORDER BY` sorts the result set. `DESC` sorts highest to lowest (or
Z to A). `ASC` sorts lowest to highest (or A to Z). `ASC` is the
default if you don't specify.

```sql
SELECT *
FROM customers
ORDER BY score DESC;
```

You can sort by multiple columns. SQL sorts by the first column
listed, then uses the second column to break ties within that:

```sql
SELECT *
FROM customers
ORDER BY
    country ASC,
    score DESC;
```

This sorts customers alphabetically by country, and within each
country, by highest score first.

## GROUP BY

`GROUP BY` collapses multiple rows into a summary row per group. Used
together with aggregate functions like `SUM`, `AVG`, `COUNT`, `MIN`,
`MAX`.

```sql
SELECT
    country,
    SUM(score) AS total_score,
    COUNT(id) AS total_customers
FROM customers
GROUP BY country;
```

This returns one row per country, showing the total score and total
number of customers in that country, not one row per customer.

Every column in the `SELECT` list that isn't an aggregate function
must appear in the `GROUP BY` clause.

## HAVING

`HAVING` filters groups after aggregation. It's the equivalent of
`WHERE`, but for grouped or aggregated data rather than individual
rows.

```sql
SELECT
    country,
    AVG(score) AS avg_score
FROM customers
WHERE score != 0
GROUP BY country
HAVING AVG(score) > 430;
```

**Key distinction: WHERE vs HAVING**
- `WHERE` filters individual rows before grouping happens.
- `HAVING` filters groups after aggregation has already happened.

This is why `WHERE score != 0` excludes rows before they're grouped,
while `HAVING AVG(score) > 430` excludes whole country groups based
on their calculated average.

## DISTINCT

`DISTINCT` removes duplicate values from the result, returning only
unique entries.

```sql
SELECT DISTINCT country
FROM customers;
```

If there are 100 customers across 5 countries, this returns just the
5 unique country names, not 100 rows.

## TOP

`TOP` limits the number of rows returned. On its own, it just grabs
the first N rows in whatever order the data happens to be stored,
which is essentially arbitrary.

```sql
SELECT TOP 3 *
FROM customers;
```

`TOP` becomes meaningful when combined with `ORDER BY`. This lets you
get the "top N" by an actual ranking, like highest or lowest score:

```sql
-- Top 3 by highest score
SELECT TOP 3 *
FROM customers
ORDER BY score DESC;

-- Lowest 2 by score
SELECT TOP 2 *
FROM customers
ORDER BY score ASC;
```

Without `ORDER BY`, `TOP` doesn't guarantee any meaningful selection.
Always pair them when the "top" or "bottom" matters.

## Coding & Execution Order

These are two different things, and mixing them up is a common early
mistake.

**Coding order** (how you physically write the query):

SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY


**Execution order** (how SQL Server actually processes it):

FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY


Why this matters: SQL Server figures out which table to use first
(`FROM`), then filters rows (`WHERE`), then groups them (`GROUP BY`),
then filters the groups (`HAVING`), and only then decides which
columns to display (`SELECT`). Sorting (`ORDER BY`) happens last of
all.

This explains why you can use a column alias (defined in `SELECT`)
in `ORDER BY`, but not in `WHERE`. `WHERE` runs before `SELECT` even
exists yet, but `ORDER BY` runs after.

## Key Takeaways

- `WHERE` filters rows before grouping. `HAVING` filters groups after
  aggregation.
- `TOP` needs `ORDER BY` to be meaningful, otherwise "top" is
  arbitrary.
- `DISTINCT` removes duplicate rows or values from the result.
- Coding order and execution order are different. Understanding
  execution order explains a lot of "why doesn't this work" moments
  later on, such as with aliases or nested filtering.