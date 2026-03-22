---
applyTo: "**/*.sql,**/etl*.py,**/bigquery/**"
---

# BigQuery & ETL Standards

## SQL Files

- **Never break backtick-quoted identifiers across lines.** Formatters can split `` `project.dataset.table` `` onto multiple lines, causing `Unclosed identifier literal` errors. Keep all backtick identifiers on a single line.
- **`JSON_EXTRACT` path must be a constant expression.** `JSON_EXTRACT(col, CONCAT('$.', var))` fails. Use individual checks: `IF(JSON_EXTRACT(col, '$.key') IS NOT NULL, 1, 0)`.

## ETL Data Coercion (pg8000 → BigQuery)

- **pg8000 returns `Decimal` for numeric columns.** Always coerce before `load_table_from_json`: `Decimal → float`, `datetime → isoformat()`, `dict/list → json.dumps(default=str)`.
- **Postgres JSONB → BQ STRING, not JSON type.** Serialize JSONB to Python string with `json.dumps()` and store as BQ `STRING`. BQ `JSON` type causes autodetect to infer `STRUCT`, which breaks MERGE.

## Staging Tables

- **Never use `autodetect=True` for staging tables.** Autodetect infers types from data (e.g., `"1.0"` → FLOAT64 instead of STRING), then MERGE fails on type mismatch. Always use the target table's schema: `bq_client.get_table(target).schema`.

## Schema Changes

- **BQ tables need delete + recreate after significant schema changes.** Terraform can't remove or rename columns in-place. `bq rm -f` the tables, then `terraform apply` to recreate.
- **BigQuery dataset `access` blocks replace all defaults.** Always include the three default `special_group` entries (`projectOwners`, `projectWriters`, `projectReaders`) alongside custom grants.
