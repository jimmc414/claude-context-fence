---
description: "Tabular recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-tabular with file paths and context included"
when-to-use: "Receives prepared arguments from task-tabular with file paths and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Tabular Data Processing Recipes

**Tools:** awk, cut, sort, uniq, column, paste, join, comm, tsort, miller (mlr), datamash, qsv, xsv, q, textql, numaverage, numbound, numgrep, numinterval, numnormalize, numprocess, numrandom, numrange, numround, numsum

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Extract column 3 | cut | `cut -d',' -f3 file.csv` |
| Extract multiple columns | cut | `cut -d',' -f1,3,5 file.csv` |
| Sum column 3 | awk | `awk -F',' '{s+=$3} END {print s}' file.csv` |
| Average column 3 | awk | `awk -F',' '{s+=$3; n++} END {print s/n}' file.csv` |
| Filter rows | awk | `awk -F',' '$3 > 100' file.csv` |
| Sort by column 2 | sort | `sort -t',' -k2 file.csv` |
| Sort numerically | sort | `sort -t',' -k2 -n file.csv` |
| Unique values | cut+sort | `cut -d',' -f2 file.csv \| sort -u` |
| Count by category | awk | `awk -F',' '{c[$1]++} END {for(k in c) print k,c[k]}' file.csv` |
| Format as table | column | `column -t -s',' file.csv` |
| Topological sort | tsort | `tsort dependencies.txt` |
| Modern CSV query | mlr | `mlr --csv filter '$revenue > 1000' data.csv` |
| Fast CSV toolkit | qsv | `qsv stats data.csv` |
| SQL on CSV | q | `q "SELECT * FROM data.csv WHERE age > 30"` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Simple column extract | cut | Fastest, simplest, no install needed |
| Multiple columns, reorder | cut or awk | cut for simple, awk for complex |
| Conditional filtering | awk or mlr | awk is universal; mlr handles CSV quoting |
| Math on columns | awk or datamash | awk for custom; datamash for standard stats |
| Sorting | sort | Purpose-built, fast for large files |
| Counting/grouping | awk or mlr | Associative arrays (awk) or verb chains (mlr) |
| Pretty display | column | Aligns columns nicely |
| Full CSV processing | mlr or qsv | Handles quoting, headers, streaming |
| SQL queries on CSV | q or textql | Write SQL instead of awk/sed |
| Statistical summary | datamash or qsv stats | Quick stats across columns |

---

## Extracting Columns

### With cut (simple extraction)
```bash
# Single column (1-indexed)
cut -d',' -f3 file.csv

# Multiple columns
cut -d',' -f1,3,5 file.csv

# Range of columns
cut -d',' -f2-5 file.csv

# Tab-delimited (default)
cut -f2 file.tsv

# Custom delimiter
cut -d'|' -f2 file.txt
```

### With awk (more control)
```bash
# Single column
awk -F',' '{print $3}' file.csv

# Multiple columns with new delimiter
awk -F',' -v OFS='\t' '{print $1, $3, $5}' file.csv

# Last column
awk -F',' '{print $NF}' file.csv

# Second-to-last
awk -F',' '{print $(NF-1)}' file.csv

# Skip empty values
awk -F',' '$3 != "" {print $3}' file.csv
```

---

## Filtering Rows

### By Field Value
```bash
# Numeric comparison
awk -F',' '$3 > 100' file.csv
awk -F',' '$3 >= 50 && $3 <= 100' file.csv
awk -F',' '$3 != 0' file.csv

# String match (exact)
awk -F',' '$2 == "active"' file.csv
awk -F',' '$2 != "pending"' file.csv

# String match (contains)
awk -F',' '$2 ~ /error/' file.csv
awk -F',' '$2 ~ /^test/' file.csv        # Starts with
awk -F',' '$2 ~ /done$/' file.csv        # Ends with

# Case insensitive
awk -F',' 'tolower($2) ~ /error/' file.csv

# Not matching
awk -F',' '$2 !~ /test/' file.csv
```

### Skip Header
```bash
awk -F',' 'NR > 1' file.csv
tail -n +2 file.csv
sed '1d' file.csv
```

### Keep Header + Filter
```bash
awk -F',' 'NR == 1 || $3 > 100' file.csv
```

### Non-empty Fields
```bash
awk -F',' '$3 != ""' file.csv
awk -F',' 'length($3) > 0' file.csv
```

### Multiple Conditions
```bash
awk -F',' '$2 == "active" && $3 > 100' file.csv
awk -F',' '$2 == "error" || $2 == "warning"' file.csv
```

---

## Sorting

