
# 📘 Kubernetes: Nodes – Complete Summary

## 🔹 What is a Node?

A **Node** is a physical or virtual machine in a Kubernetes cluster where **Pods** run. Nodes are managed by the **Control Plane** and contain the essential services to run Pods like:

- `kubelet` (agent that talks to the control plane)
- `container runtime` (e.g., Docker or containerd)
- `kube-proxy` (networking service)

### Example:
If you have a 3-node cluster, each of the 3 VMs or servers is a node where Kubernetes can place Pods to run your applications.

---

## 🔹 Node Management

### 🚀 How Nodes are Added:
There are two main ways a Node becomes part of the cluster:
1. **Self-registration**: Kubelet automatically registers the node.
2. **Manual addition**: An admin/user creates a Node object using tools like `kubectl`.

Once a node is added, the control plane checks if:
- A kubelet is running on that machine.
- The node is healthy and eligible to run Pods.

If not healthy, Kubernetes keeps the node object but **ignores it** until it becomes healthy.

### ✅ Node Name Must Be Unique:
- Node names must follow DNS subdomain rules.
- No two nodes can share the same name.
- Re-using names can cause conflicts if the machine’s config or state has changed.

---

## 🔹 Node Self-registration (Automatic)

If `--register-node=true` (default), the kubelet registers itself using several flags:

| Flag | Purpose |
|------|---------|
| `--kubeconfig` | Auth credentials to talk to API server |
| `--cloud-provider` | To fetch metadata from cloud |
| `--register-node` | Enables auto-registration |
| `--register-with-taints` | Registers with specific taints |
| `--node-ip` | IP address for node communication |
| `--node-labels` | Labels for identifying node role/type |
| `--node-status-update-frequency` | How often node updates its status |

### ⚠️ Important:
If you change node labels and **don't re-register**, the changes may not apply correctly. It’s better to:
- **Remove the node object**
- **Re-register with the updated kubelet flags**

### 🔄 Why?
Pods may rely on node labels. Changing labels without proper re-registration can lead to:
- Misplaced Pods
- Scheduling issues
- Taint conflicts

---

## 🔹 Manual Node Administration

You can manage Nodes manually using `kubectl`.

### 🛠 When using manual mode:
- Start kubelet with `--register-node=false`.
- You can still modify node objects later (e.g., change labels or mark as unschedulable).

### 🔖 Labeling Nodes:
Use `node-role.kubernetes.io/<role>: <role>` to define a node’s role.
> ⚠️ The value is ignored; only the key matters.

### 🧭 Use Labels for Scheduling:
Example:
```yaml
nodeSelector:
  node-role.kubernetes.io/database: database
```
This ensures that the Pod only runs on database-role nodes.

### ❌ Marking a Node Unschedulable:
Prevents new Pods from being scheduled. Use before maintenance:
```bash
kubectl cordon <node-name>
```
> DaemonSet Pods still run on unschedulable nodes.

---

## 🔹 Node Status

Node `.status` contains:
- **Addresses**: IPs assigned to the node
- **Conditions**: e.g., Ready, OutOfDisk, etc.
- **Capacity/Allocatable**: CPU, memory, etc.
- **Info**: OS, architecture, container runtime version, etc.

Check status with:
```bash
kubectl describe node <node-name>
```

---

## 🔹 Node Heartbeats

Heartbeats determine if a node is **available or unreachable**.

### 🫀 Two Forms:
1. **Node status updates** – Regular updates to `.status`
2. **Lease objects** – Lightweight, faster check in `kube-node-lease` namespace

---

## 🔹 Node Controller (Control Plane Component)

Responsible for:
1. **Assigning CIDR** (IP ranges for Pods) to nodes (if enabled).
2. **Updating the node list** by querying the cloud provider.
3. **Monitoring node health**:

### 🔍 What it does:
- If node is unreachable, sets `.status.conditions.Ready = Unknown`
- Starts **evicting Pods** after 5 minutes (default)

You can configure:
```bash
--node-monitor-period         # How often to check (default: 5s)
--node-eviction-rate          # Default eviction: 0.1 per sec (1 node every 10s)
--secondary-node-eviction-rate # When zone has issues: 0.01 per sec
--unhealthy-zone-threshold    # If >55% nodes in zone are unhealthy
--large-cluster-size-threshold # Clusters with ≤50 nodes stop eviction
```

### ⚖️ Example:
- In a 100-node cluster, if 60 nodes in one zone become unhealthy (60% > 55%), the node controller **slows down evictions** to reduce disruption.
- If **entire cluster is unhealthy**, Kubernetes **pauses all evictions** assuming a control plane outage.

---

## 🔹 Node Taints & Pod Eviction

- Node controller **adds taints** for issues like:
  - NotReady
  - Unreachable
- Pods without proper tolerations are **evicted** from such nodes.

---

## 🔹 Resource Capacity Tracking

Nodes report resources like:
- Memory
- CPU

Scheduler ensures total requests of all Pods ≤ Node capacity.

If you **manually add** a node, you must define its capacity yourself.

### 🔧 Example:
```yaml
capacity:
  cpu: "4"
  memory: "16Gi"
```

> Note: This excludes processes started outside kubelet (e.g., manually launched containers or system daemons)

You can **reserve resources** for such processes if needed.

---

## 🔹 Node Topology

### 🧠 Topology Manager (from v1.27):
Helps assign CPU/memory resources more efficiently based on hardware layout (e.g., NUMA nodes).

---

## 🔹 Swap Memory Management

### 🔄 NodeSwap (from v1.30 beta):
- Allows swap memory on nodes (was previously disabled).
- Requires:
  - Feature gate `NodeSwap=true`
  - `--fail-swap-on=false`

> 💡 Useful when managing memory-hungry applications that tolerate some swap usage.

---

## ✅ Key Takeaways for Exam

- Node = place where Pods run; can be VM or physical machine
- Added by: self-registration (default) or manual `kubectl`
- Must be **unique name** (DNS compliant)
- Healthy node = kubelet is running & reachable
- Manual management: `--register-node=false`
- Labels and taints help control where Pods go
- `kubectl cordon` marks a node unschedulable
- Node controller monitors health, evicts Pods if needed
- Heartbeats are crucial to detect node failures
- Nodes report CPU/memory; scheduler respects capacity
- TopologyManager (v1.27+) and NodeSwap (v1.30+) add performance control

