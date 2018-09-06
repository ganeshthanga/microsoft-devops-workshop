# Liveness and Readiness Probes

Each container in a pod, can define a livenessProbe and a readinessProbe. It is common for containers to have both of these defined in a similar fashion.

Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations.

Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup. In such cases, you don’t want to kill the application, but you don’t want to send it requests either. Kubernetes provides readiness probes to detect and mitigate these situations.

In addition to the items above, Kubernetes uses these probes to perform rolling updates with no down time. For instance, if you had 3 instances of your application, and a rolling update strategy to replace 1 at a time, the probes let Kubernetes know that it should wait before moving on to replacing the next instance.

## Command Based

The following probe type is using a script, file, or command within your container to determine the status. If the command exits non-zero, the probe fails.

```
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

```
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Request Based

The following probe type is making a request into the container. If the request returns a non-200 level status code, the probe fails.

```
readinessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: X-Custom-Header
      value: Awesome
  initialDelaySeconds: 3
  periodSeconds: 3
```

```
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: X-Custom-Header
      value: Awesome
  initialDelaySeconds: 3
  periodSeconds: 3
```

## TCP Based

The following probe type is making a TCP connection to the container, on the port specified.

```
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```

```
readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```