### Basic Sort
```bash
# By column 2, alphabetic
sort -t',' -k2 file.csv

# By column 2, numeric
sort -t',' -k2 -n file.csv

# Reverse order
sort -t',' -k2 -rn file.csv

# Multiple keys (col 1 alpha, then col 2 numeric)
sort -t',' -k1,1 -k2,2n file.csv

# Ignore case
sort -t',' -k2 -f file.csv
```

### Sort and Remove Duplicates
```bash
sort -t',' -k2 -u file.csv
```

### Keep Header While Sorting
```bash
(head -1 file.csv && tail -n +2 file.csv | sort -t',' -k2)
(head -1 file.csv && tail -n +2 file.csv | sort -t',' -k2 -rn)
```

### Sort by Multiple Columns
```bash
# Primary: column 1, Secondary: column 2 (numeric)
sort -t',' -k1,1 -k2,2n file.csv
```

---

## Aggregation

### Sum a Column
```bash
awk -F',' '{sum += $3} END {print sum}' file.csv

# Skip header
awk -F',' 'NR > 1 {sum += $3} END {print sum}' file.csv

# With label
awk -F',' 'NR > 1 {sum += $3} END {print "Total:", sum}' file.csv
```

### Average
```bash
awk -F',' 'NR > 1 {sum += $3; n++} END {print sum/n}' file.csv

# With formatting
awk -F',' 'NR > 1 {sum += $3; n++} END {printf "%.2f\n", sum/n}' file.csv
```

### Min / Max
```bash
# Maximum value in column 3
awk -F',' 'NR==2 || $3 > max {max=$3} END {print max}' file.csv

# Minimum value
awk -F',' 'NR==2 || $3 < min {min=$3} END {print min}' file.csv

# Row with maximum
awk -F',' 'NR==2 || $3 > max {max=$3; row=$0} END {print row}' file.csv
```

### Count Rows
```bash
wc -l < file.csv                           # Total lines
awk -F',' 'NR > 1' file.csv | wc -l        # Excluding header
awk -F',' '$2 == "error"' file.csv | wc -l # Matching rows
```

### Count Distinct Values
```bash
awk -F',' 'NR > 1 {seen[$2]} END {print length(seen)}' file.csv
cut -d',' -f2 file.csv | tail -n +2 | sort -u | wc -l
```

---

## Grouping and Counting

### Count by Category
```bash
awk -F',' 'NR > 1 {count[$1]++} END {for (k in count) print k, count[k]}' file.csv

# Sorted output
awk -F',' 'NR > 1 {count[$1]++} END {for (k in count) print k, count[k]}' file.csv | sort -k2 -rn
```

### Sum by Category
```bash
awk -F',' 'NR > 1 {sum[$1] += $3} END {for (k in sum) print k, sum[k]}' file.csv
```

### Average by Category
```bash
awk -F',' 'NR > 1 {sum[$1] += $3; count[$1]++} END {for (k in sum) print k, sum[k]/count[k]}' file.csv
```

### Multiple Aggregations
```bash
awk -F',' 'NR > 1 {
    count[$1]++
    sum[$1] += $3
} END {
    for (k in count)
        printf "%s: count=%d sum=%.2f avg=%.2f\n", k, count[k], sum[k], sum[k]/count[k]
}' file.csv
```

### Group by Multiple Columns
```bash
awk -F',' 'NR > 1 {key = $1 ":" $2; count[key]++} END {for (k in count) print k, count[k]}' file.csv
```

---

## Unique Values

### Unique Values in Column
```bash
cut -d',' -f2 file.csv | sort -u
awk -F',' '!seen[$2]++ {print $2}' file.csv  # Preserves order
```

### Count Unique Values
```bash
cut -d',' -f2 file.csv | sort -u | wc -l
```

### Unique Rows (full line)
```bash
sort -u file.csv
awk '!seen[$0]++' file.csv      # Preserves order
```

### Unique by Specific Column
```bash
awk -F',' '!seen[$2]++' file.csv
```

### Deduplicate Keeping First/Last
```bash
# Keep first occurrence
awk -F',' '!seen[$1]++' file.csv

# Keep last occurrence
tac file.csv | awk -F',' '!seen[$1]++' | tac
```

---

## Reformatting

### Change Delimiter
```bash
# CSV to TSV
awk -F',' -v OFS='\t' '{$1=$1; print}' file.csv
sed 's/,/\t/g' file.csv
tr ',' '\t' < file.csv

# TSV to CSV
awk -F'\t' -v OFS=',' '{$1=$1; print}' file.tsv

# To pipe-delimited
awk -F',' -v OFS='|' '{$1=$1; print}' file.csv
```

