---
description: "Text search recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-text-search with file paths and context included"
when-to-use: "Receives prepared arguments from task-text-search with file paths and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Text Search and Transformation Recipes

**Tools:** grep, rg (ripgrep), ag, ast-grep, sed, tr, awk, sd, head, tail, tac, rev, nl, wc, fmt, fold, expand, unexpand, split, csplit, pr, iconv, recode, strings

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Find lines with pattern | grep | `grep "pattern" file` |
| Find in all files | grep | `grep -rn "pattern" dir/` |
| Fast recursive search | rg | `rg "pattern" dir/` |
| Case insensitive | grep | `grep -i "pattern" file` |
| Invert (lines NOT matching) | grep | `grep -v "pattern" file` |
| Count matches | grep | `grep -c "pattern" file` |
| Only show match (not line) | grep | `grep -o "pattern" file` |
| Show context | grep | `grep -B2 -A2 "pattern" file` |
| Replace in file | sed | `sed -i 's/old/new/g' file` |
| Delete lines matching | sed | `sed '/pattern/d' file` |
| Structural code search | ast-grep | `ast-grep -p 'console.log($$$)' dir/` |
| Fast search (legacy) | ag | `ag "pattern" dir/` |
| Modern sed alternative | sd | `sd 'old' 'new' file` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Quick search | grep | Universal, simple, no install needed |
| Large codebase search | rg | Much faster, respects .gitignore |
| Search + replace in files | sed / sd | sed is universal; sd has simpler syntax |
| Complex regex | grep -P or rg -P | Perl-compatible regex |
| Extract structured parts | grep -o or rg -o | Just the match, not the line |
| Structural code patterns | ast-grep | Understands AST, not just text; refactoring |
| Remove console.log/debug | ast-grep | Matches code structure, not string patterns |
| Legacy projects using ag | ag | Slower than rg but widely deployed |

Note: On Ubuntu, ripgrep is `rg` (package: ripgrep).

---

## Searching for Text

### Basic Search
```bash
grep "error" file.log
grep "error" *.log                    # Multiple files
grep -r "TODO" src/                   # Recursive
grep -rn "TODO" src/                  # With line numbers
```

### Case Insensitive
```bash
grep -i "error" file.log
grep -i "ERROR\|Warning" file.log     # Multiple patterns
rg -i "error" file.log
```

### Whole Word Only
```bash
grep -w "error" file.log              # Won't match "errors"
grep -w "in" file.txt                 # Won't match "string"
```

### With Context (surrounding lines)
```bash
grep -B2 "error" file.log             # 2 lines before
grep -A2 "error" file.log             # 2 lines after
grep -C2 "error" file.log             # 2 lines both sides
grep -B5 -A10 "error" file.log        # Asymmetric
```

### Fast Search with ripgrep
```bash
rg "pattern" dir/                     # Recursive, fast
rg "pattern" --type py                # Only Python files
rg "pattern" -g "*.js"                # Glob pattern
rg "pattern" --hidden                 # Include hidden files
rg "pattern" -l                       # List files only
rg "pattern" -c                       # Count per file
rg "pattern" --no-ignore              # Include gitignored files
```

### Fixed Strings (no regex)
```bash
grep -F "exact[string]" file          # Literal match
rg -F "literal.string" file           # No regex interpretation
fgrep "pattern" file                  # Alias for grep -F
```

---

## Regex Patterns

### Common Patterns
```bash
grep "^Error" file                    # Lines starting with Error
grep "error$" file                    # Lines ending with error
grep "^$" file                        # Empty lines
grep "." file                         # Non-empty lines
grep "[0-9]" file                     # Lines with digits
grep "[0-9]\{3\}" file                # 3+ consecutive digits
grep "^[^#]" file                     # Lines not starting with #
```

### Extended Regex (-E or egrep)
```bash
grep -E "error|warning" file          # OR
grep -E "^(GET|POST)" file            # Alternatives
grep -E "[0-9]{1,3}\.[0-9]{1,3}" file # IP-like patterns
grep -E "https?://" file              # http or https
grep -E "v[0-9]+\.[0-9]+\.[0-9]+" file # Version numbers
```

