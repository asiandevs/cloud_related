# Azure Virtual Machine Core Service Design Template

## Document Control

| Item | Details |
|------|---------|
| **Title** | [Customer Name] - Azure Virtual Machine Core Service Design |
| **File Name** | [Customer Name]-Azure-VM-Core-Service-Design-v1.0.docx |
| **Version** | 1.0 |
| **Status** | [Draft/Review/Released] |
| **Release Date** | [DD/MM/YYYY] |
| **Prepared By** | [Name, Role] |
| **Authorized By** | [Name, Role] |

## Version History

| Version | Remarks | Change Requested | Pages Affected | Release Date |
|---------|---------|------------------|----------------|--------------|
| 1.0 | Initial Release | N/A | All | [DD/MM/YYYY] |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Executive Summary](#2-executive-summary)
3. [Resource Cost Estimation](#3-resource-cost-estimation)
4. [WAF and Security Control Alignment](#4-waf-and-security-control-alignment)
5. [Architecture Summary](#5-architecture-summary)
6. [Azure Policies](#6-azure-policies)
7. [Configuration Templates](#7-configuration-templates)
8. [Acceptance](#8-acceptance)

---

## 1. Overview

### 1.1 Purpose and Audience

This document outlines the standard design and configuration for Azure Virtual Machines (IaaS) as a core service within **[Customer Name]**'s Azure tenancy. It serves as a baseline for virtual machine deployments across all environments.

**Intended Audience:**
- Cloud Architects
- Infrastructure Engineers
- System Administrators
- DevOps Teams
- IT Management
- Security & Compliance Teams

**Prerequisites:**
- Familiarity with Azure Cloud concepts
- Understanding of virtualization and compute resources
- Knowledge of Windows and Linux operating systems
- Basic understanding of networking and storage

### 1.2 Scope and Key Deliverables

**Scope:**
- Define baseline deployment requirements for Azure Virtual Machines
- Establish standards aligned with Microsoft Well-Architected Framework (WAF)
- Provide compliance with [Customer Security Framework/Standards]
- Enable repeatable, consistent VM deployments
- Cover Windows and Linux workloads

**Out of Scope:**
- Specific application configurations
- Third-party software licensing details
- Detailed disaster recovery procedures (covered in separate DR design)
- Azure Virtual Machine Scale Sets (covered in separate design)

**Key Deliverables:**
1. Level 2 baseline standards documentation
2. Technical configuration templates for different VM tiers
3. Infrastructure as Code (IaC) templates
4. RBAC and governance guidelines
5. Patching and update management strategy
6. Backup and recovery standards

### 1.3 Glossary and Definitions

| Term | Definition |
|------|------------|
| **VM** | Virtual Machine |
| **WAF** | Well-Architected Framework - Microsoft's architectural best practices |
| **CAF** | Cloud Adoption Framework |
| **IaaS** | Infrastructure as a Service |
| **Level 1** | Resource designed to CAF standards |
| **Level 2** | Resource designed to WAF standards with organizational security controls |
| **SLA** | Service Level Agreement as defined by Microsoft |
| **IaC** | Infrastructure as Code |
| **NSG** | Network Security Groups |
| **OS** | Operating System |
| **vCPU** | Virtual Central Processing Unit |
| **IOPS** | Input/Output Operations Per Second |
| **RTO** | Recovery Time Objective |
| **RPO** | Recovery Point Objective |
| **ASR** | Azure Site Recovery |
| **ADE** | Azure Disk Encryption |
| **RBAC** | Role-Based Access Control |
| **MMA** | Microsoft Monitoring Agent (Legacy) |
| **AMA** | Azure Monitor Agent |
| **JIT** | Just-In-Time (access) |
| **TDE** | Transparent Data Encryption |
| **Spot VM** | Low-priority VM instance at reduced cost |

---

## 2. Executive Summary

This design establishes baseline standards for Azure Virtual Machines as a core service, assessed against the five pillars of Microsoft's Well-Architected Framework and [Customer Security Requirements].

### Key Design Decisions

**Virtual Machine Configuration:**
- Standardized VM SKU families per workload type
- Availability Sets or Availability Zones for high availability
- Managed Disks with appropriate performance tiers
- Azure Backup for all production VMs
- Azure Monitor Agent for monitoring and logging
- Azure Update Management for patching
- Disk encryption enabled by default

**Operating System Standards:**

| OS Platform | Supported Versions | Image Source |
|-------------|-------------------|--------------|
| **Windows Server** | 2019, 2022 | Azure Marketplace / Custom |
| **Linux** | RHEL 8/9, Ubuntu 20.04/22.04, SLES 15 | Azure Marketplace / Custom |

**Environment Differentiation:**

| Feature | Production | Non-Production |
|---------|-----------|----------------|
| Availability | Availability Zones (3 zones) | Single zone or no zone |
| Backup | Daily, 30-day retention | Weekly, 14-day retention |
| Monitoring | Full diagnostics + alerts | Basic diagnostics |
| Disk Tier | Premium SSD or Ultra Disk | Standard SSD |
| VM SKU | Production-grade | Cost-optimized |
| Patching Window | Scheduled maintenance window | Flexible |
| Snapshot Frequency | Pre/post change | As needed |

**Security Baseline:**
- Azure Disk Encryption mandatory for all VMs
- Just-In-Time VM access for management
- Network Security Groups with restrictive rules
- Azure Security Center/Defender for Cloud enabled
- Vulnerability assessment enabled
- Anti-malware extensions deployed
- No public IP addresses except for specific approved scenarios

**Naming Convention:**
`vm-[env]-[region]-[os]-[workload]-[instance]`

Example: `vm-prd-aue-win-sqldb-01`

---

## 3. Resource Cost Estimation

### Azure Virtual Machine Pricing Components

Azure VM costs consist of multiple components:

1. **Compute** - VM size (vCPUs, RAM)
2. **Storage** - Managed disks (OS and data)
3. **Networking** - Bandwidth, Load Balancers, Public IPs
4. **Licensing** - Windows Server, SQL Server (if applicable)
5. **Backup** - Azure Backup storage
6. **Monitoring** - Log Analytics data ingestion

### Sample Pricing (Pay-as-you-go)

**Windows Server VMs:**

| VM Size | vCPU | RAM | Typical Use Case | Est. Monthly Cost* |
|---------|------|-----|------------------|-------------------|
| **D2s_v5** | 2 | 8 GB | Small web server | $[XXX] |
| **D4s_v5** | 4 | 16 GB | Application server | $[XXX] |
| **D8s_v5** | 8 | 32 GB | Database server | $[XXX] |
| **E16s_v5** | 16 | 128 GB | Memory-intensive apps | $[XXX] |

**Linux VMs:**

| VM Size | vCPU | RAM | Typical Use Case | Est. Monthly Cost* |
|---------|------|-----|------------------|-------------------|
| **D2s_v5** | 2 | 8 GB | Small web server | $[XXX] |
| **D4s_v5** | 4 | 16 GB | Application server | $[XXX] |
| **D8s_v5** | 8 | 32 GB | Database server | $[XXX] |
| **E16s_v5** | 16 | 128 GB | Memory-intensive apps | $[XXX] |

*Note: Costs vary by region. Linux VMs are ~40% cheaper than Windows due to licensing.

**Managed Disk Pricing:**

| Disk Type | Size | IOPS | Throughput | Est. Monthly Cost |
|-----------|------|------|------------|-------------------|
| **Standard HDD** | 128 GB | 500 | 60 MB/s | $[XX] |
| **Standard SSD** | 128 GB | 500 | 60 MB/s | $[XX] |
| **Premium SSD** | 128 GB | 500 | 100 MB/s | $[XX] |
| **Premium SSD** | 512 GB | 2,300 | 150 MB/s | $[XXX] |
| **Ultra Disk** | 512 GB | 10,000+ | 400+ MB/s | $[XXX+] |

### Cost Optimization Strategies

**Reserved Instances:**
- 1-year commitment: ~40% savings
- 3-year commitment: ~60% savings
- Recommended for steady-state production workloads

**Azure Hybrid Benefit:**
- Use existing Windows Server licenses
- Save up to 40% on Windows VMs
- Requires Software Assurance

**VM Sizing:**
- Right-size VMs based on actual utilization
- Review Azure Advisor recommendations monthly
- Start smaller and scale up as needed

**Startup/Shutdown Schedules:**
- Automatically stop non-production VMs after hours
- Potential savings: 60-70% for dev/test environments

**Spot VMs:**
- Use for fault-tolerant workloads
- Savings: up to 90% compared to pay-as-you-go
- Not recommended for production

**Example Monthly Cost Scenario:**

Production Application Server (Windows):
- VM: D4s_v5 (4 vCPU, 16 GB RAM): $[XXX]
- OS Disk: 128 GB Premium SSD: $[XX]
- Data Disk: 512 GB Premium SSD: $[XXX]
- Backup (30-day retention): $[XX]
- Monitoring (Log Analytics): $[XX]
- Networking: $[XX]
- **Total: $[XXX] per month**

With 3-year Reserved Instance + Hybrid Benefit:
- **Optimized: $[XXX] per month (XX% savings)**

---

## 4. WAF and Security Control Alignment

### Microsoft Well-Architected Framework Pillars

1. **Reliability** - System availability and failure recovery
2. **Cost Optimization** - Eliminate waste, optimize spending
3. **Operational Excellence** - Deployment automation and monitoring
4. **Performance Efficiency** - Scalability and resource optimization
5. **Security** - Threat resistance and data protection

### 4.1 Reliability

**Principles:**
- Design for high availability
- Implement redundancy and failover
- Plan for disaster recovery
- Test backup and restore procedures
- Monitor VM health continuously

#### Reliability Checklist

| ID | Requirement | Applicable | Enforcement | Implementation Phase |
|----|-------------|-----------|-------------|---------------------|
| **R1** | Use Availability Zones for production VMs | Yes | IaC Template | Design |
| **R2** | Deploy VMs in Availability Sets (if zones unavailable) | Yes | IaC Template | Design |
| **R3** | Implement Azure Backup for all production VMs | Yes | Azure Policy | Deployment |
| **R4** | Configure backup retention per data classification | Yes | Governance | Design |
| **R5** | Use Managed Disks (not unmanaged) | Yes | Azure Policy | Deployment |
| **R6** | Test backup restore procedures quarterly | Yes | Governance | Operational |
| **R7** | Configure VM auto-shutdown for non-production | Yes | IaC Template | Deployment |
| **R8** | Implement Azure Site Recovery for critical VMs | Conditional | Governance | Design |
| **R9** | Use health probes with Load Balancers | Yes | IaC Template | Design |
| **R10** | Document RTO and RPO requirements | Yes | Governance | Design |

**Implementation Notes:**

**Availability Zones:**
- Provide datacenter-level fault tolerance
- Minimum 3 VMs across 3 zones for 99.99% SLA
- Zone-redundant storage recommended
- Consider latency between zones

**Availability Sets:**
- Use when Availability Zones not available
- Update domains: 5-20 (default: 5)
- Fault domains: 2-3 (default: 3)
- 99.95% SLA with 2+ VMs

**Backup Configuration:**

| Environment | Frequency | Retention | Recovery Point |
|-------------|-----------|-----------|----------------|
| **Production** | Daily | 30 days | < 24 hours |
| **UAT/Staging** | Daily | 14 days | < 24 hours |
| **Development** | Weekly | 7 days | < 7 days |

**SLA Targets:**

| Configuration | SLA | Monthly Downtime |
|---------------|-----|------------------|
| Single VM (Premium SSD) | 99.9% | ~43 minutes |
| Availability Set | 99.95% | ~22 minutes |
| Availability Zones | 99.99% | ~4 minutes |

### 4.2 Cost Optimization

**Principles:**
- Right-size VM instances
- Use Reserved Instances for steady workloads
- Implement auto-shutdown for non-production
- Leverage Azure Hybrid Benefit
- Monitor and optimize continuously

#### Cost Optimization Checklist

| ID | Requirement | Applicable | Enforcement | Implementation Phase |
|----|-------------|-----------|-------------|---------------------|
| **CO1** | Right-size VMs based on utilization metrics | Yes | Governance | Operational |
| **CO2** | Review Azure Advisor cost recommendations | Yes | Governance | Operational |
| **CO3** | Implement auto-shutdown for non-production VMs | Yes | IaC Template | Deployment |
| **CO4** | Use Reserved Instances for production workloads | Yes | Governance | Planning |
| **CO5** | Apply Azure Hybrid Benefit where applicable | Yes | Governance | Deployment |
| **CO6** | Use Standard SSD for non-production workloads | Yes | IaC Template | Design |
| **CO7** | Remove orphaned disks and snapshots | Yes | Governance | Operational |
| **CO8** | Consolidate underutilized VMs | Yes | Governance | Operational |
| **CO9** | Use B-series for low CPU workloads | Conditional | Governance | Design |
| **CO10** | Implement tagging strategy for cost allocation | Yes | Azure Policy | Deployment |

**Implementation Notes:**

**VM Sizing Best Practices:**
- Start with smaller sizes and scale up
- Monitor CPU, memory, disk, and network utilization
- Resize during maintenance windows
- Use burst-capable SKUs (B-series) for variable workloads

**Auto-Shutdown Configuration:**
```
Development: Shutdown at 7 PM, start at 7 AM weekdays
Test: Shutdown at 8 PM, start at 6 AM weekdays
UAT: No automatic shutdown (manual control)
Production: No automatic shutdown
```

**Tagging Strategy:**
| Tag Name | Purpose | Example Values |
|----------|---------|----------------|
| Environment | Workload classification | Production, Development, Test |
| CostCenter | Billing allocation | IT-001, Finance-002 |
| Owner | Technical contact | john.smith@company.com |
| Application | Application name | PayrollSystem, CRM |
| Criticality | Business impact | Critical, High, Medium, Low |

### 4.3 Operational Excellence

**Principles:**
- Automate deployment and configuration
- Implement consistent patch management
- Monitor performance and availability
- Use Infrastructure as Code
- Document operational procedures

#### Operational Excellence Checklist

| ID | Requirement | Applicable | Enforcement | Implementation Phase |
|----|-------------|-----------|-------------|---------------------|
| **OE1** | Deploy VMs using IaC templates | Yes | Governance | Deployment |
| **OE2** | Enable Azure Monitor Agent on all VMs | Yes | Azure Policy | Deployment |
| **OE3** | Configure diagnostic settings | Yes | IaC Template | Deployment |
| **OE4** | Implement Azure Update Management | Yes | Azure Policy | Deployment |
| **OE5** | Define maintenance windows for patching | Yes | Governance | Design |
| **OE6** | Use Azure Automation for operational tasks | Yes | IaC Template | Deployment |
| **OE7** | Configure VM alerts (CPU, memory, disk) | Yes | IaC Template | Deployment |
| **OE8** | Implement change management procedures | Yes | Governance | Operational |
| **OE9** | Document VM build specifications | Yes | Governance | Design |
| **OE10** | Use custom VM images for standard builds | Yes | Governance | Deployment |
| **OE11** | Enable boot diagnostics | Yes | IaC Template | Deployment |
| **OE12** | Configure VM extensions (antimalware, monitoring) | Yes | IaC Template | Deployment |

**Implementation Notes:**

**Patch Management Schedule:**

| Environment | Patch Window | Frequency | Reboot Allowed |
|-------------|--------------|-----------|----------------|
| **Production** | Saturday 2-6 AM | Monthly | Yes (planned) |
| **UAT/Staging** | Thursday 8-11 PM | Weekly | Yes |
| **Development** | Anytime | Weekly | Yes |

**Monitoring Configuration:**

| Metric | Threshold | Alert Severity |
|--------|-----------|----------------|
| CPU > 85% | 15 minutes | Warning |
| CPU > 95% | 5 minutes | Critical |
| Memory > 90% | 15 minutes | Warning |
| Disk space < 10% | N/A | Critical |
| Disk IOPS > 90% | 10 minutes | Warning |

**VM Extensions (Standard Deployment):**
1. Azure Monitor Agent
2. Microsoft Antimalware (Windows) / Azure Security (Linux)
3. Dependency Agent (for VM Insights)
4. Custom Script Extension (for post-deployment config)
5. Azure Disk Encryption extension

### 4.4 Performance Efficiency

**Principles:**
- Select appropriate VM sizes for workload
- Use Premium storage for I/O intensive workloads
- Implement caching strategies
- Monitor performance metrics
- Scale vertically or horizontally as needed

#### Performance Efficiency Checklist

| ID | Requirement | Applicable | Enforcement | Implementation Phase |
|----|-------------|-----------|-------------|---------------------|
| **PE1** | Select VM size based on workload requirements | Yes | Governance | Design |
| **PE2** | Use Premium SSD for production databases | Yes | IaC Template | Design |
| **PE3** | Configure host caching appropriately | Yes | IaC Template | Deployment |
| **PE4** | Separate OS and data disks | Yes | IaC Template | Design |
| **PE5** | Use accelerated networking where available | Yes | IaC Template | Deployment |
| **PE6** | Monitor VM performance metrics | Yes | IaC Template | Deployment |
| **PE7** | Conduct performance testing pre-production | Yes | Governance | Testing |
| **PE8** | Plan for vertical/horizontal scaling | Yes | Governance | Design |
| **PE9** | Use proximity placement groups for low latency | Conditional | IaC Template | Design |
| **PE10** | Consider VM families optimized for workload | Yes | Governance | Design |

**Implementation Notes:**

**VM Family Selection Guide:**

| VM Series | Use Case | Characteristics |
|-----------|----------|-----------------|
| **B-series** | Dev/test, low-traffic web | Burstable, cost-effective |
| **D-series** | General purpose | Balanced CPU-to-memory |
| **E-series** | Memory-intensive | High memory-to-CPU ratio |
| **F-series** | Compute-intensive | High CPU-to-memory ratio |
| **L-series** | Storage-intensive | High disk throughput |
| **M-series** | Large in-memory | Up to 4 TB RAM |
| **N-series** | GPU workloads | Graphics rendering, AI/ML |

**Disk Performance Tiers:**

| Disk Type | Max IOPS | Max Throughput | Use Case |
|-----------|----------|----------------|----------|
| **Standard HDD** | 500 | 60 MB/s | Archive, backup |
| **Standard SSD** | 6,000 | 750 MB/s | Dev/test, web servers |
| **Premium SSD** | 20,000 | 900 MB/s | Production databases |
| **Ultra Disk** | 160,000 | 4,000 MB/s | SAP HANA, top-tier DBs |

**Host Caching Recommendations:**

| Disk Type | Workload | Cache Setting |
|-----------|----------|---------------|
| OS Disk | All | Read/Write |
| Data Disk | Read-heavy | Read-only |
| Data Disk | Write-heavy | None |
| Data Disk | Database transaction logs | None |
| Data Disk | Database data files | Read-only |

**Accelerated Networking:**
- Reduces latency by up to 30%
- Enables SR-IOV to VM
- Available on most VM sizes (2+ vCPUs)
- Enable by default for production VMs

### 4.5 Security

**Principles:**
- Implement defense-in-depth
- Encrypt data at rest and in transit
- Use identity-based access control
- Monitor for threats and vulnerabilities
- Apply least privilege principle
- Keep systems patched and updated

#### Security Control Alignment

**Microsoft Security Benchmark:**

| Control ID | Requirement | Implementation |
|------------|-------------|----------------|
| **NS-1** | Establish network segmentation | NSGs, VNet subnets |
| **NS-2** | Secure cloud services with network controls | Private endpoints, service endpoints |
| **NS-5** | Deploy intrusion detection/prevention | Azure Firewall, NSG flow logs |
| **IM-1** | Use centralized identity and authentication | Azure AD, managed identities |
| **IM-3** | Manage application identities securely | Managed identities for Azure resources |
| **IM-8** | Restrict credential exposure | No passwords in code/config |
| **PA-3** | Manage and review access regularly | JIT VM access, RBAC reviews |
| **PA-7** | Use just-in-time access | JIT VM access enabled |
| **DP-1** | Discover and classify data | Data classification tags |
| **DP-2** | Monitor anomalies and threats | Microsoft Defender for Cloud |
| **DP-3** | Encrypt sensitive data in transit | TLS 1.2+, IPSec |
| **DP-4** | Encrypt sensitive data at rest | Azure Disk Encryption |
| **DP-5** | Use customer-managed keys | Optional for compliance |
| **LT-1** | Enable threat detection | Microsoft Defender for Cloud |
| **LT-4** | Enable logging for investigation | Diagnostic settings, Log Analytics |
| **PV-6** | Rapidly and automatically remediate vulnerabilities | Update Management, Qualys |
| **ES-1** | Use endpoint detection and response | Microsoft Defender for Endpoint |
| **ES-2** | Use modern anti-malware software | Azure Security extension |

#### Security Checklist

| ID | Requirement | Applicable | Enforcement | Implementation Phase |
|----|-------------|-----------|-------------|---------------------|
| **S1** | Enable Azure Disk Encryption on all VMs | Yes | Azure Policy | Deployment |
| **S2** | Use Azure Key Vault for encryption keys | Yes | IaC Template | Deployment |
| **S3** | Deploy VMs in private subnets (no public IPs) | Yes | Azure Policy | Design |
| **S4** | Configure NSGs with restrictive rules | Yes | IaC Template | Deployment |
| **S5** | Enable Just-In-Time VM access | Yes | Azure Policy | Deployment |
| **S6** | Deploy antimalware extension | Yes | Azure Policy | Deployment |
| **S7** | Enable Microsoft Defender for Cloud | Yes | Subscription-level | Planning |
| **S8** | Configure vulnerability assessment | Yes | Azure Policy | Deployment |
| **S9** | Disable unnecessary services and ports | Yes | Governance | Build |
| **S10** | Implement strong password policies | Yes | Governance | Build |
| **S11** | Enable MFA for administrative access | Yes | Azure AD Policy | Planning |
| **S12** | Use managed identities instead of credentials | Yes | Governance | Development |
| **S13** | Enable Azure Security Baseline | Yes | Azure Policy | Deployment |
| **S14** | Configure audit logging to Log Analytics | Yes | IaC Template | Deployment |
| **S15** | Implement file integrity monitoring | Yes | Governance | Deployment |

**Implementation Notes:**

**Azure Disk Encryption:**
- Uses BitLocker (Windows) or DM-Crypt (Linux)
- Keys stored in Azure Key Vault
- Mandatory for all production VMs
- Enable before storing sensitive data

**Network Security:**

```
Production NSG Rules (Inbound):
- Allow 443 from Load Balancer
- Allow 1433 from App tier subnet (SQL Server)
- Allow 22/3389 from Bastion subnet only
- Deny all other traffic

Development NSG Rules (Inbound):
- Allow 22/3389 from corporate VPN
- Allow application ports from same VNet
- Deny all other traffic
```

**Just-In-Time Access:**
- Maximum session time: 3 hours
- Request approval required for production
- Audit all JIT sessions
- Automatic revocation after session

**Antimalware Configuration:**
- Real-time protection enabled
- Scheduled scans: Daily at 2 AM
- Quarantine suspicious files
- Alert on malware detection
- Auto-update signatures

**[Customer Security Framework] Controls:**
- *[Map customer-specific security controls here]*
- *[Include compliance requirements: ISO 27001, PCI-DSS, HIPAA, etc.]*
- *[Document any exceptions or deviations]*

---

## 5. Architecture Summary

### 5.1 Resource Overview

Azure Virtual Machines provide on-demand, scalable computing resources with Infrastructure-as-a-Service (IaaS) capabilities. VMs support both Windows and Linux operating systems and can run a wide variety of workloads.

**Key Components:**
- **Compute** - vCPU and RAM resources
- **Storage** - Managed disks (OS and data)
- **Networking** - Virtual Network integration, NICs
- **Extensions** - Additional capabilities (monitoring, security)
- **Availability** - Zones, Sets, or standalone
- **Management** - Backup, monitoring, patching

**Typical Architecture Pattern:**

```
Internet/On-Premises
        ↓
   Azure Firewall / Gateway
        ↓
   Application Gateway / Load Balancer
        ↓
   Web Tier VMs (Availability Zone 1, 2, 3)
        ↓
   Application Tier VMs (Availability Zone 1, 2, 3)
        ↓
   Database Tier VMs (Availability Zone 1, 2, 3)
        ↓
   Backup to Azure Backup / ASR
```

### 5.2 VM Sizing and SKU Selection

#### General Guidelines

**Assessment Criteria:**
1. **Workload Type** - Web, app, database, batch processing
2. **CPU Requirements** - Processing power needed
3. **Memory Requirements** - RAM footprint
4. **Storage IOPS** - I/O operations per second
5. **Network Bandwidth** - Data transfer needs
6. **GPU Requirements** - Graphics or AI/ML workloads

#### Standard VM Sizes by Workload

**Web Servers (IIS, Apache, Nginx):**

| Environment | VM Size | vCPU | RAM | Use Case |
|-------------|---------|------|-----|----------|
| Production | D4s_v5 | 4 | 16 GB | Standard web traffic |
| Production | D8s_v5 | 8 | 32 GB | High-traffic sites |
| Non-Prod | D2s_v5 | 2 | 8 GB | Dev/test environments |

**Application Servers (Java, .NET, Node.js):**

| Environment | VM Size | vCPU | RAM | Use Case |
|-------------|---------|------|-----|----------|
| Production | D8s_v5 | 8 | 32 GB | Standard app workloads |
| Production | E16s_v5 | 16 | 128 GB | Memory-intensive apps |
| Non-Prod | D4s_v5 | 4 | 16 GB | Dev/test environments |

**Database Servers (SQL Server, Oracle, PostgreSQL):**

| Environment | VM Size | vCPU | RAM | Use Case |
|-------------|---------|------|-----|----------|
| Production | E8s_v5 | 8 | 64 GB | Small to medium databases |
| Production | E16s_v5 | 16 | 128 GB | Large databases |
| Production | M64s | 64 | 1 TB | Enterprise databases |
| Non-Prod | E4s_v5 | 4 | 32 GB | Dev/test databases |

**File Servers:**

| Environment | VM Size | vCPU | RAM | Use Case |
|-------------|---------|------|-----|----------|
| Production | D4s_v5 | 4 | 16 GB | General file shares |
| Production | Ls8_v3 | 8 | 64 GB | High-throughput storage |

### 5.3 Storage Configuration

#### Disk Architecture Standards

**OS Disk:**
- Size: 127 GB minimum (Windows), 30 GB minimum (Linux)
- Type: Premium SSD (production), Standard SSD (non-prod)
- Caching: Read/Write
- Encryption: Enabled

**Data Disks:**
- Separate from OS disk
- Size: Based on workload requirements
- Type: Premium SSD or Ultra Disk (production)
- Caching: Workload-dependent
- Encryption: Enabled

**Temporary Disk:**
- Ephemeral storage (lost on VM stop/deallocate)
- Not suitable for persistent data
- Use for page files, temp files, caching only

#### Disk Configuration by Workload

**Web Server:**
```
OS Disk: 127 GB Premium SSD (Read/Write cache)
Data Disk 1: 128 GB Premium SSD (application files)
Temp Disk: Default (page file)
```

**Application Server:**
```
OS Disk: 127 GB Premium SSD (Read/Write cache)
Data Disk 1: 256 GB Premium SSD (application binaries)
Data Disk 2: 128 GB Premium SSD (application logs)
Temp Disk: Default (page file, temp files)
```

**Database Server (SQL Server):**
```
OS Disk: 127 GB Premium SSD (Read/Write cache)
Data Disk 1: 1 TB Premium SSD (database data files, Read-only cache)
Data Disk 2: 512 GB Premium SSD (database log files, No cache)
Data Disk 3: 256 GB Premium SSD (TempDB on SSD, Read/Write cache)
Temp Disk: Not used for SQL Server
```

### 5.4 Networking Configuration

#### Network Interface Cards (NICs)

**Standard Configuration:**
- Primary NIC: Management and application traffic
- Secondary NIC (optional): Backup or replication traffic
- Accelerated networking: Enabled for production VMs
- Private IP allocation: Static (production), Dynamic (non-prod)

**IP Address Standards:**

| Environment | Allocation | Assignment |
|-------------|-----------|------------|
| Production | Static | Documented in IPAM |
| UAT | Static | Documented in IPAM |
| Development | Dynamic | DHCP |

#### Network Security Groups (NSGs)

**NSG Assignment:**
- Subnet-level NSGs (primary security boundary)
- NIC-level NSGs (additional security for specific VMs)

**Standard NSG Rules:**

**Inbound Rules (Production Web Server):**
| Priority | Name | Source | Destination | Port | Action |
|----------|------|--------|-------------|------|--------|
| 100 | Allow-HTTPS-Inbound | Internet | * | 443 | Allow |
| 110 | Allow-HTTP-Inbound | Internet | * | 80 | Allow |
| 200 | Allow-Bastion-RDP | Bastion-Subnet | * | 3389 | Allow |
| 210 | Allow-Bastion-SSH | Bastion-Subnet | * | 22 | Allow |
| 300 | Allow-LB-Health | AzureLoadBalancer | * | * | Allow |
| 4096 | Deny-All-Inbound | * | * | * | Deny |

**Outbound Rules (Standard):**
| Priority | Name | Source | Destination | Port | Action |
|----------|------|--------|-------------|------|--------|
| 100 | Allow-Azure-Services | * | AzureCloud | * | Allow |
| 110 | Allow-Internet | * | Internet | * | Allow |
| 200 | Allow-VNet | * | VirtualNetwork | * | Allow |

### 5.5 High Availability Configuration

#### Availability Options

**Option 1: Availability Zones (Preferred)**
- Deploy VMs across 3 zones in the same region
- Provides datacenter-level fault tolerance
- 99.99% SLA with 2+ VMs
- Zone-redundant load balancer required

**Configuration Example:**
```
Web Tier: 3 VMs (1 per zone) + Zone-redundant Load Balancer
App Tier: 3 VMs (1 per zone) + Zone-redundant Load Balancer
DB Tier: 3 VMs (1 per zone) + SQL Always On Availability Groups
```

**Option 2: Availability Sets**
- Use when zones not available in region
- 99.95% SLA with 2+ VMs
- Update domains: 5 (default)
- Fault domains: 2 or 3

**Configuration Example:**
```
Web Tier: Availability Set with 2+ VMs
App Tier: Availability Set with 2+ VMs
DB Tier: Availability Set with 2+ VMs + SQL clustering
```

**Option 3: Single Instance**
- 99.9% SLA with Premium SSD
- Not recommended for production
- Acceptable for non-critical workloads

#### Load Balancing

**Azure Load Balancer (Layer 4):**
- Use for: Non-HTTP(S) traffic
- Configuration: Zone-redundant (production)
- Health probe: TCP or HTTP
- Session persistence: Configure as needed

**Azure Application Gateway (Layer 7):**
- Use for: HTTP(S) web applications
- Features: WAF, SSL termination, URL routing
- See Application Gateway design document

### 5.6 RBAC Configuration

#### Built-in Azure Roles for VMs

| Role Name | Description | Typical Assignment |
|-----------|-------------|-------------------|
| **Virtual Machine Contributor** | Manage VMs but not network/storage | VM administrators |
| **Virtual Machine Administrator Login** | Login to VMs with admin privileges | Windows/Linux administrators |
| **Virtual Machine User Login** | Login to VMs with user privileges | Application support teams |
| **Reader** | View resources only | Monitoring teams, auditors |
| **Contributor** | Manage all resources except access | Senior engineers (limited scope) |

#### Custom Role Examples

**VM Operator (Limited Management):**
```json
{
  "Name": "[Customer Name] - VM Operator",
  "Description": "Start, stop, restart VMs only",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/deallocate/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/[subscription-id]"
  ]
}
```

**Backup Operator:**
```json
{
  "Name": "[Customer Name] - Backup Operator",
  "Description": "Manage VM backups",
  "Actions": [
    "Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/backup/action",
    "Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read",
    "Microsoft.RecoveryServices/Vaults/backupPolicies/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/[subscription-id]"
  ]
}
```

### 5.7 Design Decisions and Justifications

#### 5.7.1 Managed Disks vs Unmanaged Disks

**Decision:** Use Managed Disks exclusively

**Justification:**
- Simplified management (no storage account management)
- Better scalability (20,000 disks per subscription vs 40 per storage account)
- Improved reliability (automatic storage redundancy)
- Availability Set integration
- Snapshot and image support
- Encryption integration
- Better performance and IOPS limits

**Policy Enforcement:** Deny creation of VMs with unmanaged disks

#### 5.7.2 Availability Zones vs Availability Sets

**Decision:** Use Availability Zones for production workloads where available

**Justification:**
- Higher SLA (99.99% vs 99.95%)
- Better fault isolation (datacenter-level vs rack-level)
- Support for zone-redundant services
- Future-proof architecture
- Better disaster recovery posture

**Fallback:** Use Availability Sets in regions without zone support

#### 5.7.3 Azure Backup vs Third-Party Solutions

**Decision:** Use Azure Backup as the primary backup solution

**Justification:**
- Native integration with Azure VMs
- No backup infrastructure to manage
- Cost-effective for Azure workloads
- Application-consistent backups
- Long-term retention support
- Instant restore capabilities
- Policy-based protection

**Exception:** Third-party solutions allowed for specific compliance requirements

#### 5.7.4 Azure Monitor Agent vs Legacy MMA

**Decision:** Deploy Azure Monitor Agent (AMA) on all new VMs

**Justification:**
- MMA is deprecated (August 2024)
- Better performance and reliability
- Simplified configuration via DCR (Data Collection Rules)
- Multi-homing support
- Better security (managed identity support)
- Enhanced filtering and transformation

**Migration Plan:** Migrate existing MMA deployments to AMA by [Target Date]

#### 5.7.5 Public IP Addresses

**Decision:** No public IP addresses on VMs except for approved scenarios

**Justification:**
- Reduced attack surface
- Better security posture
- Compliance with zero-trust principles
- Force traffic through security controls
- Simplified NSG management

**Approved Scenarios:**
- Bastion/Jump hosts (with JIT access)
- VPN gateways
- NAT gateways
- Specific approved use cases with governance approval

#### 5.7.6 VM Extensions

**Decision:** Deploy standard extensions via IaC for all VMs

**Justification:**
- Consistent configuration
- Automated deployment
- Better compliance
- Simplified management
- Reduced manual errors

**Standard Extensions:**

| Extension | Windows | Linux | Purpose |
|-----------|---------|-------|---------|
| Azure Monitor Agent | ✓ | ✓ | Monitoring and logging |
| Dependency Agent | ✓ | ✓ | VM Insights, service maps |
| Microsoft Antimalware | ✓ | - | Malware protection |
| Azure Security | - | ✓ | Malware protection |
| Azure Disk Encryption | ✓ | ✓ | Disk encryption |
| Custom Script Extension | ✓ | ✓ | Post-deployment configuration |

#### 5.7.7 Update Management

**Decision:** Use Azure Update Management (Azure Automation) for patching

**Justification:**
- Centralized patch management
- Scheduled maintenance windows
- Pre and post-patch scripts
- Compliance reporting
- Cross-platform support (Windows/Linux)
- Integration with Log Analytics

**Alternative (Preview):** Azure Update Manager for newer deployments

#### 5.7.8 Boot Diagnostics

**Decision:** Enable boot diagnostics on all VMs

**Justification:**
- Troubleshoot boot failures
- View console output and screenshots
- No additional cost (managed storage)
- Essential for incident response

**Configuration:** Store in managed storage account (default)

#### 5.7.9 Accelerated Networking

**Decision:** Enable accelerated networking for production VMs (2+ vCPUs)

**Justification:**
- Lower latency (up to 30% improvement)
- Higher packets per second
- Lower CPU utilization for networking
- Better overall performance
- No additional cost

**Supported:** Most VM sizes with 2+ vCPUs

#### 5.7.10 Encryption

**Decision:** Enable Azure Disk Encryption on all VMs at deployment

**Justification:**
- Compliance requirement (data at rest encryption)
- BitLocker (Windows) or dm-crypt (Linux)
- Keys managed in Azure Key Vault
- No performance impact (hardware-accelerated)
- Industry best practice

**Key Management Options:**
- Platform-managed keys (default)
- Customer-managed keys (for compliance)

---

## 6. Azure Policies

### Recommended Policy Assignments

#### Compute Policies

| Policy Name | Effect | Scope | Purpose |
|-------------|--------|-------|---------|
| Allowed virtual machine size SKUs | Deny | Subscription | Control VM costs and standardization |
| Virtual machines should be in an availability set | Audit | Resource Group | Ensure HA configuration |
| Managed disks should be used | Deny | Subscription | Prevent unmanaged disk creation |
| Virtual machines should encrypt temp disks, caches, and data flows | AuditIfNotExists | Subscription | Ensure encryption compliance |

#### Backup Policies

| Policy Name | Effect | Scope | Purpose |
|-------------|--------|-------|---------|
| Azure Backup should be enabled for Virtual Machines | AuditIfNotExists | Subscription | Ensure backup protection |
| Configure backup on VMs with a given tag | DeployIfNotExists | Subscription | Automated backup deployment |

#### Security Policies

| Policy Name | Effect | Scope | Purpose |
|-------------|--------|-------|---------|
| Microsoft Defender for Cloud should be enabled | AuditIfNotExists | Subscription | Security monitoring |
| Just-In-Time network access control should be applied | AuditIfNotExists | Subscription | Limit management access |
| Endpoint protection should be installed | AuditIfNotExists | Subscription | Malware protection |
| System updates should be installed | AuditIfNotExists | Subscription | Patch compliance |

#### Monitoring Policies

| Policy Name | Effect | Scope | Purpose |
|-------------|--------|-------|---------|
| Azure Monitor Agent should be installed | DeployIfNotExists | Subscription | Ensure monitoring coverage |
| Diagnostic settings should be enabled | AuditIfNotExists | Subscription | Enable logging |

#### Network Policies

| Policy Name | Effect | Scope | Purpose |
|-------------|--------|-------|---------|
| Virtual machines should not have public IP addresses | Deny | Subscription | Reduce attack surface |
| Network interfaces should not have public IPs | Audit | Subscription | Monitor public IP usage |

### Custom Policy Examples

#### Policy: Require Specific Naming Convention

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "field": "name",
          "notMatch": "vm-[prd|dev|tst|uat]-[aue|ase]-[win|lnx]-*-##"
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

#### Policy: Require Mandatory Tags

```json
{
  "mode": "Indexed",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "anyOf": [
            {
              "field": "tags['Environment']",
              "exists": "false"
            },
            {
              "field": "tags['CostCenter']",
              "exists": "false"
            },
            {
              "field": "tags['Owner']",
              "exists": "false"
            }
          ]
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

#### Policy: Enforce Boot Diagnostics

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "field": "Microsoft.Compute/virtualMachines/diagnosticsProfile.bootDiagnostics.enabled",
          "notEquals": "true"
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

---

## 7. Configuration Templates

### Naming Convention Standards

**Format:** `vm-[env]-[region]-[os]-[workload]-[instance]`

**Components:**
- **env**: Environment (prd, dr, uat, tst, dev)
- **region**: Azure region abbreviation (aue, ase, etc.)
- **os**: Operating system (win, lnx)
- **workload**: Application or function (web, app, sql, file, etc.)
- **instance**: Two-digit instance number (01-99)

**Examples:**
- `vm-prd-aue-win-web-01` - Production Windows web server
- `vm-prd-aue-lnx-app-02` - Production Linux application server
- `vm-dev-ase-win-sql-01` - Development Windows SQL server
- `vm-uat-aue-lnx-web-01` - UAT Linux web server

**Related Resources:**
- Disks: `disk-[vm-name]-[osdisk|data##]`
- NICs: `nic-[vm-name]-[01-99]`
- NSGs: `nsg-[subnet-name]`
- Public IPs: `pip-[vm-name]`

### 7.1 Windows Server Production Configuration

#### Web Server Configuration

| Configuration Item | Value | Notes |
|-------------------|-------|-------|
| **VM Name** | vm-prd-[region]-win-web-## | |
| **Subscription** | [Production Subscription] | |
| **Resource Group** | rg-prd-[region]-[app]-web | |
| **Region** | [Primary Region] | e.g., Australia East |
| **Availability** | Availability Zones (1, 2, 3) | 3 VMs across zones |
| **VM Size** | Standard_D4s_v5 | 4 vCPU, 16 GB RAM |
| **Image** | Windows Server 2022 Datacenter | Latest marketplace image |
| **OS Disk** | 127 GB Premium SSD | Encryption enabled |
| **OS Disk Caching** | Read/Write | |
| **Data Disk 1** | 128 GB Premium SSD | Application files |
| **Data Disk 1 Caching** | Read-only | |
| **Virtual Network** | vnet-prd-[region]-[app] | |
| **Subnet** | snet-prd-[region]-web | |
| **Private IP** | Static (IPAM documented) | |
| **Public IP** | None | |
| **NSG** | nsg-prd-[region]-web | Subnet-level |
| **Load Balancer** | lb-prd-[region]-web | Zone-redundant |
| **Accelerated Networking** | Enabled | |
| **Boot Diagnostics** | Enabled | Managed storage |
| **Azure Backup** | Enabled | Daily, 30-day retention |
| **Backup Policy** | policy-prd-daily-30day | |
| **Monitoring** | Azure Monitor (AMA) | |
| **Log Analytics Workspace** | law-prd-[region]-monitoring | |
| **Update Management** | Enabled | Saturday 2-6 AM |

**VM Extensions:**
```
- AzureMonitorWindowsAgent
- DependencyAgentWindows
- MicrosoftAntimalware
- AzureDiskEncryption
- CustomScriptExtension (post-deployment config)
```

**Tags:**
```json
{
  "Environment": "Production",
  "CostCenter": "[Cost Center Code]",
  "Owner": "[Technical Owner Email]",
  "Application": "[Application Name]",
  "Criticality": "High",
  "BackupRequired": "Yes",
  "PatchGroup": "Production-Saturday"
}
```

#### Application Server Configuration

| Configuration Item | Value | Notes |
|-------------------|-------|-------|
| **VM Name** | vm-prd-[region]-win-app-## | |
| **VM Size** | Standard_D8s_v5 | 8 vCPU, 32 GB RAM |
| **OS Disk** | 127 GB Premium SSD | Encryption enabled |
| **Data Disk 1** | 256 GB Premium SSD | Application binaries |
| **Data Disk 2** | 128 GB Premium SSD | Application logs |
| **Other Settings** | Same as web server | Adjust subnet and NSG |

#### SQL Server Configuration

| Configuration Item | Value | Notes |
|-------------------|-------|-------|
| **VM Name** | vm-prd-[region]-win-sql-## | |
| **VM Size** | Standard_E16s_v5 | 16 vCPU, 128 GB RAM |
| **OS Disk** | 127 GB Premium SSD | Encryption enabled |
| **Data Disk 1** | 1 TB Premium SSD (P30) | SQL data files, 5000 IOPS |
| **Data Disk 1 Caching** | Read-only | |
| **Data Disk 2** | 512 GB Premium SSD (P20) | SQL log files, 2300 IOPS |
| **Data Disk 2 Caching** | None | Write-intensive |
| **Data Disk 3** | 256 GB Premium SSD | TempDB |
| **Data Disk 3 Caching** | Read/Write | |
| **Availability** | Availability Zones | SQL Always On AG |
| **Load Balancer** | Internal Load Balancer | AG listener |
| **Backup** | Azure Backup + SQL native | |
| **Other Settings** | Same as web server | Adjust subnet and NSG |

**SQL-Specific Configuration:**
- SQL Server Enterprise or Standard edition
- SQL IaaS Agent Extension installed
- Automated backups configured
- SQL Server Management Studio pre-installed
- Performance monitoring enabled

### 7.2 Linux Production Configuration

#### Web Server Configuration

| Configuration Item | Value | Notes |
|-------------------|-------|-------|
| **VM Name** | vm-prd-[region]-lnx-web-## | |
| **Subscription** | [Production Subscription] | |
| **Resource Group** | rg-prd-[region]-[app]-web | |
| **Region** | [Primary Region] | |
| **Availability** | Availability Zones (1, 2, 3) | |
| **VM Size** | Standard_D4s_v5 | 4 vCPU, 16 GB RAM |
| **Image** | Ubuntu 22.04 LTS or RHEL 9 | Latest marketplace image |
| **OS Disk** | 30 GB Premium SSD | Encryption enabled |
| **Data Disk 1** | 128 GB Premium SSD | Application files |
| **Authentication** | SSH key (no password) | |
| **Virtual Network** | vnet-prd-[region]-[app] | |
| **Subnet** | snet-prd-[region]-web | |
| **Private IP** | Static (IPAM documented) | |
| **Public IP** | None | |
| **NSG** | nsg-prd-[region]-web | |
| **Load Balancer** | lb-prd-[region]-web | |
| **Accelerated Networking** | Enabled | |
| **Boot Diagnostics** | Enabled | |
| **Azure Backup** | Enabled | Daily, 30-day retention |
| **Monitoring** | Azure Monitor (AMA) | |
| **Update Management** | Enabled | Saturday 2-6 AM |

**VM Extensions:**
```
- AzureMonitorLinuxAgent
- DependencyAgentLinux
- AzureSecurityLinuxAgent
- AzureDiskEncryptionForLinux
- CustomScriptForLinux (post-deployment config)
```

**Tags:** Same as Windows configuration

#### Database Server Configuration (PostgreSQL/MySQL)

| Configuration Item | Value | Notes |
|-------------------|-------|-------|
| **VM Name** | vm-prd-[region]-lnx-db-## | |
| **VM Size** | Standard_E16s_v5 | 16 vCPU, 128 GB RAM |
| **OS Disk** | 30 GB Premium SSD | |
| **Data Disk 1** | 1 TB Premium SSD | Database data |
| **Data Disk 2** | 512 GB Premium SSD | Database logs |
| **Other Settings** | Same as Linux web server | Adjust subnet and NSG |

### 7.3 Non-Production Configuration

#### Development/Test Environment

| Configuration Item | Value | Notes |
|-------------------|-------|-------|
| **VM Name** | vm-[env]-[region]-[os]-[workload]-## | |
| **Subscription** | [Non-Prod Subscription] | |
| **Resource Group** | rg-[env]-[region]-[app] | |
| **Availability** | None (single instance) | Cost optimization |
| **VM Size** | Smaller than production | D2s_v5 or B-series |
| **OS Disk** | 127 GB Standard SSD | Windows |
| **OS Disk** | 30 GB Standard SSD | Linux |
| **Data Disks** | Standard SSD | As needed |
| **Public IP** | None | |
| **Backup** | Weekly, 7-14 day retention | |
| **Auto-Shutdown** | Enabled | 7 PM weekdays |
| **Tags** | Include Environment tag | Dev, Test, UAT |

**Cost Optimizations:**
- Auto-shutdown after business hours
- Smaller VM sizes
- Standard SSD instead of Premium
- No availability zones
- Reduced backup retention

### 7.4 Disaster Recovery Configuration

| Configuration Item | Value | Notes |
|-------------------|-------|-------|
| **VM Name** | vm-dr-[region]-[os]-[workload]-## | |
| **Subscription** | [DR Subscription or same] | |
| **Resource Group** | rg-dr-[region]-[app] | |
| **Region** | [Secondary Region] | Paired region |
| **Replication** | Azure Site Recovery | RPO: 15 minutes |
| **Test Failover** | Quarterly | Documented procedure |
| **Configuration** | Mirror production | Same specs |

**ASR Configuration:**
- Recovery Services Vault: `rsv-dr-[region]`
- Replication policy: Continuous
- Recovery plan: Documented with runbooks
- Network mapping: Configured
- Test failover: Isolated VNet

### 7.5 Infrastructure as Code Examples

#### Terraform Module Structure

```
terraform/
├── modules/
│   └── virtual-machine/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       ├── vm.tf
│       ├── disks.tf
│       ├── networking.tf
│       ├── extensions.tf
│       └── README.md
└── environments/
    ├── production/
    │   ├── main.tf
    │   ├── terraform.tfvars
    │   └── backend.tf
    └── non-production/
        ├── main.tf
        ├── terraform.tfvars
        └── backend.tf
```

#### Bicep Module Structure

```
bicep/
├── modules/
│   ├── vm-windows.bicep
│   ├── vm-linux.bicep
│   ├── vm-disks.bicep
│   └── vm-extensions.bicep
└── parameters/
    ├── prod-windows-web.parameters.json
    ├── prod-linux-app.parameters.json
    └── dev-windows.parameters.json
```

#### Sample Terraform Variables

```hcl
variable "vm_config" {
  description = "Virtual Machine configuration"
  type = object({
    name                = string
    size                = string
    os_type             = string  # Windows or Linux
    admin_username      = string
    enable_backup       = bool
    enable_monitoring   = bool
    availability_zones  = list(string)
    data_disks          = list(object({
      name              = string
      size_gb           = number
      tier              = string
      caching           = string
    }))
  })
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
}
```

---

## 8. Operational Procedures

### 8.1 VM Provisioning Process

**Standard Workflow:**
1. **Request Submission** - Submit VM request with business justification
2. **Design Review** - Validate sizing, availability, and security requirements
3. **Approval** - Obtain governance approval (production VMs)
4. **Provisioning** - Deploy via IaC pipeline
5. **Configuration** - Apply baseline configuration and extensions
6. **Testing** - Validate functionality and performance
7. **Handover** - Document and transfer to operations team

**Required Information:**
- Application name and purpose
- Environment (production, UAT, development)
- Operating system and version
- Sizing requirements (vCPU, RAM, storage)
- Availability requirements (SLA, RTO, RPO)
- Network connectivity requirements
- Security and compliance requirements
- Cost center and owner information

### 8.2 Patching and Update Management

**Patch Schedule:**

| Environment | Day | Time Window | Approval Required |
|-------------|-----|-------------|-------------------|
| Development | Tuesday | 8 PM - 11 PM | No |
| Test | Wednesday | 8 PM - 11 PM | No |
| UAT | Thursday | 8 PM - 11 PM | Change Manager |
| Production | Saturday | 2 AM - 6 AM | Change Manager |

**Patching Process:**
1. **Assessment** - Review available updates weekly
2. **Testing** - Deploy to development environment first
3. **Validation** - Test application functionality
4. **Approval** - Submit change request for production
5. **Deployment** - Apply patches during maintenance window
6. **Verification** - Confirm successful deployment
7. **Rollback** - Revert if issues detected

**Emergency Patching:**
- Critical security patches: Within 7 days
- Zero-day vulnerabilities: Within 48 hours
- Follow expedited change management process

### 8.3 Backup and Restore Procedures

**Backup Verification:**
- Daily: Automated backup job success check
- Weekly: Restore test for production VMs
- Monthly: Full disaster recovery test
- Quarterly: Business continuity exercise

**Restore Procedures:**

**File-Level Restore:**
1. Navigate to Recovery Services Vault
2. Select VM and recovery point
3. Choose files/folders to restore
4. Mount recovery point or restore to alternate location
5. Copy required files
6. Unmount recovery point

**Full VM Restore:**
1. Stop or delete failed VM
2. Navigate to Recovery Services Vault
3. Select VM and recovery point
4. Choose restore options (new VM or replace existing)
5. Verify network and storage configuration
6. Initiate restore
7. Validate VM functionality
8. Update DNS/load balancer if needed

**Typical Restore Times:**
- File-level: 10-30 minutes
- Full VM restore: 30 minutes - 2 hours (depending on size)

### 8.4 Monitoring and Alerting

**Standard Alerts:**

| Alert | Threshold | Severity | Action |
|-------|-----------|----------|--------|
| CPU > 85% | 15 minutes | Warning | Investigate performance |
| CPU > 95% | 5 minutes | Critical | Immediate action |
| Memory > 90% | 15 minutes | Warning | Investigate memory leak |
| Memory > 95% | 5 minutes | Critical | Immediate action |
| Disk space < 20% | N/A | Warning | Clean up or expand disk |
| Disk space < 10% | N/A | Critical | Immediate action required |
| VM not responding | 5 minutes | Critical | Investigate and restart |
| Backup failed | N/A | Critical | Investigate backup issue |

**Monitoring Dashboard:**
- VM availability and uptime
- Resource utilization trends
- Backup status
- Patch compliance
- Security alerts
- Cost tracking

### 8.5 Incident Response

**Severity Levels:**

| Severity | Description | Response Time | Resolution Time |
|----------|-------------|---------------|-----------------|
| **Critical** | Production down | 15 minutes | 4 hours |
| **High** | Degraded performance | 1 hour | 24 hours |
| **Medium** | Non-critical issue | 4 hours | 72 hours |
| **Low** | Informational | 24 hours | Best effort |

**Common Issues and Resolutions:**

**VM won't start:**
1. Check boot diagnostics
2. Review activity logs
3. Verify disk health
4. Check network connectivity
5. Restore from backup if needed

**High CPU usage:**
1. Identify process causing high CPU
2. Check for malware or unauthorized processes
3. Review application logs
4. Consider VM resizing
5. Optimize application code if needed

**Disk space full:**
1. Identify large files/folders
2. Clean up temp files and logs
3. Archive old data
4. Expand disk if needed
5. Implement log rotation

---

## 9. Acceptance

### Sign-Off Requirements

This design document requires formal acceptance before implementation commences.

**Acceptance Criteria:**
- [ ] Technical design reviewed and approved
- [ ] Security controls validated
- [ ] Cost estimation accepted
- [ ] RBAC model approved
- [ ] Naming conventions agreed
- [ ] Backup and DR strategy approved
- [ ] Patching schedule approved
- [ ] Monitoring and alerting defined
- [ ] Compliance requirements met
- [ ] Operational procedures documented

### Approval Signatures

**Document Details:**

| Item | Value |
|------|-------|
| **Project** | [Project Name] |
| **Document Version** | 1.0 |
| **Review Date** | [DD/MM/YYYY] |

---

**Signed on behalf of [Customer Name]:**

| | |
|---|---|
| **Name** | _________________________ |
| **Position** | _________________________ |
| **Signature** | _________________________ |
| **Date** | _________________________ |

---

**Signed on behalf of [Implementation Partner]:**

| | |
|---|---|
| **Name** | _________________________ |
| **Position** | _________________________ |
| **Signature** | _________________________ |
| **Date** | _________________________ |

---

## Appendix A: VM Size Selection Guide

### General Purpose (D-series)

**Use Cases:** Web servers, application servers, small databases, dev/test

| VM Size | vCPU | RAM | Temp Storage | Max IOPS | Max NICs | Price Range |
|---------|------|-----|--------------|----------|----------|-------------|
| D2s_v5 | 2 | 8 GB | 75 GB | 3,750 | 2 | $ |
| D4s_v5 | 4 | 16 GB | 150 GB | 6,400 | 2 | $ |
| D8s_v5 | 8 | 32 GB | 300 GB | 12,800 | 4 | $$ |
| D16s_v5 | 16 | 64 GB | 600 GB | 25,600 | 8 | $$ |

### Memory Optimized (E-series)

**Use Cases:** Large databases, in-memory analytics, SAP applications

| VM Size | vCPU | RAM | Temp Storage | Max IOPS | Max NICs | Price Range |
|---------|------|-----|--------------|----------|----------|-------------|
| E4s_v5 | 4 | 32 GB | 150 GB | 6,400 | 2 | $ |
| E8s_v5 | 8 | 64 GB | 300 GB | 12,800 | 4 | $$ |
| E16s_v5 | 16 | 128 GB | 600 GB | 25,600 | 8 | $$ |
| E32s_v5 | 32 | 256 GB | 1,200 GB | 51,200 | 8 | $$$ |

### Compute Optimized (F-series)

**Use Cases:** Batch processing, web servers, gaming servers

| VM Size | vCPU | RAM | Temp Storage | Max IOPS | Max NICs | Price Range |
|---------|------|-----|--------------|----------|----------|-------------|
| F4s_v2 | 4 | 8 GB | 32 GB | 6,400 | 2 | $ |
| F8s_v2 | 8 | 16 GB | 64 GB | 12,800 | 4 | $$ |
| F16s_v2 | 16 | 32 GB | 128 GB | 25,600 | 8 | $$ |

### Burstable (B-series)

**Use Cases:** Dev/test, low-traffic web servers, proof of concepts

| VM Size | vCPU | RAM | Baseline CPU% | Max Burst | Price Range |
|---------|------|-----|---------------|-----------|-------------|
| B2s | 2 | 4 GB | 40% | 200% | $ |
| B2ms | 2 | 8 GB | 60% | 200% | $ |
| B4ms | 4 | 16 GB | 90% | 400% | $ |

**Note:** B-series accumulates credits when below baseline and consumes when above

---

## Appendix B: Disk Performance Reference

### Managed Disk Comparison

**Standard HDD:**

| Size | IOPS | Throughput | Price | Use Case |
|------|------|------------|-------|----------|
| 32 GB | 500 | 60 MB/s | $ | Archive |
| 128 GB | 500 | 60 MB/s | $ | Backup |
| 512 GB | 500 | 60 MB/s | $ | Cold storage |
| 1 TB | 500 | 60 MB/s | $ | Long-term archive |

**Standard SSD:**

| Size | IOPS | Throughput | Price | Use Case |
|------|------|------------|-------|----------|
| 128 GB | 500 | 60 MB/s | $ | Dev/test |
| 256 GB | 500 | 60 MB/s | $ | Non-critical apps |
| 512 GB | 500 | 60 MB/s | $$ | Non-prod databases |
| 1 TB | 500 | 60 MB/s | $$ | File shares |

**Premium SSD:**

| Size | Disk Name | IOPS | Throughput | Price | Use Case |
|------|-----------|------|------------|-------|----------|
| 128 GB | P10 | 500 | 100 MB/s | $$ | OS disks |
| 256 GB | P15 | 1,100 | 125 MB/s | $$ | Small databases |
| 512 GB | P20 | 2,300 | 150 MB/s | $$ | Database logs |
| 1 TB | P30 | 5,000 | 200 MB/s | $$$ | Database data |
| 2 TB | P40 | 7,500 | 250 MB/s | $$$ | Large databases |

**Premium SSD v2:**
- Customizable IOPS (3,000 - 80,000)
- Customizable throughput (125 - 1,200 MB/s)
- Pay for exact capacity, IOPS, and throughput needed
- No fixed disk sizes
- Ideal for workloads with variable performance needs

**Ultra Disk:**
- Up to 160,000 IOPS
- Up to 4,000 MB/s throughput
- Sub-millisecond latency
- Dynamically adjustable performance
- Use for: SAP HANA, top-tier databases, transaction-heavy workloads

---

## Appendix C: Security Baseline Configuration

### Windows Server Security Baseline

**User Rights Assignment:**
- Deny log on locally: Guests
- Deny log on through Remote Desktop: Guests, Local account
- Shut down the system: Administrators only

**Security Options:**
- Interactive logon: Do not display last user name: Enabled
- Network access: Do not allow anonymous enumeration: Enabled
- Network security: LAN Manager authentication level: NTLMv2 only
- User Account Control: Admin Approval Mode: Enabled

**Audit Policy:**
```
Account Logon Events: Success, Failure
Account Management: Success, Failure
Logon Events: Success, Failure
Object Access: Failure
Policy Change: Success, Failure
Privilege Use: Failure
System Events: Success, Failure
```

**Services to Disable:**
- Remote Registry
- Print Spooler (if not needed)
- Server Message Block 1.0 (SMB1)
- Windows Remote Management (if not needed)

**Windows Firewall:**
- Domain Profile: Enabled
- Private Profile: Enabled
- Public Profile: Enabled
- Inbound connections: Block by default
- Outbound connections: Allow

**Additional Hardening:**
- Disable PowerShell v2
- Enable PowerShell logging
- Configure AppLocker (optional)
- Enable Windows Defender
- Configure BitLocker (via Azure Disk Encryption)

### Linux Security Baseline

**SSH Configuration (/etc/ssh/sshd_config):**
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 0
Protocol 2
```

**Firewall Configuration (firewalld/iptables):**
- Default deny incoming
- Allow SSH from specific sources only
- Allow application ports as needed
- Enable logging for denied connections

**System Hardening:**
- Disable unnecessary services
- Remove unnecessary packages
- Configure SELinux (RHEL) or AppArmor (Ubuntu): Enforcing
- Set file permissions appropriately
- Configure sudo access with least privilege
- Enable automatic security updates

**Audit Configuration (auditd):**
```
Monitor privileged commands
Monitor file access to sensitive files
Monitor user and group changes
Monitor network connections
Monitor system calls
```

**Additional Hardening:**
- Implement fail2ban or similar IDS
- Configure log forwarding to Azure Monitor
- Disable USB storage (if not needed)
- Configure kernel parameters for security
- Enable AIDE for file integrity monitoring

---

## Appendix D: Backup and Recovery Matrix

### Backup Retention Policies

| Environment | Backup Type | Frequency | Retention | Recovery Point |
|-------------|-------------|-----------|-----------|----------------|
| **Production - Critical** | Full | Daily | 90 days | < 4 hours |
| **Production - Standard** | Full | Daily | 30 days | < 24 hours |
| **UAT/Staging** | Full | Daily | 14 days | < 24 hours |
| **Development** | Full | Weekly | 7 days | < 7 days |
| **Test** | Full | Weekly | 7 days | Best effort |

### Application-Specific Backup Requirements

**SQL Server:**
- Full backup: Daily at 2 AM
- Differential backup: Every 6 hours
- Transaction log backup: Every 15 minutes
- Retention: 30 days (production)
- Native SQL backup + Azure Backup

**File Servers:**
- Full backup: Daily at 3 AM
- Retention: 30 days
- Item-level recovery enabled

**Application Servers:**
- Full backup: Daily at 4 AM
- Pre-change snapshot
- Retention: 30 days

### Disaster Recovery Tiers

| Tier | RTO | RPO | Recovery Method | Cost |
|------|-----|-----|-----------------|------|
| **Tier 0 (Critical)** | < 1 hour | < 15 min | ASR + Hot standby | $$ |
| **Tier 1 (High)** | < 4 hours | < 1 hour | ASR + Warm standby | $$ |
| **Tier 2 (Medium)** | < 24 hours | < 4 hours | ASR + Cold standby | $ |
| **Tier 3 (Low)** | < 72 hours | < 24 hours | Backup restore | $ |

---

## Appendix E: Change Management Templates

### VM Modification Change Request

**Change Details:**
- Change Type: [Resize/Add Disk/Configuration Change]
- VM Name: [VM Name]
- Environment: [Production/Non-Production]
- Scheduled Date/Time: [DD/MM/YYYY HH:MM]
- Duration: [Estimated downtime]
- Rollback Plan: [Describe rollback procedure]

**Business Justification:**
- Reason for change
- Business impact if not implemented
- Risk of not implementing

**Technical Details:**
- Current configuration
- Proposed configuration
- Testing completed
- Validation steps

**Approval:**
- Technical Lead: _______________
- Change Manager: _______________
- Business Owner: _______________

### VM Decommission Checklist

**Pre-Decommission:**
- [ ] Verify VM is no longer needed
- [ ] Check for dependencies (other systems, applications)
- [ ] Export configuration for documentation
- [ ] Take final backup
- [ ] Notify stakeholders (7 days advance notice)
- [ ] Obtain approval from VM owner
- [ ] Document decommission in CMDB

**Decommission Process:**
- [ ] Remove from monitoring
- [ ] Remove from backup policy
- [ ] Remove from patching schedule
- [ ] Stop VM
- [ ] Retain for 30 days (production) or 7 days (non-prod)
- [ ] Delete VM and associated resources
- [ ] Remove DNS entries
- [ ] Update load balancer configuration
- [ ] Release IP address
- [ ] Delete NSG rules (if dedicated)
- [ ] Update documentation

**Post-Decommission:**
- [ ] Verify no orphaned resources (disks, NICs, public IPs)
- [ ] Update asset register
- [ ] Confirm cost reduction
- [ ] Archive final backup (if required)

---

## Appendix F: Troubleshooting Guide

### Common Issues and Resolutions

**Issue: VM in Failed State**

**Symptoms:** VM shows as "Failed" in portal

**Troubleshooting Steps:**
1. Check Activity Log for error details
2. Review boot diagnostics
3. Check for platform issues (Azure status page)
4. Verify subscription limits not exceeded
5. Check for failed extensions

**Resolution:**
- Redeploy VM to new host
- Remove failed extensions
- Restart VM
- Contact Azure support if platform issue

---

**Issue: Cannot Connect via RDP/SSH**

**Symptoms:** Connection timeout or refused

**Troubleshooting Steps:**
1. Verify VM is running
2. Check NSG rules (subnet and NIC level)
3. Verify private IP address
4. Check if JIT access is enabled and approved
5. Review VM network interface
6. Check if public IP exists (if connecting from internet)
7. Test connectivity with Network Watcher

**Resolution:**
- Add NSG rule to allow RDP (3389) or SSH (22)
- Request JIT access
- Use Azure Bastion as alternative
- Reset network interface
- Use serial console for troubleshooting

---

**Issue: Disk Performance Issues**

**Symptoms:** High latency, low IOPS, application slowness

**Troubleshooting Steps:**
1. Check disk metrics (IOPS, throughput)
2. Verify disk tier (Standard/Premium/Ultra)
3. Check if hitting IOPS/throughput limits
4. Review VM size limits
5. Check host caching configuration
6. Look for disk throttling events

**Resolution:**
- Upgrade to Premium SSD or Ultra Disk
- Increase disk size for higher IOPS
- Enable/adjust host caching
- Resize VM for higher limits
- Optimize application I/O patterns
- Consider disk striping (RAID)

---

**Issue: High Memory Usage**

**Symptoms:** Memory consistently above 90%, application crashes

**Troubleshooting Steps:**
1. Identify processes consuming memory
2. Check for memory leaks
3. Review application logs
4. Check if page file configured properly
5. Look for malware or unauthorized processes

**Resolution:**
- Restart problematic services
- Resize VM to larger memory
- Optimize application memory usage
- Add page file disk (temporary disk)
- Investigate and fix memory leaks

---

**Issue: Boot Issues**

**Symptoms:** VM won't start, boot loop, boot errors

**Troubleshooting Steps:**
1. Review boot diagnostics screenshot
2. Check serial console output
3. Look for recent changes (extensions, updates)
4. Check for disk issues
5. Review activity log

**Resolution:**
- Remove problematic extensions
- Boot into safe mode (if possible)
- Repair boot configuration
- Restore from backup
- Attach OS disk to recovery VM

---

**Issue: Extension Installation Failed**

**Symptoms:** Extension shows as "Provisioning failed"

**Troubleshooting Steps:**
1. Check extension logs
2. Verify internet connectivity from VM
3. Check if previous extension version needs removal
4. Verify VM agent is running
5. Check for conflicting extensions

**Resolution:**
- Remove and reinstall extension
- Update VM agent
- Check firewall rules for extension endpoints
- Review extension prerequisites
- Contact Azure support for persistent issues

---

## Appendix G: Azure Monitor Queries

### Useful KQL Queries for VM Monitoring

**CPU Usage Over Time:**
```kusto
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where Computer == "vm-name"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart
```

**Memory Usage:**
```kusto
Perf
| where ObjectName == "Memory" and CounterName == "Available MBytes"
| where Computer == "vm-name"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart
```

**Disk Space:**
```kusto
Perf
| where ObjectName == "LogicalDisk" and CounterName == "% Free Space"
| where Computer == "vm-name"
| summarize avg(CounterValue) by InstanceName, bin(TimeGenerated, 1h)
| render timechart
```

**Failed Login Attempts (Windows):**
```kusto
SecurityEvent
| where EventID == 4625
| where Computer == "vm-name"
| summarize count() by Account, IpAddress
| order by count_ desc
```

**Failed Login Attempts (Linux):**
```kusto
Syslog
| where Facility == "auth" or Facility == "authpriv"
| where SyslogMessage contains "Failed password"
| where Computer == "vm-name"
| summarize count() by ProcessName, SyslogMessage
| order by count_ desc
```

**Top Processes by CPU:**
```kusto
Perf
| where ObjectName == "Process" and CounterName == "% Processor Time"
| where Computer == "vm-name"
| summarize avg(CounterValue) by InstanceName
| order by avg_CounterValue desc
| take 10
```

**Network Traffic:**
```kusto
Perf
| where ObjectName == "Network Adapter" 
| where CounterName == "Bytes Total/sec"
| where Computer == "vm-name"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart
```

**VM Availability:**
```kusto
Heartbeat
| where Computer == "vm-name"
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| extend IsOnline = iff(LastHeartbeat > ago(5m), "Online", "Offline")
```

---

## Appendix H: Cost Optimization Checklist

### Monthly Cost Review Tasks

**Resource Optimization:**
- [ ] Review Azure Advisor recommendations
- [ ] Identify and remove orphaned resources
  - Unattached disks
  - Unused NICs
  - Orphaned public IPs
  - Old snapshots
- [ ] Right-size underutilized VMs (< 5% CPU average)
- [ ] Convert appropriate workloads to Reserved Instances
- [ ] Apply Azure Hybrid Benefit where applicable
- [ ] Review and optimize disk tiers

**Operational Efficiency:**
- [ ] Verify auto-shutdown schedules for non-production VMs
- [ ] Review VM running time reports
- [ ] Consolidate underutilized VMs
- [ ] Evaluate spot VM opportunities
- [ ] Review backup retention and optimize
- [ ] Clean up old recovery points

**Cost Allocation:**
- [ ] Verify all resources are tagged
- [ ] Review cost by tag/resource group
- [ ] Generate cost reports by application
- [ ] Identify cost anomalies
- [ ] Update budget alerts if needed

**Savings Opportunities:**
- [ ] Evaluate 1-year Reserved Instance savings
- [ ] Evaluate 3-year Reserved Instance savings
- [ ] Consider Azure Hybrid Benefit for Windows/SQL
- [ ] Review region pricing differences
- [ ] Evaluate Azure Savings Plans

### Annual Cost Review

- [ ] Review Reserved Instance commitments (renewal/change)
- [ ] Assess VM family generation upgrades (e.g., v4 → v5)
- [ ] Evaluate workload migration to PaaS where appropriate
- [ ] Review disaster recovery costs vs. requirements
- [ ] Benchmark against industry standards

---

## Appendix I: Compliance and Audit

### Compliance Checklist

**Data Protection:**
- [ ] Encryption at rest enabled (Azure Disk Encryption)
- [ ] Encryption in transit (TLS 1.2+)
- [ ] Data classification tags applied
- [ ] Backup retention meets requirements
- [ ] Geographic data residency requirements met

**Access Control:**
- [ ] RBAC configured with least privilege
- [ ] Just-In-Time access enabled
- [ ] MFA required for administrative access
- [ ] Privileged access documented and reviewed
- [ ] Service principal usage audited

**Monitoring and Logging:**
- [ ] Diagnostic settings enabled
- [ ] Logs retained per policy (90-365 days)
- [ ] Security alerts configured
- [ ] Audit logs reviewed regularly
- [ ] Incident response plan documented

**Vulnerability Management:**
- [ ] Vulnerability assessment enabled
- [ ] Patch compliance > 95%
- [ ] Critical vulnerabilities remediated within SLA
- [ ] Third-party software patching documented
- [ ] Penetration testing completed annually

**Operational:**
- [ ] Asset inventory current and accurate
- [ ] Change management procedures followed
- [ ] Disaster recovery tested quarterly
- [ ] Documentation up to date
- [ ] Business continuity plan validated

### Audit Evidence Collection

**For Compliance Audits, Prepare:**
1. VM inventory with classification
2. RBAC role assignments report
3. Backup compliance report
4. Patch compliance report
5. Security alert history
6. Encryption status report
7. Access logs (successful and failed)
8. Change management records
9. Disaster recovery test results
10. Vulnerability scan reports

---

## Appendix J: Migration Planning

### Migration Approach

**Assessment Phase:**
1. Inventory existing VMs
2. Document dependencies
3. Classify by criticality
4. Assess Azure readiness
5. Estimate costs
6. Identify blockers

**Planning Phase:**
1. Define migration waves
2. Create detailed runbooks
3. Plan network connectivity
4. Design Azure architecture
5. Establish testing criteria
6. Prepare rollback plans

**Migration Methods:**

| Method | Use Case | Downtime | Complexity |
|--------|----------|----------|------------|
| **Lift and Shift** | IaaS migration | Hours | Low |
| **Azure Migrate** | Physical/VMware/Hyper-V | Minutes | Medium |
| **ASR** | Disaster recovery enabled migration | Minutes | Medium |
| **Backup and Restore** | Small VMs | Hours | Low |
| **Manual Rebuild** | Modernization opportunity | Varies | High |

### Pre-Migration Checklist

**Source Environment:**
- [ ] Document current configuration
- [ ] Verify backup exists
- [ ] Check for installed agents
- [ ] Document application dependencies
- [ ] Test current functionality
- [ ] Identify static IPs and DNS entries
- [ ] Review security configurations
- [ ] Document installed software and licenses

**Azure Environment:**
- [ ] Subscription and resource groups created
- [ ] Virtual networks configured
- [ ] NSGs defined
- [ ] RBAC configured
- [ ] Naming convention validated
- [ ] Tagging strategy defined
- [ ] Backup policies created
- [ ] Monitoring configured

### Post-Migration Checklist

**Validation:**
- [ ] VM powered on successfully
- [ ] OS accessible (RDP/SSH)
- [ ] Network connectivity verified
- [ ] Application functionality tested
- [ ] Performance validated
- [ ] Backup configured and tested
- [ ] Monitoring alerts receiving data
- [ ] DNS updated (if needed)
- [ ] Load balancer updated (if applicable)

**Optimization:**
- [ ] VM sized appropriately
- [ ] Disk performance optimized
- [ ] Accelerated networking enabled
- [ ] Extensions deployed
- [ ] Security baseline applied
- [ ] Cost optimization applied
- [ ] Documentation updated

**Decommission Source:**
- [ ] Verify all functionality in Azure
- [ ] Obtain approval to decommission
- [ ] Power off source VM
- [ ] Retain for rollback period (30 days)
- [ ] Delete source VM
- [ ] Update CMDB

---

## Appendix K: Support and Escalation

### Support Tiers

| Issue Type | Support Level | Contact | Response Time |
|------------|---------------|---------|---------------|
| **Informational** | Self-service | Documentation, KB | N/A |
| **Low Priority** | L1 Support | Service desk | 24 hours |
| **Medium Priority** | L2 Support | Infrastructure team | 4 hours |
| **High Priority** | L3 Support | Azure specialists | 1 hour |
| **Critical** | Azure Support | Microsoft support | 15 minutes |

### Azure Support Plans

| Plan | Price | Response Time (Critical) | Channels |
|------|-------|-------------------------|----------|
| **Basic** | Free | N/A | Community forums |
| **Developer** | $29/month | 8 hours | Email (business hours) |
| **Standard** | $100/month | 1 hour | Email (24/7) |
| **Professional Direct** | $1,000/month | 1 hour | Email + phone (24/7) |

### Escalation Path

**Level 1: Service Desk**
- Initial triage
- Basic troubleshooting
- Password resets
- Access requests

**Level 2: Infrastructure Team**
- VM management
- Performance troubleshooting
- Configuration changes
- Backup/restore operations

**Level 3: Azure Specialists**
- Complex technical issues
- Azure platform issues
- Architecture reviews
- Performance optimization

**Level 4: Microsoft Support**
- Platform-level issues
- Service outages
- Critical production issues
- Bugs and product defects

### Contact Information

**Internal Contacts:**
- Service Desk: [Phone/Email]
- Infrastructure Team: [Phone/Email]
- Azure Team: [Phone/Email]
- Security Team: [Phone/Email]

**External Contacts:**
- Microsoft Support: [Support Portal URL]
- Azure Status: https://status.azure.com
- Azure Documentation: https://docs.microsoft.com/azure

---

*End of Document*

**Document Classification:** [Internal/Confidential]  
**Next Review Date:** [DD/MM/YYYY]  
**Owner:** [Team/Department Name]  
**Distribution:** [Distribution List]

---

## Document Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | [DD/MM/YYYY] | [Name] | Initial release |
| | | | |
| | | | |

---

**Quick Reference Card**

**Emergency Contacts:**
- Critical Issue Hotline: [Phone]
- Azure Support: [Portal/Phone]
- Security Incident: [Phone/Email]

**Common Commands:**

**PowerShell:**
```powershell
# Check VM status
Get-AzVM -ResourceGroupName "rg-name" -Name "vm-name" -Status

# Start VM
Start-AzVM -ResourceGroupName "rg-name" -Name "vm-name"

# Stop VM
Stop-AzVM -ResourceGroupName "rg-name" -Name "vm-name" -Force

# Restart VM
Restart-AzVM -ResourceGroupName "rg-name" -Name "vm-name"
```

**Azure CLI:**
```bash
# Check VM status
az vm get-instance-view --resource-group rg-name --name vm-name

# Start VM
az vm start --resource-group rg-name --name vm-name

# Stop VM
az vm deallocate --resource-group rg-name --name vm-name

# Restart VM
az vm restart --resource-group rg-name --name vm-name
```

```
Key Features of This VM Template:
Comprehensive Coverage:

Complete WAF Analysis - All five pillars with detailed checklists
Security Deep-Dive - Windows and Linux security baselines, encryption, compliance
Operational Procedures - Patching, backup/restore, monitoring, incident response
Cost Optimization - Reserved Instances, Hybrid Benefit, rightsizing strategies
High Availability - Availability Zones, Sets, load balancing configurations

Practical Implementation:

VM sizing guides by workload type
Disk performance reference tables
Network security group templates
Standard configurations for Windows and Linux
IaC examples (Terraform, Bicep)
KQL queries for monitoring

10 Comprehensive Appendices:

A: VM Size Selection Guide
B: Disk Performance Reference
C: Security Baseline Configuration
D: Backup and Recovery Matrix
E: Change Management Templates
F: Troubleshooting Guide
G: Azure Monitor Queries
H: Cost Optimization Checklist
I: Compliance and Audit
J: Migration Planning
K: Support and Escalation

How to Customize:

Replace all [Customer Name] placeholders
Update pricing with current Azure rates
Add customer-specific security frameworks
Customize naming conventions
Adjust backup retention per requirements
Map to customer's RBAC structure
Update support contacts and escalation paths

This template provides everything needed for enterprise-grade Azure VM deployments with consistent standards, security controls, and operational excellence!
```
