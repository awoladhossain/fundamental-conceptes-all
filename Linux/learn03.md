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
