# Troubleshooting Pods and Containers

How to identify root cause of failures.

## When create a new pod object

- `ImagePullBackOff` or `ErrImagePull`: image could not be pulled from the
  registry.
  - Verify image name
  - Verify image exists
  - Verify network access from node to registry
  - Ensure auth is set properly
- `CrashLoopBackOff`: app or command run in container and crashes
  - Check the command is executed properly
- `CreateContainerConfigError`:  `configmap` or `secret` by container cannot be
  found
  - check correct name of the config obj
  - verify the existence of the config obj in the namespace

## Inspecting Events of a Specific Pod

```bash
k get events
```
Will list the events across the pods. You can also check the logs of the pod.
Go inside the pod with `k exec -it`.

To debug a container that does not expose a shell. We can use:
```
k debug mypod -it --image=busybox
```
this will create an interactive debugging session in pod `mypod` and
immediately attach to it.

You can also do `k cp <namespace>/<pod>:<file> <local path>` to copy files



