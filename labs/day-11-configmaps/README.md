\# Day 11 — ConfigMaps



\## Goal



Learn how Kubernetes ConfigMaps separate configuration from container images and how configuration can be injected into Pods using:



\- Environment variables

\- Mounted files (volumes)



\---



\# What is a ConfigMap?



A ConfigMap is a Kubernetes resource used to store non-sensitive configuration data outside the container image.



Typical examples:



\- Application settings

\- URLs

\- Feature flags

\- Environment names

\- Log levels



This makes container images more portable because the same image can be reused across environments like:



\- dev

\- test

\- prod



Only the configuration changes.



\---



\# Create ConfigMap



\## configmap.yaml



```yaml

apiVersion: v1

kind: ConfigMap

metadata:

&#x20; name: app-config

data:

&#x20; APP\_COLOR: blue

&#x20; APP\_ENV: production

&#x20; WELCOME\_MESSAGE: "Hello from ConfigMap"

```



Apply:



```powershell

kubectl apply -f configmap.yaml

```



Verify:



```powershell

kubectl get configmaps

```



Describe:



```powershell

kubectl describe configmap app-config

```



\---



\# ConfigMap as Environment Variables



\## pod-env.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: env-demo-pod

spec:

&#x20; containers:

&#x20;   - name: env-demo

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "env \&\& sleep 3600"]



&#x20;     env:

&#x20;       - name: APP\_COLOR

&#x20;         valueFrom:

&#x20;           configMapKeyRef:

&#x20;             name: app-config

&#x20;             key: APP\_COLOR



&#x20;       - name: APP\_ENV

&#x20;         valueFrom:

&#x20;           configMapKeyRef:

&#x20;             name: app-config

&#x20;             key: APP\_ENV

```



Apply:



```powershell

kubectl apply -f pod-env.yaml

```



Check logs:



```powershell

kubectl logs env-demo-pod

```



You should see:



\- APP\_COLOR

\- APP\_ENV



\---



\# ConfigMap as Mounted Files



\## multi-configmap.yaml



```yaml

apiVersion: v1

kind: ConfigMap

metadata:

&#x20; name: file-config

data:

&#x20; app.properties: |

&#x20;   color=green

&#x20;   environment=dev

&#x20;   version=1.0



&#x20; database.properties: |

&#x20;   host=db.company.local

&#x20;   port=5432

```



Apply:



```powershell

kubectl apply -f multi-configmap.yaml

```



\---



\# Mount ConfigMap as Volume



\## pod-volume.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: volume-demo-pod

spec:

&#x20; containers:

&#x20;   - name: volume-demo

&#x20;     image: busybox

&#x20;     command: \["sh", "-c", "sleep 3600"]



&#x20;     volumeMounts:

&#x20;       - name: config-volume

&#x20;         mountPath: /etc/config



&#x20; volumes:

&#x20;   - name: config-volume

&#x20;     configMap:

&#x20;       name: file-config

```



Apply:



```powershell

kubectl apply -f pod-volume.yaml

```



Enter Pod:



```powershell

kubectl exec -it volume-demo-pod -- sh

```



List files:



```sh

ls /etc/config

```



Expected files:



```text

app.properties

database.properties

```



Read files:



```sh

cat /etc/config/app.properties

```



```sh

cat /etc/config/database.properties

```



\---



\# Important Concept



ConfigMap keys automatically become filenames.



Example:



```yaml

data:

&#x20; app.properties: |

```



becomes:



```text

/etc/config/app.properties

```



\---



\# Dynamic Updates



Edit ConfigMap:



```powershell

kubectl edit configmap file-config

```



Change:



```text

version=2.0

```



Wait about 10–30 seconds.



Read file again inside the Pod:



```sh

cat /etc/config/app.properties

```



The mounted file updates automatically.



\---



\# Critical Difference



\## Environment variables



Changes are NOT updated automatically inside a running Pod.



The Pod must be restarted.



\---



\## Mounted files (volumes)



Changes ARE updated automatically.



\---



\# Debugging



List ConfigMaps:



```powershell

kubectl get configmaps

```



Describe ConfigMap:



```powershell

kubectl describe configmap app-config

```



Describe Pod:



```powershell

kubectl describe pod env-demo-pod

```



\---



\# Cleanup



```powershell

kubectl delete pod env-demo-pod

kubectl delete pod volume-demo-pod



kubectl delete configmap app-config

kubectl delete configmap file-config

```



\---



\# Key Takeaways



\- ConfigMaps externalize configuration

\- Same image can be reused across environments

\- ConfigMaps can be injected as:

&#x20; - environment variables

&#x20; - mounted files

\- Mounted files update automatically

\- Environment variables require Pod restart

\- ConfigMap keys become filenames when mounted as volumes



\---



\# Mini Quiz



1\. Why do ConfigMaps make images more portable?



2\. What is the difference between:

&#x20;  - environment variables

&#x20;  - mounted files



3\. When must Pods be restarted?



4\. How do ConfigMap keys become files?



5\. Why are ConfigMaps better than hardcoded configuration?



6\. What happens if a ConfigMap is deleted while a Pod is still running?



\---

