````markdown
# Application Migration Project Plan Template

---

## Document Control

| Field             | Value                         |
|-------------------|-------------------------------|
| Application Name  | Customer Portal               |
| Application ID    | APP-1029                      |
| Migration Wave    | Wave 2                        |
| Project Manager   | Sarah Johnson                 |
| Technical Lead    | Michael Chen                  |

---

## Document History

| Version | Issue Date | Changes       | Author   |
|---------|------------|---------------|----------|
| 0.1     | 2025-08-01 | Initial draft | M. Chen  |
| 1.0     | 2025-09-01 | Approved      | S. Johnson |

---

## Document Approvals

| Role              | Name          | Signature | Date       |
|-------------------|---------------|-----------|------------|
| Application Owner | David Roberts |           | 2025-09-03 |
| Technical Lead    | Michael Chen  |           | 2025-09-03 |
| Security Lead     | Priya Patel   |           | 2025-09-04 |
| Project Manager   | Sarah Johnson |           | 2025-09-04 |

---

# 1. Application Overview

## 1.1 Application Description
- **Business Function:** Provides online customer account management, billing, and support services.  
- **Technology Stack:** Windows Server 2016, IIS, Oracle 12c, .NET Framework 4.7  
- **Current Environment:** On-premises data center (Sydney) with 4 VMs and Oracle RAC cluster  
- **Business Criticality:** High – required for customer billing and SLA compliance  

## 1.2 Migration Objectives
**Primary Goals:**  
- [x] Improve performance and scalability  
- [x] Reduce operational costs  
- [x] Enhance security posture  
- [x] Modernize infrastructure  

**Success Criteria:**  
- Application functionality maintained post-migration  
- Response times ≤ 300ms under load  
- PCI-DSS compliance validated  
- Operations handover completed with updated runbooks  

## 1.3 Migration Strategy
- **Selected Strategy:** Replatform (RDS, EC2)  
- **Justification:** Reduces management overhead while preserving business logic  
- **Migration Pattern:** Lift-and-Shift with partial modernization  

---

# 2. Current State Assessment

## 2.1 Technical Architecture

**Server Details:**

| Server Name  | Role        | OS              | CPU  | Memory | Storage | Dependencies |
|--------------|-------------|-----------------|------|--------|---------|--------------|
| CUST-APP-01  | App Server  | Windows 2016    | 8 vCPU | 32 GB | 500 GB | IIS, .NET    |
| CUST-DB-01   | DB Primary  | Oracle Linux 7  | 16 vCPU| 64 GB | 1 TB   | Oracle 12c   |
| CUST-DB-02   | DB Standby  | Oracle Linux 7  | 16 vCPU| 64 GB | 1 TB   | Oracle 12c   |

**Database Information:**
- Type: Oracle 12c Enterprise Edition  
- Version: 12.2.0.1  
- Size: 2 TB  
- Backup Strategy: RMAN nightly, weekly full, 30-day retention  

**Network Configuration:**
- Current Subnets: 10.10.0.0/24 (App), 10.10.1.0/24 (DB)  
- Firewall Rules: Allow HTTPS (443), SQLNet (1521) internal only  
- Load Balancers: F5 LTM  
- External Connections: Payment Gateway, CRM SOAP API  

## 2.2 Dependencies

**Application Dependencies:**

| Dependency Type | System/Service     | Impact Level | Migration Considerations |
|-----------------|--------------------|--------------|--------------------------|
| Database        | Oracle 12c         | High         | Replace with AWS RDS     |
| File System     | Shared SMB Storage | Medium       | Migrate to EFS           |
| Web Service     | Payment Gateway    | High         | Update endpoint configs  |
| API             | Salesforce CRM     | Medium       | Validate OAuth flow      |

**Infrastructure Dependencies:**
- Active Directory: Yes – used for authentication  
- DNS Requirements: customerportal.company.com  
- Certificate Dependencies: TLS cert from DigiCert  
- Monitoring Tools: SolarWinds, AppDynamics  

## 2.3 Security Assessment
**Current Security Controls:**  
- [x] Antivirus software (Symantec)  
- [x] Endpoint protection  
- [x] Network segmentation  
- [x] Access controls (AD groups)  
- [x] Encryption at rest (Oracle TDE)  
- [x] Encryption in transit (TLS 1.2)  

**Compliance Requirements:** PCI-DSS, ISO 27001  

## 2.4 Performance Baseline
**Current Performance Metrics:**
- CPU Utilization: 45% avg, 70% peak  
- Memory Utilization: 65% avg, 85% peak  
- Disk I/O: 10K IOPS peak  
- Network Throughput: 200 Mbps peak  
- Response Times: 250ms average under 1K users  

---

# 3. Target State Design

## 3.1 AWS Architecture
- **Target AWS Account:** 123456789012  
- **Target Region:** ap-southeast-2 (Sydney)  
- **Availability Zones:** ap-southeast-2a, 2b  

**Compute Resources:**

