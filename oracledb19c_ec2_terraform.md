# Oracle Database 19c on AWS EC2 - Terraform Script

This Terraform script provisions an AWS EC2 instance and automatically installs Oracle Database 19c with all necessary configurations.

## 📋 Prerequisites

- **Terraform** installed (version 0.12+)
- **AWS CLI** configured with appropriate credentials
- **Oracle Database 19c** installation files (downloaded from Oracle's official website)
- **AWS Key Pair** created in your target region
- **Oracle Account** with valid license for Database 19c

## ⚠️ Important Legal Notice

Oracle Database 19c requires a valid Oracle license. Ensure you have proper licensing before using this script in production environments. This script is for educational and development purposes.

## 🚀 Quick Start

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd oracle-db-terraform
   ```

2. **Initialize Terraform**
   ```bash
   terraform init
   ```

3. **Customize variables** (see Configuration section)

4. **Plan and apply**
   ```bash
   terraform plan
   terraform apply
   ```

## ⚙️ Configuration

### Required Modifications

Before running this script, you **must** update the following:

| Item | Location | Description |
|------|----------|-------------|
| `key_name` | `aws_instance.oracle_db` | Replace with your AWS key pair name |
| Oracle Download URL | `user_data` script | Update with actual Oracle 19c download path |
| Response Files | `user_data` script | Ensure response files match your requirements |
| AWS Region | `provider "aws"` | Change to your preferred region |

### AWS Provider Configuration

```hcl
provider "aws" {
  region = "us-west-2"  # Modify as needed
}
```

### Variables (Recommended Enhancement)

Add these variables for better flexibility:

```hcl
variable "aws_region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-west-2"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.large"  # Recommended for Oracle DB
}

variable "key_name" {
  description = "AWS Key Pair name"
  type        = string
}

variable "oracle_download_url" {
  description = "URL to Oracle Database 19c installation files"
  type        = string
  sensitive   = true
}
```

## 🖥️ EC2 Instance Configuration

```hcl
resource "aws_instance" "oracle_db" {
  ami           = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI (update for your region)
  instance_type = "t3.large"              # Upgraded from t2.medium for better performance
  key_name      = var.key_name
  
  # Security group with Oracle ports
  vpc_security_group_ids = [aws_security_group.oracle_sg.id]
  
  # Increased storage for Oracle installation
  root_block_device {
    volume_type = "gp3"
    volume_size = 50  # GB
    encrypted   = true
  }
  
  tags = {
    Name        = "OracleDBInstance"
    Environment = "Development"
    Database    = "Oracle19c"
  }

  user_data = base64encode(templatefile("${path.module}/oracle_install.sh", {
    oracle_url = var.oracle_download_url
  }))
}
```

## 🔒 Security Group Configuration

```hcl
resource "aws_security_group" "oracle_sg" {
  name_prefix = "oracle-db-sg"
  description = "Security group for Oracle Database instance"

  # SSH access
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Restrict this in production
  }

  # Oracle Database port
  ingress {
    from_port   = 1521
    to_port     = 1521
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/8"]  # Restrict to your VPC
  }

  # Oracle Enterprise Manager
  ingress {
    from_port   = 5500
    to_port     = 5500
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/8"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "oracle-db-security-group"
  }
}
```

## 📝 Installation Script (oracle_install.sh)

Create a separate file `oracle_install.sh` for better maintainability:

```bash
#!/bin/bash

# Oracle Database 19c Installation Script
# This script runs during EC2 instance initialization

set -e  # Exit on any error

LOG_FILE="/var/log/oracle_install.log"

log() {
    echo "$(date): $1" | tee -a $LOG_FILE
}

log "Starting Oracle Database 19c installation"

# System preparation
log "Updating system packages"
yum update -y
yum groupinstall -y "Development Tools"
yum install -y wget unzip libaio bc flex

# Create Oracle user and groups
log "Creating Oracle user and groups"
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
useradd -u 54321 -g oinstall -G dba,oper oracle

# Create Oracle directories
log "Creating Oracle directories"
mkdir -p /u01/app/oracle/product/19.0.0/dbhome_1
mkdir -p /u01/app/oraInventory
chown -R oracle:oinstall /u01/app/oracle
chown -R oracle:oinstall /u01/app/oraInventory
chmod -R 775 /u01/app/oracle

# System configuration
log "Configuring system parameters"
cat >> /etc/sysctl.conf << 'EOF'
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
fs.file-max = 6815744
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
EOF

sysctl -p

# Download Oracle Database 19c
log "Downloading Oracle Database 19c"
cd /tmp
wget "${oracle_url}" -O oracle19c.zip
unzip oracle19c.zip -d oracle19c/

# Prepare response files
log "Preparing installation response files"
cat > /tmp/db_install.rsp << 'EOF'
oracle.install.option=INSTALL_DB_SWONLY
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/u01/app/oraInventory
ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.OSDBA_GROUP=dba
oracle.install.db.OSOPER_GROUP=oper
oracle.install.db.OSBACKUPDBA_GROUP=dba
oracle.install.db.OSDGDBA_GROUP=dba
oracle.install.db.OSKMDBA_GROUP=dba
oracle.install.db.OSRACDBA_GROUP=dba
DECLINE_SECURITY_UPDATES=true
EOF

