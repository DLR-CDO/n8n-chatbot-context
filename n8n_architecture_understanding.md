N8n Architecture

# Overview

This document provides detailed information on each of the components created as part of the n8n architecture for the production setup. It covers networking, security, governance, and backup strategies.

# High‑Level Architecture

Architecture includes Azure VNet, Application Gateway (WAF), AKS with Managed Identity, Azure Key Vault for secrets, Azure Container Registry for n8n images, PostgreSQL with private endpoint, Persistent Volumes, and Azure Monitor.

![Architecture](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/Architecture.png)

## DNS Server:

- Users can access our app with a **friendly domain name** (e.g., n8n-dev.digitalrealty.com).
- DNS handles the translation → users don’t need to know or remember the backend service’s real domain/IP.

## Azure VNet (private)
Needed for security, scalability, reliability and compliance with dedicated subnets for:

- appgw (Application Gateway) –Azure requires app gateway to be in a dedicated subnet
- aks-nodepool (AKS nodes) –This is where n8n application lives. Apply **Network Security Groups** (NSGs) to only allow traffic from App Gateway subnet.
- private-endpoints (PostgreSQL) –No public internet access

## Azure Application Gateway (WAF)
WAF is integrated with AKS by manually setting the connection to AKS

- Handles incoming network traffic
- Provides TSL/SSL termination, routing, and WAF (Web Application Firewall –protects from SQL Injection) protection.
- Forwards requests securely to the AKS cluster hosting n8n.

## What is Termination? 
When a client (browser) makes an **HTTPS request**:

1. Browser starts a **TLS handshake** with the server.
2. They agree on encryption keys.
3. Secure channel established → HTTP data flows inside TLS.

**TLS/SSL termination** is where that handshake stops (i.e., where encrypted traffic is decrypted).

**TLS Termination:**
TLS is terminated at the Azure Application Gateway (WAF v2), which manages SSL certificates  The gateway offloads HTTPS and forwards HTTP traffic over a private IP to the AKS ingress controller inside the VNet.

**WAF Rulesets:**
The Application Gateway is configured with the OWASP 3.2 managed ruleset in Prevention mode, providing protection against common web vulnerabilities such as SQL injection, XSS, and protocol violations. Additional custom rules are defined for IP allowlisting and blocking specific request patterns.

**Pod Security on VNet:**
Pods run on a private AKS cluster with Azure CNI, NSGs, and network policies for secure intra-VNet traffic.

## Single Sign-On(SSO)
**SSO** is to validate the identity of the user (using Azure Entra ID, MFA)

## AKS 
AKS (private cluster) with Managed Identity and (recommended) OIDC/Workload Identity for pods.

Managed Identity is used to **pull images from ACR**, and for access control on other azure resources*

**What is a Managed Identity in AKS?**

- It’s an **Azure AD (Entra ID) identity automatically created** for your AKS cluster and/or node pool.
- This identity can be given RBAC roles on other Azure resources.
- In our architecture AKS managed identity is used to grant **AcrPull** on Azure container registry to pull n8n image - **acrdlrn8nprd.azurecr.io/n8n:stable**.

## Azure Key Vault
Azure Key vault is for managing the secrets.

## Azure Container Registry (ACR)
ACR will be hosting the n8n image (mirrored from n8n Docker Hub).

## Azure Database for PostgreSQL (PaaS) accessible via Private Endpoint.

### Why we chose stand-alone Azure Database for PostgreSQL instead of running it inside AKS

- **Reliability & Backups**: Azure DB gives us built-in automated backups and point-in-time restore. If AKS goes down, the database still survives.
- **High Availability**: Zone-redundant, managed failover is included. We don’t need to build and maintain HA ourselves.
- **Security & Governance**: Entra ID integration, private endpoints, and compliance features are available out-of-the-box.
- **Operational Simplicity**: Azure manages patching, scaling, and upgrades — freeing us from running Postgres ourselves.
- **Separation of Concerns**: AKS runs the workflows; Azure DB keeps the records safe. This reduces risk and makes upgrades or cluster rebuilds much easier.

## Persistent Volume Claim (PVC)
PVC is for n8n’s /home/node/.n8n folder.
- **Persists local data** (/home/node/.n8n) across pod restarts or upgrades.
- Stores **binary files**, **cached credentials**, and **runtime state** that workflows may depend on.
- Prevents data loss if the pod is rescheduled.
- Acts as a **safety net** for anything n8n doesn’t store in PostgreSQL.

In short:*** Postgres keeps workflows safe, PVC keeps local workflow files and state safe. Both are needed for reliable production use.***

## Azure Monitor / Log Analytics  for full observability.

## Traffic flow:
 User → App Gateway (WAF/TLS) → n8n service (ClusterIP) → n8n pods. DB and secrets are accessed privately via the VNet.

