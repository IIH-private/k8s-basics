# Phase 4.2 – Environment Variables

Environment variables are used to pass configuration values into containers without modifying the container image.

## Define Environment Variables

Environment variables are defined under the individual container:

    spec:
      containers:
      - name: app
        image: busybox:1.36
        env:
        - name: APP_COLOR
          value: blue
        - name: APP_VERSION
          value: "1.0"

The YAML path is:

    spec.containers[].env

Each container in a Pod can have its own environment variables.

## Values Are Strings

Environment variable values are strings.

Values that look like numbers or booleans should normally be quoted:

    env:
    - name: PORT
      value: "8080"
    - name: DEBUG
      value: "true"
    - name: APP_VERSION
      value: "2.5"

## Using Environment Variables in Commands

Kubernetes makes the environment variables available inside the container.

A shell is required when using shell variable expansion:

    command:
    - sh
    - -c
    - |
      echo "Application: $APP_NAME"
      echo "Version: $APP_VERSION"
      sleep 3600

Kubernetes does not expand shell variables directly.

This command prints the variable name literally:

    command:
    - echo
    - $APP_NAME

Output:

    $APP_NAME

This command uses the shell to expand the variable:

    command:
    - sh
    - -c
    - echo "$APP_NAME"

Kubernetes provides the environment variable to the container, while the shell performs the variable expansion.

## YAML Block Scalars

The literal block scalar `|` preserves line breaks and is useful for shell scripts:

    command:
    - sh
    - -c
    - |
      echo "Starting application"
      echo "Environment: $ENVIRONMENT"
      sleep 3600

The folded block scalar `>` replaces most line breaks with spaces and is normally not suitable for multi-line shell scripts unless command separators are included.

## Updating Pod Environment Variables

Most fields in an existing Pod specification are immutable.

After changing an environment variable, the Pod can be deleted and recreated:

    kubectl delete pod company-pod
    kubectl apply -f 02-company-pod.yaml

A faster lab and exam approach is:

    kubectl replace --force -f 02-company-pod.yaml

`kubectl replace --force` deletes the existing Pod and creates a new Pod from the manifest.

## Imperative Environment Variable Management

`kubectl set env` can be used to inspect and modify environment variables on workloads such as Deployments.

### List Environment Variables

List the environment variables defined in a Deployment's Pod template:

    kubectl set env deployment/env-repair --list

Example output:

    # Deployment env-repair, container nginx
    LOG_LEVEL=debug

This inspects the environment variables defined in the workload configuration.

It does not execute `env` inside a running container.

### Set an Environment Variable

Set a new environment variable:

    kubectl set env deployment/env-repair LOG_LEVEL=debug

For a Deployment, this modifies the Pod template.

The sequence is:

    Deployment Pod template changes
            ↓
    Deployment starts a rollout
            ↓
    A new ReplicaSet is created
            ↓
    New Pod(s) are created
            ↓
    Old ReplicaSet is scaled down
            ↓
    Old Pod(s) are removed

Environment variables of an already running process are not modified.

### Update an Existing Environment Variable

The same syntax is used to update an existing variable:

    kubectl set env deployment/env-repair LOG_LEVEL=info

Verify the rollout:

    kubectl rollout status deployment/env-repair

Verify the configured value:

    kubectl set env deployment/env-repair --list

### Remove an Environment Variable

Add `-` after the variable name to remove it:

    kubectl set env deployment/env-repair LOG_LEVEL-

This removes `LOG_LEVEL` from the Pod template and causes the Deployment to roll out new Pod(s).

Verify:

    kubectl set env deployment/env-repair --list

## Inspect Configuration vs Running Container

There is an important difference between inspecting the workload configuration and inspecting a running container.

### Inspect the Deployment Pod Template

    kubectl set env deployment/env-repair --list

This shows environment variables defined in:

    spec.template.spec.containers[].env

It answers:

    What environment variables are configured for new Pods?

### Inspect the Running Container

    kubectl exec <pod-name> -- env

This executes `env` inside a running container.

It answers:

    What environment variables does this running container actually have?

Filter for a specific variable:

    kubectl exec <pod-name> -- env | grep LOG_LEVEL

If `grep` finds no match, it normally produces no output and exits with exit code `1`.

## Multi-Container Pods

Environment variables belong to individual containers.

