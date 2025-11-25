# Backup & Recovery Strategy for n8n Development & Production 


# 1. Purpose

The purpose of this document is to outline the backup and recovery strategy for n8n, ensuring that critical workflows, configurations, and credentials are protected against failures, human errors, or security incidents.  
  
This document ensures:

- __Business users__ understand why backups are critical.
- __Admins__ know how to perform and verify backups.
- __Developers__ know how workflows/configs are preserved.
- __Support teams__ have a clear recovery plan.

# 2. Scope

__In Scope:__ Backup and recovery of databases, encryption keys, workflow configurations, custom code, and infrastructure components for both development and production n8n environments.

__Out of Scope:__ External system backups (e.g., third-party APIs, external data stores not managed by n8n).

# 3. Audience

- __Executives / High-level Users:__ Understand risks, SLAs, and business impact.
- __Admins:__ Implement and monitor backup jobs, restore on failure.
- __Developers:__ Track workflow versioning and custom node preservation.
- __Citizen Users:__ Be aware that their workflows are protected and recoverable.

# 4. Summary

n8n is a workflow automation platform that powers integrations across systems. It stores workflows, credentials, users, logs, and configurations in PostgreSQL and Kubernetes volumes.  
  
Without proper backups:

- Workflows could be lost.
- Credentials would be unrecoverable without the encryption key.
- Business automation could stop, leading to downtime and data loss.

This strategy ensures that we can recover quickly from failures, with defined RPO (data loss tolerance) and RTO (downtime tolerance) for both dev and production environments.

# 5. Backup Strategy Overview

## Workflow Backups & Recovery

While we maintain daily backups of the Postgres database and version control system for disaster recovery, users are strongly recommended to create personal backups of their workflows for additional security and portability.

### Why Backup Your Workflows?

1. Protection against accidental deletion or modification
2. Easy migration between n8n instances
3. Version control for your own workflow iterations
4. Quick recovery from workflow corruption
5. Ability to share workflows with team members

### Backup & Restore Workflows 

**Export Single Workflow (JSON)**

1. Open your workflow in the n8n editor
2. Click the workflow menu (three dots in the top-right corner)
3. Select "Download" from the dropdown menu
4. Choose your export format: 
	- **JSON** (recommended) - Contains complete workflow structure
5. Save the file to your local computer with version numbers or dates for better organization.

**Restoring a Single Workflow**

1. Go to Workflows page in n8n
2. Click "Import from File" or "+" to create new workflow
3. Select "Import from File" option
4. Browse and select your saved JSON file
5. Click "Import"
6. Review imported workflow and test functionality
7. Save the workflow once confirmed working

See “n8n backup strategy.docx” for n8n Enterprise backup and disaster recovery strategy.

## Postgres Database Backup & Recovery

Key Components to Protect: __*(admin will take care of this)*__

- __Database (PostgreSQL):__ Workflows, credentials, logs, users.
- __Encryption Key (N8N_ENCRYPTION_KEY):__ Required to decrypt credentials.
- __Code & Config:__ Helm charts, manifests, custom nodes.
- __Persistent Volume Claims (PVCs):__ Local cache, binary data files.

High-Level Backup Approach:

__Dev:__ pg_dump + Blob storage backups, PVC snapshots, GitOps versioning.  
__Prod:__ Azure-managed backups (daily + WAL for PITR), plus logical dumps for portability, PVC snapshots, GitOps versioning

# 6. Roles & Responsibilities

| Component | What It Stores | Who Owns Backup | How It’s Protected | Notes |
|------------|----------------|-----------------|--------------------|--------|
| **Azure PostgreSQL (PaaS)** | Workflows, execution logs, user accounts, credential metadata (encrypted) | Prod: Azure<br>Dev: Support Team | Automated daily full backups, WAL for PITR (up to 35 days), optional geo-redundant backups | No custom backup jobs needed unless you want logical dumps (`pg_dump`) for archiving/audit |
| **AKS PVC (/home/node/.n8n)** | Local workflow data, cached credentials, binary file storage (if not using external Blob/S3) | Support team | AKS Persistent Volume Snapshots, or Velero backups to Blob | Back up at least daily if you store binary data in n8n |
| **N8N_ENCRYPTION_KEY (and API keys)** | Encryption key for secrets, external system credentials | Support team | Store in Azure Key Vault with soft-delete + purge protection; back up vault to storage | **Critical** — without this, DB secrets can’t be decrypted |
| **Kubernetes Manifests / Helm values** | Deployment config (services, ingress, scaling policies, env vars) | Support team | Git repo (GitOps / Terraform / Bicep), versioned & backed up | This is your "blueprint" for rebuilding the cluster |
| **App Gateway & Networking Config** | WAF rules, TLS bindings, routing | Azure + Support team | Infra as Code (Bicep/Terraform) + Azure Policy backups | Keep definitions in Git to quickly recreate infra |
| **Monitoring / Logs** | Audit logs, execution logs, metrics | Azure | Azure Monitor / Log Analytics with retention policies | Export critical logs to storage account for long-term compliance, alerts for backup failures |
| **Code Version Control** | Workflows, credential JSON | Support team | Git repo | Version controlling, ability to rollback to any version |

