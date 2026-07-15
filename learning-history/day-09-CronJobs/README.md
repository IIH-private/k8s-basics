\# Day 9 — Kubernetes CronJobs



\## Topics covered

\- CronJobs

\- Cron schedules

\- Job templates

\- Concurrency policies

\- History limits

\- Suspend/resume

\- Manual job triggering

\- Missed schedules



\---



\## Key concepts



\### CronJob flow



CronJob → Job → Pod



A CronJob does not create Pods directly.

It creates Jobs based on a schedule, and each Job creates its own Pod.



\---



\### Cron schedule format



```text

\* \* \* \* \*

│ │ │ │ │

│ │ │ │ └── weekday

│ │ │ └──── month

│ │ └────── day

│ └──────── hour

└────────── minute

```



Examples:



```yaml

schedule: "\* \* \* \* \*"

```



Every minute.



```yaml

schedule: "\*/10 \* \* \* \*"

```



Every 10 minutes.



```yaml

schedule: "0 \* \* \* \*"

```



Every hour.



```yaml

schedule: "30 2 \* \* \*"

```



Every day at 02:30.



\---



\## Concurrency policies



\### Allow



```yaml

concurrencyPolicy: Allow

```



Multiple Jobs may run simultaneously.



Use cases:

\- email processing

\- queue workers

\- independent batch tasks



\---



\### Forbid



```yaml

concurrencyPolicy: Forbid

```



If a previous Job is still running, the next scheduled run is skipped.



Use cases:

\- database backups

\- cleanup jobs

\- maintenance tasks



\---



\### Replace



```yaml

concurrencyPolicy: Replace

```



The currently running Job is stopped and replaced by a new one.



Use cases:

\- metrics aggregation

\- cache refresh

\- report generation



\---



\## History limits



```yaml

successfulJobsHistoryLimit: 2

failedJobsHistoryLimit: 1

```



Used to automatically clean up old Jobs and Pods.



Benefits:

\- reduces cluster clutter

\- reduces etcd load

\- keeps troubleshooting history



\---



\## Suspend / Resume



```yaml

suspend: true

```



Pauses future scheduled executions.



Important:

\- existing running Jobs continue

\- only future schedules are paused



\---



\## Manual triggering



Create a Job manually from a CronJob:



```bash

kubectl create job manual-test --from=cronjob/hello-cronjob

```



Useful for:

\- testing

\- reruns after failure

\- ad hoc execution



\---



\## Starting deadline



```yaml

startingDeadlineSeconds: 30

```



Prevents Kubernetes from starting too many missed Jobs after downtime.



Useful after:

\- control plane outages

\- API server downtime

\- cluster recovery



\---



\## Files



\- cronjob.yaml



\---



\## Commands used



```bash

kubectl apply -f cronjob.yaml



kubectl get cronjobs



kubectl get jobs



kubectl get pods



kubectl logs <pod-name>



kubectl delete job <job-name>



kubectl create job manual-test --from=cronjob/hello-cronjob

```



\---



\## What I learned



\- How CronJobs schedule Jobs automatically

\- Difference between Job and CronJob

\- How concurrency policies affect execution

\- How Kubernetes handles overlapping Jobs

\- How to pause and resume CronJobs

\- Why history cleanup matters

\- How to manually trigger CronJobs

\- Why missed schedules can become dangerous

