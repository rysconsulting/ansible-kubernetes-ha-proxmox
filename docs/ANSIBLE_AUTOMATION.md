# Ansible Automation

## Playbook Structure

```
ansible/
├── site.yml                 # Main playbook
├── inventory/
│   ├── hosts               # Inventory file
│   └── hosts.example       # Example inventory
├── group_vars/
│   ├── all.yml             # Variables for all hosts
│   ├── control_planes.yml  # Control plane variables
│   └── workers.yml         # Worker node variables
├── host_vars/
│   ├── cp-1.yml            # CP-1 specific variables
│   └── ...
├── roles/
│   ├── common/             # Common role (all nodes)
│   ├── load_balancer/      # HAProxy + Keepalived
│   ├── kube_prereqs/       # Kubernetes prerequisites
│   ├── kube_control/       # Control plane setup
│   ├── kube_worker/        # Worker node setup
│   └── kube_cni/           # Calico CNI installation
└── templates/
    ├── haproxy.cfg.j2
    ├── keepalived.conf.j2
    └── ...
```

## Playbook Execution

### Full Cluster Deployment

```bash
# Run complete deployment
ansible-playbook site.yml -i inventory/hosts

# Verbose output
ansible-playbook site.yml -i inventory/hosts -v

# Very verbose (includes system calls)
ansible-playbook site.yml -i inventory/hosts -vv

# Extra verbose (SSH connection details)
ansible-playbook site.yml -i inventory/hosts -vvv
```

### Role-Based Execution

```bash
# Run only load balancer setup
ansible-playbook site.yml -i inventory/hosts --tags load_balancer

# Run only control plane setup
ansible-playbook site.yml -i inventory/hosts --tags control_plane

# Run only worker setup
ansible-playbook site.yml -i inventory/hosts --tags worker

# Run only Calico CNI
ansible-playbook site.yml -i inventory/hosts --tags cni
```

### Check Mode (Dry Run)

```bash
# Check what would be changed without making changes
ansible-playbook site.yml -i inventory/hosts --check

# Check mode with verbose output
ansible-playbook site.yml -i inventory/hosts --check -v
```

### Step-by-Step Execution

```bash
# Run playbook step-by-step, prompting after each task
ansible-playbook site.yml -i inventory/hosts --step

# Run specific starting task
ansible-playbook site.yml -i inventory/hosts --start-at-task="Task Name"
```

## Inventory Management

### Inventory Structure

```ini
# inventory/hosts
[control_planes]
cp-1 ansible_host=<ip> kubernetes_role=control-plane
cp-2 ansible_host=<ip> kubernetes_role=control-plane
cp-3 ansible_host=<ip> kubernetes_role=control-plane

[workers]
worker-1 ansible_host=<ip> kubernetes_role=worker
worker-2 ansible_host=<ip> kubernetes_role=worker
worker-3 ansible_host=<ip> kubernetes_role=worker

[load_balancers]
lb-1 ansible_host=<vip> ansible_user=root

[kubernetes:children]
control_planes
workers

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### Dynamic Inventory

```bash
# Use dynamic inventory from Proxmox
ansible-playbook site.yml -i proxmox.py

# List hosts from inventory
ansible-inventory -i inventory/hosts --list

# Host variables
ansible-inventory -i inventory/hosts --host cp-1
```

## Variables and Configuration

### Global Variables (group_vars/all.yml)

```yaml
---
# Kubernetes versions
kubernetes_version: "1.30"
kubeadm_version: "1.30.0-1.0"
kubelet_version: "1.30.0-1.0"
kubectl_version: "1.30.0-1.0"

# Network configuration
pod_network_cidr: "10.244.0.0/16"
service_cidr: "10.96.0.0/12"

# VIP Configuration
vip: "192.168.1.100"
vip_interface: "eth0"

# Calico
calico_version: "v3.27.0"

# Container runtime
container_runtime: "containerd"
containerd_version: "1.7.0"
```

### Role-Based Variables

```bash
# Control plane specific
group_vars/control_planes.yml

# Worker specific
group_vars/workers.yml

# Host specific
host_vars/cp-1.yml
```

## Ansible Roles

### Common Role
- System package updates
- SSH configuration
- NTP synchronization
- Firewall setup
- Container runtime installation

### Load Balancer Role
- HAProxy installation and configuration
- Keepalived setup
- Virtual IP configuration
- Health checks

### Kubernetes Prerequisite Role
- CRI (Container Runtime Interface) setup
- kubelet, kubeadm, kubectl installation
- Kernel module configuration
- System parameter tuning

### Control Plane Role
- kubeadm init (bootstrap first control plane)
- kubeadm join (additional control planes)
- CNI plugin placeholder
- kubelet service setup

### Worker Node Role
- kubeadm join (join worker nodes)
- kubelet configuration
- Container runtime configuration

### Calico CNI Role
- Calico operator installation
- Calico custom resources
- Network policies
- IP pool configuration

## Advanced Automation

### Conditional Execution

```bash
# Run only on first control plane
--limit "cp-1"

# Run on all except cp-1
--limit "!cp-1"

# Run on multiple specific hosts
--limit "cp-1,cp-2"
```

### Callback Plugins

```bash
# Enable detailed profiling
ANSIBLE_STDOUT_CALLBACK=profile_tasks ansible-playbook site.yml -i inventory/hosts

# Human-readable output
ANSIBLE_STDOUT_CALLBACK=yaml ansible-playbook site.yml -i inventory/hosts

# JSON output
ANSIBLE_STDOUT_CALLBACK=json ansible-playbook site.yml -i inventory/hosts
```

### Error Handling

```bash
# Continue on errors
ansible-playbook site.yml -i inventory/hosts --force-handlers

# Retry failed tasks
ansible-playbook site.yml -i inventory/hosts --retry-failed

# Ignore all errors
ansible-playbook site.yml -i inventory/hosts --force
```

## Troubleshooting Automation

### Debug Tasks

```bash
# Enable verbose debugging
ansible-playbook site.yml -i inventory/hosts -vvv

# Add debug modules to playbook
- name: Debug variable
  debug:
    var: variable_name
```

### Connection Issues

```bash
# Test inventory connectivity
ansible all -i inventory/hosts -m ping

# SSH connectivity test
ansible all -i inventory/hosts -m command -a "uptime"

# Check SSH key authentication
ansible all -i inventory/hosts -m authorized_key -a "user=ubuntu state=present"
```

### Log Review

```bash
# Enable Ansible logging
export ANSIBLE_LOG_PATH=./ansible.log
ansible-playbook site.yml -i inventory/hosts

# Check logs
tail -f ansible.log
```

## Idempotency

All playbooks are designed to be idempotent:

```bash
# First run
ansible-playbook site.yml -i inventory/hosts

# Second run (should report no changes)
ansible-playbook site.yml -i inventory/hosts

# Check mode on deployed cluster
ansible-playbook site.yml -i inventory/hosts --check

# Expected: All tasks show "ok" (no changes)
```

## Best Practices

- ✅ Use inventory variables for environment-specific config
- ✅ Use role-based organization for maintainability
- ✅ Always test with `--check` first
- ✅ Use tags for selective execution
- ✅ Maintain idempotent tasks
- ✅ Keep playbooks version-controlled
- ✅ Document variables and their purposes
- ✅ Use `--step` for new or modified playbooks

