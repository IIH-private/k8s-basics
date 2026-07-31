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

## Key Points

- `env` belongs to an individual container.
- Environment variable values are strings.
- Kubernetes provides the variables to the container.
- The shell performs variable expansion such as `$APP_NAME`.
- Use `sh -c` when shell functionality is required.
- Use `|` for readable multi-line shell scripts.
- Changing Pod environment variables normally requires recreating the Pod.