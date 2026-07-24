# Pods

## Goal

Learn how to create, configure, troubleshoot, and verify Kubernetes Pods using both imperative commands and YAML manifests.

Pods are the smallest deployable workload objects in Kubernetes and form the foundation for ReplicaSets, Deployments, Jobs, and other controllers.

---

## Folder Structure

    01-pods/
    ├── 00-introduction/
    │   ├── invalid-pod.yaml
    │   └── nginx-pod.yaml
    ├── 01-labels-lab/
    ├── 02-pod-yaml-lab/
    │   ├── init-container.yaml
    │   ├── liveness-http.yaml
    │   ├── logger-pod.yaml
    │   ├── node-info.yaml
    │   ├── readiness-http.yaml
    │   ├── restart-always.yaml
    │   ├── restart-never.yaml
    │   ├── restart-never2.yaml
    │   └── shared-storage.yaml
    ├── 03-ckad-tasks/
    │   ├── task1.yaml
    │   ├── task2.yaml
    │   ├── task3.yaml
    │   └── task5.yaml
    ├── 04-troubleshooting/
    │   └── task4-broken.yaml
    └── README.md

---

## Pod Fundamentals

A Pod is the smallest deployable workload object in Kubernetes.

A Pod can contain one or more containers. Containers in the same Pod share:

- the Pod IP address
- the network namespace
- `localhost`
- Pod-level volumes
- scheduling and lifecycle context

A Pod itself is not restarted.

When a container stops, Kubernetes may restart the container inside the same Pod according to the Pod's `restartPolicy`.

When a Pod is deleted and replaced, the replacement is a new Pod with a new UID and normally a new IP address.

---

## Creating Pods

Create a Pod imperatively:

    kubectl run nginx --image=nginx

Generate a YAML template:

    kubectl run nginx --image=nginx \
      --dry-run=client \
      -o yaml > nginx-pod.yaml

Create or update a Pod from YAML:

    kubectl apply -f nginx-pod.yaml

Delete a Pod:

    kubectl delete pod nginx

Delete using the manifest:

    kubectl delete -f nginx-pod.yaml

---

## Pod YAML Structure

A basic Pod manifest contains:

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx

Important YAML structures:

- `metadata` is a map
- `spec` is a map
- `containers` is a list
- each item in `containers` is a map
- `env` is a list
- `volumeMounts` is a list
- `volumes` is a list
- probes are maps

Use `kubectl explain` when the structure or field names are uncertain:

    kubectl explain pod
    kubectl explain pod.spec
    kubectl explain pod.spec.containers
    kubectl explain pod.spec.containers.resources
    kubectl explain pod.spec.containers.livenessProbe
    kubectl explain pod.spec.initContainers

---

## Commands and Arguments

Container commands and arguments can override the image defaults.

Example:

    spec:
      containers:
      - name: worker
        image: busybox
        command:
        - sleep
        args:
        - "3600"

The equivalent Linux command is:

    sleep 3600

For shell functionality such as redirection, variables, pipes, or multiple commands:

    command:
    - /bin/sh
    - -c
    - echo "Hello CKAD" > /shared/message.txt; sleep 3600

A volume mount does not automatically become the working directory.

Use an absolute path or configure:

    workingDir: /shared

---

## Environment Variables

Environment variables can be defined directly in a container:

    env:
    - name: APP_ENV
      value: production

Verify the value:

    kubectl exec <pod> -- printenv APP_ENV

Environment variables were introduced here because they were needed for practical Pod exercises. They will be reviewed again in their official roadmap position under Phase 4 – Configuration.

---

## Resource Requests and Limits

Example:

    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 256Mi

Important distinction:

- `requests` are used by the scheduler when placing the Pod.
- `limits` are enforced while the container runs.

CPU examples:

    100m = 0.1 CPU
    500m = 0.5 CPU
    1    = 1 CPU

Memory examples:

    128Mi
    256Mi
    1Gi

Resource field names are case-sensitive:

    cpu
    memory

Resources were introduced here because they were needed for practical Pod exercises. They will be reviewed again in their official roadmap position under Phase 4 – Configuration.

---

## QoS Classes

Kubernetes assigns a QoS class to a Pod based on its resource configuration.

### Guaranteed

Every container has equal CPU and memory requests and limits.

### Burstable

At least one container has a request or limit, but the Pod does not meet all Guaranteed requirements.

### BestEffort

No container has CPU or memory requests or limits.

Check the QoS class:

    kubectl get pod <pod> \
      -o jsonpath='{.status.qosClass}{"\n"}'

---

## Restart Policy

