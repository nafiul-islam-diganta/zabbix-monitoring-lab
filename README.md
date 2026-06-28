# Zabbix 7.0 LTS — Infrastructure Monitoring Lab

A fully self-deployed network monitoring stack built from scratch on Linux,
monitoring a heterogeneous environment (Ubuntu Server + Windows 11) using
Zabbix 7.0 LTS.

> Not a tutorial follow-along. Built independently with real errors diagnosed
> and fixed through server logs.

---

## 📸 Dashboard

<!-- Add dashboard screenshot here after upload -->

---

## 🖥️ Stack

| Component | Details |
|---|---|
| Monitoring platform | Zabbix 7.0.27 LTS |
| Server OS | Ubuntu Server 24.04 LTS |
| Database | MySQL 8 |
| Web server | Nginx + PHP 8.3-FPM |
| Hypervisor | VMware Workstation Pro |
| Monitored devices | Ubuntu Server VM + Windows 11 laptop |

---

## 📡 What's Monitored

### Device 1 — Ubuntu Server 24.04 (Linux VM)
- 75+ auto-discovered metrics via Low-Level Discovery (LLD)
- CPU utilization, memory availability, disk space
- Network interface traffic (ens33)
- SNMP — queried real OIDs including system uptime
  (OID: 1.3.6.1.2.1.1.3.0)
- TCP service availability check (Simple Check)

### Device 2 — Windows 11 Laptop
- 161 auto-discovered metrics
- CPU, memory, 3 drives (C:/D:/Y:)
- 78 Windows services monitored (Kaspersky,
  Windows Defender, DNS Client, Task Scheduler etc.)
- Network interface — Qualcomm Atheros QCA9377 Wi-Fi adapter
- OS version, uptime, running processes, logged-in users

### Total
- 392 items collecting live data
- 226 triggers configured
- 3 hosts monitored (including SNMP interface)

---

## ⚡ Key Features Built

### 1. Three Monitoring Types
- **Zabbix Agent** — Linux VM and Windows laptop
- **SNMP** — installed snmpd, queried real OIDs
- **Simple Check** — TCP port monitoring for
  external service availability

### 2. Smart Triggers
Built trigger expressions using `avg()` over time
windows instead of `last()` — prevents false alarms
from momentary spikes. Critical distinction for real
NOC environments.

### 3. Reusable Templates
Created custom host template with `{HOST.CONN}`
macro — automatically adapts to any host it's
applied to without hardcoding IPs.

### 4. Full Email Alert Pipeline
Trigger fires → Action evaluates → Gmail SMTP → Inbox
Tested live — received real email alert:
"Problem: Google is unreachable — Severity: High"

### 5. Role-Based Access Control (RBAC)
| Role | Access |
|---|---|
| Super Admin | Full configuration access |
| NOC Analyst | Read-only, scoped to Linux servers host group |

### 6. NOC Dashboard
Single dashboard showing:
- CPU utilization — Linux VM (10 days history)
- Memory availability — Linux VM
- CPU utilization — Windows 11 laptop
- Host availability matrix (Agent + SNMP)
- Active problems panel with severity coding
- System health: 5.92 new values/sec

---

## 🔧 Real Errors Hit & Fixed

| Error | Cause | Fix |
|---|---|---|
| Dependency failures | Wrong Ubuntu version (26.04 vs 24.04) | Rebuilt VM with correct 24.04 LTS ISO |
| MySQL access denied | Password mismatch in zabbix_server.conf | Reset via ALTER USER, updated DBPassword= |
| Missing packages | universe/multiverse repos not enabled | sudo add-apt-repository universe multiverse |
| No IPv4 on VM | VMware set to NAT not Bridged | Configured port forwarding via VMware NAT settings |

---

## 📁 Repository Structure
zabbix-monitoring-lab/

│

├── screenshots/

│   ├── dashboard.png

│   ├── email-alert.png

│   ├── snmp-terminal.png

│   ├── server-status.png

│   └── windows-latest-data.png

│

├── config/

│   ├── zabbix_server.conf (sanitised)

│   └── nginx.conf

│

└── README.md

---

## 📸 Screenshots

### Server Status (Active & Running)
<!-- Add active.png here -->

### SNMP Terminal Output
<!-- Add CLI_image.png here -->

### Email Alert Received
<!-- Add email.png here -->

### Windows Laptop — 161 Metrics
<!-- Add laptop_2_data.png here -->

---

## 🛠️ Installation Summary

Full step-by-step install on Ubuntu 24.04:

```bash
# 1. Add Zabbix repository
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/\
zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
sudo apt update

# 2. Install Zabbix components
sudo apt install zabbix-server-mysql zabbix-frontend-php \
zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2 -y

# 3. Create database
sudo mysql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user 'zabbix'@'localhost' identified by 'yourpassword';
grant all privileges on zabbix.* to 'zabbix'@'localhost';
set global log_bin_trust_function_creators = 1;
quit;

# 4. Import schema
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz \
| mysql -u zabbix -p zabbix

# 5. Start services
sudo systemctl restart zabbix-server zabbix-agent2 nginx php8.3-fpm
sudo systemctl enable zabbix-server zabbix-agent2 nginx php8.3-fpm
```

---

## 👤 Author

**Nafiul Islam Diganta**
ECE Graduate — BRAC University, Dhaka
[LinkedIn](https://linkedin.com/in/nafiul-islam-diganta)

---

*Built June 2026 | Zabbix 7.0.27 LTS*
