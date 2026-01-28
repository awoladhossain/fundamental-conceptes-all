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
