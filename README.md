# AWS MariaDB with EC2 Bastion Lab

This repository demonstrates a complete AWS hands-on scenario:
- **Custom VPC** with public/private subnets
- **EC2 Bastion Host** in public subnet for secure access
- **RDS MariaDB** instance in private subnet
- Network ACLs, security groups, and full destroy steps
- SQL procedures for data load and performance test

## Prerequisites
- AWS CLI & credentials
- SSH key pair

## Usage
1. Deploy networking & compute using IaC or AWS Console
2. SSH to Bastion, install client tools
3. Launch & connect to RDS, run SQL init/data scripts
4. Clean up all resources!

## Structure
- infra/: Terraform/CloudFormation
- scripts/: shell automation
- sql/: database/schema scripts
