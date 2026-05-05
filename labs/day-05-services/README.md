\# Day 5 – Kubernetes Services \& Ingress



\## Overview



In this lab, I implemented Kubernetes networking concepts, focusing on how to expose applications inside and outside the cluster using:



\* \*\*Service (L4 routing)\*\*

\* \*\*Ingress (L7 routing)\*\*

\* \*\*Path-based routing\*\*

\* \*\*URL rewriting\*\*



The goal was to understand how traffic flows from a client to a Pod in a Kubernetes environment.



\---



\## Architecture



```

Client (Browser)

&#x20;  ↓

Ingress (L7 – path-based routing)

&#x20;  ↓

Service (L4 – load balancing)

&#x20;  ↓

Pods (nginx containers)

```



\---



\## Key Concepts



\### Service (Layer 4 – Transport)



A Service provides:



\* Stable IP address

\* Load balancing across Pods

\* Label-based Pod selection



Example:



```

Service → routes traffic based on IP + port

```



Important:



\* Does NOT understand HTTP paths (`/app1`, `/api`)

\* Works on TCP/UDP level



\---



\### Ingress (Layer 7 – Application)



Ingress provides:



\* HTTP/HTTPS routing

\* Path-based routing

\* Host-based routing

\* Integration with Ingress Controller (NGINX)



Example:



```

/app1 → Service 1

/app2 → Service 2

```



\---



\### Ingress Controller



Ingress resources do nothing by themselves.



An \*\*Ingress Controller\*\* (NGINX in this lab):



\* Reads Ingress YAML

\* Configures routing rules

\* Handles incoming HTTP traffic



\---



\### Rewrite (Important Concept)



Without rewrite:



```

Request: /app1

→ forwarded as /app1

→ nginx returns 404

```



With rewrite:



```

Request: /app1

→ rewritten to /

→ nginx returns default page

```



Annotation used:



```yaml

nginx.ingress.kubernetes.io/rewrite-target: /

```



\---



\## Implementation



\### 1. Deployments



Two nginx applications were deployed:



\* `nginx-deployment-1`

\* `nginx-deployment-2`



Each with different labels:



```yaml

app: nginx1

app: nginx2

```



\---



\### 2. Services



Two Services were created:



\* `nginx-service-1`

\* `nginx-service-2`



Each Service:



\* Selects Pods via labels

\* Exposes port 80 internally



\---



\### 3. Ingress



Ingress routes traffic:



```yaml

/app1 → nginx-service-1

/app2 → nginx-service-2

```



\---



\## Testing



Access via port-forward:



```

kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:80

```



Test in browser:



```

http://localhost:8080/app1

http://localhost:8080/app2

```



\---



\## Troubleshooting Learnings



\### 1. 404 Not Found



Cause:



\* Backend does not recognize path



Fix:



\* Use rewrite annotation



\---



\### 2. Service without endpoints



Check:



```

kubectl get endpoints

```



If `<none>`:



\* Label mismatch between Service and Pods



\---



\### 3. Ingress not working



Check:



```

kubectl describe ingress

kubectl get pods -n ingress-nginx

```



\---



\## Key Learnings



\* Service = L4 (IP + port routing)

\* Ingress = L7 (HTTP-aware routing)

\* Ingress requires a controller

\* Services provide abstraction over dynamic Pods

\* Rewrite is needed when backend does not support external paths



\---



\## Real-World Perspective



In production:



\* `LoadBalancer` exposes cluster externally

\* `Ingress` handles routing for multiple apps

\* `Service` ensures stable internal communication



Example:



```

Internet → LoadBalancer → Ingress → Services → Pods

```



\---



\## Conclusion



This lab demonstrates how Kubernetes separates concerns:



\* \*\*Ingress\*\* decides \*which application\*

\* \*\*Service\*\* decides \*which Pod\*

\* \*\*Pod\*\* runs the application



This separation enables scalability, flexibility, and resilience in real-world systems.



