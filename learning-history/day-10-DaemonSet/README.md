\# Day 10 — Kubernetes DaemonSet



\## Goal

Learn how DaemonSets work in Kubernetes and understand how they differ from Deployments.



\---



\# What is a DaemonSet?



A DaemonSet ensures that a Pod runs on every node in the cluster.



When a new node joins:

\- Kubernetes automatically creates a new Pod on that node.



When a node is removed:

\- The Pod is automatically removed.



\---



\# Deployment vs DaemonSet



| Deployment | DaemonSet |

|---|---|

| Fixed number of replicas | One Pod per node |

| App workloads | Node-level services |

| Replica-based | Node-based |

| Manual scaling | Automatic scaling with nodes |



\---



\# Typical DaemonSet Use Cases



\- Monitoring agents

\- Logging agents

\- Security agents

\- Networking components

\- Storage plugins



Examples:

\- Prometheus Node Exporter

\- Fluentd

\- Filebeat

\- Falco



\---



\# DaemonSet Example



```yaml

apiVersion: apps/v1

kind: DaemonSet



metadata:

&#x20; name: nginx-daemonset



spec:

&#x20; selector:

&#x20;   matchLabels:

&#x20;     app: nginx-daemon



&#x20; template:

&#x20;   metadata:

&#x20;     labels:

&#x20;       app: nginx-daemon



&#x20;   spec:

&#x20;     tolerations:

&#x20;     - operator: Exists



&#x20;     containers:

&#x20;     - name: nginx

&#x20;       image: nginx:latest

