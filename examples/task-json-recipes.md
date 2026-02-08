---
description: "JSON recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-json with file paths and context included"
when-to-use: "Receives prepared arguments from task-json with file paths and context included"
context: fork
model: inherit
allowed-tools: Read
---

# JSON Processing Recipes

**Tools:** jq, yq, xq, jo, jd, gron, dasel, fx, jless, xmlstarlet, jsonlint, yamllint, dyff

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Pretty print | `jq '.' file.json` |
| Compact output | `jq -c '.' file.json` |
| Extract field | `jq '.name' file.json` |
| Extract nested | `jq '.user.address.city' file.json` |
| Raw output (no quotes) | `jq -r '.name' file.json` |
| Array element | `jq '.[0]' file.json` |
| All array elements | `jq '.[]' file.json` |
| Filter array | `jq '.[] | select(.active == true)' file.json` |
| Map/transform | `jq '.items | map(.name)' file.json` |
| Count items | `jq '.items | length' file.json` |
| Sum values | `jq '.items | map(.price) | add' file.json` |
| To CSV | `jq -r '.[] | [.name, .value] | @csv' file.json` |
| Keys of object | `jq 'keys' file.json` |
| Type of value | `jq '.field | type' file.json` |
| Flatten nested | `jq '[.. | numbers]' file.json` |
| Query multi-format | `dasel -f config.yaml '.server.port'` |
| Interactive JSON | `fx data.json` |
| Browse JSON (TUI) | `jless data.json` |
| Diff YAML/JSON | `dyff between old.yaml new.yaml` |
| Process XML | `xmlstarlet sel -t -v '//name' file.xml` |

---

## When to Use What

| Tool | Best For | Limitation |
|------|----------|------------|
| jq | Standard JSON querying and transformation | JSON only, steep learning curve |
| dasel | Multi-format (JSON, YAML, TOML, XML) queries | Less powerful transforms than jq |
| gron | Making JSON greppable with standard tools | One-way (gron/ungron), no transforms |
| fx | Interactive exploration of JSON | Not scriptable for pipelines |
| jless | Terminal TUI for browsing large JSON | Read-only viewer |
| yq | YAML processing (jq-like syntax) | Slower than jq for pure JSON |
| xmlstarlet | XML querying and transformation | XML only, verbose XPath syntax |

---

## Extract Fields and Values from JSON/YAML/TOML

### Single Field
```bash
jq '.name' file.json
jq -r '.name' file.json          # Raw output, no quotes
jq -r '.name // "N/A"' file.json # Default if null
```

### Nested Fields
```bash
jq '.user.address.city' file.json
jq '.data.results[0].id' file.json
jq '.config.database.host' file.json
```

### Multiple Fields
```bash
jq '{name: .name, email: .email}' file.json
jq '.items[] | {id, name, price}' file.json   # Shorthand
jq '[.name, .email, .age]' file.json          # As array
```

### Safe Navigation (won't error on null)
```bash
jq '.data.items[]?' file.json
jq '.missing // "default"' file.json           # Alternative operator
jq '.a.b.c? // empty' file.json               # Suppress null entirely
```

### Recursive Descent (..)
```bash
jq '.. | .email? // empty' file.json          # Find all "email" fields at any depth
jq '[.. | strings | select(test("@"))]' file.json  # All strings containing @
jq '[.. | numbers]' file.json                  # All numeric values anywhere
jq '[.. | .name? // empty] | unique' file.json # All unique "name" values
```

### Keys and Types
```bash
jq 'keys' file.json                           # Top-level keys
jq '.items[0] | keys' file.json               # Keys of first array element
jq '.items[0] | to_entries[] | "\(.key): \(.value | type)"' file.json  # Key: type pairs
jq 'path(..)|[.[]|tostring]|join(".")' file.json  # All paths in document
```

---

## Filter and Search within JSON Arrays

### By Value
```bash
jq '.[] | select(.status == "active")' file.json
jq '.[] | select(.price > 100)' file.json
jq '.[] | select(.price >= 10 and .price <= 50)' file.json
jq '.[] | select(.age > 18 and .country == "US")' file.json
```

### By String Content
```bash
jq '.[] | select(.name | contains("test"))' file.json
jq '.[] | select(.name | startswith("prefix"))' file.json
jq '.[] | select(.name | endswith("suffix"))' file.json
jq '.[] | select(.email | test("@gmail\\.com$"))' file.json   # Regex
jq '.[] | select(.email | test("pattern"; "i"))' file.json    # Case-insensitive regex
```

### Null Checks
```bash
jq '.[] | select(.email != null)' file.json
jq '.[] | select(.email)' file.json           # Truthy check
jq '.[] | select(.tags | length > 0)' file.json  # Non-empty array
```

### By Type
```bash
jq '.[] | select(type == "object")' file.json
jq '.[] | select(.value | type == "number")' file.json
jq '[.[] | select(.id | type == "string")]' file.json
jq 'to_entries[] | select(.value | type == "array")' file.json  # Keys with array values
```

### Negation
```bash
jq '.[] | select(.status != "deleted")' file.json
jq '.[] | select(.name | contains("test") | not)' file.json
jq '[.[] | select(.deprecated | not)]' file.json
```

### Combined Filter + Transform
```bash
jq '.items | map(select(.active) | {name, price})' file.json
jq '[.users[] | select(.role == "admin") | .email]' file.json
```

---

## Transforming Data

### Reshape Structure
```bash
# Flatten nested data
jq '.users[] | {name, email: .contact.email, city: .address.city}' file.json

# Create summary object
jq '{total: (.items | length), names: [.items[].name]}' file.json

# Rename keys
jq '.items[] | {user_name: .name, user_email: .email}' file.json
```

### Array Operations
```bash
jq '.items | sort_by(.name)' file.json
jq '.items | sort_by(.date) | reverse' file.json
jq '.items | unique_by(.id)' file.json
jq '.items | group_by(.category)' file.json
jq '.items[:10]' file.json                    # First 10
jq '.items[-5:]' file.json                    # Last 5
jq '.items | flatten' file.json               # Flatten nested arrays
jq '.items | flatten(1)' file.json            # Flatten one level
jq '[.items[] | .tags[]] | unique | sort' file.json  # All unique tags
```

### String Interpolation
```bash
jq -r '.items[] | "\(.name): $\(.price)"' file.json
jq -r '.users[] | "\(.first) \(.last) <\(.email)>"' file.json
```

### to_entries / from_entries / with_entries
```bash
# Object to key-value pairs
jq 'to_entries' file.json                     # [{key: "a", value: 1}, ...]

# Key-value pairs back to object
jq 'to_entries | from_entries' file.json

# Transform keys and values
jq 'with_entries(.key |= "prefix_" + .)' file.json                # Prefix all keys
jq 'with_entries(.value |= tostring)' file.json                   # Stringify all values
jq 'with_entries(select(.value != null))' file.json               # Remove null values
jq 'with_entries(select(.key | test("^(name|email|id)$")))' file.json  # Keep only specific keys
```

### map_values
```bash
jq 'map_values(. + 1)' file.json             # Increment all values in object
jq 'map_values(if type == "string" then ascii_downcase else . end)' file.json
```

---

## Aggregation

### Sum, Average, Min, Max
```bash
jq '.items | map(.price) | add' file.json                    # Sum
jq '.items | map(.score) | add / length' file.json           # Average
jq '.items | min_by(.price)' file.json                       # Min object
jq '.items | max_by(.score)' file.json                       # Max object
jq '[.items[].price] | min' file.json                        # Min value
jq '[.items[].price] | max' file.json                        # Max value
```

### Group and Aggregate
```bash
jq 'group_by(.category) | map({
  category: .[0].category,
  count: length,
  total: map(.price) | add
})' file.json
```

### Count Occurrences
```bash
jq '[.items[].status] | group_by(.) | map({(.[0]): length}) | add' file.json
jq '[.logs[].level] | group_by(.) | map({key: .[0], value: length}) | from_entries' file.json
```

### Reduce
```bash
jq 'reduce .items[] as $x (0; . + $x.price)' file.json           # Sum prices
jq 'reduce .items[] as $x ({}; . + {($x.id|tostring): $x.name})' file.json  # Build lookup
jq 'reduce .events[] as $e ({}; .[$e.type] += 1)' file.json      # Count by type
```

---

## Object Construction and Merging

### Build Objects
```bash
jq -n '{name: "test", items: [1,2,3]}'       # From scratch
jq -n --arg name "$NAME" '{name: $name}'      # From shell variable
jq -n --argjson count 5 '{count: $count}'     # Numeric from shell
```

### Computed Keys
```bash
jq '.[] | {(.name): .value}' file.json        # Dynamic key from field
jq '[.[] | {(.id|tostring): .}] | add' file.json  # Build lookup by id
```

### Merge Objects
```bash
jq -s '.[0] * .[1]' base.json override.json   # Deep merge (later wins)
jq '. + {newfield: "value"}' file.json         # Add field
jq '. * {nested: {deep: "value"}}' file.json   # Deep merge single field
jq '[.items[]] | map({id, name}) | add' file.json  # Won't work - use below
```

### Concatenate Arrays
```bash
jq -s 'add' file1.json file2.json             # Merge arrays
jq -s '.[0].items + .[1].items' a.json b.json # Merge specific arrays
jq -s 'map(.items) | add | unique_by(.id)' a.json b.json  # Merge + deduplicate
```

### Slurp Mode
```bash
jq -s '.' file1.json file2.json               # Read all as array
jq -s 'map(select(.valid))' file1.json file2.json  # Filter across files
```

---

## Format Conversion

### JSON to CSV
```bash
# Simple
jq -r '.[] | [.name, .email, .age] | @csv' file.json

# With headers
jq -r '["name","email","age"], (.[] | [.name, .email, .age]) | @csv' file.json

# Dynamic headers from keys
jq -r '(.[0] | keys_unsorted) as $keys | $keys, (.[] | [.[$keys[]]]) | @csv' file.json
```

### JSON to TSV
```bash
jq -r '.[] | [.name, .email] | @tsv' file.json
jq -r '["name","email"], (.[] | [.name, .email]) | @tsv' file.json
```

### CSV to JSON
```bash
# Using jq -Rs (raw slurp)
jq -Rs 'split("\n") | .[1:] | map(select(length > 0) | split(",") | {name: .[0], email: .[1], age: .[2]})' data.csv

# Using miller (if available)
mlr --icsv --ojson cat data.csv
```

### JSON to YAML
```bash
yq -P '.' file.json                           # JSON to YAML (yq)
yq eval '.' file.json -o yaml                 # Explicit output format
```

### YAML to JSON
```bash
yq -o=json '.' file.yaml
yq eval '.' file.yaml -o json
```

### JSON Flattening
```bash
# Flatten nested to dot-notation paths
gron file.json                                 # json.users[0].name = "Alice";
gron file.json | grep email                   # Find all email paths
gron file.json | grep email | gron -u         # Extract and re-assemble

# Manual flattening with jq
jq '[path(..)|map(tostring)|join(".")] | unique' file.json
```

---

## String Functions

### Case and Trim
```bash
jq '.name | ascii_downcase' file.json
jq '.name | ascii_upcase' file.json
jq '.name | ltrimstr("prefix_")' file.json
jq '.name | rtrimstr("_suffix")' file.json
jq '.items[] | .name | gsub("^\\s+|\\s+$"; "")' file.json  # Trim whitespace
```

### Split and Join
```bash
jq '.path | split("/")' file.json             # Split string to array
jq '.tags | join(", ")' file.json             # Join array to string
jq '.csv_line | split(",") | {name: .[0], value: .[1]}' file.json
```

### Regex (test / match / capture)
```bash
jq '.[] | select(.email | test("@gmail\\.com$"))' file.json
jq '.text | match("([0-9]+)-([a-z]+)") | .captures[].string' file.json
jq '.text | capture("(?<num>[0-9]+)-(?<word>[a-z]+)")' file.json   # Named captures
jq '.items[] | .name | gsub("[^a-zA-Z0-9]"; "_")' file.json       # Replace non-alphanumeric
jq '.text | scan("[0-9]+")' file.json          # Find all matches (array)
```

### Format Strings (@base64, @uri, @html, @csv, @tsv, @sh, @json)
```bash
jq -r '.secret | @base64' file.json           # Base64 encode
jq -r '.encoded | @base64d' file.json         # Base64 decode
jq -r '.query | @uri' file.json               # URL-encode
jq -r '.text | @html' file.json               # HTML-escape
jq -r '.args | @sh' file.json                 # Shell-escape (for eval)
jq -r '.data | @json' file.json               # Re-encode as JSON string
```

