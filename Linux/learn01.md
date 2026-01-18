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

***What This Does**
- Relative paths start from current location. projects/web goes down two levels. .. goes up one level. ../.. goes up two levels. No leading / means relative.
- Start in devops-practice, move to web, back to projects, back to devops-practice. pwd after each confirms.

***Pro Tips***
1.  . = current directory
2. .. = parent directory
3. ../.. = grandparent (two levels up)
4. ../../other = go up two, then into 'other'
5. Paths without leading / are relative

**Common Mistakes to Avoid**
1. Confusing absolute (/home/user/dir) with relative (dir) paths
2. Forgetting pwd and getting lost in deep directory structures
