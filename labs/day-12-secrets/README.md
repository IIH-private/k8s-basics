\# Day 12 - Kubernetes Secrets



\## Goal



Learn how Kubernetes Secrets work, how they differ from ConfigMaps, and how Pods consume sensitive data securely.



\---



\## What I Learned



\- What Kubernetes Secrets are

\- Difference between ConfigMaps and Secrets

\- Difference between encoding and encryption

\- Why base64 is NOT security

\- How to create Secrets

\- How Pods consume Secrets

\- Using Secrets as environment variables

\- Mounting Secrets as files

\- Difference between `data` and `stringData`

\- Why Secrets should not be stored in Git repositories

\- Why enterprises use external secret management systems



\---



\# Create Secret Using YAML



\## secret.yaml



```yaml

apiVersion: v1

kind: Secret

metadata:

&#x20; name: demo-secret



type: Opaque



data:

&#x20; username: YWRtaW4=

&#x20; password: c2VjcmV0MTIz

```



\---



\# Apply Secret



```powershell

kubectl apply -f secret.yaml

```



\---



\# View Secrets



```powershell

kubectl get secrets



kubectl describe secret demo-secret



kubectl get secret demo-secret -o yaml

```



\---



\# Decode Base64 in PowerShell



```powershell

\[System.Text.Encoding]::UTF8.GetString(

&#x20;   \[System.Convert]::FromBase64String("YWRtaW4=")

)

```



\---



\# Pod Using Secret as Environment Variables



\## pod-secret-env.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: secret-env-pod



spec:

&#x20; containers:

&#x20;   - name: demo

&#x20;     image: nginx



&#x20;     env:

&#x20;       - name: USERNAME

&#x20;         valueFrom:

&#x20;           secretKeyRef:

&#x20;             name: demo-secret

&#x20;             key: username



&#x20;       - name: PASSWORD

&#x20;         valueFrom:

&#x20;           secretKeyRef:

&#x20;             name: demo-secret

&#x20;             key: password

```



\---



\# Deploy Pod



```powershell

kubectl apply -f pod-secret-env.yaml

```



\---



\# Verify Environment Variables



```powershell

kubectl exec -it secret-env-pod -- sh

```



```sh

env

```



Expected output:



```text

USERNAME=admin

PASSWORD=secret123

```



\---



\# Pod Using Secret as Mounted Files



\## pod-secret-volume.yaml



```yaml

apiVersion: v1

kind: Pod

metadata:

&#x20; name: secret-volume-pod



spec:

&#x20; containers:

&#x20;   - name: demo

&#x20;     image: nginx



&#x20;     volumeMounts:

&#x20;       - name: secret-volume

&#x20;         mountPath: /etc/secret-data

&#x20;         readOnly: true



&#x20; volumes:

&#x20;   - name: secret-volume

&#x20;     secret:

&#x20;       secretName: demo-secret

```



\---



\# Deploy Pod



```powershell

kubectl apply -f pod-secret-volume.yaml

```



\---



\# Verify Mounted Secret Files



```powershell

kubectl exec -it secret-volume-pod -- sh

```



```sh

ls /etc/secret-data



cat /etc/secret-data/username



cat /etc/secret-data/password

```



Expected files:



```text

/etc/secret-data/username

/etc/secret-data/password

```



\---



\# Create Secret Using CLI



```powershell

kubectl create secret generic db-secret `

&#x20; --from-literal=username=dbadmin `

&#x20; --from-literal=password=supersecret

```



\---



\# Secret Using stringData



\## simple-secret.yaml



```yaml

apiVersion: v1

kind: Secret

metadata:

&#x20; name: simple-secret



type: Opaque



stringData:

&#x20; username: admin

&#x20; password: secret123

```



\---



\# Important Notes



\- Base64 is encoding, NOT encryption

\- Kubernetes automatically decodes Secret values for containers

\- Secrets are safer than hardcoded credentials

\- Mounted files are often safer than environment variables

\- Secret files are mounted as read-only

\- Secrets should never be committed to Git repositories

\- Enterprises often use Vault or cloud secret managers



\---



\# ConfigMap vs Secret



| ConfigMap | Secret |

|---|---|

| Normal configuration | Sensitive configuration |

| Plain text | Base64 encoded |

| Non-sensitive data | Sensitive data |

| General application settings | Passwords, tokens, certificates |



\---



\# Key Takeaways



\- Secret keys become files when mounted as volumes

\- `secretKeyRef` injects Secret values into environment variables

\- `data` requires base64 values

\- `stringData` accepts plaintext values

\- Kubernetes Secrets are not fully secure by default

\- Secret management is a major DevOps and security topic



\---



\# Cleanup



```powershell

kubectl delete pod secret-env-pod

kubectl delete pod secret-volume-pod



kubectl delete secret demo-secret

kubectl delete secret db-secret

```