---

## Streaming and Large Files

### JSONL (Newline-Delimited JSON)
```bash
# Process each line
jq -c 'select(.level == "error")' file.jsonl

# Aggregate across lines
jq -s 'group_by(.user) | map({user: .[0].user, count: length})' file.jsonl

# Count by field
jq -s '[.[].status] | group_by(.) | map({(.[0]): length}) | add' file.jsonl

# First N matching lines (memory efficient)
jq -c 'select(.level == "error")' file.jsonl | head -20
```

### Streaming Mode (--stream)
```bash
# Stream large file without loading into memory
jq --stream 'select(.[0][-1] == "email") | .[1]' huge.json

# Truncate stream to rebuild objects
jq -cn --stream 'fromstream(1|truncate_stream(inputs))' huge.json

# Count elements without loading full array
jq -cn --stream '[., inputs] | map(select(length == 1 and .[0] == 0)) | length - 1' huge.json
```

### Passing External Data
```bash
jq --arg name "$NAME" '.[] | select(.name == $name)' file.json        # String arg
jq --argjson id 42 '.[] | select(.id == $id)' file.json               # JSON arg
jq --slurpfile lookup ids.json '.[] | select(.id == ($lookup[][]))' file.json  # From file
jq --rawfile template tmpl.txt '{template: $template}' file.json      # Raw string from file
jq --jsonargs -n '$ARGS.positional' -- '{"a":1}' '{"b":2}'           # Args as JSON
```

---

## Error Handling

### Safe Access
```bash
jq '.items[]?' file.json                       # ? suppresses errors on non-iterable
jq '.data.items // []' file.json              # Default empty array if null
jq 'if .data then .data else empty end' file.json
```

### try-catch
```bash
jq '.items[] | try .name catch "unknown"' file.json
jq 'try (.value | tonumber) catch null' file.json
jq '[.[] | try {name, age: (.age | tonumber)} catch empty]' file.json  # Skip bad records
```

### Debug Tracing
```bash
jq '.items | debug | map(.name)' file.json    # Print intermediate to stderr
jq '.items[] | debug("processing") | .name' file.json
```

### Conditional Logic
```bash
jq 'if .error then .error.message else .data end' file.json
jq '.items[] | if .price > 100 then "expensive" elif .price > 50 then "moderate" else "cheap" end' file.json
jq '.[] | .value // 0 | if . > 0 then "positive" else "non-positive" end' file.json
```

---

## Modifying JSON

### Add/Update Field
```bash
jq '. + {newfield: "value"}' file.json
jq '.name = "newname"' file.json
jq '.items[0].status = "updated"' file.json
jq '.items |= map(. + {processed: true})' file.json    # Add field to all items
jq '.timestamp = now' file.json                         # Add current timestamp
```

### Delete Field
```bash
jq 'del(.unwanted)' file.json
jq 'del(.items[] | select(.obsolete))' file.json
jq 'del(.items[].internal_id)' file.json               # Remove field from all items
jq 'walk(if type == "object" then del(.metadata) else . end)' file.json  # Remove everywhere
```

### Add to Array
```bash
jq '.items += [{"name": "new"}]' file.json
jq '.items |= . + [{"name": "appended"}]' file.json
jq '.items |= [{"name": "prepended"}] + .' file.json   # Prepend
```

### In-place Editing (write back to file)
```bash
jq '.status = "updated"' file.json > tmp.json && mv tmp.json file.json
# Or with sponge (moreutils):
jq '.status = "updated"' file.json | sponge file.json
```

### Walk (recursive transform)
```bash
jq 'walk(if type == "string" then ascii_downcase else . end)' file.json
jq 'walk(if type == "object" then with_entries(select(.value != null)) else . end)' file.json
```

---

## Common Patterns

### API Response Processing
```bash
# Extract data, handle missing
jq '.data // []' response.json
jq 'if .error then .error.message else .data end' response.json

# Unwrap paginated responses
jq -s 'map(.data) | add' page1.json page2.json page3.json
```

