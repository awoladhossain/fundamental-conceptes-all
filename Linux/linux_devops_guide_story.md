# Linux Commands: Real-World DevOps Scenarios

A practical guide with real scenarios showing when and how to use Linux commands to solve actual problems.

---

## Table of Contents
1. [Day 1: Your First Day as DevOps Engineer](#day-1-your-first-day-as-devops-engineer)
2. [Week 1: Setting Up a Web Server](#week-1-setting-up-a-web-server)
3. [Week 2: The Database Crisis](#week-2-the-database-crisis)
4. [Week 3: Application Deployment](#week-3-application-deployment)
5. [Month 2: Production Incident](#month-2-production-incident)
6. [Month 3: Security Audit](#month-3-security-audit)
7. [Month 4: Scaling Issues](#month-4-scaling-issues)
8. [Month 6: Advanced Troubleshooting](#month-6-advanced-troubleshooting)

---

# Day 1: Your First Day as DevOps Engineer

## Scenario: Getting Access to Servers

**The Situation:**  
You join TechCorp as a DevOps Engineer. Your manager gives you IP addresses of three servers: 
- `web-server-01` (192.168.1.10)
- `app-server-01` (192.168.1.20)
- `db-server-01` (192.168.1.30)

Your task: Get access and explore the system.

### Step 1: Connect to the Server

```bash
# Try to SSH into web server
ssh admin@192.168.1.10
# Permission denied (publickey)
```

**Problem:** You can't login! 

**Solution:** You need SSH keys. Your manager sends you a private key file `techcorp-key.pem`

```bash
# Download the key to your local machine
# Give it proper permissions (important!)
chmod 400 techcorp-key.pem

# Now connect using the key
ssh -i techcorp-key.pem admin@192.168.1.10
# Success! You're in!
```

**Why chmod 400?** SSH requires private keys to be readable only by you for security.

### Step 2: Explore the System

```bash
# Where am I?
pwd
# Output: /home/admin

# Who am I?
whoami
# Output: admin

# What's my user info?
id
# Output: uid=1001(admin) gid=1001(admin) groups=1001(admin),27(sudo)

# Check system information
uname -a
# Linux web-server-01 5.15.0-56-generic x86_64 GNU/Linux

hostnamectl
# Shows: Ubuntu 22.04 LTS

# How long has server been running?
uptime
# 14:23:45 up 45 days, 3:42, 1 user, load average: 0.15, 0.20, 0.18
```

### Step 3: Check What's Running

```bash
# Check running processes
ps aux | head -10
# Shows top 10 processes

# Better view - install htop
sudo apt update
sudo apt install htop

htop
# Interactive view - press F10 to exit
```

### Step 4: Check Disk Space (First Important Check!)

```bash
# Check disk usage
df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        50G   35G   12G  75% /
# /dev/sdb1       100G   89G    6G  94% /var/log

# WARNING: /var/log is 94% full!
```

**Problem Detected:** Logs partition is almost full!

```bash
# Find what's using space
du -sh /var/log/* | sort -rh | head -10
# 8.5G  /var/log/nginx
# 2.1G  /var/log/application
# 850M  /var/log/syslog

# Old nginx logs are taking too much space!
# Check the largest files
find /var/log/nginx -type f -exec ls -lh {} \; | sort -k5 -rh | head -10

# Clean old logs (older than 30 days)
sudo find /var/log/nginx -name "*.log.*" -mtime +30 -delete

# Verify space freed
df -h /var/log
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sdb1       100G   80G   15G  85% /var/log
# Better! But we need log rotation
```

### Step 5: Document Your Findings

```bash
# Create a notes directory
mkdir ~/notes
cd ~/notes

# Create a file with your findings
nano day1-findings.txt
```

Write:
```
Day 1 Findings - Web Server 01
================================
- Server: Ubuntu 22.04 LTS
- Uptime: 45 days
- Issue: /var/log partition 94% full
- Action: Deleted logs older than 30 days
- Freed: 9GB space
- TODO: Setup log rotation
```

Save (Ctrl+O, Enter, Ctrl+X)

---

# Week 1: Setting Up a Web Server

## Scenario: Deploy a New Application

**The Situation:**  
Your team developed a Node.js application. You need to:
1. Create a dedicated user for the app
2. Set up the application
3. Configure Nginx as reverse proxy
4. Ensure it runs on boot

### Step 1: Create Application User

```bash
# Create user without home directory login
sudo useradd -r -s /bin/bash appuser

# Why -r? Creates a system user (no login, no home directory needed)
# But we need a home directory for our app

sudo useradd -m -s /bin/bash appuser
# This creates /home/appuser

# Set password (required for some operations)
sudo passwd appuser
# Enter password twice

# Verify user created
id appuser
# uid=1002(appuser) gid=1002(appuser) groups=1002(appuser)

cat /etc/passwd | grep appuser
# appuser:x:1002:1002::/home/appuser:/bin/bash
```

### Step 2: Create Application Directory Structure

```bash
# Create app directory
sudo mkdir -p /opt/myapp/{logs,config,data}

# Check what we created
tree /opt/myapp
# /opt/myapp
# ├── logs
# ├── config
# └── data

# Give ownership to appuser
sudo chown -R appuser:appuser /opt/myapp

# Set proper permissions
sudo chmod 755 /opt/myapp
sudo chmod 750 /opt/myapp/logs
sudo chmod 750 /opt/myapp/config
sudo chmod 770 /opt/myapp/data

# Verify permissions
ls -la /opt/myapp
# drwxr-xr-x  5 appuser appuser 4096 Jan 18 10:30 .
# drwxr-xr-x  3 root    root    4096 Jan 18 10:28 ..
# drwxr-x---  2 appuser appuser 4096 Jan 18 10:30 config
# drwxrwx---  2 appuser appuser 4096 Jan 18 10:30 data
# drwxr-x---  2 appuser appuser 4096 Jan 18 10:30 logs
```

### Step 3: Install Application Dependencies

```bash
# Switch to appuser
sudo su - appuser

# Upload your application code
cd /opt/myapp
# (Assume code is uploaded via git or scp)

# Install Node.js (switch back to admin)
exit  # Exit from appuser

# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Verify installation
node --version
# v18.12.0

npm --version
# 8.19.2
```

### Step 4: Install Application

```bash
# Switch to appuser
sudo su - appuser
cd /opt/myapp

# Create a simple test app
cat > app.js << 'EOF'
const http = require('http');
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello from MyApp!\n');
});

server.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
EOF

# Test the app
node app.js &
# Server running on port 3000

# Test it works
curl localhost:3000
# Hello from MyApp!

# Kill the test process
pkill -f "node app.js"
```

### Step 5: Create Systemd Service

```bash
# Exit from appuser
exit

# Create service file
sudo nano /etc/systemd/system/myapp.service
```

Write:
```ini
[Unit]
Description=My Node.js Application
After=network.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/app.js
Restart=always
RestartSec=10
StandardOutput=append:/opt/myapp/logs/app.log
StandardError=append:/opt/myapp/logs/error.log

[Install]
WantedBy=multi-user.target
```

Save and exit.

```bash
# Reload systemd
sudo systemctl daemon-reload

# Start the service
sudo systemctl start myapp

# Check status
sudo systemctl status myapp
# ● myapp.service - My Node.js Application
#    Loaded: loaded (/etc/systemd/system/myapp.service; disabled)
#    Active: active (running) since...

# Enable on boot
sudo systemctl enable myapp

# Test it's working
curl localhost:3000
# Hello from MyApp!

# Check logs
tail -f /opt/myapp/logs/app.log
# Server running on port 3000
```

### Step 6: Install and Configure Nginx

```bash
# Install Nginx
sudo apt install nginx

# Check if installed
nginx -v
# nginx version: nginx/1.18.0

# Check if running
sudo systemctl status nginx
# Active: active (running)

# Create Nginx configuration
sudo nano /etc/nginx/sites-available/myapp
```

Write:
```nginx
server {
    listen 80;
    server_name myapp.techcorp.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Save and exit.

```bash
# Create symbolic link to enable site
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Reload Nginx
sudo systemctl reload nginx

# Test via Nginx
curl localhost
# Hello from MyApp!

# Check if port 80 is listening
sudo ss -tulnp | grep :80
# tcp   LISTEN 0   511   0.0.0.0:80   0.0.0.0:*   users:(("nginx",pid=1234))
```

**Success!** Your application is now running and accessible via Nginx.

---

# Week 2: The Database Crisis

## Scenario: Database Server is Slow

**The Situation:**  
8:00 AM - Your phone rings. "The website is down!" says the frantic manager.

### Step 1: Quick Investigation

```bash
# SSH to web server
ssh -i techcorp-key.pem admin@192.168.1.10

# Check if app is running
sudo systemctl status myapp
# Active: active (running) - OK

# Check if it responds
curl localhost:3000
# (hangs... no response)

# Check app logs
tail -50 /opt/myapp/logs/error.log
# Error: connect ETIMEDOUT 192.168.1.30:3306
# Error: Cannot connect to database
# Error: connect ETIMEDOUT 192.168.1.30:3306
```

**Problem:** Can't connect to database server!

### Step 2: Check Network Connectivity

```bash
# Ping database server
ping -c 4 192.168.1.30
# 64 bytes from 192.168.1.30: icmp_seq=1 ttl=64 time=0.5 ms
# Server is reachable

# Check if MySQL port is open
nc -zv 192.168.1.30 3306
# Connection refused

# Or use telnet
telnet 192.168.1.30 3306
# Unable to connect
```

**Problem:** Database port is not accessible!

### Step 3: SSH to Database Server

```bash
# Connect to database server
ssh -i techcorp-key.pem admin@192.168.1.30

# Check if MySQL is running
sudo systemctl status mysql
# ● mysql.service - MySQL Community Server
#    Loaded: loaded
#    Active: inactive (dead)
```

**Found it!** MySQL is not running!

### Step 4: Start MySQL

```bash
# Try to start MySQL
sudo systemctl start mysql

# Check status
sudo systemctl status mysql
# Failed to start MySQL Community Server.
# Jan 18 08:05:32 systemd[1]: mysql.service: Main process exited, code=exited, status=1/FAILURE
```

**Problem:** MySQL won't start! Check logs.

```bash
# Check MySQL error log
sudo tail -100 /var/log/mysql/error.log
# [ERROR] InnoDB: Cannot allocate memory for the buffer pool
# [ERROR] InnoDB: Plugin initialization aborted
# [ERROR] Cannot start InnoDB
```

**Root Cause:** Out of memory!

### Step 5: Check System Resources

```bash
# Check memory
free -h
#               total        used        free      shared  buff/cache   available
# Mem:           2.0G        1.9G         50M        10M        100M         30M
# Swap:          1.0G        1.0G          0B

# Almost no free memory!

# Check what's using memory
ps aux --sort=-%mem | head -10
# USER       PID %CPU %MEM    VSZ   RSS COMMAND
# backup   12345  0.0 45.0 950000 920000 /usr/bin/backup-daemon
# mysql    23456  0.0 30.0 750000 610000 /usr/sbin/mysqld
# root      1234  0.0  5.0 125000 102000 /usr/bin/log-collector
```

**Found it!** A backup daemon is using 45% of memory!

### Step 6: Fix the Memory Issue

```bash
# Check the backup process
ps aux | grep backup
# backup   12345  0.0 45.0 ... /usr/bin/backup-daemon --full-backup

# This is a runaway backup process
# Kill it
sudo kill 12345

# Verify it's gone
ps aux | grep backup
# (nothing)

# Check memory again
free -h
#               total        used        free      shared  buff/cache   available
# Mem:           2.0G        1.0G        800M        10M        200M        900M

# Much better!

# Clear cache to free more memory
sudo sync
sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'

# Try starting MySQL again
sudo systemctl start mysql

# Check status
sudo systemctl status mysql
# ● mysql.service - MySQL Community Server
#    Active: active (running)

# Success!
```

### Step 7: Verify Everything Works

```bash
# Test MySQL connection
mysql -u root -p -e "SELECT 1;"
# +---+
# | 1 |
# +---+
# | 1 |
# +---+

# Check MySQL is listening
sudo ss -tulnp | grep 3306
# tcp   LISTEN 0   151   0.0.0.0:3306   0.0.0.0:*   users:(("mysqld",pid=34567))

# Go back to web server
exit

# Test application
curl localhost
# Hello from MyApp! (with data from database)

# Success! Application is working again.
```

### Step 8: Prevent Future Issues

```bash
# SSH back to database server
ssh -i techcorp-key.pem admin@192.168.1.30

# Check the backup cron job
sudo crontab -l -u backup
# 0 2 * * * /usr/bin/backup-daemon --full-backup

# The backup is running every night but not completing properly

# Fix: Add memory limit to backup process
sudo nano /etc/systemd/system/backup.service
```

Write:
```ini
[Service]
MemoryMax=500M
MemoryHigh=400M
```

```bash
# Also set up monitoring
sudo apt install monit

# Configure monit to alert on high memory
sudo nano /etc/monit/conf.d/memory-check
```

Write:
```
check system localhost
    if memory usage > 85% for 5 cycles then alert
```

```bash
# Restart monit
sudo systemctl restart monit

# Create incident report
nano ~/incident-week2.txt
```

Write:
```
Incident Report - Database Outage
===================================
Date: Week 2, 8:00 AM
Duration: 45 minutes
Cause: Runaway backup process consumed all memory
Impact: Website completely down
Resolution: Killed backup process, restarted MySQL
Prevention: 
  - Added memory limits to backup service
  - Configured memory monitoring
  - Need to review backup strategy
```

---

# Week 3: Application Deployment

## Scenario: Deploy New Application Version

**The Situation:**  
Developers pushed version 2.0 of the application. You need to deploy it with zero downtime.

### Step 1: Prepare for Deployment

```bash
# SSH to web server
ssh -i techcorp-key.pem admin@192.168.1.10

# Create backup of current version
sudo su - appuser
cd /opt

# Create backup with timestamp
tar -czf myapp-backup-$(date +%Y%m%d-%H%M%S).tar.gz myapp/

# Verify backup created
ls -lh myapp-backup-*
# -rw-r--r-- 1 appuser appuser 2.3M Jan 18 14:30 myapp-backup-20260118-143000.tar.gz

# Move backup to safe location
mkdir -p ~/backups
mv myapp-backup-* ~/backups/

# Keep only last 5 backups
cd ~/backups
ls -t | tail -n +6 | xargs -r rm
```

### Step 2: Download New Version

```bash
# Go to app directory
cd /opt/myapp

# Create a deployment script
cat > deploy.sh << 'EOF'
#!/bin/bash
set -e  # Exit on any error

echo "Starting deployment..."

# Pull latest code from git
git fetch origin
git checkout main
git pull origin main

# Install dependencies
npm install --production

# Run database migrations if needed
# npm run migrate

echo "Deployment complete!"
EOF

chmod +x deploy.sh

# For this example, let's update manually
# Update app.js to version 2.0
nano app.js
```

Update to:
```javascript
const http = require('http');
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'application/json');
  res.end(JSON.stringify({
    version: '2.0',
    message: 'Hello from MyApp v2.0!',
    timestamp: new Date().toISOString()
  }));
});

server.listen(port, () => {
  console.log(`Server v2.0 running on port ${port}`);
});
```

### Step 3: Test New Version

```bash
# Don't restart main service yet
# Start test instance on different port
sed 's/3000/3001/g' app.js > app-test.js

# Start test instance
node app-test.js &
# [1] 45678

# Test it
curl localhost:3001
# {"version":"2.0","message":"Hello from MyApp v2.0!","timestamp":"2026-01-18T14:35:00.000Z"}

# Kill test instance
kill %1
rm app-test.js
```

### Step 4: Zero-Downtime Deployment

```bash
# Exit from appuser
exit

# Reload systemd (if service file changed)
sudo systemctl daemon-reload

# Restart application service
sudo systemctl restart myapp

# Monitor the restart
sudo systemctl status myapp
# Check logs for errors
tail -f /opt/myapp/logs/app.log
# Server v2.0 running on port 3000

# Test new version
curl localhost
# {"version":"2.0","message":"Hello from MyApp v2.0!","timestamp":"..."}

# Test via Nginx
curl http://myapp.techcorp.com
# Works!
```

### Step 5: Monitor After Deployment

```bash
# Watch logs for errors
sudo tail -f /opt/myapp/logs/error.log
# (wait 5 minutes, watch for errors)

# Check resource usage
htop
# (press F4, type "node", enter to filter)
# Check CPU and memory usage is normal

# Monitor connections
watch -n 5 'ss -tan | grep :3000 | wc -l'
# Shows number of active connections every 5 seconds

# Check response times
time curl localhost > /dev/null
# real    0m0.024s

# All good!
```

### Step 6: Rollback Plan (Just in Case)

```bash
# If something goes wrong, rollback:
# sudo su - appuser
# cd ~/backups
# tar -xzf myapp-backup-20260118-143000.tar.gz -C /opt/
# exit
# sudo systemctl restart myapp
```

Create rollback script:
```bash
sudo nano /opt/rollback.sh
```

Write:
```bash
#!/bin/bash
# Quick rollback script

BACKUP_FILE="$1"

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup-file>"
    echo "Available backups:"
    ls -lh /home/appuser/backups/
    exit 1
fi

echo "Rolling back to $BACKUP_FILE..."
sudo systemctl stop myapp
sudo tar -xzf "/home/appuser/backups/$BACKUP_FILE" -C /opt/
sudo systemctl start myapp
echo "Rollback complete!"
```

```bash
sudo chmod +x /opt/rollback.sh
```

---

# Month 2: Production Incident

## Scenario: Website is Extremely Slow

**The Situation:**  
2:00 PM - Users report website is very slow. Load time is 30+ seconds!

### Step 1: Quick Health Check

```bash
# SSH to web server
ssh -i techcorp-key.pem admin@192.168.1.10

# Check system load
uptime
# 14:05:23 up 15 days, 5:32, 2 users, load average: 8.45, 7.23, 5.12
# Load average is very high! (should be < 2.0 for 2-CPU system)

# Check CPU
top -bn1 | head -20
# %Cpu(s): 85.2 us, 10.3 sy, 0.0 ni, 4.1 id
# CPU is 96% utilized!

# Which process is using CPU?
ps aux --sort=-%cpu | head -10
# USER       PID %CPU %MEM COMMAND
# appuser  12345 145.2  5.3 node /opt/myapp/app.js
# appuser  12346  89.3  4.2 node /opt/myapp/app.js
# appuser  12347  78.1  3.9 node /opt/myapp/app.js
```

**Problem:** Multiple node processes eating CPU, but we only have one app?

### Step 2: Investigate Processes

```bash
# Check all node processes
ps aux | grep node
# Shows 15 node processes!

# Check systemd service
sudo systemctl status myapp
# Shows only 1 process in the cgroup

# Something is spawning extra processes

# Check process tree
pstree -p | grep node
# Shows parent-child relationships

# Check what's listening on port 3000
sudo lsof -i :3000
# COMMAND   PID     USER
# node    12345  appuser
# node    12346  appuser
# node    12347  appuser
# ... (multiple processes!)
```

### Step 3: Check Application Logs

```bash
# Check recent logs
sudo tail -100 /opt/myapp/logs/app.log
# Unhandled Promise Rejection
# Error: EMFILE: too many open files
# Error: EMFILE: too many open files
# Error: EMFILE: too many open files
```

**Found it!** Too many open files - likely a file descriptor leak.

```bash
# Check open files for main process
sudo lsof -p 12345 | wc -l
# 5023 (should be much less!)

# Check file descriptor limit
sudo cat /proc/12345/limits | grep "open files"
# Max open files    1024    1024    files
```

**Problem:** Application has file descriptor leak and hit the limit!

### Step 4: Emergency Fix

```bash
# Check current connections
sudo ss -tan | grep :3000 | wc -l
# 1893 connections

# Many are in CLOSE_WAIT state (not properly closed)
sudo ss -tan | grep CLOSE_WAIT | grep :3000 | wc -l
# 1654

# Temporary fix: Restart application
sudo systemctl restart myapp

# Wait for restart
sleep 5

# Check if better
uptime
# load average: 2.34, 5.12, 5.23
# Much better!

# Check connections again
sudo ss -tan | grep :3000 | wc -l
# 45
# Much better!

# Check response time
time curl localhost
# real    0m0.156s
# Back to normal!
```

### Step 5: Find Root Cause

```bash
# Check application code for file descriptor leaks
sudo su - appuser
cd /opt/myapp

# Search for file operations
grep -r "fs\." *.js
grep -r "createReadStream" *.js
grep -r "createWriteStream" *.js

# Check for database connections
grep -r "mysql.createConnection" *.js
# Found in db.js: multiple connections created but not closed!

# Review the code
cat db.js
```

Found problematic code:
```javascript
// BAD: Creates new connection on every request
function query(sql) {
  const connection = mysql.createConnection(config);
  connection.query(sql, function(err, results) {
    // BUG: Connection never closed!
    return results;
  });
}
```

Should be:
```javascript
// GOOD: Use connection pool
const pool = mysql.createPool(config);

function query(sql) {
  return new Promise((resolve, reject) => {
    pool.query(sql, function(err, results) {
      if (err) reject(err);
      else resolve(results);
    });
  });
}
```

### Step 6: Apply Permanent Fix

```bash
# Fix the code (developers will do this)
# For now, increase file descriptor limit

exit  # Exit from appuser

# Edit systemd service
sudo nano /etc/systemd/system/myapp.service
```

Add:
```ini
[Service]
LimitNOFILE=65536
```

```bash
# Reload systemd
sudo systemctl daemon-reload

# Restart service
sudo systemctl restart myapp

# Verify new limit
PID=$(systemctl show -p MainPID myapp | cut -d= -f2)
cat /proc/$PID/limits | grep "open files"
# Max open files    65536    65536    files
```

### Step 7: Set Up Monitoring

```bash
# Install monitoring script
sudo nano /usr/local/bin/check-app-health.sh
```

Write:
```bash
#!/bin/bash

# Get app PID
PID=$(systemctl show -p MainPID myapp | cut -d= -f2)

# Count open file descriptors
FD_COUNT=$(sudo lsof -p $PID 2>/dev/null | wc -l)

# Check if approaching limit
if [ $FD_COUNT -gt 50000 ]; then
    echo "WARNING: High file descriptor usage: $FD_COUNT"
    # Send alert (email, Slack, etc.)
    logger -p user.warning "myapp: High FD count: $FD_COUNT"
fi

# Check CLOSE_WAIT connections
CLOSE_WAIT=$(ss -tan | grep :3000 | grep CLOSE_WAIT | wc -l)

if [ $CLOSE_WAIT -gt 100 ]; then
    echo "WARNING: High CLOSE_WAIT connections: $CLOSE_WAIT"
    logger -p user.warning "myapp: High CLOSE_WAIT: $CLOSE_WAIT"
fi
```

```bash
# Make executable
sudo chmod +x /usr/local/bin/check-app-health.sh

# Add to cron (every 5 minutes)
sudo crontab -e
```

Add:
```
*/5 * * * * /usr/local/bin/check-app-health.sh
```

### Step 8: Document the Incident

```bash
nano ~/incident-month2.txt
```

Write:
```
Production Incident - Slow Website Performance
================================================
Date: Month 2, 2:00 PM
Duration: 30 minutes
Severity: High (user-facing)

Symptoms:
- Website response time 30+ seconds
- Load average 8.45 (normal: <2.0)
- 15 node processes (should be 1)

Root Cause:
- File descriptor leak in database connection code
- Connections created but never closed
- Hit system limit of 1024 open files
- Caused process to spawn multiple instances

Immediate Fix:
- Restarted application service
- Response time back to <200ms

Permanent Fix:
- Increased file descriptor limit to 65536
- Code fix required: implement connection pooling
- Added monitoring for file descriptors
- Added monitoring for CLOSE_WAIT connections

Lessons Learned:
- Always use connection pooling for databases
- Monitor file descriptors in production
- Set up alerts before hitting limits
- Review code for resource leaks

Action Items:
- [ ] Developers fix connection pooling
- [ ] Add automated tests for resource leaks
- [ ] Review all file operations in code
- [ ] Set up better application monitoring
```

---

# Month 3: Security Audit

## Scenario: Failed Security Audit

**The Situation:**  
Security team found multiple vulnerabilities during audit. You must fix them immediately.

### Finding 1: Users with Weak Passwords

```bash
# Check password policy
sudo cat /etc/pam.d/common-password | grep pam_unix
# password [success=1 default=ignore] pam_unix.so obscure sha512
# No minimum length!

# Install password quality checker
sudo apt install libpam-pwquality

# Configure password policy
sudo nano /etc/security/pwquality.conf
```

Set:
```
minlen = 12
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
maxrepeat = 3
```

```bash
# Force password change for weak passwords
sudo nano /usr/local/bin/check-weak-passwords.sh
```

Write:
```bash
#!/bin/bash
# Check for users with passwords that never expire

echo "Users with no password expiry:"
awk -F: '$5 == "" {print $1}' /etc/shadow

# Force password expiry for regular users
for user in $(awk -F: '$3 >= 1000 && $3 < 60000 {print $1}' /etc/passwd); do
    echo "Setting password expiry for $user"
    sudo chage -M 90 -W 7 $user
done
```

```bash
sudo chmod +x /usr/local/bin/check-weak-passwords.sh
sudo /usr/local/bin/check-weak-passwords.sh
```

### Finding 2: Unnecessary Services Running

```bash
# List all listening services
sudo ss -tulnp
# Shows many services

# Check what's installed
sudo systemctl list-units --type=service --state=running

# Identify unnecessary services
sudo systemctl status apache2
# Active! But we use Nginx!

# Stop and disable Apache
sudo systemctl stop apache2
sudo systemctl disable apache2

# Remove if not needed
sudo apt remove apache2

# Check for other unnecessary services
sudo systemctl | grep running | grep -v systemd
```

### Finding 3: Improper File Permissions

```bash
# Find world-writable files (security risk!)
sudo find / -type f -perm -002 -ls 2>/dev/null
# /home/appuser/config.json is world-writable!

# Fix it
sudo chmod 640 /home/appuser/config.json
sudo chown appuser:appuser /home/appuser/config.json

# Find SUID files (can be dangerous)
sudo find / -type f -perm -4000 -ls 2>/dev/null > ~/suid-files.txt

# Review the list
cat ~/suid-files.txt
# Check if any unauthorized SUID files exist

# Find files owned by deleted users
sudo find / -nouser -o -nogroup 2>/dev/null

# Find sensitive files with wrong permissions
sudo find /etc -name "*.conf" -perm -044 -ls
```

### Finding 4: No SSH Key-Only Authentication

```bash
# Check SSH configuration
sudo cat /etc/ssh/sshd_config | grep PasswordAuthentication
# PasswordAuthentication yes
# This allows password logins (less secure)

# Disable password authentication
sudo nano /etc/ssh/sshd_config
```

Change:
```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

```bash
# Test configuration
sudo sshd -t

# Reload SSH
sudo systemctl reload sshd

# IMPORTANT: Make sure you have SSH key access before doing this!
```

### Finding 5: No Firewall Configured

```bash
# Check firewall status
sudo ufw status
# Status: inactive

# Enable UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (important! Do this first!)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow from specific IPs only (for database)
sudo ufw allow from 192.168.1.10 to any port 3306

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status numbered
# Status: active
#      To                         Action      From
#      --                         ------      ----
# [ 1] 22/tcp                     ALLOW IN    Anywhere
# [ 2] 80/tcp                     ALLOW IN    Anywhere
# [ 3] 443/tcp                    ALLOW IN    Anywhere
# [ 4] 3306                       ALLOW IN    192.168.1.10
```

### Finding 6: Audit Logging Not Enabled

```bash
# Install auditd
sudo apt install auditd

# Start auditd
sudo systemctl start auditd
sudo systemctl enable auditd

# Add audit rules
sudo nano /etc/audit/rules.d/custom.rules
```

Add:
```
# Monitor file changes in /etc
-w /etc/ -p wa -k etc_changes

# Monitor user/group changes
-w /etc/passwd -p wa -k passwd_changes
-w /etc/group -p wa -k group_changes
-w /etc/shadow -p wa -k shadow_changes

# Monitor sudo usage
-w /var/log/sudo.log -p wa -k sudo_log

# Monitor network configuration
-w /etc/network/ -p wa -k network_changes
```

```bash
# Reload rules
sudo augenrules --load

# Test: View audit logs
sudo ausearch -k passwd_changes
```

### Finding 7: Unpatched Packages

```bash
# Check for updates
sudo apt update

# List upgradable packages
apt list --upgradable
# Shows many packages!

# Check for security updates
sudo unattended-upgrades --dry-run -d

# Apply security updates
sudo apt upgrade -y

# Configure automatic security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Verify configuration
cat /etc/apt/apt.conf.d/50unattended-upgrades
```

### Finding 8: Create Security Hardening Script

```bash
sudo nano /usr/local/bin/security-hardening.sh
```

Write:
```bash
#!/bin/bash
# Security Hardening Script

echo "=== Security Hardening Check ==="

# Check password policy
echo -e "\n[+] Checking password policy..."
grep -E "minlen|dcredit|ucredit" /etc/security/pwquality.conf

# Check for users without password expiry
echo -e "\n[+] Users with no password expiry:"
awk -F: '$5 == "" {print $1}' /etc/shadow

# Check for world-writable files
echo -e "\n[+] World-writable files:"
find /home /opt -type f -perm -002 2>/dev/null | head -5

# Check SSH configuration
echo -e "\n[+] SSH Configuration:"
grep -E "PasswordAuthentication|PermitRootLogin" /etc/ssh/sshd_config

# Check firewall status
echo -e "\n[+] Firewall Status:"
sudo ufw status

# Check for unpatched packages
echo -e "\n[+] Unpatched packages:"
apt list --upgradable 2>/dev/null | grep -v "Listing" | wc -l

# Check failed login attempts
echo -e "\n[+] Failed login attempts (last 24h):"
sudo journalctl --since "24 hours ago" | grep "Failed password" | wc -l

# Check listening services
echo -e "\n[+] Listening services:"
sudo ss -tulnp | grep LISTEN

echo -e "\n=== Check Complete ==="
```

```bash
sudo chmod +x /usr/local/bin/security-hardening.sh

# Run it
sudo /usr/local/bin/security-hardening.sh
```

---

# Month 4: Scaling Issues

## Scenario: Black Friday - Traffic Surge

**The Situation:**  
Tomorrow is Black Friday. Expecting 10x normal traffic. Must scale infrastructure.

### Step 1: Current Capacity Assessment

```bash
# Check current resource usage
ssh -i techcorp-key.pem admin@192.168.1.10

# Monitor for 1 hour during peak time
# CPU usage
mpstat 5 720 > cpu-stats.txt &
# 5 second intervals, 720 times (1 hour)

# Memory usage
vmstat 5 720 > memory-stats.txt &

# Network connections
watch -n 5 'ss -s >> network-stats.txt'

# Analyze after 1 hour
# Average CPU
awk '{sum+=$3} END {print "Avg CPU:", sum/NR"%"}' cpu-stats.txt
# Peak CPU
sort -k3 -rn cpu-stats.txt | head -1

# Average connections
awk '{print $2}' network-stats.txt | awk '{sum+=$1} END {print "Avg:", sum/NR}'
```

Current capacity:
- CPU: 60% average, 85% peak
- Memory: 1.5GB / 2GB used
- Connections: 200 avg, 450 peak
- Expected tomorrow: 2000-4500 connections

### Step 2: Optimize Current Server

```bash
# Enable Nginx caching
sudo nano /etc/nginx/sites-available/myapp
```

Add:
```nginx
# Cache settings
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=app_cache:10m max_size=1g inactive=60m;

server {
    listen 80;
    server_name myapp.techcorp.com;

    # Enable caching
    proxy_cache app_cache;
    proxy_cache_valid 200 5m;
    proxy_cache_use_stale error timeout http_500 http_502 http_503;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_cache_bypass $http_cache_control;
        add_header X-Cache-Status $upstream_cache_status;
        
        # Headers
        proxy_http_version 1.1;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Host $host;
    }
    
    # Cache static files longer
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        proxy_pass http://localhost:3000;
        proxy_cache app_cache;
        proxy_cache_valid 200 30m;
        expires 30d;
    }
}
```

```bash
# Create cache directory
sudo mkdir -p /var/cache/nginx
sudo chown www-data:www-data /var/cache/nginx

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Test caching
curl -I http://localhost
# Look for: X-Cache-Status: MISS (first time)
curl -I http://localhost
# Look for: X-Cache-Status: HIT (second time)
```

### Step 3: Tune Application

```bash
# Increase Node.js memory limit
sudo nano /etc/systemd/system/myapp.service
```

Modify:
```ini
[Service]
Environment="NODE_ENV=production"
Environment="NODE_OPTIONS=--max-old-space-size=1536"
ExecStart=/usr/bin/node /opt/myapp/app.js
```

```bash
# Use PM2 for clustering (multiple Node processes)
sudo npm install -g pm2

# Create PM2 config
sudo su - appuser
cat > /opt/myapp/ecosystem.config.js << 'EOF'
module.exports = {
  apps: [{
    name: 'myapp',
    script: './app.js',
    instances: 'max',  // Uses all CPU cores
    exec_mode: 'cluster',
    max_memory_restart: '500M',
    env: {
      NODE_ENV: 'production'
    }
  }]
};
EOF

# Test PM2
pm2 start ecosystem.config.js
pm2 list
# Should show 2-4 instances (based on CPU cores)

pm2 stop all
exit
```

### Step 4: Set Up Load Balancer

```bash
# Install HAProxy
sudo apt install haproxy

# Backup original config
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.backup

# Configure HAProxy
sudo nano /etc/haproxy/haproxy.cfg
```

Add:
```
global
    maxconn 4096
    
defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http_front
    bind *:8080
    default_backend http_back
    
    # Rate limiting
    stick-table type ip size 100k expire 30s store http_req_rate(10s)
    http-request track-sc0 src
    http-request deny if { sc_http_req_rate(0) gt 100 }

backend http_back
    balance roundrobin
    option httpchk GET /health
    
    # If you have multiple servers:
    # server web1 192.168.1.10:3000 check
    # server web2 192.168.1.11:3000 check
    
    # For single server with PM2:
    server app1 localhost:3000 check
    server app2 localhost:3001 check
    server app3 localhost:3002 check
    server app4 localhost:3003 check
```

### Step 5: Set Up Application Clustering

```bash
# Update app.js to run on different ports
sudo su - appuser
cd /opt/myapp

# Create app instances
for i in {0..3}; do
    PORT=$((3000 + i))
    sed "s/const port = 3000/const port = $PORT/" app.js > app-$i.js
done

# Create systemd services for each
exit

for i in {0..3}; do
    sudo cp /etc/systemd/system/myapp.service /etc/systemd/system/myapp-$i.service
    sudo sed -i "s|app.js|app-$i.js|" /etc/systemd/system/myapp-$i.service
    sudo sed -i "s|myapp|myapp-$i|" /etc/systemd/system/myapp-$i.service
done

# Reload systemd
sudo systemctl daemon-reload

# Start all instances
for i in {0..3}; do
    sudo systemctl start myapp-$i
    sudo systemctl enable myapp-$i
done

# Verify all running
for i in {0..3}; do
    systemctl is-active myapp-$i
done
```

### Step 6: Database Connection Pooling

```bash
ssh -i techcorp-key.pem admin@192.168.1.30

# Check MySQL configuration
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Optimize:
```ini
[mysqld]
# Connection settings
max_connections = 500
connect_timeout = 10

# Buffer settings
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M

# Query cache (for read-heavy workloads)
query_cache_type = 1
query_cache_size = 64M

# Slow query log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow-query.log
long_query_time = 2
```

```bash
# Restart MySQL
sudo systemctl restart mysql

# Monitor connections
watch -n 2 'mysql -u root -p -e "SHOW STATUS LIKE \"Threads_connected\";"'
```

### Step 7: Monitor Everything

```bash
# Create monitoring dashboard script
nano ~/monitor-dashboard.sh
```

Write:
```bash
#!/bin/bash

while true; do
    clear
    echo "=== System Monitor - $(date) ==="
    echo ""
    
    echo "=== CPU & Memory ==="
    echo "Load: $(uptime | awk -F'load average:' '{print $2}')"
    free -h | grep Mem
    echo ""
    
    echo "=== Application Status ==="
    for i in {0..3}; do
        status=$(systemctl is-active myapp-$i)
        echo "myapp-$i: $status"
    done
    echo ""
    
    echo "=== Connections ==="
    echo "Total: $(ss -tan | grep :3000 | wc -l)"
    echo "ESTABLISHED: $(ss -tan | grep ESTABLISHED | grep :3000 | wc -l)"
    echo ""
    
    echo "=== HAProxy Stats ==="
    echo "Total connections: $(ss -tan | grep :8080 | wc -l)"
    echo ""
    
    echo "=== Database ==="
    mysql -u root -p"$DB_PASS" -e "SHOW STATUS LIKE 'Threads_connected';" 2>/dev/null | tail -1
    echo ""
    
    echo "=== Nginx Cache ==="
    curl -s -I http://localhost | grep X-Cache-Status
    
    sleep 5
done
```

```bash
chmod +x ~/monitor-dashboard.sh
```

### Step 8: Load Testing

```bash
# Install Apache Bench
sudo apt install apache2-utils

# Test current capacity
ab -n 10000 -c 100 http://localhost/
# -n: total requests
# -c: concurrent requests

# Results:
# Requests per second: 1250
# Time per request: 80ms (mean)
# Failed requests: 0

# Test with higher concurrency
ab -n 50000 -c 500 http://localhost/

# Monitor during test
watch -n 1 'systemctl status myapp-* | grep Active'
```

### Step 9: Create Scale-Up Playbook

```bash
nano ~/scale-up-playbook.md
```

Write:
```markdown
# Black Friday Scale-Up Playbook

## Before Event (24 hours)
- [ ] Run load tests: `ab -n 50000 -c 500 http://localhost/`
- [ ] Verify all app instances running: `systemctl status myapp-*`
- [ ] Check disk space: `df -h` (should be <70%)
- [ ] Verify backups: `ls -lh ~/backups/`
- [ ] Clear old logs: `find /var/log -name "*.log.*" -mtime +7 -delete`
- [ ] Alert team on standby

## During Event (monitor every 15 minutes)
- [ ] CPU usage: `top` (should be <85%)
- [ ] Memory: `free -h` (should have >500MB free)
- [ ] Connections: `ss -tan | grep :3000 | wc -l`
- [ ] Error logs: `tail -f /opt/myapp/logs/error.log`
- [ ] Database: `mysql -e "SHOW PROCESSLIST"`

## If CPU > 90%
1. Check which process: `ps aux --sort=-%cpu | head`
2. Scale horizontally: Start more app instances
3. Consider vertical scaling (more CPU)

## If Memory > 90%
1. Restart app instances one by one
2. Clear cache: `sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'`
3. Check for memory leaks: `pmap PID | tail -1`

## If Database Slow
1. Check slow queries: `tail /var/log/mysql/slow-query.log`
2. Check connections: `mysql -e "SHOW PROCESSLIST"`
3. Kill long queries if needed
4. Enable query cache

## Emergency Contacts
- On-call Engineer: +1-555-0100
- Database Team: +1-555-0200
- Management: +1-555-0300
```

---

# Month 6: Advanced Troubleshooting

## Scenario: Mysterious Disk Space Issue

**The Situation:**  
Disk space keeps filling up every few hours, even after cleanup. Root partition goes from 40% to 95% full.

### Investigation Part 1: The Hunt

```bash
# Check disk usage
df -h
# /dev/sda1    50G   47G   3G  95% /

# Find large directories
du -sh /* | sort -rh | head -10
# 18G  /var
# 12G  /opt
# 8G   /usr
# 3G   /home

# Dig deeper into /var
du -sh /var/* | sort -rh | head -10
# 15G  /var/log
# 2G   /var/cache
# 800M /var/lib

# Check /var/log
du -sh /var/log/* | sort -rh | head -10
# 12G  /var/log/journal
# 2G   /var/log/nginx
# 500M /var/log/syslog

# Journal is huge!
journalctl --disk-usage
# Archived and active journals take up 12.0G in the file system.
```

**Found it!** Journal logs are huge. But wait...

```bash
# Clean journal
sudo journalctl --vacuum-size=1G
# Deleted archived journal /var/log/journal/.../system@xxx.journal (size)

# Check again after 1 hour
df -h
# /dev/sda1    50G   47G   3G  94% /
# Still filling up!
```

### Investigation Part 2: Real-Time Monitoring

```bash
# Monitor disk usage in real-time
watch -n 60 'df -h /'

# In another terminal, monitor file creation
sudo apt install inotify-tools

# Watch for file changes
sudo inotifywait -m -r -e create -e modify /var/log/
# Shows files being created/modified

# See many files being created in /var/log/application/
# But we don't see them with ls!

# Check for deleted files still held open
sudo lsof / | grep deleted
# node    12345  appuser   10w   REG   8,1   8589934592   /var/log/application/debug.log (deleted)
# 8GB file deleted but still open!
```

**Found it!** Application has open file handle to deleted file!

### Investigation Part 3: The Fix

```bash
# Check which process holds the file
sudo lsof +L1 /
# Shows files deleted but still open
# COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NLINK   NODE NAME
# node    12345 appuser  10w   REG    8,1 8589934592     0 123456 /var/log/application/debug.log (deleted)

# The application needs to close and reopen the file
# Quick fix: Restart application
sudo systemctl restart myapp

# Check if space freed
df -h
# /dev/sda1    50G   38G   12G  76% /
# 9GB freed!

# Permanent fix: Proper log rotation
sudo nano /etc/logrotate.d/myapp
```

Write:
```
/var/log/application/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 appuser appuser
    sharedscripts
    postrotate
        /usr/bin/systemctl reload myapp > /dev/null 2>&1 || true
    endscript
}
```

```bash
# Test logrotate
sudo logrotate -d /etc/logrotate.d/myapp
# -d: debug mode (doesn't actually rotate)

# Force rotate to test
sudo logrotate -f /etc/logrotate.d/myapp

# Verify
ls -lh /var/log/application/
# debug.log
# debug.log.1.gz
```

## Scenario: Network Performance Issues

**The Situation:**  
API responses are slow, but CPU/Memory are fine. Network seems to be the issue.

### Investigation

```bash
# Check network interface statistics
ip -s link show eth0
# RX errors: 45000
# TX dropped: 12000
# Errors detected!

# Check detailed error counters
ethtool -S eth0 | grep -i error
# rx_crc_errors: 45000
# rx_frame_errors: 3000

# Physical layer issue! Check cable/switch

# Check network latency
ping -c 100 192.168.1.30 | tail -5
# rtt min/avg/max/mdev = 0.234/15.432/250.123/45.234 ms
# High latency variation!

# Trace route to database
mtr --report 192.168.1.30
# Shows packet loss at specific hop

# Check for network congestion
sar -n DEV 1 10
# Shows network interface stats every second

# Check TCP retransmissions
ss -ti | grep retrans
# Many retransmissions = network issues

# Check socket buffer sizes
sysctl net.core.rmem_max
sysctl net.core.wmem_max

# Increase buffer sizes
sudo nano /etc/sysctl.conf
```

Add:
```
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
```

```bash
# Apply changes
sudo sysctl -p

# Check TCP connection stats
ss -s
# Shows connections by state

# Monitor in real-time
watch -n 1 'ss -s'
```

---

## Practical Tips & Commands Summary

### When Server is Slow
```bash
# Quick health check (memorize this!)
uptime && free -h && df -h && ps aux --sort=-%cpu | head -5
```

### When Application Crashes
```bash
# Check logs immediately
sudo journalctl -u servicename -n 100 --no-pager
sudo tail -100 /var/log/application/error.log

# Check if process exists
ps aux | grep appname

# Check system messages
dmesg | tail -50
```

### When Disk is Full
```bash
# Quick space check
df -h && du -sh /* | sort -rh | head -5

# Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Check deleted but open files
lsof +L1
```

### When Network is Down
```bash
# Quick network check
ping -c 3 8.8.8.8  # Internet
ping -c 3 192.168.1.1  # Gateway
ip route show  # Routing table
ss -tulnp  # What's listening
```

### When You Don't Know What's Wrong
```bash
# The universal troubleshooting sequence
1. uptime  # Load
2. dmesg | tail  # Kernel messages
3. vmstat 1  # CPU, memory, I/O
4. mpstat -P ALL 1  # CPU per core
5. pidstat 1  # Process stats
6. iostat -xz 1  # Disk I/O
7. free -m  # Memory
8. sar -n DEV 1  # Network
9. sar -n TCP,ETCP 1  # TCP stats
10. top  # Overview
```

---

## Final Advice: The DevOps Mindset

### Always Ask These Questions:
1. **What changed?** (Most issues come from changes)
2. **Can I reproduce it?** (Intermittent vs consistent)
3. **Is it really a problem?** (Understand normal behavior first)
4. **What does the data say?** (Don't guess, measure)
5. **Can I automate this?** (If you do it twice, script it)

### The Golden Rules:
1. **Test in dev first** - Never test fixes in production
2. **Have a rollback plan** - Always know how to undo changes
3. **Document everything** - Your future self will thank you
4. **Monitor continuously** - Know your baselines
5. **Automate recovery** - Humans are slow, scripts are fast
6. **Keep it simple** - Complex solutions create complex problems
7. **Learn from incidents** - Every problem is a learning opportunity

---

**Remember:** Every senior DevOps engineer was once a beginner who got paged at 2 AM, broke production, and learned from it. The commands are tools, but problem-solving is the skill!

Good luck on your DevOps journey! 🚀