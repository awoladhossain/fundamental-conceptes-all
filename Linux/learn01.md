# Navigation & File Operations

## Check your current location

### Learn to see where you are in the filesystem using pwd (print working directory).
```bash
pwd
```

- pwd shows your current location in the filesystem. When you open a terminal, you typically start in your home directory (/home/username on Linux or /Users/username on macOS).

**Pro Tips**
1. pwd stands for 'print working directory'
2. Think of it as 'where am I right now?'
3. Use pwd whenever you're unsure of your location

***Common Mistakes to Avoid***
1. Forgetting where you are before running commands
2. Assuming you're in a specific directory without checking


### List files in your current directory

```bash
ls
```
```bash
ls -l
```
```bash
ls -la
```

**What This Does**
ls lists directory contents. -l shows detailed info (permissions, size, date). -a shows hidden files (those starting with .). -la combines both flags.

**Expected Outcome**
First command shows visible files/folders. -l shows detailed listing with permissions and sizes. -la includes hidden files like .bashrc and .profile

***Pro Tips**
1. ls = basic list
2. ls -l = long format (detailed)
3. ls -a = all files (including hidden)
4. ls -lh = human-readable sizes (KB, MB instead of bytes)
5. Hidden files start with a dot (.)

***Common Mistakes to Avoid**
1. Not using -a and missing hidden configuration files
2. Forgetting that . is current directory and .. is parent directory


### Navigate to a different directory
```bash
cd /tmp
pwd
cd ~
pwd
cd -
pwd
```
**What This Does**
- cd changes your current directory. /tmp is an absolute path. ~ is shorthand for home directory. - returns to previous directory. Always pwd after cd to confirm location.
- First cd takes you to /tmp. cd ~ takes you home. cd - returns to /tmp. Each pwd confirms the change.

**Pro Tips**
1. cd with no arguments goes to home directory
2. cd ~ explicitly goes to home
3. cd - toggles between last two directories (super useful!)
4. cd .. goes up one level to parent directory

**Common Mistakes to Avoid**
1. Typing 'CD' or 'Cd' - Unix is case-sensitive, use lowercase
2. Forgetting the space between cd and the path
3. Not understanding the difference between / (root) and ~ (home)


### Create a practice directory

```bash
cd ~

mkdir devops-practice

ls -l

cd devops-practice

pwd
```

**What This Does**
- mkdir creates a new directory. We create it in home (~), verify with ls, then enter it with cd. pwd confirms we're inside.
- New devops-practice directory appears in ls output. pwd shows /home/username/devops-practice or similar.

**Pro Tips**
1. mkdir = 'make directory'
2. Use meaningful names with hyphens or underscores (no spaces!)
3. Check with ls before and after to see the change

**Common Mistakes to Avoid**
1. Using spaces in directory names (use-hyphens-instead)
2. Trying to create a directory that already exists (error message)
3. Creating directories in system locations without sudo

### Create nested directories
```bash
mkdir -p projects/web/frontend

ls

ls projects

ls projects/web
```

***What This Does***
- mkdir -p creates parent directories as needed. Without -p, mkdir fails if parent doesn't exist. This creates projects, then web inside it, then frontend inside web.
- projects directory created. Inside it: web directory. Inside web: frontend directory. All created with one command.

**Pro Tips**
1. -p flag means 'create parents as needed'
2. Without -p, you'd need three separate mkdir commands
3. Use -p when creating nested structures
4. -p doesn't error if directory already exists (safe to re-run)

**Common Mistakes to Avoid**
1. Forgetting -p and getting 'No such file or directory' errors
2. Not understanding the directory tree structure created

### Navigate using relative paths

```bash
pwd

cd projects/web

pwd

cd ..

pwd

cd ../..

pwd
```

***What This Does***
- Relative paths start from current location. projects/web goes down two levels. .. goes up one level. ../.. goes up two levels. No leading / means relative.
- Start in devops-practice, move to web, back to projects, back to devops-practice. pwd after each confirms.

***Pro Tips***
1.        . = current directory
2.         .. = parent directory
3.           ../.. = grandparent (two levels up)
4.            ../../other = go up two, then into 'other'
5.               Paths without leading / are relative

