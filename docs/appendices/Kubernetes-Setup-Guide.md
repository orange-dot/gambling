# Kubernetes Setup Guide: Self-Managed On-Premises Cluster

**Document Version**: 1.0
**Date**: 2025-10-25
**Purpose**: Production-ready self-managed Kubernetes cluster for е-Играч
**Infrastructure**: On-premises datacenter (no cloud dependencies)

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Hardware Requirements](#hardware-requirements)
3. [Prerequisites](#prerequisites)
4. [Cluster Installation with kubeadm](#cluster-installation-with-kubeadm)
5. [Networking Setup (Calico CNI)](#networking-setup-calico-cni)
6. [Load Balancing (MetalLB)](#load-balancing-metallb)
7. [Storage Configuration](#storage-configuration)
8. [Monitoring Stack Setup](#monitoring-stack-setup)
9. [Security Hardening](#security-hardening)
10. [Backup & Disaster Recovery](#backup--disaster-recovery)
11. [Operations & Maintenance](#operations--maintenance)

---

## Architecture Overview

### Cluster Topology

```
┌──────────────────────────────────────────────────────────────────┐
│                 е-Играч Kubernetes Cluster                       │
│                     (On-Premises Datacenter)                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────── Control Plane ───────────────┐                 │
│  │  • k8s-master01  (16 CPU, 64GB RAM)       │                 │
│  │  • k8s-master02  (16 CPU, 64GB RAM)       │                 │
│  │  • k8s-master03  (16 CPU, 64GB RAM)       │                 │
│  │                                            │                 │
│  │  Components:                               │                 │
│  │  - kube-apiserver (HA, load-balanced)     │                 │
│  │  - etcd (3-node cluster, Raft consensus)  │                 │
│  │  - kube-scheduler                          │                 │
│  │  - kube-controller-manager                 │                 │
│  └────────────────────────────────────────────┘                 │
│                          ↕                                      │
│  ┌────────────── Worker Nodes ────────────────┐                 │
│  │  • k8s-worker01  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker02  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker03  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker04  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker05  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker06  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker07  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker08  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker09  (64 CPU, 256GB RAM)      │                 │
│  │  • k8s-worker10  (64 CPU, 256GB RAM)      │                 │
│  │                                            │                 │
│  │  Workloads:                                │                 │
│  │  - Microservices (Node.js)                │                 │
│  │  - MongoDB StatefulSets (3+3 replicas)    │                 │
│  │  - Kafka StatefulSets (5 brokers)         │                 │
│  │  - Flink (JobManager + TaskManagers)      │                 │
│  │  - Kong API Gateway                        │                 │
│  └────────────────────────────────────────────┘                 │
│                                                                  │
│  ┌─────────── Infrastructure Components ──────┐                 │
│  │  • Calico CNI (Pod networking)            │                 │
│  │  • MetalLB (Load balancer for bare-metal) │                 │
│  │  • Local Path Provisioner (PVs on SSDs)   │                 │
│  │  • Prometheus + Grafana (Monitoring)      │                 │
│  │  • ELK Stack (Logging)                    │                 │
│  │  • Velero (Backup & restore)              │                 │
│  └───────────────────────────────────────────┘                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Design Decisions

| Decision | Rationale |
|----------|-----------|
| **Self-Managed Kubernetes (kubeadm)** | Full control, no cloud dependencies, data sovereignty |
| **3 Control Plane Nodes** | High availability (survives 1 node failure), etcd quorum |
| **10 Worker Nodes** | Sufficient for 100k users, room for horizontal scaling |
| **Calico CNI** | Network policies, BGP support, high performance |
| **MetalLB** | Load balancing for bare-metal (no cloud LB), Layer 2/BGP modes |
| **Local Path Provisioner** | Fast local SSD storage, no external SAN dependency |

---

## Hardware Requirements

### Control Plane Nodes (3 nodes)

| Component | Specification |
|-----------|---------------|
| **CPU** | 16 cores (Intel Xeon or AMD EPYC) |
| **RAM** | 64 GB DDR4 ECC |
| **Disk** | 1 TB NVMe SSD (for etcd data) |
| **Network** | 10 Gbps Ethernet (dual NICs for redundancy) |
| **OS** | Ubuntu Server 22.04 LTS (kernel 5.15+) |

**Naming**: `k8s-master01`, `k8s-master02`, `k8s-master03`

### Worker Nodes (10 nodes)

| Component | Specification |
|-----------|---------------|
| **CPU** | 64 cores (Intel Xeon or AMD EPYC) |
| **RAM** | 256 GB DDR4 ECC |
| **Disk (OS)** | 500 GB NVMe SSD |
| **Disk (Data)** | 4 TB NVMe SSD (for MongoDB, Kafka, Flink state) |
| **Network** | 10 Gbps Ethernet (dual NICs for redundancy) |
| **OS** | Ubuntu Server 22.04 LTS |

**Naming**: `k8s-worker01` through `k8s-worker10`

### Network Infrastructure

| Component | Specification |
|-----------|---------------|
| **Core Switch** | 10 Gbps, redundant (2 switches, LACP bonding) |
| **ToR Switches** | 10 Gbps, one per rack |
| **Firewall** | Hardware firewall (isolated from external internet) |
| **DNS** | Internal DNS server (forward/reverse zones) |
| **NTP** | NTP server (time synchronization critical for Kafka, Flink) |

### Total Infrastructure

- **13 Physical Servers** (3 control + 10 workers)
- **Rack Space**: 2-3 racks (assuming 42U racks)
- **Power**: Dual PSUs per server, redundant PDUs
- **Cooling**: In-row cooling or hot/cold aisle containment

---

## Prerequisites

### 1. Operating System Setup

**Install Ubuntu Server 22.04 LTS on all nodes**:

```bash
# Update all packages
sudo apt-get update && sudo apt-get upgrade -y

# Install essential tools
sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release \
  vim \
  htop \
  net-tools

# Disable swap (required for Kubernetes)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Load kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set sysctl parameters
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

# Verify modules loaded
lsmod | grep br_netfilter
lsmod | grep overlay
```

### 2. Install Container Runtime (containerd)

```bash
# Install containerd
sudo apt-get install -y containerd

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable SystemdCgroup (required for Kubernetes)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 3. Install Kubernetes Components

```bash
# Add Kubernetes repository
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubelet, kubeadm, kubectl
sudo apt-get update
sudo apt-get install -y kubelet=1.28.4-1.1 kubeadm=1.28.4-1.1 kubectl=1.28.4-1.1

# Hold versions (prevent auto-updates)
sudo apt-mark hold kubelet kubeadm kubectl

# Enable kubelet
sudo systemctl enable kubelet
```

### 4. Configure DNS & Hostnames

**On all nodes**, add entries to `/etc/hosts`:

```bash
cat <<EOF | sudo tee -a /etc/hosts
# Kubernetes Cluster
192.168.10.11   k8s-master01
192.168.10.12   k8s-master02
192.168.10.13   k8s-master03

192.168.10.21   k8s-worker01
192.168.10.22   k8s-worker02
192.168.10.23   k8s-worker03
192.168.10.24   k8s-worker04
192.168.10.25   k8s-worker05
192.168.10.26   k8s-worker06
192.168.10.27   k8s-worker07
192.168.10.28   k8s-worker08
192.168.10.29   k8s-worker09
192.168.10.30   k8s-worker10

# Virtual IP for HA Control Plane
192.168.10.10   k8s-api.eigrac.local
EOF
```

### 5. Set Up Load Balancer for Control Plane (HAProxy)

**On a separate server or first master node**:

```bash
# Install HAProxy
sudo apt-get install -y haproxy

# Configure HAProxy
cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg
global
    log /dev/log local0
    chroot /var/lib/haproxy
    user haproxy
    group haproxy
    daemon

defaults
    log global
    option tcplog
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend k8s-api
    bind 192.168.10.10:6443
    mode tcp
    option tcplog
    default_backend k8s-api-backend

backend k8s-api-backend
    mode tcp
    balance roundrobin
    server k8s-master01 192.168.10.11:6443 check
    server k8s-master02 192.168.10.12:6443 check
    server k8s-master03 192.168.10.13:6443 check
EOF

# Restart HAProxy
sudo systemctl restart haproxy
sudo systemctl enable haproxy
```

---

## Cluster Installation with kubeadm

### Step 1: Initialize First Control Plane Node

**On `k8s-master01`**:

```bash
# Create kubeadm config
cat <<EOF | sudo tee kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.28.4
controlPlaneEndpoint: "k8s-api.eigrac.local:6443"
networking:
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"
etcd:
  local:
    dataDir: /var/lib/etcd
    extraArgs:
      quota-backend-bytes: "8589934592"  # 8 GB
apiServer:
  certSANs:
    - "k8s-api.eigrac.local"
    - "192.168.10.10"
    - "192.168.10.11"
    - "192.168.10.12"
    - "192.168.10.13"
  extraArgs:
    audit-log-path: /var/log/kubernetes/audit.log
    audit-log-maxage: "30"
    audit-log-maxbackup: "10"
    audit-log-maxsize: "100"
controllerManager:
  extraArgs:
    bind-address: "0.0.0.0"
scheduler:
  extraArgs:
    bind-address: "0.0.0.0"
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
maxPods: 250
EOF

# Initialize cluster
sudo kubeadm init --config=kubeadm-config.yaml --upload-certs

# Save output (join commands for control plane and workers)
# Example output:
#   Control plane join: kubeadm join k8s-api.eigrac.local:6443 --token ... \
#     --control-plane --certificate-key ...
#   Worker join: kubeadm join k8s-api.eigrac.local:6443 --token ...

# Set up kubectl for root user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 2: Join Additional Control Plane Nodes

**On `k8s-master02` and `k8s-master03`**:

```bash
# Use join command from Step 1 output
sudo kubeadm join k8s-api.eigrac.local:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --control-plane \
  --certificate-key <CERT_KEY>

# Set up kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 3: Verify Control Plane

```bash
# Check nodes (masters should be Ready after CNI installed)
kubectl get nodes

# Check control plane components
kubectl get pods -n kube-system

# Check etcd cluster
kubectl exec -n kube-system etcd-k8s-master01 -- etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  member list
```

---

## Networking Setup (Calico CNI)

### Install Calico

```bash
# Download Calico operator manifest
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

# Create custom resource for Calico installation
cat <<EOF | kubectl apply -f -
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.244.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
  nodeUpdateStrategy:
    type: RollingUpdate
EOF

# Wait for Calico to be ready
kubectl wait --for=condition=Ready pods --all -n calico-system --timeout=600s

# Verify Calico
kubectl get pods -n calico-system
kubectl get nodes  # Should show Ready
```

### Configure Network Policies (Optional, for Security)

```bash
# Example: Deny all traffic by default, allow explicit rules
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: eigrac-production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF
```

---

## Load Balancing (MetalLB)

### Install MetalLB

```bash
# Install MetalLB
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.11/config/manifests/metallb-native.yaml

# Wait for MetalLB to be ready
kubectl wait --namespace metallb-system \
  --for=condition=ready pod \
  --selector=app=metallb \
  --timeout=600s
```

### Configure MetalLB IP Address Pool

```bash
# Define IP range for LoadBalancer services
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: eigrac-ip-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.10.100-192.168.10.150  # 51 IPs for LoadBalancer services
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: eigrac-l2-advert
  namespace: metallb-system
spec:
  ipAddressPools:
  - eigrac-ip-pool
EOF
```

### Test MetalLB

```bash
# Deploy test service
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Check external IP assigned
kubectl get svc nginx  # Should show EXTERNAL-IP in 192.168.10.100-150 range

# Test connectivity
curl http://<EXTERNAL-IP>

# Cleanup
kubectl delete svc nginx
kubectl delete deployment nginx
```

---

## Storage Configuration

### Install Local Path Provisioner

```bash
# Install Local Path Provisioner (uses node local storage)
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

# Verify installation
kubectl get pods -n local-path-storage
kubectl get sc  # Should show 'local-path' StorageClass
```

### Configure Storage for MongoDB & Kafka

**Create Dedicated StorageClass for StatefulSets**:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-ssd-retain
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain  # Retain data if PVC deleted
parameters:
  nodePath: /mnt/disks/ssd  # Mount point for 4TB SSDs
EOF
```

**On each worker node, prepare SSD mount**:

```bash
# Format 4TB SSD (assuming /dev/nvme1n1)
sudo mkfs.ext4 /dev/nvme1n1

# Create mount point
sudo mkdir -p /mnt/disks/ssd

# Add to /etc/fstab
echo "/dev/nvme1n1  /mnt/disks/ssd  ext4  defaults  0  2" | sudo tee -a /etc/fstab

# Mount
sudo mount -a

# Verify
df -h /mnt/disks/ssd
```

---

## Join Worker Nodes

**On each worker node (`k8s-worker01` through `k8s-worker10`)**:

```bash
# Use worker join command from control plane init output
sudo kubeadm join k8s-api.eigrac.local:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

**Verify workers joined**:

```bash
# On master node
kubectl get nodes

# Expected output:
# NAME            STATUS   ROLES           AGE   VERSION
# k8s-master01    Ready    control-plane   30m   v1.28.4
# k8s-master02    Ready    control-plane   25m   v1.28.4
# k8s-master03    Ready    control-plane   20m   v1.28.4
# k8s-worker01    Ready    <none>          10m   v1.28.4
# ...
# k8s-worker10    Ready    <none>          2m    v1.28.4
```

---

## Monitoring Stack Setup

### Install Prometheus Operator

```bash
# Add Prometheus Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Create monitoring namespace
kubectl create namespace monitoring

# Install kube-prometheus-stack
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=100Gi \
  --set grafana.adminPassword='CHANGE_ME'
```

### Expose Grafana via LoadBalancer

```bash
kubectl patch svc prometheus-grafana -n monitoring \
  -p '{"spec": {"type": "LoadBalancer"}}'

# Get Grafana external IP
kubectl get svc prometheus-grafana -n monitoring
# Access Grafana: http://<EXTERNAL-IP>:80
# Username: admin, Password: CHANGE_ME
```

---

## Security Hardening

### 1. Enable RBAC (Default in k8s 1.28)

```bash
# Verify RBAC is enabled
kubectl api-versions | grep rbac
```

### 2. Create Service Accounts with Minimal Permissions

```bash
# Example: Service account for е-Играч backend
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eigrac-backend
  namespace: eigrac-production
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: eigrac-backend-role
  namespace: eigrac-production
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind:RoleBinding
metadata:
  name: eigrac-backend-rolebinding
  namespace: eigrac-production
subjects:
- kind: ServiceAccount
  name: eigrac-backend
  namespace: eigrac-production
roleRef:
  kind: Role
  name: eigrac-backend-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

### 3. Enable Pod Security Standards

```bash
# Enforce restricted policy for eigrac-production namespace
kubectl label namespace eigrac-production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted
```

### 4. Encrypt etcd Data at Rest

```bash
# Create encryption config
cat <<EOF | sudo tee /etc/kubernetes/pki/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: $(head -c 32 /dev/urandom | base64)
      - identity: {}
EOF

# Update kube-apiserver manifest on all control plane nodes
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
# Add to spec.containers[0].command:
#   - --encryption-provider-config=/etc/kubernetes/pki/encryption-config.yaml
# Add to spec.containers[0].volumeMounts:
#   - mountPath: /etc/kubernetes/pki/encryption-config.yaml
#     name: encryption-config
#     readOnly: true
# Add to spec.volumes:
#   - hostPath:
#       path: /etc/kubernetes/pki/encryption-config.yaml
#       type: File
#     name: encryption-config

# Restart kube-apiserver (automatic when manifest changes)
```

---

## Backup & Disaster Recovery

### Backup etcd

```bash
# Install etcdctl
ETCD_VER=v3.5.9
curl -L https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o etcd.tar.gz
tar xzvf etcd.tar.gz
sudo mv etcd-${ETCD_VER}-linux-amd64/etcdctl /usr/local/bin/

# Backup etcd
sudo ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot-$(date +%Y%m%d).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
sudo ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot-$(date +%Y%m%d).db
```

### Restore etcd (Disaster Recovery)

```bash
# Stop kube-apiserver on all control plane nodes
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

# Restore etcd snapshot
sudo ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot-20251025.db \
  --data-dir=/var/lib/etcd-restore

# Replace etcd data
sudo rm -rf /var/lib/etcd
sudo mv /var/lib/etcd-restore /var/lib/etcd

# Start kube-apiserver
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

---

## Operations & Maintenance

### Upgrade Kubernetes

```bash
# Upgrade control plane (one node at a time)
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.28.5-1.1
sudo apt-mark hold kubeadm

sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.28.5

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.28.5-1.1 kubectl=1.28.5-1.1
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Repeat for other control plane nodes and workers
```

### Scale Cluster (Add Worker Node)

```bash
# On new worker node, repeat prerequisites
# Join cluster
sudo kubeadm join k8s-api.eigrac.local:6443 --token <TOKEN> ...

# Verify
kubectl get nodes
```

---

## Document Control

**Version**: 1.0
**Author**: е-Играч DevOps Team
**Last Updated**: 2025-10-25
**Status**: Ready for Implementation

**Next Steps**:
1. Provision hardware (13 servers)
2. Install Ubuntu Server 22.04 LTS
3. Execute this guide step-by-step
4. Deploy е-Играч applications (MongoDB, Kafka, Flink, microservices)

---

**END OF DOCUMENT**
