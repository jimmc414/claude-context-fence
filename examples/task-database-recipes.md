---
description: "Database operations recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-database with database type and context included"
when-to-use: "Receives prepared arguments from task-database with database type and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Database Operations Recipes

**Tools:** sqlite3, sqlite-utils, duckdb, psql, mysql, redis-cli, datasette

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Open SQLite | `sqlite3 database.db` |
| Run query | `sqlite3 db.db "SELECT * FROM t LIMIT 10"` |
| Import CSV | `sqlite-utils insert db.db tablename data.csv --csv` |
| Export CSV | `sqlite3 -header -csv db.db "SELECT * FROM t" > out.csv` |
| Query CSV directly | `duckdb -c "SELECT * FROM 'file.csv' LIMIT 10"` |
| Query Parquet | `duckdb -c "SELECT * FROM 'file.parquet'"` |
| Schema inspect | `sqlite3 db.db ".schema"` |
| Redis get/set | `redis-cli SET key "value"` / `redis-cli GET key` |
| Postgres connect | `psql -h host -U user -d dbname` |
| Serve SQLite | `datasette serve db.db` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Local SQL queries | sqlite3 | Zero-config, file-based |
| Analytical queries on files | duckdb | Fast columnar engine, reads CSV/Parquet |
| SQLite management | sqlite-utils | Python-powered CLI for SQLite |
| PostgreSQL queries | psql | Standard Postgres client |
| MySQL queries | mysql | Standard MySQL client |
| Redis operations | redis-cli | Key-value store client |
| Data exploration UI | datasette | Web UI for SQLite databases |
| Import CSV to SQL | sqlite-utils insert | One-command CSV to SQLite |

---

## SQLite

### Open and Create

```bash
sqlite3 database.db                        # Open or create database
sqlite3 :memory:                           # In-memory database
sqlite3 -header -column db.db              # Open with formatted output
sqlite3 -json db.db "SELECT * FROM t"      # JSON output mode
```

### Run Queries

```bash
# Inline query
sqlite3 db.db "SELECT * FROM users WHERE age > 30"

# Multiple statements with -cmd
sqlite3 -cmd ".headers on" -cmd ".mode column" db.db "SELECT * FROM users"

# Run from file
sqlite3 db.db < script.sql
sqlite3 db.db ".read script.sql"

# Multi-line query
sqlite3 db.db <<'SQL'
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.name
HAVING order_count > 5
ORDER BY order_count DESC;
SQL
```

### Import CSV

```bash
# Classic method (dot commands)
sqlite3 db.db <<'SQL'
.mode csv
.import data.csv tablename
SQL

# With separator handling
sqlite3 db.db <<'SQL'
.mode csv
.separator "\t"
.import data.tsv tablename
SQL

# Import skipping header when table exists (SQLite 3.32+)
sqlite3 db.db <<'SQL'
.mode csv
.import --skip 1 data.csv existing_table
SQL

# Create table from CSV structure manually
sqlite3 db.db <<'SQL'
CREATE TABLE IF NOT EXISTS sales (
    date TEXT, product TEXT, amount REAL, quantity INTEGER
);
.mode csv
.import --skip 1 sales.csv sales
SQL
```

### Export Data

```bash
# Export to CSV
sqlite3 -header -csv db.db "SELECT * FROM users" > users.csv

# Export to JSON
sqlite3 -json db.db "SELECT * FROM users" > users.json

# Export specific columns
sqlite3 -header -csv db.db "SELECT name, email FROM users WHERE active = 1" > active.csv

# Export as INSERT statements
sqlite3 db.db <<'SQL'
.mode insert users
SELECT * FROM users;
SQL

# Export with dot commands
sqlite3 db.db <<'SQL'
.headers on
.mode csv
.output results.csv
SELECT * FROM orders WHERE date > '2024-01-01';
.output stdout
SQL
```

### Schema Inspection

```bash
# List all tables
sqlite3 db.db ".tables"

# Show full schema
sqlite3 db.db ".schema"

# Show schema for specific table
sqlite3 db.db ".schema users"

# Table info (columns, types)
sqlite3 db.db "PRAGMA table_info(users)"

# List indexes on a table
sqlite3 db.db "PRAGMA index_list(users)"

# Show index columns
sqlite3 db.db "PRAGMA index_info(idx_users_email)"

# Foreign keys
sqlite3 db.db "PRAGMA foreign_key_list(orders)"

# Table row counts (all tables)
sqlite3 db.db <<'SQL'
SELECT name, (SELECT COUNT(*) FROM pragma_table_info(name)) as cols
FROM sqlite_master WHERE type='table' ORDER BY name;
SQL

# Database size info
sqlite3 db.db "PRAGMA page_count; PRAGMA page_size;"
```

### Pragmas (Performance and Config)

```bash
# WAL mode (better concurrent reads)
sqlite3 db.db "PRAGMA journal_mode=WAL"

# Performance pragmas
sqlite3 db.db <<'SQL'
PRAGMA synchronous=NORMAL;
PRAGMA cache_size=-64000;
PRAGMA mmap_size=268435456;
PRAGMA temp_store=MEMORY;
SQL

# Enable foreign keys (off by default!)
sqlite3 db.db "PRAGMA foreign_keys=ON"

# Integrity check
sqlite3 db.db "PRAGMA integrity_check"
sqlite3 db.db "PRAGMA quick_check"
```

### JSON Functions

