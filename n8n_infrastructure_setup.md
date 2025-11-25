# Azure Deployment Guide for n8n Web Application (self-hosted)

# 1. Document Control

**Document Title:** Azure Deployment Guide for n8n Web Application (self-hosted)

**Version:** 1.0

**Author(s):** Ashutosh Anand, Tirupathirao Patnala

**Reviewed By:** Kevin Clarke, Abhishek Sirigi

**Approved By:** kevin Clarke, Abhishek Sirigi

**Date Created:** 16/09/2025

**Last Updated:** 23/09/2025

# 2. Purpose

This document provides step-by-step instructions for deploying a secure, self-hosted n8n workflow automation platform on Azure. The architecture ensures security through network isolation, scalability via AKS, and reliability using managed PostgreSQL, following Azure best practices for production workloads.

# 3. Scope
**In Scope:** Complete Azure infrastructure deployment (VNet, AKS, PostgreSQL, Application Gateway), n8n configuration, security setup, and version control.

**Out of Scope:** Application-level troubleshooting, custom node development, and third-party integrations.

# **4. Audience**

**Azure Administrators/DevOps Engineers:** Primary users who will execute the deployment steps and maintain the infrastructure.

**Security Teams:** Responsible for reviewing network security, access controls, and Key Vault configurations.

**Application Owners:** Need to understand the architecture and access patterns for n8n instance management.

**Database Administrators:** Involved in PostgreSQL configuration, backup, and performance monitoring.

**CI/CD Pipeline Developers:** May use this guide to automate future deployments.

# 5. Summary

This guide outlines the deployment of n8n - a workflow automation platform - on Azure using an enterprise-grade architecture. The solution leverages Azure Kubernetes Service (AKS) for container orchestration, Azure Database for PostgreSQL for data persistence, and Application Gateway for secure public access.

Key features of this deployment include:

- **Network Isolation:** All components reside within a dedicated Virtual Network with subnet segregation
- **Private Connectivity:** Database and internal services communicate through private endpoints
- **Secure Access:** Public access is controlled through Application Gateway with backend isolation
- **Credential Security:** Sensitive information is managed through Azure Key Vault with external secrets integration
- **Version Control:** All Kubernetes configurations are maintained in Git for auditability and reproducibility

The deployment follows a phased approach, starting with foundational Azure resources and progressing through container registry setup, database configuration, Kubernetes deployment, and security implementations.

# Phase 1: Foundation - Resource Group and Virtual Network

## Step 1.1: Create a Resource Group

