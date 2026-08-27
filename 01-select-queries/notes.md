
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