```bash
# Extract from JSON column
sqlite3 db.db "SELECT json_extract(data, '$.name') FROM records"
sqlite3 db.db "SELECT data->>'$.name' FROM records"

# Query JSON arrays
sqlite3 db.db <<'SQL'
SELECT value FROM records, json_each(records.data, '$.tags')
WHERE json_extract(records.data, '$.active') = 1;
SQL

# Build JSON from columns
sqlite3 db.db "SELECT json_group_array(json_object('id', id, 'name', name)) FROM users"

# JSON validation and type
sqlite3 db.db "SELECT json_valid(data) FROM records"
sqlite3 db.db "SELECT json_type(data, '$.count') FROM records"

# JSON array length
sqlite3 db.db "SELECT json_array_length(data, '$.items') FROM records"

# Modify JSON
sqlite3 db.db "SELECT json_set(data, '$.updated', 1) FROM records"
sqlite3 db.db "SELECT json_insert(data, '$.new_field', 'value') FROM records"
sqlite3 db.db "SELECT json_remove(data, '$.obsolete') FROM records"
sqlite3 db.db "SELECT json_replace(data, '$.status', 'done') FROM records"
sqlite3 db.db "SELECT json_patch(data, '{\"version\": 2}') FROM records"
```

### FTS5 (Full-Text Search)

```bash
# Create FTS table
sqlite3 db.db <<'SQL'
CREATE VIRTUAL TABLE docs_fts USING fts5(title, body, content='docs', content_rowid='id');
INSERT INTO docs_fts(docs_fts) VALUES('rebuild');
SQL

# Search
sqlite3 db.db "SELECT * FROM docs_fts WHERE docs_fts MATCH 'database AND query'"
sqlite3 db.db "SELECT * FROM docs_fts WHERE docs_fts MATCH 'prefix*'"
sqlite3 db.db "SELECT * FROM docs_fts WHERE docs_fts MATCH 'NEAR(sql query, 5)'"

# Ranked results
sqlite3 db.db "SELECT *, rank FROM docs_fts WHERE docs_fts MATCH 'search term' ORDER BY rank"
sqlite3 db.db "SELECT *, bm25(docs_fts) as score FROM docs_fts WHERE docs_fts MATCH 'term' ORDER BY score"

# Highlight and snippet
sqlite3 db.db "SELECT highlight(docs_fts, 1, '<b>', '</b>') FROM docs_fts WHERE docs_fts MATCH 'term'"
sqlite3 db.db "SELECT snippet(docs_fts, 1, '<b>', '</b>', '...', 32) FROM docs_fts WHERE docs_fts MATCH 'term'"
```

### Window Functions

```bash
sqlite3 db.db <<'SQL'
-- Row number, rank, dense rank
SELECT name, score,
    ROW_NUMBER() OVER (ORDER BY score DESC) as row_num,
    RANK() OVER (ORDER BY score DESC) as rnk,
    DENSE_RANK() OVER (ORDER BY score DESC) as dense_rnk
FROM students;

-- Lag and Lead (previous/next row)
SELECT date, amount,
    LAG(amount, 1) OVER (ORDER BY date) as prev_amount,
    LEAD(amount, 1) OVER (ORDER BY date) as next_amount
FROM sales;

-- Running sum
SELECT date, amount,
    SUM(amount) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING) as running_total
FROM transactions;

-- First value per partition
SELECT department, name, salary,
    FIRST_VALUE(name) OVER (PARTITION BY department ORDER BY salary DESC) as top_earner
FROM employees;

-- NTILE (divide into N groups)
SELECT name, score,
    NTILE(4) OVER (ORDER BY score DESC) as quartile
FROM students;
SQL
```

### CTEs (Common Table Expressions)

```bash
sqlite3 db.db <<'SQL'
-- Simple CTE
WITH active_users AS (
    SELECT * FROM users WHERE active = 1
)
SELECT department, COUNT(*) FROM active_users GROUP BY department;

-- Recursive CTE (hierarchy)
WITH RECURSIVE org_chart(id, name, manager_id, level) AS (
    SELECT id, name, manager_id, 0 FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, name;

-- Recursive CTE (sequence generation)
WITH RECURSIVE dates(d) AS (
    VALUES('2024-01-01')
    UNION ALL
    SELECT date(d, '+1 day') FROM dates WHERE d < '2024-12-31'
)
SELECT d FROM dates;
SQL
```

### Date and Time

```bash
sqlite3 db.db "SELECT date('now')"
sqlite3 db.db "SELECT datetime('now', 'localtime')"
sqlite3 db.db "SELECT strftime('%Y-%m', date_col) as month, COUNT(*) FROM events GROUP BY month"
sqlite3 db.db "SELECT date('now', '-7 days')"
sqlite3 db.db "SELECT date('now', 'start of month')"
sqlite3 db.db "SELECT date('now', 'weekday 0')"
sqlite3 db.db "SELECT unixepoch('now')"
sqlite3 db.db "SELECT datetime(1700000000, 'unixepoch')"
sqlite3 db.db "SELECT julianday('now') - julianday(created_at) as days_old FROM records"
```

### Query Plans and Indexes

```bash
# Explain query plan
sqlite3 db.db "EXPLAIN QUERY PLAN SELECT * FROM users WHERE email = 'test@test.com'"

# Create indexes
sqlite3 db.db "CREATE INDEX idx_users_email ON users(email)"
sqlite3 db.db "CREATE UNIQUE INDEX idx_users_username ON users(username)"

# Partial index
sqlite3 db.db "CREATE INDEX idx_active_users ON users(name) WHERE active = 1"

# Covering index (includes extra columns)
sqlite3 db.db "CREATE INDEX idx_users_email_name ON users(email, name)"

# Expression index
sqlite3 db.db "CREATE INDEX idx_users_lower_email ON users(LOWER(email))"
```

### Backup and Restore

