# Phase 4.5 – Resource Management (Requests & Limits)

## Overview

Kubernetes allows CPU and memory resources to be configured for containers.

The two main resource settings are:

    requests
    limits

They serve different purposes:

    requests -> used primarily for scheduling
    limits   -> control how much resource a container is allowed to use

Example:

    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "250m"
        memory: "128Mi"

Resource settings belong to individual containers:

    spec:
      containers:
        - name: nginx
          image: nginx:1.29
          resources:
            requests:
              cpu: "100m"
              memory: "64Mi"
            limits:
              cpu: "250m"
              memory: "128Mi"

---

## 1. Resource Requests

A resource request tells Kubernetes how much CPU or memory a container requests.

The Scheduler uses requests when deciding which Node can run a Pod.

Example:

    resources:
      requests:
        cpu: "200m"
        memory: "128Mi"

This means:

    CPU request:     200m
    Memory request:  128Mi

The Scheduler must find a Node with enough available resources to satisfy the Pod's requests.

Important:

    requests are used for scheduling
    actual current resource usage is not the primary scheduling value

A request is not a guarantee that the application will constantly consume that amount.

---

## 2. Resource Limits

A resource limit specifies the maximum amount of a resource that a container is allowed to use.

Example:

    resources:
      limits:
        cpu: "500m"
        memory: "256Mi"

CPU and memory limits behave differently when exceeded.

### CPU

If a container tries to consume more CPU than its CPU limit:

    CPU usage exceeds limit
            |
            v
       CPU throttling

The container is normally not killed simply because it attempts to use more CPU.

### Memory

If a container exceeds its memory limit:

    memory usage exceeds limit
            |
            v
        OOMKilled
            |
            v
    container terminates

The container may then be restarted depending on the Pod's `restartPolicy`.

---

## 3. Requests vs Limits

Example:

    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "250m"
        memory: "128Mi"

Think of the values as:

    Request:
      Scheduler -> "I need this much resource to place the Pod."

    Limit:
      Runtime -> "This container must not exceed this configured limit."

A useful summary:

    CPU request     -> scheduling
    CPU limit       -> throttling

    Memory request  -> scheduling
    Memory limit    -> possible OOMKilled

---

## 4. CPU Units

CPU resources can be expressed as whole CPUs, decimal CPUs, or millicpu.

Examples:

    cpu: "1"
    cpu: "0.5"
    cpu: "500m"

The following values are equivalent:

    1 CPU     = 1000m
    0.75 CPU  = 750m
    0.5 CPU   = 500m
    0.25 CPU  = 250m
    0.1 CPU   = 100m

For values below one CPU, millicpu is often easier to read:

    cpu: "300m"

instead of:

    cpu: "0.3"

### Important CPU unit mistake

These values are very different:

    cpu: "150m"

and:

    cpu: "150"

They mean:

    150m = 0.15 CPU
    150  = 150 CPUs

Forgetting the `m` can therefore create an extremely large CPU request.

Example:

    requests:
      cpu: "150"

on a Node with only:

    2 CPUs

can cause the Pod to remain:

    Pending

with an event such as:

    FailedScheduling
    Insufficient cpu

---

## 5. Memory Units

Memory is commonly specified using:

    Ki
    Mi
    Gi

Examples:

    memory: "64Mi"
    memory: "128Mi"
    memory: "512Mi"
    memory: "1Gi"

Binary units use powers of 1024:

    1 Mi = 1024 Ki
    1 Gi = 1024 Mi

`Mi` and `Gi` are different from decimal units such as `M` and `G`.

For Kubernetes manifests, `Mi` and `Gi` are commonly used.

---

## 6. Scheduling and Requests

The Scheduler considers resource requests when placing Pods on Nodes.

Assume a Node has:

    Allocatable CPU: 2 CPUs = 2000m

and existing Pods already request:

    1700m

A new Pod requests:

    500m

The Scheduler cannot place the Pod:

    1700m + 500m = 2200m

but only:

    2000m

is allocatable.

The Pod can remain:

    Pending

A typical event is:

    FailedScheduling
    0/1 nodes are available: 1 Insufficient cpu

Use:

    kubectl describe pod <pod-name>

and inspect:

    Conditions
    Events

---

## 7. Inspect Node Capacity and Allocated Resources

Inspect a Node:

    kubectl describe node <node-name>

Important sections include:

    Capacity
    Allocatable
    Allocated resources

Example:

    Capacity:
      cpu:     2
      memory:  ...

    Allocatable:
      cpu:     2
      memory:  ...

`Capacity` describes the Node's total resources.

