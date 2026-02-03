# SQL Translations

## Dialect-Specific SQL Conversions

Reference for how Sensei translates SQL between database dialects.

---

### Oracle to PostgreSQL

#### Functions

| Oracle                    | PostgreSQL                                  | Notes            |
| ------------------------- | ------------------------------------------- | ---------------- |
| `NVL(a, b)`               | `COALESCE(a, b)`                            | Null handling    |
| `NVL2(a, b, c)`           | `CASE WHEN a IS NOT NULL THEN b ELSE c END` |                  |
| `DECODE(a, b, c, d)`      | `CASE a WHEN b THEN c ELSE d END`           |                  |
| `SYSDATE`                 | `CURRENT_TIMESTAMP`                         | Current datetime |
| `SYSTIMESTAMP`            | `CURRENT_TIMESTAMP`                         | With timezone    |
| `TO_DATE(s, fmt)`         | `TO_TIMESTAMP(s, fmt)`                      | Date parsing     |
| `TO_CHAR(d, fmt)`         | `TO_CHAR(d, fmt)`                           | Similar format   |
| `TO_NUMBER(s)`            | `CAST(s AS NUMERIC)`                        |                  |
| `SUBSTR(s, p, l)`         | `SUBSTRING(s FROM p FOR l)`                 | 1-based          |
| `INSTR(s, sub)`           | `POSITION(sub IN s)`                        |                  |
| `LENGTH(s)`               | `LENGTH(s)`                                 | Direct           |
| `TRIM(s)`                 | `TRIM(s)`                                   | Direct           |
| `UPPER(s)`                | `UPPER(s)`                                  | Direct           |
| `LOWER(s)`                | `LOWER(s)`                                  | Direct           |
| `LPAD(s, n, c)`           | `LPAD(s, n, c)`                             | Direct           |
| `RPAD(s, n, c)`           | `RPAD(s, n, c)`                             | Direct           |
| `REPLACE(s, a, b)`        | `REPLACE(s, a, b)`                          | Direct           |
| `REGEXP_LIKE(s, p)`       | `s ~ p`                                     | Regex match      |
| `REGEXP_REPLACE(s, p, r)` | `REGEXP_REPLACE(s, p, r)`                   | Direct           |
| `LISTAGG(col, ',')`       | `STRING_AGG(col, ',')`                      | Aggregation      |
| `ROWNUM`                  | `ROW_NUMBER() OVER ()`                      | Window function  |
| `CONNECT BY`              | `WITH RECURSIVE`                            | Hierarchical     |

#### Date Formats

| Oracle Format | PostgreSQL Format |
| ------------- | ----------------- |
| `YYYY`        | `YYYY`            |
| `MM`          | `MM`              |
| `DD`          | `DD`              |
| `HH24`        | `HH24`            |
| `HH12`        | `HH12`            |
| `MI`          | `MI`              |
| `SS`          | `SS`              |
| `AM/PM`       | `AM`              |
| `DAY`         | `Day`             |
| `MONTH`       | `Month`           |
| `DY`          | `Dy`              |
| `MON`         | `Mon`             |

#### Syntax

```sql
-- Oracle outer join
SELECT * FROM a, b WHERE a.id = b.id(+);
-- PostgreSQL
SELECT * FROM a LEFT JOIN b ON a.id = b.id;

-- Oracle ROWNUM
SELECT * FROM t WHERE ROWNUM <= 10;
-- PostgreSQL
SELECT * FROM t LIMIT 10;

-- Oracle sequence
SELECT seq.NEXTVAL FROM DUAL;
-- PostgreSQL
SELECT NEXTVAL('seq');

-- Oracle MINUS
SELECT * FROM a MINUS SELECT * FROM b;
-- PostgreSQL
SELECT * FROM a EXCEPT SELECT * FROM b;
```

---

### SQL Server to PostgreSQL

#### Functions

| SQL Server             | PostgreSQL                             | Notes  |
| ---------------------- | -------------------------------------- | ------ |
| `ISNULL(a, b)`         | `COALESCE(a, b)`                       |        |
| `GETDATE()`            | `CURRENT_TIMESTAMP`                    |        |
| `GETUTCDATE()`         | `CURRENT_TIMESTAMP AT TIME ZONE 'UTC'` |        |
| `DATEADD(unit, n, d)`  | `d + INTERVAL 'n unit'`                |        |
| `DATEDIFF(unit, a, b)` | `EXTRACT(unit FROM b - a)`             |        |
| `CONVERT(type, val)`   | `CAST(val AS type)`                    |        |
| `CAST(val AS type)`    | `CAST(val AS type)`                    | Direct |
| `LEN(s)`               | `LENGTH(s)`                            |        |
| `CHARINDEX(sub, s)`    | `POSITION(sub IN s)`                   |        |
| `SUBSTRING(s, p, l)`   | `SUBSTRING(s FROM p FOR l)`            |        |
| `STUFF(s, p, l, r)`    | `OVERLAY(s PLACING r FROM p FOR l)`    |        |
| `IIF(cond, t, f)`      | `CASE WHEN cond THEN t ELSE f END`     |        |
| `STRING_AGG(col, ',')` | `STRING_AGG(col, ',')`                 | Direct |
| `NEWID()`              | `GEN_RANDOM_UUID()`                    |        |
| `@@IDENTITY`           | `LASTVAL()`                            |        |
| `SCOPE_IDENTITY()`     | `LASTVAL()`                            |        |

#### Syntax