```bash
# SQL dump
sqlite3 db.db .dump > backup.sql
sqlite3 db.db .dump | gzip > backup.sql.gz

# Binary backup (safe with WAL)
sqlite3 db.db ".backup backup.db"

# Restore from dump
sqlite3 newdb.db < backup.sql
gunzip -c backup.sql.gz | sqlite3 newdb.db

# Dump specific table
sqlite3 db.db ".dump users" > users.sql
```

---

## sqlite-utils

### Create and Insert

```bash
# Create table from CSV
sqlite-utils insert db.db tablename data.csv --csv

# Insert JSON
sqlite-utils insert db.db tablename data.json

# Insert newline-delimited JSON
sqlite-utils insert db.db tablename data.ndjson --nl

# Insert from stdin (pipe)
cat data.json | sqlite-utils insert db.db tablename -

# Upsert (insert or update on conflict)
sqlite-utils upsert db.db tablename data.json --pk id
```

### Query

```bash
sqlite-utils query db.db "SELECT * FROM users WHERE active = 1"
sqlite-utils query db.db "SELECT * FROM users" --csv > output.csv
sqlite-utils query db.db "SELECT * FROM users" --table   # pretty table
```

### Schema and Info

```bash
sqlite-utils tables db.db
sqlite-utils schema db.db
sqlite-utils indexes db.db
sqlite-utils tables db.db --counts   # row counts per table
```

### Transform and Modify

```bash
# Add column
sqlite-utils add-column db.db users age integer

# Rename column
sqlite-utils transform db.db users --rename old_name new_name

# Change column type
sqlite-utils transform db.db users --type age integer

# Enable full-text search
sqlite-utils enable-fts db.db articles title body
sqlite-utils search db.db articles "search term"
```

### Memory Mode (no file needed)

```bash
# Query CSV directly in memory
sqlite-utils memory data.csv "SELECT * FROM data WHERE amount > 100"
sqlite-utils memory data.csv "SELECT category, SUM(amount) FROM data GROUP BY category"

# Multiple files
sqlite-utils memory sales.csv products.csv "SELECT s.*, p.name FROM sales s JOIN products p ON s.product_id = p.id"

# Pipe from stdin
cat data.csv | sqlite-utils memory - "SELECT COUNT(*) FROM stdin"
```

### Convert (Apply Python Expression)

```bash
sqlite-utils convert db.db users email 'value.lower()'
sqlite-utils convert db.db users name 'value.strip().title()'
```

### Extract (Normalize)

```bash
# Extract repeated values into a lookup table
sqlite-utils extract db.db orders country --table countries
```

---

## DuckDB

### Query Files Directly

```bash
# CSV with auto-detection
duckdb -c "SELECT * FROM 'data.csv' LIMIT 10"
duckdb -c "SELECT * FROM read_csv_auto('data.csv') LIMIT 10"

# CSV with explicit options
duckdb -c "SELECT * FROM read_csv('data.csv', header=true, sep='|', columns={'id': 'INT', 'name': 'VARCHAR'})"

# Parquet
duckdb -c "SELECT * FROM 'data.parquet' LIMIT 10"
duckdb -c "SELECT * FROM read_parquet('data.parquet')"

# JSON
duckdb -c "SELECT * FROM read_json_auto('data.json')"
duckdb -c "SELECT * FROM 'data.json'"

# Multiple files with glob
duckdb -c "SELECT * FROM 'data/*.csv'"
duckdb -c "SELECT * FROM 'logs/2024-*.parquet'"
```

### Analytical Queries

```bash
# Group by with aggregates
duckdb -c "SELECT category, COUNT(*), AVG(price), SUM(quantity) FROM 'sales.csv' GROUP BY category"

# HAVING clause
duckdb -c "SELECT dept, COUNT(*) as cnt FROM 'emp.csv' GROUP BY dept HAVING cnt > 10"

# QUALIFY (filter window functions)
duckdb -c "SELECT * FROM 'sales.csv' QUALIFY ROW_NUMBER() OVER (PARTITION BY customer ORDER BY date DESC) = 1"

# Window functions
duckdb -c "SELECT *, SUM(amount) OVER (PARTITION BY customer ORDER BY date) as running_total FROM 'transactions.csv'"

# PIVOT
duckdb -c "PIVOT 'sales.csv' ON month USING SUM(revenue) GROUP BY product"

# UNPIVOT
duckdb -c "UNPIVOT 'wide_data.csv' ON (jan, feb, mar) INTO NAME month VALUE revenue"
```

### Create and Export

```bash
# Create persistent table from CSV
duckdb mydb.duckdb -c "CREATE TABLE sales AS SELECT * FROM 'sales.csv'"

# Export to Parquet
duckdb -c "COPY (SELECT * FROM 'data.csv' WHERE status='active') TO 'output.parquet' (FORMAT PARQUET)"

# Export to CSV
duckdb -c "COPY (SELECT * FROM 'data.parquet') TO 'output.csv' (HEADER, DELIMITER ',')"

# Export to JSON
duckdb -c "COPY (SELECT * FROM 'data.csv') TO 'output.json' (FORMAT JSON, ARRAY true)"
```

### Table Inspection

```bash
duckdb -c "DESCRIBE SELECT * FROM 'data.csv'"
duckdb -c "SUMMARIZE SELECT * FROM 'data.csv'"
```

### Regex and String Functions

```bash
duckdb -c "SELECT * FROM 'logs.csv' WHERE regexp_matches(message, 'ERROR|WARN')"
duckdb -c "SELECT regexp_extract(url, '/api/v(\d+)/', 1) as api_version FROM 'requests.csv'"
duckdb -c "SELECT regexp_replace(phone, '[^0-9]', '', 'g') as clean_phone FROM 'contacts.csv'"
```

### Advanced Types