`Allocatable` describes the resources available for Pods after Kubernetes reserves what it needs.

The `Allocated resources` section shows resource requests and limits assigned to Pods on the Node.

Example:

    Allocated resources:
      Resource   Requests      Limits
      cpu        1550m (77%)   1300m (65%)
      memory     700Mi (24%)   1030Mi (35%)

Important:

    Scheduler decisions are based on requests,
    not on whether the application is currently using all requested resources.

---

## 8. Unschedulable Pod Due to CPU Request

A Pod can request more CPU than any available Node can provide.

Example:

    resources:
      requests:
        cpu: "1500m"
        memory: "64Mi"

If the Node cannot satisfy the request, the Pod remains:

    Pending

Inspect it:

    kubectl describe pod <pod-name>

Typical output:

    Conditions:
      PodScheduled: False

    Events:
      FailedScheduling
      Insufficient cpu

Important troubleshooting rule:

    Pending Pod
        |
        v
    kubectl describe pod
        |
        v
    inspect Events
        |
        v
    identify scheduling failure

---

## 9. CPU Limit Behavior

Example:

    resources:
      requests:
        cpu: "100m"
      limits:
        cpu: "250m"

The container can use CPU up to its configured CPU limit.

If it attempts to consume more CPU:

    CPU demand > CPU limit
            |
            v
         throttling

The process normally continues running but receives less CPU time.

Important:

    CPU limit exceeded
        !=
    container killed

CPU is a compressible resource.

---

## 10. Memory Limit Behavior and OOMKilled

Example:

    resources:
      requests:
        memory: "32Mi"
      limits:
        memory: "64Mi"

If the container attempts to use more than the configured memory limit, it can be terminated with:

    Reason: OOMKilled
    Exit Code: 137

Inspect with:

    kubectl describe pod <pod-name>

Typical output:

    State:
      Terminated
      Reason: OOMKilled
      Exit Code: 137

If the Pod's restart policy causes the container to restart, repeated memory allocation can produce repeated restarts.

Example:

    STATUS       OOMKilled
    RESTARTS     4

Important:

    OOMKilled affects the container.

It does not automatically mean that the Pod object itself is deleted.

---

## 11. OOMKilled vs Pod Eviction

These are different situations.

### OOMKilled

Usually occurs when a container exceeds its memory limit:

    container exceeds memory limit
              |
              v
          OOMKilled
              |
              v
    container terminates
              |
              v
    may be restarted

This operates at the container level.

### Pod Eviction

Eviction can happen when a Node is under resource pressure.

Example:

    Node under memory pressure
              |
              v
    Kubernetes selects a Pod
              |
              v
           eviction
              |
              v
    Pod is removed from the Node

This operates at the Pod level.

If the Pod belongs to a controller such as a Deployment, the controller can create a replacement Pod to restore the desired state.

Remember:

    OOMKilled -> container level
    Eviction  -> Pod level

---

## 12. Requests and Limits When Only One Is Specified

If a limit is specified and no corresponding request is explicitly configured, Kubernetes can use the limit as the request for that resource.

Example:

    resources:
      limits:
        cpu: "500m"
        memory: "128Mi"

The resulting effective values can be:

    CPU:
      request = 500m
      limit   = 500m

    Memory:
      request = 128Mi
      limit   = 128Mi

This behavior is important when determining scheduling requirements and QoS class.

If a request exists without a corresponding limit, the request still participates in scheduling, but there is no explicit limit for that resource unless another cluster mechanism supplies one.

---

## 13. Kubernetes QoS Classes

Kubernetes assigns each Pod a Quality of Service (QoS) class.

The three classes are:

    Guaranteed
    Burstable
    BestEffort

Check the QoS class:

    kubectl describe pod <pod-name>

Look for:

    QoS Class:

It can also be read directly from the Pod status:

    kubectl get pod <pod-name> \
      -o jsonpath='{.status.qosClass}'; echo

For multiple Pods:

    kubectl get pods \
      -o custom-columns='NAME:.metadata.name,QOS:.status.qosClass'

---

## 14. Guaranteed QoS

A Pod receives the `Guaranteed` QoS class when every container meets the required CPU and memory conditions.

For each container:

    CPU request    = CPU limit
    Memory request = Memory limit

Example:

    resources:
      requests:
        cpu: "200m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"

Result:

    QoS Class: Guaranteed

In a multi-container Pod, every container must meet the Guaranteed requirements.

The containers do not need to use the same resource values as each other.

