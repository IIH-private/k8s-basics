\# Day 15 — Kubernetes Volumes



\## Formål



Forstå hvordan data håndteres i Kubernetes, og forskellen mellem midlertidig og persistent storage.



\---



\## 1. Container filesystem vs Volume



\* En container har sit eget filesystem

\* Data går tabt ved container restart

\* Kubernetes bruger \*\*Volumes\*\* til at håndtere data korrekt



```text

Volume tilhører Pod’en — ikke containeren

```



\---



\## 2. emptyDir (midlertidig storage)



\### Egenskaber



\* Oprettes når Pod starter

\* Slettes når Pod slettes

\* Overlever container restart (inden i samme Pod)



\### Brug



\* Cache

\* Midlertidige filer

\* Deling mellem containers



\---



\## 3. Multi-container + shared volume



Flere containers i samme Pod kan dele data via et volume.



\### Eksempel



\* writer container skriver til `/shared/data.txt`

\* reader container læser samme fil



\### Vigtigt



```text

Containers deler IKKE filesystem

De deler KUN data via volumes

```



\---



\## 4. ConfigMap som volume



\### Forskellen



| Type     | Opdatering           |

| -------- | -------------------- |

| env vars | kræver Pod restart   |

| volume   | opdateres automatisk |



\### Observation



\* `production` → `staging`

\* uden restart af Pod



```text

ConfigMap mounted som volume opdateres dynamisk

```



\---



\## 5. PersistentVolume (PV) \& PersistentVolumeClaim (PVC)



\### Problem



```text

emptyDir mister data når Pod slettes

```



\### Løsning



```text

Pod → PVC → PV → storage

```



\---



\### PVC



\* Requester storage

\* Fx: 1Gi



\### PV



\* Selve storage

\* Lever uafhængigt af Pods



\---



\## 6. Persistence test



\### Scenario



1\. Pod skriver til `/data/file.txt`

2\. Pod slettes

3\. Ny Pod oprettes med samme PVC



\### Resultat



```text

file.txt eksisterer stadig

data er bevaret

```



\---



\## 7. Sammenligning



| Type                       | Overlever Pod delete |

| -------------------------- | -------------------- |

| container filesystem       | ❌                    |

| emptyDir                   | ❌                    |

| ConfigMap volume           | ❌                    |

| PersistentVolume (via PVC) | ✅                    |



\---



\## 8. CKAD key points



\* emptyDir er Pod-bound

\* PV/PVC er persistent

\* Multi-container sharing kræver volume

\* ConfigMap som volume opdateres uden restart

\* PVC bruges af Pods — ikke PV direkte



\---



\## Konklusion



```text

Volumes bruges til data i Pods

emptyDir = midlertidig data

PVC/PV = persistent data

```



