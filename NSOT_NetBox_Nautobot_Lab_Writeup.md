# Network Source of Truth (NSOT) - NetBox & Nautobot
## IPAM/DCIM Infrastructure Documentation Platform

**Author:** Alex Lux  
**Date:** December 2025  
**Environment:** Home Lab - Ubuntu VM on Proxmox  
**VM IP:** 192.168.5.9  
**Repository:** [NSOT-Network-Source-of-Truth](https://github.com/alexlux58/NSOT-Network-Source-of-Truth)

> **Quick Reference Available**: For command cheat sheets and quick troubleshooting, see `QUICK_REFERENCE.md` in this directory.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [VM Setup](#vm-setup)
5. [NetBox Deployment](#netbox-deployment)
6. [Nautobot Deployment](#nautobot-deployment)
7. [Configuration](#configuration)
8. [Access & Usage](#access--usage)
9. [Maintenance & Operations](#maintenance--operations)
10. [Troubleshooting](#troubleshooting)
11. [Integration with Other Lab Services](#integration-with-other-lab-services)

---

## Project Overview

### Objective
Deploy containerized Network Source of Truth (NSOT) platforms using Docker to document and manage network infrastructure, IP addresses, devices, circuits, and more. This project provides both NetBox and Nautobot as Docker-based solutions for network infrastructure management.

### Why NetBox & Nautobot?

**NetBox:**
- Open-source IP address management (IPAM) and data center infrastructure management (DCIM) tool
- Industry-standard for network documentation
- Extensive API for automation
- Active community and regular updates

**Nautobot:**
- Network Source of Truth platform (originally forked from NetBox)
- Enhanced extensibility and plugin architecture
- Modern development practices
- Strong focus on automation and integration

### Use Cases

- **IP Address Management**: Track and manage IP address allocations
- **Device Documentation**: Document network devices, their locations, and connections
- **Circuit Management**: Track network circuits and providers
- **Cable Management**: Document physical cable connections
- **Network Automation**: Source of truth for automation tools
- **Change Management**: Track network changes and configurations

---

## Architecture Overview

### Deployment Architecture

```
Proxmox Host (192.168.4.146)
   ↓
Ubuntu VM (192.168.5.9)
   ↓
Docker Engine
   ├── NetBox Stack
   │   ├── netbox (Web Application)
   │   ├── netbox-worker (Background Tasks)
   │   ├── postgres (Database)
   │   ├── redis (Task Queue)
   │   └── redis-cache (Caching)
   │
   └── Nautobot Stack
       ├── nautobot (Web Application)
       ├── nautobot-worker (Background Tasks)
       ├── nautobot-beat (Scheduler)
       ├── postgres (Database)
       └── redis (Task Queue & Cache)
```

### Network Configuration

- **VM IP**: 192.168.5.9 (Ubuntu VM on Proxmox)
- **NetBox Port**: 8000 (default, configurable)
- **Nautobot Port**: 8081 (default, configurable)
- **Access**: LAN only (no WAN exposure)

### Repository Structure

```
NSOT-Network-Source-of-Truth/
├── netbox-docker/          # NetBox Docker deployment
│   ├── configuration/      # NetBox configuration files
│   ├── docker/            # Docker-specific configuration
│   ├── env/               # Environment variable files
│   ├── docker-compose.yml # NetBox Docker Compose configuration
│   └── Dockerfile         # Custom NetBox Docker image build
│
└── nautobot-docker/        # Nautobot Docker deployment
    ├── env/               # Environment variable files
    └── docker-compose.yml # Nautobot Docker Compose configuration
```

---

## Prerequisites

### Software Requirements

- **Docker**: Version 20.10.10 or higher
- **Docker Compose**: Version 1.28.0 or higher (or Docker Compose V2)
- **containerd**: Version 1.5.6 or higher (if using containerd)

### Verify Installation

```bash
docker --version
docker compose version
```

### Hardware Requirements

- **VM RAM**: 4GB minimum (8GB+ recommended)
- **VM Storage**: 50GB+ (for databases and media files)
- **CPU**: 2+ cores recommended

---

## VM Setup

### Create Ubuntu VM on Proxmox

1. **Download Ubuntu Server ISO**
   - Ubuntu 22.04 LTS or 24.04 LTS recommended

2. **Create VM in Proxmox**
   - **Name**: NSOT or Network-Source-of-Truth
   - **OS Type**: Linux
   - **RAM**: 4GB minimum (8GB recommended)
   - **CPU**: 2+ cores
   - **Storage**: 50GB+ (SSD recommended)
   - **Network**: Bridge to vmbr0 (management network)

3. **Install Ubuntu Server**
   - Follow standard Ubuntu installation
   - Enable SSH server
   - Set static IP: 192.168.5.9/22
   - Or configure DHCP reservation in router

### Install Docker

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

### Clone Repository

```bash
# Clone the repository
git clone https://github.com/alexlux58/NSOT-Network-Source-of-Truth.git
cd NSOT-Network-Source-of-Truth
```

---

## NetBox Deployment

### Quick Start

1. **Navigate to NetBox directory:**
   ```bash
   cd netbox-docker
   ```

2. **Create port mapping override (optional, for local access):**
   ```bash
   tee docker-compose.override.yml <<EOF
   services:
     netbox:
       ports:
         - 8000:8080
   EOF
   ```

3. **Pull latest images and start services:**
   ```bash
   docker compose pull
   docker compose up -d
   ```

4. **Wait for services to start** (may take 2-3 minutes):
   ```bash
   docker compose ps
   ```

5. **Access NetBox:**
   - URL: `http://192.168.5.9:8000`
   - Default credentials: admin/admin (change immediately!)

6. **Create admin user:**
   ```bash
   docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser
   ```

### Verify NetBox Services

```bash
# Check all services are running
docker compose ps

# View logs
docker compose logs -f netbox

# Check database connection
docker compose exec netbox /opt/netbox/netbox/manage.py dbshell
```

**Expected services:**
- `netbox`: Web application
- `netbox-worker`: Background task worker
- `postgres`: Database
- `redis`: Task queue
- `redis-cache`: Caching

---

## Nautobot Deployment

### Quick Start

1. **Navigate to Nautobot directory:**
   ```bash
   cd nautobot-docker
   ```

2. **Create environment file (if not already present):**
   ```bash
   mkdir -p env
   # Create env/.env with required configuration
   # See Configuration section below
   ```

3. **Start the services:**
   ```bash
   docker compose up -d
   ```

4. **Wait for services to start:**
   ```bash
   docker compose ps
   ```

5. **Access Nautobot:**
   - URL: `http://192.168.5.9:8081`
   - Default credentials: admin/admin (change immediately!)

6. **Create admin user:**
   ```bash
   docker compose exec nautobot nautobot-server createsuperuser
   ```

### Verify Nautobot Services

```bash
# Check all services are running
docker compose ps

# View logs
docker compose logs -f nautobot

# Check database connection
docker compose exec nautobot nautobot-server dbshell
```

**Expected services:**
- `nautobot`: Web application
- `nautobot-worker`: Background task worker
- `nautobot-beat`: Celery beat scheduler
- `postgres`: Database
- `redis`: Task queue and caching

---

## Configuration

### NetBox Configuration

NetBox configuration is managed through:

**Environment files** (located in `netbox-docker/env/`):
- `netbox.env` - Main NetBox configuration
- `postgres.env` - PostgreSQL database settings
- `redis.env` - Redis task queue settings
- `redis-cache.env` - Redis cache settings

**Configuration files** (located in `netbox-docker/configuration/`):
- `configuration.py` - Main configuration (read from environment variables)
- `extra.py` - Additional custom settings
- `logging.py` - Logging configuration
- `plugins.py` - Plugin configuration

### Key Environment Variables

**NetBox (`netbox-docker/env/netbox.env`):**
```bash
SECRET_KEY=<generate-strong-secret-key>
DB_NAME=netbox
DB_USER=netbox
DB_PASSWORD=<strong-password>
DB_HOST=postgres
DB_PORT=5432
REDIS_PASSWORD=<strong-redis-password>
ALLOWED_HOSTS=192.168.5.9,localhost,127.0.0.1
```

**Generate Secret Key:**
```bash
python3 -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

### Nautobot Configuration

Nautobot configuration is managed through:

**Environment file** (`nautobot-docker/env/.env`):

```bash
NAUTOBOT_SECRET_KEY=<generate-strong-secret-key>
POSTGRES_DB=nautobot
POSTGRES_USER=nautobot
POSTGRES_PASSWORD=<strong-password>
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
NAUTOBOT_ALLOWED_HOSTS=192.168.5.9,localhost,127.0.0.1
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=<strong-redis-password>
```

**Generate Secret Key:**
```bash
python3 -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

### Security Configuration

**Important Security Settings:**

1. **Change default passwords** immediately after first login
2. **Use strong SECRET_KEY** - Generate unique keys for each deployment
3. **Configure ALLOWED_HOSTS** - Only include trusted hostnames/IPs
4. **Enable HTTPS** (production) - Use reverse proxy (nginx, Traefik) with SSL/TLS
5. **Firewall rules** - Restrict access to LAN only (no WAN exposure)

---

## Access & Usage

### Web Access

**NetBox:**
- URL: `http://192.168.5.9:8000`
- Default login: admin/admin (change immediately!)

**Nautobot:**
- URL: `http://192.168.5.9:8081`
- Default login: admin/admin (change immediately!)

### Initial Setup

**NetBox:**
1. Log in with default credentials
2. Change admin password immediately
3. Configure site settings
4. Add IP prefixes (e.g., 192.168.4.0/22, 192.168.5.0/24)
5. Add device types and manufacturers
6. Document network devices

**Nautobot:**
1. Log in with default credentials
2. Change admin password immediately
3. Configure site settings
4. Add IP prefixes
5. Add device types and manufacturers
6. Document network devices

### API Access

Both platforms provide REST APIs:

**NetBox API:**
```bash
# Get API token from: Admin > API Tokens
curl -H "Authorization: Token <your-token>" \
     http://192.168.5.9:8000/api/dcim/devices/
```

**Nautobot API:**
```bash
# Get API token from: Admin > API Tokens
curl -H "Authorization: Token <your-token>" \
     http://192.168.5.9:8081/api/dcim/devices/
```

---

## Maintenance & Operations

### Viewing Logs

**NetBox:**
```bash
cd netbox-docker
docker compose logs -f netbox
docker compose logs -f netbox-worker
```

**Nautobot:**
```bash
cd nautobot-docker
docker compose logs -f nautobot
docker compose logs -f nautobot-worker
```

### Stopping Services

**NetBox:**
```bash
cd netbox-docker
docker compose down
```

**Nautobot:**
```bash
cd nautobot-docker
docker compose down
```

### Starting Services

**NetBox:**
```bash
cd netbox-docker
docker compose up -d
```

**Nautobot:**
```bash
cd nautobot-docker
docker compose up -d
```

### Updating

**NetBox:**
```bash
cd netbox-docker
# Pull latest images
docker compose pull

# Stop services
docker compose down

# Start with new images
docker compose up -d

# Run database migrations
docker compose exec netbox /opt/netbox/netbox/manage.py migrate
```

**Nautobot:**
```bash
cd nautobot-docker
# Pull latest images
docker compose pull

# Stop and restart services
docker compose down
docker compose up -d

# Run database migrations
docker compose exec nautobot nautobot-server migrate
```

### Backup and Restore

**Backup Database:**

**NetBox:**
```bash
cd netbox-docker
docker compose exec postgres pg_dump -U netbox netbox > backup-$(date +%Y%m%d).sql
```

**Nautobot:**
```bash
cd nautobot-docker
docker compose exec postgres pg_dump -U nautobot nautobot > backup-$(date +%Y%m%d).sql
```

**Restore Database:**

**NetBox:**
```bash
cd netbox-docker
docker compose exec -T postgres psql -U netbox netbox < backup-YYYYMMDD.sql
```

**Nautobot:**
```bash
cd nautobot-docker
docker compose exec -T postgres psql -U nautobot nautobot < backup-YYYYMMDD.sql
```

### Data Persistence

Both deployments use Docker volumes to persist data:

**NetBox:**
- `netbox-postgres-data` - Database data
- `netbox-redis-data` - Redis task queue data
- `netbox-redis-cache-data` - Redis cache data
- `netbox-media-files` - Uploaded media files
- `netbox-reports-files` - Report files
- `netbox-scripts-files` - Custom scripts

**Nautobot:**
- `./postgres` - Database data (local directory)
- `./redis` - Redis data (local directory)
- `nautobot-media` - Uploaded media files
- `nautobot-static` - Static files

---

## Troubleshooting

### Services Won't Start

**Check logs:**
```bash
docker compose logs
```

**Check Docker status:**
```bash
docker ps -a
docker compose ps
```

**Verify ports aren't in use:**
```bash
sudo ss -lntup | grep -E '8000|8081'
```

### Database Connection Errors

**Check database service:**
```bash
docker compose ps postgres
docker compose logs postgres
```

**Test database connection:**
```bash
# NetBox
docker compose exec netbox /opt/netbox/netbox/manage.py dbshell

# Nautobot
docker compose exec nautobot nautobot-server dbshell
```

### Web UI Not Accessible

**Check service status:**
```bash
docker compose ps
```

**Check firewall:**
```bash
sudo ufw status
# Allow ports if needed
sudo ufw allow 8000/tcp  # NetBox
sudo ufw allow 8081/tcp   # Nautobot
```

**Check network connectivity:**
```bash
# From another device
curl http://192.168.5.9:8000  # NetBox
curl http://192.168.5.9:8081  # Nautobot
```

### Out of Disk Space

**Check disk usage:**
```bash
df -h
docker system df
```

**Clean up Docker:**
```bash
docker system prune -a
```

**Check volume sizes:**
```bash
docker volume ls
docker volume inspect <volume-name>
```

---

## Integration with Other Lab Services

### Security Onion Integration

**Document Security Onion VM:**
- Add Security Onion VM (192.168.4.155) to NetBox/Nautobot
- Document network interfaces
- Track IP addresses
- Link to related devices

### PiVPN/Pi-hole Integration

**Document PiVPN Host:**
- Add PiVPN host (192.168.4.127) to NetBox/Nautobot
- Document services (Pi-hole DNS, WireGuard VPN)
- Track IP addresses and subnets
- Document VPN client IP allocations

### Network Documentation

**Document Network Infrastructure:**
- Add IP prefixes (192.168.4.0/22, 10.155.185.0/24)
- Document VLANs and subnets
- Track IP address allocations
- Document network devices (switches, routers)

### Automation Integration

**Use as Source of Truth:**
- NetBox/Nautobot APIs can be queried by automation tools
- Track network changes and configurations
- Generate network diagrams
- Export data for other tools

---

## Key Takeaways

### Critical Success Factors

1. **Strong Passwords**: Change default credentials immediately
2. **Secret Keys**: Generate unique SECRET_KEY for each deployment
3. **Backups**: Regular database backups are essential
4. **Network Access**: Restrict to LAN only (no WAN exposure)
5. **Resource Allocation**: Ensure sufficient RAM and storage

### Common Pitfalls

1. **Default Credentials**: Always change admin passwords
2. **Weak Secret Keys**: Use generated keys, not simple strings
3. **No Backups**: Database corruption can lose all data
4. **Port Conflicts**: Ensure ports 8000 and 8081 aren't in use
5. **Insufficient Resources**: Low RAM can cause service failures

### Best Practices

1. **Regular Updates**: Keep Docker images updated
2. **Documentation**: Document all network changes in NetBox/Nautobot
3. **API Usage**: Use APIs for automation instead of manual entry
4. **Backup Strategy**: Automated daily backups
5. **Monitoring**: Monitor service health and disk usage

---

## Conclusion

This project deployed NetBox and Nautobot as containerized Network Source of Truth platforms, providing comprehensive IPAM and DCIM capabilities for network infrastructure documentation and management.

**Final Status:**
- ✅ NetBox deployed and accessible
- ✅ Nautobot deployed and accessible
- ✅ Database persistence configured
- ✅ Services running and healthy
- ✅ Integration with other lab services documented

**Next Steps:**
- Document all network devices and IP addresses
- Set up automated backups
- Configure API tokens for automation
- Integrate with other lab services
- Explore plugins and extensions

---

## Additional Resources

### NetBox

- [NetBox Documentation](https://docs.netbox.dev/)
- [NetBox Docker Wiki](https://github.com/netbox-community/netbox-docker/wiki)
- [NetBox Community Slack](https://netdev.chat/)

### Nautobot

- [Nautobot Documentation](https://docs.nautobot.com/)
- [Nautobot GitHub](https://github.com/nautobot/nautobot)

### Repository

- [NSOT-Network-Source-of-Truth GitHub](https://github.com/alexlux58/NSOT-Network-Source-of-Truth)

---

**End of Lab Writeup**