| Component         | AWS Service | Instance Type | Sizing Rationale           |
|-------------------|-------------|---------------|----------------------------|
| Application Server| EC2         | m6i.xlarge    | Matches CPU/memory needs   |
| Database          | RDS Oracle  | db.m6i.4xlarge| CPU/IO throughput required |

**Storage Design:**
- Root Volume: 100 GB gp3  
- Data Volumes: 500 GB gp3 for App, 2 TB io1 for DB  
- Backup Strategy: AWS Backup, daily snapshots, PITR enabled  

**Network Configuration:**
- VPC: vpc-0ab12345  
- Subnets: 10.20.0.0/24 (App), 10.20.1.0/24 (DB)  
- Security Groups: Allow HTTPS, Oracle within VPC only  
- Load Balancer: ALB for HTTPS traffic  

## 3.2 Security Design
**Security Controls:**  
- [x] IAM roles and policies defined  
- [x] Security groups configured  
- [x] NACLs configured  
- [x] EBS/RDS/S3 encryption enabled  
- [x] CloudTrail enabled  
- [x] GuardDuty enabled  
- [x] Session Manager enabled  

**Compliance Validation:**  
- [x] Security group rules reviewed  
- [x] Encryption standards (AES-256) met  
- [x] S3 access logging configured  
- [x] CloudWatch/GuardDuty alerts active  

---

# 4. Migration Plan

## 4.1 Migration Approach
- **Migration Method:** AWS MGN for servers, AWS DMS for DB  
- **Migration Window:** 8 hours (weekend cutover)  
- **Rollback Plan:** Failback to on-prem Oracle + DNS revert  

## 4.2 Pre-Migration Activities
- [x] Application discovery completed  
- [x] Performance baseline established  
- [x] Test plan approved  
- [x] Migration runbook created  
- [x] AWS resources provisioned  
- [x] Security validation completed  
- [x] Backup verification completed  
- [x] Stakeholder communication sent  

