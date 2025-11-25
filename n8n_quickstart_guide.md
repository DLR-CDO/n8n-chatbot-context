# **n8n Quick Start Guide**

This guide is designed to help new users get started with n8n quickly and confidently.  
By the end, you will know how to:

- Request and access n8n environments  
 
For general n8n support, users can email to [**CdoPlatformSupport@digitalrealty.com**](mailto:CdoPlatformSupport@digitalrealty.com)

# Purpose

This guide helps new users quickly get started with the n8n automation platform. It provides step-by-step instructions for requesting access,
It ensures that:

- Platform usage remains consistent and auditable across environments.

# Summary

n8n is a workflow automation platform that allows users to integrate systems, automate repetitive tasks, and manage workflows efficiently. This guide provides step-by-step instructions for new users to get started,  backups, and version control integration.

Following this guide ensures that users can confidently create workflows, maintain them securely, and leverage n8n features effectively, minimizing downtime and errors.


# Getting Started with n8n

## 1.1 Requesting Access 

To request access to n8n, create a **ServiceNow ticket** using - https://interxion.service-now.com/ and ask to be added to the **AD/Entra group `N8N-DEV-SSO-Users`**, providing a brief justification for your project or role.  
Once approved, your access is provisioned automatically through Azure AD.

## 1.2 Accessing n8n

- **Development (Dev):** For building and testing workflows.  
- **Production (Prod):** Managed by platform admins for approved workflows only.  

### **Dev Environment**
You can log in to n8n at this link: https://n8n-dev.digitalrealty.com/

To Login, click “Continue with SSO”.  Username & password login is disabled for security compliance.

![accessing-n8n](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/LoginScreen.png)

If you have not already requested access, create a ServiceNow ticket requesting to be added to the AD/Entra group N8N-DEV-SSO-Users with a brief justification as to why you need access.

If your SSO login fails and you’ve confirmed that you’ve been added to AD/Entra group **N8N-DEV-SSO-Users**, then contact **CdoPlatformSupport@digitalrealty.com** or tag the Platform Admin (Ashutosh Anand) in your ServiceNow ticket for assistance.

### **Production Environment**
Access to the Production environment cannot be granted by default.

If there is an exceptional business requirement, please send an email to [**CdoPlatformSupport@digitalrealty.com**](mailto:CdoPlatformSupport@digitalrealty.com) with proper justification for review and approval.

Please note that the n8n Production instance is intended solely for deployment and monitoring purposes.
To ensure stability and prevent unintended changes, no user development or workflow modifications should be performed in the Production environment.

All development, testing, and configuration activities must be carried out in the n8n Development instance. Once workflows are finalized and approved, they will be deployed to Production through the designated process.

To maintain this structure, user access to the Production instance will be restricted. Only the admin and deployment team will have the necessary permissions for monitoring and deployment.

Users must always develop and validate workflows in **Dev** before requesting deployment to **Prod** to ensure consistency, version control, and stability.

## 1.3 n8n Roles

There are three account types:  
- **Owner:** Full administrative access  
- **Admin:** Can manage credentials, Git operations, and global settings  
- **Member:** Can create and edit workflows assigned to them  

![Roles&Permissions](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/Permissions.png)

Refer to the [official n8n RBAC documentation](https://docs.n8n.io/user-management/rbac/role-types/) for detailed permissions.


# References Documents
- Workflow creation guide - [WorkflowCreationHelp]()
- Credential management - [n8nCredential]()
- Comple n8n infrastructure setup - [N8n-infrastructure-setup.docx](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_infrastructure_setup/n8n_infrastructure_setup.md)
- Postgres backup & recovery - [n8n backup strategy.docx](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_backup_strategy/n8n_backup_strategy.md)
- N8n architecture overview - [n8n architecture understanding.docx](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_infrastructure_setup/n8n_architecture_understanding.md)
- N8n Git version control - [N8n-version-control.docx]([markdown/n8n_versioncontrol.md](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_versioncontrol/n8n_versioncontrol.md))

