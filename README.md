![Linux](https://img.shields.io/badge/Linux-Troubleshooting-blue)
![DevOps](https://img.shields.io/badge/DevOps-Interview-important)
![GitHub stars](https://img.shields.io/github/stars/iam-srikanthmergu/Linux-Interview-Scenarios) 
# Linux Real-Time Interview Scenarios

Real-world Linux troubleshooting scenarios every DevOps engineer should know before interviews.

This repository focuses on practical production-level issues, troubleshooting approaches, important Linux commands, and real-time interview preparation.

---

## Why This Repository?

Most Linux interview content available online is theoretical.

This repository is different because it focuses on:

- Real-time production scenarios
- Troubleshooting mindset
- Root cause analysis
- Commands with explanations
- Step-by-step investigation flow
- Interview-oriented answers
- Practical DevOps learning

---

# Topics Covered

| Topic | Status |
|-------|--------|
| CPU Troubleshooting | ✅ |
| Memory Issues | ✅ |
| Disk Space Issues | ✅ |
| Service Failures | Coming Soon |
| Networking Troubleshooting | Coming Soon |
| Log Analysis | Coming Soon |
| Process Management | Coming Soon |
| Shell Scripting | Coming Soon |

---

# Real-Time Linux Scenarios

## 1. Server CPU Suddenly Spikes to 100%

### Symptoms

- Server becomes slow
- APIs start timing out
- Applications become unresponsive
- High load average

---

## Step 1 — Check Real-Time CPU Usage

```bash
top
```

### Why We Use It

- Shows real-time CPU utilization
- Helps identify whether issue is temporary or continuous
- Displays top CPU-consuming processes

### Important Metrics

- `us` → User CPU usage
- `sy` → System CPU usage
- `id` → Idle CPU
- `wa` → I/O wait

---

## Step 2 — Identify Top CPU Consuming Process

```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
```

### Why We Use It

- Lists processes consuming highest CPU
- Helps identify problematic applications or scripts

---

## Step 3 — Check Load Average

```bash
uptime
```

### Why We Use It

Load average indicates how many processes are waiting for CPU resources.

Example:

```bash
load average: 8.50, 7.90, 7.20
```

If server has:
- 2 CPUs → High load
- 8 CPUs → Normal

---

## Step 4 — Check Logs

```bash
journalctl -xe
```

OR

```bash
tail -f /var/log/messages
```

### Why We Use It

- Detect service crashes
- Identify application failures
- Check system-level issues

---

## Step 5 — Verify Recent Deployments

Questions to ask:
- Was any deployment done recently?
- Any cron jobs triggered?
- Any batch jobs running?
- Any infinite loop in application?

---

## Possible Root Causes

- Infinite loop in application
- Heavy cron job
- Memory leak
- High traffic spike
- Malware/mining process
- Database query issue

---

## Temporary Fix

Kill problematic process:

```bash
kill -9 <PID>
```

Restart service if required:

```bash
systemctl restart nginx
```

---

## Permanent Fix

- Optimize application code
- Add monitoring alerts
- Configure autoscaling
- Optimize database queries
- Limit resource usage

---

# Common Linux Commands Every DevOps Engineer Must Know

| Command | Purpose |
|---------|----------|
| top | Real-time CPU usage |
| htop | Interactive process monitoring |
| free -m | Memory usage |
| df -h | Disk usage |
| du -sh | Folder size |
| ps -ef | Running processes |
| netstat -tulnp | Open ports |
| ss -tulnp | Socket statistics |
| systemctl | Manage services |
| journalctl | View logs |
| tail -f | Live log monitoring |
| grep | Search text |
| find | Locate files |
| chmod | Change permissions |
| chown | Change ownership |

---

# Repository Structure

```text
Linux-Real-Time-Interview-Scenarios/
│
├── README.md
│
├── CPU/
├── Memory/
├── Disk/
├── Networking/
├── Services/
├── Logs/
├── Scripts/
│
└── Kubernetes/
```

---

# Who Is This Repository For?

- DevOps Engineers
- Linux Administrators
- SRE Engineers
- Cloud Engineers
- Freshers preparing for interviews
- Anyone learning Linux troubleshooting

---

# Upcoming Content

- Real-time Kubernetes scenarios
- Docker troubleshooting
- Jenkins pipeline failures
- Terraform issues
- AWS production issues
- CI/CD troubleshooting
- DevSecOps interview scenarios

---

# Connect With Me

## GitHub
https://github.com/iam-srikanthmergu

---

# Support

If this repository helps you in interview preparation, consider giving it a ⭐

---

# Contributions

Feel free to contribute new real-time Linux troubleshooting scenarios.

---

# License

This project is licensed under the MIT License.
