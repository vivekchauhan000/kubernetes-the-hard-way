# kubernetes-the-hard-way
A hands-on project for setting up Kubernetes from scratch the hard way, manually configuring each component including etcd, control plane, worker nodes, and networking without automation tools.A step-by-step Kubernetes cluster setup from scratch following Kelsey Hightower's guide. Covers TLS certificates, etcd cluster.


# ☸️ Kubernetes The Hard Way

![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28+-326CE5.svg?logo=kubernetes)
![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC.svg?logo=terraform)
![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)
![Status](https://img.shields.io/badge/Status-Active-success.svg)

A hands-on project for setting up Kubernetes from scratch the hard way,
manually configuring each component including etcd, control plane, worker
nodes, and networking without automation tools.

> 💡 Inspired by [Kelsey Hightower's](https://github.com/kelseyhightower/kubernetes-the-hard-way)
> original guide — rebuilt and documented step by step.

---

## 📋 Table of Contents

- [About](#about)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Tech Stack](#tech-stack)
- [Setup Guide](#setup-guide)
- [Components](#components)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

---

## 📌 About

This project walks through bootstrapping a **production-grade Kubernetes
cluster** from scratch — without using automated installers like `kubeadm`,
`kops`, or managed services.

The goal is to deeply understand every component of Kubernetes by manually
configuring each part of the cluster including **TLS certificates**, **etcd**,
**control plane**, **worker nodes**, **networking**, and **DNS**.

---

## 🏗️ Architecture

```
                        ┌─────────────────────┐
                        │   Load Balancer      │
                        │   (API Server LB)    │
                        └────────┬────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
     ┌────────▼──────┐  ┌────────▼──────┐  ┌───────▼───────┐
     │  Controller-0  │  │  Controller-1  │  │  Controller-2  │
     │  (Master Node) │  │  (Master Node) │  │  (Master Node) │
     └───────────────┘  └───────────────┘  └───────────────┘
              │                  │                  │
     ┌────────▼──────────────────▼──────────────────▼────────┐
     │                    etcd Cluster                        │
     └────────────────────────────────────────────────────────┘
              │
     ┌────────┴───────┐
     │                │
┌────▼────┐      ┌────▼────┐
│ Worker-0 │      │ Worker-1 │
│  (Node)  │      │  (Node)  │
└─────────┘      └─────────┘
```

---

## ✅ Prerequisites

Before you begin, make sure you have:

| Tool | Version | Purpose |
|---|---|---|
| ☁️ **GCP/AWS/Azure** | Latest | Cloud infrastructure |
| 🏗️ **Terraform** | v1.0+ | Infrastructure provisioning |
| 🔐 **cfssl & cfssljson** | v1.6+ | TLS certificates |
| ⚙️ **kubectl** | v1.28+ | Kubernetes CLI |
| 🖥️ **tmux** | Latest | Terminal multiplexer |
| 🐧 **Linux/Mac** | - | Operating system |

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| ☸️ **Kubernetes v1.28+** | Container orchestration |
| 🗄️ **etcd v3.5+** | Distributed key-value store |
| 🔐 **cfssl** | TLS certificate generation |
| 🏗️ **Terraform** | Infrastructure as Code |
| 🐳 **containerd** | Container runtime |
| 🌐 **CNI Plugins** | Pod networking |
| ⚖️ **CoreDNS** | Cluster DNS |
| ☁️ **GCP/AWS** | Cloud provider |

---

## 🚀 Setup Guide

### Step 1 — Install Client Tools

```bash
# Install cfssl
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson

chmod +x cfssl cfssljson
sudo mv cfssl cfssljson /usr/local/bin/

# Verify
cfssl version
```

### Step 2 — Provision Infrastructure

```bash
# Initialize Terraform
terraform init

# Plan infrastructure
terraform plan

# Apply infrastructure
terraform apply
```

### Step 3 — Generate TLS Certificates

```bash
# Generate CA certificate
cfssl gencert -initca ca-csr.json | cfssljson -bare ca

# Generate Admin certificate
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```

### Step 4 — Bootstrap etcd Cluster

```bash
# Download etcd
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.5.9/etcd-v3.5.9-linux-amd64.tar.gz"

# Extract and install
tar -xvf etcd-v3.5.9-linux-amd64.tar.gz
sudo mv etcd-v3.5.9-linux-amd64/etcd* /usr/local/bin/

# Start etcd service
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

### Step 5 — Bootstrap Control Plane

```bash
# Download Kubernetes binaries
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.28.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.28.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.28.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.28.0/bin/linux/amd64/kubectl"

# Make executable
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```

### Step 6 — Bootstrap Worker Nodes

```bash
# Install container runtime
sudo apt-get update
sudo apt-get -y install socat conntrack ipset

# Download and install containerd
wget -q --show-progress --https-only --timestamping \
  https://github.com/containerd/containerd/releases/download/v1.7.0/containerd-1.7.0-linux-amd64.tar.gz

# Start worker services
sudo systemctl daemon-reload
sudo systemctl enable containerd kubelet kube-proxy
sudo systemctl start containerd kubelet kube-proxy
```

### Step 7 — Verify Cluster

```bash
# Check cluster nodes
kubectl get nodes

# Check system pods
kubectl get pods -n kube-system

# Check cluster info
kubectl cluster-info
```

---

## 🔧 Components

| Component | Description | Status |
|---|---|---|
| 🔐 **TLS Certificates** | PKI infrastructure for all components | ✅ |
| 🗄️ **etcd Cluster** | Distributed key-value store | ✅ |
| ⚙️ **API Server** | Kubernetes API endpoint | ✅ |
| 🎮 **Controller Manager** | Manages cluster controllers | ✅ |
| 📅 **Scheduler** | Pod scheduling | ✅ |
| 👷 **Worker Nodes** | Run containerized workloads | ✅ |
| 🌐 **Pod Networking** | CNI plugin configuration | ✅ |
| 🔍 **CoreDNS** | Cluster DNS resolution | ✅ |
| 🔑 **RBAC** | Role-based access control | ✅ |

---

## 📁 Project Structure

```
kubernetes-the-hard-way/
│
├── terraform/
│   ├── main.tf                   # Main infrastructure
│   ├── variables.tf              # Input variables
│   └── outputs.tf                # Output values
│
├── certs/
│   ├── ca-config.json            # CA configuration
│   ├── ca-csr.json               # CA certificate signing request
│   └── admin-csr.json            # Admin CSR
│
├── configs/
│   ├── kube-apiserver.service    # API server service file
│   ├── kube-controller-manager.service
│   ├── kube-scheduler.service
│   ├── kubelet.service
│   └── kube-proxy.service
│
├── scripts/
│   ├── 01-install-tools.sh       # Install client tools
│   ├── 02-provision-infra.sh     # Provision infrastructure
│   ├── 03-generate-certs.sh      # Generate TLS certs
│   ├── 04-bootstrap-etcd.sh      # Bootstrap etcd
│   ├── 05-bootstrap-control.sh   # Bootstrap control plane
│   ├── 06-bootstrap-workers.sh   # Bootstrap worker nodes
│   └── 07-configure-dns.sh       # Configure CoreDNS
│
├── docs/
│   └── setup-guide.md            # Detailed setup documentation
│
├── README.md                     # Project documentation
└── LICENSE                       # Apache 2.0 License
```

---

## 🧪 Smoke Tests

```bash
# Test data encryption
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"

# Test deployments
kubectl create deployment nginx --image=nginx

# Test port forwarding
kubectl port-forward nginx-xxxx 8080:80

# Test logs
kubectl logs nginx-xxxx

# Test exec
kubectl exec -it nginx-xxxx -- nginx -v
```

---

## 🤝 Contributing

Contributions are welcome!

1. Fork the repository
2. Create your feature branch
   (`git checkout -b feature/NewFeature`)
3. Commit your changes
   (`git commit -m 'Add NewFeature'`)
4. Push to the branch
   (`git push origin feature/NewFeature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the
[Apache 2.0 License](LICENSE).

---

## 👤 Author

**Vivek Chauhan**
GitHub: [@vivekchauhan000](https://github.com/vivekchauhan000)

---

## 🔗 References

- [Kubernetes The Hard Way - Kelsey Hightower](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [Kubernetes Documentation](https://kubernetes.io/docs)
- [etcd Documentation](https://etcd.io/docs)
- [Terraform Documentation](https://developer.hashicorp.com/terraform)

---

⭐ If you found this helpful, please give it a star!

---

⭐ If you found this helpful, please give it a star!
