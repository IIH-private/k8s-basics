\# Day 7 — Scaling, Rollouts and Rollbacks



\## Goal



Learn how Kubernetes handles:



\- Scaling applications

\- Rolling updates

\- Rollback to previous versions

\- Relationship between Deployment, ReplicaSet and Pods



\---



\# Key Concepts



\## Scaling



Scaling means changing the number of Pod replicas running for an application.



Example:



```bash

kubectl scale deployment nginx-deployment --replicas=3

```



Kubernetes then ensures that exactly 3 Pods are running.



\---



\## Deployment Architecture



```text

Deployment

&#x20;   ↓

ReplicaSet

&#x20;   ↓

Pods

&#x20;   ↓

Containers

```



\### Responsibilities



| Resource | Responsibility |

|---|---|

| Deployment | Manages rollout and rollback |

| ReplicaSet | Maintains desired Pod count |

| Pod | Runs containers |



\---



\# Commands Used



\## View resources



```bash

kubectl get deployments

kubectl get pods

kubectl get rs

```



\---



\## Scale application



Scale up:



```bash

kubectl scale deployment nginx-deployment --replicas=3

```



Scale down:



```bash

kubectl scale deployment nginx-deployment --replicas=1

```



\---



\## Rollout new version



Update image version inside deployment YAML:



```yaml

image: nginx:1.26

```



Apply changes:



```bash

kubectl apply -f deployment.yaml

```



\---



\## Watch rollout live



```bash

kubectl get pods -w

```



\---



\## Check rollout status



```bash

kubectl rollout status deployment/nginx-deployment

```



\---



\## Rollout history



```bash

kubectl rollout history deployment/nginx-deployment

```



\---



\## Rollback



```bash

kubectl rollout undo deployment/nginx-deployment

```



\---



\# Important Learnings



\## Why Kubernetes creates a new ReplicaSet



A new ReplicaSet is created whenever the Pod template changes.



This allows:

\- rolling updates

\- version history

\- rollback support



\---



\## Why old Pods are not removed immediately



Old Pods stay alive temporarily to:

\- avoid downtime

\- maintain availability

\- allow gradual replacement



\---



\## Why readinessProbe is important during rollout



Traffic should only be sent to Pods that are fully ready.



Without readiness checks:

\- users may hit broken Pods

\- rollout may cause downtime



\---



\# Mental Model



Kubernetes continuously works toward the desired state.



If:

\- a Pod crashes

\- a Pod is deleted

\- replicas change



Kubernetes automatically reconciles the cluster back to the desired state.



\---



\# Result



Today I learned:

\- how scaling works

\- how Deployments manage rollouts

\- how ReplicaSets manage Pods

\- how Kubernetes performs rolling updates

\- how rollback works

\- why readiness probes matter during deployment updates

