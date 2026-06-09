# Architecture Diagram

## Kubernetes HA Cluster Architecture

```
                              ┌─────────────────┐
                              │  External        │
                              │  Clients/Users  │
                              └────────┬─────────┘
                                       │
                                       ▼
                              ┌─────────────────┐
                              │  Virtual IP     │
                              │  (Keepalived)   │
                              └────────┬─────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    ▼                  ▼                  ▼
              ┌──────────┐        ┌──────────┐      ┌──────────┐
              │ HAProxy  │        │ HAProxy  │      │ HAProxy  │
              │ LB       │        │ LB       │      │ LB       │
              └────┬─────┘        └────┬─────┘      └────┬─────┘
                   │                   │                  │
    ┌──────────────┼───────────────────┼──────────────────┐
    ▼              ▼                   ▼                  ▼
┌─────────┐   ┌─────────┐        ┌─────────┐      ┌─────────┐
│   CP1   │   │   CP2   │        │   CP3   │      │         │
│Control  │   │Control  │        │Control  │      │ Network │
│ Plane   │   │ Plane   │        │ Plane   │      │Policies │
└────┬────┘   └────┬────┘        └────┬────┘      │(Calico) │
     │             │                  │            │         │
     └─────────────┼──────────────────┘            └─────────┘
                   │
    ┌──────────────┼──────────────────┬──────────────────┐
    ▼              ▼                  ▼                  ▼
┌─────────┐   ┌─────────┐        ┌─────────┐      ┌─────────┐
│   W1    │   │   W2    │        │   W3    │      │  etcd   │
│ Worker  │   │ Worker  │        │ Worker  │      │ Cluster │
│ Node    │   │ Node    │        │ Node    │      │         │
└─────────┘   └─────────┘        └─────────┘      └─────────┘
```

## Component Details

### Load Balancing Layer
- **HAProxy**: Distributes traffic across API servers on control planes
- **Keepalived**: Maintains Virtual IP (VIP) for high availability
- Multiple HAProxy instances in active/backup configuration

### Control Plane Layer
- **3 Control Plane Nodes**: Full HA configuration
- **etcd Cluster**: Distributed state management
- **API Server**: Kubernetes API endpoints
- **Scheduler**: Pod scheduling decisions
- **Controller Manager**: Cluster state management

### Worker Node Layer
- **3 Worker Nodes**: Container runtime environments
- **Kubelet**: Node agent managing pod lifecycle
- **Container Runtime**: Docker/containerd
- **Kube-proxy**: Network proxy for services

### Networking Layer
- **Calico CNI**: Pod-to-pod networking
- **Service Networking**: ClusterIP, NodePort, LoadBalancer
- **Multi-subnet**: Supports multiple network subnets

## Data Flow

1. **Client Request** → Virtual IP (Keepalived managed)
2. **HAProxy** → Routes to available API servers
3. **API Server** → Processes requests, updates etcd
4. **Controllers** → Reconcile desired vs. actual state
5. **Scheduler** → Places pods on worker nodes
6. **Kubelet** → Creates/manages containers
7. **Calico** → Manages pod networking

## High Availability Features

- **3-node Control Plane**: Maintains quorum even with 1 node failure
- **Virtual IP Failover**: Automatic recovery if any node fails
- **Load Balancing**: Distributes load across control planes
- **etcd Replication**: State replicated across 3 nodes
- **Multi-path redundancy**: No single point of failure

