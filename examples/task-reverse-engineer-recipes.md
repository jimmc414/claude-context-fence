---
description: "Reverse engineering recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-reverse-engineer with file paths and context included"
when-to-use: "Receives prepared arguments from task-reverse-engineer with file paths and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Reverse Engineering Recipes

**Tools:** file, strings, binwalk, hexdump, xxd, od, readelf, objdump, nm, ldd, size, objcopy, strace, ltrace, gdb, tcpdump, tshark, ngrep, socat, nc, pdfinfo, pdftotext, zipinfo, exiftool, mediainfo, base64, hashid, openssl, radare2, yara

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Identify file type | `file unknown_file` |
| Extract strings | `strings -a binary \| head -100` |
| Hex dump | `xxd binary \| head -50` |
| Find embedded files | `binwalk -e firmware.bin` |
| ELF headers | `readelf -h binary` |
| List symbols | `nm binary` |
| Disassemble | `objdump -d binary \| head -200` |
| Shared libraries | `ldd binary` |
| Trace syscalls | `strace -f ./binary 2>&1` |
| Trace library calls | `ltrace ./binary 2>&1` |
| Capture packets | `tcpdump -i eth0 -w capture.pcap` |
| Decode packets | `tshark -r capture.pcap -T json` |
| Identify hash type | `hashid 'hash_string'` |
| Java bytecode | `javap -c -p ClassName` |
| APK analysis | `apktool d app.apk` |
| Memory forensics | `volatility -f dump.raw imageinfo` |
| Pattern matching | `yara rules.yar suspect_file` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| ELF header analysis | readelf -h | Detailed header breakdown |
| Symbol table | nm | Function/variable names |
| Disassembly | objdump -d | Instruction-level view |
| Shared library deps | ldd | What libraries are linked |
| Runtime syscalls | strace | Observe system calls |
| Runtime library calls | ltrace | Observe library function calls |
| Firmware carving | binwalk | Detect/extract embedded files |
| Hex inspection | xxd | Hex + ASCII side by side |
| Pattern rules | yara | Match binary signatures |
| Interactive analysis | radare2 | Full reversing framework |

---

## File Triage

### Identify File Type
```bash
file unknown_file                     # Magic byte identification
file -i unknown_file                  # MIME type
file -b unknown_file                  # Brief (no filename)
file -z compressed_file               # Look inside compressed
file -L symlink                       # Follow symlinks
file -k file                          # Keep going (multiple matches)
```

### Batch Identification
```bash
file *                                # All files in directory
find . -type f -exec file {} \;       # Recursive
find . -type f | xargs file           # Faster for many files
```

---

## String Extraction

### Basic Extraction
```bash
strings -a binary                     # All sections (not just data)
strings binary                        # Default (data section only)
strings -n 10 binary                  # Minimum 10 characters
strings -t x binary                   # With hex offset
strings -t d binary                   # With decimal offset
strings -e l binary                   # Little-endian 16-bit
strings -e b binary                   # Big-endian 16-bit
strings -e L binary                   # Little-endian 32-bit
```

### Filtering Strings
```bash
strings -a binary | grep -i password
strings -a binary | grep -i secret
strings -a binary | grep -i key
strings -a binary | grep -i token
strings -a binary | grep -i api
strings -a binary | grep -E 'http://|https://'
strings -a binary | grep -E '[A-Za-z0-9+/]{40,}={0,2}'  # Base64
strings -a binary | grep -E '[0-9a-f]{32,}'              # Hex strings
```

### Sort and Deduplicate
```bash
strings -a binary | sort -u           # Unique sorted
strings -a binary | sort | uniq -c | sort -rn | head -20  # By frequency
```

---

## Hex Inspection

### xxd (recommended)
```bash
xxd binary | head -50                 # Hex dump with ASCII
xxd -s 0x100 -l 64 binary             # Offset 0x100, 64 bytes
xxd -s -0x100 binary | tail -20       # Last 256 bytes
xxd -c 32 binary | head -20           # 32 bytes per line
xxd -g 1 binary | head -20            # Group by 1 byte
xxd -g 4 binary | head -20            # Group by 4 bytes
xxd -i binary > binary.h              # C array output
xxd -r hexdump.txt binary             # Reverse (hex to binary)
xxd -p binary                         # Plain hex (no ASCII)
```

