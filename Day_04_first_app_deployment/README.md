# Day 04 - First Application Deployment with Argo CD

In this lesson, we will deploy applications using three different Argo CD approaches and understand how Argo CD implements GitOps principles by continuously synchronizing Kubernetes resources with the desired state stored in Git.

By the end of this lesson, you will know how to deploy and manage applications using the Argo CD UI, CLI, and declarative Application manifests.

---

## Learning Objectives

After completing this lesson, you will be able to:

- Deploy applications using the Argo CD Web UI.
- Create and manage applications using the Argo CD CLI.
- Define Argo CD Applications using Kubernetes YAML manifests.
- Understand the difference between imperative and declarative deployments.
- Verify application Sync Status and Health Status.
- Understand why Git is considered the source of truth in GitOps.

---

## Repository

Fork and clone the repository:

```bash
git clone https://github.com/<your-github-username>/GitOps-ArgoCD-for-Kubernetes.git

cd GitOps-ArgoCD-for-Kubernetes
```

Alternatively, you can directly clone the repository:

```bash
git clone https://github.com/TechWithSatishThapak/GitOps-ArgoCD-for-Kubernetes.git

cd GitOps-ArgoCD-for-Kubernetes
```

Verify the repository:

```bash
git remote -v
git status
```

---

## Project Structure

```text
Day_04_first_app_deployment
├── cli_approach
├── declarative_approach
└── ui_approach
    ├── nginx
    │   ├── nginx_deployment.yml
    │   ├── nginx_svc.yml
    │   └── README.md
    │
    └── output_images
```

---

# Approach 1 - UI Deployment (NGINX)

In this approach, we use the Argo CD Dashboard to create and manage an application through the graphical interface.

### What You Will Learn

- Accessing the Argo CD UI
- Creating an Application
- Connecting a Git repository
- Synchronizing resources
- Monitoring application health
- Visualizing Kubernetes resources

### Sample Application

```text
NGINX
```

### Documentation

For a complete step-by-step guide:

```text
ui_approach/nginx/README.md
```

---

# Approach 2 - CLI Deployment

In this approach, we use the Argo CD CLI to create and manage applications directly from the terminal.

The CLI approach is useful for automation, scripting, and operational tasks.

### What You Will Learn

- Logging into Argo CD
- Creating applications
- Synchronizing applications
- Viewing application status
- Performing command-line operations

### Common Commands

Create an application:

```bash
argocd app create <app-name> \
  --repo <git-repository-url> \
  --path <manifest-path> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Check application status:

```bash
argocd app get <app-name>
```

Synchronize an application:

```bash
argocd app sync <app-name>
```

Wait until the application becomes healthy:

```bash
argocd app wait <app-name> --health
```

### Documentation

```text
cli_approach/README.md
```

---

# Approach 3 - Declarative Deployment

This is the recommended approach for implementing GitOps.

Instead of creating applications manually through the UI or CLI, we define the Argo CD Application as a Kubernetes manifest.

The entire application configuration becomes version-controlled and can be reviewed, approved, and managed through Git workflows.

### Benefits of the Declarative Approach

- Version Control
- Auditability
- Reproducibility
- Team Collaboration
- Git as Source of Truth
- Production-Ready GitOps Workflow

### Example Application Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: sample-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/TechWithSatishThapak/GitOps-ArgoCD-for-Kubernetes.git
    targetRevision: HEAD
    path: declarative_approach

  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

Apply the Application:

```bash
kubectl apply -f application.yaml
```

### Documentation

```text
declarative_approach/README.md
```

---

# Comparison: UI vs CLI vs Declarative

| Feature | UI Approach | CLI Approach | Declarative Approach |
|----------|----------|----------|----------|
| Application Creation | Dashboard | Command Line | YAML Manifest |
| Deployment Style | Imperative | Imperative | Declarative |
| Beginner Friendly | ✅ | ⚠️ | ⚠️ |
| Automation Friendly | ❌ | ✅ | ✅ |
| Git Version Control | ❌ | ❌ | ✅ |
| Reproducibility | Limited | Moderate | High |
| Production Recommended | ❌ | ⚠️ | ✅ |
| GitOps Ready | ⚠️ | ⚠️ | ✅ |

---

## Which Approach Should You Use?

### UI Approach

Best for:

- Learning Argo CD
- Demonstrations
- Visualizing resources
- Understanding synchronization behavior

### CLI Approach

Best for:

- Operators
- CI/CD automation
- Scripting repetitive tasks
- Troubleshooting

### Declarative Approach

Best for:

- Production environments
- Team collaboration
- GitOps workflows
- Infrastructure as Code

---

# Understanding Argo CD Status

After deploying an application, always check the following two statuses.

## Sync Status

Sync Status indicates whether the live cluster state matches the configuration stored in Git.

Common statuses:

```text
Synced
OutOfSync
Unknown
```

### Synced

The Kubernetes cluster matches the desired state stored in Git.

### OutOfSync

A difference exists between Git and the running cluster.

### Unknown

Argo CD cannot determine synchronization status.

---

## Health Status

Health Status indicates whether deployed resources are functioning correctly.

Common statuses:

```text
Healthy
Progressing
Degraded
Missing
Suspended
Unknown
```

### Healthy

Application resources are running normally.

### Progressing

Resources are still being created or updated.

### Degraded

One or more resources have failed.

### Missing

Expected resources are not present in the cluster.

### Suspended

Resource operation is intentionally suspended.

### Unknown

Argo CD cannot determine resource health.

> **Important:** An application can be **Synced** but not **Healthy**.  
> For example, Kubernetes resources may match Git perfectly, but pods may still be failing due to application errors.

---

# Verification Commands

List all Argo CD applications:

```bash
kubectl get applications -n argocd
```

View application details:

```bash
argocd app get <application-name>
```

List application resources:

```bash
argocd app resources <application-name>
```

View Kubernetes resources:

```bash
kubectl get deployments,pods,svc
```

Check Kubernetes events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# Key GitOps Concepts Learned

## Desired State

The desired state is defined in Git.

```text
Git Repository
```

Example:

```text
nginx_deployment.yml
nginx_svc.yml
application.yaml
```

---

## Current State

The current state represents resources running in the Kubernetes cluster.

```text
Deployments
Pods
Services
Ingresses
ConfigMaps
```

---

## Reconciliation

Argo CD continuously compares:

```text
Desired State (Git)
         VS
Current State (Cluster)
```

Whenever differences are detected, Argo CD can synchronize the cluster back to the desired state.

---




