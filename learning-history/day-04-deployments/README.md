\# Day 4 – Kubernetes Deployments



\## 🎯 Formål



Formålet med denne lab er at forstå og arbejde med Kubernetes Deployments:



\* sikre ønsket antal pods (replicas)

\* håndtere self-healing

\* udføre rollout (opdateringer)

\* kunne rollback til tidligere version



\---



\## 🧠 Hvad er et Deployment?



Et Deployment er en controller, der styrer pods via en ReplicaSet.



Den sikrer:



\* at det ønskede antal pods altid kører

\* at pods genoprettes hvis de fejler

\* at opdateringer sker kontrolleret (rolling updates)



\---



\## 📄 YAML eksempel



```yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&#x20; name: nginx-deployment

spec:

&#x20; replicas: 2

&#x20; selector:

&#x20;   matchLabels:

&#x20;     app: nginx

&#x20; template:

&#x20;   metadata:

&#x20;     labels:

&#x20;       app: nginx

&#x20;   spec:

&#x20;     containers:

&#x20;     - name: nginx

&#x20;       image: nginx:1.25

&#x20;       ports:

&#x20;       - containerPort: 80

```



\---



\## ⚙️ Commands



\### Deploy



```

kubectl apply -f deployment.yaml

```



\### Se resources



```

kubectl get deployments

kubectl get pods

```



\### Test self-healing



```

kubectl delete pod <pod-name>

```



\### Scaling



```

kubectl scale deployment nginx-deployment --replicas=3

```



\---



\## 🔄 Rollout



\### Opdater image



```

kubectl apply -f deployment.yaml

```



\### Status



```

kubectl rollout status deployment nginx-deployment

```



\### Historik



```

kubectl rollout history deployment nginx-deployment

```



\---



\## ⏪ Rollback



```

kubectl rollout undo deployment nginx-deployment

```



\---



\## 🧪 Hvad jeg har lært



\* Deployment styrer pods via ønsket tilstand (desired state)

\* replicas definerer antal pods

\* rollout opdaterer pods gradvist

\* rollback virker via revisionshistorik (ReplicaSets)

\* pods er ikke længere noget man styrer direkte



\---



\## ⚠️ Vigtige pointer



\* selector skal matche labels i template

\* Deployment bruger apiVersion: apps/v1 (ikke v1)

\* apply ændrer desired state, rollout er processen



\---