Example:

    web:
      CPU:     200m request / 200m limit
      Memory:  128Mi request / 128Mi limit

    sidecar:
      CPU:     100m request / 100m limit
      Memory:   64Mi request / 64Mi limit

The Pod can still be:

    Guaranteed

---

## 15. Burstable QoS

A Pod is `Burstable` when it has CPU or memory requests/limits but does not meet all requirements for `Guaranteed`.

Example:

    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "250m"
        memory: "128Mi"

Because:

    100m != 250m
    64Mi != 128Mi

the Pod is:

    Burstable

A useful rule:

    Burstable -> everything between Guaranteed and BestEffort

For a multi-container Pod, one container that does not satisfy the Guaranteed conditions can make the whole Pod Burstable.

Example:

    web:
      CPU request = CPU limit
      Memory request = Memory limit

    sidecar:
      CPU request != CPU limit

Result:

    whole Pod -> Burstable

---

## 16. BestEffort QoS

A Pod receives the `BestEffort` QoS class when none of its containers have CPU or memory requests or limits.

Example:

    containers:
      - name: nginx
        image: nginx:1.29

Result:

    QoS Class: BestEffort

For a multi-container Pod:

    every container must have no CPU/memory requests or limits

for the whole Pod to be BestEffort.

Example:

    web:
      requests + limits configured

    sidecar:
      no requests
      no limits

The Pod is NOT BestEffort because at least one container has resource configuration.

It is typically:

    Burstable

---

## 17. QoS and Resource Pressure

QoS is related to how Kubernetes handles Pods under Node resource pressure.

A useful conceptual model is:

    BestEffort
        |
        v
    generally more vulnerable

    Burstable
        |
        v
    depends on factors such as usage vs requests

    Guaranteed
        |
        v
    generally better protected

However, do not use the oversimplified rule:

    BestEffort always evicted first
    Burstable always second
    Guaranteed always last

Eviction decisions can also consider factors such as:

    Pod Priority
    resource usage
    whether usage exceeds requests

QoS class is therefore not the only factor involved in eviction decisions.

---

## 18. Multi-Container Pod Requests

Resource requests are configured per container.

Example:

    web:
      CPU request:     200m
      Memory request:  128Mi

    sidecar:
      CPU request:     100m
      Memory request:   64Mi

For scheduling, the requests of regular application containers are combined:

    CPU:
      200m + 100m = 300m

    Memory:
      128Mi + 64Mi = 192Mi

The Scheduler must find a Node that can satisfy the Pod's combined scheduling requirement.

Remember:

    Scheduler places the entire Pod on one Node,
    not each container on separate Nodes.

---

## 19. Multi-Container Limits

Limits are configured for individual containers.

Example:

    web:
      CPU limit:     500m
      Memory limit:  256Mi

    sidecar:
      CPU limit:     200m
      Memory limit:  128Mi

The limits can be summarized as:

    CPU limits total:     700m
    Memory limits total:  384Mi

but enforcement still applies to the individual containers.

For example, the `web` container cannot use `700m` CPU simply because the `sidecar` is not using its CPU allocation.

Its own limit remains:

    web CPU limit = 500m

---

## 20. Inspect Resources in a Multi-Container Pod

Use:

    kubectl describe pod <pod-name>

Resources are displayed separately for each container:

    Containers:
      web:
        Limits:
          cpu:     500m
          memory:  256Mi
        Requests:
          cpu:     200m
          memory:  128Mi

      sidecar:
        Limits:
          cpu:     200m
          memory:  128Mi
        Requests:
          cpu:     100m
          memory:  64Mi

Resources can also be inspected from the Pod specification:

    kubectl get pod <pod-name> -o yaml

Example JSONPath for a specific container:

    kubectl get pod <pod-name> \
      -o jsonpath='{.spec.containers[?(@.name=="web")].resources}'; echo

---

## 21. Resources in Deployments

Resource requests and limits are part of the container definition inside the Deployment's Pod template.

Path:

    spec.template.spec.containers[].resources

Example:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: resource-web
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: resource-web
      template:
        metadata:
          labels:
            app: resource-web
        spec:
          containers:
            - name: nginx
              image: nginx:1.29
              resources:
                requests:
                  cpu: "100m"
                  memory: "64Mi"
                limits:
                  cpu: "250m"
                  memory: "128Mi"

Resources do NOT belong directly under:

    spec.resources

or:

    spec.template.resources

They belong to the container.

---

## 22. Total Requests Across Deployment Replicas

Assume a Deployment has:

    replicas: 3

and each Pod requests:

    CPU:     200m
    Memory:  128Mi

