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

---

## Lab 2.4 – kube-apiserver

### Objective

Understand the role of the kube-apiserver as the central communication component in Kubernetes.

### Key points

- The kube-apiserver is the entry point to the Kubernetes Cluster.
- All kubectl commands communicate with the kube-apiserver.
- The kube-apiserver authenticates users and authorizes requests.
- It validates Kubernetes objects before accepting them.
- Only the kube-apiserver reads from and writes directly to etcd.
- The kube-apiserver does not schedule Pods or run containers.

### Common kubectl commands

    kubectl get
    kubectl apply
    kubectl describe
    kubectl delete
    kubectl logs
    kubectl exec

### Summary

The kube-apiserver is the central communication hub of Kubernetes. Every request to the cluster passes through it, making it one of the most important components in the Kubernetes architecture.

---

## Lab 2.5 – etcd

### Objective

Understand the role of etcd as the Kubernetes database.

### Key points

- etcd is the distributed key-value database of Kubernetes.
- It stores the cluster's desired and current state.
- Kubernetes objects such as Pods, Deployments, Services, ConfigMaps, Secrets and Nodes are stored in etcd.
- etcd does not store container images or application data.
- Only the kube-apiserver reads from and writes directly to etcd.
- etcd does not schedule Pods or run containers.

### Summary

etcd is the single source of truth for the Kubernetes Cluster. It stores the complete cluster state, allowing Kubernetes components to work together and maintain the desired state of the system.

---

## Lab 2.6 – Scheduler

### Objective

Understand how the Kubernetes Scheduler selects the best Worker Node for a Pod.

### Key points

- The Scheduler is responsible for assigning Pods to Worker Nodes.
- It does not run Pods; it only selects the most suitable Node.
- Scheduling is performed in two phases:
  - Filtering: Remove Nodes that cannot run the Pod.
  - Scoring: Rank the remaining Nodes and select the best one.
- The Scheduler works with the desired state stored in etcd through the kube-apiserver.
- After a Node is selected, the kubelet on that Worker Node starts the Pod.

### Summary

The Kubernetes Scheduler is responsible for deciding where a Pod should run. It evaluates available Worker Nodes, filters out unsuitable candidates, scores the remaining Nodes, and assigns the Pod to the best available Node. The kubelet on the selected Worker Node is then responsible for starting the Pod.

---

## Lab 2.7 – Controller Manager

### Objective

Understand the role of the Kubernetes Controller Manager and how it maintains the desired state of the cluster.

### Key points

- The Controller Manager continuously compares the desired state with the current state.
- It detects differences between the two states.
- It creates or removes Kubernetes objects to reconcile the cluster state.
- The Controller Manager does not select Worker Nodes.
- The Scheduler selects the Node, and the kubelet starts the Pod.
- The Controller Manager consists of multiple specialized controllers, each responsible for a specific Kubernetes resource.

### Example

Desired state:

    Deployment replicas: 3

Current state:

    2 Pods running

Result:

    The Controller Manager detects the missing Pod and requests the creation of a new Pod. The Scheduler assigns it to a Worker Node, and the kubelet starts it.

### Summary

The Controller Manager is responsible for keeping the Kubernetes Cluster in the desired state. It continuously monitors the cluster and ensures that the actual state matches the user's declared configuration.

---

## Lab 2.8 – kubelet

### Objective

Understand the role of the kubelet as the agent running on every Worker Node.

### Key points

- A kubelet runs on every Worker Node.
- The kubelet continuously watches the kube-apiserver for Pods assigned to its Node.
- The kubelet requests the Container Runtime to create and start containers.
- The kubelet monitors the health of running Pods.
- The kubelet reports Pod status back to the kube-apiserver.
- The kubelet does not schedule Pods or communicate directly with the Scheduler.

### Summary

The kubelet is the Worker Node agent responsible for ensuring that Pods assigned to its Node are running correctly. It receives Pod assignments through the kube-apiserver, works with the Container Runtime to start containers, monitors Pod health, and continuously reports status back to the Kubernetes Control Plane.

---

## Lab 2.9 – kube-proxy

### Objective

Understand the role of kube-proxy in Kubernetes networking.

### Key points

- A kube-proxy runs on every Worker Node.
- kube-proxy manages networking rules for Kubernetes Services.
- It forwards traffic from a Service to the appropriate Pods.
- It provides load balancing across multiple Pod replicas.
- It updates networking rules when Pods are created, removed or receive new IP addresses.
- kube-proxy does not schedule Pods or start containers.

### Summary

The kube-proxy is the networking agent running on every Worker Node. It maintains the networking rules that allow Kubernetes Services to reliably route traffic to the correct Pods, even when Pod IP addresses change.

---

## Lab 2.10 – Container Runtime

### Objective

Understand the role of the Container Runtime in Kubernetes.

### Key points

- A Container Runtime runs on every Worker Node.
- The kubelet communicates with the Container Runtime.
- The Container Runtime downloads container images when needed.
- It creates, starts, stops and removes containers.
- Common Container Runtimes include containerd and CRI-O.
- The Container Runtime does not schedule Pods or manage the cluster state.

### Summary

The Container Runtime is responsible for running containers on Worker Nodes. It works closely with the kubelet, which instructs it to pull images and create or remove containers as required by the Kubernetes Control Plane.