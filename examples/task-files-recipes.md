---
description: "File operations recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-files with file paths and context included"
when-to-use: "Receives prepared arguments from task-files with file paths and context included"
context: fork
model: inherit
allowed-tools: Read
---

# File Operations Recipes

**Tools:** find, fd (fdfind), locate, updatedb, bfs, which, whereis, file, stat, readlink, realpath, namei, ls, tree, diff, diff3, sdiff, cmp, difft, difftastic, wiggle, patch, cp, mv, rm, ln, mkdir, rmdir, touch, install, shred, rename, rsync, scp, rclone, sshfs, du, df, ncdu, dust, duf, lsblk, findmnt, mount, umount, blkid, losetup, fallocate, wipefs

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Find by name | find | `find . -name "*.py"` |
| Find by name (fast) | fd | `fdfind "\.py$"` |
| Find recent files | find | `find . -mtime -1` |
| Find large files | find | `find . -size +100M` |
| Compare files | diff | `diff file1 file2` |
| Sync directories | rsync | `rsync -av src/ dest/` |
| Disk usage | du | `du -sh */` |
| Directory tree | tree | `tree -L 2` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Find by name | fd | Faster, simpler syntax |
| Complex find criteria | find | More options |
| Quick diff | diff | Universal |
| Side-by-side diff | diff -y | Visual comparison |
| Sync/backup | rsync | Efficient, preserves attrs |
| Simple copy | cp | Small operations |
| Disk usage summary | du -sh | Per-directory totals |
| Visual tree | tree | Quick overview |

Note: On Ubuntu, fd is `fdfind` (package: fd-find).

---

## Finding Files

### By Name
```bash
find . -name "*.py"                   # Exact glob
find . -iname "*.py"                  # Case insensitive
find . -name "test_*"                 # Prefix match
find . -name "*config*"               # Contains

# fd (faster, simpler)
fdfind "\.py$"                        # Regex by default
fdfind -e py                          # By extension
fdfind "test_"                        # Partial match
fdfind -g "*.py"                      # Glob mode
fdfind "config"                       # Fuzzy match
```

### By Type
```bash
find . -type f                        # Files only
find . -type d                        # Directories only
find . -type l                        # Symlinks
find . -type f -o -type l             # Files or symlinks

fdfind --type f                       # Files
fdfind --type d                       # Directories
fdfind -t f -t l                      # Files and symlinks
```

### By Time
```bash
find . -mtime -1                      # Modified in last 24h
find . -mtime +7                      # Modified more than 7 days ago
find . -mtime 0                       # Modified today
find . -mmin -60                      # Modified in last 60 minutes
find . -newer reference.txt           # Newer than reference file
find . -newermt "2024-01-01"          # Newer than date

fdfind --changed-within 1d            # Last day
fdfind --changed-within 1h            # Last hour
fdfind --changed-before 1w            # More than 1 week ago
```

### By Size
```bash
find . -size +100M                    # Larger than 100MB
find . -size -1k                      # Smaller than 1KB
find . -size +10M -size -100M         # Between 10MB and 100MB
find . -empty                         # Empty files/dirs

fdfind --size +100m                   # Larger than 100MB
fdfind -S +1g                         # Larger than 1GB
fdfind -S -1k                         # Smaller than 1KB
```

### By Permissions
```bash
find . -perm 755                      # Exact permissions
find . -perm -u+x                     # User executable
find . -perm /u+x,g+x                 # User OR group executable
find . -executable                    # Any executable
```

### By Owner
```bash
find . -user root                     # Owned by root
find . -group www-data                # Group www-data
find . -nouser                        # No valid owner
```

### Combining Criteria
```bash
find . -name "*.log" -mtime +30 -size +10M
find . -type f \( -name "*.py" -o -name "*.js" \)
find . -type f -not -name "*.pyc"
fdfind -e py -e js --size +1k
fdfind -e log --changed-before 30d
```

### Exclude Patterns
```bash
find . -name "*.py" -not -path "*/venv/*"
find . -type f -not -path "*/.git/*"
fdfind -e py --exclude venv
fdfind "." -E node_modules -E .git
```

---

## Execute on Results

### Delete Matches
```bash
find . -name "*.tmp" -delete          # Delete matches (safe)
find . -name "*.bak" -type f -delete
fdfind -e tmp -x rm                   # Delete matches
fdfind -e pyc --exec rm {}            # Explicit syntax
```

