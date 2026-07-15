# Phase 2 – Kubernetes Fundamentals

## Lab 2.1 – Cluster

### Objective

Understand what a Kubernetes Cluster is and how it is structured.

### Key points

- A Kubernetes Cluster consists of a Control Plane and one or more Worker Nodes.
- The Control Plane manages the cluster.
- Worker Nodes run application workloads (Pods).
- kubectl communicates with the Control Plane (via the kube-apiserver).
- A cluster can consist of a single node or multiple nodes.

## Lab 2.2 – Node

### Objective

Understand what a Kubernetes Node is and how it fits into a Kubernetes Cluster.

### Key points

- A Node is a physical or virtual machine in a Kubernetes Cluster.
- There are two node types: Control Plane and Worker Node.
- Worker Nodes run Pods.
- The Scheduler decides which Worker Node should run a Pod.
- A Node must be in the Ready state before new Pods can be scheduled on it.

### Useful commands

```bash
kubectl get nodes
kubectl get no
kubectl describe node <node-name>
```