`restartPolicy` is configured at Pod level:

    spec:
      restartPolicy: Always

Possible values:

- `Always`
- `OnFailure`
- `Never`

The policy belongs to the Pod but is applied to containers in the Pod when they stop.

### Always

Restart the container after both successful and unsuccessful termination.

### OnFailure

Restart the container only when it exits with a non-zero exit code.

### Never

Do not restart the container.

A container restart does not recreate the Pod.

During a container restart:

- the Pod name stays the same
- the Pod UID stays the same
- the Pod IP stays the same
- the container ID changes
- an `emptyDir` volume remains available

---

## Container States

A container can be:

- `Waiting`
- `Running`
- `Terminated`

Example from `kubectl describe pod`:

    State:          Terminated
    Reason:         Completed
    Exit Code:      0
    Restart Count:  0

`Terminated` means that the container process has finished. Kubernetes still retains status information while the Pod object exists.

---

## Container Logs

Logs from the current or latest container instance:

    kubectl logs <pod>

Logs from a specific container:

    kubectl logs <pod> -c <container>

Logs from the previous container instance after a restart:

    kubectl logs --previous <pod> -c <container>

Deleting and recreating a Pod creates a new Pod. The new Pod does not inherit the old Pod's local container-log history.

---

## Volumes

### emptyDir

An `emptyDir` belongs to the Pod.

    volumes:
    - name: shared-data
      emptyDir: {}

A container mounts it using:

    volumeMounts:
    - name: shared-data
      mountPath: /shared

`emptyDir`:

- is created when the Pod is assigned to a node
- survives container restarts inside the same Pod
- can be shared between containers in the same Pod
- is deleted when the Pod is deleted

### hostPath

A `hostPath` mounts a path from the Kubernetes node:

    volumes:
    - name: host-logs
      hostPath:
        path: /var/log

A Pod scheduled to another node accesses that other node's path. Data from the previous node does not automatically follow the Pod.

`hostPath` is node-specific and should be used carefully.

Storage topics will be reviewed and expanded in their official roadmap position under Phase 6 – Storage.

---

## Read-only Volume Mounts

The same Pod-level volume can be mounted differently by separate containers.

Read/write mount:

    volumeMounts:
    - name: shared-data
      mountPath: /data

Read-only mount:

    volumeMounts:
    - name: shared-data
      mountPath: /data
      readOnly: true

`readOnly` belongs to the individual mount, not to the Pod-level volume definition.

---

## Health Probes

Kubernetes supports three probe mechanisms:

- HTTP GET
- TCP socket
- Exec command

### Liveness Probe

A liveness probe checks whether the application should be restarted.

Example:

    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
      timeoutSeconds: 2
      failureThreshold: 3

When the configured number of liveness checks fails, kubelet stops the container. The Pod's `restartPolicy` then determines whether the container is restarted.

### Readiness Probe

A readiness probe checks whether the container is ready to receive traffic.

Example:

    readinessProbe:
      exec:
        command:
        - test
        - -f
        - /shared/ready.txt
      periodSeconds: 5
      failureThreshold: 2

When a readiness probe fails:

- the container is not restarted
- the Pod can remain `Running`
- the Pod becomes `NotReady`
- Services stop routing normal traffic to the Pod

A Pod can therefore show:

    READY    STATUS
    0/1      Running

When the readiness probe succeeds again, the Pod automatically becomes Ready.

### Startup Probe

A startup probe protects applications that need a long time to start.

Example:

    startupProbe:
      httpGet:
        path: /
        port: http
      periodSeconds: 2
      failureThreshold: 15

While the startup probe has not succeeded, the liveness probe is disabled.

When the startup probe succeeds, the normal liveness and readiness checks take over.

### Common Probe Fields

- `initialDelaySeconds`
- `periodSeconds`
- `timeoutSeconds`
- `failureThreshold`
- `successThreshold`

`successThreshold` can be greater than `1` for readiness probes. For liveness and startup probes it must remain `1`.

---

## HTTP Probe

    httpGet:
      path: /
      port: 80

The kubelet sends the request directly to the Pod IP.

Successful HTTP status codes are from `200` through `399`.

---

## TCP Probe

    tcpSocket:
      port: 6379

A TCP probe verifies that a TCP connection can be opened.

It confirms that something is listening on the port, but it does not prove that the application protocol works correctly.

---

## Exec Probe

    exec:
      command:
      - redis-cli
      - ping

The command runs inside the container.

- exit code `0` means success
- a non-zero exit code means failure

The `command` field is a list where the first item is the executable and the remaining items are its arguments.

---

## Multi-container Pods