### Perl Regex (-P, most powerful)
```bash
grep -oP '(?<=user=)[^&]+' file       # Lookbehind: extract after "user="
grep -oP '(?<=")[^"]+(?=")' file      # Content between quotes
grep -oP '\d{4}-\d{2}-\d{2}' file     # Date pattern
grep -oP '[\w.+-]+@[\w.-]+\.\w+' file # Email addresses
grep -oP '(?<=password=)[^\s]+' file  # Password values
```

### Ripgrep Regex (PCRE2)
```bash
rg '\b\w+@\w+\.\w+\b' file            # Email pattern
rg '(?i)error' file                   # Inline case-insensitive
rg '\d{1,3}(\.\d{1,3}){3}' file       # IP addresses
```

---

## Filtering Lines

### Exclude Lines
```bash
grep -v "debug" file.log              # Lines NOT containing debug
grep -v "^#" file                     # Non-comment lines
grep -v "^$" file                     # Non-empty lines
grep -v "^[[:space:]]*$" file         # Non-blank (spaces count)
```

### Multiple Conditions
```bash
grep "error" file | grep -v "ignore"  # AND NOT
grep -E "error|warning" file          # OR
grep "error" file | grep "user"       # AND (both patterns)
grep "error" file | grep -v "test" | grep -v "debug"  # Complex filter
```

### Filter by File Type (with rg)
```bash
rg "TODO" --type py                   # Python files
rg "import" --type js --type ts       # JS and TS files
rg "pattern" -g "!*.min.js"           # Exclude minified
rg "pattern" -g "!node_modules"       # Exclude directory
rg "pattern" -g "*.{js,ts,tsx}"       # Multiple extensions
```

### Line Number Ranges
```bash
sed -n '10,20p' file                  # Lines 10-20
sed -n '100p' file                    # Line 100 only
awk 'NR>=10 && NR<=20' file           # Lines 10-20
head -20 file | tail -10              # Lines 11-20
```

---

## Counting and Statistics

### Count Matches
```bash
grep -c "error" file.log              # Lines matching
grep -o "error" file | wc -l          # Total occurrences
rg -c "error" file.log                # Count (ripgrep)
rg --count-matches "error" file       # All occurrences
```

### List Files with Matches
```bash
grep -l "pattern" *.py                # Files containing
grep -L "pattern" *.py                # Files NOT containing
rg -l "pattern" dir/                  # Files (ripgrep)
```

### Count by File
```bash
grep -c "error" *.log                 # Count per file
rg -c "error" dir/                    # Count per file (recursive)
grep -rc "error" dir/                 # Recursive count per file
```

### Statistics Summary
```bash
# Count occurrences by pattern
grep -oE "ERROR|WARNING|INFO" file.log | sort | uniq -c

# Most common errors
grep -i "error" file.log | sort | uniq -c | sort -rn | head -10
```

---

## Extracting Content

### Only the Match
```bash
grep -o "error[^:]*" file             # Just the matching part
grep -oP '\d+\.\d+\.\d+' file         # Extract version numbers
grep -oP '(?<=email=)[^&\s]+' file    # Extract email values
grep -oE '[0-9]+' file                # All numbers
```

### First/Last Match
```bash
grep -m1 "pattern" file               # First match only
grep "pattern" file | tail -1         # Last match only
grep -m5 "pattern" file               # First 5 matches
```

### Specific Fields
```bash
# Extract URLs
grep -oP 'https?://[^\s<>"]+' file

# Extract IPs
grep -oP '\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b' file

# Extract quoted strings
grep -oP '"[^"]*"' file

# Extract function calls
grep -oP '\w+\(' file | tr -d '('
```

---

## Search and Replace

### Basic Replace with sed
```bash
sed 's/old/new/' file                 # First occurrence per line
sed 's/old/new/g' file                # All occurrences
sed -i 's/old/new/g' file             # Edit file in place
sed -i.bak 's/old/new/g' file         # In place with backup
```

### Replace Patterns
```bash
sed 's/[0-9]\+/NUM/g' file            # Numbers to "NUM"
sed 's/  */ /g' file                  # Collapse multiple spaces
sed 's/^/prefix: /' file              # Add prefix to lines
sed 's/$/ suffix/' file               # Add suffix to lines
sed 's/^[[:space:]]*//' file          # Trim leading whitespace
sed 's/[[:space:]]*$//' file          # Trim trailing whitespace
```