```bash
# List operations
duckdb -c "SELECT list_value(1,2,3), list_aggregate([1,2,3], 'sum')"
duckdb -c "SELECT tags FROM 'data.json' WHERE list_contains(tags, 'urgent')"

# Struct operations
duckdb -c "SELECT struct_pack(name := 'test', value := 42)"

# Generate series
duckdb -c "SELECT * FROM generate_series(1, 100)"
duckdb -c "SELECT * FROM generate_series(DATE '2024-01-01', DATE '2024-12-31', INTERVAL 1 DAY)"
```

### Remote and Extensions

```bash
# Query remote files (httpfs extension)
duckdb -c "INSTALL httpfs; LOAD httpfs; SELECT * FROM 'https://example.com/data.csv' LIMIT 10"

# S3
duckdb -c "INSTALL httpfs; LOAD httpfs; SET s3_region='us-east-1'; SELECT * FROM 's3://bucket/data.parquet'"

# Attach SQLite database
duckdb -c "INSTALL sqlite; LOAD sqlite; ATTACH 'mydb.db' AS sqlite_db (TYPE SQLITE); SELECT * FROM sqlite_db.users"

# Attach PostgreSQL
duckdb -c "INSTALL postgres; LOAD postgres; ATTACH 'postgresql://user:pass@host/db' AS pg (TYPE POSTGRES); SELECT * FROM pg.users"
```

### EXPLAIN and Performance

```bash
duckdb -c "EXPLAIN ANALYZE SELECT * FROM 'large.csv' WHERE id > 1000"
duckdb -c "PRAGMA threads=4"

# Sample for quick analysis
duckdb -c "SELECT * FROM 'huge.csv' USING SAMPLE 1000"
duckdb -c "SELECT * FROM 'huge.csv' TABLESAMPLE 10 PERCENT"
```

### Parquet Metadata

```bash
duckdb -c "SELECT * FROM parquet_metadata('data.parquet')"
duckdb -c "SELECT * FROM parquet_schema('data.parquet')"
```

---

## PostgreSQL

### psql Meta-Commands

```bash
# Connect
psql -h localhost -U username -d dbname
psql "postgresql://user:pass@host:5432/dbname"

# Meta-commands
psql -d mydb -c "\dt"                 # List tables
psql -d mydb -c "\d users"            # Describe table
psql -d mydb -c "\l"                  # List databases
psql -d mydb -c "\du"                 # List roles
psql -d mydb -c "\df"                 # List functions
psql -d mydb -c "\di"                 # List indexes
psql -d mydb -c "\dn"                 # List schemas
psql -d mydb -c "\x" -c "SELECT * FROM users LIMIT 5"   # Expanded display
```

### Run Queries

```bash
# Single query
psql -d mydb -c "SELECT * FROM users WHERE active = true LIMIT 10"

# From file
psql -d mydb -f script.sql

# With timing
psql -d mydb -c "\timing on" -c "SELECT COUNT(*) FROM large_table"

# Copy to/from CSV
psql -d mydb -c "\copy users TO 'users.csv' WITH CSV HEADER"
psql -d mydb -c "\copy users FROM 'import.csv' WITH CSV HEADER"
psql -d mydb -c "\copy (SELECT * FROM users WHERE active) TO 'active.csv' WITH CSV HEADER"

# Watch (repeat query every N seconds)
psql -d mydb -c "\watch 5" -c "SELECT COUNT(*) FROM queue WHERE status = 'pending'"
```

### EXPLAIN (Query Performance)

```bash
psql -d mydb -c "EXPLAIN SELECT * FROM users WHERE email = 'test@test.com'"
psql -d mydb -c "EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@test.com'"
psql -d mydb -c "EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) SELECT * FROM orders JOIN users ON orders.user_id = users.id"

# Reading EXPLAIN output:
# Seq Scan = full table scan (needs index?)
# Index Scan = using index (good)
# Bitmap Index Scan = multiple index conditions
# Nested Loop / Hash Join / Merge Join = join strategies
# cost=startup..total  rows=estimated  actual time=real ms
```

### Common Patterns

```bash
# CTEs
psql -d mydb <<'SQL'
WITH monthly AS (
    SELECT date_trunc('month', created_at) as month, COUNT(*) as cnt
    FROM orders GROUP BY 1
)
SELECT month, cnt, SUM(cnt) OVER (ORDER BY month) as cumulative
FROM monthly ORDER BY month;
SQL

# DISTINCT ON (first row per group)
psql -d mydb -c "SELECT DISTINCT ON (customer_id) * FROM orders ORDER BY customer_id, created_at DESC"

# Aggregation functions
psql -d mydb -c "SELECT array_agg(name) FROM users WHERE department = 'eng'"
psql -d mydb -c "SELECT string_agg(tag, ', ') FROM tags WHERE post_id = 1"
psql -d mydb -c "SELECT * FROM generate_series('2024-01-01'::date, '2024-12-31'::date, '1 month'::interval)"
psql -d mydb -c "SELECT date_trunc('hour', created_at) as hour, COUNT(*) FROM events GROUP BY 1 ORDER BY 1"
psql -d mydb -c "SELECT EXTRACT(dow FROM created_at) as day_of_week, COUNT(*) FROM orders GROUP BY 1"

# LATERAL join
psql -d mydb <<'SQL'
SELECT u.name, recent.* FROM users u,
LATERAL (SELECT * FROM orders WHERE user_id = u.id ORDER BY created_at DESC LIMIT 3) recent;
SQL
```

### Index Types

