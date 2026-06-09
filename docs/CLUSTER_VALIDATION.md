# Cluster Validation

## Pre-Deployment Checks

### Proxmox VE Verification

```bash
# Check Proxmox cluster status
pvecm status

# Verify network connectivity
ping -c 3 <control-plane-ip>
ping -c 3 <worker-node-ip>
```

### VM Template Verification

```bash
# Verify Ubuntu template exists
qm list

# Check template resources
qm config <template-vmid>
```

## Post-Deployment Validation

### Cluster Initialization

```bash
# Check Kubernetes cluster info
kubectl cluster-info

# Get cluster version
kubectl version

# Check API server status
kubectl get componentstatuses
```

### Node Status Validation

```bash
# Get all nodes
kubectl get nodes

# Detailed node information
kubectl get nodes -o wide

# Node descriptions
kubectl describe node <node-name>

# Expected output:
# NAME       STATUS   ROLES           AGE    VERSION
# cp-1       Ready    control-plane   10m    v1.30.0
# cp-2       Ready    control-plane   8m     v1.30.0
# cp-3       Ready    control-plane   6m     v1.30.0
# worker-1   Ready    <none>          4m     v1.30.0
# worker-2   Ready    <none>          3m     v1.30.0
# worker-3   Ready    <none>          2m     v1.30.0
```

### Pod Status Validation

```bash
# Get system pods
kubectl get pods -n kube-system

# Check pod distribution
kubectl get pods -n kube-system -o wide

# Verify all kube-system pods are running
kubectl get pods -n kube-system --field-selector=status.phase!=Running

# Should return: No resources found
```

### etcd Cluster Validation

```bash
# Check etcd member status
kubectl exec -n kube-system etcd-cp-1 -- etcdctl member list

# Expected: All members healthy

# Check etcd health
kubectl exec -n kube-system etcd-cp-1 -- etcdctl endpoint health

# Expected: All endpoints: healthy
```

### Calico/CNI Validation

```bash
# Get Calico nodes
kubectl get nodes -o custom-columns=NAME:.metadata.name,CALICO_STATUS:.status.conditions[*].message

# Check Calico pods
kubectl get pods -n calico-system

# Verify pod networking
kubectl run test-pod --image=nginx:latest
kubectl exec test-pod -- ping -c 3 <another-pod-ip>
```

### Service Networking

```bash
# Create a test service
kubectl run web --image=nginx:latest --expose --port=80

# Test service connectivity
kubectl run -it --rm debug --image=busybox:latest --restart=Never -- wget -qO- http://web

# Expected: HTML content from nginx
```

### HAProxy & Keepalived Validation

```bash
# Check HAProxy status (on load balancer node)
sudo systemctl status haproxy
sudo haproxy -c -f /etc/haproxy/haproxy.cfg

# Check Keepalived status
sudo systemctl status keepalived
ip addr show | grep <vip>

# Test VIP failover
sudo systemctl stop keepalived
# Verify VIP moves to another node
ip addr show | grep <vip>
```

### Proxy/API Server Connectivity

```bash
# Test API server through VIP
kubectl --server=https://<vip>:6443 cluster-info

# Check all API server endpoints
kubectl get endpoints kubernetes -n default
```

## Health Checks

### Control Plane Health

```bash
# Check all control plane components
kubectl get componentstatuses

# Check kube-apiserver
kubectl get pods -n kube-system -l component=kube-apiserver

# Check kube-controller-manager
kubectl get pods -n kube-system -l component=kube-controller-manager

# Check kube-scheduler
kubectl get pods -n kube-system -l component=kube-scheduler
```

### Worker Node Health

```bash
# Check all nodes ready
kubectl get nodes --no-headers | awk '{print $2}' | sort | uniq -c

# Expected: All nodes should show "Ready"

# Check for tainted nodes
kubectl describe nodes | grep Taints
```

### Event Log Review

```bash
# Check cluster events
kubectl get events -A --sort-by='.lastTimestamp'

# Check recent warnings/errors
kubectl get events -A --field-selector type=Warning
```

## Cluster Metrics

```bash
# Check resource usage (requires metrics-server)
kubectl top nodes
kubectl top pods -A

# Monitor node capacity
kubectl describe nodes | grep -A 5 "Allocatable"
```

## Common Validation Issues

| Issue | Solution |
|-------|----------|
| Node NotReady | Check kubelet service: `systemctl status kubelet` |
| Pod Pending | Check resource availability: `kubectl describe pod <pod>` |
| etcd errors | Verify etcd storage space and disk I/O |
| Network issues | Check Calico pods and routing: `calicoctl get nodes` |
| API connectivity | Verify VIP and HAProxy: `curl -k https://<vip>:6443` |