1. In the [Azure Portal](https://portal.azure.com/), click "Create a resource".
2. Search for "Resource group" and click Create.
3. Basics Tab:
	- Subscription: Choose your subscription.
	- Resource group: n8n-rg
	- Region: West Europe or your preferred region.
4. Click Review \+ create, then Create.

## Step 1.2: Create a Virtual Network (VNet) with Subnets

1. Inside n8n-rg, click "Create a resource" -> Search for "Virtual network" -> Create.
2. Basics Tab:
	- Name: n8n-vnet
	- Region: *Same as your Resource Group.*
	- IPv4 address space: 10.10.0.0/16
3. IP Addresses Tab: Add these subnets:
	- Application Gateway Subnet: appgw-snet -> 10.10.0.0/24
	- AKS Subnet: aks-snet -> 10.10.1.0/24
	- Database Subnet: db-snet -> 10.10.2.0/24
4. Click Review \+ create, then Create.

# **Phase 2: Container Registry - Azure Container Registry (ACR)**

## **Step 2.1: Create the Container Registry**

1. In n8n-rg, click "Create a resource" -> Search for "Container Registry" -> Create.
2. Basics Tab:
	- Registry name: n8nacr (must be globally unique).
	- Location: *Same as your Resource Group.*
	- SKU: Premium (Required for VNet integration and enhanced throughput).
3. Click Review \+ create, then Create.

## Step 2.2: Pulling n8n Image into AKS (Controlled Versioning)

We want to control the exact n8n version deployed in AKS by pulling images from a private Azure Container Registry (ACR).

Import the required n8n version: Specify the resource name and version of the image in the following command

***az acr import --name {your_acr_resourcename} --source docker.io/n8nio/n8n:{image_version} --image n8n:{image_version}***

Update the n8n-deployment.yaml file to use the ACR image:

***image: {your_acr_resourcename}.azurecr.io/n8n:{image_version}***

Ensure AKS has a managed identity with the **AcrPull** role assigned on your ACR.

# Phase 3: Database - Azure Database for PostgreSQL

## **Step 3.1: Create PostgreSQL Flexible Server**

1. In n8n-rg, click "Create a resource" -> Search for "Azure Database for PostgreSQL" -> Select "Flexible Server" -> Create.
2. Basics Tab:
	- Resource group: n8n-rg
	- Server name: n8n-postgres-server (must be unique).
	- Region: *Same as your VNet.*
	- Admin username: n8nadmin
	- Password: *Choose a strong password and SAVE IT.*
3. Networking Tab:
	- Connectivity method: Private access (VNet Integration)
	- Virtual network: n8n-vnet
	- Subnet: db-snet
4. Click Review \+ create, then Create.

## **Step 3.2: Create the n8n Database**

1. Go to the n8n-postgres-server resource.
2. Under Settings, click Databases -> \+ Create.
3. Database name: n8n-db
4. Click OK.

# Phase 4: Kubernetes Cluster - AKS

## Step 4.1: Create the AKS Cluster & Integrate with ACR 

1. In n8n-rg, click "Create a resource" -> Search for "Kubernetes Service" -> Create.
2. Basics Tab:
	- Resource group: n8n-rg
	- Kubernetes cluster name: n8n-aks
	- Region: *Same as your VNet.*
	- Preset configuration: Standard
3. Node pools Tab:
	- Node size: Change to Standard B2s (dev) or Standard D2s v3 (production).
	- Node count: 2
4. Networking Tab (CRITICAL):
	- Network configuration: Azure CNI
	- Virtual network: n8n-vnet
	- Node subnet: aks-snet
5. Integrations Tab:
	- Container registry: Select the ACR you created (n8nacr). *This automatically grants the AKS cluster the necessary pull permissions.*
6. Click Review \+ create, then Create.


# Phase 5: Deployment & Configuration

## Step 5.1: Connect to Azure Kubernetes Service (AKS)

1. Navigate to the **AKS resource** created in the Azure Portal.
2. Ensure the cluster is **running**.
3. From the **Overview** page, click **Connect**.
4. Azure will attempt to execute the following commands automatically. If not, run them manually:

	*az account set --subscription subscription_id*

	*az aks get-credentials --resource-group {your_rg_name} --name {your_resource_name} --overwrite-existing*

	*kubelogin convert-kubeconfig -l azurecli*

## Step 5.2: Verify Git Installation

1. Check if Git is installed in the Azure CLI environment:

*git --version*

2. If Git is not installed, install it before proceeding.

## **Step 5.3: Clone the n8n Hosting Repository**

1. Clone the official **n8n hosting** repository:
2. *git clone git@github.com:n8n-io/n8n-hosting.git*

Or via HTTPS:

*git clone https://github.com/n8n-io/n8n-hosting.git*

## **Step 5.4: Create Kubernetes Namespace**

1. Navigate to the kubernetes folder inside the cloned repository.
2. Run the following command to create the namespace:

	*kubectl apply -f namespace.yaml*

This will create the namespace n8n.

## **Step 5.5: Create Persistent Volume Claim**

1. Apply the storage configuration:

	*kubectl apply -f n8n-claim0-persistentvolumeclaim.yaml*

This allocates persistent storage for n8n.

## **Step 5.6: Configure PostgreSQL Secrets**

1. Open the PostgreSQL secrets file for editing:
2. nano postgres-secret.yaml
3. Update it with the database values created in **Phase 3**:

	*POSTGRES_USER: <admin-username>*

	*POSTGRES_PASSWORD: <admin-password>*

	*POSTGRES_DB: <database-name>*

	*POSTGRES_NON_ROOT_USER: <n8n-user>*

	*POSTGRES_NON_ROOT_PASSWORD: <n8n-password>*

4. Save the file and apply the updated secrets:

	*kubectl apply -f postgres-secret.yaml*

## **Step 5.7: Configure n8n Deployment**

1. Open the deployment manifest for editing:

	*nano n8n-deployment.yaml*

2. Update the **image** field to pull from the Azure Container Registry (ACR) instead of the public Docker registry: - image: {your_acr_resourcename}.azurecr.io/n8n:stable

3. Add or update the following environment variables to configure n8n:

	*- name: DB_TYPE*

	*  value: postgresdb*

	*- name: DB_POSTGRESDB_HOST*

	*  value: pgsql-cdo-n8n-prd.postgres.database.azure.com*

	*- name: DB_POSTGRESDB_PORT*

	*  value: "5432"*

	*- name: DB_POSTGRESDB_DATABASE*

	*  value: n8n*

	*- name: DB_POSTGRESDB_USER*

	*  valueFrom:*

	*    secretKeyRef:*

	*      name: postgres-secret*

	*      key: POSTGRES_NON_ROOT_USER*

	*- name: DB_POSTGRESDB_PASSWORD*

	*  valueFrom:*

	*    secretKeyRef:*

	*      name: postgres-secret*

	*      key: POSTGRES_NON_ROOT_PASSWORD*

	*- name: DB_POSTGRESDB_SSL_ENABLED*

	*  value: "true"*

   * **n8n Application Configuration** 

	*- name: N8N_PROTOCOL*

	*  value: http*

	*- name: N8N_PORT*

	*  value: "5678"*

	*- name: N8N_SECURE_COOKIE*

	*  value: "false"*

	*- name: N8N_METRICS*

	*  value: "true"*

	*- name: N8N_TRUST_PROXY*

	*  value: "true"*

	*- name: N8N_PROXY_HOPS*

	*  value: "<interger depends on reverse-proxy>"*

	*- name: N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS*

	*  value: "true"*

	*- name: N8N_RUNNERS_ENABLED*

	*  value: "true"*

	*- name: N8N_TRUSTED_PROXIES*

	*  value: "0.0.0.0/0"*

	*- name: N8N_RATE_LIMIT*

	*  value: "false"*

	**Licensing**

	*- name: N8N_LICENSE_ACTIVATION_KEY*

	*  value: "<Your-License-Key>"*

	**Workflow Settings**

	*- name: N8N_WORKFLOW_ACTIVATION_BATCH_SIZE*

	*  value: "0"*

	**URLs**

	*- name: N8N_HOST*

	* value: n8n.digitalrealty.com*

	*- name: WEBHOOK_URL*

	*  value: https://n8n.digitalrealty.com*

	- name: N8N_EDITOR_BASE_URL*

	*  value: https://n8n.digitalrealty.com*

4. Save the changes to the file.
5. Run the following command to deploy n8n 

	*Kubectl apply -f n8n-deployment.yaml*

6. Run the following command to start n8n as Load Balancer

	*Kubectl apply -f n8n-service.yaml*

## Set a custom encryption key
n8n creates a random encryption key automatically on the first launch and saves it in the ~/.n8n folder. n8n uses that key to encrypt the credentials before they get saved to the database. If the key isn't yet in the settings file, you can set it using an environment variable, so that n8n uses your custom key instead of generating a new one.
export N8N_ENCRYPTION_KEY=<SOME RANDOM STRING>

Follow this [link](https://docs.n8n.io/hosting/configuration/environment-variables/deployment/) to know more about  environment variables.

## Step 5.9: Verify Deployment Status

1. Check whether the deployment and pods are running successfully:

	*kubectl get pods -n n8n*

2. The pods should display status Running.If they are in CrashLoopBackOff or Error, describe the pod for troubleshooting:

	*kubectl describe pod <pod-name> -n n8n*

	*kubectl logs <pod-name> -n n8n*

## **Step 5.10: Access n8n Service**

1. Get the **external IP** of the service:

	*kubectl get svc -n n8n*

	**Example output:**

	| NAME         | TYPE          | CLUSTER-IP   | EXTERNAL-IP   | PORT(S)       | AGE |
	|---------------|---------------|--------------|----------------|----------------|------|
	| n8n-service  | LoadBalancer  | 10.0.123.45  | 52.176.24.89  | 80:31234/TCP  | 2m  |

Here, the **EXTERNAL-IP** is 52.176.24.89.

2. Access n8n in a browser using the external IP - http://52.176.24.89.

At this point, n8n deployment should be **running on AKS** and accessible via the external IP provided by the LoadBalancer service.

# Phase 6: Setting Up Version Control for AKS Configuration Files

To ensure consistency, traceability, and collaboration, all Kubernetes configuration files should be maintained in a central Git repository. This allows for better change tracking, auditability, and rollback options in case of issues.

## Step 6.1: Central Repository for AKS Config Files

1. **Verify Git Installation**  
Ensure that Git is installed in your Azure CLI environment.
	git --version

2. **Set Up SSH Key for Repository Access**  
Configure an SSH key for SSO-based access to the DLR-CDO repository.

	*ssh-keygen -t rsa -b 4096 -C "your_email@example.com"*
	*eval "$(ssh-agent -s)"*
	*ssh-add ~/.ssh/id_rsa*

3. Add the public key (~/.ssh/id_rsa.pub) to your GitHub account under **SSH and GPG keys**.
   
4. **Clone the Repository**
	*git clone git@github.com:DLR-CDO/aks-cdo-n8n-prd.git*

5. **Navigate to the Kubernetes Directory** cd aks-cdo-n8n-prd/Kubernetes
6. **Modify the Required Configuration File**
	- Open the relevant YAML file(s).
	- Apply your required changes (e.g., updating deployments, services).
	- Save the file(s).
7. **Check the Status of Changes** - *git status*
	- If no changes are shown, add files manually:
	- git add <filename>      add a specific file
	- git add .               add all modified files
8. **Verify Staged Changes** - *git status*
9.  Confirm that your intended files are listed under **Changes to be committed**.
10. **Commit Your Changes** - *git commit -m "Describe your changes"*
11. **Push Changes to the Main Branch - ***git push origin main*

At this stage, AKS configuration files are safely version-controlled in the central repository. Anyone with access to DLR-CDO & AKS cluster can review, pull, and apply the latest changes consistently.

# Phase 7: Setting Up Azure Key Vault for Managing Credentials

Azure Key Vault is used to securely store and manage sensitive credentials and secrets, such as database usernames, passwords, and API keys, outside of application code or configuration files. It provides centralized secret management, controlled access, auditing, and automated secret rotation, reducing security risks and simplifying the use of credentials across applications and workflows.

Note: Any n8n configuration variables can be added in n8n-deployment.yaml file as a variable but if it's a secret to be secret to be used in credentials must be added to Azure key vault.
Example : 
## Step 7.1: Create an Azure Key Vault

1. Navigate to the Azure Portal → Create Resource → Key Vault → Create.
2. Provide the **Key Vault name**, **subscription**, **resource group**, **region**, and **pricing tier**.
3. Review and create the Key Vault.
4. Add the secrets to use in n8n credentials.

## Step 7.2: Setup external secrets in n8n

1. Create an app-registration and grant **Key vault secret reader** on key vault to the app
2. Navigate to settings/external secrets in n8n
3. Click on Azure KeyVault and fill the following 

KeyVault name: <KeyVault create in 8.1 section>

Tenant ID: <Your tenant id>

Client ID: <App registration client id>

Client Secret: <App registration client secret>


# Phase 8: Application Gateway - Application Gateway

## Step 8.1: Create a Public IP

1. In n8n-rg, click "Create a resource" -> Search for "Public IP address" -> Create.
2. Name: n8n-appgw-pip
3. SKU: Standard
4. Click Create.

## Step 8.2: Create the Application Gateway

1. In n8n-rg, click "Create a resource" -> Search for "Application Gateway" -> Create.
2. Basics Tab:
	- Name: n8n-appgw
	- Region: *Same as your VNet.*
	- Tier: Standard v2
	- Enable autoscaling: Yes
3. Frontends Tab: Select the public IP n8n-appgw-pip.
4. Backends Tab: Add a backend pool named n8n-aks-backend-pool without targets.
5. Configuration Tab: Add a routing rule:
	- Rule name: n8n-http-rule
	- Listener: n8n-http-listener (HTTP, Port 80)
	- Backend target: n8n-aks-backend-pool
	- HTTP settings: n8n-http-settings (Port 80, Override hostname: Pick from backend target)
6. Click Review \+ create, then **Create**.
7. Now, you can access the n8n application from http://{DNS_Name_of_public_IP}/.{region}.cloudapp.azure.com
8. If you don't like the domain provided by azure and want to keep it in your Organization domain, Request your network team to add a CNAME record for http://{DNS_Name_of_public_IP}/.{region}.cloudapp.azure.com to company DNS server
   Example: **Add a CNAME record for http://n8n-dev.southcentralus.cloudapp.azure.com in DigitalRealty domain to make it http://n8n-dev.digitalrealty.com**
9. Now, we will need to request for a SSL cert for the FQDN **n8n-dev.digitalrealty.com** to make it HTTPS.
10. Once you have the SSL Cert, Upload the .pfx file in certificates tab in listeners section.

## 9. Setup SSO, Admin Panel Restriction, and RBAC

n8n is integrated with **Microsoft Entra ID (Azure AD)** for authentication.

### SSO Configuration
1. Configure **OIDC** in n8n:
   ```env
   N8N_AUTH_BACKEND=oidc
   N8N_OIDC_CLIENT_ID=<entra-client-id>
   N8N_OIDC_CLIENT_SECRET=<entra-secret>
   N8N_OIDC_ISSUER_URL=https://login.microsoftonline.com/<tenant-id>/v2.0

2. Register the app in Azure AD → App Registrations.
3. Enable redirect URIs for your n8n domain (e.g., https://n8n-dev.digitalrealty.com/signin-oidc).
### Restricting Admin Access
- Disable local login (set N8N_BASIC_AUTH_ACTIVE=false).
- Limit Admin role assignment to designated personnel.
- Audit users monthly through n8n’s User Management UI.

### 9.2 Enabling Git Integration and Managing Access
Only Instance Admins can configure Git integration at the instance level.

**Setup**
- From n8n UI → Settings → Version Control
- Enter Git repository URL (e.g., git@github.com:DLR-CDO/n8n-dev-repo.git).
- Add SSH key or service credential with write permissions.

# Release Manager: Scaling, Monitoring & Environment Consistency

## 10.1 Deciding When to Scale or Move to Queue Mode (Redis)

Each n8n instance is sized based on typical workflow load.  
However, as automation volume grows, you may need to **scale replicas** or **switch to queue mode**.

### When to Add Pod Replicas
Increase pod replicas when:
- CPU usage exceeds **70–80%** for more than 15 minutes.
- Average memory usage stays above **75%**.
- Concurrent workflow executions exceed **configured concurrent capacity** (default 10).
- Scheduled workflows start overlapping or showing latency in triggers.

You can scale manually or use Horizontal Pod Autoscaler (HPA):

***kubectl scale deployment n8n-deployment --replicas=4***

***kubectl autoscale deployment n8n-deployment --cpu-percent=70 --min=2 --max=5***


### Ensuring Cross-Environment Consistency (Dev ↔ Prod)

Reproducibility is critical to ensure that Dev and Prod run identical container images.

#### Image Digest Pinning

Both environments must reference the same version of n8n from ACR image digest, not just the tag.
Admin/owner has to make sure they rollout version update to both environments one after the other to make sure it breaks nothing.

# Phase 11. Windows VM for Database Access**

Since PostgreSQL is deployed as a Private Endpoint, direct public access is blocked.
Use the Jump Host (Windows VM) to connect securely.

To securely connect to the PostgreSQL Flexible Server (which is only accessible inside the VNet), a Windows VM was provisioned in the same Virtual Network.  
  
Steps 9.1:  
1. In the Azure Portal, create a Virtual Machine.  
2. Resource Group: RG-CDO-N8N-PRD  
3. Name: VM-CDO-N8N-PRD  
4. Region: same as other resources  
5. Image: Windows Server 2022 Datacenter  
6. Administrator username and password: set securely  
7. Networking: Attach the VM to the existing VNet (VNET-CDO-N8N-PRD) and VM subnet (snet-vm)  
8. Allow inbound RDP (3389) to connect to the VM  
  
Once deployed, connect to the VM via RDP using the public IP. From inside the VM, use tools like psql to connect to the PostgreSQL database using its private IP address.

