# Multi-Cloud Network Infrastructure with Terraform

A Terraform configuration that creates Virtual Private Cloud (VPC) infrastructure on AWS or Virtual Network (VNet) infrastructure on Azure based on provider selection.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
- [Configuration](#configuration)
- [AWS Resources](#aws-resources)
- [Azure Resources](#azure-resources)
- [Variables](#variables)
- [Outputs](#outputs)
- [Contributing](#contributing)

## Overview

This Terraform script provides a flexible solution for deploying network infrastructure across multiple cloud providers. Using a simple variable configuration, you can deploy either AWS VPC or Azure VNet resources with their associated networking components.

## Features

### AWS Configuration
- VPC with configurable CIDR block
- Public and private subnets across multiple availability zones
- Internet Gateway for public internet access
- NAT Gateway for private subnet internet access
- Route tables with proper associations
- Security group with common ports (SSH, HTTP, HTTPS, Oracle DB)

### Azure Configuration
- Resource Group creation
- Virtual Network with configurable address space
- Public and private subnets
- Network Security Group (configuration to be completed)

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) >= 0.12
- AWS CLI configured (for AWS deployment)
- Azure CLI configured (for Azure deployment)
- Appropriate cloud provider credentials

## Usage

1. Clone this repository:
```bash
git clone <repository-url>
cd <repository-name>
```

2. Initialize Terraform:
```bash
terraform init
```

3. Plan the deployment:
```bash
# For AWS deployment
terraform plan -var="provider=aws"

# For Azure deployment
terraform plan -var="provider=azure"
```

4. Apply the configuration:
```bash
# For AWS deployment
terraform apply -var="provider=aws"

# For Azure deployment
terraform apply -var="provider=azure"
```

5. Destroy resources when no longer needed:
```bash
terraform destroy -var="provider=<aws|azure>"
```

## Configuration

### Provider Configuration

```hcl
variable "provider" {
  description = "The cloud provider to use (aws or azure)"
  type        = string
  default     = "aws"
}

provider "aws" {
  region = "us-west-2"
  count  = var.provider == "aws" ? 1 : 0
}

provider "azurerm" {
  features {}
  count = var.provider == "azure" ? 1 : 0
}
```

## AWS Resources

When `provider = "aws"`, the following resources are created:

| Resource Type | Name | CIDR/Configuration |
|---------------|------|-------------------|
| VPC | aws-vpc | 10.0.0.0/16 |
| Public Subnet 1 | aws-public-subnet-1 | 10.0.1.0/24 |
| Public Subnet 2 | aws-public-subnet-2 | 10.0.2.0/24 |
| Private Subnet 1 | aws-private-subnet-1 | 10.0.3.0/24 |
| Private Subnet 2 | aws-private-subnet-2 | 10.0.4.0/24 |
| Internet Gateway | aws-igw | - |
| NAT Gateway | aws-nat-gateway | In public-subnet-1 |
| Security Group | aws-security-group | Ports: 22, 80, 443, 1521 |

### Security Group Rules
- **Inbound**: SSH (22), HTTP (80), HTTPS (443), Oracle DB (1521)
- **Outbound**: All traffic allowed

## Azure Resources

When `provider = "azure"`, the following resources are created:

| Resource Type | Name | CIDR/Configuration |
|---------------|------|-------------------|
| Resource Group | azure-resource-group | West US |
| Virtual Network | azure-vnet | 10.0.0.0/16 |
| Public Subnet 1 | azure-public-subnet-1 | 10.0.1.0/24 |
| Public Subnet 2 | azure-public-subnet-2 | 10.0.2.0/24 |
| Private Subnet 1 | azure-private-subnet-1 | 10.0.3.0/24 |
| Private Subnet 2 | azure-private-subnet-2 | 10.0.4.0/24 |

> **Note**: Azure configuration appears incomplete in the source. Additional resources like Network Security Group, NAT Gateway equivalent, and route tables may need to be added.

## Variables

| Variable | Description | Type | Default |
|----------|-------------|------|---------|
| provider | The cloud provider to use | string | "aws" |

## Outputs

Consider adding outputs for:
- VPC/VNet ID
- Subnet IDs
- Security Group IDs
- NAT Gateway IPs
- Route Table IDs

## Network Architecture

```
┌─────────────────────────────────────┐
│              VPC/VNet               │
│            10.0.0.0/16              │
│                                     │
│  ┌─────────────┐  ┌─────────────┐   │
│  │   Public    │  │   Public    │   │
│  │  Subnet 1   │  │  Subnet 2   │   │
│  │10.0.1.0/24  │  │10.0.2.0/24  │   │
│  └─────────────┘  └─────────────┘   │
│                                     │
│  ┌─────────────┐  ┌─────────────┐   │
│  │   Private   │  │   Private   │   │
│  │  Subnet 1   │  │  Subnet 2   │   │
│  │10.0.3.0/24  │  │10.0.4.0/24  │   │
│  └─────────────┘  └─────────────┘   │
└─────────────────────────────────────┘
```

## Best Practices

- Always run `terraform plan` before `terraform apply`
- Use remote state storage for production environments
- Implement proper IAM roles and policies
- Regular backup of Terraform state files
- Use consistent naming conventions
- Tag all resources appropriately

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

[Add your license information here]

## Support

For issues and questions:
- Create an issue in this repository
- Check the [Terraform AWS Provider documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- Check the [Terraform AzureRM Provider documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