### Capture Groups
```bash
# Swap order
sed 's/\(first\) \(second\)/\2 \1/' file
sed -E 's/(first) (second)/\2 \1/' file    # Extended regex

# Wrap in tags
sed 's/\(.*\)/<p>\1<\/p>/' file

# Extract and transform
sed -E 's/.*name="([^"]+)".*/\1/' file
```

### Delete Lines
```bash
sed '/pattern/d' file                 # Delete matching lines
sed '/^#/d' file                      # Delete comments
sed '/^$/d' file                      # Delete empty lines
sed '1d' file                         # Delete first line (header)
sed '$d' file                         # Delete last line
sed '1,5d' file                       # Delete lines 1-5
```

### Replace in Multiple Files
```bash
sed -i 's/old/new/g' *.py             # All .py files
find . -name "*.py" -exec sed -i 's/old/new/g' {} +
rg -l "old" --type py | xargs sed -i 's/old/new/g'
```

### sd (modern sed alternative)
```bash
sd 'old' 'new' file                   # Simple replace
sd -F 'literal[text]' 'new' file      # Fixed string
sd '(\w+)@(\w+)' '$2@$1' file         # Capture groups
```

### Conditional Replace
```bash
sed '/error/s/foo/bar/g' file         # Replace only on lines matching "error"
sed '/^#/!s/old/new/g' file           # Replace except on comments
awk '/pattern/ {gsub(/old/,"new")} {print}' file
```

---

## Common Patterns for Logs

### Extract Timestamps
```bash
grep -oP '\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}' file.log
grep -oP '\[\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}\]' file.log
```

### Extract IP Addresses
```bash
grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' file.log
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' file.log
```

### Filter by Date
```bash
grep "2024-01-15" file.log
grep -E "2024-01-(15|16|17)" file.log
```

### Filter by Log Level
```bash
grep -E "ERROR|FATAL" file.log
grep -vE "DEBUG|TRACE" file.log       # Exclude verbose
```

### Error Summary
```bash
grep -i "error" file.log | sort | uniq -c | sort -rn
grep -oP '(?<=ERROR: )[^:]+' file.log | sort | uniq -c | sort -rn
```

### Last N Minutes of Logs (if timestamped)
```bash
# Assuming ISO timestamps at start of line
awk -v cutoff="$(date -d '5 minutes ago' '+%Y-%m-%d %H:%M')" '$0 >= cutoff' file.log
```

---

## Character Transformations

### tr (translate characters)
```bash
tr 'a-z' 'A-Z' < file                 # Lowercase to uppercase
tr 'A-Z' 'a-z' < file                 # Uppercase to lowercase
tr -d '\r' < file                     # Remove carriage returns
tr -s ' ' < file                      # Squeeze multiple spaces
tr ',' '\t' < file                    # CSV to TSV
tr -d '[:punct:]' < file              # Remove punctuation
```

### Case Conversion
```bash
awk '{print toupper($0)}' file        # Uppercase
awk '{print tolower($0)}' file        # Lowercase
sed 's/.*/\U&/' file                  # Uppercase (GNU sed)
sed 's/.*/\L&/' file                  # Lowercase (GNU sed)
```

---

## Advanced Patterns

### Multiline Search
```bash
# With pcregrep
pcregrep -M 'start.*\n.*end' file

# With perl
perl -0777 -ne 'print $& while /start.*?end/gs' file

# Stack traces (lines starting with whitespace after error)
awk '/error/{p=1} p{print} /^[^ ]/ && !/error/{p=0}' file.log
```

### Context-Aware Extraction
```bash
# Extract function bodies
awk '/^def /{p=1} p; /^[^ ]/ && p && !/^def /{p=0}' file.py

# Extract between markers
sed -n '/START/,/END/p' file
awk '/START/,/END/' file
```

### File Sections
```bash
# Split by pattern
csplit file '/^Chapter/' '{*}'        # Split on "Chapter" lines

# Extract section
sed -n '/\[section\]/,/\[/p' file.ini # INI file section
```

---

## Performance Tips

- Use `rg` (ripgrep) for large codebases - much faster than grep
- Use `-F` for fixed strings (no regex) - faster
- Use `--include`/`--exclude` to limit file scope
- Pipe to `head` if you only need first few matches

