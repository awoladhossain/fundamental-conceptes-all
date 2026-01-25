# Linux

### Follow log files in real-time

```bash
tail -f log.txt &

echo 'New log entry' >> log.txt

echo 'Another entry' >> log.txt

echo 'ERROR: Connection failed' >> log.txt

jobs

kill %1
```

**What This Does**
- tail -f follows file updates in real-time. New lines appear as they're written. Essential for monitoring logs. & runs in background. kill %1 stops it.tail -f shows initial content, then new lines appear as you echo them. Simulates real-time log monitoring. kill stops the tail process.

**Pro Tips**
1. tail -f monitors logs continuously (Ctrl+C to stop)
2. & runs command in background
3. jobs shows background processes
4. kill %1 stops job #1
5. Use for: tail -f /var/log/syslog (system logs)
6. tail -F follows even if file is rotated/recreated

**Common Mistakes to Avoid**
-  Forgetting Ctrl+C to exit tail -f (it runs forever)
- Not using -f for log monitoring (missing real-time updates)


### Search file contents with grep

```bash
grep ERROR log.txt

grep berry fruits.txt

grep -i error log.txt

grep -n ERROR log.txt

grep -v ERROR log.txt

```

**What This Does**
- grep searches file contents for patterns. -i ignores case. -n shows line numbers. -v inverts match (show non-matching lines). Powerful text filtering tool.
- First finds ERROR line. Second finds lines with 'berry'. -i finds 'ERROR' case-insensitive. -n shows line numbers. -v shows lines without ERROR.


Pro Tips
1. grep is case-sensitive by default
2. `-i for case-insensitive search`
3. `-n shows line numbers`
4. `-v inverts match (show what doesn't match)`
5. `-r searches directories recursively`
6. grep 'pattern' file.txt (quotes if pattern has spaces)


**Common Mistakes to Avoid**
- Forgetting -i and missing matches due to case
- Not quoting patterns with spaces or special characters

### Find files by name

```bash
find . -name '*.txt'

find . -name 'log*'

find ~ -name 'fruits.txt'

find . -type f

find . -type d
```

**What This Does**
- find searches directory trees. . means current directory. -name matches filenames. -type f finds files. -type d finds directories. * wildcard needs quotes to prevent shell expansion.
- First finds all .txt files in current dir. Second finds files starting with 'log'. Third searches home for fruits.txt. Fourth lists all files. Fifth lists all directories.

**Pro Tips**
1. find . starts search in current directory
2. find ~ searches entire home directory
3. `-name is case-sensitive`
4. `-iname for case-insensitive`
5. `-type f = files only`
6. `-type d = directories only`
7. ALWAYS quote patterns: '*.txt' not *.txt


**Common Mistakes to Avoid**
- Not quoting wildcards: find . -name *.txt (wrong!)
- Starting find in / (searches entire filesystem, very slow)
- Forgetting . means current directory


### Find files by name

```bash
find . -name '*.txt'

find . -name 'log*'

find ~ -name 'fruits.txt'

find . -type f

find . -type d
```

**What This Does**
- find searches directory trees. . means current directory. -name matches filenames. -type f finds files. -type d finds directories. * wildcard needs quotes to prevent shell expansion.
- First finds all .txt files in current dir. Second finds files starting with 'log'. Third searches home for fruits.txt. Fourth lists all files. Fifth lists all directories.


**Pro Tips**
1. find . starts search in current directory
2. find ~ searches entire home directory
3. `-name is case-sensitive`
4. `-iname for case-insensitive`
5. `-type f = files only`
6. `-type d = directories only`
7. ALWAYS quote patterns: '*.txt' not *.txt

**Common Mistakes to Avoid**
- Not quoting wildcards: find . -name *.txt (wrong!)
- Starting find in / (searches entire filesystem, very slow)
- Forgetting . means current directory

