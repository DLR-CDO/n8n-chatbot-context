# n8n Version Control

## 1. Purpose

The purpose of this document is to outline the version control strategy for **n8n**, ensuring that critical workflows, configurations, and credentials are safely managed using **Git**.

This document ensures:

- Business users understand how versioning preserves workflows.  
- Admins know how to implement Git integration, manage branches, and migrate workflows.  
- Developers understand how to commit, push, and track workflow changes.  
- Support teams have clear procedures for rollback and recovery of workflows.


## 2. Scope

- **In Scope:** Version control of workflows, credentials, and configurations for both development and production n8n environments.  
- **Out of Scope:** External system backups, manual workflow exports outside Git, and non-n8n system configurations.


## 3. Audience

- **Executives / High-level Users:** Understand risks, SLAs, and business impact.  
- **Admins:** Implement Git integration, manage branches, approve PRs, perform migration.  
- **Developers:** Commit, push, and test workflows, follow versioning rules.  
- **Users:** Awareness that workflows are version controlled and recoverable.


## 4. Summary

n8n is a workflow automation platform storing workflows, credentials, users, logs, and configurations in PostgreSQL.

Without proper version control:

- Workflow changes could be lost.  
- Conflicts in JSON workflows could break automation.  
- Inconsistent deployment between Dev and Prod environments could occur.

This strategy ensures consistent **Dev → Prod** workflow promotion with Git, maintaining clear version history, rollback capabilities, and defined branching rules.


## 6. n8n Version Control and Git Setup Guide

### 6.1 Overview

# Version Control

**Objects and Git saving**: n8n objects are not automatically saved to the Git repository.

**n8n Git integration**: Git integration works at the **instance level** and is managed by admins. A single Git repository is configured for the entire n8n deployment.
- Regular users/members can not connect their git repos to n8n instance.
- If you need to share the workflow with your team, you can export Json manually and share with them.

**Regular members**: Can save workflows in the n8n database but cannot connect their workflows to a personal Git repository from within n8n.

**Workaround**: A user could manually export a workflow JSON and commit it to their own repository outside of n8n, but this falls outside the built-in Git feature.

**Version control operations**:

- **Instance admins** → Can **push and pull**.
- **Project admins** → Can **push only** (pull is restricted).
- **Regular members** → Cannot push or pull.


**Why n8n restricts Git operations to admins**

- **Audience**: n8n is built for a wide range of users, including citizen developers and ops teams. Many of these users aren’t experienced with Git, especially when it comes to handling merge conflicts.
- **Data format**: n8n workflows are stored as JSON. Unlike source code, JSON doesn’t merge cleanly — conflicts are difficult to resolve and can easily break workflows.

For Admin users please see migration guide (section 6.4) in - [N8n-version-control.docx](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_versioncontrol/n8n_versioncontrol.md)

If you need Code Repository support, please reach out to **CdoPlatformSupport@digitalrealty.com**

