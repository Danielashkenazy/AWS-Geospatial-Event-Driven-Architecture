# 🌍 Event-Driven GeoJSON Processing Platform on AWS

[![Terraform](https://img.shields.io/badge/Terraform-IaC-623CE4.svg)](https://www.terraform.io/)
[![AWS](https://img.shields.io/badge/AWS-ECS%20Fargate-FF9900.svg)](https://aws.amazon.com/)
[![Python](https://img.shields.io/badge/Python-3.10-blue.svg)](https://www.python.org/)
[![PostGIS](https://img.shields.io/badge/PostGIS-Spatial%20DB-336791.svg)](https://postgis.net/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED.svg)](https://www.docker.com/)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF.svg)](https://github.com/features/actions)

## 📖 Overview

An **enterprise-grade, event-driven geospatial data processing platform** built entirely with Infrastructure as Code on AWS. This project demonstrates advanced DevOps and cloud-native architecture by combining serverless computing (ECS Fargate), event-driven automation (EventBridge), spatial databases (PostGIS/RDS), and modern CI/CD practices.

The platform automatically validates and processes GeoJSON files uploaded to S3, stores spatial data in a PostGIS-enabled database, and provides on-demand processing of GeoTIFF raster files through a microservice architecture.

### 🎯 What Makes This Unique

- **100% Terraform**: Zero manual configuration, fully reproducible infrastructure
- **Event-Driven Architecture**: S3 → EventBridge → ECS Fargate automation
- **Spatial Data Processing**: PostGIS for vector data, GDAL for raster processing
- **Microservices on Fargate**: Serverless containers with zero management overhead
- **Multi-Tier Security**: VPC isolation, private subnets, IAM least privilege
- **CI/CD Pipeline**: GitHub Actions with automated ECR deployments
- **Modular Design**: Reusable Terraform modules for any AWS region

## 🏗️ Architecture

```
┌───────────────────────────────────────────────────────────────────────────┐
│                           AWS Cloud (us-east-2)                            │
│                                                                            │
│  ┌──────────────────────────────────────────────────────────────────────┐ │
│  │                         VPC (Custom CIDR)                             │ │
│  │                                                                       │ │
│  │  ┌─────────────────┐                      ┌─────────────────┐       │ │
│  │  │ Public Subnet   │                      │ Public Subnet   │       │ │
│  │  │   (Multi-AZ)    │                      │   (Multi-AZ)    │       │ │
│  │  │                 │                      │                 │       │ │
│  │  │  ┌───────────┐  │                      │  ┌───────────┐ │       │ │
│  │  │  │  Bastion  │  │                      │  │    NAT    │ │       │ │
│  │  │  │    Host   │  │                      │  │  Gateway  │ │       │ │
│  │  │  └─────┬─────┘  │                      │  └─────┬─────┘ │       │ │
│  │  └────────┼────────┘                      └────────┼────────┘       │ │
│  │           │                                        │                │ │
│  │           │ SSH Access                             │                │ │
│  │           ▼                                        ▼                │ │
│  │  ┌─────────────────────────────────────────────────────────────┐   │ │
│  │  │              Private Subnet (Multi-AZ)                       │   │ │
│  │  │                                                              │   │ │
│  │  │  ┌──────────────────────────────────────────────────────┐   │   │ │
│  │  │  │      ECS Cluster: geo-processing-cluster             │   │   │ │
│  │  │  │                                                       │   │   │ │
│  │  │  │  ┌──────────────────────────────────────────────┐    │   │   │ │
│  │  │  │  │  Service 1: GeoJSON Validation (Event-Driven) │   │   │   │ │
│  │  │  │  │  - Triggered by S3 upload via EventBridge     │   │   │   │ │
│  │  │  │  │  - Validates GeoJSON structure                │   │   │   │ │
│  │  │  │  │  - Inserts into PostGIS database              │   │   │   │ │
│  │  │  │  │  - ECS Fargate (serverless)                   │   │   │   │ │
│  │  │  │  └──────────────────────────────────────────────┘    │   │   │ │
│  │  │  │                                                       │   │   │ │
│  │  │  │  ┌──────────────────────────────────────────────┐    │   │   │ │
│  │  │  │  │  Service 2: Geo Processing (On-Demand)        │   │   │   │ │
│  │  │  │  │  - Internal ALB (VPC-only access)             │   │   │   │ │
│  │  │  │  │  - POST /process with S3 path                 │   │   │   │ │
│  │  │  │  │  - Processes GeoTIFF files with GDAL          │   │   │   │ │
│  │  │  │  │  - Returns raster metadata (JSON)             │   │   │   │ │
│  │  │  │  │  - ECS Fargate (serverless)                   │   │   │   │ │
│  │  │  │  └──────────────────────────────────────────────┘    │   │   │ │
│  │  │  └──────────────────────────────────────────────────────┘   │   │ │
│  │  │                                                              │   │ │
│  │  │  ┌──────────────────────────────────────────────────────┐   │   │ │
│  │  │  │      RDS PostgreSQL + PostGIS                        │   │   │ │
│  │  │  │  - Multi-AZ for high availability                    │   │   │ │
│  │  │  │  - PostGIS extension for spatial data                │   │   │ │
│  │  │  │  - Automated backups                                 │   │   │ │
│  │  │  │  - Accessible via Bastion or VPC                     │   │   │ │
│  │  │  └──────────────────────────────────────────────────────┘   │   │ │
│  │  └──────────────────────────────────────────────────────────────┘   │ │
│  └──────────────────────────────────────────────────────────────────────┘ │
│                                                                            │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐   │
│  │  S3 Bucket │    │ EventBridge│    │    ECR     │    │    IAM     │   │
│  │ (GeoJSON & │───▶│   Rules    │    │ (Container │    │  (Roles &  │   │
│  │  GeoTIFF)  │    │            │    │  Registry) │    │  Policies) │   │
│  └────────────┘    └────────────┘    └────────────┘    └────────────┘   │
│         │                                     ▲                           │
│         │ S3 Event                            │ CI/CD Push                │
│         ▼                                     │                           │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │              Automated Workflow (EventBridge)                     │    │
│  │  1. .geojson uploaded → 2. Event triggers → 3. ECS Task runs     │    │
│  └──────────────────────────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                     ┌───────────────────────────┐
                     │   GitHub Actions CI/CD     │
                     │  - Build Docker images     │
                     │  - Push to ECR             │
                     │  - Tag with commit SHA     │
                     └───────────────────────────┘
```

## ✨ Key Features

### Infrastructure (Terraform)
- **Modular Architecture**: 5 reusable modules (network, bastion_rds, PaaS, s3_fargate_geojson, microservice)
- **Multi-AZ Deployment**: High availability across availability zones
- **Private Networking**: Services in private subnets, no public exposure
- **Security Groups**: Least privilege network access
- **IAM Automation**: Task execution roles, task roles with minimal permissions
- **State Management**: S3 backend for Terraform state (optional)

### Geospatial Services
- **GeoJSON Validation Service**:
  - Event-driven: Triggered by S3 uploads
  - Validates GeoJSON FeatureCollection structure
  - Stores validated features in PostGIS
  - Automatic geometry type detection
  - SRID 4326 (WGS84) by default

- **Geo Processing Service**:
  - On-demand HTTP API (Flask)
  - Processes GeoTIFF raster files
  - GDAL-based metadata extraction
  - Returns: dimensions, projection, geotransform
  - Internal ALB for VPC-only access

### Database & Storage
- **PostGIS/RDS**: PostgreSQL with spatial extension
- **S3 Bucket**: Private bucket for GeoJSON and GeoTIFF storage
- **Block Public Access**: Enabled by default
- **Automated Backups**: RDS daily snapshots

### CI/CD
- **GitHub Actions**: Automated build and deployment
- **ECR Integration**: Immutable image tags
- **Multi-Image Support**: Separate pipelines for each service
- **Automatic Triggers**: Push to main branch

## 📋 Prerequisites

### Required Tools
- **Terraform**: >= 1.0
- **AWS CLI**: Configured with credentials
- **Docker**: For local testing (optional)
- **SSH Key Pair**: For Bastion host access

### AWS Requirements
- AWS Account with permissions for:
  - VPC, Subnets, Route Tables
  - ECS, Fargate, ECR
  - RDS, S3
  - IAM, EventBridge
  - CloudWatch Logs
- Configured AWS CLI: `aws configure`

## 🚀 Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/Danielashkenazy/GeoJson_Processing_Saas_Paas.git
cd GeoJson_Processing_Saas_Paas
```

### 2. Configure Variables

Create `Terraform/terraform.tfvars`:

```hcl
region                  = "us-east-2"
db_name                 = "geodb"
db_username             = "geouser"
db_password             = "ChangeMe123!"  # Use AWS Secrets Manager in production
key_name                = "your-ec2-key"
key_path                = "~/.ssh/your-key.pem"
allowed_bastion_ip      = "YOUR_IP/32"
instance_type           = "t3.medium"
instance_type_wordpress_ecs = "t3.medium"

# Optional WordPress configuration
WORDPRESS_DB_NAME       = "wordpress"
WORDPRESS_DB_USERNAME   = "wpuser"
WORDPRESS_DB_PASSWORD   = "WpPass123!"
```

### 3. Build and Push Docker Images

```bash
# GeoJSON Validation Service
cd Docker/geojson-app
docker build -t geojson-validator:latest .
docker tag geojson-validator:latest <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com/geojson-validator:latest

# Geo Processing Service
cd ../geo-processing-app
docker build -t geo-processing:latest .
docker tag geo-processing:latest <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com/geo-processing:latest

# Authenticate and push
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com
docker push <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com/geojson-validator:latest
docker push <ACCOUNT_ID>.dkr.ecr.us-east-2.amazonaws.com/geo-processing:latest
```

### 4. Deploy Infrastructure

```bash
cd Terraform

# Initialize Terraform
terraform init

# Review planned changes
terraform plan

# Deploy
terraform apply -auto-approve
```

**Deployment takes ~10-15 minutes** (RDS creation is the bottleneck).

### 5. Verify Deployment

```bash
# Get outputs
terraform output

# Important outputs:
# - bastion_public_ip: For SSH access
# - rds_endpoint: Database connection
# - s3_bucket_name: For uploading files
# - geo_processing_alb_dns: Internal ALB endpoint
```

## 📁 Project Structure

```
GeoJson_Processing_Saas_Paas/
├── Docker/
│   ├── geojson-app/
│   │   ├── app.py              # S3 → PostGIS validation service
│   │   ├── Dockerfile
│   │   └── requirements.txt
│   └── geo-processing-app/
│       ├── app.py              # Flask API for GeoTIFF processing
│       ├── Dockerfile
│       └── Requirements.txt
│
├── Terraform/
│   ├── main.tf                 # Root module
│   ├── variables.tf            # Input variables
│   ├── outputs.tf              # Output values
│   └── modules/
│       ├── network/            # VPC, Subnets, Routing
│       │   ├── main.tf
│       │   ├── outputs.tf
│       │   └── variables.tf
│       ├── bastion_rds/        # Bastion + RDS PostgreSQL
│       │   ├── main.tf
│       │   ├── outputs.tf
│       │   └── variables.tf
│       ├── s3_fargate_geojson/ # S3 + EventBridge + ECS
│       │   ├── main.tf
│       │   ├── outputs.tf
│       │   └── variables.tf
│       ├── microservice/       # Geo Processing Service
│       │   ├── main.tf
│       │   ├── outputs.tf
│       │   └── variables.tf
│       └── PaaS/               # Optional WordPress (bonus)
│           ├── main.tf
│           ├── output.tf
│           └── variables.tf
│
├── .github/
│   └── workflows/
│       └── main.yml            # CI/CD pipeline
│
└── README.md
```

## 🧪 Testing the Platform

### Test 1: GeoJSON Validation Service

Upload a valid GeoJSON file to trigger automatic processing:

```bash
# Create a test GeoJSON file
cat > test.geojson << 'EOF'
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [34.7818, 32.0853]
      },
      "properties": {
        "name": "Haifa, Israel"
      }
    }
  ]
}
EOF

# Upload to S3 (triggers automatic processing)
aws s3 cp test.geojson s3://<S3_BUCKET_NAME>/test.geojson
```

**What happens:**
1. S3 event triggers EventBridge rule
2. EventBridge invokes ECS Fargate task
3. Task validates GeoJSON structure
4. Valid features are inserted into PostGIS
5. Logs appear in CloudWatch

**View logs:**
```bash
aws logs tail /ecs/geo-validation-logs --follow
```

**Verify database:**
```bash
# SSH to Bastion
ssh -i your-key.pem ubuntu@<BASTION_IP>

# Connect to PostgreSQL
psql -h <RDS_ENDPOINT> -U geouser -d geodb

# Query data
SELECT id, name, ST_AsText(geom) FROM geo_features;
```

### Test 2: Geo Processing Service

Process a GeoTIFF file via the internal API:

```bash
# Upload a .tif file to S3
aws s3 cp sample.tif s3://<S3_BUCKET_NAME>/sample.tif

# SSH to Bastion (or any EC2 in the VPC)
ssh -i your-key.pem ubuntu@<BASTION_IP>

# Call the processing API
curl -X POST http://<INTERNAL_ALB_DNS>:8080/process \
  -H "Content-Type: application/json" \
  -d '{
    "s3_path": "s3://<S3_BUCKET_NAME>/sample.tif"
  }'
```

**Expected response:**
```json
{
  "RasterXSize": 1024,
  "RasterYSize": 768,
  "Projection": "GEOGCS[...]",
  "GeoTransform": [...]
}
```

**View processing logs:**
```bash
aws logs tail /ecs/geo-processing-logs --follow
```

## 🔐 Security Best Practices

### Network Security
- **Private Subnets**: All services run in private subnets
- **No Public Exposure**: Only Bastion and NAT have public IPs
- **Security Groups**: Minimal required ports (5432 for RDS, 8080 for ALB)
- **VPC Flow Logs**: Enable for audit trails (recommended)

### IAM Security
- **Task Execution Role**: Pull images, write logs
- **Task Role**: Access S3 bucket, read secrets
- **Least Privilege**: Each service has minimal permissions
- **No Hardcoded Credentials**: Use IAM roles and env vars

### Data Security
- **S3 Block Public Access**: Enabled by default
- **RDS Encryption**: Enable at-rest encryption (recommended)
- **VPC Endpoints**: Use for S3 and ECR (cost optimization)
- **Secrets Manager**: Store DB passwords (production best practice)

## 📊 Monitoring & Logging

### CloudWatch Logs

All services log to CloudWatch:
- `/ecs/geo-validation-logs`
- `/ecs/geo-processing-logs`

```bash
# View logs
aws logs tail /ecs/geo-validation-logs --follow --format short

# Filter logs
aws logs filter-log-events \
  --log-group-name /ecs/geo-validation-logs \
  --filter-pattern "ERROR"
```

### CloudWatch Metrics

Monitor ECS services:
```bash
# CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/ECS \
  --metric-name CPUUtilization \
  --dimensions Name=ServiceName,Value=geojson-validation \
  --start-time 2025-01-01T00:00:00Z \
  --end-time 2025-01-02T00:00:00Z \
  --period 3600 \
  --statistics Average
```

### Health Checks

```bash
# Check ECS service status
aws ecs describe-services \
  --cluster geo-processing-cluster \
  --services geojson-validation geo-processing

# Check RDS status
aws rds describe-db-instances \
  --db-instance-identifier <DB_INSTANCE_ID>

# Check S3 bucket
aws s3 ls s3://<S3_BUCKET_NAME>
```

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

Located in `.github/workflows/main.yml`:

```yaml
name: Build and Deploy to ECR

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
      - name: Configure AWS credentials
      - name: Login to ECR
      - name: Build geojson-validator
      - name: Push geojson-validator
      - name: Build geo-processing
      - name: Push geo-processing
```

### Trigger Deployment

```bash
git add .
git commit -m "Update services"
git push origin main

# GitHub Actions will automatically:
# 1. Build both Docker images
# 2. Push to ECR with commit SHA tag
# 3. Update ECS services (manual step required)
```

## 🐛 Troubleshooting

### ECS Tasks Not Starting

```bash
# Check task status
aws ecs list-tasks --cluster geo-processing-cluster

# Describe stopped tasks
aws ecs describe-tasks \
  --cluster geo-processing-cluster \
  --tasks <TASK_ARN>

# Common issues:
# - Image pull errors (check ECR permissions)
# - Missing environment variables
# - Insufficient resources (CPU/memory)
```

### GeoJSON Validation Not Triggering

```bash
# Check EventBridge rule
aws events describe-rule --name geojson-s3-upload-rule

# Check S3 event notifications
aws s3api get-bucket-notification-configuration \
  --bucket <S3_BUCKET_NAME>

# Verify IAM permissions for EventBridge
```

### RDS Connection Issues

```bash
# Test from Bastion
telnet <RDS_ENDPOINT> 5432

# Check security group rules
aws ec2 describe-security-groups \
  --group-ids <RDS_SG_ID>

# Verify RDS is in correct VPC and subnets
```

### Geo Processing API Not Responding

```bash
# Check ALB target health
aws elbv2 describe-target-health \
  --target-group-arn <TARGET_GROUP_ARN>

# Curl health endpoint
curl http://<INTERNAL_ALB_DNS>:8080/health

# Check security group allows port 8080
```

## 📈 Performance Optimization

### Fargate Resource Allocation

Adjust CPU and memory in Terraform:
```hcl
resource "aws_ecs_task_definition" "geojson_validator" {
  cpu    = "512"   # 0.5 vCPU
  memory = "1024"  # 1 GB RAM
}
```

### RDS Performance

- **Instance Type**: Use db.t3.medium or larger for production
- **Storage**: Use gp3 with provisioned IOPS
- **Connection Pooling**: Implement PgBouncer
- **Indexes**: Add spatial indexes on geometry columns

### Cost Optimization

- **Fargate Spot**: Use Spot capacity for non-critical tasks
- **RDS Reserved Instances**: 1-year or 3-year commitments
- **S3 Lifecycle Policies**: Move old data to Glacier
- **VPC Endpoints**: Reduce NAT Gateway costs

## 📝 TODO / Roadmap

- [ ] Add authentication for geo-processing API
- [ ] Implement API Gateway for public access
- [ ] Add Grafana dashboards for metrics
- [ ] Create automated testing suite
- [ ] Add support for GeoPackage format
- [ ] Implement data retention policies
- [ ] Add CloudFormation templates as alternative
- [ ] Create Helm charts for EKS deployment
- [ ] Add Lambda-based preprocessing
- [ ] Implement data validation webhooks

## 🤝 Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Test your changes thoroughly
4. Submit a Pull Request

## 📄 License

MIT License - see LICENSE file for details

## 🙏 Acknowledgments

- **PostGIS** team for spatial database excellence
- **GDAL** for geospatial data processing
- **AWS** for Fargate and managed services
- **Terraform** community for IaC best practices

## 📞 Contact & Support

- **GitHub Issues**: [Report bugs or request features](https://github.com/Danielashkenazy/GeoJson_Processing_Saas_Paas/issues)
- **Pull Requests**: Contributions welcome!

## 🌟 Show Your Support

If you find this project useful:
- ⭐ Star the repository
- 🍴 Fork for your own projects
- 📢 Share with the GIS community
- 💬 Provide feedback

---

**Built with ❤️ for geospatial data and cloud-native architecture**

*"Map the world, automate the infrastructure!"* 🗺️
