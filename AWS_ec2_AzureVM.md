# Multi-Cloud Terraform Script

This Terraform script provisions cloud resources based on the selected provider:
- **AWS**: Creates an EC2 instance
- **Azure**: Creates a Virtual Machine with supporting infrastructure

## Prerequisites

- Terraform installed (version 0.12+)
- Valid AWS credentials (for AWS deployment)
- Azure CLI configured (for Azure deployment)
- SSH key pair for AWS EC2 access

## Usage

1. Clone this repository
2. Initialize Terraform:
   ```bash
   terraform init
   ```
3. Set the provider variable:
   ```bash
   # For AWS deployment
   terraform apply -var="provider=aws"
   
   # For Azure deployment
   terraform apply -var="provider=azure"
   ```

## Configuration

### Variables

| Variable | Description | Type | Default |
|----------|-------------|------|---------|
| `provider` | Cloud provider to use (`aws` or `azure`) | string | `"aws"` |

### Provider Configuration

The script automatically configures the appropriate provider based on the `provider` variable:

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

- **EC2 Instance**: `t2.micro` instance with Amazon Linux 2
- **User Data Script**: Installs and starts Apache HTTP server

```hcl
resource "aws_instance" "aws_ec2_instance" {
  count           = var.provider == "aws" ? 1 : 0
  ami             = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI
  instance_type   = "t2.micro"
  key_name        = "your-key-pair"  # Replace with your key pair name

  tags = {
    Name = "aws-ec2-instance"
  }

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              EOF
}
```

## Azure Resources

When `provider = "azure"`, the following resources are created:

- **Resource Group**: Container for all Azure resources
- **Virtual Network**: Network infrastructure with subnet
- **Network Interface**: VM network interface
- **Virtual Machine**: Ubuntu 18.04 LTS VM

```hcl
resource "azurerm_resource_group" "azure_resource_group" {
  count    = var.provider == "azure" ? 1 : 0
  name     = "azure-resource-group"
  location = "West US"
}

resource "azurerm_virtual_network" "azure_vnet" {
  count               = var.provider == "azure" ? 1 : 0
  name                = "azure-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.azure_resource_group[0].location
  resource_group_name = azurerm_resource_group.azure_resource_group[0].name
}

resource "azurerm_subnet" "azure_subnet" {
  count                = var.provider == "azure" ? 1 : 0
  name                 = "azure-subnet"
  resource_group_name  = azurerm_resource_group.azure_resource_group[0].name
  virtual_network_name = azurerm_virtual_network.azure_vnet[0].name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "azure_nic" {
  count               = var.provider == "azure" ? 1 : 0
  name                = "azure-nic"
  location            = azurerm_resource_group.azure_resource_group[0].location
  resource_group_name = azurerm_resource_group.azure_resource_group[0].name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.azure_subnet[0].id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_virtual_machine" "azure_vm" {
  count                 = var.provider == "azure" ? 1 : 0
  name                  = "azure-vm"
  location              = azurerm_resource_group.azure_resource_group[0].location
  resource_group_name   = azurerm_resource_group.azure_resource_group[0].name
  network_interface_ids = [azurerm_network_interface.azure_nic[0].id]
  vm_size               = "Standard_B1s"

  storage_os_disk {
    name              = "osdisk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  os_profile {
    computer_name  = "hostname"
    admin_username = "adminuser"
    admin_password = "Password1234!"
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }

  tags = {
    Name = "azure-vm"
  }
}
```

## Outputs

The script provides different outputs based on the selected provider:

### AWS Outputs
```hcl
output "aws_instance_id" {
  value = var.provider == "aws" ? aws_instance.aws_ec2_instance[0].id : null
}

output "aws_public_ip" {
  value = var.provider == "aws" ? aws_instance.aws_ec2_instance[0].public_ip : null
}
```

### Azure Outputs
```hcl
output "azure_vm_id" {
  value = var.provider == "azure" ? azurerm_virtual_machine.azure_vm[0].id : null
}

output "azure_private_ip" {
  value = var.provider == "azure" ? azurerm_network_interface.azure_nic[0].private_ip_address : null
}
```

## Important Notes

⚠️ **Before deploying:**
- Replace `"your-key-pair"` with your actual AWS key pair name
- Ensure you have appropriate permissions for resource creation
- Review and adjust VM sizes, regions, and other configurations as needed
- Consider using Terraform variables file (`.tfvars`) for sensitive data

🔒 **Security Considerations:**
- The Azure VM uses password authentication - consider using SSH keys instead
- Review security groups and network access rules
- Use strong, unique passwords and store them securely

## Clean Up

To destroy the created resources:

```bash
terraform destroy -var="provider=aws"  # For AWS resources
terraform destroy -var="provider=azure"  # For Azure resources
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request