cat > /tmp/dbca.rsp << 'EOF'
responseFileVersion=/oracle/assistants/rspfmt_dbca.xml
gdbName=ORCLDB
sid=ORCLDB
createAsContainerDatabase=false
templateName=General_Purpose.dbc
sysPassword=OraclePassword123
systemPassword=OraclePassword123
datafileDestination=/u01/app/oracle/oradata
recoveryAreaDestination=/u01/app/oracle/fast_recovery_area
characterSet=AL32UTF8
nationalCharacterSet=AL16UTF16
listeners=LISTENER
automaticMemoryManagement=true
totalMemory=2048
EOF

# Install Oracle Database software
log "Installing Oracle Database software"
cd /tmp/oracle19c
sudo -u oracle ./runInstaller -silent -responseFile /tmp/db_install.rsp -waitforcompletion

# Run root scripts
log "Running root configuration scripts"
/u01/app/oraInventory/orainstRoot.sh
/u01/app/oracle/product/19.0.0/dbhome_1/root.sh

# Set environment variables
log "Setting up Oracle environment"
cat >> /home/oracle/.bash_profile << 'EOF'
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export ORACLE_SID=ORCLDB
export PATH=$PATH:$ORACLE_HOME/bin
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
EOF

# Source the environment
source /home/oracle/.bash_profile

# Create database
log "Creating Oracle database"
sudo -u oracle $ORACLE_HOME/bin/dbca -silent -createDatabase -responseFile /tmp/dbca.rsp

# Configure listener
log "Configuring Oracle listener"
sudo -u oracle $ORACLE_HOME/bin/netca -silent -responseFile $ORACLE_HOME/assistants/netca/netca.rsp

# Start listener
sudo -u oracle $ORACLE_HOME/bin/lsnrctl start

# Verify installation
log "Verifying Oracle installation"
ORACLE_STATUS=$(sudo -u oracle bash -c "source /home/oracle/.bash_profile && echo 'SELECT status FROM v\$instance;' | sqlplus -s / as sysdba" | grep -i "OPEN" || true)

if [[ -n "$ORACLE_STATUS" ]]; then
    log "Oracle Database installation completed successfully"
    log "Database Status: $ORACLE_STATUS"
else
    log "Oracle Database installation may have issues. Check logs for details."
    exit 1
fi

# Configure automatic startup
log "Configuring automatic startup"
sed -i 's/:N/:Y/g' /etc/oratab

# Create systemd service
cat > /etc/systemd/system/oracle.service << 'EOF'
[Unit]
Description=Oracle Database
After=network.target

[Service]
Type=forking
User=oracle
ExecStart=/u01/app/oracle/product/19.0.0/dbhome_1/bin/dbstart /u01/app/oracle/product/19.0.0/dbhome_1
ExecStop=/u01/app/oracle/product/19.0.0/dbhome_1/bin/dbshut /u01/app/oracle/product/19.0.0/dbhome_1
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

systemctl enable oracle.service
systemctl start oracle.service

log "Oracle Database 19c installation and configuration completed!"
```

## 📤 Outputs

```hcl
output "instance_id" {
  description = "EC2 Instance ID"
  value       = aws_instance.oracle_db.id
}

output "public_ip" {
  description = "Public IP address of the instance"
  value       = aws_instance.oracle_db.public_ip
}

output "private_ip" {
  description = "Private IP address of the instance"
  value       = aws_instance.oracle_db.private_ip
}

output "oracle_connection_string" {
  description = "Oracle database connection string"
  value       = "${aws_instance.oracle_db.public_ip}:1521/ORCLDB"
}

output "ssh_command" {
  description = "SSH command to connect to the instance"
  value       = "ssh -i ${var.key_name}.pem ec2-user@${aws_instance.oracle_db.public_ip}"
}
```

## 🔍 Post-Installation Verification

After the Terraform apply completes:

1. **SSH into the instance:**
   ```bash
   ssh -i your-key-pair.pem ec2-user@<public-ip>
   ```

2. **Check Oracle processes:**
   ```bash
   sudo su - oracle
   ps -ef | grep oracle
   ```

3. **Connect to the database:**
   ```bash
   sqlplus / as sysdba
   SELECT status FROM v$instance;
   ```

4. **Check listener status:**
   ```bash
   lsnrctl status
   ```

## 🛡️ Security Best Practices

- **Network Security**: Restrict security group rules to specific IP ranges
- **Encryption**: Enable encryption at rest and in transit
- **Passwords**: Use strong passwords and consider AWS Systems Manager Parameter Store
- **Updates**: Regularly update the system and Oracle patches
- **Monitoring**: Implement CloudWatch monitoring and logging

## 💰 Cost Considerations

- **Instance Type**: `t3.large` is recommended minimum for Oracle DB
- **Storage**: Consider using GP3 volumes for better cost/performance
- **Reserved Instances**: Use RI for production workloads
- **Auto-shutdown**: Implement auto-shutdown for development environments

## 🧹 Cleanup

To destroy all resources:

```bash
terraform destroy
```

## 📚 Additional Resources

- [Oracle Database 19c Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**⚠️ Disclaimer**: This script is provided for educational and development purposes. Ensure you have proper Oracle licensing and follow your organization's security policies before using in production.