```bash
# B-tree (default, most common)
psql -d mydb -c "CREATE INDEX idx_users_email ON users(email)"

# Hash (equality only, smaller)
psql -d mydb -c "CREATE INDEX idx_users_token ON users USING hash(api_token)"

# GIN (arrays, JSONB, full-text search)
psql -d mydb -c "CREATE INDEX idx_posts_tags ON posts USING gin(tags)"
psql -d mydb -c "CREATE INDEX idx_docs_data ON docs USING gin(data jsonb_path_ops)"

# GiST (geometry, ranges)
psql -d mydb -c "CREATE INDEX idx_locations_geom ON locations USING gist(geom)"

# BRIN (large sorted tables, small index)
psql -d mydb -c "CREATE INDEX idx_events_created ON events USING brin(created_at)"
```

### JSON Operators (JSONB)

```bash
psql -d mydb -c "SELECT data->'name' FROM docs"                    # JSON object
psql -d mydb -c "SELECT data->>'name' FROM docs"                   # Text value
psql -d mydb -c "SELECT data#>'{address,city}' FROM docs"          # Path access
psql -d mydb -c "SELECT * FROM docs WHERE data @> '{\"active\": true}'"  # Contains
psql -d mydb -c "SELECT jsonb_path_query(data, '$.items[*].name') FROM docs"
psql -d mydb -c "SELECT jsonb_agg(row_to_json(u)) FROM users u"
psql -d mydb -c "SELECT jsonb_build_object('id', id, 'name', name) FROM users"
```

### Array Operations

```bash
psql -d mydb -c "SELECT * FROM posts WHERE 'python' = ANY(tags)"
psql -d mydb -c "SELECT * FROM posts WHERE tags @> ARRAY['python','sql']"
psql -d mydb -c "SELECT unnest(tags) as tag, COUNT(*) FROM posts GROUP BY tag ORDER BY 2 DESC"
psql -d mydb -c "SELECT array_agg(DISTINCT tag) FROM posts, unnest(tags) as tag"
```

### Backup and Restore

```bash
# Custom format (compressed, selective restore)
pg_dump -Fc -d mydb -f backup.dump
pg_restore -d mydb --clean --jobs 4 backup.dump

# Plain SQL
pg_dump -Fp -d mydb -f backup.sql
pg_dump -Fp --schema-only -d mydb -f schema.sql
pg_dump -Fp --data-only -d mydb -f data.sql

# Specific tables
pg_dump -Fc -d mydb --table=users --table=orders -f partial.dump

# Directory format (parallel dump)
pg_dump -Fd -d mydb -j 4 -f backup_dir/
pg_restore -d newdb -j 4 backup_dir/
```

### Role Management

```bash
psql -d mydb -c "CREATE ROLE readonly LOGIN PASSWORD 'pass'"
psql -d mydb -c "GRANT CONNECT ON DATABASE mydb TO readonly"
psql -d mydb -c "GRANT USAGE ON SCHEMA public TO readonly"
psql -d mydb -c "GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly"
psql -d mydb -c "ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly"
```

### Connection Strings and Variables

```bash
# Connection string
psql "postgresql://user:pass@host:5432/dbname?sslmode=require"

# Environment variables
PGHOST=localhost PGUSER=myuser PGDATABASE=mydb psql

# psql variables
psql -d mydb -v table_name='users' -c "SELECT * FROM :table_name LIMIT 10"
```

---

## MySQL / MariaDB

### Connect and Query

```bash
mysql -h localhost -u username -p -D dbname
mysql -h localhost -u root -p -e "SELECT * FROM users LIMIT 10" mydb
mysql -h localhost -u root -p mydb < script.sql
```

### Schema Inspection

```bash
mysql -u root -p -e "SHOW DATABASES"
mysql -u root -p -D mydb -e "SHOW TABLES"
mysql -u root -p -D mydb -e "DESCRIBE users"
mysql -u root -p -D mydb -e "SHOW CREATE TABLE users"
mysql -u root -p -D mydb -e "SHOW PROCESSLIST"
mysql -u root -p -D mydb -e "SHOW VARIABLES LIKE '%max_connections%'"
mysql -u root -p -D mydb -e "SHOW INDEX FROM users"
```

### Backup and Restore

```bash
# Full database dump
mysqldump -u root -p --single-transaction mydb > backup.sql
mysqldump -u root -p --single-transaction --routines --triggers mydb > full_backup.sql

# Specific tables
mysqldump -u root -p mydb users orders > tables.sql

# With WHERE clause
mysqldump -u root -p mydb users --where="created_at > '2024-01-01'" > recent_users.sql

# Restore
mysql -u root -p mydb < backup.sql
```

### User Management

```bash
mysql -u root -p -e "CREATE USER 'appuser'@'%' IDENTIFIED BY 'password'"
mysql -u root -p -e "GRANT SELECT, INSERT, UPDATE ON mydb.* TO 'appuser'@'%'"
mysql -u root -p -e "FLUSH PRIVILEGES"
mysql -u root -p -e "SHOW GRANTS FOR 'appuser'@'%'"
```

### JSON Functions
```bash
# Extract JSON value
mysql -u root -p -D mydb -e "SELECT JSON_EXTRACT(data, '$.name') FROM records"

# Shorthand arrow operators
mysql -u root -p -D mydb -e "SELECT data->'$.name' FROM records"        # JSON value
mysql -u root -p -D mydb -e "SELECT data->>'$.name' FROM records"       # Unquoted text

# Search in JSON arrays
mysql -u root -p -D mydb -e "SELECT * FROM records WHERE JSON_CONTAINS(data->'$.tags', '\"urgent\"')"

# Modify JSON
mysql -u root -p -D mydb -e "UPDATE records SET data = JSON_SET(data, '$.status', 'done') WHERE id = 1"

# JSON aggregation
mysql -u root -p -D mydb -e "SELECT JSON_ARRAYAGG(name) FROM users WHERE active = 1"
mysql -u root -p -D mydb -e "SELECT JSON_OBJECTAGG(id, name) FROM users"
```