### hexdump
```bash
hexdump -C binary | head -50          # Canonical hex+ASCII
hexdump -n 256 binary                 # First 256 bytes
hexdump -s 0x100 -n 64 binary         # Skip to offset
```

### od (octal dump)
```bash
od -A x -t x1z binary | head -50      # Hex with ASCII
od -A x -t x4 binary | head -20       # 4-byte hex
od -A x -t d4 binary | head -20       # 4-byte decimal
od -A x -t f4 binary | head -20       # 4-byte float
```

---

## Entropy Analysis

### Detect Encryption/Compression
```bash
binwalk -E binary                     # Entropy analysis
binwalk -E binary --save              # Save entropy plot
ent binary                            # Entropy calculation
```

High entropy (>7.5) suggests encryption or compression.

---

## Firmware & Embedded Files

### Scan and Extract
```bash
binwalk firmware.bin                  # Scan for embedded files
binwalk -e firmware.bin               # Auto-extract
binwalk -Me firmware.bin              # Recursive extract
binwalk --dd='.*' firmware.bin        # Extract everything
binwalk -D 'png image:png' fw.bin     # Extract specific type
```

### Search Patterns
```bash
binwalk -A firmware.bin               # Scan for opcodes
binwalk -R '\x89PNG' firmware.bin     # Search raw bytes
binwalk -R 'ELF' firmware.bin         # Find ELF headers
binwalk --include=filesystem fw.bin   # Only filesystems
binwalk --exclude=compressed fw.bin   # Skip compressed
```

### After Extraction
```bash
find _firmware.bin.extracted -type f -exec file {} \;
strings -a _firmware.bin.extracted/* | grep -i admin
grep -r "password" _firmware.bin.extracted/
```

---

## ELF Binary Analysis

### Headers and Structure
```bash
readelf -h binary                     # ELF header
readelf -l binary                     # Program headers (segments)
readelf -S binary                     # Section headers
readelf -s binary                     # Symbol table
readelf -d binary                     # Dynamic section
readelf -r binary                     # Relocations
readelf -n binary                     # Notes
readelf -a binary                     # All info
readelf -p .rodata binary             # Print section as strings
```

### Symbols
```bash
nm binary                             # List symbols
nm -C binary                          # Demangle C++ names
nm -D binary                          # Dynamic symbols only
nm --defined-only binary              # Only defined symbols
nm -u binary                          # Only undefined (external)
nm -g binary                          # Only global symbols
nm -S binary                          # With symbol sizes
nm -n binary                          # Sort by address
```

### Symbol Types
- `T/t`: Code (text) section
- `D/d`: Initialized data
- `B/b`: Uninitialized data (BSS)
- `U`: Undefined (external)
- `W`: Weak symbol

### Disassembly
```bash
objdump -d binary                     # Disassemble .text
objdump -D binary                     # Disassemble all sections
objdump -d -M intel binary            # Intel syntax (vs AT&T)
objdump -d --no-show-raw-insn binary  # Hide machine code
objdump -d -j .text binary            # Specific section
objdump -t binary                     # Symbol table
objdump -T binary                     # Dynamic symbols
objdump -x binary                     # All headers
objdump -s -j .rodata binary          # Dump section contents
objdump -R binary                     # Dynamic relocations
```

### Disassemble Single Function
```bash
objdump -d binary | sed -n '/<main>:/,/^$/p'
objdump -d binary | awk '/<main>:/,/^$/'
```

### Dependencies
```bash
ldd binary                            # Shared library deps
ldd -v binary                         # Version info
ldd -u binary                         # Unused direct deps
readelf -d binary | grep NEEDED       # Required libraries
size binary                           # Section sizes
size -A binary                        # Section sizes (detailed)
```

