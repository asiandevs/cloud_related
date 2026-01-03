# Creating Single-Instance Oracle Database on Exadata Cloud Service VM Clusters

[![Oracle](https://img.shields.io/badge/Oracle-F80000?style=for-the-badge&logo=oracle&logoColor=white)](https://www.oracle.com/)
[![Exadata](https://img.shields.io/badge/Exadata-Cloud-blue?style=for-the-badge)](https://www.oracle.com/cloud/)
[![Database](https://img.shields.io/badge/Database-26ai%20%7C%2019c-green?style=for-the-badge)](https://www.oracle.com/database/)
[![AI Database](https://img.shields.io/badge/AI_Database-26ai-orange?style=for-the-badge)](https://www.oracle.com/database/)

> A comprehensive guide to creating single-instance Oracle databases on Exadata Cloud Service VM clusters without RAC licensing requirements.

---

## 📑 Table of Contents

- [Introduction](#introduction)
  - [Overview](#overview)
  - [Supported Database Versions](#supported-database-versions)
  - [What You'll Deliver](#what-youll-deliver)
- [Prerequisites](#prerequisites)
  - [1. Oracle RDBMS Software Installation](#1-oracle-rdbms-software-installation)
  - [2. ASM Diskgroups Availability](#2-asm-diskgroups-availability)
  - [3. Database Template](#3-database-template)
  - [4. Gather Source Database Parameters](#4-gather-source-database-parameters)
- [Step-by-Step Database Creation Process](#step-by-step-database-creation-process)
  - [Step 1: Set Up the Environment](#step-1-set-up-the-environment)
  - [Step 2: Create Recovery Area Directory](#step-2-create-recovery-area-directory)
  - [Step 3: Execute Database Creation in Silent Mode](#step-3-execute-database-creation-in-silent-mode)
- [Post-Creation Configuration](#post-creation-configuration)
  - [Step 4: Modify Database Parameters](#step-4-modify-database-parameters)
  - [Step 5: Configure Redo Log Groups](#step-5-configure-redo-log-groups)
  - [Step 6: Configure Password File in ASM](#step-6-configure-password-file-in-asm)
- [Special Configuration: ArcGIS SDE Schema Requirements](#special-configuration-arcgis-sde-schema-requirements)
  - [Modify External Procedure Configuration](#modify-external-procedure-configuration)
  - [Deploy ArcSDE Binaries](#deploy-arcsde-binaries)
- [Verification Steps](#verification-steps)
- [Database Deletion (If Needed)](#database-deletion-if-needed)
- [Best Practices and Recommendations](#best-practices-and-recommendations)
- [Conclusion](#conclusion)
- [Additional Resources](#additional-resources)
- [Contributing](#contributing)
- [License](#license)

---

## Introduction

### Overview

Oracle Exadata Cloud Service offers a powerful capability that many organizations overlook: you can run single-instance databases in VM clusters on a single VM without requiring a RAC (Real Application Clusters) license. This provides significant cost savings while maintaining the robust infrastructure and performance benefits of Exadata.

This comprehensive guide walks you through the complete process of creating a single-instance Oracle database on an Exadata Cloud Service VM cluster, including all prerequisites, configuration steps, and post-creation tasks.

| **Attribute** | **Details** |
|---------------|-------------|
| **Estimated Execution Time** | Approximately 1 hour |
| **Target Audience** | Oracle Database Administrators working with Exadata Cloud Service |
| **Skill Level Required** | Basic to intermediate Oracle DBA knowledge |
| **Supported Database Versions** | Oracle AI Database 26ai, Oracle Database 19c, 12c R2, 12c R1, 11g R2 |

### Supported Database Versions

**Exadata Database Service on Dedicated Infrastructure** supports the following Oracle Database releases:

| Database Version | Status | Notes |
|-----------------|--------|-------|
| **Oracle AI Database 26ai** | ✅ Fully Supported | Latest AI-enhanced database with advanced features |
| **Oracle Database 19c** | ✅ Fully Supported | Current long-term support release (recommended) |
| **Oracle Database 12c R2 (12.2)** | ⚠️ Upgrade Support Required | Supported but requires upgrade path |
| **Oracle Database 12c R1 (12.1)** | ⚠️ Upgrade Support Required | Supported but requires upgrade path |
| **Oracle Database 11g R2 (11.2)** | ⚠️ Upgrade Support Required | Supported but requires upgrade path |

> 📝 **Important Notes:**
> - Earlier database versions are supported on a 19c cloud VM cluster and can be created at any time
> - Cloud VM clusters created with earlier Oracle Database versions will not automatically support Oracle Database 19c
> - For detailed release schedules, refer to My Oracle Support Note [742060.1](https://support.oracle.com/epmos/faces/DocContentDisplay?id=742060.1) - "Release Schedule of Current Database Releases"
> - This guide demonstrates database creation using Oracle Database 19c and Oracle AI Database 26ai examples

### What You'll Deliver

By the end of this process, you'll have:

- ✅ A fully functional single-instance Oracle database
- ✅ Service policy configuration
- ✅ Monitoring configuration
- ✅ Backup configuration
- ✅ First full backup of the database

[🔝 Back to top](#-table-of-contents)

---

## Prerequisites

### 1. Oracle RDBMS Software Installation

Before proceeding, ensure that the Oracle RDBMS software is installed on your target VM with the correct version and patch level. The Oracle Home should be properly configured and accessible.

**Supported Oracle Home Paths (Examples):**
```bash
# Oracle AI Database 26ai
/u01/app/oracle/product/26ai

# Oracle Database 19c
/u01/app/oracle/product/19c

# Oracle Database 12c R2
/u01/app/oracle/product/12.2.0.1
```

**Verify your installation:**
```bash
echo $ORACLE_HOME
$ORACLE_HOME/bin/sqlplus -v
```

### 2. ASM Diskgroups Availability

Ensure the following ASM diskgroup is available and has sufficient space:

- **+DATA** - For database files and datafiles

**Verify diskgroup availability:**
```bash
asmcmd lsdg
```

### 3. Database Template

Verify that the database template **New_Database.dbt** is available in the template directory:

```bash
ls -l $ORACLE_HOME/assistants/dbca/templates/New_Database.dbt
```

### 4. Gather Source Database Parameters

If you're migrating from an existing database, collect the following parameters from the source system:

```sql
-- Connect to source database
SQL> select * from NLS_DATABASE_PARAMETERS where parameter like '%CHARACTERSET%';
select * from NLS_DATABASE_PARAMETERS where parameter like '%LANG%';
select * from NLS_DATABASE_PARAMETERS where parameter like '%TERRIT%';
show parameter sga;
show parameter pga;
SHOW PARAMETER PROCESS
show parameter open_cur
```

> 📝 **Note:** Record these values as you'll need them to configure your new database to match the source environment.

[🔝 Back to top](#-table-of-contents)

---

## Step-by-Step Database Creation Process

### Step 1: Set Up the Environment

First, set your Oracle Home environment variable based on your database version:

**For Oracle AI Database 26ai:**
```bash
export ORACLE_HOME=/u01/app/oracle/product/26ai
```

**For Oracle Database 19c:**
```bash
export ORACLE_HOME=/u01/app/oracle/product/19c
```

**For Oracle Database 12c R2:**
```bash
export ORACLE_HOME=/u01/app/oracle/product/12.2.0.1
```

> ⚠️ **Important:** Replace the path with your actual Oracle Home location based on your installed version.

### Step 2: Create Recovery Area Directory

Create a dedicated directory for the Fast Recovery Area (FRA):

```bash
mkdir -p /singleaz_reco/dtuporawu02/FRA/GISACSE
```

> 📝 **Note:** Replace `GISACSE` with your actual database name and adjust the path according to your environment's directory structure.

### Step 3: Execute Database Creation in Silent Mode

Use the Database Configuration Assistant (DBCA) to create the database in silent mode. This approach is ideal for automation and repeatability.

**For Oracle AI Database 26ai:**
```bash
$ORACLE_HOME/bin/dbca -silent -createDatabase \
-storageType ASM \
-templateName New_Database.dbt \
-gdbName <DB NAME> \
-sid <DB NAME> \
-sysPassword <Password> \
-systemPassword <Password> \
-databaseConfigType SI \
-emConfiguration NONE \
-asmsnmpPassword <Password> \
-diskGroupName +DATA \
-datafileDestination +DATA \
-recoveryAreaDestination /singleaz_reco/dtuporawu02/FRA/GISACSE \
-characterSet WE8ISO8859P1 \
-nationalCharacterSet AL16UTF16 \
-databaseType MULTIPURPOSE \
-automaticMemoryManagement false \
-totalMemory 0 \
-redoLogFileSize 200 \
```

**For Oracle Database 19c:**
```bash
$ORACLE_HOME/bin/dbca -silent -createDatabase \
-storageType ASM \
-templateName New_Database.dbt \
-gdbName <DB NAME> \
-sid <DB NAME> \
-sysPassword <Password> \
-systemPassword <Password> \
-databaseConfigType SI \
-emConfiguration NONE \
-asmsnmpPassword <Password> \
-diskGroupName +DATA \
-datafileDestination +DATA \
-recoveryAreaDestination /singleaz_reco/dtuporawu02/FRA/GISACSE \
-characterSet WE8ISO8859P1 \
-nationalCharacterSet AL16UTF16 \
-databaseType MULTIPURPOSE \
-automaticMemoryManagement false \
-totalMemory 0 \
-redoLogFileSize 200 \
```

#### Key Parameters Explained

| Parameter | Description |
|-----------|-------------|
| `-databaseConfigType SI` | Specifies Single Instance configuration (no RAC) |
| `-storageType ASM` | Uses Automatic Storage Management |
| `-emConfiguration NONE` | Disables Enterprise Manager configuration |
| `-automaticMemoryManagement false` | Disables AMM (required for Exadata) |
| `-characterSet WE8ISO8859P1` | Set according to your requirements |
| `-redoLogFileSize 200` | Creates redo logs of 200MB each |

> 💡 **Version-Specific Notes:**
> - **Oracle AI Database 26ai**: Includes built-in AI and machine learning capabilities
> - **Oracle Database 19c**: Long-term support release, recommended for production
> - **Oracle Database 12c**: Ensure you have the latest patches applied

[🔝 Back to top](#-table-of-contents)

---

## Post-Creation Configuration

### Step 4: Modify Database Parameters

After the database is created, adjust the memory and system parameters to match your source database or performance requirements:

```sql
SQL> alter system set sga_target=1G;
alter system set sga_max_size=1G;
alter system set pga_aggregate_target=300M;

alter system set log_archive_dest_1='LOCATION=USE_DB_RECOVERY_FILE_DEST';
alter system set nls_language=AMERICAN,processes=550;
alter system set nls_territory=AMERICA;
alter system set open_cursors=5000;
alter system set use_large_pages=only;
alter system set os_authent_prefix='' scope=spfile;
alter system set db_recovery_file_dest_size=150G;
```

> ⚠️ **Important:** Adjust the memory values (SGA, PGA) based on your source database parameters collected in the prerequisites section.

### Step 5: Configure Redo Log Groups

Oracle best practices recommend having at least 5 redo log groups with 2 members each for optimal performance and protection.

**Check current log configuration:**

```sql
set line 200 pages 200
col member for a35
select g.group#,g.member,l.bytes/1024/1024 "Meg",l.status from v$log l,v$logfile g where l.group# = g.group# order by 1,2;
```

**Add additional log groups** (the template typically creates 3 groups):

```sql
alter database add logfile group 4 size 200M;
alter database add logfile group 5 size 200M;
```

### Step 6: Configure Password File in ASM

For proper database management, store the password file in ASM:

**Check current password file configuration:**
```bash
srvctl config database -d <DB NAME> | grep -i password
```

**Create the password file in ASM:**
```bash
orapwd file=+DATA/<DB NAME>/pw<DB NAME> dbuniquename=<DB NAME> format=12
```

> 🔑 When prompted, enter the SYS password.

**Update the database configuration:**
```bash
srvctl modify database -d <DB NAME> -pwfile +DATA/<DB NAME>/pw<DB NAME>
```

**Verify the configuration:**
```bash
srvctl config database -d <DB NAME> | grep -i password
```

[🔝 Back to top](#-table-of-contents)

---

## Special Configuration: ArcGIS SDE Schema Requirements

If you're using ArcGIS with SDE (Spatial Database Engine), you'll need to configure external procedures. This is a one-time configuration per server (or per Oracle Home if you have multiple homes).

> 📝 **Note:** These configurations apply to all supported database versions (26ai, 19c, 12c, 11g)

### Modify External Procedure Configuration

Edit the extproc.ora file (adjust path based on your Oracle Home version):

**For Oracle AI Database 26ai:**
```bash
vi /u01/app/oracle/product/26ai/hs/admin/extproc.ora
```

**For Oracle Database 19c:**
```bash
vi /u01/app/oracle/product/19c/hs/admin/extproc.ora
```

**For Oracle Database 12c:**
```bash
vi /u01/app/oracle/product/12.2.0.1/hs/admin/extproc.ora
```

Add the following line to the file:
```
SET EXTPROC_DLLS=ONLY:/u01/app/arcsde/sdeexe1081/lib/libst_shapelib.so
```

### Deploy ArcSDE Binaries

Transfer the ArcSDE software to your server:

```bash
# Archive on source
tar -czf arcsde.tar.gz /u01/app/arcsde

# Copy to target server and extract
tar -xzf arcsde.tar.gz -C /u01/app/
```

[🔝 Back to top](#-table-of-contents)

---

## Verification Steps

After completing the database creation and configuration, perform these verification steps:

### 1. Verify Database Status
```sql
SQL> select name, open_mode, database_role from v$database;
```

### 2. Verify Memory Configuration
```sql
SQL> show parameter sga
show parameter pga
```

### 3. Verify Log Groups
```sql
SQL> select group#, members, bytes/1024/1024 MB, status from v$log;
```

### 4. Verify ASM Files
```bash
asmcmd ls -l +DATA/<DB_NAME>
```

### 5. Check Database Services
```bash
srvctl status database -d <DB NAME>
```

[🔝 Back to top](#-table-of-contents)

---

## Database Deletion (If Needed)

If you need to remove the database, use the following command:

```bash
$ORACLE_HOME/bin/dbca -silent -deleteDatabase -sourceDB <DB NAME> -sysDBAUserName SYS -forceArchiveLogDeletion -sysDBAPassword <Password>
```

> ⚠️ **Warning:** This operation is irreversible. Ensure you have proper backups before proceeding with database deletion.

[🔝 Back to top](#-table-of-contents)

---

## Best Practices and Recommendations

| Practice | Recommendation |
|----------|----------------|
| **Memory Configuration** | Size your SGA and PGA appropriately based on your workload. The example shows 1GB SGA and 300MB PGA, but adjust these based on your requirements. |
| **Recovery Area Sizing** | Ensure your Fast Recovery Area is sized appropriately (150GB in this example) to accommodate your backup and archive log retention policies. |
| **Redo Log Sizing** | 200MB redo logs are suitable for many workloads, but high-transaction environments may benefit from larger logs to reduce log switches. |
| **Large Pages** | The `use_large_pages=only` setting improves memory management performance on Linux systems. |
| **Password Management** | Store passwords securely and follow your organization's password policies. |
| **Documentation** | Document all custom parameters and configurations for your database, especially deviations from defaults. |
| **Backup Strategy** | Implement and test your backup strategy immediately after database creation. |

[🔝 Back to top](#-table-of-contents)

---

## Conclusion

Creating a single-instance Oracle database on Exadata Cloud Service VM clusters provides an excellent balance of performance, manageability, and cost-effectiveness. By following this guide, you've successfully deployed a production-ready database environment without the complexity and licensing requirements of RAC.

The single-instance configuration on Exadata gives you access to Exadata's powerful storage, compression, and Smart Scan capabilities while maintaining a simpler architecture that's easier to manage for workloads that don't require the high availability features of RAC.

Remember to implement proper monitoring, backup, and maintenance procedures to ensure your database operates reliably in production. Regular patching and adherence to Oracle's recommended practices will help maintain optimal performance and security.

[🔝 Back to top](#-table-of-contents)

---

## Additional Resources

### Official Oracle Documentation

- [Exadata Database Service on Dedicated Infrastructure](https://docs.oracle.com/en-us/iaas/exadatacloud/doc/manage-databases.html)
- [Exadata Database Service FAQ](https://www.oracle.com/engineered-systems/exadata/database-service/faq/)
- [Oracle Database Administrator's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/)
- [Oracle ASM Administrator's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/)

### My Oracle Support Resources

- [Doc ID 742060.1](https://support.oracle.com/epmos/faces/DocContentDisplay?id=742060.1) - Release Schedule of Current Database Releases
- [Doc ID 2333222.1](https://support.oracle.com/epmos/faces/DocContentDisplay?id=2333222.1) - Additional Exadata Cloud Resources
- [My Oracle Support (MOS)](https://support.oracle.com/) - For patches and troubleshooting

### Database Version Information

| Version | Support Status | Documentation |
|---------|---------------|---------------|
| Oracle AI Database 26ai | Current | [26ai Documentation](https://docs.oracle.com/en/database/) |
| Oracle Database 19c | Long-Term Support | [19c Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/) |
| Oracle Database 12c | Upgrade Support | [12c Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/) |

[🔝 Back to top](#-table-of-contents)

---

## Contributing

Contributions are welcome! If you have suggestions for improvements or find any issues:

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## License

This documentation is provided as-is for educational and professional use.

---

**Author:** Oracle Database Expert  
**Last Updated:** January 2026  
**Version:** 2.0  
**Supported Database Versions:** Oracle AI Database 26ai, Oracle Database 19c, 12c R2, 12c R1, 11g R2

---

<div align="center">

**⭐ If you found this guide helpful, please consider giving it a star! ⭐**

</div>
