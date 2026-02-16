# Unix Commands Guide — From Zero to Practical

A hands-on guide to Unix commands, organized from fundamentals to real-world usage.
Every command includes actual execution output so you can see exactly what to expect.

## Table of Contents

1. [File & Directory Management](#1-file--directory-management)
   - [1.1 Navigating the Filesystem](#11-navigating-the-filesystem)
   - [1.2 Creating Files & Directories](#12-creating-files--directories)
   - [1.3 Copying, Moving & Deleting](#13-copying-moving--deleting)
   - [1.4 Viewing File Content](#14-viewing-file-content)
   - [1.5 Finding Files](#15-finding-files)
   - [1.6 File Permissions](#16-file-permissions)
   - [1.7 Links](#17-links)
   - [1.8 Disk Usage](#18-disk-usage)
   - [1.9 Archive & Compression](#19-archive--compression)
   - [1.10 File Comparison](#110-file-comparison)
2. [Text Processing](#2-text-processing)
   - [2.1 Basic Text Tools](#21-basic-text-tools)
   - [2.2 grep — Pattern Searching](#22-grep--pattern-searching)
   - [2.3 sed — Stream Editor](#23-sed--stream-editor)
   - [2.4 awk — Pattern Scanning & Processing](#24-awk--pattern-scanning--processing)
   - [2.5 Pipes & Redirection](#25-pipes--redirection)
   - [2.6 Regular Expressions](#26-regular-expressions)
3. [Process & System Management](#3-process--system-management)
   - [3.1 Viewing Processes](#31-viewing-processes)
   - [3.2 Managing Processes](#32-managing-processes)
   - [3.3 System Information](#33-system-information)
   - [3.4 Networking Basics](#34-networking-basics)
   - [3.5 SSH & Remote Operations](#35-ssh--remote-operations)
   - [3.6 Environment Variables](#36-environment-variables)
   - [3.7 Scheduling Tasks](#37-scheduling-tasks)
4. [Shell Scripting](#4-shell-scripting)
   - [4.1 Getting Started](#41-getting-started)
   - [4.2 Variables & Parameter Expansion](#42-variables--parameter-expansion)
   - [4.3 Conditionals](#43-conditionals)
   - [4.4 Loops](#44-loops)
   - [4.5 Functions](#45-functions)
   - [4.6 Error Handling](#46-error-handling)
   - [4.7 Practical Script Examples](#47-practical-script-examples)
   - [4.8 Debugging Scripts](#48-debugging-scripts)
5. [Common Pitfalls](#5-common-pitfalls)
6. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
7. [Tips for Daily Use](#tips-for-daily-use)

---

## 1. File & Directory Management

### 1.1 Navigating the Filesystem

```bash
pwd                     # Print current working directory
ls                      # List files in current directory
ls -la                  # List all files (including hidden) with details
ls -lh                  # Human-readable file sizes (KB, MB, GB)
ls -lt                  # Sort by modification time (newest first)

cd /path/to/dir         # Change to specific directory
cd ..                   # Go up one level
cd -                    # Go back to previous directory
cd                      # Go to home directory
```

**Example output:**

```
$ pwd
/home/claude/demo_project

$ ls -la
total 26
drwxr-xr-x 5 root root 4096 Feb 16 12:51 .
drwxr-xr-x 1  999 ubuntu 4096 Feb 16 12:51 ..
-rw-r--r-- 1 root root  182 Feb 16 12:51 data.csv
drwxr-xr-x 2 root root 4096 Feb 16 12:51 docs
-rw-r--r-- 1 root root  634 Feb 16 12:51 server.log
drwxr-xr-x 2 root root 4096 Feb 16 12:51 src
drwxr-xr-x 2 root root 4096 Feb 16 12:51 tests

$ ls -lh
total 18K
-rw-r--r-- 1 root root  182 Feb 16 12:51 data.csv
drwxr-xr-x 2 root root 4.0K Feb 16 12:51 docs
-rw-r--r-- 1 root root  634 Feb 16 12:51 server.log
drwxr-xr-x 2 root root 4.0K Feb 16 12:51 src
drwxr-xr-x 2 root root 4.0K Feb 16 12:51 tests
```

### 1.2 Creating Files & Directories

```bash
touch file.txt          # Create empty file (or update timestamp)
mkdir mydir             # Create directory
mkdir -p a/b/c          # Create nested directories at once

echo "hello" > file.txt         # Write (overwrites existing)
echo "world" >> file.txt        # Append to file
cat > file.txt << EOF            # Write multi-line content
line 1
line 2
EOF
```

### 1.3 Copying, Moving & Deleting

```bash
cp file.txt backup.txt          # Copy file
cp -r srcdir/ destdir/          # Copy directory recursively
cp -i file.txt dest/            # Interactive (prompt before overwrite)

mv file.txt newname.txt         # Rename file
mv file.txt /other/dir/         # Move file to another directory

rm file.txt                     # Delete file
rm -r mydir/                    # Delete directory recursively
rm -ri mydir/                   # Interactive recursive delete (safer)
rm -rf mydir/                   # Force delete without confirmation (⚠️ dangerous)
```

### 1.4 Viewing File Content

```bash
cat file.txt                    # Print entire file
head -n 20 file.txt             # First 20 lines
tail -n 20 file.txt             # Last 20 lines
tail -f logfile.log             # Follow file in real-time (great for logs)
less file.txt                   # Scrollable viewer (q to quit)
wc -l file.txt                  # Count lines
wc -w file.txt                  # Count words
```

**Example output:**

```
$ cat src/index.ts
console.log('hello world');

$ head -n 3 server.log
2025-02-16 09:01:12 [INFO] Server started on port 3000
2025-02-16 09:01:13 [INFO] Connected to database
2025-02-16 09:02:45 [WARN] Slow query detected: 2340ms

$ tail -n 3 server.log
2025-02-16 09:07:00 [INFO] Cron job executed: cleanup
2025-02-16 09:08:15 [INFO] User login: admin
2025-02-16 09:09:00 [WARN] Deprecated API called: /v1/users

$ wc -l src/*.ts src/*.tsx
  1 src/index.ts
  1 src/utils.ts
  6 src/App.tsx
  8 total
```

### 1.5 Finding Files

```bash
find . -name "*.js"                      # Find all .js files from current dir
find . -name "*.log" -mtime -7           # .log files modified in last 7 days
find . -type d -name "node_modules"      # Find directories named node_modules
find . -size +100M                       # Files larger than 100MB
find . -empty                            # Find empty files and directories
find . -newer reference.txt              # Files modified after reference.txt

find . -name "*.tmp" -delete             # Delete all .tmp files
find . -name "*.js" -exec wc -l {} +    # Count lines in all .js files

# Exclude directories from search
find . -path ./node_modules -prune -o -name "*.js" -print

which node                               # Full path of a command
whereis git                              # Locate binary, source, and man page
```

**Example output:**

```
$ find . -name "*.ts"
./src/index.ts
./src/utils.ts
./tests/test.ts

$ find . -name "*.tsx"
./src/App.tsx

$ find . -type d
.
./src
./docs
./tests
```

### 1.6 File Permissions

```bash
# Permission format: rwxrwxrwx (owner/group/others)
# r=4, w=2, x=1

chmod 755 script.sh         # Owner: rwx, Group: r-x, Others: r-x
chmod +x script.sh          # Add execute permission for everyone
chmod u+w file.txt          # Add write permission for owner only
chmod -R 644 ./src          # Apply recursively

chown user:group file.txt   # Change owner and group
chown -R user:group dir/    # Change ownership recursively
```

**Permission bit breakdown:**

```
 -rwxr-xr-x
 │├┤├┤├┤
 │ │ │ └── others: r-x (read + execute) = 5
 │ │ └──── group:  r-x (read + execute) = 5
 │ └────── owner:  rwx (read + write + execute) = 7
 └──────── type:   - (regular file), d (directory), l (symlink)

 Common patterns:
   755 = rwxr-xr-x  (executables, scripts)
   644 = rw-r--r--  (regular files)
   700 = rwx------  (private scripts)
   600 = rw-------  (private files, SSH keys)
```

**Example output:**

```
$ ls -l run.sh               # Before
-rw-r--r-- 1 root root 28 Feb 16 12:52 run.sh

$ chmod +x run.sh

$ ls -l run.sh               # After: x bit added
-rwxr-xr-x 1 root root 28 Feb 16 12:52 run.sh
```

### 1.7 Links

```bash
ln -s /path/to/original symlink_name    # Create symbolic link (shortcut)
ln original hardlink_name               # Create hard link (same inode)
readlink -f symlink_name                # Resolve full path of symlink
```

### 1.8 Disk Usage

```bash
df -h                   # Disk space usage (all mounted filesystems)
du -sh ./src             # Total size of a directory
du -sh ./* | sort -hr    # Size of each item, sorted largest first
```

**Example output:**

```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
none            9.9G  2.2M  9.9G   1% /
none            315G     0  315G   0% /dev

$ du -sh ./*
512     ./data.csv
4.5K    ./docs
1.0K    ./server.log
5.5K    ./src
4.5K    ./tests
```

### 1.9 Archive & Compression

```bash
# tar — tape archive (the standard for Unix)
tar -cf archive.tar dir/              # Create archive (no compression)
tar -czf archive.tar.gz dir/         # Create with gzip compression
tar -cjf archive.tar.bz2 dir/       # Create with bzip2 compression
tar -tf archive.tar.gz               # List contents without extracting
tar -xzf archive.tar.gz              # Extract gzipped archive
tar -xzf archive.tar.gz -C /dest/   # Extract to specific directory

# Exclude patterns
tar -czf backup.tar.gz --exclude=node_modules --exclude=.git ./project

# Compression tools (standalone)
gzip file.txt                # Compress → file.txt.gz (removes original)
gzip -k file.txt             # Compress and keep original
gunzip file.txt.gz           # Decompress
gzip -l file.txt.gz          # Show compression ratio

zip -r archive.zip dir/      # Create zip archive
unzip archive.zip             # Extract zip archive
unzip -l archive.zip          # List contents

# rsync — smart file sync (only transfers differences)
rsync -av src/ dest/                          # Local sync
rsync -av --delete src/ dest/                 # Sync and remove extra files in dest
rsync -avz src/ user@host:/path/              # Sync to remote server
rsync -av --exclude='node_modules' src/ dest/ # Sync with exclusions
```

**Example output:**

```
$ tar -czf project-backup.tar.gz --exclude=node_modules ./src ./docs
$ tar -tf project-backup.tar.gz
./src/
./src/index.ts
./src/utils.ts
./src/App.tsx
./docs/
./docs/README.md

$ gzip -l server.log.gz
         compressed        uncompressed  ratio uncompressed_name
                298                 634  55.5% server.log
```

### 1.10 File Comparison

```bash
diff file1.txt file2.txt              # Show differences
diff -u file1.txt file2.txt           # Unified format (most readable)
diff -r dir1/ dir2/                   # Compare directories recursively
diff -y file1.txt file2.txt           # Side-by-side comparison

# Generate and apply patches
diff -u original.txt modified.txt > changes.patch
patch original.txt < changes.patch    # Apply patch
patch -R original.txt < changes.patch # Reverse (undo) patch

# Compare sorted files
comm file1.txt file2.txt              # Three columns: only-in-1, only-in-2, both
comm -12 file1.txt file2.txt          # Lines common to both files
comm -23 file1.txt file2.txt          # Lines only in file1

# Byte-level comparison
cmp file1 file2                       # First difference only
```

**Example output:**

```
$ diff -u src/old.ts src/new.ts
--- src/old.ts  2025-02-16 09:00:00
+++ src/new.ts  2025-02-16 10:00:00
@@ -1,3 +1,4 @@
 import { utils } from './utils';
-const PORT = 3000;
+const PORT = process.env.PORT || 3000;
+const HOST = '0.0.0.0';
 console.log(`Listening on ${PORT}`);

$ comm <(sort list1.txt) <(sort list2.txt)
        alice
    bob
charlie
        diana
```

> `comm` output has three columns separated by tabs: lines only in file1, lines only in file2, lines in both. The indentation tells you which column each line belongs to.

---

## 2. Text Processing

### 2.1 Basic Text Tools

```bash
sort data.txt                   # Sort lines alphabetically
sort -n data.txt                # Sort numerically
sort -u data.txt                # Sort and remove duplicates
sort data.txt | uniq -c         # Count occurrences of each line

cut -d',' -f1,3 data.csv       # Extract columns 1 and 3 (comma-delimited)
cut -c1-10 file.txt             # Extract first 10 characters per line

tr 'a-z' 'A-Z' < file.txt      # Convert lowercase to uppercase
tr -d '\r' < file.txt           # Remove carriage returns (Windows to Unix)
```

### 2.2 grep — Pattern Searching

```bash
grep "error" logfile.log                # Find lines containing "error"
grep -i "error" logfile.log             # Case-insensitive search
grep -r "TODO" ./src                    # Recursive search in directory
grep -n "function" app.js              # Show line numbers
grep -c "import" app.js                # Count matching lines
grep -l "useState" ./src/*.tsx          # List filenames only
grep -v "debug" logfile.log            # Invert match (exclude lines)
grep -A 3 -B 1 "error" log.txt        # Show 3 lines After, 1 Before match
grep -C 2 "error" log.txt             # 2 lines of Context (before and after)

grep -E "error|warn|fatal" log.txt     # Match any of multiple patterns (extended regex)
grep -E "^import" app.js               # Lines starting with "import"
grep -F "exact[string]" file.txt       # Fixed string match (no regex, faster)

grep -rn "console\.log" ./src --include="*.ts"
grep -rn "TODO\|FIXME" ./src
```

**Example output:**

```
$ grep "ERROR" server.log
2025-02-16 09:03:01 [ERROR] Failed to fetch user: timeout
2025-02-16 09:03:05 [ERROR] Connection refused: ECONNREFUSED
2025-02-16 09:06:33 [ERROR] Unhandled promise rejection

$ grep -n "WARN" server.log
3:2025-02-16 09:02:45 [WARN] Slow query detected: 2340ms
8:2025-02-16 09:05:12 [WARN] Memory usage above 80%
12:2025-02-16 09:09:00 [WARN] Deprecated API called: /v1/users

$ grep -c "INFO" server.log
6
$ grep -c "ERROR" server.log
3
$ grep -c "WARN" server.log
3

$ grep -v "INFO" server.log
2025-02-16 09:02:45 [WARN] Slow query detected: 2340ms
2025-02-16 09:03:01 [ERROR] Failed to fetch user: timeout
2025-02-16 09:03:05 [ERROR] Connection refused: ECONNREFUSED
2025-02-16 09:05:12 [WARN] Memory usage above 80%
2025-02-16 09:06:33 [ERROR] Unhandled promise rejection
2025-02-16 09:09:00 [WARN] Deprecated API called: /v1/users

$ grep -rn "TODO\|FIXME" ./src/
./src/App.tsx:1:// TODO: fix this component
./src/App.tsx:3:// FIXME: performance issue

$ grep -rn "console.log" ./src/
./src/index.ts:1:console.log('hello world');
./src/App.tsx:5:  console.log('debug');
```

### 2.3 sed — Stream Editor

```bash
sed 's/old/new/' file.txt               # Replace first occurrence per line
sed 's/old/new/g' file.txt              # Replace ALL occurrences (global)
sed -i 's/old/new/g' file.txt           # Edit file in-place (⚠️ modifies file)
sed -i.bak 's/old/new/g' file.txt      # In-place with backup (.bak)

sed -n '5p' file.txt                    # Print only line 5
sed -n '10,20p' file.txt               # Print lines 10 through 20
sed '3d' file.txt                       # Delete line 3
sed '/^#/d' file.txt                    # Delete lines starting with #
sed '/^$/d' file.txt                    # Delete empty lines

sed '3i\New line here' file.txt         # Insert before line 3
sed '3a\New line here' file.txt         # Append after line 3
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt   # Multiple operations

sed 's/http:\/\//https:\/\//g' urls.txt         # HTTP to HTTPS
sed -n '/START/,/END/p' config.txt              # Print between markers
sed 's/^/  /' file.txt                           # Add 2-space indent
```

> **BSD vs GNU sed:** macOS ships BSD sed, which requires `sed -i '' 's/...'` (empty string argument after `-i`). GNU sed (Linux) uses `sed -i 's/...'` without the extra argument.

**Example output:**

```
$ cat src/index.ts
console.log('hello world');

$ sed 's/hello/goodbye/' src/index.ts
console.log('goodbye world');

$ sed -n '5p' server.log
2025-02-16 09:03:02 [INFO] Retrying request...

$ sed -n '3,6p' server.log
2025-02-16 09:02:45 [WARN] Slow query detected: 2340ms
2025-02-16 09:03:01 [ERROR] Failed to fetch user: timeout
2025-02-16 09:03:02 [INFO] Retrying request...
2025-02-16 09:03:05 [ERROR] Connection refused: ECONNREFUSED

$ sed '/^\/\//d' src/App.tsx
import React from 'react';
export default function App() {
  console.log('debug');
  return <div>Hello</div>;
}

$ sed 's/^/    /' src/utils.ts
    export function add(a, b) { return a + b; }
```

### 2.4 awk — Pattern Scanning & Processing

```bash
awk '{print $1}' file.txt               # First column
awk '{print $1, $3}' file.txt           # First and third columns
awk '{print NR, $0}' file.txt           # Line number + entire line

awk -F',' '{print $2}' data.csv         # Second column of CSV
awk -F':' '{print $1}' /etc/passwd      # Usernames from passwd

awk '$3 > 100' data.txt                 # Lines where column 3 > 100
awk '/error/' logfile.log               # Lines containing "error"

awk '{sum += $1} END {print sum}' nums.txt              # Sum
awk '{sum += $1; count++} END {print sum/count}' data    # Average
awk 'BEGIN {max=0} $1>max {max=$1} END {print max}'      # Max

awk -F',' '{printf "%-20s %10d\n", $1, $2}' data.csv    # Formatted
```

**Example output:**

```
$ cat data.csv
name,age,role,salary
Alice,28,frontend,85000
Bob,35,backend,92000
Charlie,42,devops,98000
Diana,31,frontend,88000
Eve,26,intern,45000
Frank,38,backend,95000
Grace,29,fullstack,90000

$ awk -F',' '{print $1, $3}' data.csv
name role
Alice frontend
Bob backend
Charlie devops
Diana frontend
Eve intern
Frank backend
Grace fullstack

$ awk -F',' 'NR>1 && $4 > 90000 {print $1, $3, "$"$4}' data.csv
Bob backend $92000
Charlie devops $98000
Frank backend $95000

$ awk -F',' '$3 == "frontend" {print $1, "$"$4}' data.csv
Alice $85000
Diana $88000

$ awk -F',' 'NR>1 {sum+=$4; count++} END {printf "Average: $%d (%d people)\n", sum/count, count}' data.csv
Average: $84714 (7 people)

$ awk -F'[][]' '{print $2}' server.log | sort | uniq -c | sort -rn
      6 INFO
      3 WARN
      3 ERROR

$ awk -F',' 'NR>1 {printf "%-10s | %-10s | $%6d\n", $1, $3, $4}' data.csv
Alice      | frontend   | $ 85000
Bob        | backend    | $ 92000
Charlie    | devops     | $ 98000
Diana      | frontend   | $ 88000
Eve        | intern     | $ 45000
Frank      | backend    | $ 95000
Grace      | fullstack  | $ 90000
```

### 2.5 Pipes & Redirection

**How pipes work:**

```mermaid
graph LR
    A[cmd1<br/>stdout] -->|pipe| B[cmd2<br/>stdin → stdout] -->|pipe| C[cmd3<br/>stdin → stdout]
```

```bash
ls -la | grep ".js" | wc -l            # Count .js files

command > file.txt          # Redirect stdout (overwrite)
command >> file.txt         # Redirect stdout (append)
command 2> error.log        # Redirect stderr
command > out.txt 2>&1      # Redirect both stdout and stderr
command &> all.log          # Shorthand for both (bash)

command > /dev/null 2>&1    # Silence all output

command | tee output.txt            # Display and save
command | tee -a output.txt         # Display and append
```

**File descriptors:**

```
┌──────────────────────────────────────────┐
│  FD 0 (stdin)  ← keyboard / pipe / file  │
│  FD 1 (stdout) → terminal / pipe / file  │
│  FD 2 (stderr) → terminal / file         │
└──────────────────────────────────────────┘

 cmd > file        # FD 1 → file
 cmd 2> file       # FD 2 → file
 cmd > file 2>&1   # FD 1 → file, then FD 2 → wherever FD 1 goes (= file)
 cmd 2>&1 | next   # Both FD 1 and FD 2 piped to next command
```

**xargs — build commands from stdin:**

```bash
find . -name "*.log" | xargs rm             # Delete found files
find . -name "*.ts" | xargs grep "TODO"     # Search in found files

# ⚠️ Safe xargs for filenames with spaces or special characters
find . -name "*.log" -print0 | xargs -0 rm          # Null-delimited (safe)
find . -name "*.ts" -print0 | xargs -0 grep "TODO"

# Custom placeholder
find . -name "*.bak" | xargs -I {} mv {} /backup/    # Move each file
```

> **Why `-print0 | xargs -0`?** By default, `xargs` splits on whitespace. A filename like `my file.log` would be treated as two arguments (`my` and `file.log`). Using null byte as delimiter avoids this.

**Example output:**

```
$ grep "ERROR" server.log | awk '{print $2}'
09:03:01
09:03:05
09:06:33

$ find . -name "*.ts" -o -name "*.tsx" | xargs wc -l | sort -rn
  9 total
  6 ./src/App.tsx
  1 ./tests/test.ts
  1 ./src/utils.ts
  1 ./src/index.ts

$ awk -F',' 'NR>1 {print $3}' data.csv | sort | uniq -c | sort -rn
      2 frontend
      2 backend
      1 intern
      1 fullstack
      1 devops

$ grep "ERROR" server.log > errors_only.txt
$ cat errors_only.txt
2025-02-16 09:03:01 [ERROR] Failed to fetch user: timeout
2025-02-16 09:03:05 [ERROR] Connection refused: ECONNREFUSED
2025-02-16 09:06:33 [ERROR] Unhandled promise rejection

$ grep "WARN" server.log | tee warnings.txt
2025-02-16 09:02:45 [WARN] Slow query detected: 2340ms
2025-02-16 09:05:12 [WARN] Memory usage above 80%
2025-02-16 09:09:00 [WARN] Deprecated API called: /v1/users

$ find . -name "*.ts" -o -name "*.tsx" | xargs grep -n "import"
./src/App.tsx:2:import React from 'react';
./tests/test.ts:1:import { add } from '../src/utils';
```

### 2.6 Regular Expressions

Used by `grep`, `sed`, `awk`, `find`, and most Unix tools.

**Metacharacters:**

| Symbol | Meaning | Example | Matches |
|--------|---------|---------|---------|
| `.` | Any single character | `h.t` | hat, hot, hit |
| `*` | Zero or more of previous | `ab*c` | ac, abc, abbc |
| `+` | One or more of previous (extended) | `ab+c` | abc, abbc (not ac) |
| `?` | Zero or one of previous (extended) | `colou?r` | color, colour |
| `^` | Start of line | `^import` | Lines starting with "import" |
| `$` | End of line | `\.js$` | Lines ending with ".js" |
| `[]` | Character class | `[aeiou]` | Any vowel |
| `[^]` | Negated class | `[^0-9]` | Any non-digit |
| `\b` | Word boundary | `\berror\b` | "error" but not "errors" |
| `\d` | Digit (grep -P) | `\d{3}` | Three digits |
| `()` | Group (extended) | `(ab)+` | ab, abab, ababab |
| `\|` | Or (basic) / `|` (extended) | `cat\|dog` | cat or dog |

**Basic vs Extended regex:**

```bash
# Basic regex (grep, sed) — escape +, ?, |, (), {}
grep 'error\|warn' log.txt
sed 's/\(old\)/[\1]/g' file.txt

# Extended regex (grep -E, sed -E, awk) — no escaping needed
grep -E 'error|warn' log.txt
sed -E 's/(old)/[\1]/g' file.txt
```

**Common patterns:**

```bash
grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}' log.txt     # Date YYYY-MM-DD
grep -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' file.txt  # Email
grep -E '^https?://' urls.txt                        # HTTP/HTTPS URLs
grep -E '([0-9]{1,3}\.){3}[0-9]{1,3}' log.txt      # IPv4 address
grep -E '^\s*$' file.txt                             # Blank lines (empty or whitespace)
```

---

## 3. Process & System Management

### 3.1 Viewing Processes

```bash
ps                          # Current user's processes
ps aux                      # All processes with details
ps aux | grep node          # Find node processes
ps -ef --forest             # Process tree view

top                         # Real-time process monitor (q to quit)
htop                        # Better interactive process viewer (if installed)

pgrep -f "node"             # Find PID by process name
pidof nginx                 # Get PID of a running program
```

**Example output:**

```
$ ps aux | head -5
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  1.3  0.2 487868 22636 ?        Ssl  12:51   0:01 /process_api ...
root        89 50.0  0.0  10848  2844 ?        S    12:53   0:00 /bin/sh ...
root        90  100  0.0  15996  7392 ?        R    12:53   0:00 ps aux
```

### 3.2 Managing Processes

```bash
command &                           # Run command in background
nohup command &                     # Run even after terminal closes
nohup command > output.log 2>&1 &   # Background with logging

jobs                        # List background jobs
fg %1                       # Bring job 1 to foreground
bg %1                       # Send job 1 to background
Ctrl+Z                      # Suspend current foreground process
Ctrl+C                      # Terminate current foreground process

kill PID                    # Send SIGTERM (graceful shutdown)
kill -9 PID                 # Send SIGKILL (force kill)
killall node                # Kill all processes by name
pkill -f "node server"      # Kill by pattern match
```

**Example output:**

```
$ sleep 300 &
[1] 95

$ ps aux | grep "sleep 300" | grep -v grep
root        95 25.0  0.0  10744  3432 ?        S    12:53   0:00 sleep 300

$ kill 95
[1]+  Terminated              sleep 300
```

### 3.3 System Information

```bash
uname -a                    # System info (kernel, architecture)
hostname                    # Machine hostname
uptime                      # How long system has been running
whoami                      # Current username
id                          # User ID, group ID, groups

free -h                     # Memory usage (human-readable)
lscpu                       # CPU information
lsblk                       # Block devices (disks)
```

**Example output:**

```
$ uname -a
Linux runsc 4.4.0 #1 SMP Sun Jan 10 15:06:54 PST 2016 x86_64 GNU/Linux

$ whoami
root

$ uptime
 12:53:06 up 1 min,  0 user,  load average: 0.00, 0.00, 0.00

$ free -h
               total        used        free      shared  buff/cache   available
Mem:           9.0Gi        13Mi       9.0Gi          0B       8.3Mi       9.0Gi
Swap:             0B          0B          0B
```

### 3.4 Networking Basics

```bash
curl https://example.com                  # Fetch URL content
curl -o file.zip https://example.com/f    # Download to file
curl -I https://example.com              # Headers only
curl -X POST -d '{"key":"val"}' -H 'Content-Type: application/json' URL

wget https://example.com/file.zip        # Download file
wget -q -O - URL                         # Download to stdout (quiet)

ping google.com             # Test connectivity
ping -c 4 google.com        # Send exactly 4 pings then stop
netstat -tlnp               # List listening ports
ss -tlnp                    # Modern alternative to netstat
lsof -i :3000               # What process is using port 3000
```

### 3.5 SSH & Remote Operations

```bash
# Connect to remote server
ssh user@hostname                        # Basic connection
ssh -p 2222 user@hostname                # Custom port
ssh -i ~/.ssh/mykey.pem user@hostname    # Specific key file

# Run a command on remote without opening a shell
ssh user@host "ls -la /var/log"
ssh user@host "cat /etc/nginx/nginx.conf" > local-copy.conf

# Key management
ssh-keygen -t ed25519 -C "your@email.com"   # Generate key pair (recommended type)
ssh-copy-id user@hostname                    # Copy public key to server
ssh-add ~/.ssh/id_ed25519                    # Add key to SSH agent

# Secure copy (scp)
scp file.txt user@host:/path/              # Local → remote
scp user@host:/path/file.txt ./            # Remote → local
scp -r ./dir user@host:/path/             # Copy directory recursively

# Port forwarding
ssh -L 8080:localhost:3000 user@host       # Local forwarding (access remote:3000 at localhost:8080)
ssh -R 9090:localhost:3000 user@host       # Remote forwarding (expose local:3000 on remote:9090)

# rsync over SSH (preferred over scp for large/repeated transfers)
rsync -avz -e ssh ./project/ user@host:/path/
rsync -avz --exclude='node_modules' -e ssh ./project/ user@host:/path/
```

> **scp vs rsync:** `scp` copies everything every time. `rsync` only transfers what changed. For repeated transfers or large directories, always use `rsync`.

### 3.6 Environment Variables

```bash
echo $PATH                  # Print PATH variable
echo $HOME                  # Home directory
env                         # List all environment variables
export MY_VAR="value"       # Set variable for current session
unset MY_VAR                # Remove variable

echo 'export MY_VAR="value"' >> ~/.bashrc
source ~/.bashrc            # Reload config
```

### 3.7 Scheduling Tasks

```bash
crontab -l                  # List current cron jobs
crontab -e                  # Edit cron jobs

# Cron format: minute hour day_of_month month day_of_week command
# ┌───── minute (0-59)
# │ ┌───── hour (0-23)
# │ │ ┌───── day of month (1-31)
# │ │ │ ┌───── month (1-12)
# │ │ │ │ ┌───── day of week (0-6, Sunday=0)
# │ │ │ │ │
# * * * * * command

0 9 * * *     /path/script.sh      # Every day at 9:00 AM
*/5 * * * *   /path/script.sh      # Every 5 minutes
0 0 * * 0     /path/backup.sh      # Every Sunday at midnight
30 8 1 * *    /path/report.sh      # 1st of every month at 8:30 AM

# Special strings (shortcuts)
@reboot       /path/startup.sh     # Run once at boot
@daily        /path/cleanup.sh     # Same as 0 0 * * *
@hourly       /path/check.sh       # Same as 0 * * * *
@weekly       /path/report.sh      # Same as 0 0 * * 0
@yearly       /path/audit.sh       # Same as 0 0 1 1 *
```

**Common cron pitfalls:**

```bash
# ⚠️ Cron runs with a minimal PATH — use full paths for commands
# WRONG
* * * * * node /app/script.js

# CORRECT
* * * * * /usr/local/bin/node /app/script.js

# ⚠️ Capture output for debugging — cron failures are silent by default
*/5 * * * * /path/script.sh >> /var/log/myscript.log 2>&1

# ⚠️ Environment variables from your shell profile are NOT available in cron
# Set them explicitly at the top of crontab
PATH=/usr/local/bin:/usr/bin:/bin
NODE_ENV=production
```

---

## 4. Shell Scripting

### 4.1 Getting Started

```bash
#!/bin/bash
# The shebang (first line) tells the system which interpreter to use

chmod +x script.sh      # Make script executable
./script.sh             # Run script
bash script.sh          # Alternative way to run
```

### 4.2 Variables & Parameter Expansion

```bash
#!/bin/bash

name="Claude"
count=42

echo "Hello, $name"
echo "Count is ${count} items"

# Command substitution
today=$(date +%Y-%m-%d)
file_count=$(ls | wc -l)

# Read user input
read -p "Enter your name: " username
echo "Hello, $username"

# Special variables
echo $0          # Script name
echo $1          # First argument
echo $#          # Number of arguments
echo $@          # All arguments
echo $?          # Exit status of last command
echo $$          # Current process ID
```

**Parameter expansion — manipulate variables without external commands:**

```bash
file="src/components/Header.tsx"

# Substrings & removal
${file%.tsx}           # src/components/Header     (remove shortest suffix match)
${file%%/*}            # src                       (remove longest suffix match)
${file#*/}             # components/Header.tsx     (remove shortest prefix match)
${file##*/}            # Header.tsx                (remove longest prefix match — like basename)

# Substitution
${file/src/lib}        # lib/components/Header.tsx (replace first match)
${file//o/0}           # src/c0mp0nents/Header.tsx (replace all matches)

# Default values
${PORT:-3000}          # Use 3000 if PORT is unset or empty
${PORT:=3000}          # Same, but also assign 3000 to PORT
${PORT:?"PORT is required"}  # Exit with error if PORT is unset

# Length
${#file}               # 26 (character count)

# Case conversion (bash 4+)
name="hello world"
${name^}               # Hello world (capitalize first)
${name^^}              # HELLO WORLD (all uppercase)
${name,,}              # hello world (all lowercase)
```

**Example output:**

```
$ name="hyukho"
$ today=$(date +%Y-%m-%d)
$ file_count=$(find ./src -type f | wc -l)
$ echo "Hello, $name! Today is ${today}, src has ${file_count} files."
Hello, hyukho! Today is 2026-02-16, src has 3 files.

$ path="/home/user/docs/report.pdf"
$ echo "Directory: ${path%/*}"
Directory: /home/user/docs
$ echo "Filename: ${path##*/}"
Filename: report.pdf
$ echo "No extension: ${path%.pdf}"
No extension: /home/user/docs/report
```

### 4.3 Conditionals

```bash
#!/bin/bash

if [ "$1" = "start" ]; then
    echo "Starting..."
elif [ "$1" = "stop" ]; then
    echo "Stopping..."
else
    echo "Usage: $0 {start|stop}"
    exit 1
fi

# Numeric: -eq -ne -gt -lt -ge -le
if [ "$count" -gt 10 ]; then
    echo "More than 10"
fi

# String tests
if [ -z "$var" ]; then echo "Empty"; fi
if [ -n "$var" ]; then echo "Not empty"; fi

# File tests
if [ -f "file.txt" ]; then echo "File exists"; fi
if [ -d "mydir" ]; then echo "Directory exists"; fi
if [ -x "script.sh" ]; then echo "Executable"; fi

# Logical operators
if [ "$a" -gt 0 ] && [ "$a" -lt 100 ]; then
    echo "Between 1 and 99"
fi

# Modern [[ ]] syntax (bash-specific)
if [[ "$name" == *"pattern"* ]]; then
    echo "Contains pattern"
fi
```

**Example output:**

```
$ if [ -f "server.log" ]; then echo "server.log exists"; else echo "not found"; fi
server.log exists

$ error_count=$(grep -c "ERROR" server.log)
$ if [ "$error_count" -gt 2 ]; then
>     echo "⚠️ $error_count errors found! Check needed."
> fi
⚠️ 3 errors found! Check needed.
```

### 4.4 Loops

```bash
#!/bin/bash

# for — iterate over list
for item in apple banana cherry; do
    echo "Fruit: $item"
done

# for — iterate over files
for file in *.js; do
    echo "Processing $file"
done

# for — C-style
for ((i = 0; i < 5; i++)); do
    echo "Index: $i"
done

# for — range
for i in {1..10}; do
    echo "Number: $i"
done

# while
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# while — read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# break and continue
for i in {1..10}; do
    [ $i -eq 3 ] && continue   # Skip 3
    [ $i -eq 8 ] && break      # Stop at 8
    echo $i
done
```

**Example output:**

```
$ for file in src/*; do
>     lines=$(wc -l < "$file")
>     echo "$file: ${lines} lines"
> done
src/App.tsx: 6 lines
src/index.ts: 1 lines
src/utils.ts: 1 lines

$ grep "ERROR" server.log | while IFS= read -r line; do
>     time=$(echo "$line" | awk '{print $2}')
>     msg=$(echo "$line" | sed 's/.*\] //')
>     echo "  Time: $time | Msg: $msg"
> done
  Time: 09:03:01 | Msg: Failed to fetch user: timeout
  Time: 09:03:05 | Msg: Connection refused: ECONNREFUSED
  Time: 09:06:33 | Msg: Unhandled promise rejection
```

### 4.5 Functions

```bash
#!/bin/bash

greet() {
    local name=$1
    echo "Hello, $name!"
}
greet "World"

is_file() {
    [ -f "$1" ] && return 0 || return 1
}

if is_file "test.txt"; then
    echo "File exists"
fi

get_timestamp() {
    echo $(date +%Y%m%d_%H%M%S)
}
backup_name="backup_$(get_timestamp).tar.gz"
```

**Example output:**

```
$ analyze_log() {
>     local logfile=$1
>     local level=$2
>     local count=$(grep -c "$level" "$logfile" 2>/dev/null || echo 0)
>     echo "[$level] $count entries"
> }

$ analyze_log server.log "INFO"
[INFO] 6 entries
$ analyze_log server.log "WARN"
[WARN] 3 entries
$ analyze_log server.log "ERROR"
[ERROR] 3 entries
```

### 4.6 Error Handling

```bash
#!/bin/bash

set -e              # Script stops if any command fails
set -u              # Treat unset variables as errors
set -o pipefail     # Pipe fails if any command in pipe fails
set -euo pipefail   # Combined (recommended)

# Trap — run cleanup on exit or error
cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/myapp_*
}
trap cleanup EXIT
trap cleanup ERR

# Check command exists
if ! command -v node &> /dev/null; then
    echo "Error: node is not installed" >&2
    exit 1
fi

# Default values
name=${1:-"default_name"}
```

> **`set -e` gotchas:** Commands in `if` conditions, `||` chains, and `&&` chains don't trigger exit on failure. This is by design. Use `|| true` to intentionally ignore failures: `grep "maybe" file.txt || true`

### 4.7 Practical Script Examples

#### Example 1: Project Setup Script

```bash
#!/bin/bash
set -euo pipefail

PROJECT_NAME=${1:?"Usage: $0 <project-name>"}

echo "Creating project: $PROJECT_NAME"

mkdir -p "$PROJECT_NAME"/{src,tests,docs}
touch "$PROJECT_NAME"/src/index.ts
touch "$PROJECT_NAME"/README.md

cat > "$PROJECT_NAME/package.json" << EOF
{
  "name": "$PROJECT_NAME",
  "version": "1.0.0",
  "main": "src/index.ts"
}
EOF

echo "Project $PROJECT_NAME created successfully!"
```

#### Example 2: Log Analyzer

```bash
#!/bin/bash
set -euo pipefail

LOGFILE=${1:?"Usage: $0 <logfile>"}

if [ ! -f "$LOGFILE" ]; then
    echo "Error: $LOGFILE not found" >&2
    exit 1
fi

echo "=== Log Analysis: $LOGFILE ==="
total=$(wc -l < "$LOGFILE")
errors=$(grep -c -i "error" "$LOGFILE" || true)
warnings=$(grep -c -i "warn" "$LOGFILE" || true)

echo "Total lines:  $total"
echo "Errors:       $errors"
echo "Warnings:     $warnings"

if [ "$errors" -gt 0 ]; then
    echo ""
    echo "=== Last 5 Errors ==="
    grep -i "error" "$LOGFILE" | tail -5
fi
```

#### Example 3: Project Report Generator

```bash
#!/bin/bash
set -euo pipefail

PROJECT_DIR=${1:-.}
PROJECT_NAME=$(basename "$(cd "$PROJECT_DIR" && pwd)")
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "============================================"
echo "  PROJECT REPORT: $PROJECT_NAME"
echo "  Generated: $DATE"
echo "============================================"

# File statistics
echo ""
echo "[File Statistics]"
total=$(find "$PROJECT_DIR" -type f | wc -l | tr -d ' ')
echo "  Total files: $total"

for ext in ts tsx csv log md sh; do
    count=$(find "$PROJECT_DIR" -name "*.$ext" -type f | wc -l | tr -d ' ')
    if [ "$count" -gt 0 ]; then
        echo "  .$ext files: $count"
    fi
done

# Code quality indicators
echo ""
echo "[Code Quality]"
todo=$(grep -rn "TODO\|FIXME" "$PROJECT_DIR/src" 2>/dev/null | wc -l | tr -d ' ')
logs=$(grep -rn "console.log" "$PROJECT_DIR/src" 2>/dev/null | wc -l | tr -d ' ')
echo "  TODO/FIXME: $todo"
echo "  console.log: $logs"

# Log summary (if server.log exists)
if [ -f "$PROJECT_DIR/server.log" ]; then
    echo ""
    echo "[Log Summary]"
    for level in INFO WARN ERROR; do
        count=$(grep -c "$level" "$PROJECT_DIR/server.log" || true)
        echo "  $level: $count"
    done
fi

echo ""
echo "============================================"
echo "  Report complete!"
echo "============================================"
```

**Example output:**

```
============================================
  PROJECT REPORT: demo_project
  Generated: 2026-02-16 12:53:31
============================================

[File Statistics]
  Total files: 8
  .ts files: 3
  .tsx files: 1
  .csv files: 1
  .log files: 1
  .md files: 1
  .sh files: 1

[Code Quality]
  TODO/FIXME: 2
  console.log: 2

[Log Summary]
  INFO: 6
  WARN: 3
  ERROR: 3

============================================
  Report complete!
============================================
```

### 4.8 Debugging Scripts

```bash
#!/bin/bash

# Trace mode — prints each command before executing
set -x          # Enable trace
set +x          # Disable trace

# Syntax check without executing
bash -n script.sh

# Run with trace from command line (don't modify the script)
bash -x script.sh

# Trace only a specific section
set -x
some_complex_function "$arg1" "$arg2"
set +x

# Debug with custom prefix (shows line numbers)
export PS4='+ ${BASH_SOURCE}:${LINENO}: '
set -x
```

**Example output:**

```
$ bash -x analyze.sh server.log
+ set -euo pipefail
+ LOGFILE=server.log
+ '[' '!' -f server.log ']'
+ wc -l
+ total=12
+ grep -c -i error server.log
+ errors=3
+ echo 'Total lines:  12'
Total lines:  12
+ echo 'Errors:       3'
Errors:       3
```

> `set -x` is the single most useful debugging tool for shell scripts. When something breaks, add `set -x` before the failing section and read the trace output.

---

## 5. Common Pitfalls

Things that trip up everyone at some point.

### Quoting

```bash
# ⚠️ Unquoted variables break on spaces
file="my document.txt"
cat $file        # Tries to open "my" and "document.txt" — WRONG
cat "$file"      # Opens "my document.txt" — CORRECT

# Rule of thumb: always double-quote variables
for f in "$@"; do echo "$f"; done     # CORRECT — preserves arguments
for f in $@; do echo "$f"; done       # WRONG — splits on spaces
```

### Glob Expansion

```bash
# ⚠️ Globs that match nothing behave differently across shells
ls *.xyz         # If no .xyz files: error (or literal "*.xyz" depending on shell)

# Safe pattern: check first or use nullglob
shopt -s nullglob    # bash: unmatched globs expand to nothing
for f in *.xyz; do echo "$f"; done   # Safe — loop body just doesn't execute
```

### rm Dangers

```bash
# ⚠️ Variable expansion + rm = potential disaster
dir=""
rm -rf "$dir/"       # Expands to rm -rf / — deletes everything

# Safer pattern: validate before deleting
[ -z "$dir" ] && { echo "dir is empty, aborting"; exit 1; }
rm -rf "$dir/"
```

### Pipeline Variable Scope

```bash
# ⚠️ Pipes run in subshells — variables set inside don't propagate
count=0
cat file.txt | while read line; do
    ((count++))          # This count lives in a subshell
done
echo "$count"            # Still 0!

# Fix: use process substitution or here-string
count=0
while read line; do
    ((count++))
done < file.txt
echo "$count"            # Correct count
```

### Integer Comparison vs String Comparison

```bash
# ⚠️ Using wrong comparison operator
[ "10" > "9" ]     # String comparison: "1" < "9", so this is FALSE
[ 10 -gt 9 ]       # Numeric comparison: 10 > 9, TRUE

# [ ] uses:  =, !=             for strings
#            -eq, -ne, -gt...   for numbers
# [[ ]] uses: ==, !=           for strings (with glob support)
#             -eq, -ne, -gt...  for numbers
```

### Silent Failures in Pipes

```bash
# ⚠️ Without pipefail, only the LAST command's exit code matters
false | true
echo $?      # 0 — the pipe "succeeded" even though false failed

set -o pipefail
false | true
echo $?      # 1 — now it catches the failure
```

---

## Quick Reference Cheat Sheet

| Category | Command | Description |
|----------|---------|-------------|
| **Navigate** | `cd`, `pwd`, `ls -la` | Move around, see where you are |
| **Create** | `touch`, `mkdir -p`, `echo >` | Create files and directories |
| **Copy/Move** | `cp -r`, `mv`, `rm -r` | Copy, move, delete |
| **View** | `cat`, `head`, `tail -f`, `less` | Read file content |
| **Search files** | `find . -name`, `which` | Find files by name or path |
| **Search content** | `grep -rn`, `grep -E` | Find text inside files |
| **Transform** | `sed 's/a/b/g'`, `awk '{print $1}'` | Edit and extract text |
| **Sort/Count** | `sort`, `uniq -c`, `wc -l` | Order and count things |
| **Compare** | `diff -u`, `comm`, `cmp` | Compare files |
| **Archive** | `tar -czf`, `tar -xzf`, `zip` | Compress and extract |
| **Sync** | `rsync -avz`, `scp` | Transfer and sync files |
| **Process** | `ps aux`, `kill`, `top`, `htop` | Manage running processes |
| **Disk** | `df -h`, `du -sh` | Check disk usage |
| **Network** | `curl`, `ping`, `ss -tlnp` | Network operations |
| **Remote** | `ssh`, `scp`, `rsync -e ssh` | Remote server operations |
| **Permissions** | `chmod`, `chown` | File access control |
| **Pipe** | `cmd1 \| cmd2`, `>`, `>>`, `tee` | Chain commands |
| **Schedule** | `crontab -e`, `@daily` | Automate recurring tasks |
| **Debug** | `set -x`, `bash -n` | Trace and syntax-check scripts |

---

## Tips for Daily Use

**Use `man` and `--help` often.** When you forget options, `man grep` or `grep --help` is your best friend.

**Build complex commands incrementally.** Start simple, add pipes one at a time, and verify each step before adding the next.

**Use `history` and `Ctrl+R`.** Search through previous commands instead of retyping them. `history | grep "docker"` to find past docker commands.

**Create aliases for frequent commands.** Add to `~/.bashrc` or `~/.zshrc`:
```bash
alias ll='ls -la'
alias gs='git status'
alias gd='git diff'
alias dc='docker compose'
```

**Use `!!` and `!$` shortcuts.** `sudo !!` reruns the last command with sudo. `!$` reuses the last argument.

**Know when to use `xargs -0`.** Any time filenames might contain spaces or special characters, use `find -print0 | xargs -0` instead of plain `find | xargs`.

**Quote your variables.** `"$var"` is almost always what you want. Unquoted `$var` will split on spaces and expand globs — a common source of bugs in scripts.