### Pretty Print as Table
```bash
column -t -s',' file.csv
column -t -s$'\t' file.tsv
```

### Add Line Numbers
```bash
awk -F',' '{print NR","$0}' file.csv
nl -s',' file.csv
cat -n file.csv
```

### Reorder Columns
```bash
awk -F',' -v OFS=',' '{print $3, $1, $2}' file.csv
cut -d',' -f3,1,2 file.csv     # Note: cut outputs in original order
```

### Add/Remove Quotes
```bash
# Add quotes to all fields
awk -F',' '{for(i=1;i<=NF;i++) $i="\""$i"\""; print}' OFS=',' file.csv

# Remove quotes
sed 's/"//g' file.csv
```

---

## Combining Files

### Vertical Append (stack rows)
```bash
cat file1.csv file2.csv > combined.csv

# Skip header from second file
cat file1.csv <(tail -n +2 file2.csv) > combined.csv
```

### Horizontal Merge (add columns)
```bash
paste -d',' file1.csv file2.csv
```

### Join on Common Column
```bash
# Files must be sorted on join column
join -t',' -1 1 -2 1 <(sort -t',' -k1 file1.csv) <(sort -t',' -k1 file2.csv)
```

### Compare Files
```bash
# Lines only in file1
comm -23 <(sort file1.csv) <(sort file2.csv)

# Lines only in file2
comm -13 <(sort file1.csv) <(sort file2.csv)

# Lines in both
comm -12 <(sort file1.csv) <(sort file2.csv)
```

---

## Combining Operations

### Filter, Transform, Sort
```bash
awk -F',' 'NR > 1 && $3 > 100 {print $1","$3}' file.csv | sort -t',' -k2 -rn
```

### Top N by Value
```bash
# Top 10 by column 3
(head -1 file.csv && tail -n +2 file.csv | sort -t',' -k3 -rn | head -10)
```

### Running Total
```bash
awk -F',' 'NR > 1 {sum += $3; print $0","sum}' file.csv
```

### Percentage of Total
```bash
awk -F',' 'NR > 1 {sum += $3; rows[NR] = $0; vals[NR] = $3}
    END {for (i=2; i<=NR; i++) printf "%s,%.2f%%\n", rows[i], vals[i]*100/sum}' file.csv
```

---

## Edge Cases

### Handle Quoted Fields (CSV with commas in values)
```bash
# Use a proper CSV parser for complex cases
python3 -c "import csv,sys; [print(','.join(row)) for row in csv.reader(sys.stdin)]" < file.csv

# Or use qsv/xsv
qsv select 1,3 file.csv
xsv select 1,3 file.csv
```

### Handle Empty Lines
```bash
awk -F',' 'NF > 0' file.csv
grep -v '^$' file.csv
```

### Handle Windows Line Endings
```bash
sed 's/\r$//' file.csv
tr -d '\r' < file.csv
dos2unix file.csv
```

### Very Large Files
```bash
# Process without loading into memory
awk -F',' '$3 > 100' large.csv > filtered.csv

# Sample first N rows
head -1 large.csv; tail -n +2 large.csv | shuf | head -1000
```

---

## Miller (mlr) - Modern Alternative

```bash
# Extract columns
mlr --csv cut -f name,email file.csv

# Filter rows
mlr --csv filter '$age > 30' file.csv

# Sort
mlr --csv sort -f name file.csv

# Aggregate
mlr --csv stats1 -a sum,mean -f amount -g category file.csv

# Convert formats
mlr --icsv --ojson cat file.csv
mlr --ijson --ocsv cat file.json
```

---

## Common Patterns

### CSV to JSON (simple)
```bash
awk -F',' 'NR==1 {for(i=1;i<=NF;i++) h[i]=$i; next}
    {printf "{"; for(i=1;i<=NF;i++) printf "\"%s\":\"%s\"%s", h[i], $i, (i<NF?",":""); print "}"}' file.csv
```

### Report Generation
```bash
echo "=== Summary Report ==="
echo "Total rows: $(tail -n +2 file.csv | wc -l)"
echo "Sum of column 3: $(awk -F',' 'NR>1 {s+=$3} END {print s}' file.csv)"
echo "Unique categories: $(cut -d',' -f1 file.csv | tail -n +2 | sort -u | wc -l)"
```

---

## datamash

