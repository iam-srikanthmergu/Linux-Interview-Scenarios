Linux DevOps Interview Questions & Answers
1. Production & Troubleshooting
1. Your server CPU suddenly spikes to 100%. How do you identify the root cause?
# Check overall CPU usage
top
htop

# Check load average
uptime

# Find top CPU-consuming processes
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head

# Check per-core CPU usage
mpstat -P ALL 1

# Check if Java process is causing issue
top -H -p <PID>

# Check logs
journalctl -xe
tail -f /var/log/messages
Real-Time Troubleshooting Flow

In production, first I check whether CPU spike is due to:

Application issue
Infinite loop
Traffic spike
Background job
Memory swapping
Malware/miner process
Example Scenario

A Java application suddenly started consuming 350% CPU.

Steps followed:

Ran top
Identified Java PID consuming high CPU
Used:
top -H -p <java_pid>
Found one thread consuming high CPU
Converted thread HEX value:
printf "%x\n" <thread_id>
Collected thread dump:
jstack <PID> > thread.dump
Developer identified infinite loop in API logic
Temporary fix → restarted service
Permanent fix → code patch deployed
Important Commands
sar -u 1 5
vmstat 1
iostat -x 1
pidstat 1
2. A process is consuming high memory but you don’t know which one. How do you find and fix it?
free -m
top
htop

# Find top memory-consuming processes
ps -eo pid,cmd,%mem,%cpu --sort=-%mem | head

# Detailed memory info
pmap -x <PID>

# Check OOM logs
dmesg | grep -i oom
Real-Time Scenario

One production server became very slow.

Investigation
free -m

Memory was almost full.

Then:

ps -eo pid,cmd,%mem --sort=-%mem | head

Found Python process consuming 12GB RAM.

Root Cause

Memory leak in application.

Fix
Restarted service temporarily
Enabled memory limits
Added monitoring alerts
Developers fixed memory leak
Additional Checks
vmstat 1
sar -r 1 5
3. A production server becomes slow at random times. What steps will you take to troubleshoot?
Step-by-Step Approach
1. Check System Load
uptime
top
2. Check CPU
mpstat -P ALL 1
3. Check Memory
free -m
vmstat 1
4. Check Disk I/O
iostat -x 1
iotop
5. Check Network
ss -s
netstat -antp
6. Check Logs
journalctl -xe
tail -f /var/log/messages
Real-Time Scenario

Server slowdown happened every night at 2 AM.

Found Issue

Backup job running with high compression causing:

