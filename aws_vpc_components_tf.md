# AWS VPC Infrastructure with Terraform

A comprehensive Terraform configuration for creating a production-ready AWS Virtual Private Cloud (VPC) with public and private subnets, NAT Gateway, and security groups.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Resources Created](#resources-created)
- [Security](#security)
- [Customization](#customization)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Overview

This Terraform script creates a robust AWS VPC infrastructure suitable for hosting applications that require both public-facing and private resources. The configuration follows AWS best practices for network security and high availability.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    VPC (10.0.0.0/16)                   │
│                                                         │
│  ┌─────────────────┐              ┌─────────────────┐   │
│  │  Public Subnet 1│              │  Public Subnet 2│   │
│  │   10.0.1.0/24   │              │   10.0.2.0/24   │   │
│  │                 │              │                 │   │
│  │  ┌─────────────┐│              │                 │   │
│  │  │ NAT Gateway ││              │                 │   │
│  │  └─────────────┘│              │                 │   │
│  └─────────────────┘              └─────────────────┘   │
│           │                                │           │
│           │         Internet Gateway       │           │
│           └────────────────┬───────────────┘           │
│                           │                           │
│  ┌─────────────────┐      │       ┌─────────────────┐   │
│  │ Private Subnet 1│      │       │ Private Subnet 2│   │
│  │   10.0.3.0/24   │      │       │   10.0.4.0/24   │   │
│  │                 │      │       │                 │   │
│  └─────────────────┘      │       └─────────────────┘   │
└────────────────────────────┼─────────────────────────────┘
                            │
                      ┌─────────────┐
                      │  Internet   │
                      └─────────────┘
```

## Features

- **High Availability**: Resources spread across multiple subnets
- **Security**: Separate public and private subnets with controlled access
- **Internet Access**: NAT Gateway for secure outbound internet access from private subnets
- **Flexible Security Groups**: Configurable ingress and egress rules
- **Cost Optimized**: Single NAT Gateway to minimize costs while maintaining functionality
- **Production Ready**: Following AWS Well-Architected Framework principles

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) >= 0.14
- [AWS CLI](https://aws.amazon.com/cli/) configured with appropriate credentials
- An AWS account with necessary IAM permissions

### Required IAM Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*"
      ],
      "Resource": "*"
    }
  ]
}
```

## Quick Start

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd <repository-name>
   ```

2. **Initialize Terraform**:
   ```bash
   terraform init
   ```

3. **Review the plan**:
   ```bash
   terraform plan
   ```

4. **Apply the configuration**:
   ```bash
   terraform apply
   ```

5. **Cleanup when done**:
   ```bash
   terraform destroy
   ```

## Configuration

### Provider Configuration

```hcl
provider "aws" {
  region = "us-west-2"  # Change to your preferred region
}
```

**Supported Regions**: All AWS regions where VPC is available

### Network Configuration

| Component | CIDR Block | Description |
|-----------|------------|-------------|
| VPC | 10.0.0.0/16 | Main VPC network |
| Public Subnet 1 | 10.0.1.0/24 | First public subnet |
| Public Subnet 2 | 10.0.2.0/24 | Second public subnet |
| Private Subnet 1 | 10.0.3.0/24 | First private subnet |
| Private Subnet 2 | 10.0.4.0/24 | Second private subnet |

## Resources Created

### Core Networking

| Resource | Name | Type | Description |
|----------|------|------|-------------|
| VPC | main-vpc | `aws_vpc` | Main virtual private cloud |
| Internet Gateway | main-igw | `aws_internet_gateway` | Provides internet access |
| NAT Gateway | main-nat-gateway | `aws_nat_gateway` | Outbound internet for private subnets |
| Elastic IP | - | `aws_eip` | Static IP for NAT Gateway |

### Subnets

| Subnet | CIDR | Type | Auto-assign Public IP |
|--------|------|------|--------------------|
| public-subnet-1 | 10.0.1.0/24 | Public | Yes |
| public-subnet-2 | 10.0.2.0/24 | Public | Yes |
| private-subnet-1 | 10.0.3.0/24 | Private | No |
| private-subnet-2 | 10.0.4.0/24 | Private | No |

### Route Tables

| Route Table | Associated Subnets | Default Route |
|-------------|-------------------|---------------|
| public-route-table | Public Subnets 1 & 2 | Internet Gateway |
| private-route-table | Private Subnets 1 & 2 | NAT Gateway |

## Security

### Security Group Rules

| Direction | Protocol | Port | Source/Destination | Purpose |
|-----------|----------|------|-------------------|---------|
| Inbound | TCP | 22 | 0.0.0.0/0 | SSH Access |
| Inbound | TCP | 80 | 0.0.0.0/0 | HTTP Traffic |
| Inbound | TCP | 443 | 0.0.0.0/0 | HTTPS Traffic |
| Inbound | TCP | 1521 | 0.0.0.0/0 | Oracle Database |
| Outbound | All | All | 0.0.0.0/0 | All Outbound Traffic |

> ⚠️ **Security Notice**: The current configuration allows inbound access from any IP (0.0.0.0/0). Consider restricting these ranges to specific IP blocks for production use.

### Security Improvements

For production environments, consider:

```hcl
# Example: Restrict SSH access to specific IP
ingress {
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["203.0.113.0/24"]  # Your office IP range
}
```

## Customization

### Variables (Recommended Enhancement)

Consider adding these variables to make the configuration more flexible:

```hcl
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b"]
}
```

### Outputs (Recommended Enhancement)

Add outputs to reference resources in other configurations:

```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = [aws_subnet.public_subnet_1.id, aws_subnet.public_subnet_2.id]
}

output "private_subnet_ids" {
  description = "IDs of the private subnets"
  value       = [aws_subnet.private_subnet_1.id, aws_subnet.private_subnet_2.id]
}
```

## Best Practices

### Production Checklist

- [ ] **Enable VPC Flow Logs** for network monitoring
- [ ] **Use specific CIDR blocks** instead of 0.0.0.0/0 for security groups
- [ ] **Add availability zone specifications** for subnets
- [ ] **Implement resource tagging strategy** for cost management
- [ ] **Enable CloudTrail** for API logging
- [ ] **Set up monitoring** with CloudWatch
- [ ] **Use remote state backend** (S3 + DynamoDB)

### Cost Optimization

- **NAT Gateway**: Consider NAT instances for development environments to reduce costs
- **Elastic IPs**: Release unused Elastic IPs to avoid charges
- **Resource Tagging**: Implement comprehensive tagging for cost allocation

### Security Hardening

- **Network ACLs**: Add network-level security controls
- **VPC Endpoints**: Use VPC endpoints for AWS services to avoid internet routing
- **Private Subnets**: Keep databases and application servers in private subnets
- **Least Privilege**: Apply principle of least privilege to security groups

## Troubleshooting

### Common Issues

1. **Insufficient IAM Permissions**
   ```
   Error: UnauthorizedOperation: You are not authorized to perform this operation
   ```
   **Solution**: Ensure your AWS credentials have the necessary EC2 permissions

2. **Region Mismatch**
   ```
   Error: InvalidGroup.NotFound
   ```
   **Solution**: Verify all resources are being created in the same region

3. **CIDR Conflicts**
   ```
   Error: InvalidVpc.Range
   ```
   **Solution**: Ensure CIDR blocks don't overlap with existing VPCs

### Debug Commands

```bash
# Validate configuration
terraform validate

# Check current state
terraform show

# View detailed logs
export TF_LOG=DEBUG
terraform apply
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Setup

```bash
# Install pre-commit hooks
pre-commit install

# Run terraform format
terraform fmt -recursive

# Run validation
terraform validate
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- 📖 [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- 🐛 [Create an Issue](../../issues/new/choose)
- 💬 [Discussions](../../discussions)

## Estimated Costs

> **Note**: Costs may vary by region and usage

| Resource | Estimated Monthly Cost (us-west-2) |
|----------|-----------------------------------|
| NAT Gateway | ~$45.00 |
| Elastic IP | ~$3.65 |
| **Total** | **~$48.65** |

*Data transfer charges apply separately*
