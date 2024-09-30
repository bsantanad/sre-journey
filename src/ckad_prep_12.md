# Probes

It helps with detecting and correcting application issues.

The probes will run a _miniprocess_ that check for runtime conditions. For
example, an http get request. For other apps open a tcp socket, or stuff
similar like that.

The verification method is exec by `kubelet`.

## Readiness Probe

_Is the app ready to server requests?_

This checks if the app is up and running, the kubelet will check the config
of the readiness probe periodically.


## Liveness Probe

_Does the app still function without errors?_

Check for deadlocks, memoryleaks. This will restart the container and try to
get the app into a running state

## Startup Probe

Legacy app may need longer to start. Hold off on starting the liveness
probe.

## Health Verification Methods

We can choose from different verification methods, in any type of probe

- custom command: `exec.command`, execute a command inside the container and
  check the exit code
- http get request: `httpGet`, make a request and check if it is between
  200-399
- `tcp` socket connection: `tcpSocket`, open socket at port
- `gRPC`:  `grpc`

You can customize each of these a bit more.

- `initalDelaySeconds`
- `periodSeconds`
- `timeoutSeconds`
- `successThreshold` - number of successful attempts until probe is consider
  successful after a failure `failureThreshold`.
- `terminationGracePeriodSeconds` - grace period before forcing a container
  stop upon failure.

Probes are not different objects. You need to add the config inside the
container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness # here we start
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
```