High CPU
High Disk I/O
Solution
Changed backup timing
Reduced compression level
Added resource limits using nice and ionice
nice -n 10 backup.sh
ionice -c3 backup.sh
4. Disk space is 100% full on /. How will you handle it without downtime?
First Check
df -h
Find Large Files
du -sh /* 2>/dev/null
find / -type f -size +1G
Common Cleanup Areas
/var/log
/tmp
/var/cache
Clean Old Logs
truncate -s 0 /var/log/large.log
Clean Package Cache

Ubuntu:

apt clean

RHEL:

yum clean all
Remove Old Docker Images
docker system prune -a
Real-Time Scenario

Production server disk became 100%.

Root cause:

Huge application logs
Log rotation missing
Fix
Truncated logs
Configured logrotate
Added monitoring alerts
5. A service is down but the server is running. How do you debug this?
Check Service Status
systemctl status nginx
Check Logs
journalctl -u nginx -xe
Check Port
ss -tulnp | grep 80
Verify Process
ps -ef | grep nginx
Check Configuration
nginx -t
Real-Time Scenario

Nginx service stopped after deployment.

Root Cause

Wrong syntax in config file.

Fix
nginx -t
systemctl restart nginx
2. Processes & System Behavior
6. A process is stuck and not getting killed even with kill -9. What will you do?
Possible Reasons
Process in uninterruptible sleep (D state)
Waiting on disk/NFS I/O
Kernel-level issue
Check Process State
ps -eo pid,state,cmd | grep <PID>
If State = D

Usually related to:

Disk issue
NFS mount issue
Hardware issue
Check
iostat -x 1
dmesg
Real-Time Scenario

Backup process stuck in D state because NFS server became unreachable.

Fix
Restored NFS connectivity
Process recovered automatically
In worst case → reboot server
7. How do you find which process is using a specific port (e.g., 8080)?
lsof -i :8080

ss -tulnp | grep 8080

netstat -tulnp | grep 8080
Example Output
java  1234 root  TCP *:8080 (LISTEN)
8. You accidentally started a process in the foreground. How do you move it to background without stopping it?
Steps

Press:

CTRL + Z

Then:

bg

To detach fully:

disown
Example
./script.sh
CTRL + Z
bg
disown
9. What happens internally when you run a command in Linux?
Internal Flow
Shell receives command
Shell checks PATH variable
Creates process using fork()
Child process loads program using exec()
Kernel schedules CPU
Command executes
Output returned to terminal
Exit status returned
Example
ls -l

Shell:

Finds /bin/ls
Creates child process
Executes binary
10. Difference between zombie and orphan processes?
Zombie Process
Process finished execution
Entry still exists in process table
Parent did not read exit status
Check Zombies
ps -el | grep Z
Orphan Process
Parent process terminated
Child adopted by PID 1 (systemd/init)
Real-Time Scenario

Improperly written application created many zombie processes causing PID exhaustion.

Fix
Restarted parent application
Developers fixed child process handling
3. File System & Storage
11. How do you find large files quickly?
find / -type f -size +1G 2>/dev/null

du -ah / | sort -rh | head -20
12. What will you do if inode usage is 100% but disk space is available?
Check Inodes
df -i
Root Cause

Too many small files.

Find Directory
for i in /*; do echo $i; find $i | wc -l; done
Real-Time Scenario

Application generated millions of temp files.

Fix
Deleted temp files
Added cleanup cron job
13. Real-time scenario where symbolic links are useful?
Example

Current app release:

/opt/app/releases/v1
/opt/app/releases/v2

Create symlink:

ln -sfn /opt/app/releases/v2 /opt/app/current

Application always points to:

/opt/app/current
Benefit

Easy rollback during deployment.

14. How do you recover a deleted file in Linux?
Case 1: File Open by Process
lsof | grep deleted

Copy from:

/proc/<PID>/fd/
Case 2

Restore from:

Backup
Snapshot
EBS snapshot
LVM snapshot
Important

Linux usually cannot recover permanently deleted files easily.

15. Difference between soft link and hard link?
Soft Link	Hard Link
Points to filename	Points to inode
Can cross filesystem	Cannot cross filesystem
Breaks if original deleted	Works even if original deleted
Commands
ln -s file softlink
ln file hardlink
Real Use Case

Soft link:

Application releases
Shared configs

Hard link:

Backup systems
File redundancy
4. Networking
16. Server is not reachable. How do you troubleshoot?
Step-by-Step
ping <server>

traceroute <server>

ip addr

ip route

ss -tulnp

systemctl status firewalld
Check Firewall
iptables -L
ufw status
Check Network Interface
ethtool eth0
Real-Time Scenario

Server unreachable after reboot.

Root Cause

Network interface not coming UP automatically.

Fix
nmcli connection up eth0
17. How do you check if a remote port is open?
telnet <ip> 443

nc -zv <ip> 443

curl -v telnet://<ip>:443
18. DNS is not resolving. What will you check?
Check DNS Config
cat /etc/resolv.conf
Test DNS
nslookup google.com
dig google.com
Check Connectivity
ping 8.8.8.8
Real-Time Scenario

DNS failed after VPN change.

Root Cause

Wrong DNS server entry.

19. Difference between netstat, ss, and lsof?
netstat

Old tool for:

Ports
Connections
Routing
ss

Modern replacement for netstat.
Faster on large systems.

ss -tulnp
lsof

Shows:

Open files
Open ports
Process mapping
lsof -i :8080
20. Application cannot connect to DB. How do you debug?
Check Connectivity
ping db-server
nc -zv db-server 3306
Check DNS
nslookup db-server
Check Firewall
iptables -L
Check DB Service
systemctl status mysql
Real-Time Scenario

Application unable to connect after security group change in AWS.

Root Cause

DB port blocked in security group.

5. Permissions & Security
21. User cannot access file though permissions look correct. Why?
Possible Reasons
Parent directory permission issue
ACL rules
SELinux context
Wrong ownership
Check
namei -l file.txt

getfacl file.txt

ls -Z file.txt
22. Difference between chmod 777 and 755?
chmod 777

Everyone:

Read
Write
Execute
Risk

Huge security risk.

chmod 755

Owner:

Read/write/execute

Others:

Read/execute only
Real-World Usage

755 used for:

Scripts
Application directories
Web servers

777 should be avoided in production.

23. How do you give sudo access safely?
visudo

Add:

username ALL=(ALL) ALL
Better Approach

Create limited sudo rules.

Example:

username ALL=(ALL) NOPASSWD:/bin/systemctl restart nginx
24. Sticky bit, SUID, SGID?
Sticky Bit

Users can delete only their own files.

chmod +t /shared

Used on:

/tmp
SUID

Runs command as file owner.

chmod u+s file

Example:

passwd command
SGID

Files inherit group ownership.

Useful in shared directories.

25. Script runs manually but fails in cron. Why?
Common Reasons
PATH variable missing
Permissions issue
Environment variables missing
Relative paths used
Fix

Use full paths.

Wrong:

python app.py

Correct:

/usr/bin/python3 /opt/app/app.py
Check Logs
grep CRON /var/log/syslog
6. Logs & Monitoring
26. Where do you check logs for system issues?
Common Locations
/var/log/messages
/var/log/syslog
/var/log/secure
/var/log/auth.log
journalctl
27. How do you monitor logs in real time?
tail -f app.log

tail -f app.log | grep ERROR

journalctl -f
28. Application logs growing rapidly. How do you manage?
Configure logrotate
/etc/logrotate.conf
/etc/logrotate.d/
Example
/var/log/app.log {
 daily
 rotate 7
 compress
 missingok
}
29. What is log rotation?
Purpose
Prevent disk full issue
Compress old logs
Retain limited logs
Importance

Critical in production systems.

30. How do you debug failed systemd service?
systemctl status app

journalctl -u app -xe

systemctl cat app
Check Config
ExecStart path
Permissions
Environment variables
7. Package & System Management
31. Check installed packages

Ubuntu:

dpkg -l

RHEL:

rpm -qa
32. Package install failed due to dependencies

Ubuntu:

apt --fix-broken install

RHEL:

yum install --skip-broken
33. Difference between apt and yum
apt	yum
Debian/Ubuntu	RHEL/CentOS
Uses .deb	Uses .rpm
dpkg backend	rpm backend
34. Update system without breaking services
Best Practice
Take backup/snapshot
Update in staging first
Use rolling updates
Restart only required services
Commands
apt update && apt upgrade

yum update
35. Check OS and kernel version
cat /etc/os-release

uname -r

hostnamectl
8. Cron & Automation
36. Cron job not running. How do you debug?
Check Cron Service
systemctl status cron
Check User Crontab
crontab -l
Check Logs
grep CRON /var/log/syslog
Common Issues
Wrong path
Missing permissions
Environment variables
37. Schedule every 5 minutes
*/5 * * * * /opt/script.sh
38. Difference between cron and at
cron	at
Repeated jobs	One-time jobs
Scheduled recurring	Scheduled once
39. Where are cron logs stored?

