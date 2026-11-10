# Day 04 – First Application Deployment with Argo CD

In this lesson, we will learn how to deploy applications using Argo CD through three different approaches:

1. **UI Approach** – Deploy an NGINX application using the Argo CD Dashboard.
2. **CLI Approach** – Deploy an Apache application using the Argo CD CLI.
3. **Declarative Approach** – Deploy an Online Shop application using an Argo CD Application YAML manifest.

Each approach has its own advantages and use cases.

However, the declarative approach is the preferred way to implement GitOps because the desired application configuration is stored and managed in Git.

---

## 1. Learning Objectives

By the end of this lesson, you will understand:

- How to deploy applications using the Argo CD UI.
- How to create applications using the Argo CD CLI.
- How to define applications using Kubernetes YAML manifests.
- The difference between imperative and declarative approaches.
- How Argo CD synchronizes applications with Git repositories.
- How to check application Sync Status and Health Status.

---

## 2. Fork and Clone the Demo Repository

We will use an existing GitHub repository containing Kubernetes manifests and instructions for all three deployment approaches.

### Step 1: Fork the repository

Open the following GitHub repository and click the **Fork** button:

https://github.com/TechWithSatishThapak/Argocd_demo.git

This will create a copy of the repository under your GitHub account.

### Step 2: Clone your fork

Run the following command on your local machine:

```bash
git clone https://github.com/TechWithSatishThapak/Argocd_demo.git

cd argocd-demos
```

Replace `<your-username>` with your GitHub username.

Verify the repository:

```bash
git remote -v

git status
```
---

## 3. Three Approaches to Deploy Applications

We will now explore each deployment method.

### Approach 1 – UI Deployment (NGINX)

In this approach, we create an Argo CD Application through the web dashboard.

#### Steps

1. Open the Argo CD UI.
2. Log in using your Argo CD credentials.
3. Click **New App**.
4. Provide the application name, for example, `nginx`.
5. Select the Argo CD project.
6. Configure the Git repository URL.
7. Specify the path containing the NGINX Kubernetes manifests.
8. Select the destination cluster and namespace.
9. Create the application and synchronize it.

#### What happens behind the scenes?

- Argo CD connects to the configured Git repository.
- It reads the Kubernetes manifests from the specified path.
- It compares the desired configuration with the live Kubernetes resources.
- After synchronization, Argo CD applies the required resources to the cluster.

#### Verify the deployment

```bash
kubectl get applications -n argocd

kubectl get pods -A
```

For detailed instructions and the exact repository configuration, refer to:

`ui_approach/nginx/README.md`

---

### Approach 2 – CLI Deployment (Apache)

In this approach, we create an Argo CD Application using the Argo CD CLI.

The CLI is useful when we want to manage applications directly from the terminal or automate repetitive operations.

#### Step 1: Create an Application

General command:

```bash
argocd app create <APP_NAME> \
  --repo <GIT_REPOSITORY_URL> \
  --path <PATH_TO_MANIFESTS> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <TARGET_NAMESPACE>
```

Replace the placeholders with the actual values from the Apache example.

#### Step 2: Check Application status

```bash
argocd app get <APP_NAME>
```

#### Step 3: Synchronize the application

```bash
argocd app sync <APP_NAME>
```

#### Step 4: Wait for the application to become healthy

```bash
argocd app wait <APP_NAME> --health
```

#### What happens behind the scenes?

- The CLI sends the application creation request to the Argo CD API server.
- Argo CD creates an Application resource.
- Argo CD monitors the configured Git repository.
- During synchronization, the required Kubernetes resources are deployed.

**Note:** `https://kubernetes.default.svc` represents the Kubernetes API server of the cluster where Argo CD is running. If deploying to another cluster, configure the appropriate destination server.

For detailed instructions, refer to:

`cli_approach/apache/README.md`

---

### Approach 3 – Declarative Deployment (Online Shop)

This is the most important approach from a GitOps perspective.

Instead of creating an application manually through the UI or CLI, we define the Argo CD Application using a Kubernetes YAML manifest.

The manifest describes:

- Application name.
- Argo CD project.
- Git repository URL.
- Repository path.
- Target Kubernetes cluster.
- Target namespace.
- Synchronization policy.