A Pod can contain multiple containers:

    spec:
      containers:
      - name: writer
        image: busybox
      - name: reader
        image: busybox

Containers in the same Pod share:

- the Pod IP
- the network namespace
- `localhost`
- Pod-level volumes

They do not automatically share their individual container filesystems.

Shared files require a volume such as `emptyDir`.

Use `-c` when working with a specific container:

    kubectl exec -it <pod> -c <container> -- sh
    kubectl logs <pod> -c <container>

---

## Sidecar Pattern

A sidecar is a design pattern, not a separate Kubernetes object.

Example:

    Pod
    ├── application
    └── log-agent

The sidecar supports the main application by performing tasks such as:

- log forwarding
- proxying
- metrics export
- configuration synchronization

The main container and sidecar run together in the same Pod.

---

## Init Containers

Init containers run before the regular application containers:

    spec:
      initContainers:
      - name: init-writer
        image: busybox
        command:
        - /bin/sh
        - -c
        - echo "ready" > /shared/ready.txt

      containers:
      - name: app
        image: busybox
        command:
        - sleep
        - "3600"

Important rules:

- Init containers run sequentially.
- Each init container must complete successfully.
- The next init container does not start until the previous one succeeds.
- Regular containers do not start until all init containers succeed.
- Init containers are suitable for preparation work required before the application starts.

Typical uses:

- writing initial configuration
- waiting for an external dependency
- preparing a volume
- downloading files
- setting permissions

An init container differs from a startup probe:

- an init container performs work before the application starts
- a startup probe checks the application after its process has started

---

## Useful kubectl Commands

View Pods:

    kubectl get pods

Watch a Pod:

    kubectl get pod <name> -w

Show additional details:

    kubectl get pod <name> -o wide

Describe a Pod:

    kubectl describe pod <name>

View logs:

    kubectl logs <pod>

View logs from a specific container:

    kubectl logs <pod> -c <container>

View previous container logs:

    kubectl logs <pod> -c <container> --previous

Run a command:

    kubectl exec <pod> -- <command>

Open a shell:

    kubectl exec -it <pod> -- sh

Open a shell in a specific container:

    kubectl exec -it <pod> -c <container> -- sh

Inspect the full manifest:

    kubectl get pod <name> -o yaml

Delete and recreate a Pod from a manifest:

    kubectl delete -f pod.yaml
    kubectl apply -f pod.yaml

---

## Troubleshooting Workflow

A practical troubleshooting sequence:

1. Check Pod status.

       kubectl get pod <name>

2. Inspect events and container states.

       kubectl describe pod <name>

3. Read the relevant container logs.

       kubectl logs <name> -c <container>

4. Check previous logs after a restart.

       kubectl logs <name> -c <container> --previous

5. Inspect the running container.

       kubectl exec -it <name> -c <container> -- sh

6. Compare the running object with the local YAML.

       kubectl get pod <name> -o yaml
       cat pod.yaml

Typical errors encountered:

- invalid YAML
- misspelled field names
- wrong capitalization
- incorrect volume names
- invalid `volumeMounts`
- incorrect `mountPath`
- probe failures
- incorrect commands
- incorrect executable paths

---

## CKAD Mini Mock

Five CKAD-style Pod tasks were completed.

The tasks covered:

- Pod creation
- resource requests and limits
- environment variables
- liveness, readiness, and startup probes
- HTTP and exec probes
- shared `emptyDir` volumes
- multi-container Pods
- sidecar containers
- init containers
- read-only mounts
- named ports
- troubleshooting invalid volume references
- verification using `kubectl get`, `describe`, `exec`, and `logs`

The final task combined:

- an init container
- an nginx main container
- a BusyBox sidecar
- shared web content
- three health probes
- resources
- environment variables
- a named HTTP port
- read-only volume access

---

## Important Lessons

- Kubernetes restarts containers, not Pods.
- Deleting and recreating a Pod produces a new Pod.
- A Pod-level `restartPolicy` is applied to containers that stop.
- Container IDs change after container restarts.
- An `emptyDir` survives container restarts but not Pod deletion.
- A volume mount does not change the container's working directory.
- Containers in the same Pod can communicate through `localhost`.
- Readiness controls traffic.
- Liveness controls container restart.
- Startup probes protect slow startup.
- Init containers must complete before application containers start.
- Sidecars run alongside the main application.
- Exact field names, capitalization, volume names, and mount paths matter.

---

## Status

The Pods topic is completed and passed through practical labs, troubleshooting exercises, and a five-task CKAD Mini Mock.

Next roadmap topic:

    ReplicaSets