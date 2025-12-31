# Network Observability Stack
## Metrics, Logs, and Monitoring Platform

**Author:** Alex Lux  
**Date:** December 2025  
**Environment:** Home Lab - Ubuntu VM on Proxmox  
**VM IP:** 192.168.5.34  
**Repository:** [Network-Observability-Stack](https://github.com/alexlux58/Network-Observability-Stack)

> **Quick Reference Available**: For command cheat sheets and quick troubleshooting, see `QUICK_REFERENCE.md` in this directory.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [VM Setup](#vm-setup)
5. [Installation](#installation)
6. [Configuration](#configuration)
7. [Access & Usage](#access--usage)
8. [Data Retention & Cleanup](#data-retention--cleanup)
9. [Maintenance & Operations](#maintenance--operations)
10. [Troubleshooting](#troubleshooting)
11. [Integration with Other Lab Services](#integration-with-other-lab-services)

---

## Project Overview

### Objective
Deploy a comprehensive observability stack using Docker Compose to collect, process, store, and visualize metrics and logs from the entire home lab infrastructure. This stack provides unified monitoring and logging capabilities across all lab services.

### Why This Stack?

**Prometheus:**
- Time-series metrics database
- Pull-based metrics collection
- Powerful query language (PromQL)
- Alerting capabilities

**Grafana:**
- Visualization and dashboards
- Multiple data source support
- Alerting and notifications
- User management

**Elasticsearch + Kibana:**
- Centralized log storage and search
- Real-time log analysis
- Index lifecycle management
- Advanced query capabilities

**Logstash + Filebeat:**
- Log collection and processing
- Log parsing and enrichment
- Multiple input/output support

**Alertmanager:**
- Alert routing and grouping
- Notification channels
- Alert deduplication

**Exporters:**
- **Node Exporter**: Host metrics
- **cAdvisor**: Container metrics
- **Blackbox Exporter**: Uptime and endpoint monitoring

### Use Cases

- **Infrastructure Monitoring**: Track CPU, memory, disk, network across all VMs
- **Application Monitoring**: Monitor service health and performance
- **Log Aggregation**: Centralized log collection and analysis
- **Alerting**: Proactive notification of issues
- **Capacity Planning**: Historical data for resource planning
- **Troubleshooting**: Correlate metrics and logs for root cause analysis

---

## Architecture Overview

### Deployment Architecture

```
Proxmox Host (192.168.4.146)
   ↓
Ubuntu VM (192.168.5.34)
   ↓
Docker Compose Stack
   ├── Prometheus (Metrics Collection)
   ├── Grafana (Visualization)
   ├── Elasticsearch (Log Storage)
   ├── Kibana (Log Visualization)
   ├── Logstash (Log Processing)
   ├── Filebeat (Log Collection)
   ├── Alertmanager (Alert Routing)
   ├── Node Exporter (Host Metrics)
   ├── cAdvisor (Container Metrics)
   └── Blackbox Exporter (Uptime Monitoring)
```

### Network Configuration

- **VM IP**: 192.168.5.34 (Ubuntu VM on Proxmox)
- **Grafana Port**: 3000 (default)
- **Kibana Port**: 5601 (default)
- **Prometheus Port**: 9090 (default)
- **Elasticsearch Port**: 9200 (default)
- **Access**: LAN only (no WAN exposure recommended)

### Service Ports

| Service | Port | Protocol | Exposure |
|---------|------|----------|----------|
| Grafana | 3000 | TCP | LAN only |
| Kibana | 5601 | TCP | LAN only |
| Prometheus | 9090 | TCP | LAN only |
| Elasticsearch | 9200 | TCP | LAN only |
| Alertmanager | 9093 | TCP | LAN only |
| cAdvisor | 8080 | TCP | LAN only |
| Node Exporter | 9100 | TCP | LAN only |
| Blackbox Exporter | 9115 | TCP | LAN only |

### Repository Structure

```
Network-Observability-Stack/
├── docker-compose.yml        # Main compose file
├── manage.sh                 # Unified management script
├── setup.sh                  # Initial setup script
├── install-node-exporter.sh  # Script to install monitoring on remote VMs
├── .env.example             # Environment variable template
├── alertmanager/
│   └── alertmanager.yml     # Alert routing config
├── blackbox/
│   └── blackbox.yml         # Blackbox exporter config
├── filebeat/
│   └── filebeat.yml        # Log collection config
├── logstash/
│   └── pipeline/
│       └── logstash.conf   # Log processing pipeline
└── prometheus/
    ├── prometheus.yml      # Scrape config
    └── alerts.yml          # Alert rules
```

---

## Prerequisites

### Software Requirements

- **Ubuntu Server**: 20.04 LTS or 22.04 LTS (recommended)
- **Docker**: Version 20.10.10 or higher
- **Docker Compose**: Version 1.28.0 or higher (or Docker Compose V2)
- **Git**: For cloning the repository

### Hardware Requirements

- **VM RAM**: 8GB minimum (16GB+ recommended)
- **VM Storage**: 100GB+ (for metrics and log storage)
- **CPU**: 4+ cores recommended

### Network Requirements

- Internet access (for Docker image downloads)
- Network connectivity to monitored hosts
- SSH access to remote hosts (for Node Exporter installation)

---

## VM Setup

### Create Ubuntu VM on Proxmox

1. **Download Ubuntu Server ISO**
   - Ubuntu 22.04 LTS recommended

2. **Create VM in Proxmox**
   - **Name**: Observability-Stack or Monitoring-Stack
   - **OS Type**: Linux
   - **RAM**: 8GB minimum (16GB recommended)
   - **CPU**: 4+ cores
   - **Storage**: 100GB+ (SSD recommended)
   - **Network**: Bridge to vmbr0 (management network)

3. **Install Ubuntu Server**
   - Follow standard Ubuntu installation
   - Enable SSH server
   - Set static IP: 192.168.5.34/22
   - Or configure DHCP reservation in router

### Install Docker and Docker Compose

```bash
# Update system
sudo apt-get update
sudo apt-get -y upgrade

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose (if not included)
sudo apt-get install -y docker-compose-plugin

# Verify installation
docker --version
docker compose version

# Log out and back in for group changes to take effect
```

---

## Installation

### Clone Repository

```bash
# Clone the repository
git clone https://github.com/alexlux58/Network-Observability-Stack.git
cd Network-Observability-Stack
```

### Initial Setup

**Run setup script:**
```bash
# Make scripts executable
chmod +x *.sh

# Run setup script
./setup.sh
```

**Or manual setup:**
```bash
# Create environment file
cp .env.example .env

# Edit .env file with your settings
nano .env

# Create data directories
mkdir -p data/{prometheus,grafana,elasticsearch,logstash,filebeat,alertmanager}
```

### Start Services

**Using management script:**
```bash
# Start all services
./manage.sh start

# Check status
./manage.sh status

# View logs
./manage.sh logs
```

**Using Docker Compose:**
```bash
# Start all services
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f
```

### Verify Installation

**Check all services are running:**
```bash
docker compose ps
```

**Expected services:**
- prometheus
- grafana
- elasticsearch
- kibana
- logstash
- filebeat
- alertmanager
- cadvisor
- node-exporter
- blackbox-exporter

**Test service endpoints:**
```bash
# Grafana
curl http://localhost:3000

# Kibana
curl http://localhost:5601

# Prometheus
curl http://localhost:9090/api/v1/status/config

# Elasticsearch
curl http://localhost:9200
```

---

## Configuration

### Environment Variables

**Edit `.env` file:**
```bash
# Grafana
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=admin

# Elasticsearch
ELASTICSEARCH_PASSWORD=changeme

# Timezone
TZ=America/New_York
```

### Prometheus Configuration

**Edit `prometheus/prometheus.yml`:**

Add targets to scrape:
```yaml
scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
  
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
  
  - job_name: 'blackbox'
    static_configs:
      - targets: ['blackbox-exporter:9115']
  
  # Add your lab services
  - job_name: 'security-onion'
    static_configs:
      - targets: ['192.168.4.155:443']
  
  - job_name: 'pivpn'
    static_configs:
      - targets: ['192.168.4.127:80']
```

### Grafana Data Sources

**Add data sources in Grafana UI:**

1. **Prometheus:**
   - URL: `http://prometheus:9090`
   - Access: Server (default)

2. **Elasticsearch:**
   - URL: `http://elasticsearch:9200`
   - Index name: `filebeat-*`
   - Time field: `@timestamp`

### Install Node Exporter on Remote Hosts

**Use the provided script:**
```bash
# Install on remote host
./install-node-exporter.sh <remote-host-ip> <ssh-user>

# Example
./install-node-exporter.sh 192.168.4.155 ubuntu
```

**Manual installation:**
```bash
# SSH to remote host
ssh user@remote-host

# Download and install Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvfz node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

**Add to Prometheus config:**
```yaml
- job_name: 'remote-node'
  static_configs:
    - targets: ['192.168.4.155:9100']
```

### Log Collection Configuration

**Filebeat configuration (`filebeat/filebeat.yml`):**

Collect system logs:
```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/syslog
      - /var/log/auth.log
    fields:
      log_type: system
```

**Logstash pipeline (`logstash/pipeline/logstash.conf`):**

Process and enrich logs:
```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][log_type] == "system" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:timestamp} %{IPORHOST:host} %{PROG:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:message}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "filebeat-%{+YYYY.MM.dd}"
  }
}
```

---

## Access & Usage

### Web Access

**Grafana:**
- URL: `http://192.168.5.34:3000`
- Default login: `admin` / `admin` (change immediately!)

**Kibana:**
- URL: `http://192.168.5.34:5601`
- No authentication by default (configure security for production)

**Prometheus:**
- URL: `http://192.168.5.34:9090`
- Query interface and targets status

**Alertmanager:**
- URL: `http://192.168.5.34:9093`
- Alert status and routing

### Initial Grafana Setup

1. **Login** with default credentials
2. **Change password** when prompted
3. **Add data sources:**
   - Prometheus: `http://prometheus:9090`
   - Elasticsearch: `http://elasticsearch:9200`
4. **Import dashboards:**
   - Node Exporter Full: Dashboard ID `1860`
   - Docker Container Stats: Dashboard ID `6417`
   - Prometheus 2.0 Stats: Dashboard ID `893`

### Useful Grafana Dashboard IDs

**Host Metrics:**
- **1860** - Node Exporter Full (comprehensive host metrics)
- **11074** - Node Exporter for Prometheus (modern React-based)

**Container Metrics:**
- **6417** - Docker Container Stats
- **11074** - Node Exporter for Prometheus (includes container metrics)

**Prometheus Monitoring:**
- **893** - Prometheus 2.0 Stats

**Browse more:** [Grafana Dashboards](https://grafana.com/grafana/dashboards/)

### Kibana Setup

1. **Create index pattern:**
   - Go to Management > Index Patterns
   - Pattern: `filebeat-*`
   - Time field: `@timestamp`

2. **Explore logs:**
   - Go to Discover
   - Select `filebeat-*` index pattern
   - View and search logs

3. **Create visualizations:**
   - Go to Visualize
   - Create charts and graphs from log data

### Management Script Usage

**Available commands:**
```bash
# Start all services
./manage.sh start

# Stop all services
./manage.sh stop

# Restart all services
./manage.sh restart

# View status
./manage.sh status

# View logs
./manage.sh logs [service-name]

# Clean containers and images (keeps volumes)
./manage.sh clean

# User management (Grafana)
./manage.sh user create <username> <password> <role>
./manage.sh user delete <username>
```

---

## Data Retention & Cleanup

### Log and Data Retention

**Docker Container Logs:**
- **Rotation:** Automatic via Docker logging driver
- **Per service:** Max 10MB per log file, keeps 3 files
- **Total per service:** ~30MB maximum
- **Configuration:** Set in `docker-compose.yml` logging section

**Elasticsearch Log Data:**
- **Retention:** 14 days (configurable via ILM policy)
- **Auto-deletion:** Old indices automatically deleted after retention period
- **Storage:** `./data/elasticsearch/` directory

**Prometheus Metrics:**
- **Retention:** 15 days OR 8GB (whichever comes first)
- **Auto-cleanup:** Prometheus automatically deletes old data
- **Storage:** `./data/prometheus/` directory

### Configure Elasticsearch Retention

**Set replicas to 0 (single node friendly):**
```bash
curl -s -X PUT "http://localhost:9200/_index_template/filebeat-template" \
  -H 'Content-Type: application/json' -d '{
    "index_patterns": ["filebeat-*"],
    "template": { "settings": { "index.number_of_replicas": 0 } },
    "priority": 500
  }'
```

**Configure 14-day retention:**
```bash
# Create ILM policy (one-time setup)
curl -s -X PUT "http://localhost:9200/_ilm/policy/filebeat-retain-14d" \
  -H 'Content-Type: application/json' -d '{
    "policy": {
      "phases": {
        "hot": { "actions": {} },
        "delete": { "min_age": "14d", "actions": { "delete": {} } }
      }
    }
  }'

# Apply to template
curl -s -X PUT "http://localhost:9200/_index_template/filebeat-template" \
  -H 'Content-Type: application/json' -d '{
    "index_patterns": ["filebeat-*"],
    "template": {
      "settings": {
        "index.number_of_replicas": 0,
        "index.lifecycle.name": "filebeat-retain-14d"
      }
    },
    "priority": 500
  }'
```

**Verify retention:**
```bash
# Check ILM policy
curl -s http://localhost:9200/_ilm/policy/filebeat-retain-14d?pretty

# Check index template
curl -s http://localhost:9200/_index_template/filebeat-template?pretty

# Monitor index lifecycle status
curl -s 'http://localhost:9200/_cat/indices/filebeat-*?v&h=index,creation.date,status'
```

### Check Disk Usage

```bash
# Check data directory sizes
du -sh ./data/*

# Check Docker log sizes
docker system df

# Check specific service logs
docker inspect <container-name> | grep -A 5 LogPath
```

---

## Maintenance & Operations

### Service Management

**Start/Stop/Restart:**
```bash
# Using management script
./manage.sh start
./manage.sh stop
./manage.sh restart

# Using Docker Compose
docker compose start
docker compose stop
docker compose restart
```

**View Logs:**
```bash
# All services
./manage.sh logs

# Specific service
./manage.sh logs prometheus
docker compose logs -f grafana
```

**Check Status:**
```bash
./manage.sh status
docker compose ps
```

### Updates

**Update services:**
```bash
# Stop services
./manage.sh stop

# Pull new images
docker compose pull

# Start services
./manage.sh start
```

### Backup and Restore

**Backup Elasticsearch indices:**
```bash
# Create snapshot repository
curl -X PUT "http://localhost:9200/_snapshot/backup" -H 'Content-Type: application/json' -d '{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/backup"
  }
}'

# Create snapshot
curl -X PUT "http://localhost:9200/_snapshot/backup/snapshot_1?wait_for_completion=true"
```

**Backup Grafana:**
```bash
# Export dashboards as JSON from Grafana UI
# Or backup data directory
tar -czf grafana-backup-$(date +%Y%m%d).tar.gz ./data/grafana
```

**Backup Prometheus:**
```bash
# Backup data directory
tar -czf prometheus-backup-$(date +%Y%m%d).tar.gz ./data/prometheus
```

### Cleanup

**Clean containers and images (keeps volumes):**
```bash
./manage.sh clean
```

**Remove everything including volumes (DANGEROUS):**
```bash
docker compose down -v
docker system prune -a -f
docker volume prune -f
```

---

## Troubleshooting

### Services Won't Start

**Check logs:**
```bash
docker compose logs
./manage.sh logs
```

**Check Docker status:**
```bash
docker ps -a
docker compose ps
```

**Verify ports aren't in use:**
```bash
sudo ss -lntup | grep -E '3000|5601|9090|9200'
```

### Elasticsearch Issues

**Out of memory:**
```bash
# Check memory usage
docker stats elasticsearch

# Increase VM memory or adjust Elasticsearch heap
# Edit docker-compose.yml: ES_JAVA_OPTS="-Xms2g -Xmx2g"
```

**Disk space:**
```bash
# Check disk usage
df -h
du -sh ./data/elasticsearch

# Clean old indices
curl -X DELETE "http://localhost:9200/filebeat-YYYY.MM.DD"
```

### Prometheus Issues

**Targets not scraping:**
```bash
# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# Verify network connectivity
docker compose exec prometheus ping <target-host>
```

**High memory usage:**
```bash
# Check retention settings
# Edit docker-compose.yml: --storage.tsdb.retention.time=15d
```

### Grafana Issues

**Can't connect to data sources:**
```bash
# Verify services are running
docker compose ps

# Check network connectivity
docker compose exec grafana ping prometheus
docker compose exec grafana ping elasticsearch
```

**Dashboards not loading:**
- Check data source connectivity
- Verify time range selection
- Check dashboard JSON for errors

### Log Collection Issues

**Filebeat not collecting logs:**
```bash
# Check Filebeat logs
docker compose logs filebeat

# Verify log paths in filebeat.yml
# Check file permissions
ls -la /var/log/syslog
```

**Logs not appearing in Kibana:**
```bash
# Check Logstash logs
docker compose logs logstash

# Verify Elasticsearch indices
curl http://localhost:9200/_cat/indices/filebeat-*
```

---

## Integration with Other Lab Services

### Monitor Security Onion

**Add to Prometheus:**
```yaml
- job_name: 'security-onion'
  static_configs:
    - targets: ['192.168.4.155:443']
```

**Install Node Exporter:**
```bash
./install-node-exporter.sh 192.168.4.155 ubuntu
```

### Monitor PiVPN/Pi-hole

**Add to Prometheus:**
```yaml
- job_name: 'pihole'
  static_configs:
    - targets: ['192.168.4.127:80']
```

**Monitor Pi-hole metrics:**
- Use Pi-hole Exporter or custom metrics endpoint
- Create Grafana dashboard for DNS queries

### Monitor NSOT (NetBox/Nautobot)

**Add to Prometheus:**
```yaml
- job_name: 'netbox'
  static_configs:
    - targets: ['192.168.5.9:8000']
```

**Monitor Docker containers:**
- cAdvisor automatically collects container metrics
- View in Grafana with container dashboards

### Monitor DevOps Tools VM

**Add to Prometheus:**
```yaml
- job_name: 'devops-tools'
  static_configs:
    - targets: ['192.168.5.8:9100']  # Node Exporter
```

**Collect Jenkins logs:**
```yaml
# Add to filebeat.yml
- type: log
  paths:
    - /var/lib/jenkins/logs/*.log
```

### Complete Monitoring Coverage

**All Lab Services:**
- Security Onion VM: Node Exporter + custom metrics
- PiVPN Host: Node Exporter + Pi-hole metrics
- NSOT VM: Node Exporter + Docker metrics
- DevOps Tools VM: Node Exporter + Jenkins metrics
- Observability Stack VM: Self-monitoring via Node Exporter

**Create unified dashboard:**
- Combine all metrics in Grafana
- Create overview dashboard showing all services
- Set up alerts for critical services

---

## Key Takeaways

### Critical Success Factors

1. **Resource Allocation**: Sufficient RAM (8GB+) and storage (100GB+)
2. **Data Retention**: Configure ILM policies to prevent disk space issues
3. **Network Connectivity**: Ensure all monitored hosts are reachable
4. **Security**: Change default passwords and enable authentication
5. **Backups**: Regular backups of Grafana dashboards and configurations

### Common Pitfalls

1. **Disk Space**: Elasticsearch and Prometheus can fill disk quickly
2. **Memory Usage**: Elasticsearch requires significant RAM
3. **Network Issues**: Targets not reachable from Prometheus
4. **Default Credentials**: Forgetting to change admin passwords
5. **No Retention Policy**: Data growing indefinitely

### Best Practices

1. **Configure Retention**: Set appropriate retention periods
2. **Monitor Disk Usage**: Regular checks and cleanup
3. **Use Management Script**: Consistent operations via `manage.sh`
4. **Version Control**: Store configurations in Git
5. **Documentation**: Document custom dashboards and alerts

---

## Conclusion

This lab successfully deployed a comprehensive observability stack providing metrics collection, log aggregation, and visualization capabilities for the entire home lab infrastructure.

**Final Status:**
- ✅ Prometheus collecting metrics
- ✅ Grafana visualizing data
- ✅ Elasticsearch storing logs
- ✅ Kibana analyzing logs
- ✅ All exporters running
- ✅ Alertmanager configured

**Next Steps:**
- Create custom dashboards for lab services
- Set up alerting rules
- Integrate with all lab services
- Configure log collection from all hosts
- Set up automated backups

---

## Additional Resources

### Official Documentation

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Elasticsearch Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Kibana Documentation](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Filebeat Documentation](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)

### Repository

- [Network-Observability-Stack GitHub](https://github.com/alexlux58/Network-Observability-Stack)

---

## Related Projects

This project monitors and integrates with all lab services:

- **Security Onion**: Monitor VM metrics and correlate alerts with infrastructure health
- **PiVPN/Pi-hole**: Collect DNS query metrics and visualize in Grafana
- **NSOT (NetBox/Nautobot)**: Monitor Docker container metrics for NetBox and Nautobot
- **DevOps Tools**: Monitor Jenkins build metrics and Ansible execution times
- **AWS Services**: Monitor CloudWatch metrics for AWS Game Server, NetKnife, and Cost Alerting

For complete lab documentation, see `README.md` in the HomeLabWriteup directory.

---

**End of Lab Writeup**

