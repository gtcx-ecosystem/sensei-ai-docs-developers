---
description: Cross-database type mappings
---

# Data Type Mappings

## Cross-Database Type Conversions

Reference for how Sensei maps data types between different database systems.

---

### Oracle to PostgreSQL

| Oracle                         | PostgreSQL       | Notes                     |
| ------------------------------ | ---------------- | ------------------------- |
| VARCHAR2(n)                    | VARCHAR(n)       | Direct mapping            |
| NVARCHAR2(n)                   | VARCHAR(n)       | UTF-8 encoding            |
| CHAR(n)                        | CHAR(n)          | Fixed length preserved    |
| NCHAR(n)                       | CHAR(n)          | UTF-8 encoding            |
| CLOB                           | TEXT             | Unlimited length          |
| NCLOB                          | TEXT             | UTF-8 encoding            |
| NUMBER                         | NUMERIC          | Full precision            |
| NUMBER(p)                      | NUMERIC(p)       | Precision preserved       |
| NUMBER(p,s)                    | NUMERIC(p,s)     | Precision and scale       |
| NUMBER(1-4)                    | SMALLINT         | Integer optimization      |
| NUMBER(5-9)                    | INTEGER          | Integer optimization      |
| NUMBER(10-18)                  | BIGINT           | Integer optimization      |
| BINARY_FLOAT                   | REAL             | 32-bit float              |
| BINARY_DOUBLE                  | DOUBLE PRECISION | 64-bit float              |
| DATE                           | TIMESTAMP        | Oracle DATE includes time |
| TIMESTAMP                      | TIMESTAMP        | Direct mapping            |
| TIMESTAMP WITH TIME ZONE       | TIMESTAMPTZ      | Direct mapping            |
| TIMESTAMP WITH LOCAL TIME ZONE | TIMESTAMPTZ      | Converted to UTC          |
| INTERVAL YEAR TO MONTH         | INTERVAL         | Direct mapping            |
| INTERVAL DAY TO SECOND         | INTERVAL         | Direct mapping            |
| BLOB                           | BYTEA            | Binary data               |
| RAW(n)                         | BYTEA            | Binary data               |
| LONG RAW                       | BYTEA            | Deprecated type           |
| LONG                           | TEXT             | Deprecated type           |
| ROWID                          | VARCHAR(18)      | As string                 |
| UROWID                         | VARCHAR(4000)    | As string                 |
| XMLTYPE                        | XML              | Native XML                |
| SDO_GEOMETRY                   | GEOMETRY         | PostGIS required          |
| BOOLEAN                        | BOOLEAN          | Oracle 23c+               |

---

### SQL Server to PostgreSQL

| SQL Server       | PostgreSQL       | Notes              |
| ---------------- | ---------------- | ------------------ |
| VARCHAR(n)       | VARCHAR(n)       | Direct mapping     |
| VARCHAR(MAX)     | TEXT             | Unlimited length   |
| NVARCHAR(n)      | VARCHAR(n)       | UTF-8 encoding     |
| NVARCHAR(MAX)    | TEXT             | Unlimited length   |
| CHAR(n)          | CHAR(n)          | Fixed length       |
| NCHAR(n)         | CHAR(n)          | UTF-8 encoding     |
| TEXT             | TEXT             | Deprecated type    |
| NTEXT            | TEXT             | Deprecated type    |
| INT              | INTEGER          | Direct mapping     |
| BIGINT           | BIGINT           | Direct mapping     |
| SMALLINT         | SMALLINT         | Direct mapping     |
| TINYINT          | SMALLINT         | No unsigned in PG  |
| BIT              | BOOLEAN          | 0/1 to false/true  |
| DECIMAL(p,s)     | NUMERIC(p,s)     | Direct mapping     |
| NUMERIC(p,s)     | NUMERIC(p,s)     | Direct mapping     |
| MONEY            | NUMERIC(19,4)    | Fixed precision    |
| SMALLMONEY       | NUMERIC(10,4)    | Fixed precision    |
| FLOAT            | DOUBLE PRECISION | 64-bit             |
| REAL             | REAL             | 32-bit             |
| DATE             | DATE             | Direct mapping     |
| TIME             | TIME             | Direct mapping     |
| DATETIME         | TIMESTAMP        | Direct mapping     |
| DATETIME2        | TIMESTAMP        | Higher precision   |
| SMALLDATETIME    | TIMESTAMP        | Lower precision    |
| DATETIMEOFFSET   | TIMESTAMPTZ      | With timezone      |
| BINARY(n)        | BYTEA            | Fixed binary       |
| VARBINARY(n)     | BYTEA            | Variable binary    |
| VARBINARY(MAX)   | BYTEA            | Large binary       |
| IMAGE            | BYTEA            | Deprecated type    |
| UNIQUEIDENTIFIER | UUID             | Direct mapping     |
| XML              | XML              | Direct mapping     |
| GEOGRAPHY        | GEOMETRY         | PostGIS, SRID 4326 |
| GEOMETRY         | GEOMETRY         | PostGIS required   |
| HIERARCHYID      | VARCHAR          | As string path     |
| SQL_VARIANT      | TEXT             | As JSON string     |

---

### MySQL to PostgreSQL