The total application requests are:

    CPU:
      3 x 200m = 600m

    Memory:
      3 x 128Mi = 384Mi

The Scheduler still schedules each Pod individually, but every replica contributes its requests to Node resource allocation.

---

## 23. Updating Deployment Resources

Changing resources in a Deployment's Pod template changes:

    spec.template

This causes the Deployment to perform a rollout.

Conceptually:

    update Deployment Pod template
              |
              v
       new ReplicaSet
              |
              v
         new Pods
              |
              v
       old Pods replaced

Check rollout status:

    kubectl rollout status deployment/<deployment-name>

A standalone Pod is different because many Pod specification fields cannot simply be changed in place.

For a standalone Pod, a common workflow is:

    edit manifest
        |
        v
    delete/recreate Pod

For a Deployment:

    update spec.template
        |
        v
    Deployment performs rollout

---

## 24. Update Resources with `kubectl set resources`

Resources can be changed imperatively.

Example:

    kubectl set resources deployment resource-web \
      --requests=cpu=150m,memory=96Mi \
      --limits=cpu=300m,memory=192Mi

Verify:

    kubectl get deployment resource-web \
      -o jsonpath='{.spec.template.spec.containers[0].resources}'; echo

Check rollout:

    kubectl rollout status deployment/resource-web

### Target a specific container

For a workload with multiple containers:

    kubectl set resources deployment resource-web \
      -c web \
      --requests=cpu=150m,memory=96Mi \
      --limits=cpu=300m,memory=192Mi

`-c web` selects the `web` container.

### Important syntax

Options require `--`:

    --requests=...
    --limits=...

Without `--`, `kubectl` may interpret the text as another resource name.

---

## 25. Generate and Modify Deployment YAML

A useful CKAD workflow is to generate a Deployment manifest first:

    kubectl create deployment web-production \
      --image=nginx:1.29 \
      --replicas=3 \
      --dry-run=client \
      -o yaml > web-production.yaml

Then edit the generated manifest and add:

    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "250m"
        memory: "128Mi"

Apply:

    kubectl apply -f web-production.yaml

This is often faster and less error-prone than writing the complete Deployment manifest from memory.

---

## 26. Dry-Run Resource Changes

A resource change can be inspected without applying it:

    kubectl set resources deployment resource-web \
      --requests=cpu=200m,memory=128Mi \
      --limits=cpu=400m,memory=256Mi \
      --dry-run=client \
      -o yaml

This is useful for:

    generating YAML
    verifying syntax
    inspecting the resulting Pod template
    avoiding accidental changes

A useful CKAD workflow is:

    imperative command
          |
          v
    --dry-run=client -o yaml
          |
          v
    inspect / edit
          |
          v
    kubectl apply

---

## 27. Resource Changes Can Block a Deployment Rollout

A Deployment normally uses a rolling update strategy.

When resource requests are increased, Kubernetes may need enough free Node resources to run a new Pod while old Pods are still running.

If the new Pod cannot be scheduled:

    Deployment update
          |
          v
    new ReplicaSet
          |
          v
    new Pod created
          |
          v
    FailedScheduling
          |
          v
    Pod remains Pending
          |
          v
    rollout waits

`kubectl rollout status` may therefore continue waiting.

Investigate the Pending Pod:

    kubectl get pods

then:

    kubectl describe pod <pending-pod-name>

Look under:

    Conditions
    Events

A typical cause is:

    FailedScheduling
    Insufficient cpu

Stopping:

    kubectl rollout status ...

with `Ctrl+C` only stops the local waiting command. It does not cancel or delete the Deployment.

---

## 28. Troubleshooting Resource Problems

Start with:

    kubectl get pods

If a Pod is Pending:

    kubectl describe pod <pod-name>

Look especially at:

    Status
    Conditions
    Events

For scheduling problems, typical events include:

    FailedScheduling
    Insufficient cpu
    Insufficient memory

Inspect Node resources:

    kubectl describe node <node-name>

Look at:

    Capacity
    Allocatable
    Non-terminated Pods
    Allocated resources

List Pods across all namespaces:

    kubectl get pods -A

Show CPU requests across all namespaces:

    kubectl get pods -A \
      -o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,NODE:.spec.nodeName,CPU:.spec.containers[*].resources.requests.cpu'

This is important because system Pods in namespaces such as:

    kube-system
    ingress-nginx

also consume Node resources.

Do not assume that:

    kubectl get pods

shows every Pod on the Node. Without `-A`, it only shows Pods in the current namespace.

---

## 29. Troubleshooting CPU Unit Errors

A common mistake is:

    --requests=cpu=150,memory=96Mi

