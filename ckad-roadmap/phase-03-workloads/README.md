# Phase 3 – Core Workloads

## Goal

Learn how Kubernetes runs and manages workloads using Pods and higher-level controllers.

## Topics Covered

- Pods
- ReplicaSets
- Deployments
- Namespaces
- Labels
- Annotations
- Multi-container Pods
- Init Containers

## Skills Gained

- Create and manage Pods
- Scale applications with ReplicaSets
- Perform rolling updates and rollbacks with Deployments
- Organize resources using Namespaces
- Select resources using Labels and Selectors
- Add metadata with Annotations
- Build Pods containing multiple containers
- Initialize applications using Init Containers

## Common kubectl Commands

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl apply -f <file>.yaml
kubectl delete -f <file>.yaml
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl scale deployment <name> --replicas=<number>
```

## CKAD Focus

This phase covers the fundamental workload objects used throughout the CKAD exam. A solid understanding of Pods, controllers, metadata, and multi-container patterns is essential before moving on to application configuration.