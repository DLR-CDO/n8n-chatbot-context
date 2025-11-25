# AKS Node Pool Resize Runbook

## Overview
This document is a step-by-step runbook for resizing an AKS node pool by creating new node pools with the desired VM SKU, migrating workloads, and removing old pools. Follow each step carefully and stop if you see unexpected results.

## When Node Pool Resizing Needed?
Here are some common reasons for resizing an AKS cluster node pool:
1. **Performance Optimisation**: Increasing the size of the nodes in the node pool can improve the performance of applications running in the AKS cluster. This is especially relevant if you notice resource constraints or performance issues caused by inadequate compute resources.
2. **Cost Optimisation**: Downscaling the node pool by reducing the VM size or the number of nodes can help optimise costs, especially if the cluster is consistently underutilised or if resource requirements have decreased.
3. **Application Scaling**: As your application workload grows, you may need to scale up the AKS cluster by adding larger VMs or additional nodes to accommodate the increased demand and maintain performance levels.
4. **Resource Requirements**: Changing the node pool size might be necessary if your application's resource requirements have changed.
5. **Workload Variation**: If your application experiences fluctuating workloads, you may want to consider automating the resizing of the node pool using tools like the Kubernetes Horizontal Pod Autoscaler (HPA) or the Azure Kubernetes Service (AKS) Cluster Autoscaler. These tools automatically adjust the number of nodes based on workload demands, ensuring optimal resource allocation.

## Summary (one line)
Create replacement node pool(s) with the target SKU, migrate workloads (cordon → drain or patch scheduling), validate, scale old pools to 0, then delete.

## Prerequisites
- az CLI and kubectl installed and logged in to the target subscription.
- kubectl context pointed to the correct AKS cluster.
- Azure RBAC or subscription rights to manage AKS node pools and VMSS.
- Knowledge of which workloads use node-local storage (hostPath/local PVs) or Azure Disks (RWO).

**Commands to confirm basics:**

*az account show --query id -o tsv* 
*kubectl config current-context*
*kubectl get nodes --show-labels*

## Step 0 — Plan
Identify node pools to replace (az aks nodepool list).

Note current pool modes (System vs User), VM SKU, zones, and min/max counts.

Confirm PVs (PVCs) and whether they are RWO (Azure Disk) or RWX (Azure File). If RWO, accept single-node mounting or plan data migration to RWX.

**commands**
*az aks nodepool list -g {RG} -n {CLUSTER} -o table*
*kubectl get pvc --all-namespaces -o wide*
*kubectl get storageclass -o wide*

## Step 1 — Create replacement nodepool(s)
Create one replacement pool for each role (system/user) you are replacing. Match required labels and mode.

**Example User pool**

*az aks nodepool add \
  --resource-group {RG} \
  --cluster-name {CLUSTER} \
  --name userpool1 \
  --node-count 2 \
  --node-vm-size Standard_D2ads_v5 \
  --mode User \
  --labels purpose=n8n-test*

For System pools, set --mode System if replacing the only system pool.

Wait for nodes to reach Ready:

*kubectl get nodes -l kubernetes.azure.com/agentpool=userpool1 -o wide*

## Step 2 — Prepare workloads for rescheduling (safe options)
Choose one approach:

A. Patch Deployments to target new pool (zero-downtime if 1 replica) - Add nodeSelector or nodeAffinity to deployment spec and rely on rolling update.

B. Cordon & drain nodes (controlled move) - Cordon node(s) to stop new scheduling. - Drain node(s) one-by-one to evict pods and allow them to reschedule on the new pool.

Commands (cordon & drain):

*kubectl cordon {node-name}*

*kubectl drain {node-name} --ignore-daemonsets --delete-local-data --grace-period=60*

Notes: - If kubectl drain fails due to PodDisruptionBudgets (PDBs), either relax PDBs temporarily or perform controlled scaling of replicas. - If PVCs are RWO (Azure Disk), you cannot run multiple replicas across nodes that require the same disk. Plan accordingly.

## Step 3 — Execute rollout and validate
If you patched deployments, monitor rollout:

kubectl rollout status deployment/{name} -n {ns} --timeout=10m

kubectl get pods -n {ns} -o wide
Validate:
- All critical pods Running and Ready.
- System pods (kube-system) present on new system pool (if replaced).
- PVs attached and Billing expectations met.
- Application health checks and end-to-end smoke tests.
Commands:

*kubectl get pods --all-namespaces -o wide*

*kubectl get pvc -n {ns}*

*kubectl describe pod {pod} -n {ns}*

Keep the cluster running on new pools for a validation window (24–72 hours is typical).

## Step 4 — Scale old nodepools to zero (reversible)
If old pools have min-count } 0, update it first and then scale to 0.

*az aks nodepool update -g {RG} --cluster-name {CLUSTER} --name {oldpool} --min-count 0*

*az aks nodepool scale -g {RG} --cluster-name {CLUSTER} --name {oldpool} --node-count 0*

Verify nodes are gone:

*kubectl get nodes*

## Step 5 — Delete old nodepools (final)
After validation window and backups/snapshots:

*az aks nodepool delete -g {RG} --cluster-name {CLUSTER} --name {oldpool}*

## Safety & Edge Cases (brief)
- **AzureDisk (RWO):** cannot be attached to multiple nodes simultaneously—either run single replica or migrate to RWX (AzureFile) for HA.
- **Ephemeral OS disks:** target SKU must support ephemeral OS disk sizes; otherwise create pool with managed OS disks.
- **DaemonSets:** drain ignores DaemonSets; ensure critical DaemonSets run on new nodes.
- **PDBs:** can block eviction—plan to temporarily relax when safe.
- **Zones:** ensure new nodepool spans required zones if PVs are zonal.

## Rollback plan (if things fail)
1.	Revert any nodeSelector/nodeAffinity patches.
2.	Scale back old nodepool from 0 to previous node count:

    ***az aks nodepool scale -g {RG} --cluster-name {CLUSTER} --name {oldpool} --node-count {previous-count}***

3.	kubectl rollout undo deployment/{name} -n {ns} for application rollbacks.


## Quick checklist (copy-paste) before deleting pools
- All workloads running on new nodepools
- kubectl get pods --all-namespaces shows no pods on old nodes
- PVCs attached & PVs healthy
- System pods running on new system pool
- Snapshots created for any managed OS disks you need to retain


## Useful commands (one-liners)
### list nodepools

***az aks nodepool list -g {RG} --cluster-name {CLUSTER} -o table***

###  add nodepool
***az aks nodepool add -g {RG} --cluster-name {CLUSTER} --name userpool1 --node-count 2 --node-vm-size Standard_D2ads_v5 --mode User***

### cordon all nodes in a pool

***kubectl get nodes -l kubernetes.azure.com/agentpool={oldpool} -o name | xargs -n1 kubectl cordon***

### drain all nodes in a pool

***kubectl get nodes -l kubernetes.azure.com/agentpool={oldpool} -o name | xargs -n1 kubectl drain --ignore-daemonsets --delete-emptydir-data --grace-period=60***

### scale nodepool to zero

***az aks nodepool update -g {RG} --cluster-name {CLUSTER} --name {oldpool} --min-count 0***
***az aks nodepool scale -g {RG} --cluster-name {CLUSTER} --name {oldpool} --node-count 0***

### delete nodepool

***az aks nodepool delete -g {RG} --cluster-name {CLUSTER} --name {oldpool}***