```sql
-- SQL Server TOP
SELECT TOP 10 * FROM t;
-- PostgreSQL
SELECT * FROM t LIMIT 10;

-- SQL Server identity
CREATE TABLE t (id INT IDENTITY(1,1));
-- PostgreSQL
CREATE TABLE t (id SERIAL);

-- SQL Server temp table
SELECT * INTO #temp FROM t;
-- PostgreSQL
CREATE TEMP TABLE temp AS SELECT * FROM t;

-- SQL Server CTE with UPDATE
WITH cte AS (SELECT * FROM t) UPDATE cte SET col = 1;
-- PostgreSQL
UPDATE t SET col = 1 FROM (SELECT * FROM t) cte WHERE t.id = cte.id;
```

---

### Teradata to Snowflake

#### Functions

| Teradata               | Snowflake                | Notes          |
| ---------------------- | ------------------------ | -------------- |
| `ZEROIFNULL(a)`        | `NVL(a, 0)`              |                |
| `NULLIFZERO(a)`        | `NULLIF(a, 0)`           |                |
| `CURRENT_DATE`         | `CURRENT_DATE`           | Direct         |
| `CURRENT_TIMESTAMP`    | `CURRENT_TIMESTAMP`      | Direct         |
| `ADD_MONTHS(d, n)`     | `DATEADD(month, n, d)`   |                |
| `MONTHS_BETWEEN(a, b)` | `DATEDIFF(month, b, a)`  |                |
| `TRUNC(d, 'MM')`       | `DATE_TRUNC('month', d)` |                |
| `EXTRACT(YEAR FROM d)` | `EXTRACT(YEAR FROM d)`   | Direct         |
| `TO_DATE(s, fmt)`      | `TO_DATE(s, fmt)`        | Format differs |
| `TO_CHAR(d, fmt)`      | `TO_CHAR(d, fmt)`        | Format differs |
| `CAST(val AS type)`    | `CAST(val AS type)`      | Direct         |
| `COALESCE(a, b)`       | `COALESCE(a, b)`         | Direct         |
| `CASE WHEN`            | `CASE WHEN`              | Direct         |
| `REGEXP_SUBSTR(s, p)`  | `REGEXP_SUBSTR(s, p)`    | Direct         |
| `INDEX(s, sub)`        | `CHARINDEX(sub, s)`      |                |
| `TRIM(s)`              | `TRIM(s)`                | Direct         |

#### Syntax

```sql
-- Teradata QUALIFY
SELECT * FROM t QUALIFY ROW_NUMBER() OVER (ORDER BY col) = 1;
-- Snowflake (also supports QUALIFY)
SELECT * FROM t QUALIFY ROW_NUMBER() OVER (ORDER BY col) = 1;

-- Teradata sample
SELECT * FROM t SAMPLE 100;
-- Snowflake
SELECT * FROM t SAMPLE (100 ROWS);

-- Teradata SET table
CREATE SET TABLE t (col INT);
-- Snowflake (no SET, use DISTINCT)
CREATE TABLE t (col INT);

-- Teradata MERGE
MERGE INTO target USING source ON condition
WHEN MATCHED THEN UPDATE
WHEN NOT MATCHED THEN INSERT;
-- Snowflake
MERGE INTO target USING source ON condition
WHEN MATCHED THEN UPDATE
WHEN NOT MATCHED THEN INSERT;
```

---

### MySQL to PostgreSQL

#### Functions

| MySQL                          | PostgreSQL                            | Notes          |
| ------------------------------ | ------------------------------------- | -------------- |
| `IFNULL(a, b)`                 | `COALESCE(a, b)`                      |                |
| `IF(cond, t, f)`               | `CASE WHEN cond THEN t ELSE f END`    |                |
| `NOW()`                        | `CURRENT_TIMESTAMP`                   |                |
| `CURDATE()`                    | `CURRENT_DATE`                        |                |
| `DATE_ADD(d, INTERVAL n unit)` | `d + INTERVAL 'n unit'`               |                |
| `DATE_SUB(d, INTERVAL n unit)` | `d - INTERVAL 'n unit'`               |                |
| `DATE_FORMAT(d, fmt)`          | `TO_CHAR(d, fmt)`                     | Format differs |
| `STR_TO_DATE(s, fmt)`          | `TO_TIMESTAMP(s, fmt)`                |                |
| `CONCAT(a, b, c)`              | `CONCAT(a, b, c)`                     | Direct         |
| `GROUP_CONCAT(col)`            | `STRING_AGG(col, ',')`                |                |
| `FIND_IN_SET(s, list)`         | `s = ANY(STRING_TO_ARRAY(list, ','))` |                |
| `LIMIT n, m`                   | `LIMIT m OFFSET n`                    | Order differs  |
| `AUTO_INCREMENT`               | `SERIAL`                              |                |
| `UNSIGNED`                     | (remove)                              | No unsigned    |

#### Syntax

```sql
-- MySQL backticks
SELECT `column` FROM `table`;
-- PostgreSQL double quotes
SELECT "column" FROM "table";

-- MySQL LIMIT with offset
SELECT * FROM t LIMIT 10, 20;
-- PostgreSQL
SELECT * FROM t LIMIT 20 OFFSET 10;

-- MySQL INSERT IGNORE
INSERT IGNORE INTO t VALUES (1);
-- PostgreSQL
INSERT INTO t VALUES (1) ON CONFLICT DO NOTHING;

-- MySQL REPLACE
REPLACE INTO t VALUES (1, 'a');
-- PostgreSQL
INSERT INTO t VALUES (1, 'a') ON CONFLICT (id) DO UPDATE SET col = 'a';
```

---

### Custom Translations

Override or add translations in configuration:

```yaml
sql_translations:
  functions:
    - source: 'CUSTOM_FUNC(a)'
      target: 'MY_FUNC(a)'
  syntax:
    - pattern: 'PROPRIETARY_CLAUSE'
      replacement: 'STANDARD_CLAUSE'
```

→ [Data Type Mappings](data-types.md)
→ [Configuration Reference](configuration-reference.md)