when the intended value is:

    --requests=cpu=150m,memory=96Mi

The first value means:

    150 CPUs

not:

    150 millicpu

Inspect the actual configured value:

    kubectl get deployment <deployment-name> \
      -o jsonpath='{.spec.template.spec.containers[0].resources}'; echo

Example incorrect output:

    "requests":{"cpu":"150","memory":"96Mi"}

Correct:

    "requests":{"cpu":"150m","memory":"96Mi"}

If a Pod remains Pending with:

    Insufficient cpu

always verify both:

    Node allocatable resources

and:

    actual Pod resource requests

Do not rely only on what the command was intended to configure.

---

## 30. Useful Verification Commands

Inspect Pod resources:

    kubectl describe pod <pod-name>

Inspect Pod YAML:

    kubectl get pod <pod-name> -o yaml

Inspect one container's resources:

    kubectl get pod <pod-name> \
      -o jsonpath='{.spec.containers[0].resources}'; echo

Inspect Deployment template resources:

    kubectl get deployment <deployment-name> \
      -o jsonpath='{.spec.template.spec.containers[0].resources}'; echo

Check QoS:

    kubectl get pod <pod-name> \
      -o jsonpath='{.status.qosClass}'; echo

Inspect Node resources:

    kubectl describe node <node-name>

Check rollout:

    kubectl rollout status deployment/<deployment-name>

List all Pods across namespaces:

    kubectl get pods -A

---

## CKAD Quick Reference

    # Generate a Deployment manifest
    kubectl create deployment web-production \
      --image=nginx:1.29 \
      --replicas=3 \
      --dry-run=client \
      -o yaml > web-production.yaml

    # Example container resources
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "250m"
        memory: "128Mi"

    # Inspect Pod resources
    kubectl describe pod <pod-name>

    # Inspect Deployment resources
    kubectl get deployment <deployment-name> \
      -o jsonpath='{.spec.template.spec.containers[0].resources}'; echo

    # Update Deployment resources
    kubectl set resources deployment <deployment-name> \
      --requests=cpu=150m,memory=96Mi \
      --limits=cpu=300m,memory=192Mi

    # Update one container
    kubectl set resources deployment <deployment-name> \
      -c <container-name> \
      --requests=cpu=150m,memory=96Mi \
      --limits=cpu=300m,memory=192Mi

    # Check rollout
    kubectl rollout status deployment/<deployment-name>

    # Check QoS
    kubectl get pod <pod-name> \
      -o jsonpath='{.status.qosClass}'; echo

    # Inspect Node capacity and allocations
    kubectl describe node <node-name>

    # Troubleshoot a Pending Pod
    kubectl describe pod <pending-pod-name>

    # Show Pods from every namespace
    kubectl get pods -A

---

## Key Exam Rules

1. Resource requests are used by the Scheduler when placing Pods on Nodes.
2. Resource limits control the maximum configured resource usage for a container.
3. CPU requests and limits can be expressed in CPUs or millicpu.
4. `1000m = 1 CPU`.
5. `150` CPU and `150m` CPU are completely different values.
6. CPU usage above a CPU limit normally results in throttling, not container termination.
7. Exceeding a memory limit can result in `OOMKilled`.
8. `OOMKilled` operates at the container level; eviction operates at the Pod level.
9. A Pending Pod with `Insufficient cpu` should be investigated with `kubectl describe pod`.
10. Node `Allocated resources` are based on requests assigned to Pods, not current application usage.
11. System Pods also consume Node resources; use `kubectl get pods -A` when investigating the whole Node.
12. The three QoS classes are `Guaranteed`, `Burstable`, and `BestEffort`.
13. For `Guaranteed`, every container must meet the CPU and memory request/limit requirements.
14. `BestEffort` requires that none of the containers have CPU or memory requests or limits.
15. Everything between `Guaranteed` and `BestEffort` is generally `Burstable`.
16. QoS class is not the only factor considered during eviction.
17. In a normal multi-container Pod, container requests are combined for scheduling.
18. All containers in a Pod are scheduled together onto the same Node.
19. Resource limits are configured and enforced per container.
20. Deployment resources belong under `spec.template.spec.containers[].resources`.
21. Changing a Deployment's Pod template triggers a rollout.
22. `kubectl set resources` can update requests and limits imperatively.
23. Use `-c <container-name>` when targeting a specific container with `kubectl set resources`.
24. Always include the correct resource units, especially `m` for millicpu.
25. When troubleshooting resource problems, verify the actual stored configuration rather than only the command that was intended to create it.