## 4.3 Migration Timeline
```mermaid
gantt
    title Migration Timeline
    dateFormat  YYYY-MM-DD
    section Pre-Migration
    Environment Setup :done,    des1, 2025-09-05, 3d
    Data Sync Initial :active,  des2, 2025-09-08, 2d
    section Migration
    Testing Preparation :        des3, 2025-09-10, 2d
    Final Data Sync     :        des4, 2025-09-12, 1d
    Application Cutover :        des5, 2025-09-13, 1d
    DNS Changes         :        des6, 2025-09-13, 1d
    section Post-Migration
    Smoke Testing       :        des7, 2025-09-14, 1d
    Performance Validation :     des8, 2025-09-15, 1d
    User Acceptance     :        des9, 2025-09-16, 3d
````

---

# 5. Testing Strategy

## 5.1 Test Plan Overview

* **Testing Approach:** Hybrid (automated + manual)
* **Test Environment:** Staging environment in AWS
* **Testing Tools:** JMeter, Selenium, AWS Inspector

## 5.2 Test Scenarios

| Test Type   | Description               | Acceptance Criteria          | Owner         |
| ----------- | ------------------------- | ---------------------------- | ------------- |
| Functional  | Customer login, billing   | 100% pass rate               | QA Lead       |
| Performance | 1K concurrent users       | ≤ 300ms response time        | Perf Engineer |
| Security    | IAM roles, SG rules       | No high-risk vulnerabilities | Security Lead |
| Integration | CRM API, Payment Gateway  | All integrations successful  | App Team      |
| DR          | Backup/restore validation | RTO ≤ 2h, RPO ≤ 15m          | DBA           |

## 5.3 Rollback Criteria

* [ ] Critical functionality failure
* [ ] Response times exceed 500ms
* [ ] Security vulnerabilities found
* [ ] Data corruption detected

---

# 6. Risk Assessment

## 6.1 Migration Risks

| Risk                       | Impact | Probability | Mitigation Strategy             | Owner     |
| -------------------------- | ------ | ----------- | ------------------------------- | --------- |
| Data loss during migration | High   | Low         | Multi-level backups + PITR      | DBA       |
| Extended downtime          | Medium | Medium      | Weekend cutover + rollback      | PM        |
| Performance issues         | Medium | Medium      | Load testing pre/post-migration | Perf Eng. |
| Integration failures       | High   | Low         | API mocks & staging validation  | App Lead  |

## 6.2 Business Continuity

* **Maximum Tolerable Downtime:** 8 hours
* **RTO:** 2 hours
* **RPO:** 15 minutes
* **Business Process Impact:** Billing delayed, customer access blocked

---

# 7. Resource Requirements

## 7.1 Team Structure

| Role                | Responsibility      | Resource      | Effort (Hours) |
| ------------------- | ------------------- | ------------- | -------------- |
| Migration Lead      | Overall execution   | Michael Chen  | 120            |
| System Admin        | Server config       | Anna Williams | 80             |
| DBA                 | DB migration        | Ravi Kumar    | 100            |
| Network Engineer    | Networking setup    | Tom Nguyen    | 60             |
| Security Specialist | Security validation | Priya Patel   | 70             |
| Application Owner   | Business validation | David Roberts | 40             |

## 7.2 Infrastructure Requirements

* **Development/Testing:** AWS sandbox, test data sets
* **Production:** AWS prod account, DNS changes, routing updates

---

# 8. Communication Plan

## 8.1 Stakeholder Matrix

| Stakeholder Group | Interest Level | Communication Method | Frequency  |
| ----------------- | -------------- | -------------------- | ---------- |
| Application Users | High           | Email notifications  | Pre/During |
| Business Owners   | High           | Status meetings      | Weekly     |
| Technical Teams   | High           | Teams/Slack updates  | Daily      |
| Management        | Medium         | Dashboard reports    | Weekly     |

## 8.2 Communication Schedule

| Milestone                   | Audience         | Message                      | Timeline  |
| --------------------------- | ---------------- | ---------------------------- | --------- |
| Migration Planning Complete | All stakeholders | Migration schedule confirmed | T-2 weeks |
| Pre-migration Testing       | Tech teams       | Test results readiness       | T-1 week  |
| Migration Start             | All users        | Maintenance window begins    | T-0       |
| Migration Complete          | All stakeholders | System restored, validated   | T+4 hrs   |

---

# 9. Post-Migration Activities

## 9.1 Validation Checklist

**Technical Validation:**

* [x] Services running
* [x] DB connectivity confirmed
* [x] Functional validation passed
* [x] Response time ≤ 300ms
* [x] Security controls active
* [x] Monitoring operational
* [x] Backup tested

**Business Validation:**

* [x] UAT signed-off
* [x] Billing workflows verified
* [x] Customer portal accessible

## 9.2 Optimization Activities

**Immediate (Week 1):** Right-sizing, security config review, cost analysis
**Short-term (Month 1):** Reserved instances, autoscaling, CloudWatch dashboards

## 9.3 Knowledge Transfer

* Architecture diagrams updated
* Ops runbook updated
* Troubleshooting guide provided
* Training delivered to ops team

---

# 10. Success Metrics and KPIs

## 10.1 Technical Metrics

| Metric             | Baseline | Target | Actual | Status |
| ------------------ | -------- | ------ | ------ | ------ |
| Application Uptime | 99.5%    | 99.9%  | 100%   | ✅ Met  |
| Response Time (ms) | 250      | 300    | 280    | ✅ Met  |
| CPU Utilization    | 70%      | <65%   | 60%    | ✅ Met  |
| Memory Utilization | 85%      | <75%   | 70%    | ✅ Met  |

## 10.2 Business Metrics

| Metric                | Baseline | Target | Actual | Status  |
| --------------------- | -------- | ------ | ------ | ------- |
| User Satisfaction     | 78%      | 90%    | 88%    | ⚠️ Near |
| Business Process Time | 4 hrs    | 2 hrs  | 2 hrs  | ✅ Met   |
| Error Rate            | 5%       | <2%    | 1.5%   | ✅ Met   |

## 10.3 Cost Metrics

* Migration Cost: \$150K (Actual: \$140K)
* Monthly Operating Cost: \$50K (Actual: \$45K)
* Cost Savings: \$20K/month (Actual: \$22K)

---

# 11. Lessons Learned and Recommendations

## 11.1 What Went Well

* Smooth cutover within 6 hours
* Automation reduced manual errors
* Strong team collaboration

## 11.2 Areas for Improvement

* CRM integration testing started too late
* Communication gaps during DNS cutover

## 11.3 Recommendations

* Start integration tests earlier
* Add automated DNS failover
* Allocate extra buffer in timeline

---

# 12. Sign-off and Closure

## 12.1 Migration Acceptance

| Role               | Name          | Signature | Date       | Comments       |
| ------------------ | ------------- | --------- | ---------- | -------------- |
| Application Owner  | David Roberts |           | 2025-09-17 |                |
| Technical Lead     | Michael Chen  |           | 2025-09-17 |                |
| Security Lead      | Priya Patel   |           | 2025-09-18 |                |
| Operations Manager | James Wong    |           | 2025-09-18 |                |
| Project Manager    | Sarah Johnson |           | 2025-09-18 | Final approval |

## 12.2 Project Closure Checklist

* [x] Deliverables completed
* [x] Knowledge transfer completed
* [x] Documentation updated
* [x] Lessons learned captured
* [x] Post-implementation review scheduled
* [x] Support transition completed
* [x] Project resources released

**Project Closure Date:** 2025-09-18
**Post-Implementation Review Date:** 2025-10-02

---

# Appendices

* **Appendix A:** Technical Specifications – AWS architecture diagrams
* **Appendix B:** Test Results – JMeter reports, UAT sign-off
* **Appendix C:** Security Validation – PCI-DSS scan results
* **Appendix D:** Runbook – Migration cutover steps
* **Appendix E:** Contact Information – Escalation list and emergency contacts