**Common Mistakes to Avoid**
1. Confusing absolute (/home/user/dir) with relative (dir) paths
2. Forgetting pwd and getting lost in deep directory structures


## List contents recursively
```bash
pwd
ls -R
tree
```

***What This Does***
- ls -R lists recursively (shows all subdirectories). tree shows a visual tree structure (may need installation). Both reveal the full hierarchy.
- ls -R shows all nested directories and their contents. tree (if installed) shows beautiful tree diagram.

***Pro Tips***
1. ls -R shows everything but can be overwhelming
2. tree is not built-in but super useful (install with: brew install tree or apt install tree)
3. Use tree when you want to visualize structure
4. Limit depth with: tree -L 2 (only 2 levels deep)

***Common Mistakes to Avoid***
1. Running ls -R in home directory (shows everything, takes forever!)
2. Expecting tree to be available without installing it first


### Remove directories

```bash
cd ~/devops-practice

rmdir projects/web/frontend

rmdir projects/web

rmdir projects

ls
```

***What This Does***
- rmdir removes empty directories only. Must delete from deepest level upward. rm -r removes directories and contents (dangerous!). Always use rmdir when possible for safety.
- Directories removed in order: frontend first, then web, then projects. ls shows they're gone.

***Pro Tips***
1. rmdir only works on EMPTY directories (safe)
2. rm -r removes directory AND all contents (dangerous, no undo!)
3. rm -rf is extremely dangerous (force removal, no confirmation)
4. Always double-check with pwd before rm -r
5. Practice with test directories before using on real files

### Tab completion saves time

```bash
cd ~

mkdir -p long-directory-name/with-many-levels/to-practice-tab

cd lon<TAB>

cd wit<TAB>

cd to-<TAB>

pwd
```

***What This Does***
- Press Tab after typing first few letters. Shell completes the rest. Double-Tab shows all matches if ambiguous. This prevents typos and saves time.
- Tab autocompletes each directory name. You navigate to full path by typing ~20 characters instead of 60+. Much faster!

***Pro Tips***
1. Tab is your best friend in the shell
2. Double-Tab shows all possible completions
3. Works for commands, files, directories, and options
4. Prevents typos in long paths
5. If Tab does nothing, no match exists (typo or wrong location)

***Common Mistakes to Avoid***
1. Not using Tab and typing everything manually (slow + error-prone)
2. Giving up when Tab doesn't complete (try double-Tab to see options)

### Clean up practice directory

```bash
cd ~

rm -r devops-practice

ls
```

***What This Does***
- rm -r removes directory and all contents. Safe here since it's practice files. Always pwd before rm -r to confirm location!
- devops-practice directory completely removed. ls confirms it's gone.

***Pro Tips***
1.  Always pwd before rm -r (safety check)
2. Add -i flag for confirmation: rm -ri (prompts before each deletion)
3. There is NO undo for rm (files gone forever)
4. Consider using trash/trash-cli instead of rm for safety

***Common Mistakes to Avoid***
1. Forgetting you can't undo rm (lost files forever)
2. Not checking pwd before rm -r (deleting wrong directory!)

### Create a practice workspace

```bash
cd ~
mkdir -p shell-practice/backups
cd shell-practice
pwd
```
***What This Does***
- Create a practice directory with a backups subdirectory. We'll use this safe space to practice file manipulation without risking real files.New shell-practice directory with backups folder inside. pwd confirms you're in ~/shell-practice

**Pro Tips**
1. Always practice destructive commands in a test directory first
2. Use meaningful directory names for organization

**Common Mistakes to Avoid**
- Practicing file operations in important directories (risk of data loss)

### Create empty files with touch

```bash
touch readme.txt

touch app.js config.json data.csv

ls -l
```

**What This Does**
- touch creates empty files if they don't exist. Can create multiple files in one command (space-separated). If file exists, touch updates its modification time without changing content.Four empty files created: readme.txt, app.js, config.json, data.csv. ls -l shows size 0 for all.

