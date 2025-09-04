# AWS RDS MariaDB with EC2 Bastion Setup

**Deploy a secure MariaDB RDS instance in a private subnet, accessible via an EC2-based bastion host in a public subnet. Full step-by-step infrastructure creation, connection, SQL/test routines, and clean teardown.**

***

## Architecture Overview

- **Custom VPC:** Isolated network, with public and private subnets.
- **Bastion EC2 Host:** In public subnet for SSH and DB client access.
- **MariaDB RDS:** Deployed in private subnet, never exposed publicly.
- **Security Groups:** Fine-grained control for SSH (22), RDS (3306), etc.
- **Automated Scripts:** Shell/SQL scripts for database and data operations.

***

## Prerequisites

- AWS CLI configured with necessary permissions.
- An SSH key pair, e.g. `key.pem`
- Familiarity with basic AWS Console and CLI operations.

***

## Step-by-Step Lab

### 1. Create VPC and Subnets

- VPC with CIDR: `10.0.0.0/16`
    - Public Subnet (e.g. `10.0.1.0/24`)
    - Private Subnet (e.g. `10.0.2.0/24`)
- Attach an Internet Gateway to the VPC.
- Route all outbound traffic from the public subnet to the Internet Gateway.

### 2. Set Up Routing & Security

- Create a public route table; associate with public subnet.
- Main route table for private subnet (no direct internet route).
- Security Groups:
    - Bastion EC2: Allow SSH (`22`) only from trusted IPs.
    - RDS: Allow `3306` only from Bastion EC2 security group.
- Optional: Configure Network ACLs if additional control is desired.

### 3. Launch Bastion EC2 Instance

- Instance Type: t3.micro or similar.
- AMI: Amazon Linux 2/2023.
- Subnet: Public subnet with auto-assigned public IP.
- Assign SSH key (e.g. `key.pem`).
- Inbound rules: SSH (`22`) from your IP only.

**Install MariaDB client on the Bastion:**

```sh
sudo dnf update -y
sudo dnf install mariadb105-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

### 4. Configure RDS MariaDB Instance

- Engine: MariaDB (latest supported)
- Identifier: e.g. `rds-mariadb-lab`
- Instance class: db.t3.micro
- Storage: 20 GB (gp2/gp3)
- Place inside private subnet.
- Set **Public access** to **No**.
- Security group: Only allow `3306` access from Bastion EC2 security group.

### 5. Connect from Bastion to RDS

On Bastion, connect using:

```sh
mysql -h <RDS-ENDPOINT> -u admin -p
```
Use master password set during RDS creation.

***

### 6. Run SQL Setup and Test

**Create database and table:**

```sql
CREATE DATABASE testdb;
USE testdb;
CREATE TABLE benchmark (
  id INT AUTO_INCREMENT PRIMARY KEY,
  data VARCHAR(255)
);
```

**Bulk load data:**

```sql
DELIMITER $$
CREATE PROCEDURE insert_data()
BEGIN
    DECLARE i INT DEFAULT 1;
    WHILE i <= 10000 DO
        INSERT INTO benchmark (data) VALUES (UUID());
        SET i = i + 1;
    END WHILE;
END$$
DELIMITER ;
CALL insert_data();
```

**Verify row count:**

```sql
SELECT COUNT(*) FROM benchmark;
```

**Cleanup:**

```sql
DELETE FROM benchmark;
DROP TABLE IF EXISTS benchmark;
DROP DATABASE testdb;
```

***

### 7. Teardown (Clean Up)

- Delete RDS instance.
- Terminate Bastion EC2 instance.
- Delete subnets, route tables, internet gateway, and VPC.
- Remove any associated security groups.

***


## Best Practices

- Restrict ingress/egress at every layer (SGs, NACL, least privilege).
- Never enable public access on database.
- Monitor with CloudWatch, log connections, rotate credentials.
- Always fully delete test resources to avoid AWS billing surprises.

***

## Connect

<a href="https://www.linkedin.com/in/hiranmaya-biswas-505a1823a/" target="_blank">
  <img src="https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin" alt="LinkedIn" height="30">
</a>
<a href="https://github.com/Harry-404" target="_blank" style="margin-left:10px;">
  <img src="https://img.shields.io/badge/GitHub-Follow-black?logo=github" alt="GitHub" height="30">
</a>

