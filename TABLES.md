# analytics.duckdb — Table Catalog

DuckDB store for SAP ECC data-dictionary metadata. Path: `app/backend/app/data/analytics.duckdb` (tracked via Git LFS).

## Summary

| Table | Rows | Origin | Purpose |
|-------|-----:|--------|---------|
| `dd02l` | 174,450 | User upload (CSV → DuckDB) | SAP table headers |
| `dd03l` | 1,848,894 | User upload (CSV → DuckDB) | SAP table field definitions |
| `dd04t` | 431,960 | User upload (CSV → DuckDB) | SAP data-element texts (multilingual) |
| `dd04t_friendly` | 263,806 | Pre-loaded, derived | One row per ROLLNAME with friendly snake_case names |
| `_dd_schema_cache` | 83 | Auto-rebuilt on upload | Pre-joined schema descriptions used by `/schemas` endpoint |

All columns are `VARCHAR` and nullable — schemas mirror the raw SAP export.

## Uploadable tables (UC-DD-003)

Loaded via `POST /api/v1/data-dictionary/tables/{name}/upload`. CSV is converted and persisted as a DuckDB table via `CREATE OR REPLACE TABLE` (see [ADR-001](../../../../docs/architecture/adrs/ADR-001-duckdb-parquet-storage-pattern.md)). Each upload **fully replaces** the existing table.

### `dd02l` — Table headers

Top-level SAP table catalog (one row per SAP table).

Key columns: `TABNAME`, `TABCLASS`, `SQLTAB`, `APPLCLASS`, `MASTERLANG`, `AS4LOCAL`, `AS4VERS`, `AS4USER`, `AS4DATE`, `AS4TIME`.
Full column list (44): TABNAME, AS4LOCAL, AS4VERS, TABCLASS, SQLTAB, DATMIN, DATMAX, DATAVG, CLIDEP, BUFFERED, COMPRFLAG, LANGDEP, ACTFLAG, APPLCLASS, AUTHCLASS, AS4USER, AS4DATE, AS4TIME, MASTERLANG, MAINFLAG, CONTFLAG, RESERVETAB, GLOBALFLAG, PROZPUFF, VIEWCLASS, VIEWGRANT, MULTIPLEX, SHLPEXI, PROXYTYPE, EXCLASS, WRONGCL, ALWAYSTRP, ALLDATAINCL, WITH_PARAMETERS, EXVIEW_INCLUDED, KEYMAX_FEATURE, KEYLEN_FEATURE, TABLEN_FEATURE, NONTRP_INCLUDED, VIEWREF, VIEWREF_ERR, TBFUNC_INCLUDED, IS_GTT, SESSION_VAR_EX.

### `dd03l` — Table field definitions

One row per (TABNAME, FIELDNAME). Joins to `dd02l` on `TABNAME` and to `dd04t` on `ROLLNAME`.

Key columns: `TABNAME`, `FIELDNAME`, `POSITION`, `KEYFLAG`, `ROLLNAME`, `CHECKTABLE`, `DATATYPE`, `LENG`, `DECIMALS`, `DOMNAME`, `NOTNULL`, `INTTYPE`, `INTLEN`, `REFTABLE`, `REFFIELD`.
Full column list (30): TABNAME, FIELDNAME, AS4LOCAL, AS4VERS, POSITION, KEYFLAG, MANDATORY, ROLLNAME, CHECKTABLE, ADMINFIELD, INTTYPE, INTLEN, REFTABLE, PRECFIELD, REFFIELD, CONROUT, NOTNULL, DATATYPE, LENG, DECIMALS, DOMNAME, SHLPORIGIN, TABLETYPE, DEPTH, COMPTYPE, REFTYPE, LANGUFLAG, DBPOSITION, ANONYMOUS, OUTPUTSTYLE.

### `dd04t` — Data-element texts

Multilingual descriptions for data elements (ROLLNAME). Two languages present: **E** (English, ~248K rows) and **S** (Spanish, ~184K rows).

Columns (9): ROLLNAME, DDLANGUAGE, AS4LOCAL, AS4VERS, DDTEXT, REPTEXT, SCRTEXT_S, SCRTEXT_M, SCRTEXT_L.

## Derived / system tables

### `dd04t_friendly` — Pre-loaded, read-only

Processed view of `dd04t` plus rows for `dd03l` fields with NULL `ROLLNAME`. One row per ROLLNAME with language picked deterministically: **Spanish preferred → English fallback → empty for manual entries**.

Distribution: S=184,253 · E=64,141 · ''=15,412.

Columns (7): ROLLNAME, DDLANGUAGE, DDTEXT, SCRTEXT_S, SCRTEXT_M, SCRTEXT_L, **FRIENDLY_NAME** (DataBricks-style snake_case label, e.g., `indicador_de_contabilizacion`).

### `_dd_schema_cache` — Auto-managed

Precomputed flattened schema used by `GET /api/v1/data-dictionary/schemas`. Rebuilt by `rebuild_schema_cache()` after every upload (see [TUC-006](../../../../docs/technical-use-cases/TUC-006-data-dictionary-api.md)).

Columns (4): `table_name`, `column_name`, `data_type`, `description` — where `description` is the joined `dd04t.DDTEXT` for the field's `ROLLNAME`. Currently holds 83 rows covering `dd02l`, `dd03l`, `dd04t`.

## Backup tables

If a recovery copy was created prior to a destructive upload, it appears as `bak_dd02l`, `bak_dd03l`, or `bak_dd04t`. Not present by default.

## Caveats

- Automated E2E tests must **never** upload to `dd02l`, `dd03l`, or `dd04t` — uploads fully replace the table and 174K+ rows of real SAP metadata would be lost. UC-DD-003 is tested manually.
- All columns are `VARCHAR`; numeric/date semantics live in `dd03l.DATATYPE`, `LENG`, `DECIMALS`.