# Troubleshooting and errors
- Please check the n8n Community forum for any similar errors or existing solutions: [https://community.n8n.io/](https://community.n8n.io/)

- If you notice something unusual in production, you can reach out to - [**CdoPlatformSupport@digitalrealty.com**](mailto:CdoPlatformSupport@digitalrealty.com)



### 6.2 GitHub Best Practices for n8n

- Treat workflows as code.  
- Commit changes frequently with **descriptive messages**.  
- Use **feature branches** for workflow migration and development.  
- **Dev instance** connected to *development branch*; **Prod instance** connected to *main branch*.  
- **PR-based promotion** ensures peer review before deployment.  
- **Tag releases** for key milestones.

### 6.3 Branching Strategy

***push from n8n-dev instance to --> Development Branch --> Feature branch --> Main branch --> pull from Main branch to Prod n8n instance(admin only activity)***

- **Development Branch:** For Dev instance workflows.  
- **Main Branch:** For Prod instance workflows (protected — merges require PR and approval).  
- **Feature Branches:** Used for workflow migration and isolated development.  
- **Commit Messages:** Clear, concise, and meaningful.

### Versioning Rules
- Never pull before pushing local changes.  
- Pull latest changes before starting new work.  
- All Dev workflows must exist in Git.  
- Prod workflows are pull-only; no direct edits.

### 6.4 Migration Guide (Admins Only)

1.	Clone the repository: ***git clone git@github.com:DLR-CDO/n8n-dev-repo.git***
2.	Switch to main branch: ***git checkout main***
3.	Pull latest: ***git pull origin main***
4.	Fetch all updates: ***git fetch origin***
5.	Ensure development branch exists: ***git rev-parse --verify development || git checkout -b development origin/development***
6.	Create a feature branch: 
  ***git checkout -b migration-{project_name} -YYYYMMDD*** 

      ***git rm -rf .***

7.	Add workflows or credentials from development branch:

      ***git checkout development -- workflows/{workflow_id}.json***

      ***git checkout development -- credential_stubs/{credential_id}.json***

8.	Stage files: ***git add .***
9.	Commit changes: ***git commit -m "{meaningful commit message}"***
10.	Push feature branch: ***git push origin migration-{project_name}-YYYYMMDD***

### 6.5 Promotion Pipelines
- Use GitHub PRs for promotion from Dev to Prod.
- Peer review required before merging.
- Rollback procedures documented in Git using previous commits/tags.

## 7. Roles, Responsibilities & Approvals

### 7.1 Role Responsibilities
| Role | Responsibilities |
|------|-------------------|
| **Instance Admin** | Configure Git integration, manage access, enforce branching policies, approve merges to main, handle rollback, and coordinate with GitHub for CI/CD integration. |
| **Project Admin** | Maintain development branches, validate workflows, create PRs for migration, and ensure feature completeness. |
| **General User** | Build and test workflows within the Dev environment. Submit n8n objects or change requests to project/Instance admin for migration.

### 7.2 Review and Approval Workflow (n8n Centralized Git Model)

In our n8n deployment, users and developers **do not have direct Git access** (push/pull privileges are restricted).  
All commits to the Git repository are handled by **Project Admins** or **Instance Admins** on behalf of developers.

#### **Workflow Promotion Process**

1. **Developer Stage**
   - Developer builds and tests the workflow in the **n8n Dev environment**.
   - Once validated, the developer shares the workflow with the **Project Admin** for review.

2. **Project Admin Review**
   - The Project Admin reviews workflow logic, naming standards, and error handling.
   - If the workflow meets requirements, the admin commits it to the **development branch** in Git.

3. **Migration to Main Branch**
   - The Project Admin (or Instance Admin) follows the **Migration Guide (Section 6.4)** to create a migration branch and push changes up to the **main** branch.
   - This step represents approval for the workflow to move toward production.

4. **Production Sync**
   - Once the workflow is in the **main branch**, an email should be sent to  
     [**CdoPlatformSupport@digitalrealty.com**](mailto:CdoPlatformSupport@digitalrealty.com)  
     requesting the **Prod instance** to pull the latest changes from main.

5. **Special Case – No Project Admin**
   - If a Project Admin is unavailable, developers can request the **Instance Admin** to perform the Git operations.
   - In such cases, developers must provide clear context, test results, and supporting evidence to give admins confidence in the workflow’s quality, since they may not have deep knowledge of the workflow’s logic.

This process ensures strict version control, prevents unauthorized commits, and maintains the Dev → Main → Prod promotion discipline.

No workflow moves to production without PR review and approval.

## 8. Git Operations & Hygiene

### 8.1 Reviewing and Pushing Tested Workflows
- Run and validate workflows in **n8n Dev** before committing.  
- Ensure no test or dummy data remains in workflow nodes.  
- Use the **JSON diff view** in GitHub to verify meaningful changes only (avoid credential deltas).

### 8.2 Naming, Branch Protection, and Tagging Standards
- **Branch naming:** `feature/{project_name}-{date}` or `migration/{project_name}-{date}`  
- **Commit messages:** `Add Salesforce_LeadSync workflow` or `Fix retry logic for SharePoint job`  
- **Tags:**  Attach version number to workflows `vMajor.Minor.Patch` (e.g., `v1.3.0`) for releases.  
- **Protected branches:** `main` and `development` are protected; direct commits are blocked.

## 9. Quality & Risk Management
### 9.1 PR Review Checklist

Before approving any PR to main, reviewers must check:

✅ Workflow naming aligns with convention

✅ No credentials or tokens in plain text

✅ Proper error handling (Try/Catch or IF logic where applicable)

✅ Performance (loops, delays, and paginations optimized)

✅ Governance: PII masked, Key Vault references used

✅ Dependencies and APIs tested successfully

### 9.2 Rollback Procedure

If a new workflow breaks production:

Identify the last working commit using Git history.

Run:

  ***git revert {commit_id}***
  ***git push origin main***


Notify the Release Manager and re-deploy previous container image if necessary.

If database changes are involved, coordinate with Admins for PITR (Point-In-Time Recovery).
