# Senior Azure DevOps Engineer – Interview Preparation Q&A

**Role:** Senior Azure DevOps Engineer  
**Focus:** Azure | Azure DevOps | Terraform | Bicep | AKS | Kubernetes | Helm | GitOps | Networking | Security | Monitoring | CI/CD

> **Interview strategy:** For senior-level questions, answer in this order:  
> **What it is → Why we use it → How I implemented it → Troubleshooting → Security/production considerations.**

---

## Table of Contents

1. [Azure DevOps Pipelines](#1-azure-devops-pipelines)
2. [Terraform and Infrastructure as Code](#2-terraform-and-infrastructure-as-code)
3. [AKS and Kubernetes](#3-aks-and-kubernetes)
4. [Azure Networking](#4-azure-networking)
5. [Azure Identity, RBAC and Security](#5-azure-identity-rbac-and-security)
6. [Monitoring and Observability](#6-monitoring-and-observability)
7. [Production and Incident Management](#7-production-and-incident-management)
8. [Team and Behavioral Questions](#8-team-and-behavioral-questions)
9. [Databricks and Azure Services](#9-databricks-and-azure-services)
10. [60-Second Introduction](#10-60-second-introduction)
11. [Last-Minute Priority Topics](#11-last-minute-priority-topics)

---

# 1. Azure DevOps Pipelines

## 1.1 How do you trigger an Azure DevOps pipeline?

There are several ways:

- **CI trigger** – automatically when code is pushed to a configured branch.
- **PR trigger** – when a Pull Request is created or updated.
- **Scheduled trigger** – pipeline runs at a defined time.
- **Manual trigger** – user selects **Run pipeline**.
- **Pipeline completion trigger** – one pipeline triggers another.
- **REST API / Azure CLI** – pipeline can be triggered programmatically.

Example:

```yaml
trigger:
  branches:
    include:
      - main

pr:
  branches:
    include:
      - main
```

---

## 1.2 How do you define and use approval gates in Azure DevOps?

I use **Azure DevOps Environments → Approvals and Checks**.

For example:

```text
Build
  ↓
DEV
  ↓
UAT
  ↓
Production Environment
  ↓
Approval
  ↓
Production Deployment
```

Checks can include:

- Manual approval
- Branch control
- Business-hours checks
- Azure Monitor checks
- REST/API checks
- Required reviewers

For production, I prefer Environment-level approvals/checks because they provide stronger governance than relying only on a YAML approval task.

---

## 1.3 Parameters vs Variables in Azure DevOps

| Parameters | Variables |
|---|---|
| Control pipeline behavior | Store values used by pipeline |
| Evaluated during template expansion | Commonly available during runtime |
| Strongly typed | Generally strings |
| Useful for selecting environments/options | Useful for image tags/configuration |
| Not intended for secrets | Can contain secret variables |

Example parameter:

```yaml
parameters:
- name: environment
  type: string
  default: dev
```

Example variable:

```yaml
variables:
  imageTag: $(Build.BuildId)
```

### Easy interview answer

> **Parameter controls what the pipeline does; variable controls the values the pipeline uses.**

---

## 1.4 If a pipeline fails at a particular stage, how do you troubleshoot?

I follow a systematic approach:

```text
Identify failed stage
      ↓
Identify failed task
      ↓
Read logs/error
      ↓
Check inputs/configuration
      ↓
Check dependencies
      ↓
Check permissions/service connections
      ↓
Check variables/secrets
      ↓
Check network/target resources
      ↓
Fix root cause
      ↓
Re-run
```

I check:

- Pipeline logs
- Task error messages
- Service connection
- Permissions
- Variables/parameters
- Secrets
- Agent availability
- Network connectivity
- Azure resource status
- Artifact/image availability

I avoid repeatedly rerunning the pipeline without identifying the root cause.

---

## 1.5 If production deployment fails, what approach do you follow?

My first priority is **service availability and customer impact**.

```text
Detect
  ↓
Assess impact
  ↓
Stop further deployment
  ↓
Check logs/metrics
  ↓
Rollback or forward-fix
  ↓
Validate service
  ↓
Communicate
  ↓
RCA
  ↓
Prevent recurrence
```

If the deployment caused the issue and rollback is safe, I roll back to the previous known-good version.

After recovery:

- Perform RCA.
- Document the issue.
- Identify preventive actions.
- Improve monitoring/tests/deployment controls.

---

## 1.6 How do you configure a pipeline to trigger only for one branch?

For example, only `main`:

```yaml
trigger:
  branches:
    include:
      - main
```

If PR validation is required, configure the `pr:` section separately.

---

## 1.7 How do you ensure subsequent stages don't execute if one stage fails?

I use stage dependencies and conditions.

```yaml
- stage: Deploy
  dependsOn: Build
  condition: succeeded()
```

Flow:

```text
Build
  ↓
Test
  ↓
Security
  ↓
Deploy
```

If Security fails:

```text
Security ❌
   ↓
Deploy ⛔
```

---

## 1.8 Can the same pipeline template be reused across multiple environments?

**Yes.**

I create reusable YAML templates and pass environment-specific parameters.

Example:

```text
templates/
├── build.yml
├── security.yml
└── deploy.yml
```

Usage:

```yaml
- template: templates/deploy.yml
  parameters:
    environment: dev
```

The same template can be reused for:

```text
DEV
UAT
PROD
```

with different parameters, variable groups, values files, namespaces, subscriptions, etc.

---

## 1.9 How do you promote code from one environment to another?

I prefer **Build Once, Deploy Many**.

```text
Build
  ↓
Image/Artifact
  ↓
DEV
  ↓
UAT
  ↓
PROD
```

I promote the **same immutable artifact/container image** instead of rebuilding separately for every environment.

Example:

```text
myapp:1.2.35
```

The exact same image is tested in UAT and deployed to Production.

---

## 1.10 How do you manage secrets in Azure DevOps Pipelines?

Preferred approach:

```text
Azure Key Vault
      ↓
ADO Pipeline / Variable Group
      ↓
Pipeline
```

I use:

- Azure Key Vault
- Secret variables
- Variable groups
- Workload Identity / federated authentication where appropriate

I never hardcode passwords, client secrets or certificates in YAML or Git.

---

## 1.11 How do you add PR approvers?

In Azure DevOps:

```text
Repos
  ↓
Branches
  ↓
main
  ↓
Branch Policies
  ↓
Required Reviewers
```

Configure:

- Minimum number of reviewers
- Required reviewer teams
- Build validation
- Comment resolution
- Work item linking

Typical flow:

```text
Developer
   ↓
Feature Branch
   ↓
Pull Request
   ↓
Required Reviewers
   ↓
Build Validation
   ↓
Merge to main
```

---

## 1.12 How do you add approvals after a developer pushes code?

Use **Branch Policies** for PR approval before merge.

For example:

```text
Feature Branch
     ↓
Developer creates PR
     ↓
Required reviewer approval
     ↓
Build validation
     ↓
Merge to main
```

For deployment approvals:

```text
main
 ↓
Build
 ↓
DEV
 ↓
UAT
 ↓
Production Environment
 ↓
Environment Approval
 ↓
Production
```

---

## 1.13 Explain your CI/CD pipeline stage by stage

A strong AKS-oriented flow:

```text
Developer
   ↓
Feature Branch
   ↓
Pull Request
   ↓
CI
   ├── Checkout
   ├── Maven Build
   ├── Unit Tests
   ├── SonarQube
   ├── SAST
   ├── Docker Build
   ├── Trivy Scan
   └── Push Image to ACR
          ↓
       Artifact/Image
          ↓
       DEV Deployment
          ↓
        Approval
          ↓
       UAT Deployment
          ↓
        Approval
          ↓
       Production
          ↓
   Health Check/Monitoring
```

---

# 2. Terraform and Infrastructure as Code

## 2.1 How do you create an Azure VM using Terraform?

A simplified example:

```hcl
resource "azurerm_resource_group" "rg" {
  name     = "rg-vm-prod"
  location = "East US"
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "vm-prod-01"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  size                = "Standard_D4s_v5"
  admin_username      = "azureadmin"

  network_interface_ids = [
    azurerm_network_interface.nic.id
  ]

  admin_ssh_key {
    username   = "azureadmin"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "ubuntu"
    sku       = "22_04-lts"
    version   = "latest"
  }
}
```

In a production project, I would use reusable modules rather than keeping everything in one file.

---

## 2.2 How do you back up a VM created through Terraform?

Use **Azure Backup** with a **Recovery Services Vault**.

Architecture:

```text
VM
 ↓
Recovery Services Vault
 ↓
Backup Policy
 ↓
Recovery Point
```

Terraform provisions the backup infrastructure; Azure Backup performs the actual backup.

Example structure:

```hcl
resource "azurerm_recovery_services_vault" "vault" {
  name                = "rsv-prod"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  sku                 = "Standard"
}

resource "azurerm_backup_policy_vm" "policy" {
  name                = "daily-policy"
  resource_group_name = azurerm_resource_group.rg.name
  recovery_vault_name = azurerm_recovery_services_vault.vault.name

  timezone = "UTC"

  backup {
    frequency = "Daily"
    time      = "23:00"
  }

  retention_daily {
    count = 30
  }
}
```

---

## 2.3 How do you identify Terraform drift?

Run:

```bash
terraform plan
```

Terraform compares:

```text
Terraform configuration
        +
Terraform state
        +
Actual Azure infrastructure
```

Example:

```text
Terraform says:
VM size = D4s_v5

Azure:
VM size = D8s_v5
```

Terraform plan will show the difference.

If the manual change is valid, update Terraform code. If it is unauthorized, revert it through Terraform.

---

## 2.4 What is Terraform state?

`terraform.tfstate` stores Terraform's knowledge of resources it manages.

It is used for:

- Mapping Terraform resources to actual infrastructure
- Planning changes
- Tracking dependencies
- Managing resource lifecycle

For team environments, use a remote backend such as Azure Storage.

Important:

> Never commit Terraform state to Git because it may contain sensitive information.

---

## 2.5 How do you handle Terraform state in a team?

Use remote state:

```text
Azure Storage Account
        ↓
Blob Container
        ↓
terraform.tfstate
```

Benefits:

- Centralized state
- Team collaboration
- State locking/concurrency control where supported
- Better security and backup

---

## 2.6 What is Terraform drift?

Drift means:

> **Actual infrastructure differs from what Terraform configuration/state expects.**

Example:

```text
Terraform:
NSG allows 443

Portal:
NSG allows 443 + 22
```

`terraform plan` can reveal the difference.

---

## 2.7 What are Terraform Lifecycle blocks?

Common settings:

### `prevent_destroy`

```hcl
lifecycle {
  prevent_destroy = true
}
```

Prevents Terraform from destroying a protected resource.

### `create_before_destroy`

```hcl
lifecycle {
  create_before_destroy = true
}
```

Creates the replacement before destroying the old resource where supported.

### `ignore_changes`

```hcl
lifecycle {
  ignore_changes = [
    tags
  ]
}
```

Tells Terraform to ignore changes to selected attributes.

---

## 2.8 Can lifecycle settings prevent Terraform drift?

**Not generally.**

`ignore_changes` can intentionally ignore selected differences, but it does not remove actual drift.

Interview answer:

> **Lifecycle controls Terraform's behavior when changes occur; it is not a general drift-prevention mechanism.**

---

## 2.9 How many resources have you deployed through Terraform?

Do not invent a number.

A safe answer:

> “I've used Terraform to provision multiple Azure resources including resource groups, VNets, subnets, NSGs, route tables, public IPs, storage accounts, ACR and AKS. For larger implementations, I organize infrastructure into reusable modules and environment-specific configurations.”

If the interviewer insists on a number, give your actual approximate experience.

---

## 2.10 Do you use Storage Accounts?

Yes.

Common uses:

- Terraform remote state
- Blob Storage
- Application data
- Logs/archive
- Backup-related storage use cases

For Terraform:

```text
Storage Account
      ↓
Blob Container
      ↓
terraform.tfstate
```

Use RBAC, private access and appropriate security controls.

---

# 3. AKS and Kubernetes

## 3.1 How do you upgrade AKS?

I follow a controlled process:

```text
Check supported upgrade path
       ↓
Review release notes
       ↓
Check deprecated APIs
       ↓
Test in non-prod
       ↓
Validate workloads
       ↓
Upgrade control plane
       ↓
Upgrade node pools
       ↓
Application validation
       ↓
Monitor
```

I also check:

```bash
az aks get-upgrades
```

---

## 3.2 What happens if you upgrade only the AKS control plane?

The managed Kubernetes **control plane** is upgraded.

Conceptually:

```text
Control Plane
├── API Server
├── Scheduler
├── Controller Manager
└── etcd
```

The existing worker node pools do **not** become upgraded merely because the control plane was upgraded.

Node pools need their own controlled upgrade.

---

## 3.3 How do you upgrade node pools?

After the control plane:

```text
Control Plane
     ↓
System Node Pool
     ↓
User Node Pools
     ↓
Application Validation
```

Consider:

- `maxSurge`
- PodDisruptionBudgets
- Available capacity
- Pod scheduling
- Drain behavior
- Application readiness
- Version-skew/support requirements

---

## 3.4 Which node pools are you using?

Typical architecture:

```text
AKS
├── System Node Pool
│   └── Core AKS components
│
└── User Node Pools
    ├── Application workloads
    ├── Backend workloads
    └── Specialized workloads
```

System pools host critical system components. User pools are primarily for application workloads.

---

## 3.5 How do you deploy application Pods only to a user pool?

Use labels and node selection.

Example node label:

```text
workload=application
```

Deployment:

```yaml
spec:
  nodeSelector:
    workload: application
```

For stronger isolation:

```text
Node Affinity
+
Taints
+
Tolerations
```

---

## 3.6 What is Workload Identity?

Microsoft Entra Workload Identity allows Kubernetes workloads to authenticate to Azure services without storing client secrets in Pods.

Flow:

```text
AKS Pod
   ↓
Kubernetes ServiceAccount
   ↓
OIDC Federation
   ↓
Microsoft Entra ID
   ↓
Managed Identity
   ↓
Key Vault / Storage / Azure Service
```

Example:

```text
Pod
 ↓
Workload Identity
 ↓
Managed Identity
 ↓
Key Vault
```

No client secret needs to be stored inside the Pod.

---

## 3.7 How do you restrict users from changing Pods and Deployments?

Use Kubernetes RBAC.

For example, give a developer:

```text
get
list
watch
```

on Pods but not:

```text
create
update
delete
```

for Production Deployments.

Prefer CI/CD or GitOps as the controlled deployment path.

---

## 3.8 How do you create Kubernetes RBAC roles?

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

Bind it:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: dev
subjects:
- kind: User
  name: developer@example.com
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 3.9 How do you troubleshoot Pods consuming high CPU/memory?

First:

```bash
kubectl top pods -n <namespace>
```

Then:

```bash
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

Check:

- CPU/memory usage
- Requests/limits
- OOMKilled
- Restarts
- Events
- Application logs
- Traffic increase
- Memory leaks
- Inefficient application behavior

Do not simply increase limits without understanding the root cause.

---

## 3.10 How do you check Pod logs?

```bash
kubectl logs <pod> -n <namespace>
```

Follow logs:

```bash
kubectl logs -f <pod> -n <namespace>
```

Previous crashed container:

```bash
kubectl logs <pod> -n <namespace> --previous
```

Specific container:

```bash
kubectl logs <pod> -c <container> -n <namespace>
```

For production, use centralized Azure Monitor/Log Analytics where configured.

---

## 3.11 Why do we use Namespaces?

Namespaces provide logical isolation and organization.

Example:

```text
AKS
├── dev
├── uat
├── prod
└── monitoring
```

Benefits:

- RBAC isolation
- Resource quotas
- Network policies
- Environment separation
- Easier application organization

---

## 3.12 Pod lifecycle

Typical lifecycle:

```text
Pending
   ↓
ContainerCreating
   ↓
Running
   ↓
Succeeded / Failed
```

Repeated container failures can result in:

```text
Running
  ↓
Crash
  ↓
Restart
  ↓
CrashLoopBackOff
```

Useful commands:

```bash
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

---

## 3.13 What happens when you deploy a Deployment?

Suppose:

```yaml
kind: Deployment
replicas: 3
```

Run:

```bash
kubectl apply -f deployment.yaml
```

Backend flow:

```text
kubectl
  ↓
API Server
  ↓
etcd
  ↓
Deployment Controller
  ↓
ReplicaSet
  ↓
Pods
  ↓
Scheduler
  ↓
Kubelet
  ↓
Container Runtime
  ↓
Containers
```

### Step-by-step

**1. kubectl sends the request**

```text
kubectl → API Server
```

**2. API Server validates and stores desired state**

The desired state is persisted in etcd.

**3. Deployment Controller detects desired state**

It creates/updates a ReplicaSet.

**4. ReplicaSet creates Pods**

```text
ReplicaSet
├── Pod 1
├── Pod 2
└── Pod 3
```

**5. Scheduler selects nodes**

Scheduler considers:

- Resource requests
- Node affinity
- Taints/tolerations
- Topology
- Availability

**6. Kubelet starts the Pod**

Kubelet works with the container runtime to pull the image and start containers.

**7. Readiness probe succeeds**

Once the Pod is Ready, it can receive Service traffic.

### Rolling update

When the image changes:

```text
Old ReplicaSet
      ↓
New ReplicaSet
      ↓
New Pods
      ↓
Readiness
      ↓
Old Pods gradually removed
```

---

## 3.14 How do you deploy multiple applications using Helm?

For Frontend, Backend and APIs, I can use separate charts:

```text
helm/
├── frontend/
├── backend/
└── api/
```

Or an umbrella chart:

```text
application/
├── Chart.yaml
├── values.yaml
└── charts/
    ├── frontend
    ├── backend
    └── api
```

Environment-specific values:

```text
values-dev.yaml
values-uat.yaml
values-prod.yaml
```

Deploy:

```bash
helm upgrade --install frontend ./helm/frontend
helm upgrade --install backend ./helm/backend
helm upgrade --install api ./helm/api
```

---

# 4. Azure Networking

## 4.1 What is an NSG?

**NSG = Network Security Group.**

It controls inbound and outbound traffic for Azure resources associated with subnets/NICs.

Rules include:

- Source
- Destination
- Port
- Protocol
- Direction
- Priority
- Allow/Deny

Example:

```text
Inbound TCP 443 → Allow
Unnecessary SSH from Internet → Deny
```

---

## 4.2 What is inbound and outbound?

### Inbound

Traffic **coming into** a resource.

```text
Internet → VM
```

### Outbound

Traffic **leaving** a resource.

```text
VM → Internet
```

In NSGs, both directions can have rules.

---

## 4.3 What is a NIC?

**NIC = Network Interface Card.**

In Azure, a NIC provides network connectivity to a VM.

It is associated with:

- Private IP
- Optional public IP
- NSG
- Subnet

Example:

```text
VM
 ↓
NIC
 ↓
Subnet
 ↓
VNet
```

---

## 4.4 What is DNS and why do we need it?

DNS translates names into IP addresses.

```text
app.company.com
      ↓
DNS
      ↓
10.10.10.20
```

In AKS, DNS is essential for:

- Kubernetes service discovery
- Pod-to-service communication
- External DNS resolution
- Internal applications
- Private endpoint resolution

AKS commonly uses **CoreDNS** for cluster DNS.

---

## 4.5 Why do we need Ingress?

Ingress provides centralized HTTP/HTTPS routing to multiple Kubernetes Services.

Without ingress:

```text
Service A → LoadBalancer → Public IP
Service B → LoadBalancer → Public IP
Service C → LoadBalancer → Public IP
```

With ingress:

```text
                 Ingress
                    ↓
          ┌─────────┼─────────┐
          ↓         ↓         ↓
      Frontend    Backend     API
```

Benefits:

- Host-based routing
- Path-based routing
- TLS termination
- Centralized routing
- Fewer external endpoints

---

## 4.6 Application Gateway vs API Gateway vs Ingress

### Application Gateway

Azure-managed Layer-7 load balancer/reverse proxy.

Useful for:

- HTTP/HTTPS routing
- TLS termination
- WAF
- Backend routing
- Health probes

### API Management (API Gateway)

Focused on API management:

- Authentication
- Authorization
- Rate limiting
- Policies
- Transformations
- API lifecycle
- Analytics

### Kubernetes Ingress

Kubernetes-native HTTP/HTTPS routing configuration implemented by an ingress controller.

Example architecture:

```text
Application Gateway
        ↓
Ingress Controller
        ↓
Kubernetes Services
        ↓
Pods
```

They can complement each other depending on the architecture.

---

## 4.7 Why use an Ingress Controller if we already have Application Gateway?

They operate at different layers of the architecture.

- Application Gateway can be the Azure external Layer-7 entry point and WAF.
- Ingress controller handles Kubernetes-native routing configuration.
- API Management provides API-specific management features.

Depending on the architecture, Application Gateway and an ingress controller can be used together, or a different ingress approach can be chosen.

---

## 4.8 What is an NSG used for?

NSGs provide network-level traffic filtering.

Example:

```text
Internet
   ↓
NSG
   ↓
Subnet/NIC
   ↓
VM/Workload
```

They help implement least-privilege network access.

---

# 5. Azure Identity, RBAC and Security

## 5.1 Managed Identity vs Service Principal

| Managed Identity | Service Principal |
|---|---|
| Identity associated with Azure resource/workload | Application identity in Microsoft Entra ID |
| Avoids managing credentials for Azure-native workloads | Can use secrets/certificates/federation |
| Common for Azure services | Common for automation/external apps |
| Preferred where Azure-managed identity is suitable | Useful where a standalone application identity is needed |

Modern Azure DevOps can use **Workload Identity Federation** to avoid long-lived client secrets.

---

## 5.2 What is Workload Identity?

See [AKS Workload Identity](#36-what-is-workload-identity).

Easy answer:

> It allows a Kubernetes workload to access Azure resources through federated identity without storing a client secret in the Pod.

---

## 5.3 What is Azure User Identity?

An Azure user identity is a human identity represented in **Microsoft Entra ID**.

Flow:

```text
Human
  ↓
Microsoft Entra ID
  ↓
User Identity
  ↓
Azure RBAC
  ↓
Azure Resource
```

Memory shortcut:

```text
User Identity = Human
Managed Identity = Workload
RBAC = Permissions
```

---

## 5.4 How do you grant custom access to a specific Azure service?

Use **Azure RBAC**.

Process:

```text
Identify user/group
      ↓
Identify target resource
      ↓
Choose least-privilege role
      ↓
Assign role
      ↓
Validate access
```

If built-in roles don't meet the requirement, create a **custom RBAC role** with only required actions.

Example:

```text
User
 ↓
Custom Role
 ↓
Specific Azure Resource
```

Always follow least privilege.

---

## 5.5 How do you protect Azure resources from accidental deletion?

Use:

- Resource Locks
- RBAC
- Azure Policy
- Terraform controls
- Approval workflows
- Backup/DR
- Activity Log monitoring

For example:

```text
CanNotDelete
```

resource lock can prevent deletion while still allowing appropriate modifications.

---

## 5.6 What are Azure Policies used for?

Azure Policy provides governance and compliance controls across Azure resources.

Examples:

- Restrict allowed regions
- Require tags
- Restrict SKUs
- Enforce security configurations
- Audit public endpoints
- Require diagnostic settings
- Enforce encryption/security standards

Scope can include:

```text
Management Group
Subscription
Resource Group
Resource
```

---

## 5.7 What is the Azure hierarchy?

```text
Microsoft Entra Tenant
        ↓
Management Groups
        ↓
Subscriptions
        ↓
Resource Groups
        ↓
Resources
```

### Tenant

Identity boundary.

### Management Group

Organizes subscriptions and governance.

### Subscription

Resource/billing boundary.

### Resource Group

Logical container for resources.

### Resource

Actual service such as AKS, VM, VNet, Key Vault or Storage Account.

---

## 5.8 What is Azure RBAC?

Azure RBAC controls **who can perform which actions on which Azure resources**.

It is based on:

```text
Security Principal
      +
Role Definition
      +
Scope
```

Example:

```text
User
 ↓
Contributor
 ↓
Resource Group
```

Better:

```text
User
 ↓
Specific custom role
 ↓
Specific resource
```

---

## 5.9 What are Access Policies in Azure Key Vault?

Key Vault supports authorization approaches including **Azure RBAC** and the older **Key Vault access policy model**.

Access policies specify what operations an identity can perform against Key Vault objects, such as:

- Secrets
- Keys
- Certificates

For modern deployments, Azure RBAC is generally preferred when appropriate.

---

## 5.10 What services are available in Azure Key Vault?

Key Vault manages:

- **Secrets**
- **Keys**
- **Certificates**

It is commonly used for:

```text
Passwords
API keys
Connection strings
Encryption keys
TLS certificates
```

---

## 5.11 How do you connect AKS to Key Vault?

A common secure design:

```text
AKS Pod
   ↓
Workload Identity
   ↓
Managed Identity
   ↓
Azure RBAC
   ↓
Key Vault
```

For Kubernetes secret consumption, the **Secrets Store CSI Driver with the Azure Key Vault provider** can be used.

Grant only the required permissions, such as secret/certificate read access.

---

## 5.12 What is SAS token in Azure Storage?

**SAS = Shared Access Signature.**

It provides delegated, time-limited access to Azure Storage resources.

It can restrict:

- Permissions
- Resource
- Start/expiry time
- Protocol
- IP range where applicable

Example permissions for a read-only use case:

```text
Read
```

For upload:

```text
Create
Write
```

Give only the permissions required.

Avoid using broad or long-lived SAS tokens.

---

# 6. Monitoring and Observability

## 6.1 What monitoring tools do you use?

A strong answer:

> “We use Azure Monitor and Log Analytics for Azure/AKS monitoring and Prometheus and Grafana for metrics and dashboards. We configure alerts for infrastructure, application health, resource utilization and availability.”

---

## 6.2 How does Grafana collect and display metrics?

Grafana normally does **not** collect metrics itself.

Flow:

```text
Application / Kubernetes
        ↓
Prometheus
        ↓
Metrics
        ↓
Grafana
        ↓
Dashboard
```

Grafana queries the configured data source and visualizes the results.

---

## 6.3 What is the purpose of health probes?

Health probes determine whether a backend endpoint is healthy and capable of receiving traffic.

For Application Gateway/load balancers:

```text
Gateway
   ↓
Health Probe
   ↓
Backend
```

If the backend fails the health probe, the gateway can stop sending traffic to it.

---

## 6.4 What are `/health`, `/healthz`, and `/ready`?

The names are conventions; exact behavior depends on the application.

### `/health`

General application health endpoint.

### `/healthz`

Common Kubernetes-style health endpoint. Usually indicates whether the application/process is healthy.

### `/ready`

Usually indicates whether the application is **ready to receive traffic**.

For Kubernetes:

```text
Liveness Probe
   ↓
Should container be restarted?

Readiness Probe
   ↓
Should traffic be sent to this Pod?
```

A service can be running but not ready.

---

## 6.5 What do you check for a 503?

A 503 usually means the request reached a component but there is no healthy/available backend to serve it, though the exact cause depends on the architecture.

Check layer by layer:

```text
Client
 ↓
DNS
 ↓
Application Gateway / Ingress
 ↓
Health Probe
 ↓
Ingress Controller
 ↓
Kubernetes Service
 ↓
Endpoints
 ↓
Pods
 ↓
Application
```

Check:

1. Gateway/Ingress health.
2. Backend pool.
3. Health probe status.
4. Ingress controller logs.
5. Kubernetes Service.
6. Endpoints/EndpointSlices.
7. Pod readiness.
8. Pod logs/events.
9. Network policies/NSGs/firewall.
10. Application availability.

Useful commands:

```bash
kubectl get pods -n <namespace>
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl get events -n <namespace>
kubectl describe ingress -n <namespace>
kubectl logs <ingress-controller-pod> -n <namespace>
```

---

# 7. Production and Incident Management

## 7.1 How do you handle production deployment failure?

Use:

```text
Detect
 ↓
Assess impact
 ↓
Communicate
 ↓
Stop deployment
 ↓
Rollback/forward fix
 ↓
Validate
 ↓
Monitor
 ↓
RCA
 ↓
Prevent recurrence
```

Key senior-level point:

> **Customer impact and service restoration come before deep RCA.**

---

## 7.2 Certificate expired and production job failed

Immediate action:

```text
Confirm certificate expiry
        ↓
Identify affected service/job
        ↓
Renew/replace certificate
        ↓
Update Key Vault
        ↓
Ensure workload uses new version
        ↓
Restart/redeploy if needed
        ↓
Validate
```

Then prevention:

- Certificate expiry alerts
- Automated renewal
- Key Vault integration
- Monitoring
- Runbooks

---

## 7.3 ACR has too many container images

Investigate:

- Repository size
- Image count
- Image size
- Untagged images
- Old tags
- Build frequency
- Storage usage

Actions:

- Remove obsolete images
- Configure retention/purge policies
- Use immutable/versioned tags
- Avoid unnecessary builds
- Automate cleanup
- Monitor registry usage

---

## 7.4 How do you identify a Production image?

Prefer immutable identifiers.

Example:

```text
myapp:1.2.35
```

Even better:

```text
myapp@sha256:<digest>
```

Flow:

```text
CI Build
   ↓
Image 1.2.35
   ↓
DEV
   ↓
UAT
   ↓
PROD
```

This ensures the exact tested image is promoted.

---

## 7.5 How do you handle customer/management escalations?

My approach:

```text
Understand impact
 ↓
Communicate status
 ↓
Prioritize recovery
 ↓
Engage required teams
 ↓
Provide regular updates
 ↓
Resolve
 ↓
RCA
 ↓
Preventive action
```

I communicate:

- Business/customer impact
- Current status
- What is being done
- Dependencies/blockers
- Next update/action

---

# 8. Team and Behavioral Questions

## 8.1 How are tasks assigned to you daily?

> “We follow Agile/Scrum. Requirements are created as Azure Boards/Jira work items. During sprint planning, work is prioritized and assigned based on priority, skills and availability. I take ownership of my assigned tasks, provide updates during daily Scrum and raise blockers early.”

---

## 8.2 How do requirements reach your team?

Typical flow:

```text
Customer/Product Owner
        ↓
Business Requirement
        ↓
User Story
        ↓
Acceptance Criteria
        ↓
Backlog Refinement
        ↓
Sprint Planning
        ↓
DevOps/Engineering Task
```

I clarify requirements, dependencies and acceptance criteria before implementation.

---

## 8.3 What happens during Daily Scrum?

Keep it short:

1. What I completed.
2. What I am working on.
3. Blockers/dependencies.

---

## 8.4 Have you participated in KT sessions?

Strong answer:

> “Yes. I've participated as both a learner and a contributor. I document architecture, deployment procedures, troubleshooting, monitoring, incidents and runbooks. For important areas, I conduct walkthroughs and hands-on sessions so the receiving team can independently support the environment.”

---

## 8.5 What are your hobbies?

Keep it genuine.

Example:

> “Outside work, I enjoy spending time with my family, travelling when possible, and learning about new technology, especially cloud and DevOps.”

---

## 8.6 Why are you looking for a job change?

Recommended answer:

> “I'm looking for a role where I can take more ownership of cloud-native DevOps and platform engineering. My experience has grown across Azure, AKS, Terraform, CI/CD, monitoring and automation, and I'm looking for an opportunity where I can work on larger-scale environments, contribute to architecture and automation, and continue growing technically.”

Avoid criticizing your current employer.

---

# 9. Databricks and Azure Services

## 9.1 What is Azure Databricks and why do we use it?

Azure Databricks is a managed analytics platform based on Apache Spark.

It is commonly used for:

- Data engineering
- Big-data processing
- Data analytics
- ETL/ELT
- Machine learning workloads

From a DevOps perspective, focus on:

- Infrastructure provisioning
- CI/CD
- Networking
- Identity/security
- Governance
- Monitoring

If your hands-on Databricks experience is limited, be honest:

> “I understand Databricks at an architectural level and can work on the DevOps/infrastructure side, but I would not claim deep data-engineering expertise unless required.”

---

## 9.2 What Azure Web Apps do you use?

Azure App Service / Web Apps is a managed platform for hosting web applications and APIs without managing the underlying VM infrastructure.

Common capabilities:

- Web Apps
- API Apps
- Deployment slots
- Autoscaling
- TLS/SSL
- Custom domains
- Application Insights integration
- Managed identities
- CI/CD integration

---

## 9.3 What Azure Storage types are available?

Common storage services include:

- **Blob Storage** – object storage
- **Azure Files** – managed file shares
- **Queue Storage** – messaging
- **Table Storage** – NoSQL key-value storage
- **Managed Disks** – block storage for VMs

---

## 9.4 Which Azure storage is used for AKS Pods?

It depends on workload requirements.

### Azure Disk

Best for:

- Block storage
- Single-node attachment scenarios
- Databases/stateful workloads where appropriate

### Azure Files

Best for:

- Shared file storage
- Multiple Pods needing shared access
- RWX scenarios

### Azure Blob

Best for:

- Object data
- Documents
- Media
- Backups
- Application objects

AKS storage can be integrated through CSI drivers.

---

## 9.5 What permissions do you give AKS to access Storage?

Prefer identity-based access rather than storage account keys.

For example:

```text
AKS Pod
 ↓
Workload Identity
 ↓
Managed Identity
 ↓
Azure RBAC
 ↓
Storage Account
```

Assign the least-privilege role required, such as a Blob data reader/contributor role depending on the workload.

---

## 9.6 GRS vs LRS and other storage redundancy options

### LRS — Locally Redundant Storage

Multiple copies within a single primary region/facility boundary.

Good for lower-cost local redundancy.

### ZRS — Zone-Redundant Storage

Replicates across availability zones within the region.

Provides stronger availability against a zone failure.

### GRS — Geo-Redundant Storage

Replicates data to a secondary region.

### GZRS — Geo-Zone-Redundant Storage

Combines zone redundancy in the primary region with geo-replication.

### RA-GRS / RA-GZRS

Adds read access to the secondary region.

Easy memory:

```text
LRS  = Local
ZRS  = Zones
GRS  = Geo
GZRS = Geo + Zones
```

---

# 10. 60-Second Introduction

Use this at the beginning of the interview:

> “I have around 9 years of experience in DevOps and Azure cloud, with strong hands-on experience in Azure DevOps, CI/CD, Terraform, AKS, Docker and Kubernetes. I've worked on designing and maintaining CI/CD pipelines, infrastructure automation using Terraform, AKS deployments and troubleshooting, Azure networking, Application Gateway, WAF, Azure Firewall and monitoring using Azure Monitor, Prometheus and Grafana.
>
> I've also worked with Helm and GitOps concepts using Argo CD, and I'm familiar with security practices including RBAC, managed identities, Workload Identity, SonarQube and container security scanning. I've been involved not only in deployment automation but also production troubleshooting, performance, reliability and incident resolution.
>
> I'm now looking for a senior role where I can take more ownership of Azure DevOps and cloud-native platform engineering.”

---

# 11. Last-Minute Priority Topics

If preparation time is limited, prioritize these topics.

## Priority 1 — CI/CD

Know:

- CI/CD architecture
- Maven
- SonarQube
- SAST
- DAST
- Trivy
- Docker
- ACR
- Helm
- Approvals
- Branch policies
- Build once/deploy many
- Rollback

## Priority 2 — AKS

Know:

- AKS architecture
- System vs user node pools
- Pod lifecycle
- Deployment lifecycle
- Services
- Ingress
- Helm
- Probes
- Scheduling
- Node affinity
- Taints/tolerations
- RBAC
- Workload Identity
- AKS upgrades

## Priority 3 — Azure Networking

Know:

- VNet
- Subnet
- NIC
- NSG
- Route table
- Azure Firewall
- Application Gateway
- WAF
- Ingress
- Private Endpoint
- DNS
- Azure CNI
- CNI Overlay

## Priority 4 — Terraform

Know:

- Providers
- Resources
- Variables
- Outputs
- Modules
- State
- Remote backend
- Plan/apply
- Drift
- Lifecycle
- Import
- Dependencies
- Terraform + Azure DevOps

## Priority 5 — Security

Know:

- Microsoft Entra ID
- User identity
- Managed Identity
- Service Principal
- Workload Identity
- RBAC
- Custom roles
- Key Vault
- Azure Policy
- NSG
- WAF
- SAST/DAST
- Container scanning

## Priority 6 — Monitoring

Know:

- Prometheus
- Grafana
- Azure Monitor
- Log Analytics
- Alerts
- Health probes
- Readiness vs liveness
- 503 troubleshooting
- CPU/memory troubleshooting

---

# Quick Interview Memory Sheet

## Identity

```text
User Identity   = Human
Managed Identity = Azure workload identity
Service Principal = Application identity
Workload Identity = Kubernetes workload → Azure
RBAC             = Permissions
```

## Kubernetes

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↓
Container
```

## Deployment backend

```text
kubectl
 ↓
API Server
 ↓
etcd
 ↓
Controller
 ↓
ReplicaSet
 ↓
Scheduler
 ↓
Kubelet
 ↓
Container Runtime
```

## CI/CD

```text
Git
 ↓
Build
 ↓
Test
 ↓
SonarQube
 ↓
Security
 ↓
Docker
 ↓
Trivy
 ↓
ACR
 ↓
Helm
 ↓
AKS
 ↓
Monitor
```

## Production promotion

```text
Build once
   ↓
DEV
   ↓
UAT
   ↓
Approval
   ↓
PROD
```

## Azure hierarchy

```text
Tenant
 ↓
Management Group
 ↓
Subscription
 ↓
Resource Group
 ↓
Resource
```

## Networking

```text
Internet
   ↓
DNS
   ↓
Application Gateway / WAF
   ↓
Ingress
   ↓
Service
   ↓
Pod
```

## Storage

```text
Blob       = Object
Files      = Shared files
Disk       = Block storage
Queue      = Messages
Table      = NoSQL key/value
```

## Terraform

```text
Code
 ↓
terraform init
 ↓
terraform plan
 ↓
Approval
 ↓
terraform apply
 ↓
Azure
```

---

# Final Interview Tips

### 1. Don't answer only with definitions

Instead of:

> “Workload Identity is...”

Say:

> “We use Workload Identity so Pods can access Azure resources without storing client secrets. The Pod uses a Kubernetes ServiceAccount, OIDC federation and a Microsoft Entra managed identity, which is granted the required RBAC permissions.”

### 2. Use production examples

Whenever possible mention:

- Security
- High availability
- Monitoring
- Rollback
- Least privilege
- Automation
- Disaster recovery
- Cost
- Reliability

### 3. If you don't know something

Don't invent an answer.

Say:

> “I haven't implemented that directly, but I understand the concept and this is how I would approach it.”

This is much better than giving an incorrect production answer.

### 4. For troubleshooting questions

Always explain your investigation **layer by layer**.

Example for 503:

```text
DNS
 ↓
Gateway
 ↓
Health Probe
 ↓
Ingress
 ↓
Service
 ↓
Endpoints
 ↓
Pods
 ↓
Application
```

### 5. For senior-level questions

Think like an owner:

```text
Design
 ↓
Security
 ↓
Automation
 ↓
Monitoring
 ↓
Reliability
 ↓
Troubleshooting
 ↓
Recovery
 ↓
Prevention
```

---

**Good luck with your Senior Azure DevOps Engineer interview!**
