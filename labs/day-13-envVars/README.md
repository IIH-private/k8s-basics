\# Day 13 - Environment Variables (env vars)



\## Goal



Learn how Kubernetes injects configuration into containers using:

\- env

\- envFrom

\- ConfigMaps

\- Secrets



Understand:

\- how applications receive configuration

\- difference between hardcoded values and env vars

\- when to use mounted files instead of env vars



\---



\# Simple Environment Variable



\## env-demo.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: env-demo

spec:

&#x20; containers:

&#x20;   - name: app

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "env \&\& sleep 3600"]

&#x20;     env:

&#x20;       - name: APP\_ENV

&#x20;         value: production

```



Apply:



```powershell

kubectl apply -f env-demo.yaml

```



Check logs:



```powershell

kubectl logs env-demo

```



\---



\# Multiple Environment Variables



\## env-demo-2.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: env-demo-2

spec:

&#x20; containers:

&#x20;   - name: app

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "env \&\& sleep 3600"]

&#x20;     env:

&#x20;       - name: APP\_ENV

&#x20;         value: production



&#x20;       - name: APP\_VERSION

&#x20;         value: "1.0"



&#x20;       - name: LOG\_LEVEL

&#x20;         value: debug

```



\---



\# ConfigMap as Environment Variables



\## configmap-env.yaml



```yaml

apiVersion: v1

kind: ConfigMap

metadata:

&#x20; name: app-config

data:

&#x20; APP\_ENV: production

&#x20; LOG\_LEVEL: info

&#x20; REGION: eu-west

```



Apply:



```powershell

kubectl apply -f configmap-env.yaml

```



\---



\# envFrom Example



\## env-from-configmap.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: env-from-configmap

spec:

&#x20; containers:

&#x20;   - name: app

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "env \&\& sleep 3600"]



&#x20;     envFrom:

&#x20;       - configMapRef:

&#x20;           name: app-config

```



Check logs:



```powershell

kubectl logs env-from-configmap

```



\---



\# Single Key from ConfigMap



\## env-single-key.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: env-single-key

spec:

&#x20; containers:

&#x20;   - name: app

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "env \&\& sleep 3600"]



&#x20;     env:

&#x20;       - name: MY\_REGION

&#x20;         valueFrom:

&#x20;           configMapKeyRef:

&#x20;             name: app-config

&#x20;             key: REGION

```



\---



\# Secret as Environment Variables



\## secret-env.yaml



```yaml

apiVersion: v1

kind: Secret

metadata:

&#x20; name: db-secret

stringData:

&#x20; DB\_USER: admin

&#x20; DB\_PASSWORD: supersecret

```



Apply:



```powershell

kubectl apply -f secret-env.yaml

```



\---



\# Secret Injection via envFrom



\## env-secret-demo.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: env-secret-demo

spec:

&#x20; containers:

&#x20;   - name: app

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "env \&\& sleep 3600"]



&#x20;     envFrom:

&#x20;       - secretRef:

&#x20;           name: db-secret

```



\---



\# Access Environment Variables Inside Container



Open shell:



```powershell

kubectl exec -it env-secret-demo -- sh

```



Show variables:



```sh

env

```



Single variable:



```sh

echo $DB\_USER

```



Exit shell:



```sh

exit

```



\---



\# Key Concepts Learned



\- `env` defines individual environment variables

\- `envFrom` imports all keys from a ConfigMap or Secret

\- Environment variables are injected only at container startup

\- Updating a ConfigMap does NOT automatically update env vars

\- Deployment restart is usually required after ConfigMap changes

\- Mounted files can update dynamically

\- ConfigMaps store non-sensitive configuration

\- Secrets store sensitive information

\- Base64 encoding is NOT encryption



\---



\# Important Production Understanding



Typical production setup:



\- ConfigMap → application settings

\- Secret → credentials

\- Deployment → injects both into Pods



\---



\# Useful Commands



```powershell

kubectl apply -f <file>

kubectl logs <pod>

kubectl exec -it <pod> -- sh

kubectl rollout restart deployment <deployment-name>

```



\---



\# Summary



Today I learned:

\- how Kubernetes injects configuration into containers

\- how env vars work internally

\- difference between env and envFrom

\- how ConfigMaps and Secrets connect to Pods

\- why env vars require Pod restart after changes

\- when mounted files are better than env vars

