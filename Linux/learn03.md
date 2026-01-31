# Permissions & Ownership

### Set up practice files
```bash
cd ~
mkdir perms-practice && cd perms-practice
touch script.sh config.conf data.txt
ls -l
```
**What This Does**
- ls -l shows detailed file information including permissions. First column shows permissions string. We'll learn to read and modify these.Three empty files created. ls -l shows permission string like -rw-r--r-- for each file, plus owner, group, size, date.

**Pro Tips**
1. ls -l shows long format with permissions
2. First character: - = file, d = directory, l = link
3. Next 9 characters: permissions (rwxrwxrwx)
4. Then: owner, group, size, date, filename

**Common Mistakes to Avoid**
- Not understanding what ls -l output means
- Ignoring permissions when they cause 'Permission denied' errors

### Understand permission string format

```bash
ls -l script.sh
```

**What This Does**
- Permission string has 10 characters: [type][owner perms][group perms][other perms]. Example: -rwxr-xr-- means file(-), owner can read+write+execute(rwx), group can read+execute(r-x), others can read only(r--). r=read, w=write, x=execute, -=no permission.

- You see something like -rw-r--r--. Break it down: - (file), rw- (owner can read/write), r-- (group can read), r-- (others can read).


**Pro Tips**
1. 10 characters total: [type][owner][group][other]
2. r = read (4), w = write (2), x = execute (1)
3. Owner = you (file creator)
4. Group = users in same group
5. Other = everyone else
6. Directory needs x to cd into it

**Common Mistakes to Avoid**
- Confusing owner/group/other sections
- Not knowing x is needed to execute scripts AND enter directories
- Reading permissions right-to-left (it's left-to-right)

### Make a script executable

```bash
echo '#!/bin/bash' > script.sh
echo 'echo "Hello from script"' >> script.sh
./script.sh
chmod +x script.sh
./script.sh
ls -l script.sh
```

**What This Does**
- chmod changes permissions. +x adds execute permission for everyone. Before chmod, script can't run (Permission denied). After, it executes. ls -l shows x permission added.

- First ./script.sh fails with 'Permission denied'. chmod +x makes it executable. Second ./script.sh runs successfully, prints 'Hello from script'. ls -l shows -rwxr-xr-x (x added).


**Pro Tips**
1. chmod +x adds execute for owner, group, and other
2. +x is shorthand for making scripts runnable
3. `./script.sh` runs script in current directory
4. `#!/bin/bash` is shebang (tells system which interpreter to use)
5. Scripts need both r and x permissions to run

**Common Mistakes to Avoid**
- Trying to run script without execute permission (Permission denied)
- Typing script.sh instead of ./script.sh (command not found)
- Not understanding ./ means current directory

### Remove write permission

```bash
chmod -w config.conf

ls -l config.conf

echo 'test' > config.conf

chmod +w config.conf

echo 'test' > config.conf

ls -l config.conf
```

**What This Does**
- chmod -w removes write permission. File becomes read-only (protection against accidental changes). chmod +w restores write permission. Useful for configuration files you want to protect.
- `-w` makes file read-only (r--r--r--). Trying to write fails with 'Permission denied'. +w restores write (rw-r--r--). Write succeeds.


**Pro Tips**
1. `+ adds permission, - removes permission`
2. chmod -w makes read-only (can't modify or delete)
3. chmod +w makes writable again
4. Useful for: protecting production configs, preventing accidental rm
5. Owner can still chmod even if file is read-only

**Common Mistakes to Avoid**
- Removing write and not knowing how to restore it
- Thinking read-only prevents deletion (directory permissions matter too)

### Set specific user/group permissions

```bash
chmod u+x,g-w,o-r data.txt

ls -l data.txt

chmod u=rw,g=r,o= config.conf

ls -l config.conf
```

**What This Does**
- u=owner, g=group, o=other. u+x adds execute for owner. g-w removes write for group. o-r removes read for others. Comma separates multiple changes. = sets exact permissions (replacing existing).

- First command modifies data.txt permissions specifically for each category. Second command sets config.conf to: owner read+write, group read-only, others no permissions.

**Pro Tips**
1. u = user/owner, g = group, o = other, a = all
2. `+ adds, - removes, = sets exactly`
3. Combine with commas: chmod u+x,g+x,o-r file
4. chmod u=rw,g=r,o= means owner rw, group r, other nothing
5. Most secure: owner only (u=rw,g=,o=)

**Common Mistakes to Avoid**
- Spaces around commas: chmod u+x, g+x (wrong! no spaces)
- Confusing = (set) with + (add) and - (remove)
- Not knowing o= removes all permissions for others


### Use numeric (octal) permissions

```bash
chmod 755 script.sh
ls -l script.sh
chmod 644 config.conf
ls -l config.conf
chmod 600 data.txt
ls -l data.txt
```

**What This Does**
- Numeric format: 3 digits for owner/group/other. Each digit is sum: r=4, w=2, x=1. 755 = rwxr-xr-x (7=4+2+1, 5=4+1, 5=4+1). 644 = rw-r--r--. 600 = rw------- (owner only). Faster than symbolic mode.

- 755 makes script executable by all (rwxr-xr-x). 644 makes config readable by all, writable by owner (rw-r--r--). 600 makes data private to owner (rw-------).


**Pro Tips**
1. r=4, w=2, x=1 (add them up)
2. 7 = rwx (4+2+1)
3. 6 = rw- (4+2)
4. 5 = r-x (4+1)
5. 4 = r-- (4)
6. 0 = --- (none)
7. Common: 755 (scripts), 644 (files), 600 (secrets), 700 (private dirs)

***Common Mistakes to Avoid***
- Not understanding how to calculate: 6=rw, 4=r, 0=none
- Using 777 (dangerous! everyone can do everything)
- Using 666 on executables (they need x permission)

### Common permission patterns

```bash
chmod 755 script.sh

chmod 644 config.conf

chmod 600 data.txt

ls -l
```

**What This Does**
- Best practices: 755 for scripts (owner can modify, all can execute). 644 for config/text files (owner can modify, all can read). 600 for sensitive data (owner only). 700 for private directories.
- script.sh: rwxr-xr-x (755). config.conf: rw-r--r-- (644). data.txt: rw------- (600). Each has appropriate security level.

**Pro Tips**
1. Scripts/binaries: 755 (rwxr-xr-x)
2. Config files: 644 (rw-r--r--)
3. Secrets/keys: 600 (rw-------)
4. Directories: 755 (rwxr-xr-x)
5. Private dirs: 700 (rwx------)
6. Never: 777 (security risk!)

**Common Mistakes to Avoid**
- Using 777 on everything (major security hole)
- Not securing SSH keys (should be 600)
- Not making scripts executable (should be 755)


### Understand file ownership
```bash
ls -l
whoami
groups
id
```

**What This Does**
- ls -l shows owner and group (columns 3 and 4). whoami shows your username. groups shows groups you belong to. id shows user ID (UID) and group IDs (GIDs). Ownership determines who can modify permissions.

- ls -l shows your username as owner. whoami confirms your username. groups lists your groups. id shows numeric IDs.


**Pro Tips**
1. Every file has an owner and a group
2. Owner can always change permissions (even if read-only)
3. Root user can change any file's permissions
4. Groups allow sharing files among team members
5. User IDs (UID) start at 1000 for regular users

**Common Mistakes to Avoid**
- Not understanding owner vs group vs other
- Trying to modify files you don't own (Permission denied)


### Directory permissions

```bash
mkdir testdir

chmod 644 testdir

ls testdir

cd testdir

chmod 755 testdir

cd testdir

pwd
```

**What This Does**
- Directories need different permissions. r = list contents (ls). w = create/delete files inside. x = enter directory (cd). Without x, can't cd even if you have r. 755 is standard for directories.

- 644 on directory allows ls but not cd (Permission denied). 755 allows both ls and cd. You successfully enter testdir after chmod 755

**Pro Tips**
1. Directories MUST have x to enter them (cd)
2. Directory r = list files (ls)
3. Directory w = create/delete files inside
4. Directory x = enter/traverse (cd)
5. Standard dir permissions: 755 (rwxr-xr-x)
6. Private dirs: 700 (rwx------)


**Common Mistakes to Avoid**
- Setting directory to 644 (can't cd into it)
- Not understanding x means 'traverse' for directories
- Using file permissions on directories


### Recursive permission changes

```bash
cd ~/perms-practice

mkdir -p project/{src,tests,docs}

touch project/src/app.js project/tests/test.js project/docs/readme.md

ls -lR project

chmod -R 755 project

ls -lR project
```
**What This Does**
- chmod -R applies permissions recursively (directory + all contents). Useful for setting permissions on entire project trees. -R means recursive. Changes all files and subdirectories.
- Project tree created with default permissions. chmod -R 755 sets rwxr-xr-x on all directories and files. ls -lR shows changed permissions throughout tree.


**Pro Tips**
1. chmod -R applies to everything inside
2. Use for: project directories, website files, shared folders
3. Be careful: chmod -R 777 is dangerous (security risk)
4. Common: chmod -R 755 for web directories
5. Common: chmod -R 600 for sensitive config directories


**Common Mistakes to Avoid**
- Using chmod -R 777 (massive security hole)
- Not understanding -R changes EVERYTHING inside
- Recursively removing execute on directories (breaks cd)


### Clean up and review

```bash
cd ~
chmod -R 755 perms-practice
rm -r perms-practice
ls
```

**What This Does**
- Set 755 to ensure we can delete everything (write permission). rm -r removes directory tree. Good practice: check permissions before rm if deletion fails.

- perms-practice directory and all contents removed. Clean workspace.


**Pro Tips**

1. If rm fails, check permissions with ls -l
2. chmod -R 755 makes everything deletable
3. pwd before rm -r (safety check)
4. Remember: 755 scripts, 644 files, 600 secrets, 700 private


**Common Mistakes to Avoid**
-  Can't delete files due to permissions (need write on directory)
- Not reviewing permission fundamentals before moving on
