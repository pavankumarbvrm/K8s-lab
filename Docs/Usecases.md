# 🧠 USE CASE: KubeConfig Bloat Problem and Remedy

As someone who works with multiple Kubernetes clusters, managing the KubeConfig file can quickly become difficult. Common issues include:

- **Old clusters, users, and contexts** staying in the config even after the cluster is deleted.
- **Manual cleanups** becoming tedious and error-prone.
- **Slow and confusing environment switching** due to clutter in the KubeConfig file.

Over time, this leads to a **bloated KubeConfig file**, making it harder to manage and interact with clusters effectively.

---

## What is a KubeConfig File?

A **KubeConfig file** holds information about clusters, users, and contexts, allowing Kubernetes to manage connections and enable easy interaction across different environments.

### Breakdown of a KubeConfig File:

- **Clusters**: Contains details of Kubernetes clusters, such as the API server endpoint and the cluster's Certificate Authority (CA).

    ```yaml
    clusters:
    - name: techopsexamples-cluster
      cluster:
        server: https://k8s.techopsexamples.com
        certificate-authority-data: <Cluster CA>
    ```

- **Users**: Stores credentials (tokens or certificates) for authenticating to the clusters.

    ```yaml
    users:
    - name: techopsexamples-user
      user:
        token: abc123tokenxyz
    ```

- **Contexts**: Links a user to a specific cluster, helping you switch between environments.

    ```yaml
    contexts:
    - name: techopsexamples-context
      context:
        cluster: techopsexamples-cluster
        user: techopsexamples-user
    ```

- **Current Context**: Specifies which user-cluster combination is currently active.

    ```yaml
    current-context: techopsexamples-context
    ```

---

## Managing the KubeConfig File with Kubectl

You can use `kubectl` to manage and edit the KubeConfig file. Here are some useful commands:

- **View the KubeConfig**:

    ```bash
    kubectl config view
    ```

- **Switch to a different context**:

    ```bash
    kubectl config use-context techopsexamples-context
    ```

- **Add a new cluster**:

    ```bash
    kubectl config set-cluster techopsexamples-cluster --server=https://techopsexamples.cluster.com
    ```

- **Add a new user**:

    ```bash
    kubectl config set-credentials techopsexamples-user --token=abc123tokenxyz
    ```

---

## KubeConfig Bloat Problem

Creating many short-lived clusters or environments can result in **KubeConfig bloat**, with references to:

- **Deleted clusters**
- **Unused users**
- **Irrelevant contexts**

This clutter makes it harder to manage necessary configurations and leads to slower, more confusing environment switching.

---

## Existing Solutions

There are several ways to handle KubeConfig bloat, but they have limitations:

- **Manual Edits**: You can manually remove entries from the KubeConfig file, but this is slow and error-prone.
- **Splitting Files**: Organizing configurations by splitting into multiple files works but complicates switching between them.
- **Custom Scripts**: These can automate cleanup but require regular updates and may not be flexible enough for changing setups.

---

## Better Solution: KubeTidy

**KubeTidy** is a tool designed to automatically remove outdated clusters, users, and contexts from your KubeConfig file. It provides:

- **Automatic cleanup** of old entries.
- **Simplified management** by keeping only relevant configurations.
- **Backup support**, ensuring your KubeConfig file is saved before any changes.

### How to Use KubeTidy

- Available on PowerShell (Windows/Linux/macOS) or as a **krew plugin** with Krew (Linux/macOS).

Sample cleanup summary:

```bash
$ kubetidy
✅ Cleaned up 5 old clusters
✅ Removed 3 outdated users
✅ Contexts reduced by 4
Backup of original file saved at: ~/.kube/config.bak

----

# 🧠 USE CASE: Hidden Risk of Relying on Labels in Kubernetes Security

A while ago, a client approached me with a request to **optimize the network security** of their Kubernetes cluster. They had a complex architecture with microservices communicating with each other, and they were using **NetworkPolicy** to control communication between these services.

However, despite their policies, they observed **unintended traffic flows** that raised concerns about the cluster's security. Upon investigation, I noticed that their policies were built around **pod labels**.

Here’s an example (identity modified) of what they had in place:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-control-database
spec:
  podSelector:
    matchLabels:
      app: techops-examples-db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: admin

The intention was clear: only pods with the role: admin label could access the techops-examples-db pod.

On closer inspection, I noticed potential issues:

    Labels Could Be Modified: Developers could label any pod as  role: admin at runtime, granting it database access.

    Namespace Confusion: Policies often overlooked namespaces, allowing a role: admin pod in the dev environment to mistakenly access production services. 

After discussing the concerns with the client, I asked if they were using Istio.

They were already using Istio for traffic management but hadn’t considered it for network security.

This presented a good opportunity !

Leveraging the Existing Setup:

We switched from pod labels to ServiceAccounts for more secure access control.

Here’s how the updated policy looks like:

apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: access-control-istio
spec:
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/techops/sa/admin-service-account"]

Now, only pods associated with the admin-service-account could access the techops-examples-db.

This was more reliable, as ServiceAccounts offer secure identity without relying on easily altered or misconfigured labels.
Why It Worked Better?

    Tighter Control: ServiceAccounts eliminated the risk of unauthorized access via label changes by tying access to pod identity.

    Built-In Security: Network traffic is encrypted with TLS, preserving identity across clusters and environments.

    No New Tools: The client was already using Istio, so no additional deployment was needed.

If you're using Kubernetes and relying on labels for access control, consider alternatives for better security and scalability.

Until next time, stay secure and keep optimizing!