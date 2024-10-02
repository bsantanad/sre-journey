# Security Context

Privilege and access control for a pod. For example This container needs to run
with a non-root user.

These are defined in `spec.securityContext` or
`spec.containers[].securityContext`.

Some attributes on the Pod and container level are the same. Container-level
ones take precedence.

## Defining Container Security

There is a Security Context API

- PodSecurityContext: pod level security attr.
- SecurityContext: container level security attr.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busy-security-context
  name: busy-security-context
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - args:
    - sleep
    - "1h"
    image: busybox:1.28
    name: busy-security-context
    volumeMounts:
    - mountPath: /data/test
      name: volume-empty
    resources: {}
  volumes:
  - name: volume-empty
    emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
