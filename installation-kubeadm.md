Sure! Here's a **detailed, easy-to-understand, markdown-formatted** summary of the kubeadm installation guide with necessary points and examples:

---

# 📦 Kubeadm Installation Guide (Kubernetes v1.32)

This guide walks through installing **kubeadm**, **kubelet**, and **kubectl** on a Linux system to set up a Kubernetes cluster.

> For creating the cluster after installation, refer to the **"Creating a cluster with kubeadm"** documentation.

---

## 🛠️ Prerequisites

Before installing anything, ensure the following system requirements are met:

### ✅ Minimum System Requirements
- **OS**: Linux (Debian/RedHat-based or generic)
- **RAM**: At least 2 GB per node
- **CPU**: At least 2 CPUs for control plane node
- **Connectivity**: Full network access between nodes (can be public/private)
- **Unique Identity**:
  - Unique **hostname**
  - Unique **MAC address** → `ip link` or `ifconfig -a`
  - Unique **product_uuid** → `sudo cat /sys/class/dmi/id/product_uuid`
> 📌 Kubernetes uses MAC and UUID to uniquely identify nodes. Duplicates (often in VMs) can cause installation failures.

### 🔌 Open Ports
Make sure required ports are open (for example):
```bash
nc 127.0.0.1 6443 -zv -w 2
```
> Pod network plugins might require additional ports. Refer to the plugin documentation.

### 🔁 Swap Configuration
Kubelet **does not allow swap by default**:
- ❌ Disable swap:
  ```bash
  sudo swapoff -a
  ```
  - Make it permanent: Edit `/etc/fstab` or relevant config
- ✅ Allow swap:
  - Set `failSwapOn: false` in kubelet config or pass it via CLI
  - Set `swapBehavior` (e.g., `LimitedSwap`) in kubelet config to allow workloads to use swap

---

## 🧱 Container Runtime Setup

Kubernetes requires a container runtime to manage pods.

### 🔧 Default Interface: **CRI** (Container Runtime Interface)
- Kubeadm auto-detects runtime using known CRI endpoints.
- If multiple/no runtimes found, you must specify one manually.

### 🐳 Docker Note
Docker doesn't support CRI **natively** anymore (from v1.24). Use `cri-dockerd` to bridge:
```text
Runtime                | Socket Path
-----------------------|----------------------------
containerd             | unix:///var/run/containerd/containerd.sock
CRI-O                  | unix:///var/run/crio/crio.sock
Docker (cri-dockerd)   | unix:///var/run/cri-dockerd.sock
```

---

## 🔧 Installing kubeadm, kubelet & kubectl

Install these on **all cluster nodes**:

| Component | Description |
|----------|-------------|
| `kubeadm` | Tool to bootstrap Kubernetes cluster |
| `kubelet` | Agent running on all nodes, manages pods |
| `kubectl` | CLI to interact with the cluster |

> 📌 Ensure versions of kubeadm, kubelet, and kubectl match your desired Kubernetes control plane version.  
> ✅ One minor version skew is allowed. For example:
- `kubelet` v1.7.0 ✅ with API server v1.8.0
- `kubelet` v1.9.0 ❌ with API server v1.8.0

---

## ⚠️ Version Management

- Do NOT upgrade kubeadm/kubelet/kubectl with regular system upgrades
- Refer to:
  - [Kubernetes version skew policy](https://kubernetes.io/releases/version-skew-policy/)
  - [Kubeadm-specific version skew](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

---

## ❗ Legacy Repositories Deprecated

From **Sep 13, 2023**, old repos like:
- `apt.kubernetes.io`
- `yum.kubernetes.io`

...are **deprecated**. Use new repositories from `https://pkgs.k8s.io`.

---

## 🧩 Debian-based Installation Steps

### 1. Update packages and install dependencies:
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

### 2. Download Kubernetes signing key:
```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### 3. Add the Kubernetes apt repository:
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 4. Install packages:
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl  # Prevent auto-upgrade
```

### 5. (Optional) Enable kubelet service:
```bash
sudo systemctl enable --now kubelet
```

> 📌 kubelet might restart often initially, waiting for kubeadm instructions.

---

## 🧠 Important Concept: cgroup Driver

- Both container runtime and kubelet must use the **same cgroup driver**.
- If mismatched → kubelet **fails to start**.

> 🔎 See [Configuring a cgroup driver](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/#cgroup-driver-configuration) for details.

---

## 📚 Example Summary

Let’s say you are setting up a 3-node cluster with Ubuntu 22.04. You would:
1. Ensure each VM has 2 GB RAM, 2 CPUs, unique MAC & UUID
2. Disable swap → `sudo swapoff -a`
3. Install containerd runtime
4. Install kubeadm, kubelet, kubectl
5. Configure repository using new `pkgs.k8s.io` links
6. Verify open ports (e.g., 6443 for API server)
7. Match cgroup driver for containerd and kubelet
8. Run `kubeadm init` to initialize control plane

---

> ✅ Now your node is ready to be part of a Kubernetes cluster!

--- 

Let me know if you'd like a similar summary for **creating a cluster using kubeadm** or **joining worker nodes**.