```bash
# Column statistics
datamash -t',' mean 3 median 3 sstdev 3 < file.csv

# Group-by aggregation
datamash -t',' -H --group 1 sum 3 mean 3 count 3 < file.csv

# Transpose rows to columns
datamash transpose < file.tsv

# Cross-tabulation (pivot table)
datamash -t',' --group 1,2 count 3 < file.csv

# Frequency count
datamash -t',' --group 1 count 1 < file.csv | sort -t',' -k2 -rn

# Range, min, max
datamash -t',' min 3 max 3 range 3 < file.csv

# Percentiles
datamash -t',' perc:25 3 perc:50 3 perc:75 3 perc:95 3 < file.csv

# Round values
datamash -t',' round 3 < file.csv

# Check if sorted
datamash -t',' --group 1 check < file.csv
```

---

## qsv / xsv (Fast CSV Toolkit)

```bash
# Show headers with column numbers
qsv headers file.csv
xsv headers file.csv

# Column statistics (min, max, mean, stddev, nulls)
qsv stats file.csv | qsv table
xsv stats file.csv | xsv table

# Select specific columns
qsv select name,email file.csv
xsv select 1,3,5 file.csv

# Search for pattern in any column
qsv search "error" file.csv
xsv search "error" file.csv

# Sort by column
qsv sort -s amount -R file.csv     # Reverse sort by amount
xsv sort -s amount -R file.csv

# Join two CSVs on a column
qsv join id file1.csv id file2.csv
xsv join id file1.csv id file2.csv

# Frequency counts for a column
qsv frequency -s category file.csv | qsv table
xsv frequency -s category file.csv

# Sample random rows
qsv sample 100 file.csv

# Split large CSV into chunks
qsv split -s 10000 output_dir/ file.csv
xsv split -s 10000 output_dir/ file.csv

# Deduplicate rows
qsv dedup file.csv

# Count rows
qsv count file.csv
xsv count file.csv

# Slice rows (head/tail equivalent)
qsv slice -s 0 -l 100 file.csv      # First 100 rows
qsv slice -i 500 file.csv           # Skip first 500 rows
```

---

## q / textql (SQL on CSV)

```bash
# Run SQL directly on CSV files
q -H -d',' "SELECT name, SUM(amount) FROM file.csv GROUP BY name ORDER BY 2 DESC"

# Join two CSV files with SQL
q -H -d',' "SELECT a.name, b.total FROM file1.csv a JOIN file2.csv b ON a.id = b.id"

# textql (alternative)
textql -header -dlm=',' -sql "SELECT * FROM file WHERE amount > 100" file.csv
textql -header -dlm=',' -sql "SELECT category, COUNT(*) FROM file GROUP BY category" file.csv
```

---

## Numeric Utilities (num-utils)

```bash
# Average of numbers (one per line)
seq 1 100 | numaverage

# Sum of numbers
seq 1 100 | numsum

# Round to N decimal places
echo -e "3.14159\n2.71828" | numround 2

# Find range (max - min)
echo -e "10\n20\n30\n5\n15" | numrange

# Filter numbers matching condition
seq 1 100 | numgrep '/^[2-4]/'     # Numbers 2-49

# Pipeline: extract column, compute stats
cut -d',' -f3 file.csv | tail -n +2 | numaverage
cut -d',' -f3 file.csv | tail -n +2 | numsum
awk -F',' 'NR>1 {print $3}' file.csv | numround 2
```

---

## tsort (Topological Sort)

```bash
# Basic dependency resolution
echo -e "a b\nb c\nc d" | tsort
# Output: a b c d (valid dependency order)

# From dependency file
# deps.txt: each line is "prerequisite dependent"
tsort deps.txt

# Detect circular dependencies (exits with error)
echo -e "a b\nb a" | tsort
# tsort: -: input contains a loop

# Build order from Makefile-like deps
echo -e "compile link\nlink test\ntest deploy" | tsort
```

---

## Advanced Numeric Utilities

```bash
# Bounds (min and max)
echo -e "5\n2\n8\n1\n9" | numbound

# Intervals between consecutive values
echo -e "10\n15\n22\n30" | numinterval

# Normalize to 0-1 range
echo -e "5\n10\n15\n20" | numnormalize

# Apply formula to each number
seq 1 10 | numprocess '/^(\d+)$/$1*$1/e'

# Generate random numbers
numrandom 1..100 | head -10
```

---

## Pivot and Cross-Tabulation

```bash
# datamash cross-tabulation
datamash -t',' -s --group 1 crosstab 2,3 unique 4 < data.csv

# awk-based pivot: count by two dimensions
awk -F',' 'NR>1 {a[$1][$2]++} END{for(r in a) for(c in a[r]) print r, c, a[r][c]}' data.csv

# Miller reshape (wide to long)
mlr --icsv reshape --idr "^val_" --odr "metric" --ovr "value" data.csv

# Miller reshape (long to wide)
mlr --icsv reshape -o "metric" "value" data.csv
```