#### Example: Application YAML

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: online-shop
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/<your-username>/argocd-demos.git
    targetRevision: HEAD
    path: declarative_approach/online_shop

  destination:
    server: https://kubernetes.default.svc
    namespace: online-shop

  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

This is an illustrative example. Verify the actual repository path and deployment configuration before applying it.

#### Step 1: Save the Application manifest

For example:

```text
applications/
└── online-shop.yaml
```

#### Step 2: Apply the manifest

```bash
kubectl apply -f applications/online-shop.yaml
```

#### Step 3: Verify the Application

```bash
kubectl get applications -n argocd
```

#### Step 4: Check the application status

```bash
argocd app get online-shop
```

Argo CD will monitor the configured Git repository and compare the desired state with the live Kubernetes resources.

**Why is this GitOps?**

Because the desired application configuration is stored in Git.

This provides:

- Version control.
- Audit history.
- Reproducible deployments.
- Easier collaboration.
- A clear record of configuration changes.

**Important:** Applying an Application YAML manually is a declarative operation. For a complete GitOps workflow, keep the Application definition itself in Git and manage its changes through Git-based workflows.

For the complete Online Shop example, refer to:

`declarative_approach/online_shop/README.md`

---

## 4. Comparison: UI vs CLI vs Declarative

| Feature | UI Approach | CLI Approach | Declarative Approach |
|---|---|---|---|
| Application creation | Argo CD Dashboard | Argo CD CLI | Application YAML |
| Method | Imperative | Imperative | Declarative |
| Configuration management | Usually created directly in the cluster | Usually created directly in the cluster | Stored and reviewed in Git |
| Best for | Learning and demos | Operators and scripting | GitOps and team workflows |
| Reproducibility | Limited without documented steps | Possible with scripts | High when managed through Git |
| Version control | Not automatic | Not automatic | Supported through Git |

### Which approach should we use?

- **UI:** Best for beginners who want to understand Argo CD visually.
- **CLI:** Useful for administrators and automation tasks.
- **Declarative:** Recommended for production-oriented GitOps workflows.

Remember that Argo CD manages Applications created through all three approaches. The main difference is how we create and maintain the Application configuration.

---

## 5. Understanding Sync Status and Health Status

After deploying an application, always check these two important Argo CD statuses.

### Sync Status

Indicates whether the live Kubernetes resources match the desired configuration from Git.

Common statuses:

- Synced
- OutOfSync
- Unknown

### Health Status

Indicates whether the deployed resources are healthy according to Argo CD's health assessment.

Common statuses:

- Healthy
- Progressing
- Degraded
- Suspended
- Missing
- Unknown

**Important:** An application can be `Synced` but not `Healthy`.

For example, Kubernetes resources may match the Git configuration, but a Pod may still be failing.

Always check both statuses.

---

## 6. Verification Commands

List all Argo CD Applications:

```bash
kubectl get applications -n argocd
```

Check application details:

```bash
argocd app get <APP_NAME>
```

Check Kubernetes resources:

```bash
kubectl get deployments,pods,services -n <TARGET_NAMESPACE>
```

Check application events:

```bash
kubectl get events -n <TARGET_NAMESPACE> \
  --sort-by=.lastTimestamp
```

---

## 7. Key Takeaways

- **UI:** Easy and visual; useful for learning and demonstrations.
- **CLI:** Scriptable and convenient for terminal-based operations.
- **Declarative:** Keeps application configuration in YAML and supports GitOps.
- Argo CD manages applications created through all three approaches.
- Git should be the source of truth for the desired application configuration.
- Always verify both Sync Status and Health Status.

---

## 8. Day 04 Checklist

- [ ] Forked the demo repository into my GitHub account.
- [ ] Cloned my fork to the local system.
- [ ] Explored the repository structure.
- [ ] Created the NGINX Application through the Argo CD UI.
- [ ] Created the Apache Application using the Argo CD CLI.
- [ ] Created the Online Shop Application using YAML.
- [ ] Verified application synchronization.
- [ ] Checked application health.
- [ ] Verified Kubernetes resources in their target namespaces.
- [ ] Understood the difference between imperative and declarative deployment.

---

**Next: Day 05 – Argo CD Sync Policies, Automated Sync, Self-Healing and Pruning.**