```bash
grep -F "exact string" file           # Fixed string, no regex
rg -F "exact string" dir/             # Fast fixed string search
grep "pattern" file | head -20        # Stop after 20 matches
rg -m 20 "pattern" dir/               # First 20 matches (fast)
```

---

## Edge Cases

### Binary Files
```bash
grep -a "pattern" binary              # Treat as text
strings binary | grep "pattern"       # Extract strings first
grep -I "pattern" *                   # Skip binary files
```

### Files with Special Names
```bash
grep "pattern" -- "-filename"         # Handle leading dash
grep "pattern" "./-filename"          # Alternative
find . -name "*.log" -print0 | xargs -0 grep "pattern"  # Handle spaces
```

### Very Large Files
```bash
# Stream processing (no memory load)
rg "pattern" large.log > matches.txt

# Sample from large file
head -100000 large.log | grep "pattern"
```

---

## sed Transformation Patterns

### Address Ranges
```bash
sed '10,20s/old/new/g' file          # Replace only on lines 10-20
sed '1,/^END/s/foo/bar/g' file       # From line 1 to first "END" line
sed '/start/,/end/d' file            # Delete between patterns (inclusive)
sed '/start/,/end/{/start/!{/end/!d}}' file  # Delete between but keep markers
```

### Insert and Append
```bash
sed '3i\new line here' file          # Insert before line 3
sed '3a\new line here' file          # Append after line 3
sed '/pattern/i\inserted line' file  # Insert before pattern match
sed '/pattern/a\appended line' file  # Append after pattern match
```

### Change and Translate
```bash
sed '/pattern/c\replacement line' file  # Replace entire matching line
sed 'y/abc/ABC/' file                # Translate characters (like tr)
```

### Hold Space (multi-line)
```bash
# Join every two lines
sed 'N;s/\n/ /' file

# Print line before match
sed -n '/pattern/{x;p;x;p}; h' file

# Accumulate lines between patterns
sed -n '/START/,/END/H; /END/{x;s/\n/ /g;p}' file
```

### Multiple Commands
```bash
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file    # Multiple substitutions
sed -e '/^#/d' -e '/^$/d' file                 # Delete comments and empty lines
sed '1{h;d}; G' file                           # Move first line to end
```

---

## awk Operations

### Field Processing
```bash
awk '{print $1}' file                 # First field (whitespace delimited)
awk '{print $1, $3}' file            # First and third fields
awk -F',' '{print $2}' file          # CSV: second column
awk -F'\t' '{print $1, $NF}' file   # TSV: first and last fields
awk '{print NR, $0}' file           # Prepend line numbers
awk '{print NF}' file               # Number of fields per line
```

### Conditional Processing
```bash
awk '$3 > 100' file                  # Lines where field 3 > 100
awk '$1 == "error"' file             # Lines where field 1 is "error"
awk 'length > 80' file              # Lines longer than 80 chars
awk 'NR % 2 == 0' file              # Even-numbered lines
awk '!seen[$0]++' file              # Remove duplicate lines (preserves order)
```

### BEGIN and END Blocks
```bash
awk 'BEGIN{sum=0} {sum+=$1} END{print sum}' file             # Sum column
awk 'BEGIN{FS=","} {sum+=$2} END{print "avg:", sum/NR}' file  # CSV average
awk 'BEGIN{print "Name\tScore"} {print $1"\t"$2}' file       # Add header
```

### Associative Arrays
```bash
awk '{count[$1]++} END{for(k in count) print k, count[k]}' file  # Count by field
awk -F',' '{sum[$1]+=$2} END{for(k in sum) print k, sum[k]}' file  # Sum by group
awk '{lines[$1]=lines[$1] ORS $0} END{for(k in lines) print lines[k]}' file  # Group lines
```

### printf Formatting
```bash
awk '{printf "%-20s %10.2f\n", $1, $2}' file   # Formatted columns
awk '{printf "%04d %s\n", NR, $0}' file         # Zero-padded line numbers
awk -F',' '{printf "%s: $%.2f\n", $1, $2}' file # Currency format
```

### In-place Editing (GNU awk)
```bash
awk -i inplace '{gsub(/old/, "new")} 1' file    # Replace in-place
awk -i inplace 'NR!=3' file                      # Delete line 3 in-place
```