### Extract Sections
```bash
objcopy -O binary --only-section=.text binary text.bin
objcopy -O binary --only-section=.data binary data.bin
objcopy -O binary --only-section=.rodata binary rodata.bin
objcopy --dump-section .text=text.bin binary
```

---

## Dynamic Analysis

### System Call Tracing (strace)
```bash
strace ./binary                       # Trace syscalls
strace -f ./binary                    # Follow forks
strace -ff -o trace ./binary          # Separate files per process
strace -p PID                         # Attach to running process
strace -c ./binary                    # Syscall statistics
strace -o trace.log ./binary          # Output to file
strace -s 1000 ./binary               # Longer string output (default 32)
strace -x ./binary                    # Hex output for non-ASCII
```

### Filter by Syscall Type
```bash
strace -e open,read,write ./binary    # Specific syscalls
strace -e trace=file ./binary         # File operations
strace -e trace=network ./binary      # Network calls
strace -e trace=process ./binary      # Process management
strace -e trace=signal ./binary       # Signals
strace -e trace=memory ./binary       # Memory operations
strace -e trace=ipc ./binary          # IPC operations
strace -e trace=desc ./binary         # File descriptor ops
```

### Timing
```bash
strace -T ./binary                    # Time spent in syscalls
strace -r ./binary                    # Relative timestamps
strace -t ./binary                    # Wall clock time
strace -tt ./binary                   # With microseconds
```

### Library Call Tracing (ltrace)
```bash
ltrace ./binary                       # Trace library calls
ltrace -e malloc+free ./binary        # Specific functions
ltrace -e '*' ./binary                # All functions
ltrace -c ./binary                    # Call statistics
ltrace -s 200 ./binary                # Longer strings
ltrace -n 2 ./binary                  # Indent nested calls
ltrace -l libfoo.so ./binary          # Only specific library
```

### GDB Batch Mode
```bash
gdb -batch -ex 'info functions' ./binary
gdb -batch -ex 'disas main' ./binary
gdb -batch -ex 'x/100s 0x08048000' ./binary
gdb -batch -ex 'info files' ./binary
gdb -batch -ex 'break main' -ex 'run' -ex 'bt' ./binary
gdb -batch -ex 'set pagination off' -ex 'info variables' ./binary
```

### GDB Scripting
```bash
# Create gdb_script.txt:
# set pagination off
# break main
# run
# info registers
# bt
# quit

gdb -batch -x gdb_script.txt ./binary
```

---

## Network & Protocol Analysis

### Packet Capture (tcpdump)
```bash
tcpdump -i eth0                       # Capture on interface
tcpdump -i any                        # All interfaces
tcpdump -D                            # List interfaces
tcpdump -w capture.pcap               # Write to file
tcpdump -r capture.pcap               # Read from file
tcpdump -c 100                        # Capture 100 packets
tcpdump -n                            # Don't resolve names
```

### Filtering (tcpdump)
```bash
tcpdump -i eth0 port 80               # By port
tcpdump -i eth0 host 10.0.0.1         # By host
tcpdump -i eth0 src 10.0.0.1          # Source only
tcpdump -i eth0 dst 10.0.0.1          # Destination only
tcpdump -i eth0 net 10.0.0.0/24       # By network
tcpdump -i eth0 'port 80 or port 443' # Multiple ports
tcpdump -i eth0 'tcp and port 80'     # Protocol + port
```

### Output Formats (tcpdump)
```bash
tcpdump -A -i eth0                    # ASCII output
tcpdump -X -i eth0                    # Hex + ASCII
tcpdump -XX -i eth0                   # Include link header
tcpdump -v -i eth0                    # Verbose
tcpdump -vvv -i eth0                  # Very verbose
```

### Protocol Decoding (tshark)
```bash
tshark -r capture.pcap                # Read pcap
tshark -r capture.pcap -T json        # JSON output
tshark -r capture.pcap -T fields -e ip.src -e ip.dst
tshark -r capture.pcap -T fields -e http.request.uri
tshark -r capture.pcap -Y 'http'      # Display filter
tshark -r capture.pcap -Y 'tcp.port==443'
tshark -i eth0 -f 'port 443'          # Live capture with filter
```

