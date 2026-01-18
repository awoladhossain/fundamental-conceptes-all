# Linux Commands Guide for DevOps Engineers

A comprehensive reference from beginner to advanced level commands every DevOps engineer should know.

## Table of Contents
1. [File and Directory Operations](#file-and-directory-operations)
2. [User and Group Management](#user-and-group-management)
3. [File Permissions](#file-permissions)
4. [Process Management](#process-management)
5. [Network Commands](#network-commands)
6. [Package Management](#package-management)
7. [System Monitoring](#system-monitoring)
8. [File Search and Text Processing](#file-search-and-text-processing)
9. [Compression and Archives](#compression-and-archives)
10. [System Information](#system-information)
11. [SSH and Remote Operations](#ssh-and-remote-operations)
12. [Docker Commands](#docker-commands)
13. [Git Commands](#git-commands)
14. [Advanced Shell Scripting](#advanced-shell-scripting)

---

## File and Directory Operations

### Basic Commands (You Already Know)
```bash
ls          # List directory contents
ls -la      # List all files with details (including hidden)
ls -lh      # Human-readable file sizes
mkdir       # Create directory
mkdir -p    # Create nested directories
pwd         # Print working directory
cd          # Change directory
cd ..       # Go up one level
cd ~        # Go to home directory
cd -        # Go to previous directory
```

### Essential File Operations
```bash
touch filename              # Create empty file or update timestamp
cp source dest              # Copy file
cp -r source dest           # Copy directory recursively
mv source dest              # Move or rename file/directory
rm filename                 # Remove file
rm -r directory             # Remove directory recursively
rm -rf directory            # Force remove (use carefully!)
rmdir directory             # Remove empty directory

# View file contents
cat filename                # Display entire file
less filename               # View file with pagination
head filename               # Show first 10 lines
head -n 20 filename         # Show first 20 lines
tail filename               # Show last 10 lines
tail -f filename            # Follow file updates (logs)
tail -n 50 filename         # Show last 50 lines

# File editing
nano filename               # Simple text editor
vim filename                # Advanced text editor
```

### Advanced File Operations
```bash
find /path -name "*.log"    # Find files by name
find /path -type f -mtime -7  # Files modified in last 7 days
find /path -size +100M      # Files larger than 100MB
locate filename             # Quick file search (uses database)
updatedb                    # Update locate database

ln -s source link           # Create symbolic link
ln source link              # Create hard link

tree                        # Display directory tree structure
tree -L 2                   # Limit depth to 2 levels
```

---

## User and Group Management

### User Management
```bash
# Create user
useradd username                    # Basic user creation
useradd -m username                 # Create with home directory
useradd -m -s /bin/bash username    # Specify shell
adduser username                    # Interactive user creation (Debian/Ubuntu)

# Modify user
usermod -aG groupname username      # Add user to group
usermod -l newname oldname          # Rename user
usermod -d /new/home username       # Change home directory
usermod -s /bin/bash username       # Change shell

# Delete user
userdel username                    # Delete user
userdel -r username                 # Delete user and home directory

# User information
id username                         # Display user ID and groups
whoami                              # Current user
who                                 # Who is logged in
w                                   # Who is logged in and what they're doing
last                                # Login history
```

### Password Management
```bash
passwd username                     # Change password
passwd -l username                  # Lock user account
passwd -u username                  # Unlock user account
chage -l username                   # View password aging info
chage -M 90 username                # Set password expiry to 90 days
```

### Group Management
```bash
groupadd groupname                  # Create group
groupdel groupname                  # Delete group
groups username                     # Show user's groups
gpasswd -a username groupname       # Add user to group
gpasswd -d username groupname       # Remove user from group
newgrp groupname                    # Switch to group
```

### Switch Users
```bash
su username                         # Switch user
su -                                # Switch to root
sudo command                        # Execute as superuser
sudo -u username command            # Execute as specific user
sudo -i                             # Interactive root shell
```

---

## File Permissions

### Understanding Permissions
```
Permission Format: drwxrwxrwx
- d: directory (- for file)
- rwx: owner permissions (read, write, execute)
- rwx: group permissions
- rwx: others permissions
```

### Numeric Permissions
```
4 = read (r)
2 = write (w)
1 = execute (x)
7 = rwx (4+2+1)
6 = rw- (4+2)
5 = r-x (4+1)
```

### Change Permissions
```bash
chmod 755 filename              # rwxr-xr-x
chmod 644 filename              # rw-r--r--
chmod +x filename               # Add execute permission
chmod -w filename               # Remove write permission
chmod u+x filename              # Add execute for user
chmod g-w filename              # Remove write for group
chmod o+r filename              # Add read for others
chmod -R 755 directory          # Recursive permission change
```

### Change Ownership
```bash
chown user filename             # Change owner
chown user:group filename       # Change owner and group
chown -R user:group directory   # Recursive ownership change
chgrp group filename            # Change group only
```

### Special Permissions
```bash
chmod u+s filename              # Set SUID (run as owner)
chmod g+s directory             # Set SGID (inherit group)
chmod +t directory              # Set sticky bit (only owner can delete)
chmod 4755 filename             # SUID with 755 permissions
chmod 2755 directory            # SGID with 755 permissions
chmod 1755 directory            # Sticky bit with 755 permissions
```

### View Permissions
```bash
ls -l filename                  # View file permissions
stat filename                   # Detailed file information
getfacl filename                # Get file ACL
setfacl -m u:username:rw file   # Set ACL for user
```

---

## Process Management

### View Processes
```bash
ps                              # Current shell processes
ps aux                          # All processes (detailed)
ps -ef                          # All processes (full format)
ps -u username                  # Processes by user
pstree                          # Process tree
top                             # Real-time process monitor
htop                            # Interactive process viewer (better than top)
```

### Process Control
```bash
kill PID                        # Terminate process
kill -9 PID                     # Force kill process
killall process_name            # Kill by name
pkill pattern                   # Kill by pattern
pgrep pattern                   # Find process by pattern

# Background and foreground
command &                       # Run in background
jobs                            # List background jobs
fg %1                           # Bring job 1 to foreground
bg %1                           # Resume job 1 in background
Ctrl+Z                          # Suspend current process
Ctrl+C                          # Terminate current process

nohup command &                 # Run process immune to hangups
```

### Service Management (systemd)
```bash
systemctl start service         # Start service
systemctl stop service          # Stop service
systemctl restart service       # Restart service
systemctl reload service        # Reload configuration
systemctl status service        # Check service status
systemctl enable service        # Enable at boot
systemctl disable service       # Disable at boot
systemctl list-units            # List all units
systemctl daemon-reload         # Reload systemd configuration
journalctl -u service           # View service logs
journalctl -f                   # Follow system logs
```

---

## Network Commands

### Network Configuration
```bash
ip addr show                    # Show IP addresses
ip link show                    # Show network interfaces
ip route show                   # Show routing table
ifconfig                        # Network interface configuration (older)
hostname                        # Show hostname
hostname -I                     # Show all IP addresses
```

### Network Testing
```bash
ping hostname                   # Test connectivity
ping -c 4 hostname              # Ping 4 times
traceroute hostname             # Trace route to host
mtr hostname                    # Combined ping and traceroute
nslookup domain                 # DNS lookup
dig domain                      # DNS lookup (detailed)
host domain                     # DNS lookup (simple)
```

### Network Connections
```bash
netstat -tuln                   # Show listening ports
netstat -an                     # Show all connections
ss -tuln                        # Socket statistics (modern netstat)
ss -tulnp                       # Show process using ports
lsof -i :80                     # Show what's using port 80
telnet host port                # Test port connectivity
nc -zv host port                # Port scanning with netcat
```

### File Transfer
```bash
scp file user@host:/path        # Secure copy to remote
scp user@host:/path file        # Secure copy from remote
scp -r directory user@host:/    # Copy directory
rsync -avz source dest          # Sync files (efficient)
rsync -avz -e ssh src user@host:/dest  # Rsync over SSH

wget url                        # Download file
wget -c url                     # Resume download
curl -O url                     # Download file
curl -I url                     # Get headers only
```

### Firewall Management
```bash
# UFW (Ubuntu/Debian)
ufw status                      # Check firewall status
ufw enable                      # Enable firewall
ufw allow 22                    # Allow SSH
ufw deny 80                     # Deny HTTP
ufw allow from 192.168.1.0/24   # Allow from subnet

# firewalld (CentOS/RHEL)
firewall-cmd --state            # Check firewall state
firewall-cmd --list-all         # List all rules
firewall-cmd --add-port=80/tcp  # Add port
firewall-cmd --reload           # Reload firewall

# iptables (advanced)
iptables -L                     # List rules
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```

---

## Package Management

### APT (Debian/Ubuntu)
```bash
apt update                      # Update package index
apt upgrade                     # Upgrade packages
apt full-upgrade                # Upgrade with dependency handling
apt install package             # Install package
apt remove package              # Remove package
apt purge package               # Remove package and config
apt autoremove                  # Remove unused dependencies
apt search keyword              # Search for packages
apt show package                # Show package info
apt list --installed            # List installed packages
```

### YUM/DNF (CentOS/RHEL/Fedora)
```bash
yum update                      # Update packages
yum install package             # Install package
yum remove package              # Remove package
yum search keyword              # Search packages
yum info package                # Package information
yum list installed              # List installed packages

dnf update                      # DNF (newer than yum)
dnf install package
dnf remove package
```

### Snap (Universal)
```bash
snap install package            # Install snap package
snap list                       # List installed snaps
snap remove package             # Remove snap
snap refresh                    # Update all snaps
```

---

## System Monitoring

### Resource Monitoring
```bash
top                             # Process monitor
htop                            # Better process monitor
uptime                          # System uptime and load
free -h                         # Memory usage (human-readable)
df -h                           # Disk usage
du -sh directory                # Directory size
du -h --max-depth=1             # Disk usage by subdirectory
iostat                          # CPU and I/O statistics
vmstat                          # Virtual memory statistics
mpstat                          # Multi-processor statistics
```

### Disk Operations
```bash
lsblk                           # List block devices
fdisk -l                        # List disk partitions
mount                           # Show mounted filesystems
mount /dev/sdb1 /mnt            # Mount device
umount /mnt                     # Unmount
df -i                           # Inode usage
ncdu                            # NCurses disk usage (interactive)
```

### System Logs
```bash
journalctl                      # View systemd logs
journalctl -f                   # Follow logs
journalctl -u service           # Service-specific logs
journalctl --since "1 hour ago" # Recent logs
journalctl -p err               # Error messages only
dmesg                           # Kernel ring buffer
dmesg -T                        # Human-readable timestamps
tail -f /var/log/syslog         # Follow system log
```

---

## File Search and Text Processing

### Search Inside Files
```bash
grep pattern file               # Search in file
grep -r pattern directory       # Recursive search
grep -i pattern file            # Case-insensitive
grep -v pattern file            # Invert match
grep -n pattern file            # Show line numbers
grep -c pattern file            # Count matches
grep -l pattern files           # Show filenames only
grep -E "regex" file            # Extended regex
egrep "pattern1|pattern2" file  # Multiple patterns
```

### Text Processing
```bash
sed 's/old/new/' file           # Replace first occurrence
sed 's/old/new/g' file          # Replace all occurrences
sed -i 's/old/new/g' file       # Edit file in-place
awk '{print $1}' file           # Print first column
awk -F: '{print $1}' /etc/passwd  # Custom delimiter
cut -d: -f1 /etc/passwd         # Cut by delimiter
sort file                       # Sort lines
sort -r file                    # Reverse sort
sort -n file                    # Numeric sort
uniq file                       # Remove duplicates
wc -l file                      # Count lines
wc -w file                      # Count words
tr 'a-z' 'A-Z' < file           # Translate characters
```

### Advanced Text Processing
```bash
# Column manipulation
column -t file                  # Columnate output
paste file1 file2               # Merge files line by line

# Comparison
diff file1 file2                # Compare files
comm file1 file2                # Compare sorted files
vimdiff file1 file2             # Visual diff

# Stream editor examples
sed -n '5,10p' file             # Print lines 5-10
sed '5d' file                   # Delete line 5
sed '/pattern/d' file           # Delete matching lines
```

---

## Compression and Archives

### TAR Archives
```bash
tar -cvf archive.tar files      # Create archive
tar -czvf archive.tar.gz files  # Create gzip archive
tar -cjvf archive.tar.bz2 files # Create bzip2 archive
tar -xvf archive.tar            # Extract archive
tar -xzvf archive.tar.gz        # Extract gzip archive
tar -xjvf archive.tar.bz2       # Extract bzip2 archive
tar -tvf archive.tar            # List contents
tar -xvf archive.tar -C /path   # Extract to specific directory
```

### Compression
```bash
gzip file                       # Compress file
gunzip file.gz                  # Decompress file
bzip2 file                      # Better compression
bunzip2 file.bz2                # Decompress bzip2
zip archive.zip files           # Create zip
unzip archive.zip               # Extract zip
unzip -l archive.zip            # List zip contents
```

---

## System Information

### Hardware Information
```bash
lscpu                           # CPU information
lshw                            # Hardware configuration
lspci                           # PCI devices
lsusb                           # USB devices
dmidecode                       # DMI/SMBIOS info
free -h                         # Memory info
cat /proc/cpuinfo               # Detailed CPU info
cat /proc/meminfo               # Detailed memory info
```

### System Information
```bash
uname -a                        # System information
uname -r                        # Kernel version
hostnamectl                     # System hostname info
timedatectl                     # Time and date info
cat /etc/os-release             # OS information
lsb_release -a                  # Distribution info
uptime                          # System uptime
date                            # Current date/time
cal                             # Calendar
```

---

## SSH and Remote Operations

### SSH Basics
```bash
ssh user@host                   # Connect to remote host
ssh -p 2222 user@host           # Custom port
ssh -i keyfile user@host        # Use specific key
ssh user@host command           # Execute remote command
```

### SSH Key Management
```bash
ssh-keygen                      # Generate SSH key
ssh-keygen -t rsa -b 4096       # Generate 4096-bit RSA key
ssh-copy-id user@host           # Copy key to remote host
ssh-add ~/.ssh/id_rsa           # Add key to agent
ssh-agent bash                  # Start SSH agent
```

### SSH Configuration
```bash
# ~/.ssh/config
Host myserver
    HostName 192.168.1.100
    User admin
    Port 2222
    IdentityFile ~/.ssh/mykey

# Then simply: ssh myserver
```

### SSH Tunneling
```bash
# Local port forwarding
ssh -L 8080:localhost:80 user@host

# Remote port forwarding
ssh -R 8080:localhost:80 user@host

# Dynamic port forwarding (SOCKS proxy)
ssh -D 1080 user@host
```

---

## Docker Commands

### Container Management
```bash
docker ps                       # List running containers
docker ps -a                    # List all containers
docker run image                # Run container
docker run -d image             # Run in background
docker run -it image bash       # Interactive mode
docker run -p 8080:80 image     # Port mapping
docker run -v /host:/container image  # Volume mount
docker stop container           # Stop container
docker start container          # Start container
docker restart container        # Restart container
docker rm container             # Remove container
docker rm -f container          # Force remove
docker exec -it container bash  # Execute command in container
docker logs container           # View logs
docker logs -f container        # Follow logs
```

### Image Management
```bash
docker images                   # List images
docker pull image               # Download image
docker build -t name .          # Build image
docker push image               # Upload image
docker rmi image                # Remove image
docker tag image newtag         # Tag image
docker history image            # Show image layers
```

### Docker Compose
```bash
docker-compose up               # Start services
docker-compose up -d            # Start in background
docker-compose down             # Stop services
docker-compose ps               # List services
docker-compose logs             # View logs
docker-compose logs -f service  # Follow service logs
docker-compose build            # Build services
docker-compose restart service  # Restart service
```

### System Commands
```bash
docker system df                # Show disk usage
docker system prune             # Remove unused data
docker volume ls                # List volumes
docker network ls               # List networks
docker inspect container        # Detailed container info
```

---

## Git Commands

### Repository Basics
```bash
git init                        # Initialize repository
git clone url                   # Clone repository
git status                      # Check status
git add file                    # Stage file
git add .                       # Stage all changes
git commit -m "message"         # Commit changes
git push                        # Push to remote
git pull                        # Pull from remote
git fetch                       # Fetch from remote
```

### Branching
```bash
git branch                      # List branches
git branch newbranch            # Create branch
git checkout branch             # Switch branch
git checkout -b newbranch       # Create and switch
git merge branch                # Merge branch
git branch -d branch            # Delete branch
git push origin --delete branch # Delete remote branch
```

### Advanced Operations
```bash
git log                         # View commit history
git log --oneline               # Compact history
git log --graph                 # Graph view
git diff                        # Show changes
git diff --staged               # Show staged changes
git stash                       # Stash changes
git stash pop                   # Apply stash
git reset --hard HEAD           # Discard all changes
git revert commit               # Revert commit
git cherry-pick commit          # Apply specific commit
git rebase branch               # Rebase branch
```

### Configuration
```bash
git config --global user.name "Name"
git config --global user.email "email@example.com"
git config --list               # Show configuration
git remote -v                   # Show remotes
git remote add origin url       # Add remote
```

---

## Advanced Shell Scripting

### Variables and Environment
```bash
VAR="value"                     # Set variable
echo $VAR                       # Print variable
export VAR="value"              # Environment variable
env                             # Show all environment variables
printenv                        # Print environment
unset VAR                       # Remove variable
readonly VAR="value"            # Read-only variable
```

### Conditional Statements
```bash
if [ condition ]; then
    commands
elif [ condition ]; then
    commands
else
    commands
fi

# File test operators
[ -f file ]                     # File exists
[ -d directory ]                # Directory exists
[ -r file ]                     # Readable
[ -w file ]                     # Writable
[ -x file ]                     # Executable
[ -s file ]                     # File not empty

# String comparisons
[ "$a" = "$b" ]                 # Equal
[ "$a" != "$b" ]                # Not equal
[ -z "$a" ]                     # String is empty
[ -n "$a" ]                     # String is not empty

# Numeric comparisons
[ "$a" -eq "$b" ]               # Equal
[ "$a" -ne "$b" ]               # Not equal
[ "$a" -gt "$b" ]               # Greater than
[ "$a" -lt "$b" ]               # Less than
```

### Loops
```bash
# For loop
for i in 1 2 3 4 5; do
    echo $i
done

for file in *.txt; do
    echo $file
done

# While loop
while [ condition ]; do
    commands
done

# Until loop
until [ condition ]; do
    commands
done
```

### Functions
```bash
function_name() {
    echo "Hello $1"
    return 0
}

function_name "World"           # Call function
```

### Useful Shell Tricks
```bash
command1 && command2            # Run command2 if command1 succeeds
command1 || command2            # Run command2 if command1 fails
command1 | command2             # Pipe output
command > file                  # Redirect output
command >> file                 # Append output
command 2> file                 # Redirect errors
command &> file                 # Redirect all output
command < file                  # Input from file
command1 $(command2)            # Command substitution
command1 `command2`             # Command substitution (old style)
```

### Useful One-Liners
```bash
# Find and delete files
find . -name "*.tmp" -delete

# Count files in directory
ls -1 | wc -l

# Find largest files
du -ah /path | sort -rh | head -20

# Monitor log file
tail -f /var/log/syslog | grep ERROR

# Find process using port
lsof -ti:8080

# Kill all processes by name
pkill -f process_name

# Create backup with timestamp
tar -czf backup-$(date +%Y%m%d).tar.gz /path

# Find files modified in last 24 hours
find /path -mtime -1

# Replace text in multiple files
sed -i 's/old/new/g' *.txt

# Show directory sizes sorted
du -sh */ | sort -h

# Monitor network traffic
watch -n 1 'netstat -an | grep ESTABLISHED'
```

---

## Best Practices for DevOps

1. **Always use version control** - Track all configuration changes
2. **Document everything** - Scripts should be self-documenting
3. **Test in development first** - Never test commands in production
4. **Use configuration management** - Ansible, Puppet, Chef, etc.
5. **Implement monitoring** - Know when things break
6. **Automate repetitive tasks** - Write scripts for common operations
7. **Follow principle of least privilege** - Don't use root unnecessarily
8. **Keep systems updated** - Regular patching schedule
9. **Backup regularly** - Test your backups
10. **Use strong passwords and SSH keys** - Security first

---

## Learning Path

### Beginner (1-2 months)
- File operations: ls, cd, cp, mv, rm, cat, grep
- Basic permissions: chmod, chown
- Process basics: ps, kill, top
- Package management: apt/yum install

### Intermediate (3-6 months)
- User management: useradd, usermod, groups
- Advanced permissions: ACLs, special permissions
- System services: systemctl
- Networking: ip, netstat, ssh
- Text processing: sed, awk, grep
- Shell scripting basics

### Advanced (6-12 months)
- Advanced shell scripting
- System monitoring and troubleshooting
- Container orchestration: Docker, Kubernetes
- CI/CD pipelines: Jenkins, GitLab CI
- Infrastructure as Code: Terraform, Ansible
- Log analysis and monitoring tools

---

## Quick Reference Card

### Essential Daily Commands
```bash
# Check system resources
df -h && free -h && uptime

# Find large files
du -sh /* | sort -rh | head -10

# Check active connections
ss -tulnp

# Monitor logs
journalctl -f

# Check service status
systemctl status servicename

# Quick backup
tar -czf backup-$(date +%F).tar.gz /path
```

### Emergency Commands
```bash
# Kill unresponsive process
kill -9 PID

# Free up disk space
du -sh /* | sort -rh | head
apt clean
docker system prune -a

# Check what's using disk
lsof +L1

# Network troubleshooting
ping -c 4 8.8.8.8
traceroute google.com
nslookup domain.com
```

---

## Additional Resources

- Official documentation: `man command` or `command --help`
- Online: https://linux.die.net
- Practice: Set up virtual machines or use cloud instances
- Communities: r/linux, Stack Overflow, Linux Discord servers

---

**Last Updated:** January 2025  
**Version:** 1.0  
**Author:** DevOps Learning Guide

*This guide is meant to be a living document. Update it as you learn new commands and techniques!*