| MySQL        | PostgreSQL       | Notes               |
| ------------ | ---------------- | ------------------- |
| VARCHAR(n)   | VARCHAR(n)       | Direct mapping      |
| CHAR(n)      | CHAR(n)          | Fixed length        |
| TEXT         | TEXT             | Direct mapping      |
| TINYTEXT     | TEXT             | No size limit in PG |
| MEDIUMTEXT   | TEXT             | No size limit in PG |
| LONGTEXT     | TEXT             | No size limit in PG |
| TINYINT      | SMALLINT         | No unsigned         |
| SMALLINT     | SMALLINT         | Direct mapping      |
| MEDIUMINT    | INTEGER          | No medium in PG     |
| INT          | INTEGER          | Direct mapping      |
| BIGINT       | BIGINT           | Direct mapping      |
| DECIMAL(p,s) | NUMERIC(p,s)     | Direct mapping      |
| FLOAT        | REAL             | 32-bit              |
| DOUBLE       | DOUBLE PRECISION | 64-bit              |
| BIT(n)       | BIT(n)           | Direct mapping      |
| DATE         | DATE             | Direct mapping      |
| TIME         | TIME             | Direct mapping      |
| DATETIME     | TIMESTAMP        | Direct mapping      |
| TIMESTAMP    | TIMESTAMPTZ      | With timezone       |
| YEAR         | SMALLINT         | As number           |
| BINARY(n)    | BYTEA            | Binary data         |
| VARBINARY(n) | BYTEA            | Binary data         |
| BLOB         | BYTEA            | Binary data         |
| TINYBLOB     | BYTEA            | Binary data         |
| MEDIUMBLOB   | BYTEA            | Binary data         |
| LONGBLOB     | BYTEA            | Binary data         |
| ENUM         | VARCHAR + CHECK  | Or custom type      |
| SET          | VARCHAR[]        | Array type          |
| JSON         | JSONB            | Binary JSON         |
| GEOMETRY     | GEOMETRY         | PostGIS required    |
| BOOLEAN      | BOOLEAN          | Direct mapping      |

---

### PostgreSQL to Snowflake

| PostgreSQL       | Snowflake     | Notes                 |
| ---------------- | ------------- | --------------------- |
| VARCHAR(n)       | VARCHAR(n)    | Max 16MB in Snowflake |
| TEXT             | VARCHAR       | Max 16MB              |
| CHAR(n)          | CHAR(n)       | Direct mapping        |
| SMALLINT         | SMALLINT      | Direct mapping        |
| INTEGER          | INTEGER       | Direct mapping        |
| BIGINT           | BIGINT        | Direct mapping        |
| NUMERIC(p,s)     | NUMBER(p,s)   | Direct mapping        |
| REAL             | FLOAT         | 64-bit in Snowflake   |
| DOUBLE PRECISION | FLOAT         | 64-bit                |
| BOOLEAN          | BOOLEAN       | Direct mapping        |
| DATE             | DATE          | Direct mapping        |
| TIME             | TIME          | Direct mapping        |
| TIMESTAMP        | TIMESTAMP_NTZ | No timezone           |
| TIMESTAMPTZ      | TIMESTAMP_TZ  | With timezone         |
| INTERVAL         | VARCHAR       | As ISO 8601 string    |
| BYTEA            | BINARY        | Direct mapping        |
| UUID             | VARCHAR(36)   | As string             |
| JSON             | VARIANT       | Semi-structured       |
| JSONB            | VARIANT       | Semi-structured       |
| XML              | VARCHAR       | As string             |
| ARRAY            | ARRAY         | Direct mapping        |
| GEOMETRY         | GEOGRAPHY     | Snowflake native      |

---

### Teradata to Snowflake

| Teradata                 | Snowflake     | Notes             |
| ------------------------ | ------------- | ----------------- |
| VARCHAR(n)               | VARCHAR(n)    | Direct mapping    |
| CHAR(n)                  | CHAR(n)       | Direct mapping    |
| CLOB                     | VARCHAR       | Max 16MB          |
| BYTEINT                  | SMALLINT      | 8-bit to 16-bit   |
| SMALLINT                 | SMALLINT      | Direct mapping    |
| INTEGER                  | INTEGER       | Direct mapping    |
| BIGINT                   | BIGINT        | Direct mapping    |
| DECIMAL(p,s)             | NUMBER(p,s)   | Direct mapping    |
| FLOAT                    | FLOAT         | Direct mapping    |
| NUMBER                   | NUMBER(38,0)  | Default precision |
| DATE                     | DATE          | Direct mapping    |
| TIME                     | TIME          | Direct mapping    |
| TIMESTAMP                | TIMESTAMP_NTZ | No timezone       |
| TIMESTAMP WITH TIME ZONE | TIMESTAMP_TZ  | With timezone     |
| INTERVAL                 | VARCHAR       | As string         |
| BYTE(n)                  | BINARY(n)     | Direct mapping    |
| VARBYTE(n)               | BINARY        | Variable binary   |
| BLOB                     | BINARY        | Max 8MB           |
| JSON                     | VARIANT       | Semi-structured   |
| XML                      | VARCHAR       | As string         |
| GEOMETRY                 | GEOGRAPHY     | Snowflake native  |
| PERIOD                   | VARCHAR       | As string         |

---

### Notes on Type Mapping

**Precision handling:**

- MABA preserves precision by default
- Integer optimization available for NUMBER types
- Configure in migration settings

**NULL handling:**

- NULL semantics preserved across databases
- Empty string vs NULL handled per database convention

**Encoding:**

- All string types normalized to UTF-8
- Source encoding auto-detected
- Configurable fallback encoding

**Custom mappings:**
Override default mappings in configuration:

```yaml
type_mappings:
  - source_type: 'NUMBER(10)'
    target_type: 'INTEGER'
  - source_type: 'VARCHAR2(4000)'
    target_type: 'TEXT'
```

→ [SQL Translations](sql-translations.md)
→ [Configuration Reference](configuration-reference.md)
