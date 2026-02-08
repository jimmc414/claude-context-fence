---
description: "Reverse engineering - what does this program do, suspicious executable, unknown file format, extract hardcoded secrets, binary analysis, file inspection, protocol decoding, and forensic analysis (readelf, objdump, strings, strace, binwalk, yara)"
when_to_use: "Use when: user asks what does this program do, encounters suspicious or unknown executable, wants to understand unknown file format, needs to extract hardcoded secrets or credentials, investigating malware, analyzing unknown binaries, extracting strings from executables, inspecting file formats, tracing program behavior, decoding protocols, identifying file types, examining ELF structure, capturing network traffic, firmware analysis. Triggers: what does this binary do, suspicious file, unknown format, extract secrets, hardcoded credentials, reverse engineer, binary analysis, strings, hexdump, readelf, objdump, strace, file type, disassemble, firmware, protocol decode, unknown file, forensics, analyze binary, hex dump, elf, pe file, decompile, symbols, library dependencies, memory dump, apk analysis, java bytecode, javap, apktool, volatility, yara, radare2."
when-to-use: "Use when: user asks what does this program do, encounters suspicious or unknown executable, wants to understand unknown file format, needs to extract hardcoded secrets or credentials, investigating malware, analyzing unknown binaries, extracting strings from executables, inspecting file formats, tracing program behavior, decoding protocols, identifying file types, examining ELF structure, capturing network traffic, firmware analysis. Triggers: what does this binary do, suspicious file, unknown format, extract secrets, hardcoded credentials, reverse engineer, binary analysis, strings, hexdump, readelf, objdump, strace, file type, disassemble, firmware, protocol decode, unknown file, forensics, analyze binary, hex dump, elf, pe file, decompile, symbols, library dependencies, memory dump, apk analysis, java bytecode, javap, apktool, volatility, yara, radare2."
argument-hint: "[describe what you want to analyze or investigate]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Reverse Engineering Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the reverse engineering recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Goal
What RE operation is needed?
- Identify file type, extract strings, analyze binary structure, trace execution, decode protocol, inspect firmware

### 2. Target
What are you analyzing?
- Binary executable path
- Firmware image
- Network capture file
- Unknown file
- Running process

### 3. Analysis Type
What level of analysis?
- **Static**: examine without running (strings, headers, disassembly)
- **Dynamic**: observe behavior (strace, ltrace, debugging)
- **Network**: capture and decode traffic

### 4. Specific Focus
What are you looking for?
- Hardcoded secrets/credentials
- Function names and symbols
- Library dependencies
- System calls made
- Network connections
- File format structure
- Embedded files

### 5. Output Needs
- Raw output for further processing
- Formatted for human reading
- Specific fields only

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Identify file type | file | Magic byte and MIME detection |
| Extract strings | strings | Printable character extraction |
| Binary analysis (ELF) | readelf | Headers, sections, symbols |
| Disassembly | objdump | Instruction-level disassembly |
| Symbol listing | nm | Function and variable names |
| Shared library deps | ldd | Linked library resolution |
| Firmware extraction | binwalk | Embedded file detection/extraction |
| Syscall tracing | strace | Observe runtime behavior |
| Library call tracing | ltrace | Trace shared library calls |
| Hex inspection | xxd / hexdump | Raw byte examination |
| Network traffic | tcpdump / tshark | Packet capture and decode |
| Pattern matching | yara | Malware/pattern rule matching |

---

## Argument Format

Construct a structured argument string:

```
<goal> on <target_path> - analysis: <type>, focus: <what>, output: <format>
```

**Include only relevant components.**

### Examples

**File identification:**
```
identify file type of /tmp/unknown_file - include MIME type and magic bytes
```

**String extraction:**
```
extract strings from /usr/bin/mystery - focus: passwords/credentials, minimum length: 8
```

**Binary structure:**
```
analyze ELF structure of ./suspicious_binary - show headers, symbols, dependencies
```

**Dynamic tracing:**
```
trace system calls of /opt/app/server - focus: file and network operations
```

**Firmware extraction:**
```
extract embedded files from firmware.bin - recursive extraction, identify file systems
```

**Protocol decoding:**
```
decode network traffic in capture.pcap - filter: HTTP requests, output: JSON
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-reverse-engineer-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Target file path is unclear
- Need authorization context for security analysis
- Multiple analysis approaches possible

**Use Read tool first if:**
- Need to verify file exists
- Need to check file size before analysis
