# Phase 2 – Kubernetes Fundamentals

This phase focuses on understanding how Kubernetes works internally. A solid understanding of these concepts makes the core CKAD topics much easier later.

---

## Lab 2.1 – Cluster

### Objective

Understand what a Kubernetes Cluster is and how it is structured.

### Key points

- A Kubernetes Cluster consists of a Control Plane and one or more Worker Nodes.
- The Control Plane manages the cluster.
- Worker Nodes run application workloads (Pods).
- kubectl communicates with the Control Plane (via the kube-apiserver).
- A cluster can consist of a single node or multiple nodes.

---

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

    kubectl get nodes
    kubectl get no
    kubectl describe node <node-name>

---

## Lab 2.3 – Control Plane

### Objective

Understand the role of the Kubernetes Control Plane and its main components.

### Key points

- The Control Plane is the brain of the Kubernetes Cluster.
- It manages the desired state of the cluster.
- It continuously compares the desired state with the current state.
- Applications normally run on Worker Nodes, not on the Control Plane.
- All kubectl requests go through the kube-apiserver.

### Main components

- kube-apiserver
- etcd
- Scheduler
- Controller Manager

### Request flow

    kubectl
        ↓
    kube-apiserver
        ↓
    etcd
        ↓
    Controller Manager
        ↓
    Scheduler
        ↓
    Worker Node
        ↓
    Pod

### Summary

The Control Plane coordinates the entire Kubernetes Cluster. It receives requests through the kube-apiserver, stores the desired state in etcd, detects differences between the desired and current state, schedules workloads to Worker Nodes, and ensures that the cluster continuously matches the desired state.