### Create JSON from Scratch
```bash
jq -n '{name: "test", items: [1,2,3]}'
jq -n --arg name "$NAME" --arg email "$EMAIL" '{name: $name, email: $email}'
jq -n '[range(10)] | map({id: ., name: "item-\(.)"})'  # Generate test data
```

### Diff Two JSON Files
```bash
jd a.json b.json                               # Semantic diff
diff <(jq -S '.' a.json) <(jq -S '.' b.json)  # Text diff of sorted JSON
jq -S '.' a.json > /tmp/a_sorted.json && jq -S '.' b.json > /tmp/b_sorted.json && diff /tmp/a_sorted.json /tmp/b_sorted.json
```

### Extract Unique Values
```bash
jq '[.items[].category] | unique | sort' file.json
jq '[.items[].tags[]] | unique' file.json      # Flatten then unique
```

### Count Occurrences by Field
```bash
jq '[.items[].status] | group_by(.) | map({(.[0]): length}) | add' file.json
```

### Transform API Response to Table
```bash
jq -r '["ID","Name","Status"], (.items[] | [.id, .name, .status]) | @tsv' file.json
```

### Merge Multiple Files
```bash
jq -s 'reduce .[] as $item ({}; . * $item)' conf1.json conf2.json conf3.json
```

### Select by Path Existence
```bash
jq '.[] | select(has("email"))' file.json      # Has key
jq '.[] | select(.address | has("city"))' file.json  # Has nested key
jq '.[] | select(.tags | any(. == "important"))' file.json  # Array contains value
jq '.[] | select(.roles | index("admin"))' file.json  # Array contains value (alt)
jq '.[] | select(any(.[]; . == "target"))' file.json   # Any value matches
```

---

## gron (Make JSON Greppable)

```bash
# Flatten JSON to assignment statements
gron file.json                                 # json.users[0].name = "Alice";

# Find specific paths
gron file.json | grep email                   # All email-related paths
gron file.json | grep 'users\[' | grep name  # User names specifically

# Ungron: convert back to JSON
gron file.json | grep email | gron -u         # Extract matching paths as JSON

# Pipeline: find and extract
gron api_response.json | grep 'error' | gron -u | jq '.'

# From URL (with curl)
curl -s https://api.example.com/data | gron | grep status
```

---

## YAML Processing (yq)

```bash
yq '.key' file.yaml                    # Extract
yq -i '.key = "value"' file.yaml       # Modify in place
yq -o=json '.' file.yaml               # YAML to JSON
yq -P '.' file.json                    # JSON to YAML
yq eval-all 'select(.kind == "Deployment")' *.yaml  # Filter multi-doc YAML
yq 'del(.metadata.annotations)' file.yaml  # Delete key
yq '.spec.replicas = 3' file.yaml      # Update value
yq -i '.env += [{"name": "NEW_VAR", "value": "val"}]' file.yaml  # Append to array
```

---

## XML Processing (xq)

```bash
xq '.' file.xml                        # XML to JSON
xq -r '.root.items[].name' file.xml    # Extract from XML via JSON path
cat file.xml | xq '.root.item[] | select(.price > "100")' # Filter XML
```

---

## JSON Creation with jo

```bash
jo name=test count=5 active=true       # {"name":"test","count":5,"active":true}
jo -a 1 2 3                           # [1,2,3]
jo -a $(seq 1 5)                      # [1,2,3,4,5]
jo name=test tags=$(jo -a web api)    # Nested: {"name":"test","tags":["web","api"]}
```

---

## Validation and Formatting

```bash
jq '.' file.json                       # Pretty print (validates too)
jq -e '.' file.json > /dev/null        # Validate only (exit code 0 = valid)
jq -S '.' file.json                    # Sort keys
jq -c '.' file.json                    # Compact single line
jsonlint file.json                     # Dedicated linter
python3 -m json.tool file.json         # Python pretty printer
```

---

## Performance Tips

- Use `jq -c` for compact output when piping between commands
- Use `--stream` for files too large to fit in memory
- Use `--slurpfile` instead of `-s` when only one file needs slurping
- Use `jq -e` to get proper exit codes (0=truthy, 1=falsy, 5=error)
- Pre-filter with `grep` for JSONL: `grep '"error"' file.jsonl | jq ...`
- Use `--rawfile` and `--arg` to pass data without shell escaping issues

### dasel (Multi-Format Query)