### Execute Command
```bash
find . -name "*.py" -exec wc -l {} +  # Count lines (efficient)
find . -name "*.bak" -exec rm {} \;   # Remove each (slower)
find . -type f -exec chmod 644 {} +   # Fix permissions

fdfind -e py -x wc -l                 # Count lines
fdfind -e jpg -x convert {} {.}.png   # Convert images
```

### Batch Processing
```bash
find . -name "*.txt" -print0 | xargs -0 wc -l   # Handle spaces
find . -type f -print0 | xargs -0 -I{} cp {} backup/
fdfind -0 -e py | xargs -0 wc -l
```

### Interactive Delete
```bash
find . -name "*.log" -ok rm {} \;     # Prompt before each
```

---

## Comparing Files

### Basic Diff
```bash
diff file1 file2                      # Show differences
diff -u file1 file2                   # Unified format (for patches)
diff -y file1 file2                   # Side by side
diff -y --width=120 file1 file2       # Wider side-by-side
diff -q file1 file2                   # Just report if different
```

### Directory Diff
```bash
diff -rq dir1/ dir2/                  # Compare directories
diff -r dir1/ dir2/                   # Show all differences
diff -rq dir1/ dir2/ | grep "Only"    # Only files unique to one dir
```

### Ignore Options
```bash
diff -w file1 file2                   # Ignore whitespace
diff -B file1 file2                   # Ignore blank lines
diff -I "^#" file1 file2              # Ignore lines matching pattern
diff -b file1 file2                   # Ignore space changes
```

### Better Diff (with delta)
```bash
diff -u file1 file2 | delta           # Syntax highlighted diff
git diff | delta                      # For git diffs
```

### Diff Statistics
```bash
diff -u file1 file2 | diffstat        # Summary statistics
diff --brief file1 file2              # Just same/different
```

### Three-way Diff
```bash
diff3 mine base yours                 # Three-way merge
```

---

## Copying and Syncing

### Basic Copy
```bash
cp file dest/                         # Copy file
cp -r dir/ dest/                      # Copy directory
cp -a dir/ dest/                      # Archive mode (preserves all)
cp -p file dest/                      # Preserve timestamps
cp -i file dest/                      # Interactive (prompt)
cp -u file dest/                      # Only if newer
```

### Rsync (better for large/repeated copies)
```bash
rsync -av src/ dest/                  # Archive + verbose
rsync -av --delete src/ dest/         # Mirror (delete extras in dest)
rsync -av --dry-run src/ dest/        # Preview only
rsync -avz src/ remote:dest/          # With compression (for remote)
rsync -av --progress src/ dest/       # Show progress
rsync -av --info=progress2 src/ dest/ # Overall progress bar
```

### Rsync Exclusions
```bash
rsync -av --exclude="*.log" src/ dest/
rsync -av --exclude-from=.rsyncignore src/ dest/
rsync -av --exclude=".git" --exclude="node_modules" src/ dest/
rsync -av --include="*.py" --exclude="*" src/ dest/  # Only .py files
```

### Rsync Options
```bash
rsync -av --backup --suffix=.bak src/ dest/   # Backup changed files
rsync -av --update src/ dest/                  # Skip newer files in dest
rsync -av --checksum src/ dest/                # Compare by checksum
rsync -av --partial --progress src/ dest/      # Resume interrupted
```

### Remote Sync
```bash
rsync -avz src/ user@host:dest/       # Push to remote
rsync -avz user@host:src/ dest/       # Pull from remote
rsync -avz -e "ssh -p 2222" src/ user@host:dest/  # Custom SSH port
```

---

## Disk Usage

### Summary
```bash
du -sh directory/                     # Total size
du -sh */                             # Size of each subdir
du -sh * | sort -h                    # Sorted by size
du -sh * | sort -hr | head -10        # Top 10 largest
```

### Detailed Usage
```bash
du -h directory/                      # All subdirs
du -h --max-depth=1 directory/        # One level deep
du -h --max-depth=2 directory/        # Two levels deep
du -ah directory/                     # Include files
```

### Find Large Items
```bash
du -ah . | sort -rh | head -20        # Top 20 largest
du -h --max-depth=1 | sort -rh        # Largest subdirs

# ncdu (interactive)
ncdu .                                # Navigate interactively
ncdu -x /                             # Exclude other filesystems
```

### Disk Free Space
```bash
df -h                                 # All filesystems
df -h .                               # Current filesystem
df -h /home                           # Specific mount
df -i                                 # Inode usage
duf                                   # Modern df alternative
```

### By File Type
```bash
# Total size of Python files
find . -name "*.py" -exec du -ch {} + | tail -1

# Size by extension
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn
```