### Query Performance (EXPLAIN)
```bash
# Basic explain
mysql -u root -p -D mydb -e "EXPLAIN SELECT * FROM users WHERE email = 'test@test.com'"

# Extended explain with JSON format
mysql -u root -p -D mydb -e "EXPLAIN FORMAT=JSON SELECT * FROM orders WHERE user_id = 42"

# Analyze actual execution
mysql -u root -p -D mydb -e "EXPLAIN ANALYZE SELECT * FROM orders JOIN users ON orders.user_id = users.id"

# Show slow queries
mysql -u root -p -e "SHOW VARIABLES LIKE 'slow_query_log%'"
mysql -u root -p -e "SET GLOBAL slow_query_log = 'ON'"
mysql -u root -p -e "SET GLOBAL long_query_time = 1"
```

### Process and Connection Management
```bash
# Show active queries
mysql -u root -p -e "SHOW PROCESSLIST"
mysql -u root -p -e "SHOW FULL PROCESSLIST"

# Kill a query
mysql -u root -p -e "KILL QUERY 42"
mysql -u root -p -e "KILL 42"          # Kill connection

# Connection and session info
mysql -u root -p -e "SHOW STATUS LIKE 'Threads_connected'"
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections'"
mysql -u root -p -e "SELECT @@global.max_connections"
```

### Index Management
```bash
# Create indexes
mysql -u root -p -D mydb -e "CREATE INDEX idx_users_email ON users(email)"
mysql -u root -p -D mydb -e "CREATE UNIQUE INDEX idx_users_username ON users(username)"

# Composite index
mysql -u root -p -D mydb -e "CREATE INDEX idx_orders_user_date ON orders(user_id, created_at)"

# Full-text index
mysql -u root -p -D mydb -e "CREATE FULLTEXT INDEX idx_posts_content ON posts(title, body)"
mysql -u root -p -D mydb -e "SELECT * FROM posts WHERE MATCH(title, body) AGAINST('search term')"

# Show indexes
mysql -u root -p -D mydb -e "SHOW INDEX FROM users"

# Drop index
mysql -u root -p -D mydb -e "DROP INDEX idx_users_email ON users"
```

### Variables and Configuration
```bash
# Show key variables
mysql -u root -p -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size'"
mysql -u root -p -e "SHOW VARIABLES LIKE '%timeout%'"
mysql -u root -p -e "SHOW VARIABLES LIKE '%log%'"

# Show server status
mysql -u root -p -e "SHOW GLOBAL STATUS LIKE 'Innodb_rows%'"
mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G"

# Session-level settings
mysql -u root -p -D mydb -e "SET SESSION group_concat_max_len = 1000000"
mysql -u root -p -D mydb -e "SET SESSION sql_mode = 'STRICT_TRANS_TABLES'"
```

### Connection Patterns
```bash
# Connect with specific charset
mysql -h localhost -u user -p --default-character-set=utf8mb4 mydb

# Execute and format output
mysql -u root -p -D mydb -e "SELECT * FROM users" --table     # Table format
mysql -u root -p -D mydb -e "SELECT * FROM users" --batch      # Tab-separated
mysql -u root -p -D mydb -e "SELECT * FROM users" --xml        # XML output
mysql -u root -p -D mydb -e "SELECT * FROM users" --json       # JSON output (8.0+)

# Connect via socket
mysql -S /var/run/mysqld/mysqld.sock -u root -p

# Import with progress
pv backup.sql | mysql -u root -p mydb
```

---

## Redis

### Strings

```bash
redis-cli SET mykey "hello"
redis-cli GET mykey
redis-cli SET counter 0
redis-cli INCR counter
redis-cli INCRBY counter 10
redis-cli DECR counter
redis-cli DECRBY counter 5
redis-cli APPEND mykey " world"
redis-cli STRLEN mykey
redis-cli MSET key1 "val1" key2 "val2" key3 "val3"
redis-cli MGET key1 key2 key3
redis-cli SET session "data" EX 3600            # Expire in 3600 seconds
redis-cli SET session "data" PX 60000           # Expire in 60000 milliseconds
redis-cli SET lock "owner" NX                   # Only if not exists (distributed lock)
redis-cli SET config "val" XX                   # Only if exists (update only)
redis-cli GETSET counter 0                      # Get old value, set new
redis-cli SETNX lockkey "holder"                # SET if Not eXists
```

### Hashes

```bash
redis-cli HSET user:1 name "Alice" email "alice@test.com" age 30
redis-cli HGET user:1 name
redis-cli HGETALL user:1
redis-cli HMSET user:2 name "Bob" email "bob@test.com"
redis-cli HDEL user:1 age
redis-cli HEXISTS user:1 email
redis-cli HKEYS user:1
redis-cli HVALS user:1
redis-cli HINCRBY user:1 age 1
redis-cli HSCAN user:1 0 MATCH "na*"
```

### Lists

```bash
redis-cli LPUSH queue "task1"                   # Push left
redis-cli RPUSH queue "task2"                   # Push right
redis-cli LPOP queue                            # Pop left
redis-cli RPOP queue                            # Pop right
redis-cli LRANGE queue 0 -1                     # All elements
redis-cli LLEN queue                            # Length
redis-cli LINDEX queue 0                        # Element at index
redis-cli LSET queue 0 "updated_task"           # Set at index
redis-cli LTRIM queue 0 99                      # Keep only first 100
redis-cli BRPOP queue 30                        # Blocking pop (30s timeout)
```

### Sets