```bash
# Query JSON
dasel -f config.json '.database.host'          # Get value
dasel -f config.json -s '.database.port' 5432  # Set value (in-place)
dasel -f config.json 'select(.users.all().name)'  # Select from array

# Query YAML (same syntax)
dasel -f config.yaml '.services.web.ports.[0]'

# Query TOML
dasel -f config.toml '.database.host'

# Convert between formats
dasel -f config.yaml -w json                   # YAML → JSON
dasel -f config.json -w yaml                   # JSON → YAML
```

### fx (Interactive JSON Viewer)

```bash
fx data.json                                   # Interactive exploration
fx data.json '.users'                          # Pre-filtered view
cat data.json | fx '.users | length'           # Pipe mode with expression
fx data.json '.users[] | select(.active)'      # Filter in expression
```

### jless (Terminal JSON Viewer)

```bash
jless file.json                                # Open interactive viewer
jless file.yaml                                # Also supports YAML
# Navigation: j/k (up/down), h/l (collapse/expand), / (search), q (quit)
```

### dyff (YAML/JSON Diff)

```bash
dyff between old.yaml new.yaml                 # Semantic diff of YAML files
dyff between old.json new.json                 # Also works with JSON
dyff between --output human old.yaml new.yaml  # Human-readable output
dyff between --set-exit-code old.yaml new.yaml # Non-zero exit on diff (for CI)
```

### xmlstarlet (XML Processing)

```bash
# Select/extract values
xmlstarlet sel -t -v "//element" file.xml      # Extract element text
xmlstarlet sel -t -v "/root/item/@name" file.xml  # Extract attribute
xmlstarlet sel -t -m "//item" -v "@name" -o ": " -v "." -n file.xml  # Formatted output

# Edit XML
xmlstarlet ed -u "//element" -v "new_value" file.xml  # Update value
xmlstarlet ed -i "//item" -t attr -n "id" -v "1" file.xml  # Add attribute
xmlstarlet ed -d "//unwanted" file.xml         # Delete element

# Validate
xmlstarlet val file.xml                        # Validate well-formedness
xmlstarlet val -e -s schema.xsd file.xml       # Validate against schema
```

### Combined Workflows

```bash
# curl → jq pipeline
curl -s https://api.example.com/data | jq '.items[] | {name, status}'

# Find files → parse JSON from each
fdfind -e json | xargs -I{} jq -r '.version // empty' {}

# YAML config diff in CI
dyff between --set-exit-code base.yaml changed.yaml || echo "Config changed!"

# JSON → CSV extraction
jq -r '.users[] | [.name, .email, .role] | @csv' users.json > users.csv
```

---

## jc (Convert CLI Output to JSON)

### Basic Usage
```bash
# Convert any CLI output to parseable JSON
ps aux | jc --ps                              # Process list as JSON
df -h | jc --df                               # Disk usage as JSON
ls -l | jc --ls                               # Directory listing as JSON
ifconfig | jc --ifconfig                      # Network config as JSON
netstat -tuln | jc --netstat                  # Connections as JSON
mount | jc --mount                            # Mount points as JSON
```

### Pipe with jq
```bash
# Combine jc + jq for powerful CLI data queries
ps aux | jc --ps | jq '.[] | select(.cpu_percent > 10)'
df -h | jc --df | jq '.[] | select(.use_percent > 80)'
ls -la | jc --ls | jq '.[] | select(.size > 1000000) | .filename'
```

### Install
```bash
pip install jc
```

---

## Installation

```bash
# jq (essential)
sudo apt install jq

# yq (YAML processing)
pip install yq                        # Python yq
# or: snap install yq                 # Go yq (different syntax)

# gron (greppable JSON)
go install github.com/tomnomnom/gron@latest
# or download from https://github.com/tomnomnom/gron/releases

# xq (XML to JSON, part of yq)
pip install yq                        # Includes xq

# jo (JSON creation)
sudo apt install jo

# jd (JSON diff)
go install github.com/josephburnett/jd@latest

# fx (interactive JSON viewer)
npm install -g fx

# jless (terminal JSON viewer)
# https://github.com/PaulJuliworlds/jless/releases

# dasel (multi-format query)
# https://github.com/TomWright/dasel/releases

# jsonlint
npm install -g jsonlint
```