---

## tr Character Operations

### Translation
```bash
tr 'a-z' 'A-Z' < file                # Lowercase to uppercase
tr 'A-Z' 'a-z' < file                # Uppercase to lowercase
tr ',' '\t' < file                    # CSV to TSV
tr '\t' ',' < file                    # TSV to CSV
tr ' ' '_' < file                    # Spaces to underscores
tr '[:lower:]' '[:upper:]' < file    # POSIX class uppercase
```

### Squeeze Repeats (-s)
```bash
tr -s ' ' < file                     # Collapse multiple spaces to one
tr -s '\n' < file                    # Collapse blank lines
tr -s '[:blank:]' '\n' < file        # One word per line
```

### Delete Characters (-d)
```bash
tr -d '\r' < file                    # Remove carriage returns (DOS to Unix)
tr -d '[:digit:]' < file             # Remove all digits
tr -d '[:punct:]' < file             # Remove all punctuation
tr -d '[:space:]' < file             # Remove all whitespace
tr -d '\n' < file                    # Join all lines (remove newlines)
```

### Complement (-c)
```bash
tr -cd '[:print:]' < file            # Keep only printable characters
tr -cd '[:alnum:]\n' < file          # Keep only alphanumeric + newlines
tr -cd '0-9\n' < file                # Keep only digits + newlines
tr -cs '[:alpha:]' '\n' < file       # Extract words (one per line)
```

---

## Structural Code Search and Refactoring (ast-grep)

### Pattern Matching
```bash
# Find all console.log calls
ast-grep -p 'console.log($$$)' -l js

# Find all function definitions in Python
ast-grep -p 'def $FUNC($$$):' -l py

# Find try/catch blocks in JavaScript
ast-grep -p 'try { $$$ } catch($ERR) { $$$ }' -l js

# Find React useState hooks
ast-grep -p 'const [$STATE, $SETTER] = useState($$$)' -l tsx

# Find unused imports pattern (no usage after import)
ast-grep -p 'import $NAME from "$MOD"' -l js

# Find all async functions
ast-grep -p 'async function $NAME($$$) { $$$ }' -l js

# Find if statements without else
ast-grep -p 'if ($COND) { $$$ }' -l js
```

### Rewrite Rules
```bash
# Remove console.log calls
ast-grep -p 'console.log($$$)' -r '' --lang js

# Convert var to const
ast-grep -p 'var $NAME = $VAL' -r 'const $NAME = $VAL' --lang js

# Convert string concatenation to template literal
ast-grep -p '"" + $EXPR' -r '`${$EXPR}`' --lang js
```

### YAML Rule Files
```bash
# Scan with rule directory
ast-grep scan --rule rules/

# Export results as JSON
ast-grep scan --json | jq '.[] | {file: .file, line: .range.start.line, text: .text}'

# Interactive mode
ast-grep scan --interactive
```

### vs Regex Comparison
```bash
# Regex: matches string "console.log" anywhere (even in comments/strings)
rg 'console\.log\(' --type js

# ast-grep: only matches actual console.log function calls (AST-aware)
ast-grep -p 'console.log($$$)' -l js
# ast-grep understands code structure - won't match comments or string literals
```

---

## Fast Legacy Text Search (ag / The Silver Searcher)

```bash
# Basic search (respects .gitignore)
ag "pattern" dir/

# Search only Python files
ag "pattern" --py dir/

# List files with matches
ag -l "TODO" dir/

# Ignore specific directories
ag --ignore node_modules "import" dir/

# Count matches per file
ag -c "error" dir/

# Case insensitive with context
ag -i -C3 "pattern" dir/
```

---

## Encoding Conversion (iconv / recode)

```bash
# Convert encoding
iconv -f ISO-8859-1 -t UTF-8 file.txt > file_utf8.txt

# Convert in-place (via temp file)
iconv -f WINDOWS-1252 -t UTF-8 file.txt -o file_utf8.txt && mv file_utf8.txt file.txt

# List available encodings
iconv -l

# Detect file encoding
file -bi file.txt

# Batch convert all text files
find . -name "*.txt" -exec sh -c 'iconv -f latin1 -t utf-8 "$1" -o "$1.utf8" && mv "$1.utf8" "$1"' _ {} \;

# Transliterate (strip accents)
iconv -f UTF-8 -t ASCII//TRANSLIT file.txt
```