### Follow Streams
```bash
tshark -r capture.pcap -z follow,tcp,ascii,0
tshark -r capture.pcap -z follow,http,ascii,0
```

### Network Grep (ngrep)
```bash
ngrep -d eth0 'password'              # Match pattern
ngrep -q -d eth0 'GET|POST'           # HTTP methods
ngrep -W byline -d eth0 ''            # Line-based output
ngrep -d eth0 '' 'port 80'            # With filter
```

### Raw Network (netcat/socat)
```bash
nc -l -p 8080                         # Listen on port
nc host 80                            # Connect to host
nc -z host 1-1000                     # Port scan
nc -v host 80                         # Verbose connect
socat TCP-LISTEN:8080,fork TCP:target:80  # TCP relay
socat TCP-LISTEN:8080,fork -           # Echo server
```

---

## File Format Analysis

### Archives
```bash
zipinfo archive.zip                   # ZIP structure
unzip -l archive.zip                  # List contents
unzip -t archive.zip                  # Test integrity
tar -tvf archive.tar                  # List tar
tar -tvf archive.tar.gz               # List gzipped tar
7z l archive.7z                       # List 7z
rar l archive.rar                     # List rar
```

### PDF
```bash
pdfinfo document.pdf                  # PDF metadata
pdftotext document.pdf -              # Extract text
pdftotext -layout document.pdf -      # Preserve layout
pdfimages -list document.pdf          # List images
pdfimages -all document.pdf img/      # Extract images
pdftk document.pdf dump_data          # Dump metadata
```

### Media
```bash
exiftool file.jpg                     # All metadata
exiftool -json file.jpg               # JSON output
exiftool -GPS* file.jpg               # GPS data only
exiftool -all= file.jpg               # Strip all metadata
mediainfo file.mp4                    # Media details
mediainfo --Output=JSON file.mp4      # JSON output
ffprobe -v quiet -print_format json -show_format -show_streams file.mp4
```

### Office Documents
```bash
unzip -l document.docx                # DOCX is a ZIP
unzip document.docx -d docx_content/  # Extract
xmllint --format docx_content/word/document.xml | head -100
```

---

## Encoding & Crypto Detection

### Decode Common Encodings
```bash
base64 -d <<< 'encoded_string'        # Base64 decode
echo 'encoded' | base64 -d            # From pipe
base64 -d file.b64 > decoded          # From file
xxd -r -p <<< 'hex_string'            # Hex to binary
echo '68656c6c6f' | xxd -r -p         # Hex to ASCII
```

### URL Encoding
```bash
python3 -c 'import sys,urllib.parse;print(urllib.parse.unquote(sys.stdin.read()))' <<< 'url%20encoded'
python3 -c 'import sys,urllib.parse;print(urllib.parse.quote(sys.stdin.read()))'
```

### Identify Hash Types
```bash
hashid 'hash_string'                  # Identify hash algorithm
hashid -m 'hash_string'               # With hashcat mode
hashid -e 'hash_string'               # Extended output
```

### Common Hash Lengths
- MD5: 32 hex chars
- SHA1: 40 hex chars
- SHA256: 64 hex chars
- SHA512: 128 hex chars

### Certificate/Key Inspection
```bash
openssl x509 -in cert.pem -text -noout    # View certificate
openssl x509 -in cert.pem -dates          # Validity dates
openssl x509 -in cert.pem -subject        # Subject
openssl rsa -in key.pem -text -noout      # View RSA key
openssl rsa -in key.pem -check            # Verify key
openssl s_client -connect host:443        # TLS handshake
openssl s_client -connect host:443 -showcerts  # Show cert chain
openssl pkcs12 -info -in file.p12         # PKCS12 info
```

---

## Pattern Matching (YARA)

### Basic Usage
```bash
yara rules.yar target_file            # Match rules
yara -r rules.yar directory/          # Recursive
yara -s rules.yar target_file         # Show matching strings
yara -m rules.yar target_file         # Show metadata
yara -c rules.yar target_file         # Count matches
```

