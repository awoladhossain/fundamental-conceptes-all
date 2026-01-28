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
