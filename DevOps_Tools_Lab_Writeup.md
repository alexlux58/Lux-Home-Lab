# DevOps Tools Stack - Ansible, AWS, Puppet & Jenkins
## Infrastructure Automation & CI/CD Platform

**Author:** Alex Lux  
**Date:** December 2025  
**Environment:** Home Lab - Ubuntu VM on Proxmox  
**VM IP:** 192.168.5.8

> **Quick Reference Available**: For command cheat sheets and quick troubleshooting, see `QUICK_REFERENCE.md` in this directory.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [VM Setup](#vm-setup)
5. [Ansible Installation & Configuration](#ansible-installation--configuration)
6. [AWS CLI & SDK Setup](#aws-cli--sdk-setup)
7. [Puppet Installation & Configuration](#puppet-installation--configuration)
8. [Jenkins Installation & Configuration](#jenkins-installation--configuration)
9. [Integration & Workflows](#integration--workflows)
10. [Access & Usage](#access--usage)
11. [Maintenance & Operations](#maintenance--operations)
12. [Troubleshooting](#troubleshooting)
13. [Integration with Other Lab Services](#integration-with-other-lab-services)

---

## Project Overview

### Objective
Deploy a comprehensive DevOps toolchain on a single Ubuntu VM, providing infrastructure automation, configuration management, and CI/CD capabilities. This stack enables automated infrastructure provisioning, configuration management, cloud integration, and continuous integration/deployment workflows.

### Why This Stack?

**Ansible:**
- Agentless configuration management and automation
- Simple YAML-based playbooks
- Extensive module library
- Idempotent operations

**AWS Integration:**
- Cloud infrastructure provisioning
- Service integration and automation
- Cost-effective cloud resource management
- Hybrid cloud capabilities

**Puppet:**
- Declarative configuration management
- Agent-based model for continuous enforcement
- Strong compliance and reporting features
- Enterprise-grade scalability

**Jenkins:**
- Open-source CI/CD automation server
- Extensive plugin ecosystem
- Pipeline as Code support
- Integration with version control systems

### Use Cases

- **Infrastructure as Code**: Automate VM and service provisioning
- **Configuration Management**: Ensure consistent system configurations
- **CI/CD Pipelines**: Automate build, test, and deployment workflows
- **Cloud Automation**: Manage AWS resources programmatically
- **Compliance**: Enforce and report on system configurations
- **Multi-Environment Management**: Manage dev, staging, and production environments

---

## Architecture Overview

### Deployment Architecture

```
Proxmox Host (192.168.4.146)
   ↓
Ubuntu VM (192.168.5.8)
   ├── Ansible
   │   ├── Control Node
   │   ├── Playbooks & Roles
   │   └── Inventory Management
   ├── AWS CLI/SDK
   │   ├── AWS CLI
   │   ├── boto3 (Python SDK)
   │   └── AWS Credentials
   ├── Puppet
   │   ├── Puppet Master/Server
   │   ├── Puppet Agent
   │   └── Manifests & Modules
   └── Jenkins
       ├── Jenkins Server
       ├── Build Agents
       └── Pipeline Jobs
```

### Network Configuration

- **VM IP**: 192.168.5.8 (Ubuntu VM on Proxmox)
- **Jenkins Port**: 8080 (default, configurable)
- **Puppet Master Port**: 8140 (default)
- **Access**: LAN only (no WAN exposure recommended)

### Service Ports

| Service | Port | Protocol | Exposure |
|---------|------|----------|----------|
| Jenkins | 8080 | TCP | LAN only |
| Puppet Master | 8140 | TCP | LAN only |
| Ansible | SSH (22) | TCP | LAN only |
| AWS | HTTPS (443) | TCP | Internet (API calls) |

---

## Prerequisites

### Software Requirements

- **Ubuntu Server**: 20.04 LTS or 22.04 LTS (recommended)
- **Python**: 3.8+ (for Ansible and AWS SDK)
- **Java**: OpenJDK 11 or 17 (for Jenkins)
- **Ruby**: 2.7+ (for Puppet, if using older versions)
- **Git**: For version control integration

### Hardware Requirements

- **VM RAM**: 4GB minimum (8GB+ recommended)
- **VM Storage**: 50GB+ (for Jenkins workspaces and builds)
- **CPU**: 2+ cores recommended

### Network Requirements

- Internet access (for package downloads and AWS API calls)
- SSH access to managed nodes (for Ansible)
- Network connectivity to Puppet agents (if using agent-based model)

---

## VM Setup

### Create Ubuntu VM on Proxmox

1. **Download Ubuntu Server ISO**
   - Ubuntu 22.04 LTS recommended

2. **Create VM in Proxmox**
   - **Name**: DevOps-Tools or Automation-Stack
   - **OS Type**: Linux
   - **RAM**: 4GB minimum (8GB recommended)
   - **CPU**: 2+ cores
   - **Storage**: 50GB+ (SSD recommended)
   - **Network**: Bridge to vmbr0 (management network)

3. **Install Ubuntu Server**
   - Follow standard Ubuntu installation
   - Enable SSH server
   - Set static IP: 192.168.5.8/22
   - Or configure DHCP reservation in router
   - Install standard system utilities

### Initial System Setup

```bash
# Update system
sudo apt-get update
sudo apt-get -y upgrade

# Install essential tools
sudo apt-get install -y \
    curl \
    wget \
    git \
    vim \
    net-tools \
    openssh-server \
    python3 \
    python3-pip \
    software-properties-common

# Configure SSH (optional hardening)
sudo systemctl enable ssh
sudo systemctl start ssh
```

---

## Ansible Installation & Configuration

### Installation

**Method 1: Using pip (Recommended)**
```bash
# Install Ansible via pip
sudo pip3 install ansible

# Verify installation
ansible --version
```

**Method 2: Using apt (Ubuntu)**
```bash
# Add Ansible PPA
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt-get update
sudo apt-get install -y ansible

# Verify installation
ansible --version
```

### Initial Configuration

**Create Ansible directory structure:**
```bash
mkdir -p ~/ansible/{playbooks,roles,inventory,group_vars,host_vars}
cd ~/ansible
```

**Create inventory file (`inventory/hosts`):**
```ini
[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa

[lab_servers]
securityonion ansible_host=192.168.4.155
pivpn ansible_host=192.168.4.127
nsot ansible_host=192.168.5.9

[proxmox_hosts]
proxmox1 ansible_host=192.168.4.146

[local]
localhost ansible_connection=local
```

**Create Ansible configuration (`ansible.cfg`):**
```ini
[defaults]
inventory = inventory/hosts
host_key_checking = False
retry_files_enabled = False
roles_path = roles
collections_path = ~/.ansible/collections
```

### Test Ansible Installation

```bash
# Test connectivity to all hosts
ansible all -m ping

# Test with specific inventory
ansible all -i inventory/hosts -m ping
```

### Example Playbook

**Create a simple playbook (`playbooks/ping-all.yml`):**
```yaml
---
- name: Test connectivity to all hosts
  hosts: all
  gather_facts: yes
  tasks:
    - name: Display hostname
      debug:
        msg: "Hostname: {{ inventory_hostname }}"
    
    - name: Display OS information
      debug:
        msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

**Run the playbook:**
```bash
ansible-playbook playbooks/ping-all.yml
```

---

## AWS CLI & SDK Setup

### AWS CLI Installation

**Install AWS CLI v2:**
```bash
# Download AWS CLI installer
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Install unzip if needed
sudo apt-get install -y unzip

# Extract and install
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version
```

**Alternative: Using pip**
```bash
pip3 install awscli
```

### AWS CLI Configuration

**Configure AWS credentials:**
```bash
# Configure AWS CLI
aws configure

# You'll be prompted for:
# - AWS Access Key ID
# - AWS Secret Access Key
# - Default region name (e.g., us-east-1)
# - Default output format (json, yaml, text, table)
```

**Manual configuration (if needed):**
```bash
# Create credentials file
mkdir -p ~/.aws
cat > ~/.aws/credentials <<EOF
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
EOF

# Create config file
cat > ~/.aws/config <<EOF
[default]
region = us-east-1
output = json
EOF
```

**Set permissions:**
```bash
chmod 600 ~/.aws/credentials
chmod 600 ~/.aws/config
```

### Test AWS CLI

```bash
# Test AWS connection
aws sts get-caller-identity

# List S3 buckets
aws s3 ls

# List EC2 instances
aws ec2 describe-instances
```

### AWS SDK (boto3) Installation

**Install boto3 (Python SDK):**
```bash
pip3 install boto3

# Verify installation
python3 -c "import boto3; print(boto3.__version__)"
```

**Example Python script using boto3:**
```python
#!/usr/bin/env python3
import boto3

# Create EC2 client
ec2 = boto3.client('ec2')

# List all regions
regions = ec2.describe_regions()
for region in regions['Regions']:
    print(region['RegionName'])
```

---

## Puppet Installation & Configuration

### Installation Options

**Option 1: Puppet Server (Master/Agent Model)**

**Install Puppet Server:**
```bash
# Add Puppet repository
wget https://apt.puppet.com/puppet7-release-focal.deb
sudo dpkg -i puppet7-release-focal.deb
sudo apt-get update

# Install Puppet Server
sudo apt-get install -y puppetserver

# Configure memory (edit /etc/default/puppetserver)
# JAVA_ARGS="-Xms2g -Xmx2g -XX:MaxPermSize=256m"
sudo sed -i 's/-Xms2g -Xmx2g/-Xms1g -Xmx1g/' /etc/default/puppetserver

# Start Puppet Server
sudo systemctl enable puppetserver
sudo systemctl start puppetserver

# Verify installation
sudo systemctl status puppetserver
```

**Install Puppet Agent (on same VM for testing):**
```bash
# Install Puppet Agent
sudo apt-get install -y puppet-agent

# Configure Puppet Agent
sudo /opt/puppetlabs/bin/puppet config set server $(hostname)
sudo /opt/puppetlabs/bin/puppet config set certname $(hostname)

# Start Puppet Agent
sudo systemctl enable puppet
sudo systemctl start puppet

# Request certificate
sudo /opt/puppetlabs/bin/puppet agent --test
```

**Option 2: Standalone Puppet (Agentless)**

**Install Puppet:**
```bash
# Add Puppet repository
wget https://apt.puppet.com/puppet7-release-focal.deb
sudo dpkg -i puppet7-release-focal.deb
sudo apt-get update

# Install Puppet
sudo apt-get install -y puppet

# Configure Puppet
sudo puppet config set server $(hostname)
```

### Puppet Configuration

**Create manifest directory:**
```bash
sudo mkdir -p /etc/puppetlabs/code/environments/production/manifests
sudo mkdir -p /etc/puppetlabs/code/environments/production/modules
```

**Create site manifest (`/etc/puppetlabs/code/environments/production/manifests/site.pp`):**
```puppet
# Site manifest
node default {
  # Ensure SSH is installed
  package { 'openssh-server':
    ensure => installed,
  }
  
  # Ensure SSH service is running
  service { 'ssh':
    ensure => running,
    enable => true,
    require => Package['openssh-server'],
  }
  
  # Create a test file
  file { '/tmp/puppet-test.txt':
    ensure  => present,
    content => "Puppet managed this file at ${timestamp}\n",
    mode    => '0644',
  }
}
```

**Test Puppet:**
```bash
# Apply manifest manually
sudo puppet apply /etc/puppetlabs/code/environments/production/manifests/site.pp

# Or run agent (if using server/agent model)
sudo /opt/puppetlabs/bin/puppet agent --test
```

### Puppet Modules

**Install useful modules:**
```bash
# Install via Puppet Forge
sudo /opt/puppetlabs/bin/puppet module install puppetlabs-stdlib
sudo /opt/puppetlabs/bin/puppet module install puppetlabs-apt
sudo /opt/puppetlabs/bin/puppet module install puppetlabs-ntp
```

---

## Jenkins Installation & Configuration

### Installation

**Method 1: Using apt (Recommended)**
```bash
# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update and install Jenkins
sudo apt-get update
sudo apt-get install -y jenkins

# Start Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

# Check status
sudo systemctl status jenkins
```

**Method 2: Using Docker (Alternative)**
```bash
# Install Docker (if not already installed)
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

# Run Jenkins in Docker
sudo docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

### Initial Jenkins Setup

**Get initial admin password:**
```bash
# Get Jenkins initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Access Jenkins Web UI:**
1. Open browser: `http://192.168.5.8:8080`
2. Enter initial admin password
3. Install suggested plugins
4. Create admin user
5. Configure instance URL: `http://192.168.5.8:8080`

### Jenkins Configuration

**Install useful plugins:**
- **Ansible Plugin**: For Ansible integration
- **Pipeline**: For Pipeline as Code
- **Git Plugin**: For Git integration
- **AWS Steps Plugin**: For AWS integration
- **SSH Agent Plugin**: For SSH key management
- **Blue Ocean**: Modern UI (optional)

**Configure SSH keys for Ansible:**
```bash
# Generate SSH key (if not exists)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Add to Jenkins
# Jenkins > Credentials > Add Credentials > SSH Username with private key
```

**Configure AWS credentials in Jenkins:**
```bash
# Add AWS credentials
# Jenkins > Credentials > Add Credentials > AWS Credentials
# - Access Key ID
# - Secret Access Key
```

### Create Sample Jenkins Job

**Create a simple pipeline job:**

1. **New Item** > **Pipeline**
2. **Pipeline Definition**: Pipeline script
3. **Script:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Hello') {
            steps {
                echo 'Hello from Jenkins!'
                sh 'hostname'
                sh 'uname -a'
            }
        }
        
        stage('Ansible Test') {
            steps {
                sh 'ansible --version'
            }
        }
        
        stage('AWS Test') {
            steps {
                sh 'aws --version'
                sh 'aws sts get-caller-identity'
            }
        }
    }
}
```

---

## Integration & Workflows

### Ansible + Jenkins Integration

**Jenkins Pipeline with Ansible:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Deploy with Ansible') {
            steps {
                ansiblePlaybook(
                    playbook: 'playbooks/deploy.yml',
                    inventory: 'inventory/hosts',
                    credentialsId: 'ansible-ssh-key'
                )
            }
        }
    }
}
```

### AWS + Ansible Integration

**Ansible playbook using AWS modules:**
```yaml
---
- name: Manage AWS EC2 instances
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Create EC2 instance
      ec2_instance:
        name: test-instance
        instance_type: t2.micro
        image_id: ami-0c55b159cbfafe1f0
        region: us-east-1
        key_name: my-key
        wait: yes
      register: ec2_result
```

### Puppet + Jenkins Integration

**Jenkins Pipeline with Puppet:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Apply Puppet Manifest') {
            steps {
                sh 'sudo puppet apply /etc/puppetlabs/code/environments/production/manifests/site.pp'
            }
        }
    }
}
```

### Complete CI/CD Workflow Example

**Jenkins Pipeline for Infrastructure Deployment:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }
        
        stage('Ansible Lint') {
            steps {
                sh 'ansible-lint playbooks/*.yml'
            }
        }
        
        stage('Deploy Infrastructure') {
            steps {
                sh 'ansible-playbook -i inventory/hosts playbooks/deploy.yml'
            }
        }
        
        stage('Configure with Puppet') {
            steps {
                sh 'sudo puppet apply manifests/site.pp'
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh 'ansible all -m ping'
            }
        }
    }
}
```

---

## Access & Usage

### Web Access

**Jenkins:**
- URL: `http://192.168.5.8:8080`
- Default login: Use initial admin password from `/var/lib/jenkins/secrets/initialAdminPassword`

**Puppet Enterprise Console (if installed):**
- URL: `https://192.168.5.8:4433` (if using Puppet Enterprise)

### Command Line Access

**Ansible:**
```bash
# Run ad-hoc commands
ansible all -m shell -a "uptime"

# Run playbooks
ansible-playbook playbooks/deploy.yml
```

**AWS CLI:**
```bash
# List resources
aws ec2 describe-instances
aws s3 ls
aws iam list-users
```

**Puppet:**
```bash
# Apply manifests
sudo puppet apply manifest.pp

# Run agent
sudo /opt/puppetlabs/bin/puppet agent --test
```

**Jenkins CLI:**
```bash
# Download jenkins-cli.jar
wget http://192.168.5.8:8080/jnlpJars/jenkins-cli.jar

# List jobs
java -jar jenkins-cli.jar -s http://192.168.5.8:8080 -auth user:token list-jobs
```

---

## Maintenance & Operations

### Service Management

**Jenkins:**
```bash
# Start/Stop/Restart
sudo systemctl start jenkins
sudo systemctl stop jenkins
sudo systemctl restart jenkins

# View logs
sudo journalctl -u jenkins -f
sudo tail -f /var/log/jenkins/jenkins.log
```

**Puppet Server:**
```bash
# Start/Stop/Restart
sudo systemctl start puppetserver
sudo systemctl stop puppetserver
sudo systemctl restart puppetserver

# View logs
sudo journalctl -u puppetserver -f
```

### Backup and Restore

**Jenkins Backup:**
```bash
# Backup Jenkins home directory
sudo tar -czf jenkins-backup-$(date +%Y%m%d).tar.gz /var/lib/jenkins

# Restore Jenkins
sudo systemctl stop jenkins
sudo tar -xzf jenkins-backup-YYYYMMDD.tar.gz -C /
sudo systemctl start jenkins
```

**Puppet Backup:**
```bash
# Backup Puppet code
sudo tar -czf puppet-backup-$(date +%Y%m%d).tar.gz /etc/puppetlabs/code
```

**Ansible Backup:**
```bash
# Backup Ansible playbooks and inventory
tar -czf ansible-backup-$(date +%Y%m%d).tar.gz ~/ansible
```

### Updates

**Ansible:**
```bash
pip3 install --upgrade ansible
```

**AWS CLI:**
```bash
# Update AWS CLI
pip3 install --upgrade awscli
# Or for v2
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli
```

**Puppet:**
```bash
sudo apt-get update
sudo apt-get upgrade puppetserver puppet-agent
```

**Jenkins:**
```bash
sudo apt-get update
sudo apt-get upgrade jenkins
sudo systemctl restart jenkins
```

### Monitoring

**Check service status:**
```bash
# All services
sudo systemctl status jenkins
sudo systemctl status puppetserver
sudo systemctl status puppet

# Disk usage
df -h
du -sh /var/lib/jenkins
```

---

## Troubleshooting

### Ansible Issues

**Connection refused:**
```bash
# Check SSH connectivity
ssh user@target-host

# Test with verbose output
ansible all -m ping -vvv
```

**Permission denied:**
```bash
# Check SSH key permissions
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# Test SSH key
ssh -i ~/.ssh/id_rsa user@target-host
```

### AWS CLI Issues

**Credentials not found:**
```bash
# Check credentials file
cat ~/.aws/credentials

# Reconfigure
aws configure
```

**Region not set:**
```bash
# Set default region
aws configure set region us-east-1

# Or use environment variable
export AWS_DEFAULT_REGION=us-east-1
```

### Puppet Issues

**Agent can't connect to server:**
```bash
# Check Puppet Server status
sudo systemctl status puppetserver

# Check firewall
sudo ufw status

# Test connection
sudo /opt/puppetlabs/bin/puppet agent --test --debug
```

**Certificate issues:**
```bash
# Regenerate certificate
sudo rm -rf /var/lib/puppet/ssl
sudo /opt/puppetlabs/bin/puppet agent --test
```

### Jenkins Issues

**Jenkins won't start:**
```bash
# Check logs
sudo journalctl -u jenkins -n 50

# Check Java version
java -version

# Check port availability
sudo ss -lntup | grep 8080
```

**Plugins not installing:**
```bash
# Check Jenkins logs
sudo tail -f /var/log/jenkins/jenkins.log

# Manual plugin installation
# Download .hpi file and install via UI
```

**Out of disk space:**
```bash
# Clean Jenkins workspace
sudo find /var/lib/jenkins/workspace -type d -mtime +30 -exec rm -rf {} \;

# Clean old builds
# Jenkins > Manage Jenkins > Script Console
```

---

## Integration with Other Lab Services

### Ansible + Security Onion

**Automate Security Onion configuration:**
```yaml
---
- name: Configure Security Onion
  hosts: securityonion
  tasks:
    - name: Update rules
      command: sudo so-rule-update
```

### Ansible + PiVPN/Pi-hole

**Manage Pi-hole configuration:**
```yaml
---
- name: Manage Pi-hole
  hosts: pivpn
  tasks:
    - name: Check Pi-hole status
      command: pihole status
```

### Jenkins + NetBox/Nautobot

**Document deployments in NetBox:**
```groovy
stage('Update NetBox') {
    steps {
        sh '''
            curl -X POST http://192.168.5.9:8000/api/dcim/devices/ \
                -H "Authorization: Token YOUR_TOKEN" \
                -H "Content-Type: application/json" \
                -d '{"name": "deployed-server", "device_type": 1}'
        '''
    }
}
```

### Complete Automation Workflow

**End-to-end automation:**
1. **Jenkins** triggers on Git commit
2. **Ansible** provisions infrastructure (AWS or local)
3. **Puppet** configures systems
4. **Ansible** deploys applications
5. **NetBox/Nautobot** documents changes
6. **Security Onion** monitors for anomalies

---

## Key Takeaways

### Critical Success Factors

1. **SSH Key Management**: Proper SSH key setup for Ansible
2. **AWS Credentials**: Secure storage and rotation of AWS keys
3. **Jenkins Security**: Strong passwords and access control
4. **Puppet Certificates**: Proper certificate management
5. **Version Control**: Git integration for all code

### Common Pitfalls

1. **SSH Key Permissions**: Wrong permissions prevent Ansible from working
2. **AWS Credentials**: Exposed credentials in code or logs
3. **Jenkins Disk Space**: Workspace cleanup needed regularly
4. **Puppet Firewall**: Firewall blocking Puppet agent communication
5. **Network Connectivity**: Services need network access to function

### Best Practices

1. **Infrastructure as Code**: Store all configs in version control
2. **Secrets Management**: Use Jenkins credentials or external vaults
3. **Regular Backups**: Backup Jenkins and Puppet configurations
4. **Monitoring**: Monitor service health and disk usage
5. **Documentation**: Document playbooks, manifests, and pipelines

---

## Conclusion

This project deployed a comprehensive DevOps toolchain including Ansible, AWS CLI/SDK, Puppet, and Jenkins, providing infrastructure automation, configuration management, and CI/CD capabilities.

**Final Status:**
- ✅ Ansible installed and configured
- ✅ AWS CLI and SDK configured
- ✅ Puppet installed and configured
- ✅ Jenkins deployed and accessible
- ✅ Integration workflows established

**Next Steps:**
- Create reusable Ansible roles
- Build Jenkins pipelines for common tasks
- Develop Puppet modules for lab infrastructure
- Integrate with other lab services
- Set up automated testing workflows

---

## Additional Resources

### Official Documentation

- [Ansible Documentation](https://docs.ansible.com/)
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
- [Puppet Documentation](https://puppet.com/docs/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)

### Learning Resources

- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Puppet Learning VM](https://puppet.com/download-learning-vm)
- [Jenkins Pipeline Examples](https://www.jenkins.io/doc/pipeline/examples/)

---

**End of Lab Writeup**

