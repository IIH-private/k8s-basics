\# Day 8 — Kubernetes Jobs



\## Goal



Learn how Kubernetes Jobs work and understand the difference between:

\- long-running workloads

\- one-time batch workloads



This lab covered:

\- successful Jobs

\- failed Jobs

\- retries

\- `backoffLimit`

\- parallel Jobs

\- `completions`

\- `parallelism`



\---



\# What is a Job?



A Kubernetes `Job` is used for workloads that should:

\- run once

\- complete successfully

\- stop afterward



Typical examples:

\- database backups

\- data imports

\- cleanup scripts

\- report generation

\- migrations



Unlike a `Deployment`, a `Job` does not try to keep Pods running forever.



\---



\# Key Difference



\## Deployment



```text

Keep application running continuously

```



\## Job



```text

Run task until completion

```



\---



\# Lab 1 — Successful Job



\## YAML



```yaml

apiVersion: batch/v1

kind: Job

metadata:

&#x20; name: hello-job

spec:

&#x20; template:

&#x20;   spec:

&#x20;     containers:

&#x20;     - name: hello

&#x20;       image: busybox

&#x20;       command: \["sh", "-c", "echo Hello from Kubernetes Job \&\& sleep 5"]

&#x20;     restartPolicy: Never

```



\## Result



The Pod completed successfully:



```text

STATUS: Completed

```



Important observation:

\- Kubernetes keeps completed Pods for logs and debugging

\- completed Pods act as execution history



\---



\# Lab 2 — Failed Job



\## YAML



```yaml

apiVersion: batch/v1

kind: Job

metadata:

&#x20; name: failed-job

spec:

&#x20; template:

&#x20;   spec:

&#x20;     containers:

&#x20;     - name: fail

&#x20;       image: busybox

&#x20;       command: \["sh", "-c", "exit 1"]

&#x20;     restartPolicy: Never

```



\## What happened?



The container exited with:



```text

exit code 1

```



The Job controller created new Pods automatically.



Important:

\- `restartPolicy: Never` only prevents container restart inside the same Pod

\- the Job controller can still create completely new Pods



\---



\# backoffLimit



Default behavior:



```text

1 initial attempt + 6 retries = 7 Pods total

```



Example:



```yaml

backoffLimit: 2

```



Result:



```text

1 initial attempt + 2 retries = 3 Pods total

```



\---



\# Important Observation



Even though the Job retried several times:



```text

RESTARTS = 0

```



Reason:

\- containers were never restarted inside the same Pod

\- Kubernetes created entirely new Pods instead



\---



\# Lab 3 — Parallel Jobs



\## YAML



```yaml

apiVersion: batch/v1

kind: Job

metadata:

&#x20; name: parallel-job

spec:

&#x20; completions: 3

&#x20; parallelism: 3

&#x20; template:

&#x20;   spec:

&#x20;     containers:

&#x20;     - name: worker

&#x20;       image: busybox

&#x20;       command: \["sh", "-c", "echo Processing \&\& sleep 20"]

&#x20;     restartPolicy: Never

```



\## Meaning



\### completions



```text

How many successful Pods are required

```



\### parallelism



```text

How many Pods may run simultaneously

```



Result:

\- 3 Pods completed successfully

\- all 3 Pods ran at the same time



\---



\# ImagePullBackOff in Jobs



A Job can become stuck if Kubernetes cannot pull the container image.



Example statuses:



```text

ErrImagePull

ImagePullBackOff

```



The Job may never reach completion because the workload never starts.



\---



\# Cleanup



Delete Jobs and their Pods:



```powershell

kubectl delete job hello-job failed-job backoff-job parallel-job

```



\---



\# Key Learnings



\- Jobs are for finite workloads

\- Deployments are for continuously running applications

\- `restartPolicy` affects containers inside a Pod

\- Job retries create new Pods

\- `backoffLimit` controls retry count

\- `completions` defines required successful Pods

\- `parallelism` defines concurrent execution

\- completed and failed Pods are kept for history and debugging