---

## Directory Listings

### Tree View
```bash
tree                                  # Full tree
tree -L 2                             # 2 levels deep
tree -L 3 -d                          # 3 levels, directories only
tree -a                               # Include hidden
tree -I "node_modules|.git|__pycache__"  # Exclude patterns
tree --dirsfirst                      # Dirs before files
tree -h                               # Human-readable sizes
tree -D                               # Print last modification date
tree -p                               # Print permissions
tree --prune                          # Don't show empty dirs
```

### Tree Output Formats
```bash
tree -J                               # JSON output
tree -X                               # XML output
tree -H . -o tree.html                # HTML output
```

### Enhanced Listing (eza)
```bash
eza -la                               # Long listing with all
eza --tree --level=2                  # Tree view
eza -l --git                          # With git status
eza -l --icons                        # With icons
eza -l --header                       # Column headers
```

### Standard ls
```bash
ls -la                                # Long format, all files
ls -lah                               # Human-readable sizes
ls -lt                                # Sort by time
ls -lS                                # Sort by size
ls -lR                                # Recursive
ls -1                                 # One per line
```

---

## File Operations

### Move/Rename
```bash
mv oldname newname                    # Rename
mv file dir/                          # Move to directory
mv -i file dir/                       # Interactive (prompt)
mv -n file dir/                       # No overwrite

# Bulk rename with rename
rename 's/\.txt$/.md/' *.txt          # .txt to .md
rename 's/^/prefix_/' *               # Add prefix
rename 'y/A-Z/a-z/' *                 # Lowercase
```

### Create Directories
```bash
mkdir dirname                         # Single directory
mkdir -p path/to/nested/dir           # Create parents
mkdir -m 755 dirname                  # With permissions
```

### Symlinks
```bash
ln -s /path/to/target linkname        # Create symlink
ln -sf /path/to/target linkname       # Force overwrite
readlink linkname                     # Read link target
realpath linkname                     # Resolve to absolute path
```

### Touch (timestamps)
```bash
touch newfile                         # Create or update timestamp
touch -d "2024-01-01" file            # Set specific date
touch -r reference file               # Copy timestamp from reference
```

---

## Common Patterns

### Find and Delete Old Files
```bash
find . -name "*.log" -mtime +30 -delete
find /tmp -type f -atime +7 -delete
fdfind -e log --changed-before 30d -x rm
```

### Find Duplicates (by name)
```bash
find . -type f | xargs -n1 basename | sort | uniq -d
```

### Backup with Date
```bash
rsync -av src/ "backup_$(date +%Y%m%d)/"
tar czf "backup_$(date +%Y%m%d).tar.gz" src/
```

### Find Recently Modified
```bash
find . -type f -mmin -30 -ls          # Last 30 minutes
find . -type f -mtime 0               # Today
fdfind --changed-within 30min
fdfind --changed-within 1d
```

### Find Empty Files/Directories
```bash
find . -type f -empty                 # Empty files
find . -type d -empty                 # Empty directories
find . -type d -empty -delete         # Delete empty dirs
```

### Safe Delete (move to trash)
```bash
mkdir -p ~/.trash
mv file ~/.trash/
# Or use trash-cli
trash-put file
```

### Secure Delete
```bash
shred -vzu file                       # Overwrite and remove
shred -v -n 3 file                    # 3 passes, keep file
```

---

## File Information

### Basic Info
```bash
file unknown_file                     # Identify type
stat file                             # Detailed metadata
ls -l file                            # Quick info
```

### Checksums
```bash
md5sum file                           # MD5 hash
sha256sum file                        # SHA-256 hash
sha256sum file1 file2 | tee checksums.txt
sha256sum -c checksums.txt            # Verify
```

---

## Advanced rsync

### Filter Rules
```bash
rsync -av --include="*.py" --include="*/" --exclude="*" src/ dest/   # Only .py (need dirs)
rsync -av --filter="- .git/" --filter="- __pycache__/" src/ dest/    # Filter syntax
rsync -av --filter="merge .rsync-filter" src/ dest/                  # Rules from file
```

### Partial and Resume
```bash
rsync -av --partial --progress src/ dest/         # Resume interrupted transfers
rsync -av --partial-dir=.rsync-partial src/ dest/  # Store partials in hidden dir
rsync -avP src/ dest/                              # Shorthand: --partial --progress
```