---

## Window Functions and Running Calculations

```bash
# Running (cumulative) sum
awk -F',' 'NR>1 {sum+=$3; print $0","sum}' data.csv

# Running average
awk -F',' 'NR>1 {sum+=$3; n++; print $0","sum/n}' data.csv

# Lag value (previous row)
awk -F',' 'NR>1 {if(prev!="") print $0","prev; prev=$3}' data.csv

# Moving average (window of 3)
awk -F',' 'NR>1 {a[NR]=$3; if(NR>=4) printf "%s,%.2f\n",$0,(a[NR]+a[NR-1]+a[NR-2])/3}' data.csv

# Cumulative distribution
awk -F',' 'NR>1 {sum+=$3; rows[NR]=$0; vals[NR]=sum} END{for(i=2;i<=NR;i++) printf "%s,%.4f\n",rows[i],vals[i]/sum}' data.csv
```

---

## Histogram and Bucketing

```bash
# Bucket values into ranges (buckets of 10)
awk -F',' 'NR>1 {b=int($3/10)*10; h[b]++} END{for(k in h) printf "%d-%d: %d\n",k,k+9,h[k]}' data.csv | sort -n

# datamash bin
datamash -t',' bin:10 3 < data.csv | sort | uniq -c

# Character histogram (frequency bar chart)
awk -F',' 'NR>1 {c[$1]++} END{for(k in c) {printf "%-20s ", k; for(i=0;i<c[k];i++) printf "#"; print ""}}' data.csv

# Percentile buckets
awk -F',' 'NR>1 {a[NR-1]=$3} END{n=NR-1; asort(a); printf "p25=%.1f p50=%.1f p75=%.1f p95=%.1f\n", a[int(n*0.25)], a[int(n*0.5)], a[int(n*0.75)], a[int(n*0.95)]}' data.csv
```

---

## Fixed-Width Parsing

```bash
# Extract character positions
cut -c1-10,20-30 file.txt

# awk substr for fixed-width fields
awk '{print substr($0,1,10), substr($0,20,10)}' file.txt

# Remove columns by character position
colrm 1 5 < file.txt                    # Remove first 5 chars

# Format fixed-width output
awk '{printf "%-20s %10s %8.2f\n", $1, $2, $3}' data.txt
```

---

## Date/Time Column Operations

```bash
# Convert date column to epoch
awk -F',' '{cmd="date -d \""$1"\" +%s"; cmd | getline ts; close(cmd); print ts","$2}' data.csv

# Extract date parts
awk -F'[,T-]' '{print $1"-"$2","$NF}' data.csv

# Miller date operations
mlr --icsv put '$epoch = strftime(strptime($date, "%Y-%m-%d"), "%s")' data.csv
mlr --icsv put '$day = strftime(strptime($date, "%Y-%m-%d"), "%A")' data.csv
```

---

## Combined Workflows

```bash
# API response → formatted table
curl -s API | jq -r '.[] | [.name, .status, .count] | @tsv' | column -t

# Log analysis to table
grep "ERROR" app.log | cut -d' ' -f5 | sort | uniq -c | sort -rn | column -t

# qsv pipeline: select, filter, sort, display
qsv select name,amount file.csv | qsv search "error" | qsv sort -s amount -R | qsv table

# CSV → JSON → filter
mlr --icsv --ojson cat data.csv | jq '.[] | select(.amount > 100)'

# Line counts per file type
fdfind -e py -e js -e ts | xargs wc -l | sort -rn | head
```

---

## Math and Unit Conversion

### bc (calculator)
```bash
echo "2^32" | bc                              # Exponentiation
echo "scale=4; 22/7" | bc                     # Division with precision
echo "ibase=16; FF" | bc                      # Hex to decimal
echo "obase=2; 255" | bc                      # Decimal to binary
```

### units (unit conversion)
```bash
units "5 miles" "km"                          # Miles to kilometers
units "100 celsius" "fahrenheit"              # Temperature conversion
units "1 gigabyte" "megabytes"                # Data size conversion
units "3 hours + 45 minutes" "seconds"        # Time conversion
```

---

## Installation

```bash
# cut, sort, column are pre-installed (coreutils, util-linux)

# awk (usually pre-installed, or install gawk for full features)
sudo apt install gawk

# Miller (modern CSV processor)
sudo apt install miller

# qsv (fast CSV toolkit)
# Download from https://github.com/jqnatividad/qsv
```
