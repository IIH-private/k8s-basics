\# Day 14 – Config Injection (ConfigMap \& Secret)



\## Formål



Forstå hvordan konfiguration bliver injiceret i Pods via:



\* Environment variables (`env`, `envFrom`)

\* Volume mounts (ConfigMap/Secret som filer)



\---



\## 🔹 Env vs Volume (core forskel)



| Metode        | Type                  | Opdatering                 |

| ------------- | --------------------- | -------------------------- |

| env / envFrom | Environment variables | ❌ statisk (kræver restart) |

| volume mount  | Filer i container     | ✅ opdateres automatisk     |



👉 Husk:



\* Env = snapshot ved container start

\* Volume = live data (filer)



\---



\## 🔹 ConfigMap → envFrom



```yaml

apiVersion: v1

kind: ConfigMap

metadata:

&#x20; name: app-config

data:

&#x20; MODE: production

&#x20; LOG\_LEVEL: debug

\---

apiVersion: v1

kind: Pod

metadata:

&#x20; name: pod-env

spec:

&#x20; containers:

&#x20;   - name: app

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "echo MODE=$MODE; echo LOG\_LEVEL=$LOG\_LEVEL; sleep 3600"]

&#x20;     envFrom:

&#x20;       - configMapRef:

&#x20;           name: app-config

```



👉 Resultat:



```bash

kubectl logs pod-env

```



```text

MODE=production

LOG\_LEVEL=debug

```



\---



\## 🔹 ConfigMap → Volume (filer)



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: pod-volume

spec:

&#x20; containers:

&#x20;   - name: app

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "ls /etc/app-config; cat /etc/app-config/MODE; sleep 3600"]

&#x20;     volumeMounts:

&#x20;       - name: config-vol

&#x20;         mountPath: /etc/app-config

&#x20; volumes:

&#x20;   - name: config-vol

&#x20;     configMap:

&#x20;       name: app-config

```



👉 Resultat:



```bash

kubectl exec -it pod-volume -- ls /etc/app-config

kubectl exec -it pod-volume -- cat /etc/app-config/MODE

```



```text

MODE

LOG\_LEVEL

production

```



\---



\## 🔹 Opdatering af ConfigMap



```bash

kubectl apply -f configmap.yaml

```



\### Resultat:



\* Env vars → ❌ ændres ikke

\* Volume filer → ✅ opdateres automatisk



\---



\## 🔹 Restart effekt



```bash

kubectl delete pod pod-env

kubectl apply -f pod-env.yaml

```



👉 Env vars opdateres først ved:



\* ny Pod

\* container restart



\---



\## 🔹 CKAD Debug – læring



Fundne fejl i øvelse:



1\. Forkert ConfigMap navn:



```yaml

name: wrong-config   # ❌

```



2\. Case mismatch:



```yaml

mode vs MODE

```



3\. Volume name mismatch:



```yaml

config-vol vs config-volume

```



\---



\## 🔹 Vigtige læringspunkter



\* ConfigMap keys er case-sensitive

\* Kubernetes mapper ikke keys automatisk

\* Env vars opdateres ikke dynamisk

\* Volume mounts opdateres – men kun filer

\* Applikationen skal selv reagere på ændringer



\---



\## 🔹 Real-world praksis



| Situation                             | Valg          |

| ------------------------------------- | ------------- |

| statisk config                        | env           |

| dynamisk config (app kan reload)      | volume        |

| dynamisk config (app kan ikke reload) | env + restart |



\---



\## 🔹 Mental model



\* Env = procesniveau

\* Volume = filsystem



👉 Kubernetes opdaterer data – ikke din applikation



