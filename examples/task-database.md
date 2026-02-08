---
description: "Database operations - database locked, slow query, connection failed, table doesn't exist, query timing out, SQL queries, data import/export, and schema inspection (sqlite, duckdb, psql, mysql, redis)"
when_to_use: "Use when: database is locked or unresponsive, query is slow or timing out, connection failed or refused, table doesn't exist, need to inspect schema, import/export data, running SQL queries, managing SQLite databases, importing CSV to database, querying with DuckDB, using Redis. Triggers: database locked, slow query, connection failed, table doesn't exist, query timing out, database unresponsive, can't connect to database, query error, sqlite, sql query, database, psql, postgres, mysql, redis, duckdb, import csv, export data, schema, index, query plan, create table, database schema, migration, connection string, join tables, aggregate query, database size, vacuum, backup database."
when-to-use: "Use when: database is locked or unresponsive, query is slow or timing out, connection failed or refused, table doesn't exist, need to inspect schema, import/export data, running SQL queries, managing SQLite databases, importing CSV to database, querying with DuckDB, using Redis. Triggers: database locked, slow query, connection failed, table doesn't exist, query timing out, database unresponsive, can't connect to database, query error, sqlite, sql query, database, psql, postgres, mysql, redis, duckdb, import csv, export data, schema, index, query plan, create table, database schema, migration, connection string, join tables, aggregate query, database size, vacuum, backup database."
argument-hint: "[describe the database task]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Database Task Router

You have access to the **full conversation context**. Your job is to analyze the user's database need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Database tasks often depend on which database engine, what data, and what schema. Analyze the conversation to identify:

### 1. Goal
What database operation is needed?
- Query data (SELECT, aggregation, joins)
- Import data (CSV/JSON to table)
- Export data (table to CSV/JSON/Parquet)
- Schema inspection (tables, columns, types)
- Create/modify schema (CREATE, ALTER, INDEX)
- Performance analysis (EXPLAIN, indexes)
- Backup/restore
- Key-value operations (Redis)
- Ad-hoc CSV/Parquet analysis (DuckDB)

### 2. Database Type
What engine are we working with?
- SQLite (file path from conversation)
- DuckDB (analytical queries, Parquet)
- PostgreSQL (connection string/host)
- MySQL/MariaDB
- Redis
- No database yet (suggest based on need)

### 3. Data Context
- Table names mentioned
- Column names/types discussed
- CSV/JSON file to import
- Query results discussed earlier
- Schema shown in conversation

### 4. Error Context (if debugging)
- Query syntax error
- Performance issue (slow query)
- Import failure (type mismatch, encoding)
- Connection refused

---

## Goal --> Tool Guide

| Goal | Primary Tool | Argument Prefix |
|------|-------------|----------------|
| Query SQLite | sqlite3 | "query sqlite" |
| Query CSV/Parquet | duckdb | "query with duckdb" |
| Import CSV | sqlite-utils/sqlite3 | "import csv to" |
| Export data | sqlite3/duckdb | "export from" |
| Schema inspection | sqlite3/.schema | "inspect schema of" |
| PostgreSQL query | psql | "query postgres" |
| MySQL query | mysql | "query mysql" |
| Redis operations | redis-cli | "redis" |
| Serve data | datasette | "serve with datasette" |
| Query performance | EXPLAIN | "explain query on" |

---

## Argument Format

```
<goal> on <database/file> - engine: <type>, tables: <relevant>, data: <file if importing>, context: <prior findings>
```

### Examples

**Query SQLite:**
```
query sqlite /tmp/app.db - find users with orders > 5, tables: users, orders
```

**Import CSV to SQLite:**
```
import csv /tmp/sales.csv to sqlite /tmp/analytics.db - has headers, date column needs parsing
```

**DuckDB CSV analysis:**
```
query with duckdb /tmp/logs.csv - count by status_code, filter date > 2024-01-01
```

**PostgreSQL schema:**
```
inspect schema of postgres - host: localhost, database: myapp, show all tables with row counts
```

**Redis caching:**
```
redis set hash for user sessions - fields: token, expires_at, user_id, TTL 3600
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-database-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Database engine is unclear
- Table/column names not evident from context
- Connection details needed but not provided

**Use Read tool first if:**
- Need to inspect a .db file schema
- Need to check CSV headers before import
- Need to verify file path exists