```bash
redis-cli SADD tags "python" "sql" "redis"
redis-cli SMEMBERS tags
redis-cli SISMEMBER tags "python"
redis-cli SCARD tags                            # Count
redis-cli SINTER set1 set2                      # Intersection
redis-cli SUNION set1 set2                      # Union
redis-cli SDIFF set1 set2                       # Difference
redis-cli SINTERSTORE result set1 set2          # Store intersection
redis-cli SRANDMEMBER tags 2                    # 2 random members
redis-cli SPOP tags                             # Remove random member
```

### Sorted Sets

```bash
redis-cli ZADD leaderboard 100 "alice" 85 "bob" 92 "charlie"
redis-cli ZRANGE leaderboard 0 -1 WITHSCORES          # Ascending
redis-cli ZREVRANGE leaderboard 0 -1 WITHSCORES       # Descending
redis-cli ZRANGEBYSCORE leaderboard 80 100             # Score range
redis-cli ZRANK leaderboard "alice"                    # Rank (0-based)
redis-cli ZSCORE leaderboard "alice"                   # Score
redis-cli ZINCRBY leaderboard 5 "bob"                  # Increment score
redis-cli ZCARD leaderboard                            # Count
redis-cli ZRANGEBYLEX leaderboard "[a" "[d"            # Lexicographic range
```

### Pub/Sub

```bash
# Subscribe (blocks waiting for messages)
redis-cli SUBSCRIBE notifications

# Publish (in another terminal)
redis-cli PUBLISH notifications "new event"

# Pattern subscribe
redis-cli PSUBSCRIBE "events.*"
```

### Key Management

```bash
redis-cli SCAN 0 MATCH "user:*" COUNT 100       # Safe iteration (use in prod!)
redis-cli KEYS "user:*"                          # Pattern match (avoid in prod, use SCAN)
redis-cli TYPE mykey
redis-cli DEL key1 key2
redis-cli UNLINK key1 key2                       # Async delete (non-blocking)
redis-cli EXPIRE mykey 3600                      # Set TTL in seconds
redis-cli PEXPIRE mykey 60000                    # Set TTL in milliseconds
redis-cli TTL mykey                              # Remaining TTL (seconds)
redis-cli PTTL mykey                             # Remaining TTL (ms)
redis-cli PERSIST mykey                          # Remove expiration
redis-cli EXISTS mykey
redis-cli RENAME oldkey newkey
redis-cli RANDOMKEY
redis-cli OBJECT ENCODING mykey                  # Internal encoding
```

### Persistence and Admin

```bash
redis-cli SAVE                                   # Synchronous save (blocks)
redis-cli BGSAVE                                 # Background save
redis-cli LASTSAVE                               # Timestamp of last save
redis-cli BGREWRITEAOF                           # Rewrite AOF file
redis-cli CONFIG SET appendonly yes               # Enable AOF
redis-cli CONFIG SET save "60 1000"               # RDB: save every 60s if 1000 keys changed
redis-cli INFO server                            # Server info
redis-cli INFO memory                            # Memory usage
redis-cli DBSIZE                                 # Key count
redis-cli FLUSHDB                                # Delete all keys in current DB
```

### Transactions

```bash
# Multi/Exec (atomic batch)
redis-cli <<'EOF'
MULTI
SET balance:alice 100
SET balance:bob 200
EXEC
EOF

# Watch (optimistic locking)
redis-cli WATCH balance:alice
redis-cli MULTI
redis-cli DECRBY balance:alice 50
redis-cli INCRBY balance:bob 50
redis-cli EXEC                                   # Fails if balance:alice changed
```

### Lua Scripting

```bash
redis-cli EVAL "return redis.call('get',KEYS[1])" 1 mykey
redis-cli EVAL "redis.call('set',KEYS[1],ARGV[1]); return redis.call('get',KEYS[1])" 1 mykey "newvalue"
redis-cli EVAL "local val = redis.call('get',KEYS[1]); return tonumber(val) * 2" 1 counter
```

### redis-cli Patterns

```bash
redis-cli --pipe < commands.txt                  # Mass insert (Redis protocol)
redis-cli --bigkeys                              # Find large keys
redis-cli --memkeys                              # Memory usage per key
redis-cli --scan --pattern "session:*"           # Safe key scan
redis-cli --stat                                 # Continuous stats
redis-cli --latency                              # Latency measurement
redis-cli -n 2                                   # Use database 2
redis-cli --raw                                  # Raw output (no formatting)
redis-cli --csv                                  # CSV output
redis-cli monitor                                # Real-time command log
```

---

## datasette

### Serve and Explore

```bash
datasette serve db.db                            # Start server on port 8001
datasette serve db.db --port 8080                # Custom port
datasette serve db.db --host 0.0.0.0             # Listen on all interfaces
datasette serve db.db -o                         # Open browser automatically
datasette serve db.db --cors                     # Enable CORS headers
datasette serve db.db --setting max_returned_rows 5000
datasette serve db1.db db2.db                    # Multiple databases
```

### SQL API

```bash
# Query via URL
curl "http://localhost:8001/db.json?sql=SELECT+*+FROM+users+LIMIT+10"
curl "http://localhost:8001/db/-/sql.json?sql=SELECT+count(*)+FROM+users"
```

### Publish

```bash
datasette publish cloudrun db.db --service my-datasette
datasette publish vercel db.db --project my-datasette
```

### Plugins

```bash
pip install datasette-vega                       # Charts
pip install datasette-cluster-map                # Map visualization
pip install datasette-dashboards                 # Dashboard builder
datasette serve db.db --install datasette-vega   # Install plugin at startup
```

---

## Cross-Database Patterns

### CSV-to-SQL Import Workflow

```bash
# 1. Inspect CSV headers and sample
head -5 data.csv
duckdb -c "DESCRIBE SELECT * FROM 'data.csv'"

# 2. Import to SQLite (quick)
sqlite-utils insert analysis.db data data.csv --csv

# 3. Verify
sqlite-utils tables analysis.db --counts
sqlite-utils query analysis.db "SELECT * FROM data LIMIT 5"
```