### Example YARA Rule
```yara
rule suspicious_strings {
    meta:
        description = "Detect suspicious strings"
        author = "analyst"
    strings:
        $a = "password" nocase
        $b = "/bin/sh"
        $c = { 89 E5 83 EC }          // Hex pattern
        $d = /https?:\/\/[^\s]+/      // Regex
    condition:
        any of them
}
```

---

## Radare2 Quick Reference

### Analysis
```bash
r2 -A binary                          # Analyze all
r2 -AA binary                         # More analysis
r2 -c 'aaa; pdf @ main' binary        # Analyze + disasm main
r2 -c 'aaa; afl' binary               # List functions
r2 -c 'aaa; axt @ sym.main' binary    # Cross-references to main
```

### Strings and Data
```bash
r2 -c 'izz' binary                    # All strings
r2 -c 'iz' binary                     # Strings in data section
r2 -c 'px 100 @ 0x08048000' binary    # Hex dump at address
```

### Disassembly
```bash
r2 -c 'pdf @ main' binary             # Disassemble main
r2 -c 's main; pdf' binary            # Seek and disasm
r2 -qc 'aaa; s main; pdc' binary      # Pseudo-C decompile
```

---

## Windows PE Analysis

```bash
# PE headers and structure
objdump -x file.exe | head -60

# Extract strings from DLL/EXE
strings -a file.dll | grep -iE "version|copyright|company"

# Identify PE file type
file file.exe

# List imports
objdump -p file.exe | grep -A1000 "Import Tables" | head -50

# List exports (DLL)
objdump -p file.dll | grep -A1000 "Export Tables" | head -50

# DLL dependencies
objdump -p file.exe | grep "DLL Name"

# Extract PE sections
objcopy -O binary --only-section=.text file.exe text.bin
```

---

## Java Bytecode Analysis

```bash
# Decompile class file
javap -c -p MyClass.class

# Show all members (including private)
javap -p MyClass.class

# List contents of JAR
jar tf app.jar

# Extract JAR and inspect classes
unzip app.jar -d extracted/
find extracted/ -name "*.class" | xargs javap -p | head -100

# Decompile with CFR
java -jar cfr.jar app.jar --outputdir decompiled/

# Search for patterns in JAR
jar tf app.jar | grep -i config
unzip -p app.jar "*.properties" | head -50
```

---

## Python Bytecode Analysis

```bash
# Disassemble Python module
python3 -m dis module.py

# Disassemble specific function
python3 -c "import dis, module; dis.dis(module.function_name)"

# Decompile .pyc files
uncompyle6 compiled.pyc > source.py
decompyle3 compiled.pyc > source.py

# Inspect .pyc header (Python 3.8+)
python3 -c "import struct; f=open('file.pyc','rb'); magic=f.read(4); flags=struct.unpack('<I',f.read(4))[0]; print(f'magic={magic.hex()} flags={flags}')"

# Extract from __pycache__
find . -name "*.pyc" -path "*__pycache__*"
```

---

## Android APK Analysis

```bash
# Extract APK contents
apktool d app.apk -o output/

# APK metadata
aapt dump badging app.apk

# Decompile to Java source
jadx -d output/ app.apk

# Quick string search in APK
unzip -p app.apk classes.dex | strings | grep -iE 'http|api|key|secret' | head -30

# List permissions
aapt dump permissions app.apk

# Extract signing certificate
unzip -p app.apk META-INF/*.RSA | openssl pkcs7 -inform DER -print_certs
```

---

## Memory Dump Analysis (Volatility)

```bash
# Identify OS profile
vol.py -f memory.raw imageinfo

# List processes
vol.py -f memory.raw --profile=Win10x64 pslist

# Process tree
vol.py -f memory.raw --profile=Win10x64 pstree

# Scan for files
vol.py -f memory.raw --profile=Win10x64 filescan

# Dump files from memory
vol.py -f memory.raw --profile=Win10x64 dumpfiles -D output/

# Network connections
vol.py -f memory.raw --profile=Win10x64 netscan

# Command line history
vol.py -f memory.raw --profile=Win10x64 cmdscan
```

