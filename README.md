# Kubernetes HA Cluster on Proxmox VE

Production-ready High Availability Kubernetes platform deployed on Proxmox VE using Infrastructure as Code principles.

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Cluster Topology](#cluster-topology)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Deployment Workflow](#deployment-workflow)
- [Validation](#validation)
- [Configuration](#configuration)
- [Automation Highlights](#automation-highlights)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Features

- ✅ Kubernetes v1.30
- ✅ 3 Control Plane Nodes
- ✅ 3 Worker Nodes
- ✅ HAProxy Load Balancing
- ✅ Keepalived Virtual IP Failover
- ✅ Calico CNI Networking
- ✅ Full Ansible Automation
- ✅ Multi-Subnet Architecture
- ✅ Repeatable Infrastructure Deployment

## Architecture

```
Client
   │
   ▼
Virtual IP (VIP)
   │
   ▼
HAProxy + Keepalived
   │
   ▼
┌─────────────────┐
│ Control Planes  │
│ CP1 CP2 CP3     │
└─────────────────┘
   │
   ▼
┌─────────────────┐
│ Worker Nodes    │
│ W1 W2 W3        │
└─────────────────┘
```

## Technology Stack

| Component | Purpose |
|-----------|---------|
| Kubernetes | Container Orchestration |
| Ansible | Automation |
| Proxmox VE | Virtualization Platform |
| HAProxy | Load Balancer |
| Keepalived | VIP Failover |
| Calico | Container Networking |

## Cluster Topology

| Node | Role |
|------|------|
| cp-1 | Control Plane |
| cp-2 | Control Plane |
| cp-3 | Control Plane |
| worker-1 | Worker |
| worker-2 | Worker |
| worker-3 | Worker |

## Prerequisites

- Proxmox VE 8.0 or higher
- Ubuntu 22.04 LTS template
- Ansible 2.10+
- Network connectivity between all nodes
- Minimum 2 CPU cores per VM
- Minimum 2GB RAM per VM
- At least 30GB storage per VM

## Quick Start

1. **Clone this repository:**
   ```bash
   git clone https://github.com/yourusername/ansible-kubernetes-ha-proxmox.git
   cd ansible-kubernetes-ha-proxmox
   ```

2. **Configure inventory:**
   ```bash
   cp inventory/hosts.example inventory/hosts
   # Edit inventory/hosts with your environment details
   ```

3. **Run the deployment playbook:**
   ```bash
   ansible-playbook site.yml -i inventory/hosts
   ```

## Deployment Workflow

1. Create Ubuntu Template
2. Configure Proxmox SDN
3. Provision Virtual Machines
4. Deploy HAProxy + Keepalived
5. Prepare Kubernetes Nodes
6. Bootstrap Control Plane
7. Join Additional Control Planes
8. Join Worker Nodes
9. Install Calico
10. Validate Cluster Health

## Validation

Verify cluster health with:

```bash
kubectl get nodes
```

Expected result:

```
NAME       STATUS   ROLES           AGE    VERSION
cp-1       Ready    control-plane   10m    v1.30.0
cp-2       Ready    control-plane   8m     v1.30.0
cp-3       Ready    control-plane   6m     v1.30.0
worker-1   Ready    <none>          4m     v1.30.0
worker-2   Ready    <none>          3m     v1.30.0
worker-3   Ready    <none>          2m     v1.30.0
```

Check pod status:

```bash
kubectl get pods -A
kubectl get endpoints -A
```

## Configuration

### Inventory Configuration

Edit `inventory/hosts` to set:
- Control plane IPs
- Worker node IPs
- Virtual IP (VIP) address
- Network interface names
- Domain names (if applicable)

### Ansible Variables

Key variables in `group_vars/`:
- `kubernetes_version`: Target Kubernetes version
- `pod_network_cidr`: Pod network CIDR block
- `service_cidr`: Service network CIDR block
- `calico_version`: Calico CNI version

## Automation Highlights

- **Infrastructure as Code** — All infrastructure defined declaratively
- **Idempotent Ansible Playbooks** — Safe to re-run without side effects
- **Automated HA Kubernetes Deployment** — Complete cluster setup in one run
- **Automated Cluster Expansion** — Easy to add new nodes
- **Repeatable Environment Provisioning** — Consistent deployments across environments

## Troubleshooting

### Common Issues

**Control plane not ready:**
```bash
kubectl describe node cp-1
kubectl logs -n kube-system -l component=kubelet
```

**Pod network issues:**
```bash
kubectl get nodes -o wide
kubectl get pods -n kube-system
calicoctl get nodes
```

**LoadBalancer connectivity:**
```bash
# Check HAProxy status
systemctl status haproxy
haproxy -c -f /etc/haproxy/haproxy.cfg

# Check Keepalived status
systemctl status keepalived
```

For more detailed debugging, check the Proxmox VE node logs and Ansible playbook output.

## Screenshots

### Architecture Diagram

See [Architecture Diagram](docs/ARCHITECTURE_DIAGRAM.md) for detailed visual representation of the cluster topology and component relationships.

### Cluster Validation

See [Cluster Validation](docs/CLUSTER_VALIDATION.md) for screenshots and steps to verify successful cluster deployment.

### Ansible Automation

See [Ansible Automation](docs/ANSIBLE_AUTOMATION.md) for insights into the automation workflows and playbook execution.

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Author

**Roy Sakai**  
DevOps Engineer  
Kubernetes • Ansible • Proxmox • Automation

## License

This project is licensed under the MIT License - see the LICENSE file for details.