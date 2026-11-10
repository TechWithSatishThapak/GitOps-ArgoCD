# Day 03 – Argo CD Setup and Installation

In this lesson, we will create a Kubernetes cluster using **Kind**, install **Argo CD**, access its web UI, and connect to it using the Argo CD CLI.

We will cover two installation methods:
1. Helm
2. Official Kubernetes manifests

> **Recommendation:** Choose one installation method for your cluster. Do not install Argo CD using both methods in the same namespace.

## Learning objectives

By the end of this lesson, you should be able to:
- Check the tools required for an Argo CD lab.
- Create a multi-node Kind cluster.
- Install Argo CD using Helm or Kubernetes manifests.
- Verify Argo CD components.
- Access the Argo CD web UI safely.
- Retrieve the initial admin password.
- Install and use the Argo CD CLI.

## 1. Prerequisites

| Tool | Why do we need it? |
|---|---|
| Docker | Kind uses containers as Kubernetes nodes. |
| Kind | Creates a local Kubernetes cluster using Docker. |
| kubectl | Manages the Kubernetes cluster. |
| Helm | Required only for the Helm installation method. |
| curl | Downloads the Argo CD CLI binary. |

### 1.1 Install Docker

On Ubuntu:

```bash
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable --now docker
```

Add your current user to the Docker group:

```bash
sudo usermod -aG docker "$USER"
newgrp docker
```

Verify:

```bash
docker --version
docker ps
```

> **Note:** Membership in the `docker` group grants privileges that are effectively root-level. Only do this on a machine you trust.

### 1.2 Install Kind

