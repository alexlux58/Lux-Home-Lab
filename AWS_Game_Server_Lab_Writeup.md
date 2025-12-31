# AWS Infrastructure as Code Game Server
## Terraform + Ansible Game Server Deployment

**Author:** Alex Lux  
**Date:** December 2025  
**Environment:** AWS EC2 Instance  
**Repository:** [aws_iac-game-server](https://github.com/alexlux58/aws_iac-game-server)

> **Quick Reference Available**: For command cheat sheets and quick troubleshooting, see `QUICK_REFERENCE.md` in this directory.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [Initial Setup](#initial-setup)
5. [Terraform Configuration](#terraform-configuration)
6. [Ansible Configuration](#ansible-configuration)
7. [Deployment](#deployment)
8. [Access & Usage](#access--usage)
9. [Adding New Games](#adding-new-games)
10. [Maintenance & Operations](#maintenance--operations)
11. [Troubleshooting](#troubleshooting)
12. [Cost Management](#cost-management)

---

## Project Overview

### Objective
Deploy a web-based game server on AWS EC2 using Infrastructure as Code (Terraform) and configuration management (Ansible). The server hosts multiple browser-based games accessible via a unified web interface, with support for both single-player and multiplayer games.

### Why This Stack?

**Terraform:**
- Infrastructure as Code (IaC)
- Declarative infrastructure management
- State management and versioning
- Multi-cloud support

**Ansible:**
- Configuration management
- Idempotent operations
- Simple YAML-based playbooks
- Agentless architecture

**AWS EC2:**
- Scalable compute resources
- Pay-as-you-go pricing
- Integration with AWS services
- Global availability

### Supported Games

- **2048** - Classic number puzzle game
- **PvP** - Multiplayer game
- **Pac-Man** - Classic arcade game
- **Super Mario Bros** - Platformer game
- **QuakeJS** - Browser-based Quake engine (requires Docker)

### Use Cases

- **Personal Gaming Server**: Host games for friends and family
- **Learning IaC**: Practice Terraform and Ansible
- **Cost-Effective Hosting**: Pay only for what you use
- **Scalable Infrastructure**: Easy to scale up or down
- **Automated Deployment**: One-command deployment

---

## Architecture Overview

### Deployment Architecture

```
AWS Account
   ↓
Terraform
   ├── EC2 Instance (t3.micro/t3.small)
   ├── Security Group (HTTP/HTTPS/SSH)
   ├── Elastic IP (optional)
   ├── S3 Bucket (for backups)
   └── CloudWatch Alarms (billing)
   ↓
Ansible
   ├── Nginx Web Server
   ├── Game Files Deployment
   ├── Docker (for QuakeJS)
   └── SSL/TLS Configuration
   ↓
Web Interface
   ├── Game Selection Page
   └── Individual Game Pages
```

### Network Configuration

- **EC2 Instance**: Public IP (dynamic or Elastic IP)
- **Security Group**: 
  - Port 22 (SSH) - Restricted to specific IP
  - Port 80 (HTTP) - Public
  - Port 443 (HTTPS) - Public (if SSL configured)
- **Domain**: Optional (via Cloudflare)

### Repository Structure

```
aws_iac-game-server/
├── terraform/              # Terraform infrastructure code
│   ├── main.tf            # Main infrastructure
│   ├── variables.tf      # Variable definitions
│   ├── outputs.tf         # Output values
│   └── ...
├── ansible/               # Ansible playbooks
│   ├── playbook.yml       # Main playbook
│   ├── roles/             # Ansible roles
│   │   └── web-game/     # Game server role
│   └── ...
├── scripts/               # Helper scripts
├── .env.example          # Environment variable template
├── Makefile              # Build automation
└── README.md             # Project documentation
```

---

## Prerequisites

### Software Requirements

- **Terraform**: Version 1.0+ (latest recommended)
- **Ansible**: Version 2.9+ (latest recommended)
- **AWS CLI**: Version 2.0+ (for AWS operations)
- **Git**: For cloning repository
- **SSH Client**: For server access
- **Make**: For build automation (optional)

### AWS Requirements

- **AWS Account**: Active AWS account
- **AWS Credentials**: Access key and secret key
- **IAM Permissions**: Admin or Power User access
- **Region**: Choose your preferred AWS region

### Verify Installation

```bash
# Check Terraform
terraform --version

# Check Ansible
ansible --version

# Check AWS CLI
aws --version

# Verify AWS credentials
aws sts get-caller-identity
```

---

## Initial Setup

### Clone Repository

```bash
# Clone the repository
git clone https://github.com/alexlux58/aws_iac-game-server.git
cd aws_iac-game-server
```

### Configure Environment Variables

**Create `.env` file:**
```bash
# Copy example file
cp .env.example .env

# Edit with your values
nano .env
```

**Required variables:**
```bash
# AWS Configuration
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1

# SSH Configuration
SSH_KEY_PATH=~/.ssh/id_rsa
SSH_ALLOWED_CIDR=your.ip.address/32

# Instance Configuration
INSTANCE_TYPE=t3.micro
AMI_ID=ami-0c55b159cbfafe1f0  # Amazon Linux 2023

# S3 Configuration
S3_BUCKET_NAME=your-unique-bucket-name

# Cloudflare (Optional)
CLOUDFLARE_API_TOKEN=your-token
CLOUDFLARE_ZONE_ID=your-zone-id
CLOUDFLARE_ZONE_NAME=yourdomain.com
CLOUDFLARE_SUBDOMAIN=games
```

**Important Notes:**
- S3 bucket names must be globally unique
- SSH key must exist and have correct permissions: `chmod 400 ~/.ssh/id_rsa`
- Cloudflare token is optional (use dummy value if not using)

### Generate SSH Key (if needed)

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/aws_game_server -N ""

# Set permissions
chmod 400 ~/.ssh/aws_game_server

# Update .env with key path
SSH_KEY_PATH=~/.ssh/aws_game_server
```

---

## Terraform Configuration

### Initialize Terraform

```bash
cd terraform

# Initialize Terraform
terraform init

# Verify providers
terraform providers
```

### Review Configuration

**Key Terraform resources:**
- EC2 instance (game server)
- Security group (firewall rules)
- Elastic IP (optional, for static IP)
- S3 bucket (for backups)
- CloudWatch billing alarm
- Cloudflare DNS record (if configured)

### Validate Configuration

```bash
# Validate Terraform syntax
terraform validate

# Plan deployment
terraform plan

# Review what will be created
```

---

## Ansible Configuration

### Configure Ansible

**Inventory file:**
Ansible uses dynamic inventory from Terraform outputs.

**Playbook structure:**
- Main playbook: `ansible/playbook.yml`
- Role: `ansible/roles/web-game/`
- Tasks: Game deployment, Nginx configuration, Docker setup

### Test Ansible Connection

```bash
# After Terraform deployment
cd ansible

# Test connection (after instance is created)
ansible all -i terraform-inventory.sh -m ping
```

---

## Deployment

### Quick Deployment

**Using Makefile (Recommended):**
```bash
# Full deployment
make deploy

# Or step-by-step
make terraform-apply
make ansible-deploy
```

**Manual Deployment:**

**Step 1: Deploy Infrastructure (Terraform)**
```bash
cd terraform

# Initialize
terraform init

# Plan
terraform plan

# Apply
terraform apply

# Note the outputs (public IP, instance ID)
terraform output
```

**Step 2: Deploy Configuration (Ansible)**
```bash
cd ansible

# Wait 60 seconds for SSH to be ready
sleep 60

# Run playbook
ansible-playbook -i terraform-inventory.sh playbook.yml

# Or use Makefile
make ansible-deploy
```

### Verify Deployment

**Check EC2 instance:**
```bash
# Get public IP
cd terraform
terraform output public_ip

# SSH to server
ssh -i ~/.ssh/id_rsa ec2-user@$(terraform output -raw public_ip)

# Check services
sudo systemctl status nginx
sudo docker ps  # Check QuakeJS container
```

**Test web interface:**
```bash
# Get public IP
PUBLIC_IP=$(cd terraform && terraform output -raw public_ip)

# Test game server
curl http://$PUBLIC_IP/

# Test individual games
curl http://$PUBLIC_IP/2048/
curl http://$PUBLIC_IP/pvp/
curl http://$PUBLIC_IP/quakejs/
```

---

## Access & Usage

### Web Access

**Game Server URL:**
- Primary: `https://games.alexflux.com/`
- HTTP: `http://<EC2_PUBLIC_IP>/` (fallback)
- HTTPS: `https://<EC2_PUBLIC_IP>/` (if SSL configured)

**Get Public IP:**
```bash
cd terraform
terraform output public_ip
```

### Game Selection Page

The main page displays all available games with:
- Game icons and titles
- Game descriptions
- Feature lists
- Badge indicators (singleplayer/multiplayer, genre)
- Direct play links

### Individual Games

Each game is accessible at:
- `http://<IP>/2048/` - 2048 puzzle game
- `http://<IP>/pvp/` - PvP multiplayer game
- `http://<IP>/pacman/` - Pac-Man arcade game
- `http://<IP>/mario/` - Super Mario Bros platformer
- `http://<IP>/quakejs/` - QuakeJS (requires Docker)

### SSH Access

```bash
# Get public IP
cd terraform
PUBLIC_IP=$(terraform output -raw public_ip)

# SSH to server
ssh -i ~/.ssh/id_rsa ec2-user@$PUBLIC_IP
```

---

## Adding New Games

### Step-by-Step Guide

#### 1. Create Deployment Task

Create `ansible/roles/web-game/tasks/deploy-[gamename].yml`:

```yaml
---
# Deploy [Game Name] to /[gamename]/ subdirectory

- name: Create [Game Name] game directory
  ansible.builtin.file:
    path: "{{ web_game_dir }}/[gamename]"
    state: directory
    mode: "0755"
    owner: "{{ nginx_user }}"
    group: "{{ nginx_user }}"

- name: Download [Game Name] game
  ansible.builtin.get_url:
    url: https://github.com/user/repo/archive/refs/heads/master.zip
    dest: /tmp/[gamename]-master.zip
    mode: "0644"
  register: game_download

- name: Extract [Game Name] game
  ansible.builtin.unarchive:
    src: /tmp/[gamename]-master.zip
    dest: /tmp
    remote_src: true
    creates: /tmp/[gamename]-master

- name: Copy [Game Name] game files
  ansible.builtin.shell: |
    if [ -d /tmp/[gamename]-master ]; then
      find /tmp/[gamename]-master -mindepth 1 -maxdepth 1 -exec cp -r {} {{ web_game_dir }}/[gamename]/ \;
      chown -R {{ nginx_user }}:{{ nginx_user }} {{ web_game_dir }}/[gamename]/
    fi
  changed_when: true
```

#### 2. Add to Main Deployment

Edit `ansible/roles/web-game/tasks/main.yml`:

```yaml
- name: Deploy [Game Name] game
  ansible.builtin.include_tasks: deploy-[gamename].yml
```

#### 3. Add Game Card

Edit `ansible/roles/web-game/templates/game-selection.html.j2`:

```html
<a href="/[gamename]/" class="game-card">
    <span class="game-icon">🎮</span>
    <h2 class="game-title">[Game Name]</h2>
    <span class="badge singleplayer">Single Player</span>
    <span class="badge action">Action</span>
    <p class="game-description">
        [Game description]
    </p>
    <ul class="game-features">
        <li>Feature 1</li>
        <li>Feature 2</li>
    </ul>
    <span class="play-btn">Play Now →</span>
</a>
```

**Badge options:**
- `singleplayer` - Blue badge
- `multiplayer` - Green badge
- `puzzle` - Orange badge
- `action` - Red badge
- `arcade` - Purple badge
- `platformer` - Pink badge

#### 4. Add Nginx Route

Edit `ansible/roles/web-game/templates/nginx.conf.j2`:

```nginx
location /[gamename]/ {
    alias {{ web_game_dir }}/[gamename]/;
    try_files $uri $uri/ /[gamename]/index.html;
}
```

#### 5. Deploy

```bash
make deploy
```

---

## Maintenance & Operations

### Update Games

**Redeploy with Ansible:**
```bash
cd ansible
ansible-playbook -i terraform-inventory.sh playbook.yml
```

### Update Infrastructure

**Modify Terraform:**
```bash
cd terraform

# Edit configuration files
nano main.tf

# Plan changes
terraform plan

# Apply changes
terraform apply
```

### Backup

**Backup game files:**
```bash
# SSH to server
ssh -i ~/.ssh/id_rsa ec2-user@$(cd terraform && terraform output -raw public_ip)

# Backup to S3
sudo tar -czf /tmp/games-backup-$(date +%Y%m%d).tar.gz /var/www/html/
aws s3 cp /tmp/games-backup-*.tar.gz s3://your-bucket-name/backups/
```

### Monitoring

**Check instance status:**
```bash
# Via AWS CLI
aws ec2 describe-instances --instance-ids $(cd terraform && terraform output -raw instance_id)

# Via Terraform
cd terraform
terraform show
```

**Check CloudWatch metrics:**
```bash
# CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=$(cd terraform && terraform output -raw instance_id) \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average
```

### Logs

**View Nginx logs:**
```bash
# SSH to server
ssh -i ~/.ssh/id_rsa ec2-user@$(cd terraform && terraform output -raw public_ip)

# Access logs
sudo tail -f /var/log/nginx/access.log

# Error logs
sudo tail -f /var/log/nginx/error.log
```

**View QuakeJS logs:**
```bash
# Docker logs
sudo docker logs quakejs
```

---

## Troubleshooting

### Common Issues

**1. Terraform init fails - Cloudflare provider error**

**Solution:** Create `.env` with `CLOUDFLARE_API_TOKEN="dummy"` even if not using Cloudflare.

**2. Ansible fails - Python version error**

**Solution:** Already fixed! Python 3.8 is auto-installed via EC2 user_data.

**3. SSH connection refused**

**Solutions:**
- Verify SSH key path in `.env`
- Check key permissions: `chmod 400 terraform.pem`
- Ensure `ssh_allowed_cidr` matches the IP
- Wait 60 seconds after instance creation for SSH to be ready

**4. Games not loading**

**Solutions:**
- Check Nginx is running: `sudo systemctl status nginx`
- Verify files exist: `ls -la /var/www/html/[gamename]/`
- Check Nginx logs: `sudo tail -f /var/log/nginx/error.log`
- Test locally: `curl http://localhost/[gamename]/`

**5. QuakeJS not working**

**Solutions:**
- Check Docker container: `sudo docker ps`
- View container logs: `sudo docker logs quakejs`
- Verify pak0.pk3 exists: `ls -lh /opt/quakejs/baseoa/pak0.pk3`
- Restart container: `cd /opt/quakejs && sudo docker-compose restart`

**6. Terraform apply fails with "AccessDenied"**

**Solutions:**
```bash
# Verify AWS credentials
aws sts get-caller-identity

# Check IAM permissions (need Admin or Power User)
aws iam get-user --user-name <your-username>
```

**7. "Bucket name already exists" error**

**Solution:** S3 bucket names are globally unique. Change bucket names in `.env` to include your account ID.

**8. Not receiving budget alert emails**

**Solutions:**
1. Check SNS subscription confirmation:
   ```bash
   aws sns list-subscriptions
   # Look for Status="PendingConfirmation"
   ```
2. Check email spam folder
3. Verify budget exists:
   ```bash
   aws budgets describe-budgets --account-id <account-id>
   ```

**9. CloudWatch Billing Alarm stuck in "INSUFFICIENT_DATA"**

**Solutions:**
1. Enable Billing Alerts (one-time): AWS Console → Billing → Preferences → Check "Receive CloudWatch Billing Alerts"
2. Wait 15-30 minutes for metric to appear
3. Verify in us-east-1 region

### Debug Commands

```bash
# Check Terraform state
cd terraform && terraform show

# SSH to server
ssh -i terraform.pem ec2-user@$(terraform output -raw public_ip)

# Check game server status
sudo systemctl status nginx
sudo docker ps  # Check QuakeJS container

# View logs
sudo journalctl -u nginx -f
sudo docker logs quakejs  # View QuakeJS container logs

# Test game URLs
curl http://localhost/2048/
curl http://localhost/pvp/
curl http://localhost/quakejs/  # Test QuakeJS
```

---

## Cost Management

### Estimated Costs

**EC2 Instance (t3.micro):**
- ~$7-10/month (depending on usage)
- Free tier eligible (first 12 months)

**Elastic IP:**
- Free if attached to running instance
- $0.005/hour if not attached

**S3 Storage:**
- ~$0.023/GB/month
- Minimal for backups

**Data Transfer:**
- First 1GB/month free
- $0.09/GB after

**Total Estimated Cost:** ~$8-12/month (without free tier)

### Cost Optimization

**1. Use Free Tier:**
- t2.micro or t3.micro instances
- First 12 months free (750 hours/month)

**2. Stop Instance When Not in Use:**
```bash
# Stop instance
aws ec2 stop-instances --instance-ids $(cd terraform && terraform output -raw instance_id)

# Start instance
aws ec2 start-instances --instance-ids $(cd terraform && terraform output -raw instance_id)
```

**3. Use Spot Instances:**
- Modify Terraform to use spot instances
- Up to 90% cost savings
- Risk of interruption

**4. Set Up Billing Alarms:**
- Already configured in Terraform
- Receive alerts at $10, $25, $50 thresholds

### Monitor Costs

**AWS Cost Explorer:**
- AWS Console → Cost Management → Cost Explorer
- View daily/monthly costs
- Forecast future costs

**CloudWatch Billing Alarms:**
- Configured via Terraform
- Email notifications at thresholds

---

## Key Takeaways

### Critical Success Factors

1. **AWS Credentials**: Valid AWS access keys with proper permissions
2. **SSH Key**: Correct permissions and path configuration
3. **Unique Names**: S3 bucket names must be globally unique
4. **Network Access**: Security group allows the IP for SSH
5. **Wait Time**: Allow 60 seconds after instance creation for SSH

### Common Pitfalls

1. **Wrong SSH Key Permissions**: Must be 400 (`chmod 400`)
2. **S3 Bucket Name Conflicts**: Use unique names with account ID
3. **Security Group Rules**: Ensure the IP is whitelisted
4. **Python Version**: EC2 user_data handles this automatically
5. **Cloudflare Token**: Use dummy value if not using Cloudflare

### Best Practices

1. **Version Control**: Store `.env` in `.gitignore`, use `.env.example`
2. **State Management**: Use Terraform remote state (S3 backend)
3. **Backup Strategy**: Regular backups to S3
4. **Cost Monitoring**: Set up billing alarms
5. **Security**: Use least privilege IAM policies

---

## Conclusion

This project deployed a web-based game server on AWS using Infrastructure as Code (Terraform) and configuration management (Ansible), covering modern DevOps practices and cloud infrastructure management.

**Final Status:**
- ✅ Terraform infrastructure deployed
- ✅ Ansible configuration applied
- ✅ Game server accessible via web
- ✅ Multiple games deployed
- ✅ Cost monitoring configured

**Next Steps:**
- Add more games
- Configure custom domain
- Set up SSL/TLS
- Implement automated backups
- Add monitoring and alerting

---

## Additional Resources

### Official Documentation

- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Ansible AWS Modules](https://docs.ansible.com/ansible/latest/collections/amazon/aws/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [AWS Cost Management](https://docs.aws.amazon.com/cost-management/)

### Repository

- [aws_iac-game-server GitHub](https://github.com/alexlux58/aws_iac-game-server)

---

**End of Lab Writeup**

