---
description: "Text search and transformation - can't find where text appears in codebase, locate error message source, need batch text replacements, finding patterns, filtering lines, and extracting matches (ripgrep, grep, sed, ast-grep)"
when_to_use: "Use when: can't find where a specific text or error message appears, need to locate where a string is defined or used, need to do batch search and replace across files, removing debug statements, finding TODO comments, searching codebases for patterns, filtering log files, replacing text in files. Triggers: can't find text, where does this appear, locate error message, batch replace, find in codebase, where is this defined, grep, search text, find pattern, filter lines, replace text, regex match, log search, ripgrep, sed, find in files, ast-grep, structural search, code pattern, encoding, iconv, diff files, word count, remove console.log, find function, code cleanup, refactor pattern, search and replace, find unused, dead code, find imports, find definition, bulk rename, remove debug statements, find TODO, codemod."
when-to-use: "Use when: can't find where a specific text or error message appears, need to locate where a string is defined or used, need to do batch search and replace across files, removing debug statements, finding TODO comments, searching codebases for patterns, filtering log files, replacing text in files. Triggers: can't find text, where does this appear, locate error message, batch replace, find in codebase, where is this defined, grep, search text, find pattern, filter lines, replace text, regex match, log search, ripgrep, sed, find in files, ast-grep, structural search, code pattern, encoding, iconv, diff files, word count, remove console.log, find function, code cleanup, refactor pattern, search and replace, find unused, dead code, find imports, find definition, bulk rename, remove debug statements, find TODO, codemod."
argument-hint: "[describe what you want to search for or replace]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Text Search Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the text search recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Goal
What text operation is needed?
- Search for pattern, filter lines, count matches, replace text, extract content

### 2. File/Directory Path(s)
Where to search?
- Explicit paths mentioned by user
- Files created/modified earlier in conversation
- Files referenced in error messages
- Directory for recursive search

### 3. Pattern
What to search for?
- **Literal text**: exact string match
- **Regex**: pattern with special characters
- **Case sensitivity**: case-insensitive needed?
- **Word boundaries**: whole word only?

### 4. Context Requirements
What surrounding information?
- Lines before/after match
- Line numbers
- Filename with match
- Just the matching part (not whole line)

### 5. Transformation (if replacing)
- What to replace with
- All occurrences or first only
- In-place or to stdout

### 6. Scope
Any boundaries?
- File type filter (*.py, *.log)
- Exclude patterns
- Recursive or single file
- First N matches only

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Search text in files | rg (ripgrep) | Fast, respects .gitignore |
| Basic pattern search | grep | Pre-installed everywhere |
| Structural code refactor | ast-grep | Syntax-aware search/replace |
| Replace text in files | sed | Stream editor, in-place with -i |
| Modern find & replace | sd | Simpler regex syntax than sed |
| Count lines/words | wc | Line, word, byte counts |
| Character transliteration | tr | Delete or replace characters |
| Split file into pieces | split / csplit | By lines, bytes, or pattern |

---

## Argument Format

Construct a structured argument string:

```
<goal> in <path> - pattern: <what>, options: <flags>, output: <format>
```

**Include only relevant components.**

### Examples

**Simple search:**
```
find lines containing "error" in /var/log/app.log - case insensitive, show line numbers
```

**Recursive search:**
```
search for "TODO" in ./src/ - file types: py,js, show context: 2 lines
```

**Extract matches:**
```
extract email addresses from /tmp/contacts.txt - pattern: regex for emails, output: only matches
```

**Replace:**
```
replace "http://" with "https://" in ./config/*.yaml - all occurrences, in-place
```

**Filter logs:**
```
filter lines NOT containing "DEBUG" from /var/log/app.log - also exclude empty lines
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-text-search-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- File path cannot be determined from context
- Pattern is ambiguous (literal vs regex)
- Replace operation needs confirmation

**Use Read tool first if:**
- Need to see sample content to build pattern
- Need to verify file exists