### Delete and Mirror
```bash
rsync -av --delete src/ dest/                     # Mirror: delete extras in dest
rsync -av --delete --dry-run src/ dest/           # Preview what would be deleted
rsync -av --delete-excluded --exclude="*.log" src/ dest/  # Delete excluded from dest
rsync -av --delete-after src/ dest/               # Delete after transfer (safer)
```

### SSH and Remote
```bash
rsync -avz -e "ssh -p 2222" src/ user@host:dest/  # Custom SSH port
rsync -avz -e "ssh -i ~/.ssh/mykey" src/ user@host:dest/  # Custom SSH key
rsync -avz --bwlimit=5000 src/ user@host:dest/    # Bandwidth limit (KB/s)
```

---

## Advanced Disk Analysis

### du Patterns
```bash
du -sh * | sort -h                    # Sorted human-readable sizes
du --max-depth=1 -h /path | sort -h  # One level, sorted
du -ah --exclude='*.log' /path | sort -rh | head -20  # Largest non-log files
du -sh --apparent-size dir/           # Actual bytes (not disk blocks)
du -BM --max-depth=1 /path | sort -rn  # In megabytes, numeric sort
```

### df Patterns
```bash
df -h                                 # Human-readable all filesystems
df -h /home                           # Specific mount point
df -ih                                # Inode usage (important for many small files)
df -T                                 # Show filesystem type
df --output=source,fstype,size,used,avail,pcent /  # Custom columns
```

### ncdu (Interactive)
```bash
ncdu /path                            # Interactive disk usage browser
ncdu -x /                             # Exclude other filesystems
ncdu -e /path                         # Enable file deletion within ncdu
ncdu --exclude='*.log' /path          # Exclude pattern
ncdu -o report.json /path             # Export as JSON
ncdu -f report.json                   # Load from export
```

---

## Directory Comparison

### diff -rq (Quick Compare)
```bash
diff -rq dir1/ dir2/                  # Report differences only
diff -rq dir1/ dir2/ | grep "Only"    # Files unique to one side
diff -rq dir1/ dir2/ | grep "differ"  # Files that differ in content
diff -rq --exclude=".git" dir1/ dir2/ # Exclude patterns
```

### rsync --dry-run (Detailed Compare)
```bash
rsync -avn --delete src/ dest/        # Show what rsync would do
rsync -avn --itemize-changes src/ dest/  # Detailed change codes
# Output codes: >f = file transfer, *deleting = delete, .d = directory
```

### Checksums for Verification
```bash
# Generate checksums for directory
find dir/ -type f -exec sha256sum {} + > checksums.txt

# Verify directory against checksums
sha256sum -c checksums.txt

# Compare two directories by checksum
diff <(cd dir1 && find . -type f | sort | xargs sha256sum) \
     <(cd dir2 && find . -type f | sort | xargs sha256sum)
```

---

## Archiving and Compression

### tar
```bash
# Create compressed archives
tar czf archive.tar.gz /data              # gzip
tar cjf archive.tar.bz2 /data             # bzip2 (smaller, slower)
tar cJf archive.tar.xz /data              # xz (smallest, slowest)

# Extract archives
tar xzf archive.tar.gz                    # gzip
tar xjf archive.tar.bz2                   # bzip2
tar xJf archive.tar.xz                    # xz
tar xzf archive.tar.gz -C /dest           # Extract to specific dir

# List contents without extracting
tar tzf archive.tar.gz
tar tzf archive.tar.gz | head -20

# Extract single file
tar xzf archive.tar.gz path/to/file

# Create with excludes
tar czf archive.tar.gz /data --exclude='node_modules' --exclude='.git' --exclude='*.log'

# Create from file list
find . -name "*.py" -newer last_backup | tar czf backup.tar.gz -T -

# Selective archiving
tar czf archive.tar.gz -T filelist.txt
```

### zip / unzip
```bash
# Create zip archive
zip -r archive.zip directory/
zip -r archive.zip dir/ -x "*.log" -x "node_modules/*"

# Unzip
unzip archive.zip
unzip archive.zip -d /dest               # Extract to specific dir
unzip -l archive.zip                      # List contents
unzip -t archive.zip                      # Test integrity

# Update existing zip
zip -u archive.zip newfile.txt
```

### gzip / 7z
```bash
# gzip single file (replaces original)
gzip file.txt                             # Creates file.txt.gz
gunzip file.txt.gz                        # Decompress

# Keep original
gzip -k file.txt

# 7z (highest compression)
7z a archive.7z directory/
7z x archive.7z                           # Extract
7z l archive.7z                           # List contents
```

---

## locate / updatedb

