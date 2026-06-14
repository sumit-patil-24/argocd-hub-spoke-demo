# Multi Cluster Deployment using Argo CD


An enterprise-grade continuous delivery ecosystem leveraging AWS EKS and Argo CD to orchestrate, sync, and self-heal applications across a distributed multi-cluster topology using a centralized Hub-Spoke design pattern.

---

## 🏗️ Architecture Overview

This platform moves away from legacy push-based CI/CD pipelines (e.g., executing direct shell scripts or Ansible playbooks via an orchestrator) to adopt a declarative, pull-based GitOps paradigm. 

The infrastructure is composed of:
1. **The Management Plane (Hub Cluster):** A centralized AWS EKS cluster running the core Argo CD controller fleet, handling metric sync, state compilation, and target cluster mapping.
2. **The Workload Plane (Spoke Clusters):** Independent downstream EKS clusters hosting localized business logic microservices. They remain lightweight, running zero deployment controllers internally.
3. **The Source of Truth (Git Repository):** Holds all structural Kubernetes YAML definitions (`ConfigMaps`, `Services`, `Deployments`).

```text
                     +---------------------------------------+
                     |         GitHub Repository             |
                     |  (Source of Truth: YAML Manifests)    |
                     +---------------------------------------+
                                         |
                                         | Pulls Desired State
                                         v
+-----------------------------------------------------------------------------------------+
| AWS CLOUD                                                                               |
|                                                                                         |
|    +-------------------------------------------------------------------------------+    |
|    |                      HUB CLUSTER (Management Plane)                           |    |
|    |                                                                               |    |
|    |    +-------------------------+         +---------------------------------+    |    |
|    |    |    Argo CD Server       |         |    Argo CD App Controller       |    |    |
|    |    | (Web UI Dashboard / CLI)|         |  (Monitors Drift & Auto-Heals)  |    |    |
|    |    +-------------------------+         +---------------------------------+    |    |
|    +-------------------------------------------------------------------------------+    |
|                                              /                           \              |
|                     Pushes Desired State    /                             \             |
|                     To Target API Server   /                               \            |
|                                           v                                 v           |
|    +------------------------------------------+ +------------------------------------------+    |
|    | SPOKE CLUSTER 1 (Dev Environment)        | | SPOKE CLUSTER 2 (QA Environment)         |    |
|    |                                          | |                                          |    |
|    |  +------------------+ +---------------+  | |  +------------------+ +---------------+  |    |
|    |  | guest-book App   | |  Service / CM |  | |  | guestbook-2 App  | |  Service / CM |  |    |
|    |  +------------------+ +---------------+  | |  +------------------+ +---------------+  |    |
|    +------------------------------------------+ +------------------------------------------+    |
+-----------------------------------------------------------------------------------------+

```

---

## 🛠️ Core Features & Capabilities

* **Centralized Administration Plane:** Unified control over a massive cluster fleet from a single web dashboard or CLI session, slashing operation and patching overhead.
* **Declarative State Drift Detection:** Continuous monitoring of live cluster state against the designated tracking branch in Git.
* **Automated Auto-Healing & Reconciliation:** Automated rollback algorithms that capture unauthorized manual hotfixes (`kubectl edit`) inside production namespaces and forcefully rewrite the environment back to the approved Git baseline.

---

## 🚀 Deployment Walkthrough

### Phase 1: Cluster Provisioning

Using the AWS `eksctl` utility, initialize three isolated EKS clusters backed by dedicated EC2 node groups across your regional cloud network.

```bash
eksctl create cluster --name hub-cluster --region us-west-1 --nodegroup-name hub-nodes --node-type t3.medium --nodes 2
eksctl create cluster --name spoke-cluster-1 --region us-west-1 --nodegroup-name spoke1-nodes --node-type t3.small --nodes 2
eksctl create cluster --name spoke-cluster-2 --region us-west-1 --nodegroup-name spoke2-nodes --node-type t3.small --nodes 2

```

Verify your client local runtime has securely imported all distinct cryptographic contexts:

```bash
kubectl config get-contexts

```

### Phase 2: Core GitOps Engine Initialization

Target the Management Plane (`hub-cluster`) context to provision the necessary CRDs and instantiate the standard Argo CD controller images.

```bash
kubectl config use-context <YOUR_HUB_CLUSTER_ARN>
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

```

To expose the administrative engine interface safely for initial validation, swap the default `ClusterIP` definition out for a node-routable alternative:

```bash
kubectl edit svc argocd-server -n argocd
# Modify 'type: ClusterIP' to 'type: NodePort'

```

### Phase 3: Cluster Federation & Spoke Registration

Log into the remote administrative engine controller via the local CLI tool using your Base64 decoded admin secrets, and securely map external targets into the Hub memory cache.

```bash
# Securely log into the management CLI plane
argocd login <HUB_NODE_IP>:<NODE_PORT> --username admin --password <DECODED_SECRET>

# Add Spoke Clusters to the Hub registry 
argocd cluster add <SPOKE_CLUSTER_1_CONTEXT_NAME>
argocd cluster add <SPOKE_CLUSTER_2_CONTEXT_NAME>

```

Navigate to your web management dashboard to ensure the control system recognizes all target infrastructure nodes.

---

## 📈 Verification & Drift Defusal Proof

### Multi-Target Synchronization

We declare two distinct `Application` resources mapping our tracking code directly to separate destination networks. When synced, the interface validates deployment success simultaneously:

Verify live component state maps correctly onto target system pods:

```bash
kubectl config use-context <YOUR_SPOKE_1_ARN>
kubectl get all -n default

```

### Configuration Drift Defusal In Action

To test structural stability, we deliberately create a drift anomaly by logging directly into a live spoke container namespace to manually patch a production parameter away from the code base definition:

```bash
kubectl edit configmap guest-book -n default
# Alter key/value data configuration manually

```

The centralized controller instantly identifies the mutation against the remote tracking repository, displaying a warning indicator within the interface:

Because the **Auto-Healing engine** is explicitly enabled, the controller instantly fires an isolated reconciliation action, wiping out the manual edit and forcefully pulling the clean, immutable definition back from Git to achieve instant self-healing:
