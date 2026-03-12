# 🐧 Linux Learning Notes

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [Learn 01](learn01.md) | Linux Basics | ✅ Done |
| [Learn 02](learn02.md) | Intermediate Commands | ✅ Done |
| [Learn 03](learn03.md) | Advanced Topics | ✅ Done |
| [DevOps Guide](linux_devops_guide.md) | Linux for DevOps | ✅ Done |
| [DevOps Story](linux_devops_guide_story.md) | Learning Journey | ✅ Done |

## 🎯 Quick Reference

### File Operations
```bash
# List files
ls -la

# Change directory
cd /path/to/dir

# Copy file
cp source.txt dest.txt

# Move/Rename
mv old.txt new.txt

# Remove file
rm file.txt

# Remove directory
rm -rf folder/
```

### File Permissions
```bash
# Change permissions
chmod 755 file.sh    # rwxr-xr-x
chmod +x script.sh   # Add execute

# Change owner
chown user:group file.txt
```

### Process Management
```bash
# List processes
ps aux

# Kill process
kill -9 <pid>

# Background process
command &

# View resource usage
top / htop
```

### Useful Commands
```bash
# Find file
find /path -name "*.txt"

# Search in files
grep "pattern" file.txt
grep -r "pattern" ./

# Disk usage
df -h
du -sh folder/

# System info
uname -a
cat /etc/os-release
```

## 📚 Resources
- [Linux Command Library](https://linuxcommandlibrary.com/)
- [TLDR Pages](https://tldr.sh/)

---
*Last Updated: March 2026*
