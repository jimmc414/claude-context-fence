---
description: "Archive and compression recipes reference (internal) - Receives prepared arguments from task-archives with archive type and context included"
argument-hint: "[prepared argument from task-archives router]"
context: fork
model: inherit
allowed-tools: Read
---

# Archive and Compression Tasks

**Tools:** tar, gzip, gunzip, bzip2, bunzip2, xz, unxz, zstd, zip, unzip, 7z, rar, unrar, pigz, cpio, ar, zrun

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Create tar.gz | `tar -czvf archive.tar.gz directory/` |
| Extract tar.gz | `tar -xzvf archive.tar.gz` |
| Create zip | `zip -r archive.zip directory/` |
| Extract zip | `unzip archive.zip` |
| Create 7z | `7z a archive.7z directory/` |
| Extract 7z | `7z x archive.7z` |
| Compress file | `gzip file` |
| Decompress | `gunzip file.gz` |
| List contents | `tar -tvf archive.tar.gz` |

---

## Tar Archives

```bash
tar -czvf archive.tar.gz directory/   # Create tar.gz
tar -cjvf archive.tar.bz2 directory/  # Create tar.bz2
tar -cJvf archive.tar.xz directory/   # Create tar.xz
tar --zstd -cvf archive.tar.zst dir/  # Create tar.zst

tar -xvf archive.tar.gz               # Extract (auto-detect)
tar -xvf archive.tar.gz -C /target/   # Extract to directory
tar -tvf archive.tar.gz               # List contents
```

---

## Zip Archives

```bash
zip -r archive.zip directory/
zip -9 -r archive.zip directory/      # Max compression
unzip archive.zip
unzip archive.zip -d /target/
unzip -l archive.zip                  # List contents
```

---

## 7z Archives

```bash
7z a archive.7z directory/
7z a -p archive.7z directory/         # With password
7z x archive.7z
7z l archive.7z                       # List contents
```

---

## Individual File Compression

```bash
gzip file              # Creates file.gz
gzip -k file           # Keep original
gunzip file.gz

zstd file              # Creates file.zst (fast + good compression)
zstd -19 file          # Max compression
unzstd file.zst
```

---

## Installation

```bash
# Most tools pre-installed
sudo apt install p7zip-full zstd pigz
```
