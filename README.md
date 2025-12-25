# ELK SIEM with Winlogbeat (Home Lab)

## Overview
This project demonstrates a working SIEM setup using the ELK Stack
(Elasticsearch, Logstash, Kibana) with Winlogbeat forwarding Windows
event logs via Logstash into Elasticsearch for analysis.

## Architecture
Windows Machine → Winlogbeat → Logstash → Elasticsearch → Kibana

## Tools Used
- Docker & Docker Compose
- Elasticsearch 8.x
- Logstash 8.x
- Kibana 8.x
- Winlogbeat
- Ubuntu Desktop (VMware)
- Windows 11 (log source)

## Setup Summary
1. Deployed ELK stack using Docker Compose
2. Configured Logstash to accept Beats input on port 5044
3. Installed and configured Winlogbeat on Windows
4. Forwarded Windows logs to Logstash
5. Verified ingestion in Kibana Discover

## Verification
- Docker containers running successfully
- Winlogbeat Windows service running
- Logs visible in Kibana Discover (`beats-*` index)
- Windows Security, System, and Application logs parsed correctly

## Screenshots
See `/screenshots` directory for validation evidence.

## Learning Outcomes
- SIEM deployment and troubleshooting
- Log ingestion pipelines
- Beats → Logstash → Elasticsearch flow
- Kibana data views and log analysis

## Author
Akshay