Follow the official [Kind installation guide](https://kind.sigs.k8s.io/docs/user/quick-start/#installation).

```bash
kind version
```

### 1.3 Install kubectl

Follow the official [kubectl installation guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

```bash
kubectl version --client
```

### 1.4 Install Helm

Helm is required only for Method 1. Follow the official [Helm installation guide](https://helm.sh/docs/intro/install/).

```bash
helm version
```

---

## 2. Create a Kind cluster

For this lab, we will create a cluster with one control-plane node and two worker nodes.

Create `kind-config.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
```

Create the cluster:

```bash
kind create cluster --name argocd-cluster --config kind-config.yaml
```

Verify:

```bash
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods -A
```

### Understanding `apiServerAddress` and `apiServerPort`

You may see these optional settings in some Kind configurations:

```yaml
networking:
  apiServerAddress: "127.0.0.1"
  apiServerPort: 33893
```

- `apiServerAddress` controls the address on the **host** to which Kind binds the Kubernetes API server port.
- `apiServerPort` selects the host port used to reach that API server.
- These settings are useful when you need a predictable host-side API endpoint or a specific network binding.

> **Important correction:** These settings are not needed to make Argo CD Pods communicate with the Kubernetes API. In-cluster workloads normally reach the API through the Kubernetes service address and cluster networking.

For a lab on EC2, do not automatically bind the Kubernetes API to the instance's private or public IP. Keep it bound to localhost unless you have a clear networking requirement and have secured access with suitable firewall rules. A predictable port alone does not make the API server safely accessible.

---

## 3. Install Argo CD

Argo CD can be installed using Helm or the official installation manifest. Choose one method.

### Method 1 – Install Argo CD using Helm

Helm is useful when you want to customize installation settings through a `values.yaml` file and manage releases with Helm.

#### Step 1: Add the Argo Helm repository

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

#### Step 2: Create the namespace

```bash
kubectl create namespace argocd
```

#### Step 3: Install Argo CD

```bash
helm install argocd argo/argo-cd --namespace argocd
```

For a customized installation, create a `values.yaml` file and pass it with `-f values.yaml`. Review the chart's values and version before using it in production.

#### Step 4: Verify the installation

```bash
kubectl get pods -n argocd
kubectl get services -n argocd
kubectl get deployments -n argocd
```

Wait until the required Pods are ready.

### Method 2 – Install Argo CD using official manifests

This method is straightforward and useful for learning and quick lab environments.

#### Step 1: Create the namespace

```bash
kubectl create namespace argocd
```

#### Step 2: Apply the official installation manifest

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

For repeatable or production installations, prefer a specific Argo CD release version rather than relying on the moving `stable` URL.

#### Step 3: Verify the installation

```bash
kubectl get pods -n argocd
kubectl get services -n argocd
kubectl get deployments -n argocd
```

Wait for the Argo CD components to become ready.

---

## 4. Access the Argo CD web UI

For a secure lab setup, bind port-forwarding to localhost:

```bash
kubectl port-forward --namespace argocd \
  svc/argocd-server 8080:443
```

Keep this command running. Open this address in a browser on the same machine:

```text
https://localhost:8080
```

The browser may show a certificate warning because the default installation uses a self-signed certificate. For a temporary lab, proceed only if you understand and trust the local setup.

### Access the UI from your computer when Argo CD runs on EC2

Avoid exposing the port-forward process on every network interface with `--address=0.0.0.0` unless you have deliberately secured that access.

A safer option is an SSH tunnel. Run this on your **local computer**:

```bash
ssh -L 8080:127.0.0.1:8080 <user>@<EC2_PUBLIC_IP>
```

Keep the SSH session open. On the EC2 instance, run the `kubectl port-forward` command above. Then open this address on your local computer:

```text
https://localhost:8080
```

Replace `<user>` and `<EC2_PUBLIC_IP>` with your SSH username and EC2 public IP. Ensure the EC2 security group allows SSH only from trusted source IPs.

---

## 5. Get the initial Argo CD admin password

For a fresh installation, retrieve the initial password:

```bash
kubectl get secret argocd-initial-admin-secret \
  --namespace argocd \
  -o jsonpath='{.data.password}' | base64 --decode
echo
```

Log in to the web UI with:
- **Username:** `admin`
- **Password:** The value returned by the command above

Store the password securely. After logging in, change the initial admin password and follow your organization's access-control practices. Do not commit passwords or credential-containing Kubernetes Secrets to Git.

---

## 6. Install the Argo CD CLI on Ubuntu/Linux

The Argo CD server runs inside Kubernetes. The Argo CD CLI is a separate client that lets you manage Argo CD from your terminal.

### Step 1: Download the CLI

The following example downloads the Linux AMD64 binary:

```bash
curl -sSL -o argocd-linux-amd64 \
  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```

Install it:

```bash
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

> Choose the correct binary for your CPU architecture. The command above is for Linux AMD64.

### Step 2: Verify the CLI

```bash
argocd version --client
```

### Step 3: Log in to Argo CD

If you are using the local port-forward setup:

```bash
argocd login localhost:8080 \
  --username admin \
  --password '<INITIAL_PASSWORD>' \
  --insecure
```

Replace `<INITIAL_PASSWORD>` with the password retrieved earlier. Avoid putting real passwords into shell history or shared scripts.

The `--insecure` option skips certificate verification. It can be used temporarily in a trusted lab with the default self-signed certificate. For production, configure and verify a trusted TLS certificate instead.

### Step 4: Check the logged-in user

```bash
argocd account get-user-info
```

You should see information about the authenticated account.

---

## 7. Helm vs. manifest installation

| Area | Helm | Official manifests |
|---|---|---|
| Configuration | Customizable through chart values | Requires managing manifest configuration |
| Prerequisite | Helm and kubectl | kubectl |
| Release management | Helm tracks the release | Managed directly through Kubernetes resources |
| Useful for | Repeatable, customized deployments | Learning and straightforward installations |

Neither method is automatically production-ready. Production setup also requires decisions about version pinning, TLS, authentication, RBAC, secrets, backups, upgrades, and high availability.

---

## 8. Troubleshooting checks

If the UI does not open or Argo CD does not become ready, start with:

```bash
kubectl get pods -n argocd
kubectl get services -n argocd
kubectl get events -n argocd --sort-by=.lastTimestamp
kubectl describe pod -n argocd <POD_NAME>
kubectl logs -n argocd <POD_NAME>
```

Common checks:
- Confirm that the Kind cluster is running: `kind get clusters`.
- Confirm the current kubectl context: `kubectl config current-context`.
- Check whether Argo CD Pods are still starting or reporting errors.
- Confirm that the port-forward command is still running.
- Confirm that the SSH tunnel is active when connecting from your computer to EC2.
- Check whether local port `8080` is already in use.

---

## 9. Professional practices

- Use a dedicated namespace such as `argocd`; avoid installing into `default`.
- Pin and review Argo CD and Helm chart versions for repeatable environments.
- Use a trusted TLS certificate and proper authentication in production.
- Apply least-privilege RBAC; avoid using the admin account for routine work.
- Do not expose the Argo CD UI or Kubernetes API publicly without appropriate network controls.
- Do not store passwords, tokens, or unencrypted Secrets in Git.
- Keep application manifests and configuration in version-controlled Git repositories.
- Plan backups and test upgrades before changing production installations.
- Monitor Argo CD components and Kubernetes cluster health.

---

## 10. Quick revision

| Command | Purpose |
|---|---|
| `kind create cluster --name argocd-cluster --config kind-config.yaml` | Creates the Kind cluster |
| `kubectl get nodes` | Lists cluster nodes |
| `kubectl create namespace argocd` | Creates the Argo CD namespace |
| `kubectl get pods -n argocd` | Checks Argo CD Pods |
| `kubectl get svc -n argocd` | Lists Argo CD services |
| `kubectl port-forward -n argocd svc/argocd-server 8080:443` | Forwards the UI to localhost |
| `argocd version --client` | Checks the CLI installation |
| `argocd account get-user-info` | Shows the current CLI account |

## 11. Day 03 checklist

- [ ] Docker is installed and working.
- [ ] Kind, kubectl, and Helm (if using Helm) are installed.
- [ ] The Kind cluster has one control-plane node and two worker nodes.
- [ ] Argo CD is installed using one method.
- [ ] Argo CD Pods are ready.
- [ ] The web UI is accessible through a secure local connection.
- [ ] The initial admin password was retrieved securely.
- [ ] The Argo CD CLI is installed.
- [ ] CLI login is successful.
- [ ] The initial admin password has been changed.

**Next:** In Day 04, we can create our first Argo CD Application and deploy a sample application from a Git repository.