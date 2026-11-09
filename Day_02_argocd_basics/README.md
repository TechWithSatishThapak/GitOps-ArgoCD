# Day_02: Argo CD Basics

## What You Will Learn

By the end of this chapter, you should be able to:

- Explain why teams use Argo CD for GitOps.
- Understand the basic differences between Argo CD, Flux CD, and Jenkins X.
- Identify the main components in Argo CD's architecture.
- Understand important Argo CD concepts such as Applications, Projects, Sync, Health, and Auto-Healing.
- Recognize common sync options, hooks, and sync waves.

---

## 2.1 Why Use Argo CD for GitOps?

Argo CD is a GitOps continuous delivery tool built for Kubernetes.

It watches the configuration stored in Git and compares it with the resources running in a Kubernetes cluster. This helps teams keep their applications in the state described in Git.

### Why Do Teams Choose Argo CD?

- **Built for Kubernetes:** Argo CD is designed to deploy and manage applications on Kubernetes.
- **Git as the source of truth:** Application configuration is stored and version-controlled in Git.
- **Multi-cluster support:** One Argo CD instance can manage applications in multiple Kubernetes clusters, when configured with the right access.
- **UI, CLI, and API:** You can manage applications through a web interface, command line, or API.
- **Access control:** It supports role-based access control (RBAC), authentication integrations, and audit-related logging.
- **Part of the Argo ecosystem:** It works alongside other Argo projects, such as Argo Workflows, Argo Rollouts, and Argo Events.

### In Simple Words

Argo CD makes it easier to see what is deployed, identify differences between Git and Kubernetes, and manage application deployments in a consistent way.

---

## 2.2 Argo CD vs Flux CD vs Jenkins X

Argo CD, Flux CD, and Jenkins X all support Kubernetes delivery workflows, but they approach the work differently.

| Tool | Strengths | Things to Consider |
| --- | --- | --- |
| **Argo CD** | Visual web UI, application-focused view, multi-cluster support, and clear sync and health status. | Requires resources to run and is focused mainly on Kubernetes delivery. |
| **Flux CD** | Lightweight, GitOps-focused, and works with Helm and Kustomize. Its controllers are modular. | The core project is primarily CLI and controller based; a separate UI can be added if needed. |
| **Jenkins X** | Brings together CI/CD workflows and GitOps-based promotion for Kubernetes. | Has a broader set of concepts and components, so setup and operation may feel complex. |

### Which One Should You Choose?

There is no single best tool for every team.

- Choose **Argo CD** if you want a visual, application-focused way to manage Kubernetes deployments.
- Choose **Flux CD** if you prefer a modular, controller-based GitOps setup.
- Consider **Jenkins X** if you want a platform that combines several CI/CD and Kubernetes delivery workflows.

For this learning path, we will use Argo CD.

---

## 2.3 Argo CD Architecture

Argo CD has a few main components. Each component has a specific responsibility.