For example:

    spec:
      containers:
      - name: web
        image: nginx:1.27
        env:
        - name: LOG_LEVEL
          value: info

      - name: logger
        image: busybox:1.36
        env:
        - name: LOG_LEVEL
          value: debug

### List Environment Variables in a Multi-Container Template

    kubectl set env deployment/env-multi --list

The output identifies the container associated with each set of environment variables.

For example:

    # Deployment env-multi, container web
    LOG_LEVEL=info

    # Deployment env-multi, container logger
    LOG_LEVEL=debug

### Set an Environment Variable on All Containers

Without `--containers`, `kubectl set env` updates all containers in the Pod template:

    kubectl set env deployment/env-multi ENVIRONMENT=production

If the Pod template contains `web` and `logger`, both containers receive:

    ENVIRONMENT=production

### Set an Environment Variable on One Container

Use `--containers` to target a specific container:

    kubectl set env deployment/env-multi \
      --containers=web \
      WEB_MODE=frontend

Only the `web` container receives:

    WEB_MODE=frontend

The `logger` container does not receive it.

`--containers` uses a container name/pattern. It is not a comma-separated list of container names.

For example, this does not mean "web and logger":

    --containers=web,logger

To target individual containers explicitly, run separate commands or edit the manifest directly.

### Inspect a Specific Container

With `kubectl exec`, use `-c` to select a container in a multi-container Pod:

    kubectl exec <pod-name> -c web -- env

and:

    kubectl exec <pod-name> -c logger -- env

Without `-c`, kubectl selects a default container if one is configured; otherwise it normally uses the first container in the Pod.

When working with multi-container Pods, explicitly using `-c` avoids ambiguity.

`kubectl exec` does not provide an `--all-containers` option.

To inspect both containers, execute the command separately:

    kubectl exec <pod-name> -c web -- env
    kubectl exec <pod-name> -c logger -- env

`--all-containers` is available with commands such as `kubectl logs`, for example:

    kubectl logs <pod-name> --all-containers

## Useful Commands

Create the Pod:

    kubectl apply -f 02-company-pod.yaml

View the Pod:

    kubectl get pod company-pod

View container output:

    kubectl logs company-pod

List all environment variables inside the container:

    kubectl exec company-pod -- env

Filter specific variables:

    kubectl exec company-pod -- env | grep -E '^(COMPANY|APP_NAME|APP_VERSION|LOG_LEVEL)='

Replace the Pod after changing the manifest:

    kubectl replace --force -f 02-company-pod.yaml

List environment variables configured in a Deployment:

    kubectl set env deployment/env-repair --list

Set or update an environment variable:

    kubectl set env deployment/env-repair LOG_LEVEL=info

Remove an environment variable:

    kubectl set env deployment/env-repair LOG_LEVEL-

Set an environment variable for one container:

    kubectl set env deployment/env-multi --containers=web WEB_MODE=frontend

Inspect one container in a multi-container Pod:

    kubectl exec <pod-name> -c web -- env

## CKAD Quick Reference

    # List configured environment variables
    kubectl set env deployment/app --list

    # Set/update a variable on all containers
    kubectl set env deployment/app KEY=value

    # Remove a variable
    kubectl set env deployment/app KEY-

    # Set a variable on one container
    kubectl set env deployment/app --containers=web KEY=value

    # Inspect the actual environment in a running container
    kubectl exec <pod-name> -- env

    # Inspect a specific container
    kubectl exec <pod-name> -c web -- env

## Key Points

- `env` belongs to an individual container.
- Environment variable values are strings.
- Kubernetes provides environment variables to the container.
- The shell performs variable expansion such as `$APP_NAME`.
- Use `sh -c` when shell functionality is required.
- Use `|` for readable multi-line shell scripts.
- Changing environment variables in a standalone Pod normally requires recreating the Pod.
- `kubectl set env` can inspect, set, update, and remove environment variables on workloads.
- `kubectl set env deployment/app --list` inspects the configured Pod template.
- `kubectl exec <pod-name> -- env` inspects the actual environment inside a running container.
- Changing a Deployment's Pod template triggers a rollout.
- Without `--containers`, `kubectl set env` applies the variable to all containers in the Pod template.
- Use `--containers=<name>` to target a specific container.
- In a multi-container Pod, use `kubectl exec -c <container>` to explicitly select the container.
- `kubectl exec` does not support `--all-containers`.