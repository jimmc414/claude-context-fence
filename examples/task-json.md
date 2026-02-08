---
description: "JSON processing - can't read nested API response, JSON structure too complex, malformed JSON error, extracting fields, filtering arrays, transforming and reshaping JSON data (jq, gron, dasel, yq)"
when_to_use: "Use when: can't read or understand nested API response, JSON structure is too complex to navigate, getting malformed JSON errors, need to extract specific fields from deep JSON, converting between JSON and CSV/YAML, processing API responses, filtering JSON arrays, merging JSON files. Triggers: can't read json, malformed json, json too complex, nested json, api response confusing, json parse error, json field, json array, json filter, json to csv, api response, json transform, jq, parse json, query json, transform json, extract field, flatten json, yaml, toml, xml, validate json, pretty print json, merge json, json path, gron, dasel."
when-to-use: "Use when: can't read or understand nested API response, JSON structure is too complex to navigate, getting malformed JSON errors, need to extract specific fields from deep JSON, converting between JSON and CSV/YAML, processing API responses, filtering JSON arrays, merging JSON files. Triggers: can't read json, malformed json, json too complex, nested json, api response confusing, json parse error, json field, json array, json filter, json to csv, api response, json transform, jq, parse json, query json, transform json, extract field, flatten json, yaml, toml, xml, validate json, pretty print json, merge json, json path, gron, dasel."
argument-hint: "[describe what you want to do with JSON]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# JSON Task Router

You have access to the **full conversation context**. Your job is to analyze the user's JSON processing need and build a well-formed argument for the recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

JSON requests vary from simple extractions to complex reshaping. Analyze the conversation to identify all relevant context.

### 1. Goal
What JSON operation is needed?
- Extract field(s) or nested values
- Filter array elements by condition
- Transform / reshape structure
- Aggregate (count, sum, average, group)
- Convert format (JSON to CSV, TSV, YAML, or reverse)
- Merge / combine multiple files
- Modify (add, update, delete fields)
- Validate or pretty-print
- Make JSON greppable (gron)
- Stream-process large files

### 2. File Path(s)
Where is the JSON data?
- Explicit paths mentioned by user
- Files created/modified earlier in conversation
- Files referenced in error messages
- API response piped or saved
- stdin / JSONL stream

### 3. Structure Hints
How is the data organized?
- Nested path: `.data.users[].contact.email`
- Array at root `[{...}, {...}]` vs object with array `.items[]`
- Key naming patterns (camelCase, snake_case)
- Mixed types or inconsistent schema

### 4. Filter Conditions
What constraints apply?
- Value comparison: `.price > 100`, `.status == "active"`
- String matching: `contains("error")`, `test("regex")`
- Null handling: `!= null`, `// "default"`
- Type checks: `type == "string"`, `numbers`
- Boolean logic: `and`, `or`, `not`

### 5. Output Format
What should the result look like?
- Raw values (`-r`, one per line)
- CSV / TSV (`@csv`, `@tsv`)
- Formatted JSON (pretty or compact `-c`)
- Custom structure (new object shape)
- Count or aggregation result

### 6. Scope Limits
Any boundaries?
- First/last N items (slicing `[:10]`)
- Unique values only
- Sorted output
- Specific keys only

### 7. Error Context
What's gone wrong?
- Previous jq errors or unexpected output
- Null propagation issues
- Type mismatch errors
- Large file / memory concerns

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Extract, filter, transform | jq | Default for all JSON work |
| Make JSON greppable | gron | Flatten to grep-friendly paths |
| Interactive exploration | fx / jless | Terminal JSON viewer |
| Multi-format queries | dasel | JSON + YAML + TOML + XML |
| YAML processing | yq | jq syntax for YAML |
| XML to JSON | xq | jq wrapper for XML |
| JSON diffing | jd | Semantic JSON diff |
| JSON creation | jo | Build JSON from shell args |

---

## Argument Format

Construct a structured argument string:

```
<goal> from <file_path> - structure: <path hints>, filter: <conditions>, output: <format>
```

**Include only relevant components.** Not every task needs all fields.

### Examples

**Simple extraction:**
```
extract user emails from /tmp/api-response.json - nested in .data.users[].contact.email
```

**Filter and transform:**
```
filter active products from ./products.json - where .status == "active" and .price > 0, output as CSV with name and price columns
```

**Aggregation:**
```
count items per category from /home/user/orders.json - group by .category, include total price sum per group
```

**Format conversion:**
```
convert /tmp/data.json to CSV - array at root, columns: id, name, email, created_at, include headers
```

**Merge files:**
```
merge base.json and overrides.json - deep merge, overrides win on conflict
```

**Large file streaming:**
```
extract error events from /var/log/app.jsonl - streaming JSONL, filter .level == "error", last 24 hours
```

**Make greppable:**
```
find all paths containing "email" in /tmp/complex.json - use gron to flatten then grep
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool to invoke `task-json-recipes` with your constructed argument.

### When to Ask Instead

**Ask the user if:**
- File path cannot be determined from context
- Multiple files could apply and it's ambiguous
- The goal itself is unclear ("fix the JSON" with no context)

**Use Read tool first if:**
- Need to inspect file structure before building the argument
- Need to verify file exists or check its format
- User says "that file" but path is uncertain