---

## Binary Diffing

```bash
# Byte-level comparison
cmp -l binary1 binary2 | head -20

# Hex diff
diff <(xxd binary1) <(xxd binary2) | head -40

# Visual binary diff
vbindiff binary1 binary2

# radare2 binary diff
radiff2 binary1 binary2

# Count differing bytes
cmp -l binary1 binary2 | wc -l
```

---

## Packer Detection and Unpacking

```bash
# Test if UPX packed
upx -t binary

# Unpack UPX
upx -d packed_binary

# Check entropy (high entropy suggests packing/encryption)
binwalk -E binary | tail -5

# Detect packer with DIE
diec binary

# Strings comparison (packed vs unpacked)
echo "Strings before: $(strings packed | wc -l)"
upx -d packed -o unpacked 2>/dev/null
echo "Strings after: $(strings unpacked | wc -l)"
```

---

## C++ Demangling

```bash
# Demangle C++ symbols from file
nm binary | c++filt | head -30

# Demangle nm output directly
nm -C binary | head -30

# Demangle single symbol
echo "_ZN4MyClass6methodEv" | c++filt

# Search for demangled class names
nm -C binary | grep "MyClass"

# Demangle from input file
c++filt < symbols.txt
```

---

## Common Workflows

### Unknown Binary Triage
```bash
file binary
strings -a binary | head -50
readelf -h binary 2>/dev/null || echo "Not ELF"
nm binary 2>/dev/null | head -20
ldd binary 2>/dev/null
```

### Find Hardcoded Secrets
```bash
strings -a binary | grep -iE 'password|secret|key|token|api'
strings -a binary | grep -E '[A-Za-z0-9+/]{40,}={0,2}'  # Base64
strings -a binary | grep -E '[0-9a-f]{32,}'              # Hex strings
strings -a binary | grep -E '-----BEGIN'                 # PEM
```

### Trace Program Behavior
```bash
strace -f -e trace=file,network ./binary 2>&1 | tee trace.log
ltrace -e '*' ./binary 2>&1 | head -100
```

### Extract from Firmware
```bash
binwalk -Me firmware.bin
find _firmware.bin.extracted -type f -exec file {} \;
strings -a _firmware.bin.extracted/* | grep -i admin
grep -r "password" _firmware.bin.extracted/
```

### Analyze Crash
```bash
dmesg | tail -20                      # Kernel messages
gdb ./binary core                     # Load core dump
# In GDB: bt, info registers, x/10i $pc
```

---

## Combined Workflows

```bash
# Automated binary triage: find sensitive strings
file * | grep ELF | cut -d: -f1 | xargs -I{} sh -c \
  'echo "=== {} ===" && strings -a {} | grep -iE "password|secret|key|api" | head -5'

# Firmware extraction and sensitive file search
binwalk -Me firmware.bin
find _firmware.bin.extracted -type f -exec file {} \; | grep -i 'text\|config' | \
  cut -d: -f1 | xargs grep -li 'password'

# Combined binary report
echo "=== Headers ===" && readelf -h binary && \
echo "=== Dynamic ===" && nm -D binary | head -20 && \
echo "=== Dependencies ===" && ldd binary

# Function size analysis (largest functions)
nm --size-sort -S binary | tail -20

# Cross-reference strings with disassembly
ADDR=$(strings -t x binary | grep "password" | awk '{print $1}')
objdump -d binary | grep -A5 "$ADDR"
```

---

## Installation

```bash
# Core tools (usually pre-installed)
sudo apt install binutils file xxd

# Analysis tools
sudo apt install binwalk strace ltrace gdb
sudo apt install patchelf

# Network tools
sudo apt install tcpdump tshark ngrep socat netcat-openbsd

# Format tools
sudo apt install poppler-utils exiftool mediainfo
sudo apt install p7zip-full unrar

# RE tools
sudo apt install radare2 yara hashid
pip install hashid
```