**Pro Tips**
1. touch is fastest way to create empty files
2. Can create multiple files: touch file1 file2 file3
3. If file exists, only updates timestamp (won't overwrite)
4. Use .txt, .md, .json extensions to indicate file type

**Common Mistakes to Avoid**
- Expecting touch to add content (it only creates empty files)
- Not using file extensions (makes files harder to identify later)

### Copy files with cp

```bash
cp readme.txt readme-backup.txt

cp app.js app-v1.js

ls -l
```

***What This Does***
- cp copies files. Syntax: cp `source` `destination`. Creates exact duplicate with new name. Original file remains unchanged. Useful for backups before editing.Two new files created: readme-backup.txt (copy of readme.txt) and app-v1.js (copy of app.js). ls shows all 6 files.


**Pro Tips**
1. Syntax: cp <source> <destination>
2. Make backups before editing: cp file.txt file.txt.backup
3. cp doesn't delete the original (unlike mv)
4. Add -i flag for confirmation: cp -i (asks before overwriting)


**Common Mistakes to Avoid**
- Reversing source and destination (cp new old instead of cp old new)
- Overwriting important files without -i flag

### Copy files to different directory

```bash
cp readme.txt backups/

cp config.json data.csv backups/

ls backups/
```

***What This Does***
- cp can copy to different directories. When destination is a directory, file keeps same name. Can copy multiple files to one directory in single command.backups/ now contains: readme.txt, config.json, data.csv. Original files still in shell-practice/.

**Pro Tips**
1. Trailing / on directory not required but makes intent clear
2. Copy multiple files: cp file1 file2 file3 destination/
3. Last argument is always the destination
4. Use -v flag to see what's being copied: cp -v file dest/


**Common Mistakes to Avoid**
1. Forgetting destination directory must exist (create with mkdir first)
2. Trying to copy multiple sources with a filename destination

### Copy directories recursively

```bash
mkdir project

touch project/index.html project/style.css

cp -r project project-backup

ls -R
```

**What This Does**
- cp -r copies directories recursively (directory + all contents). -r flag is required for directories. Creates complete duplicate of directory tree.project-backup directory created with exact copy of index.html and style.css. ls -R shows both directories with identical contents.

**Pro Tips**
1. MUST use -r flag to copy directories
2. -r means 'recursive' (includes all subdirectories)
3. Copies entire directory tree, not just top level
4. Use -a instead of -r to preserve timestamps and permissions

**Common Mistakes to Avoid**
- Forgetting -r flag when copying directories (error: omitting directory)
- Not understanding cp without -r only works for files


### Move and rename files with mv

```bash
mv readme-backup.txt README.md

ls -l

mv README.md project/

ls project/
```

***What This Does***
- mv moves OR renames files. Same syntax as cp but original disappears. When destination is different directory, file moves. When same directory, file is renamed.readme-backup.txt renamed to README.md, then moved into project/. No longer in shell-practice/.

**Pro Tips**
1. mv both moves AND renames (depending on arguments)
2. Same directory = rename: mv old.txt new.txt
3. Different directory = move: mv file.txt other-dir/
4. mv is instant (doesn't copy then delete)
5. Use -i flag to prevent accidental overwrites: mv -i


**Common Mistakes to Avoid**
- Expecting mv to create a copy (it moves, not copies)
- Accidentally overwriting existing files without -i flag
- Not realizing mv can rename (thinking it only moves)


### Move multiple files

```bash
ls

mv app.js app-v1.js config.json backups/

ls

ls backups/
```

***What This Does***
- mv can move multiple files to one directory in single command. Last argument is destination (must be existing directory). All other arguments are source files.app.js, app-v1.js, and config.json moved to backups/. No longer in shell-practice/. backups/ now has 6 files total.

**Pro Tips**
1. Last argument is always the destination
2. Destination must be a directory when moving multiple files
3. Use -v flag to see what's being moved: mv -v file1 file2 dest/
4. Faster than multiple mv commands

**Common Mistakes to Avoid**
- Last argument not being a directory (error when moving multiple files)
- Moving important files without checking destination first

### Delete files with rm

```bash
ls

rm data.csv

ls

rm backups/readme.txt backups/config.json

ls backups/
```

**What This Does**
- rm permanently deletes files. NO TRASH/RECYCLE BIN. No undo. Can delete multiple files in one command. Use with extreme caution.data.csv deleted from shell-practice/. readme.txt and config.json deleted from backups/. Files gone forever.

**Pro Tips**
1. rm is PERMANENT - no undo, no recovery
2. Use -i flag for confirmation: rm -i file (asks before deleting)
3. Can delete multiple files: rm file1 file2 file3
4. Check pwd before rm to ensure you're in correct directory
5. Consider using `trash-cli` instead of rm for safety


**Common Mistakes to Avoid**
- Deleting files without -i flag (no confirmation)
- Using rm `* without understanding what * matches` (deletes everything!)
- Expecting to recover deleted files (they're gone forever)
- Not checking pwd before rm (deleting from wrong directory)


### Understand file wildcards

```bash
touch test1.txt test2.txt test3.log demo.txt

ls *.txt

ls test*

rm test*.txt

ls
```

**What This Does**
- `* matches any characters. *.txt matches all .txt files. test* `matches anything starting with 'test'. Wildcards expand before command runs. Powerful but dangerous with rm.ls `*.txt shows all .txt files. ls test* `shows test1, test2, test3. rm test*.txt deletes test1.txt and test2.txt. demo.txt and test3.log remain.

**Pro Tips**
1. `* matches` zero or more characters
2. ? matches exactly one character
3. `*.txt` matches all text files
4. test* matches test1, test2, testing, etc.
5. ALWAYS test with ls before using with rm!
6. Double-check: ls `*.txt then rm *.txt`


**Common Mistakes to Avoid**
- Using rm * without understanding what it matches (deletes everything!)
- Not testing wildcard with ls before rm (accidental deletion)
- Spaces around *: rm * .txt deletes ALL files plus errors on .txt


### Practice safe file operations


```bash
ls -li backups/

cp -i backups/app.js backups/app-v1.js

rm -i test3.log

rm -i demo.txt
```

**What This Does**
-i flag adds interactive mode (asks confirmation). Use with cp, mv, rm to prevent accidental overwrites/deletions. Respond y (yes) or n (no) to each prompt.cp -i prompts if app-v1.js exists. rm -i asks confirmation before deleting each file. Gives you chance to say no.

**Pro Tips**
1. Always use -i flag when learning: rm -i, cp -i, mv -i
2. Create shell aliases: alias rm='rm -i'
3. Test wildcards with ls before rm
4. Keep backups of important files
5. pwd before destructive operations
6. There is NO undo in terminal

**Common Mistakes to Avoid**
- Getting comfortable and skipping -i flag
- Pressing 'y' without reading the prompt
- Assuming deleted files can be recovered (they can't)


### Clean up practice workspace

```bash
cd ~

rm -r shell-practice

ls
```

**What This Does**
rm -r removes directory and all contents recursively. Use for directories. Always pwd first to confirm location.shell-practice directory and everything inside deleted. Clean slate.

**Pro Tips**
1. rm -r deletes directories + contents
2. rm -rf forces deletion without confirmation (VERY dangerous)
3. Always check pwd before rm -r
4. Consider keeping practice directories for future reference


**Common Mistakes to Avoid**
- Using rm -rf habitually (forces deletion, no safety net)
- Running rm -r in wrong directory (data loss)

### Set up practice files with content

```bash
cd ~

mkdir search-practice && cd search-practice

echo 'Hello World' > hello.txt

echo -e 'Line 1\nLine 2\nLine 3\nERROR: Something failed\nLine 5' > log.txt

echo -e 'apple\nbanana\ncherry\ndate\nelderberry' > fruits.txt
```

***What This Does***
echo writes text to files. > creates new file. -e enables escape sequences like \n (newline). We're creating test files with known content to practice viewing and searching.Three files created: hello.txt (1 line), log.txt (5 lines with ERROR), fruits.txt (5 lines of fruit names).


**Pro Tips**
1. echo writes to stdout (screen) by default
2. `> redirects output to file (creates or overwrites)`
3. `>> appends to file instead of overwriting`
4. `-e flag interprets \n as newline`

**Common Mistakes to Avoid**
- Using > instead of >> and accidentally overwriting files
- Forgetting quotes around text with spaces
