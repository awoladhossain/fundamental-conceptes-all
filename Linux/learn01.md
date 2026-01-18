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
