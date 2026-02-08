---
description: "Archive and compression - file too large to transfer, can't open archive, archive corrupted, reduce file size for sharing, creating/extracting archives, and compression (tar, zip, 7z, gzip, zstd)"
when_to_use: "Use when: file or directory too large to transfer or share, can't open or extract an archive, archive appears corrupted, need to reduce file size, bundling files for distribution, creating tar archives, extracting zip files, handling compressed files. Triggers: file too large, can't open archive, archive corrupted, reduce file size, too big to email, too big to upload, tar, zip, unzip, gzip, compress, extract, archive, 7z, zstd, rar, untar, gunzip, bzip2, pigz, password-protected archive, split archive, self-extracting, decompress, pack, unpack, tar.gz, tar.bz2, tar.xz."
when-to-use: "Use when: file or directory too large to transfer or share, can't open or extract an archive, archive appears corrupted, need to reduce file size, bundling files for distribution, creating tar archives, extracting zip files, handling compressed files. Triggers: file too large, can't open archive, archive corrupted, reduce file size, too big to email, too big to upload, tar, zip, unzip, gzip, compress, extract, archive, 7z, zstd, rar, untar, gunzip, bzip2, pigz, password-protected archive, split archive, self-extracting, decompress, pack, unpack, tar.gz, tar.bz2, tar.xz."
argument-hint: "[describe what you want to archive or extract]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Archive Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the archive recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Operation
What archive operation is needed?
- Create archive, extract archive, list contents, test integrity
- Compress files, decompress files
- Add to existing archive, extract specific files

### 2. Archive Type
What format?
- tar.gz / tar.bz2 / tar.xz / tar.zst
- zip / 7z / rar
- Single file compression (gzip, zstd, xz)

### 3. Source Path(s)
What files/directories to archive?
- Explicit paths from conversation
- Files discussed earlier
- Glob patterns

### 4. Destination
Where should the result go?
- Output archive path
- Extraction target directory

### 5. Options
Special requirements?
- Compression level (fast vs max)
- Password protection
- Exclude patterns
- Preserve permissions
- Split into parts

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Create tar.gz/bz2/xz | tar | Most common archive format |
| Create zip | zip | Cross-platform compatibility |
| Maximum compression | 7z / zstd -19 | Best compression ratio |
| Parallel compression | pigz | Multi-core gzip replacement |
| Fast + good compression | zstd | Best speed/ratio tradeoff |
| Password-protected | 7z -p | Encrypted archives |
| Extract any format | tar -xvf / unzip / 7z x | Auto-detect with tar |
| List archive contents | tar -tvf / unzip -l / 7z l | Preview before extract |

---

## Argument Format

Construct a structured argument string:

```
<operation> <source_path> - format: <type>, destination: <path>, options: <settings>
```

**Include only relevant components.**

### Examples

**Create tar.gz:**
```
create tar.gz archive of /home/user/project - destination: /tmp/project-backup.tar.gz, exclude: node_modules,.git
```

**Extract archive:**
```
extract /tmp/data.tar.gz - destination: /home/user/data/
```

**Maximum compression:**
```
compress /home/user/logs/ - format: 7z, compression: ultra
```

**List contents:**
```
list contents of /tmp/mystery-archive.tar.gz
```

**Password-protected:**
```
create encrypted zip of /home/user/sensitive/ - password-protected, destination: /tmp/secure.zip
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-archives-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Archive format preference is unclear
- Source paths are ambiguous
- Password needed but not specified

**Use Read tool first if:**
- Need to verify source paths exist
- Need to check available disk space