### Format Conversion with DuckDB

```bash
# CSV to Parquet
duckdb -c "COPY (SELECT * FROM 'data.csv') TO 'data.parquet' (FORMAT PARQUET)"

# Parquet to CSV
duckdb -c "COPY (SELECT * FROM 'data.parquet') TO 'data.csv' (HEADER, DELIMITER ',')"

# JSON to CSV
duckdb -c "COPY (SELECT * FROM read_json_auto('data.json')) TO 'data.csv' (HEADER)"

# CSV to JSON
duckdb -c "COPY (SELECT * FROM 'data.csv') TO 'data.json' (FORMAT JSON, ARRAY true)"

# Parquet to JSON
duckdb -c "COPY (SELECT * FROM 'data.parquet') TO 'data.json' (FORMAT JSON, ARRAY true)"
```

### Data Migration Between Databases

```bash
# SQLite to PostgreSQL
sqlite3 -csv db.db "SELECT * FROM users" > /tmp/users.csv
psql -d target -c "\copy users FROM '/tmp/users.csv' WITH CSV HEADER"

# PostgreSQL to SQLite
psql -d source -c "\copy (SELECT * FROM users) TO '/tmp/users.csv' WITH CSV HEADER"
sqlite-utils insert target.db users /tmp/users.csv --csv

# SQLite to DuckDB (attach)
duckdb mydb.duckdb -c "INSTALL sqlite; LOAD sqlite; CREATE TABLE users AS SELECT * FROM sqlite_scan('old.db', 'users')"

# Compare row counts across databases
echo "SQLite:"; sqlite3 db.db "SELECT COUNT(*) FROM users"
echo "Postgres:"; psql -d mydb -tc "SELECT COUNT(*) FROM users"
```

### Query Results to File

```bash
# SQLite query to CSV
sqlite3 -header -csv db.db "SELECT * FROM report_view" > report.csv

# PostgreSQL query to CSV
psql -d mydb -c "\copy (SELECT * FROM users WHERE created_at > '2024-01-01') TO 'recent_users.csv' WITH CSV HEADER"

# DuckDB query to Parquet
duckdb -c "COPY (SELECT date, SUM(amount) FROM 'transactions.csv' GROUP BY date) TO 'daily_totals.parquet' (FORMAT PARQUET)"
```

### SQLite Maintenance

```bash
# VACUUM — reclaim space and defragment
sqlite3 mydb.db "VACUUM;"                      # Rebuild entire database
sqlite3 mydb.db "PRAGMA auto_vacuum = FULL;"   # Enable auto-vacuum

# Integrity check
sqlite3 mydb.db "PRAGMA integrity_check;"      # Verify database integrity
sqlite3 mydb.db "PRAGMA quick_check;"          # Faster partial check

# Database info
sqlite3 mydb.db ".dbinfo"                      # Database metadata
sqlite3 mydb.db "PRAGMA page_count; PRAGMA page_size;"  # Size info
```

### PostgreSQL Active Queries and Locks

```bash
# View active queries
psql -d mydb -c "SELECT pid, now() - pg_stat_activity.query_start AS duration, query, state
FROM pg_stat_activity WHERE state != 'idle' ORDER BY duration DESC;"

# Find blocking locks
psql -d mydb -c "SELECT blocked.pid AS blocked_pid, blocked.query AS blocked_query,
  blocking.pid AS blocking_pid, blocking.query AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_locks bl ON bl.pid = blocked.pid
JOIN pg_locks kl ON kl.locktype = bl.locktype AND kl.relation = bl.relation AND kl.pid != bl.pid
JOIN pg_stat_activity blocking ON blocking.pid = kl.pid
WHERE NOT bl.granted;"

# Kill long-running query
psql -d mydb -c "SELECT pg_cancel_backend(PID);"      # Graceful cancel
psql -d mydb -c "SELECT pg_terminate_backend(PID);"   # Force terminate
```

### Redis Streams

```bash
# Add to stream
redis-cli XADD mystream '*' key1 value1 key2 value2   # Auto-generated ID

# Read from stream
redis-cli XRANGE mystream - +                  # All entries
redis-cli XRANGE mystream - + COUNT 10         # Last 10 entries
redis-cli XREAD COUNT 5 BLOCK 5000 STREAMS mystream '$'  # Block-read new entries

# Consumer groups
redis-cli XGROUP CREATE mystream mygroup 0     # Create consumer group
redis-cli XREADGROUP GROUP mygroup consumer1 COUNT 1 STREAMS mystream >  # Read as consumer
redis-cli XACK mystream mygroup entry-id       # Acknowledge processing
redis-cli XINFO STREAM mystream                # Stream info
```

### Combined Workflows

```bash
# PostgreSQL query → JSON
psql -d mydb -Atc "SELECT row_to_json(t) FROM (SELECT id, name, email FROM users LIMIT 10) t" | jq .

# CSV → DuckDB analysis
duckdb -c "SELECT column_name, COUNT(*), AVG(value) FROM 'data.csv' GROUP BY column_name ORDER BY COUNT(*) DESC"

# SQLite → datasette for instant web API
datasette serve mydb.db --port 8001            # Browse at http://localhost:8001
```

---

## Installation

```bash
# SQLite (usually pre-installed)
sudo apt install sqlite3

# sqlite-utils
pip install sqlite-utils

# DuckDB
pip install duckdb

# PostgreSQL client
sudo apt install postgresql-client

# MySQL client
sudo apt install mysql-client

# Redis
sudo apt install redis-tools

# datasette
pip install datasette
```
