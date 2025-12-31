# NetKnife - Network & Security Swiss Army Knife
## Serverless Web Application for Network Engineers

**Author:** Alex Lux  
**Date:** December 2025  
**Environment:** AWS Serverless (Lambda + CloudFront)  
**Repository:** [NetKnife](https://github.com/alexlux58/NetKnife)

> **Quick Reference Available**: For command cheat sheets and quick troubleshooting, see `QUICK_REFERENCE.md` in this directory.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [Initial Setup](#initial-setup)
5. [Infrastructure Deployment](#infrastructure-deployment)
6. [Frontend Deployment](#frontend-deployment)
7. [Access & Usage](#access--usage)
8. [Adding New Tools](#adding-new-tools)
9. [Maintenance & Operations](#maintenance--operations)
10. [Troubleshooting](#troubleshooting)
11. [Integration with Other Lab Services](#integration-with-other-lab-services)

---

## Project Overview

### Objective
Deploy a serverless web application providing network engineers with essential tools accessible from any browser without installing local software. NetKnife combines offline browser-based tools with remote API-powered tools, all deployed on AWS using Infrastructure as Code (Terraform).

### Why NetKnife?

**Serverless Architecture:**
- No server management
- Auto-scaling
- Pay-per-use pricing
- High availability

**Browser-Based:**
- No local installation required
- Works on any device
- Offline tools run entirely in browser
- Cross-platform compatibility

**Custom Tools:**
- Tailored to network engineer needs
- Combines multiple tools in one interface
- Similar to CyberChef but customized

### Features

**Offline Tools (Browser-only, no data leaves your machine):**
- **Network Calculators**: Subnet/CIDR calculator, CIDR range checker, IP address converter
- **Security Tools**: Password generator, hash generator, PEM decoder
- **Development Helpers**: Regex helper, JWT decoder, encoder/decoder, timestamp converter, cron builder

**Remote Tools (API-powered):**
- **DNS Tools**: DNS lookup, reverse DNS, DNS propagation check
- **Network Analysis**: RDAP lookup, TLS inspector, SSL Labs-style analysis, HTTP headers scanner
- **BGP & Routing**: PeeringDB query, ASN information, BGP looking glass, traceroute
- **Security Analysis**: Email auth (SPF/DKIM/DMARC), HIBP password breach check, IP reputation (AbuseIPDB), Shodan, VirusTotal, SecurityTrails, Censys, GreyNoise

### Use Cases

- **Network Troubleshooting**: DNS lookups, traceroute, BGP queries
- **Security Analysis**: IP reputation, certificate inspection, breach checking
- **Development**: Regex testing, encoding/decoding, timestamp conversion
- **Network Planning**: Subnet calculations, CIDR range checking
- **Quick Reference**: All tools in one place, accessible from anywhere

---

## Architecture Overview

### Deployment Architecture

```
AWS Account
   ↓
Terraform Infrastructure
   ├── API Gateway (HTTP API)
   ├── Lambda Functions (Backend)
   │   ├── DNS lookup
   │   ├── RDAP lookup
   │   ├── TLS inspector
   │   ├── SSL Labs analysis
   │   ├── HTTP headers scanner
   │   ├── PeeringDB query
   │   ├── ASN details
   │   ├── BGP looking glass
   │   ├── Traceroute
   │   ├── Reverse DNS
   │   ├── Email auth (SPF/DKIM/DMARC)
   │   ├── HIBP breach check
   │   ├── AbuseIPDB reputation
   │   ├── Shodan integration
   │   ├── VirusTotal integration
   │   ├── SecurityTrails integration
   │   ├── Censys integration
   │   └── GreyNoise integration
   ├── AWS Cognito (Authentication)
   ├── DynamoDB (Caching)
   ├── S3 (Frontend hosting)
   ├── CloudFront (CDN)
   └── CloudWatch (Logging & Monitoring)
```

### Network Configuration

- **Frontend**: CloudFront distribution with custom domain (e.g., `tools.alexflux.com`)
- **API**: API Gateway HTTP API endpoint
- **Authentication**: AWS Cognito user pool
- **Storage**: S3 for frontend, DynamoDB for caching

### Repository Structure

```
NetKnife/
├── frontend/                # React TypeScript frontend
│   ├── src/
│   │   ├── tools/
│   │   │   ├── offline/    # Browser-only tools
│   │   │   └── remote/     # API-powered tools
│   │   └── ...
│   └── ...
├── backend/
│   └── functions/          # Lambda function code
│       ├── dns/
│       ├── rdap/
│       ├── tls/
│       └── ...
├── infra/                  # Terraform IaC
│   ├── modules/
│   │   ├── api/           # API Gateway + Lambdas
│   │   ├── auth/          # Cognito
│   │   ├── static_site/  # S3 + CloudFront
│   │   ├── ops/           # CloudWatch alarms
│   │   └── cost/          # Budgets + anomaly detection
│   └── envs/
│       └── dev/           # Development environment
└── scripts/                # Helper scripts
```

---

## Prerequisites

### Software Requirements

- **Terraform**: Version 1.0+ (latest recommended)
- **Node.js**: Version 18+ (for frontend build)
- **npm** or **yarn**: Package manager
- **AWS CLI**: Version 2.0+ (for AWS operations)
- **Git**: For cloning repository

### AWS Requirements

- **AWS Account**: Active AWS account
- **AWS Credentials**: Access key and secret key
- **IAM Permissions**: Admin or Power User access
- **Domain**: Optional (for custom domain via Cloudflare)
- **API Keys**: For external services (Shodan, VirusTotal, etc.) - optional

### Verify Installation

```bash
# Check Terraform
terraform --version

# Check Node.js
node --version
npm --version

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
git clone https://github.com/alexlux58/NetKnife.git
cd NetKnife
```

### Configure Environment Variables

**Create `.env` file in `infra/envs/dev/`:**
```bash
cd infra/envs/dev
cp .env.example .env
nano .env
```

**Required variables:**
```bash
# AWS Configuration
AWS_REGION=us-west-2
AWS_ACCOUNT_ID=your-account-id

# Domain Configuration (Optional)
CLOUDFLARE_API_TOKEN=your-token
CLOUDFLARE_ZONE_ID=your-zone-id
CLOUDFLARE_ZONE_NAME=alexflux.com
CLOUDFLARE_SUBDOMAIN=tools

# API Keys (Optional, for external services)
SHODAN_API_KEY=your-shodan-key
VIRUSTOTAL_API_KEY=your-virustotal-key
SECURITYTRAILS_API_KEY=your-securitytrails-key
CENSYS_API_ID=your-censys-id
CENSYS_API_SECRET=your-censys-secret
GREYNOISE_API_KEY=your-greynoise-key
```

### Install Frontend Dependencies

```bash
cd frontend
npm install
```

---

## Infrastructure Deployment

### Deploy with Terraform

**Navigate to Terraform directory:**
```bash
cd infra/envs/dev
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
- API Gateway HTTP API
- Lambda functions (one per tool)
- AWS Cognito user pool
- DynamoDB table (for caching)
- S3 bucket (for frontend)
- CloudFront distribution
- CloudWatch alarms
- Cost budgets

### Verify Deployment

**Check Terraform outputs:**
```bash
terraform output
```

**Expected outputs:**
- API Gateway URL
- CloudFront distribution ID
- Cognito user pool ID
- S3 bucket name

**Test API Gateway:**
```bash
API_URL=$(terraform output -raw api_gateway_url)
curl -X POST "$API_URL/dns" \
  -H "Content-Type: application/json" \
  -d '{"name": "google.com", "type": "A"}'
```

---

## Frontend Deployment

### Build Frontend

```bash
cd frontend

# Install dependencies (if not done)
npm install

# Build for production
npm run build
```

### Deploy to S3

**Get S3 bucket name:**
```bash
cd infra/envs/dev
S3_BUCKET=$(terraform output -raw s3_bucket_name)
```

**Sync to S3:**
```bash
cd ../../../
cd frontend
aws s3 sync dist/ s3://$S3_BUCKET --delete --region us-west-2
```

### Invalidate CloudFront Cache

**Get CloudFront distribution ID:**
```bash
cd infra/envs/dev
CF_DIST_ID=$(terraform output -raw cloudfront_distribution_id)
```

**Create invalidation:**
```bash
aws cloudfront create-invalidation \
  --distribution-id $CF_DIST_ID \
  --paths "/*" \
  --region us-west-2
```

### Automated Deployment Script

**Create deployment script:**
```bash
#!/bin/bash
# deploy.sh

cd frontend
npm run build

cd ../infra/envs/dev
S3_BUCKET=$(terraform output -raw s3_bucket_name)
CF_DIST_ID=$(terraform output -raw cloudfront_distribution_id)

cd ../../../
cd frontend
aws s3 sync dist/ s3://$S3_BUCKET --delete --region us-west-2

aws cloudfront create-invalidation \
  --distribution-id $CF_DIST_ID \
  --paths "/*" \
  --region us-west-2

echo "Deployment complete!"
```

---

## Access & Usage

### Web Access

**Production URL:**
- Custom domain: `https://tools.alexflux.com` (if configured)
- CloudFront URL: `https://<distribution-id>.cloudfront.net`

**Get CloudFront URL:**
```bash
cd infra/envs/dev
terraform output cloudfront_url
```

### Authentication

**Create User:**
```bash
# Via AWS CLI
aws cognito-idp admin-create-user \
  --user-pool-id $(cd infra/envs/dev && terraform output -raw cognito_user_pool_id) \
  --username alex.lux \
  --user-attributes Name=email,Value=alex.lux@example.com \
  --temporary-password "TempPass123!" \
  --message-action SUPPRESS \
  --region us-west-2
```

**Set Permanent Password:**
```bash
aws cognito-idp admin-set-user-password \
  --user-pool-id $(cd infra/envs/dev && terraform output -raw cognito_user_pool_id) \
  --username alex.lux \
  --password "YourSecurePassword123!" \
  --permanent \
  --region us-west-2
```

**Login:**
1. Navigate to NetKnife URL
2. Click "Login"
3. Enter username and password
4. Access authenticated tools

### Using Tools

**Offline Tools:**
- Work entirely in browser
- No data sent to server
- Instant results
- Examples: Subnet calculator, password generator, regex helper

**Remote Tools:**
- Require authentication
- Make API calls to Lambda functions
- Results cached in DynamoDB
- Examples: DNS lookup, TLS inspector, IP reputation

---

## Adding New Tools

### Offline Tool (Browser-only)

**Step 1: Create Tool Component**
```typescript
// frontend/src/tools/offline/YourTool.tsx
import React, { useState } from 'react';

export const YourTool: React.FC = () => {
  const [input, setInput] = useState('');
  const [output, setOutput] = useState('');

  const process = () => {
    // Your tool logic here
    setOutput(processInput(input));
  };

  return (
    <div className="tool-container">
      <h2>Your Tool</h2>
      <textarea value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={process}>Process</button>
      <textarea value={output} readOnly />
    </div>
  );
};
```

**Step 2: Register in Registry**
```typescript
// frontend/src/tools/registry.tsx
import { YourTool } from './offline/YourTool';

export const tools = [
  // ... existing tools
  {
    id: 'your-tool',
    name: 'Your Tool',
    category: 'offline',
    component: YourTool,
  },
];
```

**Step 3: Deploy Frontend**
```bash
cd frontend
npm run build
# Deploy to S3 and invalidate CloudFront
```

### Remote Tool (API-powered)

**Step 1: Create Lambda Function**
```javascript
// backend/functions/yourtool/index.js
exports.handler = async (event) => {
  const { body } = event;
  const params = JSON.parse(body);

  // Your tool logic here
  const result = await processTool(params);

  return {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*',
    },
    body: JSON.stringify(result),
  };
};
```

**Step 2: Add Terraform Resources**
```hcl
# infra/modules/api/main.tf
resource "aws_lambda_function" "yourtool" {
  filename         = "yourtool.zip"
  function_name    = "${var.app_name}-${var.env}-yourtool"
  role            = aws_iam_role.lambda.arn
  handler         = "index.handler"
  runtime         = "nodejs18.x"
  source_code_hash = data.archive_file.yourtool.output_base64sha256
}

resource "aws_apigatewayv2_route" "yourtool" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "POST /yourtool"
  target    = "integrations/${aws_apigatewayv2_integration.yourtool.id}"
}
```

**Step 3: Create Frontend Component**
```typescript
// frontend/src/tools/remote/YourTool.tsx
// Similar to offline tool but makes API calls
```

**Step 4: Deploy**
```bash
# Deploy infrastructure
cd infra/envs/dev
terraform apply

# Deploy frontend
cd ../../../
cd frontend
npm run build
# Deploy to S3
```

---

## Maintenance & Operations

### View Logs

**Lambda Function Logs:**
```bash
# Tail logs for a Lambda function
aws logs tail /aws/lambda/netknife-dev-dns --since 5m --region us-west-2

# Filter for errors
aws logs filter-log-events \
  --log-group-name /aws/lambda/netknife-dev-dns \
  --filter-pattern "ERROR" \
  --region us-west-2
```

**CloudFront Logs:**
```bash
# Check CloudFront distribution status
aws cloudfront get-distribution \
  --id $(cd infra/envs/dev && terraform output -raw cloudfront_distribution_id) \
  --query 'Distribution.Status'
```

### Test Lambda Functions

**Direct Lambda Invocation:**
```bash
# Test DNS Lambda
aws lambda invoke \
  --function-name netknife-dev-dns \
  --payload '{"body": "{\"name\": \"google.com\", \"type\": \"A\"}"}' \
  --cli-binary-format raw-in-base64-out \
  --region us-west-2 \
  /tmp/dns-response.json && cat /tmp/dns-response.json
```

### Update Lambda Functions

**Redeploy via Terraform:**
```bash
cd infra/envs/dev
terraform apply -target=module.api -auto-approve
```

**Manual Update:**
```bash
cd backend/functions/dns
zip -r /tmp/dns.zip index.js
aws lambda update-function-code \
  --function-name netknife-dev-dns \
  --zip-file fileb:///tmp/dns.zip \
  --region us-west-2
```

### Cache Management

**View Cache Entries:**
```bash
aws dynamodb scan \
  --table-name netknife-dev-cache \
  --region us-west-2 \
  --max-items 10
```

**Delete Cache Entry:**
```bash
aws dynamodb delete-item \
  --table-name netknife-dev-cache \
  --key '{"cache_key": {"S": "dns:google.com:A"}}' \
  --region us-west-2
```

### User Management

**List Users:**
```bash
aws cognito-idp list-users \
  --user-pool-id $(cd infra/envs/dev && terraform output -raw cognito_user_pool_id) \
  --region us-west-2
```

**Reset User Password:**
```bash
aws cognito-idp admin-set-user-password \
  --user-pool-id $(cd infra/envs/dev && terraform output -raw cognito_user_pool_id) \
  --username alex.lux \
  --password "NewPassword123!" \
  --permanent \
  --region us-west-2
```

---

## Troubleshooting

### Common Issues

**1. Lambda 500 Errors**

**Check CloudWatch logs:**
```bash
aws logs tail /aws/lambda/netknife-dev-dns --since 5m --region us-west-2
```

**Test Lambda directly:**
```bash
aws lambda invoke \
  --function-name netknife-dev-dns \
  --payload '{"body": "{\"name\": \"example.com\", \"type\": \"A\"}"}' \
  --cli-binary-format raw-in-base64-out \
  --region us-west-2 \
  /tmp/test.json && cat /tmp/test.json
```

**2. Authentication Issues**

**Check user status:**
```bash
aws cognito-idp admin-get-user \
  --user-pool-id $(cd infra/envs/dev && terraform output -raw cognito_user_pool_id) \
  --username alex.lux \
  --region us-west-2
```

**Force user re-verification:**
```bash
# Disable and re-enable user
aws cognito-idp admin-disable-user \
  --user-pool-id $(cd infra/envs/dev && terraform output -raw cognito_user_pool_id) \
  --username alex.lux

aws cognito-idp admin-enable-user \
  --user-pool-id $(cd infra/envs/dev && terraform output -raw cognito_user_pool_id) \
  --username alex.lux
```

**3. Frontend Not Updating**

**Invalidate CloudFront cache:**
```bash
aws cloudfront create-invalidation \
  --distribution-id $(cd infra/envs/dev && terraform output -raw cloudfront_distribution_id) \
  --paths "/*" \
  --region us-west-2
```

**Check S3 sync:**
```bash
S3_BUCKET=$(cd infra/envs/dev && terraform output -raw s3_bucket_name)
aws s3 ls s3://$S3_BUCKET --recursive
```

**4. API Gateway Issues**

**Check API routes:**
```bash
aws apigatewayv2 get-routes \
  --api-id $(cd infra/envs/dev && terraform output -raw api_gateway_id) \
  --region us-west-2
```

**5. DynamoDB Cache Issues**

**Clear cache:**
```bash
# Items with TTL will auto-expire
# For immediate clearing, scan and delete
aws dynamodb scan --table-name netknife-dev-cache --region us-west-2 \
  --projection-expression "cache_key" \
  --query "Items[*].cache_key.S" --output text | \
  xargs -I {} aws dynamodb delete-item \
    --table-name netknife-dev-cache \
    --key '{"cache_key": {"S": "{}"}}' \
    --region us-west-2
```

**6. Terraform State Issues**

**Refresh state:**
```bash
cd infra/envs/dev
terraform refresh
```

**Show state:**
```bash
terraform state list
terraform state show module.api.aws_lambda_function.dns
```

**7. DNS / Custom Domain Issues**

**Check Cloudflare DNS:**
```bash
curl -X GET "https://api.cloudflare.com/client/v4/zones/$CLOUDFLARE_ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" | jq '.result[] | {name, type, content}'
```

**Check ACM certificate:**
```bash
aws acm describe-certificate \
  --certificate-arn $(cd infra/envs/dev && terraform output -raw acm_certificate_arn) \
  --region us-east-1
```

### Health Check Script

```bash
#!/bin/bash
# health-check.sh

API_URL=$(cd infra/envs/dev && terraform output -raw api_gateway_url)
SITE_URL="https://tools.alexflux.com"

echo "=== NetKnife Health Check ==="

# Check frontend
echo -n "Frontend: "
curl -s -o /dev/null -w "%{http_code}" "$SITE_URL" && echo " ✓" || echo " ✗"

# Check API (will return 401 without auth, that's OK)
echo -n "API Gateway: "
STATUS=$(curl -s -o /dev/null -w "%{http_code}" -X POST "$API_URL/dns")
[[ "$STATUS" == "401" || "$STATUS" == "200" ]] && echo "$STATUS ✓" || echo "$STATUS ✗"

# Check Lambda
echo -n "DNS Lambda: "
aws lambda invoke --function-name netknife-dev-dns \
  --payload '{"body": "{\"name\": \"example.com\", \"type\": \"A\"}"}' \
  --cli-binary-format raw-in-base64-out \
  --region us-west-2 /tmp/health.json 2>/dev/null && echo "✓" || echo "✗"

echo "=== Done ==="
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
- Document API Gateway endpoints
- Track CloudFront distributions
- Monitor costs and usage

### Automate with DevOps Tools

**Jenkins Pipeline:**
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy Infrastructure') {
            steps {
                sh 'cd infra/envs/dev && terraform apply -auto-approve'
            }
        }
        stage('Build Frontend') {
            steps {
                sh 'cd frontend && npm run build'
            }
        }
        stage('Deploy Frontend') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

**Ansible Playbook:**
```yaml
- name: Deploy NetKnife
  hosts: localhost
  tasks:
    - name: Apply Terraform
      command: terraform apply -auto-approve
      args:
        chdir: infra/envs/dev
```

---

## Key Takeaways

### Critical Success Factors

1. **AWS Credentials**: Valid AWS access keys with proper permissions
2. **Terraform State**: Proper state management (consider S3 backend)
3. **CloudFront Cache**: Remember to invalidate after deployments
4. **Cognito Users**: Proper user management and password policies
5. **API Keys**: External service API keys for full functionality

### Common Pitfalls

1. **Forgot CloudFront Invalidation**: Changes not visible immediately
2. **Lambda Timeout**: Some tools may need longer timeout
3. **CORS Issues**: Ensure API Gateway CORS configured correctly
4. **Cache Issues**: DynamoDB cache may need manual clearing
5. **State Drift**: Terraform state may drift from actual resources

### Best Practices

1. **Version Control**: Store all code in Git
2. **Environment Variables**: Use `.env` files, never commit secrets
3. **State Management**: Use remote state (S3 backend)
4. **Monitoring**: Set up CloudWatch alarms
5. **Cost Management**: Monitor Lambda invocations and data transfer

---

## Conclusion

This project deployed NetKnife, a serverless network and security tool suite, covering modern serverless architecture, Infrastructure as Code, and web application development practices.

**Final Status:**
- ✅ Terraform infrastructure deployed
- ✅ Lambda functions operational
- ✅ Frontend deployed to CloudFront
- ✅ Authentication configured
- ✅ All tools accessible

**Next Steps:**
- Add more tools
- Enhance UI/UX
- Add monitoring dashboards
- Implement automated deployments
- Add more external service integrations

---

## Additional Resources

### Official Documentation

- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [AWS API Gateway Documentation](https://docs.aws.amazon.com/apigateway/)
- [AWS CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)
- [AWS Cognito Documentation](https://docs.aws.amazon.com/cognito/)

### Repository

- [NetKnife GitHub](https://github.com/alexlux58/NetKnife)

---

**End of Lab Writeup**

