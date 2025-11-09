Geo-Processing Platform – AWS Terraform Deployment
Overview

This project demonstrates the design and deployment of a fully automated, reproducible AWS-based infrastructure using Terraform.
It provides a complete DevOps workflow including infrastructure provisioning, containerized services, automation, and monitoring, while following AWS best practices for networking, IAM, and resource separation.

All infrastructure is built and managed exclusively through Terraform, without any manual configuration.
Once executed, the Terraform scripts provision the entire environment from scratch, including networking, IAM roles, ECS Fargate clusters, ECR repositories, RDS databases, and load balancers.

Architecture Summary

The project deploys a multi-tier architecture comprising several integrated layers:

1. Networking (VPC Layer)

Single VPC with segregated public and private subnets across multiple Availability Zones.

NAT Gateway for outbound access from private subnets.

Internal routing for secure communication between services.

Security Groups following a least-privilege model:

Public access restricted to designated load balancers.

Private access confined to internal VPC communication.

2. Platform as a Service (PaaS)

RDS PostgreSQL with PostGIS extension for spatial data storage and processing.

Private S3 bucket for ingestion of GeoJSON and GeoTIFF files.

All IAM roles and policies defined explicitly through Terraform, ensuring minimal and auditable permissions.

CloudWatch Logs configured for operational visibility.

3. Microservices (Application Layer)

Two independent Fargate-based services are deployed under the same ECS cluster:

Geo Processing Service

Triggered manually via HTTP requests to an internal ALB endpoint.

Accepts a .tif file path (stored in S3).

Processes the file using a containerized Python-based service and logs results to CloudWatch.

Accessible only within the VPC for internal service-to-service communication.

Geo Validation Service

Triggered automatically when a new .geojson file lands in the S3 bucket.

The event is captured by CloudWatch, which invokes an ECS Fargate task through an EventBridge rule.

The service validates the GeoJSON file structure and, if valid, inserts it into the PostGIS-enabled RDS database.

Both services are hosted in private subnets and utilize internal load balancers for isolation and security.

Automation and Deployment
Terraform

Modular structure split across:

network/ – VPC, subnets, routing, and gateways.

bastion_rds/ – Bastion host and RDS setup.

s3_fargate_geojson/ – Event-driven Fargate service for GeoJSON ingestion.

microservice/ – Geo Processing and ECS infrastructure.

PaaS/ – Platform-level resources (RDS, S3, IAM).

Configurable via variables.tf to support multi-region deployment.

Supports full teardown and re-deployment in any AWS region with a single terraform apply.

Continuous Deployment

GitHub Actions pipeline builds, tests, and deploys container images to ECR.

Auto-triggered on merges to the main branch.

Each service image is immutable (image_tag_mutability = "IMMUTABLE") and scanned upon push.

How to Validate the System
1. Testing the Geo Validation Service

Upload a valid .geojson file to the designated S3 bucket (e.g., geojsonbucket-xxxxxxx).

The upload triggers the ECS Fargate task automatically.

Validation results appear in CloudWatch Logs under /ecs/geo-validation-logs.

If the file passes validation, it is loaded into the PostgreSQL RDS database under the spatial table defined in the containerized application.

You can also upload an invalid .geojson file to verify that the logs properly show validation errors.

2. Testing the Geo Processing Service

Obtain the internal ALB DNS name from the Terraform output (e.g., internal-geo-processing-lb-xxxxxxx.us-east-2.elb.amazonaws.com).

Execute the following curl command from a bastion host or EC2 instance within the same VPC:

curl -X POST http://internal-geo-processing-lb-xxxxxxx.us-east-2.elb.amazonaws.com:8080/process \
     -H "Content-Type: application/json" \
     -d '{"s3_path": "s3://geojsonbucket-xxxxxxx/sample.tif"}'


Processing results and logs will appear in CloudWatch under /ecs/geo-processing-logs.

Security Practices

Minimal IAM permissions with separate roles for execution and task operations.

Internal communication restricted to VPC CIDR ranges.

No public exposure of private services.

S3 buckets have block public access enabled.

RDS accessible only via Bastion host or through internal VPC.

Reproducibility

The entire infrastructure can be recreated end-to-end by running:

terraform init
terraform apply


All parameters, including region, CIDR ranges, and service names, are configurable through Terraform variables.

Future Extensions

Deploy both services on a Kubernetes cluster (EKS or local k3s).

Integrate AWS Step Functions for orchestrated workflows.

Add CI/CD testing stages and automated rollback mechanisms.

Implement monitoring dashboards using Prometheus and Grafana.
