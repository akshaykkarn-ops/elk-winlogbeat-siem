# ELK SIEM with Winlogbeat (Windows Logs → Docker ELK)

## Overview

This project documents the **end-to-end deployment, troubleshooting, and final working solution** for forwarding **Windows Event Logs** to an **ELK Stack (Elasticsearch, Logstash, Kibana)** using **Winlogbeat**.

The setup was **not straightforward** and involved multiple real-world issues across:

* Docker vs system services
* Logstash input binding
* Winlogbeat service installation on Windows
* Execution policy restrictions
* Index pattern mismatches in Kibana
* Git repository structure problems

This repository captures **what broke, why it broke, and how it was fixed** — exactly how SOC and Blue Team work happens in practice.

---

## Architecture

**Ubuntu VM (VMware Workstation)**

* Docker
* Elasticsearch (container)
* Logstash (container)
* Kibana (container)

**Windows 11 Host**

* Winlogbeat (Windows service)
* Logs → Logstash → Elasticsearch → Kibana

---

## ELK Stack Deployment (Docker)

### Why Docker?

* Avoids dependency conflicts
* No systemd/service issues
* Easier troubleshooting and portability

### Running Containers

```bash
docker ps
```

Confirmed services:

* Elasticsearch → `9200`
* Logstash → `5044`
* Kibana → `5601`

---

## Logstash Troubleshooting (Port 5044)

### Problem

* No service listening on `5044`
* `systemctl restart logstash` failed
* Confusion between Linux service and Docker container

### Root Cause

Logstash was running **inside Docker**, not as a system service.

### Fix

* Stop using `systemctl`
* Validate container:

```bash
docker ps | grep logstash
```

* Check logs:

```bash
docker logs logstash --tail 50
```

### Working Logstash Configuration

```conf
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "beats-%{+YYYY.MM.dd}"
  }
}
```

✅ Port `5044` successfully opened after container restart.

---

## Winlogbeat Service Installation (Windows)

### Problems Faced

* `Start-Service winlogbeat` → service not found
* `winlogbeat.exe install` → unknown command
* PowerShell execution policy blocked scripts

### Root Cause

* Winlogbeat **does not install services via the binary**
* Service installation is handled by a **PowerShell script**
* Script execution was blocked by policy

### Final Working Fix

Run **PowerShell as Administrator**:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
cd "C:\Program Files\Winlogbeat"
.\install-service-winlogbeat.ps1
Start-Service winlogbeat
Get-Service winlogbeat
```

Expected output:

```
Status : Running
Name   : winlogbeat
```

---

## Winlogbeat Configuration (Critical Fix)

### Problem

* Winlogbeat running
* Logstash running
* No logs visible in Kibana

### Root Cause

Logs were indexed as:

```
beats-YYYY.MM.DD
```

not `winlogbeat-*`.

### Working `winlogbeat.yml` (Key Sections)

```yaml
output.logstash:
  hosts: ["192.168.198.132:5044"]

setup.kibana:
  host: "http://192.168.198.132:5601"
```

❌ `output.elasticsearch` disabled
✅ Logs forwarded **only via Logstash**

---

## Kibana Data View Issue

### Problem

* Kibana showed “Create a data view”
* `winlogbeat-*` returned no matches

### Root Cause

Index name mismatch.

### Fix

Create data view:

```
beats-*
```

Timestamp field:

```
@timestamp
```

✅ Logs appeared immediately in **Discover**.

---

## Git Repository Issue (Embedded Repository)

### Problem

```text
warning: adding embedded git repository: winlogbeat
```

### Root Cause

The `winlogbeat/` directory already contained a `.git` folder.

### Fix

```bash
rm -rf winlogbeat/.git
git add .
git commit -m "Add Docker ELK, Logstash config, Winlogbeat setup and screenshots"
```

---

## Validation Checklist

* ✅ Elasticsearch running
* ✅ Logstash listening on `5044`
* ✅ Winlogbeat service running on Windows
* ✅ Logs visible in Kibana Discover
* ✅ Correct data view configured
* ✅ Repository clean and documented

---

## Key Learnings

* ELK failures are usually **configuration and architecture issues**
* Docker vs systemd confusion is a common pitfall
* Winlogbeat service installation is **PowerShell-script based**
* Index patterns must match **actual indices**
* Troubleshooting skill matters more than “happy-path” setups

---

## Future Improvements

* Sysmon integration
* ECS field normalization
* Detection rules in Kibana
* Alerting with Kibana rules or ElastAlert

---

## Author

**Akshay**
SOC / Blue Team focused cybersecurity practitioner
Hands-on experience with SIEM deployment, log pipelines, and real troubleshooting