Ubuntu:

/var/log/syslog

RHEL:

/var/log/cron
40. Prevent overlapping cron jobs
Use flock
flock -n /tmp/job.lock /opt/job.sh
9. DevOps-Oriented Linux
41. How Linux supports Docker containers?

Linux provides:

Namespaces
cgroups
OverlayFS
Networking
Process isolation

Docker uses Linux kernel features directly.

42. What are namespaces and cgroups?
Namespaces

Provide isolation for:

PID
Network
Mount
Users
cgroups

Control:

CPU
Memory
I/O limits
43. Docker container not starting. How do you debug?
Check Logs
docker logs <container>
Check Exit Code
docker ps -a
Inspect Container
docker inspect <container>
Common Reasons
Wrong CMD
Port conflict
Missing env variable
Permission issue
44. Check resource usage of containers
docker stats

For Kubernetes:

kubectl top pod
45. How Linux handles container networking?

Linux uses:

Bridge networking
veth pairs
iptables/NAT
Network namespaces
Example

Docker default bridge:

docker0
10. Advanced / Real Interview Level
46. What happens when system runs out of memory?

Linux tries:

Free cache
Swap usage
OOM Killer triggers

Applications may:

Crash
Become slow
Get killed
47. What is OOM Killer?

OOM = Out Of Memory

Kernel kills process to protect system.

Check Logs
dmesg | grep -i kill
Real-Time Scenario

Java app consumed all memory.
Kernel killed application automatically.

48. How do you tune Linux for high traffic applications?
Tune Limits
ulimit -n
Sysctl Tuning
sysctl -a
Common Tunings
net.core.somaxconn
fs.file-max
net.ipv4.ip_local_port_range
Also
Use load balancer
Optimize app
Use caching
Enable monitoring
49. What is load average?
Example
uptime

Output:

load average: 1.20, 0.90, 0.70
Meaning
1 min
5 min
15 min averages
Interpretation

On 4 CPU server:

Load 4 = normal
Load > 4 = overloaded
50. How do you troubleshoot high I/O wait?
Check I/O
iostat -x 1
iotop
Check Disk Latency
sar -d 1
Common Reasons
Slow disks
Heavy writes
Backup jobs
DB queries
Real-Time Scenario

High iowait during backup window.

Fix
Moved backups off peak hours
Used faster SSD storage
Tuned DB queries
