---
description: Source schema ingestion pipeline
---

# Ingestion Pipeline

## 50+ Formats, One Normalized Output

MABA's ingestion layer reads data from virtually any source — relational databases, document stores, spreadsheets, file formats, unstructured documents, and legacy systems — and normalizes everything into a common intermediate representation for analysis and transformation.

---

### Format Detection

File formats aren't always what they claim to be. A file with a `.csv` extension might actually be tab-delimited. A `.json` file might contain NDJSON (newline-delimited). An `.xlsx` file might actually be an older `.xls` binary format renamed.

MABA detects formats using three methods in order:

1. **Magic byte inspection:** Binary file signatures that identify format regardless of extension
2. **Extension analysis:** File extension as a hint (not authoritative)
3. **Statistical content analysis:** Character distribution, delimiter frequency, and structural patterns

This layered approach correctly identifies formats even when extensions are missing, wrong, or misleading.

---

### Supported Formats

#### Relational Databases

| Database         | Connection Method               | Metadata Extraction                                                    |
| ---------------- | ------------------------------- | ---------------------------------------------------------------------- |
| PostgreSQL       | Native driver (psycopg2)        | Full catalog: tables, columns, constraints, functions, triggers, views |
| MySQL / MariaDB  | Native driver (mysql-connector) | Full catalog                                                           |
| Oracle           | cx_Oracle / oracledb            | Full catalog including PL/SQL stored procedures                        |
| SQL Server       | pyodbc / pymssql                | Full catalog including T-SQL procedures                                |
| SQLite           | Built-in Python sqlite3         | Schema tables, no stored procedures                                    |
| IBM DB2          | ibm_db                          | Full catalog including RPG/COBOL program references                    |
| Sybase / SAP ASE | pyodbc                          | Full catalog                                                           |

#### Document Formats

| Format      | Parser                                                                  | Notes                                           |
| ----------- | ----------------------------------------------------------------------- | ----------------------------------------------- |
| CSV         | Custom (handles edge cases: quoted delimiters, mixed line endings, BOM) | Auto-detects delimiter, encoding, quoting style |
| TSV         | Same as CSV with tab delimiter                                          |                                                 |
| Fixed-width | Custom positional parser                                                | Auto-detects column boundaries from sample rows |
| JSON        | Built-in + custom nested flattener                                      | Handles nested objects, arrays, mixed types     |
| NDJSON      | Streaming parser                                                        | Line-by-line for large files                    |
| XML         | lxml with XPath                                                         | Auto-detects record boundaries                  |
| YAML        | PyYAML                                                                  | For configuration-as-data scenarios             |
| Parquet     | Apache Arrow                                                            | Columnar format, schema embedded                |
| Avro        | fastavro                                                                | Schema embedded, fast deserialization           |
| ORC         | pyarrow                                                                 | Hadoop ecosystem columnar format                |

#### Document Stores

| Store         | Connection       | Notes                                                                |
| ------------- | ---------------- | -------------------------------------------------------------------- |
| MongoDB       | pymongo          | Handles nested documents, array fields, schema inference from sample |
| CouchDB       | couchdb-python   | JSON document extraction                                             |
| Elasticsearch | elasticsearch-py | Index mapping extraction, scroll API for large indices               |

#### Spreadsheets

| Format          | Parser            | Notes                                        |
| --------------- | ----------------- | -------------------------------------------- |
| Excel XLSX      | openpyxl          | Multi-sheet, formulas captured, named ranges |
| Excel XLS       | xlrd              | Legacy binary format                         |
| Google Sheets   | Google Sheets API | Real-time connection                         |
| LibreOffice ODS | odfpy             | Open document format                         |

#### Unstructured Documents

| Format          | Parser                                 | Notes                                                           |
| --------------- | -------------------------------------- | --------------------------------------------------------------- |
| PDF             | pdfplumber + Tesseract OCR             | Table extraction from formatted PDFs; OCR for scanned documents |
| Word (DOCX/DOC) | python-docx                            | Table extraction from Word documents                            |
| Scanned images  | Tesseract OCR + custom table detection | For photographed or scanned paper forms                         |

#### Legacy Formats

| Format                       | Parser                   | Notes                                                  |
| ---------------------------- | ------------------------ | ------------------------------------------------------ |
| COBOL copybooks              | Custom COBOL parser      | COMP-3 packed decimal, EBCDIC, record-level processing |
| EBCDIC flat files            | Custom codec             | Code page detection and conversion                     |
| Fixed-format mainframe dumps | Custom positional parser | Record boundary detection from RDW headers             |
| Non-standard delimited files | Adaptive parser          | Handles pipes, tildes, multi-character delimiters      |

---

### The Intermediate Representation

Every parser produces a normalized IR that captures:

```yaml
IntermediateRepresentation:
  source:
    format: PostgreSQL
    connection: (metadata only, no credentials stored)
    tables: 247

  table: customer_data_v2_final
  columns:
    - name: cust_ref
      declared_type: VARCHAR(15)
      inferred_type: formatted_identifier
      null_rate: 0.0%
      distinct_ratio: 1.0
      sample_values: ["CR-2024-00147", "CR-2024-00148", ...]
      pattern_match: "XX-NNNN-NNNNN" (100%)
      encoding: UTF-8

    - name: status
      declared_type: NUMBER(2)
      inferred_type: enumeration
      null_rate: 0.1%
      distinct_values: [1, 2, 3, 10, 99]
      distribution: {1: 45%, 2: 30%, 3: 15%, 10: 8%, 99: 2%}
      encoding: N/A (numeric)
```

This IR is the input to the [Schema Analysis Engine](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md). By normalizing all source formats to a common representation, the analysis and transformation pipeline doesn't need to know whether the source was Oracle, MongoDB, or a scanned PDF.

---

### Pluggable Architecture

New format parsers can be added without modifying the core pipeline. Each parser implements a standard interface:

1. `detect(input) → confidence_score` — Can this parser handle this input?
2. `extract_metadata(input) → schema_definition` — What's the structure?
3. `read_sample(input, n) → rows` — Read n sample rows
4. `stream(input) → iterator[row]` — Stream all rows for processing

Community-contributed parsers can extend MABA's format support for niche formats specific to industries or regions.

→ [Schema Analysis](../../../product/platform/maba/ingestion-pipeline/schema-analysis.md)
→ [How MABA Works](how-it-works.md)
