---
description: "Tabular data processing - need to analyze spreadsheet data, CSV has wrong delimiter, calculate totals from columns, find duplicates in data, extracting columns, filtering rows, and aggregating CSV/TSV (awk, cut, miller, datamash)"
when_to_use: "Use when: need to analyze spreadsheet or CSV data from command line, CSV has wrong delimiter or encoding, need to calculate totals or averages from columns, finding duplicates in tabular data, extracting specific columns, filtering rows by condition, sorting or grouping data, changing delimiters. Triggers: analyze spreadsheet, wrong delimiter, calculate totals, find duplicates, summarize data, csv, tsv, column, sum column, average, filter rows, sort by field, delimited, tabular, awk, cut, parse csv, extract columns, group by, pivot table, data processing, csv filter, aggregate, statistics, frequency count, histogram, fixed width, top N, crosstab, data summary, miller, datamash, qsv, xsv."
when-to-use: "Use when: need to analyze spreadsheet or CSV data from command line, CSV has wrong delimiter or encoding, need to calculate totals or averages from columns, finding duplicates in tabular data, extracting specific columns, filtering rows by condition, sorting or grouping data, changing delimiters. Triggers: analyze spreadsheet, wrong delimiter, calculate totals, find duplicates, summarize data, csv, tsv, column, sum column, average, filter rows, sort by field, delimited, tabular, awk, cut, parse csv, extract columns, group by, pivot table, data processing, csv filter, aggregate, statistics, frequency count, histogram, fixed width, top N, crosstab, data summary, miller, datamash, qsv, xsv."
argument-hint: "[describe what you want to do with tabular data]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Tabular Data Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the tabular recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Goal
What tabular operation is needed?
- Extract columns, filter rows, sort, aggregate, reformat, count, group

### 2. File Path(s)
Where is the data?
- Explicit paths mentioned by user
- Files created/modified earlier in conversation
- Files referenced in error messages

### 3. Structure
How is the data organized?
- **Delimiter**: comma (CSV), tab (TSV), pipe, space, custom
- **Header**: first row is header? (important for skip logic)
- **Column numbers/names**: which columns to operate on (1-indexed)

### 4. Filter Conditions
What constraints apply?
- Numeric: `$3 > 100`, `$2 >= 50 && $2 <= 100`
- String: `$2 == "active"`, `$2 ~ /error/`
- Empty/non-empty: `$3 != ""`

### 5. Output Format
What should the result look like?
- Same delimiter or different
- Specific columns only
- With/without header
- Pretty-printed table

### 6. Aggregation
If aggregating:
- Sum, average, min, max, count
- Which column(s)
- Grouped by which column?

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Sum/average column | awk | Built-in arithmetic |
| Extract columns | cut | Simplest for positional extraction |
| Query CSV like SQL | mlr (miller) | Full DSL for tabular data |
| High-perf CSV ops | qsv / xsv | Rust-based, very fast |
| Sort rows | sort | Purpose-built, handles large files |
| Deduplicate rows | sort -u / awk | sort -u for simple, awk for by-column |
| Pretty-print table | column -t | Aligns columns for display |
| Statistics (mean, stdev) | datamash | GNU datamash for column stats |
| Frequency counts | awk / sort+uniq | Associative arrays or pipeline |

---

## Argument Format

Construct a structured argument string:

```
<goal> from <file_path> - delimiter: <char>, columns: <which>, filter: <condition>, output: <format>
```

**Include only relevant components.**

### Examples

**Column extraction:**
```
extract columns 1,3,5 from /tmp/data.csv - delimiter: comma, output: tab-separated
```

**Filtering:**
```
filter rows from /home/user/sales.csv - delimiter: comma, filter: column 3 > 1000, skip header
```

**Aggregation:**
```
sum column 4 from ./transactions.tsv - delimiter: tab, group by column 1, skip header
```

**Sorting:**
```
sort by column 2 numeric descending from /data/scores.csv - delimiter: comma, skip header
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-tabular-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- File path cannot be determined from context
- Delimiter is ambiguous (could be comma or tab)
- Column numbers are unclear

**Use Read tool first if:**
- Need to inspect file to determine delimiter
- Need to see column headers to map names to numbers
