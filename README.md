\# Kubernetes Learning Journey 🚀



This repository documents my hands-on learning journey with Kubernetes, focusing on real-world concepts, practical exercises, and DevOps best practices.



\---



\## 🎯 Goal



To build a solid, practical understanding of Kubernetes by:



\* Working hands-on with a local cluster (kind)

\* Using declarative YAML configurations

\* Understanding how applications are deployed, exposed, and managed

\* Learning debugging and operational concepts



\---



\## 🧱 What I’ve learned (Day 1)



\### Core Concepts



\* Pod, Deployment, ReplicaSet, Node

\* Desired state vs imperative commands



\### Networking



\* Service (ClusterIP vs NodePort)

\* Port-forwarding

\* Load balancing between Pods



\### YAML (Declarative approach)



\* Writing Deployment and Service manifests

\* Labels and selectors (critical concept)



\### Debugging



\* `kubectl get`, `describe`, `logs`, `events`

\* Understanding common failure states:



&#x20; \* CrashLoopBackOff

&#x20; \* ImagePullBackOff



\### Application Health \& Stability



\* Readiness probes (traffic control)

\* Liveness probes (self-healing)

\* CrashLoopBackOff behavior and backoff mechanism



\### Deployment Strategy



\* Rolling updates

\* Zero-downtime deployments using readiness probes



\---



\## 🔁 Architecture Overview



```

Client → Service → Pod → Container



Deployment → ReplicaSet → Pods

```



\---



\## 🧪 Labs



Hands-on exercises are located in:



```

labs/

&#x20; day-01-nginx-basics/

```



\---



\## 📂 Manifests



Reusable Kubernetes manifests:



```

manifests/

&#x20; deployments/

&#x20; services/

&#x20; configmaps/

&#x20; secrets/

```



\---



\## 🧠 Key Takeaways



\* Kubernetes is declarative – you define the desired state

\* Services provide stable access and load balancing

\* Readiness probes are essential for zero-downtime deployments

\* Liveness probes enable self-healing but must be used carefully

\* Debugging requires a structured approach



\---



\## 🛠️ Tools Used



\* Kubernetes (kind)

\* kubectl

\* Docker

\* PowerShell

\* Git \& GitHub



\---



\## 🚀 Next Steps



\* Deep dive into YAML structure and best practices

\* Pods vs Deployments (advanced understanding)

\* Labels \& selector design

\* Real-world debugging scenarios



\---



\## 📌 Note



This repository is part of a structured learning plan aimed at transitioning into a Cloud / DevOps / Platform Engineering role.



\---



\## 👤 Author



Ingvar Hardzei

Senior IT Engineer transitioning into Cloud / DevOps