# 7. Backup Strategy Detail (notes for Admin and Support Team)

## Dev Environment (AKS-hosted PostgreSQL + n8n) 

1. __Database Backup__   
- Use pg_dump or pgBackRest for logical backups (data and metadata)   
- Store backups in Azure Blob Storage/VM (via azcopy or rclone)   
- Automate with Kubernetes CronJob/Windows task scheduler 
2. __Database Restore__   
- Use pg_restore with dump files 
3. __Encryption Key Backup__   
- Store N8N_ENCRYPTION_KEY in Azure Key Vault   
- Keep a secure copy (need to think on where to store other than Azure Key Vault) 
4. __Code & Config Backup__   
- GitOps: workflows JSON exports, Helm charts, YAML manifests   
- Backup custom nodes/config from AKS volumes 
5. __Persistent Volume Claim Backup (stores user cache and data files that are used in workflows)–__ This needs to be daily backup to a storage account 

__Recovery Goal:__ Redeploy n8n, restore DB, set N8N_ENCRYPTION_KEY. __*Admin to test this entire setup and make sure this approach works*__   
 

## Production Environment (Standalone PostgreSQL + n8n) 

1. __Database Backup__ - As we use Azure SQL database for Postgresql, Azure already provides: 

	__Automatic backups__ 

	Full daily backups, __Retention__: Configurable (7–35 days). __– *Admin Team will check this*__

	Add __daily/weekly pg_dump exports to Blob Storage__ for portability (e.g., restore just one DB/table, or keep longer retention).   
 

2. __Encryption Key__   
- Store in Azure Key Vault with rotation policy   
- Keep offline secure copy 
3. __Code & Config Backup__   
- Git-based workflow export and version control   
- Backup n8n config + custom nodes 

Rest all backups are same as dev backup strategy. 

__Recovery Goal:__ Restore from PITR or dumps, redeploy manifests, reapply encryption key. __*Admin to test this entire setup and make sure this approach works*__ 

# 8. Governance & Compliance

- __Data Security:__ All sensitive keys stored in Key Vault with purge protection.
- __Auditability:__ Backups logged in monitoring systems.
- __Retention Policies:__ Dev = daily backups, Prod = 7–35 days + optional long-term.
- __Compliance:__ Aligns with org security and Azure SLAs.

# 9. Service SLAs – Uptime, Backup, and Recovery Expectations	

## Development Environment (AKS-hosted PostgreSQL + n8n)

- __Uptime:__ Best effort, no formal uptime guarantee. Expected to tolerate occasional outages for testing and upgrades.
- __Backups:__ Covered in the section__ “__Backup Strategy Detail”
- __Recovery Objectives:__
	- __RPO (Recovery Point Objective):__ ≤ 24 hours (based on daily backup cadence).
	- __RTO (Recovery Time Objective):__ ≤ 4–8 hours (redeploy AKS workloads, restore DB, reapply manifests).

## Production Environment (Azure PostgreSQL + n8n)

- __Uptime:__ Target availability aligned with Azure PostgreSQL SLA (≥ 99.99%). n8n application uptime targeted at ≥ 99.5% with Kubernetes HA deployment.
- __Backups:__ Covered in the section__ “__Backup Strategy Detail”
- __Recovery Objectives:__
	- __RPO:__ ≤ 24 hours (based on Azure Managed Daily backup).
	- __RTO:__ ≤ 4–8 hours (restore DB, redeploy n8n, reapply manifests/configs).

## Response Timelines for Issues in Production

- __Critical Severity (Prod Down / Major Data Loss):__ Response within __30 minutes__; mitigation or failover initiated immediately.
- __High Severity (Degraded Service, Backups Failing):__ Response within __2 hours__, resolution or workaround within 24 hours.
- __Medium Severity (Minor Issues, Non-critical Data Loss in Dev/Non-Prod):__ Response within __2 business days.__
- __Low Severity (Cosmetic/Feature Gaps):__ Response within __2–3 business days__.

# 10. Glossary

- __RPO (Recovery Point Objective):__ Max acceptable data loss.

- __RTO (Recovery Time Objective):__ Max acceptable downtime.

- __PVC (Persistent Volume Claim):__ Kubernetes storage for n8n data.