![Argo CD Architecture](https://argo-cd.readthedocs.io/en/stable/assets/argocd_architecture.png)

### Main Components

#### 1. API Server

The API Server is the entry point for users and other tools.

It provides access through the web UI, CLI, and API. It handles requests and applies configured authentication and authorization rules, including RBAC.

#### 2. Repository Server

The Repository Server connects to the Git repositories configured in Argo CD.

Its main jobs are to:

- Fetch the application configuration from Git.
- Generate Kubernetes manifests from supported sources.
- Support plain YAML, Helm, and Kustomize.
- Provide the generated manifests to other Argo CD components.

For example, when a repository contains a Helm chart, the Repository Server renders the chart into Kubernetes manifests.

#### 3. Application Controller

The Application Controller is responsible for monitoring Argo CD Applications.

It:

- Compares the desired state from Git with the live state in Kubernetes.
- Detects differences, also known as drift.
- Reports application sync and health status.
- Performs sync operations when requested or when automatic sync is configured.

**Think of the Application Controller as the component that keeps checking whether Kubernetes matches the configuration stored in Git.**

#### 4. Web UI and CLI

Argo CD provides two common ways to interact with it:

- **Web UI:** A visual dashboard where you can view Applications, resources, sync status, health, and differences.
- **CLI (`argocd`):** A command-line tool for managing Applications and automating tasks in scripts and pipelines.

---

## 2.4 Key Argo CD Concepts

### 1. Application

An **Application** is one of the main resources in Argo CD.

It tells Argo CD:

- Which Git repository to use.
- Which folder or path contains the application configuration.
- Which Kubernetes cluster is the destination.
- Which namespace should receive the resources.

In simple terms, an Application connects a source in Git to a destination in Kubernetes.

### 2. Project

An **AppProject** (usually called a Project) is used to group and control Applications.

A Project can define:

- Which Git repositories Applications are allowed to use.
- Which Kubernetes clusters they can deploy to.
- Which namespaces they can deploy into.
- Which kinds of Kubernetes resources are allowed.
- Which users or teams have specific permissions through configured RBAC.

**Example:** A `team-a` Project can be configured to allow deployments only from approved repositories into namespaces used by Team A.

Projects are useful when multiple teams share one Argo CD installation.

### 3. Repository

A repository is a Git source registered with or referenced by Argo CD.

It can contain:

- Plain Kubernetes YAML manifests.
- Helm charts.
- Kustomize configuration and overlays.

Argo CD reads the selected path or chart from the repository to find the desired application configuration.

### 4. Sync Status

Sync status tells you whether the live Kubernetes resources match the desired configuration in Git.

Common sync statuses include:

| Status | Meaning |
| --- | --- |
| **Synced** | The live resources match the desired configuration Argo CD has generated from Git. |
| **OutOfSync** | One or more live resources differ from the desired configuration, or a required resource is missing. |
| **Unknown** | Argo CD cannot determine the sync status. |

**Important:** Sync status and health status are different. An Application can be `Synced` but still be unhealthy.

### 5. Health Status

Health status gives an indication of whether the application resources are working as expected.

Common health statuses include:

| Status | Meaning |
| --- | --- |
| **Healthy** | The resources are considered healthy. |
| **Progressing** | Resources are still being created, updated, or becoming ready. |
| **Degraded** | One or more resources are not healthy. |
| **Suspended** | A resource is intentionally paused or suspended. |
| **Missing** | A resource expected by the Application is not present. |
| **Unknown** | Argo CD cannot determine the health status. |

For example, a Deployment may be degraded if its Pods are repeatedly failing to start.

### 6. Rollbacks

A rollback means returning an application to an earlier version.

With Argo CD, you can use application history to identify an earlier deployment revision. You can also revert the relevant change in Git and sync the Application.

**GitOps best practice:** For Git-managed applications, prefer reverting the configuration in Git. This keeps Git as the source of truth.

Note that Argo CD rollback behavior has restrictions when automated sync is enabled. Check the version-specific documentation before relying on the rollback command in an automated-sync setup.

### 7. Auto-Sync and Self-Heal

Argo CD supports automatic synchronization.

When **automated sync** is enabled, Argo CD can apply changes from Git without requiring someone to start each sync manually.

**Self-heal** is an additional automated-sync setting. When enabled, Argo CD can correct certain changes made directly in the cluster that cause resources to differ from Git.

#### Example

Suppose the Git configuration says that a Deployment should have three replicas.

Someone manually changes the Deployment in Kubernetes to one replica:

```bash
kubectl scale deployment web-app --replicas=1
```

If automated sync and self-heal are configured, Argo CD can detect the difference and restore the replica count to three.

**Remember:** If a Pod is deleted, Kubernetes' Deployment controller normally creates a replacement Pod by itself. That is Kubernetes self-recovery, not proof that Argo CD self-heal has run.

### 8. Sync Policies and Sync Options

A sync policy controls how Argo CD applies the desired configuration.

#### Manual Sync

With manual sync, Argo CD detects changes and reports that the Application is `OutOfSync`. A user must start the sync.

Example:

```bash
argocd app sync web-app
```

#### Automated Sync

With automated sync enabled, Argo CD can apply detected Git changes without a manual sync command.

Automated sync can be configured with additional options such as pruning and self-healing.

#### Sync Waves

Sync waves help control the order in which resources are applied.

For example, you may want a database-related resource to be created before an application Deployment.

A resource can be assigned a wave using an annotation:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

Resources with lower wave numbers are applied before resources with higher wave numbers, subject to Argo CD's sync behavior and health checks.

#### Sync Hooks

Hooks are special resources that run at particular stages of a sync operation. They are commonly implemented as Kubernetes Jobs.

| Hook | When it runs |
| --- | --- |
| **PreSync** | Before the normal sync begins. |
| **Sync** | During the sync phase. |
| **PostSync** | After a successful sync and after the required resources are healthy. |
| **SyncFail** | When the sync operation fails. |

For example, a `PreSync` Job could run a database migration before the application is updated.

#### Useful Sync Options

| Option | What it does |
| --- | --- |
| **Prune** | Deletes resources managed by the Application when they are no longer present in Git. |
| **Create Namespace** | Creates the destination namespace if it does not already exist. |
| **Prune Last** | Prunes resources at the end of the sync operation. |
| **Apply Out of Sync Only** | Applies only resources that are out of sync. |
| **Respect Ignore Differences** | Uses configured ignored differences when applying resources. |
| **Server-Side Apply** | Uses Kubernetes server-side apply to manage resource changes. |
| **Skip Schema Validation** | Skips schema validation during apply. Use this only when you understand why it is needed. |
| **Replace** | Uses replace/create behavior instead of the usual apply behavior. This can be disruptive for some resources. |

Some options are configured on the Application, while others can be passed to a sync command. Always check the option's supported configuration and impact before using it in production.

---

## 2.5 A Simple Argo CD Workflow

Here is a basic example of how Argo CD works:

1. A developer updates Kubernetes YAML files in a Git repository.
2. The developer commits and pushes the changes.
3. Argo CD checks the repository for updates.
4. Argo CD compares the desired configuration with the live Kubernetes resources.
5. Argo CD reports the Application as `OutOfSync` if it finds differences.
6. A user starts a manual sync, or Argo CD syncs automatically if automated sync is enabled.
7. Argo CD checks the resources and reports their sync and health status.

This process helps teams manage deployments through reviewed and version-controlled changes.

---

## 2.6 Quick Revision

| Question | Short Answer |
| --- | --- |
| Why use Argo CD? | It helps manage Kubernetes deployments using GitOps. |
| What does the API Server do? | Provides the UI, CLI, and API entry point and handles access control. |
| What does the Repository Server do? | Fetches source configuration and generates Kubernetes manifests. |
| What does the Application Controller do? | Compares desired and live state, monitors Applications, and handles sync. |
| What is an Application? | A link between a Git source and a Kubernetes destination. |
| What is a Project? | A way to group Applications and apply deployment restrictions. |
| What does `OutOfSync` mean? | The live resources do not match the desired configuration. |
| Is `Synced` the same as `Healthy`? | No. Sync describes configuration matching; health describes resource condition. |
| What is self-heal? | An option that lets Argo CD correct certain live changes when enabled. |
| What are sync waves? | A way to control the order in which resources are synced. |
| What are sync hooks? | Resources that run at specific stages of a sync operation. |

---

## Key Takeaways

- Argo CD is a GitOps continuous delivery tool for Kubernetes.
- The Git repository describes the desired application configuration.
- The API Server, Repository Server, and Application Controller are key parts of Argo CD.
- Applications connect Git sources to Kubernetes destinations.
- Projects help organize Applications and control where they can deploy.
- Sync status and health status describe different things.
- Automated sync applies Git changes automatically when enabled.
- Self-heal can correct certain live changes when configured.
- Sync waves and hooks help control deployment order and run tasks during a sync.

---
