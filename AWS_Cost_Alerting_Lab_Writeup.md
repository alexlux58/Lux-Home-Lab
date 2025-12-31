# AWS Cost Alerting System
## Daily Cost Reports & Budget Monitoring

**Author:** Alex Lux  
**Date:** December 2025  
**Environment:** AWS Serverless (Lambda + EventBridge + SES)  
**Repository:** [AWS-ALERTING](https://github.com/alexlux58/AWS-ALERTING)

> **Quick Reference Available**: For command cheat sheets and quick troubleshooting, see `QUICK_REFERENCE.md` in this directory.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [Initial Setup](#initial-setup)
5. [Infrastructure Deployment](#infrastructure-deployment)
6. [Email Configuration](#email-configuration)
7. [Access & Usage](#access--usage)
8. [Configuration Management](#configuration-management)
9. [Maintenance & Operations](#maintenance--operations)
10. [Troubleshooting](#troubleshooting)
11. [Integration with Other Lab Services](#integration-with-other-lab-services)

---

## Project Overview

### Objective
Deploy an automated AWS cost monitoring and alerting system that sends daily cost reports via email at 7am, providing visibility into AWS spending and helping prevent cost overruns. The system uses serverless architecture with Infrastructure as Code (Terraform) for easy deployment and management.

### Why This System?

**Automated Cost Monitoring:**
- Daily cost reports via email
- Budget alerts and thresholds
- Top services cost breakdown
- Month-to-date and daily cost tracking

**Serverless Architecture:**
- No server management
- Pay-per-use pricing
- Auto-scaling
- High availability

**Infrastructure as Code:**
- Terraform for deployment
- Version controlled
- Reproducible
- Easy to modify

### Features

- ✅ **Daily Cost Reports**: Email at 7am with previous day's costs
- ✅ **Budget Alerts**: AWS Budget integration with SNS notifications
- ✅ **Top Services**: Breakdown of highest cost services
- ✅ **Cost Drivers**: Identification of cost drivers
- ✅ **Report Archiving**: S3 storage for historical reports
- ✅ **Remediation**: Optional automatic cost remediation (stop tagged instances)
- ✅ **Monitoring**: CloudWatch alarms for system health

### Use Cases

- **Daily Cost Review**: Start each day with cost visibility
- **Budget Management**: Stay within budget thresholds
- **Cost Optimization**: Identify high-cost services
- **Anomaly Detection**: Spot unusual spending patterns
- **Compliance**: Track and report on cloud spending

---

## Architecture Overview

### Deployment Architecture

```
AWS Account
   ↓
Terraform Infrastructure
   ├── Lambda Functions
   │   ├── cost-alerting-reporter (Daily cost reports)
   │   └── cost-alerting-remediation (Optional cost remediation)
   ├── EventBridge Scheduler
   │   └── Daily 7am trigger
   ├── AWS SES
   │   ├── Verified sender email
   │   └── Verified recipient email
   ├── S3 Bucket
   │   └── Report archives
   ├── SSM Parameter Store
   │   └── Runtime configuration
   ├── CloudWatch Alarms
   │   └── System health monitoring
   ├── AWS Budget
   │   └── Budget alerts
   └── SSM Automation Document
       └── Cost remediation automation
```

### Network Configuration

- **Lambda Functions**: Deployed in AWS Lambda (serverless)
- **SES**: Email service (us-east-1 for best deliverability)
- **S3**: Report storage
- **EventBridge**: Scheduled execution (us-east-1)

### Repository Structure

```
AWS-ALERTING/
├── infra/                    # Terraform infrastructure
│   ├── *.tf                 # Terraform configuration files
│   ├── ssm-automation-document.yaml  # SSM Automation YAML
│   └── terraform.tfvars.example     # Configuration template
├── lambda/                   # Cost reporter Lambda
│   ├── app.py               # Main Lambda handler
│   └── requirements.txt     # Python dependencies
├── cost_remediation_lambda/ # Remediation Lambda
│   └── app.py              # Remediation handler
├── scripts/                 # Helper scripts
├── README.md                # Project documentation
├── Makefile                 # Automation commands
└── terraform-policy.json    # IAM policy for testing
```

---

## Prerequisites

### Software Requirements

- **Terraform**: Version 1.0+ (latest recommended)
- **Python**: Version 3.9+ (for Lambda functions)
- **AWS CLI**: Version 2.0+ (for AWS operations)
- **Git**: For cloning repository
- **Make**: For build automation (optional)

### AWS Requirements

- **AWS Account**: Active AWS account
- **AWS Credentials**: Access key and secret key
- **IAM Permissions**: Admin or Power User access
- **SES**: Email service (may need to request production access)
- **Cost Explorer**: Must be enabled (free)

### Verify Installation

```bash
# Check Terraform
terraform --version

# Check Python
python3 --version

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
git clone https://github.com/alexlux58/AWS-ALERTING.git
cd AWS-ALERTING
```

### Configure Terraform Variables

**Create `terraform.tfvars` file:**
```bash
cd infra
cp terraform.tfvars.example terraform.tfvars
nano terraform.tfvars
```

**Required variables:**
```bash
# Email Configuration
report_from = "alerts@yourdomain.com"
report_to   = "alex.lux@example.com"

# Schedule Configuration
schedule_time = "7:00"  # 24-hour format
schedule_timezone = "America/New_York"

# Report Configuration
top_n_services = 10
include_mtd = true
include_drivers = true

# Optional: Remediation
enable_remediation = false
remediation_tags = {
  "CostControl" = "AutoStop"
}
```

### Verify Email Addresses in SES

**Important:** SES requires email verification before sending emails.

**Verify sender email:**
```bash
aws ses verify-email-identity \
  --email-address alerts@yourdomain.com \
  --region us-east-1
```

**Verify recipient email (if in sandbox mode):**
```bash
aws ses verify-email-identity \
  --email-address alex.lux@example.com \
  --region us-east-1
```

**Check your inbox** for verification emails and click the verification links!

**Check verification status:**
```bash
aws ses get-identity-verification-attributes \
  --identities alerts@yourdomain.com alex.lux@example.com \
  --region us-east-1
```

---

## Infrastructure Deployment

### Deploy with Terraform

**Navigate to Terraform directory:**
```bash
cd infra
```

**Initialize Terraform:**
```bash
terraform init
```

**Plan deployment:**
```bash
terraform plan
```

**Apply infrastructure:**
```bash
terraform apply
```

**Expected resources:**
- Lambda functions (reporter and optional remediation)
- EventBridge Scheduler (daily 7am trigger)
- S3 bucket (report archives)
- SSM parameters (configuration)
- SES email identities
- CloudWatch alarms
- AWS Budget (optional)
- SSM Automation document (if remediation enabled)

### Verify Deployment

**Check Terraform outputs:**
```bash
terraform output
```

**Test Lambda function:**
```bash
aws lambda invoke \
  --function-name cost-alerting-reporter \
  --region us-east-1 \
  --payload '{}' \
  /tmp/cost-report.json && cat /tmp/cost-report.json
```

**Check EventBridge Scheduler:**
```bash
aws scheduler get-schedule \
  --name cost-alerting-daily-8pm \
  --group-name default \
  --region us-east-1 \
  --query 'State'
```

**Expected:** `"ENABLED"`

---

## Email Configuration

### SES Sandbox vs Production

**Sandbox Mode (Default):**
- Can only send to verified email addresses
- Limited to 200 emails/day
- Good for testing

**Production Mode:**
- Can send to any email address
- Higher sending limits
- Requires AWS support request

### Request Production Access

**Via AWS Console:**
1. Go to AWS SES Console
2. Click "Request production access"
3. Fill out the form
4. Wait for approval (usually 24-48 hours)

**Via AWS CLI:**
```bash
aws sesv2 put-account-details \
  --production-access-enabled \
  --mail-type TRANSACTIONAL \
  --website-url https://yourdomain.com \
  --use-case-description "Daily cost reporting for AWS account" \
  --region us-east-1
```

### Email Deliverability

**Common Issues:**
- Emails going to spam
- Unverified email addresses
- SES sandbox limitations

**Solutions:**
- Verify all email addresses
- Check spam/junk folders
- Request SES production access
- Configure SPF/DKIM records (if using custom domain)

**See:** `EMAIL_DELIVERABILITY.md` in repository for detailed guide.

---

## Access & Usage

### Daily Cost Reports

**Report Schedule:**
- **Time**: 7:00 AM (configurable)
- **Timezone**: America/New_York (configurable)
- **Frequency**: Daily

**Report Contents:**
- Previous day's total cost
- Month-to-date cost (if enabled)
- Top N services breakdown
- Cost drivers (if enabled)
- Link to archived report in S3

![AWS Cost Alert Email](images/AWS-costalert-email.png)

*Example daily cost report email showing previous day's costs, month-to-date totals, and top services breakdown.*

### Manual Report Generation

**Invoke Lambda manually:**
```bash
aws lambda invoke \
  --function-name cost-alerting-reporter \
  --region us-east-1 \
  --payload '{}' \
  /tmp/cost-report.json && cat /tmp/cost-report.json
```

**Check email:**
- Report should arrive within 1-2 minutes
- Check spam/junk folder if not received

### View Archived Reports

**List reports in S3:**
```bash
S3_BUCKET=$(cd infra && terraform output -raw archive_bucket)
aws s3 ls s3://$S3_BUCKET/reports/ --recursive
```

**Download a report:**
```bash
aws s3 cp s3://$S3_BUCKET/reports/2025-12-19-report.json /tmp/report.json
cat /tmp/report.json
```

### Budget Alerts

**Configure AWS Budget:**
- Budget is created automatically via Terraform
- Alerts at 50%, 80%, and 100% of budget
- SNS notifications sent to configured email

**View budget:**
```bash
aws budgets describe-budgets \
  --account-id $(aws sts get-caller-identity --query Account --output text) \
  --region us-east-1
```

---

## Configuration Management

### Update Configuration via SSM

**Change recipient email (no redeploy needed):**
```bash
# Verify new email in SES first
aws ses verify-email-identity \
  --email-address new-email@example.com \
  --region us-east-1

# Update SSM parameter
aws ssm put-parameter \
  --name "/cost-alerting/report_to" \
  --type "String" \
  --value "new-email@example.com" \
  --overwrite \
  --region us-east-1
```

**Change report configuration:**
```bash
# Update top N services
aws ssm put-parameter \
  --name "/cost-alerting/top_n_services" \
  --type "String" \
  --value "15" \
  --overwrite \
  --region us-east-1

# Toggle month-to-date
aws ssm put-parameter \
  --name "/cost-alerting/include_mtd" \
  --type "String" \
  --value "false" \
  --overwrite \
  --region us-east-1
```

### Update Configuration via Terraform

**Edit `terraform.tfvars`:**
```bash
cd infra
nano terraform.tfvars
```

**Apply changes:**
```bash
terraform plan
terraform apply
```

### Available SSM Parameters

- `/cost-alerting/report_to` - Recipient email
- `/cost-alerting/report_from` - Sender email
- `/cost-alerting/archive_bucket` - S3 bucket name
- `/cost-alerting/top_n_services` - Number of top services to show
- `/cost-alerting/include_mtd` - Include month-to-date (true/false)
- `/cost-alerting/include_drivers` - Include cost drivers (true/false)

---

## Maintenance & Operations

### View Logs

**Lambda function logs:**
```bash
# Tail logs for cost reporter
aws logs tail /aws/lambda/cost-alerting-reporter --since 1h --region us-east-1

# Filter for errors
aws logs filter-log-events \
  --log-group-name /aws/lambda/cost-alerting-reporter \
  --filter-pattern "ERROR" \
  --region us-east-1
```

**EventBridge Scheduler logs:**
```bash
# Check scheduler execution history
aws scheduler list-schedule-groups --region us-east-1
```

### Monitor System Health

**Check CloudWatch alarms:**
```bash
aws cloudwatch describe-alarms \
  --alarm-name-prefix cost-alerting \
  --region us-east-1 \
  --query 'MetricAlarms[*].[AlarmName,StateValue]'
```

**Expected alarms:**
- `cost-alerting-reporter-errors` - Lambda errors
- `cost-alerting-reporter-duration` - Lambda duration
- `cost-alerting-scheduler-dlq-messages` - Failed invocations

### Check Dead Letter Queue

**View DLQ messages:**
```bash
aws sqs receive-message \
  --queue-url $(aws sqs get-queue-url \
    --queue-name cost-alerting-scheduler-dlq \
    --query QueueUrl --output text) \
  --region us-east-1
```

### Update Lambda Code

**Redeploy via Terraform:**
```bash
cd infra
terraform apply -target=module.lambda -auto-approve
```

**Manual update:**
```bash
cd lambda
zip -r /tmp/cost-reporter.zip app.py
aws lambda update-function-code \
  --function-name cost-alerting-reporter \
  --zip-file fileb:///tmp/cost-reporter.zip \
  --region us-east-1
```

### Clean Up Old Reports

**S3 lifecycle policy:**
- Automatically configured via Terraform
- Reports older than 90 days are deleted
- Can be customized in `infra/s3.tf`

**Manual cleanup:**
```bash
S3_BUCKET=$(cd infra && terraform output -raw archive_bucket)
# Delete reports older than 90 days
aws s3api list-objects-v2 \
  --bucket $S3_BUCKET \
  --prefix reports/ \
  --query 'Contents[?LastModified<`2024-09-01`].Key' \
  --output text | \
  xargs -I {} aws s3 rm s3://$S3_BUCKET/{}
```

---

## Troubleshooting

### Common Issues

**1. Emails Not Received**

**Most common cause:** Unverified email addresses

**Solutions:**
```bash
# Check verification status
aws ses get-identity-verification-attributes \
  --identities alerts@yourdomain.com alex.lux@example.com \
  --region us-east-1

# Verify emails
aws ses verify-email-identity \
  --email-address alerts@yourdomain.com \
  --region us-east-1
aws ses verify-email-identity \
  --email-address alex.lux@example.com \
  --region us-east-1
```

**Check:**
- Inbox (including spam/junk folder)
- Lambda logs for errors
- SES can send test email
- SSM parameter has correct email address

**2. Lambda Not Receiving Invocations**

**Check scheduler:**
```bash
aws scheduler get-schedule \
  --name cost-alerting-daily-8pm \
  --group-name default \
  --region us-east-1 \
  --query 'State'
```

**Should return:** `"ENABLED"`

**Check DLQ:**
```bash
aws sqs receive-message \
  --queue-url $(aws sqs get-queue-url \
    --queue-name cost-alerting-scheduler-dlq \
    --query QueueUrl --output text)
```

**3. Cost Explorer Queries Failing**

**Important:** Cost Explorer API is only available in `us-east-1`

**Check:**
- Lambda region is us-east-1
- Lambda has `ce:GetCostAndUsage` permission
- Cost Explorer is enabled in AWS account
- CloudWatch logs for detailed error messages

**4. Changing Email Address**

**Quick method (no redeploy):**
```bash
# Step 1: Verify new email
aws ses verify-email-identity \
  --email-address new-email@example.com \
  --region us-east-1

# Step 2: Update SSM parameter
aws ssm put-parameter \
  --name "/cost-alerting/report_to" \
  --type "String" \
  --value "new-email@example.com" \
  --overwrite \
  --region us-east-1

# Step 3: Test
aws lambda invoke \
  --function-name cost-alerting-reporter \
  --region us-east-1 \
  --payload '{}' \
  /tmp/cost-report.json
```

**Permanent method (via Terraform):**
```bash
# Step 1: Update terraform.tfvars
cd infra
nano terraform.tfvars
# Change: report_to = "new-email@example.com"

# Step 2: Verify email
aws ses verify-email-identity \
  --email-address new-email@example.com \
  --region us-east-1

# Step 3: Apply
terraform apply
```

**5. SSM Automation Document Creation Fails**

**Check:**
- `infra/ssm-automation-document.yaml` exists
- YAML syntax is valid
- File encoding is UTF-8

### Verification Checklist

**Lambda Functions:**
```bash
aws lambda list-functions \
  --query 'Functions[?contains(FunctionName, `cost-alerting`)].FunctionName'
```

**S3 Buckets:**
```bash
aws s3 ls | grep cost-alerting
```

**EventBridge Scheduler:**
```bash
aws scheduler list-schedules \
  --schedule-group default \
  --query 'Schedules[?contains(Name, `cost-alerting`)].Name'
```

**SES Identities:**
```bash
aws ses list-identities --query 'Identities'
```

**SSM Parameters:**
```bash
aws ssm describe-parameters \
  --parameter-filters "Key=Name,Values=/cost-alerting/" \
  --query 'Parameters[*].Name'
```

**CloudWatch Alarms:**
```bash
aws cloudwatch describe-alarms \
  --alarm-name-prefix cost-alerting \
  --query 'MetricAlarms[*].AlarmName'
```

---

## Integration with Other Lab Services

### Monitor with Observability Stack

**Add CloudWatch metrics to Prometheus:**
```yaml
# prometheus/prometheus.yml
- job_name: 'cloudwatch'
  static_configs:
    - targets: ['cloudwatch-exporter:9106']
```

**Monitor Lambda invocations:**
- CloudWatch metrics automatically available
- Can be scraped by Prometheus
- Visualize in Grafana

### Document in NetBox/Nautobot

**Track infrastructure:**
- Document Lambda functions
- Track S3 buckets
- Monitor costs and usage

### Automate with DevOps Tools

**Jenkins Pipeline:**
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy Cost Alerting') {
            steps {
                sh 'cd infra && terraform apply -auto-approve'
            }
        }
    }
}
```

**Ansible Playbook:**
```yaml
- name: Deploy AWS Cost Alerting
  hosts: localhost
  tasks:
    - name: Apply Terraform
      command: terraform apply -auto-approve
      args:
        chdir: infra
```

### Use with Other AWS Projects

**Monitor AWS Game Server costs:**
- Daily reports include all AWS costs
- Track EC2 instance costs
- Monitor S3 storage costs

**Monitor NetKnife costs:**
- Track Lambda invocation costs
- Monitor CloudFront data transfer
- Track API Gateway costs

---

## Key Takeaways

### Critical Success Factors

1. **Email Verification**: All email addresses must be verified in SES
2. **Region**: Cost Explorer API only works in us-east-1
3. **SSM Parameters**: Configuration stored in SSM for runtime updates
4. **Scheduler**: EventBridge Scheduler must be enabled
5. **IAM Permissions**: Lambda needs Cost Explorer and SES permissions

### Common Pitfalls

1. **Unverified Emails**: Most common issue - emails not verified
2. **Wrong Region**: Cost Explorer only in us-east-1
3. **SES Sandbox**: Limited to verified emails only
4. **Missing Permissions**: Lambda needs proper IAM permissions
5. **Timezone Confusion**: Schedule time in 24-hour format

### Best Practices

1. **Email Verification**: Verify all emails before deployment
2. **Production SES**: Request production access for better deliverability
3. **Monitoring**: Set up CloudWatch alarms
4. **Archiving**: Keep historical reports for analysis
5. **Budget Alerts**: Use AWS Budget for threshold alerts

---

## Conclusion

This project deployed an automated AWS cost monitoring and alerting system, providing daily cost visibility and helping prevent cost overruns through proactive monitoring and alerting.

**Final Status:**
- ✅ Terraform infrastructure deployed
- ✅ Lambda functions operational
- ✅ EventBridge Scheduler configured
- ✅ Email reports working
- ✅ Budget alerts configured
- ✅ Monitoring in place

**Next Steps:**
- Review daily cost reports
- Optimize high-cost services
- Adjust budget thresholds
- Enable remediation (if needed)
- Integrate with other monitoring tools

---

## Additional Resources

### Official Documentation

- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [AWS Cost Explorer API](https://docs.aws.amazon.com/cost-management/latest/APIReference/API_GetCostAndUsage.html)
- [AWS SES Documentation](https://docs.aws.amazon.com/ses/)
- [EventBridge Scheduler Documentation](https://docs.aws.amazon.com/scheduler/)

### Repository

- [AWS-ALERTING GitHub](https://github.com/alexlux58/AWS-ALERTING)

---

## Related Projects

This project monitors costs for all AWS lab services:

- **AWS Game Server**: Tracks EC2 instance costs, S3 storage, and data transfer
- **NetKnife**: Monitors Lambda invocation costs, API Gateway, CloudFront, and DynamoDB usage
- **DevOps Tools**: Can track AWS CLI/SDK usage costs if using AWS services
- **Network Observability Stack**: Can integrate CloudWatch metrics for cost correlation

For complete lab documentation, see `README.md` in the HomeLabWriteup directory.

---

**End of Lab Writeup**