---

## Text Formatting (fmt / fold / expand)

```bash
# Reformat paragraph to 80 columns
fmt -w 80 paragraph.txt

# Fold long lines at word boundary
fold -w 80 -s file.txt

# Fold at exact width (may break words)
fold -w 80 file.txt

# Convert tabs to spaces (4-space tabs)
expand -t 4 file.py

# Convert spaces to tabs
unexpand -t 4 file.py

# Uniform spacing (single space between words)
fmt -u file.txt
```

---

## File Splitting (split / csplit)

```bash
# Split by line count
split -l 1000 bigfile.txt chunk_
# Creates chunk_aa, chunk_ab, etc.

# Split by byte size
split -b 100M largefile.bin part_

# Split with numeric suffixes
split -l 1000 -d bigfile.txt chunk_
# Creates chunk_00, chunk_01, etc.

# Split at pattern boundaries
csplit log.txt '/^====/' '{*}'
# Splits at every line starting with ====

# Split at pattern, keep N parts
csplit data.txt '/^---/' '{3}'

# Reassemble split files
cat chunk_* > reassembled.txt
```

---

## diff Recipes

```bash
# Unified diff (for patches)
diff -u old.txt new.txt > changes.patch

# Side-by-side comparison
diff -y --width=120 file1.txt file2.txt

# Recursive directory diff
diff -r dir1/ dir2/

# Report only whether files differ
diff -q file1.txt file2.txt

# Colorized diff
diff --color file1.txt file2.txt

# Ignore whitespace differences
diff -w file1.txt file2.txt

# Compare using process substitution
diff <(sort a.txt) <(sort b.txt)
diff <(curl -s URL1) <(curl -s URL2)

# Show only added/removed lines
diff --changed-group-format='%>' --unchanged-group-format='' old.txt new.txt
```

---

## Extended wc Patterns

```bash
# Word count
wc -w file.txt

# Character count
wc -m file.txt

# Byte count
wc -c file.txt

# Multiple files sorted by line count
wc -l *.py | sort -n

# Longest line length
wc -L file.txt

# Total words across files
cat *.txt | wc -w
```

---

## Unicode and Special Character Patterns

```bash
# Find emoji characters
grep -P '\p{Emoji}' file.txt

# Find CJK characters
rg '[\x{4E00}-\x{9FFF}]' file.txt

# Strip non-printable characters
tr -cd '[:print:]\n' < file.txt

# Find non-ASCII characters
grep -P '[^\x00-\x7F]' file.txt

# Show non-printable characters
cat -A file.txt                           # Shows tabs as ^I, line ends as $

# Remove BOM (byte order mark)
sed -i '1s/^\xEF\xBB\xBF//' file.txt
```

---

## Combined Workflows

```bash
# Search and replace across files
rg -l "old_api" --type py | xargs sed -i 's/old_api/new_api/g'

# Error frequency from logs
rg -oP '(?<=ERROR: )\w+' *.log | sort | uniq -c | sort -rn | head -20

# Find files then search contents
fdfind -e py | xargs rg "def main"

# Lint all shell scripts found
fdfind -e sh | xargs shellcheck

# Top URLs from access log
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -20

# ast-grep results to JSON analysis
ast-grep scan --json | jq '.[] | {file: .file, line: .range.start.line, text: .text}'

# Encoding detection and conversion pipeline
file -bi *.txt | grep -v utf-8 | cut -d: -f1 | while read f; do
  iconv -f "$(file -bi "$f" | grep -oP 'charset=\K\S+')" -t utf-8 "$f" -o "$f.utf8" && mv "$f.utf8" "$f"
done

# Extract, transform, count pattern
rg -oP '\b[A-Z][a-z]+Error\b' *.py | sort | uniq -c | sort -rn
```

---

## Installation

```bash
# grep, sed, awk, and tr are pre-installed

# ripgrep (faster alternative)
sudo apt install ripgrep

# sd (modern sed)
cargo install sd
# or download from https://github.com/chmln/sd

# pcregrep (multiline regex)
sudo apt install pcregrep
```