```bash
# Update the database (requires sudo)
sudo updatedb

# Find files by name
locate filename
locate -i filename                        # Case insensitive
locate "*.py"                             # Glob pattern
locate --regex '\.py$'                    # Regex pattern

# Count matches
locate -c "*.log"

# Limit results
locate -l 10 "config"                     # First 10 matches
```

---

## patch

```bash
# Create a patch
diff -u original.txt modified.txt > changes.patch

# Apply a patch
patch < changes.patch
patch original.txt < changes.patch

# Apply patch from directory diff
patch -p1 < feature.patch

# Dry run (test without applying)
patch --dry-run < changes.patch

# Reverse a patch
patch -R < changes.patch

# Apply patch with fuzz (tolerate offset)
patch --fuzz=3 < changes.patch
```

---

## Remote File Access (scp / sshfs)

```bash
# Copy file to remote
scp file.txt user@host:/path/
scp -r directory/ user@host:/path/        # Recursive
scp -P 2222 file.txt user@host:/path/     # Custom port

# Copy from remote
scp user@host:/path/file.txt .
scp -r user@host:/path/dir/ .

# Mount remote filesystem
mkdir -p /mnt/remote
sshfs user@host:/path /mnt/remote
# Access files at /mnt/remote/...

# Unmount
fusermount -u /mnt/remote
```

---

## Path Resolution (which / whereis / namei)

```bash
# Find executable location
which python3
which -a python3                          # All matches in PATH

# Find binary, source, and man page
whereis nginx

# Trace symlink chain
namei -l /usr/local/bin/python
# Shows every component of the path with permissions and symlink targets

# Resolve to absolute path
realpath ./relative/path
readlink -f /usr/local/bin/python         # Follow all symlinks
```

---

## Modern Disk Tools (dust / duf)

```bash
# dust: visual disk usage (like du but better)
dust                                      # Current directory
dust -n 20 /var                           # Top 20 in /var
dust -d 2                                 # Depth 2

# duf: visual df (disk free)
duf                                       # All filesystems
duf --only local                          # Local filesystems only
duf /home                                 # Specific mount
```

---

## Block Devices and Mounts

```bash
# List block devices with filesystem info
lsblk -f

# Show all mount points as tree
findmnt --real

# Show UUID and filesystem type
blkid

# Mount/unmount
mount /dev/sdb1 /mnt/usb
umount /mnt/usb
mount -o ro /dev/sdb1 /mnt/usb            # Read-only
```

---

## File Watching

```bash
# Watch for changes (inotify)
inotifywait -m -r /path -e modify,create,delete

# Run command on file change (entr)
ls *.py | entr pytest                     # Rerun tests on change
ls *.py | entr -r python app.py           # Restart app on change
fdfind -e py | entr -r python app.py      # Using fd for file list

# fswatch (cross-platform)
fswatch -r /path | while read f; do echo "Changed: $f"; done
```

---

## Duplicate Detection

```bash
# Find duplicate files (fdupes)
fdupes -r /path                           # Recursive
fdupes -r -S /path                        # Show sizes
fdupes -r -d /path                        # Interactive delete

# jdupes (faster fdupes alternative)
jdupes -r /path

# Checksum-based duplicate detection
find . -type f -exec md5sum {} + | sort | uniq -D -w32

# Find duplicate filenames (not content)
find . -type f | xargs -n1 basename | sort | uniq -d
```

---

## Combined Workflows

```bash
# Find large files with details
fdfind -S +100M | xargs ls -lhS

# Selective archive of recently changed files
find . -name "*.py" -newer last_backup | tar czf backup.tar.gz -T -

# Identify file types in directory
find . -type f -exec file {} + | grep "Python" | cut -d: -f1

# Diff and patch round-trip
diff -u original modified > fix.patch && patch original < fix.patch

# Disk space investigation
du -ah /var | sort -rh | head -20

# rsync dry-run to find what would be deleted
rsync -avn --delete src/ dest/ | grep 'deleting'

# Find world-writable files
find / -type f -perm -o+w -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null

# Archive excluding version control
tar czf project-clean.tar.gz project/ --exclude='.git' --exclude='node_modules' --exclude='__pycache__'
```

---

## Installation

```bash
# find, diff, cp, du, df, ls, rsync are pre-installed

# Modern alternatives
sudo apt install fd-find              # fd (use as fdfind)
sudo apt install tree
sudo apt install ncdu                 # Interactive du
sudo apt install eza                  # Modern ls replacement
sudo apt install trash-cli            # Safe delete
```
